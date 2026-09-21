# Security Model — Mid-Autumn Prompt Builder

> **Audience**: future maintainers + security auditors
> **Status**: v1.2.3 2026-09-22
> **Updated**: when image-gen provider changes or new features ship

---

## 1. Threat Model (v1.2.3 — Image Generation Hook)

### What's at stake
- **Personal data**: student name (free text, optional input, stored in localStorage)
- **Costs**: depends on API provider (Pollinations.ai = free, no cost)
- **Reputation**: AI image generation may produce inappropriate output (model-controlled, not us)
- **Classroom disruption**: if students get unrestricted access to image gen

### What we explicitly do NOT protect against
- ❌ Compromised browser (we run JS in user's own browser)
- ❌ Malicious student typing weird prompts (out of scope; teacher oversight)
- ❌ Network sniffing (HTTPS only, but we don't add extra layers)

### What we DO protect against
| Threat | Mitigation |
|---|---|
| **API key leak** | ✅ Pollinations.ai requires NO key — no secret to leak. If we add DALL-E later, backend proxy holds key (NEVER in client). |
| **Cross-site request forgery** | ✅ No authenticated endpoint, so CSRF irrelevant |
| **Server-side data leak** | ✅ No backend; nothing to leak. |
| **XSS via prompt content** | ✅ All user input rendered via `textContent` (not innerHTML). The prompt we send to Pollinations is built from validated `data` array, never raw user input. |
| **Long-running fetch leak** | ✅ Each generation uses AbortController; cancelled on reset. |
| **Quota exhaustion attack** | ✅ Pollinations has no per-key quota. If we add DALL-E, teacher toggle controls access. |
| **Student unauthorized access** | ✅ Default OFF. Teacher must explicitly toggle ON each session (per SPEC §3 Sprint v1.2.3). |
| **Generated image stored on server** | ✅ Pollinations doesn't store (no account). DALL-E images cached by OpenAI but not associated with our app. |
| **localStorage data exfiltration** | ✅ localStorage contains no sensitive data: PIN (4-digit), history (10 prompts), UI prefs. No API keys, no student PII beyond optional input name. |

### Trust boundaries
- **Trusted**: User's own browser, GitHub Pages CDN, Pollinations.ai public API
- **Untrusted**: Generated image content (model-controlled)
- **Boundary crossings**:
  - User picks → our JS → Pollinations URL → image bytes returned → rendered inline

---

## 2. Code-Level Security Rules

### Never
- ❌ Never `localStorage.setItem('apiKey', ...)` — keys are session-only at most
- ❌ Never `innerHTML` with user-controlled content — always `textContent`
- ❌ Never log user prompts to console in production (info leak via DevTools)
- ❌ Never enable CORS wildcard on any backend we add later

### Always
- ✅ Use `URLSearchParams` or `encodeURIComponent` for all user data sent in URLs
- ✅ Validate response status (`fetch().ok`) before rendering
- ✅ Use `AbortController` for any long-running request
- ✅ Wrap image render in try/catch
- ✅ Display error to user with retry option (no silent failures)

### Provider-specific
- **Pollinations.ai**: GET `https://image.pollinations.ai/prompt/{encodeURIComponent(prompt)}?width=512&height=512&seed={random}&nologo=true`. Returns image/jpeg binary. CORS open. No key.
- **DALL-E 3 (future)**: Must go through backend proxy that holds the key. Frontend POSTs prompt to our `/api/generate`, proxy forwards to OpenAI with key.

---

## 3. Operational Security

### Pre-deploy checklist
- [ ] No `api_key` / `secret` / `token` strings in committed HTML
- [ ] `git diff --cached --name-only` reviewed before each push
- [ ] No console.log of user data
- [ ] No `eval()` / `Function()` / `new Function()`
- [ ] All `dangerouslySetInnerHTML`-equivalents (we use none) audited

### Incident response
- If API key leaked: rotate key + invalidate prior key + notify user
- If inappropriate image generated: report to provider's moderation team + remove UI surface
- If localStorage XSS discovered: clear all `midautumn_*` keys + bump version + force refresh

---

## 4. v1.2.3 Specific Decisions

| Question | Decision | Reasoning |
|---|---|---|
| API provider | **Pollinations.ai** | Free, no key, CORS open, client-side direct. Lowest security risk (no key to leak). |
| Student access default | **OFF** | Teacher must explicitly toggle ON per session. Cost control + classroom oversight. |
| Where to display image | Inline in result panel, below tabs | One-click generation flow; no popup. |
| Image storage | None (Pollinations URL only, no caching) | Privacy + simplicity. |
| Request method | GET with URL-encoded prompt | Standard Pollinations API. |
| Timeout | 60s via AbortController | Pollinations typically 5-30s; 60s is generous safety margin. |
| Retry | One-click "重試" button | Per F4 silent data loss bug family (memory rule 13). |
| Image output | 512×512 jpeg (smallest tier; no watermark) | Default Pollinations params. |

---

## 5. Future Threat Scenarios (v1.3+)

If we add DALL-E:
- Must deploy backend proxy (key storage)
- Must add rate limiting per session
- Must add cost dashboard for teacher

If we add student profile sync:
- Must add data retention policy
- Must add GDPR/HK privacy compliance
- Must encrypt at rest

If we add multi-user / cloud sync:
- Must add auth (OAuth / magic link)
- Must add per-user data segregation
- Must add audit log
