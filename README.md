<div align="center">

**한국어** · [English](README.en.md)

# Graphics Optimizer

**언리얼 엔진 5.5 그래픽 스케일러빌리티 자동 벤치마크 플러그인**

목표 FPS를 맞추기 위한 Scalability 설정 조합을 손으로 하나씩 바꿔가며 재는 대신, PIE 안에서 자동으로 측정·탐색·검증하는 런타임/에디터 플러그인입니다.

</div>

---

## Overview

`Graphics Optimizer`는 언리얼 엔진 프로젝트의 **Shadow / Global Illumination / Reflection / Post Process / Texture / Effects / Resolution Scale** 조합을 PIE에서 직접 측정해, 목표 FPS를 만족하는 가장 화질 손실이 적은 설정을 찾아주는 도구입니다.

- **측정 기반 탐색** — 감으로 설정을 낮추는 대신, 각 조합의 FPS · 1% low · Frame/Game/Draw/GPU 시간을 실제로 측정하고 비교합니다.
- **품질 회복 패스** — 목표를 만족한 뒤에는 변경한 설정을 하나씩 원복해보며, 목표를 유지하는 선에서 화질을 최대한 되살립니다.
- **반복 검증** — 최종 후보는 3회 재측정으로 반복 가능성을 확인한 뒤에만 "목표 달성"으로 확정합니다.
- **원본 복원 보장** — 모든 실험은 종료·취소·PIE 종료 시 그래픽 설정과 일시정지 상태를 원래대로 되돌립니다.

---

## Key Features

| 기능 | 설명 |
|---|---|
| **설정 조합 탐색 (Phase 2)** | 개별 설정을 한 단계씩 낮춰 이득이 큰 항목부터 조합, 목표 FPS 충족 시 품질 회복 패스로 화질 복원 |
| **반복 검증** | 선택된 후보를 3회 재측정해 median/range 계산, 3회 모두 목표 충족 시에만 "반복 가능"으로 판정 |
| **GPU Pass A/B & 트레이스 (Phase 3)** | GI/Shadow 개별 저하 A/B, `.utrace` 캡처와 UE Insights 연동, 주요 렌더 패스(디퍼드 라이팅·Lumen·섀도우 뎁스) 타이밍 분석 |
| **섀도우 캐스터 스캔** | 씬에서 동적 그림자를 드리우는 스태틱 메시 그룹을 인벤토리화하고, 그룹 단위 `CastShadow` on/off A/B 측정 |
| **VSM/인스턴싱 후보 스캔** | 무빙 섀도우 라이트, WPO 캐스터, 반복되는 메시/머티리얼 그룹을 탐지 (측정이 아닌 후보 제시) |
| **Lumen 스크린 프로브 A/B** | `r.Lumen.ScreenProbeGather.DownsampleFactor` 조정 전/후 FPS·GPU·이미지 비교 |
| **비주얼 A/B 캡처** | 베이스라인/선택 설정을 동일 카메라에서 캡처해 Side-by-Side PNG 생성 |
| **모션 A/B (선택)** | A/D 자동 입력으로 4초 구간을 각각 녹화, `ffmpeg`이 PATH에 있으면 좌우 비교 MP4 생성 |
| **적용 / 프리셋 저장** | 선택한 조합을 PIE에 임시 적용하거나, 프로젝트 로컬 프리셋(.ini)으로 저장·재적용. 게임 인스턴스 서브시스템 API로 패키지 빌드에서도 저장/로드 가능 |
| **리포트 히스토리** | 완료/중단된 모든 실험이 `Saved/GraphicsOptimizer/ReportHistory/`에 자동 축적, 프로젝트 누적 리포트 복사 지원 |

전체 상세 동작(각 패널 버튼, 캡·프레임 상한 처리, 렌더 스레드 타이밍 구분 등)은 [`Plugins/GraphicsOptimizer/README.md`](Plugins/GraphicsOptimizer/README.md)에 있습니다.

---

## Getting Started

