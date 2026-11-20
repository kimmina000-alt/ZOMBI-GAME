# Codex 작업 규칙 — Zombie Survival: Last Base

## 범위
- Roblox Studio MCP에서 **Place ID 120848184984636**만 수정. 열려 있는 다른 Place(122010099115917, 89906407011275)는 손대지 않는다.
- 이 저장소 docs/를 기획 기준으로 사용하며, 상충하는 기획은 docs/10-DECISIONS-AND-OPEN-QUESTIONS.md에 기록.
- 기술 구현 상태는 Studio의 실제 스크립트를 직접 읽고 재검증. 과거 ChatGPT 프롬프트가 곧 구현 완료라는 뜻이 아니다.
- **편집 전에** 영향 범위 조사·기존 오브젝트/스크립트 원본 백업·롤백 경로 마련·수정 대상 확인.

## 게임 기본
- 높은 상부 안전 기지, 낮고 긴 직선형 하부 Zone01~05, 플레이어용 사다리, 왼쪽 좀비 차단 안전 통로.
- 투척 발판 진입 시 세로 게이지, 돌 폭발 처치/그물 기절·포획, 비행 카메라. 게이지/거리/피해 수치는 GameConfig 또는 별도 공유 설정에서 관리.
- Zone이 멀수록 지형 난이도·좀비 레벨/희귀도·생존자 레벨/희귀도 증가.
- 생존자는 구조 후 안전 통로로 줄지어 자동 귀환(일반형); 등에 업기는 특수 구조/별도 기능이며 최근 사용자가 직접 업는 연출을 요청한 기록도 있으므로 두 흐름의 UX를 독립 구현한다.
- 건물은 지정 부지에 미리 제작하여 숨긴 다음, 조건 충족 시 구매 발판을 밟아 자원 소비 + 건설 애니메이션으로 공개. 자유 배치 UI는 단계적으로 대체.

## 구현 안전
- 기존 MainServer, PlayerDataService, BaseBuildingServer, ClientController, CombatClient, BuildingClient 동작을 확인하여 회귀 방지.
- ZombieSystemServer는 과거 확인 당시 ServerScriptService.Services에 있으며 Enabled=false였음. 실제 현재 상태부터 검사. 좀비 스폰 루프·공격 처리기 중복 금지.
- 기존 DataStore 이름 `ZombieSurvival_PlayerData_v1` 유지. Wood/Iron/Food/Parts/Coins 및 기존 Buildings, ZombieIndex, Survivors 데이터를 보존. Stone 등 새 필드는 하위호환 마이그레이션.
- 서버가 공격, 물리 충돌 판정, 포획, 터치 구매, 자원 차감, 소유권, 보상, 구조 완료, 리스폰을 검증. RemoteEvent 호출을 신뢰하지 않는다.
- 여러 플레이어의 접근·동시 구매·중복 보상·데이터 저장 경쟁·퇴장 후 처리에 안전해야 한다.
- Studio 메인 Place에 대규모 변경 전 가능하면 별도 백업/테스트 브랜치와 로컬 파일을 준비. 공개 배포·저장 데이터 삭제 금지.
- PC 및 모바일에서 모두 플레이 가능하도록 GUI 크기·입력 통일. Roblox 애니메이션 에셋 ID가 없으면 임시 포즈와 교체 지점을 명시하며 완성했다고 주장하지 않는다.

## 매 작업 보고
1. 현재 상태 및 발견 문제
2. 수정 파일·Explorer 경로 및 오브젝트 좌표
3. 백업 위치·롤백 방법
4. 자동 테스트·Studio 수동 테스트 결과(통과/실패/미검증)
5. 데이터 호환성·보안 영향
6. 다음 작업

AGENTS.md를 바꿔도 Roblox Studio의 실제 스크립트에 자동 반영되는 것은 아니다.
