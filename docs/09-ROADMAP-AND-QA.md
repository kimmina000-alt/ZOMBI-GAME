# 09. 로드맵·검증·리스크

## 실제로 확인된 과거 상태(2026-11-20 조사 시점)
- Place ID 120848184984636 편집 인스턴스.
- ReplicatedStorage.Shared.GameConfig/DefaultData, Remotes.TrainRequest.
- ServerScriptService.MainServer, BaseBuildingServer, Services.PlayerDataService, Services.ZombieSystemServer(당시 **Enabled=false**).
- ServerStorage.ZombieModels.BasicZombie(Humanoid, HumanoidRootPart, Animate 등); SurvivorModels/BuildingModels/WeaponModels 비어 있음.
- StarterPlayerScripts.ClientController/CombatClient/BuildingClient.
- Workspace.BaseBuildZone, TrainingAreas.TrainingPad, Worlds.SafeZone, ZombieSpawns.Spawn1, PlayerBases 등.
- CombatClient가 AttackRequest를 기다렸으며 활성 서버에서 생성하지 않아 UI 대기 가능성 **확인된 문제**.
- 그 뒤 Codex에 전달한 지시의 실행 완료 여부는 보고받지 못했으므로 새 기능은 아직 **미검증**.

## 마일스톤
| 단계 | 목표 | 수락 조건 |
|---|---|---|
| M0 | 저장소 문서·Studio 원본 백업·스크립트 현행화 | 변경 전·후 비교 가능 |
| M1 | 상부/하부 안전 지형, 산악 측벽, 빈 절벽 보강, 사다리 출구 | 낙하/막힘 없음 |
| M2 | 투척 발판 + 타이밍 UI + 돌 비행/폭발/Zone 판정 | PC/모바일 투척·보상, 중복 보상 없음 |
| M3 | 좀비 AI, 그물 기절·포획·수용소·도감·리스폰 | 안전 구역 침입 없음, 데이터 복원 |
| M4 | 웅크린 생존자, 랜덤 출현, 안전 통로 자동 행렬 귀환 | 여러 NPC가 줄지어 기지 도착 |
| M5 | 고정형 건물 발판과 해금·건설 애니메이션, Stone, 저장 | 재접속 복원·중복 구매 없음 |
| M6 | 작업장 자동생산·무기 연구·희귀도 확장 | 성장경제 수치 정상 |
| M7 | 전체 5 Zone 디테일·균형·모바일 성능 | 장거리 투척·구조 루프 완주 |

## 검증 체크리스트
- 맵: 투척 플랫폼 끝 낙하 방지, 기지 아래 벽 채움, 양쪽 산 상단 높이, 사다리 상부 출구 및 귀환 안전 통로.
- 전투: 발판 진입 즉시 게이지, 투척 거리, 돌이 안전 경계 통과, Zone별 폭발, 생존자 피해 없음, 리스폰 규칙.
- AI: 좀비 상부 침입/안전 통로 진입 불가, 다중 플레이어 추적 권한, ZombieSystemServer 단독 활성.
- 생존자: 랜덤 스폰, 웅크림, 구조 상호작용, 줄서기, NPC 막힘 복구, 특수 등에 업기.
- 건물: 자원 부족 비활성, 조건 충족 표시, 발판 중복 터치, 건설 애니메이션, 건물 기능 활성, 재접속 복원.
- 데이터: Studio API 서비스 허용, DataStore 저장·로드·동시 접근·타입 병합·Stone 추가·세이브 실패 안전성.
- 모바일: HUD 겹침, 탭 입력, FPS/네트워크 지연, 소형 해상도.

## 중요 리스크
- Roblox Studio MCP 편집과 GitHub 저장소는 **자동 동기화되지 않음**. 별도의 Rojo 파일 동기화·export·Git commit 워크플로 필요.
- 서버 DataStore 저장 충돌, 멀티플레이어 기지 소유/포획·건물 발판 범위, 대량 NPC 경로 탐색, 원본 영상 에셋/애니메이션 저작권.
- 테스트 여부는 PASS/FAIL/UNTESTED로 기록하며 미검증 항목을 성공이라고 적지 않는다.
