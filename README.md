# Rifle & UFO Zombie Cleanup — UE5 Blueprint 액션 실습

소총을 획득해 조준·사격으로 좀비를 처치하고, UFO와도 상호작용하는 Unreal Engine 5 개인 액션 실습 프로젝트입니다.

> 이 문서는 프로젝트 설정, 대표 맵 참조, Blueprint 직렬화 정보로 교차 확인했습니다. 정확한 개발 버전인 Unreal Engine 5.5가 로컬에 없어 에디터 실행·패키징 검증은 아직 진행하지 않았습니다.

## 프로젝트 정보

| 항목 | 내용 |
| --- | --- |
| 개발 형태 | 개인 실습 |
| 엔진 | Unreal Engine 5.5 |
| 구현 방식 | Blueprint only |
| 대표 맵 | `Content/_TEST2/StarterMap.umap` |
| 렌더링 설정 | Windows / DirectX 12 / Shader Model 6 / Lumen / Ray Tracing |
| 현재 상태 | 정적 검증 완료, UE 5.5 실행·진단 검증 대기 |

## 프로젝트 개요

3인칭 캐릭터가 맵에서 소총을 획득하고 좀비를 사격하는 전투 흐름을 구현했습니다. 사격은 화면 방향 Line Trace로 대상을 판정하고 데미지를 전달하며, 적이 모두 제거되면 승리 UI와 재시작 흐름으로 이어집니다.

대표 맵에는 소총 Actor, UFO 1대, 좀비 3개가 배치되어 있습니다. UFO와 겹치면 캐릭터를 부착하고 이동 모드를 Flying으로 전환하는 별도의 상호작용도 포함합니다.

## 담당 범위

저장소의 커밋 기여자는 `douyun0623` 한 명입니다. 아래 범위는 커스텀 Blueprint와 맵에서 확인한 구현이며, 캐릭터·무기·UFO 모델, 애니메이션, 음향과 Epic Starter Content는 직접 제작 범위에 포함하지 않습니다.

- 캐릭터 이동·시점·점프 입력 구성
- 소총 획득과 캐릭터 소켓 부착
- 조준 상태와 사격 Animation Montage 연결
- Line Trace 기반 명중 판정과 데미지 전달
- 피격 지점 파티클과 무기 음향 재생
- 좀비 데미지 수신, 제거, 남은 수 확인
- 승리·재시작 UMG 위젯 연결
- UFO Overlap, 캐릭터 부착, Flying 전환
- 대표 맵과 Game Mode 구성

## 핵심 구현

### 1. 소총 획득과 조준

캐릭터 Blueprint인 `Granny`가 소총과의 상호작용과 전투 상태를 관리합니다.

- 맵의 `BP_Rilfle` 계열 Actor 획득
- 캐릭터의 무기 소켓에 소총 부착
- 조준 상태에 따라 애니메이션 레이어 전환
- 소총 조준·발사 Montage 재생

### 2. Line Trace 사격과 데미지

발사 입력은 시선 방향으로 `LineTraceSingle`을 실행하고, 명중 결과를 전투 피드백으로 연결합니다.

- Line Trace 시작점과 방향 계산
- 명중 Actor에 `ApplyDamage` 호출
- 명중 위치에 `P_Explosion` 파티클 생성
- Animation Notify와 무기 사운드 연결
- 사격 결과와 애니메이션을 한 흐름으로 처리

### 3. 적 제거와 승리 흐름

`BP_Zombie`는 `ReceiveAnyDamage`로 데미지를 받고 제거됩니다.

- 데미지 이벤트 수신
- Actor 제거와 같은 클래스의 남은 적 조회
- 남은 적이 없으면 승리 위젯 표시
- `BP_Win`, `BP_restart` UMG를 통한 결과·재시작 흐름
- `OpenLevel`을 이용한 레벨 재시작

### 4. UFO 상호작용

`BP_UFO`는 캐릭터와의 Overlap을 별도 이동 상태로 전환합니다.

- 캐릭터를 UFO에 부착
- Character Movement를 `MOVE_Flying`으로 전환
- `OnUFO`, `JumpHigh` 상태 변수 갱신