1. **Unreal Engine 5.5**를 설치합니다.
2. 이 저장소를 클론하거나, `Plugins/GraphicsOptimizer` 폴더를 대상 프로젝트의 `Plugins/`에 복사합니다.
3. `GraphicsOptimizer.uproject`를 우클릭 → **Generate Visual Studio project files**.
4. `GraphicsOptimizer.sln`을 열어 **Development Editor / Win64**로 빌드합니다.
5. 에디터 실행 후 플러그인이 활성화되어 있는지 확인합니다 (`Edit > Plugins > Graphics Optimizer`).
6. **Play In Editor**로 카메라를 고정한 안정적인 씬을 준비합니다.
7. 메뉴에서 **Tools > Graphics Optimizer**를 엽니다.

> 모션 A/B 비디오 인코딩을 사용하려면 `ffmpeg.exe`가 시스템 PATH에 있어야 합니다 (없어도 PNG 프레임과 FPS 리포트는 생성됩니다).

---

## Usage

1. 패널은 목적별로 구성되어 있습니다 — **Quality settings**(설정 탐색·결과 선택·비교·적용/저장), **Shadow casters and GPU traces**, **Lumen detail and scene structure**, **Complete analysis and final decision**(전체 스터디를 한 번에 실행). 필요한 영역만 펼쳐서 사용합니다.
2. 목표 FPS와 측정 시간을 입력한 뒤 **Run All Analyses + Copy Report**(전체 실행) 또는 **Run Optimization**(설정 탐색만)을 실행합니다.
3. 실행 중에는 엔진의 `stat unit` 오버레이가 자동으로 켜지고, 뷰포트별 Frame/Game/Draw/GPU 원시 값을 샘플링합니다. 로그 카테고리는 `LogGraphicsOptimizer`입니다.
4. 완료 후 **Copy Settings/Shadow/GPU/Scene/Lumen Report**로 개별 리포트를, **Copy Accumulated Project Reports**로 프로젝트 전체 누적 리포트를 클립보드에 복사할 수 있습니다.
5. 결과 행을 선택해 **Capture A/B: Baseline vs Selected**, **Apply Selected to PIE**, **Save Selected Preset** 등을 실행합니다. 모든 산출물(PNG, 리포트, 프리셋)은 프로젝트의 `Saved/GraphicsOptimizer/` 아래에 저장됩니다.

> 이 도구는 **PIE 런타임 실험** 도구입니다. 메시나 레벨을 저장하지 않으며, 설정 적용/프리셋 저장은 항상 별도의 명시적 동작입니다.

---

## Project Structure

```
Graphic-Optimizer/
├─ GraphicsOptimizer.uproject
├─ Source/GraphicsOptimizer/        기본 프로젝트 게임 모듈
├─ Plugins/GraphicsOptimizer/
│  ├─ Source/
│  │  ├─ GraphicsOptimizerRuntime/  측정·탐색·적용 로직 (Runtime 모듈)
│  │  └─ GraphicsOptimizerEditor/   에디터 패널 UI (Editor 모듈)
│  ├─ EnginePatches/UE5.5/          엔진 헤더 관련 보조/패치 참고 자료
│  ├─ Docs/BenchmarkResults/        실행 리포트 아카이브 (git 추적 대상 아님)
│  └─ README.md                    플러그인 상세 사용 설명서
├─ Config/                          프로젝트 설정
└─ Content/                         테스트용 콘텐츠 (에셋)
```

---

## Requirements

| | |
|---|---|
| **엔진** | Unreal Engine 5.5 |
| **언어** | C++17 |
| **플랫폼** | Windows |
| **선택 의존성** | `ffmpeg.exe` (PATH) — 모션 A/B 비디오 인코딩용 |
| **상태** | Beta (`IsBetaVersion: true`) |

---

## Notes

- 모든 벤치마크는 원본 그래픽 설정과 일시정지 상태를 실행 종료/취소/PIE 종료 시 복원합니다.
- GPU 타이밍은 일부 RHI/에디터 구성에서 사용 불가할 수 있으며, 이 경우 GPU 비교는 생략됩니다.
- 엔진의 프레임 상한(effective max tick rate)이 목표 FPS 근처에 있으면 결과는 "capped / target inconclusive"로 표시됩니다.
- `Docs/BenchmarkResults/`, `Saved/`, `Binaries/`, `Intermediate/` 등 생성물은 `.gitignore`로 제외되어 있습니다.
