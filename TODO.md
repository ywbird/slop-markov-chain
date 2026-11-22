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
- Editing UI: no side panel and no top bar. Background / node / edge right-click menus, double-click on a node for its sound, wheel on an edge for probability, inline rename. Node color comes from an 8-color palette and is stored per node (`color`).
- Weights are percentages (`weight` = % of a node's outgoing traffic); `setEdgeWeight` keeps each node's outgoing sum at exactly 100. Node fields `gravity`, `repel`, `pinned` drive the optional layout simulation (`physics` flag is saved).
- Physics is a placement tool: it settles, stops, and saves the result. The camera follows the centre of mass only while it runs.

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
    - [x] 간선 가중치를 각각 값을 갖는게 아니라 한 노드에서 나오는 합은 항상 100. 스크롤로 조정 하면 그 그 간선의 값만 올리고, 나머지는 기존 비율 유지. 10, 10, 80 이었고, 첫번째 간선을 조정했다면, 나머지 간선은 1:8 비율을 유지하는 방식
        - 단위: 휠 1%p, Shift+휠 10%p (확인함)
        - 100으로 올리면 나머지 0 + 직전 비율 저장, 내리면 복원 (확인함)
        - 새 간선은 100/n, 나머지는 100*(n-1)/n을 기존 비율로 (확인함)
        - 나가는 간선이 1개면 항상 100이라 휠 무시
        - 저장 파일의 옛 절대 가중치는 불러올 때 퍼센트로 변환
    - [x] 상단 바 제거. 배경 우클릭으로 해결.
        - 배경 메뉴: 노드 추가 / 워커 수 / 시작 방식 / 중력 / 저장·불러오기·비우기
        - 시작·정지는 좌하단 아이콘 버튼 (배경 없음, 확인함)
- [x] toggle 버튼 추가: 중력
    - 노드끼리 gravity + repel 각 노드마다 그 값 지정 가능
        - 배치 도구. 감쇠하며 정착한 뒤 스스로 멈춤 (확인함)
        - 노드별 gravity·repel 슬라이더 0~3, 0.1 단위, 기본 1.0 (확인함)
        - 간선은 전역 고정 세기의 스프링 (확인함)
        - 노드 우클릭에 "중력 무시 (고정)" 핀 (확인함)
        - 중력이 도는 동안만 카메라가 노드 무게중심을 따라가고, 드래그 중·정착 후엔 고정 (확인함)
- [ ] 소리 끝나고 워커가 이동하기 전에 딜레이 옵션을 배경 우클릭 옵션에 넣기

### 미뤄둔 피드백.

아직 진행 금지.

- [ ] 회귀 간선 형태 변경: 항상 노드의 위쪽에 있으니까 이상함. 다른 간선이 없는 방향으로 + 원형 보다는 물방울 같은 모양의 곡선
- [ ] 간선 형태 변경: 둥글게. 모든 간선이 직선인건 이상함.
- [ ] 전체적 look 변경: 파스텔 톤 + 둥글둥글한 ui + 두꺼운 border

## Implementation Rules

- When feedback work is ambiguous, indent under that item and list the points to confirm before coding.
- Commit after every change.
