# astro-ai-i18n

**An Astro site that translates itself.**

A small, deliberately plain Astro site in four languages, and the workflow that
keeps the other three up to date with the first one. English is written by
hand; French, German and Spanish are produced by [AI Glot](https://ai-glot.com)
from the command line, written back as ordinary files, and reviewed in a diff.

```bash
npm install
npm run dev              # http://localhost:4321
npm run i18n:status      # what is missing, what is out of date, what it costs
```

A fresh clone builds and serves in four languages with no account, no key and
no network: every translation it ships is a committed file. The account is
needed only to produce new ones.

## What is actually interesting here

The site is dummy content on purpose: a home page, an about page, a Markdown
journal, four black-and-white SVGs and no colour. What is worth copying is the
four things around it.

**Translation debt is a number, not a feeling.** `.i18n-lock.json` stores the
hash of the English text each translation was made from, so the tooling can
tell *never translated* from *translated, then the English changed*:

```text
UI copy — src/i18n/strings/en.json, 60 strings

  fr   60 current     0 missing     0 out of date
  de   60 current     0 missing     0 out of date
  es   45 current    15 missing     0 out of date

Journal — src/content/blog/en/, 4 posts

  fr    4 current     0 missing     0 out of date
  de    4 current     0 missing     0 out of date
  es    3 current     1 missing     0 out of date   +typography-notes

~421 source words behind (~141 credits in Lite, ~421 in Standard).

  + missing    ~ English changed since the translation was made
```

That is a real fresh clone. Edit one English sentence and run it again: that
string moves to `out of date` in all three languages, without anyone having to
remember that it changed.

**One command translates exactly that**, for every language, in one batch:

```bash
npm run i18n:translate -- --dry-run   # price it, spend nothing
npm run i18n:translate                # send it, write the files back
git diff                              # the review step, which stays with a person
```

**An untranslated page is a decision, not an accident.** One word in
`src/i18n/config.ts` decides whether `/de/about` serves the English text with a
notice, or sends the reader to `/about`. Either way it canonicalises to the
English URL and is never advertised as the German alternate, which is what
stops four URLs competing for one page's ranking.

**An agent can run the whole thing.** `AGENTS.md` (symlinked as `CLAUDE.md`) is
the manual: the workflow, how to set AI Glot up, the REST endpoints if you
prefer them to the CLI, and the mistakes this shape of site makes.
`.claude/skills/translate-site/` is the same thing as a skill.

## What is in the repository

```
src/
  i18n/config.ts          locales, URL shape, what an untranslated page does
  i18n/strings/en.json    the site's copy. Hand-written, the only source
  i18n/strings/*.json     generated. Never edited by hand
  i18n/ui.ts              loads them, falls back to English key by key
  i18n/blog.ts            resolves a post: the translation, or the English standing in
  content/blog/{en,fr,de,es}/   the Markdown CMS, one folder per language
  pages/[...locale]/      four route files that answer for every locale
  components/, layouts/, styles/global.css
scripts/
  i18n.mjs                status · translate · bless · check
  lib/aiglot.mjs          the only file that talks to AI Glot
test/                     assertions that need no network and no key
```

## How the languages work

| | |
| --- | --- |
| URL shape | `/about` is English, `/fr/about` is French. The default locale is never prefixed, so adopting a language later moves no existing URL |
| Translation key | the path after the locale segment. `fr/blog/quiet-tools` is the French `blog/quiet-tools` because the paths match. There is no mapping table |
| Missing string | falls back to English, key by key, and is still counted by `i18n:status` |
| Missing page | `fallback` (English body, notice, canonical home) or `redirect`, chosen in `src/i18n/config.ts` |
| Language picker | in the navbar, no JavaScript, keeps the reader on the same page, and marks languages that do not have it |
| Adding a language | one entry in `src/i18n/config.ts`, then `npm run i18n:translate` |

Every French, German and Spanish string and post in this repository was
produced by AI Glot through `npm run i18n:translate`, not written by hand. The
one exception is deliberate: Spanish is left without the About page and one
journal post, so that `i18n:status`, the fallback banner and the picker's `EN`
mark are all visible on a fresh clone. `npm run i18n:translate` fills them in
for about 141 credits.

## Setting AI Glot up

1. Create a key at [app.ai-glot.com/developer](https://app.ai-glot.com/developer).
2. `cp .env.example .env` and put it in `AIGLOT_API_KEY`.
3. `npm i -g @ai-glot/cli && aiglot auth login` — or set
   `AIGLOT_TRANSPORT=api` and use no CLI at all.
4. `npm run i18n:translate -- --dry-run`.

Creating a translation and planning it are free and repeatable; only approval
spends credits, and `--dry-run` never reaches it. The full setup walkthrough,
the endpoint table and the cost model are in [AGENTS.md](./AGENTS.md).

## Making it yours

1. Replace `src/i18n/strings/en.json` with your copy, and `src/content/blog/en/`
   with your posts. Delete the other locale folders and both
   `.i18n-lock.json` files.
2. Set your languages in `src/i18n/config.ts` and your origin in
   `astro.config.ts`.
3. Point the URLs in `.github/ISSUE_TEMPLATE/config.yml`, `SECURITY.md` and
   `CODE_OF_CONDUCT.md` at your repository, or delete those files. They name
   this project's maintainer, who cannot act on anything in yours.
4. `npm run i18n:translate`.

Nothing in `src/` or `scripts/` needs editing to change language, copy or
behaviour. If you find something that does, that is a gap worth an issue.

## Status and licence

A working example, maintained on a best-effort basis by one person. It is not a
published package: fork it, read it, take the parts you want.

MIT. See [LICENSE](./LICENSE), [CONTRIBUTING.md](./CONTRIBUTING.md) and
[SECURITY.md](./SECURITY.md).

AI Glot is a paid product and this repository is not affiliated with your use of
it beyond calling its public CLI and API.
