# 로봇 AI 학습 스택 정리: Isaac Sim, Isaac Lab, Cosmos, GR00T, OSMO

> 기준 시점: 2026년 8월. NVIDIA 공식 Isaac Sim, Cosmos, Isaac GR00T, OSMO, Robotics Newsroom 자료를 바탕으로 정리.

## 1. 핵심 변화 요약

로봇 AI 개발에서 시뮬레이션은 더 이상 단순한 강화학습용 가상 환경에 머물지 않는다. 최신 흐름은 **물리 기반 시뮬레이션, 합성 데이터 생성, world foundation model, 로봇 foundation model, 정책 평가, 엣지 배포**가 하나의 반복 루프로 묶이는 방향이다.

특히 Cosmos의 역할은 예전처럼 “시뮬레이션 결과를 더 사실적으로 렌더링하는 도구”로만 보기 어렵다. 현재 Cosmos는 **물리 AI(Physical AI)를 위한 World Foundation Model(WFM) 플랫폼**에 가깝고, 데이터 증강, 미래 상태 예측, 물리 세계 추론, 정책 학습 보조까지 역할이 확장되었다.

## 2. 주요 구성요소와 역할

### NVIDIA Omniverse

Omniverse는 OpenUSD 기반의 3D 협업, 시뮬레이션, 디지털 트윈 생태계다. 로봇 개발에서는 장면, 자산, 센서, 물리 정보가 OpenUSD를 통해 연결되는 기반 계층에 가깝다.

Omniverse 자체를 “모든 학습이 이루어지는 단일 프로그램”으로 보기보다는, Isaac Sim, Isaac Lab, NuRec, SimReady 자산 등 여러 도구가 공유하는 3D/시뮬레이션 인프라로 이해하는 편이 정확하다.

### NVIDIA Isaac Sim

Isaac Sim은 Omniverse 라이브러리 위에 구축된 오픈소스 로봇 시뮬레이션 및 합성 데이터 생성 프레임워크다. 로봇, 센서, CAD/URDF/MJCF 자산, OpenUSD 장면을 불러와 물리 기반 환경에서 테스트하고 데이터를 만들 수 있다.

주요 역할은 다음과 같다.

- 로봇과 환경의 물리 기반 시뮬레이션
- RTX 기반 카메라/LiDAR 등 센서 시뮬레이션
- Replicator, MobilityGen 등을 통한 합성 데이터 생성
- software-in-the-loop, hardware-in-the-loop 검증
- Isaac Lab과 연결한 정책 학습 및 평가 환경 제공

### NVIDIA Isaac Lab

Isaac Lab은 로봇 정책 학습을 위한 오픈소스 프레임워크다. Isaac Sim 기반의 물리 환경을 이용해 강화학습, 모방학습, 커리큘럼 학습, 도메인 무작위화 등을 수행한다.

과거식 설명으로는 “Isaac Sim에서 Teacher AI를 만든다”라고 단순화할 수 있지만, 현재는 Isaac Lab을 별도 학습 프레임워크로 구분하는 것이 더 정확하다.

### NVIDIA Cosmos

Cosmos는 물리 AI를 위한 World Foundation Model 플랫폼이다. 단순 렌더러가 아니라, 실제/합성 센서 데이터와 시뮬레이션 데이터를 바탕으로 물리 세계를 이해하고, 변형하고, 예측하고, 추론하는 모델군으로 보는 것이 맞다.

대표적인 역할은 다음과 같다.

- **Cosmos Transfer**: 시뮬레이션 또는 실제 영상 데이터를 photorealistic하게 증강한다. RGB, depth, segmentation 같은 입력을 바탕으로 날씨, 조명, 재질, 환경 다양성을 추가해 sim-to-real gap을 줄이는 데 쓰인다.
- **Cosmos Predict**: 물리 세계의 가능한 미래 상태나 영상을 예측하는 world simulation 계열 역할을 한다.
- **Cosmos Reason**: 물리 세계에 대한 reasoning을 수행하는 VLM 계열 모델이다. 모호한 지시를 해석하거나 상황을 단계적으로 이해하는 데 활용된다.
- **Cosmos 3**: 시각, 언어, 행동, 사운드 등을 통합해 reasoning, world simulation, synthetic video generation, policy model 학습 보조까지 확장된 open physical AI foundation model로 소개된다.

