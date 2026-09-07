## /machine/configuredProtocol
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

이 연결이 쓰는 **프로토콜 식별자**입니다. `"nc_focas2_fanuc"`·`"nc_opcua_siemens"`·`"nc_ezsocket_mitsubishi"` 중 하나. 필터 없음. 반환 `string`, 읽기 전용. 연결 시 정해지는 값이라 연결된 뒤에는 NC 통신 없이 즉시 응답합니다 (미연결 상태에서는 다른 주소처럼 연결 확인이 먼저라 상태 `-10` 입니다).

`configuredMachineName` 과 마찬가지로 **설정에서 온 값**입니다. 기계에 물어본 결과가 아니라 `deemesh_create` 의 `protocol` 필드(또는 `config.json` 의 머신 설정)를 그대로 돌려줍니다. 그래서 주소에 `configured` 가 붙습니다.

**용도는 좁습니다.** 대부분의 주소는 기종을 감추도록 설계되어 있어 분기가 필요 없습니다. 이 값이 필요한 곳은 **값 공간이 기종 소유인 소수의 자리**입니다. PLC 주소 문법(`D100` 대 `DB10.DBB56`), 진단 번호 체계, 공구 타입 코드처럼 카탈로그가 "기종에 따라 다르다" 고 명시한 곳들입니다.

**지원 여부 판단에는 쓰지 마세요.** "이 기종은 이 주소를 못 쓰니 건너뛰자" 는 판단은 상태 `-20` 으로 해야 합니다. 이 값으로 분기해 두면 나중에 그 기종 지원이 추가돼도 코드가 계속 건너뜁니다.

## /machine/configuredMachineName
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

`config.json`(허브) 또는 `deemesh_create` 설정의 `machine_name` 을 그대로 돌려줍니다. 장비가 보고하는 이름이 아니라 **설정에서 온 값**입니다. 이름에 `configured` 를 넣은 것도 그 때문입니다. 연결이 의도한 장비로 갔는지 확인하거나 응답을 식별할 때 씁니다.

## /machine/cncModel
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

CNC 모델 문자열입니다.

- **Fanuc**: 시리즈 번호 문자열입니다. `"15"`, `"16"`, `"18"`, `"21"`, `"30"`, `"31"`, `"32"`, `"35"`, `"0"`(0i), `"PD"`/`"PH"`(Power Mate i), `"PM"`(Power Motion i). `desc` 에 시리즈명이 함께 옵니다 (예: `"31"` → `Series 31i`). **제어기가 세대를 알려 주면 `desc` 끝에 그 글자를 붙입니다** (예: `Series 31i-B`, `Series 0i-F`). 세대를 알려 주지 않는 제어기에서는 붙지 않습니다 (0i-A/B/C · 30i-A · 그 이전 시리즈). `value` 는 어느 쪽이든 같습니다. `desc` 는 표시용 문자열이라 바뀔 수 있으므로 세대 분기 조건으로 쓰지 마세요
- **Siemens**: 모델명 그대로 (예: `"840D sl"`)
- **Mitsubishi**: NC 시스템 S/W 번호·이름 문자열입니다 (벤더 `GetVersion`). 이 항목을 제공하지 않는 장비에서는 상태 `-20` 으로 답합니다. 실물 하드웨어 정보가 없는 시뮬레이터가 그렇습니다. `desc` 는 없습니다

## /machine/machineType
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

장비 종류입니다. 반환 `string`. 값 자체가 뜻을 담은 enum 이라 별도 코드표가 필요 없습니다. 나올 수 있는 값 전체:

- `"machiningCenter"`: 머시닝센터 (Fanuc M/MM, Siemens M, Mitsubishi `…M` 계열)
- `"lathe"`: 선반 (Fanuc T/TT/MT, Siemens T, Mitsubishi `…L` 계열)
- `"punchPress"`: 펀치 프레스 (Fanuc 전용)
- `"laser"`: 레이저 (Fanuc 전용)
- `"wireCut"`: 와이어 컷 (Fanuc 전용)
- `"unknown"`: 판별 불가 (Mitsubishi 는 밀/선반이 갈리지 않는 계열: C70·C80·C6/C64·PC 카드형)

Mitsubishi 는 이 값을 **설정한 `system_type` 에서** 가져옵니다. 설정을 그대로 믿는 것이 아니라, 벤더가 `…M` 을 머시닝센터 시스템 · `…L` 을 선반 시스템으로 정의하고 연결 시 그 구분을 **실제로 검증**하기 때문입니다. 밀에 `…L` 을 지정하면 연결 자체가 거부됩니다. 즉 연결이 성립했다는 것이 곧 장비가 이 값을 확인해 준 것입니다.

## /machine/currentDateTime
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

장비의 **현재 날짜/시각**입니다. 반환 `string`, ISO 8601 초 단위 (`"2026-07-11T14:30:00"`).

- **장비 로컬 시계**: 타임존 정보가 없으므로 TZ 접미사(`Z`/`+09:00`)를 붙이지 않습니다. ISO 8601 의 로컬 시각 형식이며, 오프셋을 필수로 요구하는 RFC 3339 파서는 이 값을 거부할 수 있습니다
- **자바스크립트 `new Date()` 에 그대로 넣지 마세요**: 오프셋 없는 날짜+시각을 **보는 사람의 시간대**로 해석합니다. 이 값은 보는 사람이 아니라 **장비의 벽시계**입니다
- 서버 PC 시계가 아니라 **CNC 의 시계**입니다. 장비 시계가 틀어져 있으면 그대로 반영
- Fanuc: `cnc_gettimer` / Siemens: `sysTimeBCD` / Mitsubishi: `GetClockData`
- **장비 헬스체크 권장 주소**: 모든 프로토콜에서 실제 NC 왕복을 일으키는 부담 적은 읽기라, 주기 폴링 후 `status` 판정(`0`=정상, 상태 `-10`/상태 `-14`=링크 이상)으로 장비별 통신 상태 감시에 쓰세요. (`machineType` 등 캐시 서빙 주소는 링크가 죽어도 성공할 수 있어 부적합)

시각 계열 주소는 항상 ISO 8601 문자열입니다 (`…At` = 이벤트 시점, `…DateTime` = 시계 읽기).

## /machine/powerOnDuration
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

장비의 **누적 전원투입 시간**입니다. 전원을 껐다 켜도 계속 누적됩니다. 반환 `int` (초) + `unit:"s"`.

- **Fanuc**: 파라미터 6750. **분 해상도**라 값이 항상 60의 배수입니다. 차분 계산 (가동률 등) 시 ±60초 오차 내재
- **Siemens**: `setupTime` 이라 분 미만 해상도까지 반영합니다 (초로 환산). 일반 전원 재투입에는 리셋되지 않지만, **기본값으로 제어기를 부팅하면 `0`** 이 됩니다 (드문 정비 작업)
- **Mitsubishi**: `GetAliveTime`. **초 해상도**입니다. 조작반 통합 시간 화면의 `Power ON`(전원 ON~OFF 누적)과 같은 값이며, **제어기는 `59999:59:59` 에서 누적을 멈추고 그 값을 유지**합니다 (M800 조작 매뉴얼 §9.3.1). EZSocket 문서에는 이 값이 `HHHHMMSS` 8자리(최대 `9999:59:59`)로 적혀 있어, 9999시간(약 416일)을 넘은 뒤 API 가 무엇을 돌려주는지는 확인하지 못했습니다. 상한에 닿은 뒤로는 차분이 계속 `0` 이 됩니다

경과시간 계열 주소는 항상 **초 정규화 int** 입니다 (`…Duration` 접미사 규칙). **초 미만은 버립니다**: `59.9`초는 `59` 입니다. 조작반의 경과시간 표시와 같은 방식이고, 아직 지나지 않은 초를 세지 않습니다. 모든 기종·모든 `…Duration` 주소가 같습니다.

## /machine/channelCount
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

CNC 의 채널(계통) 수입니다. 반환 `int`, 읽기 전용. 연결 시 캐싱된 값이라 추가 통신 없이 즉시 반환됩니다. `channel` 필터의 유효 범위가 `1`~이 값입니다.

출처는 Fanuc `cnc_getpath` 의 최대 경로 수, Siemens `/Nck/Configuration/numChannels` 이고, Mitsubishi 는 연결 때 계통 `1`~`8` 을 차례로 열어 보며 세어 둔 값입니다.

## /machine/channel/toolAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

채널이 사용하는 공구 영역(tool area) 번호입니다. 공구 트리 주소들의 `toolArea` 필터에 넣는 값입니다.

**이 값을 읽어서 그대로 `toolArea` 에 넣으세요.** 번호를 매기는 방식이 기종마다 다르므로 직접 정하지 마세요.

Siemens 는 NCK 설정(`toNo`)값이라, 여러 채널이 **같은 번호**를 받을 수 있습니다. 그러면 그 채널들이 공구를 공유한다는 뜻입니다.

Fanuc·Mitsubishi 에는 공구 영역이라는 별도 계층이 없고 공구 데이터가 **경로(파트 시스템)에 딸려** 있습니다. 그래서 채널 번호가 그대로 돌아옵니다. 이 두 기종의 공구 주소는 `channel` 필터를 받지 않으므로, 경로를 지목하는 일을 `toolArea` 가 맡습니다.

## /machine/channel/executionStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

프로그램 **실행 상태** 코드입니다 (뜻은 `desc` 로 함께 옵니다). `operateMode`(무슨 모드인가)와 짝을 이루는 "지금 돌고 있는가":

- `0` = Reset · `1` = Stop · `2` = Hold · `3` = Run (실행 중)
- `4` = MSTR (Fanuc: 리트랙션/복구) · `5` = Interrupted (Siemens: 아래 참조) · `99` = Unknown (Fanuc 한정. Siemens 는 미등재 값을 상태 `-17` 에러로 돌려줍니다)

`1` 과 `2` 는 정지의 **종류**가 다릅니다. 누가, 어디서 세웠는가로 갈립니다:

- `Stop` = **프로그램이 예정된 지점에서** 세운 것. 싱글블록 모드로 블록이 끝났거나(세 기종 확인), M0/M1 을 만난 경우입니다. 항상 블록 경계에 서 있습니다.
  - ⚠️ **Fanuc 의 M0/M1 은 `Stop` 이 아닐 수 있습니다.** 제어기는 M 코드를 내보낸 뒤 PLC 의 완료 신호를 기다리는데, 그동안을 "사이클 진행 중" 으로 보아 **`3`(Run)** 을 냅니다. 테스트 환경(31i, M0 을 처리하지 않는 PLC 구성)에서 프로그램이 `M00` 블록에 멈춰 서 있고 조작반의 사이클 스타트 램프가 깜빡이는 동안 이 주소는 내내 `3` 이었고, 사이클 스타트를 다시 주자 재개됐습니다. PLC 가 M0 을 처리하는 기계에서는 다를 수 있어 확인된 범위만 적습니다. Siemens 는 같은 상황에서 `1`(Stop)입니다.
- `Hold` = **조작자가 임의 시점에** 세운 것. 조작반의 정지 키(피드홀드)를 누른 경우입니다. 블록 중간에서도 멈춥니다.

⚠️ **버튼 이름과 상태 이름이 어긋납니다** (업계 관례). 조작반의 **정지(Stop) 버튼을 누르면 상태는 `Hold`** 가 됩니다. `Stop` 상태는 버튼이 아니라 프로그램(M0/M1·싱글블록)이 만듭니다. 재개는 둘 다 Cycle Start 입니다.

**알람이 걸렸을 때의 답이 기종에 따라 다릅니다.** 같은 상황(없는 서브프로그램을 불러 자동운전이 멎음)을 두 기종의 테스트 환경에서 밟은 결과입니다:

| | `executionStatus` | `alarmStatus` |
|---|---|---|
| Fanuc | `1` (Stop) | `2` |
| Mitsubishi | `3` (Run), 리셋할 때까지 | `2` |

제어기마다 자기 자동운전 상태를 표현하는 방식이 달라서입니다. 디메시는 이것을 일괄로 뒤집지 않습니다. 알람이 가공을 멈추는지는 알람마다 다르고(경고성 알람은 안 멈춥니다), 우리에겐 알람별로 그걸 아는 지식이 없어 강등하면 멀쩡한 경우를 틀리게 만듭니다.

**비상정지 때의 답도 기종에 따라 다릅니다.** Fanuc 은 시험 환경(NC Guide)에서 자동운전 중 비상정지를 걸었을 때 `0`(Reset)이었고, Mitsubishi 도 테스트 환경에서 `0`(Reset), Siemens 는 `5`(Interrupted)입니다. Fanuc 쪽은 그 시뮬레이터에서 본 결과이므로 다른 설비에서도 같다고 단정하지 마세요. **비상정지 자체를 감지하려면 이 주소가 아니라 `/machine/channel/emergencyStatus` 를 쓰세요** - 그 주소가 기종 차이를 흡수합니다.

**그래서 `3`(Run)을 "지금 깎고 있다" 로 읽지 마세요.** 이 값은 자동 운전이 끝나지 않았다는 뜻이지 축이 움직인다는 뜻이 아닙니다. `3` 인데 서 있는 경우가 실제로 둘 확인됐습니다: **알람이 걸렸을 때**(`alarmStatus` 가 `0` 이 아님)와 **M 코드 완료를 기다릴 때**(알람도 없습니다). 정말 멈춰 있는지 알아야 하면 `/machine/channel/programCurrentBlock` 이 더 이상 바뀌지 않는지 보거나 `alarmStatus` 를 함께 읽으세요.

**Mitsubishi 는 `0`~`3` 만 냅니다.** 이 기종은 상태 코드가 아니라 자동 운전 플래그 셋(운전 중 · 진행 중 · 일시정지)을 주므로 디메시가 위 어휘로 합칩니다. `Stop` 과 `Hold` 의 구분은 벤더 정의와 그대로 맞아떨어집니다. 벤더가 말하는 "일시정지" 가 *명령을 실행하던 도중 멈춤* 이라 위 `Hold` 와 같은 상태이고, 자동 운전 중이면서 진행도 일시정지도 아닌 자리가 블록 경계에 선 `Stop` 입니다.

`5` (Siemens 전용) 는 **장비가 비정상이라 멈춘 경우**입니다. 테스트 환경에서 확인된 것은 둘입니다: **비상정지**와 **알람 정지**. 정상 정지(M0·싱글블록·조작자 정지)는 모두 `1`/`2` 로 갈리므로, `5` 를 보면 `/machine/channel/alarmStatus` 와 `/machine/channel/emergencyStatus` 로 어느 쪽인지 가르면 됩니다. 제어기는 정지 사유를 이보다 잘게 보고하므로 그 밖의 사유가 여기로 떨어질 수 있고, 그래서 이름은 여전히 "분류되지 않은 정지" 입니다. "멈췄다" 는 사실은 확실하니, 종류가 중요하지 않은 소비자는 `1`/`2`/`5` 를 묶어 "정지" 로 다뤄도 됩니다.

## /machine/channel/operateMode
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

현재 운전 모드 코드입니다 (뜻은 `desc` 로 함께 옵니다). 기종 무관 통일 코드:

- `0` = Jog · `1` = MDI · `2` = Memory (자동) · `5` = 모드 없음 · `6` = Edit · `7` = Handle (핸들)
- `8` = Teach in Jog · `9` = Teach in Handle · `10` = INC feed · `11` = Reference (원점복귀) · `12` = Remote (DNC)
- `13` = Jog-REPOS · `14` = MDI-Reference · `15` = MDI-Teach in · `16` = MDI-Teach in-Reference · `17` = Auto-Teach in-Reference · `18` = MDI-REPOS · `19` = MDI-Teach in-REPOS
- `99` = Unknown

`13`~`19` 는 **Siemens 전용**입니다. 기본 모드(Jog/MDI/Auto)에 보조 기능(REPOS·원점복귀·Teach in)이 겹쳐진 상태로, 조작반에서 그 조합을 고르면 나옵니다 (기능 매뉴얼 K1: JOG 는 REF·REPOS, MDI 는 REF·REPOS·Teach in 을 겹칠 수 있습니다). Fanuc 은 같은 상황을 기본 모드 코드로만 내보내므로 이 값이 나오지 않습니다.

Mitsubishi 의 **RAPID**(수동 급속이송)는 `0`(Jog) 로 나옵니다. 수동 연속 이송이라는 점에서 Jog 와 같은 부류이고, 기종이 늘 때마다 번호를 새로 만들지 않는다는 규칙을 따릅니다. 조작반의 STEP 은 `10`(INC feed), TAPE 는 `12`(Remote)입니다.

`5`(모드 없음)는 **Fanuc 과 Mitsubishi** 에서 나옵니다. 어느 기본 모드도 선택돼 있지 않은 상태로, 조작반이 Fanuc 은 모드 자리에 `****` 를, Mitsubishi 는 "모드없음" 을 표시합니다. Mitsubishi 는 다계통 장비에서 **계통마다 모드가 따로**라 흔히 봅니다 (한쪽만 쓰는 동안 다른 계통이 이 값입니다. 그때 그 계통의 `alarmList` 에 `M01 0101 운전모드없음` 이 함께 올라옵니다). `99`(Unknown)와 다릅니다: 이쪽은 장비가 "모드 없음" 이라고 분명히 답한 것이고, `99` 는 우리가 그 값을 해석하지 못한 것입니다. Siemens 엔 대응 상태가 없습니다.

## /machine/channel/emergencyStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

비상정지 상태입니다 (뜻은 `desc` 로 함께 옵니다): `0` = 정상, `1` = 비상정지. Fanuc 은 `2`(Reset: E-stop 을 해제하는 순간의 과도값, 1초 미만)가 스칠 수 있습니다 (테스트 환경에서 확인). `0` 이 아니면 "정상 아님" 으로 다루면 안전합니다.

**비상정지를 감지할 때는 이 주소를 쓰세요.** 비상정지가 드러나는 통로는 기종마다 달라서, 알람 목록에서 직접 찾는 방식은 기종에 따라 결과가 갈립니다. 이 주소가 그 차이를 흡수해 세 기종에서 같은 뜻의 값을 냅니다.

Mitsubishi 는 알람 목록에 `EMG` 구분이 있으면 `1` 입니다. 비상정지의 **원인과 무관하게** 잡습니다.

**Siemens 는 두 단계로 판정합니다.** 모드 그룹 준비 신호(`readyActive`, PLC 인터페이스 DB11 DBX6.3)가 켜져 있으면 추가 통신 없이 `0` 이고, 꺼져 있을 때만 알람 스냅샷을 가져와 **비상정지 알람 `3000` 이 있으면 `1`, 없으면 `0`** 입니다. 준비 신호는 비상정지 말고도 "모드 그룹 준비 해제" 반응을 가진 알람(드라이브·측정계·원점복귀 실패 등)이 전부 끄기 때문에, 그것만 보면 Fanuc 의 비상정지 신호·Mitsubishi 의 `EMG` 보다 넓은 뜻이 됩니다 (기능 매뉴얼 A2). 그런 경우 이 값은 `0` 이고 멈춘 원인은 `alarmStatus`(`2`)와 `/machine/channel/alarmList` 가 말합니다. 비상정지를 풀어도 `3000` 은 확인(acknowledge)·리셋될 때까지 남으므로 `1` 이 그만큼 더 유지되며, 준비 해제 상태에서 알람 스냅샷을 가져오지 못하면 지어내지 않고 상태 `-17` 입니다.

## /machine/channel/motionStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축 이동 상태 코드입니다 (뜻은 `desc` 로 함께 옵니다):

- `0` = None/Idle · `1` = Motion (이동 중) · `2` = Dwell (드웰 중)
- `3` = 다계통 동기 대기 (Fanuc) · `4` = Not dwelling (드웰은 아님, 그 이상은 모름)

**드웰은 정지 중에도 계속 줄어듭니다** (Siemens 테스트 환경 확인). 조작자가 정지시켜 `executionStatus` 가 `2`(Hold)인 동안에도 `G4` 의 잔여 시간은 `0` 까지 내려가고, 그 뒤 이 주소는 `2`(Dwell)에서 `4`(Not dwelling)로 바뀝니다. 정지가 드웰을 얼리지는 않으므로, 재개하면 그 블록은 이미 끝나 있어 다음 블록부터 진행합니다.

**`4` 는 `0`·`1`·`3` 의 상위집합입니다**. 그 셋 중 하나인데 어느 것인지 좁힐 수 없다는 뜻입니다. Siemens·Mitsubishi 에서는 디메시가 잔여 드웰 시간으로 판정하므로 그 두 기종에서는 `2` 아니면 `4` 만 나옵니다 (`0`·`1`·`3` 은 Fanuc 전용).

축이 실제로 움직이는지가 필요한데 기종이 `4` 를 낸다면 이 주소로는 알 수 없습니다. 자동 운전 중인지는 `/machine/channel/executionStatus` 로 확인하세요 (수동 조작 중의 이동은 두 주소 모두 답하지 않습니다).

## /machine/channel/alarmStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

**알람 심각도**입니다. 기종과 무관하게 **`0` / `1` / `2` 세 값**만 반환하며, 그 기종에서 무엇이 문제인지는 `desc` 로 함께 옵니다.

| 값 | 의미 |
|---|---|
| `0` | 정상: 알람도 메시지도 없음 |
| `1` | 경고: 장비 이상은 아님 (안내 메시지, 정상적인 프로그램 정지 등) |
| `2` | 알람: **장비 이상** |

**신호등으로 생각하면 됩니다**: `0` 초록 · `1` 노랑 · `2` 빨강. 세 값을 그대로 경광등이나 화면 표시등에 물리면 됩니다. 이 눈금이 **제어기가 자기 메시지를 칠하는 방식과 같기 때문**입니다: Mitsubishi 매뉴얼은 NC 알람과 PLC 알람을 빨강 배경으로, 경고·스톱 코드·오퍼레이터 메시지를 노랑 배경으로 칠합니다. 그래서 노랑이 들어오는 것은 고장 알림이 아니라 **"장비가 지금 무언가를 기다리고 있다"** 는 신호입니다.

💡 **가장 튼튼한 사용법은 `0` 인지 아닌지입니다.** `0` 은 세 기종에서 뜻이 정확히 같고("알람도 메시지도 없음") `alarmCount` 가 `0` 인 것과 맞물립니다. 반면 `1` 과 `2` 의 경계는 **그 기종이 자기 알람을 어떻게 분류하느냐**에 기대므로 기종마다 미세하게 다를 수 있습니다. 심각도로 분기해야 한다면 `alarmList` 의 항목별 `severity` 와 본문을 함께 보세요.

⚠️ **`1` 은 "무언가 잘못됐다" 가 아닙니다. 정상 가공 중에도 뜨고, 그 빈도가 기종마다 다릅니다.** 아무 문제 없는 자동 운전(드웰만 도는 프로그램)을 두 기종의 테스트 환경에서 측정한 결과입니다:

| | `alarmStatus` | 목록에 담긴 것 |
|---|---|---|
| Fanuc | `0` | 없음 |
| Mitsubishi | `1` | 스톱 코드 |

**Mitsubishi 에만 "스톱 코드" 라는 채널이 있기 때문입니다.** 제어기가 *자동 운전의 상태*를 알리는 자리로, 벤더 매뉴얼도 알람(빨강)이 아니라 경고와 같은 노랑으로 칠합니다. 담기는 것은 `T03 0301`(싱글블록 정지)·`T10`(M 코드 완료 대기)·`T02 0202`(소프트 리밋에 걸린 축 있음) 같은 것들이라 **고장이 아니라 "지금 이걸 기다리는 중"** 입니다. 위에서 확인했을 때 올라와 있던 것도 `T10` 이었고, 프로그램이 끝나자 사라졌습니다.

**코드의 앞 글자는 계열이고 뒤 네 자리가 사유입니다** (`T01` 사이클 스타트 불가 · `T02` 자동 운전 정지 · `T03` 블록 단위 정지 · `T04` 대조 정지 · `T10` 완료 대기). 그래서 같은 `T02` 라도 `0202` 는 소프트 리밋, `0204` 는 피드홀드입니다. **조작자가 피드홀드나 싱글블록을 누르기만 해도 스톱 코드가 올라와 이 주소가 `1` 이 됩니다** (테스트 환경에서 확인: 피드홀드 `T02 0204`, 싱글블록 정지 `T03 0301`). 사유가 필요하면 `alarmList` 항목의 `code` 를 네 자리까지 보세요. Fanuc·Siemens 는 이런 상태를 알람 목록에 싣지 않아 `0` 이 유지됩니다.

**그래서 `1` 을 운영자 호출 신호로 쓰지 마세요**. Mitsubishi 장비에서는 정상 자동 운전 중에도 `1` 이 됩니다. 호출에는 `2` 를 쓰거나 `alarmList` 항목의 `severity` 와 `category` 를 보고 판단하세요.

**기준은 "장비에 이상이 있느냐" 이지 "가공이 멈췄느냐" 가 아닙니다.** 둘은 대개 함께 가지만 갈릴 때가 있습니다. `M0`(프로그램 정지)나 싱글블록으로 선 기계는 멀쩡하므로 `2` 가 되지 않습니다 (Mitsubishi 는 스톱 코드로 값 `1`, Fanuc·Siemens 는 목록에 싣지 않아 값 `0`). 지금 가공이 멈췄는지는 `/machine/channel/executionStatus` 로 판단하세요.

- **숫자는 기종이 늘어도 이 셋뿐입니다.** 벤더 코드를 그대로 내보내지 않으므로, 어느 기종에 붙였는지 몰라도 `value` 로 바로 분기할 수 있습니다.
- **원인은 `desc` 로 옵니다.** Fanuc 은 원인 계열(`{"value": 1, "desc": "Memory backup battery voltage low (CNC or Amplifier)"}`), Siemens 는 가장 무거운 알람의 본문(`{"value": 2, "desc": "Emergency stop"}`), Mitsubishi 는 알람 종류(`{"value": 2, "desc": "NC alarm"}`). `desc` 는 사람이 읽는 문자열이므로 **분기 조건으로 쓰지 마세요.** 분기는 `value` 로.
- **Mitsubishi**: **제어기가 메시지를 칠하는 색 그대로**입니다. 빨강(NC 알람·PLC 알람 메시지)은 `2`, 노랑(NC 경고·스톱 코드·오퍼레이터 메시지)은 `1`. 벤더 API(`GetAlarm2`)는 NC 알람과 NC 경고를 한 종류로 주므로 **카테고리 코드로 색을 되가릅니다**: 운전 에러 `M00`/`M01`, 운전 경고 `M50`, 서보 경고 `S52`·`S53`, 스마트 안전 경고 `V50`~`V54` 가 노랑(`1`)이고, 그 밖의 NC 알람(`S01`~`S05`·`S51`·`Y`·`Z`·`Z7x`·`Z8x`·`EMG`·`L`·`U`·`N`·`P`·`V01`~`V07` 등)과 PLC 알람은 빨강(`2`)입니다. 모르는 카테고리는 `2` 입니다. 그래서 운전 모드 미선택(`M01 0101`)이나 오버라이드 0(`M01 0102`) 같은 대기 상태는 `1` 이고, 비상정지(`EMG`)는 `2` 입니다. ⚠ 소프트 스트로크 엔드는 Mitsubishi 가 `M01 0007` 운전 에러(노랑 → `1`)로 다루지만 Fanuc 은 OT 알람(`2`)입니다. 같은 상황을 두 제어기가 다른 등급으로 치는 자리라, 이 주소는 각 제어기의 등급을 그대로 따릅니다
- 판정이 애매한 벤더 코드는 **보수적으로 `2`** 로 분류합니다. 정지를 경고로 낮춰 부르는 쪽이 그 반대보다 위험하기 때문입니다. Fanuc 제어기가 SDK 에 없는 코드를 내보내도 `2` 로 분류합니다. Siemens 는 서버가 이벤트마다 주는 심각도를 씁니다. 오류(`1000`)만 `2` 이고 벤더가 **경고(`500`)라 답한 것은 `1`** 입니다. `alarmList` 의 `severity` 와 **같은 기준**입니다.
- 알람 **목록·번호·메시지**가 필요하면 `alarmList`, **개수**만 필요하면 `alarmCount` 를 쓰세요. 이 주소는 고빈도 폴링용 요약이라 목록보다 가볍습니다 (Fanuc 은 오퍼레이터 메시지 존재 확인 때문에 왕복이 하나 더 붙지만, 다른 상태 주소와 함께 물으면 그 하나만 늘어납니다).
- ⚠️ **`alarmStatus` 가 `0` 이 아닌데 `alarmCount` 가 `0` 일 수 있습니다.** Fanuc 은 요약이 조작반 **상태표시줄**까지 보는 반면 목록은 알람·메시지만 담아, 요약에만 값이 있는 상태가 존재합니다 (배터리 저하·전원 경고·절연 저하 계열). 반대 방향(`alarmStatus` 가 `0` 인데 목록에 항목이 있음)은 없습니다.
- **Siemens 는 `channel` 값을 쓰지 않습니다** (`alarmList`/`alarmCount` 와 같은 방침). 셋 다 NCK 전역 스냅샷에서 나오므로 한 번의 요청으로 함께 답하며, 서로 어긋나지 않습니다. 채널별로 나눌 수 없는 이유는 알람 이벤트에 채널 정보가 없기 때문입니다. 알람 소스는 `HMI`/`NCK`/`PLC` 셋뿐입니다. 어느 기종이든 `channel` 값은 범위 검증됩니다.
- **Siemens 는 기계 제작사의 PLC 알람도 함께 봅니다** (유압·윤활·도어 인터록 등). 종전에는 NCK 알람만 보는 노드를 읽어 그런 알람이 떠 있어도 `0`(정상)이 나왔습니다.

## /machine/channel/alarmCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

활성 알람/메시지 **개수**입니다 (= `alarmList` 항목 수, severity 불문). 반환 `int`. 대시보드 배지처럼 개수만 필요할 때 쓰세요.

- **비용 주의 (Fanuc)**: 목록을 받아 세므로 내부적으로 `alarmList` 와 **같은 비용**입니다. 둘을 함께 배치 요청하면 fetch 1회로 합쳐집니다. 저비용 존재 판정만 필요하면 `alarmStatus` 를 쓰세요
- **비용 주의 (Siemens)**: 위 둘과 같습니다. 개수를 `alarmList` 와 **같은 이벤트 스냅샷**에서 세므로 같은 비용이고, 함께 배치 요청하면 fetch 1회로 합쳐집니다. NCK 전역 개수라 `channel` 값은 무시됩니다 (`alarmList` 와 동일 방침)
- **비용 주의 (Mitsubishi)**: Fanuc 과 같습니다. 목록을 받아 세므로 `alarmList` 와 같은 비용이고, 함께 배치 요청하면 fetch 1회로 합쳐집니다

## /machine/channel/alarmList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

채널의 **활성 알람 + 오퍼레이터/매크로 메시지** 목록입니다. 반환 타입 `objectArray`, 없으면 빈 배열 `[]`. Siemens 는 알람이 NCK 전역이라 `channel` 값은 무시됩니다.

원소: `{"code": "OH0700", "message": "SPINDLE OVERHEAT", "category": "Overheat", "severity": "alarm", "raisedAt": "2026-07-29T11:11:03Z"}`

키 집합은 **기종과 무관하게 항상 같습니다.** 값이 없으면 키가 빠지는 게 아니라 `null` 입니다 (`entry` 와 같은 규약). `severity` 는 `"alarm"` / `"warning"` 두 값뿐입니다.

- **code**: **조작반이 보여주는 식별자 그대로의 문자열**입니다. Fanuc 은 알람 타입 약어 + 4자리 번호(`"OT0501"`·`"PS0010"`, 오퍼레이터/매크로 메시지는 메시지 번호 `"2000"`), Siemens 는 알람 번호(`"4230"`·`"700015"`. 번호 범위가 출처를 말합니다: `0`~`9999` 일반, `10000`~`19999` 채널, `20000`~`29999` 축·스핀들, `60000`~`69999` 사이클, `100000`~`199999` HMI, `200000`~`299999` 드라이브(SINAMICS), `300000`~`399999` 드라이브·I/O, `400000`~`899999` PLC, 그중 `500000`~`899999` 는 기계 제작사가 정의. 진단 매뉴얼 §2.3), Mitsubishi 는 구분 + 상세(`"M01 0101"`·`"S01 0051"`·`"EMG EXIN"`). 문자가 섞이고 앞자리 0 이 의미를 가지므로 **숫자로 변환하지 말고 문자열 그대로** 매뉴얼에서 찾으세요. 식별자를 못 얻으면 `null` 이 아니라 `""` 입니다 (세 기종 모두 실제로는 항상 채워집니다). 1.2.0 에서 정수 `number` 를 대체했습니다
- **message**: 표시 텍스트
- **category**: Fanuc: 알람은 원인 계열 (`Servo`, `Overheat`, `Spindle`, `PLC` 등: 미정의 타입은 숫자 문자열), 메시지는 출처 (`Operator message` = PMC/외부입력, `Macro message` = 파트프로그램 #3006). Siemens: 서버가 알려주는 소스 (예: `NCU`: 비어 있으면 `Alarm`). Mitsubishi: 조작반에 뜨는 알람 구분 (`EMG`, `S01`, `M01` 등)
- **severity**: `"alarm"` = 장비 이상(가공 불가) / `"warning"` = 정보성(가공 가능). Fanuc: 알람은 백그라운드 편집 에러(BG)만 warning 이고 나머지 alarm, 오퍼레이터/매크로 메시지는 전부 warning. Siemens: 서버의 심각도(1~1000)를 500 경계로 번역. Mitsubishi: 제어기가 빨강으로 칠하는 것(NC 알람·PLC 알람)은 alarm, 노랑(NC 경고 `M00`/`M01`/`M50`/`S52`/`S53`/`V5x`·스톱 코드·오퍼레이터 메시지)은 warning 이며 `alarmStatus` 와 같은 기준입니다. "지금 가공이 멈췄는가"는 이 필드가 아니라 `executionStatus` 로 판단하되, **알람 중에는 그 값도 기종에 따라 `3`(Run)일 수 있습니다** (그 주소 설명 참조). warning 인데 정지 상태면 매크로 `#3006` 등 오퍼레이터 개입 대기입니다
- Fanuc 은 한 번에 활성 **알람 최대 100건**, 오퍼레이터/매크로 **메시지 최대 17건**까지 실어 옵니다 (벤더 API 버퍼 한도). Mitsubishi 는 알람 종류별로 **10건씩**이라 합계 **최대 40건**입니다 (벤더 API 상한). 목록은 무거운 종류부터 담기므로, 넘칠 때 잘리는 쪽은 가벼운 메시지입니다.
- **raisedAt**: 발생 시각. **`Z` 로 끝나는 UTC** 입니다 (`"2026-07-29T11:11:03Z"`). Siemens 는 실제 시각, Fanuc·Mitsubishi 는 항상 `null` (활성 알람에 시각 정보가 없음)
  - 장비 화면(HMI)이 보여주는 시각과 **숫자가 다릅니다.** HMI 는 장비 시간대로 표시하고 이 값은 UTC 입니다. 같은 순간을 다르게 표기한 것이며, 변환은 장비의 시간대를 아는 쪽(호스트 앱)이 합니다. OPC-UA 이벤트에는 시간대 오프셋 필드가 있지만, 실측한 장비에서는 비어 있었습니다
  - **`/machine/currentDateTime` 과 직접 빼지 마세요.** 시간대가 다를 뿐 아니라 **출처 시계가 다릅니다.** 한 장비에서 두 시계가 18분가량 어긋나 있는 것을 실측했습니다 (시계 설정은 현장마다 다릅니다)

## /machine/channel/singleBlockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

싱글블록 스위치 상태입니다 (`true` = 켜짐). Fanuc 은 F4 신호 비트, Siemens 는 `singleBlockActive`, Mitsubishi 는 조작반 신호 블록의 PLC 출력(Y) 비트입니다.

## /machine/channel/dryRunOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

드라이런 스위치 상태입니다 (`true` = 켜짐). `channel` 필터. Fanuc·Siemens 와 함께 **Mitsubishi 도 지원**하며, Mitsubishi 는 `singleBlockOn` 과 같은 조작반 신호 블록의 PLC 출력(Y) 비트라 두 주소를 함께 요청하면 한 번의 조회로 처리됩니다.

## /machine/channel/optionalStopOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens"]
write: []
```

옵셔널 스톱(M01 유효) 스위치 상태입니다 (`true` = 켜짐). `channel` 필터. **Siemens 만 지원**합니다.

**Fanuc 은 상태 `-20` 입니다.** 옵셔널 스톱 스위치는 기계 조작반에서 PMC 로 들어가는 입력이고 제어기는 그 상태를 신호로 내지 않습니다 (Connection Manual B-64483EN-1 §10 이 M00/M01 은 코드·스트로브·디코드 신호만 보내고 정지와 옵셔널 스톱 제어는 PMC 쪽에서 설계한다고 밝힙니다). 제어기가 내는 `DM01`(`F9.6`)은 프로그램이 `M01` 블록을 지령했다는 디코드 신호라 스위치와 무관합니다 (NC Guide 실측: 스위치를 켜도 `F9` 는 `0`). 래더가 쓰는 디바이스를 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요.

**Mitsubishi 도 상태 `-20` 입니다.** PLC 인터페이스 매뉴얼에 따르면 이 제어기에는 옵셔널 스톱 스위치의 NC 신호가 없습니다. `M01` 이 나오면 **기계 제작사의 PLC 가 자기 스위치 입력을 보고** 싱글블록 신호(`SBK`)를 걸어 멈추게 하는 구조라, 스위치 상태는 그 장비 래더의 디바이스에만 있고 기종 무관 주소로는 읽을 수 없습니다 (`plcAddress` 와 같은 부류). 그 장비의 래더가 쓰는 디바이스를 알면 `/machine/plcAddress/plcType/plcValue` 로 직접 읽을 수 있습니다.

## /machine/channel/blockSkipOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

블록 스킵(`/`) 스위치 상태입니다 (`true` = 켜짐). `channel` 필터. 세 기종 모두 지원합니다.

**스킵 레벨이 여러 개인 기종에서도 이 주소는 평범한 `/` 하나만 봅니다.** 블록 앞에 번호를 붙여(`/2`·`/3` …) 구간마다 다른 스위치로 건너뛰게 하는 기능이 있는 제어기들이 있는데 (Siemens 는 레벨 `0`~`9`, Mitsubishi 는 `BDT1`~`BDT9` 로 문서화합니다), 이 주소가 답하는 것은 언제나 **번호 없는 `/`** 입니다. 번호 붙은 레벨을 읽는 주소는 없습니다.

## /machine/channel/machineLockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

머신 록(축 이동 잠금) 상태입니다 (`true` = 켜짐). `channel` 필터. 세 기종 모두 지원하며, Siemens 는 프로그램 테스트(`progTestActive`) 상태입니다.

**Mitsubishi 는 이 신호를 축별로 내는데 이 주소는 채널 하나입니다.** 그래서 **그 채널의 전 축이 잠겼을 때만 `true`** 입니다. 일부 축만 잠긴 상태를 켜짐으로 부르면 "아무것도 움직이지 않는다" 로 읽혀 실제 가공을 시험 운전으로 오판하게 되기 때문입니다. 조작반 스위치는 전 축을 함께 움직이므로 통상적인 장비에서는 이 구분이 드러나지 않습니다.

## /machine/channel/rapidOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

급속이송 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. **Fanuc 은 기계 제작사 래더가 고른 방식에 따라 갈립니다** (Connection Manual B-64483EN-1 §7.1.7): 기본인 `ROV1`/`ROV2`(`G14`) 방식은 단계식이라 `100`/`50`/`25`/`0` 네 값만 나오고(`0` 은 F0, 파라미터 `1421` 의 속도), 1% 단계 방식(`HROV`, `G96`)이면 `0`~`100` 의 정수, 0.1% 단계 방식(`FHROV`, `G353`)이면 정수로 떨어지는 값은 그대로 내고 `87.5` 처럼 정수가 아닌 값은 상태 `-17` 입니다 (이 주소는 `int`). 다경로 장비는 그 경로의 신호(경로마다 `1000` 씩 밀린 주소)를 읽습니다. Siemens 는 연속값이며, 이송 오버라이드와 같이 스위치 값이 아니라 **유효값**입니다. PLC 유효 신호 `DB21.DBX6.6` 이 꺼진 동안은 다이얼과 무관하게 값 `100` 이 나옵니다 (언제 끄는지는 기계 제작사 래더에 달려 있고, 실측한 장비는 비상정지부터 조작반의 준비 버튼(MC READY)을 누를 때까지 껐습니다). 급속 전용 다이얼이 없는 장비는 래더가 이송 다이얼을 급속에도 적용하는 구성이 흔한데, 그렇게 적용될 때 100% 를 넘는 이송 오버라이드는 급속에는 제어기가 100% 로 상한을 겁니다 (Basic Functions 매뉴얼 §18.4). 실측: 이송 다이얼 110 에서 이송 오버라이드 주소는 값 `110`, 이 주소는 값 `100` 이었습니다. **Mitsubishi 는 기계 제작사 래더가 고른 방식에 따라 갈립니다** (PLC 인터페이스 매뉴얼): 방식 선택 신호 `ROVS`(`YC6F`)가 꺼져 있으면 코드 신호 `ROV1`/`ROV2`(`YC68`/`YC69`)라 `100`/`50`/`25`/`0`(매뉴얼의 `1%` 단계를 Fanuc 과 같이 `0` 으로 냅니다) 네 값이고, 켜져 있으면 계통별 레지스터 `R2502`(0~100% 1% 단위)라 연속값입니다.

두 기종의 `0` 은 "멈춤" 이 아니라 **그 장비가 정한 가장 느린 급속이송 단계**입니다 (조작반의 최저 단계). 실제 속도는 장비 설정에 달려 있어 이 주소로는 알 수 없습니다.

## /machine/channel/feedOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

이송 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. Fanuc 은 PMC `G12` 신호(`*FV0`~`*FV7`, 반전 2진 0~254%)에서 읽으며, 신호가 전부 꺼진 상태는 제어기와 같이 `0` 으로 냅니다 (Connection Manual B-64483EN-1 §7.1.6). 스위치 신호 값이라 오버라이드 취소 신호(`OVC`)가 켜져 실제 배율이 100% 인 동안에도 스위치 값이 나오고, 제2 이송 오버라이드(`G13`)는 반영하지 않습니다. 다경로 장비는 그 경로의 신호(2경로 `G1012` 처럼 경로마다 `1000` 씩 밀린 주소)를 읽습니다. Siemens 는 `feedRateIpoOvr` 노드에서 읽는데, 이것은 스위치 값이 아니라 **보간기에 실제로 걸리는 유효값**입니다. PLC 유효 신호 `DB21.DBX6.7` 이 꺼진 동안 제어기는 오버라이드를 내부적으로 100% 로 두므로 (Basic Functions 매뉴얼 §18.4) 다이얼과 무관하게 값 `100` 이 나옵니다. 이 신호를 언제 끄는지는 기계 제작사 래더에 달려 있습니다. 실측한 장비는 비상정지를 걸면 꺼지고 비상정지를 해제하고 리셋한 뒤에도 꺼진 채라, 조작반의 준비 버튼(그 장비 표기로 MC READY)을 누를 때까지 값 `100` 이 나오다가 준비가 켜지자 다이얼 값으로 복귀했습니다. 그동안에도 다이얼 위치 자체는 PLC 에 정상 도착하고 있었습니다. **Mitsubishi 는 기계 제작사 래더가 고른 방식을 따라 읽습니다** (PLC 인터페이스 매뉴얼): 방식 선택 신호 `FVS`(`YC67`)가 꺼져 있으면 오버라이드 코드 신호(`YC60`~`YC64`, 0~300% 10% 단계)를, 켜져 있으면 계통별 레지스터 `R2500`(0~300% 1% 단위)을 읽습니다. 코드 신호가 전부 꺼진 상태는 제어기가 "이전 값 유지" 로 다루므로 읽을 값이 없어 상태 `-17` 입니다.

## /machine/channel/feedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

지령 이송속도 (F 지령값)입니다. 반환 `float`. Fanuc 은 모달 F, Siemens 는 `cmdFeedRateIpo`, Mitsubishi 는 `F command feed speed`(FA)입니다.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/feedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

채널의 실제 이송속도입니다. **공구 끝이 프로그램 경로를 따라가는 속도**. 반환 `float`.

`F` 지령이 정하는 값이 이것입니다. 이동 방향이 바뀌어도 이 속도는 지령대로 유지되고, 오버라이드·가감속·코너 감속·피드홀드가 걸리면 그만큼 떨어집니다. "지금 지령대로 깎이고 있나" 는 이 값으로 판단합니다.

Fanuc 은 `actf`, Siemens 는 `actFeedRateIpo` 입니다. Mitsubishi 는 벤더가 실효 이송을 **자동 운전용과 수동 조작용으로 나눠** 주므로 둘을 함께 읽어 냅니다. 조그·핸들로 축을 움직이는 중에도 값이 나옵니다.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axisCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

채널의 **사용자 축 수**입니다. 연결 시 캐싱. `axis` 필터의 유효 범위가 `1`~이 값입니다.

기하축과 **비스핀들 보조축**(인덱싱 로터리 테이블·심압대 등)을 함께 세고 스핀들은 제외합니다. 스핀들은 `spindleCount` 와 `spindle` 필터가 담당합니다.

**경로마다 다릅니다.** 다경로 장비에서 채널별로 축 구성이 다르며, 축이 하나도 없는 경로는 `0` 입니다. 그 채널의 축 주소들은 상태 `-20` 으로 답합니다.

## /machine/channel/axis/axisName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축 이름입니다 (예: `"X"`, `"Z1"`). 반환 `string`, 읽기 전용. `axis` 번호 ↔ 실제 축 대응을 확인할 때 사용.

출처는 Fanuc 서보 부하 미터 데이터의 축 이름(`cnc_rdsvmeter`, 연결 때 캐시), Siemens `/Channel/GeometricAxis/name`, Mitsubishi 축 파라미터 `#1013` 입니다.

## /machine/channel/axis/machinePosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축의 기계 좌표(machine coordinate)입니다. `axis` 필터로 축을 지정하며, 범위(`axis=1-3`)나 복수 지정(`axis=1,2`)이 가능합니다. 반환 타입은 `float` (64비트 배정밀도).

위치 계열 4종(machinePosition/workPosition/distanceToGo/relativePosition)은 모두 **실거리**: 장비 설정 단위(mm/inch) 그대로이며 조작반 표시와 일치합니다 (Fanuc 의 내부 정수 표현은 SDK 가 축별 소수점 배율로 정규화). 네 값 모두 축이 원점을 확립한 뒤에만 유효합니다. 전원 투입 직후라면 `/machine/channel/axis/axisReferencedOn` 을 먼저 확인하세요 (Mitsubishi 는 그 주소가 상태 `-20` 이라 조작반에서 확인하거나, "지금 원점 위치에 있는가" 만 주는 `/machine/channel/axis/axisAtReferencePositionOn` 으로 갈음하세요) (미확립 상태에서도 그럴듯한 좌표값이 에러 없이 반환되므로, 값만 봐서는 가려낼 수 없습니다).

⚠️ **이 값은 공구 기준점(스핀들 끝단)의 좌표입니다.** `workPosition` 은 공구 선단이라 두 값의 차이에 **공구 길이 보정**이 들어갑니다. `machinePosition − 영점이동 = workPosition` 은 성립하지 않습니다 (활성 워크좌표계의 회전·배율·미러도 함께 걸립니다). 워크 좌표가 필요하면 직접 계산하지 말고 `workPosition` 을 읽으세요.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/workPosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축의 공작물 좌표(절대 좌표)입니다. 반환 `float`.

이 값은 **공구 선단** 기준이며 활성 영점이동·회전·배율·미러·공구 길이 보정이 **모두 적용된 결과**입니다. 장비가 계산한 최종 좌표라 `machinePosition` 에서 직접 구할 필요가 없습니다 (뺄셈으로는 맞지 않습니다).

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/relativePosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축의 상대 좌표입니다. 반환 `float`.

⚠️ **기준점이 고정이 아닙니다.** 조작자가 원점 설정·카운터 리셋(또는 `G92` 프리셋)으로 언제든 0 으로 만들 수 있는 카운터라, 이 값만으로는 기계의 어디인지 알 수 없습니다. 고정 기준이 필요하면 `machinePosition`, 공작물 좌표가 필요하면 `workPosition` 을 쓰세요.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/distanceToGo
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

현재 블록에서 축의 **잔여 이동량**입니다. 반환 `float`.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/totalWorkOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

지금 **실제로 걸려 있는 영점이동 총량**입니다 (축별 평행이동). 반환 `float`. **읽기 전용**. 조작반의 `Total WO` 행에 해당합니다.

`workOffsetValue` 가 **표에 저장된 값**이라면 이 주소는 **지금 적용되고 있는 값**입니다. 총량은 층으로 쌓입니다 (실측 장비의 값):

```
표에 저장된 값      workOffsetValue?workOffset=G54     80.400   활성 좌표계는 gModalCategory=7 로 확인
+ 표 밖의 이동      기준 오프셋 (실제값 설정·터치오프)     20.000   조작반 Basic reference 행
                   기본 프레임 · 프로그램 TRANS · 사이클 프레임
= 지금 걸린 총량    totalWorkOffsetValue              100.400
```

기준 오프셋은 작업자가 JOG 에서 실제값 설정이나 터치오프·측정 사이클로 원점을 잡을 때 들어가는 몫이라(Siemens 시스템 프레임 `$P_SETFRAME`), 어느 좌표계를 골라도 항상 더해집니다. 역할은 Fanuc·Mitsubishi 의 `EXT` 와 같지만 Siemens 에서는 표 밖에 있어 `workOffsetValue` 로는 보이지 않습니다. "설정은 그대로인데 부품이 어긋난다" 를 진단할 때 두 주소를 비교하세요. 저장된 값만 읽으면 표 밖에서 더해진 몫이 보이지 않습니다.

⚠️ **공구 보정은 이 층에 없습니다.** 이 값은 "공작물 원점을 어디로 옮겼나" 까지이고, "공구가 얼마나 긴가" 는 그 다음 층입니다. 그래서 어느 기종에서든 `machinePosition` 에서 `workPosition` 을 빼도 이 값이 나오지 않습니다. 그 차이에는 공구 길이 보정이 섞여 공구축 값이 어긋납니다. 회전·배율·미러도 여기 없습니다. 공작물 좌표가 필요하면 `/machine/channel/axis/workPosition` 을 읽으세요. 장비가 그 모두를 적용한 결과입니다.

`workOffset` 필터를 받지 않습니다. "지금 걸린 것" 이라 지정자를 고를 대상이 없습니다. `axis=1-3` 확장을 지원하고, 쓰기는 지원하지 않습니다 (합산 결과라 되돌려 쓸 대상이 아닙니다).

**Siemens 전용**입니다. Fanuc·Mitsubishi 는 디메시가 사용하는 벤더 API 범위에서 이 총량을 직접 내주는 호출을 찾지 못해 지원하지 않습니다. 그쪽에서는 `workOffset=EXT` 로 공통 오프셋을 따로 읽어 활성 좌표계 값과 더하세요. 다만 그 합은 **이 주소와 뜻이 다릅니다**. 프로그램이 건 시프트(`G92`·`G52` 등)가 빠지는데, 그것이 바로 이 주소가 드러내려는 차이입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/axisFeedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

그 축 방향의 실제 이송 **성분**입니다 (**Siemens 전용**). 반환 `float`. Fanuc 은 상태 `-20` 입니다 (디메시가 쓰는 FOCAS2 호출 범위에서 축별 이송 성분을 얻지 못합니다).

공구가 경로를 따라가는 속도 자체가 아니라, 그 속도를 축 방향으로 분해한 몫입니다. 그래서 **같은 지령에서도 이동 방향에 따라 계속 변합니다.** XY 평면에서 `F1000` 을 지령했을 때 X 축만 움직이는 구간에서는 이 값이 1000 이지만, 45° 대각선 구간에서는 약 707 입니다.

축 성분들을 **더해도 경로 속도가 되지 않습니다** (벡터 크기라 707+707 이 아니라 √(707²+707²)=1000). 이 값은 해당 축이 자기 속도 한계에 걸려 경로를 제한하고 있는지 보는 용도입니다.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Mitsubishi 도 상태 `-20` 입니다.**

## /machine/channel/axis/axisLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축(서보) 부하율입니다. 반환 `float` + `unit:"%"` (세 기종 동일). Fanuc 은 서보 부하 미터, Siemens 는 드라이브 부하(`$VA_LOAD`, PROFIdrive 드라이브에서만 제공), Mitsubishi 는 서보 모니터의 부하 전류(정격 대비 비율)입니다. 셋 다 **실측값의 현재값**입니다. ⚠️ **Siemens 쪽은 값이 나오는 것을 보지 못했습니다.** 시험 장비(840D sl 벤치)에서 **늘 `0`** 이었습니다. 축이 1498mm/min 으로 움직이는 동안에도, 스핀들이 1425rpm 으로 도는 동안에도 `0` 이었고, 같은 순간 `axisCurrent`·`spindleLoad` 는 값이 움직였으므로 드라이브가 죽어서가 아닙니다.

**디메시의 읽기 문제가 아니라는 것은 확인했습니다.** 프로그램 운전으로 네 축이 모두 움직이고 스핀들 부하가 `spindleLoad` 로 정상 표시되는 순간, 이 값의 채널 블록과 전역 블록을 같은 요청으로 대조했는데 **양쪽 다 전 축이 `0`** 이었습니다. 남는 설명은 그 장비가 값을 주지 않는다는 것입니다 (매뉴얼이 PROFIdrive 드라이브 한정으로 적는 그대로). 다만 **값을 내는 장비는 실제로 보지 못했습니다.**

**그래서 Siemens 에서 이 주소를 감시에 쓰기 전에 값이 실제로 나오는지 확인하세요.** 제어기가 `0` 과 "값 없음" 을 구분해 주지 않아 읽기만 해서는 알 수 없습니다. 축을 움직이면서 `axisCurrent` 를 함께 읽어, 전류는 움직이는데 이 값이 계속 `0` 이면 그 기계에서는 이 주소로 부하를 감시할 수 없습니다.

**`axisCurrent` 와 같은 물리량입니다.** 서보는 토크가 전류에 비례하므로 부하를 재는 것이 곧 전류를 재는 것이고, 이쪽은 그 값을 **모터 정격 연속전류로 나눈 비율**입니다. 그래서 기계·축이 달라도 비교되는 반면(80% 는 어디서나 80%), 절대 전류값이 필요하면 `axisCurrent` 를 쓰세요. 둘 사이 환산에는 그 모터의 정격 전류가 필요한데 디메시는 그 값을 내지 않으므로, **한쪽만 지원하는 기종에서는 다른 쪽을 계산해낼 수 없습니다.**

## /machine/channel/axis/axisLoadCommandedPeak
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_ezsocket_mitsubishi"]
write: []
```

축 모터 **전류 지령**의 최근 2초 피크입니다. **Mitsubishi 전용**, 반환 `float` + `unit:"%"` (연속전류 환산이라 `axisLoad` 와 같은 척도).

`axisLoad` 와 두 가지가 다릅니다. 실측이 아니라 **지령**이고, 현재값이 아니라 **피크**입니다. 그래서 같은 부하 상태에서도 `axisLoad` 보다 높게 읽힙니다. 두 값을 같은 것으로 비교하지 마세요.

느린 주기로 폴링할 때 유용합니다. `axisLoad` 는 현재값이라 샘플링하는 순간에 따라 절삭 중에도 낮게 잡힐 수 있지만, 이 값은 직전 2초 안의 최댓값이라 그 사이 부하를 놓치지 않습니다.

**Fanuc·Siemens 는 상태 `-20` 입니다.** 지령 전류의 최근 피크를 내주는 것은 Mitsubishi 드라이브 모니터뿐입니다.

## /machine/channel/axis/axisCurrent
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

축 모터 전류입니다. 반환 `float` + `unit:"Ampere"` (양 기종 동일). Fanuc 은 서보 축 데이터의 부하 전류(암페어 항목), Siemens 는 드라이브 파라미터 `R0078`.

**`axisLoad` 와 같은 물리량이며 단위만 다릅니다** (그쪽은 모터 정격 대비 `%`). **Mitsubishi 는 상태 `-20`** 이고, 같은 측정이 `axisLoad` 로 `%` 로 나옵니다.

**Siemens**: 값이 드라이브에서 오므로, 그 축에 드라이브가 배정되지 않은 채널에서는 상태 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

## /machine/channel/axis/axisTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축 모터 온도입니다. 반환 `float` + `unit:"°C"` (세 기종 동일). Fanuc 은 진단 308번, Mitsubishi 는 서보 드라이브 모니터의 모터 온도입니다.

**Siemens**: 값이 드라이브에서 오므로, 그 축에 드라이브가 배정되지 않은 채널에서는 상태 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

## /machine/channel/axis/axisPower
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

그 축이 **지금 쓰고 있는 전력**입니다. `channel` + `axis` 필터. 반환 `float` + `unit:"W"`.

**회생 중에는 음수**입니다. 감속하는 축은 전력을 되돌려 보내므로 부호가 뒤집힙니다.

**누적 `axisEnergy*`(Wh) 와 짝입니다.** 그쪽은 전원 투입 이후 쌓인 양이라 구간 사용량을 알려면 두 번 읽어 빼야 하는데, 이 주소는 지금 이 순간의 값을 바로 줍니다.

**Fanuc**: 진단 `4901`. **Siemens**: `$VA_POWER`(`vaPower`)이며 **PROFIdrive 드라이브에서만 값이 있습니다** (`axisLoad` 와 같은 제약). 그렇지 않은 축은 `0` 입니다. ⚠️ **Siemens 쪽은 값이 나오는 것을 보지 못했습니다.** 시험 장비(840D sl 벤치)에서 **늘 `0`** 이었습니다. 축이 1498mm/min 으로 움직이는 동안에도, 스핀들이 1425rpm 으로 도는 동안에도 `0` 이었고, 같은 순간 `axisCurrent`·`spindleLoad` 는 값이 움직였으므로 드라이브가 죽어서가 아닙니다.

**디메시의 읽기 문제가 아니라는 것은 확인했습니다.** 프로그램 운전으로 네 축이 모두 움직이고 스핀들 부하가 `spindleLoad` 로 정상 표시되는 순간, 이 값의 채널 블록과 전역 블록을 같은 요청으로 대조했는데 **양쪽 다 전 축이 `0`** 이었습니다. 남는 설명은 그 장비가 값을 주지 않는다는 것입니다 (매뉴얼이 PROFIdrive 드라이브 한정으로 적는 그대로). 다만 **값을 내는 장비는 실제로 보지 못했습니다.**

**그래서 Siemens 에서 이 주소를 감시에 쓰기 전에 값이 실제로 나오는지 확인하세요.** 제어기가 `0` 과 "값 없음" 을 구분해 주지 않아 읽기만 해서는 알 수 없습니다. 축을 움직이면서 `axisCurrent` 를 함께 읽어, 전류는 움직이는데 이 값이 계속 `0` 이면 그 기계에서는 이 주소로 전력을 감시할 수 없습니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/channel/axis/axisEnergyNet
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

축의 순소비 **전력량**(누적 소비 − 누적 회생)입니다. 반환 `float` + `unit:"Wh"`, 읽기 전용.

전력량(Wh)은 전력(W)과 다릅니다. W 는 순간값, Wh 는 누적량입니다. 이 값은 주행거리계처럼 계속 쌓이므로, 특정 구간의 사용량은 **앞뒤로 두 번 읽어 빼세요**. 평균 전력(W)이 필요하면 `ΔWh ÷ Δ시간(h)` 로 구합니다.

**Fanuc 전용**입니다 (진단 4920). Siemens 는 디메시가 읽는 노드 범위에 누적 전력량이 없어 상태 `-20` 입니다 (순시 전력은 `axisPower`). Mitsubishi 는 상태 `-20` 입니다.

## /machine/channel/axis/axisEnergyConsumed
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

축의 누적 소비 **전력량**입니다. 반환 `float` + `unit:"Wh"`, 읽기 전용.

전력량(Wh)은 전력(W)과 다릅니다. W 는 순간값, Wh 는 누적량입니다. 이 값은 주행거리계처럼 계속 쌓이므로, 특정 구간의 사용량은 **앞뒤로 두 번 읽어 빼세요**. 평균 전력(W)이 필요하면 `ΔWh ÷ Δ시간(h)` 로 구합니다.

**Fanuc 전용**입니다 (진단 4921). Siemens 는 디메시가 읽는 노드 범위에 누적 전력량이 없어 상태 `-20` 입니다 (순시 전력은 `axisPower`). Mitsubishi 는 상태 `-20` 입니다.

## /machine/channel/axis/axisEnergyRegenerated
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

축의 누적 회생 **전력량**입니다 (감속 시 돌려받은 몫). 반환 `float` + `unit:"Wh"`, 읽기 전용.

전력량(Wh)은 전력(W)과 다릅니다. W 는 순간값, Wh 는 누적량입니다. 이 값은 주행거리계처럼 계속 쌓이므로, 특정 구간의 사용량은 **앞뒤로 두 번 읽어 빼세요**. 평균 전력(W)이 필요하면 `ΔWh ÷ Δ시간(h)` 로 구합니다.

**Fanuc 전용**입니다 (진단 4922). Siemens 는 디메시가 읽는 노드 범위에 누적 전력량이 없어 상태 `-20` 입니다 (순시 전력은 `axisPower`). Mitsubishi 는 상태 `-20` 입니다.

## /machine/channel/axis/axisReferencedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

축이 **기계 원점(레퍼런스)을 확립했는지**입니다. 반환 `boolean`. **읽기 전용**.

`false` 인 축은 좌표계가 아직 서 있지 않은 상태입니다. **위치 주소들(`machinePosition`·`workPosition`·`relativePosition`·`distanceToGo`)이 그럴듯한 숫자를 주더라도 무의미할 수 있습니다.** 값이 없다는 에러가 나는 것이 아니라 기준이 서지 않은 좌표가 그대로 반환되므로, 전원 투입 직후의 위치를 소비하는 쪽은 이 값을 먼저 확인하세요. 증분형 엔코더 장비는 원점복귀를 마쳐야 좌표가 성립합니다.

**절대위치 엔코더 장비는 전원을 꺼도 기준을 잃지 않습니다.** 그런 축은 전원 투입 직후부터 `true` 라, `false` 를 한 번도 보지 못할 수 있습니다. 고장이 아니라 정상입니다. 어느 방식인지 알 필요는 없습니다. **위치를 쓰기 전에 이 값을 확인한다**는 규칙 하나면 양쪽 다 옳게 동작합니다.

한 번 확립되면 축이 어디로 움직여도 `true` 로 유지됩니다. "지금 원점 위치에 있는가" 라는 순간 상태가 아니라 **좌표계 유효성**입니다. 그 순간 상태는 형제 주소 `/machine/channel/axis/axisAtReferencePositionOn` 이 따로 냅니다.

Fanuc 은 CNC→PMC 표준 신호 ZRF(`F120` 의 축별 비트)를, Siemens 는 `refPtStatus` 를 읽습니다 (둘 다 테스트 환경에서 확인). **Mitsubishi 는 상태 `-20` 입니다**: 디메시가 이 제어기에서 읽는 것은 "지금 원점 위치에 있는가"(조작반의 `#1` 표시·PLC `ZP1n`·`GetAxisStatus`)와 절대위치 검출계의 원점 초기 설정 완료(`ZSF`)이고, 한 번 확립되면 유지되는 좌표계 확립 상태는 그 둘로 만들 수 없습니다 (테스트 환경에서 원점 복귀 뒤 축을 움직이자 그 비트가 바로 꺼지는 것을 확인). 그 순간 상태는 `/machine/channel/axis/axisAtReferencePositionOn` 이 세 기종 공통 뜻으로 냅니다. Fanuc 은 축 16개까지 지원하며, 다경로 장비에서는 그 경로의 신호를 읽습니다.

## /machine/channel/axis/axisAtReferencePositionOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

축이 **지금 1번 레퍼런스(원점) 위치에 서 있는지**입니다. 반환 `boolean`. **읽기 전용**.

원점 복귀가 끝나 그 자리에 서 있으면 `true`, 이동 지령으로 원점을 벗어나면 `false` 입니다. **순간 상태**라 가공 중에는 대개 `false` 이고, 그것이 정상입니다. 조작반이 기계 위치 옆에 붙이는 원점 표시(Mitsubishi 의 `#1` 등)와 같은 뜻입니다.

**`axisReferencedOn` 과 다른 질문입니다.** 그쪽은 "좌표계가 확립됐는가"(한 번 확립되면 움직여도 `true`)이고, 이쪽은 "지금 원점에 있는가"(움직이면 `false`)입니다. 전원 투입 직후 위치를 믿어도 되는지는 `axisReferencedOn` 으로, 공구 교환 위치나 기동 준비처럼 "축이 원점에 돌아왔는가" 를 기다릴 때는 이 주소로 확인합니다. 테스트 환경의 Fanuc(NC Guide)이 `ZRF=7`(3축 확립)·`ZP=0`(전부 원점 밖)을 함께 낸 것이 두 주소의 차이 그 자체입니다.

Fanuc 은 CNC→PMC 표준 신호 ZP(`F094` 의 축별 비트, "reference position return completion": 원점에 있을 때 `1`, 벗어나면 `0`)를, Mitsubishi 는 `GetAxisStatus` 의 1번 레퍼런스 위치 복귀 완료 비트(PLC `ZP1n`·조작반 `#1` 과 같은 값)를 읽습니다 (둘 다 테스트 환경에서 확인). **Siemens 는 상태 `-20`** 입니다: 디메시가 읽는 NC 변수 중에 "원점 위치에 있음" 을 뜻하는 것이 없습니다(`refPtStatus` 는 확립 상태, `refPtBusy`·`refPtCamNo`·`refPtPhase` 는 복귀 동작 중의 진행 정보). Fanuc 은 축 16개까지 지원하고 다경로 장비에서는 그 경로의 신호를 읽으며, 2번 이후의 레퍼런스 위치(Fanuc `ZP2`~, Mitsubishi `ZP2n`~)는 이 주소가 다루지 않습니다.

## /machine/channel/axis/axisInterlockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

축 인터록 상태입니다 (`true` = 인터록 걸림). 반환 `boolean`, 읽기 전용. 인터록은 래더가 그 축의 이동을 막아 둔 상태라, `true` 인 동안 그 축은 지령이 있어도 움직이지 않습니다.

**Fanuc 전용**입니다. 축별 데이터(`cnc_rdaxisdata`)의 상태 플래그 bit 2(Interlock state)를 읽습니다. Siemens·Mitsubishi 는 상태 `-20` 입니다. 두 기종에서는 이 상태를 축 데이터의 한 비트로 내주는 통로를 디메시가 쓰지 않으며, 인터록은 기계 제작사 래더가 정하는 PLC 신호라 필요하면 그 장비의 신호를 `plcAddress` 로 읽으세요.

## /machine/channel/axis/axisSoftLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 축의 **소프트 리미트(저장형 스트로크 체크 1) 양의 방향 좌표 가운데 지금 효력이 있는 값**입니다. 기계좌표 기준이고 읽기 전용입니다.

"소프트" 는 리밋 스위치 같은 물리 장치가 아니라 **제어기가 좌표로 판정하는 한계**라는 뜻입니다. 지령이 이 값을 넘으면 제어기가 스트로크 초과 알람으로 막습니다.

**소프트 리미트는 영역이 여러 벌일 수 있고, 어느 벌이 효력을 갖는지는 PLC 가 실시간으로 고릅니다.** 이 주소는 그 선택을 따라가 **지금 이 순간 이 방향을 막는 값**을 냅니다. 영역별 설정값을 읽고 쓰려면 `axisSoftLimitArea/axisSoftLimitPositive` 에 `axisSoftLimitArea` 필터로 영역을 지목하고, 영역 수는 `axisSoftLimitAreaCount`, 지금 선택된 영역 번호는 `axisSoftLimitPositiveAreaNumber` 로 읽습니다. 이 주소에 쓰기가 없는 이유가 그것입니다. PLC 가 선택을 바꾸는 순간 "지금 값" 에 쓴 것이 어느 영역에 들어갔는지 갈리므로, 설정은 영역을 명시하는 폴더 주소로만 씁니다.

**거리라서 `unit` 을 붙이지 않습니다.** mm 인지 inch 인지는 기계 설정이므로 `/machine/channel/gModalCategory/gModal?gmodalcategory=4` 로 확인하세요 (`G21`/`G71`/`G710`=mm, `G20`/`G70`/`G700`=inch). `machinePosition` 등 다른 거리 값과 같은 처리입니다.

**Fanuc**: 영역 I 이 파라미터 `1320`, II 가 `1326`, III~VIII 이 `1350`~`1360`(짝수) 입니다. 파라미터 `1301#0`(`DLM`)이 `1` 이면 축·방향별 신호 `+EXLx` 가 I/II 를 고르고, 아니면 `1300#2`(`LMS`)가 `1` 일 때 `EXLM`·`EXLM2`·`EXLM3` 세 신호의 조합이 I~VIII 를 고릅니다. 둘 다 `0` 이면 언제나 I 입니다. 이 주소는 그 규칙대로 고른 영역의 값을 냅니다. 자릿수는 장비가 알려주는 값을 그대로 쓰므로 `/machine/channel/parameter/index/parameterValue` 로 같은 번호를 읽은 값과 항상 일치합니다. **직경 지령 축(선반 X 등)은 직경값**입니다 (매뉴얼 명시). 검사는 원점 복귀 뒤부터이고, 전원 투입 직후부터 검사하도록 설정한 장비(`1311#0`=`1`)에서만 `1300#6` 이 원점 복귀 전 검사 여부를 정합니다.

**Siemens**: 1차 소프트웨어 리밋 스위치 `$MA_POS_LIMIT_PLUS`(MD 36110)와 2차 `$MA_POS_LIMIT_PLUS2`(MD 36130) 가운데 축 인터페이스 신호 `DBX12.3` 이 고른 쪽입니다 (`1` 이면 2차). 기계축 좌표계 값이고 원점 복귀 뒤부터 모든 모드에서 유효합니다 (`PRESET` 뒤에는 다시 원점 복귀할 때까지 꺼지고, 모듈로 회전축은 감시하지 않습니다. 기능 매뉴얼 A3). 위반하면 알람 `10720`(블록 준비 단계)·`10620`(실행 중)·`10621`(JOG 로 스위치에 닿음)이 뜹니다 (진단 매뉴얼). 제어기가 채널 축과 기계축의 대응을 알려주지 않는 드문 구성에서는 상태 `-20` 입니다.

**Mitsubishi**: 파라미터 `#2014` (`OT+`) 입니다. 영역이 한 벌뿐이라 설정값과 같은 값이고, `/machine/channel/parameter/index/parameterValue?parameter=2014` 로 읽은 값과 항상 일치합니다. `#2013` 과 `#2014` 가 0 이 아닌 같은 값이면 제어기는 이 리미트를 무효로 봅니다 (매뉴얼 명시). 실사용 범위를 더 좁히는 셋업 레벨 울타리는 `axisWorkAreaLimitPositive` 입니다.

**켜고 끄는 스위치는 없습니다.** 그래서 켜짐을 묻는 형제 주소도 없고, 유효 여부는 아래 "두 값이 같거나 뒤집혀 있을 때" 규칙처럼 값의 모양이 정합니다. 원점 복귀 후 항상 적용되는 설치 레벨 울타리입니다 (작업 영역 제한의 `axisWorkAreaLimitPositiveOn` 같은 것이 여기엔 없습니다). 기종별로 대신 있는 것이 위의 영역 선택입니다.

**두 값이 같거나 뒤집혀 있을 때의 뜻은 기종마다 다릅니다.** 값의 모양으로 "미설정" 을 판정하려면 기종을 가리세요. **Fanuc**: 양쪽이 같은 값이면 **전 영역이 금지**됩니다 (조작 설명서 §6.3 CAUTION 1). 양의 값이 음의 값보다 작게 뒤집혀 있으면 검사가 걸리지 않습니다. 조작 설명서 §6.3 CAUTION 2 도 영역 크기를 잘못 설정하면 스트로크 제한이 없어진다고 밝히며, NC Guide 초기값 -1/+1 에서 실제로 제한 없이 움직였습니다. **Mitsubishi**: `#2013`=`#2014`(0 이 아닌 같은 값)이면 **무효**입니다. 뒤집힌 경우의 동작은 확인하지 못했습니다. **Siemens**: 두 값의 관계에 따른 규칙이 없습니다. 기본값이 ±1.0e8 이라 사실상 무제한이고 각 방향이 독립입니다. 같은 값이 Fanuc 에선 "잠김", Mitsubishi 에선 "무효" 라는 정반대의 뜻이므로 기종 무관 규칙 하나로 읽지 마세요.

## /machine/channel/axis/axisSoftLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 축의 **소프트 리미트(저장형 스트로크 체크 1) 음의 방향 좌표 가운데 지금 효력이 있는 값**입니다. 기계좌표 기준이고 읽기 전용입니다.

"소프트" 는 리밋 스위치 같은 물리 장치가 아니라 **제어기가 좌표로 판정하는 한계**라는 뜻입니다. 지령이 이 값 아래로 내려가면 제어기가 스트로크 초과 알람으로 막습니다.

**소프트 리미트는 영역이 여러 벌일 수 있고, 어느 벌이 효력을 갖는지는 PLC 가 실시간으로 고릅니다.** 이 주소는 그 선택을 따라가 **지금 이 순간 이 방향을 막는 값**을 냅니다. 영역별 설정값을 읽고 쓰려면 `axisSoftLimitArea/axisSoftLimitNegative` 에 `axisSoftLimitArea` 필터로 영역을 지목하고, 영역 수는 `axisSoftLimitAreaCount`, 지금 선택된 영역 번호는 `axisSoftLimitNegativeAreaNumber` 로 읽습니다. 이 주소에 쓰기가 없는 이유가 그것입니다. PLC 가 선택을 바꾸는 순간 "지금 값" 에 쓴 것이 어느 영역에 들어갔는지 갈리므로, 설정은 영역을 명시하는 폴더 주소로만 씁니다.

**거리라서 `unit` 을 붙이지 않습니다.** mm 인지 inch 인지는 기계 설정이므로 `/machine/channel/gModalCategory/gModal?gmodalcategory=4` 로 확인하세요 (`G21`/`G71`/`G710`=mm, `G20`/`G70`/`G700`=inch). `machinePosition` 등 다른 거리 값과 같은 처리입니다.

**Fanuc**: 영역 I 이 파라미터 `1321`, II 가 `1327`, III~VIII 이 `1351`~`1361`(홀수) 입니다. 파라미터 `1301#0`(`DLM`)이 `1` 이면 축·방향별 신호 `-EXLx` 가 I/II 를 고르고, 아니면 `1300#2`(`LMS`)가 `1` 일 때 `EXLM`·`EXLM2`·`EXLM3` 세 신호의 조합이 I~VIII 를 고릅니다. 둘 다 `0` 이면 언제나 I 입니다. 이 주소는 그 규칙대로 고른 영역의 값을 냅니다. 자릿수는 장비가 알려주는 값을 그대로 쓰므로 `/machine/channel/parameter/index/parameterValue` 로 같은 번호를 읽은 값과 항상 일치합니다. **직경 지령 축(선반 X 등)은 직경값**입니다 (매뉴얼 명시). 검사는 원점 복귀 뒤부터이고, 전원 투입 직후부터 검사하도록 설정한 장비(`1311#0`=`1`)에서만 `1300#6` 이 원점 복귀 전 검사 여부를 정합니다.

**Siemens**: 1차 소프트웨어 리밋 스위치 `$MA_POS_LIMIT_MINUS`(MD 36100)와 2차 `$MA_POS_LIMIT_MINUS2`(MD 36120) 가운데 축 인터페이스 신호 `DBX12.2` 가 고른 쪽입니다 (`1` 이면 2차). 기계축 좌표계 값이고 원점 복귀 뒤부터 모든 모드에서 유효합니다 (`PRESET` 뒤에는 다시 원점 복귀할 때까지 꺼지고, 모듈로 회전축은 감시하지 않습니다. 기능 매뉴얼 A3). 위반하면 알람 `10720`(블록 준비 단계)·`10620`(실행 중)·`10621`(JOG 로 스위치에 닿음)이 뜹니다 (진단 매뉴얼). 제어기가 채널 축과 기계축의 대응을 알려주지 않는 드문 구성에서는 상태 `-20` 입니다.

**Mitsubishi**: 파라미터 `#2013` (`OT-`) 입니다. 영역이 한 벌뿐이라 설정값과 같은 값이고, `/machine/channel/parameter/index/parameterValue?parameter=2013` 로 읽은 값과 항상 일치합니다. `#2013` 과 `#2014` 가 0 이 아닌 같은 값이면 제어기는 이 리미트를 무효로 봅니다 (매뉴얼 명시). 실사용 범위를 더 좁히는 셋업 레벨 울타리는 `axisWorkAreaLimitNegative` 입니다.

**켜고 끄는 스위치는 없습니다.** 그래서 켜짐을 묻는 형제 주소도 없고, 유효 여부는 아래 "두 값이 같거나 뒤집혀 있을 때" 규칙처럼 값의 모양이 정합니다. 원점 복귀 후 항상 적용되는 설치 레벨 울타리입니다 (작업 영역 제한의 `axisWorkAreaLimitNegativeOn` 같은 것이 여기엔 없습니다). 기종별로 대신 있는 것이 위의 영역 선택입니다.

**두 값이 같거나 뒤집혀 있을 때의 뜻은 기종마다 다릅니다.** 값의 모양으로 "미설정" 을 판정하려면 기종을 가리세요. **Fanuc**: 양쪽이 같은 값이면 **전 영역이 금지**됩니다 (조작 설명서 §6.3 CAUTION 1). 양의 값이 음의 값보다 작게 뒤집혀 있으면 검사가 걸리지 않습니다. 조작 설명서 §6.3 CAUTION 2 도 영역 크기를 잘못 설정하면 스트로크 제한이 없어진다고 밝히며, NC Guide 초기값 -1/+1 에서 실제로 제한 없이 움직였습니다. **Mitsubishi**: `#2013`=`#2014`(0 이 아닌 같은 값)이면 **무효**입니다. 뒤집힌 경우의 동작은 확인하지 못했습니다. **Siemens**: 두 값의 관계에 따른 규칙이 없습니다. 기본값이 ±1.0e8 이라 사실상 무제한이고 각 방향이 독립입니다. 같은 값이 Fanuc 에선 "잠김", Mitsubishi 에선 "무효" 라는 정반대의 뜻이므로 기종 무관 규칙 하나로 읽지 마세요.

## /machine/channel/axis/axisSoftLimitAreaCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

이 제어기가 소프트 리미트(저장형 스트로크 체크 1)에 대해 가진 **영역(값 한 벌)의 수**입니다. `axisSoftLimitArea/axisSoftLimitPositive`·`…Negative` 의 `axisSoftLimitArea` 필터가 받는 상한이며, 그 폴더를 `1` 부터 이 값까지 순회하면 모든 영역의 설정값을 읽을 수 있습니다. 값은 축과 무관하지만 `axis` 필터는 다른 축 주소처럼 범위를 검증합니다.

- **Fanuc**: 영역 III 의 파라미터(`1350`)가 파라미터 표에 있으면 `8`, 없으면 `2` 입니다. 연결할 때 한 번 확인합니다. Series 30i·0i-F 계열은 영역 확장 옵션과 무관하게 `1350`~`1361` 이 표에 있어 `8` 이 나오는데, III~VIII 를 실제로 고를 수 있는지는 그 옵션에 달려 있습니다 (옵션이 없으면 제어기가 `EXLM2`·`EXLM3` 신호를 보지 않습니다). 옵션 유무를 FOCAS2 로 확인할 방법을 찾지 못해 이 값은 표의 크기입니다.
- **Siemens**: 언제나 `2` (1차·2차 소프트웨어 리밋 스위치).
- **Mitsubishi**: 언제나 `1` (`#2013`/`#2014` 한 벌).

## /machine/channel/axis/axisSoftLimitPositiveAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 축의 **양의 방향 소프트 리미트에 지금 효력이 있는 영역의 번호**입니다 (`1` 부터 `axisSoftLimitAreaCount` 까지). `axisSoftLimitPositive` 가 어느 영역의 값을 내고 있는지를 말하며, 설정을 바꾸려면 이 번호를 `axisSoftLimitArea/axisSoftLimitPositive` 의 `axisSoftLimitArea` 필터에 넣습니다. 방향별로 따로 있는 이유는 Fanuc 과 Siemens 모두 방향마다 다른 영역을 고를 수 있어서입니다.

- **Fanuc**: PLC 선택 신호에서 계산합니다. 파라미터 `1301#0`(`DLM`)이 `1` 이면 `+EXLx`(`Gn104` 의 축 비트)가 `0`=I, `1`=II 를 고르고, 아니면 `1300#2`(`LMS`)가 `1` 일 때 `EXLM3`·`EXLM2`·`EXLM`(`Gn531.7`·`Gn531.6`·`Gn007.6`)의 세 비트 값에 `1` 을 더한 것(`000`=1 … `111`=8)입니다. 둘 다 `0` 이면 `1` 입니다. 영역 확장 옵션이 없는 장비에서는 제어기가 `EXLM2`·`EXLM3` 를 보지 않는데 이 값은 신호 그대로 계산하므로, 그런 장비의 PLC 가 그 두 신호를 세워 두었다면 제어기가 실제로 쓰는 영역과 다를 수 있습니다.
- **Siemens**: 축 인터페이스 신호 `DBX12.3`(2차 소프트웨어 리밋 스위치 플러스)이 `1` 이면 `2`, 아니면 `1` 입니다.
- **Mitsubishi**: 영역이 하나라 언제나 `1` 입니다.

**쓰기는 없습니다. 어느 영역을 쓸지는 기계가 정합니다.** 영역이 여러 벌인 이유가 심압대 위치나 어태치먼트 같은 기계 상황에 따라 유효 스트로크를 바꾸려는 것이고, 그 판단은 기계 제작사의 래더가 PLC 신호로 매 스캔 내립니다. 이 값은 그 결과를 보고하는 관측값이라 `executionStatus` 와 같은 부류입니다. 바깥에서 그 신호를 쓰면 래더가 다음 스캔에 덮어 아무것도 안 바뀌거나, 래더가 안 쓰는 장비에서는 기계 상황과 무관하게 보호 영역이 바뀝니다. 전환이 필요하면 래더나 조작반 스위치로 하세요. 영역의 설정값 자체는 `axisSoftLimitArea/axisSoftLimitPositive` 로 씁니다.

## /machine/channel/axis/axisSoftLimitNegativeAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 축의 **음의 방향 소프트 리미트에 지금 효력이 있는 영역의 번호**입니다 (`1` 부터 `axisSoftLimitAreaCount` 까지). `axisSoftLimitNegative` 가 어느 영역의 값을 내고 있는지를 말하며, 설정을 바꾸려면 이 번호를 `axisSoftLimitArea/axisSoftLimitNegative` 의 `axisSoftLimitArea` 필터에 넣습니다. 방향별로 따로 있는 이유는 Fanuc 과 Siemens 모두 방향마다 다른 영역을 고를 수 있어서입니다.

- **Fanuc**: PLC 선택 신호에서 계산합니다. 파라미터 `1301#0`(`DLM`)이 `1` 이면 `-EXLx`(`Gn105` 의 축 비트)가 `0`=I, `1`=II 를 고르고, 아니면 `1300#2`(`LMS`)가 `1` 일 때 `EXLM3`·`EXLM2`·`EXLM`(`Gn531.7`·`Gn531.6`·`Gn007.6`)의 세 비트 값에 `1` 을 더한 것(`000`=1 … `111`=8)입니다. 둘 다 `0` 이면 `1` 입니다. 영역 확장 옵션이 없는 장비에서는 제어기가 `EXLM2`·`EXLM3` 를 보지 않는데 이 값은 신호 그대로 계산하므로, 그런 장비의 PLC 가 그 두 신호를 세워 두었다면 제어기가 실제로 쓰는 영역과 다를 수 있습니다.
- **Siemens**: 축 인터페이스 신호 `DBX12.2`(2차 소프트웨어 리밋 스위치 마이너스)가 `1` 이면 `2`, 아니면 `1` 입니다.
- **Mitsubishi**: 영역이 하나라 언제나 `1` 입니다.

**쓰기는 없습니다. 어느 영역을 쓸지는 기계가 정합니다.** 영역이 여러 벌인 이유가 심압대 위치나 어태치먼트 같은 기계 상황에 따라 유효 스트로크를 바꾸려는 것이고, 그 판단은 기계 제작사의 래더가 PLC 신호로 매 스캔 내립니다. 이 값은 그 결과를 보고하는 관측값이라 `executionStatus` 와 같은 부류입니다. 바깥에서 그 신호를 쓰면 래더가 다음 스캔에 덮어 아무것도 안 바뀌거나, 래더가 안 쓰는 장비에서는 기계 상황과 무관하게 보호 영역이 바뀝니다. 전환이 필요하면 래더나 조작반 스위치로 하세요. 영역의 설정값 자체는 `axisSoftLimitArea/axisSoftLimitNegative` 로 씁니다.

## /machine/channel/axis/axisSoftLimitArea/axisSoftLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis", "axisSoftLimitArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

`axisSoftLimitArea` 필터로 지목한 **영역의 소프트 리미트(저장형 스트로크 체크 1) 양의 방향 설정값**입니다. 기계좌표 기준이고 읽기·쓰기 모두 됩니다. 영역 번호는 `1` 부터 `axisSoftLimitAreaCount` 까지이며, 그 밖은 상태 `-18` 입니다.

지금 어느 영역이 효력을 갖는지는 PLC 가 고르므로 여기 읽은 값이 곧 강제되는 값은 아닙니다. 강제되는 값은 `axisSoftLimitPositive`, 선택된 영역 번호는 `axisSoftLimitPositiveAreaNumber` 로 읽습니다. 설정을 바꿀 때는 그 번호를 이 필터에 넣으면 지금 효력 있는 영역을 고치는 것이 됩니다.

**거리라서 `unit` 을 붙이지 않습니다.** mm 인지 inch 인지는 기계 설정이므로 `/machine/channel/gModalCategory/gModal?gmodalcategory=4` 로 확인하세요 (`G21`/`G71`/`G710`=mm, `G20`/`G70`/`G700`=inch).

- **Fanuc**: 영역 I `1320`, II `1326`, III~VIII `1350`·`1352`·…·`1360` 입니다. `/machine/channel/parameter/index/parameterValue` 로 같은 번호를 읽은 값과 항상 일치합니다. **직경 지령 축(선반 X 등)은 직경값**입니다 (매뉴얼 명시).
- **Siemens**: 영역 `1` 은 `$MA_POS_LIMIT_PLUS`(MD 36110), `2` 는 `$MA_POS_LIMIT_PLUS2`(MD 36130) 입니다. **쓴 값의 효력은 NEW CONF 등급**입니다. 그 축이 멈추고 축이 속한 모드 그룹의 채널이 리셋 상태일 때 활성화되며(조작반 'MD 활성화'·`NEWCONF` 명령과 같은 절차), 운전 중에 쓰면 그때까지 이전 값이 강제됩니다. 제어기가 채널 축과 기계축의 대응을 알려주지 않는 드문 구성에서는 상태 `-20` 입니다.
- **Mitsubishi**: 영역 `1` 만 있고 파라미터 `#2014`(`OT+`) 입니다.

**쓰기는 장비의 쓰기 허가 상태를 따릅니다.** Fanuc 과 Mitsubishi 는 파라미터 쓰기가 허가되지 않은 상태면 상태 `-22`(기계 상태)로 거절되고, Siemens 는 머신 데이터라 보호 레벨 등으로 거절되면 상태 `-17` 에 벤더 사유가 실립니다. 이 값을 잘못 쓰면 축이 필요한 곳까지 못 가거나 반대로 보호가 느슨해지므로, 실제 장비에서는 값을 먼저 읽어 두고 바꾸세요. 두 값이 같거나 뒤집혀 있을 때의 기종별 뜻은 `axisSoftLimitPositive` 에 적혀 있습니다.

## /machine/channel/axis/axisSoftLimitArea/axisSoftLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis", "axisSoftLimitArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

`axisSoftLimitArea` 필터로 지목한 **영역의 소프트 리미트(저장형 스트로크 체크 1) 음의 방향 설정값**입니다. 기계좌표 기준이고 읽기·쓰기 모두 됩니다. 영역 번호는 `1` 부터 `axisSoftLimitAreaCount` 까지이며, 그 밖은 상태 `-18` 입니다.

지금 어느 영역이 효력을 갖는지는 PLC 가 고르므로 여기 읽은 값이 곧 강제되는 값은 아닙니다. 강제되는 값은 `axisSoftLimitNegative`, 선택된 영역 번호는 `axisSoftLimitNegativeAreaNumber` 로 읽습니다. 설정을 바꿀 때는 그 번호를 이 필터에 넣으면 지금 효력 있는 영역을 고치는 것이 됩니다.

**거리라서 `unit` 을 붙이지 않습니다.** mm 인지 inch 인지는 기계 설정이므로 `/machine/channel/gModalCategory/gModal?gmodalcategory=4` 로 확인하세요 (`G21`/`G71`/`G710`=mm, `G20`/`G70`/`G700`=inch).

- **Fanuc**: 영역 I `1321`, II `1327`, III~VIII `1351`·`1353`·…·`1361` 입니다. `/machine/channel/parameter/index/parameterValue` 로 같은 번호를 읽은 값과 항상 일치합니다. **직경 지령 축(선반 X 등)은 직경값**입니다 (매뉴얼 명시).
- **Siemens**: 영역 `1` 은 `$MA_POS_LIMIT_MINUS`(MD 36100), `2` 는 `$MA_POS_LIMIT_MINUS2`(MD 36120) 입니다. **쓴 값의 효력은 NEW CONF 등급**입니다. 그 축이 멈추고 축이 속한 모드 그룹의 채널이 리셋 상태일 때 활성화되며(조작반 'MD 활성화'·`NEWCONF` 명령과 같은 절차), 운전 중에 쓰면 그때까지 이전 값이 강제됩니다. 제어기가 채널 축과 기계축의 대응을 알려주지 않는 드문 구성에서는 상태 `-20` 입니다.
- **Mitsubishi**: 영역 `1` 만 있고 파라미터 `#2013`(`OT-`) 입니다.

**쓰기는 장비의 쓰기 허가 상태를 따릅니다.** Fanuc 과 Mitsubishi 는 파라미터 쓰기가 허가되지 않은 상태면 상태 `-22`(기계 상태)로 거절되고, Siemens 는 머신 데이터라 보호 레벨 등으로 거절되면 상태 `-17` 에 벤더 사유가 실립니다. 이 값을 잘못 쓰면 축이 필요한 곳까지 못 가거나 반대로 보호가 느슨해지므로, 실제 장비에서는 값을 먼저 읽어 두고 바꾸세요. 두 값이 같거나 뒤집혀 있을 때의 기종별 뜻은 `axisSoftLimitNegative` 에 적혀 있습니다.

## /machine/channel/axis/axisWorkAreaLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

그 축의 **작업 영역 제한 양의 방향 좌표**입니다. `axisSoftLimitPositive`(장비 설치 때 고정되는 한계) 안쪽에 셋업마다 조정하는 두 번째 울타리로, Fanuc·Mitsubishi 는 기계좌표, Siemens 는 기본 좌표계(BCS) 기준이고 읽기·쓰기 모두 됩니다.

**Fanuc**: 저장형 스트로크 체크 2 (파라미터 `1322`) 입니다. 이 기능은 설정에 따라 "이 안에 머물러라" 도 되고 "여기 들어가지 마라"(척 배리어 등) 도 되는데, **이 주소는 "이 안에 머물러라" 뜻만 약속합니다.** 그래서 장비가 진입 금지 상자로 설정돼 있으면(파라미터 `1300#0`=`0`) 값을 내보내지 않고 상태 `-20` 으로 거절합니다. 그 값을 "이 사이면 안전" 으로 계산하면 정확히 거꾸로가 되기 때문입니다 (그때는 `axisWorkAreaLimitOn` 도 같은 상태 `-20` 입니다). **직경 지령 축(선반 X 등)은 직경값**입니다 (매뉴얼 명시). 축별 사용 여부(`1310#0`)와 모달(`G22`/`G23`)은 `axisWorkAreaLimitOn` 에 적었습니다.

**Mitsubishi**: 저장형 스트로크 리미트 II (파라미터 `#8205` `OT-CHECK-P`) 입니다. 매뉴얼의 소프트 리미트 I 항목이 실사용 범위를 좁히는 용도로 안내하는 파라미터입니다 (`#8204`/`#8205`). Fanuc 과 같은 판정 조건이 있되 **축별**이고 **극성이 Fanuc 과 반대**입니다: `#8210 OT INSIDE` 가 `0`(바깥 금지 = II, 머무름)이면 답하고, `1`(안쪽 금지 = IIB, 진입 금지 상자)이면 그 축만 상태 `-20` 입니다. 체크 사용 여부는 `axisWorkAreaLimitOn` 에 있고, `#8204`=`#8205` 이면 무효입니다. 선반 G코드 리스트 6·7 에서는 프로그램이 `G22 X_ Z_ I_ K_` 로 `#8204`/`#8205` 를 바꾸며 켜고 `G23` 으로 끌 수 있어(비모달, 선반 프로그래밍 매뉴얼 §21.2) 이 값이 프로그램 실행 중에 바뀔 수 있습니다. 머시닝센터의 `G22`/`G23` 은 다른 기능(이동 전 스트로크 체크: 프로그램이 지정한 진입 금지 상자를 이동 전에 검사해 `P452` 에러, §21.1)이라 이 주소와 무관합니다.

**Siemens**: 설정 데이터 `$SA_WORKAREA_LIMIT_PLUS` (SD 43420) 입니다. 이 기능은 언제나 머무름 뜻이라 모드 거절이 없습니다. 위반하면 알람 `10730`(블록 준비 단계)·`10630`(실행 중)·`10631`(JOG)이 뜹니다 (진단 매뉴얼). 제어기가 채널 축과 기계축의 대응을 알려주지 않는 드문 구성에서는 상태 `-20` 입니다. **쓰기는 채널이 Reset 상태일 때만 됩니다.** 설정 데이터는 Siemens 가 채널 상태에 따라 외부 변경을 거절하는 데이터라, 프로그램 실행 중·정지 중·비상정지 중에는 제어기가 거절하고 알람 `4230` "Data alteration from external not possible in current channel state" 를 띄우며 이 주소는 상태 `-22`(기계 상태)로 답합니다 (에러 문구에 이 사유를 함께 싣습니다). 진단 매뉴얼도 파트 프로그램 실행 중에는 이 데이터를 입력할 수 없다고 적으며 작업 영역 제한의 설정 데이터와 드라이런 이송을 예로 듭니다. 시험 장비에서는 비상정지로 중단된 채널에서도 거절됐습니다. 머신 데이터인 소프트 리미트에는 이 제약이 없습니다. 매뉴얼(List Manual 12/2019) 기준으로 이 설정 데이터는 **기본 좌표계(BCS)** 값이고(변환이 없는 기계에서는 기계좌표와 같습니다) 효력은 즉시, 보호 레벨은 사용자(7/7)입니다. 프로그램의 `G26`(양의 방향)/`G25`(음의 방향)으로도 바뀌며, 그렇게 바뀐 값이 리셋 뒤에 남는지는 머신 데이터 `10710`(`$MN_PROG_SD_RESET_SAVE_TAB`)에 달렸습니다. `WALIMOF` 동안은 값이 있어도 무시됩니다. 감시 기준점은 **공구 선단**이라 공구 길이가 자동으로 고려되고(반경은 머신 데이터 `21020` 을 켜야), AUTO 와 JOG 양쪽에서 감시합니다 (기능 매뉴얼 A3). 워크좌표계(WCS/SZS) 기준의 좌표계별 작업 영역 제한(`WALCS0`~`WALCS10`)은 별개 기능이라 이 주소와 무관합니다 (프로그래밍 매뉴얼 §15.3.2).

**값이 있어도 켜져 있어야 강제됩니다.** 스위치는 Fanuc·Mitsubishi 가 축 단위 `axisWorkAreaLimitOn`, Siemens 가 방향별 `axisWorkAreaLimitPositiveOn`·`axisWorkAreaLimitNegativeOn` 이고, 기종별 확인법(Fanuc `G22`/`G23` 모달, Siemens `WALIMON` + 스위치, Mitsubishi `#8202`)은 그 주소들에 적었습니다.

**거리라서 `unit` 을 붙이지 않습니다.** mm/inch 는 기계 설정입니다 (`axisSoftLimitPositive` 와 같은 처리). 쓰기는 장비의 파라미터 쓰기 허가 상태를 따르며, 막혀 있으면 상태 `-22`(기계 상태)입니다.

**두 값이 같거나 뒤집혀 있을 때의 뜻도 기종마다 다릅니다.** **Fanuc**: 양쪽이 같은 값이면 체크 2 는 **전 영역이 이동 가능**합니다 (조작 설명서 §6.3 CAUTION 1, 소프트 리미트인 체크 1 과 반대). 뒤집혀 있으면 두 점을 꼭짓점으로 하는 직육면체를 그대로 경계로 삼습니다 (CAUTION 2). **Mitsubishi**: `#8204`=`#8205`(부호·값 동일)이면 무효입니다. 뒤집혀 있으면 II(머무름)는 **전 범위 금지**, IIB(진입 금지 상자)는 두 점 사이가 금지입니다 (매뉴얼 명시). **Siemens**: 두 값의 관계에 따른 규칙이 없고 각 방향이 독립이며, 켜고 끄는 것은 방향별 스위치 주소가 맡습니다.

## /machine/channel/axis/axisWorkAreaLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

그 축의 **작업 영역 제한 음의 방향 좌표**입니다. `axisSoftLimitNegative`(장비 설치 때 고정되는 한계) 안쪽에 셋업마다 조정하는 두 번째 울타리로, Fanuc·Mitsubishi 는 기계좌표, Siemens 는 기본 좌표계(BCS) 기준이고 읽기·쓰기 모두 됩니다.

**Fanuc**: 저장형 스트로크 체크 2 (파라미터 `1323`) 입니다. 이 기능은 설정에 따라 "이 안에 머물러라" 도 되고 "여기 들어가지 마라"(척 배리어 등) 도 되는데, **이 주소는 "이 안에 머물러라" 뜻만 약속합니다.** 그래서 장비가 진입 금지 상자로 설정돼 있으면(파라미터 `1300#0`=`0`) 값을 내보내지 않고 상태 `-20` 으로 거절합니다. 그 값을 "이 사이면 안전" 으로 계산하면 정확히 거꾸로가 되기 때문입니다 (그때는 `axisWorkAreaLimitOn` 도 같은 상태 `-20` 입니다). **직경 지령 축(선반 X 등)은 직경값**입니다 (매뉴얼 명시). 축별 사용 여부(`1310#0`)와 모달(`G22`/`G23`)은 `axisWorkAreaLimitOn` 에 적었습니다.

**Mitsubishi**: 저장형 스트로크 리미트 II (파라미터 `#8204` `OT-CHECK-N`) 입니다. 매뉴얼의 소프트 리미트 I 항목이 실사용 범위를 좁히는 용도로 안내하는 파라미터입니다 (`#8204`/`#8205`). Fanuc 과 같은 판정 조건이 있되 **축별**이고 **극성이 Fanuc 과 반대**입니다: `#8210 OT INSIDE` 가 `0`(바깥 금지 = II, 머무름)이면 답하고, `1`(안쪽 금지 = IIB, 진입 금지 상자)이면 그 축만 상태 `-20` 입니다. 체크 사용 여부는 `axisWorkAreaLimitOn` 에 있고, `#8204`=`#8205` 이면 무효입니다. 선반 G코드 리스트 6·7 에서는 프로그램이 `G22 X_ Z_ I_ K_` 로 `#8204`/`#8205` 를 바꾸며 켜고 `G23` 으로 끌 수 있어(비모달, 선반 프로그래밍 매뉴얼 §21.2) 이 값이 프로그램 실행 중에 바뀔 수 있습니다. 머시닝센터의 `G22`/`G23` 은 다른 기능(이동 전 스트로크 체크: 프로그램이 지정한 진입 금지 상자를 이동 전에 검사해 `P452` 에러, §21.1)이라 이 주소와 무관합니다.

**Siemens**: 설정 데이터 `$SA_WORKAREA_LIMIT_MINUS` (SD 43430) 입니다. 이 기능은 언제나 머무름 뜻이라 모드 거절이 없습니다. 위반하면 알람 `10730`(블록 준비 단계)·`10630`(실행 중)·`10631`(JOG)이 뜹니다 (진단 매뉴얼). 제어기가 채널 축과 기계축의 대응을 알려주지 않는 드문 구성에서는 상태 `-20` 입니다. **쓰기는 채널이 Reset 상태일 때만 됩니다.** 설정 데이터는 Siemens 가 채널 상태에 따라 외부 변경을 거절하는 데이터라, 프로그램 실행 중·정지 중·비상정지 중에는 제어기가 거절하고 알람 `4230` "Data alteration from external not possible in current channel state" 를 띄우며 이 주소는 상태 `-22`(기계 상태)로 답합니다 (에러 문구에 이 사유를 함께 싣습니다). 진단 매뉴얼도 파트 프로그램 실행 중에는 이 데이터를 입력할 수 없다고 적으며 작업 영역 제한의 설정 데이터와 드라이런 이송을 예로 듭니다. 시험 장비에서는 비상정지로 중단된 채널에서도 거절됐습니다. 머신 데이터인 소프트 리미트에는 이 제약이 없습니다. 매뉴얼(List Manual 12/2019) 기준으로 이 설정 데이터는 **기본 좌표계(BCS)** 값이고(변환이 없는 기계에서는 기계좌표와 같습니다) 효력은 즉시, 보호 레벨은 사용자(7/7)입니다. 프로그램의 `G26`(양의 방향)/`G25`(음의 방향)으로도 바뀌며, 그렇게 바뀐 값이 리셋 뒤에 남는지는 머신 데이터 `10710`(`$MN_PROG_SD_RESET_SAVE_TAB`)에 달렸습니다. `WALIMOF` 동안은 값이 있어도 무시됩니다. 감시 기준점은 **공구 선단**이라 공구 길이가 자동으로 고려되고(반경은 머신 데이터 `21020` 을 켜야), AUTO 와 JOG 양쪽에서 감시합니다 (기능 매뉴얼 A3). 워크좌표계(WCS/SZS) 기준의 좌표계별 작업 영역 제한(`WALCS0`~`WALCS10`)은 별개 기능이라 이 주소와 무관합니다 (프로그래밍 매뉴얼 §15.3.2).

**값이 있어도 켜져 있어야 강제됩니다.** 스위치는 Fanuc·Mitsubishi 가 축 단위 `axisWorkAreaLimitOn`, Siemens 가 방향별 `axisWorkAreaLimitPositiveOn`·`axisWorkAreaLimitNegativeOn` 이고, 기종별 확인법(Fanuc `G22`/`G23` 모달, Siemens `WALIMON` + 스위치, Mitsubishi `#8202`)은 그 주소들에 적었습니다.

**거리라서 `unit` 을 붙이지 않습니다.** mm/inch 는 기계 설정입니다 (`axisSoftLimitNegative` 와 같은 처리). 쓰기는 장비의 파라미터 쓰기 허가 상태를 따르며, 막혀 있으면 상태 `-22`(기계 상태)입니다.

**두 값이 같거나 뒤집혀 있을 때의 뜻도 기종마다 다릅니다.** **Fanuc**: 양쪽이 같은 값이면 체크 2 는 **전 영역이 이동 가능**합니다 (조작 설명서 §6.3 CAUTION 1, 소프트 리미트인 체크 1 과 반대). 뒤집혀 있으면 두 점을 꼭짓점으로 하는 직육면체를 그대로 경계로 삼습니다 (CAUTION 2). **Mitsubishi**: `#8204`=`#8205`(부호·값 동일)이면 무효입니다. 뒤집혀 있으면 II(머무름)는 **전 범위 금지**, IIB(진입 금지 상자)는 두 점 사이가 금지입니다 (매뉴얼 명시). **Siemens**: 두 값의 관계에 따른 규칙이 없고 각 방향이 독립이며, 켜고 끄는 것은 방향별 스위치 주소가 맡습니다.

## /machine/channel/axis/axisWorkAreaLimitOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

그 축의 **작업 영역 제한 스위치(축 단위)**입니다. `true` 면 `axisWorkAreaLimitPositive`·`axisWorkAreaLimitNegative` 두 값이 유효한 설정이고, `false` 면 둘 다 무시됩니다. 읽기·쓰기 모두 됩니다.

작업 영역 제한의 스위치는 기종에 따라 단위가 다릅니다. Fanuc·Mitsubishi 는 **축마다 하나**라 이 주소로 읽고 쓰고, Siemens 는 **축 × 방향**이라 이 주소가 상태 `-20`(미지원)이며 대신 방향별 주소 `axisWorkAreaLimitPositiveOn`·`axisWorkAreaLimitNegativeOn` 을 씁니다. 한쪽이 상태 `-20` 이면 다른 쪽이 그 장비의 스위치입니다. 어느 쪽이든 다른 `…On` 주소들(`singleBlockOn` 등)과 같은 뜻의 **설정 상태**입니다. 스위치가 켜져 있다는 것이지 지금 이 순간 제한이 걸리고 있다는 뜻은 아니며, 실제로 강제되는지는 기종별로 한 가지를 더 봐야 합니다.

- **Fanuc**: 파라미터 `1310#0`(`OT2x`: `0`=사용 안 함, `1`=사용)입니다. 비트 파라미터라 쓰기는 그 바이트를 읽어 bit 0 만 바꾸고 나머지 비트(`#1` 은 체크 3 스위치)는 그대로 되씁니다. 값 주소와 같은 판정 조건을 지납니다: 장비가 진입 금지 상자로 설정돼 있으면(파라미터 `1300#0`=`0`) 그 스위치는 작업 영역 스위치가 아니므로 읽기·쓰기 모두 상태 `-20` 입니다. 체크 2 전체의 켜짐은 모달 `G22`(켬)/`G23`(끔)이며 `gModalList` 로 보이고, 전원 투입 시 어느 쪽으로 시작하는지는 파라미터 `3402#7` 이 정합니다. 체크 2 옵션이 없는 장비는 `G22` 여도 강제하지 않습니다. 두 값이 같으면 스위치가 켜져 있어도 체크 2 는 전 영역을 이동 가능으로 다루어 사실상 제한이 없습니다 (조작 설명서 §6.3 CAUTION 1). 쓰기는 장비의 파라미터 쓰기 허가 상태를 따르며, 막혀 있으면 상태 `-22`(기계 상태)입니다.
- **Mitsubishi**: 파라미터 `#8202 OT-CHECK OFF` 를 뒤집은 값입니다 (`0`=사용이 `true`). Fanuc 과 같은 게이트가 **축별**로 있습니다: `#8210 OT INSIDE` 가 `1`(안쪽 금지 = IIB, 진입 금지 상자)이면 그 축은 읽기·쓰기 모두 상태 `-20` 입니다. `#8204`=`#8205` 이면 스위치가 켜져 있어도 체크가 무효입니다.
- **Siemens**: 축 단위 스위치가 없어 상태 `-20` 입니다. 방향별 주소를 쓰세요.

## /machine/channel/axis/axisWorkAreaLimitPositiveOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 축의 **작업 영역 제한 양의 방향 스위치**입니다. `true` 면 `axisWorkAreaLimitPositive` 의 값이 유효한 설정이고, `false` 면 그 값은 무시됩니다. 읽기·쓰기 모두 됩니다.

이 주소는 다른 `…On` 주소들(`singleBlockOn` 등)과 같은 뜻의 **설정 상태**입니다. 스위치가 켜져 있다는 것이지 지금 이 순간 제한이 걸리고 있다는 뜻은 아닙니다. **실제로 강제되는지는 기종별로 한 가지를 더 봐야 합니다**:

- **Siemens**: 이 스위치(SD 43400) **그리고** 채널 모달 `WALIMON` (`/machine/channel/gModalList` 에서 `WALIMON`/`WALIMOF`). 프로그램이 `WALIMOF` 를 내리면 스위치가 `true` 여도 그동안은 걸리지 않습니다. 이 스위치는 설정 데이터라 **쓰기는 채널이 Reset 상태일 때만** 되고, 아니면 알람 `4230` 과 함께 상태 `-22`(기계 상태)입니다. 매뉴얼상 `BOOLEAN`, 효력 즉시, 사용자 보호 레벨이며 조작반 '파라미터' 영역에서 작업 영역 제한을 켜고 끄는 바로 그 값입니다.
- **Fanuc**: 스위치가 방향이 아니라 **축 단위**라 이 주소는 상태 `-20` 입니다. `axisWorkAreaLimitOn` 을 읽고 쓰세요 (파라미터 `1310#0`, 모달 `G22`/`G23` 설명도 거기에 있습니다).
- **Mitsubishi**: 역시 축 단위라 상태 `-20` 입니다. `axisWorkAreaLimitOn` 을 쓰세요 (파라미터 `#8202`, 게이트 `#8210` 설명도 거기에 있습니다).

## /machine/channel/axis/axisWorkAreaLimitNegativeOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 축의 **작업 영역 제한 음의 방향 스위치**입니다. `true` 면 `axisWorkAreaLimitNegative` 의 값이 유효한 설정이고, `false` 면 그 값은 무시됩니다. 읽기·쓰기 모두 됩니다.

이 주소는 다른 `…On` 주소들(`singleBlockOn` 등)과 같은 뜻의 **설정 상태**입니다. 스위치가 켜져 있다는 것이지 지금 이 순간 제한이 걸리고 있다는 뜻은 아닙니다. **실제로 강제되는지는 기종별로 한 가지를 더 봐야 합니다**:

- **Siemens**: 이 스위치(SD 43410) **그리고** 채널 모달 `WALIMON` (`/machine/channel/gModalList` 에서 `WALIMON`/`WALIMOF`). 프로그램이 `WALIMOF` 를 내리면 스위치가 `true` 여도 그동안은 걸리지 않습니다. 이 스위치는 설정 데이터라 **쓰기는 채널이 Reset 상태일 때만** 되고, 아니면 알람 `4230` 과 함께 상태 `-22`(기계 상태)입니다. 매뉴얼상 `BOOLEAN`, 효력 즉시, 사용자 보호 레벨이며 조작반 '파라미터' 영역에서 작업 영역 제한을 켜고 끄는 바로 그 값입니다.
- **Fanuc**: 스위치가 방향이 아니라 **축 단위**라 이 주소는 상태 `-20` 입니다. `axisWorkAreaLimitOn` 을 읽고 쓰세요 (파라미터 `1310#0`, 모달 `G22`/`G23` 설명도 거기에 있습니다).
- **Mitsubishi**: 역시 축 단위라 상태 `-20` 입니다. `axisWorkAreaLimitOn` 을 쓰세요 (파라미터 `#8202`, 게이트 `#8210` 설명도 거기에 있습니다).

## /machine/channel/spindleCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

채널의 스핀들 수입니다. 연결 시 캐싱. `spindle` 필터의 유효 범위가 `1`~이 값입니다.

**Fanuc·Siemens 는 경로마다 다릅니다.** 스핀들이 없는 경로는 `0` 이고, 그 채널의 스핀들 주소들은 상태 `-20` 으로 답합니다 (채널 공통값인 `spindleOverride`·`spindleSpeedCommanded` 도 마찬가지입니다).

**Mitsubishi 는 NC 전체의 스핀들 수입니다** (파라미터 `#1039 spinno`, 기본 공통 파라미터). 모든 채널이 같은 값을 냅니다. EZSocket 이 주는 스핀들 수는 NC 전체 값이고 스핀들 번호도 NC 전역으로 보이므로, 어느 채널에서든 `spindle=1`~이 값으로 모든 스핀들을 가리킵니다. 스핀들이 둘 이상인 다계통 장비에서는 확인하지 못했습니다.

## /machine/channel/spindle/spindleOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. **Fanuc 은 기본적으로 채널 공통값**(`G30` 신호 `SOV0`~`SOV7`, 2진 0~254%, 전부 켜진 상태는 제어기와 같이 `0`)이라 모든 스핀들에 같은 값이 나오고, 파라미터 `3713#3`(MSC)과 `#4`(EOV)가 모두 켜진 장비는 **스핀들별 값**(2번 `G376`, 3번 `G377`, 4번 `G378`; 5번 이상은 상태 `-20`)입니다 (Connection Manual B-64483EN-1 §11.11). 다경로 장비는 그 경로의 신호(경로마다 `1000` 씩 밀린 주소)를 읽습니다. Siemens·Mitsubishi 는 스핀들별 값. **Mitsubishi 는 기계 제작사 래더가 고른 방식을 따라 읽습니다** (PLC 인터페이스 매뉴얼): 스핀들별 방식 선택 신호 `SPS`(`Y188F`, 스핀들마다 `+0x60`)가 꺼져 있으면 코드 신호 `SP1`/`SP2`/`SP4`(50~120% 10% 단계)를, 켜져 있으면 레지스터 `R7008`(0~200% 1% 단위, 스핀들마다 `+50`)을 읽습니다.

## /machine/channel/spindle/spindleSpeedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들 **S 지령값**입니다. 반환 `float`. **`unit` 을 붙이지 않습니다**. 지령의 뜻이 스핀들 속도 모드에 따라 갈리기 때문입니다 (회전수 일정이면 회전수, 주속 일정이면 주속). 세 기종 모두 같습니다. 지금 어느 모드인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=8` 로 확인하세요. 응답의 `desc` 가 기종 무관 의미를 말합니다: `constant surface speed` 면 주속 일정(`G96`, Siemens 는 `G961`/`G962` 도), `constant spindle speed (rpm)` 이면 회전수 일정(`G97`, Siemens 는 `G971`/`G972`/`G973` 도)입니다. **Fanuc 은 채널 모달 S 값** (`spindle` 필터 무시: S 지령이 채널 단위 개념), Siemens 는 스핀들별 `cmdSpeed`, Mitsubishi 는 스핀들별 S 지령 모달값입니다.

이 주소의 예전 이름은 `/machine/channel/spindle/speedCommanded` 입니다. 옛 주소 그대로도 동작하지만, 이 문서는 새 이름만 안내합니다.

## /machine/channel/spindle/spindleSpeedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들별 실제 회전수입니다. 반환 `float` + `unit:"rpm"` (세 기종 동일). Fanuc 은 `cnc_acts2`, Siemens 는 스핀들별 `actSpeed`, Mitsubishi 는 스핀들 모니터의 회전수 항목입니다. 세 기종 모두 `spindle` 필터로 대상 스핀들을 지정하며, **오버라이드가 반영된 실측값**입니다. **Siemens 는 회전 방향에 따라 부호가 붙습니다** (`$AA_S` 의 부호가 회전 방향, 매뉴얼 명시). Fanuc·Mitsubishi 의 부호 규칙은 확인하지 못했습니다.

이 주소의 예전 이름은 `/machine/channel/spindle/speedActual` 입니다. 옛 주소 그대로도 동작하지만, 이 문서는 새 이름만 안내합니다.

## /machine/channel/spindle/spindleLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들 부하율입니다. 반환 `float` + `unit`. Siemens 와 Mitsubishi 는 항상 `unit:"%"` 입니다. Fanuc 은 벤더 응답이 단위를 함께 실어 주어 장비 설정에 따라 `%` 또는 `rpm` 이 오고, 벤더가 그 밖의 단위 코드를 주는 드문 경우에는 `unit` 키가 생략됩니다. 소비자는 `%` 를 가정하지 말고 `unit` 을 보세요. Mitsubishi 는 스핀들 모니터의 부하 항목입니다.

**`unit` 이 `%` 일 때는 `spindleCurrent` 와 같은 물리량입니다**. 제어기가 모터 전류를 정격으로 나눠 부하율로 내기 때문입니다. 그래서 기계가 달라도 비교되는 반면(80% 는 어디서나 80%), 절대 전류값이 필요하면 `spindleCurrent` 를 쓰세요. 환산에는 그 모터의 정격 전류가 필요한데 디메시는 그 값을 내지 않으므로 **한쪽만 지원하는 기종에서는 다른 쪽을 계산해낼 수 없습니다.** Fanuc 이 `unit:"rpm"` 을 줄 때는 부하가 아니라 회전수라 이 관계가 성립하지 않습니다.

## /machine/channel/spindle/spindleCurrent
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_opcua_siemens"]
write: []
```

스핀들 모터 전류입니다. **Siemens 전용** (드라이브 파라미터 `R0078`). 반환 `float` + `unit:"Ampere"`.

**`spindleLoad` 와 같은 물리량이며 단위만 다릅니다** (그쪽은 모터 정격 대비 `%`). Fanuc·Mitsubishi 는 상태 `-20` 이고, 같은 측정을 `spindleLoad` 로 받을 수 있습니다. 다만 Fanuc 은 장비 설정에 따라 `unit` 이 `rpm` 일 수 있으니 확인하고 쓰세요.

## /machine/channel/spindle/spindleTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들 모터 온도입니다. 반환 `float` + `unit:"°C"` (세 기종 동일). Fanuc 은 진단 403번, Siemens 는 드라이브 파라미터 `R0035`, Mitsubishi 는 스핀들 드라이브 모니터의 모터 온도입니다.

## /machine/channel/spindle/spindlePower
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

그 스핀들이 **지금 쓰고 있는 전력**입니다. `channel` + `spindle` 필터. 반환 `float` + `unit:"W"`. 규칙은 `axisPower` 와 같습니다 (회생 중 음수, 누적은 `spindleEnergy*`).

**Fanuc**: 진단 `4902`. **Siemens**: 스핀들 전용 노드가 없어 그 스핀들의 **기계축** 값을 읽습니다 (`vaPower`). 제어기가 그 스핀들을 기계축에 대응시키지 못하는 드문 구성에서는 상태 `-20` 입니다. ⚠️ **Siemens 쪽은 값이 나오는 것을 보지 못했습니다** (시험 벤치에서 스핀들이 1425rpm 으로 도는 동안에도 `0`. 자세히는 `axisPower` 에 적었습니다).

**전체 소비 전력을 내는 주소는 없습니다.** Fanuc 은 제어기가 직접 재는 값을 갖고 있지만 Siemens 에는 그런 값이 없어 우리가 축·스핀들을 더해야 하는데, 그러면 같은 주소가 한쪽은 제어기가 잰 값, 다른 쪽은 우리가 더한 값이 됩니다. 게다가 제어기가 각 값을 서로 다른 순간에 재므로 **전체와 부분의 합이 같지 않을 수 있습니다**. 합계가 필요하면 축·스핀들 값을 받아 직접 더하시고, 그것이 제어기가 재는 전체와 다를 수 있다는 것을 감안하세요.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/channel/spindle/spindleEnergyNet
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

스핀들의 순소비 **전력량**(누적 소비 − 누적 회생)입니다. 반환 `float` + `unit:"Wh"`, 읽기 전용.

전력량(Wh)은 전력(W)과 다릅니다. W 는 순간값, Wh 는 누적량입니다. 이 값은 주행거리계처럼 계속 쌓이므로, 특정 구간의 사용량은 **앞뒤로 두 번 읽어 빼세요**. 평균 전력(W)이 필요하면 `ΔWh ÷ Δ시간(h)` 로 구합니다.

**Fanuc 전용**입니다 (진단 4930). Siemens 는 디메시가 읽는 노드 범위에 누적 전력량이 없어 상태 `-20` 입니다 (순시 전력은 `spindlePower`). Mitsubishi 는 상태 `-20` 입니다.

## /machine/channel/spindle/spindleEnergyConsumed
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

스핀들의 누적 소비 **전력량**입니다. 반환 `float` + `unit:"Wh"`, 읽기 전용.

전력량(Wh)은 전력(W)과 다릅니다. W 는 순간값, Wh 는 누적량입니다. 이 값은 주행거리계처럼 계속 쌓이므로, 특정 구간의 사용량은 **앞뒤로 두 번 읽어 빼세요**. 평균 전력(W)이 필요하면 `ΔWh ÷ Δ시간(h)` 로 구합니다.

**Fanuc 전용**입니다 (진단 4931). Siemens 는 디메시가 읽는 노드 범위에 누적 전력량이 없어 상태 `-20` 입니다 (순시 전력은 `spindlePower`). Mitsubishi 는 상태 `-20` 입니다.

## /machine/channel/spindle/spindleEnergyRegenerated
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

스핀들의 누적 회생 **전력량**입니다 (감속 시 돌려받은 몫). 반환 `float` + `unit:"Wh"`, 읽기 전용.

전력량(Wh)은 전력(W)과 다릅니다. W 는 순간값, Wh 는 누적량입니다. 이 값은 주행거리계처럼 계속 쌓이므로, 특정 구간의 사용량은 **앞뒤로 두 번 읽어 빼세요**. 평균 전력(W)이 필요하면 `ΔWh ÷ Δ시간(h)` 로 구합니다.

**Fanuc 전용**입니다 (진단 4932). Siemens 는 디메시가 읽는 노드 범위에 누적 전력량이 없어 상태 `-20` 입니다 (순시 전력은 `spindlePower`). Mitsubishi 는 상태 `-20` 입니다.

## /machine/channel/workOffset/axis/workOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**워크좌표계 오프셋**: G54 등 워크좌표계의 축별 오프셋 거리입니다 (read + write). 반환 `float`, 실거리 (장비 설정 단위 mm/inch 그대로. Fanuc 내부 정수 표현은 SDK 가 소수점 배율 정규화).

`workOffset` 필터는 **현장 G코드 표기를 직접 입력**합니다 (`plcAddress` 처럼 열린 이름공간: 별도 번호 체계 없음). 대소문자 무시, 공백·별칭 불허:

- **Fanuc · Mitsubishi**: `EXT`(공통 오프셋: 전 좌표계 가산, 조작반 EXT 행), `G54`~`G59`, 확장 `G54.1P1`~`G54.1P300` (옵션에 없는 P번호는 벤더 에러). 두 기종이 **같은 표기를 받고 같은 문구로 거절**합니다
- **Siemens**: `G500`, `G54`~`G57`, `G505`~`G599`. 실제로 몇 개까지 있는지는 장비 설정에 달려 있어, 없는 지정자는 상태 `-18` 과 함께 **그 장비에서 허용되는 목록**을 알려줍니다

⚠️ **`G500` 은 Fanuc `EXT` 와 다릅니다.** `EXT` 는 어느 `G5x` 가 활성이든 **그 위에 더해지는** 공통 오프셋이지만, `G500` 은 `G54`~`G57` 과 **같은 모달 그룹의 배타적 멤버**라 그것들과 동시에 활성일 수 없습니다. `G500` 이 걸린 상태는 설정 오프셋이 꺼진 상태이고 그 자리의 값은 보통 `0` 입니다. 지금 어느 것이 활성인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=7` 로 확인하세요. Siemens 에서 `EXT` 처럼 전 좌표계에 가산되는 몫은 이 주소가 아니라 **별도의 프레임**(조작반의 `Basic reference`·`Total basic WO` 행)에 있고, 디메시는 그것들을 개별 주소로 내지 않습니다. 합쳐진 결과는 `/machine/channel/axis/totalWorkOffsetValue` 가 답합니다.

**Siemens 는 기본 오프셋과 미세 조정(`Fine`)의 합을 반환합니다.** 장비가 그 합을 적용하고 조작반도 한 오프셋의 두 칸으로 보여주므로, 이 주소는 **실제 적용되는 값**을 냅니다. 조작반의 `Coarse` 칸만 보고 비교하면 다르게 보일 수 있습니다. 미세 조정만 따로 보려면 `/machine/channel/workOffset/axis/workOffsetFineValue` 를 쓰세요. Fanuc·Mitsubishi 에는 미세 조정 개념이 없어 값이 하나이고, 그래서 세 기종에서 이 주소의 뜻이 같습니다.

⚠️ **이 값은 저장된 평행이동입니다.** 두 가지가 더 있습니다. ① 워크좌표계에는 **회전·배율·미러**가 걸릴 수 있어(`workOffsetRotation`·`workOffsetScale`·`workOffsetMirrorOn`) 걸려 있으면 좌표 변환이 이 값만으로 결정되지 않습니다. ② 실제로 걸리는 총량에는 기준 오프셋 등이 더해져 이 값과 다를 수 있습니다 (`totalWorkOffsetValue`). 부품 좌표가 필요하면 계산하지 말고 `/machine/channel/axis/workPosition` 을 읽으세요. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 라 이 값이 곧 변환입니다.

`axis` 는 축 번호(1~) 또는 축 이름. `axis=1-3`·`workOffset=G54,G55` 확장 지원합니다. Fanuc 은 같은 workOffset 의 축 확장이 FOCAS 호출 1회로 묶입니다.

쓰기는 `{"value": 25.4}` (단일 축). **Fanuc·Mitsubishi 지원**: Siemens 는 이 값에 직접 쓰면 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요하고 이 프로토콜이 그 절차를 노출하지 않아 상태 `-20` 으로 거절합니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/workOffset/axis/workOffsetFineValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

워크좌표계 오프셋의 **미세 조정(`Fine`) 부분**입니다. 필터와 타입은 `/machine/channel/workOffset/axis/workOffsetValue` 와 같습니다. **읽기 전용**.

미세 조정은 **기본 오프셋을 건드리지 않고 얹는 작은 보정**입니다. 처음 워크좌표계를 잡을 때 측정한 값이 기본 오프셋(`Coarse`)으로 들어가고, 이후 첫 가공품을 재보니 `0.02mm` 어긋났다면 그 `0.02` 를 미세 조정에 넣습니다. 원래 셋업 값이 그대로 남아 추적되고, 미세 조정을 `0` 으로 되돌리면 셋업 상태로 복귀합니다.

**적용되는 오프셋은 기본 오프셋 + 이 값**이고, 그 합은 `workOffsetValue` 가 답합니다. 기본 오프셋만 필요하면 `workOffsetValue` 에서 이 값을 빼세요. 기본 오프셋을 위한 별도 주소는 두지 않았습니다.

미세 조정을 쓰지 않는(또는 장비 설정으로 끈) 장비에서는 `0` 입니다.

**Siemens 전용**입니다. Fanuc 의 워크좌표계 오프셋은 값이 하나이고 미세 조정이라는 개념이 없습니다.

**Mitsubishi 도 상태 `-20` 입니다.**

## /machine/channel/workOffset/axis/workOffsetRotation
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

워크좌표계의 **축별 회전각**입니다. 필터는 `/machine/channel/workOffset/axis/workOffsetValue` 와 같습니다. 반환 `float`, `unit` 은 `deg`(도). **읽기 전용**.

`0` 이면 그 축에 회전이 걸려 있지 않습니다.

**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하세요. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 상태 `-20` 입니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 워크 오프셋 표는 평행이동만 저장합니다 (`workOffsetValue`).

## /machine/channel/workOffset/axis/workOffsetScale
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

워크좌표계의 **축별 배율**입니다. 필터는 위와 같습니다. 반환 `float`, 무차원이라 `unit` 이 없습니다. **읽기 전용**.

`1` 이면 배율이 없습니다 (실제 크기). `2` 면 그 축 방향으로 두 배로 가공합니다.

**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하세요. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 상태 `-20` 입니다.

**Siemens 전용**입니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 워크 오프셋 표는 평행이동만 저장합니다 (`workOffsetValue`).

## /machine/channel/workOffset/axis/workOffsetMirrorOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

워크좌표계의 **축별 미러(대칭) 여부**입니다. 필터는 위와 같습니다. 반환 `boolean`. **읽기 전용**.

`true` 면 그 축 방향이 반전됩니다. 조작반의 워크오프셋 상세 화면에서 축별 체크박스로 보이는 값입니다.
**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하세요. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 상태 `-20` 입니다.

**Siemens 전용**입니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 워크 오프셋 표는 평행이동만 저장합니다 (`workOffsetValue`).

## /machine/channel/gModalCategory/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalCategory"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

활성 G모달을 **기종 무관 표준 그룹 번호**로 조회합니다 (`plcType` 처럼 디메시가 정한 벤더 중립 번호: 벤더 원시 그룹 번호가 아닙니다). `gModalCategory` 필터 값:

⚠️ **중립인 것은 "무엇을 묻는가"(그룹 번호)까지입니다. 돌아오는 값은 그 기종의 G코드라 기종 무관 분기에는 쓸 수 없습니다.** 같은 상태를 Fanuc 은 `G21`, Siemens 는 `G710` 이라고 답합니다. `desc` 가 의미를 알려주지만 그것은 **사람이 읽는 문구**이지 분기용 계약이 아닙니다 (문구는 바뀔 수 있습니다). 이 주소는 **자기 기종의 G코드를 아는 호스트 앱**을 위한 것입니다. 기종을 가로지르는 판단이 필요하면 그 사실을 직접 답하는 주소를 쓰세요. 예컨대 이송 실효값은 `feedActual`, 주속 회전수는 `spindleSpeedActual` 입니다.

- `1` = motion: 이송 모드 (G00 급속 / G01 직선 / G02·G03 원호 …)
- `2` = plane: 가공 평면 (G17 XY / G18 ZX / G19 YZ)
- `3` = distanceMode: 절대/증분 (G90/G91) · **Fanuc 선반은 G코드 체계에 따라 갈립니다.** System A 에는 이 모달 그룹이 아예 없어(절대/증분을 `U`/`W` 어드레스로 표현) 상태 `-20` 으로 답하고, System B·C 에는 있어 값이 나옵니다. 체계는 파라미터 `3401` 의 `GSB`(#6)·`GSC`(#7)로 정해지며 디메시가 연결할 때 읽습니다
- `4` = units: 인치/미터 (G20·G70·G700 / G21·G71·G710). Siemens 는 `G70`/`G71` 이 좌표값만, `G700`/`G710` 이 이송·오프셋까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5)
- `5` = feedMode: 이송 지정 (분당/회전당/역시간)
- `6` = cutterComp: 공구경 보정 (G40 해제 / G41 좌 / G42 우)
- `7` = coordinateSystem: 워크좌표계 (G54~G59)
- `8` = spindleSpeedMode: 주속 일정(G96) / 회전수 일정(G97)

값은 그 기종의 G코드 문자열 + 핵심 조합엔 `desc` 로 기종 무관 의미 (예: Fanuc `{"value":"G21","desc":"metric"}`, Siemens `{"value":"G710","desc":"metric"}`). 벤더 원시 그룹 접근은 `gModalGroup/gModal`(하나) 또는 `gModalList`(전체)를 쓰세요. 그 그룹에 걸린 모달이 없으면(그 기종이 지원하지 않는 조합 포함) 값이 `null` 입니다. `gModalList` 의 그 자리와 같은 표현입니다. **Siemens 는 `feedMode`(`5`)와 `spindleSpeedMode`(`8`)가 같은 그룹이라 값이 항상 같습니다.** SINUMERIK 은 이송 방식(`G93`·`G94`·`G95`)과 주속 방식(`G96`·`G97`)을 한 G그룹에 넣어 활성값이 하나뿐이고, 물어본 쪽에 따라 `desc` 만 갈립니다:

```
기계가 G94 상태
  gModalCategory=5 → {"value":"G94","desc":"feed per minute"}
  gModalCategory=8 → {"value":"G94","desc":"constant spindle speed (rpm)"}
```

`G94` 는 이송 코드이지 주속 코드가 아닙니다. `desc` 가 저렇게 붙는 것은 "`G96` 계열이 아니면 주속 일정이 아니다" 는 뜻이며, **`value == "G97"` 로 회전수 일정을 판정하면 Siemens 에서는 영영 맞지 않습니다.** Fanuc·Mitsubishi 는 두 그룹이 분리돼 있어 값이 다릅니다.

**Mitsubishi 는 머시닝센터와 선반 모두 지원합니다** (여덟 그룹 모두). 그룹 번호는 두 계열의 프로그래밍 매뉴얼(머시닝센터 §3.4.2 · 선반 §3.4.3 G코드 리스트 표)로 같음을 확인했습니다. 선반은 G코드 리스트(파라미터 `#1037 cmdtyp`)에 따라 **같은 뜻이 다른 G코드로 나옵니다**: 절대/증분이 `G90`/`G91`(리스트 3·5·7) 또는 `G190`/`G191`(리스트 2·4·6), 이송이 `G94`/`G95` 또는 `G98`/`G99`. 그래서 값이 아니라 `desc` 로 뜻을 읽어야 하는 자리이고, 이 표기들은 모두 `desc` 가 붙습니다. 시스템 타입이 머시닝센터도 선반도 아닌 구성(`machineType`=`unknown`)에서만 상태 `-20` 입니다. 벤더 원시 접근(`gModalGroup/gModal` 낱개, `gModalList` 전체)은 기종과 무관하게 동작합니다 (원시 번호라 의미를 약속하지 않으므로).

이 주소의 예전 이름은 `/machine/channel/gGroup/gModal`(필터 `gGroup`)입니다. 옛 주소·옛 필터 그대로도 동작하지만, 이 문서는 새 이름만 안내합니다.

## /machine/channel/gModalGroup/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalGroup"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

`gModalList` 의 **한 자리를 그룹 번호로 집어** 읽습니다 (`string`). `gModalGroup` 필터는 **그 기종의 통신 규격이 매기는 그룹 번호**를 그대로 받으며, 번호 체계는 `gModalList` 의 배열 위치와 같습니다:

- **Fanuc**: FOCAS 가 매기는 번호 `0`~`36` (`cnc_rdgcode` 의 `type`)
- **Siemens**: G-펑션 그룹 `1`~`N` (N = 장비의 그룹 수. 실측 840D sl 은 `64`)
- **Mitsubishi**: 벤더 API 의 그룹 번호 `1`~`21`

⚠️ **여기 넣는 번호는 통신 규격의 번호이지 프로그래밍 매뉴얼의 `그룹` 열이 아닙니다.** 두 체계가 같은지는 기종마다 다를 수 있으므로, 매뉴얼의 그룹 번호를 그대로 넣지 말고 `gModalList` 를 한 번 읽어 자리를 확인하세요. Fanuc 은 `0`부터 시작합니다.

범위 밖 번호는 상태 `-18` 로 거절하며 허용 범위를 함께 알려줍니다. 그 그룹에 걸린 모달이 없거나 그 기종에 없는 그룹이면 값이 **`null`** 입니다. `gModalList` 의 그 자리가 `null` 인 것과 같은 표현입니다.

그룹 번호의 의미는 각 벤더 소유라 **번역하지 않으며 기종 간에 통일하지 않습니다** (`plcAddress` 와 같은 원시 통과). 기종 무관하게 그룹을 고르려면 중립 번호를 받는 `/machine/channel/gModalCategory/gModal` 을 쓰세요. 값 역시 그 기종의 G코드 문자열이라 기종을 가로지르는 분기에는 쓸 수 없습니다 (자세히는 `gModalCategory/gModal` 설명 참조).

**Mitsubishi 에서 특히 유용합니다.** 이 기종은 왕복이 그룹당 1회라 `gModalList` 가 21왕복인데 이 주소는 1왕복이고, `gModalCategory/gModal` 이 상태 `-20` 인 구성(`machineType`=`unknown`)에서도 원시 번호로는 낱개 조회가 됩니다.

## /machine/channel/gModalList
```yaml
value_type: "stringArray"
null_able: true
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

장비가 보고하는 **전체 모달 G코드 목록을 벤더 순서 그대로** 반환합니다 (`stringArray`). 장비 특화 HMI 의 모달 화면 재현용입니다. 그룹 인덱스의 의미는 각 벤더 매뉴얼 기준입니다.

⚠️ **원소가 `null` 일 수 있습니다** (세 기종 공통). 그 자리에 걸린 모달이 없거나 그 기종에 없는 그룹이라는 뜻입니다. 문자열을 기대하는 코드가 그대로 깨지므로 원소마다 확인하세요.

- **Fanuc**: FOCAS 그룹 `0`~`36` 순서로 **길이는 항상 37** 입니다. 그 기종에 없는 그룹은 `null` (실측 31i: 34개가 오고 `24`·`25`·`28` 이 빈 자리). **구형 제어기는 `21` 부터가 전부 `null`** 입니다. 30i 세대에서 추가된 그룹이라 그 기종에 존재하지 않습니다
- **Siemens**: `ncFkt` G-펑션 그룹 1~N 순서 (N = 장비의 그룹 수). 활성 G-펑션이 없는 그룹은 `null` 입니다 (실측: 64개 중 7개)
- **Mitsubishi**: 벤더 그룹 1~21 순서 (`GetGCodeCommand`). **길이는 항상 21** 이고, 그 기종에 없는 그룹(예: 15·16·21 은 M 계열 전용)이나 활성 모달이 없는 그룹은 `null` 입니다. 표기는 벤더 매뉴얼 예시대로 정수부 두 자리(`G02`·`G50.2`)라 Fanuc 과 같은 모양입니다

**배열 위치가 곧 그룹 번호입니다.** 없는 자리를 건너뛰지 않고 `null` 로 채우는 것도 그 대응을 지키기 위해서입니다. 앞으로 당겨 담으면 뒤 항목이 남의 자리에 앉습니다.

⚠️ **Mitsubishi 는 왕복이 그룹 수만큼(21회) 듭니다.** 벤더 API 는 그룹 단위 조회를 제공합니다. 모달 화면을 한 벌 그리는 용도이지 주기 폴링용이 아닙니다. 특정 그룹 하나만 필요하면 `gModalGroup/gModal`(1왕복)로 좁혀 읽으세요.

**이 가족의 세 주소 모두 값은 기종 무관이 아닙니다.** 이 목록과 `gModalGroup/gModal` 은 벤더 그룹 번호·순서 그대로이고, `gModalCategory/gModal` 도 그룹을 고르는 번호만 중립이며 값은 그 기종의 G코드입니다. **기종을 가로지르는 판단에는 셋 다 쓰지 마세요** (자세히는 `gModalCategory/gModal` 설명 참조).

길이는 카테고리 수로 고정이라 `[]` 은 나오지 않습니다. 적용된 G코드가 없는 자리가 `null` 입니다.

## /machine/channel/auxModal/auxModalValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "auxModal"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

보조 기능 모달 값입니다. `auxModal` 필터에 **레터**를 지정합니다 (예: `auxModal=M`, `S`, `T`, `D`, `H`, `F`). 반환 `float`. 예: `auxModal=T` → 지령된 공구번호, `auxModal=S` → 지령 회전수.

**한 블록에 M 을 여러 개 지령한 경우는 레터에 순번을 붙여 읽습니다**. `M` 이 첫 번째, `M2` 가 두 번째입니다 (`M8 M42 M13;` 처럼 한 줄에 거는 경우). 받는 개수는 제어기 사양이라 **Fanuc 은 `M3` 까지, Mitsubishi 는 `M4` 까지**입니다. Mitsubishi 는 `B` 에도 순번이 있어 `B4` 까지 받습니다.

**한 블록에 여러 M 이 있을 때 두 기종이 다르게 냅니다.** 같은 `M8 M5` 블록을 테스트 환경에서 확인한 결과입니다:

| | `M` | `M2` |
|---|---|---|
| Mitsubishi | `8` | `5` |
| Fanuc (시험한 제어기) | `5` | `0` |

Mitsubishi 는 **블록에 쓴 순서**대로 자리를 채웁니다 (값 순이 아닙니다. `8` 이 먼저 쓰였으므로 첫 자리). 시험한 Fanuc 제어기는 둘 중 하나만 `M` 에 싣고 순번 자리는 비어 있었습니다. 한 블록 다중 M 지령은 Fanuc 에서 제어기 옵션이라, 그 옵션이 없는 장비에서는 `M2`·`M3` 가 늘 `0` 입니다.

**그래서 "이 M 코드가 걸렸나" 를 이 주소로 이식성 있게 묻지 마세요.** 자리가 몇 개인지도, 어느 자리에 오는지도, 애초에 채워지는지도 장비에 달렸습니다. 쓰이지 않은 자리는 `0` 입니다.

**받는 레터는 기종에 따라 다릅니다.**

- **Fanuc**: `B`·`D`·`F`·`H`·`L`·`M`·`M2`·`M3`·`P`·`Q`·`R`·`S`·`T` 로 고정
- **Siemens**: 제어기에 그 레터의 모달이 있으면 받습니다 (실측에서 `E`·`A` 도 응답). 순번 형태는 받지 않습니다
- **Mitsubishi**: `B`·`B2`·`B3`·`B4`·`M`·`M2`·`M3`·`M4`·`S`·`T`. 벤더 API 가 다루는 레터는 M/S/T/B 이므로 `D`·`F`·`H` 등은 없습니다

안 받는 레터는 상태 `-18` 이고 에러 문자열이 그 기종에서 쓸 수 있는 것을 알려줍니다.

**`S` 에는 순번을 붙이지 않습니다.** Mitsubishi 의 벤더 인덱스는 스핀들 번호지만 이 주소엔 `spindle` 필터가 없어 첫 스핀들로 고정합니다. 스핀들별 지령 회전수는 `/machine/channel/spindle/spindleSpeedCommanded` 가 담당합니다.

**장비가 주는 모달 값을 그대로 냅니다. 번역하지 않습니다.** `parameter`·`diagnosis` 와 같은 범용 통로입니다. 다만 **그 레터가 지령되지 않은 상태의 표현이 기종마다 다릅니다.**

- **Siemens 는 `null` 입니다.** 제어기가 "그 레터에 할당된 것이 없음" 을 별도 상태로 보고하므로(출처 `/Channel/SelectedFunctions/`. 벤더 매뉴얼은 미할당을 음수로 낸다고 밝힙니다) 그 상태를 `null` 로 냅니다. 유휴 상태의 실측에서 `T`·`S`·`H`·`M` 이 `null`, `D` 는 `1`, `F` 는 `0` 이었습니다 (실제 값이 있으면 그 값). **실행 중 확인한 값은 블록이 아니라 모달처럼 움직였습니다**: `M3 S500 T="CUTTER 10"` 다음 블록(`G4` 드웰)에서 `S` 는 `500`, `M` 은 `3` 을 그대로 유지했습니다. 즉 `S`·`M` 은 Fanuc·Mitsubishi 와 같이 모달입니다. **다만 프로그램이 끝나면(`M30`) `null` 로 돌아갑니다.** 그 두 기종은 리셋으로도 지워지지 않으므로 여기가 갈립니다. 같은 측정에서 **`T` 는 지령했는데도 계속 `null`** 이었습니다 (그 장비는 `T` 가 교환까지 수행하는 설정이었습니다). 그러니 지령된 공구는 이 주소가 아니라 `/machine/channel/activeToolNumber` 로 보세요. `H` 는 Siemens 에서 음수 지령이 가능한데 제어기의 미할당 표시도 음수라, 음수 `H` 지령은 이 주소에서 `null` 과 구분되지 않습니다.
- **Fanuc·Mitsubishi 는 `0` 입니다.** 두 제어기는 "지령 없음" 을 따로 표현하지 않아, `0` 이 지령되지 않음과 `0` 지령(`T0` 은 실제로 쓰이는 지령입니다)을 함께 뜻합니다. 장비가 주지 않는 구분을 만들지 않으므로 `null` 로 바꾸지 않습니다.

그래서 **"지령된 게 있나" 를 `== 0` 하나로 판정하면 Siemens 에서 걸리지 않고, `== null` 하나로 판정하면 Fanuc·Mitsubishi 에서 걸리지 않습니다.** 어느 값을 "없음" 으로 볼지는 그 장비를 아는 호스트 앱이 정하세요.

## /machine/channel/partCountActual
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

지금까지 가공된 수량입니다. 작업을 바꿀 때 리셋하는 카운터입니다. `channel` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0}` 으로 리셋합니다. Linux 용 Fanuc 라이브러리는 이 쓰기가 쓰는 함수(`cnc_wrparam`)를 제공하지 않아 Linux 에서는 쓰기가 상태 `-20` 입니다.

**이 값은 "실제로 생산한 개수" 가 아닙니다.** 제어기가 프로그램 종료(`M02`/`M30`)에 반응해 올리는 카운터이고, 그 반응 여부부터 장비 설정에 달렸습니다. 설정이 꺼져 있으면 올라가지 않고, 드라이런으로 돌려도 올라가며, 같은 프로그램을 두 번 돌리면 2가 됩니다. 작업자가 조작반에서 바꿀 수도 있습니다. **양품인지 아닌지는 제어기가 알지 못합니다.** 실적 집계에 쓰려면 호스트 앱이 프로그램·시간 정보를 겹쳐 판단해야 합니다.

**Fanuc 은 쓰기 직후 다시 읽으면 이전 값이 올 수 있습니다.** 제어기가 파라미터를 반영하는 데 시간이 걸립니다 (실측: 즉시 읽으면 8회 중 6회가 이전 값, `50`ms 뒤에는 모두 정상). 쓴 값을 확인하려면 잠시 뒤에 읽으세요. 같은 Fanuc 이라도 매크로 변수 쓰기는 즉시 반영되고, 파라미터만 그렇습니다. Siemens 는 즉시 반영됩니다.

Fanuc 은 파라미터 `6711` (`M02`/`M30` 또는 파라미터 `6710` 에 정한 M 코드가 실행될 때 1 씩 늘고, `6700#0`=`1` 이면 `6710` 의 M 코드만), Siemens 는 `actParts` (`$AC_ACTUAL_PARTS`: 머신 데이터 `27880` 비트 8 이 켜져야 세고, 기본은 `M02`/`M30` 에서 1 씩, 비트 9 를 켜면 `27882` 에 정한 M 코드에서 셉니다. **목표 비교(`27880` 비트 0)가 켜져 있고 `partCountRequired` 가 `0` 보다 크면 거기 도달하는 순간 자동으로 `0` 이 됩니다.** Fanuc 이 계속 올라가며 신호만 내는 것과 다릅니다), Mitsubishi 는 파라미터 `8002` 입니다 (파라미터 `8001` 에 정한 M 코드가 실행될 때 셉니다. `8001` 이 `0` 이면 세지 않고 조작반의 가공 수 표시도 꺼집니다, M800 조작 매뉴얼). **Mitsubishi 는 읽기 전용**이라 리셋 쓰기가 상태 `-20` 입니다.

## /machine/channel/partCountRequired
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

만들어야 할 목표 수량입니다. `channel` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 100}`. `0` 이면 목표가 설정되지 않은 상태입니다. Linux 용 Fanuc 라이브러리는 이 쓰기가 쓰는 함수(`cnc_wrparam`)를 제공하지 않아 Linux 에서는 쓰기가 상태 `-20` 입니다.

가공된 수량이 이 값에 도달하면 장비가 신호를 내거나 정지하도록 설정할 수 있는데, 그 동작 여부는 장비 설정에 달렸습니다. 디메시는 값만 전달합니다.

**Fanuc 은 쓰기 직후 다시 읽으면 이전 값이 올 수 있습니다.** 제어기가 파라미터를 반영하는 데 시간이 걸립니다 (실측: 즉시 읽으면 8회 중 6회가 이전 값, `50`ms 뒤에는 모두 정상). 쓴 값을 확인하려면 잠시 뒤에 읽으세요. Siemens 는 즉시 반영됩니다.

Fanuc 은 파라미터 `6713` (`0` 이면 제어기가 목표를 무한으로 보아 도달 신호 `PRTSF` 를 내지 않습니다), Siemens 는 `reqParts` (`$AC_REQUIRED_PARTS`: 머신 데이터 `27880` 비트 0 이 켜져야 도달 비교·알람·PLC 신호가 동작하고, 그때 `0` 보다 큰 값에 도달하면 `partCountActual` 이 자동으로 `0` 이 됩니다), Mitsubishi 는 파라미터 `8003` 입니다. **Mitsubishi 는 읽기 전용**입니다.

## /machine/channel/partCountTotal
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

장비 통산 가공 수량입니다. 작업을 바꿔도 리셋하지 않는 누적값입니다. `channel` 필터. 반환 `int`, **읽기 전용**.

쓰기를 지원하지 않는 것은 의도된 제한입니다. 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다. 리셋이 필요한 카운터는 별도로 있습니다.

이 값도 제어기가 프로그램 종료에 반응해 올리는 카운터라, 드라이런이나 재실행도 함께 셉니다. **"실제로 생산한 개수" 로 쓰면 안 됩니다.**

Fanuc 은 파라미터 `6712` (`partCountActual` 과 같은 조건에서 함께 1 씩 늡니다), Siemens 는 `totalParts` (`$AC_TOTAL_PARTS`: 머신 데이터 `27880` 비트 4 가 켜져야 세고, 기본 설정에서는 `M02`/`M30` 에서 1 씩, 비트 5 를 켜면 `27882` 의 M 코드에서 셉니다) 입니다. **Mitsubishi 는 상태 `-20`** 입니다. 그 제어기에서 디메시가 읽을 수 있는 표준 값은 리셋되는 작업 카운터(`partCountActual`)뿐이고, 누적을 세는 값은 기계 제작사가 PLC 로 구현하는 현장 설정이라 SDK 가 알 수 없습니다.

## /machine/channel/programRunDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

**이번 자동운전 사이클의 실행 시간**입니다. 사이클을 새로 시작하면 `0` 부터 다시 셉니다. `channel` 필터. 반환 `int` (초) + `unit:"s"`, 읽기 전용.

누적값이 아닙니다. 장비 수명에 걸쳐 쌓이는 값(전원투입 시간·절삭 시간)과 달리 **한 번의 운전**을 잽니다. 정지 중에는 세지 않습니다. Fanuc 은 정지·홀드 시간을 빼고(파라미터 매뉴얼 명시) 전원 투입 때와 **리셋 상태에서 사이클 스타트할 때** `0` 으로 돌아갑니다 (홀드에서 재개할 때는 이어서 셉니다). Siemens 는 정지 시간과 이송 오버라이드 `0` 으로 멈춘 시간을 빼고, `M30` 도착과 리셋 상태에서의 재시작에 `0` 이 됩니다. Fanuc 의 이송 오버라이드 `0` 처리는 확인하지 못했습니다.

**초 미만은 버립니다.** 두 기종 다 밀리초 해상도를 제공하지만 경과시간 주소는 정수 초로 통일합니다. 조작반의 `CYCLE TIME` 표시와 같은 값이 나옵니다 (Fanuc 테스트 환경에서 확인). 사이클이 몇 초로 짧으면 최대 1초의 오차가 상대적으로 큽니다.

Fanuc 은 파라미터 `6758`(분) + `6757`(분 미만 ms) 를 한 번의 호출로 함께 읽고, Siemens 는 `actProgNetTime` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.** 디메시가 그 기종에서 읽는 시간 값은 통합 시간 화면의 전원 투입·자동 기동·자동 운전 시간이고, 사이클·절삭 시간은 그 안에 없습니다.

## /machine/channel/operatingDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

**자동 운전 시간의 누적값**입니다. 장비가 자동 운전을 하고 있던 시간을 계속 쌓습니다. `channel` 필터. 반환 `int` (초) + `unit:"s"`, 읽기 전용.

`programRunDuration` 과 짝을 이룹니다. 그쪽은 **이번 사이클**만 재고, 이쪽은 **장비 일생**입니다. 이름의 `program` 이 그 범위를 밝히는 자리라, 접두어가 없는 이 주소는 `powerOnDuration`·`cuttingDuration` 과 같은 누적값입니다.

**홀드·정지 중에는 세지 않습니다.** 피드홀드로 세워 둔 시간은 양 기종 모두 빠집니다 (Fanuc 은 파라미터 매뉴얼이 정지·홀드 시간을 제외한다고 명시하고 테스트 환경에서도 확인, Mitsubishi 는 홀드 포함/제외 카운터를 벤더가 나눠 주며 이 주소는 제외 쪽입니다). 프로그램을 걸어두고 자리를 비운 시간이 가동 시간으로 잡히지 않는다는 뜻입니다.

`/machine/powerOnDuration` 과 함께 읽으면 가동률의 재료가 됩니다. 켜져 있던 시간 중 실제로 운전한 시간의 비율. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

**쓰기는 지원하지 않습니다.** 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다.

Fanuc 은 파라미터 `6752`(분) + `6751`(분 미만 ms) 를 한 번의 호출로 함께 읽고(조작반 실적 화면의 `RUN TIME`), Mitsubishi 는 `GetStartTime` 입니다. 조작반 통합 시간 화면의 **`Auto strt`(자동 기동 시간: 자동 기동 버튼부터 피드홀드·블록 정지·리셋까지의 누적)** 와 같은 값이고, 홀드를 포함하는 `Auto oper`(자동 운전 시간: 기동부터 `M02`/`M30`/리셋까지, `GetRunTime`)는 쓰지 않습니다 (M800 조작 매뉴얼 §9.3.1). **제어기는 `59999:59:59` 에서 누적을 멈춥니다** (`powerOnDuration` 과 같은 상한·같은 API 문서 단서).

**Siemens 는 상태 `-20` 입니다** (디메시가 이 값을 내지 않습니다).

## /machine/channel/cuttingDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

**절삭 누적 시간**입니다. 공구가 실제로 물려 깎고 있던 시간의 누적값입니다. `channel` 필터. 반환 `int` (초) + `unit:"s"`, 읽기 전용.

전원투입 시간과 함께 층을 이룹니다. 켜져 있던 시간 중 실제로 깎은 시간이 얼마인지가 가동률 계산의 재료입니다. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

**측정이 멈추는 조건이 기종마다 다릅니다.** Fanuc 은 파라미터 매뉴얼의 정의대로 **절삭 이송(`G01`·`G02`·`G03` 등) 중의 시간만** 쌓습니다. 이송 오버라이드 `0`·드웰(휴지)·홀드 중을 어떻게 다루는지는 확인하지 못했습니다. Siemens 는 `$AC_CUTTING_TIME` 의 규칙을 따릅니다: 급속이송은 빼고 드웰(휴지) 중에는 멈추며, 기본 설정에서는 **공구가 활성일 때만** 재고 드라이런·프로그램 테스트 중에는 재지 않습니다 (머신 데이터 `27860` 의 비트 7·4·5 로 바꿀 수 있습니다). 정지 상태와 이송 오버라이드 `0` 에 대해서는 확인하지 못했습니다.

**리셋 기준점이 기종마다 다릅니다.** Fanuc 은 계속 쌓이고, Siemens 는 기본값으로 제어기를 부팅하면 `0` 이 됩니다 (일반 전원 재투입은 무관). 양쪽 모두 조작반에서 작업자가 리셋할 수 있습니다.

**Siemens 는 이 측정을 꺼둘 수 있습니다.** 머신 데이터 `27860` 의 비트 2 가 `0` 이면 측정 자체가 꺼져 있어 항상 `0` 입니다.

**쓰기는 지원하지 않습니다.** 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다.

Fanuc 은 파라미터 `6754`(분) + `6753`(분 미만 ms) 를 합산하고, Siemens 는 `cuttingTime` 입니다. 쪼개진 두 파라미터는 **한 번의 호출로 함께** 읽으므로, 읽는 도중 분이 넘어가 값이 어긋나는 일은 없습니다.

**Mitsubishi 는 상태 `-20` 입니다.** 디메시가 그 기종에서 읽는 시간 값은 통합 시간 화면의 전원 투입·자동 기동·자동 운전 시간이고, 사이클·절삭 시간은 그 안에 없습니다.

## /machine/channel/mainProgramName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

HMI 에서 **선택된 메인 프로그램**의 이름입니다. 반환 `string`, 읽기 전용. 실행 중 서브프로그램에 들어가도 변하지 않습니다 (그게 `programName` 과의 차이).

출처는 Fanuc `cnc_pdf_rdmain`, Siemens `/Channel/ProgramInfo/selectedWorkPProg`, Mitsubishi `GetProgramNumber2`(선택된 메인) 입니다.

## /machine/channel/mainProgramPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

HMI 에서 **선택된 메인 프로그램**의 전체 경로입니다 (`mainProgramName` 의 경로 버전). 실행 중 서브프로그램에 들어가도 변하지 않습니다.

**쓰기 = 프로그램 선택**: 그 경로의 프로그램을 해당 채널의 실행 대상(메인 프로그램)으로 고릅니다. 값은 경로 문자열입니다: `{"value": "//CNC_MEM/USER/O0001"}`.

- 경로 표기는 기종을 따릅니다. Fanuc `//CNC_MEM/USER/O0001`(데이터 서버는 `//DATA_SV/...`), Siemens `//NC/Part programs/PART1.MPF` (`programPath`·`entryList` 가 돌려주는 표기 그대로. `Subprograms`·`Workpieces` 도 동일), Mitsubishi `//PRG/USER/O0001` (`ncMemoryRootPath` 아래 표기 그대로). NC 파일시스템 경로는 벤더 고유라 `plcAddress` 와 같은 이유로 통일하지 않습니다
- **파일이어야 합니다.** 폴더 경로를 주면 상태 `-18`. 없는 경로도 상태 `-18`
- 선택만 할 뿐 **가공을 시작하지는 않습니다** (사이클 스타트는 조작반/PLC 몫)
- **선택이 받아들여지는 채널 상태는 제어기가 정합니다.** 그 상태가 아니라서 거절되면 상태 `-22`(기계 상태)이고, 사유에 어느 상태였는지가 실립니다. 기종별 조건:
  - Fanuc: FOCAS2 사양상 선택 함수는 MEM(자동)·EDIT 모드에서만 쓸 수 있어, MDI·JOG 같은 다른 모드에서는 어느 경로든 상태 `-22` 입니다. 자동운전이 기동된 상태(`executionStatus` 값 `3`)에서도 어느 경로든(실행 중인 프로그램 자신을 포함) 상태 `-22` 입니다. 블록 정지·피드홀드 같은 정지 상태에서 받아들일지는 장비 사정이며, 받아들여지면 선택된 프로그램이 곧바로 바뀌므로 리셋 상태에서 쓰는 것이 안전합니다. 없는 경로는 상태 `-18` 로 갈립니다
  - Siemens: 채널이 Reset 상태여야 합니다 (`Select` 메서드의 필수 조건). 그렇지 않으면 상태 `-22` 입니다
  - Mitsubishi: 프로그램 운전 중에는 운전 검색이 거절되어 상태 `-22` 입니다
- Siemens 는 서버의 파일 핸들링 `Select` 메서드를, Fanuc 은 CNC 메모리면 `cnc_pdf_slctmain`, 데이터 서버 경로면 `cnc_wrdsdncfile`, Mitsubishi 는 운전 검색 `Search` 를 씁니다

## /machine/channel/programName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

**현재 실행 중인** 프로그램의 이름(파일명)입니다. 반환 `string`, 읽기 전용. 서브프로그램에 들어가면 그 서브의 이름으로 바뀝니다 (테스트 벤치에서 서브프로그램 실행 중 그 서브의 파일명이 오고, 나오면 메인으로 돌아오는 것을 확인했습니다). HMI 에서 선택된 메인은 `mainProgramName` 참조.

출처는 Fanuc `cnc_exeprgname2`, Siemens `/Channel/ProgramInfo/workPandProgName`, Mitsubishi `GetProgramNumber2`(실행 중 프로그램) 입니다.

표기는 기종을 따릅니다. Fanuc 은 `O0003` 처럼 O 번호, Siemens 는 **확장자를 포함한 파일명**(`PART1.MPF`, 서브프로그램에 들어가면 `SUB1.SPF`)이고 경로는 붙지 않습니다 (경로가 필요하면 `programPath`). Mitsubishi 는 프로그램 번호입니다.

## /machine/channel/programPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

현재 실행 중인 프로그램의 전체 경로입니다 (예: `//CNC_MEM/USER/PATH1/O0001`). `channel` 필터. Siemens 는 NCK 내부 경로를 사용자 표기(`//NC/...`)로 변환해 돌려줍니다.

**Mitsubishi 는 폴더 부분이 실행 중인 프로그램의 것이라는 보장이 없습니다.** 디메시는 이 기종에서 디렉터리와 파일 이름을 따로 읽어 잇는데, **실행 중인 프로그램의 디렉터리를 알려 주는 값은 찾지 못했습니다**. 다만 이 기종의 NC 메모리는 **디렉터리 구성이 고정**이라(폴더를 만들 수 없습니다. `directoryExists` 참조) 메인과 서브가 다른 폴더에 놓이는 일이 드물어, 실무에서는 대개 맞습니다. 파일 이름 쪽은 언제나 실행 중인 프로그램의 것입니다.

## /machine/channel/programSequenceNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

현재 실행 중인 블록의 **시퀀스 번호 (N 번호)** 입니다. `channel` 필터. 반환 `int`.

**N 번호가 없는 블록에서는 기종별로 다릅니다** (세 기종 테스트 환경에서 확인).

| | N 없는 블록 |
|---|---|
| Fanuc · Mitsubishi | 마지막으로 실행된 N 번호가 **유지**됩니다 |
| Siemens | **항상 `0`**: 직전 N 을 유지하지 않으며 같은 파일 안에서도 마찬가지 |

`0` 을 "N 번호 없는 블록 실행 중" 으로 읽을 수 있는 것은 Siemens 쪽뿐입니다.

**유지되는 것은 운전 중까지입니다.** 프로그램이 끝나 리셋 상태가 되면 `0` 으로 돌아갑니다 (Mitsubishi 시뮬레이터에서 확인: `N400` 을 마지막으로 실행하고 `M30` 이후 `0`). 마지막 N 이 계속 남아 있다고 가정하지 마세요.

**서브프로그램에 들어가면 서브의 N 이 나옵니다** (`programName` 이 서브 이름으로 바뀌는 것과 같은 시점). 복귀하면 메인의 N 으로 돌아옵니다.

⚠️ **Fanuc 의 유지는 파일 경계를 넘습니다** (테스트 환경에서 확인). 서브에서 복귀한 직후의 N 없는 블록에서는 **서브의 마지막 N** 이, 서브 진입 직후의 N 없는 블록에서는 **메인의 N** 이 그대로 보입니다. 즉 Fanuc 에서는 이 값만으로 어느 파일의 N 인지 판단할 수 없습니다. `/machine/channel/programName` 을 함께 읽으세요. Siemens 는 값을 유지하지 않으므로 이 상황이 생기지 않습니다. Mitsubishi 는 매 조회를 **지금 실행 중인 쪽**(메인/서브)에 지정해 물으므로, 값은 항상 현재 실행 중인 프로그램의 N 입니다.

## /machine/channel/programBlockCounter
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

실행 블록 카운터입니다. `channel` 필터. 반환 `int`.

⚠️ **기종마다 세는 것이 다릅니다.** 세 값이 서로 비교되지 않으므로 **기종을 가로지르는 진척도로 쓰지 마세요** (세 기종 테스트 환경에서 확인):

| | 세는 것 | 리셋 시점 |
|---|---|---|
| Fanuc | 사이클 시작부터 실행한 블록 수 (`cnc_rdblkcount`, 서브프로그램 블록도 이어서) | Cycle Start |
| Siemens | 지금 실행 중인 **파일 안의 행 번호** (`actLineNumber`, 음수는 `0` 으로 클램프) | 파일이 바뀔 때 (서브 진입·복귀) |
| Mitsubishi | 지금 `N` 번호로부터 **몇 블록 지났는지** | `N` 번호를 만날 때마다 |

**Mitsubishi 는 `programSequenceNumber` 와 한 쌍입니다.** 이 기종은 프로그램 안의 위치를 (프로그램 이름 · `N` 번호 · 그 N 으로부터의 블록 수) 세 값으로 지정하며, 조작반의 운전 검색도 같은 세 값을 받습니다. 그래서 이 숫자만으로는 위치가 정해지지 않고 `N` 과 함께 읽어야 위치가 정해집니다 (시뮬레이터에서 확인: `N100` 구간에서 `0`→`1`→`2`→`3`, `N500` 을 만나 `0`).

Siemens 의 행 번호는 **지금 실행 중인 파일 기준**입니다. 서브프로그램에 들어가면 서브 파일의 행 번호로 바뀌고, 메인으로 복귀하면 메인 파일의 행 번호로 돌아옵니다 (테스트 환경에서 확인). 그래서 이 숫자만으로는 어느 파일의 몇 행인지 알 수 없습니다. 메인의 3행과 서브의 3행이 같은 `3` 입니다. 파일까지 특정하려면 `/machine/channel/programName` · `/machine/channel/programNestLevel` 을 함께 읽으세요. 중첩 전환 순간(1초 미만)에는 세 값의 조합이 잠시 어긋난 샘플이 나올 수 있습니다 (레벨이 이름보다 먼저 갱신됨).

## /machine/channel/programLastBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

직전에 실행된 블록의 G코드 텍스트입니다. `channel` 필터. 직전 블록이 없으면(프로그램의 첫 블록이거나 운전 중이 아닐 때) 빈 문자열입니다.

**Siemens·Mitsubishi 지원이고 Fanuc 은 상태 `-20`** 입니다. Fanuc 의 `cnc_rdexecprog` 는 선독 버퍼, 즉 **앞으로 실행할** 블록만 담아 지나간 블록이 남지 않습니다.

## /machine/channel/programCurrentBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

지금 실행 중인 블록의 **G코드 텍스트**입니다. `channel` 필터. 없으면 **빈 문자열**이며 `null` 이 아닙니다.

**"실행 중이 아닐 때" 의 답이 기종에 따라 다릅니다.** 프로그램을 걸어 두고 리셋 상태인 제어기를 테스트 환경에서 확인한 결과입니다:

| | Siemens · Mitsubishi | Fanuc |
|---|---|---|
| `programCurrentBlock` | `""` | 프로그램의 **첫 줄** |
| `programNextBlock` | 첫 블록 | 그 **다음** 줄 |

Siemens 와 Mitsubishi 는 "실행 중인 블록 없음" 을 표현할 수단이 있습니다 (Mitsubishi 는 벤더가 실행 위치를 `0`=운전 안 함으로 알려줍니다). Fanuc 의 `cnc_rdexecprog` 에는 그 표시가 없어 선독(look-ahead) 버퍼의 첫 줄이 그대로 "현재" 로 나가는데, 정지 중에 그 줄은 실제로는 **다음에 실행될 블록**입니다.

**조작반과 대조하면 바로 보입니다.** 프로그램 화면의 실행 위치 표시(강조 막대)가 리셋 상태에서는 첫 줄 **위**에 있습니다. 아직 아무 블록도 실행하지 않았다는 뜻이고, 그 상태를 그대로 옮긴 것이 빈 문자열입니다.

**그래서 "지금 무엇을 실행 중인가" 를 이 주소만으로 판단하지 마세요.** `/machine/channel/executionStatus` 를 함께 읽어 `3`(Run)일 때만 의미가 있다고 보는 것이 안전합니다.

**여러 개가 필요하면 한 요청으로 묶어 읽으세요.** `programLastBlock`·`programCurrentBlock`·`programNextBlock`·`programLookAhead` 는 함께 요청하면 **한 번의 장비 조회**로 처리되어 서로 아귀가 맞습니다. 따로 읽으면 그 사이에 블록이 넘어가 **직전·현재·다음이 연속이 아닌 조합**을 받게 됩니다 (테스트 환경에서 확인: 따로 읽어 `G04 X8.`·`G04 X7.`·`G04 X5.` 라는 한 블록을 건너뛴 조합이 나왔고, 같은 구간을 묶어 읽으면 한 번도 어긋나지 않았습니다). 빠르기도 하지만 그보다 **일관성** 때문입니다.

## /machine/channel/programNextBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

다음에 실행될 블록의 G코드 텍스트입니다. `channel` 필터. 다음 블록이 없으면(마지막 블록) 빈 문자열입니다.

`programCurrentBlock` 에 적은 **기종 차이가 이 주소에도 그대로 옵니다**. 정지 중 Fanuc 은 한 칸 밀린 줄을 냅니다.

## /machine/channel/programLookAhead
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

**현재 실행 지점 주변의 프로그램 텍스트**입니다. 현재 블록과 그 앞쪽(아직 실행하지 않은 부분)을 담은 여러 줄 문자열. 반환 `string`.

분량은 기종마다 다릅니다. Fanuc 은 선독 버퍼 전체(`cnc_rdexecprog`), Siemens 는 실행 지점 주변의 조각(`actPartProgram`), Mitsubishi 는 현재 블록부터 최대 10블록(`CurrentBlockRead`)입니다. **전체 프로그램은 이 주소로 얻을 수 없습니다.** 그건 NC 파일 시스템에서 해당 프로그램 파일을 읽어야 합니다.

줄바꿈은 기종에 상관없이 LF(`\n`) 하나로 정규화됩니다. CR 은 제거되므로 `\n` 으로 나누면 됩니다.

## /machine/channel/programNestLevel
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

프로그램 호출 중첩 단계입니다 (뜻은 `desc` 로 함께 옵니다): `0` = 프로그램 없음, `1` = 메인, `2`~ = 서브프로그램 (L1, L2, …). `channel` 필터. **Siemens·Mitsubishi 지원** (Fanuc 은 상태 `-20`).

**세는 것은 실행이 아니라 프로그램 포인터의 깊이입니다.** 프로그램이 걸려 있으면 운전 중이 아니어도 `1` 입니다 (테스트 환경에서 확인: 두 기종 모두 리셋·중단 상태에서 `1`). "지금 돌고 있나" 는 `/machine/channel/executionStatus` 가 답합니다.

Mitsubishi 는 벤더 값이 **서브프로그램을 몇 겹 파고들었는지**(메인이 `0`)라 우리 눈금과 한 칸 달라, 디메시가 맞춰서 내보냅니다. `0`(프로그램 없음)과 `1`(메인)을 가르기 위해 이 주소만 장비 조회가 한 번 더 붙습니다.

## /machine/channel/variable/variableValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "variable"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

**매크로 변수(Fanuc·Mitsubishi) / R 파라미터(Siemens)** 를 읽고/씁니다 (read + write). `variable` 필터에 변수 번호 (예: `variable=100` → Fanuc·Mitsubishi `#100`, Siemens `R100`). 반환 `float`, 쓰기는 `{"value": 3.14}`. **읽기는** 범위/콤마 확장을 지원합니다. `variable=100-105` 는 6개 값 배열. 쓰기는 항상 단일 변수입니다 (확장 문법은 상태 `-13` 으로 거절: 모든 쓰기 공통 규칙). Fanuc·Mitsubishi 의 **미설정(vacant) 매크로 변수는 `null`** 입니다. 조작반 커스텀 매크로 화면에 빈 칸(Fanuc 은 `DATA EMPTY`)으로 뜨는 그 상태이며 값 `0` 과 구분됩니다. 범위 확장에서도 그 자리만 `null` 이 됩니다 (예: `[3.14, null]`).

**쓸 수 있는 번호는 기종·옵션마다 다릅니다.** 디메시는 목록을 들지 않고 그대로 전달하므로, 그 장비에 없는 번호는 **상태 `-18`** 로 돌아옵니다 (읽기·쓰기 모두. 에러 문자열에 벤더가 밝힌 사유가 함께 실립니다). 고칠 것은 `variable` 값 하나이며, 그 장비에 실제로 있는 번호는 조작반의 변수 화면이 알려줍니다. 범위 확장에 없는 번호가 섞이면 요청 **전체가 상태 `-15`** 로 실패합니다 (부분 배열이 오지 않습니다). 미설정(vacant) 변수는 에러가 아니라 `null` 원소라 확장을 깨지 않는 것과 구분하세요. 문법 자체가 범위 밖인 번호(Fanuc 은 `0`~`89999`)도 같은 상태 `-18` 이되, 이쪽은 장비에 묻지 않고 즉시 거절합니다.

**미설정으로 되돌리는 쓰기는 지원하지 않습니다**. 값은 숫자 하나이고, 한 번 값을 넣은 변수를 다시 빈 칸으로 만들려면 조작반에서 지워야 합니다.

**Siemens 의 R 파라미터 쓰기에는 채널 상태 관문이 없습니다.** 설정 데이터 쓰기를 막는 알람 `4230`(채널 상태에서 외부 변경 불가)은 R 파라미터에 걸리지 않으며, 채널이 비상정지로 중단된 상태에서도, 자동운전 중(`executionStatus` 가 Run)에도 쓰기가 받아들여지는 것을 테스트 벤치에서 확인했습니다. 같은 시험에서 GUD(`userDataValue`)·PLC(`plcValue`)·공구 오프셋(`toolEdge/toolLengthWear` 등, 활성 공구의 활성 날 포함) 쓰기도 자동운전 중에 받아들여졌습니다.

## /machine/channel/userData/userDataValue
```yaml
value_type: "object"
null_able: false
required_filters: ["channel", "userData"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Siemens **채널 GUD(채널별 SGUD)** 사용자 변수를 읽고/씁니다 (**OPC-UA(Siemens) 전용**). `channel` 과 `userData` 두 필터가 필요하며, `channel` 이 가리키는 채널의 변수만 다룹니다. NC 전체가 공유하는 전역 변수는 이 주소로 접근하지 않습니다.

반환 타입은 `object` 입니다. GUD 는 변수마다 타입이 달라 값이 무슨 타입인지 함께 알려주는 **자기 서술적 엔벨로프** `{"type":..,"data":..}` 로 옵니다.

**userData**: `SGUD:<이름>` 또는 `SGUD:<이름>[<인덱스>]` 형식입니다. **인덱스는 장비 화면(HMI) 표기 그대로** 0부터 쓰면 됩니다 (내부 번호로 자동 변환).

- 인덱스 없음 → 스칼라 변수
- `[i]` → 1차원 배열의 원소 1개
- `[i-j]` → 1차원 배열의 i~j 범위 (양끝 포함)
- `[r,c](열수)` → 2차원 배열의 (행,열) 원소 1개. 대괄호는 화면 표기 그대로 적고, 배열의 **열 개수**만 괄호로 덧붙입니다 (장비가 열수를 알려주지 않아 함께 입력이 필요)
- 예: `?channel=1&userData=SGUD:_SC_C97[0,1](4)` (채널 1, 4열 2D 의 화면 표기 [0,1])

**type**: `BOOL`(참/거짓) · `CHAR`(문자 코드 0~255) · `INT`(정수) · `REAL`(64비트 실수) · `STRING`(문자열). 구조형 `AXIS`/`FRAME` 은 지원하지 않습니다 (그 타입으로 쓰면 상태 `-16` 으로 거절합니다).

**data**: 1개(스칼라 / `[i]` / `[r,c](열수)`)면 값 하나, `[i-j]` 범위면 JSON 배열입니다.

- 단일: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- 범위: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**쓰기**: 읽기와 **같은 object** 를 `value` 에 담습니다 (예: `{"value":{"type":"REAL","data":42.0}}`). `type` 으로 쓸 타입을 정하므로 읽기 없이 바로 씁니다. `data` 원소 개수는 범위 크기(단일이면 1)와 정확히 일치해야 합니다.

**주의**: GUD 영역이 없는 구형 NCK 에서는 상태 `-20`(미지원)으로 답합니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** GUD 는 Siemens 고유의 사용자 변수 체계이고, 두 기종의 사용자 변수(매크로 변수·공통 변수)는 `/machine/channel/variable/variableValue` 로 읽고 씁니다.

## /machine/userData/userDataValue
```yaml
value_type: "object"
null_able: false
required_filters: ["userData"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Siemens **전역 GUD(SGUD)** 사용자 변수를 읽고/씁니다 (**OPC-UA(Siemens) 전용**, NC 전체 공유 변수). 읽기와 쓰기 모두 지원합니다. GUD 는 변수마다 타입이 달라, 반환 타입은 `object` 입니다. 값이 무슨 타입인지 함께 알려주는 **타입을 함께 실은 엔벨로프** `{"type":..,"data":..}` 로 옵니다. 필터는 `userData` 하나입니다. 이 주소는 **NC 전체 공유** 변수 전용이라 채널을 지정하지 않습니다.

**userData**: `SGUD:<이름>` 또는 `SGUD:<이름>[<인덱스>]` 형식입니다. 접두사는 GUD 정의 블록 이름 (`SGUD` 만 지원합니다. `MGUD` 등 다른 블록은 지원하지 않습니다). **인덱스는 전부 장비 화면(HMI) 표기 그대로**: 화면에 보이는 번호를 그대로 입력하면 됩니다 (0부터 시작, OPC-UA 내부 번호로 자동 변환).

- 인덱스 없음 → 스칼라 변수
- `[i]` → 1차원 배열의 i번째 원소 1개 (화면에 `_ARR[3]` 으로 보이면 `[3]`)
- `[i-j]` → 1차원 배열의 i~j 범위 (양끝 포함: 필터 확장의 `1-3` 과 같은 관례)
- `[r,c](열수)` → 2차원 배열의 (행,열) 원소 1개. **대괄호는 화면 표기 그대로** 적고, 배열의 열 개수만 괄호로 덧붙입니다. 화면에 열이 `[0,3]` 까지 보이면 `(4)` (장비가 열수를 알려주지 않아 함께 입력이 필요합니다)
- 예: `SGUD:MYVAR` (스칼라), `SGUD:_SC_NCK_ROU_S[1]` (1D 의 화면 표기 [1]), `SGUD:POS[0-2]` (1D 의 [0]~[2] 3개), `SGUD:_SC_C97[0,1](4)` (4열 2D 의 화면 표기 [0,1])

**type** (엔벨로프 안). 원소의 실제 타입입니다:

- `BOOL`: 참/거짓
- `CHAR`: 문자 코드 (0~255 정수)
- `INT`: 정수
- `REAL`: 실수 (Fanuc R/매크로 변수와 같은 64비트 실수)
- `STRING`: 문자열

(구조형 GUD `AXIS` / `FRAME` 은 지원하지 않습니다. 그 타입으로 쓰면 상태 `-16` 으로 거절합니다)

**data** (엔벨로프 안). 1개(스칼라 / `[i]` / `[r,c](열수)`)면 값 하나, `[i-j]` 범위면 JSON 배열:

- 스칼라/단일: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- 범위: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**쓰기**: 읽기와 **같은 object** 를 `value` 에 담습니다 (예: `{"value":{"type":"REAL","data":42.0}}` → 화면 표기 [i] 한 칸만 변경). `type` 으로 쓸 타입을 정하므로 읽기 없이 바로 씁니다. `data` 원소 개수는 범위 크기 (단일이면 1)와 정확히 일치해야 합니다.

**주의**: GUD 영역이 없는 구형 NCK 에서는 상태 `-20`(미지원)으로 답합니다.

**Mitsubishi 도 상태 `-20` 입니다.** 그 기종의 사용자 변수(공통 변수)는 `/machine/channel/variable/variableValue` 로 읽고 씁니다.

## /machine/plcAddress/plcType/plcValue
```yaml
value_type: "float"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

**PMC/PLC 메모리의 단일 원소**를 읽고/씁니다 (Fanuc FOCAS2 `pmc_rdpmcrng`/`pmc_wrpmcrng`, Siemens OPC-UA `/Plc/` 노드, Mitsubishi EZSocket `ReadDevice`/`WriteDevice`). 읽기와 쓰기 모두 지원하며, 반환 타입은 `float` (값 하나), 쓰기도 숫자 하나입니다 (예: `{"value": 42}`). 이 주소는 **원소 하나 전용**이며, 여러 원소를 한 번에 다루려면 같은 트리의 목록형 주소를 사용합니다. `plcAddress` 와 `plcType` 두 필터가 필요합니다.

**plcAddress**: 주소 형식이 **기종별로 다릅니다**. `plcType` 과 달리 **중립화하지 않는 의도적 예외**입니다. Fanuc 의 `D100` 과 Siemens 의 `DB10.DBB56` 는 서로 다른 메모리 아키텍처를 가리키고, 둘을 잇는 대응표는 SDK 가 알 수 있는 지식이 아니라 그 장비의 래더를 어떻게 짰는지에 달린 현장 설정이기 때문입니다. 기종 간에 통일하지 않으므로, 여러 기종에서 같은 신호를 읽어야 한다면 **호스트 앱이 기종별 주소 표를 들고** 있어야 합니다.

- **Fanuc**: 첫 글자가 PMC 영역, 나머지가 바이트 번호 (예: `R5`, `D100`). `~` 로 범위 지정. 단, 이 주소(단일)는 범위가 **정확히 `plcType` 크기 1개**여야 합니다 (예: word 면 `D100~D101`)
- **Fanuc** PMC 영역 첫 글자: `G` `F` `Y` `X` `A` `R` `T` `K` `C` `D` `M` `N` `E` `Z`: 범위는 같은 영역이어야 합니다 (`D100~D101` O, `D100~R101` X)
- **Fanuc 에서는 이 주소가 비트 주소(`R38.7` 같은 `바이트.비트` 표기)를 받지 않습니다.** 바이트를 읽어 비트를 떼어 쓰세요: `plcAddress=R38&plcType=2` 의 값은 조작반 PMC 신호 화면의 `HEX` 열과 같은 바이트 값(`0xDC` 면 `220`)이고, 비트 7 은 `(220 >> 7) & 1` 입니다. 쓰기도 바이트 단위입니다. 이유: FOCAS 의 PMC 읽기/쓰기 함수에 비트 타입이 없어, 비트 쓰기를 만들면 "바이트 읽고 고쳐 되쓰기" 가 되고 그 사이 래더가 바꾼 다른 비트를 덮어쓸 수 있습니다. 신호 이름은 `바이트.비트`(예: `R0039.4`) 로 불러도 주소는 바이트로 넣으세요
- **Siemens**: 조작반의 **`NC/PLC variables` 화면에 보이는 표기 그대로** 씁니다. 값이 `/Plc/{주소}` 노드로 전달됩니다. 첨자 생략 시 `[1]` 자동 부착. 이 주소는 단일 원소만 되며, 다원소면 에러와 함께 목록형 주소를 안내합니다
- **Siemens** 형식: **주소가 오프셋을 품습니다**. `DB<n>.DBB<offset>`(바이트) · `DB<n>.DBW<offset>`(워드) · `DB<n>.DBX<byte>.<bit>`(비트) · `IW<n>` · `MB<n>` · `Q<byte>.<bit>`. 표기 예: `DB10.DBB56` · `DB31.DBX24.1` · `IW0` · `Q0.2`
- **Siemens** 첨자 `[N]` 은 **"몇 번째" 가 아니라 "몇 개"** 입니다. `DB10.DBB56[4]` 는 오프셋 56 부터 **연속 4개**(56·57·58·59)이지 "56번의 4번째" 가 아닙니다. 다른 자리를 짚으려면 첨자가 아니라 **주소를 옮깁니다** (`DB10.DBB61`)
- **Siemens** 문법 주의(기종 무관): 오프셋 없는 표기(`MB` 단독 · `DB<n>` 단독)는 문법이 아니고, 비트는 `DBB` 가 아니라 `DBX` 로 짚습니다
- **Siemens**: 어느 블록·바이트가 **그 장비에 실재하는지는 래더 구성에 달려 기계마다 다릅니다.** 위 표기 예도 형태를 보이기 위한 것이지 어느 장비에나 있는 주소가 아닙니다. 조작반의 같은 화면에서 확인하세요. 거기서 값이 보이면 여기서도 읽힙니다
- **Siemens** 828D 제약: 828D 는 **`DB9000` 이상의 고객 데이터 블록에만** 접근할 수 있습니다 (840D sl 은 제약 없음)
- **Mitsubishi**: 조작반의 PLC 화면 표기 그대로 `<디바이스><번호>` 입니다 (예: `R100`, `M50`, `Y8A0`). 점 수는 **`[N]` 첨자**로 붙이며, Siemens 와 같이 **"몇 번째" 가 아니라 "몇 개"** 입니다. `R100[4]` 는 `R100` 부터 연속 4점. 이 주소(단일)는 첨자 없이 쓰거나 `[1]` 이어야 합니다
- **Mitsubishi** 디바이스 번호의 진법이 계열마다 다릅니다. `M`·`R`·`D` 는 10진수, `X`·`Y`·`B` 는 **16진수**입니다 (조작반 표기와 같습니다)
- **Mitsubishi** 정렬: `M`·`X`·`Y` 처럼 **비트 단위로 번호가 매겨진 디바이스**는 byte·word·dword 로 읽을 때 시작 번호가 **8점 경계**에 있어야 합니다 (`Y890` O, `Y894` X). `R`·`D` 처럼 워드 단위 디바이스는 제약이 없습니다. 어긋나면 상태 `-18` 이며, 디메시가 표로 판정하지 않고 **장비에 직접 물어** 가르므로 그 장비가 받는 주소는 그대로 통과합니다

**plcType**: 원시 바이트를 어떻게 해석할지 정하는 숫자 코드입니다. **기종 무관 통일 값**이라 어느 벤더든 같은 번호를 씁니다 (어댑터가 각 벤더 코드로 번역):

- `1` = bit: 1비트 (0 / 1)
- `2` = byte: 8비트 정수 (부호없음, 0~255) · 주소 폭 1 (예 `D100`)
- `3` = word: 16비트 정수 (부호있음) · 주소 폭 2 (예 `D100~D101`)
- `4` = dword: 32비트 정수 (부호있음) · 주소 폭 4 (예 `D100~D103`)
- `5` = float32: 32비트 실수 · 주소 폭 4 (예 `D100~D103`)
- `6` = float64: 64비트 실수 · 주소 폭 8 (예 `D100~D107`)

`0` = **auto**: 소스가 타입을 결정합니다. Siemens(OPC-UA)처럼 노드가 타입을 아는 프로토콜은 그 네이티브 타입으로 읽습니다. 반면 **Fanuc** 처럼 원시 메모리를 다루는 프로토콜은 고유 타입이 없어 `0`(auto)이 오류이며 명시해야 합니다. **Fanuc(FOCAS2)** 의 PMC 읽기는 바이트 단위라 `1`(bit) 은 지원하지 않습니다. `2`(byte)~`6`(float64) 중에서 지정하세요.

**중요(Fanuc)**: `plcAddress` 범위의 바이트 수가 `plcType` 크기와 일치해야 합니다 (예: `plcType=3`(word, 2바이트)인데 `D100` 단일 주소면 실패 → `D100~D101` 로 지정). `plcType` 은 **해석 방식만** 정하며, 결과는 `float`(JSON 숫자)로 반환됩니다.

**Siemens** 는 타입이 주소 자체에 인코딩되어 있어 (`DBB`/`DBW`/`DBD` 등) `plcType=0`(auto)을 권장합니다. `1`~`6` 을 넣어도 동작은 동일합니다 (서버가 알려주는 타입으로 읽음). 쓰기는 노드를 먼저 읽어 서버 타입을 확인한 뒤 같은 타입으로 기록합니다.

**Mitsubishi** 는 `1`(bit)·`2`(byte)·`3`(word)·`4`(dword) 넷만 됩니다. `0`(auto)은 Fanuc 과 같은 이유로 안 되고(원시 메모리라 고유 타입이 없음), `5`·`6`(실수)은 이 기종의 PLC 디바이스 API 가 정수만 실어 나르기 때문입니다. 셋 다 상태 `-18` 이며 주소 자체는 정상 동작합니다. `3`(word)·`4`(dword)는 **부호 있는 정수**로 해석됩니다. 모든 비트가 1인 워드는 `65535` 가 아니라 `-1` 입니다.

**에러 코드**: 그 장비에 **실재하지 않는 주소**도 상태 `-18`(필터 값 오류)입니다. 어느 블록·바이트가 있는지는 그 장비 래더 구성에 달렸으므로, 조작반의 같은 화면에서 먼저 확인하세요. 그 기종이 못 쓰는 `plcType` 도 상태 `-18` 입니다. 규약 밖 값(`0`~`6` 이외)도 같은 상태 `-18` 이며, 두 경우 모두 대응은 같습니다(다른 `plcType` 지정). 상태 `-20` 이 아닌 이유는 **주소 자체는 그 기종에서 정상 동작**하기 때문입니다. 상태 `-20` 은 "이 주소를 이 기종에서 못 쓴다"는 뜻으로 남겨 둡니다. 에러 문자열에 허용 값이 함께 실려 옵니다.

## /machine/plcAddress/plcType/plcValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

**PMC/PLC 메모리의 원소 블록**을 배열로 읽고/씁니다. 필터·주소 형식·`plcType` 규칙은 위 `plcValue`(단일)와 동일하고, **여러 원소**를 다룬다는 점만 다릅니다. 반환 타입은 `floatArray`, 쓰기 `value` 는 숫자 배열 `[1, 2, ...]` 입니다. 단일 원소도 `[42]` 처럼 배열로 적어야 합니다.

- **Fanuc**: 범위의 바이트 수가 `plcType` 크기의 **배수**여야 하고, 원소 수 = 바이트 수 ÷ 타입 크기 (예: `D100~D107` + word = 4개 → `[v1,v2,v3,v4]`)
- **Siemens**: 다원소 첨자 허용. `[N]` 은 **개수**입니다. `DB10.DBB56[4]` 는 오프셋 56 부터 **연속 4개**를 배열로 돌려줍니다. 서버가 주는 원소들이 그대로 배열이 됩니다
- **Siemens**: 첨자를 생략하거나 `[1]` 을 줘도 **결과는 배열**입니다 (`[131.0]`). 이 주소의 반환은 `floatArray` 로 고정이라 원소가 하나여도 흔들리지 않습니다. 개수가 가변이거나 미리 모를 때 이 주소를 쓰면 파싱 코드가 분기할 필요가 없습니다
- **Mitsubishi**: `[N]` 이 개수입니다 (`R100[4]` → 4개 배열). 한 번에 읽을 수 있는 최대 점 수가 타입마다 다릅니다: bit·byte `1280`, word `640`, dword `320`. 넘기면 상태 `-18`
- **Mitsubishi**: `plcType=2`(byte)는 **한 점만 쓸 수 없습니다** (상태 `-18`). 이 기종의 단일 디바이스 쓰기 호출에 byte 타입이 없고, 블록 쓰기는 2점 이상이라 옆 디바이스까지 함께 바뀌기 때문입니다. `3`(word)을 쓰거나 이 목록형 주소로 2점 이상을 지정하세요. **읽기는 한 점도 됩니다**
- 쓰기는 **원소 수가 대상 범위/노드의 원소 수와 정확히 일치**해야 합니다

**에러 코드**: 그 장비에 **실재하지 않는 주소**도 상태 `-18`(필터 값 오류)입니다. 어느 블록·바이트가 있는지는 그 장비 래더 구성에 달렸으므로, 조작반의 같은 화면에서 먼저 확인하세요. 그 기종이 못 쓰는 `plcType` 도 상태 `-18` 입니다. 규약 밖 값(`0`~`6` 이외)도 같은 상태 `-18` 이며, 두 경우 모두 대응은 같습니다(다른 `plcType` 지정). 상태 `-20` 이 아닌 이유는 **주소 자체는 그 기종에서 정상 동작**하기 때문입니다. 상태 `-20` 은 "이 주소를 이 기종에서 못 쓴다"는 뜻으로 남겨 둡니다. 에러 문자열에 허용 값이 함께 실려 옵니다.

길이는 주소의 `[N]` 이 정하므로 `[]` 은 나오지 않습니다.

## /machine/channel/parameter/index/parameterValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "parameter", "index"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

CNC 파라미터 **한 행의 값**입니다 (**Fanuc·Mitsubishi**, `float`, 읽기/쓰기). `channel=<채널>&parameter=<번호>&index=<순번>` 로 지정하며, 번호는 그 기종 매뉴얼의 파라미터 번호 그대로입니다. **번역하지 않으며 · 기종 간에 통일하지 않습니다** (`diagnosis` 필터와 같은 벤더 소유 번호 체계라 기종 간 대응표가 성립하지 않음). 파라미터에는 경로(채널)별 값이 있어 **채널 스코프**입니다. 자주 쓰는 파라미터는 `partCountActual`·`powerOnDuration` 처럼 이름 붙은 주소로 따로 제공되며, 이 주소는 그 밖을 위한 범용 통로입니다.

**모든 파라미터는 행의 배열로 봅니다.** `index` 는 행 순번입니다: 축형 파라미터면 축 번호, 스핀들형이면 스핀들 번호 (화면의 행 순서 그대로, 1부터. 예: X1=1, Y1=2), **단일값 파라미터는 행이 1개이므로 `index=1`** (`parameterValueList` 가 단일값을 원소 1개 배열로 주는 것과 같은 모델). 범위를 넘으면 실제 행 수를 함께 실어 상태 `-18` 로 거절합니다.

- 비트 단위 파라미터는 **바이트 값 그대로**(packed 정수) 오갑니다. 비트 분해/합성은 호출자 몫입니다. 특정 비트만 바꾸려면 읽고-수정-쓰기를 하세요 (그 사이 조작반 등 다른 변경과 경합할 수 있습니다).
- 소수(real) 파라미터는 장비의 소수 자릿수가 적용된 실수로 오가며, 쓰기도 같은 자릿수로 저장됩니다.
- **Mitsubishi 에는 수치가 아닌 파라미터가 있습니다** (예: 축 이름 `#1013` 이 `X`). 이 주소는 `float` 이라 표현할 수 없어 상태 `-18` 로 거절하며, 실제로 읽힌 문자열을 에러에 실어 줍니다. `index` 는 축별 파라미터면 축 번호, 아니면 `1` 뿐입니다.
- 정수 파라미터에 범위를 넘는 값을 쓰면 상태 `-16` 으로 거절합니다 (허용 범위를 에러에 함께 실어).
- **쓰기 주의**: 파라미터는 기계 거동을 바꿉니다. 장비가 파라미터 쓰기를 막아둔 상태면 상태 `-22`(기계 상태)로 거절되고, 일부 파라미터는 변경 후 전원 재투입을 요구합니다. Linux 용 Fanuc 라이브러리는 파라미터 쓰기를 제공하지 않아 쓰기는 상태 `-20` 입니다.

**Siemens 는 상태 `-20` 입니다.** 그 기종은 번호로 부르는 파라미터 체계 대신 이름으로 부르는 머신 데이터를 쓰며, 디메시는 그 통로를 이 주소로 열지 않습니다.

## /machine/channel/parameter/parameterValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["channel", "parameter"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

CNC 파라미터의 **전 축 값 배열**입니다 (**Fanuc·Mitsubishi**, `floatArray`, 읽기 전용). 배열 길이는 벤더 검증으로 파악한 **행 수**입니다: 축형 파라미터면 축 수, 스핀들형이면 스핀들 수, 비축이면 원소 1개 (`diagnosisValueList`·형제 `parameterValue` 의 행 모델과 동일). 행별 여부·행 수는 첫 조회 때 벤더 검증으로 파악되어 채널별로 캐싱되므로 반복 폴링이 가볍고, `parameter=6711-6713` 처럼 범위/콤마 확장 시 파라미터별 경계가 보존된 중첩 배열로 옵니다.

**Siemens 는 상태 `-20` 입니다.** 그 기종은 번호로 부르는 파라미터 체계 대신 이름으로 부르는 머신 데이터를 쓰며, 디메시는 그 통로를 이 주소로 열지 않습니다.

행이 하나 이상인 파라미터만 있으므로 `[]` 은 나오지 않습니다.

## /machine/channel/diagnosis/index/diagnosisValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "diagnosis", "index"]
read: ["nc_focas2_fanuc"]
write: []
```

진단 데이터 **한 행의 값**입니다 (**Fanuc 전용**, `float`, 읽기 전용). `channel=<채널>&diagnosis=<번호>&index=<순번>` 형식이며, 파라미터 통로와 같은 행 모델입니다: 축/스핀들 종속 진단이면 `index` 가 축/스핀들 번호, **단일값 진단은 행이 1개이므로 `index=1`** (`diagnosisValueList` 가 단일값을 원소 1개 배열로 주는 것과 같은 모델). 범위를 넘으면 실제 행 수를 함께 실어 상태 `-18` 로 거절합니다. 축 하나만 주기 폴링할 때 전 행을 읽는 List 보다 가볍습니다 (호출 1회).

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** Fanuc 진단 번호 체계는 그 기종 소유라 대응이 없습니다. Mitsubishi 의 NC 내부 데이터는 `/machine/channel/diagnosisSection/diagnosisSubsection/index/diagnosisValue` 로 읽으세요.

## /machine/channel/diagnosis/diagnosisValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["channel", "diagnosis"]
read: ["nc_focas2_fanuc"]
write: []
```

임의 진단 번호의 **값**입니다 (**Fanuc 전용**, `floatArray`). 축/스핀들 종속 진단은 축 수만큼의 배열, 비종속 진단은 원소 1개 배열. 진단에는 경로(채널)별 값이 있어 **채널 스코프**입니다. `channel=` 로 경로를 지정하며, 경로 공통 진단은 어느 채널로 읽어도 같은 값이 옵니다. 진단별 형식(행별 여부·행 수)은 첫 조회 때 벤더 검증으로 파악되어 채널별로 캐싱되므로 반복 폴링이 가볍습니다. `diagnosis=301,308` 처럼 콤마/범위 확장 시 진단별 경계가 보존된 중첩 배열로 옵니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** Fanuc 진단 번호 체계는 그 기종 소유라 대응이 없습니다. Mitsubishi 의 NC 내부 데이터는 `/machine/channel/diagnosisSection/diagnosisSubsection/index/diagnosisValue` 로 읽으세요.

행이 하나 이상인 진단만 있으므로 `[]` 은 나오지 않습니다.

## /machine/channel/diagnosisSection/diagnosisSubsection/index/diagnosisValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "diagnosisSection", "diagnosisSubsection", "index"]
read: ["nc_ezsocket_mitsubishi"]
write: []
```

NC 내부 데이터 **한 행의 값**입니다 (**Mitsubishi 전용**, `float`, 읽기 전용). **섹션 번호 · 서브섹션 번호 · 축 번호**로 지정합니다: `channel=<채널>&diagnosisSection=<섹션>&diagnosisSubsection=<서브섹션>&index=<순번>`.

**번호를 이미 알고 있는 경우를 위한 통로입니다.** 이 번호 체계는 벤더 소유라 **번역하지 않으며 · 기종 간에 통일하지 않습니다** (`plcAddress`·`diagnosis` 와 같은 부류). Fanuc 의 `diagnosis` 는 번호가 하나인데 이쪽은 둘이라 주소를 따로 두었습니다. 두 기종의 진단 데이터를 같은 주소로 부를 수 없습니다.

- 자주 쓰는 값은 `axisLoad`·`spindleLoad` 처럼 **이름 붙은 주소**로 따로 제공됩니다. 그쪽이 있으면 그쪽을 쓰세요. 이 주소는 그 밖을 위한 범용 통로입니다
- `index` 는 **행 순번**입니다 (1부터). 섹션에 따라 축 번호이거나 스핀들 번호이고, 행이 없는 데이터는 1행이므로 `index=1` 입니다. 범위를 넘으면 실제 행 수를 함께 실어 상태 `-18`
- **수치가 아닌 데이터가 있습니다.** 이 통로는 10진·16진·실수·문자열을 모두 다루는데 이 주소는 `float` 이라 문자열은 상태 `-18` 로 거절하고 실제로 읽힌 값을 에러에 실어 줍니다 (16진 표시 데이터는 값 자체가 정수라 그대로 옵니다)
- 없는 섹션·서브섹션은 상태 `-18`

**Siemens 는 상태 `-20` 입니다.** 이 섹션·서브섹션 번호 체계는 Mitsubishi 소유입니다.

## /machine/ncMemorySizeTotal
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

NC 메모리 전체 용량입니다. 반환 `int` + `unit:"bytes"`, 읽기 전용.

가리키는 것은 **가공 프로그램 메모리**입니다. Fanuc 은 `cnc_rdpdf_inf` 의 드라이브 용량(필드 이름과 달리 바이트 단위), Siemens 는 NC 의 **패시브 파일 시스템 사용자 영역**(메인/서브프로그램·워크피스·GUD 정의 파일이 사는 곳, 확장 기능 매뉴얼 S7)인 `/Nck/State/usedMemDramUPassF`·`freeMemDramUPassF`, Mitsubishi 는 `GetInformation` 의 문자 수(조작반 편집 화면의 `기억용량`·`나머지` 와 같은 값)입니다. 세 값은 같은 조회에서 나오므로 `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree` 로 맞아떨어집니다.

크기는 SDK 전체에서 **바이트**로 통일되어 있습니다. `entry`/`entryList` 의 `sizeBytes` 와 같은 단위이므로 "이 파일이 남은 공간에 들어가나" 같은 계산에 변환이 필요 없습니다. 장비가 더 거친 단위로만 알려주는 경우에도 이 주소는 바이트로 환산해 내보내며, 그때 값은 그 단위의 배수가 됩니다.

## /machine/ncMemorySizeUsed
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

NC 메모리 사용량입니다. 반환 `int` + `unit:"bytes"`, 읽기 전용.

가리키는 것은 **가공 프로그램 메모리**입니다. Fanuc 은 `cnc_rdpdf_inf` 의 드라이브 용량(필드 이름과 달리 바이트 단위), Siemens 는 NC 의 **패시브 파일 시스템 사용자 영역**(메인/서브프로그램·워크피스·GUD 정의 파일이 사는 곳, 확장 기능 매뉴얼 S7)인 `/Nck/State/usedMemDramUPassF`·`freeMemDramUPassF`, Mitsubishi 는 `GetInformation` 의 문자 수(조작반 편집 화면의 `기억용량`·`나머지` 와 같은 값)입니다. 세 값은 같은 조회에서 나오므로 `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree` 로 맞아떨어집니다.

크기는 SDK 전체에서 **바이트**로 통일되어 있습니다. `entry`/`entryList` 의 `sizeBytes` 와 같은 단위이므로 "이 파일이 남은 공간에 들어가나" 같은 계산에 변환이 필요 없습니다. 장비가 더 거친 단위로만 알려주는 경우에도 이 주소는 바이트로 환산해 내보내며, 그때 값은 그 단위의 배수가 됩니다.

## /machine/ncMemorySizeFree
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

NC 메모리 잔여 용량입니다. 반환 `int` + `unit:"bytes"`, 읽기 전용.

가리키는 것은 **가공 프로그램 메모리**입니다. Fanuc 은 `cnc_rdpdf_inf` 의 드라이브 용량(필드 이름과 달리 바이트 단위), Siemens 는 NC 의 **패시브 파일 시스템 사용자 영역**(메인/서브프로그램·워크피스·GUD 정의 파일이 사는 곳, 확장 기능 매뉴얼 S7)인 `/Nck/State/usedMemDramUPassF`·`freeMemDramUPassF`, Mitsubishi 는 `GetInformation` 의 문자 수(조작반 편집 화면의 `기억용량`·`나머지` 와 같은 값)입니다. 세 값은 같은 조회에서 나오므로 `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree` 로 맞아떨어집니다.

크기는 SDK 전체에서 **바이트**로 통일되어 있습니다. `entry`/`entryList` 의 `sizeBytes` 와 같은 단위이므로 "이 파일이 남은 공간에 들어가나" 같은 계산에 변환이 필요 없습니다. 장비가 더 거친 단위로만 알려주는 경우에도 이 주소는 바이트로 환산해 내보내며, 그때 값은 그 단위의 배수가 됩니다.

**Mitsubishi 는 이 값이 250바이트 눈금으로 움직입니다**. 장비가 잔여를 250문자 단위로만 세기 때문입니다. 단위는 다른 기종과 같은 바이트이고, 값이 250의 배수가 될 뿐입니다.

## /machine/ncMemoryRootPath
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

메인 NC 메모리의 루트 경로입니다. `ncMemoryPath` 필터에 넣을 경로의 시작점. Fanuc 은 보통 `//CNC_MEM`, Siemens 는 `//NC`, Mitsubishi 는 보통 `//PRG` 입니다.

**Mitsubishi 는 이 값 자체로는 목록이 비어 있습니다.** 프로그램은 한 단계 아래에 있어 `{root}/USER` 로 조회하세요 (MDI 버퍼는 `{root}/MDI`).

## /machine/ncMemoryExternalRootPathList
```yaml
value_type: "stringArray"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

메인 NC 메모리 루트 외의 **외부 저장소 드라이브** 목록입니다 (예: 데이터 서버, 메모리 카드). 반환 타입 `stringArray`.

- 각 항목은 root 처럼 앞에 `//` 가 붙습니다 (뒤 슬래시는 없음). Fanuc `//DATA`·`//MEMCARD`, Siemens `//Local drive`, Mitsubishi `//IC1` (NC 쪽 SD카드, 조작반의 `DS`). 장착되지 않았으면 빈 배열입니다
- 이름은 **장비 HMI 표기**입니다. Siemens 의 로컬 드라이브는 OPC-UA 내부 이름이 `NCExtend` 이지만 조작반과 같게 `//Local drive` 로 내보냅니다 (옛 표기 `//NCExtend` 로 요청해도 받습니다)
- 메인 루트 자신은 이 목록에서 제외됩니다
- **캐시하지 않음.** 외부 장치는 연결/해제로 바뀔 수 있어 요청마다 새로 조회
- 필터 없음

## /machine/ncMemoryPath/entry
```yaml
value_type: "object"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

경로의 항목 1개 정보입니다 (`object`). 키 집합은 **기종과 무관하게 항상 같습니다.** 값이 없으면 키가 빠지는 게 아니라 `null` 입니다.

| 키 | 타입 | 없을 때 |
|---|---|---|
| `name` | `string` | 없음 |
| `sizeBytes` | `int` | 폴더, 또는 크기를 못 읽은 경우 `null` |
| `modifiedAt` | `string` | 폴더, 또는 기종이 수정 시각을 제공하지 않으면 `null` (Siemens 는 항상 `null`) |
| `isDir` | `boolean` | 없음 |
| `comment` | `string` | 폴더, 또는 기종이 주석을 제공하지 않으면 `null` (Siemens 는 항상 `null`) |

경로 끝 `/` 로 폴더를 명시할 수 있고, 없으면 파일 우선 검색입니다. 항목이 없으면 에러입니다.

## /machine/ncMemoryPath/entryList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

폴더 안의 파일/폴더 목록입니다 (**읽기 전용**, `objectArray`). 각 원소는 `entry` 와 **완전히 같은 객체**입니다. 키 집합·`null` 규약 모두 동일하므로 그쪽 표를 보세요. 폴더 우선, 이름 오름차순으로 정렬됩니다. 폴더의 생성/삭제는 `directoryExists` 를 사용하세요.

없는 폴더를 지정하면 상태 `-18` 입니다.

폴더가 비어 있으면 `[]` 입니다.

## /machine/ncMemoryPath/entryName
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

`ncMemoryPath` 가 가리키는 **항목의 이름**입니다. **장비에 있는 그 항목의 이름**을 돌려주고(read), **이름 변경**(write)을 합니다. 읽기는 파일·폴더 어느 쪽이든 답하며, 그 경로에 아무것도 없으면 상태 `-18` 로 거절합니다 (`entry`·`fileExists` 와 같은 판단입니다). 돌려주는 것은 요청에 적은 문자열이 아니라 **장비가 가진 이름**이라, 표기가 다르면 장비 쪽 표기가 나옵니다. 쓰기는 `{"value": "새이름"}` 이며 경로 구분자는 넣을 수 없습니다 (파일/폴더 공통). **루트의 이름은 바꿀 수 없습니다**: `ncMemoryPath` 가 `//CNC_MEM`·`//NC`·`//PRG` 나 외부 드라이브처럼 `//이름` 한 토막이면 장비에 보내지 않고 상태 `-18`(필터 값 오류)로 거절합니다 (`directoryExists` 의 루트 삭제 거절과 같은 규칙). `entry`/`entryList` 와 같은 "항목" 을 가리킵니다.

## /machine/ncMemoryPath/directoryExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

경로에 **폴더**가 존재하는지 확인하고(read), 상태를 선언적으로 씁니다(write):

- read → 폴더가 있으면 `true` (같은 이름의 파일만 있으면 `false`)
- write `{"value": true}` → 폴더 생성 (이미 있으면 상태 `-21`(이미 존재). Fanuc 데이터 서버 `//DATA_SV/` 에서는 이미 있음을 가려낼 수 없어 상태 `-17`(핸들러 에러)로 거절됩니다)
- write `{"value": false}` → 폴더 삭제 (**빈 폴더만**: 내용이 있으면 상태 `-17` 에 벤더 사유, 재귀 삭제는 지원하지 않음). **루트는 지울 수 없습니다**: `ncMemoryPath` 가 `//CNC_MEM`·`//NC`·`//PRG` 나 외부 드라이브처럼 `//이름` 한 토막이면 장비에 보내지 않고 상태 `-18`(필터 값 오류)로 거절합니다 (뒤 `/` 유무 무관)
- **Mitsubishi 의 NC 메모리 드라이브에서는 폴더 생성·삭제가 상태 `-20` 입니다.** 그 드라이브는 디렉터리 구성이 고정이라 제어기가 폴더 생성·삭제를 받지 않습니다. 읽기는 정상입니다

경로 끝 `/` 는 무시됩니다. 파일은 `fileExists` 사용.

## /machine/ncMemoryPath/fileExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

경로에 **파일**이 존재하는지 확인하고(read), 상태를 선언적으로 씁니다(write):

- read → 파일이 있으면 `true` (같은 이름의 폴더만 있으면 `false`)
- write `{"value": false}` → 파일 삭제
- write `{"value": true}` → 상태 `-16`(쓰기 값 오류)으로 거절합니다: 빈 파일 생성은 지원하지 않습니다. 파일 생성은 내용과 함께 `fileContent` 쓰기로 하세요

경로 끝 `/` 는 무시됩니다 (종류는 주소가 확정). 폴더는 `directoryExists` 사용.

## /machine/ncMemoryPath/fileContent
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

NC 파일의 **내용**을 읽고(다운로드) 씁니다(업로드: 없으면 생성, 있으면 덮어씀). 값은 문자열 (프로그램 텍스트).

- **Fanuc 쓰기 자동 처리**: `%` 미포함 시 자동 삽입, 맨 앞에 O번호/`<이름>` 이 없으면 경로의 파일명 기준으로 자동 삽입. 저장 파일명은 **내용의 O번호/이름 기준**입니다
- **Siemens·Mitsubishi 는 내용을 그대로 씁니다.** 자동 삽입이 없고, 저장 파일명은 **경로의 파일명**입니다. 내용의 O번호가 달라도 경로대로 저장됩니다 (Fanuc 과 반대). `%` 나 O번호가 필요하면 값에 직접 넣으세요
- 파일 삭제는 `fileExists` 에 `false` 쓰기

## /machine/channel/toolOffsetCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

공구 보정 레지스터의 **사용 가능 개수**입니다 (read 전용, `int`). 오프셋 번호는 `1`~이 값까지입니다. UI 가 테이블을 순회할 때 상한으로 쓰세요.

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

공구 보정 번호 하나의 **보정량**입니다 (read + write, `float`). `channel` + `toolOffset` 필터가 필요하고, 쓰기는 `{"value": 12.345}` 입니다.

**이 주소는 보정 메모리가 열로 나뉘지 않은 장비 전용입니다.** 그런 장비의 조작반 보정량 화면은 번호마다 값을 **하나만** 보여줍니다. 형상/마모도, 길이/반경도 나뉘지 않습니다. 그래서 이름이 `toolLength…` 가 아니라 `toolOffsetValue` 입니다. **장비가 그 값을 "길이" 라고 부르지 않기 때문**이며, 그것이 길이 보정으로 쓰일지 반경 보정으로 쓰일지는 프로그램이 그 번호를 어떻게 참조하느냐에 달려 있습니다.

열이 나뉜 장비에서는 상태 `-20` 이 반환되며, **에러 문자열에 그 장비에서 되는 리프 목록**이 실려 옵니다 (예: `toolLength{Geometry,Wear}, toolRadius{Geometry,Wear}`). 즉 한 번 요청해 보면 그 장비의 보정 트리 모양을 알 수 있으므로, 어느 모델인지 미리 묻는 주소는 따로 없습니다.

보정 번호의 상한은 `/machine/channel/toolOffsetCount` 입니다. 그 범위를 벗어난 번호는 상태 `-18` 입니다. 값의 단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요.

**Fanuc·Siemens 는 상태 `-20` 입니다.** Fanuc 은 보정 메모리가 열로 나뉜 표라 위의 안내대로 에러 문자열이 되는 리프를 알려주고, Siemens 는 공구 단위 표라 `/machine/toolArea/tool/toolEdge/…` 로 읽습니다.

## /machine/channel/toolOffset/toolName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 보정 번호로 불리는 **공구의 이름**입니다. `channel` + `toolOffset` 필터. 반환 `string`. 읽기·쓰기 모두 Fanuc 전용입니다.

값은 Fanuc 의 **공구 형상 크기 데이터**(조작반 `TL GEOM SIZE` 화면, 옵션 "Tool geometry size data 100/300 pairs")에서 옵니다. 그 표는 **공구 보정 번호로 색인**됩니다 (조작 설명서 B-64484EN 13.2.2.2: M 계열은 `D` 코드, T 계열은 형상 오프셋 번호와 같은 번호의 행). 그래서 이 주소는 공구관리 칸(`/machine/toolArea/tool/…`)이 아니라 `toolOffset` 폴더에 있고, 공구관리(TOOL MANAGEMENT) 옵션과는 무관합니다. 형상 크기 데이터 옵션이 없으면 상태 `-20` 입니다.

종류가 정해지지 않은 행(공구 종류 `0`)은 빈 문자열 `""` 입니다. 표의 크기는 옵션이 정하므로 `/machine/channel/toolOffsetCount` 와 같다는 보장이 없습니다 (벤치 장비는 둘 다 100). 표 밖 번호는 상태 `-18` 로 거절하며 문구에 표의 끝을 싣습니다. 여러 번호를 범위로 물으면(`toolOffset=1-100`) 한 번의 왕복으로 읽습니다.

**쓰기**는 장비에 8바이트 이내로 남는 문자열입니다 (초과는 상태 `-16`. 장비의 표시 언어 코드페이지로 옮길 수 없는 글자도 상태 `-16`). 종류가 정해지지 않은 행에는 이름을 쓸 수 없어 상태 `-18` 입니다 (Fanuc 은 종류 없는 행을 만들지 않습니다. 조작반 `TL GEOM SIZE` 화면에서 종류를 먼저 정하세요). 이미 같은 이름이면 아무것도 하지 않고 성공합니다.

Siemens 의 공구 이름은 공구 번호에 붙으므로 `/machine/toolArea/tool/toolName` 입니다. 같은 뜻(공구의 이름)이지만 키가 달라 주소가 나뉩니다.

**Mitsubishi 는 상태 `-20` 입니다.** 이 열은 Fanuc 공구 형상 크기 데이터의 것이라 Mitsubishi 의 보정 표에는 없습니다.

## /machine/channel/toolOffset/toolType
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 보정 번호로 불리는 **공구의 종류**입니다 (조작반 `TL GEOM SIZE` 화면의 `TK` 아이콘). `channel` + `toolOffset` 필터. 반환 `int` + `desc`. 읽기·쓰기 모두 Fanuc 전용입니다.

값은 **Fanuc 의 원시 코드를 그대로** 냅니다. 벤더가 소유한 열린 분류라 번역하지 않으며, 기종 간에 통일하지 않습니다 (Siemens `/machine/toolArea/tool/toolEdge/toolType` 의 DP1 코드와는 **다른 코드 공간**이고 키도 다릅니다). `desc` 는 사람이 읽는 문구라 분기는 값으로 하세요.

| 값 | 뜻 |
|---|---|
| `0` | 정의되지 않은 행 |
| `10` | 범용 선삭공구 |
| `11` | 나사 공구 |
| `12` | 홈 공구 |
| `13` | 라운드노즈 공구 |
| `14` | 포인트노즈 직선 공구 |
| `15` | 다기능 공구 |
| `20` | 드릴 |
| `21` | 카운터싱크 |
| `22` | 플랫 엔드밀 |
| `23` | 볼 엔드밀 |
| `24` | 탭 |
| `25` | 리머 |
| `26` | 보링 공구 |
| `27` | 페이스밀 |

출처·표 크기·옵션은 `/machine/channel/toolOffset/toolName` 과 같습니다: 공구 형상 크기 데이터(옵션 "Tool geometry size data 100/300 pairs"), 공구 보정 번호로 색인, 없으면 상태 `-20`, 표 밖 번호 상태 `-18`.

**쓰기**는 위 표의 코드만 받습니다 (그 밖은 상태 `-16`). 정의되지 않은 행에 `0` 이 아닌 종류를 쓰면 **그 행이 만들어집니다.** 이어서 `toolName` 을 쓰세요. **`0` 을 쓰면 그 행이 지워집니다** (이름과 치수까지 함께 사라집니다. 제어기가 종류 `0` 쓰기를 삭제로 정의합니다). 이미 같은 값이면 아무것도 하지 않고 성공합니다. **정의된 행의 종류를 다른 종류로 바꾸면 제어기가 그 행을 새로 정의해 이름과 치수가 지워집니다** (벤치 실측: `20`→`27` 로 바꾸자 이름이 `""` 로). 종류를 먼저 정하고 이름·치수를 넣으세요.

**Mitsubishi 는 상태 `-20` 입니다.** 이 열은 Fanuc 공구 형상 크기 데이터의 것이라 Mitsubishi 의 보정 표에는 없습니다.

## /machine/channel/toolOffset/toolLengthGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**M계(머시닝센터) 공구 길이 형상값**입니다 (오프셋 화면의 H 열). 공구를 측정해 넣는 기준값으로, 길이 보정(H 코드)의 바탕이 됩니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolLengthWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**M계 공구 길이 마모값**입니다 (오프셋 화면의 H 열). 가공 중 쌓이는 미세 보정분으로, 형상값은 그대로 두고 이쪽만 조정하는 것이 일반적입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**M계 공구경 형상값**입니다 (오프셋 화면의 D 열). 공구경 보정(G41/G42)이 참조하는 반경 기준값입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 **반지름**이지만, 오프셋 화면은 설정에 따라 **지름**으로 표시·입력하게 설정돼 있을 수 있습니다. 디메시는 장비가 저장한 값을 그대로 내보내며 임의로 환산하지 않습니다.

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**M계 공구경 마모값**입니다 (오프셋 화면의 D 열). 공구 마모에 따른 반경 감소를 반영합니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 **반지름**이지만, 오프셋 화면은 설정에 따라 **지름**으로 표시·입력하게 설정돼 있을 수 있습니다. 디메시는 장비가 저장한 값을 그대로 내보내며 임의로 환산하지 않습니다.

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolXGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계(선반) X 방향 공구 치수 형상값**입니다. 여기서 X 는 축 이름이 아니라 오프셋 화면의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolXWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 X 방향 공구 치수 마모값**입니다. 가공 중 누적되는 X 방향 보정분입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolZGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Z 방향 공구 치수 형상값**입니다. X 와 마찬가지로 축이 아니라 화면의 고정 열입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolZWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Z 방향 공구 치수 마모값**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolYGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Y 방향 공구 치수 형상값**입니다. X·Z 에 이은 **세 번째 열**이며, 옵션이 없는 선반에서는 상태 `-20` 이 반환됩니다.

**기계에 따라 이 열의 화면 머리글이 `Y` 가 아닐 수 있습니다.** Mitsubishi 는 이 자리를 제3축에 배정하므로 C축 선반에서는 조작반이 `공구길이 C` 로 표시합니다 (벤더 매뉴얼도 이 열을 `C (Y*)` 로 적습니다). 주소가 약속하는 것은 **세 번째 오프셋 열**이며, 그 열이 어느 축인지는 조작반의 열 머리글이 알려 줍니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolYWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Y 방향 공구 치수 마모값**입니다 (Y축 옵션 장비 전용). X·Z 에 이은 **세 번째 열**이며, 기계에 따라 화면 머리글이 `Y` 가 아닐 수 있습니다 (Mitsubishi C축 선반은 `마모 C`). 자세히는 `toolYGeometry` 를 보세요.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 노즈 반경 형상값**입니다. 노즈 반경 보정(G41/G42)이 참조하며, 팁 방향(`toolTipDirection`)과 함께 날끝 궤적을 결정합니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolNoseRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 노즈 반경 마모값**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

선반 공구의 **가상 날끝 위치 코드**입니다 (read + write). 노즈 반경 보정(G41/G42) 때 날끝이 노즈 중심 기준 어느 방위에 있는지 판정하는 코드입니다. 각도가 아니라 위치 코드이며, 배율 없는 정수 그대로 반환/입력합니다 (`{"value": 3}`). `channel` + `toolOffset` 필터가 필요합니다. **선반 계열 오프셋 메모리에만 있는 열**이라, 기종의 오프셋 화면에 이 열이 없으면 상태 `-20`(미지원)으로 거절하고 그 채널에서 쓸 수 있는 리프 목록을 에러 문자열에 실어 줍니다. 보정 메모리가 열로 나뉘지 않은 장비도 여기 해당하며, 그때는 `toolOffsetValue` 를 쓰라고 안내합니다.

- `1`~`8` = 방위, **`0`/`9` = 노즈 중심이 기준점** (가상 날끝이 아니라). 두 값은 같은 의미입니다. 노즈 중심이 기준점과 일치할 때 `0` 또는 `9` 를 쓴다고 Fanuc 0i-F 선반 매뉴얼(`B-64604EN-1/01` §5.2.2)이 정의합니다
- **`desc` 는 `0`·`9` 에만 붙습니다.** `1`~`8` 은 매뉴얼이 평면별 도해로 정의하고 그 도해가 평면(`G17`/`G18`/`G19`)별로 여러 벌이라, 같은 번호가 구성에 따라 다른 방위를 가리킵니다. 방위 해석은 그 기종 매뉴얼의 도해를 따르세요
- Siemens 대응 개념: cutting edge position (`toolArea/tool/toolEdge/toolTipDirection`). **두 트리는 같은 번호 체계와 같은 `desc` 어휘를 씁니다.** 주소만 다를 뿐 값은 그대로 비교·재사용할 수 있습니다
- **허용 범위가 기종마다 다릅니다**: Fanuc `0`~`9`, Siemens `1`~`9`, Mitsubishi `0`~`8`. 중심을 가리키는 코드도 각각 `0`/`9`, `9`, `0` 이라 **읽을 때는 `desc` 로 통일되지만 쓸 때는 그 장비의 범위를 지켜야 합니다** (Fanuc 에서 되는 `9` 를 Mitsubishi 에 그대로 보내면 상태 `-16`)

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/activeToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 채널에서 지금 활성인 공구의 번호(`T`)입니다. `channel` 필터. 반환 `int`.

**"활성" 이 되는 시점이 기종에 따라 다릅니다.** Fanuc·Mitsubishi 는 `T` 모달이라 **지령하는 순간** 바뀌고, Siemens 는 `actTNumber` 라 **교환이 끝난 뒤에** 바뀝니다:

| 상황 | Fanuc·Mitsubishi | Siemens |
|---|---|---|
| `T7` 만 지령 (아직 `M06` 전) | `7` | **그 장비의 교환 방식에 달렸습니다** (아래) |
| `T7 M06` 이 끝난 뒤 | `7` | `7` |

⚠️ **Siemens 는 `T` 만으로 교환이 끝나는 장비가 있습니다.** 교환을 `M06` 이 하는지 `T` 가 하는지는 기계 제작사가 정하는 설정입니다. 시험 장비(840D sl)에서 `T="CUTTER 10"` 만 지령했더니 **`M06` 없이 교환이 완료됐습니다**: 이 값이 곧바로 바뀌었고, 그 공구의 `toolLocationType` 이 `magazine` → `buffer`(스핀들)로, 물러난 공구는 반대로 갔습니다. 그러니 이 값을 "교환 전" 의 표시로 쓰지 마세요. 교환 여부는 `/machine/toolArea/tool/toolLocationType` 이 확실히 답합니다.

앞의 두 기종은 확인했습니다. `M06` 없이 `T7` 만 준 직후 값이 `7` 이 되며, Mitsubishi 는 조작반의 공구번호 표시가 이전 공구에 그대로 있는 것까지 함께 관측했습니다. 리셋(`M30`)으로도 지워지지 않습니다. Fanuc 쪽은 **실제 공구교환 매크로가 도는 제어기**에서 다시 확인했습니다: `T` 만 있는 블록 다음 드웰에서 이미 그 번호였고(`M06` 은 아직 실행 전), 교환이 끝난 뒤에도 같았으며, 두 번째 `T` 에서도 그 자리에서 바뀌었습니다. 프로그램이 `M30` 으로 끝난 뒤에도 값이 남았습니다.

**그래서 Fanuc·Mitsubishi 에서는 이 값을 "지금 깎고 있는 공구" 로 해석하면 안 됩니다**. 지령된 공구이기 때문입니다. 교환 시점이 중요한 용도라면 이 주소를 교환 신호로 쓰지 마시고 장비의 교환 완료 신호를 보세요. 실제로 물려 있는 공구는 이 두 기종의 SDK 통로로는 얻을 수 없습니다. 조작반에 뜨는 공구번호는 기계 제작사가 래더로 만드는 값이라 장비마다 다르고, `plcAddress` 와 같은 이유로 중립화가 성립하지 않습니다.

⚠️ **Fanuc 에 공구관리(Tool Management) 옵션이 켜져 있으면 이 값은 공구 번호가 아닙니다.** 그 옵션에서는 `T` 가 공구를 직접 가리키지 않고 **공구 타입(그룹) 번호**를 지정하며, 제어기가 그 타입에 속한 실제 공구를 골라 씁니다. 테스트 환경에서 확인: 조작반의 `EACH TOOL DATA` 가 공구 `1` 의 타입을 `4` 로 두고 있을 때 `T4` 를 지령하니 이 주소가 `4` 를 냈습니다(실제 공구는 `1`). 옵션이 꺼진 장비에서는 `T` 가 곧 공구 번호라 이 문제가 없습니다. 그 옵션을 쓰는 장비라면 이 값을 아래 공구 트리 조회에 그대로 넣지 마세요. 대신 `/machine/toolArea/toolList` 에서 `toolTNumber` 가 이 값과 같은 항목들이 후보 공구이고, `/machine/toolArea/tool/toolTNumber` 로 낱개 확인할 수 있습니다.

⚠️ **Siemens 의 공구관리(WZV)에서는 프로그램의 `T` 가 이름을 가리킵니다.** 이 주소가 내는 번호(그리고 `tool` 필터가 받는 번호)는 제어기 **내부의 공구 번호**라, 프로그램에 그대로 타이핑하는 값이 아닙니다. 시험 장비에서 `T3` 은 알람 `17190`(illegal T number)이었고 `T="CUTTER 10"` 이 통과했으며, 그 공구의 내부 번호가 바로 `3` 이었습니다. 번호 ↔ 이름은 `/machine/toolArea/toolList` 의 `toolNumber` 와 `toolName` 이 짝지어 알려줍니다.

**공구수명관리(tool life management)로 그룹을 지령해도 이 값은 공구 번호입니다.** 프로그램이 파라미터 `6810` 보다 큰 값으로 그룹을 부르면(예: `6810` 이 `1000` 인 장비에서 `T1001` = 그룹 `1`), 제어기가 그 그룹에서 쓸 공구를 골라 **그 공구 번호를 이 자리에 넣습니다.** 실측: 그룹 `1` 의 첫 공구가 `16` 인 장비에서 `T1001` 을 걸자 이 값이 `16` 이 됐고, 교환 후에도 `16` 이었습니다. 그룹 번호가 이 값으로 나오는 경우는 없습니다.

지금 수명이 깎이는 그룹은 `/machine/channel/activeToolGroupNumber` 가, 그 그룹의 공구 목록은 `/machine/toolArea/toolGroup/toolNumberList` 가 알려줍니다.

이 번호를 공구 트리의 `tool` 필터에 넣으면 그 공구의 이름·보정 세트 수·오프셋을 조회할 수 있습니다. 함께 필요한 `toolArea` 값은 `/machine/channel/toolAreaNumber` 가 알려줍니다 (연결 시 캐싱되어 추가 통신이 없습니다).

Fanuc 은 `T` 모달, Siemens 는 `actTNumber` (`$P_TOOLNO`: 지금 유효한 D 보정이 계산된 공구의 T 번호), Mitsubishi 는 `GetCommand2` 의 T 지령 모달을 읽습니다.

## /machine/channel/activeToolName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens"]
write: []
```

그 채널에서 지금 활성인 공구의 **이름**입니다. `channel` 필터. 반환 `string`.

**Siemens 전용입니다** (`actToolIdent`). 나머지 두 기종은 상태 `-20` 입니다.

- **Fanuc**: 공구관리 레코드에 이름 칸이 없습니다 (그래서 `/machine/toolArea/toolList` 의 `toolName` 도 Fanuc 에서는 빈 문자열입니다). 공구 형상 크기 데이터에 `/machine/channel/toolOffset/toolName` 이 있지만 그쪽은 **공구 보정 번호**로 색인되는 다른 표라, 활성 공구의 이름으로 풀려면 없는 대응을 지어내야 합니다.
- **Mitsubishi**: 공구관리 표에 이름 칸은 있으나, 그 기종의 `activeToolNumber` 는 `T` 모달(지령된 번호)이라 표의 행과 같다는 보장이 없습니다.

**`activeToolNumber` 와 짝입니다. 둘 다 있는 이유가 있습니다.** 공구관리(WZV)가 켜진 SINUMERIK 은 파트 프로그램의 `T` 가 **이름**을 가리킵니다. 즉 번호를 받아 `T3` 이라고 쓰면 제어기가 거절합니다 (시험 장비에서 알람 `17190` illegal T number). 프로그램에 그대로 쓸 수 있는 값은 이쪽이고, 우리 공구 트리(`/machine/toolArea/tool/…` 의 `tool` 필터)를 조회할 값은 `activeToolNumber` 입니다.

```
activeToolName   -> "CUTTER 10"   프로그램에 T="CUTTER 10"
activeToolNumber -> 3             toolArea/tool/*?tool=3
```

두 주소는 같은 묶음이라 함께 요청하면 왕복 한 번입니다.

**이름만으로는 공구가 유일하지 않을 수 있습니다.** SINUMERIK 의 공구 정체는 이름과 자매공구 번호(duplo)의 쌍이라, 같은 이름의 공구가 여럿일 수 있습니다. 그때 어느 것을 쓸지는 제어기가 정합니다. 하나를 정확히 지목해야 하면 `activeToolNumber` 를 쓰세요.

활성 공구가 없을 때는 **빈 문자열**입니다 (`activeToolNumber` 는 그 상태에서 `0`). 시험 장비에서 채널의 공구를 내려 확인했습니다. 제어기 자신은 그 자리에서 값을 주지 않지만, 내용이 없는 텍스트를 `null` 로 내지 않는 것이 이 SDK 의 규칙이라 빈 문자열로 맞춥니다.

## /machine/channel/activeToolEdgeNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens"]
write: []
```

활성 공구에서 지금 **보정이 적용되고 있는 보정 세트**의 번호(`D`)입니다. `channel` 필터. 반환 `int`. Siemens 의 `actDNumber` 입니다. 이 주소가 답하는 것은 "몇 번 보정 세트냐" 이지 "공구가 걸렸냐" 가 아닙니다 (그건 `activeToolNumber` 가 `0` 으로 답합니다).

**Siemens 전용**입니다 (Fanuc·Mitsubishi 는 상태 `-20`). 두 기종의 오프셋 모델에는 공구에 딸린 날(보정 세트) 계층이 없어 "몇 번째 날" 이라는 물음 자체가 성립하지 않습니다. 예전 판은 그 두 기종에서 고정 `1` 을 냈는데, 없는 차원을 있다고 답하는 값이라 뺐습니다. Fanuc 에서 프로그램이 부르는 보정 번호는 공구 단위의 `/machine/toolArea/tool/toolHNumber`·`toolDNumber`(공구관리 옵션) 로 읽으세요.

## /machine/channel/activeToolGroupNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

이 채널에서 **지금 수명 카운트가 돌고 있는 공구그룹** 번호입니다. `channel` 필터. 반환 `int`, 읽기 전용입니다.

공구수명관리를 쓰는 장비에서 프로그램이 그룹을 걸면 그 그룹의 수명이 깎이기 시작하는데, 그 그룹의 번호입니다. **쓰는 그룹이 없으면 `0`** 입니다.

`/machine/channel/activeToolNumber` 가 "지금 걸린 공구" 라면 이 값은 "지금 수명이 깎이는 그룹" 입니다. 둘은 다른 개념이라 함께 보셔야 합니다.

**공구수명관리(tool life management) 옵션이 있어야 합니다.** 없으면 상태 `-20` 입니다. 이름이 비슷한 공구관리(tool management)와는 **다른 옵션**이며, 지금까지 본 장비에서는 둘이 함께 켜져 있지 않았습니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/channel/nextToolGroupNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

이 채널에서 **다음에 수명 카운트를 시작할 공구그룹** 번호입니다. `channel` 필터. 반환 `int`, 읽기 전용입니다.

프로그램이 `T` 로 그룹을 고르면 제어기가 그 그룹을 여기에 잡아 두고, 실제 사용이 시작되면 `/machine/channel/activeToolGroupNumber` 로 넘어갑니다. 즉 **고르기는 했는데 아직 쓰기 시작하지 않은** 그룹입니다. 사람이 미리 예약해 두는 값이 아닙니다.

**대기 중인 그룹이 없으면 `0`** 입니다.

`/machine/channel/activeToolGroupNumber` 와 같은 벤더 호출에서 나오므로 둘을 함께 읽어도 통신은 한 번입니다.

**공구수명관리(tool life management) 옵션이 있어야 합니다.** 없으면 상태 `-20` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/channel/selectedToolGroupNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

이 채널에서 **지금 수명 카운트가 도는 공구그룹, 또는 아무것도 안 돌면 마지막으로 돌았던 그룹** 번호입니다. `channel` 필터. 반환 `int`, 읽기 전용입니다.

`/machine/channel/activeToolGroupNumber` 와 **끈적임에서 갈립니다**:

| | 그룹이 도는 중 | 아무것도 안 돌 때 |
|---|---|---|
| `activeToolGroupNumber` | 그 그룹 | `0` |
| 이 주소 | 그 그룹 (같은 값) | **마지막으로 돌았던 그룹** |

기계가 쉬고 있을 때 "직전에 어느 그룹을 썼나" 를 아는 통로입니다. 전원을 껐다 켜면 초기화되어 `0` 이 됩니다.

**공구수명관리(tool life management) 옵션이 있어야 합니다.** 없으면 상태 `-20` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 공구 영역에 **등록된 공구의 수**입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기 전용. 등록된 공구가 없으면 `0` 입니다.

**Mitsubishi 는 이 주소가 느립니다** (시뮬레이터 NC Trainer2 plus 에서 약 1.2초). 그 제어기의 공구관리 표는 999칸이고 한 칸씩 물어야 하는데, 지운 자리에 **빈 칸이 남을 수 있어** 중간에서 멈출 수 없기 때문입니다. **주기 폴링에 쓰지 마세요** - 화면을 한 벌 그리는 용도입니다. 공구 하나만 필요하면 `/machine/toolArea/tool/…` 주소가 훨씬 빠릅니다 (그쪽은 찾으면 멈춥니다).

목록(`toolList`)이 돌려주는 항목 수와 같고 **장비의 같은 값**을 봅니다. 개수만 필요할 때 목록 전체를 받지 않아도 되도록 따로 둔 주소입니다. Siemens 의 공구 17개짜리 장비에서 실측하면 목록보다 훨씬 빠릅니다 (81ms 대 684ms).

없는 공구 영역을 지정하면 상태 `-18` 로 거절됩니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 옵션이 없으면 "공구" 라는 객체 자체가 없고, 보정 레지스터의 개수는 공구 수와 다른 값이라 대신 쓰지 않습니다. 공구관리 표(조작반 TOOL MANAGER 화면의 `NO.` 행, 칸 수는 파라미터 `13220`)에서 **등록 표시가 선 칸**만 셉니다 (공구 정보의 RGS 비트, 조작반 `T-INFO` 의 마지막 글자 `R`). 등록이 풀린 칸은 값이 남아 있어도 제어기가 무효 데이터로 보므로 세지 않습니다. 칸 수 `13220` 은 옵션의 상한이 아니라 기계 제작사 설정이며(64/240/1000 pairs 옵션 범위 안), 벤치 실측으로 칸 10개 중 등록 5개였습니다.

## /machine/toolArea/toolList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 공구 영역에 **등록된 공구 전부**의 목록입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `objectArray`, 등록된 공구가 없으면 빈 배열 `[]`.

**Mitsubishi 는 이 주소가 느립니다** (시뮬레이터 NC Trainer2 plus 에서 약 1.2초). 그 제어기의 공구관리 표는 999칸이고 한 칸씩 물어야 하는데, 지운 자리에 **빈 칸이 남을 수 있어** 중간에서 멈출 수 없기 때문입니다. **주기 폴링에 쓰지 마세요** - 화면을 한 벌 그리는 용도입니다. 공구 하나만 필요하면 `/machine/toolArea/tool/…` 주소가 훨씬 빠릅니다 (그쪽은 찾으면 멈춥니다).

항목: `{"toolNumber": 16, "toolTNumber": null, "toolName": "BALLNOSE_D8", "toolEdgeCount": 4, "sisterToolNumber": 9, "magazineNumber": 0, "pocketNumber": 0, "toolLocationType": "buffer", "toolTeethCount": null, "toolBodyLength": null, "toolBodyDiameter": null, "toolOffsetNumber": null}`

공구 번호는 **연속되지 않습니다.** 공구 17개가 2번~18번을 쓰고 1번은 없는 식이라, 번호를 1부터 넣어보는 것으로는 무엇이 있는지 알 수 없습니다. 이 목록이 그 답이며, 항목의 `toolNumber` 를 그대로 `tool` 필터에 넣어 공구별 주소를 조회하는 것이 용법입니다.

**순서는 기종에 따라 다르되, 매번 같은 순서가 보장됩니다.** Fanuc·Siemens 는 `toolNumber` 오름차순입니다. Siemens 의 공구 목록 화면은 보통 이름순이고 작업자가 정렬 기준을 바꿀 수 있어 맞출 수 있는 하나의 "화면 순서" 가 없으므로 번호순으로 고정했습니다 (화면과 같은 순서로 보여주려면 `toolName` 으로 정렬하세요). **Mitsubishi 는 공구관리 표의 행 순서 그대로**입니다. 그 기종의 화면은 표 행 순서가 그대로 보이므로 이쪽이 화면과 일치하며, 지운 행이 나중에 새 공구로 채워지면 번호 오름차순이 아닐 수 있습니다.

**`toolNumber` 는 장비 화면의 `Loc.`(자리 번호)이 아닙니다.** 공구 관리를 쓰는 장비에서는 공구를 이름과 자매번호로 식별하므로 이 번호가 목록 화면에 나오지 않습니다 (공구 상세 화면의 `Tool number` 항목이 이 값입니다). 화면의 `Loc.` 을 `tool` 필터에 넣으면 **다른 공구를 조회하고도 성공으로 보입니다.** 두 번호가 우연히 같은 공구가 많아 알아채기 어렵습니다. 그 값은 `pocketNumber` 이며, 이 목록이 둘을 함께 담고 있어 대응을 확인할 수 있습니다.

- **toolNumber**: 공구 번호. `tool` 필터에 넣는 값
- **toolTNumber**: 프로그램이 `T` 로 이 공구를 부르는 번호 (Fanuc 공구관리의 `TYPE NO.`, 여러 공구가 같은 값을 가질 수 있음). 이름으로 부르는 Siemens 와, 이 개념이 없는 Mitsubishi 는 `null`
- **toolName**: 공구 이름. 이름을 쓰지 않는 장비에서는 빈 문자열
- **toolEdgeCount**: 보정 세트 개수 (인선 수가 아니고, **가장 큰 D 번호도 아닙니다**. 중간 삭제로 구멍이 나면 번호가 개수보다 클 수 있습니다: `/machine/toolArea/tool/toolEdgeCount` 참조)
- **sisterToolNumber**: 자매공구 번호 (이름이 같은 공구들을 구분하는 번호. 조작반의 `ST` 열)
- **magazineNumber**: 지금 꽂혀 있는 매거진(공구 저장고) 번호. 매거진 밖이면 `0`
- **pocketNumber**: 그 매거진 안의 포켓 번호. 매거진 밖이면 `0`
- **toolLocationType**: 자리의 종류. `"magazine"`(매거진에 있음) · `"buffer"`(스핀들 또는 교환기) · `"loading"`(반입·반출 위치) · `"none"`(실물 자리 없음)
- **toolTeethCount** · **toolBodyLength** · **toolBodyDiameter** · **toolOffsetNumber**: Mitsubishi 공구관리 표의 공구 단위 열 (뜻은 같은 이름의 단독 주소 참조). 그 열이 없는 Fanuc·Siemens 는 `null`

이 목록은 **무엇이 있고 · 어떻게 부르고 · 어디 있나** 까지 답합니다. 오프셋·마모 같은 측정값은 보정 세트 단위라 담지 않습니다.

값이 없으면 키를 빼지 않고 `null` 입니다 (위치 세 필드는 위치를 아는 기종에서는 예외로 같은 이름의 단독 주소와 같은 값을 내고, 위치를 볼 수 없는 Mitsubishi 에서는 `null` 입니다). **매번 같은 순서**로 돌려주므로 두 번 읽어 비교하는 것이 의미를 갖습니다. 없는 공구 영역을 지정하면 상태 `-18` 로 거절됩니다.

**위치 세 필드는 공구가 움직일 때마다 바뀌고**, 나머지 필드는 잘 바뀌지 않습니다. 이 목록은 화면을 그릴 때 한 벌 받아오는 용도이며, 지금 활성인 공구만 알고 싶다면 목록을 반복해 읽는 대신 `/machine/channel/activeToolNumber` 를 쓰세요 (`activeToolNumber` 는 모든 기종이 지원하며, Siemens 에서 그 값은 교환이 끝난 공구이고, 공구관리 옵션이 켜진 Fanuc 에서는 이 목록의 `toolNumber` 가 아니라 공구의 타입 번호입니다).

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 표에서 등록 표시(공구 정보의 RGS 비트, 조작반 `T-INFO` 끝의 `R`)가 선 칸만 담고, `toolNumber` 는 그 칸의 번호(조작반 `NO.` 열, 파라미터 `13220` 까지)입니다. `toolName` 은 빈 문자열이고, `toolEdgeCount` 와 `sisterToolNumber` 는 Fanuc 에 그 개념(날 계층·자매공구)이 없어 `null` 입니다. 프로그램이 `T` 로 부르는 번호는 이 번호가 아니라 공구의 **타입 번호**(조작반 `TYPE NO.`)이며 항목의 `toolTNumber` 가 그것입니다. 위치 세 필드는 단독 주소와 같은 값입니다 (`1`~`4` 매거진, 스핀들·대기 위치는 `"buffer"`, 미장착은 `"none"`). 옵션이 없는 Fanuc 은 오프셋 테이블이 `1` 부터 촘촘히 채워져 있어 열거할 대상이 없습니다.

**Mitsubishi 는 공구관리 표의 등록 행**을 담습니다. 공통 키 중 이 기종이 갖지 않거나 SDK 로 볼 수 없는 개념(`toolTNumber`·`toolName`·`toolEdgeCount`·`sisterToolNumber`, 그리고 공구 레코드가 자기 위치를 담지 않아 위치 세 필드)은 `null` 이고, 반대로 표의 열 넷(`toolTeethCount`·`toolBodyLength`·`toolBodyDiameter`·`toolOffsetNumber`)은 이 기종만 값을 채웁니다 (읽지 못한 열도 키는 남고 `null` 입니다). 공구가 어느 포켓에 있는지는 이 목록이 아니라 `/machine/toolArea/magazine/pocket/toolNumber` 로 매거진 쪽에서 확인하세요.

## /machine/toolArea/tool/toolExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

그 공구 번호가 **공구표에 등록되어 있는지** 여부입니다. `toolArea` + `tool` 필터. 반환 `boolean`. 읽기·쓰기 모두 세 기종을 지원합니다.

없는 공구를 물어도 에러가 아니라 `false` 입니다. 존재 여부를 묻는 주소이기 때문입니다.

**쓰기가 공구를 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. **이미 있는 공구에 `true` 를 쓰면 상태 `-21`(이미 존재)로 거절합니다.** 조용히 성공시키면 빈 새 공구인 줄 알고 남의 수명·오프셋·자리 위에 값을 이어 쓰게 되기 때문입니다. 지우고 다시 만들거나, 그대로 각 주소로 값을 쓰거나, 건너뛰세요. 없는 공구에 `false` 를 쓰는 것은 성공입니다 (사후 조건 "없다" 가 그대로 성립하고, 응답을 못 받아 다시 보내도 안전합니다).

**공구 번호는 `tool` 필터에 넣은 값 그대로입니다.** 장비가 다음 번호를 자동으로 붙여 주지 않습니다. 번호 공간에는 구멍이 있고(실측 장비는 공구 20개가 `2`~`18` 과 `100`~`102` 를 쓰며 `1` 번과 `19`~`99` 가 비어 있습니다) **그 빈 번호를 지정해 채울 수 있습니다.** 어느 번호가 비었는지는 `/machine/toolArea/toolList` 의 `toolNumber` 들을 보고 고르세요. 공구를 지우면 그 번호가 다시 비고, 나중에 같은 번호로 다시 만들 수 있습니다.

Siemens 에서 만들어지는 공구는 **날 1개짜리 빈 공구**입니다. 이름은 공구 번호 문자열이고, **자리 종류**는 표준값으로 채워집니다. 자리 종류는 그 공구가 들어갈 수 있는 매거진 자리를 정하는 값이라 비워 두면 조작반이 적재할 자리를 찾지 못하므로, 디메시가 조작반이 새 공구에 넣는 값과 같게 채웁니다. 공구 타입은 채우기 전까지 `9999`(미지정)이고 조작반도 종류 칸을 비워 둔 것으로 보여 주는데, **자리 종류와 달리 적재나 사용을 막지는 않습니다** (두 칸 모두 `9999` 를 쓰기 때문에 혼동하기 쉽습니다). 이어서 `/machine/toolArea/tool/toolName` 으로 이름을, `/machine/toolArea/tool/toolEdge/*` 로 오프셋을 넣고, 날을 더 붙이려면 `/machine/toolArea/tool/toolEdge/toolEdgeExists` 를 쓰세요. 공구 준비실에서 잰 값을 조작반을 거치지 않고 그대로 등록하는 흐름이 이것입니다.

**Siemens·Fanuc 에서는 매거진 포켓을 차지한 공구를 지울 수 없습니다** (상태 `-18`. Fanuc 은 스핀들·대기 위치에 있는 공구도 같습니다). 실물은 매거진에 남는데 등록만 사라지면 다음 공구 교환이 어긋나고, 되돌리려 해도 **디메시에는 공구를 그 포켓에 다시 배정하는 주소가 없습니다.** 먼저 조작반에서 공구를 빼세요. 지금 어디에 있는지는 `/machine/toolArea/tool/toolLocationType` 이 답합니다. **Mitsubishi 에서는 디메시가 이 검사를 하지 않습니다** (그 기종은 공구 레코드가 자기 위치를 담지 않아, 매거진 전체를 훑어야 알 수 있습니다). 매거진을 쓰는 장비라면 지우기 전에 `/machine/toolArea/magazine/pocket/toolNumber` 로 그 공구가 어느 포켓에 있는지 직접 확인하세요.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 그 칸의 등록 표시(공구 정보의 RGS 비트, 조작반 `T-INFO` 끝의 `R`)입니다. 파라미터 `13220`(칸 수)을 넘는 번호도 읽기는 에러가 아니라 `false` 입니다. 등록이 풀린 칸은 값이 남아 있어도 제어기가 무효로 보므로(Connection Manual B-64483EN-1 §12.3.1 이 RGS 가 0 이면 다른 항목에 값이 있어도 미등록으로 취급한다고 밝힘), 그 칸에 대한 다른 공구 주소(`toolHNumber`·수명·위치 등)는 상태 `-18` 로 거절됩니다 (벤치 실측: 등록 표시만 켜면 같은 칸이 곧바로 `true` 가 되고 `toolCount` 도 하나 늘었습니다).

Mitsubishi 의 표는 **행의 목록**이라 공구 번호가 곧 행이 아닙니다. `true` 는 **첫 번째 빈 행**에 그 번호를 써 넣고, `false` 는 그 행을 비웁니다. 행 번호는 주소 표면에 나오지 않으므로 어느 자리에 들어가는지 신경 쓸 필요가 없습니다. 만들어진 공구는 날 수·치수가 `0` 이고 **보정 번호만 공구 번호와 같은 값으로 제어기가 채워 줍니다** (시뮬레이터에서 확인). 나머지는 각 주소로 채우세요. 지운 행은 제어기가 통째로 비우므로(조작반의 `공구 클리어` 와 같습니다) 나중에 그 자리에 만들어진 공구가 옛 값을 물려받지 않습니다.

**Mitsubishi 에서 만들기와 없는 공구 지우기는 표 전체를 훑습니다** (시뮬레이터에서 2초 남짓). 표가 999행이고 중간을 지우면 구멍이 남아, "이 번호가 없다" 를 증명하려면 끝까지 봐야 하기 때문입니다. 있는 공구를 지우는 것은 그 행에서 멈춰 수십 밀리초입니다. **되풀이해 부르는 용도가 아닙니다.** 중복 번호는 제어기도 거절하지만 디메시가 먼저 보고 상태 `-21` 로 답합니다. 표에 빈 행이 하나도 없으면 상태 `-23`(자리 없음)으로 거절합니다. 값이 잘못된 것이 아니라 넣을 자리가 없는 것이므로, 공구 하나를 지워 자리를 비운 뒤 다시 요청하세요.

Fanuc 의 쓰기: `true` 는 그 칸을 **등록**합니다 (`cnc_regtool`). 만들어지는 칸은 **등록 표시만 켜진 빈 레코드**(타입 번호 `0`, 수명 관리 안 함, H/D/S/F `0`)라 어떤 `T` 지령에도 걸리지 않습니다. 제어기가 대신 넣어 주는 기본값은 없고 디메시도 지어내지 않으니, 이어서 `toolTNumber`·`toolHNumber`·`toolDNumber`·`toolLifeMonitorType`·수명 주소로 채우세요. 등록이 풀렸는데 값이 남은 칸(조작반 `T-INFO` 가 `-` 인데 다른 열에 값이 보이는 칸)은 제어기가 그대로 등록을 거절하므로 디메시가 먼저 비우고(`cnc_deltool`) 등록합니다. 남은 값은 버려집니다. `13220` 을 넘는 칸은 만들 수 없어 상태 `-18` 입니다. `false` 는 그 칸을 **삭제**합니다 (`cnc_deltool`): 레코드가 통째로 비워지고, 제어기가 매거진 관리표에서도 그 공구 번호를 지웁니다 (Connection Manual B-64483EN-1 §12.3). 조작반의 편집 잠금(`toolDataLockedOn`)은 이 삭제를 막지 않습니다 (벤치 실측). `13220` 밖에 `false` 를 쓰는 것은 읽기와 같이 에러가 아닙니다.

## /machine/toolArea/tool/toolName
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

공구의 이름입니다 (SINUMERIK `toolIdent`). `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호).

반환 `string`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": "DRILL 10"}`. **Siemens 전용**입니다. 공구 관리 기능을 쓰는 장비에서는 이름과 자매공구 번호(duplo)의 조합이 공구의 정체이므로 같은 이름을 가진 공구가 여럿 있을 수 있습니다. 이름을 쓰지 않는 장비에서는 빈 문자열이 정상입니다.

없는 공구를 지정하면 상태 `-18` 로 거절됩니다. 이 주소는 공구를 만들지 않습니다. 이름의 길이·문자 제약은 장비가 판단하며 위반하면 에러가 돌아옵니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** Fanuc 공구관리 표에는 이름 열이 없고(공구 형상 크기 데이터의 이름은 `/machine/channel/toolOffset/toolName`), Mitsubishi 공구 관리 표에도 없습니다.

## /machine/toolArea/tool/toolUseStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

그 공구의 **사용 상태**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int` + `desc`. 읽기·쓰기 모두 Siemens·Fanuc 을 지원합니다. **공구 단위**라 `toolEdge` 필터를 받지 않습니다 (보정 세트가 여럿인 공구도 상태는 공구 전체에 걸립니다).

값은 디메시가 정한 기종 무관 코드입니다 (벤더 번호가 아닙니다). 각 값은 **그 공구가 지금 어떤 상태인가**로 정의하고, 두 기종이 같은 상태일 때만 같은 값을 냅니다:

| 값 | 뜻 |
|---|---|
| `0` | 수명 관리 밖: 수명 상태가 "관리 안 함" 이라 타입 번호 검색에서 빠지는 공구 (Fanuc 전용. `/machine/toolArea/tool/toolSearchedWhenUnmanagedOn` 이 그 예외) |
| `1` | 미사용: 아직 절삭한 적 없고 잠기지 않음 |
| `2` | 사용 중: 쓰인 적 있고 잠기지 않음 |
| `3` | 수명 초과: 수명이 소진되어 제어기가 쓰지 않음 |
| `4` | 파손: 파손으로 제어기가 쓰지 않음 (Fanuc 전용) |
| `5` | 잠금: 수명이 남았는데 사람이 쓰지 말라고 표시함 (Siemens 전용) |

`3`·`4`·`5` 면 제어기가 그 공구를 쓰지 않습니다. 프로그램이 부르면 거절하거나, 자매공구가 등록되어 있으면 그쪽으로 넘어갑니다 (`/machine/toolArea/tool/sisterToolNumber`). 한 기종에서만 나오는 값이 있어도 그 값이 나올 때의 뜻은 같습니다. `desc` 는 사람이 읽는 문구라 분기는 값으로 하세요.

**Fanuc** 은 공구관리 데이터의 수명 상태(조작반 `L-STATE`)를 그대로 옮깁니다: 관리 안 함 `0` · 미사용 `1` · 사용 가능 `2` · 수명 초과 `3` · 파손 `4`. 작업자가 조작반에서 손으로 잠근 공구도 Fanuc 제어기는 `수명 초과`(`3`)로 부르므로 `5` 는 나오지 않습니다. 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만 지원하며, 없으면 상태 `-20` 입니다. 공구 정보의 LOC 비트(`/machine/toolArea/tool/toolDataLockedOn`)는 데이터 편집 잠금이라 여기 섞지 않습니다.

**Siemens** 는 공구 상태 비트(`toolState`)와 잔여 수명에서 도출합니다: 잠금 비트(Disabled)가 꺼져 있으면 "쓰인 적 있음" 비트에 따라 `1`/`2`, 켜져 있으면 수명 감시 중이고 어느 날이든 잔여 수명이 `0` 이하면 `3`, 아니면 `5` 입니다. 잠긴 공구는 잔여 수명을 읽기 위해 왕복이 한 번 더 듭니다. SINUMERIK 의 공구 상태에는 파손 구분이 없어 `4` 는 나오지 않고, 수명 감시가 꺼진 공구도 선택되므로 `0` 도 나오지 않습니다.

**쓰기**는 원하는 상태를 값으로 지정합니다. 이미 그 상태면 아무것도 하지 않고 성공합니다. 기종마다 쓸 수 있는 값이 다릅니다:

- Fanuc: `1`~`4` 를 수명 상태에 씁니다 (`cnc_wrtool2`). `3` 으로 바꾸면 같은 타입 번호의 공구가 전부 수명 초과가 되는 순간 공구 교환 신호(`TLCH`)가 켜질 수 있습니다. 수명 상태가 "관리 안 함" 인 공구는 상태 `-18` 로 거절합니다 (먼저 `/machine/toolArea/tool/toolLifeMonitorType` 을 `1`/`2` 로). `0` 은 `toolLifeMonitorType` 으로 다루고, `5` 는 Fanuc 에 없는 상태라 상태 `-16` 입니다 (`3` 을 쓰세요).
- Siemens: `5` 는 잠금 비트를 켜고, `1`/`2` 는 잠금 비트를 끄면서 "쓰인 적 있음" 비트를 각각 끄고 켭니다. `3` 은 제어기가 잔여 수명에서 도출하는 사실이라 직접 쓸 수 없어 상태 `-16` 입니다 (`/machine/toolArea/tool/toolEdge/toolLifeRemaining` 을 `0` 으로 쓰거나, 잠그려면 `5`). `4`·`0` 도 상태 `-16` 입니다.

**수명이 다해 `3` 이 된 공구를 `2` 로만 되돌리면 곧 다시 `3` 이 됩니다.** 잔여 수명(Fanuc 은 카운터)이 그대로이기 때문입니다. 인서트를 갈았다면 `/machine/toolArea/tool/toolLifeUsed`(Fanuc) 또는 `/machine/toolArea/tool/toolEdge/toolLifeRemaining`(Siemens)을 먼저 되돌리세요. 인서트를 갈지 않은 채 상태만 되돌리는 것은 다 쓴 날로 깎는다는 뜻이기도 합니다.

없는 공구를 지정하면 상태 `-18` 로 거절됩니다.

이 주소는 1.1.0 의 `/machine/toolArea/tool/toolDisabledOn`(`boolean`)을 대체합니다. 옛 주소는 상태 `-12` 로 거절되며, `toolDisabledOn` 이 `true` 였던 공구는 이 주소의 `3`·`4`·`5` 에 해당합니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/tool/toolTNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

프로그램이 **`T` 로 이 공구를 부르는 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 10}`.

`T10 M06` 의 그 `10` 입니다. `/machine/toolArea/tool/toolHNumber`(`H`)·`toolDNumber`(`D`) 와 같은 식구로, 프로그램의 글자에 대응하는 번호입니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 공구관리 데이터의 타입 번호(조작반 TOOL MANAGER 화면의 `TYPE NO.`)입니다. **공구 번호가 아니라 묶음 꼬리표입니다**: 여러 공구가 같은 번호를 가질 수 있고, 프로그램이 그 번호를 부르면 제어기가 그 번호를 가진 공구 중 잔여 수명이 가장 적은 유효한 공구를 골라 씁니다 (같으면 스핀들 위치, 대기 위치, 매거진 순, 그다음 공구 번호가 작은 것. Connection Manual B-64483EN-1 §12.3.1). 테스트 벤치는 공구 `1` 이 `4`, 공구 `2`~`5` 가 전부 `10` 이었습니다. 공구마다 다른 번호를 매긴 장비에서는 공구 번호처럼 보이지만 그건 운용 방식일 뿐입니다. 쓰기는 이 공구의 타입 번호를 바꿉니다 (`0`~`99999999` 의 정수, 밖이면 상태 `-16`). 그 번호를 가진 다른 공구들과 묶이거나 풀리므로 프로그램의 `T` 가 고르는 후보가 달라집니다.

**`/machine/channel/activeToolNumber` 와 짝입니다.** 공구관리 장비에서 그 주소가 내는 값은 이 번호이므로, `/machine/toolArea/toolList` 에서 `toolTNumber` 가 그 값과 같은 항목들이 후보 공구입니다. 어느 것이 실제로 스핀들에 물렸는지는 `/machine/toolArea/tool/toolLocationType` 의 `"buffer"` 로 좁힐 수 있습니다.

공구의 **종류**(드릴·엔드밀 등)가 아닙니다. 그것은 `/machine/toolArea/tool/toolEdge/toolType` 이고 Fanuc 에서는 별도 옵션의 형상 표에 있습니다.

**Siemens 는 상태 `-20` 입니다.** 프로그램이 공구를 이름으로 부르는 제어기라 같은 자리는 `/machine/toolArea/tool/toolName` 입니다. 등록되지 않은 공구는 상태 `-18` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/tool/toolHNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

프로그램에서 이 공구의 **길이 보정**을 부를 때 쓰는 `H` 번호입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기·쓰기 모두 지원하며 쓰기는 `{"value": 5}` 형식입니다.

`G43 H5` 처럼 프로그램이 길이 보정을 지정할 때의 그 `5` 이고, 공구가 가리키는 **공구 보정 표의 행 번호**입니다. 그 행의 길이 형상·마모를 읽으려면 `/machine/channel/toolOffset/toolLengthGeometry` 와 `/machine/channel/toolOffset/toolLengthWear` 에 `toolOffset` 필터로 이 번호를 넣으세요. **여러 공구가 같은 번호를 공유해도 됩니다** (테스트 벤치는 공구 `2`·`3`·`5` 가 `H=3` 을 함께 씁니다). `0` 은 배정되지 않음입니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 공구관리 데이터의 `H` 입니다 (조작반 EACH TOOL DATA 화면의 `H`). Fanuc 에서는 이 번호가 날이 아니라 **공구에 하나** 붙으므로 `toolEdge` 필터 없이 공구 단위로 둡니다. 등록되지 않은 공구는 상태 `-18` 입니다.

**쓰기는 그 공구가 가리키는 보정 행을 옮기는 것이라 적용되는 보정값 자체가 달라집니다** (테스트 벤치에서 `D` 를 `5`→`6` 으로 바꾸니 조작반의 `GEOM(D)/RAD` 가 `35.000`→`45.000` 으로 바뀌었습니다. `H` 도 같습니다). `0`~`999` 의 정수만 받고 밖이면 상태 `-16` 입니다. 보정값 자체를 바꾸려면 `/machine/channel/toolOffset/*` 에 쓰세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** Siemens 의 ISO 방언 `H` 는 날에 배정되는 번호라 `/machine/toolArea/tool/toolEdge/toolHNumber` 에 있습니다.

## /machine/toolArea/tool/toolDNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

프로그램에서 이 공구의 **반경 보정**을 부를 때 쓰는 `D` 번호입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기·쓰기 모두 지원하며 쓰기는 `{"value": 6}` 형식입니다.

`G41 D6` 처럼 프로그램이 공구경 보정을 지정할 때의 그 `6` 이고, **공구 보정 표의 행 번호**입니다. 그 행의 반경 형상·마모를 읽으려면 `/machine/channel/toolOffset/toolRadiusGeometry` 와 `/machine/channel/toolOffset/toolRadiusWear` 에 `toolOffset` 필터로 이 번호를 넣으세요. **같은 공구의 `H` 와 `D` 가 서로 다른 번호여도 됩니다** (길이는 `H` 가 가리키는 행에서, 반경은 `D` 가 가리키는 행에서 옵니다). 여러 공구가 같은 번호를 공유하는 것도 정상입니다. `0` 은 배정되지 않음이며 공구경 보정을 쓰지 않는 공구는 흔히 `0` 입니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 공구관리 데이터의 `D` 입니다 (조작반 EACH TOOL DATA 화면의 `D`). 날이 아니라 **공구에 하나** 붙는 번호라 `toolEdge` 필터 없이 공구 단위로 둡니다. 등록되지 않은 공구는 상태 `-18` 입니다.

**쓰기는 이 공구가 가리키는 보정 행을 옮기는 것이라 적용되는 반경 보정값 자체가 바뀝니다.** `0`~`999` 의 정수만 받고 밖이면 상태 `-16` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** Siemens 의 `D` 는 반경 보정이 아니라 보정 세트(날) 번호라 이 주소가 성립하지 않고, 그쪽에서는 반경값을 `/machine/toolArea/tool/toolEdge/toolRadiusGeometry` 로 바로 읽으면 됩니다.

## /machine/toolArea/tool/toolOffsetNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

그 공구가 쓰는 **보정 번호**입니다 (**Mitsubishi 전용**. 공구관리 표의 열이라 다른 두 기종은 상태 `-20` 입니다). `toolArea` + `tool` 필터. 반환 `int`, 읽기와 쓰기가 됩니다.

**공구 번호로 그 공구의 보정값에 닿는 고리입니다.** 이 값을 `/machine/channel/toolOffset/…` 의 `toolOffset` 필터에 그대로 넣으면 형상·마모·노즈R을 읽을 수 있습니다.

```
toolList -> 공구 5  ->  toolOffsetNumber?tool=5 -> 5  ->  toolOffset/toolXGeometry?toolOffset=5
```

**공구 번호와 다를 수 있습니다.** 우연히 같은 장비가 많지만 별개 값입니다 (시뮬레이터에서 확인: 공구번호를 `7` 로 바꿔도 보정 번호는 `5` 로 남았습니다).

⚠️ **쓰면 그 공구에 적용되는 보정값이 통째로 바뀝니다.** 값 하나를 고치는 것이 아니라 어느 보정을 볼지를 갈아 끼우는 것이라, 가공 중인 공구에 쓰면 그 자리부터 다른 치수로 움직입니다. `0` 도 받습니다 (제어기가 받아들이는 값이며, 뜻은 그 기계의 설정을 따릅니다). 상한은 `/machine/channel/toolOffsetCount` 이고 그보다 큰 번호는 제어기가 거절해 상태 `-16`(잘못된 쓰기 값)이 나갑니다. 없는 공구 번호는 상태 `-18`(잘못된 필터 값)입니다.

Mitsubishi 조작반의 공구관리 표에는 보정 열이 두 벌인데(`X5` / `Y5`) **한 번 쓰면 두 열이 함께 바뀝니다** (시뮬레이터에서 확인). 그 화면은 자동으로 갱신되지 않으므로 확인하려면 다른 화면에 갔다가 돌아와야 합니다.

**Fanuc·Siemens 는 상태 `-20` 입니다.** 두 기종은 공구 번호와 보정 번호를 잇는 열이 따로 없습니다 (Fanuc 은 `toolHNumber`·`toolDNumber`, Siemens 는 공구 자체가 보정값을 가집니다).

## /machine/toolArea/tool/sisterToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**자매공구 번호**입니다 (SINUMERIK `duploNo`, 조작반의 `ST` 열). 이름이 같은 공구들을 구분하는 번호이며, 앞선 공구의 수명이 다했을 때 어느 것이 대체 투입될지를 정합니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호).

**`1` 부터 이어지는 순번이 아닙니다.** 실측 장비에서 그 이름의 공구가 하나뿐인데 이 값이 `2` 인 공구, `5` 인 공구, `101` 인 공구가 있었습니다. 번호로 정렬하거나 `1` 이 반드시 있다고 가정하지 마세요.

**새로 만든 공구는 이 값이 공구 번호와 같게 시작합니다** (실측: `50` 번으로 만든 공구의 자매번호가 `50`). 자매공구를 쓸 생각이면 만든 뒤 이 주소로 원하는 값을 넣으세요. 그대로 두어도 동작에는 지장이 없지만, 같은 이름의 공구를 나중에 추가할 때 번호가 뒤죽박죽으로 보입니다.

⚠️ **조작반에서 바로 옆 `D` 열과 헷갈리기 쉽습니다.** `ST` 는 **어느 공구**인가, `D` 는 그 공구의 **어느 날**인가입니다. 이름이 같은 줄이 여러 개 보일 때 `ST` 가 같으면 공구 한 자루의 날 여러 개이고, `ST` 가 다르면 서로 다른 공구입니다.

반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 2}`. **Siemens 전용**입니다. 공구 관리 기능을 쓰는 장비에서는 공구 이름과 이 번호의 조합이 공구의 정체이므로, 이름이 같은 공구가 여럿일 때 이 번호로 구분합니다.

없는 공구를 지정하면 상태 `-18` 로 거절됩니다. 값이 정수가 아니거나 `0`~`65535` 를 벗어나면 상태 `-16` 입니다. 실제 유효 상한은 장비 설정이 정하며, 그보다 좁은 범위를 벗어난 값은 장비가 거절합니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 이름이 같은 공구를 번호로 구분하는 자매공구는 Siemens 공구 관리의 개념이고, Fanuc 의 대체 공구는 공구수명관리 그룹(`/machine/toolArea/toolGroup/…`)이 맡습니다.

## /machine/toolArea/tool/toolTeethCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

공구의 **날 수**입니다 ("4날 엔드밀" 이라 할 때의 그 수). `toolArea` + `tool` 필터. 반환 `int`, 읽기와 쓰기가 됩니다.

`toolEdge` 밑에도 같은 이름이 있습니다. 그쪽은 **날마다 값을 갖는** 제어기(Siemens)의 자리이고, 이쪽은 **공구 하나에 하나**인 제어기(Mitsubishi)의 자리입니다. 쓰는 기종이 갈리므로 한 장비에서 둘 다 답하는 일은 없습니다 (Fanuc 은 둘 다 상태 `-20` 입니다).

Mitsubishi 는 조작반 `절차 > 툴관리` 의 `Num. of teeth` 입니다. 그 화면은 자동으로 갱신되지 않으므로, 쓴 값을 화면에서 확인하려면 다른 화면에 갔다가 돌아와야 합니다. 쓰기는 그 표에 **이미 등록된 공구**에만 됩니다. 없는 공구 번호는 상태 `-18`(잘못된 필터 값)로 거절하고, 어떤 번호가 있는지는 `/machine/toolArea/toolList` 가 알려줍니다. 값이 그 칸이 받는 범위나 자릿수를 넘으면 제어기가 거절하고 상태 `-16`(잘못된 쓰기 값)이 나갑니다.

## /machine/toolArea/tool/toolBodyLength
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

공구 **몸통의 길이**입니다. `toolArea` + `tool` 필터. 반환 `float`, 읽기와 쓰기가 됩니다.

**보정값이 아닙니다.** 공구 실물의 치수라, 좌표에 더해지는 `toolLengthGeometry` 와는 쓰이는 곳이 다릅니다 (이쪽은 간섭 체크나 매거진 배치에 씁니다). 단위는 그 기계의 설정을 따르므로 붙이지 않습니다.

Mitsubishi 는 조작반 `절차 > 툴관리` 의 `Length :A` 입니다. 그 화면은 자동으로 갱신되지 않으므로, 쓴 값을 화면에서 확인하려면 다른 화면에 갔다가 돌아와야 합니다. 쓰기 규칙은 `toolTeethCount` 와 같습니다 (등록된 공구만, 범위 밖은 상태 `-16`). 소수 셋째 자리까지 전달합니다.

**Fanuc·Siemens 는 상태 `-20` 입니다.** 이 열은 Mitsubishi 공구 관리 표 고유의 것입니다.

## /machine/toolArea/tool/toolBodyDiameter
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

공구 **몸통의 지름**입니다 (반경이 아닙니다). `toolArea` + `tool` 필터. 반환 `float`, 읽기와 쓰기가 됩니다.

`toolBodyLength` 와 같은 부류이고 같은 이유로 단위를 붙이지 않습니다.

Mitsubishi 는 조작반 `절차 > 툴관리` 의 `Diameter :B` 입니다. 그 화면은 자동으로 갱신되지 않으므로, 쓴 값을 화면에서 확인하려면 다른 화면에 갔다가 돌아와야 합니다. 쓰기 규칙은 `toolBodyLength` 와 같습니다.

**Fanuc·Siemens 는 상태 `-20` 입니다.** 이 열은 Mitsubishi 공구 관리 표 고유의 것입니다.

## /machine/toolArea/tool/toolSpindleSpeed
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

공구관리 데이터에 **공구별로 적어 둔 주축 회전수 `S`** 입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int` + `unit:"rpm"`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 1500}` (정수).

**제어기가 `T` 를 불렀을 때 자동으로 적용하는 값이 아닙니다.** 조작 설명서(B-64484EN §10)는 이 값을 공구관리 데이터에 등록한 가공 조건으로 설명하며, 공구 교환 매크로(예: `M06`)에서 `S#8411` 처럼 코딩해 직접 지정할 수 있다고 안내합니다. 즉 기계 제작사나 작업자가 교환 매크로·가공 프로그램을 "이 공구의 등록 조건으로 돌리도록" 짜 두었을 때만 쓰이는 **참고값**이고, 그렇게 짜지 않은 장비에서는 적혀 있어도 아무 효과가 없습니다. 지금 실제 회전수는 `/machine/channel/spindle/spindleSpeedActual`, 지령값은 `spindleSpeedCommanded` 를 읽으세요.

실측 사례도 있습니다. 실제 교환 매크로가 있는 우리 시험 장비(31i)에서 공구에 `1234` 를 적고 `M06` 교환을 돌렸는데, 교환 뒤에도 지령 S 는 값 `0` 그대로였습니다. 이 값이 실려 가는지는 전적으로 그 장비의 매크로에 달렸으니, 의존하기 전에 그 장비에서 확인하세요.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 조작반 EACH TOOL DATA 화면의 `S` 칸이며 FOCAS 스펙의 유효 범위는 `1`~`99999` 라 `0` 은 적어 두지 않은 칸입니다 (테스트 벤치는 전부 `0`). 등록되지 않은 공구는 상태 `-18` 입니다. 쓰기는 `0`~`99999` 의 정수만 받고(밖이면 상태 `-16`), 그 밖의 거절은 벤더 사유와 함께 상태 `-17` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 데이터에 이런 칸이 없습니다.

## /machine/toolArea/tool/toolFeed
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

공구관리 데이터에 **공구별로 적어 둔 절삭 이송 `F`** 입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 250}` (정수).

**제어기가 `T` 를 불렀을 때 자동으로 적용하는 값이 아닙니다.** `/machine/toolArea/tool/toolSpindleSpeed` 와 같은 부류로, 기계 제작사의 교환 매크로가 `F#8412` 로 읽어 쓰도록 적어 두는 **참고값**입니다 (조작 설명서 B-64484EN §10). 그렇게 짜지 않은 장비에서는 적혀 있어도 효과가 없습니다. 지금 실제 이송은 `/machine/channel/feedActual`, 지령값은 `feedCommanded` 를 읽으세요.

실측 사례도 있습니다. 실제 교환 매크로가 있는 우리 시험 장비(31i)에서 공구에 `567` 을 적고 `M06` 교환을 돌렸는데, 교환 뒤에도 지령 F 는 이 값으로 바뀌지 않았습니다 (이전 모달 그대로). 이 값이 실려 가는지는 전적으로 그 장비의 매크로에 달렸으니, 의존하기 전에 그 장비에서 확인하세요.

**`unit` 을 붙이지 않습니다.** FOCAS 스펙이 이 칸의 단위를 mm/min·inch/min·deg/min·mm/rev·inch/rev 로 다 허용하고, 어느 단위로 적었는지는 그것을 읽는 매크로가 정하기 때문입니다 (다른 이송 주소가 기계 설정 때문에 단위를 붙이지 않는 것과 같습니다).

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 조작반 EACH TOOL DATA 화면의 `F` 칸이며 FOCAS 스펙의 범위는 `0`~`99999999` 입니다 (테스트 벤치는 전부 `0`). 등록되지 않은 공구는 상태 `-18` 입니다. 쓰기는 `0`~`99999999` 의 정수만 받고(밖이면 상태 `-16`), 단위는 쓰는 쪽이 그 장비의 매크로와 맞춰야 합니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 데이터에 이런 칸이 없습니다.

## /machine/toolArea/tool/toolDataLockedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 공구의 **공구관리 데이터가 편집 잠금되어 있는지** 여부입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `boolean`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": true}` 로 잠그고 `{"value": false}` 로 풉니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 공구 정보의 LOC 비트(조작반 `T-INFO` 의 `L`/`U`, 벤더 문서의 "Data access: Locked/Unlocked")입니다. **공구를 쓰지 말라는 뜻이 아닙니다** (그건 `/machine/toolArea/tool/toolUseStatus`): 그 공구의 공구관리 데이터를 **고치지 못하게** 하는 보호이고, 제어기의 공구 검색·교환에는 영향이 없습니다.

**이 잠금은 SDK 쓰기를 막지 않습니다.** 테스트 벤치에서 잠근 상태로 수명 카운터·예고 수명·H 번호를 FOCAS 로 썼더니 모두 성공했습니다. 잠금이 막는 것은 조작반 편집 같은 다른 경로이며, 그쪽이 실제로 막히는지는 벤치를 조작할 수 없어 확인하지 못했습니다. 키 보호(파라미터 `13204#0`)가 켜진 장비는 이 쓰기 자체가 벤더 거절로 돌아올 수 있습니다 (쓰기금지 사유면 상태 `-22`, 그 밖은 상태 `-17`. 어느 쪽인지는 확인하지 못했습니다).

쓰기는 공구 정보 워드의 이 비트 하나만 바꾸고 나머지 비트는 읽은 그대로 되씁니다. 이미 그 상태면 아무것도 하지 않고 성공합니다. 등록되지 않은 공구는 읽기·쓰기 모두 상태 `-18` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 데이터에 이런 잠금이 없습니다.

## /machine/toolArea/tool/toolSearchedWhenUnmanagedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**수명 관리를 하지 않는 공구라도 `T` 검색 대상에 넣을지** 여부입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `boolean`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": true}` / `{"value": false}`.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 공구 정보의 SEN 비트(조작반 `T-INFO` 의 `S`/`-`)입니다. 제어기는 프로그램이 `T` 로 타입 번호를 부르면 그 번호의 공구 중 하나를 고르는데, **수명 상태가 관리 안 함(`L-STATE` `NO-MNG`)인 공구는 원래 후보에서 빠집니다.** 이 값이 `true` 면 그런 공구도 잔여 수명을 보지 않고 후보에 넣습니다 (Connection Manual B-64483EN-1 §12.3.1). 수명 관리 중인 공구(`/machine/toolArea/tool/toolLifeMonitorType` 이 `0` 이 아닌 공구)에는 영향이 없습니다.

이 비트의 뜻은 Connection Manual 과 조작 설명서에서 가져왔고, 테스트 벤치(조작반이 이 비트를 `S` 로 표시하고 `cnc_wrtool2` 로 켜고 끌 수 있음)로 확인했습니다.

쓰기는 공구 정보 워드의 이 비트 하나만 바꾸고 나머지 비트는 읽은 그대로 되씁니다. 이미 그 상태면 아무것도 하지 않고 성공합니다. 등록되지 않은 공구는 읽기·쓰기 모두 상태 `-18` 입니다. 기계 제작사의 운용 방침에 속하는 플래그이므로, 바꾸기 전에 그 장비의 공구 교환 절차를 확인하세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 데이터에 이런 플래그가 없습니다.

## /machine/toolArea/tool/toolOversizedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

그 공구가 **포켓 하나보다 큰지** 여부입니다. 양옆 포켓을 비워 둬야 하는 굵은 공구입니다. `toolArea` + `tool` 필터. 반환 `boolean`, **읽기 전용**.

Siemens 의 매거진 화면에서 `Z` 열이 이 값입니다. 몇 포켓을 차지하는지가 아니라 **하나를 넘는지 여부**만 답합니다.

**쓰기는 지원하지 않습니다.** Siemens 는 초과크기를 하나의 플래그가 아니라 위·아래·좌·우로 몇 칸을 차지하는지로 저장하고 있어, `true` 를 받아도 어느 방향으로 몇 칸인지 정할 수 없습니다. 초과크기 지정은 조작반에서 하세요.

없는 공구는 상태 `-18` 로 거절됩니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구 정보의 대형 공구 비트(BDT)이며 읽기 전용입니다. 대형 공구가 차지하는 이웃 포켓은 `/machine/toolArea/magazine/pocketList` 에서 `toolNumber` `0` 으로 나옵니다 (벤치 실측: 비트를 켜니 `true`).

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/tool/toolFixedLocationOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 공구가 **고정 자리로 지정되어 있는지** 여부입니다. 늘 같은 포켓으로 돌아갑니다. `toolArea` + `tool` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원합니다. `{"value": true}` 로 지정하고 `false` 로 해제합니다.

`true` 면 공구 교환 후 원래 포켓으로 돌아가고, `false` 면 장비가 빈 포켓을 골라 넣습니다. 장비의 매거진 화면에서 `L` 열이 이 값입니다.

이미 그 상태면 아무것도 하지 않고 성공합니다. 없는 공구는 상태 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 공구 표에서 이 항목을 읽는 통로가 없습니다.

## /machine/toolArea/tool/toolLifeMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

그 공구의 **수명 감시 방식**입니다. `toolArea` + `tool` 필터. 반환 `int`. 읽기·쓰기 모두 Siemens·Fanuc 에서 지원합니다. 쓰기는 `{"value": 2}`.

| 값 | 뜻 | 수명 값의 단위 |
|---|---|---|
| `0` | 감시 없음 | 없음 |
| `1` | 시간(실제로 깎은 시간을 센다) | 벤더가 주는 단위 그대로: Siemens 는 분(`unit` 은 `"min"`), Fanuc 은 초(`unit` 은 `"s"`) |
| `2` | 개수(완성한 가공물 수를 센다) | 개 (`unit` 은 `"count"`) |
| `3` | 마모(오프셋이 한계까지 밀렸는지 본다) | 기계 설정 (mm/inch) |

**기종이 늘어도 이 넷입니다.** 장비 고유의 번호가 아니라 디메시가 정한 값이라 어느 기종에 붙었는지 몰라도 그대로 분기할 수 있습니다.

Siemens 에서는 방식은 **공구가 하나 고르고, 값은 날마다 따로**입니다. 그래서 이 주소는 `toolEdge` 를 받지 않고, 날 단위 수명 값 3종은 받습니다. Fanuc 은 수명도 공구 단위라 `/machine/toolArea/tool/toolLifeTotal`·`toolLifeUsed`·`toolLifeWarnLimit` 이 짝입니다.

`0` 이면 수명 값 3종이 상태 `-18` 로 거절됩니다. 그 공구에는 잴 것이 없습니다. 감시를 켜려면 이 주소에 방식을 먼저 쓰고, 그 다음 수명 총량을 넣으세요 (Siemens 는 `/machine/toolArea/tool/toolEdge/toolLifeTotal`, Fanuc 은 `/machine/toolArea/tool/toolLifeTotal`. Fanuc 은 조작반의 공구관리 화면에서 수명 상태(`L-STATE`)를 켜도 같습니다).

Siemens 는 여러 방식을 **동시에** 켜 둘 수도 있습니다. 그 경우 이 주소는 시간 → 개수 → 마모 순으로 하나를 골라 답하고, 수명 값 3종도 같은 순서를 따르므로 방식과 값이 어긋나지 않습니다. "갈아야 하나" 의 답인 `/machine/toolArea/tool/toolLifeWarnOn` 과 `/machine/toolArea/tool/toolUseStatus` 는 방식과 무관하게 항상 정확합니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터(`cnc_rdtool`)에서 읽습니다: 수명 상태가 `NO-MNG`(관리 안 함)인 공구는 수명 값이 표시돼 있어도 제어기가 세지 않으므로 `0`, 관리 중이면 공구 정보의 수명 종류 비트에 따라 `1`(시간) 또는 `2`(횟수)입니다. `3`(마모)은 Fanuc 공구관리에 없어 쓰기에서 상태 `-16` 입니다. **쓰기 규칙**: `0` 은 수명 상태를 관리 안 함으로 바꾸고(수명 종류 비트와 값은 그대로 남음), `1`/`2` 는 수명 종류 비트를 시간/횟수로 놓고, 관리 안 함이던 공구는 수명 카운터가 `0` 이면 미사용, 아니면 잔여 있음 상태로 켭니다. 이미 관리 중이면 종류만 바꿉니다 (종류를 바꿔도 숫자는 환산되지 않으니 수명 값을 다시 넣으세요).

이 주소의 예전 이름은 `/machine/toolArea/tool/toolMonitorType` 입니다. 옛 주소 그대로도 동작하지만, 이 문서는 새 이름만 안내합니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/tool/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 공구에 배정된 **수명 총량**(최대 수명)입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 20}`.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 최대 수명(조작반 TOOL MANAGER 화면의 `MAX-LIFE`)이고, 제어기는 사용량 카운터(`/machine/toolArea/tool/toolLifeUsed`)를 `0` 에서 이 값까지 세어 올라갑니다. 잔여는 이 값에서 사용량을 뺀 것입니다. Fanuc 의 수명은 날이 아니라 **공구에 하나** 붙으므로 `toolEdge` 필터 없이 공구 단위로 둡니다.

**단위는 벤더가 주는 그대로입니다.** 감시 방식(`/machine/toolArea/tool/toolLifeMonitorType`)이 시간이면 **초**(`unit` 은 `"s"`, 조작반은 `4H 5M 6S` 처럼 시·분·초로 표시), 횟수면 `count`. 분으로 환산하지 않습니다 (Siemens 날 단위 수명이 분인 것과 다르니 응답의 `unit` 을 보세요). 수명 상태가 관리 안 함(`toolLifeMonitorType` 이 `0`)이면 값이 남아 있어도 상태 `-18` 입니다 (방식을 먼저 쓰세요).

쓰기는 읽기와 같은 단위의 정수만 받으며(소수는 상태 `-16`), 관리 안 함 상태면 상태 `-18` 입니다. 등록되지 않은 공구는 상태 `-18` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** Siemens 의 수명은 날별이라 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 에 있습니다.

## /machine/toolArea/tool/toolLifeUsed
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 공구가 **지금까지 쓴 수명**(수명 카운터)입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0}`.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 수명 카운터(조작반 `L-COUNT`)이며, 제어기가 그 공구가 스핀들에 있는 동안 **`0` 에서 최대 수명(`/machine/toolArea/tool/toolLifeTotal`)을 향해 올려 셉니다** (Connection Manual B-64483EN-1 §12.3.1: 증가 카운터, 잔여 = 최대 − 카운터). 잔여가 필요하면 `toolLifeTotal` 에서 이 값을 빼세요. 잔여 주소를 따로 두지 않는 것은 Fanuc 이 주는 값이 이것이고, 화면(`L-COUNT`)과 같은 숫자를 내기 위해서입니다.

단위는 `toolLifeTotal` 과 같습니다 (시간 감시 초 `"s"`, 횟수 감시 `count`). 수명 상태가 관리 안 함(`toolLifeMonitorType` 이 `0`)이면 상태 `-18` 입니다.

**인서트를 갈고 카운터를 되돌릴 때 이 주소에 씁니다** (보통 `0`). 쓰기는 읽기와 같은 단위의 정수만 받으며(소수는 상태 `-16`), 관리 안 함 상태면 상태 `-18` 입니다. 수명 초과로 잠긴 공구는 카운터를 되돌린 뒤 상태도 되돌려야 쓰입니다 (`/machine/toolArea/tool/toolUseStatus` 참조). 등록되지 않은 공구는 상태 `-18` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** Siemens 는 잔여를 내려 세므로 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 을 보세요.

## /machine/toolArea/tool/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 공구의 **예고 수명**(경고선)입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 30}`.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 예고 수명(조작반 `NOTICE-L`, FOCAS 스펙의 "predictive tool life")으로, 잔여 수명(최대 수명에서 카운터를 뺀 값)이 이 값 이하가 되면 제어기가 수명 도달 예고 신호를 냅니다 (Connection Manual B-64483EN-1 §12.3.1. 예고를 공구 타입 단위로 낼지 공구 단위로 낼지는 파라미터 `13200#3` 이 정하고, 타입 단위일 때 마지막 공구의 잔여를 볼지 같은 타입 공구들의 잔여 합을 볼지는 `13200#2` 가 정합니다). `0` 이면 예고 신호를 내지 않습니다.

단위는 `toolLifeTotal` 과 같습니다. 수명 상태가 관리 안 함(`toolLifeMonitorType` 이 `0`)이면 상태 `-18` 입니다. 쓰기는 읽기와 같은 단위의 정수만 받으며(소수는 상태 `-16`), 관리 안 함 상태면 상태 `-18`, 등록되지 않은 공구는 상태 `-18` 입니다.

Fanuc 에서는 `/machine/toolArea/tool/toolLifeWarnOn` 이 상태 `-20` 입니다 (디메시가 읽을 공구별 예고 도달 플래그를 찾지 못했습니다). 필요하면 `toolLifeTotal − toolLifeUsed` 와 이 값을 비교하세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** Siemens 의 경고선은 날별이라 `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit` 에 있습니다.

## /machine/toolArea/tool/toolLifeWarnOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구가 **경고 한계에 닿았는지** 여부입니다. `toolArea` + `tool` 필터. 반환 `boolean`, **읽기 전용**.

`true` 면 잔여 수명이 경고선 밑으로 내려온 것입니다. **아직 쓸 수 있습니다.** 못 쓰게 된 것은 `/machine/toolArea/tool/toolUseStatus` 의 `3`/`5` 이고, 둘은 독립이라 경고가 `true` 인 채 잠길 수 있습니다 (수명이 다해 잠긴 공구).

곧 교체할 공구를 미리 추리는 용도입니다. 잔여가 얼마나 남았는지는 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 이 답합니다.

**수명 값은 날별인데 이 플래그는 공구 단위입니다.** 재는 곳과 조치하는 곳이 다르기 때문입니다. 깎는 것은 날이지만 교체는 공구째 하고, 제어기가 넘어가는 자매공구(`/machine/toolArea/tool/sisterToolNumber`)도 공구 단위입니다. 그래서 **어느 날이든** 자기 경고선을 넘으면 이 값이 `true` 가 됩니다.

**어느 날이 넘었는지는 알려주지 않습니다.** 필요하면 날마다 `toolLifeRemaining` 과 `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit` 을 비교하세요. 날 개수는 `/machine/toolArea/tool/toolEdgeCount` 가 답합니다.

**제어기가 정하는 값이라 쓰기는 지원하지 않습니다.** 고쳐 써도 잔여 수명은 그대로라 다음 판정에서 되돌아갑니다.

감시가 꺼져 있으면 항상 `false` 입니다. 없는 공구는 상태 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** Fanuc 은 예고 상태를 공구 레코드에 두지 않고(예고값은 `toolLifeWarnLimit`, 예고 신호는 PMC 쪽), Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/tool/toolLocationType
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

그 공구가 **어떤 종류의 자리에 있는지** 나타냅니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `string`, **읽기 전용**. 값이 곧 뜻이라 별도 코드표가 필요 없습니다.

| 값 | 뜻 |
|---|---|
| `"magazine"` | 매거진(공구 저장고)에 꽂혀 있음 |
| `"buffer"` | 스핀들 또는 교환기(가공 중이거나 옮겨지는 중) |
| `"loading"` | 반입·반출 위치 |
| `"none"` | 실물 자리 없음(공구 데이터만 등록되어 있음) |

**기종이 늘어도 이 넷입니다.** 장비는 스핀들·교환기·반입출 위치에도 자기 고유의 매거진 번호(Siemens 는 내부 버퍼 매거진 `9998`·로딩 매거진 `9999`, Fanuc 은 스핀들 위치 `11`~`14`·대기 위치 `21`~`24`, 2경로 이상에서는 경로 번호를 백의 자리에 붙인 `211`·`221` 같은 번호)를 붙이지만 그것은 벤더 상수라 소비자가 알아야 할 이유가 없습니다. 디메시가 이 넷으로 묶어 내보내므로, 어느 기종에 붙었는지 몰라도 값으로 분기할 수 있습니다.

`"buffer"` 는 스핀들과 교환기 그리퍼를 **구분하지 않습니다.** 장비가 둘을 같은 자리로 취급하기 때문입니다. 구분이 필요하면 `/machine/channel/activeToolNumber` 와 겹쳐 보되, 기종별로 대조 방법이 다릅니다. Siemens 는 그 값이 교환 완료 후의 공구 번호라 그대로 비교하면 되고, Fanuc 공구관리 장비는 그 값이 타입 번호라 `/machine/toolArea/tool/toolTNumber` 로 후보를 좁힌 뒤 비교하세요.

**쓰기는 지원하지 않습니다.** 자리를 바꾸는 것은 공구 이동의 몫이고, 이 값만 고치면 장부와 실물이 어긋납니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 매거진 번호(카트리지 관리 표의 `1`~`8`. Connection Manual B-64483EN-1 §12.3.1 은 최대 8개를 허용하고, 파라미터로 구성하는 것은 그중 `1`~`4` 입니다)는 `"magazine"`, 스핀들 위치와 대기 위치는 `"buffer"`, 어디에도 실려 있지 않으면 `"none"` 이고, Fanuc 에는 반입출 위치가 없어 `"loading"` 은 나오지 않습니다. 스핀들·대기 위치는 테스트 벤치에 설정되어 있지 않아 벤더 문서 기준입니다. Siemens 에서 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `"none"` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/tool/magazineNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

그 공구가 지금 꽂혀 있는 **매거진(공구 저장고)의 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

매거진에 없으면 `0` 입니다. 스핀들에 물려 가공 중이거나, 교환기가 옮기는 중이거나, 반입·반출 위치에 있거나, 공구 데이터만 등록되고 실물 자리가 없는 경우입니다. 장비는 이런 자리에도 자기 고유 번호(Siemens 는 내부 버퍼 매거진 `9998`·로딩 매거진 `9999`, Fanuc 은 스핀들 위치 `11`~`14`·대기 위치 `21`~`24`, 2경로 이상에서는 경로 번호를 백의 자리에 붙인 `211`·`221` 같은 번호)를 붙이지만 디메시는 그 번호를 내보내지 않고 `0` 으로 뭉뚱그립니다. 벤더 상수라 소비자가 알아야 할 이유가 없습니다.

**지금 있는 곳이지 원래 자리가 아닙니다.** 공구가 스핀들에 물리면 이 값이 `0` 으로 바뀌고 매거진으로 돌아가면 다시 번호가 붙습니다. 원래 어느 자리에서 나왔는지는 이 주소가 답하지 않습니다.

**쓰기는 지원하지 않습니다.** 이 값은 실물 위치의 기록이라 디메시는 쓰기를 열지 않습니다. 장부와 실물이 어긋나면 이후 공구 교환에 영향을 줄 수 있으므로, 위치 변경은 장비의 공구 관리 절차로 하세요.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 매거진 번호(조작반 `MG`)를 그대로 내며, 스핀들 위치와 대기 위치는 매거진이 아니라 `0` 으로 접습니다 (`/machine/toolArea/tool/toolLocationType` 이 `"buffer"`). 스핀들·대기 위치는 테스트 벤치에 설정되어 있지 않아 벤더 문서 기준입니다. Siemens 에서 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `0` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/tool/pocketNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

그 공구가 꽂혀 있는 **매거진 안의 포켓 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

매거진 번호가 아파트의 동이라면 이 값은 호수입니다. 둘을 함께 읽어야 위치가 정해집니다. 매거진에 없으면 `0` 입니다 (스핀들에서 가공 중이거나, 교환기가 옮기는 중이거나, 반입·반출 위치에 있거나, 실물 자리가 없는 경우). 장비가 그런 자리에 붙이는 고유 번호(예: `9998`)는 내보내지 않습니다.

**터렛(선반)의 스테이션도 포켓으로 나타납니다.** 장비가 터렛 위치를 매거진의 포켓으로 모델링하기 때문입니다. 현장에서 "3번 스테이션" 이라 부르는 자리가 이 주소에서는 포켓 `3` 입니다.

**지금 있는 곳이지 원래 자리가 아닙니다.** 공구가 스핀들에 물리면 이 값이 `0` 으로 바뀌고 매거진으로 돌아가면 다시 포켓 번호가 붙습니다.

**쓰기는 지원하지 않습니다.** 이 값은 실물 위치의 기록이라 디메시는 쓰기를 열지 않습니다. 장부와 실물이 어긋나면 이후 공구 교환에 영향을 줄 수 있으므로, 위치 변경은 장비의 공구 관리 절차로 하세요.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 포트 번호(조작반 `POT`)이며, 공구가 스핀들 위치나 대기 위치에 있으면 `0` 입니다. Siemens 에서 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `0` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/tool/originalMagazineNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구가 **돌아갈 매거진의 번호**입니다. `toolArea` + `tool` 필터. 반환 `int`.

**`magazineNumber` 와 짝입니다.** 그쪽은 공구가 **지금 있는** 자리, 이쪽은 **원래 자리**입니다. 공구가 매거진에 꽂혀 있는 동안에는 둘이 같고, **스핀들이나 그리퍼에 올라가 있을 때 갈립니다**: 그때 `magazineNumber` 는 `0`(진짜 매거진 밖이라는 뜻이고 `toolLocationType` 이 어디인지 답합니다)인데 이 주소는 여전히 원래 매거진을 가리킵니다. 조작반 공구 상세의 `Orig. magazine` 이 이 값입니다.

실측 예 (같은 공구, 매거진에 있을 때와 스핀들에 올라갔을 때):

```
                          매거진에 있음   스핀들에 있음
magazineNumber                  1              0
pocketNumber                    3              0
originalMagazineNumber          1              1
originalPocketNumber            3              3
toolLocationType           "magazine"      "buffer"
```

**쓸 데**: 스핀들에 물린 공구가 **어디로 돌아갈지**를 알 수 있어 공구 교환 계획이나 매거진 정리에 씁니다. 지금 어디 있는지만으로는 답이 안 나오는 질문입니다.

자리가 배정되지 않은 공구는 `0` 입니다 (`magazineNumber` 와 같은 규약). 없는 공구를 물으면 상태 `-18` 입니다.

**Siemens 만 답합니다** (`toolMyMag`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 공구 표에는 원래 자리를 기억하는 열이 없습니다 (Fanuc 은 스핀들이나 대기 자리에 오르면 `magazineNumber`·`pocketNumber` 가 `0` 이 되고 `toolLocationType` 만 그 사실을 말합니다).

## /machine/toolArea/tool/originalPocketNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구가 **돌아갈 포켓의 번호**입니다. `toolArea` + `tool` 필터. 반환 `int`.

`originalMagazineNumber` 와 한 쌍이라 동·호수를 이룹니다. 규칙은 그쪽과 같습니다: 매거진에 있는 동안에는 `pocketNumber` 와 값이 같고, 스핀들·그리퍼에 올라가면 `pocketNumber` 는 `0` 이 되는데 이 값은 원래 포켓을 유지합니다. 조작반 공구 상세의 `Orig. location` 입니다.

자리가 배정되지 않은 공구는 `0`, 없는 공구는 상태 `-18` 입니다.

**Siemens 만 답합니다** (`toolMyPlace`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 공구 표에는 원래 자리를 기억하는 열이 없습니다 (Fanuc 은 스핀들이나 대기 자리에 오르면 `magazineNumber`·`pocketNumber` 가 `0` 이 되고 `toolLocationType` 만 그 사실을 말합니다).

## /machine/toolArea/magazineCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 공구 영역의 **매거진 수**입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

**실물 공구 저장고만 셉니다.** 장비 내부에선 스핀들·교환기와 반입출 위치도 매거진으로 취급하지만(그렇게 세면 실측 장비가 `3`) 디메시는 그 자리들을 매거진으로 보지 않습니다. 공구가 거기 있으면 `/machine/toolArea/tool/toolLocationType` 이 `"buffer"` / `"loading"` 으로 답합니다.

**개수는 알려주지만 번호는 알려주지 않습니다.** 번호가 연속이 아니어서 이 값으로부터 유효한 매거진 번호를 유추할 수 없습니다. 번호가 필요하면 `/machine/toolArea/magazineList` 를 쓰세요. 개수만 필요할 때 목록 전체를 받지 않아도 되게 이 주소를 따로 둡니다.

Siemens 에서 공구 관리 기능이 없는 장비는 매거진 자체가 없어 `0` 입니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 옵션 없이는 매거진 데이터 자체가 없으므로 `0` 이라 답하지 않습니다. Fanuc 의 매거진 구성은 파라미터 `13222`/`13227`/`13232`/`13237`(매거진 `1`~`4` 의 포트 수)로 정해지며 포트 수가 `0` 이 아닌 것만 셉니다 (카트리지 관리 표는 번호 `1`~`8` 을 허용하지만 파라미터로 구성되는 것은 이 넷입니다). 스핀들 위치(`11`~`14`)와 대기 위치(`21`~`24`)는 매거진 번호를 갖지만 매거진이 아니라 세지 않습니다. `toolArea` 는 경로 번호지만 Fanuc 의 공구관리 표는 CNC 전역이라 어느 경로로 물어도 같은 값입니다.

**Mitsubishi 는 매거진 번호가 `1`~`5` 로 고정 범위**이고, 그 중 포켓이 하나라도 있는 것만 셉니다. 스핀들과 대기 자리는 이 기종에서 매거진이 아니라 별도 개념이라 애초에 세어지지 않습니다.

## /machine/toolArea/magazineList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 공구 영역의 **매거진 목록**입니다. `toolArea` 필터. 반환 `objectArray`, **읽기 전용**.

**매거진 번호가 연속이라는 보장이 없으므로 이 목록으로 확인하세요.** Siemens 실측 예로 `1`·`9998`·`9999` 였습니다 (Mitsubishi 는 `1`~`5` 범위라 연속이지만, 기종에 상관없이 이 목록을 쓰면 됩니다). Fanuc 은 파라미터로 구성되는 `1`~`4` 중 설정된 것만 나옵니다 (파라미터 `13222`/`13227`/`13232`/`13237` 의 포트 수가 `0` 이 아닌 매거진. 매트릭스형 매거진(파라미터 `13240`)은 `pocketCount` 가 행×열, `13241`×`13242` 등). **Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. `1` 부터 `/machine/toolArea/magazineCount` 까지 세어 올라가면 찾지 못합니다.

각 항목:

| 필드 | 뜻 |
|---|---|
| `magazineNumber` | 매거진 번호. `/machine/toolArea/magazine/pocketCount` 의 `magazine` 필터에 그대로 넣습니다 |
| `pocketCount` | 그 매거진의 포켓 수 |

**실물 공구 저장고만 담습니다.** 장비 내부에선 스핀들·교환기와 반입출 위치도 매거진으로 취급하고 고정 번호(버퍼 `9998`·로딩 `9999`)를 붙이지만, 디메시에서 "매거진" 은 공구 저장고 하나만 뜻합니다. 공구가 그런 자리에 있으면 `/machine/toolArea/tool/toolLocationType` 이 `"buffer"` / `"loading"` 으로 답하고 `/machine/toolArea/tool/magazineNumber` 는 `0` 을 줍니다.

**공구와 그대로 맞물립니다.** `/machine/toolArea/tool/magazineNumber` 가 돌려준 값으로 이 목록의 항목을 찾고, 그 번호를 `pocketCount` 에 그대로 넣을 수 있습니다.

**매거진 이름은 담지 않습니다.** 장비가 이름 필드를 갖고 있지만 현장에서 설정하지 않으면 뜻이 없습니다. 실측 장비에서는 40자리 매거진과 버퍼와 반입출 위치가 **모두 같은 문자열**을 돌려줬습니다. 세 항목에 같은 이름이 붙으면 목록이 고장난 것처럼 보이므로 넣지 않았습니다.

매거진이 하나도 구성되지 않은 장비는 `[]` 이고, 공구관리 옵션 자체가 없는 Fanuc 장비는 상태 `-20` 입니다.

## /machine/toolArea/magazine/pocketCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 매거진의 **포켓 수**입니다. `toolArea` + `magazine` 필터 (`magazine` 은 매거진 번호). 반환 `int`, **읽기 전용**.

**매거진 번호는 연속이 아닙니다.** 유효한 번호는 `/machine/toolArea/magazineList` 가 알려줍니다. `1` 부터 `/machine/toolArea/magazineCount` 까지 세어 올라가는 방식으로는 찾을 수 없습니다.

없는 번호는 상태 `-18` 로 거절됩니다. **스핀들·교환기·반입출 위치의 번호도 거절됩니다.** 장비는 그 자리들에도 매거진 번호를 붙이지만 디메시는 매거진으로 보지 않습니다. 범위·콤마 확장을 지원합니다.

**Fanuc**: 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만 지원하며 없으면 상태 `-20`, 설정되지 않은 매거진 번호는 상태 `-18`(설정된 번호 목록 동봉)입니다. 값은 파라미터 `13222`/`13227`/`13232`/`13237` 이고, 매트릭스형(파라미터 `13240`)은 행×열입니다. 포켓 번호는 `1` 이 아니라 **시작 포트 번호**(파라미터 `13223` 등)부터 이어지므로 포켓 번호의 범위는 `/machine/toolArea/magazine/pocketList` 로 확인하세요.

**Mitsubishi 주의**: 매거진 번호가 고정 범위 `1`~`5` 라 그 밖은 상태 `-18` 이지만, **범위 안의 실재하지 않는 매거진은 거절 대신 `0` 으로 옵니다**. 이 기종의 매거진 존재 판정이 곧 포켓 수 조회라서입니다. 실재 여부가 필요하면 `/machine/toolArea/magazineList` 를 보세요.

## /machine/toolArea/magazine/pocketList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 매거진의 **포켓을 전부, 포켓마다 무엇이 들어 있는지**입니다. `toolArea` + `magazine` 필터. 반환 `objectArray`, **읽기 전용**.

각 항목:

| 필드 | 뜻 |
|---|---|
| `pocketNumber` | 포켓 번호. 빠짐없이 나옵니다. Siemens·Mitsubishi 는 `1` 부터 `/machine/toolArea/magazine/pocketCount` 까지, Fanuc 은 시작 포트 번호(파라미터 `13223` 등, 보통 `1`)부터 포켓 수만큼 |
| `toolNumber` | 그 포켓에 든 공구 번호. **`0` 이면 빈 포켓** |

**다른 주소들이 답하지 못하는 방향입니다.** `/machine/toolArea/tool/pocketNumber` 는 "이 공구가 몇 번 포켓에 있나" 를 답하지만, "몇 번 포켓에 뭐가 있나" 와 "빈 포켓이 어디인가" 는 이 목록만 답합니다.

공구 이름·오프셋은 담지 않습니다. `/machine/toolArea/toolList` 가 번호로 그것들을 주므로 번호로 이어 붙이세요. 포켓마다 이름을 함께 읽으면 40포켓 매거진에서 읽는 값이 두 배가 됩니다.

버퍼(스핀들·교환기)와 반입출 위치의 번호는 상태 `-18` 로 거절됩니다. 디메시는 그 자리들을 매거진으로 보지 않습니다.

**Fanuc**: 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만 지원하며(없으면 상태 `-20`), 매거진 관리 테이블(`cnc_rdmagazine`)을 매거진 통째로 한 번에 읽습니다. `toolNumber` 는 공구관리 데이터 번호(조작반 `NO.` 열, `tool` 필터에 넣는 값)입니다. 대형 공구가 차지한 이웃 포켓과 예약된 원위치 포켓(각각 대형 공구 지원 옵션·확장 B 옵션)도 공구가 든 것은 아니라 `0` 으로 나오므로, 공구를 넣을 빈 자리를 고를 때는 조작반의 매거진 화면도 함께 확인하세요. 매트릭스형 매거진(파라미터 `13240`)도 같은 시작 포트 번호부터 행×열 개수만큼 이어집니다 (Fanuc Connection Manual B-64483EN-1 §12.3.3: 매거진 앞에서 보아 왼쪽 위에서 오른쪽 아래로 번호가 매겨집니다). 테스트 벤치는 체인형이라 매트릭스형은 문서 기준입니다.

**Mitsubishi**: 매거진 번호는 고정 범위 `1`~`5` 이며, 실재하지 않는 매거진은 상태 `-18` 로 거절됩니다.

포켓이 없는 매거진은 `[]` 입니다.

## /machine/toolArea/magazine/pocket/toolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 포켓에 든 **공구 번호**입니다. `toolArea` + `magazine` + `pocket` 필터. 반환 `int`, **읽기 전용**.

**`0` 은 빈 포켓**입니다 (공구 번호는 `1` 부터). 없는 포켓은 `0` 이 아니라 상태 `-18` 로 거절되므로 둘이 섞이지 않습니다. 유효한 포켓 범위는 `/machine/toolArea/magazine/pocketCount` 가 알려줍니다. Fanuc 은 포켓 번호가 시작 포트(파라미터 `13223` 등)부터 이어지므로 범위는 `/machine/toolArea/magazine/pocketList` 로 확인하세요.

포켓을 하나만 볼 때 쓰고, 매거진 전체를 훑을 때는 `/machine/toolArea/magazine/pocketList` 가 왕복 한 번으로 끝냅니다.

버퍼(스핀들·교환기)와 반입출 위치의 번호는 상태 `-18` 로 거절됩니다.

**Fanuc**: 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만 지원하며 없으면 상태 `-20` 입니다. 값은 공구관리 데이터 번호(`tool` 필터에 넣는 값)이고, 대형 공구가 차지한 이웃 포켓과 예약된 원위치 포켓도 `0` 으로 나옵니다 (`/machine/toolArea/magazine/pocketList` 참조). 매트릭스형 매거진도 같은 방식입니다 (포켓 번호 체계는 `/machine/toolArea/magazine/pocketList` 참조).

**쓰기는 지원하지 않습니다.** 포켓의 공구를 고쳐 쓰면 실물은 그대로인 채 장부만 바뀌어, 다음 공구 교환 때 교환기가 엉뚱한 포켓을 집습니다. 공구 이동은 매거진 명령의 몫이며 디메시는 그 명령을 노출하지 않습니다.

## /machine/toolArea/magazine/pocket/pocketDisabledOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 포켓이 **쓰지 말라고 표시되어 있는지** 여부입니다. 손상되었거나 비워 둬야 하는 자리입니다. `toolArea` + `magazine` + `pocket` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원합니다. `{"value": true}` 로 잠그고 `false` 로 풉니다.

`true` 면 장비가 공구를 넣을 자리를 고를 때 이 포켓을 건너뜁니다. 장비의 매거진 화면에서 `D` 열이 이 값이며, 그 화면에서 이 칸은 **포켓을 차지한 공구 행에만** 나타납니다. 포켓의 속성이지 공구의 속성이 아니기 때문입니다.

공구 쪽의 잠금은 `/machine/toolArea/tool/toolUseStatus`(값 `5`) 이고 별개입니다. 포켓이 잠겨도 그 안의 공구는 잠긴 것이 아니며, 다른 자리로 옮기면 다시 쓸 수 있습니다.

이미 그 상태면 아무것도 하지 않고 성공합니다. 없는 포켓과 버퍼·반입출 위치의 번호는 상태 `-18` 로 거절됩니다.

**Siemens 전용**입니다. Fanuc 은 이 정보가 공구관리 확장 B 옵션(`cnc_rdpot_property`)에 있고 디메시는 그 통로를 쓰지 않아 상태 `-20` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/toolGroupCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

공구그룹의 **사용 가능 개수**입니다. `toolArea` 필터. 반환 `int`, 읽기 전용입니다. 그룹 번호는 `1` 부터 이 값까지이므로, 그룹을 훑을 때 상한으로 쓰세요.

**칸 수이지 쓰이고 있는 그룹의 수가 아닙니다.** 대부분의 칸은 비어 있고, 쓰이는 번호도 띄엄띄엄합니다. 양산 장비에서 이 값이 `64` 인데 공구가 등록된 그룹은 `1` 과 `60` 둘뿐이었습니다. 어느 그룹이 쓰이는지는 번호를 훑으며 `/machine/toolArea/toolGroup/toolCount` 가 `0` 보다 큰지 보면 알 수 있습니다.

**이 주소로 옵션 유무를 알 수 있습니다.** 상태 `-20`(미지원)이 아니면 그 장비에 공구수명관리가 있는 것입니다. `/machine/toolArea/toolCount` 가 공구관리(Tool Management)에 대해 같은 구실을 하므로, 두 주소를 한 번씩 읽으면 그 장비가 어느 공구 기능을 갖췄는지 정해집니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/registeredToolGroupList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

**공구가 등록된 공구그룹의 번호** 목록입니다. `toolArea` 필터. 반환 `intArray`, 읽기 전용이며 번호가 작은 순으로 담깁니다.

**여기 담긴 번호를 `toolGroup` 필터에 그대로 넣으면 됩니다.** 계산이 필요 없습니다.

`/machine/toolArea/toolGroupToolCountList` 도 같은 사실을 담고 있지만 그쪽은 **자리로** 표현합니다 (자리 `i` = 그룹 `i+1`). 그룹 하나를 골라 파고들 때는 이 주소를, 칸 전체를 조망할 때는 그쪽을 쓰세요. 둘을 함께 요청해도 장비 왕복은 한 번입니다.

```
registeredToolGroupList   [1, 60]
toolGroupToolCountList    [3, 0, 0, ... , 1, 0, 0, 0, 0]
```

**공구수명관리(tool life management) 옵션이 없거나 그룹 칸이 할당되지 않은 장비는 `[]`** 입니다. `cnc_rdgrpinfo4` 를 쓰므로 그 함수가 없는 구형 제어기에서는 상태 `-20` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/exchangeRequiredToolGroupList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

**공구를 갈아야 하는 공구그룹** 번호 목록입니다. `toolArea` 필터. 반환 `intArray`, 읽기 전용이며 번호가 작은 순으로 담깁니다.

그룹의 공구가 전부 수명을 다하면 제어기가 교환 신호를 올립니다. 그 신호가 뜬 그룹들이 여기 담깁니다. **갈 것이 없으면 `[]`** 이고, 대개 그 상태입니다. 조작반의 `공구그룹교환요구` 표시와 같은 값입니다.

`toolGroupCount` 와 길이가 맞지 않는 것이 정상입니다. 이 주소는 칸마다의 속성이 아니라 **지금 떠 있는 것만** 담습니다 (`/machine/channel/alarmList` 와 같은 성격입니다). 칸 전체를 보려면 `/machine/toolArea/toolGroupToolCountList` 를 쓰세요.

교환할 그룹이 나왔을 때 어느 공구가 문제인지는 `/machine/toolArea/toolGroup/toolLifeStatusList` 가 알려줍니다. `2`(수명 다함)인 자리의 공구를 갈면 됩니다.

**공구수명관리(tool life management) 옵션이 없거나 그룹 칸이 할당되지 않은 장비는 `[]`** 입니다. 갈 그룹이 없다는 뜻은 같습니다. 이 함수를 지원하지 않는 제어기는 상태 `-20` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroupToolCountList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

공구그룹 **칸마다 등록된 공구 수**입니다. `toolArea` 필터. 반환 `intArray`, 읽기 전용입니다.

**자리 `i` 가 그룹 번호 `i+1`** 이고 (배열은 `0`부터, 그룹은 `1`부터), **길이는 언제나 `/machine/toolArea/toolGroupCount` 와 같습니다.** 둘이 같은 출처를 보므로 어긋날 수 없습니다.

한 번의 요청으로 두 가지를 답합니다:

| | 읽는 법 |
|---|---|
| 어느 그룹에 공구가 들어 있나 | `0` 이 아닌 자리 |
| 새 그룹을 어디에 만들 수 있나 | `0` 인 자리 |

`0` 은 **그 그룹에 등록된 공구가 없다**는 뜻입니다. 실측한 장비에서는 그런 칸이 조작반에 `타입: 데이터 없음` 으로 떴으므로 새 그룹을 만들 자리로 보면 되지만, 이 값 자체가 약속하는 것은 공구 수뿐입니다.

그룹 하나만 필요하면 `/machine/toolArea/toolGroup/toolCount` 를 쓰세요. 이 목록은 전체를 한 번에 받는 쪽입니다.

**`cnc_rdgrpinfo4` 를 쓰므로 구형 제어기에서는 상태 `-20`** 입니다. 그 경우에도 `/machine/toolArea/toolGroupCount` 는 정상입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

길이는 `toolGroupCount`(그룹 칸 수)로 고정이라 빈 배열은 나오지 않습니다. 비어 있는 그룹 칸은 `0` 입니다.

## /machine/toolArea/toolGroup/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹에 **등록된 공구의 수**입니다. `toolArea` + `toolGroup` 필터. 반환 `int`, 읽기 전용입니다.

**빈 그룹은 `0`** 입니다. 그룹 번호 자체가 장비의 범위를 벗어나면 상태 `-18` 로 거절되므로, `0` 은 "그런 그룹이 없다" 가 아니라 "그 그룹에 공구가 없다" 는 뜻입니다.

그룹 번호의 상한은 `/machine/toolArea/toolGroupCount` 가 알려줍니다. 다만 그 칸들이 다 쓰이는 것은 아니라, 훑으면 대부분 `0` 이 나옵니다.

**쓸 그룹이 없는 장비는 상태 `-20`** 입니다. 공구수명관리(tool life management) 옵션이 없거나, 옵션은 있으나 그룹 칸이 할당되지 않은 경우입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolNumberList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹에 등록된 **공구 번호 목록**입니다. `toolArea` + `toolGroup` 필터. 반환 `intArray`, 읽기 전용입니다.

**쓰이는 순서대로** 담깁니다. 번호순이 아닙니다. 양산 장비의 한 그룹이 `[16,13,2]` 였는데, `16` 번을 먼저 쓰고 수명이 다하면 `13` 번, 그다음 `2` 번으로 넘어간다는 뜻입니다.

빈 그룹은 `[]` 입니다. 항목 수는 `/machine/toolArea/toolGroup/toolCount` 와 같고, 두 주소를 함께 요청하면 장비 왕복 한 번으로 처리됩니다.

같은 그룹의 `/machine/toolArea/toolGroup/toolHNumberList`·`/machine/toolArea/toolGroup/toolDNumberList`·`/machine/toolArea/toolGroup/toolLifeStatusList` 도 **길이와 순서가 언제나 이 목록과 같습니다.** 같은 자리끼리 짝지으면 한 공구의 정보가 됩니다. 넷을 함께 요청해도 왕복은 하나입니다.

**쓸 그룹이 없는 장비는 상태 `-20`** 입니다. 공구수명관리(tool life management) 옵션이 없거나, 옵션은 있으나 그룹 칸이 할당되지 않은 경우입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolLifeMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹의 **수명 감시 방식**입니다. `toolArea` + `toolGroup` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 2}`.

| 값 | 뜻 | 수명 값의 단위 |
|---|---|---|
| `0` | 감시 없음 (공구가 등록되지 않은 칸) | 없음 |
| `1` | 시간 | 분 (`unit` 은 `"min"`) |
| `2` | 개수 | 회 (`unit` 은 `"count"`) |

`/machine/toolArea/tool/toolLifeMonitorType` 과 **같은 어휘**입니다. 그쪽은 공구마다, 이쪽은 그룹마다 정해집니다. 이 제어기에는 `3`(마모)이 없습니다.

쓰기는 `1`·`2` 만 받습니다. `0` 으로 되돌리는 것은 그룹을 지우는 일이라 이 주소가 하는 일이 아닙니다.

⚠️ **방식을 바꿔도 수명 숫자는 그대로입니다.** `50` 회수인 그룹을 시간으로 바꾸면 그 값은 `50` 분이 됩니다. 값이 환산되지 않으므로, 방식을 바꾼 뒤 `/machine/toolArea/toolGroup/toolGroupLifeTotal` 을 다시 쓰세요.

**공구가 등록된 그룹만 쓸 수 있습니다.** 빈 그룹에 쓰면 상태 `-18` 입니다. 벤더가 받아준다면 그룹 자체가 생기는 것이라, 이 주소가 약속한 "수정" 밖입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolGroupLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹에 배정된 **수명 총량**입니다. `toolArea` + `toolGroup` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 50}`.

**수명은 공구가 아니라 그룹이 가집니다.** 그룹은 서로 바꿔 쓸 수 있는 같은 종류의 공구를 모아 둔 대기줄이고, 그중 한 자루만 돕니다. 그래서 한도가 하나입니다. 도는 공구가 이 한도에 닿으면 다음 공구로 넘어가고 카운터가 `0` 부터 다시 셉니다.

**단위는 그룹마다 다를 수 있어 응답의 `unit` 에 실립니다.** 시간 방식이면 `"min"`, 횟수 방식이면 `"count"`. 값은 **조작반 화면에 뜨는 그대로**이며 초로 환산하지 않습니다. 기계 전체 기본값은 파라미터 `6800#2` 가 정하지만, M 계열은 프로그램에서 그룹마다 따로 지정할 수 있어 디메시가 그룹에 직접 물어 붙입니다.

빈 그룹은 `0` 입니다.

**쓰기는 정수만 받습니다.** 장비 필드가 정수라 소수는 상태 `-16` 으로 거절합니다. 상한도 장비가 정하며(실측 장비는 회수 `65535`, 분 `4300`) 넘으면 상태 `-16` 입니다. **공구가 등록된 그룹만** 쓸 수 있습니다.

**쓸 그룹이 없는 장비는 상태 `-20`** 입니다. 공구수명관리(tool life management) 옵션이 없거나, 옵션은 있으나 그룹 칸이 할당되지 않은 경우입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolGroupLifeUsed
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹이 **지금까지 쓴 수명**입니다. `toolArea` + `toolGroup` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0}` (카운터를 되돌릴 때 씁니다).

**지금 도는 공구 한 자루의 사용량**입니다. 그룹 전체의 누적이 아닙니다. 아직 차례가 오지 않은 공구는 `0` 이고, 수명이 다한 공구는 한도에 닿은 채 넘어간 것입니다.

남은 수명은 `/machine/toolArea/toolGroup/toolGroupLifeTotal` 에서 이 값을 빼면 됩니다. 단위 규칙과 상태 `-20` 조건은 그 주소와 같습니다.

**쓰기 제약**은 `toolGroupLifeTotal` 과 같습니다. 정수만, 장비 상한 이내, 공구가 등록된 그룹만.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolLifeStatusList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹 각 공구의 **수명 상태**입니다. `toolArea` + `toolGroup` 필터. 반환 `intArray`, 읽기 전용입니다.

| 값 | 뜻 |
|---|---|
| `1` | 아직 쓸 수 있음 |
| `2` | 수명 다함 |
| `3` | 건너뜀 |
| `0` | 그 자리에 쓸 공구가 없음 |

**이 주소는 "지금 쓰이는 공구" 를 알려주지 않습니다.** 차례를 기다리는 공구와 지금 도는 공구가 둘 다 `1` 입니다. `1` 은 "쓸 수 있음" 이지 "쓰는 중" 이 아닙니다. 어느 그룹이 쓰이는지는 `/machine/channel/activeToolGroupNumber` 가 답합니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

공구가 없는 그룹은 `[]` 입니다.

## /machine/toolArea/toolGroup/toolHNumberList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹 각 공구의 **길이 보정 번호(H)** 입니다. `toolArea` + `toolGroup` 필터. 반환 `intArray`, 읽기 전용입니다.

이 번호로 `/machine/channel/toolOffset/toolLengthGeometry` 와 `/machine/channel/toolOffset/toolLengthWear` 를 찾아가면 실제 보정값이 나옵니다. 선반에서는 이 값이 항상 `0` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

공구가 없는 그룹은 `[]` 입니다.

## /machine/toolArea/toolGroup/toolDNumberList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹 각 공구의 **반경 보정 번호(D)** 입니다. `toolArea` + `toolGroup` 필터. 반환 `intArray`, 읽기 전용입니다.

이 번호로 `/machine/channel/toolOffset/toolRadiusGeometry` 와 `/machine/channel/toolOffset/toolRadiusWear` 를 찾아가면 실제 보정값이 나옵니다. 선반에서는 이 값이 항상 `0` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

공구가 없는 그룹은 `[]` 입니다.

## /machine/toolArea/toolGroup/currentToolUseOrder
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹에서 **몇 번째 공구가 지금 차례인지**입니다. `toolArea` + `toolGroup` 필터. 반환 `int`, 읽기 전용이며 `1`부터 셉니다. 아직 그 그룹을 한 번도 안 썼으면 `0` 입니다.

이 번호는 조작반 편집 화면의 `번호` 열(`01`·`02`·`03`)과 같고, `/machine/toolArea/toolGroup/toolNumberList` 의 **같은 자리**를 가리킵니다 (`2` 면 목록의 두 번째).

⚠️ **"그 그룹이 지금 돌고 있다" 는 뜻이 아닙니다.** 그룹이 들고 있는 포인터라 오래 남습니다. 실측에서 공구 교환 직후 `1` 이 된 뒤 리셋에도, 전원을 껐다 켜도 `1` 이었습니다. 조작반의 `@`(사용중) 표시를 그대로 재현하려면 `/machine/channel/activeToolGroupNumber` 가 그 그룹일 때만 찍으세요.

**공구수명관리(tool life management) 옵션이 없거나 그룹 칸이 할당되지 않은 장비는 상태 `-20`** 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolUseOrder/toolExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 그룹의 **그 자리에 공구가 있는지** 여부입니다. `toolArea` + `toolGroup` + `toolUseOrder` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원합니다.

없는 자리를 물어도 에러가 아니라 `false` 입니다. 존재 여부를 묻는 주소이기 때문입니다.

**쓰기가 자리를 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. **이미 공구가 있는 자리에 `true` 를 쓰면 상태 `-21`(이미 존재)로 거절합니다.** 없는 자리에 `false` 를 쓰는 것은 성공입니다 (응답을 못 받아 지우기를 다시 보내도 안전합니다).

새로 만든 자리는 공구 번호·H·D 가 모두 `0` 입니다. 이어서 `/machine/toolArea/toolGroup/toolUseOrder/toolNumber` 로 번호를 채우세요.

**빈 그룹에 자리 `1` 을 만들면 그룹 자체가 생깁니다.** 공구그룹을 만드는 별도 주소는 없고 이것이 그 방법입니다. 어느 번호가 비어 있는지는 `/machine/toolArea/toolGroupToolCountList` 의 `0` 인 자리가 알려줍니다. 새 그룹은 수명이 `0` 이고 감시 방식은 장비 기본값이므로, 이어서 `/machine/toolArea/toolGroup/toolLifeMonitorType` 과 `/machine/toolArea/toolGroup/toolGroupLifeTotal` 을 쓰세요.

**마지막 공구를 지우면 그룹도 사라집니다.** 수명과 감시 방식도 함께 지워집니다 (실측 확인).

⚠️ **자리가 밀립니다.** 지우면 뒤 공구들이 한 칸씩 당겨집니다. 그래서 **한 번의 조작으로 다른 자리 주소가 가리키는 대상이 전부 바뀝니다** (`toolNumber`·`toolHNumber`·`toolDNumber`·`toolLifeStatus`·`/machine/toolArea/toolGroup/currentToolUseOrder`). 여러 자리를 다룰 때는 조작 사이에 목록을 다시 읽으세요.

**만들 수 있는 자리는 맨 뒤 하나뿐입니다.** 공구가 3개인 그룹이면 자리 `4` 만 만들 수 있고, 그보다 뒤는 구멍이 생기므로 상태 `-18` 입니다. 중간에 밀어 넣는 조작은 이 주소로 표현되지 않습니다. 중간 자리는 **이미 존재**하므로 `true` 를 써도 할 일이 없습니다. 순서를 바꾸려면 맨 뒤에 만들고 번호들을 다시 쓰세요.

그룹이 가득 차면(`/machine/toolArea/toolGroupToolCountList` 의 그 칸이 장비 상한에 닿으면) 상태 `-18` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolUseOrder/toolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 그룹 **한 자리의 공구 번호**입니다. `toolArea` + `toolGroup` + `toolUseOrder` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 16}`.

`toolUseOrder` 는 그룹 안의 자리(`1`부터)이고 조작반 편집 화면의 `번호` 열과 같습니다. 지금 차례인 자리는 `/machine/toolArea/toolGroup/currentToolUseOrder` 가 알려줍니다.

**목록으로 한 번에 받으려면** `/machine/toolArea/toolGroup/toolNumberList` 를 쓰세요. 같은 값이고 왕복도 같습니다. 이 주소는 **한 자리를 지목해 쓰기 위한** 형태입니다.

**없는 자리는 상태 `-18`** 입니다 (그 그룹의 공구 수가 상한). 벤더가 받아준다면 그건 없던 공구가 생기는 것이라, 이 주소가 약속한 "수정" 밖입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolUseOrder/toolHNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 자리 공구의 **길이 보정 번호(H)** 입니다. `toolArea` + `toolGroup` + `toolUseOrder` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 16}`.

이 번호로 `/machine/channel/toolOffset/toolLengthGeometry` 와 `/machine/channel/toolOffset/toolLengthWear` 를 찾아가면 실제 보정값이 나옵니다. 선반에서는 항상 `0` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolUseOrder/toolDNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 자리 공구의 **반경 보정 번호(D)** 입니다. `toolArea` + `toolGroup` + `toolUseOrder` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 16}`.

이 번호로 `/machine/channel/toolOffset/toolRadiusGeometry` 와 `/machine/channel/toolOffset/toolRadiusWear` 를 찾아가면 실제 보정값이 나옵니다. 선반에서는 항상 `0` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/toolGroup/toolUseOrder/toolLifeStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 자리 공구의 **수명 상태**입니다. `toolArea` + `toolGroup` + `toolUseOrder` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 1}`.

| 값 | 뜻 |
|---|---|
| `1` | 아직 쓸 수 있음 |
| `2` | 수명 다함 |
| `3` | 건너뜀 |

**인서트를 갈고 다시 쓰려면 `1` 을 쓰세요.** `2` 인 공구를 `1` 로 되돌리는 것이 그 조작입니다.

쓰기는 `1`·`2`·`3` 만 받습니다. `0`(공구 없음)은 **삭제**를 뜻하는데 그건 이 주소가 약속한 일이 아니라 상태 `-16` 으로 거절합니다.

**가공 중에는 제어기가 거절할 수 있습니다.** 그 그룹을 지금 쓰고 있거나 다음 차례로 잡아 둔 상태면 장비가 자기를 지키려고 막습니다. 상태 `-17` 에 벤더 사유가 그대로 실리므로 가공이 끝난 뒤 다시 시도하세요.

**읽기 전용 목록**은 `/machine/toolArea/toolGroup/toolLifeStatusList` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 Fanuc 공구수명관리의 계층이라 두 기종에는 같은 표가 없습니다. Siemens 는 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체하고, Mitsubishi 의 수명 관리 표는 지원하지 않습니다.

## /machine/toolArea/tool/toolEdgeCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

공구가 가진 **보정 세트(cutting edge)의 개수**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). Siemens 의 `numCuttEdges` 입니다.

**개수이지 가장 큰 번호가 아닙니다.** 보통은 `1` 부터 이어지지만, 조작반에서 중간 날을 지우면 **번호에 구멍이 생기고 뒤 번호는 밀리지 않습니다.** 예를 들어 `1`·`2`·`3` 중 `2` 를 지우면 남는 것은 `1` 과 `3` 이고 이 값은 `2` 가 됩니다. 그래서 `toolEdge` 를 `1`~이 값으로 가정하면 안 됩니다.

없는 날을 가리키는 `toolEdge/…` 주소는 읽기·쓰기 모두 상태 `-18` 로 거절되므로, 어떤 번호가 실재하는지는 읽어 보면 알 수 있습니다 (이 주소 자체는 읽기 전용입니다).

**인선 개수와는 다른 값입니다.** "2날 볼엔드밀", "4날 엔드밀" 이라 할 때의 그 날은 물리적 인선 수이고, 이 값은 제어기가 그 공구에 대해 갖고 있는 보정 세트의 수입니다. 인선이 여럿이어도 모두 같은 높이·반경이면 보정 세트는 하나면 됩니다. 실측 예로 4날 커터가 `1`, 2날 볼엔드밀이 `3` 을 답했습니다.

없는 공구를 지정하면 상태 `-18` 로 거절됩니다. 보정 세트가 `0` 개라고 답하지 않습니다.

**Siemens 전용**입니다 (Fanuc·Mitsubishi 는 상태 `-20`). 두 기종의 오프셋 모델은 오프셋(세트) 번호 하나가 곧 보정값 한 벌이라 공구에 딸린 날이라는 계층이 없습니다. 예전 판은 고정 `1` 을 냈는데, 없는 차원을 있다고 답하는 값이라 뺐습니다. Fanuc 공구관리의 공구별 데이터(`H`·`D`·수명)는 `/machine/toolArea/tool/*` 의 공구 단위 주소로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolEdgeExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 날 번호가 **그 공구에 있는지** 여부입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원.

없는 날을 물어도 에러가 아니라 `false` 입니다. `/machine/toolArea/tool/toolEdgeCount` 는 개수만 알려주고 번호에 구멍이 있을 수 있으므로, **어떤 번호가 실재하는지는 이 주소가 답합니다.**

**쓰기가 날을 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. 이미 있는 날에 `true` 를 쓰면 상태 `-21`(이미 존재)로 거절하고, 없는 날에 `false` 를 쓰는 것은 성공입니다.

**날은 순서대로만 만들어집니다.** 장비는 **비어 있는 가장 작은 번호**에 날을 만듭니다 (번호 지정 없음). 그래서 그 번호가 아닌 것을 요청하면 만들지 않고 상태 `-18` 로 거절하며, 다음에 만들어질 번호를 에러 문구에 실어 보냅니다. `D5` 가 필요하면 `3`·`4`·`5` 를 차례로 만드세요. 구멍이 있으면 그 구멍부터 채워집니다.

**1번 날은 지울 수 없습니다** (상태 `-18`). 공구가 있는 한 남습니다. 공구째 지우려면 `/machine/toolArea/tool/toolExists` 를 쓰세요.

없는 공구에 날을 만들려 하면 상태 `-18` 입니다.

**Siemens 전용**입니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

공구의 **타입 코드**입니다 (read + write, `int` + `desc`). **SINUMERIK DP1 코드를 그대로 사용**합니다. 코드 체계는 Siemens 소유의 열린 분류라 디메시가 번역하지 않으며, 정본은 SINUMERIK 공구 관리 매뉴얼입니다 (벤더가 코드를 추가해도 값은 그대로 전달). 쓰기는 정수 코드 `{"value": 500}` 입니다. 공구 셋업 자동화용이며 코드 유효성은 NCK 가 판정합니다.

| 계열 | 의미 | 예 |
|---|---|---|
| `1xx` | 밀링 공구 | `120` 엔드밀, `140` 페이스밀, `145` 나사 밀링 |
| `2xx` | 드릴 계열 | `200` 트위스트드릴, `240` 탭, `250` 리머 |
| `4xx` | 연삭 공구 | |
| `5xx` | 선삭 공구 | `500` 황삭, `510` 정삭, `530` 절단, `540` 나사 |
| `7xx` | 특수 | `711` 프로브, `730` 스톱 |

위 표는 **정본이 아니라 길잡이**입니다. 이 코드 체계는 Siemens 가 소유하므로, 정확한 목록은 그 기종 매뉴얼에서 확인하세요: 840D sl 은 *Tool Management Function Manual* §9.1.4 "List of tool types", 828D 는 *Tools Function Manual*. `desc` 의 이름은 그 목록(07/2021 판)을 따르고, 조작반의 공구 목록 화면에서도 같은 번호가 보입니다.

알려진 코드는 `desc` 로 의미가 함께 오고 (`{"value": 500, "desc": "turning roughing tool"}`), 미등재 코드는 첫 자리 계열 desc 를 대신 씁니다 (`{"value": 573, "desc": "turning tool family"}`). **위 표의 계열 밖 코드는 `desc` 키 자체가 빠집니다** (`{"value": 300}`). 없는 뜻을 지어내지 않기 위해서입니다. `desc` 가 항상 있다고 가정하지 마세요.

선삭 공구(5xx)의 길이1/2 축 배정, 반경 해석(커터/노즈)을 판별하는 기준값이기도 합니다.

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolHNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

프로그램에서 이 날의 **길이 보정**을 부를 때 쓰는 `H` 번호입니다 (ISO 방언). `toolArea` + `tool` + `toolEdge` 필터. 반환 `int`, 읽기·쓰기 모두 지원하며 쓰기는 `{"value": 5}` 형식입니다.

`G43 H5` 처럼 프로그램이 길이 보정을 지정할 때의 그 `5` 입니다. Siemens 에서는 그 날에 **배정된** 번호입니다. 프로그램의 `H5` 는 공구와 무관하게 이 번호가 `5` 인 날의 보정을 적용합니다. 번호가 겹치지 않아야 하며 **중복 검사는 장비가 합니다.** 이미 쓰이는 번호를 쓰면 거절이 에러로 돌아옵니다. 정수만 받고 음수는 상태 `-16` 으로 거절합니다.

`0` 은 **배정되지 않음**입니다. ISO 방언을 쓰지 않는 Siemens 장비에서는 모든 날이 `0` 이며, 그 경우 보정값을 `/machine/toolArea/tool/toolEdge/toolLengthGeometry` 로 바로 읽으면 됩니다.

**Siemens 전용**입니다. Fanuc 의 `H` 번호는 날이 아니라 **공구**에 붙으므로 `/machine/toolArea/tool/toolHNumber` 에 있습니다 (같은 번호를 여러 공구가 공유할 수 있고, 쓰면 그 공구가 가리키는 보정 행이 바뀝니다).

**Mitsubishi 는 상태 `-20` 입니다.** 그 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조).

## /machine/toolArea/tool/toolEdge/toolTeethCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 보정 세트의 **인선 개수**입니다. "4날 엔드밀" 이라 할 때의 그 수. `toolArea` + `tool` + `toolEdge` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 4}`.

**`toolEdgeCount` 와 다른 값입니다.** 저쪽은 제어기가 그 공구에 대해 갖고 있는 보정 세트의 수이고, 이 값은 한 세트가 기술하는 절삭날이 물리적으로 몇 개인가입니다. 실측 예로 4날 커터가 보정 세트 `1` 개에 인선 `4` 개였습니다.

**보정 세트마다 따로 저장됩니다.** 한 자루에 지름이 다른 절삭부가 둘이면 인선 수도 다를 수 있어, 공구가 아니라 날에 붙습니다.

**장비 화면의 `N` 열과 항상 같지는 않습니다.** 그 열은 밀링 공구면 인선 수를, 드릴류면 선단각을 보여주는 겸용 칸입니다. 실측에서 드릴은 이 주소가 `0` 이고 화면엔 `118.0`(선단각)이 떴습니다. 디메시는 한 주소가 공구 종류에 따라 다른 물리량이 되지 않도록 둘을 섞지 않습니다.

**Siemens 전용**입니다. 없는 공구/날(D)을 지정하면 상태 `-18` 로 거절됩니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolLengthGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**길이1 형상값**입니다 (SINUMERIK `DP3`). 선삭 공구에서는 통상 X 방향에 대응하지만, 축 대응은 공구 타입과 활성 평면이 정하는 규칙이라 SDK 는 번역하지 않습니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolLengthWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**길이1 마모값**입니다 (SINUMERIK `DP12`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolLength2Geometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**길이2 형상값**입니다 (SINUMERIK `DP4`). 선삭 공구에서는 통상 Z 방향에 대응합니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolLength2Wear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**길이2 마모값**입니다 (SINUMERIK `DP13`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolLength3Geometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**길이3 형상값**입니다 (SINUMERIK `DP5`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolLength3Wear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**길이3 마모값**입니다 (SINUMERIK `DP14`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**커터 반경 형상값**입니다 (SINUMERIK `DP6`, 밀링 공구 관점). `toolNoseRadiusGeometry` 와 **같은 저장소**를 가리키며, 어느 주소를 쓰는지가 곧 소비자의 의도 선언입니다. SDK 는 공구 타입을 검사하지 않습니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 이름 그대로 **반지름**인데, 공구 목록/오프셋 화면은 흔히 **지름(Ø)** 으로 표시합니다. 실측(2026-07): `BALLNOSE_D8` 의 저장값이 `4.0` 인데 HMI 는 `8.000` 으로 보여줍니다. 디메시는 장비가 저장한 값을 그대로 내보내며 2를 곱하지 않습니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**커터 반경 마모값**입니다 (SINUMERIK `DP15`). `toolNoseRadiusWear` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 이름 그대로 **반지름**인데, 공구 목록/오프셋 화면은 흔히 **지름(Ø)** 으로 표시합니다. 실측(2026-07): `BALLNOSE_D8` 의 저장값이 `4.0` 인데 HMI 는 `8.000` 으로 보여줍니다. 디메시는 장비가 저장한 값을 그대로 내보내며 2를 곱하지 않습니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**노즈 반경 형상값**입니다 (SINUMERIK `DP6`, 선삭 공구 관점). `toolRadiusGeometry` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**노즈 반경 마모값**입니다 (SINUMERIK `DP15`). `toolRadiusWear` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼 §9.3.5). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**날끝 위치 코드**입니다 (SINUMERIK `DP2`, `1`~`9`). 노즈 반경 보정 때 날끝이 노즈 중심 기준 어느 방위에 있는지를 나타내며, 각도가 아니라 위치 코드입니다. Fanuc 의 공구 보정 트리에도 같은 개념이 있고 (`0`~`9`), **번호 체계와 `desc` 어휘를 공유**하므로 기종이 달라도 값을 그대로 비교·재사용할 수 있습니다 (`desc` 는 `9` 에만: `1`~`8` 방위는 매뉴얼이 도해로만 정의하고 가공 구성별로 세 벌이라 싣지 않습니다). 유효 범위는 `1`~`9` 이며 **`0` 은 허용되지 않습니다** (SINUMERIK 828D Tools Function Manual 이 날끝 위치에 0 을 허용하지 않는다고 밝힙니다).

반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 3}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다.

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Mitsubishi 는 상태 `-20` 입니다.** 그 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조).

## /machine/toolArea/tool/toolEdge/toolTipAngle
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 보정 세트의 **날끝 각**입니다. 드릴이면 선단각(`118.0`), 센터드릴이면 `90.0` 같은 값. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 118.0}`.

**`toolTipDirection` 과 다른 값입니다.** 이름이 한 글자 차이인데 성격이 다릅니다. 저쪽은 날끝이 노즈 중심 기준 **어느 방위**인지를 나타내는 코드이고, 이 값은 날끝의 **각도**입니다.

**각도를 쓰지 않는 공구는 `0.0`** 입니다. 밀링 공구가 그렇습니다. 실측에서 드릴은 `118.0`, 페이스밀은 `0.0` 이었습니다. 출처는 SINUMERIK `DP24` 로, 드릴류에서는 조작반(Operate)이 선단각으로 쓰는 칸이지만 **선삭 공구에서는 같은 칸이 여유각(clearance angle)** 입니다 (공구 관리 매뉴얼의 날 데이터 표). 선삭 공구에서 이 값을 선단각으로 읽지 마세요.

**장비 화면의 `N` 열은 이 값과 `toolTeethCount` 를 겸용합니다.** 드릴류면 각도를, 밀링이면 인선 수를 그 한 칸에 보여줍니다. 디메시는 한 주소가 공구 종류에 따라 다른 물리량이 되지 않도록 둘을 따로 냅니다. 화면의 그 숫자를 찾으려면 둘 중 값이 있는 쪽을 보세요.

**Siemens 전용**입니다. 없는 공구/날(D)을 지정하면 상태 `-18` 로 거절됩니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 날에 배정된 **수명 총량**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 6}`.

제어기는 잔여를 이 값에서 시작해 깎아 내려갑니다. 단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. `/machine/toolArea/tool/toolLifeMonitorType` 참조. 감시가 꺼져 있으면 상태 `-18` 로 거절됩니다.

**값은 조작반 화면에 뜨는 그대로입니다.** 시간 감시면 분(`unit` 은 `"min"`)이고 초로 환산하지 않습니다. 다른 시간 값들(`…Duration`)이 초인 것과 다릅니다. 수명은 작업자가 화면을 보며 판단하는 값이라 숫자가 화면과 같아야 합니다.

개수 감시일 때는 **정수만 받습니다.** 소수를 보내면 장비가 성공을 답하고 값은 바뀌지 않으므로 SDK 가 먼저 거절합니다.

**Siemens 전용**입니다. Fanuc 의 수명은 날이 아니라 공구에 붙으므로 `/machine/toolArea/tool/toolLifeTotal` 에 있습니다 (그쪽은 초·횟수 단위).

**Mitsubishi 는 상태 `-20` 입니다.** 그 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조).

## /machine/toolArea/tool/toolEdge/toolLifeRemaining
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 날에 **남은 수명**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 6}`.

SINUMERIK 은 줄어드는 값(카운트다운)이라 이 값이 곧 "지금 이 날의 수명" 입니다. 얼마나 썼는지는 `toolLifeTotal` 에서 이 값을 빼면 됩니다.

**인서트를 갈고 수명을 되돌릴 때 이 주소에 씁니다.** 보통 `toolLifeTotal` 과 같은 값을 넣습니다. 수명이 다해 잠긴 공구라면 이 쓰기가 잠금을 푸는 올바른 경로입니다.

단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 감시가 꺼져 있으면 상태 `-18`, 개수 감시에 소수를 쓰면 상태 `-16` 으로 거절됩니다.

**Siemens 전용**입니다. Fanuc 은 잔여가 아니라 **올라가는 사용량 카운터**를 주므로 공구 단위의 `/machine/toolArea/tool/toolLifeUsed` 로 읽고, 잔여는 `/machine/toolArea/tool/toolLifeTotal` 에서 빼서 구하세요.

**Mitsubishi 는 상태 `-20` 입니다.** 그 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조).

## /machine/toolArea/tool/toolEdge/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**경고선**입니다. 잔여가 이 값 밑으로 내려오면 제어기가 경고를 올립니다 (`/machine/toolArea/tool/toolLifeWarnOn`, **공구 단위**). `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 3}`.

교체 공구를 준비할 시간을 벌기 위한 값이라 `toolLifeTotal` 보다 작게 잡습니다. 단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 감시가 꺼져 있으면 상태 `-18`, 개수 감시에 소수를 쓰면 상태 `-16` 으로 거절됩니다.

**Siemens 전용**입니다. Fanuc 의 예고 수명은 공구에 붙으므로 `/machine/toolArea/tool/toolLifeWarnLimit` 에 있습니다.

**Mitsubishi 는 상태 `-20` 입니다.** 그 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조).
