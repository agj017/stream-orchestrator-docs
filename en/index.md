# Stream Orchestrator

Stream Orchestrator is a video stream orchestration platform built with Go, Kubernetes, MediaMTX, RabbitMQ, PostgreSQL, and Prometheus.

It accepts stream lifecycle requests, assigns streams to healthy MediaMTX instances, configures MediaMTX paths dynamically, and exposes HLS playback through a gateway.

## Documents

- [Architecture](architecture.md)
- [Stream State Model](state-model.md)
- [GitHub Pages Deployment](deployment/github-pages.md)