따라서 “Isaac Sim이 물리 뼈대를 만들고 Cosmos가 시각적 살을 붙인다”는 설명은 입문용 비유로는 쓸 수 있지만, 최신 기술 설명으로는 부족하다. 더 정확히는 **Isaac Sim/Isaac Lab에서 만든 물리·센서·궤적 데이터를 Cosmos가 증강, 예측, 추론, 평가 보조에 활용한다**고 보는 편이 낫다.

### NVIDIA Isaac GR00T

Isaac GR00T는 범용 휴머노이드 로봇을 위한 오픈 robot foundation model 및 개발 플랫폼이다. 언어와 이미지 같은 멀티모달 입력을 받아 조작 작업을 수행하는 VLA(Vision-Language-Action) 모델 계열로 볼 수 있다.

GR00T 계열은 실제 데이터, 합성 데이터, 인터넷 규모 비디오 데이터 등을 활용하며, 특정 로봇 embodiment, 작업, 환경에 맞게 post-training하는 흐름을 따른다. Cosmos Reason과 결합해 더 복잡한 지시 이해와 물리적 reasoning을 보강하는 방향으로 발전하고 있다.

### NVIDIA OSMO

OSMO는 물리 AI 워크플로를 정의, 실행, 모니터링하는 클라우드 네이티브 오케스트레이션 프레임워크다. Isaac Sim 기반 합성 데이터 생성, Cosmos 기반 데이터 증강, 모델 학습, 검증 작업을 여러 컴퓨팅 환경에 걸쳐 관리하는 역할을 한다.

즉 “Isaac Sim이 Cosmos를 즉시 호출한다”는 식의 단순한 직접 연결보다, 실제 최신 워크플로에서는 **OSMO가 다단계 파이프라인을 묶고 실행하는 구조**가 더 자연스럽다.

### Newton Physics Engine

Newton은 Google DeepMind, Disney Research, NVIDIA가 함께 개발하고 Linux Foundation이 관리하는 오픈소스 GPU 가속 물리 엔진이다. NVIDIA Warp와 OpenUSD 기반으로 만들어졌고, Isaac Lab과 호환되는 로봇 학습용 물리 엔진으로 소개된다.

복잡한 접촉, 보행, 조작, 유연한 물체와의 상호작용처럼 기존 물리 엔진이 어려워하던 로봇 문제를 더 잘 다루기 위한 방향이다.

### Isaac Lab-Arena

Isaac Lab-Arena는 로봇 정책 평가와 벤치마킹을 위한 오픈소스 프레임워크다. 학습된 정책이 실제 배포 전에 다양한 task와 benchmark에서 얼마나 견고한지 검증하는 데 초점을 둔다.

## 3. 최신 로봇 AI 학습 파이프라인

최신 흐름은 하나의 고정된 정답 파이프라인이라기보다, 목적에 따라 아래 단계를 조합하는 방식에 가깝다.

### 1단계: 환경과 로봇 구성

Isaac Sim에서 OpenUSD 기반 장면을 구성한다. CAD, URDF, MJCF, 실제 환경 캡처, Omniverse NuRec 기반 재구성 데이터, SimReady 자산 등을 활용해 물리·시맨틱 정보가 포함된 환경을 만든다.

### 2단계: 데이터 생성

Isaac Sim, Replicator, MobilityGen 등을 이용해 로봇 궤적, RGB/depth/segmentation 센서 데이터, occupancy map, 충돌 없는 경로, teleoperation demonstration 등을 생성한다.

이 단계에서는 완전히 자동화된 랜덤 주행뿐 아니라, 사람의 원격 조작이나 실제 데이터 수집도 함께 쓰인다.

### 3단계: 정책 학습

Isaac Lab에서 강화학습, 모방학습, 커리큘럼 학습, 도메인 무작위화 등을 통해 로봇 정책을 학습한다.

Teacher-Student distillation은 여기서 사용할 수 있는 한 가지 방법이다. 예를 들어 상태 기반 teacher policy가 만든 행동을 vision 기반 student policy가 따라 배우게 할 수 있다. 다만 이것이 모든 최신 로봇 학습의 표준 파이프라인은 아니며, 작업과 모델 구조에 따라 다양한 학습 방법이 함께 쓰인다.

