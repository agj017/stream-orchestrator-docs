# 스트림 상태 모델

## 스트림 수명주기

`stream`은 오케스트레이터가 추적하는 핵심 리소스입니다.

### 상태

- `PENDING`: API가 요청을 수락했고, 데이터베이스에 저장되었으며, 오케스트레이션을 기다리는 상태입니다.
- `PROVISIONING`: 컨트롤러가 Kubernetes 또는 MediaMTX에 원하는 상태를 적용하는 중입니다.
- `RUNNING`: 스트림이 활성화되어 트래픽을 제공하는 상태입니다.
- `FAILED`: 프로비저닝 또는 상태 조정이 재시도 후 실패한 상태입니다.
- `STOPPING`: 중지 요청이 수락되었고 종료가 진행 중입니다.
- `STOPPED`: 스트림이 정상적으로 종료된 상태입니다.
- `DELETED`: 논리적으로 삭제되어 기본 운영 목록에서 숨겨진 상태입니다.

### 허용되는 상태 전이

- `PENDING -> PROVISIONING`
- `PROVISIONING -> RUNNING`
- `PROVISIONING -> FAILED`
- `FAILED -> PROVISIONING` 재시도
- `RUNNING -> STOPPING`
- `STOPPING -> STOPPED`
- `STOPPING -> FAILED`
- `PENDING -> DELETED`
- `FAILED -> DELETED`
- `STOPPED -> DELETED`

### 전이 규칙

- API는 스트림을 `PENDING` 상태로 생성합니다.
- API는 `POST /v1/streams/{streamId}/stop`으로 중지 요청을 받습니다.
- API는 `PENDING`, `FAILED`, `STOPPED` 상태의 스트림에 대해서만 `DELETE /v1/streams/{streamId}` 삭제 요청을 받습니다.
- `PROVISIONING` 이후의 상태 전이는 컨트롤러가 소유합니다.
- `DELETED` 스트림은 기본 `GET /v1/streams` 결과에서 제외됩니다.
- 모든 상태 전이는 `updated_at`을 갱신해야 합니다.
- `FAILED` 상태에는 운영 가시성을 위해 실패 이유가 포함되어야 합니다.
