# AGENTS.md

Instructions for AI coding agents (Claude Code, Copilot, Codex, Cursor and others) working on ducug.nl, the website of the Dutch Citrix User Group. Human contributors can read it too. The [README](README.md) holds the full how-to (in Dutch); this file covers the rules and the things that are easy to break.

## Language

- **Everything a visitor sees is Dutch.** Page text, headings, button labels, alt text, `title`/`description` front matter, error and empty states, the social cards. Never add English user-facing text, even as a placeholder.
- Tone is informal: address the reader as **je/jij**, not u. Keep it short and plain, the way a community talks, not marketing copy.
- Keep common IT terms in English as Dutch professionals use them (Citrix, EUC, keynote, sponsor, session, workspace). Write Dutch compounds the Dutch way: `evenementpagina`, `sponsormogelijkheden`, not `evenement pagina`.
- Code comments in templates and config are mostly Dutch. Match the language of the file you are editing.
- VS Code's spellchecker flags Dutch words as warnings throughout the repo. They are not errors; ignore them.

## Stack

- [Hugo](https://gohugo.io/) extended (CI pins 0.161.1) with the [Blowfish](https://blowfish.page/) theme as a Hugo module, plus a custom design layer.
- Hosted on **GitHub Pages**. Every push to `main` deploys via [.github/workflows/deploy.yml](.github/workflows/deploy.yml). There is no staging, so `main` is production.
- A nightly build (05:00 UTC) refreshes date-dependent output such as the "next event" and the 30-day evaluation window.

```powershell
hugo server                      # dev server on http://localhost:1313/
hugo server --disableFastRender  # full rebuild on every change
hugo --gc                        # production build to public/
```

## Where things live

| What | Where |
|------|-------|
| Homepage hero text | front matter of `content/_index.md` (`heroKicker`, `heroTitle`, `heroLede`), not the template |
| Hero background photo | `assets/img/site/banner.jpg` (optional) |
| One page per event | `content/evenementen/ducug-NN.md` |
| Event day programme | `data/agenda/ducug-NN.json`, validated by `schemas/agenda.schema.json` |
| Event photos | `assets/img/events/ducug-NN/` (max ~1600px wide), gallery appears automatically |
| Sponsors | `data/sponsors.toml` |
| Board members | `data/bestuur.toml`, photos in `assets/img/team/` |
| Short links (e.g. `/presentaties2609`) | `data/redirects.toml` |
| Design system (all `dg-*` classes) | `assets/css/custom.css` |
| Color scheme | `assets/css/schemes/ducug.css` |
| Live event-day logic, countdown, cookie notice | `layouts/partials/extend-footer.html` |
| Analytics script | `layouts/partials/extend-head.html` |

Prefer editing content and data files over templates. Most changes (new event, programme change, sponsor, board member) need no template work at all.

## Things that are easy to break

- **Redirects.** GitHub Pages ignores `_redirects` files. Moving a page means adding the old URL to the target page's `aliases` front matter. Short links to external sites go in `data/redirects.toml`; `content/_content.gotmpl` turns each entry into a redirect page.
- **`buildFuture = true`** in `config/_default/hugo.toml` must stay. Without it, upcoming events are not built and vanish from the site.
- **Social cards** (`og:image`) are drawn at build time by `layouts/partials/social-card.html`. It needs `assets/fonts/BricolageGrotesque-{Bold,Medium}.ttf`. Without those files the card silently disappears, because the CI runner has no system fonts. Do not swap them for the variable font, and keep `OFL.txt` next to them.
- **Agenda JSON wins** over `[[sessions]]` in front matter. Session `type` is one of `talk`, `sponsor`, `break`, `org`, `social`; `featured: true` adds the Keynote badge.
- **Archiving.** After an event, set `archived = true` in its markdown file. `archived = false` with a future `date` makes it the "next event" on the homepage.
- **Privacy.** Analytics (Microsoft Clarity) run cookieless on purpose, so the banner is an informational notice, not a consent prompt. Do not add consent or cookie-choice logic unless asked. The four forms (`contact`, `cfp`, `sponsor`, `evaluatie`) are Podio iframes that load Google reCAPTCHA. If you add a form or a third-party script, update `content/privacy/_index.md` to match.

## Testing changes

- Run `hugo` and make sure it builds without errors or new warnings.
- Check the page in `hugo server`, in both light and dark mode, and at phone width.
- Event-day states can be simulated on any date with a query parameter: `/?demo=live`, `/?demo=11:30`, `/?demo=binnenkort`, `/evenementen/ducug-NN/?demo=geen-agenda`. See the README for the full checklist.
- To check build output (such as a social card PNG), build to a separate folder: `hugo --destination <temp folder>`. A running `hugo server` overwrites `public/` with `localhost:1313` URLs and pollutes the result.

## Working rules

- Keep changes small and focused. Match the existing style of templates, CSS and content.
- Do not commit or push unless asked. Pushing to `main` publishes the site immediately.
- If you change how something works (a new data file, a new front matter field), update the README in Dutch as well.
