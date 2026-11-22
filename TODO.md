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
- Editing: double-click empty canvas = node, Shift+drag node→node = edge (drop on itself = self-loop), drag = move, right-click node/edge = menu, Del = delete, audio file can be dropped straight onto a node.
- Walkers: reach a node → play its clip → only then draw the next edge by weight. Independent epochs so stop/edit never leaves a walker mid-flight.
- Autosave: IndexedDB (graph + audio). Export/import is the portable path.
- Editing UI: no side panel. Node/edge context menus, double-click on a node for its sound, wheel on an edge for weight, inline rename. Node color comes from an 8-color palette and is stored per node (`color`).

## Feedback

- [x] 조작 개편
    - [x] 오른쪽 패널 삭제 (편집 UI 전부 캔버스 위로 이동)
        - [x] 노드를 더블클릭하면 소리 선택 윈도우 오픈.
            - 빈 캔버스 더블클릭은 기존대로 노드 추가 (확인함)
        - [x] 노드를 우클릭하면 메뉴
            - [x] 이름 바꾸기 — 메뉴 안 인라인 입력, Enter 확정 / Esc 취소, F2 단축키
            - [x] 색 지정 — 프리셋 팔레트 8색 (확인함), 그래프 파일에 저장됨
            - [x] 소리 제거 — 소리 있으면 "소리 바꾸기 / 미리 듣기 / 소리 제거" 노출
            - [x] 여기서 시작 — 패널이 사라져 갈 곳이 없어 메뉴에 추가 (확인함)
        - [x] 간선을 우클릭 하면 메뉴
            - [x] 삭제
            - [x] 가중치 입력 (메뉴 안 숫자 입력 + 나가는 합계 표시)
        - [x] 간선 위에서 커서를 스크롤하면 가중치 조절
            - 휠 = ±0.1, Shift+휠 = ±1, 범위 0..99 (확인함)

### 미뤄둔 피드백.

아직 진행 금지.

- [ ] 회귀 간선 형태 변경: 항상 노드의 위쪽에 있으니까 이상함. 다른 간선이 없는 방향으로 + 원형 보다는 물방울 같은 모양의 곡선
- [ ] 간선 형태 변경: 둥글게. 모든 간선이 직선인건 이상함.
- [ ] 전체적 look 변경: 파스텔 톤 + 둥글둥글한 ui + 두꺼운 border

## Implementation Rules

- When feedback work is ambiguous, indent under that item and list the points to confirm before coding.
- Commit after every change.
