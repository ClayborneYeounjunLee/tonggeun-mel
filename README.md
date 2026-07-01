# 🚆 통근권 멜버른 (tonggeun-mel)

> 멜버른 광역철도(Metro Trains) 네트워크를 기반으로, **출발지에서 기차 + 도보(옵션: 버스·트램)로 30/45/60분 안에 닿을 수 있는 통근 범위(등시권)** 를 지도 위에 그려주는 단일 파일 웹앱입니다.
> 지도 위를 탭하거나 주소를 검색해 출발지를 정하면, 다익스트라(Dijkstra) 최단시간 그래프 탐색으로 도달 가능한 역과 그 주변 도보/버스 반경을 실시간으로 시각화합니다.

🔗 **라이브 링크: <https://clayborneyeounjunlee.github.io/tonggeun-mel/>**

---

## ✨ 주요 기능

- 🗺️ **지도 위 클릭 / 드래그로 출발지 지정** — 지도를 탭하거나 마커를 드래그해 시작점을 옮기면 즉시 재계산됩니다.
- 🔎 **주소·지역 검색** — OpenStreetMap Nominatim 지오코딩으로 "Flinders St", "Carlton" 같은 주소·지명을 빅토리아주(멜버른권)로 한정해 검색합니다. 키 불필요.
- 🎯 **3개 시간 밴드 등시권** — 기본 30 / 45 / 60분(직접 편집 가능). 밴드를 눌러 원하는 시간대 하나만 표시합니다.
- 🚶 **기차 + 도보 도달권 근사 모드(기본)** — 각 역까지의 기차 소요시간을 그래프로 계산하고, 남은 시간만큼 역 주변에 도보 반경을 겹쳐 히트맵처럼 채웁니다.
- 🚌 **버스·트램 포함 옵션** — 켜면 역 접근/역에서의 이동에 근사 버스·트램(평균 속도·대기시간 반영)을 더해 도달 범위를 넓힙니다.
- 🎯 **정확 모드(Accurate mode · TravelTime)** — 사용자가 자신의 무료 TravelTime API 키를 넣으면, **실제 시간표 기반** 대중교통 등시권(평일 오전 8시 출발 기준)을 서버에서 받아 그립니다.
- ⚙️ **세밀한 계산 파라미터** — 도보 속도, 버스·트램 속도/대기, 환승 패널티, 역에서의 최대 도보 시간, 밴드 시간(분)을 슬라이더·입력으로 조정합니다.
- 📍 **도달역 점 표시** — 시간대별 색(초록/주황/빨강)으로 도달 가능한 역에 점을 찍고, 탭하면 "역 이름 · 약 N분" 팝업이 뜹니다.
- 🌗 **다크/라이트 테마** & 🌐 **한국어/영어 토글** — 테마는 모아(moa) 앱들과 `hub-theme` 키를 공유합니다.
- 💾 **설정·출발지 자동 저장** — 브라우저 `localStorage`에 저장되어 다음 방문 시 그대로 복원됩니다.
- 📱 **모바일 우선 UI** — 하단 시트(bottom sheet), 안전영역(safe-area) 대응, 탭 하이라이트 제거 등 모바일 최적화.

---

## 🧱 기술 스택 / 언어

| 항목 | 내용 |
|---|---|
| **언어** | 순수 **HTML + CSS + JavaScript (ES5 스타일, `"use strict"`)** — 빌드 도구·트랜스파일러 없음 |
| **모듈 방식** | ES 모듈 아님. 전역 IIFE `(function(){ ... })()` 하나 + 데이터는 전역 `window.NET` |
| **지도 라이브러리** | **Leaflet 1.9.4** (unpkg CDN, `crossorigin`) — 무료·키 불필요 |
| **지도 타일** | **CARTO basemaps** (`light_all` / `dark_all`) + OpenStreetMap 데이터 — 무료·키 불필요 |
| **지오코딩** | **OpenStreetMap Nominatim** (검색 + 역지오코딩) — 무료·키 불필요 |
| **정확 모드 API** | **TravelTime API v4** (`/v4/time-map` 등시권) — 사용자 본인의 무료 키 필요(선택) |
| **폰트** | 시스템/웹폰트 스택: `Pretendard`, `Apple SD Gothic Neo`, `Malgun Gothic`, `Noto Sans KR`, sans-serif (별도 임포트 없이 로컬/시스템 우선) |
| **빌드/번들러** | 없음 — 정적 파일 2개(`index.html`, `network-data.js`)만으로 동작 |
| **백엔드 / DB** | 없음 (Firebase 없음, 서버 없음). 상태는 전부 클라이언트 `localStorage` |

