# K8S Gateway API Routes

tags: #objects #network

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [K8S Gateway API Routes](#k8s-gateway-api-routes)
  - [Routes Overview](#routes-overview)

<!-- /code_chunk_output -->

---

## Routes Overview

define how requests are routed to services

- [[HTTPRoute]]
- [[TCPRoute]]
- [[UDPRoute]]
- [[TLSRoute]]
- [[GRCRoute]]

| Route Type | OSI Layer | Routing Discriminator             | TLS Support                 | Purpose                                           |
| ---------- | --------- | --------------------------------- | --------------------------- | ------------------------------------------------- |
| HTTPRoute  | Layer 7   | Host, Path, Headers, Query params | Yes (Terminate/Passthrough) | HTTP/HTTPS traffic routing with advanced matching |
| TLSRoute   | Layer 4-7 | SNI (Server Name Indication)      | Yes (Passthrough)           | TLS traffic routing based on SNI                  |
| TCPRoute   | Layer 4   | Destination Port                  | Yes (Passthrough)           | TCP traffic routing for non-HTTP protocols        |
| UDPRoute   | Layer 4   | Destination Port                  | No                          | UDP traffic routing for stateless protocols       |
| GRPCRoute  | Layer 7   | Service, Method, Headers          | Yes (Terminate/Passthrough) | gRPC service routing with method-level matching   |
