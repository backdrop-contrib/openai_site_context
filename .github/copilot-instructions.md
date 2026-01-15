# Copilot / AI agent instructions — openai_site_context

This is a Backdrop CMS module that injects global site context and safety guardrails into OpenAI calls. Use these notes to be productive quickly and avoid breaking behavior.

- **Purpose:** `openai_site_context` prepends configured guardrails and a global site prompt to OpenAI prompts and inserts them as a `system` message for chat. See `openai_site_context.module` for implementation.

- **Key files:**
  - `openai_site_context.module` — core logic, hooks, and settings form.
  - `openai_site_context.info` — module metadata; lists dependency `openai` and admin path `admin/config/openai/site-context`.
  - `config/openai_site_context.settings.json` — default values for `guardrails`, `global_prompt`, and `enabled`.
  - `README.md` — brief project description and install pointer (Backdrop module).

- **Integration points:**
  - Implements `hook_openai_prompt_alter(&$prompt, array $context)` and `hook_openai_chat_messages_alter(array &$messages, array $context)` to modify outbound payloads.
  - Depends on the `openai` module — treat that as the primary API surface for sending requests.

- **Important behaviors to preserve:**
  - The module **skips injection** when `$context['operation'] === 'embedding_generate'` to avoid contaminating embedding vectors.
  - Injection can be disabled per-call with `$context['disable_site_context'] = TRUE`.
  - For chat payloads it inserts a `system` message via `array_unshift($messages, ['role' => 'system','content' => $site])`.
  - When enabled, it logs a redacted preview via `watchdog('openai', ...)` — keep that logging pattern if you adjust injection.

- **Configuration & tokens:**
  - Admin UI: `admin/config/openai/site-context` (provided by `openai_site_context_settings_form`).
  - The module uses Backdrop Token replacement: `token_replace($combined, [], ['sanitize' => FALSE])`. Tokens like `[site:name]` are supported; some require runtime context.
  - Config keys: `openai_site_context.settings` → `enabled`, `guardrails`, `global_prompt`.

- **Conventions & patterns in this repo:**
  - Use Backdrop APIs (`config()`, `system_settings_form()`, `watchdog()`, and `token_replace()`), not custom helpers.
  - Guardrails live in config and are treated as high-priority system/developer instructions; do not expose them in responses or logs beyond the existing redacted preview.
  - Use `backdrop_strlen`/`backdrop_substr` when present; fall back to PHP `strlen`/`substr` as the module does.

- **When editing or extending:**
  - Preserve the embedding exclusion (`operation === 'embedding_generate'`) unless you add alternative safeguards and verify embeddings aren't polluted.
  - If adding new tokens or runtime data, ensure token replacement continues to use `sanitize => FALSE` only when appropriate and document any new token types in the settings UI.
  - Keep admin path and `.info` dependency in sync with changes to integration or configuration UX.

- **Examples (from repo):**
  - Prompt injection (non-chat): `$prompt = trim($site . "\n\n" . $prompt);`
  - Chat injection: `array_unshift($messages, ['role' => 'system', 'content' => $site]);`

If anything here is unclear or you want more detail (examples of tests, CI commands, or recommended local Backdrop setup), tell me which area to expand. Ready to iterate.
