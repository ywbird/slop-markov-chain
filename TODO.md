# Markov Chain Sound Graph

## Project Intent

**Goal (Why):** Build a sound-making Markov chain toy. The user saw the "Shikanokonokonokokoshitantan" Markov chain meme and wants to build the same structure with their own sounds, and see it as an interactive node graph.

**User (Who):** The author, experimenting with their own audio files, plus an audience when demoing the meme concept.

**Success:** During a demo, changing a node's sound and nudging the probabilities visibly changes the next playback's path, and multiple walkers' sounds overlap audibly. That single moment is the success criterion — not analytics or statistics.

**Constraint:**
- No server, no build step, no install, no external libraries. Single HTML file (vanilla HTML/JS/CSS).
- Audio playback via Web Audio API.
- Nodes carry a short audio file each; reaching a node plays its sound, and the walker moves to the next node only after that sound finishes.
- Multiple walkers (configurable count) start from a specific or random node and run independently at the same time.
- Graph topology (nodes, edges, probabilities) is built by hand on a canvas, not learned from text.
- Persistence: browser storage during use, plus save/load as a single portable file (graph + audio together; base64 JSON or zip-with-renamed-extension, decided at implementation time).
- Scale: a few dozen nodes, single-digit walkers. Performance and scalability are not goals.
- Design polish matters (demo use) but comes later, after core behavior works.

**Out of scope:**
- Automatic Markov generation from text/corpus (co-occurrence learning).
- Statistics / probability distribution panels.
- Accounts, server sync, cloud storage.
- Mobile optimization.
- Audio editing (trimming, effects, normalization).
- Managing multiple graphs as a library.

## Feedback

- [ ] (no feedback yet)

## Implementation Rules

- When feedback work is ambiguous, indent under that item and list the points to confirm before coding.
- Commit after every change.
