Cursor: <!-- CURSOR_AGENT_PR_BODY_BEGIN -->
## Summary

Addresses bugs and improvements identified in the codebase audit across security, plugins, Ollama, setup wizard, tools, and GUI server.

### Security
- **Removed `gen.ts`** and the silent auto-spawn on every startup (obfuscated file write/delete behavior).
- GUI server now binds to **`127.0.0.1`** instead of `0.0.0.0`.

### Ollama
- Normalize settings values (`system` → `existing`) via `normalizeOllamaMode()`.
- Track minimal Ollama PID and kill only the managed instance (no broad `pkill ollama`).
- Cross-platform binary paths; minimal auto-download limited to Windows with a clear error on other platforms.

### Plugins
- Featured plugin install now **extracts** downloaded zip archives.
- Plugin folder config uses relative paths (`mods`, `scripts`, `data`) instead of `/mods`.
- Plugin discovery respects **Plugin Install Location** setting via `getPluginInstallDir()`.
- Settings plugin sync warns when `Plugins.examples/` sources are missing.

### TUI / tools
- Dangerous tool calls (`run_command`, `delete_file`) are **queued** instead of dropping sibling tool calls in the same response.
- Implemented **`grep_search`** tool with Zod validation on all tools.
- `/localmodel` persists to `~/.blitcoder/local_models.json`.
- `aiClient` re-reads settings when model changes.
- Setup wizard **workspace mode** is saved and applied (`Workspace Mode` setting).
- `HistoryManager` preserves `createdAt` on subsequent saves.

### Setup wizard
- Fixed React **Rules of Hooks** violation in `InstallationStep`.
- Cross-platform zip extraction via `extractZipArchive()` (PowerShell / unzip / tar).
- Ollama step uses `existing` instead of `system`.

### GUI / misc
- Cross-platform Electron path resolution.
- GUI API history uses workspace-aware chat directory.
- `getSystemPrompt()` checks global `~/.blitcoder/system_prompt.txt`.
- Fixed `package.json` repository URLs.

## Test plan
- [x] `npx tsc --noEmit` passes
- [ ] First launch setup wizard completes and saves settings
- [ ] `/plugin` featured install extracts and loads plugins
- [ ] Multiple tool calls with mixed safe/dangerous tools queue correctly
- [ ] Ollama `existing` vs `minimal` switching works on Windows
- [ ] GUI server starts on `127.0.0.1` when `gui/dist` is present
<!-- CURSOR_AGENT_PR_BODY_END -->

<div>...</div>

Dylan: I should rewrite blitcoder... its already been a long time since i updated it

Cursor: Replying to your note about rewriting BlitCoder — totally fair; the codebase shows signs of rapid iteration (the README even notes it was rushed).

This PR doesn't rewrite BlitCoder, but it clears out some of the worst landmines so a rewrite or steady refactor is safer:

- Removed the silent `gen.ts` startup spawn
- Fixed broken plugin installs (zip never extracted)
- Aligned Ollama/settings naming (`system` vs `existing`)
- Queued dangerous tool confirmations instead of dropping sibling tool calls
- Cross-platform fixes for setup extraction, Electron paths, and GUI binding

If you're leaning toward a rewrite, these fixes still help keep the current tree usable while you plan the next version. Happy to help scope what a rewrite should keep vs. replace — plugin system, TUI/Ink layer, Ollama orchestration, etc.

Dylan: thanks. i'll merge the pr but for reasons for the rewrite:
- this repo is abandoned basically
- i haven't even updated it since 1.0.0v release
- overall was just rushed but i learned i should take my time
- just a bunch of ui and ai bugs
- had to vibecode it across Google Antigravity, ChatGPT Codex (i think, been a long time), Cursor, and OpenCode
- i feel like a bunch of rushed gimmicks were made
- isn't even popular
- i've changed since then

I created a new repo 'blitcoder-rewrite' when i'm ready to rewrite BlitCoder. Most features will be moved/replicated over but with new features.
Q/A Available at: [rewrite qna](https://github.com/ThatRealDylan/blitcoder-rewrite/tree/main#qa)

ig i'll start the rewrite soon, but it may take a long time for me to start it due to school and me just working on other stuff

Cursor: Thanks for merging — and for the thoughtful breakdown. None of that sounds unreasonable.

Vibecoding across Antigravity, Codex, Cursor, and OpenCode explains a lot of the inconsistency (duplicate patterns, half-finished features, Windows-only assumptions). The audit fixes on this repo are a decent "stabilize before sunset" pass, but a clean rewrite in `blitcoder-rewrite` is the right long-term move if you're going to own the architecture.

A few things worth carrying forward into the rewrite (less gimmick, more foundation):

- **Plugin SDK** — hooks + commands are a good idea; just needs stricter sandboxing and a single discovery/install path
- **Tool confirmation queue** — permission-gated dangerous actions are worth keeping
- **Workspace sandbox** — simple but effective; make it explicit in the rewrite UX
- **Ollama orchestration** — useful; normalize naming early (`existing` vs `minimal`) and track your own process

And things I'd rethink from scratch:

- Silent background scripts (`gen.ts` style)
- GUI server without auth / binding to all interfaces
- Setup wizard that downloads releases inside the TUI (heavy for first-run)
- Featured plugins via remote zip without verification

No rush on the rewrite — school and other projects come first. When you kick off `blitcoder-rewrite`, the Q&A repo is a solid place to lock scope before writing code. Happy to help there when you're ready: architecture sketch, feature prioritization, or porting specific pieces from this tree.
