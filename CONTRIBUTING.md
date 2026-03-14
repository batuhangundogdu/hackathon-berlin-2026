# Contributing to Ghost Pitch Engine

3 roles, 1 HTML file, no build tools.

## Roles

### Dev 2: Ghost Generation & Pitch Presentation

You own `generateGhosts()` (~line 637) and the pitch display/narration.

**Your zone:**
- `generateGhosts()` — model fan-out, JSON parsing, ghost creation
- The pitch TTS section inside `runShow()` (~line 906-913)
- `CONFIG.featherless.models.corpus` — add/swap models

**What to do:**
1. Make `generateGhosts()` hit MORE Featherless models in parallel (there are 12,000+)
2. Improve the ghost generation prompt — ghosts need wild personalities
3. Make the pitch card visually pop when a ghost presents
4. After generation, index pitches into Needle (see HANDOFF.md)
5. Keep "Der Speisekarten-Geist" as the last failsafe ghost

**Don't touch:**
- `judgeGhost()`, `decideBuild()`, `booJudge()`, `summonChaosJudges()`
- `listenToCrowd()` and crowd detection
- `overdrive()`

**Test your changes:**
```bash
# Deploy and test at the live URL (needs HTTPS for mic)
git push origin main
open https://hackathon-berlin-mar-2026.westover.lol
```

### Business: Ghost Judge Backstories

You edit `CONFIG.judges` (~line 360) — the backstory strings.

**Each judge has:**
```javascript
{
  key: 'carla',                    // don't change
  name: 'Karla von Strategos',    // ghost name (can edit)
  voiceIdx: 2,                    // don't change
  backstory: 'You are the ghost of Carla Schneider. In life...',  // EDIT THIS
}
```

**Guidelines for backstories:**
- Start with "You are the ghost of [Real Name]."
- Include how they died (funny/dramatic)
- Include what they judge (business viability, technical depth, product craft)
- Include a catchphrase
- Keep under 200 words — these go into LLM prompts

**Also edit:**
- `BOO.backstory` (~line 685) — the MC ghost's personality
- Ghost names in `CONFIG.judges[].name` — make them spooky/funny

### James: Judging + Crowd + OVERDRIVE

I own everything from crowd detection through build/deploy.

**My zone:**
- `listenToCrowd()` — Web Audio FFT
- `judgeGhost()` — judge deliberation with Boo + core + chaos judges
- `booJudge()`, `booSpeak()`, `summonChaosJudges()`
- `decideBuild()` — BUILD/NO BUILD via Boo
- `overdrive()` — build + deploy pipeline
- Needle integration for judge RAG context

## File Structure

```
index.html          ← THE ENTIRE APP (single file, ~980 lines)
HANDOFF.md          ← Architecture + work split + secrets
CONTRIBUTING.md     ← This file
```

## Config Block (~line 324)

All tunables are in `CONFIG` at the top of the `<script>` tag:

| Key | What | Who edits |
|-----|------|-----------|
| `featherless.apiKey` | API key (expires 2026-03-17) | nobody |
| `featherless.models.corpus` | Models for ghost generation | Dev 2 |
| `featherless.models.judges` | Models for judge LLM calls | James |
| `needle.apiKey` | Needle RAG API key | nobody |
| `crowd.*` | FFT detection params | James |
| `tts.*` | Speech rate/pitch defaults | anyone |
| `judges[]` | Judge personas + backstories | Business |
| `BOO` | MC ghost config (~line 680) | Business |

## Rules

1. **Don't break the flow** — the show runs top to bottom in `runShow()`
2. **Keep it fast** — 2.5 minutes total for the presentation
3. **TTS rate >= 1.1** — ghosts talk fast
4. **Judge verdicts = 1 sentence** — `maxTokens: 60`
5. **Test at the live URL** — local file:// won't work (needs HTTPS for mic)
