# Stream State Model

## Stream Lifecycle

`stream` is the primary resource lifecycle tracked by the orchestrator.

### States

- `PENDING`: accepted by API, persisted, and waiting for orchestration.
- `PROVISIONING`: controller is applying desired state to Kubernetes/MediaMTX.
- `RUNNING`: stream is active and serving traffic.
- `FAILED`: provisioning or reconciliation failed after retries.
- `STOPPING`: stop request accepted and termination is in progress.
- `STOPPED`: stream is terminated successfully.
- `DELETED`: stream is logically deleted and hidden from default operator lists.

### Allowed Transitions

- `PENDING -> PROVISIONING`
- `PROVISIONING -> RUNNING`
- `PROVISIONING -> FAILED`
- `FAILED -> PROVISIONING` (retry)
- `RUNNING -> STOPPING`
- `STOPPING -> STOPPED`
- `STOPPING -> FAILED`
- `PENDING -> DELETED`
- `FAILED -> DELETED`
- `STOPPED -> DELETED`

### Transition Notes

- API creates streams in `PENDING`.
- API accepts stop requests through `POST /v1/streams/{streamId}/stop`.
- API accepts delete requests through `DELETE /v1/streams/{streamId}` only for `PENDING`, `FAILED`, and `STOPPED` streams.
- Controller owns transitions from `PROVISIONING` onward.
- `DELETED` streams are excluded from `GET /v1/streams` by default.
- Every transition should update `updated_at`.
- `FAILED` must include an error reason for operations visibility.