## Assignment 02와의 관계

이 프로젝트는 이전 실습의 `GrannyController`, `GrannyGameMode`, 이동용 Blend Space 등 캐릭터 기반을 이어서 사용합니다. 공 투척과 Blueprint Interface 중심이었던 이전 전투를 소총 조준·Line Trace 사격, UFO 이동, 승리·재시작 흐름으로 바꾸고 확장한 다음 단계의 실습입니다.

두 저장소는 공통 기반이 있지만 대표 맵과 핵심 상호작용이 다릅니다. 이 저장소에서는 무기 전투의 입력부터 명중, 데미지, 적 수 확인, 결과 UI까지 이어지는 플레이 루프를 중심으로 설명합니다.

## 주요 Blueprint

| 자산 | 역할 |
| --- | --- |
| `Content/_TEST2/Granny.uasset` | 이동, 무기 획득, 조준·사격, 재시작 |
| `Content/_TEST2/GrannyController.uasset` | 입력 매핑과 시점 제어 |
| `Content/_TEST2/BP_Rilfle.uasset` | 맵에 배치되는 소총 Actor |
| `Content/_TEST2/BP_UFO.uasset` | Overlap과 Flying 전환 |
| `Content/_TEST2/BP_Zombie.uasset` | 데미지 수신, 제거, 승리 조건 확인 |
| `Content/_TEST2/BP_Win.uasset` | 승리 화면 UI |
| `Content/_TEST2/BP_restart.uasset` | 재시작 UI |
| `Content/_TEST2/GrannyMontage.uasset` | 소총 조준·발사 애니메이션 |

## 조작법

| 입력 | 동작 | 검증 상태 |
| --- | --- | --- |
| `WASD` | 이동 | 입력 매핑에서 확인 |
| 마우스 이동 | 시점 회전 | `Mouse2D` 매핑에서 확인 |
| `SpaceBar` | 점프 관련 입력 | 입력 매핑·이벤트 확인, 최종 동작 검증 필요 |
| 마우스 버튼 | 조준·사격 이벤트 | Blueprint 이벤트 확인, 버튼별 동작 검증 필요 |
| `Left Shift`, `R` | 추가 상태·재시작 관련 이벤트 | 키 이벤트 확인, 최종 동작 검증 필요 |

키 이벤트와 그래프 연결은 확인했지만 버튼별 최종 동작, 조준 감도, 재시작 조건은 UE 5.5에서 직접 검증해야 합니다.

## 실행 방법

1. Unreal Engine **5.5**를 설치합니다.
2. 저장소를 클론합니다.
3. `Test2_2023180007.uproject`를 엽니다.
4. Content Browser에서 `Content/_TEST2/StarterMap.umap`을 직접 엽니다.
5. Blueprint 전체 컴파일 후 에디터의 Play 버튼으로 실행합니다.

`EditorStartupMap`은 `StarterMap`을 가리키지만, `GameDefaultMap`은 현재 엔진의 OpenWorld 템플릿을 가리킵니다. 패키징과 독립 실행 전에 대표 맵을 기본 게임 맵으로 지정해야 합니다.

## 검증 현황

| 검증 항목 | 결과 |
| --- | --- |
| UE 버전·프로젝트 설정 확인 | 완료 |
| 대표 맵과 배치 Actor 확인 | 완료 |
| 무기·사격·데미지·승리 구조 확인 | 완료 |
| 입력 매핑과 키 이벤트 확인 | 완료 |
| UE 5.5 Blueprint Compile All | 미실시 |
| Play In Editor | 미실시 |
| Windows 패키징 | 미실시 |

로컬에는 정확한 UE 5.5가 없습니다. 원본 자산의 자동 변환을 막기 위해 상위 버전으로 열거나 저장하지 않았습니다.

## 재검증이 필요한 Blueprint 진단

직렬화된 자산 문자열에서 다음 컴파일 진단 흔적을 확인했습니다. 저장 당시의 오래된 메시지일 수 있으므로 현재 오류로 단정하지 않으며, UE 5.5에서 **Compile All Blueprints** 후 남는 항목만 수정해야 합니다.