### 실제 `<script>` / CDN (코드에서 확인)

```html
<!-- Leaflet 1.9.4 (무료·키 불필요) -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" crossorigin="">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" crossorigin=""></script>

<!-- 로컬 데이터 & 앱 로직 -->
<script src="network-data.js"></script>   <!-- window.NET = { stations, edges } -->
<script> /* 인라인 앱 로직 (IIFE) */ </script>
```

> 참고: 문서 상단 힌트의 "TravelTime 계열"은 맞으나, 근사 모드는 **자체 그래프 계산**이고, TravelTime은 오직 선택적 "정확 모드"에서만 호출됩니다.

---

## 🏗️ 시스템 구조

### 파일/부팅 흐름

1. **단일 페이지** `index.html` — 헤더, 하단 시트(검색·밴드·설정), TravelTime 키 모달, 토스트, 지도 컨테이너를 모두 포함.
2. `<head>`의 인라인 스크립트가 **깜빡임 없이 테마를 먼저 적용**합니다(`localStorage.hub-theme` 또는 OS 다크모드 선호).
3. `network-data.js`가 먼저 로드되어 전역 `window.NET`(역·간선 데이터)을 채웁니다.
4. 메인 IIFE가 실행되며:
   - i18n 사전 적용(`applyI18n`) → 다크모드 적용(`applyDark`) → `initMap()`으로 Leaflet 지도 생성.
   - `localStorage`에 저장된 출발지가 있으면 복원 후 자동 계산.

### 핵심 모듈·함수

| 함수 | 역할 |
|---|---|
| `buildAdj(transferPenalty)` | 역·노선 데이터로 **인접 리스트(그래프)** 를 만든다. 노드는 `역명␟노선코드` 형태로, 같은 역의 서로 다른 노선 노드끼리는 **환승 패널티** 간선으로 연결(환승 시간 반영). |
| `computeArrival(oLat,oLng)` | 출발 좌표에서 각 역까지의 **최소 도착시간(분)** 을 계산. 출발지 근처 역들을 도보/버스 접근시간(`accessMin`)으로 시드한 뒤, `MinHeap` 기반 **다익스트라**로 전역 최단시간 전파. 반환: `Map<역명, 분>`. |
| `MinHeap` | 다익스트라용 최소 힙(우선순위 큐) 직접 구현. |
| `haversine()` | 두 좌표 간 대원거리(m). 접근·도보 반경 계산에 사용. |
| `accessMin / walkReach / busReach` | 도보/버스 접근시간, 남은 시간에 따른 도보/버스 도달 반경(m) 계산. |
| `render()` | 도달 결과를 지도에 그림. 정확 모드면 `drawTT()`, 아니면 역 주변에 `L.circle` 도보/버스 반경을 겹쳐 히트맵 표현 + 도달역 점(`drawDots`). |
| `drawTT / fetchTTMulti / depTimeISO` | **정확 모드**: TravelTime `/v4/time-map`에 다중 밴드 등시권 요청 → GeoJSON 폴리곤을 캐시(`ttCache`)하고 그림. 출발시각은 "다음 평일 오전 8시(멜버른)"를 UTC ISO로 변환. |
| `setOrigin / recompute / fitToReach` | 출발지 설정·저장·재계산·지도 뷰 맞춤. |
| `doSearch / reverseGeo` | Nominatim 검색/역지오코딩. |

### 상태 관리 / 라우팅

- **라우팅 없음** — 단일 화면 SPA. URL 파라미터·해시 라우팅 없음.
- **상태 객체 `S`** 하나에 모든 계산 설정을 담고, 변경 시 `localStorage`(`mel_settings`)에 직렬화 저장.
- 지도 레이어는 `areaLayer`(도보·버스 반경), `ttLayer`(TravelTime 폴리곤), `dotLayer`(역 점)로 분리 관리.

---

## 🗂️ 데이터

모든 네트워크 데이터는 **`network-data.js`** 한 파일에 전역 객체 `window.NET`로 하드코딩되어 있습니다. 별도 DB·서버·JSON fetch 없음.

- **좌표 출처**: OpenStreetMap (ODbL). 파일 상단 주석에 명시.
- **역간 소요시간**: 실측 시간표가 아니라 **역 좌표 직선거리 기반 추정치(약 48km/h 가정)** — 파일 주석에 명시. (정확 모드에서만 실제 시간표가 반영됨.)

