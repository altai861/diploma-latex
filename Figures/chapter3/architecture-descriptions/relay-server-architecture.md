# Relay Server Architecture

Source image: `Figures/chapter3/pcloud-architecture-relay-server.drawio.png`

Caption in thesis: `Дамжуулагч серверийн архитектур`

## Purpose

This diagram explains the relay server used when a remote client needs to access a PCloud device behind a home router or NAT. The relay server accepts remote HTTP requests containing a `device_id`, maps that device identifier to an active outbound WebSocket tunnel from the PCloud device, forwards the request through the tunnel, and returns the HTTP response to the remote client.

## Visual Structure

Use a wide landscape architecture diagram with three main zones:

- Left: a remote client.
- Center: the relay server.
- Right: a user's PCloud device running a relay client.

On the left, draw a laptop/client icon labeled `Remote Client`. Draw an arrow from the remote client to the relay server labeled `HTTP request with device_id`. Draw a return arrow from the relay server to the remote client labeled `HTTP response`.

In the center, draw a server rack labeled `Дамжуулагч сервер`. Overlay or place inside the server five stacked internal components:

1. `Relay HTTP Router`
2. `Proxy Layer`
3. `Device Tunnel Manager`
4. `Device Registry`
5. `Pending Request Map`

On the right, draw a Raspberry Pi style device labeled `PCloud төхөөрөмж дээрх Relay Client (device_id)`. Between the relay server and the PCloud device, draw a persistent connection labeled `Outbound WebSocket Tunnel`. This tunnel should clearly originate from the PCloud device toward the relay server, emphasizing that the user's home router does not need inbound port forwarding.

Also draw request and response arrows through the tunnel:

- From relay server to PCloud device: `Proxy HTTP request (device_id)`
- From PCloud device back to relay server: `Proxy HTTP response`

## Required Components

- Remote client.
- Relay server.
- PCloud device with relay client.
- HTTP request/response arrows between remote client and relay server.
- Outbound WebSocket tunnel between PCloud device and relay server.
- Proxy request/response arrows over the tunnel.
- Internal relay server components: router, proxy layer, tunnel manager, registry, and pending request map.

## Core Message

The relay server is a routing and tunneling component, not a storage component. It keeps connection state such as active device tunnels, a device registry, and pending request mappings. File metadata, user accounts, permissions, and real file bytes remain on the user's PCloud device.

## Codebase Alignment Corrections

These corrections should be applied when regenerating the diagram from the current implementation:

- The relay backend is a Rust Axum server, default bind `0.0.0.0:7070`.
- The implemented routes are `/health`, `/api/devices/{device_id}/status`, `/api/relay/device/connect`, `/d/{device_id}`, `/d/{device_id}/`, and `/d/{device_id}/*path`.
- The PCloud device opens the WebSocket tunnel to `/api/relay/device/connect?device_id=...&token=...`.
- Device tunnel authentication currently uses one shared token from `PCLOUD_RELAY_TOKEN`. This is an MVP design, not per-device public/private key authentication.
- Remote browser/mobile traffic enters through `/d/{device_id}/...`; the relay strips the `/d/{device_id}` prefix before forwarding the path to the local server.
- The relay stores active device connections in memory as `device_id -> DeviceConnection`. It does not have PostgreSQL, Redis, or persistent device storage in the current implementation.
- The pending request map is per device connection. It maps `request_id` to response channels for active proxied requests.
- Request and response bodies are streamed through JSON WebSocket messages as base64 chunks of up to 64 KiB, with bounded queues. The relay should not be drawn as buffering whole files.
- The relay enforces a maximum request body size through `PCLOUD_RELAY_MAX_BODY_BYTES`, default 5 GiB, and response wait timeout through `PCLOUD_RELAY_REQUEST_TIMEOUT_SECONDS`, default 3600 seconds.
- HTTPS/WSS in production is provided by Caddy in the Docker compose deployment. The Rust relay itself listens as HTTP/WS behind that reverse proxy.
- Horizontal scaling with Redis or sticky load balancing is only a future design idea in the README, not current code.

## Copy-Ready AI Prompt

Create a clean wide technical architecture diagram for a PCloud relay server. Use a white background and academic thesis diagram style.

Arrange the diagram from left to right. On the left, draw a laptop icon labeled `Remote Client`. In the center, draw a server rack labeled `Дамжуулагч сервер`. On the right, draw a Raspberry Pi style device labeled `PCloud төхөөрөмж дээрх Relay Client (device_id)`.

Inside or over the relay server, show five stacked rounded rectangles with these labels: `Rust Axum Router`, `Proxy Layer`, `Device WebSocket Handler`, `In-memory Device Registry`, and `Per-device Pending Request Map`.

Draw an arrow from the remote client to the relay server labeled `HTTP request: /d/{device_id}/...`. Draw a return arrow from the relay server to the remote client labeled `HTTP response`. Add a small route callout listing `/health`, `/api/devices/{device_id}/status`, `/api/relay/device/connect`, and `/d/{device_id}/*path`.

Between the relay server and the PCloud device, show a persistent tunnel labeled `Outbound WebSocket Tunnel: /api/relay/device/connect?device_id=...&token=...`. Make the tunnel direction visually clear: the PCloud device initiates the outbound connection to the public relay server using a shared relay token. Over this tunnel, draw a request arrow from the relay server to the PCloud device labeled `request_start + body chunks`, and a response arrow from the PCloud device back to the relay server labeled `response_start + body chunks`. Add a small note: `64 KiB base64 chunks, no whole-file buffering`.

The diagram must communicate that the relay server only mediates remote access. It does not store user files, user accounts, permissions, or metadata. It stores only active WebSocket connections and pending request state in memory. The user's actual data remains on the PCloud device.

Use clear arrowheads, consistent spacing, readable labels, and a restrained professional color palette.

## Avoid

- Do not show a database or file storage attached to the relay server.
- Do not show inbound port forwarding from the internet into the home router.
- Do not make the relay server look like the main application server.
- Do not add unrelated cloud services or authentication providers.
- Do not show Redis, a distributed registry, or a load balancer as current implementation.
- Do not imply per-device key authentication; the current MVP uses a shared token.
- Do not show the relay buffering full uploads or downloads.
