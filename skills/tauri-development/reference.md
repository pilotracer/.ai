# tauri-development — reference

Supplement to `skill.md`. Detailed guidance, code patterns, architecture diagrams, and security patterns for Tauri development.

---

## How to invoke

### Cursor

```
@tauri-development
@tauri-development status
@tauri-development help
```

### Claude Code / opencode / Codex

```
Follow .ai/skills/tauri-development/skill.md — status.
Read-only: check Tauri project structure and configuration.
```

```
Follow .ai/skills/tauri-development/skill.md — help.
Domain guidance for Tauri development conventions.
```

---

## Architecture patterns

### 1. Shell strategy (webview redirection)

When a Tauri app wraps an external web service (Docker backend, local dev server), the recommended pattern is **webview redirection**:

```
Tauri App → setup → spawn bridge server (optional)
                  → start Docker stack (optional)
                  → redirect main webview to http://localhost:<SERVICE_PORT>
```

**Pros vs iframe:**
- Full browser features (cookies, storage, service workers) available to the web app
- No CORS issues — same-origin requests within webview
- Native navigation controls work naturally
- No DOM isolation boundaries to manage

**When to use iframe instead:**
- Need to overlay native Tauri UI on top of the external content (e.g., custom title bar, sidebar)
- Strict isolation required between the web app and Tauri chrome

### 2. API bridge pattern (Tauri → Docker filesystem)

When Docker containers need dynamic access to host filesystem paths (not static volume mounts):

```
┌─────────────────────────────────────────┐
│ Tauri App                                │
│  ┌────────────┐   ┌───────────────────┐  │
│  │ React UI   │←──│ Axum HTTP Server  │  │
│  │ (Webview)  │   │ (127.0.0.1:3737)  │  │
│  └────────────┘   └────────┬──────────┘  │
└─────────────────────────────┼────────────┘
                              │ host.docker.internal:3737
┌─────────────────────────────┼────────────┐
│ Docker Container            │            │
│  ┌──────────────────────┐   │            │
│  │ FileBridgeClient     │───┘            │
│  │ GET /api/file/read   │                │
│  │ POST /api/file/list  │                │
│  └──────────────────────┘                │
└──────────────────────────────────────────┘
```

**Security constraints:**
- Bind HTTP server to `127.0.0.1` only
- Each authorized folder gets a unique UUID; container requests use `path_id` + `relative_path`, never absolute paths
- Validate every request: `canonicalize(base/uuid + relative).starts_with(canonicalize(base))`
- User authorizes folders via native OS dialog (not webview file input)
- No open access — only explicitly authorized paths are served

### 3. Event-driven Docker lifecycle

When Tauri manages Docker stack lifecycle with real-time UI feedback:

```
Rust (src-tauri):
  docker::start_stack()          → pipes stdout/stderr line by line
  app_handle.emit("docker-event-log", line)
  app_handle.emit("docker-status", "running" | "stopped" | "error")

Frontend:
  listen("docker-event-log", (event) => appendLog(event.payload))
  listen("docker-status", (event) => updateStatus(event.payload))
```

**Tauri v2 event APIs:**
- Rust emit: `app_handle.emit("event-name", payload)?;`
- Frontend listen: `import { listen } from '@tauri-apps/api/event'; await listen('event-name', callback);`
- Frontend emit: `import { emit } from '@tauri-apps/api/event'; await emit('event-name', payload);`

---

## Configuration patterns

### Tauri v2 capabilities example

```json
// src-tauri/capabilities/default.json
{
  "identifier": "default",
  "description": "Default capabilities",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "dialog:default",
    "fs:default",
    "shell:default"
  ]
}
```

Enable only what the feature needs. For a strict RAG app with no shell access:

```json
{
  "identifier": "rag-app",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "dialog:allow-open",
    "fs:allow-read",
    "fs:allow-exists"
  ]
}
```

### Rust command with validation

```rust
#[derive(Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct ReadFileRequest {
    pub path_id: String,
    pub relative_path: String,
}

#[derive(Serialize)]
pub struct ReadFileResponse {
    pub content: String,
    pub mime_type: String,
}

#[tauri::command]
async fn read_file(
    state: State<'_, AppState>,
    request: ReadFileRequest,
) -> Result<ReadFileResponse, String> {
    // Validate path_id exists
    let base = state.authorized_paths.get(&request.path_id)
        .ok_or("path_id not authorized")?;

    // Prevent path traversal
    let full_path = base.join(&request.relative_path);
    let canonical = full_path.canonicalize()
        .map_err(|e| format!("invalid path: {}", e))?;

    if !canonical.starts_with(base) {
        return Err("path traversal detected".into());
    }

    // Read and return
    let content = std::fs::read_to_string(&canonical)
        .map_err(|e| format!("read error: {}", e))?;

    Ok(ReadFileResponse {
        content,
        mime_type: mime_guess::from_path(&canonical)
            .first_or_octet_stream()
            .to_string(),
    })
}
```

### Managed state setup (Tauri v2)

```rust
use std::sync::Arc;
use tokio::sync::RwLock;

pub struct AppState {
    pub authorized_paths: Arc<RwLock<HashMap<String, PathBuf>>>,
    pub docker_running: Arc<atomic::AtomicBool>,
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .manage(AppState {
            authorized_paths: Arc::new(RwLock::new(HashMap::new())),
            docker_running: Arc::new(atomic::AtomicBool::new(false)),
        })
        .invoke_handler(tauri::generate_handler![read_file, authorize_folder])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

---

## Tauri v1 vs v2 quick reference

| Area | Tauri v1 | Tauri v2 |
|------|----------|----------|
| **Package** | `tauri` 1.x | `tauri` 2.x |
| **Frontend API** | `@tauri-apps/api` v1 | `@tauri-apps/api` v2 |
| **Event emit (Rust)** | `window.emit(...)` or `app_handle.emit_all(...)` | `app_handle.emit("name", payload)?` (via `Emitter` trait) |
| **Event listen (Rust)** | `window.listen(...)` | `app_handle.listen("name", callback)` (via `Listener` trait) |
| **Commands** | `#[tauri::command]` with `InvokeError` | Same, but `Result<T, E>` with `E: Into<tauri::ipc::InvokeError>` |
| **Capabilities** | `tauri.conf.json` > `allowlist` | `src-tauri/capabilities/*.json` (separate files) |
| **Setup hook** | `|app| Ok(())` closure | Same |
| **State** | `app.manage(...)` / `State<'_, T>` | Same |

---

## Common pitfalls

| Pitfall | Resolution |
|---------|------------|
| Binding HTTP server to `0.0.0.0` exposes port to LAN | Always bind to `127.0.0.1` for internal bridge servers |
| Embedding sensitive data in Rust command results | Never return tokens, keys, or internal paths to the frontend |
| Polling from frontend instead of events | Use `emit`/`listen` for full-duplex async communication |
| Mixing Tauri v1 and v2 APIs | Check `Cargo.toml` for `tauri` version before writing code |
| Missing `tauri::Emitter` import in v2 | Explicitly import `use tauri::Emitter;` and `use tauri::Listener;` |
| Hardcoding `host.docker.internal` | Write Dockerfiles with `extra_hosts` for Linux compatibility |

---

## Further reading

- [Tauri v2 documentation](https://v2.tauri.app/)
- [Tauri v1 documentation](https://v1.tauri.app/)
- Security: [Tauri Security Guidelines](https://v2.tauri.app/start/security/)
- Capabilities: [Tauri v2 Capabilities](https://v2.tauri.app/develop/capabilities/)
