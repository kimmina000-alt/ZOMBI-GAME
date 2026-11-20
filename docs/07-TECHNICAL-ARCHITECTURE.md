# 07. Roblox 기술 구조·데이터·보안

## 과거 MCP 읽기 전용 조사 결과 (최신 Studio 상태 재확인 필요)
```text
ReplicatedStorage
  Shared
    GameConfig (ModuleScript)
    DefaultData (ModuleScript)
  Remotes
    TrainRequest (RemoteEvent)
ServerScriptService
  MainServer (Script)
  BaseBuildingServer (Script)
  Services
    PlayerDataService (ModuleScript)
    ZombieSystemServer (Script; 조사 당시 Enabled=false)
ServerStorage
  ZombieModels/BasicZombie (Model; Humanoid/HumanoidRootPart/Animate)
  SurvivorModels
  BuildingModels
  WeaponModels
StarterPlayer
  StarterPlayerScripts
    ClientController
    CombatClient
    BuildingClient
Workspace
  BaseBuildZone
  TrainingAreas/TrainingPad
  Worlds/SafeZone
  ZombieSpawns/Spawn1
  PlayerBases
```
과거 조사에서 CombatClient가 `Remotes.AttackRequest`를 기다리지만, 이를 생성하는 ZombieSystemServer가 비활성 상태여서 UI 초기화가 멈출 수 있었다. 기존 데이터 저장은 `ZombieSurvival_PlayerData_v1`. 과거 Studio API 접근 거부 문제는 사용자가 설정으로 해결했다고 보고했으나 현재 테스트 필요.

## 목표 모듈 구조 (제안)
- Shared: GameConfig, DefaultData, ZoneConfig, WeaponConfig, BuildingConfig, SurvivorConfig.
- Server: MainServer(플레이어 수명), PlayerDataService(프로필), CombatService(투척·판정), ZombieService(스폰·AI·포획), SurvivorService(스폰·구조·귀환), BuildingService(고정 건물 구매·복원), ResourceService(채집·생산).
- Client: ClientController/HUD, ThrowController, CollectionController, SurvivorController, BuildingPadController.
- 기존 코드를 무턱대고 대체하지 않고 의존성과 RemoteEvent 이름을 먼저 조사. 하나의 공격 권한 처리기와 하나의 스폰 루프 유지.

## 영속 데이터 예시 (스키마 초안)
```lua
{
 DataVersion=2,
 Level=1, XP=0, Power=0, Coins=100,
 Resources={Wood=0, Stone=0, Iron=0, Food=0, Parts=0},
 Buildings={}, -- 기존 자유배치 기록은 보존
 FixedBuildings={}, -- 새 건물 ID별 구매 상태
 ZombieIndex={}, CapturedZombies={},
 SurvivorIndex={}, RescuedSurvivors={}, AssignedSurvivors={},
 UnlockedWorlds={SafeZone=true},
 LastLogout=0,
}
```
**주의:** 실제 기존 DefaultData 필드 타입/데이터 버전과 저장 방식을 확인한 후만 스키마 변경. 누락 필드는 기본값으로 재귀 병합, 기존 데이터 덮어쓰기 금지.

## 보안
- 클라이언트가 제시한 착탄 Zone/피해량/구조 완료/구매 성공을 신뢰하지 않는다. 서버가 거리·위치·발판 접근·쿨다운·소유권·스폰 상태를 확인.
- 그물 기절/포획은 개체당 단일 상태 전이. 수용소 슬롯·동시 포획 검증.
- 자원 차감과 건물 구매는 원자적으로 처리; `Touched` 중복 발화와 여러 플레이어 구매 방지.
- 구조 NPC 예약·소유권·기지 도착/보상 중복 처리 금지.
- DataStore UpdateAsync 저장 순서 직렬화, 실패 재시도, 세션 충돌/서버 종료 안전성.
- 네트워크 소유권/Pathfinding 경계, 안전 통로 AI 접근 차단, 캐릭터 사망 후 이벤트 정리.

## Studio ↔ GitHub
GitHub 문서는 Studio에 자동 적용되지 않는다. Luau 코드까지 GitHub로 버전 관리하려면 Rojo + 파일 구조 동기화 또는 Studio export를 별도로 구성. 실제 Place 저장/퍼블리시 여부와 Git commit 여부를 구분하여 보고한다.
