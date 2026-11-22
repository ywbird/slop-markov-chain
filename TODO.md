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
- Physics is a placement tool: it settles, stops, and saves the result. The camera follows the centre of mass only while it runs, and holds still as soon as any node is pinned (a pan would slide the pinned node on screen).
- Look: dark navy background with pastel nodes/edges/walkers, 2px borders everywhere, 12–18px corner radii, Jua from Google Fonts with a system fallback. Every edge, self-loops included, is one quadratic curve; a self-loop just has both feet on the same node and is aimed at that node's emptiest side. Percentage chips sit outside the curve and are click targets of their own.

## Feedback

- [x] 여기저기 숫자 인풋에 브라우저 기본 위아래 버튼 없애
    - 모든 `input[type=number]`에 `appearance:textfield` + WebKit 스피너 `appearance:none` (확인함)
    - 대상은 워커 수 입력 하나뿐, 슬라이더는 원래 스피너 없음 (확인함)

- [x] 딜레이가 0초면, 워커의 간선 이동에도 0초가 소비돼야함. 간선 이동 애니메이션은 출력하되, 소리 딜레이는 0초.
    - 원인: 이동 애니메이션(180ms)을 `await`해서 소리 사이 실제 간격이 190ms였음 (실측)
    - 이동은 순수 연출로 바꿈. 소리는 즉시 시작하고 토큰이 그 소리 위를 미끄러짐 (사용자 선택: a) (확인함)
        - delay 0 → 소리 사이 간격 0ms, delay 0.3 → 300ms (실측)
    - 소리가 짧아 이동 중에 다음 이동이 시작되면 튀지 않고 현재 토큰 위치에서 이어서 이동 (확인함)

- [x] visual
    - [x] 회귀 간선 형태 변경: 항상 노드의 위쪽에 있으니까 이상함. 다른 간선이 없는 방향으로 + 원형 보다는 물방울 같은 모양의 곡선
        - 방향: 노드 주변 48방향을 훑어 다른 간선과 가장 먼 방향 (확인함)
        - 모양: 처음엔 이차 베지어 2개로 뾰족한 물방울 → 다른 간선과 모양 언어가 달라 지적받음
        - 고친 모양: 다른 간선과 똑같은 **이차 베지어 1개**. 양 끝은 테두리, 제어점은 빈 방향으로 R+124 (확인함)
            - 뾰족한 점 없음. 200샘플에서 곡률 변화가 튀지 않고 총 회전량 > 2rad (확인함)
            - 테두리에서 나와 테두리로 돌아오고, 곡선이 노드 안으로 파고들지 않음 (최소 거리 = R) (확인함)
        - 화살표는 복귀점(테두리)에, 퍼센트 칩은 곡선 꼭짓점 바깥 26px (확인함)
        - 다른 간선이 하나도 없으면 위쪽 (확인함)
    - [x] 간선 형태 변경: 둥글게. 모든 간선이 직선인건 이상함.
        - 모든 간선을 이차 베지어로. 길이에 비례해 휘고(12~52px) 왼쪽으로 휨 (확인함)
        - 반대 방향 쌍은 서로 반대로 휘어 자동으로 분리 (확인함)
        - 히트 테스트도 곡선을 따라감 (직선 위에서는 안 잡힘), 퍼센트 칩도 클릭 영역 (확인함)
        - 워커 이동도 같은 곡선을 따라감 (확인함)
    - [x] 전체적 look 변경: 파스텔 톤 + 둥글둥글한 ui + 두꺼운 border
        - 배경은 어두운 남색 유지, 노드·간선·워커만 파스텔 (사용자 선택: b) (확인함)
        - border 2px (버튼·입력·메뉴), 노드 테두리 2.4~4px, 간선 2.4~3.6px (확인함)
        - 모서리: 버튼 12px, 메뉴 18px, 토스트 16px, 칩 9px (확인함)
        - 노드 반지름 27 → 30 (확인함)
    - [x] 폰트 크기 증가
        - 본문 14 → 16px, 버튼 13 → 15px, 메뉴 항목 13 → 15px, 힌트 12 → 14px (확인함)
        - 캔버스: 노드 라벨 최대 13 → 16px, 간선 퍼센트 11 → 13px, 워커 토큰 10 → 12px (확인함)
    - [x] https://fonts.google.com/specimen/Jua 사용.
        - Google Fonts `<link>` (인터넷 가정), 실패 시 시스템 폰트로 폴백 (확인함)
        - 캔버스 글자도 Jua 사용 (확인함)