- `BP_ZombieAnim`이 Pawn을 상속하지 않는 `BP_Zombie`로 캐스팅해 항상 실패한다는 진단
- `BP_ZombieAnim`의 Idle → Death 전이가 연결되지 않아 실행되지 않는다는 진단
- `Granny`의 `Set MaxWalkSpeed` 변수 노드가 유효하지 않은 Target을 사용해 컴파일 과정에서 제거되었다는 진단
- `GrannyAnimation`의 Ground Moving → Winging, JumpUp → Winging 전이가 연결되지 않아 실행되지 않는다는 진단

## 대표 화면 준비 항목

현재 저장소에는 README에 사용할 플레이 스크린샷이 없습니다. UE 5.5 검증 시 다음 화면을 같은 해상도로 캡처해야 합니다.

1. 캐릭터, 소총, 좀비, UFO가 함께 보이는 대표 장면
2. 소총을 소켓에 부착하고 조준하는 장면
3. Line Trace 명중과 피격 파티클이 보이는 장면
4. 마지막 좀비 제거 직후 승리 UI
5. UFO 탑승 또는 Flying 상태의 장면
6. `Granny`의 Line Trace → `ApplyDamage` 구간과 `BP_Zombie` 수신부 그래프

## 외부 자산

게임 로직과 레벨 구성 외의 시각·음향 자산은 직접 제작물이 아닙니다.

- 외부 FBX로 임포트된 Sporty Granny·Whiteclown 캐릭터와 애니메이션 — 원 배포처 확인 필요
- [FPS Weapon Bundle — Fab](https://www.fab.com/listings/8aeb9c48-b404-4dcd-9e56-1d0ecedba7f5)
- [UFO Doodle — Fab](https://www.fab.com/listings/3ac20a1f-f426-4a6d-bff2-cf654ec54501)
- Epic Games Starter Content

저장소 메타데이터만으로 캐릭터·애니메이션의 Mixamo 출처를 확정할 수 없습니다. 원본 FBX 또는 배포 페이지로 출처와 이용 조건을 먼저 확인하고, Mixamo 자산으로 확인되는 경우 [Adobe Mixamo FAQ](https://helpx.adobe.com/kr/creative-cloud/faq/mixamo-faq.html)도 함께 확인해야 합니다. Fab 자산은 각 상품 페이지에 표시된 실제 라이선스와 원본 자산 재배포 조건을 점검하고, Fab Standard License가 적용되는 경우 [Fab EULA](https://www.fab.com/eula)도 함께 확인해야 합니다.

## 알려진 제한 사항

- UE 5.5에서 실행·컴파일·패키징하지 못했습니다.
- `GameDefaultMap`이 대표 맵으로 지정되어 있지 않습니다.
- Blueprint 진단 흔적을 UE 5.5에서 재확인해야 합니다.
- 패키징된 실행 파일, 플레이 영상, 결과 스크린샷이 없습니다.
- 약 779MB 중 약 616MB가 Starter Content이며 외부 UFO·무기 자산 비중도 큽니다.
- Git LFS가 없어 대형 `.uasset`이 일반 Git 객체로 저장됩니다.
- 저장소에 `LICENSE`가 없어 코드 재사용 조건이 명시되지 않았습니다.
- 외부 자산의 원본 라이선스와 공개 저장소 재배포 범위를 추가 확인해야 합니다.
- `DefaultEngine.ini`의 Android File Server `SecurityToken`은 미사용 시 제거하고, 사용 중이면 기존 값을 폐기한 뒤 재발급해야 합니다.

## 현재 완성도

소총 획득부터 Line Trace 명중, 데미지, 적 수 확인, 승리 UI로 이어지는 구조와 UFO 상호작용은 자산에서 확인했습니다. 포트폴리오로 공개하기 전 UE 5.5 Blueprint 전체 컴파일, 대표 맵 실행, 모든 입력·전투·승리·재시작 흐름 검증, 외부 자산 정리가 필요합니다.
