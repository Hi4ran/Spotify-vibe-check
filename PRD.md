# Vibe Check — Product Requirements Doc (v0.1)

> **TL;DR** — A bilingual (EN/ZH) natural-language interface that turns a user's mood/scene description into a 10-song playlist, by translating free text into audio-feature targets and retrieving from a catalog of 2,048 tracks. Built to demonstrate AI-PM thinking: identifying a real product gap in music discovery and shipping the thinnest end-to-end MVP that proves the bet.

---

## 1. Problem

**Music recommenders are biased toward what you've already listened to.** Spotify's Discover Weekly, "Made For You" mixes, and radio stations all leverage collaborative filtering plus your historical listening — which works well in steady state but fails in three common scenarios:

1. **Mood shift** — your usual library is upbeat, but tonight you want melancholy. The algorithm won't catch up for days.
2. **Scene-specific listening** — "I'm cleaning the apartment" or "我在健身房" are situational; your taste profile doesn't capture them.
3. **Cold start** — new users, or users on a new account, get generic recommendations until they listen for weeks.

The catalog has the signal needed to solve this (audio features: energy, valence, danceability, tempo, etc.) — but no consumer UI lets users query against those features directly. They aren't a user-facing language.

## 2. Hypothesis

If we put a natural-language layer between the user and the audio-feature vector space, users can **describe** what they want in their own words and get a playlist that matches the **audio** of that mood — independent of their listening history.

The riskiest assumption: **do users actually prefer mood-based discovery over genre-based** at a level that drives retention? That's the question v0.1 tests. (Whether an LLM specifically is the best parser is a separate, downstream question.)

## 3. Target user & moment

| | |
|---|---|
| **Primary user** | Spotify Free / Premium listener, 18–34, who treats music as functional (focus, workout, cooking, vibes). |
| **Moment** | They're starting an activity *now* and want a fitting playlist in <30 seconds. They don't want to scroll genre lists. |
| **JTBD** | "When I'm about to do X, help me feel like I have the right soundtrack for it — without me having to know what genre or BPM I want." |
| **Locale** | English and Chinese supported in v0.1. Mood is universal; language isn't. |

## 4. MVP scope

**In scope (v0.1):**
- Free-text input (EN or ZH, auto-detected) → parsed audio-feature vector
- Retrieval from 2,048 indexed tracks via weighted distance
- Diversity constraint (max 2 songs per artist)
- Natural-language rationale for the picks, in the input language
- 6 suggestion chips for common moods (reduce blank-page friction)
- Manual EN ↔ 中 UI toggle

**Explicitly out of scope (v0.1):**
- Actually playing audio (Spotify Web Playback SDK is a v0.2 task)
- User accounts / saved playlists / history
- Cross-session learning
- Languages beyond EN/ZH
- C-pop / K-pop / Mandarin music in the catalog (would require a non-Spotify dataset)
- Lyrics-based matching (we use audio features only)

## 5. Success metrics

If this were a real product, I'd ship it with these instrumented:

| Metric | Why it matters | v0.1 target |
|---|---|---|
| **Time-to-first-play (TTFP)** | Core value: speed to playlist | < 5s end-to-end |
| **Playlist completion rate** | Does the user listen to ≥ 50% of the songs? | Baseline metric to learn |
| **Thumbs-up rate on rationale** | Does the *why* actually feel right? | > 60% positive |
| **Repeat sessions / user / week** | Sticky habit, not novelty | > 2 (vs. ~0 for a one-off tool) |
| **Cost per session** | LLM cost reality-check (when AI mode on) | < $0.005 |

**Anti-metric to monitor:** prompt repetition rate. If users keep refining the same prompt, the parser is failing them.

## 6. How it works (3 steps)

