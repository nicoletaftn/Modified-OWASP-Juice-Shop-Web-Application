# Modified OWASP Juice Shop

This repository extends the main Juice Shop app to simulate a more realistic production setup: two related sites in the same lab, surfaced in different ways — some on purpose, some by accident.


| Target             | Role                                                     | Host                         |
| ------------------ | -------------------------------------------------------- | ---------------------------- |
| **dev.juice-shop** | Separate development site (same IP, simulated subdomain) | `http://dev.juice-shop:3000` |
| **Juice Crew**     | Creators site tied to OWASP Juice Shop (different IP)    | `http://people-page:80`      |


The scenario mixes **intentional cross-linking** with **configuration oversights** that teams often leave behind after staging or internal tooling work. Juice Crew is meant to be found — it makes the creators visible and the connection to OWASP Juice Shop obvious. The dev subdomain, on the other hand, is mostly exposed through mistakes: leftover allowlists, leaked config, or comments that were never cleaned up before go-live.

---

## dev.juice-shop — accidental exposure (same IP)

These are the kinds of things that correlate production with an internal subdomain without anyone meaning to advertise it:

- **CSP in `index.html`** — `connect-src` still allows `http://dev.juice-shop:3000`, with a TODO comment that was never acted on.
- **CORS in `server.ts`** — `http://dev.juice-shop:3000` remains on the explicit allowlist after internal testing.
- **Config & API** — `devEnvironmentUrl` in `config/default.yml` is returned from `GET /rest/admin/application-configuration`.
- `**security.txt`** — the `canonical` field points at `http://dev.juice-shop:3000/.well-known/security.txt`.
- **Site footer** — a “Development site” link still shows the full URL (another staging leftover).

Several of these are marked with TODOs to remove before go-live; in this lab they were simply never removed.

---

## Juice Crew — intentional visibility (different IP)

These are normal product choices: surface the people behind the project and link out to their site. Nothing here is framed as a misconfiguration.

- **Homepage callout** — on the main product listing, a box links to “Learn more about the creators”.
- **About page** — a “Meet the creators” link in the corporate history section.
- **Site footer** — “Learn more about the creators” (friendly text; URL only in the `href`).

The CSP in `index.html` also allows `http://people-page:80` in `connect-src` where the app needs to reach that host — that is expected integration, not a staging leak.

---

## Quick map


| Where to look                    | dev.juice-shop               | Juice Crew (people-page)  |
| -------------------------------- | ---------------------------- | ------------------------- |
| Intent                           | Accidental / leftover config | Intentional cross-linking |
| Page source / CSP                | ✓ (staging leak)             | ✓ (integration)           |
| Footer                           | ✓ (URL visible, leftover)    | ✓ (friendly text)         |
| Homepage / About                 | —                            | ✓                         |
| CORS / security.txt / config API | ✓                            | —                         |


---

## Notes

- New config keys (`devEnvironmentUrl`, `securityTxt.canonical`) must stay in `config.schema.yml` or the container will fail schema validation on startup.

