# 3D Digital Twin Projects: Analysis & Execution Guide

This document provides a comprehensive analysis of two Three.js-based digital twin projects:
1. **Smart Classroom** (`smart_classroom_camera_stable.html`)
2. **Bus Assembly Digital Twin** (`ThreeJS_korean_bus.html` / `index.html`)

Both files share a highly optimized, single-file architecture that relies entirely on procedural generation (code-based modeling) rather than loading external 3D assets like GLTF or OBJ files.

---

## 🚀 1. 실행 방법 (How to Execute)

두 프로젝트 모두 바닐라 자바스크립트(ES Modules)와 CDN을 통해 Three.js를 동적으로 불러오는 단일 HTML 파일입니다. 외부 에셋(이미지, 3D 파일)이 없지만, 보안 정책(CORS)과 ES 모듈(`type="module"`)의 특성상 **반드시 로컬 웹 서버 환경에서 실행**해야 정상적으로 작동합니다.

### 실행 옵션
1. **VS Code 확장 프로그램 사용 (가장 추천)**
   * VS Code에서 HTML 파일을 엽니다.
   * 확장 프로그램 탭에서 **Live Server** (Ritwick Dey 제작)를 설치합니다.
   * 에디터 하단의 `Go Live` 버튼을 누르거나, HTML 파일에서 우클릭 후 `Open with Live Server`를 선택합니다.
2. **Node.js 사용**
   * 터미널을 열고 파일이 있는 폴더로 이동합니다.
   * `npx serve` 명령어 입력 후, 출력되는 `http://localhost:3000` 링크로 접속합니다.
3. **Python 사용 (Mac/Linux/Windows 공통)**
   * 터미널에서 파일이 있는 폴더로 이동 후 `python -m http.server` 입력
   * 브라우저에서 `http://localhost:8000`으로 접속합니다.