- [x] 중력 무시 노드가 움직임. 화면 단위로 고정하라는 뜻.
    - 원인: 핀 노드의 world 좌표는 고정이었지만 물리 중 카메라가 움직여 화면에서 밀렸음
    - 핀 노드가 하나라도 있으면 물리 중 카메라 추종을 멈춤 (확인함) — 카메라가 움직이면 고정이 깨지므로
        - 처음엔 카메라 이동분만큼 핀 노드 world 좌표를 역보정했으나, 그 노드가 이웃을 계속 밀어 물리가 정착하지 못함 (실측: 카메라 944 → 1162 계속 이동)
    - 핀 없을 때는 기존대로 카메라 추종 (확인함)
        - 정착 후 카메라 재조정은 하지 않음
    - 마우스로 직접 끄는 건 허용, 놓은 자리가 새 고정 위치 (확인함)

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
        - 노드별 gravity 슬라이더 0~3 · repel 슬라이더 0~10, 0.1 단위, 기본 1.0 (확인함)
        - 간선은 전역 고정 세기의 스프링 (확인함)
        - 노드 우클릭에 "중력 무시 (고정)" 핀 (확인함)
        - 중력이 도는 동안만 카메라가 노드 무게중심을 따라가고, 드래그 중·정착 후엔 고정 (확인함)
- [x] 중력 repel을 최대로 해도 별 효과가 없음. 최대치 늘리기
    - 슬라이더 0~10 (0.1 단위), 기존 저장값 그대로 (확인함)
    - 힘 공식도 같이 완화: 거리 감쇠 d² → d^1.5, 상한 220 → 900 (확인함)
    - 4개 노드를 같은 자리에서 정착시켰을 때 repel 1 = 최소 간격 1175, repel 10 = 2618 (확인함)
- [x] 우클릭 메뉴에 확률(가중치) 표시 없애
    - 노드 메뉴의 나가는 간선별 `→ 이름 [__]%` 줄 전부 제거 (확인함)
    - 간선 메뉴의 숫자 입력은 슬라이더로 교체 (0~100, 1 단위, 값 라벨 + 휠 동기화) (확인함)
    - 나가는 간선이 하나뿐이면 슬라이더 비활성 + 100% 고정 안내 (확인함)
- [x] 소리 끝나고 워커가 이동하기 전에 딜레이 옵션을 배경 우클릭 옵션에 넣기
    - 배경 메뉴 `소리 사이 딜레이` 슬라이더, 0~2초, 0.05초 단위, 기본 0 (확인함)
    - 재생 중에 바꾸면 다음 이동부터 바로 적용 (단계마다 값을 읽음) (확인함)
    - 그래프 파일에 `delay`로 저장 (확인함)
- [x] 중력 켜져있을때 노드움직이는 동안 마우스가 움직이지 않으면 중력의 영향을 받음. 드래그 중, 드롭 이전까지 중력 영향 받으면 안됨.
    - 드래그 중인 노드는 `pinned`와 똑같이 적분에서 제외 (확인함)
    - 드롭하면 다시 중력에 참여 (확인함)

### 미뤄둔 피드백.

아직 진행 금지.

## Implementation Rules

- When feedback work is ambiguous, indent under that item and list the points to confirm before coding.
- Commit after every change.
