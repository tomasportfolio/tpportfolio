# Project rules

Personal portfolio of Tomás Peró: https://tomaspero.vercel.app
Production is `main` on the `prod` remote (tomasportfolio/tpportfolio). Vercel deploys only from
`main` (~90 s after a push) and builds no previews of other branches.

Local state and tasks (outside the repo, never committed):
@../_docs/estado.md
@../_docs/backlog.md

## Rules
1. **Two languages, always.** Every change to content, navigation or UI text goes in Spanish and
   English with the same facts, names, numbers and dates, including `alt`, `title`, `aria-label`
   and placeholders. Spanish is rioplatense, first person, with voseo.
2. **Agent mode in sync.** `/agents.md`, `/agents.txt` and `/llms.txt` are built by `lib/agents.ts`
   from `lib/site-content.ts`: update them in the same change. They only carry what helps a visiting
   agent, nothing private or internal. Update `content/assistant/*.md` (Clippy) with the same content.
3. **Never overwrite a version.** Each round ships as its own `release/rNN-<topic>` branch and `rNN`
   tag. Parallel sessions number fast: `git fetch prod --tags` and take the next free number.
   No force pushes.
4. **Document the version.** "Current version" in `README.md` + an entry in `CHANGELOG.md`.
5. **Public repository.** Never commit `.env*`, `docs/`, `AUDITORIA*`, local paths or internal notes.
6. **Years of experience** come from `lib/career.ts` (since August 2007). Never write "18"/"19" by
   hand; in `content/assistant/*.md` use `{{CAREER_YEARS}}`.
7. **Clippy stays free** (Groq free tier, `app/api/chat`): no paid models or services; QA never
   sends it messages.
8. **Archive, don't delete.** Copies or new versions, with a rollback log in `../_docs/logs/`.
   Remove junctions only with a non-recursive delete.

## Stack
Next 15.5.12 (App Router) · React 18 · TypeScript · Tailwind 3 · GSAP + Lenis · Vercel Analytics ·
Groq (Clippy) · visit counter in Vercel KV (`app/api/visits`, counts only in production).
Pages revalidate weekly (`revalidate = 604800`).

## Where things live
- `lib/site-content.ts`: identity, home frames, navigation, timeline entries.
- `lib/work-videos.ts`: every video in the work pages and the timeline.
- `lib/site-state.ts`: state names (`?lang=`, `?filter=`, storage keys, intro cookie, menu events).
- `lib/career.ts`: years of experience. `lib/agents.ts`: agent files.
- `content/assistant/*.md`: Clippy's knowledge base. `translations/index.ts`: shared UI strings.
- `BUSINESS_LOGIC.md`: how the intro, home and sections behave.
- `qa/`: Playwright QA, separate package (`qa/PLAN.md`, `qa/PROGRESS.md`; `qa/results/` not committed).

## Commands
```bash
npm run dev
npm run typecheck && npm run lint
npm run build          # move .next aside first (see Traps)
cd qa && node run.mjs --base http://localhost:3242 --label rNN --quick
```

## Checks before finishing
- `npm run typecheck` and `npm run build` pass.
- UTF-8 without BOM, real accented characters.
- Before promoting: `qa/run.mjs` with 0 console errors and no horizontal scroll.

## Promoting to production (only with Tomás's explicit OK)
1. Push the branch and the `rNN` tag to `prod`.
2. Backup: `git push prod prod/main:refs/heads/backup/production-YYYY-MM-DD-pre-rNN`.
3. Fast-forward: `git push prod release/rNN-<topic>:main` (never forced).
4. Check the live site after ~90 s; update README and `estado.md`.

Rollback: Vercel "Instant Rollback" to the previous deployment (git untouched), then fix it as a
new version (`git revert` on `release/rNN+1`, fast-forward to `main`). Resetting `main` to a
backup branch needs a force push: only with Tomás's explicit OK.

## Traps
- `npm run build` fails with an internal webpack TypeError if `.next` exists: move it aside first
  (keep one `.next_prev`, not one per build).
- Git "Filename too long" on Windows comes from `public/pdf_toolkit`: `git config core.longpaths true`.
- Git Bash rewrites `--routes /x`: prefix with `MSYS_NO_PATHCONV=1`.
- `rem` is 13 px here (html inherits monospace): rem/Tailwind sizes are ~19% smaller than nominal.
- Branches are invisible until promoted: test locally (prod build on :3242).
