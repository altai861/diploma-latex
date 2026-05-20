# System Overview Architecture

Source image: `Figures/chapter3/pcloud-architecture.drawio (2).png`

Caption in thesis: `Системийн ерөнхий архитектур`

## Purpose

This diagram explains the high-level architecture of a self-hosted personal cloud storage system called PCloud. The main message is that each user owns a separate PCloud device, files stay on that device, and a relay server is used only when the user connects from outside the local network.

## Visual Structure

Use a wide landscape architecture diagram. Place five stick-figure users across the top row, labeled `Хэрэглэгч 1`, `Хэрэглэгч 2`, `Хэрэглэгч 3`, `Хэрэглэгч 4`, and `Хэрэглэгч 5`. Place five small Raspberry Pi style devices across the bottom row, aligned under the users, labeled `PCloud 1`, `PCloud 2`, `PCloud 3`, `PCloud 4`, and `PCloud5`.

On the right side, place a server rack icon labeled `Дамжуулагч сервер`. Above the server, add the text `Интернет ашиглан холбогдоно` to show that this component is used for internet-based remote access.

Draw a large dashed rectangular area around the central part of the diagram to represent the local/direct connection zone. Inside this zone, show vertical solid arrows from each user to their own PCloud device. Label the direct local connection area with `Шууд холболт`. These arrows should communicate LAN-first behavior: when a user is on the same local network, the client connects directly to the user's own PCloud server without using the relay server.

Also draw colored dashed paths from each user to the relay server and from the relay server back to the matching PCloud device. These dashed paths represent remote access through the internet. Use a different color for each user's path so the viewer can follow each user-device pair clearly. Add the label `Дамжуулагч сервер ашиглаж холболт хийх` along the dashed relay paths.

## Required Components

- Five users at the top.
- Five owned PCloud devices at the bottom.
- One relay server on the right.
- Solid vertical arrows for direct local connections.
- Colored dashed arrows for relay-based remote connections.
- A visual distinction between LAN-first/direct access and internet/relay access.

## Core Message

The system is decentralized by ownership: each user has their own PCloud device. The relay server does not store files or user data. It only helps route remote connections to the correct device when direct local access is not possible.

## Codebase Alignment Corrections

These corrections should be applied when regenerating the diagram from the current implementation:

- The PCloud device should be shown as the actual Rust `pcloud-server` application, not only as a generic storage box.
- Local access uses the client web/API port, default `0.0.0.0:8080`, and the server advertises `_pcloud._tcp.local.` through mDNS/DNS-SD when enabled.
- The relay path should be labeled more specifically as `/d/{device_id}/...`, because the relay forwards everything after `/d/{device_id}` to the local PCloud server.
- The relay connection from each PCloud device is optional and disabled by default unless `PCLOUD_RELAY_ENABLED=true`.
- The device initiates the relay tunnel using `PCLOUD_DEVICE_ID` and `PCLOUD_RELAY_TOKEN`; remote users do not connect directly through home-router port forwarding.
- The current relay server is a single-node MVP with an in-memory active device map. Do not draw a distributed relay cluster, Redis registry, or load balancer unless showing a future extension.
- The workspace contains the Rust server, Angular web client/admin setup apps, and Rust relay server. There is no iOS app implementation in this repository, so mobile/iOS clients should be drawn as planned/external clients if they are included.

## Copy-Ready AI Prompt

Create a clean technical architecture diagram for a self-hosted personal cloud storage system named PCloud. Use a wide landscape canvas with a white background and simple flat diagram style.

Across the top row, draw five stick-figure users labeled `Хэрэглэгч 1`, `Хэрэглэгч 2`, `Хэрэглэгч 3`, `Хэрэглэгч 4`, and `Хэрэглэгч 5`. Across the bottom row, aligned under them, draw five Raspberry Pi style devices labeled `PCloud 1`, `PCloud 2`, `PCloud 3`, `PCloud 4`, and `PCloud5`. On the right side, draw a server rack labeled `Дамжуулагч сервер`, with the text `Интернет ашиглан холбогдоно` above it.

Show solid vertical arrows from each user directly down to their own PCloud device. Label this direct connection area `Шууд холболт`. These solid arrows represent local network LAN-first access to the Rust `pcloud-server` client API/web app, discovered through mDNS/DNS-SD (`_pcloud._tcp.local.`) or reached by local IP address.

Also show colored dashed arrows from each user to the relay server and from the relay server to that user's matching PCloud device. Label the remote public path `/d/{device_id}/...` and label the dashed paths `Дамжуулагч сервер ашиглаж холболт хийх`. Use one color per user-device pair so the paths are easy to trace. Make clear that each PCloud device opens an optional outbound WebSocket tunnel to the relay using `device_id` and a relay token; remote access does not require inbound port forwarding on the home router. The relay server is only a connection mediator for remote internet access, while actual files, metadata, accounts, and permissions remain on the user's own PCloud device.

If a mobile or iOS client icon is included, mark it as `Planned / external client`, because the current repository contains the Angular web client and admin setup UI, but no iOS implementation.

Keep the diagram balanced, readable, thesis-appropriate, and not overly decorative. Use consistent spacing, clear arrowheads, and legible Mongolian labels.

## Avoid

- Do not show the relay server as central file storage.
- Do not merge all users into one shared cloud server.
- Do not make the relay path look like the primary access path.
- Do not show Redis, load balancers, relay clustering, or central databases in the current system overview.
- Do not present the iOS client as implemented by this repository.
- Do not use a marketing-style hero illustration; keep it as an academic architecture diagram.
