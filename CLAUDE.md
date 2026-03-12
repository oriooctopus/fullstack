# Fullstack Presentation Project

## Overview

This project contains both a **presentation** (Slidev) and a **speech script** that accompany each other. They are two views of the same talk.

## Key Rule: Keep Presentation and Speech in Sync

Changes to the presentation slides MUST be reflected in the speech, and vice versa. They tell the same story — if you add, remove, or reorder content in one, do the same in the other. Never update one without considering the other.

## Tone and Style

- **Voice**: 28-year-old Gen Z engineer living in Brooklyn. Natural, dry, direct. Not trying to be cool — just is. Think "explaining something to a coworker at your desk," not "youth pastor at a tech meetup."
- **No cringe**: Avoid forced enthusiasm, fake-casual phrases like "hit up," "you're in," "and you're done," "pretty straightforward," "let's go," "let's zoom in real quick," etc. This is a presentation, not a YouTube tutorial. Don't narrate your own transitions — just make the point.
- **Humor**: Dry, observational. Understated. If something is annoying, just say it's annoying. Don't wrap it in exclamation marks.
- **Short and to the point** — no walls of text on slides or in the speech; get in, make the point, get out.
- **Confident but not preachy** — state things directly without hedging, but also without lecturing.

## Rubrik Tech Context

- **Caching**: Redis (GCP Memorystore for Redis Cluster in prod, local Redis in dev). In-memory: `hashicorp/golang-lru` for Go, `cachetools` for Python.
- **Codebase reference**: `/Users/oliverullman/Documents/coding/sdmain-1/`

## Presentation Tech

- Presentation: [Slidev](https://sli.dev/) — slides live in `slides.md`
- Speech: written script lives in `speech.md`
- Run dev server: `npx slidev`
