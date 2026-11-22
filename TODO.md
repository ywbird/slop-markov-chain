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

## Implementation Notes

- Single file: `index.html` (vanilla HTML/JS/CSS, no build, no deps). Self test: open with `?selftest=1`.
- Save format: one JSON file (`markov-sound-graph-*.json`). Audio is embedded as base64 data URLs inside the JSON — zip-with-renamed-extension was rejected as it needs a zip writer.
- Editing: double-click empty canvas = node, Shift+drag node→node = edge (drop on itself = self-loop), drag = move, click edge = weight panel, Del = delete, audio file can be dropped straight onto a node.
- Walkers: reach a node → play its clip → only then draw the next edge by weight. Independent epochs so stop/edit never leaves a walker mid-flight.
- Autosave: IndexedDB (graph + audio). Export/import is the portable path.

## Feedback

- [ ] 조작 개편
    - 오른쪽 패널 삭제
        - [ ] 노드를 더블클릭하면 소리 선택 윈도우 오픈.
        - [ ] 노드를 우클릭하면 메뉴
            - [ ] 이름 바꾸기
            - [ ] 색 지정
            - [ ] 소리 제거
        - [ ] 간선을 우클릭 하면 메뉴
            - [ ] 삭제
        - [ ] 간선 위에서 커서를 스크롤하면 가중치 조절

### 미뤄둔 피드백.

아직 진행 금지.

- [ ] 회귀 간선 형태 변경: 항상 노드의 위쪽에 있으니까 이상함. 다른 간선이 없는 방향으로 + 원형 보다는 물방울 같은 모양의 곡선
- [ ] 간선 형태 변경: 둥글게. 모든 간선이 직선인건 이상함.
- [ ] 전체적 look 변경: 파스텔 톤 + 둥글둥글한 ui + 두꺼운 border

## Implementation Rules

- When feedback work is ambiguous, indent under that item and list the points to confirm before coding.
- Commit after every change.