*(참고: 파일을 브라우저에 단순히 드래그 앤 드롭(file:// 프로토콜)하면 Three.js 모듈 로드 에러가 발생합니다.)*

---

## 🏗️ 2. 전체적인 문법 구조 (Architecture & Syntax)

두 파일은 구조적으로 90% 이상 동일한 프론트엔드 아키텍처 패턴을 공유하고 있습니다. "HTML/CSS 기반의 2D HUD UI" 아래에 "Three.js 기반의 3D 캔버스"가 깔려 있는 형태입니다.

### [Phase 1] HTML Skeleton & ImportMap
```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <!-- 1. 메타 데이터 및 CSS -->
  <style> ... </style>
  
  <!-- 2. ImportMap: Three.js 모듈 위치 지정 -->
  <script type="importmap">
    {
      "imports": {
        "three": "https://unpkg.com/.../three.module.js",
        "three/addons/": "https://unpkg.com/.../examples/jsm/"
      }
    }
  </script>
</head>
<body>
  <!-- 3. 2D UI 레이어 (HUD, 컨트롤 패널 등) -->
  <div id="hud"> ... </div>
  
  <!-- 4. 3D 캔버스가 렌더링될 컨테이너 -->
  <div id="canvas-container"></div>
  
  <!-- 5. 메인 로직 -->
  <script type="module"> ... </script>
</body>
</html>
```

### [Phase 2] CSS (UI Layer / Glassmorphism)
2D UI 요소들은 3D 캔버스 위에 떠 있어야 하므로 공통적으로 아래와 같은 CSS 기법이 사용되었습니다.
* **절대 위치 지정:** `position: absolute; z-index: 10; pointer-events: none;` (이벤트 통과 처리)
* **Glassmorphism:** `background: rgba(...); backdrop-filter: blur(10px);` (반투명 블러 효과)
* **상호작용 복구:** 버튼이나 패널 등 클릭이 필요한 곳에는 `pointer-events: auto;` 적용.

### [Phase 3] JavaScript (Three.js 렌더링 파이프라인)
`<script type="module">` 내부의 코드는 다음과 같은 순서로 실행됩니다.

1. **Scene Setup (환경 설정)**
   * `THREE.Scene`, `PerspectiveCamera`, `WebGLRenderer` 초기화.
   * `OrbitControls`를 연결하여 마우스 드래그/줌 인아웃 구현.
2. **Lighting (조명 설정)**
   * `HemisphereLight` (전체적인 배경광) + `DirectionalLight` (태양광 및 그림자 생성).
   * 렌더러의 `shadowMap.enabled = true`를 통해 실사풍 그림자 적용.
3. **Materials & Helpers (재질 및 헬퍼 함수)**
   * 반복적인 메쉬 생성을 줄이기 위해 커스텀 헬퍼 함수 정의 (예: `bx()` 박스 생성기, `cyl()` 원기둥 생성기).
   * 색상, 거칠기(`roughness`), 금속성(`metalness`)을 조절한 `MeshStandardMaterial` 선언.
4. **Procedural Modeling (절차적 모델링)**
   * **(교실)**: 책상, 의자, 학생(구체 형태), 창문, 칠판 등을 수치 기반의 반복문(`for`)으로 좌표를 계산하여 배치.
   * **(버스)**: 섀시, 바퀴, 골격, 외판 등의 부품을 `THREE.Group`으로 묶어 8단계의 조립 모듈로 구성.
5. **State & Event Listeners (상태 관리 및 이벤트 바인딩)**
   * HTML의 버튼(`document.getElementById`)과 3D 씬의 변화를 연결.
   * 특정 버튼 클릭 시 목표 좌표나 카메라 시점을 변경하는 상태값 업데이트.
6. **Animation Loop (렌더링 루프)**
   * `requestAnimationFrame`을 사용한 무한 루프.
   * `Vector3.lerp()`를 사용하여 카메라 시점 이동이나 부품 조립(버스)을 부드럽게(Easing) 애니메이션 처리.
   * `renderer.render(scene, camera)` 호출로 화면 업데이트.

---

## 🔍 3. 개별 파일 핵심 분석

### A. 스마트 교실 (`smart_classroom_camera_stable.html`)
* **특징**: IoT 센서 데이터 시뮬레이션 및 환경 제어.
* **핵심 로직**:
  * **JSON Config**: `<script type="application/json" id="sceneSpec">` 안에 교실의 규격, 가구 위치, 센서 기본값 등을 데이터화하여 관리.
  * **Dynamic Canvas Texture**: 칠판이나 센서 라벨 등에 텍스트를 띄우기 위해 HTML5 `<canvas>`에 글씨를 그리고(draw), 이를 `CanvasTexture`로 변환하여 3D 재질로 입히는 고급 기법 사용.
  * **Occlusion Culling**: 카메라가 벽 뒤로 넘어가면 앞벽(`frontWallOccluders`)을 투명하게(`visible = false`) 만들어 시야를 확보하는 `updateWallCutaway()` 함수 존재.
  * **Mathematical Simulation**: 시간에 따른 CO2 농도 변화나 온도 변화를 계산하여 2D 차트로 그려주는 로직 포함.

### B. 전기버스 조립 공장 (`ThreeJS_korean_bus.html`)
* **특징**: 8단계 분할 조립 시퀀스와 X-Ray 투시 등의 시각 효과.
* **핵심 로직**:
  * **Stage Data**: 각 부품 그룹에 `start` 좌표와 `explode`(분해) 좌표를 부여.
  * **Lerp Animation**: 렌더링 루프 안에서 현재 단계(`currentStage`)에 따라 부품들이 `start`에서 `busOrigin(0,0,0)`으로 부드럽게 날아와 꽂히는(`lerp`) 조립 연출 구현.
  * **Material Overrides (X-Ray)**: 버튼 클릭 시 메인 외장 재질(`matPaint`, `matRoof`)의 `transparent` 속성을 토글하여 반투명(X-Ray) 효과 구현.
  * **Raycaster (옵션 적용 시)**: 2D 마우스 클릭 위치를 3D 광선으로 변환하여 부품 객체를 감지하는 기능 포함.