1. **Intent parsing** — User text (EN or ZH) → audio-feature targets (energy, valence, danceability, acousticness, tempo) + genre weights + a short interpretation. In v0.1 ships, this is a deterministic keyword-pattern matcher with ~16 bilingual mood/scene patterns. In v0.2, this swaps to an LLM call (Claude Sonnet) — code preserved behind a flag.
2. **Retrieval** — Each track scored by weighted Euclidean distance to the target vector, multiplied by genre weight, with a small popularity tiebreaker. Top 10 picked with diversity constraints.
3. **Explanation** — A 2-paragraph rationale linking the user's prompt to the picks. Templated in v0.1 (uses the actual selected songs for specificity); LLM-generated in v0.2.

## 7. Why this design separates the product question from the AI question

The temptation as an "AI" project is to put the LLM at the center. I deliberately didn't:

- The **product hypothesis** is "users want mood-based, scene-based discovery in their own words." This is testable without an LLM at all — a keyword parser is good enough to deliver a credible MVP and see if users come back.
- The **AI question** is "which implementation of the parser gives the best quality?" — answerable later, with A/B testing of deterministic vs. LLM parsers on a real user base.

This separation is the AI PM discipline: **don't bundle a product bet with a technology bet**. Ship the cheaper thing first; upgrade when the product hypothesis is validated.

## 8. Why an LLM eventually beats hand-coded rules (in v0.2)

Keyword matching works on simple prompts. But:
- **Real prompts are compositional** — "rainy Sunday with coffee but I have a deadline tomorrow" can't be cleanly keyword-matched.
- **Mood ↔ feature mapping is fuzzy** — "melancholy" implies low valence but *not always* low energy (rock ballads exist).
- **LLM handles negation, intensity, context** — "kind of sad but not depressing" needs interpretation, not lookup.
- **Language-agnostic** — adding a new language to v0.2 means writing a new system prompt, not rebuilding a dictionary.

## 9. Risks & open questions

1. **Catalog coverage in localized markets.** The Spotify Kaggle dataset is Western-skewed. A Chinese user gets musically-accurate recommendations from a Western catalog — better than nothing, but not native-feeling. v1.0 needs C-pop / K-pop sources.
2. **LLM output reliability (v0.2).** JSON parsing failures kill the UX. Production needs `tool_use` mode or retries.
3. **Feature-space coverage.** 2,048 songs is a tiny slice. Some niche prompts won't have great matches. The product needs graceful fallback ("we don't have a perfect match, but here's what's closest").
4. **Cost at scale (v0.2).** 2 LLM calls per request × millions of users × Sonnet pricing is non-trivial. Production would use Haiku for intent parsing, reserve Sonnet for explanations, and cache common prompts.
5. **Evaluation is hard.** "Did the playlist match the vibe?" has no ground truth. Production needs human raters + thumbs-up feedback loop.

## 10. What I'd build next (v0.2 → v1.0)

| Version | Bet |
|---|---|
| **v0.2** | LLM parser (flag flip). Test if quality lift justifies cost. |
| **v0.3** | Real Spotify integration (Web Playback API). Save playlist to user's library. |
| **v0.4** | Feedback loop: thumbs up/down per track → personal preference embedding → re-rank future results. |
| **v0.6** | Localized catalogs (C-pop via QQ/网易云 partnerships; K-pop via Melon). |
| **v0.8** | Multimodal input — paste a photo or describe a scene, get a playlist. |
| **v1.0** | A/B test against Spotify's own mood playlists to prove lift. |

## 11. What this demo proves

- I can **identify a real product gap** (mood-based discovery cold-start) rather than building an AI feature looking for a problem.
- I can **scope an MVP that tests the product hypothesis** without bundling it with the AI bet.
- I can **define metrics** that distinguish "novelty" from "real value."
- I can **think about localization as a product decision**, not a translation task — picking the right Chinese keywords for each mood vector takes the same product judgment as picking the right English ones.
- I can **ship the thing** — design, build, and write the rationale, end-to-end.

---

*Built as a vibe-coding portfolio project. Data: Spotify Tracks (Kaggle, 30k songs, sampled to 2,048 for in-browser retrieval). Code & demo: see repo README.*
