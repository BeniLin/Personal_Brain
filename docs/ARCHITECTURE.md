# Personal Brain — Architecture

## 1. W1 目標架構

```text
Galaxy Watch 7
  Wear OS Watch App
  - Kotlin
  - Jetpack Compose for Wear OS
  - Speech Recognition API
  - durable local capture storage
        |
        | Wear OS Data Layer API (DataClient / DataItem)
        | Bluetooth when available; Google network relay may be used
        v
Android Phone
  Companion App
  - Kotlin / Jetpack Compose
  - Room local database
  - WorkManager upload/retry
        |
        | HTTPS over Tailscale
        | Tailscale Serve provides HTTPS endpoint/certificate
        v
Synology DS223
  Container Manager
  - FastAPI
  - SQLite
  - persistent NAS volume
```

## 2. Watch application
Responsibilities:
- Provide a low-friction voice capture interaction.
- Use speech recognition and obtain the final recognized text.
- Automatically submit after recognition completes; no additional confirmation step.
- Persist each capture locally before attempting synchronization.
- Retain unsynchronized captures across app/process/device interruptions.
- Send the same `capture_id` on every retry.
- Track synchronization state accurately.

Suggested state concepts (implementation may refine but must preserve semantics):
- `LOCAL_PENDING`: captured and durably stored on Watch, not yet acknowledged by Phone.
- `PHONE_ACKNOWLEDGED`: Phone has durably stored the capture in Room.
- `NAS_CONFIRMED`: NAS has confirmed SQLite commit and the confirmation has propagated back.

Do not show NAS success merely because speech recognition completed or the Phone received the item.

## 3. Watch-to-Phone transport
Use Wear OS Data Layer API, specifically DataClient / DataItem for the agreed relay-based design.

Confirmed choice: **Accept relay**.
- Bluetooth may be used when available.
- If Bluetooth is unavailable, Google network relay may be used.
- Data Layer communication is end-to-end encrypted, but it may transit Google infrastructure and connection metadata may exist.
- Do not claim that transport is Bluetooth-only or that no metadata is exposed.
- Do not implement custom Bluetooth RFCOMM as an alternative for W1.

Data Layer is transport/synchronization, not durable storage. The Watch must retain its own persistent copy until Phone persistence is confirmed.

Phone must write the capture to Room before acknowledging it to Watch.

## 4. Phone companion application
Responsibilities:
- Receive Watch Data Layer events.
- Validate payload and `capture_id`.
- Persist to Room before ACK.
- Use WorkManager for durable background upload and retry.
- Upload to NAS over HTTPS through Tailscale Serve.
- Preserve the same `capture_id` across retries.
- Record NAS confirmation only after a successful server response representing committed persistence.

Room is the Phone's durable local copy. Do not rely on in-memory queues or Data Layer alone for durability.

WorkManager is responsible for retryable background work and must account for Android background execution constraints and Doze behavior.

## 5. NAS backend
Target: Synology DS223, deployed as a single lightweight application container.

W1 technology:
- FastAPI
- SQLite
- One application container
- Persistent NAS volume mounted for SQLite data

Constraints:
- DS223 has 2GB RAM; keep resource use modest.
- Do not add PostgreSQL, Redis, a separate worker service, or other infrastructure in W1 without explicit approval.
- Do not expose the API to the public Internet.
- Do not configure router port forwarding.
- Keep the existing practice Verdaccio container independent.
- SQLite database must live on a persistent NAS-mounted volume, not only in the container writable layer.

## 6. HTTPS and network
Phone-to-NAS traffic:
- HTTPS over Tailscale.
- Use Tailscale Serve for the private HTTPS endpoint/certificate.
- Do not use Tailscale Funnel; Funnel would make the service publicly reachable.
- Confirm the actual Serve forwarding configuration and Tailscale ACL/access policy during deployment.
- Do not assume a hostname or port that has not been configured and verified.

## 7. Capture identity and idempotency
Each capture receives a UUID `capture_id` at creation time.

The ID must remain unchanged across:
- Watch retries
- Data Layer redelivery
- Phone WorkManager retries
- HTTP retries

Persistence layers:
- Watch local storage: unique `capture_id`.
- Phone Room: primary/unique key on `capture_id`.
- NAS SQLite: primary/unique key on `capture_id`.

Repeated delivery of the same capture must not create duplicate rows. Backend behavior should be idempotent and return a deterministic result for an already stored `capture_id`.

## 8. Raw content preservation
- Store the original speech-recognition result exactly as received, subject only to necessary transport encoding.
- Do not silently normalize, rewrite, correct spelling, translate, or replace the original text.
- Any future AI correction must be stored separately as a derived suggestion or field.
- W1 does not upload or retain original audio.

## 9. API contract — implementation details to verify
The W1 API must support:
- Submitting a capture with `capture_id`, recognized text, and required capture metadata.
- Idempotent handling of duplicate `capture_id`.
- A success response only after the SQLite transaction commits.
- A way for Phone to reconcile whether a capture is already stored, if required by the retry/recovery design.

The exact endpoint path, JSON schema, authentication mechanism, timestamp format, and response schema must be defined and reviewed during implementation. Do not invent these as already-finalized decisions.

## 10. Reliability and status semantics
The end-to-end flow must distinguish:
1. Speech recognition completed.
2. Capture saved on Watch.
3. Capture saved on Phone.
4. Capture committed on NAS.

Only state 4 qualifies as “NAS 已保存”.

Failures must not cause data loss:
- Watch retains unacknowledged data.
- Phone retains data after Watch ACK.
- Phone retries NAS upload with the same ID.
- Duplicate upload is safe.
- Restarting either app or container must not silently discard persisted captures.
