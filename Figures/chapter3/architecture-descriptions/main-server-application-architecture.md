# Main Server Application Architecture

Source image: `Figures/chapter3/pcloud-architecture-main-server.drawio.png`

Caption in thesis: `Үндсэн сервер аппликейшний архитектур`

## Purpose

This diagram describes the internal architecture of the main PCloud server application running on the user's own device, such as a Raspberry Pi. It should show the Rust Axum server, embedded Angular web apps, PostgreSQL metadata storage, local filesystem file storage, optional mDNS discovery, and optional relay client tunnel.

## Visual Structure

Use a portrait-oriented architecture diagram. The center of the diagram should be a large Raspberry Pi or single-board computer illustration representing the local PCloud device. Place the label `PCloud Үндсэн сервер апп` under the device.

At the top left, place a laptop or browser icon labeled `Angular Main Web Client`. Near it, place a smaller browser/setup icon labeled `Angular Admin Setup UI`. If a phone icon is included, label it `Planned iOS / Mobile Client` so it is clear that this client is not implemented in the current repository. Draw green arrows labeled `HTTP request` from the clients into the server application.

Inside the PCloud device, stack five horizontal translucent architecture layers from top to bottom:

1. `HTTP API Layer`
   - Subtitle: `Axum routers, Client API + static main app, Setup API + static admin app`
2. `Application Modules`
   - Subtitle: `Setup module, Auth module, Storage module, Admin/User management module, System/status module`
3. `Core Services`
   - Subtitle: `Module service logic: sessions, file/folder operations, permissions, trash, starred, shared, quotas`
4. `Persistence Layer`
   - Subtitle: `PostgreSQL metadata database, Local filesystem storage`
5. `External/Network Support`
   - Subtitle: `mDNS/DNS-SD discovery, optional Relay client tunnel`

Draw vertical downward arrows between the layers to show request processing and dependency flow. The diagram should communicate that the HTTP layer receives client requests, application modules organize features, core services enforce domain rules, persistence stores metadata and file bytes, and external/network support enables local discovery and relay connectivity.

## Required Components

- Web client and admin setup UI.
- iOS client.
- Green HTTP request arrows into the server.
- A PCloud device as the container for the server application.
- Five clearly separated internal layers.
- Downward arrows between internal layers.
- PostgreSQL metadata storage and local filesystem storage represented in the persistence layer.
- mDNS/DNS-SD discovery and relay client tunnel represented as supporting network features.

## Core Message

The main server app is the real owner of the user's data and system logic. It exposes a main client API/web surface and a temporary first-launch admin setup surface, separates features into modules, keeps domain rules in module service logic, stores metadata in PostgreSQL, stores file bytes on the local filesystem, and supports both local discovery and optional remote relay access.

## Codebase Alignment Corrections

These corrections should be applied when regenerating the diagram from the current implementation:

- The backend is a Rust Axum application. Show `Axum Router` or `Rust pcloud-server` in the HTTP layer.
- Static web apps are embedded into the Rust binary with `rust-embed`: `web/dist/main-app` for the main Angular app and `web/dist/admin-launch` for the admin setup app.
- The main client API/web app runs on `PCLOUD_CLIENT_BIND`, default `0.0.0.0:8080`.
- The admin setup API/web app runs on a separate `PCLOUD_ADMIN_BIND`, default `127.0.0.1:9090`, only while the system is not initialized. After initialization, the admin setup server is disabled.
- Admin user management is not in the setup UI after initialization; it is exposed through authenticated client API routes such as `/api/client/admin/users`.
- The implemented module folders are `setup`, `auth`, `storage`, `admin`, and `system`. There is no separate `core` directory; "Core Services" should be shown as service logic inside modules, not as an independent runtime service.
- The session/auth flow uses bearer access tokens backed by the `sessions` table, with refresh token hashes stored in PostgreSQL.
- The storage module includes search, list, upload, download, preview, batch download, create folder, rename, move, trash, restore, permanent delete, starred resources, sharing permissions, and quota checks.
- PostgreSQL stores metadata, users, roles, sessions, settings, folder/file records, permissions, starred/deleted flags, profile image metadata, and audit log rows. The actual file bytes are stored on the local filesystem under `system_settings.storage_root_path`.
- Audit logging exists in the schema, but the current implementation only writes setup initialization audit rows. Do not imply complete audit coverage for every file action unless the code is extended.
- mDNS/DNS-SD is optional but enabled by default unless `PCLOUD_MDNS_ENABLED=false`; it advertises `_pcloud._tcp.local.` with TXT values such as `api_path=/api/client`, `device_id`, `relay_path`, and `relay_base_url`.
- The relay client tunnel is optional and disabled by default unless `PCLOUD_RELAY_ENABLED=true`. It forwards relay messages to the local HTTP server using `PCLOUD_RELAY_LOCAL_BASE_URL`, defaulting to `http://127.0.0.1:{client_port}`.
- There is no iOS implementation in this workspace. If the diagram keeps an iOS icon, label it as planned/future/external rather than implemented.

## Copy-Ready AI Prompt

Create a clean portrait-oriented technical architecture diagram for the main PCloud server application running on a user's own Raspberry Pi or personal device. Use a white background and a professional academic diagram style.

At the top left, draw a laptop/browser icon labeled `Angular Main Web Client`. Beside it, draw a smaller browser/setup icon labeled `Angular Admin Setup UI`. At the top right, optionally draw a smartphone icon labeled `Planned iOS / Mobile Client`. Draw green arrows from these clients toward the server, labeled `HTTP request`.

In the center, draw a large Raspberry Pi style board as the background container. Under it, place the label `PCloud Үндсэн сервер апп`. Inside the board, place five stacked translucent horizontal layers with bold titles and smaller subtitles:

Layer 1: `HTTP API Layer`
Subtitle: `Rust Axum routers, Client API + static main app, Setup API + static admin app`

Layer 2: `Application Modules`
Subtitle: `Setup module, Auth module, Storage module, Admin/User management module, System/status module`

Layer 3: `Core Services`
Subtitle: `Module service logic: bearer sessions, file/folder operations, permissions, trash, starred, shared, quotas`

Layer 4: `Persistence Layer`
Subtitle: `PostgreSQL metadata database, Local filesystem storage`

Layer 5: `External/Network Support`
Subtitle: `mDNS/DNS-SD discovery, optional Relay client tunnel`

Add small callouts near the HTTP layer: `Client bind: 0.0.0.0:8080` and `Admin setup bind: 127.0.0.1:9090, only before initialization`. Add a note that Angular static assets are embedded into the Rust server with `rust-embed`. In the persistence layer, visually separate PostgreSQL metadata from local filesystem file bytes.

Draw simple downward arrows between the internal layers to show request flow and dependency direction. Make the layers readable and aligned. The visual emphasis should be on layered backend architecture, modular responsibilities, local-first storage, and support for both LAN discovery and remote relay connectivity.

Keep the style minimal, structured, and suitable for a university thesis. Use legible text, consistent spacing, and restrained colors.

## Avoid

- Do not make the relay server part of this diagram; this is only the main device server.
- Do not show files stored in a central cloud.
- Do not collapse all layers into one box.
- Do not show admin setup as always active after initialization.
- Do not present the iOS/mobile client as implemented in this repository.
- Do not show a separate independent core-service process; the code has module-level service logic.
- Do not imply full audit logging for every storage action unless the code is extended.
- Do not use decorative gradients or marketing visuals that reduce readability.