### 스키마

```js
window.NET = {
  // [역이름, 위도, 경도]  — 224개
  stations: [
    ["Aircraft", -37.8666, 144.76073],
    ["Flinders Street", -37.81842, 144.96648],
    ["Southern Cross", -37.81838, 144.95237],
    // ...
  ],
  // [출발역, 도착역, 소요분, 노선코드]  — 262개 간선
  edges: [
    ["Craigieburn", "Roxburgh Park", 5, "CRG"],
    ["North Melbourne", "Southern Cross", 2, "CRG"],
    ["Belgrave", "Upwey", 3, "BEL"],
    // ...
  ]
};
```

| 데이터 | 개수 | 구조 |
|---|---|---|
| `stations` | **224개** | `[name, lat, lng]` 튜플 배열 |
| `edges` | **262개** | `[from, to, minutes, lineCode]` 튜플 배열 (무향으로 양방향 처리) |
| 노선 코드 | **20종** | 아래 표 참조 |

### 노선 코드 (edges의 4번째 요소)

| 코드 | 노선(추정) | 코드 | 노선(추정) |
|---|---|---|---|
| `CRG` | Craigieburn | `PKM` | Pakenham |
| `UPF` | Upfield | `GLW` | Glen Waverley |
| `SBU` | Sunbury | `ALM` | Alamein |
| `WBE` | Werribee | `LIL` | Lilydale |
| `WBE-A` | Werribee (Altona loop) | `BEL` | Belgrave |
| `WIL` | Williamstown | `MER` | Mernda |
| `SAN` | Sandringham | `HBG` | Hurstbridge |
| `FKN` | Frankston | `FLR` | Flemington Racecourse |
| `STP` | Stony Point | `LOOP` | City Loop |
| `CRN` | Cranbourne | `MT` | Metro Tunnel(신규 지하 구간) |

> 노선 이름은 코드와 노선 구성으로부터의 합리적 매핑이며, 데이터에 문자열로 명시된 것은 **코드**뿐입니다. 도심 구간은 `LOOP`(City Loop)·`MT`로 별도 표현되어 환승 그래프에 반영됩니다.

---

## 💾 저장소 / DB

**서버·클라우드 DB 없음.** Firebase 없음, MongoDB 없음. 모든 상태는 브라우저 `localStorage`에만 저장됩니다(게스트/오프라인 기본 동작).

### localStorage 키

| 키 | 용도 |
|---|---|
| `hub-theme` | 다크/라이트 테마 (모아 허브·형제 앱들과 **공유**) |
| `mel_lang` | 언어 설정 (`ko` / `en`) |
| `mel_settings` | 계산 설정 객체 `S` 전체(JSON) — 도보속도·환승·최대도보·밴드·버스옵션·정확모드 여부 등 |
| `mel_origin` | 마지막 출발지 `{lat, lng, label}` (JSON) |
| `mel_tt_appid` | TravelTime **Application ID** (사용자 입력, 이 기기에만 저장) |
| `mel_tt_key` | TravelTime **API Key** (사용자 입력, 이 기기에만 저장) |

### `mel_settings`에 저장되는 기본 상태 `S`

```js
{ walk:4.5, transfer:5, maxwalk:15, dots:true,
  bus:false, busSpeed:16, busWait:6,
  bands:[30,45,60], on:[false,true,false], tt:false }
```

### 오프라인 / 게스트 폴백

- 역·노선 데이터는 로컬 파일이라 **인터넷 없이도 그래프 계산과 도달권 렌더가 동작**합니다(지도 타일·검색·정확 모드만 온라인 필요).
- 출발지 근처에 접근 가능한 역이 하나도 없으면, **가장 가까운 역**을 강제로 시드해 계산이 끊기지 않도록 폴백 처리합니다.
- TravelTime 키가 없으면 정확 모드는 자동으로 꺼진 채 근사 모드로 동작합니다.

---

## 🌐 외부 API · 의존성

