# Rifle & UFO Zombie Cleanup — UE5 Blueprint 액션 실습

> 소총을 획득해 좀비를 처치하고, 승리 UI와 UFO 상호작용까지 연결한 Unreal Engine 5 개인 액션 실습입니다.

## 프로젝트 개요

| 항목 | 내용 |
|---|---|
| 개발 형태 | 개인 실습 |
| 엔진 | Unreal Engine 5.5 |
| 구현 방식 | Blueprint-only, Line Trace, UMG |
| 대표 맵 | `Content/_TEST2/StarterMap.umap` |
| 렌더링 | Windows, DirectX 12, Shader Model 6, Lumen |
| 현재 상태 | Blueprint 구조 정적 확인, UE 5.5 실행 확인 필요 |

대표 맵에는 소총 1개, UFO 1대와 좀비 3개가 배치되어 있습니다. 소총 획득부터 조준·사격, 데미지 전달, 적 수 확인과 승리 UI까지 하나의 전투 흐름으로 구성했습니다.

## 구현 범위

- 캐릭터 이동·시점·점프 입력 구성
- 소총 획득과 캐릭터 소켓 부착
- 조준 상태와 사격 Animation Montage 연결
- Line Trace 기반 명중 판정과 데미지 전달
- 피격 지점 파티클과 무기 음향 재생
- 좀비 데미지 수신, 제거와 남은 적 수 확인
- 승리·재시작 UMG 위젯 연결
- UFO Overlap, 캐릭터 부착과 Flying 전환
- 대표 맵과 Game Mode 구성

캐릭터·무기·UFO 모델, Animation, 음향과 Epic Starter Content는 직접 제작 범위에 포함하지 않습니다.

## 핵심 구현

### 소총 획득과 조준

`BP_Rilfle`의 Overlap으로 무기를 획득하고 캐릭터의 무기 소켓에 부착합니다. 조준 상태에서는 Animation Layer와 소총 조준·발사 Montage를 전환합니다.

### Line Trace 사격과 데미지

- 시선 방향으로 `LineTraceSingle` 실행
- 명중 Actor에 `ApplyDamage` 호출
- 명중 위치에 `P_Explosion` 파티클 생성
- Animation Notify와 무기 사운드 연결

### 적 제거와 승리 흐름

`BP_Zombie`는 `ReceiveAnyDamage`로 데미지를 받고 제거됩니다. 같은 클래스의 남은 적이 없으면 `BP_Win` 위젯을 표시하고, `BP_restart`와 `OpenLevel`로 재시작 흐름을 연결합니다.

### UFO 상호작용

- 캐릭터를 UFO에 부착
- Character Movement를 `MOVE_Flying`으로 전환
- `OnUFO`, `JumpHigh` 상태 변수 갱신

## Assignment 02와의 관계

이전 실습의 캐릭터 이동 기반을 재사용하되, 공 투척과 Interface 중심의 상호작용을 소총 조준·Line Trace 사격, UFO 이동과 승리·재시작 흐름으로 확장했습니다.

## 주요 Blueprint

| 자산 | 역할 |
|---|---|
| `Content/_TEST2/Granny.uasset` | 이동, 무기 획득, 조준·사격, 재시작 |
| `Content/_TEST2/GrannyController.uasset` | 입력 매핑과 시점 제어 |
| `Content/_TEST2/BP_Rilfle.uasset` | 맵에 배치되는 소총 Actor |
| `Content/_TEST2/BP_UFO.uasset` | Overlap과 Flying 전환 |
| `Content/_TEST2/BP_Zombie.uasset` | 데미지 수신, 제거, 승리 조건 확인 |
| `Content/_TEST2/BP_Win.uasset` | 승리 화면 UI |
| `Content/_TEST2/BP_restart.uasset` | 재시작 UI |
| `Content/_TEST2/GrannyMontage.uasset` | 소총 조준·발사 Animation |

## 조작법

| 입력 | 동작 |
|---|---|
| `WASD` | 이동 |
| 마우스 이동 | 시점 회전 |
| `SpaceBar` | 점프 관련 입력 |
| 마우스 버튼 | 조준·사격 이벤트 |
| `Left Shift`, `R` | 추가 상태·재시작 관련 이벤트 |

버튼별 최종 동작과 재시작 조건은 UE 5.5 실행 후 확인이 필요합니다.

## 실행 방법

1. Unreal Engine **5.5**를 설치합니다.
2. 저장소를 clone합니다.
3. 루트의 `.uproject` 파일을 엽니다.
4. `Content/_TEST2/StarterMap.umap`을 엽니다.
5. Blueprint를 Compile한 뒤 Play In Editor로 실행합니다.

기본 에디터·게임 시작 맵은 `StarterMap`으로 설정되어 있습니다.

## 개발 상태

무기·사격·데미지·승리와 UFO 흐름은 구성되어 있습니다. 저장된 자산에는 캐스팅 실패, 유효하지 않은 Target과 미연결 Animation 전이 진단 흔적이 있어, 정확한 UE 5.5 환경에서 Compile All과 전체 플레이 흐름을 다시 확인해야 합니다.

## 외부 자산

외부 FBX로 임포트된 캐릭터·Animation, Fab 무기·UFO와 Epic Starter Content가 포함되어 있습니다. 해당 콘텐츠는 직접 제작물로 주장하지 않으며, 공개 배포 전 원 배포처와 재배포 조건을 별도로 확인해야 합니다.

- [FPS Weapon Bundle — Fab](https://www.fab.com/listings/8aeb9c48-b404-4dcd-9e56-1d0ecedba7f5)
- [UFO Doodle — Fab](https://www.fab.com/listings/3ac20a1f-f426-4a6d-bff2-cf654ec54501)

## 프로젝트 구조

```text
.
├─ *.uproject
├─ Config/
├─ Content/
│  ├─ _TEST2/               # 캐릭터·무기·좀비·UFO·맵
│  ├─ Fab/                  # 외부 무기·UFO 콘텐츠
│  └─ StarterContent/       # Epic 제공 콘텐츠
└─ README.md
```