### 4단계: Cosmos 기반 데이터 증강과 world modeling

Isaac Sim이나 실제 로봇에서 얻은 영상·센서·궤적 데이터를 Cosmos Transfer로 photorealistic하게 증강한다. 조명, 날씨, 재질, 반사, 배경 다양성을 늘려 sim-to-real gap을 줄인다.

Cosmos Predict 계열은 가능한 미래 상태를 예측하거나 closed-loop world simulation에 활용될 수 있고, Cosmos Reason은 장면 이해, 지시 해석, 데이터 주석, 물리적 reasoning에 활용될 수 있다.

### 5단계: 평가와 벤치마킹

학습된 정책은 Isaac Sim, Isaac Lab, Isaac Lab-Arena에서 software-in-the-loop 또는 hardware-in-the-loop 방식으로 검증한다. 도메인 무작위화, 다양한 환경, benchmark task를 적용해 실제 배포 전에 견고성을 확인한다.

### 6단계: 배포와 반복

검증된 모델은 Jetson Thor 같은 엣지 컴퓨팅 플랫폼이나 로봇 제어 시스템에 배포된다. 실제 운용 중 수집된 실패 사례와 long-tail 데이터를 다시 학습 데이터로 넣어 파이프라인을 반복한다.

## 4. 예전 이해와 현재 이해의 차이

예전에는 다음처럼 이해해도 큰 무리가 없었다.

```text
Isaac Sim = 물리 시뮬레이션
Omniverse = 3D/렌더링 기반 플랫폼
Cosmos = 고품질 합성 영상/렌더링 보강
```

현재는 이렇게 보는 것이 더 정확하다.

```text
Omniverse/OpenUSD = 장면, 자산, 디지털 트윈 기반 계층
Isaac Sim = 로봇 시뮬레이션과 합성 데이터 생성 환경
Isaac Lab = 로봇 정책 학습 프레임워크
Cosmos = world foundation model 기반 데이터 증강, 예측, reasoning, policy 보조
GR00T = 휴머노이드 중심 robot foundation model/VLA
OSMO = 데이터 생성, 증강, 학습, 평가 워크플로 오케스트레이션
Newton = 차세대 로봇 물리 시뮬레이션 엔진
Isaac Lab-Arena = 정책 평가와 벤치마킹
```

즉 Cosmos는 “렌더러에서 시작해 더 커졌다”기보다는, **초기에는 photorealistic synthetic data와 world generation 쪽 인상이 강했지만, 현재는 물리 AI를 위한 기반 모델 플랫폼으로 역할이 확장되었다**고 이해하면 된다.

## 5. 실무 관점의 요약

로봇 AI를 개발한다면 최신 NVIDIA 스택은 다음처럼 묶어서 보는 것이 좋다.

```text
환경 구축: Omniverse, OpenUSD, NuRec, SimReady assets
시뮬레이션/데이터 생성: Isaac Sim, Replicator, MobilityGen
정책 학습: Isaac Lab, GR00T, imitation learning, reinforcement learning
데이터 증강/추론: Cosmos Transfer, Cosmos Predict, Cosmos Reason, Cosmos 3
워크플로 운영: OSMO
평가: Isaac Sim, Isaac Lab-Arena, SIL/HIL testing
배포: Jetson Thor, CUDA-X runtime, robot middleware
```

핵심은 하나의 도구가 모든 것을 처리하는 구조가 아니라, **시뮬레이션-합성데이터-학습-증강-평가-배포가 반복되는 폐루프 개발 체계**라는 점이다.

## 6. 참고 자료

- NVIDIA Isaac Sim: https://developer.nvidia.com/isaac/sim
- NVIDIA Cosmos: https://www.nvidia.com/en-us/ai/cosmos/
- NVIDIA Isaac GR00T: https://developer.nvidia.com/isaac/gr00t
- NVIDIA OSMO and Isaac Sim synthetic data workflow: https://developer.nvidia.com/blog/build-synthetic-data-pipelines-to-train-smarter-robots-with-nvidia-isaac-sim/
- NVIDIA Robotics Newsroom: https://nvidianews.nvidia.com/news/nvidia-releases-new-physical-ai-models-as-global-partners-unveil-next-generation-robots
