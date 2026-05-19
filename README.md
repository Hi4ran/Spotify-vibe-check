# Vibe Check 🎧

> An AI music curator that turns a mood into a playlist. Type a scene — *"6am gym session, need to lock in"* or *"周日早上阳光正好在家打扫"* — get 10 songs that actually fit.
> 一个 AI 音乐策展人，把心情翻译成歌单。输入一个场景，得到 10 首契合的歌。

**[→ Try the live demo](#)**  ·  **[→ Read the PRD](./PRD.md)**

---

## Why this exists

This started as a school data-analysis project on 30k Spotify songs — t-tests, correlation analysis, a random-forest popularity classifier. Solid analytics, but it answered questions only an analyst would ask.

I rebuilt it as a **product**: what if the same audio-feature data powered an interface a real user would want to use? The result is *Vibe Check* — a natural-language music curator that turns mood descriptions into matching playlists.

**The product bet:** Spotify's recommender is amazing once it knows you, but weak on mood shifts, scene-specific needs, and cold starts. A natural-language interface that translates "I want to feel X" into audio-feature ranges can fill that gap with no listening history required.

## How it works

```
User types a mood (English or Chinese)
       ↓
Intent parser → target audio features + genre weights
       ↓
Local retrieval → scores 2,048 tracks by weighted distance, picks top 10
       ↓
Explanation generator → writes a human rationale for the picks
       ↓
Rendered playlist + the "why" (in the user's input language)
```

The 11-dimensional feature space (energy, valence, danceability, acousticness, tempo, etc.) is the *vocabulary*; the parser is the *translator*; retrieval is just weighted Euclidean distance.

## Bilingual from day 1

Vibe Check is fully bilingual (English + 简体中文):

- **Auto language detection** — input Chinese, get Chinese output; input English, get English. Language toggle in the top right swaps the UI manually.
- **Bilingual mood patterns** — all 16 emotional/scene patterns have keywords in both languages. *"刚被甩，想沉浸在悲伤里" → 忧郁感伤 (Melancholy)*, just like English does.
- **Honest scope note**: the song catalog is the Kaggle Spotify dataset, which is primarily Western pop / R&B / EDM / Latin / hip-hop / rock — there's no Mandarin pop in the index. A Chinese-speaking user still gets musically accurate recommendations, just from a Western catalog. A production version would add C-pop / K-pop sources.

## Two implementations, one product

This repo ships **both** versions of the parser, switchable via one flag (`USE_REAL_AI` at the top of the script):

### v0.1 — Deterministic engine (default, shipping)

A hand-crafted keyword-pattern matcher with ~16 emotional/scene patterns, each adjusting feature targets (energy, valence, tempo) and genre weights. Intensifier and softener detection ("very" / "very" / "有点") scales adjustments. The explanation layer is templated but uses the actual selected songs for specificity.

**Why ship this first?** The riskiest product assumption is *"do users prefer mood-based discovery over genre-based"* — not *"can an LLM parse mood text"* (we already know LLMs can). The deterministic engine is good enough to test that, runs offline with zero cost, and never fails in a demo.

### v0.2 — LLM-powered (preserved in code, behind flag)

Set `USE_REAL_AI = true` and the parser swaps to Claude Sonnet via the Anthropic API. The LLM handles compositional prompts ("rainy Sunday but I have a deadline tomorrow"), intensity nuance, and negation more gracefully than keyword matching can — in any language.

**Cost note:** ~$0.01 per session at current Sonnet pricing. Production would route intent parsing to a cheaper model (Haiku) and reserve Sonnet for the explanation layer, plus cache common prompts. Realistic monthly cost at scale is in the low-thousands of dollars per million sessions.

## What's in this repo

```
├── vibe_check.html              # The demo — single file, runs anywhere
├── PRD.md                       # Product requirements doc
├── data/
│   └── spotify_songs.csv        # Source: 32,833 songs × 23 features
├── preprocessing/
│   └── build_index.py           # Sample 2,048 songs, build compact JSON index
└── original_analysis/           # The original school project (kept for context)
    ├── research_question_1.py   #   country comparison + t-tests
    ├── research_question_2.py   #   tempo/valence Pearson correlation
    └── research_question_3.py   #   random forest popularity classifier
```

## Try it locally

```bash
# Just open the HTML in a browser — no build step, no API key, no backend
open vibe_check.html
```

It works completely offline in deterministic mode. Click any of the 6 mood chips, type your own prompt in English or Chinese, or use the EN / 中 toggle in the top right to switch UI language.

## Tech notes

- **No build step, no framework, no backend.** Deliberate. The goal was a polished prototype readable in 5 minutes.
- **2,048 songs, not 30k.** Stratified sample across 6 genres, weighted toward popular tracks but with a long tail. The compressed JSON (~530 KB) inlines into the HTML so the whole demo is one self-contained file.
- **Retrieval is hand-rolled, not a vector DB.** Weighted Euclidean distance in 5-D, runs in <50ms on the client. For 2k songs this is fine; for 100M songs you'd want FAISS or a proper vector DB.
- **No real audio playback.** Hooking up the Spotify Web Playback SDK is straightforward but requires OAuth + Premium — out of scope for a vibe-coding demo.

## What I learned

1. **The hardest design decision was the explanation layer.** A list of songs by itself isn't a product — it's a black box. The "why these songs" paragraph is what turns it from a toy into something a user would trust. Spotify's own recommenders don't show this, and they should.
2. **The product question came before the AI question.** I caught myself wanting to start with "let me use an LLM to..." — but the right question was "what's wrong with how music discovery works today, and is a natural-language interface the right fix?" The LLM is just one possible implementation of the parser. Shipping the deterministic version first proved the product hypothesis without depending on the AI choice.
3. **Localization is mostly a product decision, not a translation task.** Adding Chinese wasn't a matter of running text through Google Translate — it was deciding *which* Chinese words map to which audio-feature deltas. "撸铁" (lifting iron, slangy for weightlifting) should boost energy more than "运动" (exercise, generic). The interesting work is in the dictionary, not the i18n plumbing.

## Credits

- Data: [Spotify Tracks Dataset (Kaggle, 30k songs)](https://www.kaggle.com/datasets/joebeachcapital/30000-spotify-songs)
- Original analysis project: CSE 163 (University of Washington), Section AF
- Vibe Check rebuild + product framing: this is the vibe-coding pass.
