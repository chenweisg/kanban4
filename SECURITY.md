# Security notes: IT PMO Kanban

Threat model for `index-v2.html`, made with the cybersecurity-analyst skill (STRIDE + CIA triad).
`index.html` (v1) is kept unchanged for comparison.

## Scope
- **What it is:** a static single-page app on GitHub Pages or opened from a local file. It has no login and no server of its own.
- **Data:** task titles, descriptions and assignee names. They are held in memory only and lost on refresh.
- **The only outbound connection:** a JSON POST to FormSubmit when a task is added.
- **Trust boundaries:** text typed by the user becomes page HTML, and the page sends data to FormSubmit, then on to an email inbox.

## Threats and controls

| STRIDE | Threat | Control in v2 | Residual risk |
|---|---|---|---|
| Tampering / Elevation | XSS via task text (`<img onerror>`, `<script>`) | `escapeHtml()` on every value; toasts use `textContent`; CSP blocks all external script sources | Low. CSP still needs `'unsafe-inline'` because the app is one file. |
| Information disclosure | Page sends data to an unexpected host (malicious edit, injected code) | CSP `connect-src https://formsubmit.co`; endpoint must be HTTPS, host `formsubmit.co`, path `/ajax/`; `credentials: "omit"`, `no-referrer` | Low |
| Information disclosure | Sending tasks to the placeholder `YOUR_EMAIL@example.com` | Placeholder detected, so no request is sent and the user sees a warning toast | None |
| Spoofing | Lookalike text using bidi overrides / zero-width chars (e.g. `gnp.exe`) | `sanitizeText()` strips bidi, zero-width and control characters | Low |
| Tampering | HTML injected into the notification email | Angle brackets removed from values sent to FormSubmit | Low |
| Denial of service | Inbox flooding through repeated submissions or bots | Client-side limit of 1 email every 10 s; honeypot field; FormSubmit `_honey` | Medium. Client-side limits can be bypassed by calling FormSubmit directly. |
| Tampering | Unexpected data dropped onto the board | Drop handler only accepts IDs that exist in `state.tasks` | None |
| Tampering | Invalid form values | Allow-listed selects, length limits, name pattern, due date between today and +5 years | Low |
| Info disclosure | Clickjacking / framing | Not controllable from a `<meta>` tag on GitHub Pages (`frame-ancestors` needs an HTTP header) | Low. The page has no privileged actions. |

## Not applicable by design
- No authentication, cookies or storage, so no session theft or stored-data exposure.
- No third-party libraries or CDNs, so no supply-chain risk from dependencies.

## Recommendations if this became a real system
1. Serve it with HTTP security headers (CSP with script hashes, `frame-ancestors 'none'`, HSTS).
2. Move notifications to a server behind the bank's SSO, with rate limits enforced on the server.
3. Store the FormSubmit/email address as server configuration, not in the page.
4. Add audit logging for task changes (repudiation), which the demo intentionally has none of.