| 서비스 | 용도 | 키 필요? | 키 넣는 위치 |
|---|---|---|---|
| **Leaflet 1.9.4** (unpkg CDN) | 지도 렌더링 엔진 | ❌ | — |
| **CARTO basemaps** (`{s}.basemaps.cartocdn.com`) | 라이트/다크 지도 타일 | ❌ | — |
| **OpenStreetMap Nominatim** (`nominatim.openstreetmap.org`) | 주소 검색(`/search`) · 역지오코딩(`/reverse`) | ❌ | — |
| **TravelTime API v4** (`api.traveltimeapp.com/v4/time-map`) | 정확 모드: 실제 시간표 기반 대중교통 등시권 | ✅ (선택) | 앱 내 ⚙️ 모달 → `X-Application-Id` / `X-Api-Key` 헤더로 전송, `localStorage`에만 저장 |

- 검색은 `countrycodes=au` + 빅토리아주 뷰박스(`144.20,-38.55,145.80,-37.30`)로 한정해 정확도를 높입니다.
- **정확 모드 요청 예시**(코드 기준): `public_transport` 타입, 다음 평일 오전 8시(멜버른) 출발, `level_of_detail: {scale_type:"simple", level:"medium"}`, GeoJSON(`Accept: application/geo+json`) 응답.
- TravelTime 키는 **서버·타 저장소로 업로드되지 않고** 오직 사용자 기기 `localStorage`에만 보관됩니다.

> 🔐 **보안 메모**: 저장소 코드에 하드코딩된 비밀키·토큰은 **없습니다**. TravelTime 자격증명은 전적으로 사용자 입력(런타임) → `localStorage` 저장 방식입니다. Firebase/Mongo/OpenAI/AWS 키 등도 코드에 존재하지 않습니다.

---

## ▶️ 로컬 실행 방법

빌드·설치 과정이 전혀 없습니다. `package.json`도 없으므로 npm 스크립트는 없습니다. **정적 파일을 서버로 띄우기만** 하면 됩니다.

> 파일을 `file://`로 직접 열어도 대부분 동작하지만, 일부 브라우저에서 fetch/CORS 제약이 있어 로컬 정적 서버 사용을 권장합니다.

```bash
# 저장소 폴더에서 아래 중 하나 실행

# Python 3
python -m http.server 8000

# Node (http-server 전역 설치 시)
npx http-server -p 8000
```

브라우저에서 <http://localhost:8000> 접속.

---

## 🚀 배포

**GitHub Pages** 정적 호스팅으로 배포됩니다 (라이브: <https://clayborneyeounjunlee.github.io/tonggeun-mel/>).

1. 저장소 → **Settings → Pages**.
2. **Source**: `Deploy from a branch` → 브랜치 `main`, 폴더 `/ (root)`.
3. 저장 후 몇 분 뒤 `https://<username>.github.io/tonggeun-mel/`에서 공개.
4. 정적 파일만 있으므로 별도 빌드 액션 불필요 — `main`에 푸시하면 자동 반영.

---

## 📁 파일 구조

```text
tonggeun-mel/
├── index.html        # 전체 앱: UI(헤더·하단시트·모달) + CSS 디자인토큰 + 인라인 JS 로직
│                     #   - Leaflet 지도, 그래프 계산(다익스트라), 도달권 렌더
│                     #   - i18n(한/영), 다크모드, 정확 모드(TravelTime) 호출
│                     #   - localStorage 상태 저장/복원
└── network-data.js   # window.NET = { stations(224), edges(262) } — 멜버른 철도 네트워크 데이터
                      #   좌표: OpenStreetMap(ODbL), 역간 시간: 직선거리 추정(약 48km/h)
```

- **총 2개 파일**로 완결되는 프론트엔드-온리 앱입니다(그 외는 `.git`).

---

## 🔗 관련 앱 (모아 허브 · 형제 앱)

- 이 앱은 **모아(moa) 허브** 생태계의 한 앱입니다. 헤더의 ◈ 버튼으로 허브로 돌아갑니다: <https://clayborneyeounjunlee.github.io/moa/>
- **테마 공유**: `hub-theme` localStorage 키로 moa/haru/kanade 등 형제 앱들과 다크·라이트 모드를 공유합니다.
- **공통 디자인 시스템**: CSS 변수(디자인 토큰) 기반으로 moa/haru/kanade/tonggeun이 동일한 색·라운드·그림자 체계를 사용합니다.
- 모아 허브에는 `tonggeunmel` id로 등록되어 있습니다.

---

<sub>이 문서는 저장소의 실제 코드(`index.html`, `network-data.js`)를 직접 읽고 확인한 사실만 담았습니다. 역간 소요시간은 추정치이며, "정확 모드"에서만 실제 시간표(TravelTime)가 반영됩니다.</sub>
