# Stream Orchestrator

Stream Orchestrator is a video stream orchestration platform built with Go, Kubernetes, MediaMTX, RabbitMQ, PostgreSQL, and Prometheus.

It accepts stream lifecycle requests, assigns streams to healthy MediaMTX instances, configures MediaMTX paths dynamically, and exposes HLS playback through a gateway.

## What This Site Covers

- System architecture and component responsibilities.
- Stream lifecycle states and transitions.
- On-premise installation strategy.
- GitHub Pages documentation publishing workflow.

## Core Components

| Component | Responsibility |
| --- | --- |
| `orchestrator-api` | Accepts stream lifecycle API requests and records stream state. |
| `outbox-publisher` | Publishes pending outbox events to RabbitMQ. |
| `stream-provisioner` | Consumes stream events and configures MediaMTX paths. |
| `stream-gateway` | Resolves stream assignments and proxies HLS playback. |
| `stream-scaler` | Plans and applies MediaMTX StatefulSet replica changes. |
| `instance-reconciler` | Keeps stream instance health and metadata synchronized. |
| `dashboard` | Provides an operator UI for stream management and monitoring. |
