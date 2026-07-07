# Stream Orchestrator

Stream Orchestrator는 Go, Kubernetes, MediaMTX, RabbitMQ, PostgreSQL, Prometheus 기반의 비디오 스트림 오케스트레이션 플랫폼입니다.

이 시스템은 스트림 수명주기 요청을 받고, 건강한 MediaMTX 인스턴스에 스트림을 할당하며, MediaMTX path를 동적으로 설정한 뒤 HLS Gateway를 통해 재생 경로를 제공합니다.

## 문서

- [아키텍처](architecture.md)
- [스트림 상태 모델](state-model.md)
- [GitHub Pages 배포](deployment/github-pages.md)
