# openGym — Project Rules for Claude Code

## What this is
openGym is Travis's personal training app: lift logging, programme tracking, video and photo
progress records, and athletic-resilience testing. It is being moved from a PC-hosted app to a
cloud-hosted, phone-first web app so it works anywhere without a home PC running.

Owner: Travis (Christchurch, NZ). Primary device: phone. Training for HYROX Auckland, Feb 2027.

Where everything lives (PC paths, Docker, GitHub, cloud accounts): see `SOURCES.md` in the private ops repo `Tbronks88/elite-athlete-os`.
Programme content and evidence: see `docs/` in that repo.

## Target architecture
This repo is a fork of **DuarteSantos8/openGym** (React 19 + Vite web app served by nginx, plain Node API,
JSON-file storage in `./data`). Extend the existing app; do not replace its stack.

| Layer | Service | Notes |
|---|---|---|
| App + API | The existing `web` + `api` containers via Docker Compose on a small always-on server with a persistent disk | Already a PWA with passkeys, sync, progress photos and before/after slider |
| Data | The existing `./data` JSON store on the server's persistent disk | Nightly off-server backup is required |
| HTTPS | Caddy or Cloudflare Tunnel (both documented upstream) | `RP_ID` / `ORIGIN` must match the final hostname |
| Large video archive (only if needed) | Cloudflare R2, **private bucket** | Only if the built-in media upload can't hold full lift videos |
| Jump analysis (later phase) | Small Python worker running `kinemotion` | Triggered on clips tagged as jump tests |

Keep changes small and isolated so upstream releases (v1.4.3+) can still be merged.
Upstream is AGPL-3.0 (keep LICENSE and credit to Duarte Santos). Exercise media is licensed for openGym use only; don't reuse it elsewhere.

## Non-negotiable rules
1. **Nothing depends on a local PC.** Every feature must work from the phone with the PC off.
2. **Secrets never go in a repo.** Use `.env.example` with placeholder names only. Travis sets real keys on the server himself.
3. **No personal data in the public fork.** Training plans, photos, data and personal details belong in the private ops repo or on the server.
4. **Media is private by default.** No public bucket URLs. Before/after photos are sensitive.
5. **Keep originals.** Store the original upload; generate a smaller playback copy if needed. Never overwrite originals.
6. **One phase per branch and pull request.** Each PR ends with: what changed, how to test it on a phone, and any setup Travis must do.
7. **Don't rewrite what works.** Audit first, migrate second, add features third.
8. **Plain-English summaries.** Travis reviews on his phone, so keep PR descriptions short and concrete.
9. **Keep SOURCES.md current.** After each phase, update SOURCES.md in the ops repo with what changed and where things now live.

## Data model (starting point; adjust after the audit)
- `sessions`: date, type (strength A/B, pump, run, plyo, resilience, test), notes, RPE
- `sets`: session_id, exercise, set_no, weight_kg, reps, rpe, media_id (optional)
- `exercises`: name, category (lift, plyo, sprint, tendon, neck, wrist, ankle, mobility, HYROX station), cues
- `media`: storage_key, kind (video or photo), taken_at, duration, tags[], is_post_worthy, linked session/set
- `body_photos`: media_id, pose (front, side, back), date; used for before/after comparisons
- `tests`: date, test_type (CMJ height, drop-jump RSI, knee-to-wall cm, single-leg calf raises, balance seconds, neck hold seconds, dead hang seconds), value, media_id
- `plan`: the current programme phase, weekly template and progression rules

Map these onto the app's existing data structures rather than adding parallel ones.

## Media tags (for the future personal-brand library)
`lift`, `PR`, `before-after`, `test-day`, `plyo`, `sprint`, `hyrox`, `mobility`, `post-worthy`, plus the exercise name.
The media library must filter by tag, date and exercise, and bulk-download originals.

## Definition of done for any feature
- Works on a phone, tested at about 390 px width
- Data survives a refresh and a different device signing in
- No secrets committed; `.env.example` updated if a new variable was added
- PR description includes phone test steps
