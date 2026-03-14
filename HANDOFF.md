# Ghost Pitch Engine — Work Split

**Repo**: https://github.com/neherdata/hackathon-berlin-2026
**Live**: https://hackathon-berlin-mar-2026.westover.lol (public, no login)
**Deploy**: Push to `main` or run wrangler (see below)

## Architecture

Single HTML file (`index.html`), everything in-browser except OVERDRIVE build phase.

- **Featherless API** (OpenAI-compatible): `https://api.featherless.ai/v1/chat/completions`
- **Needle API** (RAG): Ghost pitches indexed → judges search for context
- **Web Speech API**: TTS for all ghost voices
- **Web Audio API**: FFT crowd reaction detection (cheers vs boos)
- **Cloudflare Pages**: deployed at `hackathon-berlin-mar-2026.westover.lol`

## Flow

```
[Boo MC Intro] → [Generate Ghosts] → for each ghost:
  → [Pitch TTS] → [Crowd Reaction 2s] → [Judging Round 1]
  → [Crowd Reaction 2s] → [Judging Round 2 + Chaos Judges]
  → [BUILD / NO BUILD] → if BUILD: [OVERDRIVE deploy]
```

**2.5 minutes total** — everything is fast. TTS rate 1.1-1.2x, judge verdicts = 1 sentence (60 tokens max).

## Work Split

### Dev 2: Ghost Generation (corpus + pitch voices)

You own `generateGhosts()` and the pitch presentation phase.

**What to build:**
1. Fan out to ALL Featherless models simultaneously (not just 4 — use as many as possible)
2. Each model generates a ghost with: name, type, pitch, tagline
3. Ghost prompts should be creative — the ghosts need personality
4. Present each ghost on screen with its card, then narrate via Web Speech API
5. The failsafe ghost "Der Speisekarten-Geist" (Menu Ghost) is always added last
6. **Index each ghost pitch into Needle** after generation (see Needle section below)

**Key function:** `generateGhosts()` around line 637
**Config:** `CONFIG.featherless` at top of file

**Models available (Feather Premium):**
- `deepseek-ai/DeepSeek-V3-0324` — best reasoning
- `MiniMaxAI/MiniMax-M1-80k` — good for agentic/tool-use ideas
- `moonshotai/Kimi-K2-Instruct` — multimodal
- `mistralai/Mistral-Nemo-Instruct-2407` — fast
- Plus 12,000+ more at featherless.ai/models

### Business Config: Ghost Backstories

Edit `CONFIG.judges` in `index.html` (~line 360). Each judge has:
```javascript
{
  key: 'carla',              // internal ID
  name: 'Karla von Strategos', // display name
  voiceIdx: 2,               // Web Speech voice index
  backstory: '...',          // Ghost backstory — how they died, what they judge
}
```

The backstories shape how each judge talks and what they focus on. Make them funny, dramatic, and relevant to the real judges. Current judges are ghostified versions of the real hackathon judges:
- **Karla von Strategos** → Carla Schneider (BCG Berlin)
- **Professor Rodrigo VectorMax** → Rod Rivera (ITAM professor, AI podcast)
- **Daniel Testflight** → Dan Makarov (ex-Google, Bird Eats Bug → BrowserStack)

Also edit `BOO.backstory` (~line 685) for the MC ghost personality.

### James: Judging + Crowd Detection + OVERDRIVE

I own the crowd meter, ghost judge panel, Boo MC, and build/deploy pipeline.

**Already built:**
- Crowd reaction FFT (2s windows, cheers vs boos vs silence)
- Ghost judge personas with configurable backstories
- Boo — MC, audience interpreter, and final announcer
- Chaos judges — 2 random absurd judges summoned in Round 2
- BUILD/NO BUILD decision via Boo

**Still building:**
- OVERDRIVE: triggers build via Jira, streams progress, deploys to `{ghost}.westover.lol`
- Needle integration for judge RAG context

## Needle Integration

Needle is a RAG platform — we use it as the knowledge thread between pitch and judging.

**API Key**: `apk_01KKNT8MKM949HHQNA0BMDQ9SR`
**Docs**: https://docs.needle.app/

**How it fits:**
1. After `generateGhosts()`, index each pitch into a Needle collection
2. During judging, judges RAG-search Needle for cross-ghost context
3. This lets judges reference OTHER ghosts' pitches ("unlike that last ghost...")
4. The Needle collection becomes a live archive of the show

```javascript
// Example: index a ghost pitch
await fetch('https://api.needle.app/v1/collections/{id}/files', {
  method: 'POST',
  headers: { 'x-api-key': CONFIG.needle.apiKey },
  body: JSON.stringify({ content: `${ghost.name}: ${ghost.pitch}` })
});

// Example: judge searches for context
await fetch('https://api.needle.app/v1/collections/{id}/search', {
  method: 'POST',
  headers: { 'x-api-key': CONFIG.needle.apiKey },
  body: JSON.stringify({ query: 'what other ghosts pitched similar ideas?' })
});
```

## How to Run

```bash
# Just open the deployed URL (needs HTTPS for mic)
open https://hackathon-berlin-mar-2026.westover.lol
```

## How to Deploy

```bash
# Option 1: Push to main (if Cloudflare Pages auto-deploy is configured)
git push origin main

# Option 2: Manual wrangler deploy
CLOUDFLARE_ACCOUNT_ID=142ba4b292818854dd51c20fd05643c7 \
CLOUDFLARE_API_TOKEN=<token> \
npx wrangler pages deploy . --project-name=hackathon-berlin-mar-2026 --branch=main --commit-dirty=true
```

## Secrets

API keys are embedded in `index.html` (throwaway hackathon keys, expire 2026-03-17).
- Featherless: `CONFIG.featherless.apiKey`
- Needle: `CONFIG.needle.apiKey` (add this)
