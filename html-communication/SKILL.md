---
name: html-communication
description: Use whenever the user asks for a plan, report, or other communication that deserves a rich view, or if they just say HTML and provide no other context
---

# HTML Communication

Use this skill when the user wants a plan, spec, write-up, findings, summary, report, comparison, or a set of UI mocks presented as readable HTML.

Do not use it for HTML that ships as part of the product.

## Document
Create one self-contained HTML file.

- Write it like a spec, not a landing page: dense and scannable, with no hero, decorative chrome, or marketing voice.
- Default to dark mode.
- Ensure it is mobile-readable.
- Use inline scripts when interactivity materially helps communicate or explore concepts.

## Delivery

Write the file to `/tmp` with a descriptive filename. Do not start a local HTTP server or run `tailscale serve` for the user. Never use, add, or recommend the `--bg` flag with `tailscale serve`.

Keep all serving information out of the HTML file itself. Do not add local or Tailscale links, terminal commands, setup instructions, or a "serve this document" section to the generated document. Put that information only in the agent's inline chat response that accompanies the file.

In the final response, always provide:

- A clickable local link: `http://127.0.0.1:8781/<filename>`.
- A clickable Tailscale link: `https://<tailscale-dns-name>:8443/<filename>`.
- The two copy-paste commands below, separately labeled so the user can choose the one that fits their situation.

Resolve the current machine's Tailscale DNS name with a read-only status check and remove any trailing period. If it cannot be resolved, keep `<tailscale-dns-name>` in the link and clearly label it as a placeholder. Never imply that either HTTP endpoint is already running.

Use this command when a server is already listening on port 8781:

```bash
tailscale serve --https=8443 http://127.0.0.1:8781
```

Use this command to start a server for `/tmp` and expose it through Tailscale:

```bash
python3 -m http.server 8781 --bind 127.0.0.1 --directory /tmp & tailscale serve --https=8443 http://127.0.0.1:8781
```
