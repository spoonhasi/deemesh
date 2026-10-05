## /machine/configuredProtocol
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

이 연결이 쓰는 **프로토콜 식별자**입니다. `"nc_focas2_fanuc"`·`"nc_opcua_siemens"`·`"nc_ezsocket_mitsubishi"`·`"nc_dnc_heidenhain"` 중 하나. 필터 없음. 반환 `string`, 읽기 전용. 연결 시 정해지는 값이라 연결된 뒤에는 NC 통신 없이 즉시 응답합니다 (미연결 상태에서는 다른 주소처럼 연결 확인이 먼저라 상태 `-10` 입니다).

`configuredMachineName` 과 마찬가지로 **설정에서 온 값**입니다. 기계에 물어본 결과가 아니라 `deemesh_create` 의 `protocol` 필드(또는 허브 `machines.json` 의 머신 설정)를 그대로 돌려줍니다. 그래서 주소에 `configured` 가 붙습니다.

**용도는 좁습니다.** 대부분의 주소는 기종을 감추도록 설계되어 있어 분기가 필요 없습니다. 이 값이 필요한 곳은 **값 공간이 기종 소유인 소수의 자리**입니다. PLC 주소 문법(`D100` 대 `DB10.DBB56`), 진단 번호 체계, 공구 타입 코드처럼 카탈로그가 "기종에 따라 다르다" 고 명시한 곳들입니다.

**지원 여부 판단에는 쓰지 마세요.** "이 기종은 이 주소를 못 쓰니 건너뛰자" 는 판단은 상태 `-20` 으로 해야 합니다. 이 값으로 분기해 두면 나중에 그 기종 지원이 추가돼도 코드가 계속 건너뜁니다.

## /machine/configuredMachineName
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

허브의 `machines.json` 또는 `deemesh_create` 설정의 `machine_name` 을 그대로 돌려줍니다. 장비가 보고하는 이름이 아니라 **설정에서 온 값**입니다. 이름에 `configured` 를 넣은 것도 그 때문입니다. 연결이 의도한 장비로 갔는지 확인하거나 응답을 식별할 때 씁니다.

## /machine/cncModel
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

CNC 모델 문자열입니다.

- **Fanuc**: 시리즈 번호 문자열입니다. `"15"`, `"16"`, `"18"`, `"21"`, `"30"`, `"31"`, `"32"`, `"35"`, `"0"`(0i), `"PD"`/`"PH"`(Power Mate i), `"PM"`(Power Motion i). `desc` 에 시리즈명이 함께 옵니다 (예: `"31"` → `Series 31i`). **제어기가 세대를 알려 주면 `desc` 끝에 그 글자를 붙입니다** (예: `Series 31i-B`, `Series 0i-F`). 세대 정보가 없는 제어기에서는 붙지 않습니다 (0i-A/B/C · 30i-A · 그 이전 시리즈). `value` 는 어느 쪽이든 같습니다. `desc` 는 표시용 문자열이라 바뀔 수 있으므로 세대 분기 조건으로 쓰지 마세요
- **Siemens**: 모델명 그대로 (예: `"840D sl"`). 연결 때 제어기의 NCK 종류를 읽지 못했거나 디메시가 모르는 종류면 `"UNKNOWN"` 입니다
- **Mitsubishi**: NC 시스템 S/W 번호·이름 문자열입니다 (벤더 `GetVersion`). 이 항목이 없는 장비에서는 상태 `-20` 으로 답합니다. 실물 하드웨어 정보가 없는 시뮬레이터가 그렇습니다. `desc` 는 없습니다
- **Heidenhain**: 제어기가 알려 주는 자기 모델 이름을 표기 그대로 냅니다 (예: `"TNC7"`. HEIDENHAIN DNC 가 주는 소프트웨어 목록의 NC 소프트웨어 항목). 연결 설정의 `system_type` 이 아니라 제어기에서 읽은 값이며, 연결할 때 한 번 읽어 둡니다. `desc` 에 NC 소프트웨어 번호가 옵니다 (예: 테스트 환경의 프로그래밍 스테이션에서 `NC software 817625 17 SP4`). 조작반 설정의 General information 에 보이는 Control model·NC-SW 와 같은 정보입니다 (TNC7 사용 설명서 'Software' 는 `817625` 를 프로그래밍 스테이션의 번호로 적습니다). 그 항목을 찾지 못하면 상태 `-20` 으로 답합니다

## /machine/machineType
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
codes: [{"value": "machiningCenter", "name": "Machining center"}, {"value": "lathe", "name": "Lathe", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]}, {"value": "punchPress", "name": "Punch press", "read": ["nc_focas2_fanuc"]}, {"value": "laser", "name": "Laser", "read": ["nc_focas2_fanuc"]}, {"value": "wireCut", "name": "Wire cut", "read": ["nc_focas2_fanuc"]}, {"value": "unknown", "name": "Unknown"}]
```

장비 종류입니다. 반환 `string`. 값 자체가 뜻을 담은 문자열입니다. 나올 수 있는 값 전체:

- `"machiningCenter"`: 머시닝센터 (Fanuc M/MM, Siemens M, Mitsubishi `…M` 계열, Heidenhain TNC7·TNC 640·TNC 620)
- `"lathe"`: 선반 (Fanuc T/TT/MT, Siemens T, Mitsubishi `…L` 계열)
- `"punchPress"`: 펀치 프레스 (Fanuc 전용)
- `"laser"`: 레이저 (Fanuc 전용)
- `"wireCut"`: 와이어 컷 (Fanuc 전용)
- `"unknown"`: 판별 불가. 장비 종류를 위 값 중 하나로 옮길 수 없을 때이며 모든 기종에서 나올 수 있습니다 (Mitsubishi 는 `system_type` 토큰에 `…M`·`…L` 이 없어 어느 쪽인지 읽어 낼 수 없는 구성, Heidenhain 은 아래 문단)

**이 주소는 기계 한 대의 값입니다.** Fanuc 의 복합기처럼 경로마다 계통이 다른 기계에서는 제어기가 기계 전체를 부르는 이름이 나옵니다 (제어기가 기계를 부르는 이름은 정해져 있습니다. 경로가 둘인 밀 계열은 `MM` 이라 `"machiningCenter"`, 경로가 둘·셋인 선반 계열은 `TT`, 복합 가공 기능이 있는 선반 계열은 `MT` 라 둘 다 `"lathe"` 입니다). 경로마다 갈리는 동작(G 모달 표·공구 오프셋 열 구성)은 디메시가 **그 경로의 계통**으로 처리하므로 이 값과 따로 움직입니다.

Mitsubishi 는 이 값을 **설정한 `system_type` 에서** 가져옵니다. 설정을 그대로 믿는 것이 아니라, 벤더가 `…M` 을 머시닝센터 시스템 · `…L` 을 선반 시스템으로 정의하고 연결 시 그 구분을 **실제로 검증**하기 때문입니다. 밀에 `…L` 을 지정하면 연결 자체가 거부됩니다. 즉 연결이 성립했다는 것이 곧 장비가 이 값을 확인해 준 것입니다.

Heidenhain 은 이 값을 **제어기가 알려 주는 모델 이름**(`cncModel` 의 값)으로 정합니다. 설정한 `system_type` 을 쓰지 않는 것은, 저희 테스트 환경의 TNC7 이 다른 `system_type` 으로도 연결을 받아 설정만으로는 장비를 확인할 수 없어서입니다. TNC7·TNC 640·TNC 620 은 Heidenhain 문서(문서 ID 1080370-04, §1.1)가 밀링 제어기로 분류하므로 `"machiningCenter"` 입니다. TNC 320·TNC 128 은 그 분류를 Heidenhain 문서에서 확인하지 못해 `"unknown"` 으로 답하고, 저희가 시험하지 않은 그 밖의 모델도 `"unknown"` 입니다. 어느 제어기인지는 `cncModel` 로 확인하세요. `cncModel` 이 상태 `-20` 인 제어기(디메시가 제어기의 소프트웨어 목록에서 모델 항목을 찾지 못한 경우)도 `"unknown"` 입니다.

## /machine/currentDateTime
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

장비의 **현재 날짜/시각**입니다. 반환 `string`, ISO 8601 초 단위 (`"2026-07-11T14:30:00"`).

- **장비 로컬 시계**(Fanuc·Siemens·Mitsubishi·Heidenhain): 타임존 정보가 없으므로 TZ 접미사(`Z`/`+09:00`)를 붙이지 않습니다. ISO 8601 의 로컬 시각 형식이며, 오프셋을 필수로 요구하는 RFC 3339 파서는 이 값을 거부할 수 있습니다
- **자바스크립트 `new Date()` 에 그대로 넣지 마세요**: 오프셋 없는 날짜+시각을 **보는 사람의 시간대**로 해석합니다. 이 값은 보는 사람이 아니라 **장비의 벽시계**입니다
- 서버 PC 시계가 아니라 **CNC 의 시계**입니다. 장비 시계가 틀어져 있으면 그대로 반영
- Fanuc: `cnc_gettimer` / Siemens: `sysTimeBCD` / Mitsubishi: `GetClockData` / Heidenhain: 기본 PLC 프로그램의 날짜·시각 심볼
- **Heidenhain 은 기본 PLC 프로그램의 날짜·시각 심볼을 읽습니다.** 제어기에 설정된 시각이라 `Z` 를 붙이지 않습니다 (저희 테스트 환경에서 제어기의 시간대를 바꾸자 이 값이 따라 바뀌었습니다. 시간대와 시각은 조작반 설정의 운영 체제 → Date/Time 에서 정합니다, TNC7 사용 설명서 'Adjust system time window'). PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 이고 `error` 에 어느 쪽인지 실립니다. 이 상태 `-20` 은 링크 이상이 아닙니다 (링크가 끊겼으면 상태 `-10`·`-14`·`-17` 로 답합니다). 디메시가 아는 날짜·시각 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다 (심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비의 이름을 알면 `/machine/plcAddress/plcText` 로 읽으세요)
- **장비 헬스체크 권장 주소**: 모든 프로토콜에서 실제 NC 왕복을 일으키는 부담 적은 읽기라, 주기 폴링 후 `status` 판정(`0`=정상, 상태 `-10`·상태 `-14`·상태 `-17`(연결 실패)=링크 이상)으로 장비별 통신 상태 감시에 쓰세요. (`machineType` 등 캐시 서빙 주소는 링크가 죽어도 성공할 수 있어 부적합)

시각 계열 주소는 항상 ISO 8601 문자열입니다 (`…At` = 이벤트 시점, `…DateTime` = 시계 읽기).

## /machine/powerOnDuration
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

장비의 **누적 전원투입 시간**입니다. 전원을 껐다 켜도 계속 누적됩니다. 반환 `int` (초) + `unit:"s"`.

- **Fanuc**: 파라미터 6750. **분 해상도**라 값이 항상 60의 배수입니다. 차분 계산 (가동률 등) 시 ±60초 오차 내재
- **Siemens**: `setupTime` 입니다. 840D sl 벤치에서는 정수 분으로 와서 값이 60의 배수였습니다 (디메시는 분에 60을 곱해 초로 옮기며 분 단위로 반올림하지 않습니다). 일반 전원 재투입에는 리셋되지 않지만, **기본값으로 제어기를 부팅하면 `0`** 이 됩니다 (드문 정비 작업)
- **Mitsubishi**: `GetAliveTime`. **초 해상도**입니다. 조작반 통합 시간 화면의 `Power ON`(전원 ON~OFF 누적)과 같은 값이며, **제어기는 `59999:59:59` 에서 누적을 멈추고 그 값을 유지**합니다 (M800 조작 매뉴얼). EZSocket 문서에는 이 값이 `HHHHMMSS` 8자리(최대 `9999:59:59`)로 적혀 있어, 9999시간(약 416일)을 넘은 뒤 API 가 무엇을 돌려주는지는 확인하지 못했습니다. 상한에 닿은 뒤로는 차분이 계속 `0` 이 됩니다
- **Heidenhain**: `GetNcUpTime` 입니다. 레퍼런스가 제어기가 켜져 있던 누적 시간이라 밝히고, 이 카운터는 되돌릴 수 없다고 적습니다. **분 해상도**라 값이 항상 60의 배수입니다. 조작반 설정 → 기계 설정 → 기계 시간의 "컨트롤 켜기" 와 같은 카운터였습니다 (테스트 환경에서 587:28:21 일 때 `2114880`)

경과시간 계열 주소는 항상 **초 정규화 int** 입니다 (`…Duration` 접미사 규칙). **초 미만은 버립니다**: `59.9`초는 `59` 입니다. 조작반의 경과시간 표시와 같은 방식이고, 아직 지나지 않은 초를 세지 않습니다. 모든 기종·모든 `…Duration` 주소가 같습니다.

## /machine/channelCount
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

CNC 의 채널(계통) 수입니다. 반환 `int`, 읽기 전용. 연결 시 캐싱된 값이라 추가 통신 없이 즉시 반환됩니다. `channel` 필터의 유효 범위가 `1`~이 값입니다.

출처는 Fanuc `cnc_getpath` 의 최대 경로 수, Siemens `/Nck/Configuration/numChannels` 이고, Mitsubishi 는 연결 때 계통 `1`~`8` 을 차례로 열어 보며 세어 둔 값, Heidenhain 은 연결 때 `GetChannelInfo` 가 준 채널 목록의 개수입니다. Siemens 와 Heidenhain 에서 연결 때 그 값을 읽지 못했으면 값을 지어내지 않고 상태 `-17` 로 답합니다.

HEIDENHAIN DNC 레퍼런스는 `GetChannelInfo` 의 채널 목록에 원소가 하나만 담긴다고 밝히므로 Heidenhain 의 값은 `1` 입니다.

## /machine/channel/toolAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

채널이 사용하는 공구 영역(tool area) 번호입니다. 공구 트리 주소들의 `toolArea` 필터에 넣는 값입니다.

**이 값을 읽어서 그대로 `toolArea` 에 넣으세요.** 번호를 매기는 방식이 기종마다 다르므로 직접 정하지 마세요.

Siemens 는 NCK 설정(`toNo`)값이라, 여러 채널이 **같은 번호**를 받을 수 있습니다. 그러면 그 채널들이 공구를 공유한다는 뜻입니다.

Fanuc·Mitsubishi 에는 공구 영역이라는 별도 계층이 없고 공구 데이터가 **경로(파트 시스템)에 딸려** 있습니다. 그래서 채널 번호가 그대로 돌아옵니다. 이 두 기종의 `/machine/toolArea/…` 주소는 `channel` 필터를 받지 않으므로, 경로를 지목하는 일을 `toolArea` 가 맡습니다. 예외로 **Mitsubishi 의 매거진은 계통이 아니라 기계 전체에 딸려** 있어, 매거진 주소(`magazineCount`·`magazineList`·`magazine/…`)는 `toolArea` 에 `1`~채널 수 중 어느 값을 넣어도 같은 매거진을 답합니다.

**Heidenhain** 은 공구 관리가 하나(공구 번호가 한 벌)라 언제나 `1` 입니다. 공구 주소의 `toolArea` 도 `1` 만 받고, 다른 값은 상태 `-18` 입니다.

## /machine/channel/executionStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
codes: [{"value": 0, "name": "Reset"}, {"value": 1, "name": "Stop"}, {"value": 2, "name": "Hold"}, {"value": 3, "name": "Run"}, {"value": 4, "name": "MSTR (retraction/recovery/JOG MDI)", "read": ["nc_focas2_fanuc"]}, {"value": 5, "name": "Interrupted", "read": ["nc_opcua_siemens"]}, {"value": 99, "name": "Unknown", "read": ["nc_focas2_fanuc"]}]
```

프로그램 **실행 상태** 코드입니다 (뜻은 `desc` 로 함께 옵니다). `operateMode`(무슨 모드인가)와 짝을 이루는 "지금 돌고 있는가":

- `0` = Reset · `1` = Stop · `2` = Hold · `3` = Run (실행 중)
- `4` = MSTR (Fanuc: 리트랙션/복구) · `5` = Interrupted (Siemens: 아래 참조) · `99` = Unknown (Fanuc 한정. Siemens·Heidenhain 은 미등재 값을 상태 `-17` 에러로 돌려줍니다)

`1` 과 `2` 는 정지의 **종류**가 다릅니다. 누가, 어디서 세웠는가로 갈립니다:

- `Stop` = **프로그램이 예정된 지점에서** 세운 것. 싱글블록 모드로 블록이 끝났거나(네 기종 모두 확인), M0/M1 을 만난 경우입니다. 항상 블록 경계에 서 있습니다.
  - ⚠️ **Fanuc 의 M0/M1 은 `Stop` 이 아닐 수 있습니다.** 제어기는 M 코드를 내보낸 뒤 PLC 의 완료 신호를 기다리는데, 그동안을 "사이클 진행 중" 으로 보아 **`3`(Run)** 을 냅니다. 31i 벤치(M0 을 처리하지 않는 PLC 구성)에서 프로그램이 `M00` 블록에 멈춰 서 있고 조작반의 사이클 스타트 램프가 깜빡이는 동안 이 주소는 내내 `3` 이었고, 사이클 스타트를 다시 주자 재개됐습니다. PLC 가 M0 을 처리하는 기계는 다를 수 있습니다. 31i-B 실장비 한 대에서는 프로그램이 `M00` 에서 멈춘 동안 `2`(Hold)였습니다. 어느 값이 나오는지는 그 기계의 래더가 정하므로, M0/M1 정지를 가려야 하면 그 기계에서 한 번 확인하세요. Siemens 는 같은 상황에서 `1`(Stop)입니다.
- `Hold` = **조작자가 임의 시점에** 세운 것. 조작반의 정지 키(피드홀드)를 누른 경우입니다. 블록 중간에서도 멈춥니다.

⚠️ **버튼 이름과 상태 이름이 어긋납니다** (업계 관례). 조작반의 **정지(Stop) 버튼을 누르면 상태는 `Hold`** 가 됩니다. `Stop` 상태는 버튼이 아니라 프로그램(M0/M1·싱글블록)이 만듭니다. 재개는 둘 다 Cycle Start 입니다.

**알람이 걸렸을 때의 답이 기종에 따라 다릅니다.** 같은 상황(없는 서브프로그램을 불러 자동운전이 멎음)을 세 기종의 테스트 환경에서 밟은 결과입니다:

| | `executionStatus` | `alarmStatus` |
|---|---|---|
| Fanuc | `1` (Stop) | `2` |
| Mitsubishi | `3` (Run), 리셋할 때까지 | `2` |
| Heidenhain | `0` (Reset), 잠깐 `2` 를 거쳐 | `2` |

제어기마다 자기 자동운전 상태를 표현하는 방식이 달라서입니다. 디메시는 이것을 일괄로 뒤집지 않습니다. 알람이 가공을 멈추는지는 알람마다 다르고(경고성 알람은 안 멈춥니다), 우리에겐 알람별로 그걸 아는 지식이 없어 강등하면 멀쩡한 경우를 틀리게 만듭니다.

**비상정지 때의 답도 기종에 따라 다릅니다.** Fanuc 은 시뮬레이터(NC Guide)와 31i 벤치에서 자동운전 중 비상정지를 걸었을 때 `0`(Reset)이었고, Mitsubishi 도 테스트 환경에서 `0`(Reset), Siemens 는 `5`(Interrupted)입니다. Heidenhain 은 테스트 환경에서 `emergencyStatus` 쓰기로 운전 중에 건 비상정지가 잠깐 `2` 를 거쳐 `0`(Reset)이었습니다. **비상정지 자체를 감지하려면 이 주소가 아니라 `/machine/channel/emergencyStatus` 를 쓰세요** - 그 주소가 기종 차이를 흡수합니다.

**그래서 `3`(Run)을 "지금 깎고 있다" 로 읽지 마세요.** 이 값은 자동 운전이 끝나지 않았다는 뜻이지 축이 움직인다는 뜻이 아닙니다. `3` 인데 서 있는 경우가 실제로 둘 확인됐습니다: **알람이 걸렸을 때**(`alarmStatus` 가 `2`)와 **M 코드 완료를 기다릴 때**(알람은 없습니다. 다만 Mitsubishi 는 이때 스톱 코드 `T10` 을 올리므로 `alarmStatus` 가 `1` 입니다). 정말 멈춰 있는지 알아야 하면 `/machine/channel/programCurrentBlock` 이 더 이상 바뀌지 않는지 보거나 `alarmStatus` 를 함께 읽으세요.

**Mitsubishi 는 `0`~`3` 만 냅니다.** 이 기종은 상태 코드가 아니라 자동 운전 플래그 셋(운전 중 · 진행 중 · 일시정지)을 주므로 디메시가 위 어휘로 합칩니다. `Stop` 과 `Hold` 의 구분은 벤더 정의와 그대로 맞아떨어집니다. 벤더가 말하는 "일시정지" 가 *명령을 실행하던 도중 멈춤* 이라 위 `Hold` 와 같은 상태이고, 자동 운전 중이면서 진행도 일시정지도 아닌 자리가 블록 경계에 선 `Stop` 입니다.

`5` (Siemens 전용) 는 **장비가 비정상이라 멈춘 경우**입니다. 840D sl 벤치에서 확인된 것은 둘입니다: **비상정지**와 **알람 정지**. 정상 정지(M0·싱글블록·조작자 정지)는 모두 `1`/`2` 로 갈리므로, `5` 를 보면 `/machine/channel/alarmStatus` 와 `/machine/channel/emergencyStatus` 로 어느 쪽인지 가르면 됩니다. 제어기는 정지 사유를 이보다 잘게 보고하는데, 디메시가 확인한 사유(M0·싱글블록·조작자 정지) 밖의 정지는 모두 여기로 떨어집니다. 그래서 `5` 는 종류를 가리지 않은 정지이고 `desc` 도 `Interrupted` 입니다. "멈췄다" 는 사실은 확실하니, 호출하는 쪽에서 종류를 가릴 필요가 없다면 `1`/`2`/`5` 를 묶어 "정지" 로 다뤄도 됩니다.

**Heidenhain** 은 파트 프로그램 상태(`GetProgramStatus`)를 옮깁니다. 테스트 환경(TNC7 프로그래밍 스테이션)에서 밟은 값입니다: 운전 중 `3`, 이동 중에 조작자가 NC 정지를 누르면 `2`(축이 블록 중간에 섭니다), `M0` 과 싱글블록 한 블록 뒤는 `1`(블록 경계), 프로그램 끝은 `0` 이었습니다 (`M30` 으로 끝나도 `0`). **오류로 프로그램이 멈추면 `2` 이고, 그 뒤는 제어기가 매긴 오류의 등급에 따라 갈립니다** (오류는 `alarmList` 에 남습니다). 프로그램을 중단하는 등급의 오류는 잠깐(테스트 환경에서 0.3초 안쪽) `2` 를 거쳐 `0` 이 됩니다: 충돌 감시 오류, 프로그램이 낸 오류(`FN 14`), 없는 서브프로그램 호출이 그랬습니다. 프로그램을 멈추게만 하는 등급의 오류는 `2` 에 머뭅니다: 테스트 환경에서 스핀들을 돌리지 않고 이송 블록을 돌렸을 때 오류를 지울 때까지 `2` 였고, 지운 뒤에도 조작자가 NC 정지를 누른 것과 같은 `2` 였습니다. TNC 는 운전 모드마다 프로그램을 따로 두므로 프로그램 실행 모드가 아닌 동안(수동·MDI)에는 그 모드의 프로그램 기준이라 대개 `0` 입니다. MDI 에서 블록을 실행하는 동안에도 `0` 이었습니다 (테스트 환경에서 그동안 HEIDENHAIN DNC 의 프로그램 상태가 '선택된 프로그램 없음' 이었고, MDI 의 실행 상태를 읽는 다른 길은 찾지 못했습니다). 조작반 용어와 이 주소의 이름은 거꾸로입니다: TNC7 사용 설명서는 `M0`·싱글블록으로 블록 경계에 선 것을 '중단(interrupt)', NC 정지 키로 선 것을 '정지(stop)' 라 부르는데, 이 주소는 상황으로 맞춰 앞의 것이 `1`(Stop), 뒤의 것이 `2`(Hold) 입니다.

## /machine/channel/operateMode
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
codes: [{"value": 0, "name": "Jog"}, {"value": 1, "name": "MDI"}, {"value": 2, "name": "Memory (Auto)"}, {"value": 5, "name": "No mode", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]}, {"value": 6, "name": "Edit", "read": ["nc_focas2_fanuc"]}, {"value": 7, "name": "Handle", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]}, {"value": 8, "name": "Teach in Jog", "read": ["nc_focas2_fanuc"]}, {"value": 9, "name": "Teach in Handle", "read": ["nc_focas2_fanuc"]}, {"value": 10, "name": "INC feed", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]}, {"value": 11, "name": "Reference"}, {"value": 12, "name": "Remote", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]}, {"value": 13, "name": "Jog-REPOS", "read": ["nc_opcua_siemens"]}, {"value": 14, "name": "MDI-Reference", "read": ["nc_opcua_siemens"]}, {"value": 15, "name": "MDI-Teach in", "read": ["nc_opcua_siemens"]}, {"value": 16, "name": "MDI-Teach in-Reference", "read": ["nc_opcua_siemens"]}, {"value": 17, "name": "Auto-Teach in-Reference", "read": ["nc_opcua_siemens"]}, {"value": 18, "name": "MDI-REPOS", "read": ["nc_opcua_siemens"]}, {"value": 19, "name": "MDI-Teach in-REPOS", "read": ["nc_opcua_siemens"]}, {"value": 20, "name": "Auto-Teach in", "read": ["nc_opcua_siemens"]}, {"value": 99, "name": "Unknown", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]}]
```

현재 운전 모드 코드입니다 (뜻은 `desc` 로 함께 옵니다). 기종 무관 통일 코드:

- `0` = Jog · `1` = MDI · `2` = Memory (자동) · `5` = 모드 없음 · `6` = Edit · `7` = Handle (핸들)
- `8` = Teach in Jog · `9` = Teach in Handle · `10` = INC feed · `11` = Reference (원점복귀) · `12` = Remote (DNC)
- `13` = Jog-REPOS · `14` = MDI-Reference · `15` = MDI-Teach in · `16` = MDI-Teach in-Reference · `17` = Auto-Teach in-Reference · `18` = MDI-REPOS · `19` = MDI-Teach in-REPOS · `20` = Auto-Teach in
- `99` = Unknown

`13`~`20` 은 **Siemens 전용**입니다. 기본 모드(Jog/MDI/Auto)에 보조 기능(REPOS·원점복귀·Teach in)이 겹쳐진 상태로, 조작반에서 그 조합을 고르면 나옵니다 (기능 매뉴얼 K1: JOG 는 REF·REPOS, MDI 는 REF·REPOS·Teach in 을 겹칠 수 있습니다. `20` 은 AUTO 에서 TEACH IN 을 누른 상태로, 840D sl 벤치에서 조작반 모드 표시가 `TEACH IN` 이었습니다). Fanuc 은 같은 상황을 기본 모드 코드로만 내보내므로 이 값이 나오지 않습니다. `8`(Teach in Jog)은 Fanuc 의 모드입니다. `17`(Auto-Teach in-Reference)은 기능 매뉴얼 K1 에서 찾지 못한 조합이고 840D sl 벤치에서도 나온 적이 없어, 실제로 나오는지 확인하지 못했습니다.

Mitsubishi 의 **RAPID**(수동 급속이송)는 `0`(Jog) 로 나옵니다. 수동 연속 이송이라는 점에서 Jog 와 같은 부류이고, 기종이 늘 때마다 번호를 새로 만들지 않는다는 규칙을 따릅니다. 조작반의 STEP 은 `10`(INC feed), TAPE 는 `12`(Remote)입니다.

`5`(모드 없음)는 **Fanuc 과 Mitsubishi** 에서 나옵니다. 어느 기본 모드도 선택돼 있지 않은 상태로, 조작반이 Fanuc 은 모드 자리에 `****` 를, Mitsubishi 는 "모드없음" 을 표시합니다. Mitsubishi 는 다계통 장비에서 **계통마다 모드가 따로**라 흔히 봅니다 (한쪽만 쓰는 동안 다른 계통이 이 값입니다. 그때 그 계통의 `alarmList` 에 `M01 0101 운전모드없음` 이 함께 올라옵니다). `99`(Unknown)와 다릅니다: 이쪽은 장비가 "모드 없음" 이라고 분명히 답한 것이고, `99` 는 우리가 그 값을 해석하지 못한 것입니다. Siemens 엔 대응 상태가 없습니다. Siemens 에서는 디메시가 해석하지 못한 모드 조합이 `99` 가 아니라 상태 `-17` 로 돌아옵니다.

**Heidenhain** 은 제어기의 실행 모드(`GetExecutionMode`)를 옮깁니다. 테스트 환경(TNC7 프로그래밍 스테이션)에서 수동 운전은 `0`, MDI 는 `1`, 프로그램 실행은 `2` 였고, 프로그램 실행에서 Single block 을 켜도 `2` 입니다 (싱글블록인지는 `/machine/channel/singleBlockOn` 이 말합니다). TNC7 사용 설명서('Overview of operating modes')는 편집기(Editor)·파일(Files)·표(Tables)도 운전 모드로 부르지만, HEIDENHAIN DNC 가 알려 주는 실행 모드는 기계를 움직이는 쪽(수동·MDI·프로그램 실행 등)이라 그 화면을 보는 동안에도 마지막 값이 나옵니다. 그래서 `6`(Edit)은 나오지 않습니다. 수동 운전 모드 안의 Setup 애플리케이션도 `0` 입니다. 핸드휠(`7`)은 수동 운전에서 핸드휠의 활성화 키로 핸드휠을 켰을 때 나왔고 다시 끄자 `0` 이었습니다 (테스트 환경의 가상 핸드휠). 프로그램 실행에서 핸드휠을 켜면 `2`, MDI 에서 켜면 `1` 그대로입니다. 원점 복귀(`11`)는 테스트 환경에서 DNC 로 그 모드로 바꿨을 때 나왔고, 조작반에서 들어간 경우는 확인하지 못했습니다 (설명서에 따르면 증분형 엔코더를 쓰는 기계는 전원을 켠 뒤 모든 축의 원점을 잡을 때까지 원점 복귀 화면에 머뭅니다. 테스트 환경은 늘 원점이 잡혀 있었습니다). 이 값들은 TNC7 에서 본 것이고, 설명서('Operating elements of the keyboard unit')는 TNC7 의 운전 모드 배치가 TNC 640 과 다르다고 적습니다 (몇몇 키가 모드를 바꾸지 않고 기능을 켭니다). TNC 640 에서 같은 값이 나오는지는 확인하지 못했습니다. 레퍼런스가 알려진 것 밖의 실행으로 분류하는 값은 `99` 입니다.

**쓰기는 Heidenhain 만 지원합니다** (`SetExecutionMode`. 다른 기종은 상태 `-20`(미지원)). 받는 값은 `0`(수동 운전)·`1`(MDI)·`2`(프로그램 실행) 셋이고, 그 밖의 코드는 상태 `-16`(쓰기 값 오류)입니다. 핸드휠(`7`)과 원점 복귀(`11`)를 받지 않는 것은, 테스트 환경에서 핸드휠 모드로 바꾸자 오버라이드가 가상 핸드휠의 값(이송·급속 `0`%, 스핀들 `50`%)으로 바뀌어 모드를 빠져나와도 남았고 (TNC7 사용 설명서는 핸드휠을 켜면 이송 다이얼이 핸드휠의 것으로 넘어간다고 적습니다), 원점 복귀 모드에서는 프로그램 실행으로 곧장 돌아오지 못했기 때문입니다. **이미 그 모드면 제어기에 보내지 않고 상태 `0` 입니다.** 읽기에서 싱글블록을 켠 프로그램 실행도 `2` 라, `2` 를 다시 써도 싱글블록은 그대로입니다 (싱글블록은 `/machine/channel/singleBlockOn` 에 씁니다). 제어기가 지금 바꾸지 않으면 상태 `-22`(기계 상태)입니다. 테스트 환경에서는 운전 중에 수동 운전으로 바꾸는 것이 그랬습니다. 모드를 바꾸면 조작반 화면도 곧바로 그 모드로 바뀌었습니다. **쓰기 주의**: 조작반 앞의 작업자가 쓰던 모드가 바뀝니다.

## /machine/channel/emergencyStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
codes: [{"value": 0, "name": "Not emergency"}, {"value": 1, "name": "Emergency"}, {"value": 2, "name": "Reset", "read": ["nc_focas2_fanuc"]}]
```

비상정지 상태입니다 (뜻은 `desc` 로 함께 옵니다): `0` = 정상, `1` = 비상정지. Fanuc 은 `2`(Reset)가 잠깐 스칩니다. 31i 벤치에서는 비상정지를 풀 때 두 번 재어 약 1.4초와 2.3초, RESET 을 누를 때 1초 미만이었습니다. `0` 이 아니면 "정상 아님" 으로 다루면 안전합니다.

**비상정지를 감지할 때는 이 주소를 쓰세요.** 비상정지가 드러나는 통로는 기종마다 달라서, 알람 목록에서 직접 찾는 방식은 기종에 따라 결과가 갈립니다. 이 주소가 그 차이를 흡수해 기종이 달라도 같은 뜻의 값을 냅니다.

Mitsubishi 는 알람 목록에 `EMG` 구분이 있으면 `1` 입니다. 비상정지의 **원인과 무관하게** 잡습니다.

**Siemens 840D sl 은 NC 가 PLC 인터페이스에 세우는 비상정지 활성 신호(`DB10.DBX106.1`) 하나로 판정합니다.** 기능 매뉴얼의 비상정지 순서에서 NC 가 이 신호를 세우는 단계와 비상정지 알람 `3000` 을 띄우는 단계는 연달아 일어나고 해제도 함께 되므로, 아래 두 단계 방식과 같은 뜻을 한 번의 읽기로 냅니다. 둘 다 기계 제작사의 PLC 프로그램이 비상정지 버튼을 NC 에 넘길 때 켜집니다. 이 신호를 읽으려면 연결에 쓰는 계정에 PLC `DB10` 읽기 권한이 있어야 하며(`SinuReadAll` 에 포함됩니다), 권한이 없어 읽지 못하면 상태 `-17` 입니다. **비상정지를 풀어도 RESET 으로 인정할 때까지 `1` 입니다** (840D sl 벤치: 신호와 알람 `3000` 이 RESET 까지 남았습니다). Fanuc 은 풀면 `2` 를 잠깐 거쳐 `0` 이 됩니다 (31i 벤치).

**그 밖의 Siemens(828D 등)는 두 단계로 판정합니다.** 모드 그룹 준비 신호(`readyActive`, PLC 인터페이스 DB11 DBX6.3)가 켜져 있으면 추가 통신 없이 `0` 이고, 꺼져 있을 때만 알람 스냅샷을 가져와 **비상정지 알람 `3000` 이 있으면 `1`, 없으면 `0`** 입니다. 준비 신호는 비상정지 말고도 "모드 그룹 준비 해제" 반응을 가진 알람(드라이브·측정계·원점복귀 실패 등)이 전부 끄기 때문에, 그것만 보면 Fanuc 의 비상정지 신호·Mitsubishi 의 `EMG` 보다 넓은 뜻이 됩니다 (기능 매뉴얼 A2). 그런 경우 이 값은 `0` 이고 멈춘 원인은 `alarmStatus`(`2`)와 `/machine/channel/alarmList` 가 말합니다. 비상정지를 풀어도 `3000` 은 확인(acknowledge)·리셋될 때까지 남으므로 `1` 이 그만큼 더 유지되며(840D sl 의 신호도 같습니다), 준비 해제 상태에서 알람 스냅샷을 가져오지 못하면(840D sl 은 신호를 읽지 못하면) 지어내지 않고 상태 `-17` 입니다. 이 판정이 읽는 알람 스냅샷은 `alarmList` 와 같은 것이라, 제어기를 켠 직후에는 `alarmList` 설명의 늦게 오는 알람 대기가 여기에도 똑같이 적용됩니다.

**Heidenhain 은 PLC API 의 CNC 비상정지 심볼을 읽습니다.** Heidenhain 이 제어기에 둔 PLC API 정의에서 이 심볼은 CNC 가 비상정지 상태라는 뜻입니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요. 저희 테스트 환경(TNC7 프로그래밍 스테이션)에서는 조작반 버튼으로 비상정지를 걸어 보지 못했고, 아래의 쓰기로 건 비상정지에서 `1` 을 확인했습니다.

**Heidenhain 은 쓰기도 됩니다 (비상정지 걸기·거두기).** `1` 을 쓰면 HEIDENHAIN DNC 로 비상정지 등급의 오류를 제어기에 띄워 비상정지로 보냅니다. 조작반 메시지 줄에 "Emergency stop requested through deemesh" 가 보이고, 운전 중이면 프로그램이 중단됩니다. ⚠️ **실제 장비에서는 가공 중이라도 곧바로 멈춥니다.** 이미 비상정지면 원인과 무관하게 아무것도 보내지 않고 상태 `0` 입니다. 테스트 환경에서는 쓴 뒤 1초 안에 이 주소가 `1` 이 됐고, 운전 중이던 `executionStatus` 가 `3` 에서 잠깐 `2` 를 거쳐 `0` 이 됐습니다.

`0` 은 **이 연결이 `1` 로 띄운 비상정지만 거둡니다.** 거두면 1초 안에 이 주소가 `0` 으로 돌아오고, 중단된 프로그램은 다시 돌지 않습니다 (테스트 환경에서 확인). 조작반의 비상정지 버튼 같은 다른 원인이 겹쳐 있으면 비상정지는 그대로입니다. 이 연결이 띄운 것이 없는데 비상정지면 상태 `-22` 이니 장비에서 푸세요. **연결이 끊기거나 디메시를 다시 시작해도 띄운 비상정지는 제어기에 남습니다.** 그 비상정지는 `0` 으로 거둘 수 없어(상태 `-22`) 조작반 메시지 창에서 그 메시지를 CE 로 지워야 합니다 (테스트 환경에서 확인). 조작반에서 CE 로 먼저 지워도 됩니다. 지금 상태를 보고 정하므로 쓰기에도 `access_password` 가 있어야 합니다. HEIDENHAIN DNC 레퍼런스는 이 기능을 TNC7 에서 DNC 1.7.1, TNC 640 에서 DNC 1.6.1 부터로 적습니다.

## /machine/channel/motionStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
codes: [{"value": 0, "name": "None (Idle)", "read": ["nc_focas2_fanuc"]}, {"value": 1, "name": "Motion", "read": ["nc_focas2_fanuc"]}, {"value": 2, "name": "Dwell"}, {"value": 3, "name": "Wait (Multi-path Synchronization)", "read": ["nc_focas2_fanuc"]}, {"value": 4, "name": "Not dwelling", "read": ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]}]
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
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
codes: [{"value": 0, "name": "No alarm"}, {"value": 1, "name": "Warning"}, {"value": 2, "name": "Alarm"}]
```

**알람 심각도**입니다. 기종과 무관하게 **`0` / `1` / `2` 세 값**만 반환하며, 그 기종에서 무엇이 문제인지는 `desc` 로 함께 옵니다.

| 값 | 의미 |
|---|---|
| `0` | 정상: 알람도 메시지도 없음 |
| `1` | 경고: 제어기가 경고·안내로 표시한 것 (안내 메시지, 정상적인 프로그램 정지, 배터리 저하 같은 경고 등) |
| `2` | 알람: **제어기가 알람으로 표시한 것** (조작반의 빨간 알람) |

**신호등으로 생각하면 됩니다**: `0` 초록 · `1` 노랑 · `2` 빨강. 세 값을 그대로 경광등이나 화면 표시등에 물리면 됩니다. 이 눈금이 **제어기가 자기 메시지를 칠하는 방식과 같기 때문**입니다: Mitsubishi 매뉴얼은 NC 알람과 PLC 알람을 빨강 배경으로, 경고·스톱 코드·오퍼레이터 메시지를 노랑 배경으로 칠합니다. 그래서 노랑은 제어기가 경고·안내로 표시한 것입니다. 배터리 저하(Fanuc)나 서보 경고(Mitsubishi `S52`·`S53`) 같은 장비 경고도 여기에 들고, Mitsubishi 의 스톱 코드처럼 **"장비가 지금 무언가를 기다리고 있다"** 는 신호도 여기에 듭니다.

💡 **가장 튼튼한 사용법은 `0` 인지 아닌지입니다.** `0` 은 모든 기종에서 뜻이 정확히 같고("알람도 메시지도 없음") `alarmCount` 가 `0` 인 것과 맞물립니다. 반면 `1` 과 `2` 의 경계는 **그 기종이 자기 알람을 어떻게 분류하느냐**에 기대므로 기종마다 미세하게 다를 수 있습니다. 심각도로 분기해야 한다면 `alarmList` 의 항목별 `severity` 와 본문을 함께 보세요.

⚠️ **`1` 이 늘 "무언가 잘못됐다" 는 뜻은 아닙니다. 정상 가공 중에도 뜰 수 있고, 그 빈도가 기종마다 다릅니다.** 아무 문제 없는 자동 운전(드웰만 도는 프로그램)을 두 기종의 테스트 환경에서 측정한 결과입니다:

| | `alarmStatus` | 목록에 담긴 것 |
|---|---|---|
| Fanuc | `0` | 없음 |
| Mitsubishi | `1` | 스톱 코드 |

**Mitsubishi 에만 "스톱 코드" 라는 채널이 있기 때문입니다.** 제어기가 *자동 운전의 상태*를 알리는 자리로, 벤더 매뉴얼도 알람(빨강)이 아니라 경고와 같은 노랑으로 칠합니다. 담기는 것은 `T03 0301`(싱글블록 정지)·`T10`(M 코드 완료 대기)·`T02 0202`(소프트 리밋에 걸린 축 있음) 같은 것들이라 **고장이 아니라 "지금 이걸 기다리는 중"** 입니다. 위에서 확인했을 때 올라와 있던 것도 `T10` 이었고, 프로그램이 끝나자 사라졌습니다. Mitsubishi 도 축이 움직이는 자동 운전에서는 `0` 이었습니다 (시뮬레이터에서 확인).

**코드의 앞 글자는 계열이고 뒤 네 자리가 사유입니다** (`T01` 사이클 스타트 불가 · `T02` 자동 운전 정지 · `T03` 블록 단위 정지 · `T04` 대조 정지 · `T10` 완료 대기). 그래서 같은 `T02` 라도 `0202` 는 소프트 리밋, `0204` 는 피드홀드입니다. **조작자가 피드홀드나 싱글블록을 누르기만 해도 스톱 코드가 올라와 이 주소가 `1` 이 됩니다** (테스트 환경에서 확인: 피드홀드 `T02 0204`, 싱글블록 정지 `T03 0301`). 사유가 필요하면 `alarmList` 항목의 `code` 를 네 자리까지 보세요. Fanuc 은 이런 상태를 알람 목록에 싣지 않아 `0` 이 유지됩니다. Siemens 는 NC 가 이런 상태를 목록에 싣지 않지만, 기계 제작사의 PLC 메시지가 올라오면 `1` 이 될 수 있습니다 (840D sl 벤치에서는 `M0` 이 PLC 메시지 `700355` 를 경고로 올렸습니다).

**그래서 `1` 을 운영자 호출 신호로 쓰지 마세요**. Mitsubishi 장비에서는 정상 자동 운전 중에도 `1` 이 될 수 있습니다. 호출에는 `2` 를 쓰거나 `alarmList` 항목의 `severity` 와 `category` 를 보고 판단하세요.

**기준은 제어기가 매긴 등급이지 "가공이 멈췄느냐" 가 아닙니다.** 둘은 대개 함께 가지만 갈릴 때가 있습니다. `M0`(프로그램 정지)나 싱글블록으로 선 기계는 알람이 아니므로 `2` 가 되지 않습니다 (Mitsubishi 는 스톱 코드로 값 `1`, Fanuc 은 목록에 싣지 않아 값 `0`. Siemens 는 NC 가 목록에 싣지 않지만 기계 제작사의 PLC 메시지로 값 `1` 이 될 수 있습니다). 반대로 Fanuc 의 백그라운드 편집 알람(`BG`, 예: 형식이 맞지 않는 프로그램을 올렸을 때의 `BG1090`)은 조작반이 빨간 알람으로 표시하므로 `2` 이지만, 자동운전은 그대로 시작되고 돌 수 있습니다 (시뮬레이터에서 확인. 리셋이나 `M30` 에서 풀립니다). 지금 가공이 멈췄는지는 `/machine/channel/executionStatus` 로 판단하세요.

- **숫자는 어느 기종에서든 이 셋뿐입니다.** 벤더 코드를 그대로 내보내지 않으므로, 어느 기종에 붙였는지 몰라도 `value` 로 바로 분기할 수 있습니다.
- **원인은 `desc` 로 옵니다.** Fanuc 은 원인 계열(`{"value": 1, "desc": "Memory backup battery voltage low (CNC or Amplifier)"}`), Siemens 는 가장 무거운 알람의 본문(`{"value": 2, "desc": "Emergency stop"}`), Mitsubishi 는 알람 종류(`{"value": 2, "desc": "NC alarm"}`). `desc` 는 사람이 읽는 문자열이므로 **분기 조건으로 쓰지 마세요.** 분기는 `value` 로.
- **Mitsubishi**: **제어기가 메시지를 칠하는 색 그대로**입니다. 빨강(NC 알람·PLC 알람 메시지)은 `2`, 노랑(NC 경고·스톱 코드·오퍼레이터 메시지)은 `1`. 벤더 API 는 NC 알람과 NC 경고를 한 종류로 주므로 줄마다 색을 되가릅니다. EZSocket `FCSB1224W100-A9` 이상에서는 제어기가 그 줄을 경고로 치는지 함께 알려 주므로(`GetAlarm3`) 그것을 따르고, 그 아래 판에서는 **카테고리 코드로** 가릅니다 (`GetAlarm2`): 운전 에러 `M00`/`M01`, 운전 경고 `M50`, 서보 경고 `S52`·`S53`, 스마트 안전 경고 `V5x`(`V5` 로 시작하는 것)가 노랑(`1`)이고, 그 밖의 NC 알람(`S01`~`S05`·`S51`·`Y`·`Z`·`Z7x`·`Z8x`·`EMG`·`L`·`U`·`N`·`P`·`V01`~`V07` 등)과 PLC 알람은 빨강(`2`)입니다. 모르는 카테고리는 `2` 입니다. 두 방법은 시뮬레이터에서 `M01`·`P114`·`EMG` 에 같은 등급을 냈습니다. 그래서 운전 모드 미선택(`M01 0101`)이나 오버라이드 0(`M01 0102`) 같은 대기 상태는 `1` 이고, 비상정지(`EMG`)는 `2` 입니다. ⚠ 소프트 스트로크 엔드는 Mitsubishi 가 `M01 0007` 운전 에러(노랑 → `1`)로 다루지만 Fanuc 은 OT 알람(`2`)입니다. 같은 상황을 두 제어기가 다른 등급으로 치는 자리라, 이 주소는 각 제어기의 등급을 그대로 따릅니다
- 판정이 애매한 벤더 코드는 **보수적으로 `2`** 로 분류합니다. 정지를 경고로 낮춰 부르는 쪽이 그 반대보다 위험하기 때문입니다. Fanuc 제어기가 SDK 에 없는 코드를 내보내도 `2` 로 분류합니다. Siemens 는 서버가 이벤트마다 주는 심각도를 씁니다. 오류(`1000`)만 `2` 이고 벤더가 **경고(`500`)라 답한 것은 `1`** 입니다. `alarmList` 의 `severity` 와 **같은 기준**입니다.
- 알람 **목록·번호·메시지**가 필요하면 `alarmList`, **개수**만 필요하면 `alarmCount` 를 쓰세요. 이 주소는 고빈도 폴링용 요약입니다. **목록보다 가벼운지는 기종마다 다릅니다.** Fanuc 은 목록을 읽지 않고 상태 정보로 판정합니다 (오퍼레이터 메시지 존재 확인 때문에 왕복이 하나 더 붙지만, 다른 상태 주소와 함께 물으면 그 하나만 늘어납니다). Mitsubishi 는 이 주소만 따로 물으면 종류별 존재만 확인해 알람이 없는 평상시 왕복 1회이고, `alarmList`·`alarmCount`·`emergencyStatus` 와 함께 물으면 목록 전체를 읽습니다. Siemens 는 세 주소가 같은 스냅샷에서 나오므로 `alarmList` 와 비용이 같습니다.
- ⚠️ **`alarmStatus` 가 `0` 이 아닌데 `alarmCount` 가 `0` 일 수 있습니다.** Fanuc 은 요약이 조작반 **상태표시줄**까지 보는 반면 목록은 알람·메시지만 담아, 요약에만 값이 있는 상태가 존재합니다 (배터리 저하·전원 경고·절연 저하 계열). 반대 방향(`alarmStatus` 가 `0` 인데 목록에 항목이 있음)은 없습니다.
- **Siemens 는 `channel` 값을 쓰지 않습니다** (`alarmList`/`alarmCount` 와 같은 방침). 셋 다 NCK 전역 스냅샷에서 나오므로 한 번의 요청으로 함께 답하며, 서로 어긋나지 않습니다. 채널별로 나눌 수 없는 이유는 알람 이벤트에 채널 정보가 없기 때문입니다 (OPC-UA 매뉴얼이 정하는 알람 발생 영역은 `HMI`/`NCK`/`PLC` 셋입니다). 어느 기종이든 `channel` 값은 범위 검증됩니다.
- **Siemens 는 기계 제작사의 PLC 알람도 함께 봅니다** (유압·윤활·도어 인터록 등). 종전에는 NCK 알람만 보는 노드를 읽어 그런 알람이 떠 있어도 `0`(정상)이 나왔습니다.
- **Siemens 는 제어기를 켠 직후에 알람이 늦게 도착할 수 있습니다** (`alarmList` 와 같은 스냅샷). 알람 구독을 만든 뒤(새 연결이나 재연결)의 첫 읽기에서 알람이 없으면 늦게 오는 알람을 최대 1초 기다렸다가 답하므로(요청의 `timeout` 에서 남은 시간 안에서만 기다립니다) 그 읽기는 그만큼 늦어지고, 그보다 늦게 온 알람은 다음 읽기에 반영됩니다. 840D sl 에서 켠 직후의 비상정지 감지에는 `emergencyStatus` 를 쓰세요 (그 기종에서는 PLC 신호를 읽습니다). 828D 등 다른 Siemens 의 `emergencyStatus` 는 같은 알람 스냅샷을 읽습니다.

**Heidenhain** 은 `GetErrorList` 의 항목마다 제어기가 매긴 등급을 따릅니다. 오류(Error) 등급이 하나라도 있으면 `2`, 경고(Warning)·안내(Info)·노트(Note)만 있으면 `1` 이고, 등급 이름이 `desc` 로 옵니다. TNC7 사용 설명서('Message menu on the information bar')의 구분과 같습니다: 오류는 지워야 계속할 수 있고(다시 시작해야 하는 오류도 있습니다), 경고·안내·노트는 지우지 않고 계속할 수 있습니다. 번호나 문구는 있는데 등급이 없거나 모르는 값인 항목은 안전한 쪽으로 오류로 보아 `2` 입니다. 두 가지를 알아 두세요. 첫째, 저희 테스트 환경(TNC7 프로그래밍 스테이션)에서는 보안되지 않은 DNC 연결이 생길 때마다 제어기가 그 사실을 안내(`130-07e2`)로 남겼습니다. 그래서 디메시가 붙은 뒤로 이 주소는 조작반 메시지 메뉴에서 그 안내를 지우기 전까지 `0` 으로 돌아가지 않았고(안내는 언제든 지울 수 있습니다), 연결을 새로 맺을 때마다 `alarmCount` 가 하나씩 늘었습니다. 보안 연결(`RPC secure`, `connection_name`)은 이 안내를 남기지 않았습니다. 둘째, 같은 테스트 환경에서는 `M0` 에서 PLC 메시지 `PLC00050` 이 오류 등급으로 올라와 값 `2` 였습니다 (재개하자 사라졌습니다). 어떤 메시지를 어느 등급으로 띄울지는 그 기계의 PLC 프로그램이 정합니다. 등급도 번호·문구도 없는 빈 항목은 세지 않습니다 (`alarmList` 참조).

## /machine/channel/alarmCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

활성 알람/메시지 **개수**입니다 (= `alarmList` 항목 수, severity 불문). 반환 `int`. 대시보드 배지처럼 개수만 필요할 때 쓰세요.

- **비용 주의 (Fanuc)**: 목록을 받아 세므로 내부적으로 `alarmList` 와 **같은 비용**입니다. 저비용 존재 판정만 필요하면 `alarmStatus` 를 쓰세요
- **비용 주의 (Siemens)**: 개수를 `alarmList` 와 **같은 이벤트 스냅샷**에서 세므로 같은 비용이며, `alarmStatus` 도 같은 스냅샷에서 나오므로 더 싸지 않습니다. NCK 전역 개수라 `channel` 값은 무시됩니다 (`alarmList` 와 동일 방침)
- **비용 주의 (Mitsubishi)**: Fanuc 과 같습니다. 목록을 받아 세므로 `alarmList` 와 같은 비용입니다
- **Siemens 는 제어기를 켠 직후에 알람이 늦게 도착할 수 있습니다** (`alarmList` 와 같은 스냅샷). 알람 구독을 만든 뒤(새 연결이나 재연결)의 첫 읽기에서 알람이 없으면 늦게 오는 알람을 최대 1초 기다렸다가 답하므로(요청의 `timeout` 에서 남은 시간 안에서만 기다립니다) 그 읽기는 그만큼 늦어지고, 그보다 늦게 온 알람은 다음 읽기에 셉니다. 840D sl 에서 켠 직후의 비상정지 감지에는 `emergencyStatus` 를 쓰세요 (그 기종에서는 PLC 신호를 읽습니다). 828D 등 다른 Siemens 의 `emergencyStatus` 는 같은 알람 스냅샷을 읽습니다

- **비용 주의 (Heidenhain)**: `alarmList` 와 같은 목록(`GetErrorList`)을 세므로 같은 비용입니다. 안내(Info) 등급 항목도 셉니다. 조작반 메시지 메뉴의 Group 표시(같은 번호를 한 줄로 묶음)와 달리 항목마다 셉니다. 테스트 환경에서는 보안되지 않은 DNC 연결마다 안내가 하나씩 남아, 연결을 새로 맺을 때마다 값이 늘었습니다. 보안 연결(`RPC secure`, `connection_name`)은 남기지 않았고, 등급도 번호·문구도 없는 빈 항목은 세지 않습니다 (`alarmStatus`·`alarmList` 참조)

## /machine/channel/alarmList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
field_codes: {"severity": [{"value": "alarm", "name": "Alarm"}, {"value": "warning", "name": "Warning"}]}
```

채널의 **활성 알람 + 오퍼레이터/매크로 메시지** 목록입니다. 반환 타입 `objectArray`, 없으면 빈 배열 `[]`. Siemens 는 알람이 NCK 전역이라 `channel` 값은 무시됩니다.

**Siemens 에서 이 목록에 나오는 것은 제어기의 알람과 기계 PLC 의 메시지(`700000` 번대)입니다.** 파트 프로그램의 `MSG()` 문구는 알람이 아니라서 나오지 않습니다 (Fanuc 의 매크로 메시지 `#3006` 과 다른 점, 840D sl 벤치에서 확인). PLC 메시지는 `warning` 으로 나오며(840D sl 벤치에서는 이송 오버라이드 `0`, `M0` 정지 때 떴습니다), 어떤 메시지를 띄우는지는 그 기계의 PLC 프로그램이 정합니다.

**Siemens 는 제어기를 켠 직후에 알람이 늦게 도착할 수 있습니다.** 840D sl 벤치를 비상정지를 건 채 재부팅하고 OPC-UA 포트가 열리고 10초가 안 되어 처음 읽었을 때, 알람 두 건이 목록이 끝났다는 표시보다 늦게 왔습니다 (17초 뒤부터는 처음부터 목록에 담겼습니다). 그래서 알람 구독을 만든 뒤(새 연결이나 재연결)의 첫 읽기에서 목록이 비어 있으면 디메시가 늦게 오는 알람을 최대 1초 기다렸다가 답합니다 (요청의 `timeout` 에서 남은 시간 안에서만 기다립니다). 그 읽기는 그만큼 늦어지고, 그보다 늦게 온 알람은 다음 읽기에 담깁니다. 840D sl 에서 켠 직후의 비상정지 감지에는 `emergencyStatus` 를 쓰세요 (그 기종에서는 알람이 아니라 PLC 신호를 읽습니다). 828D 등 다른 Siemens 의 `emergencyStatus` 는 이 목록과 같은 알람 스냅샷을 읽습니다.

원소: `{"code": "OH0700", "message": "SPINDLE OVERHEAT", "category": "Overheat", "severity": "alarm", "raisedAt": "2026-07-29T11:11:03Z"}`

키 집합은 **기종과 무관하게 항상 같습니다.** 값이 없으면 키가 빠지는 게 아니라 `null` 입니다 (`entry` 와 같은 규약). `severity` 는 `"alarm"` / `"warning"` 두 값뿐입니다.

- **code**: **조작반이 보여주는 식별자 그대로의 문자열**입니다. Fanuc 은 알람 타입 약어 + 4자리 번호(`"OT0501"`·`"PS0010"`, 오퍼레이터/매크로 메시지는 메시지 번호 `"2000"`), Siemens 는 알람 번호(`"4230"`·`"700015"`. 번호 범위가 출처를 말합니다: `0`~`9999` 일반, `10000`~`19999` 채널, `20000`~`29999` 축·스핀들, `60000`~`69999` 사이클, `100000`~`199999` HMI, `200000`~`299999` 드라이브(SINAMICS), `300000`~`399999` 드라이브·I/O, `400000`~`899999` PLC, 그중 `500000`~`899999` 는 기계 제작사가 정의. 진단 매뉴얼), Mitsubishi 는 구분 + 상세(`"M01 0101"`·`"S01 0051"`·`"EMG EXIN"`). 문자가 섞이고 앞자리 0 이 의미를 가지므로 **숫자로 변환하지 말고 문자열 그대로** 매뉴얼에서 찾으세요. 식별자를 못 얻으면 `null` 이 아니라 `""` 입니다 (Fanuc·Siemens·Mitsubishi 는 실제로는 항상 채워집니다. Heidenhain 은 번호가 없는 항목에서 `""` 입니다). 1.2.0 에서 정수 `number` 를 대체했습니다
- **message**: 표시 텍스트
- **category**: Fanuc: 알람은 원인 계열 (`Servo`, `Overheat`, `Spindle`, `PLC` 등: 미정의 타입은 숫자 문자열), 메시지는 출처 (`Operator message` = PMC/외부입력, `Macro message` = 파트프로그램 #3006). Siemens: 서버가 이벤트에 싣는 출처 이름(`SourceName`) 그대로 (예: `NCU`: 비어 있으면 `Alarm`). Mitsubishi: 조작반에 뜨는 알람 구분 (`EMG`, `S01`, `M01` 등)
- **severity**: **제어기가 그 항목에 매긴 등급**입니다. `"alarm"` = 제어기가 알람으로 표시한 것(조작반의 빨강) / `"warning"` = 경고·안내로 표시한 것. 가공이 멈췄는지를 뜻하지는 않습니다. Fanuc: 알람 목록의 알람은 백그라운드 편집 알람(`BG`)을 포함해 전부 alarm 이고 (조작반이 빨간 알람으로 표시합니다. `BG` 알람이 떠 있어도 자동운전은 시작될 수 있습니다), 오퍼레이터/매크로 메시지는 전부 warning. Siemens: 서버의 심각도(1~1000)를 500 경계로 번역. Mitsubishi: 제어기가 빨강으로 칠하는 것(NC 알람·PLC 알람)은 alarm, 노랑(NC 경고·스톱 코드·오퍼레이터 메시지)은 warning 이며 `alarmStatus` 와 같은 기준입니다. NC 알람 줄의 경고 여부는 EZSocket `FCSB1224W100-A9` 이상에서는 제어기의 판정(`GetAlarm3`)을 따르고, 그보다 앞선 판에서는 카테고리 코드로 가립니다 (`M00`/`M01`/`M50`/`S52`/`S53`/`V5x` 가 경고). "지금 가공이 멈췄는가"는 이 필드가 아니라 `executionStatus` 로 판단하되, **알람 중에는 그 값도 기종에 따라 `3`(Run)일 수 있습니다** (그 주소 설명 참조). warning 인데 정지 상태면 매크로 `#3006` 등 오퍼레이터 개입 대기입니다
- Fanuc 은 한 번에 활성 **알람 최대 100건**(디메시가 잡은 읽기 크기), 오퍼레이터/매크로 **메시지 최대 17건**(FOCAS2 스펙이 전체 읽기에 정한 개수: 오퍼레이터 메시지 16 + 매크로 메시지 1)까지 실어 옵니다. Mitsubishi 는 알람 종류별로 **10건씩**이라 합계 **최대 40건**입니다 (벤더 API 상한). 상한은 종류마다 따로 걸리므로(Fanuc 은 알람과 메시지를 따로 읽고, Mitsubishi 는 종류마다 10건), 넘칠 때는 넘친 종류 안에서만 잘립니다.
- **raisedAt**: 발생 시각. **`Z` 로 끝나는 UTC** 입니다 (`"2026-07-29T11:11:03Z"`). Siemens 는 실제 시각이고, 시각이 없는 항목(조작반에 `---` 로 보이는 것)은 `null` 입니다 (840D sl 벤치: 비상정지를 건 채 재부팅하면 생깁니다). Heidenhain 도 실제 시각입니다 (아래 참조). Fanuc·Mitsubishi 는 항상 `null` (활성 알람에 시각 정보가 없음)
  - 장비 화면(HMI)이 보여주는 시각과 **숫자가 다릅니다.** HMI 는 장비 시간대로 표시하고 이 값은 UTC 입니다. 같은 순간을 다르게 표기한 것이며, 변환은 장비의 시간대를 아는 쪽(호스트 앱)이 합니다. OPC-UA 이벤트에는 시간대 오프셋 필드가 있지만, 실측한 장비에서는 비어 있었습니다
  - **`/machine/currentDateTime` 과 직접 빼지 마세요.** 시간대가 다를 뿐 아니라 **출처 시계가 다릅니다.** 한 장비에서 두 시계가 18분가량 어긋나 있는 것을 실측했습니다 (시계 설정은 현장마다 다릅니다)

**Heidenhain** 은 `GetErrorList` 의 항목입니다. `code` 는 조작반에 보이는 번호 그대로의 문자열(`130-07e2`·`PLC00050`), `category` 는 제어기의 오류 그룹(`Operating`·`Programming`·`PLC`·`General`·`Remote`·`Python`)이고 그룹이 없는 항목은 `""`, 디메시가 모르는 그룹은 그 번호의 문자열입니다. `severity` 는 제어기가 매긴 등급으로 오류(Error) 계열이 `alarm`, 경고(Warning)·안내(Info)·노트(Note)가 `warning` 이며, 등급이 없거나 모르는 값인 항목은 안전한 쪽으로 `alarm` 입니다. `message` 는 제어기의 표시 언어로 옵니다. `raisedAt` 은 UTC(`Z`) 입니다. 레퍼런스에서 이 시각의 시간대를 확인하지 못했지만, 테스트 환경에서 PC 의 UTC 와 초 단위로 맞았고 조작반 메시지 창(제어기 현지 시각)과는 시간대만큼 달랐습니다. 안내(Info)도 목록에 담기므로, 저희 테스트 환경에서는 보안되지 않은 DNC 연결이 생길 때마다 남는 안내(`130-07e2`)가 쌓여 있었습니다. 보안 연결(`RPC secure`, `connection_name`)로 붙었을 때는 이 안내가 남지 않았습니다. 등급도 번호·문구도 없는 빈 항목은 담지 않습니다 (저희 테스트 환경에서 조작반의 메시지를 모두 지운 뒤 그런 항목이 하나 왔는데, 조작반에는 아무것도 보이지 않았습니다). 채널을 가리지 않습니다 (채널이 하나입니다).

## /machine/channel/singleBlockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

싱글블록 스위치 상태입니다 (`true` = 켜짐). Fanuc 은 F4 신호 비트, Siemens 는 `singleBlockActive`, Mitsubishi 는 조작반 신호 블록의 PLC 출력(Y) 비트입니다.

Heidenhain 은 스위치 신호가 아니라 실행 모드(`GetExecutionMode`)로 가립니다. 프로그램 실행 모드에서 Single block 이 켜져 있으면 멈춰 있어도 `true` 이고 (테스트 환경에서 확인), 수동·MDI 모드에서는 실행 모드가 수동·MDI 로 오므로 `false` 로 답합니다. TNC7 사용 설명서에 따르면 Single block 스위치는 프로그램 실행 모드에만 있고, MDI 는 스위치 없이 늘 한 블록씩 실행합니다. 이 주소는 스위치를 말하므로 MDI 에서도 `false` 입니다.

**쓰기는 Heidenhain 만 지원합니다** (`SetExecutionMode`. 다른 기종은 상태 `-20`(미지원)). 싱글블록은 프로그램 실행 모드 안의 스위치라 그 모드에서만 받고, 수동·MDI 같은 다른 모드에서는 상태 `-22`(기계 상태)입니다. 켜면 운전 모드가 바뀌어 버리기 때문이니, 먼저 `/machine/channel/operateMode` 에 `2` 를 쓰세요. 이미 그 상태면 제어기에 보내지 않고 상태 `0` 입니다. 테스트 환경에서는 운전 중에도 켜고 끌 수 있었고, 조작반 화면에 Single block 이 곧바로 표시됐습니다.

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

**Fanuc 은 상태 `-20` 입니다.** 옵셔널 스톱 스위치는 기계 조작반에서 PMC 로 들어가는 입력이라, 그 상태는 그 장비 래더의 디바이스에 있습니다 (Connection Manual B-64483EN-1 이 M00/M01 은 코드·스트로브·디코드 신호만 보내고 정지와 옵셔널 스톱 제어는 PMC 쪽에서 설계한다고 밝힙니다). 제어기가 내는 `DM01`(`F9.6`)은 프로그램이 `M01` 블록을 지령했다는 디코드 신호라 스위치와 무관합니다 (NC Guide 실측: 스위치를 켜도 `F9` 는 `0`). 래더가 쓰는 디바이스를 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요.

**Mitsubishi 도 상태 `-20` 입니다.** PLC 인터페이스 매뉴얼(IB-1501272)이 설명하는 구조에서는 `M01` 이 나오면 **기계 제작사의 PLC 가 자기 스위치 입력을 보고** 싱글블록 신호(`SBK`)를 걸어 멈추게 하므로, 스위치 상태는 그 장비 래더의 디바이스에만 있고 기종 무관 주소로는 읽을 수 없습니다 (`plcAddress` 와 같은 부류). 그 장비의 래더가 쓰는 디바이스를 알면 `/machine/plcAddress/plcType/plcValue` 로 직접 읽을 수 있습니다.

## /machine/channel/blockSkipOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

블록 스킵(`/`) 스위치 상태입니다 (`true` = 켜짐). `channel` 필터. Fanuc·Siemens·Mitsubishi 가 지원합니다. Heidenhain 은 디메시가 이 스위치를 읽는 통로를 찾지 못해 상태 `-20` 입니다 (TNC7 사용 설명서에 따르면 프로그램 실행 모드에 블록 건너뛰기 스위치가 있습니다).

**스킵 레벨이 여러 개인 기종에서도 이 주소는 평범한 `/` 하나만 봅니다.** 블록 앞에 번호를 붙여(`/2`·`/3` …) 구간마다 다른 스위치로 건너뛰게 하는 기능이 있는 제어기들이 있는데 (Siemens 는 레벨 `0`~`9`, Mitsubishi 는 `BDT1`~`BDT9` 로 문서화합니다), 이 주소가 답하는 것은 언제나 **번호 없는 `/`** 입니다. 번호 붙은 레벨을 읽는 주소는 없습니다.

## /machine/channel/machineLockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

머신 록(축 이동 잠금) 상태입니다 (`true` = 켜짐). `channel` 필터. Fanuc·Siemens·Mitsubishi 가 지원하며 (Heidenhain 은 상태 `-20`), Siemens 는 프로그램 테스트(`progTestActive`) 상태입니다.

**Mitsubishi 는 이 신호를 축별로 내는데 이 주소는 채널 하나입니다.** 그래서 **그 채널의 전 축이 잠겼을 때만 `true`** 입니다. 일부 축만 잠긴 상태를 켜짐으로 부르면 "아무것도 움직이지 않는다" 로 읽혀 실제 가공을 시험 운전으로 오판하게 되기 때문입니다. 조작반 스위치는 전 축을 함께 움직이므로 통상적인 장비에서는 이 구분이 드러나지 않습니다.

## /machine/channel/rapidOverride
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

급속이송 오버라이드 (%)입니다. 반환 `float` + `unit:"%"`. 값은 **0.1% 자리까지** 냅니다. 대개 정수 퍼센트지만 0.1% 단계로 거는 장비에서는 `87.5` 처럼 소수가 나옵니다. **Fanuc 은 기계 제작사 래더가 고른 방식에 따라 갈립니다** (Connection Manual B-64483EN-1): 기본인 `ROV1`/`ROV2`(`G14`) 방식은 단계식이라 `100`/`50`/`25`/`0` 네 값만 나오고(`0` 은 F0, 파라미터 `1421` 의 속도), 1% 단계 방식(`HROV`, `G96`)이면 `0`~`100` 의 정수, 0.1% 단계 방식(`HROV` 와 `FHROV` 를 함께 켠 경우, `G96`·`G353`)이면 `87.5` 처럼 0.1% 자리까지 그대로 나옵니다. 어느 방식이든 100% 를 넘는 값은 `100` 으로 상한이 걸립니다. 다경로 장비는 그 경로의 신호(경로마다 `1000` 씩 밀린 주소)를 읽습니다. Siemens 는 연속값이며, 이송 오버라이드와 같이 스위치 값이 아니라 **유효값**입니다. PLC 유효 신호 `DB21.DBX6.6` 이 꺼진 동안은 다이얼과 무관하게 값 `100` 이 나옵니다 (언제 끄는지는 기계 제작사 래더에 달려 있고, 실측한 장비는 비상정지부터 조작반의 준비 버튼(MC READY)을 누를 때까지 껐습니다). 급속 전용 다이얼이 없는 장비는 래더가 이송 다이얼을 급속에도 적용하는 구성이 흔한데, 그렇게 적용될 때 100% 를 넘는 이송 오버라이드는 급속에는 제어기가 100% 로 상한을 겁니다 (Basic Functions 매뉴얼). 실측: 이송 다이얼 110 에서 이송 오버라이드 주소는 값 `110`, 이 주소는 값 `100` 이었습니다. **Mitsubishi 는 기계 제작사 래더가 고른 방식에 따라 갈립니다** (PLC 인터페이스 매뉴얼 IB-1501272): 방식 선택 신호 `ROVS`(`YC6F`)가 꺼져 있으면 코드 신호 `ROV1`/`ROV2`(`YC68`/`YC69`)라 `100`/`50`/`25`/`0`(매뉴얼의 `1%` 단계를 Fanuc 과 같이 `0` 으로 냅니다) 네 값이고, 켜져 있으면 계통별 레지스터 `R2502`(0~100% 1% 단위)라 연속값입니다.

두 기종의 `0` 은 "멈춤" 이 아니라 **그 장비가 정한 가장 느린 급속이송 단계**입니다 (조작반의 최저 단계). 실제 속도는 장비 설정에 달려 있어 이 주소로는 알 수 없습니다.

Heidenhain 은 `GetOverrideInfo` 가 주는 급속이송 오버라이드(정수 퍼센트)입니다. 테스트 환경에서 쓴 값이 그대로 읽혔습니다.

**쓰기는 Heidenhain 만 지원합니다** (`SetOverrideRapid`. 다른 기종은 상태 `-20`(미지원)). 정수 퍼센트를 `{"value": 80}` 처럼 씁니다. 소수부가 `0` 이면(`80.0`) 받고, `50.5` 처럼 소수부가 있거나 음수이면 상태 `-16`(쓰기 값 오류)입니다. 받는 범위는 장비가 정하며, **테스트 환경에서는 범위 밖의 값을 제어기가 가까운 끝으로 잘라 넣고 상태 `0` 을 돌려줬습니다** (테스트 환경에서 급속이송은 `0`~`100` 이라 `150` 을 쓰면 `100` 이 걸렸습니다). 무엇이 걸렸는지는 다시 읽어 확인하세요. 테스트 환경에서는 쓴 값이 읽기에 반영되기까지 0.1초쯤 걸려, 쓴 직후에 읽으면 이전 값이 나올 수 있었습니다. 쓴 값은 자동운전 중에도 걸리고 운전이 끝난 뒤에도 남습니다. 조작반에서 오버라이드를 조작하면 그 값이 걸리고, 다시 쓰면 쓴 값이 걸립니다 (마지막에 바꾼 쪽. 테스트 환경의 가상 다이얼로 확인했고, 실제 장비의 다이얼과는 확인하지 못했습니다). **쓰기 주의**: 운전 중인 장비의 급속이송 속도가 곧바로 바뀝니다.

## /machine/channel/feedOverride
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

이송 오버라이드 (%)입니다. 반환 `float` + `unit:"%"`. 값은 **0.1% 자리까지** 냅니다. 대개 정수 퍼센트지만 0.1% 단계로 거는 장비에서는 `87.5` 처럼 소수가 나옵니다. Fanuc 은 PMC `G12` 신호(`*FV0`~`*FV7`, 반전 2진 0~254%)에서 읽으며, 신호가 전부 꺼진 상태는 제어기와 같이 `0` 으로 냅니다 (Connection Manual B-64483EN-1). 스위치 신호 값이라 오버라이드 취소 신호(`OVC`)가 켜져 실제 배율이 100% 인 동안에도 스위치 값이 나오고, 제2 이송 오버라이드(`G13`)는 반영하지 않습니다. 다경로 장비는 그 경로의 신호(2경로 `G1012` 처럼 경로마다 `1000` 씩 밀린 주소)를 읽습니다. Siemens 는 `feedRateIpoOvr` 노드에서 읽는데, 이것은 스위치 값이 아니라 **보간기에 실제로 걸리는 유효값**입니다. PLC 유효 신호 `DB21.DBX6.7` 이 꺼진 동안 제어기는 오버라이드를 내부적으로 100% 로 두므로 (Basic Functions 매뉴얼) 다이얼과 무관하게 값 `100` 이 나옵니다. 이 신호를 언제 끄는지는 기계 제작사 래더에 달려 있습니다. 실측한 장비는 비상정지를 걸면 꺼지고 비상정지를 해제하고 리셋한 뒤에도 꺼진 채라, 조작반의 준비 버튼(그 장비 표기로 MC READY)을 누를 때까지 값 `100` 이 나오다가 준비가 켜지자 다이얼 값으로 복귀했습니다. 그동안에도 다이얼 위치 자체는 PLC 에 정상 도착하고 있었습니다. **Mitsubishi 는 기계 제작사 래더가 고른 방식을 따라 읽습니다** (PLC 인터페이스 매뉴얼 IB-1501272): 방식 선택 신호 `FVS`(`YC67`)가 꺼져 있으면 오버라이드 코드 신호(`YC60`~`YC64`, 0~300% 10% 단계)를, 켜져 있으면 계통별 레지스터 `R2500`(0~300% 1% 단위)을 읽습니다. 코드 신호가 전부 꺼진 상태는 제어기가 "이전 값 유지" 로 다루므로 읽을 값이 없어 상태 `-17` 입니다.

Heidenhain 은 `GetOverrideInfo` 가 주는 이송 오버라이드(정수 퍼센트)입니다. 테스트 환경에서 조작반 표시와 같은 값이 나왔습니다.

**쓰기는 Heidenhain 만 지원합니다** (`SetOverrideFeed`. 다른 기종은 상태 `-20`(미지원)). 정수 퍼센트를 `{"value": 80}` 처럼 씁니다. 소수부가 `0` 이면(`80.0`) 받고, `50.5` 처럼 소수부가 있거나 음수이면 상태 `-16`(쓰기 값 오류)입니다. 받는 범위는 장비가 정하며, **테스트 환경에서는 범위 밖의 값을 제어기가 가까운 끝으로 잘라 넣고 상태 `0` 을 돌려줬습니다** (테스트 환경에서 이송은 `0`~`150` 이라 `200` 을 쓰면 `150` 이 걸렸습니다). TNC7 사용 설명서('Cutting data')는 이송 오버라이드 다이얼로 0%~150% 사이를 바꿀 수 있다고 적고, 핸드휠을 켠 동안은 핸드휠의 이송 다이얼이 걸린다고 적습니다(Electronic handwheel 의 'Fundamentals'). 무엇이 걸렸는지는 다시 읽어 확인하세요. 테스트 환경에서는 쓴 값이 읽기에 반영되기까지 0.1초쯤 걸려, 쓴 직후에 읽으면 이전 값이 나올 수 있었습니다. 쓴 값은 자동운전 중에도 걸리고 운전이 끝난 뒤에도 남습니다. 조작반에서 오버라이드를 조작하면 그 값이 걸리고, 다시 쓰면 쓴 값이 걸립니다 (마지막에 바꾼 쪽. 테스트 환경의 가상 다이얼로 확인했고, 실제 장비의 다이얼과는 확인하지 못했습니다). **쓰기 주의**: 운전 중인 장비의 이송 속도가 곧바로 바뀝니다.

## /machine/channel/feedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

지령 이송속도 (F 지령값)입니다. 반환 `float`. Fanuc 은 모달 F, Siemens 는 `cmdFeedRateIpo`, Mitsubishi 는 `F command feed speed`(FA)입니다.

단위는 기계 설정을 따릅니다 (분당 이송이면 mm/min 또는 inch/min). Fanuc 은 프로그램의 F 를 그대로 내므로 회전당 이송(`G95`·`G99`)에서는 회전당 값입니다 (이송 방식은 `/machine/channel/gModalCategory/gModal?gModalCategory=5` 로 확인하세요). Siemens·Mitsubishi 의 회전당 이송에서 이 값의 단위는 확인하지 못했습니다. mm 인지 inch 인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Heidenhain** 은 Heidenhain 이 제어기에 둔 PLC API 정의에서 "프로그램한 분당 이송" 인 값을 PLC 데이터로 읽습니다. 마지막에 지령한 F 라 급속이송(FMAX) 블록에서도 그대로이고, 저희 테스트 환경(TNC7 프로그래밍 스테이션)에서 프로그램의 F 와 같았습니다 (mm/min). 조작반의 측정 단위를 inch 로 바꾸고 인치 프로그램에서 `F100`(10 inch/min)을 지령해도 mm/min 으로 왔습니다 (`254`). 인치 프로그램의 F 는 0.1 inch/min 단위입니다 (TNC7 사용 설명서 'Cutting data'). 회전당 이송(`FU`. 같은 설명서는 스핀들 1회전에 가는 mm 이고 주로 선삭에 쓴다고 적습니다)을 지령하면 지령 회전수로 환산한 분당 값입니다: 테스트 환경에서 `S1000` 에 `FU0.3` 이 `300` 이었고, 스핀들 오버라이드를 50% 로 낮춰 실제 회전수가 500 일 때도 `300` 이었습니다. 음수가 오면 뜻을 몰라 상태 `-17` 입니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요.

## /machine/channel/feedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

채널의 실제 이송속도입니다. **공구 끝이 프로그램 경로를 따라가는 속도**. 반환 `float`.

`F` 지령이 정하는 값이 이것입니다. 이동 방향이 바뀌어도 이 속도는 지령대로 유지되고, 오버라이드·가감속·코너 감속·피드홀드가 걸리면 그만큼 떨어집니다. "지금 지령대로 깎이고 있나" 는 이 값으로 판단합니다.

Fanuc 은 `actf`, Siemens 는 `actFeedRateIpo` 입니다. Mitsubishi 는 벤더가 실효 이송을 **자동 운전용과 수동 조작용으로 나눠** 주므로 둘을 함께 읽어 냅니다. 조그·핸들로 축을 움직이는 중에도 값이 나옵니다.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). **Fanuc 은 `unit` 을 붙입니다** (`mm/min`·`inch/min`). 값은 늘 분당 실제 이송입니다. 조작반의 F 표시는 파라미터 `3107#3`·`3191#5` 에 따라 회전당(`MM/REV`)이 되기도 하는데 (파라미터 설명서), 그때도 디메시 값은 분당이고 `unit` 이 그것을 알려 줍니다 (31i 벤치와 테스트 환경의 0i-F 선반에서 확인: 조작반 `0.10 MM/REV`, 디메시 `50.0` `mm/min`). 회전당 값이 필요하면 `/machine/channel/spindle/spindleSpeedActual` 로 나누되, 두 값은 서로 다른 순간에 읽힙니다. 연결할 때 제어기가 알려 준 단위이며, 그 단위를 알려 주는 함수(`cnc_rdspeed`)가 없는 옛 계열(FOCAS2 설명서의 지원 표로 Series 16/18/21·0i-A·15·15i 선반)에서는 `unit` 없이 제어기가 준 값 그대로입니다. **Fanuc 은 단위와 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 나와 10배 틀릴 수 있습니다. 다른 기종은 `unit` 을 붙이지 않습니다 (기계마다 달라 고정할 수 없음). `unit` 이 없으면 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼).

**Heidenhain** 은 Heidenhain 이 제어기에 둔 PLC API 정의에서 "현재 윤곽 이송" 인 값을 PLC 데이터로 읽습니다. 조작반 상태 줄의 F 와 같았습니다 (저희 테스트 환경 TNC7 프로그래밍 스테이션에서 오버라이드 90%·1% 일 때 270·3 mm/min). 조작반의 측정 단위를 inch 로 바꿔도 mm/min 으로 왔습니다 (조작반의 F 가 `10.6` inch/min 일 때 `270`). 회전당 이송(`S1000` 에 `FU0.3`)을 지령해도 분당으로 왔고 조작반의 F 와 같았습니다 (이송 오버라이드 90% 에서 `270`). TNC7 사용 설명서('Positions workspace')도 그 화면의 F 는 어느 단위로 프로그램하든 분당으로 바꿔 보인다고 적습니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요.

## /machine/channel/axisCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

채널의 **사용자 축 수**입니다. 연결 시 캐싱. `axis` 필터의 유효 범위가 `1`~이 값입니다.

기하축과 **비스핀들 보조축**(인덱싱 로터리 테이블·심압대 등)을 함께 세고 스핀들은 제외합니다. 스핀들은 `spindleCount` 와 `spindle` 필터가 담당합니다.

**경로마다 다릅니다.** 다경로 장비에서 채널별로 축 구성이 다르며, 축이 하나도 없는 경로는 `0` 입니다. 그 채널의 축 주소들은 Fanuc 에서 상태 `-20`, 그 밖의 기종에서 상태 `-18` 로 답합니다.

**Heidenhain** 은 연결 때 `GetChannelInfo` 가 준 채널의 축 목록에서 종류가 스핀들이 아닌 축(주축·보조축의 직선축과 회전축)을 셉니다. HEIDENHAIN DNC 레퍼런스는 이 목록의 축 이름과 종류가 운전 중에 바뀔 수 있다고 밝힙니다. 디메시는 연결 때 읽은 목록을 쓰므로 그 뒤의 변화는 다시 연결할 때 반영됩니다. 테스트 환경에서는 선삭 모드 동안 회전축이 선삭 스핀들로 쓰여 이 목록에서 빠졌습니다 (밀링 모드로 연결하면 `X Y Z A C`, 선삭 모드로 연결하면 `X Y Z A`). 모드를 바꿀 때 무엇이 바뀌는지는 기계 제작사가 정합니다 (TNC7 사용 설명서 'Switching the operating mode with FUNCTION MODE': 모드를 바꾸면 기계 제작사의 매크로가 돌고 그가 정한 기구 모델이 걸립니다). 축 목록은 밀링 모드에서 연결해 읽기를 권하고, 모드를 바꾼 뒤 목록을 새로 하려면 다시 연결하세요.

## /machine/channel/axis/axisName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

축 이름입니다 (예: `"X"`, `"Z1"`). 반환 `string`, 읽기 전용. `axis` 번호 ↔ 실제 축 대응을 확인할 때 사용.

**여기 나오는 이름을 `axis` 필터에 그대로 쓸 수 있습니다** (`axis=Z1`, `axis=X,Z`). 대소문자는 가리지 않으며, 그 채널에 없는 이름은 상태 `-18` 입니다. 이름은 채널 안에서만 뜻이 있어 채널마다 따로 봅니다.

출처는 Fanuc 서보 부하 미터 데이터의 축 이름(`cnc_rdsvmeter`, 연결 때 캐시), Siemens `/Channel/GeometricAxis/name`, Mitsubishi 축 파라미터 `#1013` 이고, Heidenhain 은 연결 때 `GetChannelInfo` 가 준 채널의 축 목록에 실린 이름(프로그래밍에 쓰는 축 이름)입니다. Heidenhain 의 `axis` 번호는 그 목록에서 스핀들을 뺀 순서입니다. HEIDENHAIN DNC 레퍼런스는 이 이름이 운전 중에 바뀔 수 있다고 밝힙니다. 디메시는 연결 때 읽은 이름을 쓰므로(`axis` 필터에 쓴 이름을 번호로 옮길 때도 같습니다) 그 뒤의 변화는 다시 연결할 때 반영됩니다. 테스트 환경에서는 선삭 모드 동안 회전축이 선삭 스핀들로 쓰여 이 목록에서 빠졌습니다 (밀링 모드로 연결하면 `X Y Z A C`, 선삭 모드로 연결하면 `X Y Z A`). 모드를 바꿀 때 무엇이 바뀌는지는 기계 제작사가 정합니다 (TNC7 사용 설명서 'Switching the operating mode with FUNCTION MODE': 모드를 바꾸면 기계 제작사의 매크로가 돌고 그가 정한 기구 모델이 걸립니다). 조작반 위치 창의 축 개수와 순서는 기계 설정이 정하므로(설명서 'Positions workspace') `axis` 번호가 그 줄 순서와 다를 수 있습니다. 이름으로 확인하세요. 축 목록은 밀링 모드에서 연결해 읽기를 권하고, 모드를 바꾼 뒤 목록을 새로 하려면 다시 연결하세요.

## /machine/channel/axis/machinePosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

축의 기계 좌표(machine coordinate)입니다. `axis` 필터로 축을 지정하며, 범위(`axis=1-3`)나 복수 지정(`axis=1,2`)이 가능합니다. 반환 타입은 `float` (64비트 배정밀도).

위치 계열 4종(machinePosition/workPosition/distanceToGo/relativePosition)은 모두 **실거리**: 장비 설정 단위(mm/inch) 그대로이며 조작반 표시와 일치합니다 (Fanuc 의 내부 정수 표현은 SDK 가 축별 소수점 배율로 정규화). Heidenhain 은 조작반을 inch 로 바꿔도 mm 라 그때는 조작반 표시와 다릅니다 (아래 참조). 네 값 모두 축이 원점을 확립한 뒤에만 유효합니다. 전원 투입 직후라면 `/machine/channel/axis/axisReferencedOn` 을 먼저 확인하세요 (Mitsubishi 는 그 주소가 상태 `-20` 이라 조작반에서 확인하거나, "지금 원점 위치에 있는가" 만 주는 `/machine/channel/axis/axisAtReferencePositionOn` 으로 갈음하세요) (미확립 상태에서도 그럴듯한 좌표값이 에러 없이 반환되므로, 값만 봐서는 가려낼 수 없습니다).

⚠️ **이 값은 공구 기준점(스핀들 끝단)의 좌표입니다.** `workPosition` 은 공구 선단이라 두 값의 차이에 **공구 길이 보정**이 들어갑니다. `machinePosition − 영점이동 = workPosition` 은 성립하지 않습니다 (활성 워크좌표계의 회전·배율·미러도 함께 걸립니다). 워크 좌표가 필요하면 직접 계산하지 말고 `workPosition` 을 읽으세요.

단위는 기계 설정을 따릅니다 (mm 또는 inch, 회전축은 도). **Fanuc 은 `unit` 을 붙입니다** (`mm`·`inch`·`deg`). 연결할 때 제어기가 알려 준 단위이고, 그 단위를 알려 주는 함수(`cnc_rdposition`)가 없는 옛 계열(FOCAS2 설명서의 지원 표로 Series 16/18/21·0i-A·15·15i 선반)에서만 빠집니다. **Fanuc 은 단위와 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 나와 10배 틀릴 수 있습니다. 다른 기종은 `unit` 을 붙이지 않습니다 (기계마다 달라 고정할 수 없음). `unit` 이 없으면 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼).

**Fanuc 의 기계 좌표는 G20/G21 을 따르지 않을 수 있습니다.** 파라미터 `3104#0` 이 `0`(기본값)이면 입력 단위와 무관하게 장비의 단위(파라미터 `1001#0`)로 나오고, `1` 이면 입력 단위를 따릅니다 (파라미터 설명서 `3104`). 그래서 mm 장비를 inch 입력으로 쓰면 이 주소만 mm 이고 `workPosition`·`relativePosition`·`distanceToGo` 는 inch 입니다 (시뮬레이터에서 확인). `unit` 이 그 차이를 알려 주고, `unit` 이 없는 옛 계열에서는 `gModalCategory=4` 대신 두 파라미터를 `/machine/channel/parameter/index/parameterValue` 로 읽어 확인하세요.

**Heidenhain** 은 기본 PLC 프로그램이 NC 에서 옮겨 두는 축 위치를 PLC 데이터로 읽습니다. 조작반 위치 창의 "실제 기준 위치(RFACTL)" 와 같은 값이었고 (저희 테스트 환경 TNC7 프로그래밍 스테이션에서 정지·이동 중에 대조), 단위는 mm(회전축은 도)였습니다: 저희 테스트 환경에서 조작반의 측정 단위를 inch 로 바꿔 조작반이 inch 로 보일 때도, 인치 프로그램(`BEGIN PGM … INCH`)을 실행할 때도 mm 로 왔습니다 (조작반이 `0.9754` inch 일 때 `24.7763`). RFACTL 은 기계 좌표계(M-CS)에서 잰 공구 위치이고(TNC7 사용 설명서 'Position displays'), 선삭 모드에서도 조작반 RFACTL 과 같았습니다 (X 에 지름 표시 `⌀` 가 붙어도 숫자는 밀링 모드와 같았습니다). **원점 정보가 없는 축은 상태 `-22` 입니다**: 기본 PLC 프로그램은 원점 정보가 있는 축만 이 값을 갱신하므로, 원점을 잃은 축(`axisReferencedOn` 이 `false`)의 값은 옛 값입니다 (테스트 환경은 늘 원점이 잡혀 있어 그 경우는 확인하지 못했습니다). **지금 채널에 배정되지 않은 축도 상태 `-22` 입니다**: 밀링·선삭을 오가는 장비에서 밀링 모드로 연결한 뒤 선삭 모드로 바뀌면 회전축이 선삭 스핀들로 쓰여 채널에서 빠지는데, 테스트 환경에서 그동안 이 값은 선삭 스핀들이 돌아도 `0` 에 머물렀습니다. 밀링 모드로 돌아오면 다시 값이 나옵니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요.

## /machine/channel/axis/workPosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

축의 공작물 좌표(절대 좌표)입니다. 반환 `float`.

이 값은 **공구 선단** 기준이며 활성 영점이동·회전·배율·미러·공구 길이 보정이 **모두 적용된 결과**입니다. 장비가 계산한 최종 좌표라 `machinePosition` 에서 직접 구할 필요가 없습니다 (뺄셈으로는 맞지 않습니다).

**표를 고친 값이 이 좌표에 반영되는 시점은 기종에 따라 다릅니다.** Siemens 는 워크오프셋과 공구 보정을 **활성화 시점에 확정**합니다. 워크오프셋은 G500·G54~G599 를 프로그래밍할 때 표(`$P_UIFR`)가 채널의 활성 프레임(`$P_IFRAME`)으로 복사되고(Basic Functions K2), 공구 오프셋 데이터의 변경은 다음에 T 또는 D 번호가 프로그래밍될 때 효력을 갖습니다(Programming Fundamentals. 즉시 반영은 `MD9440` 이 켜진 장비에서만이고, 매뉴얼이 충돌 위험을 경고하는 설정입니다). 그래서 운전 중 조작반에서 G54 나 공구 길이를 고쳐도 돌고 있는 프로그램의 이 값은 다음 활성화(또는 리셋 후 재시작)까지 그대로입니다 (840D sl 벤치에서 확인: G54 X 80.4→95.0, 공구 길이 100→105 모두 반영 없음). 좌표를 감시하는 앱은 "표를 바꿨는데 좌표가 안 움직인다" 를 이상으로 보지 마세요. Fanuc 은 공구 길이 보정의 반영 시점을 파라미터 `5001#6`(EVO: `0` 이면 다음 G43/H 블록, `1` 이면 다음 버퍼링 블록)이, 반경 보정은 `5001#4`(EVR)가 정하고, 워크오프셋 변경은 이 값에 즉시 반영됩니다 (시뮬레이터에서 확인). Mitsubishi 는 자동운전 중(싱글블록 정지 포함)에 고친 공구 보정량·워크좌표계 오프셋이 다음 블록 또는 몇 블록 뒤부터 유효합니다 (Instruction Manual). 다만 자동운전 중의 `workOffsetValue` 쓰기는 제어기가 거절해 상태 `-22` 입니다 (시뮬레이터에서 확인. 공구 보정용 파라미터 `#11017` 을 `1` 로 두어도 같았습니다). 공구 보정량 쓰기는 자동운전 중에도 받았습니다 (시뮬레이터에서 확인).

단위는 기계 설정을 따릅니다 (mm 또는 inch, 회전축은 도). **Fanuc 은 `unit` 을 붙입니다** (`mm`·`inch`·`deg`). 연결할 때 제어기가 알려 준 단위이고, 그 단위를 알려 주는 함수(`cnc_rdposition`)가 없는 옛 계열(FOCAS2 설명서의 지원 표로 Series 16/18/21·0i-A·15·15i 선반)에서만 빠집니다. **Fanuc 은 단위와 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 나와 10배 틀릴 수 있습니다. 다른 기종은 `unit` 을 붙이지 않습니다 (기계마다 달라 고정할 수 없음). `unit` 이 없으면 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼).

Heidenhain 은 `GetCutterLocation` 입니다. 레퍼런스가 공구 끝의 공작물 좌표계 위치로 밝히는 값이고, 좌표 이름으로 오므로 `axisName` 의 이름과 맞춰 냅니다. 테스트 환경에서는 영점 이동(`TRANS DATUM`)·회전(사이클 10)·배율(사이클 11)을 차례로 걸 때마다 조작반 위치 창의 공구 위치(NOML)와 같은 값이었고, TNC7 사용 설명서('Position displays')는 그 표시를 입력 좌표계(I-CS)의 위치라 부릅니다. 조작반의 위치 표시를 실제 기준 위치(RFACTL)로 바꿔도 이 값은 그대로였습니다. **선삭 모드에서는 X 가 조작반처럼 지름 값입니다** (테스트 환경에서 조작반 `X ⌀ -65.247` 일 때 `-65.247`. 설명서상 선삭의 X 좌표는 공작물의 지름입니다). 디메시는 HEIDENHAIN DNC 가 준 값을 그대로 냅니다. 저희 테스트 환경에서는 조작반의 측정 단위를 inch 로 바꾸고 인치 프로그램(`BEGIN PGM … INCH`)을 실행해도 이 값은 mm 로 왔습니다 (회전축은 도). HEIDENHAIN DNC 레퍼런스는 이 값이 인치로 올 수도 있다고 적지만, 저희는 그런 경우를 만들지 못해 확인하지 못했습니다. 디메시는 Heidenhain 에서 `gModalCategory` 를 읽지 않아(상태 `-20`) 위의 방법으로 단위를 확인할 수 없습니다. 연결 때 채널에 있던 축이 지금 좌표에 없으면 상태 `-22` 입니다: 테스트 환경에서 밀링 모드로 연결한 뒤 선삭 모드로 바뀌면 회전축 `C` 가 선삭 스핀들로 쓰여 그랬고, 밀링 모드로 돌아오자 다시 값이 나왔습니다 (`/machine/channel/axisCount`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch, 회전축은 도). **Fanuc 은 `unit` 을 붙입니다** (`mm`·`inch`·`deg`). 연결할 때 제어기가 알려 준 단위이고, 그 단위를 알려 주는 함수(`cnc_rdposition`)가 없는 옛 계열(FOCAS2 설명서의 지원 표로 Series 16/18/21·0i-A·15·15i 선반)에서만 빠집니다. **Fanuc 은 단위와 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 나와 10배 틀릴 수 있습니다. 다른 기종은 `unit` 을 붙이지 않습니다 (기계마다 달라 고정할 수 없음). `unit` 이 없으면 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼).

## /machine/channel/axis/distanceToGo
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

현재 블록에서 축의 **잔여 이동량**입니다. 반환 `float`.

단위는 기계 설정을 따릅니다 (mm 또는 inch, 회전축은 도). **Fanuc 은 `unit` 을 붙입니다** (`mm`·`inch`·`deg`). 연결할 때 제어기가 알려 준 단위이고, 그 단위를 알려 주는 함수(`cnc_rdposition`)가 없는 옛 계열(FOCAS2 설명서의 지원 표로 Series 16/18/21·0i-A·15·15i 선반)에서만 빠집니다. **Fanuc 은 단위와 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 나와 10배 틀릴 수 있습니다. 다른 기종은 `unit` 을 붙이지 않습니다 (기계마다 달라 고정할 수 없음). `unit` 이 없으면 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼).

**Heidenhain** 은 기본 PLC 프로그램이 NC 에서 옮겨 두는 잔여 거리를 PLC 데이터로 읽습니다. 조작반 위치 창의 잔여 거리(Δ)와 같은 값이었고 (저희 테스트 환경 TNC7 프로그래밍 스테이션에서 이송 중에 대조), 단위는 mm 였습니다: 저희 테스트 환경에서 조작반의 측정 단위를 inch 로 바꿔 조작반이 inch 로 보일 때도, 인치 프로그램(`BEGIN PGM … INCH`)을 실행할 때도 mm 로 왔습니다. **원점 정보가 없는 축은 상태 `-22` 입니다**: 기본 PLC 프로그램은 원점 정보가 있는 축만 이 값을 갱신하므로, 원점을 잃은 축(`axisReferencedOn` 이 `false`)의 값은 옛 값입니다 (테스트 환경은 늘 원점이 잡혀 있어 그 경우는 확인하지 못했습니다). **지금 채널에 배정되지 않은 축도 상태 `-22` 입니다**: 밀링·선삭을 오가는 장비에서 밀링 모드로 연결한 뒤 선삭 모드로 바뀌면 회전축이 선삭 스핀들로 쓰여 채널에서 빠지는데, 테스트 환경에서 그동안 이 값은 선삭 스핀들이 돌아도 `0` 에 머물렀습니다. 밀링 모드로 돌아오면 다시 값이 나옵니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요.

## /machine/channel/axis/totalWorkOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

지금 **실제로 걸려 있는 영점이동 총량**입니다 (축별 평행이동). 반환 `float`. **읽기 전용**. 조작반의 `Total WO` 행에 해당합니다.

`workOffsetValue` 가 **표에 저장된 값**이라면 이 주소는 **지금 적용되고 있는 값**입니다. 총량은 층으로 쌓입니다 (840D sl 벤치에서 잰 값):

```
표에 저장된 값      workOffsetValue?workOffset=G54     80.400   골라진 좌표계는 activeWorkOffset 으로 확인
+ 표 밖의 이동      기준 오프셋 (실제값 설정·터치오프)     20.000   조작반 Basic reference 행
                   기본 프레임 · 프로그램 TRANS · 사이클 프레임
= 지금 걸린 총량    totalWorkOffsetValue              100.400
```

기준 오프셋은 작업자가 JOG 에서 실제값 설정이나 터치오프·측정 사이클로 원점을 잡을 때 들어가는 몫이라(Siemens 시스템 프레임 `$P_SETFRAME`), 어느 좌표계를 골라도 항상 더해집니다. 역할은 Fanuc·Mitsubishi 의 `EXT` 와 같지만 Siemens 에서는 표 밖에 있어 `workOffsetValue` 로는 보이지 않습니다. "설정은 그대로인데 부품이 어긋난다" 를 진단할 때 두 주소를 비교하세요. 저장된 값만 읽으면 표 밖에서 더해진 몫이 보이지 않습니다.

**이 값은 활성화 시점에 확정됩니다.** G500·G54~G599 를 프로그래밍할 때 표(`$P_UIFR`)가 채널의 활성 프레임(`$P_IFRAME`)으로 복사되고, 그 뒤 표를 고쳐도 다음 활성화(또는 리셋 후 재시작)까지 이 값은 그대로입니다 (Basic Functions K2. 840D sl 벤치에서 확인: 운전 중 G54 X 를 80.4→95.0 으로 고쳐도 이 값은 고치기 전 값 그대로). 그 사이 `workOffsetValue` 는 새 값을 내므로, 두 주소가 다르면 "표는 바뀌었는데 아직 걸리지 않았다" 는 뜻입니다. 좌표를 감시하는 앱은 이 차이를 이상으로 보지 마세요.

⚠️ **공구 보정은 이 층에 없습니다.** 이 값은 "공작물 원점을 어디로 옮겼나" 까지이고, "공구가 얼마나 긴가" 는 그 다음 층입니다. 그래서 어느 기종에서든 `machinePosition` 에서 `workPosition` 을 빼도 이 값이 나오지 않습니다. 그 차이에는 공구 길이 보정이 섞여 공구축 값이 어긋납니다. 회전·배율·미러도 여기 없습니다. 공작물 좌표가 필요하면 `/machine/channel/axis/workPosition` 을 읽으세요. 장비가 그 모두를 적용한 결과입니다.

`workOffset` 필터를 받지 않습니다. "지금 걸린 것" 이라 지정자를 고를 대상이 없습니다. `axis=1-3` 확장을 지원하고, 쓰기는 지원하지 않습니다 (합산 결과라 되돌려 쓸 대상이 아닙니다).

**Siemens 전용입니다. 이 값은 디메시가 계산하는 것이 아니라 제어기가 스스로 들고 있는 것**이라 그렇습니다. SINUMERIK 은 활성 프레임의 합(`$P_ACTFRAME`)을 하나의 값으로 관리하고 OPC-UA 로 내줍니다. Fanuc(FOCAS2)·Mitsubishi(EZSocket)에서 디메시가 읽는 것은 `EXT`·`G54`~`G59` 의 **표**이고, 프로그램이 건 `G52`(로컬 좌표계)·`G92`(좌표계 설정) 시프트 양을 읽는 호출은 확인하지 못했습니다. 표를 더한 합은 그 두 시프트가 빠져 **그럴듯하지만 틀릴 수 있는 값**이라 디메시는 그 합을 내지 않습니다 (제어기가 한 값으로 들고 있지 않은 파생값을 지어내지 않는다는 규칙). 그래서 두 기종에서는 상태 `-20`(미지원)입니다. Heidenhain 도 상태 `-20`(미지원)입니다. 디메시가 읽는 것은 프리셋 **테이블**인데, 표를 고친 값은 프리셋을 다시 활성화할 때까지 걸리지 않고 빈 칸인 축은 직전 오프셋을 유지하므로(`workOffsetValue`) 표에서 지금 걸린 값을 만들 수 없습니다.

**Fanuc·Mitsubishi 에서 필요한 것을 얻는 법**: ① 좌표가 목적이면 `/machine/channel/axis/workPosition` 을 읽으세요. 제어기가 `G52`·`G92`·공구 보정까지 전부 적용해 계산한 값이라 이 주소가 없어도 됩니다. ② 설정 오프셋 자체가 목적이면 `workOffsetValue` 로 `EXT` 와 골라진 좌표계(`/machine/channel/activeWorkOffset` 으로 확인)를 읽어 더하세요. Fanuc 은 표를 고치면 즉시 적용되므로 그 합이 곧 설정 오프셋의 유효값입니다. Mitsubishi 는 자동운전 중에 고친 값이 다음 블록 또는 몇 블록 뒤부터 유효해(Instruction Manual), 그 사이에는 합이 지금 걸린 값보다 앞서 있을 수 있습니다. ③ 프로그램이 `G52`·`G92` 를 걸었는지는 `gModalList`·`gModalCategory` 로 알 수 있지만, 그 양을 읽는 통로는 확인하지 못했습니다. 그것까지 든 총량이 필요하면 `workPosition` 과 `machinePosition` 의 관계로 판단하되 공구 보정이 섞인다는 점을 감안하세요.

**Heidenhain 에서 필요한 것을 얻는 법**: ① 좌표가 목적이면 `/machine/channel/axis/workPosition` 을 읽으세요. ② 프리셋 자체가 목적이면 `/machine/channel/activeWorkOffset` 이 내는 번호를 `workOffset` 에 넣어 `workOffsetValue` 를 읽으세요 (회전은 `workOffsetRotation`). 빈 칸(`null`)인 축은 표로 알 수 없고, 표를 고친 뒤 다시 활성화하기 전에는 표가 지금 걸린 값보다 앞서 있을 수 있습니다. 지금 걸린 프리셋과 변환은 조작반의 상태 화면이 보여 주지만(TNC7 사용 설명서 'Status workspace'), 그 합을 HEIDENHAIN DNC 로 읽는 통로는 확인하지 못했습니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

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

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**Mitsubishi 도 상태 `-20` 입니다.**

## /machine/channel/axis/axisLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

축(서보) 부하율입니다. 반환 `float` + `unit:"%"` (네 기종 동일). Fanuc 은 서보 부하 미터, Siemens 는 드라이브 부하(`$VA_LOAD`, PROFIdrive 드라이브에서만 제공), Mitsubishi 는 서보 모니터의 부하 전류(정격 대비 비율)입니다. 셋 다 **실측값의 현재값**입니다.

**Siemens 는 머신 데이터 `36730` `$MA_DRIVE_SIGNAL_TRACKING` 이 `1` 인 축에서만 값이 나옵니다.** MD 목록 매뉴얼은 이 값이 켜져 있고 드라이브가 그 값을 보낼 때만 제어기가 받는다고 적습니다. `0` 인 축은 늘 `0` 입니다 (840D sl 벤치: `0` 일 때 늘 `0` 이던 값이, `1` 로 바꾸고 전원을 다시 넣자 축과 스핀들이 움직이는 동안 나왔습니다).

**그래서 Siemens 에서 이 주소를 감시에 쓰기 전에 머신 데이터 `36730` 을 보고, 값이 실제로 나오는지 확인하세요.** 읽기만으로는 `0` 과 "값 없음" 을 가를 수 없습니다. 축을 움직이면서 `axisCurrent` 를 함께 읽어, 전류는 움직이는데 이 값이 계속 `0` 이면 그 기계에서는 이 주소로 부하를 감시할 수 없습니다.

**`axisCurrent` 와 같은 물리량입니다.** 서보는 토크가 전류에 비례하므로 부하를 재는 것이 곧 전류를 재는 것이고, 이쪽은 그 값을 **모터 정격 연속전류로 나눈 비율**입니다. 그래서 기계·축이 달라도 비교되는 반면(80% 는 어디서나 80%), 절대 전류값이 필요하면 `axisCurrent` 를 쓰세요. 둘 사이 환산에는 그 모터의 정격 전류가 필요한데 디메시는 그 값을 내지 않으므로, **한쪽만 지원하는 기종에서는 다른 쪽을 계산해낼 수 없습니다.**

**Mitsubishi 의 이 값은 실장비에서 확인하지 못했습니다.** 저희 테스트 환경(시뮬레이터)에서는 읽히는 것까지만 확인했고, 값은 늘 `0` 이었습니다.

**Heidenhain** 은 기본 PLC 프로그램이 드라이브에서 읽어 두는 모터 부하율을 PLC 데이터로 읽습니다. 조작반의 드라이브 진단 표(Utilization [%])에 보이는 값입니다. 저희 테스트 환경(TNC7 프로그래밍 스테이션)에서는 PLC 프로그램이 드라이브 대신 정해진 값을 넣어 두므로, 조작반의 드라이브 진단 표와 같은 값인 것까지 확인했고 실제 드라이브의 값은 확인하지 못했습니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요.

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

**Fanuc·Siemens 는 상태 `-20` 입니다.** 디메시는 이 값을 Mitsubishi 드라이브 모니터에서 읽으며, 다른 두 기종에서 같은 값을 읽는 통로는 확인하지 못했습니다.

**Mitsubishi 의 이 값은 실장비에서 확인하지 못했습니다.** 저희 테스트 환경(시뮬레이터)에서는 읽히는 것까지만 확인했고, 값은 늘 `0` 이었습니다.

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
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

축 모터 온도입니다. 반환 `float` + `unit:"°C"` (네 기종 동일). Fanuc 은 진단 308번, Siemens 는 드라이브 파라미터 `R0035`, Mitsubishi 는 서보 드라이브 모니터의 모터 온도입니다.

**Siemens**: 값이 드라이브에서 오므로, 그 축에 드라이브가 배정되지 않은 채널에서는 상태 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

**Mitsubishi 의 이 값은 실장비에서 확인하지 못했습니다.** 저희 테스트 환경(시뮬레이터)에서는 읽히는 것까지만 확인했고, 값은 늘 `0` 이었습니다.

**Heidenhain** 은 기본 PLC 프로그램이 드라이브에서 읽어 두는 모터 온도를 PLC 데이터로 읽습니다. 조작반의 드라이브 진단 표(Temperature [°C])에 보이는 값입니다. 저희 테스트 환경(TNC7 프로그래밍 스테이션)에서는 PLC 프로그램이 드라이브 대신 정해진 값을 넣어 두므로, 조작반의 드라이브 진단 표와 같은 값인 것까지 확인했고 실제 드라이브의 값은 확인하지 못했습니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요.

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

**Fanuc**: 진단 `4901`. **Siemens**: `$VA_POWER`(`vaPower`)이며 **PROFIdrive 드라이브에서만 값이 있습니다** (`axisLoad` 와 같은 제약). 그렇지 않은 축은 `0` 입니다.

**Siemens 는 머신 데이터 `36730` `$MA_DRIVE_SIGNAL_TRACKING` 이 `1` 인 축에서만 값이 나옵니다.** MD 목록 매뉴얼은 이 값이 켜져 있고 드라이브가 그 값을 보낼 때만 제어기가 받는다고 적습니다. `0` 인 축은 늘 `0` 입니다 (840D sl 벤치: `0` 일 때 늘 `0` 이던 값이, `1` 로 바꾸고 전원을 다시 넣자 축과 스핀들이 움직이는 동안 나왔습니다).

**그래서 Siemens 에서 이 주소를 감시에 쓰기 전에 머신 데이터 `36730` 을 보고, 값이 실제로 나오는지 확인하세요.** 읽기만으로는 `0` 과 "값 없음" 을 가를 수 없습니다. 축을 움직이면서 `axisCurrent` 를 함께 읽어, 전류는 움직이는데 이 값이 계속 `0` 이면 그 기계에서는 이 주소로 전력을 감시할 수 없습니다.

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
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

축이 **기계 원점(레퍼런스)을 확립했는지**입니다. 반환 `boolean`. **읽기 전용**.

`false` 인 축은 좌표계가 아직 서 있지 않은 상태입니다. **위치 주소들(`machinePosition`·`workPosition`·`relativePosition`·`distanceToGo`)이 그럴듯한 숫자를 주더라도 무의미할 수 있습니다.** 값이 없다는 에러가 나는 것이 아니라 기준이 서지 않은 좌표가 그대로 반환되므로, 전원 투입 직후의 위치를 소비하는 쪽은 이 값을 먼저 확인하세요. 증분형 엔코더 장비는 원점복귀를 마쳐야 좌표가 성립합니다.

**절대위치 엔코더 장비는 전원을 꺼도 기준을 잃지 않습니다.** 그런 축은 전원 투입 직후부터 `true` 라, `false` 를 한 번도 보지 못할 수 있습니다. 고장이 아니라 정상입니다. 어느 방식인지 알 필요는 없습니다. **위치를 쓰기 전에 이 값을 확인한다**는 규칙 하나면 양쪽 다 옳게 동작합니다.

한 번 확립되면 축이 어디로 움직여도 `true` 로 유지됩니다. "지금 원점 위치에 있는가" 라는 순간 상태가 아니라 **좌표계 유효성**입니다. 그 순간 상태는 형제 주소 `/machine/channel/axis/axisAtReferencePositionOn` 이 따로 냅니다.

Fanuc 은 CNC→PMC 표준 신호 ZRF(`F120` 의 축별 비트)를, Siemens 는 `refPtStatus` 를 읽습니다 (둘 다 테스트 환경에서 확인). **Mitsubishi 는 상태 `-20` 입니다**: 디메시가 이 제어기에서 읽는 것은 "지금 원점 위치에 있는가"(조작반의 `#1` 표시·PLC `ZP1n`·`GetAxisStatus`)와 절대위치 검출계의 원점 초기 설정 완료(`ZSF`)이고, 한 번 확립되면 유지되는 좌표계 확립 상태는 그 둘로 만들 수 없습니다 (테스트 환경에서 원점 복귀 뒤 축을 움직이자 그 비트가 바로 꺼지는 것을 확인). 그 순간 상태는 `/machine/channel/axis/axisAtReferencePositionOn` 이 Fanuc·Mitsubishi 공통 뜻으로 냅니다. Fanuc 은 축 16개까지 지원하며, 다경로 장비에서는 그 경로의 신호를 읽습니다.

Heidenhain 은 PLC API 의 축별 원점 정보 심볼을 읽습니다. Heidenhain 이 제어기에 둔 PLC API 정의에서 이 심볼은 그 축의 원점 정보가 있다는 뜻이라 위 뜻과 같습니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요. 저희 테스트 환경(TNC7 프로그래밍 스테이션)은 늘 원점이 잡혀 있어 `true` 만 확인했습니다. TNC7 사용 설명서에 따르면 절대 엔코더를 쓰는 기계는 원점을 잡을 필요가 없고, 증분형 엔코더를 쓰는 기계는 전원을 켠 뒤 원점 복귀 화면이 열려 모든 축의 원점을 잡기 전에는 프로그램 실행으로 바꿀 수 없습니다.

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

**Fanuc**: 영역 I 이 파라미터 `1320`, II 가 `1326`, III~VIII 이 `1350`~`1360`(짝수) 입니다. 파라미터 `1301#0`(`DLM`)이 `1` 이면 축·방향별 신호 `+EXLx` 가 I/II 를 고르고, 아니면 `1300#2`(`LMS`)가 `1` 일 때 `EXLM`·`EXLM2`·`EXLM3` 세 신호의 조합이 I~VIII 를 고릅니다. 둘 다 `0` 이면 언제나 I 입니다. 이 주소는 그 규칙대로 고른 영역의 값을 냅니다. 영역 III~VIII 은 영역 확장 옵션이 있어야 제어기가 씁니다. 옵션이 없는 장비에서 PLC 가 `EXLM2`·`EXLM3` 를 세우면 제어기는 무시하는데 이 주소는 신호대로 고른 영역의 값을 내므로 실제와 달라집니다 (31i 벤치에서 확인). 자릿수는 장비가 알려주는 값을 그대로 쓰므로 `/machine/channel/parameter/index/parameterValue` 로 같은 번호를 읽은 값과 항상 일치합니다. **직경 지령 축(선반 X 등)은 직경값**입니다 (파라미터 매뉴얼 B-64490EN, 각 파라미터의 NOTE 1). 검사는 원점 복귀 뒤부터이고, 전원 투입 직후부터 검사하도록 설정한 장비(`1311#0`=`1`)에서만 `1300#6` 이 원점 복귀 전 검사 여부를 정합니다.

**Siemens**: 1차 소프트웨어 리밋 스위치 `$MA_POS_LIMIT_PLUS`(MD 36110)와 2차 `$MA_POS_LIMIT_PLUS2`(MD 36130) 가운데 축 인터페이스 신호 `DBX12.3` 이 고른 쪽입니다 (`1` 이면 2차). 기계축 좌표계 값이고 원점 복귀 뒤부터 모든 모드에서 유효합니다 (`PRESET` 뒤에는 다시 원점 복귀할 때까지 꺼지고, 모듈로 회전축은 감시하지 않습니다. 기능 매뉴얼 A3). 위반하면 알람 `10720`(블록 준비 단계)·`10620`(실행 중)·`10621`(JOG 로 스위치에 닿음)이 뜹니다 (진단 매뉴얼 3장 NC 알람). 채널 축과 기계축의 대응을 읽을 수 없는 드문 구성에서는 상태 `-20` 입니다.

**Mitsubishi**: 파라미터 `#2014` (`OT+`) 입니다. 영역이 한 벌뿐이라 설정값과 같은 값이고, `/machine/channel/parameter/index/parameterValue?parameter=2014` 로 읽은 값과 항상 일치합니다. `#2013` 과 `#2014` 가 0 이 아닌 같은 값이면 제어기는 이 리미트를 무효로 봅니다 (Alarm/Parameter Manual IB-1501279). 실사용 범위를 더 좁히는 셋업 레벨 울타리는 `axisWorkAreaLimitPositive` 입니다.

**켜고 끄는 스위치는 없습니다.** 그래서 켜짐을 묻는 형제 주소도 없고, 유효 여부는 아래 "두 값이 같거나 뒤집혀 있을 때" 규칙처럼 값의 모양이 정합니다. 원점 복귀 후 항상 적용되는 설치 레벨 울타리입니다 (작업 영역 제한의 `axisWorkAreaLimitPositiveOn` 같은 것이 여기엔 없습니다). 기종별로 대신 있는 것이 위의 영역 선택입니다.

**두 값이 같거나 뒤집혀 있을 때의 뜻은 기종마다 다릅니다.** 값의 모양으로 "미설정" 을 판정하려면 기종을 가리세요. **Fanuc**: 양쪽이 같은 값이면 **전 영역이 금지**됩니다 (조작 설명서 CAUTION 1). 양의 값이 음의 값보다 작게 뒤집혀 있으면 검사가 걸리지 않습니다 (조작 설명서 CAUTION 2 는 영역 크기를 잘못 설정하면 스트로크 제한이 없어진다고 밝힙니다). 테스트 환경에서 원점을 확립한 뒤 양의 값을 음의 값보다 작게 설정하자, 자동운전 중 축이 두 경계를 양방향으로 막힘 없이 지나갔습니다. 값을 바꾸는 순간 현재 위치에 따라 `OT0500` 알람이 한 번 뜰 수 있으며 RESET 으로 풀립니다. **Mitsubishi**: `#2013`=`#2014`(0 이 아닌 같은 값)이면 **무효**입니다. 뒤집혀 있어도 검사는 꺼지지 않고 **방향마다 자기 값으로** 걸립니다. 테스트 환경에서 원점을 확립한 뒤 두 값을 뒤집자, 자동운전 중 양의 방향 이동은 `#2014` 에서, 음의 방향 이동은 `#2013` 에서 멈췄고, 이미 그 값을 넘어선 자리에서는 그 방향으로 출발하지 못했습니다 (스트로크 끝 경고 `M01 0007`). **Siemens**: 두 값의 관계에 따른 규칙이 없습니다. 기본값이 ±1.0e8 이라 사실상 무제한이고 각 방향이 독립입니다. 같은 값이 Fanuc 에선 "잠김", Mitsubishi 에선 "무효" 로 정반대이고, 뒤집힌 값은 Fanuc 에선 검사를 없애지만 Mitsubishi 에선 그렇지 않으므로 기종 무관 규칙 하나로 읽지 마세요.

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

**Fanuc**: 영역 I 이 파라미터 `1321`, II 가 `1327`, III~VIII 이 `1351`~`1361`(홀수) 입니다. 파라미터 `1301#0`(`DLM`)이 `1` 이면 축·방향별 신호 `-EXLx` 가 I/II 를 고르고, 아니면 `1300#2`(`LMS`)가 `1` 일 때 `EXLM`·`EXLM2`·`EXLM3` 세 신호의 조합이 I~VIII 를 고릅니다. 둘 다 `0` 이면 언제나 I 입니다. 이 주소는 그 규칙대로 고른 영역의 값을 냅니다. 영역 III~VIII 은 영역 확장 옵션이 있어야 제어기가 씁니다. 옵션이 없는 장비에서 PLC 가 `EXLM2`·`EXLM3` 를 세우면 제어기는 무시하는데 이 주소는 신호대로 고른 영역의 값을 내므로 실제와 달라집니다 (31i 벤치에서 확인). 자릿수는 장비가 알려주는 값을 그대로 쓰므로 `/machine/channel/parameter/index/parameterValue` 로 같은 번호를 읽은 값과 항상 일치합니다. **직경 지령 축(선반 X 등)은 직경값**입니다 (파라미터 매뉴얼 B-64490EN, 각 파라미터의 NOTE 1). 검사는 원점 복귀 뒤부터이고, 전원 투입 직후부터 검사하도록 설정한 장비(`1311#0`=`1`)에서만 `1300#6` 이 원점 복귀 전 검사 여부를 정합니다.

**Siemens**: 1차 소프트웨어 리밋 스위치 `$MA_POS_LIMIT_MINUS`(MD 36100)와 2차 `$MA_POS_LIMIT_MINUS2`(MD 36120) 가운데 축 인터페이스 신호 `DBX12.2` 가 고른 쪽입니다 (`1` 이면 2차). 기계축 좌표계 값이고 원점 복귀 뒤부터 모든 모드에서 유효합니다 (`PRESET` 뒤에는 다시 원점 복귀할 때까지 꺼지고, 모듈로 회전축은 감시하지 않습니다. 기능 매뉴얼 A3). 위반하면 알람 `10720`(블록 준비 단계)·`10620`(실행 중)·`10621`(JOG 로 스위치에 닿음)이 뜹니다 (진단 매뉴얼 3장 NC 알람). 채널 축과 기계축의 대응을 읽을 수 없는 드문 구성에서는 상태 `-20` 입니다.

**Mitsubishi**: 파라미터 `#2013` (`OT-`) 입니다. 영역이 한 벌뿐이라 설정값과 같은 값이고, `/machine/channel/parameter/index/parameterValue?parameter=2013` 로 읽은 값과 항상 일치합니다. `#2013` 과 `#2014` 가 0 이 아닌 같은 값이면 제어기는 이 리미트를 무효로 봅니다 (Alarm/Parameter Manual IB-1501279). 실사용 범위를 더 좁히는 셋업 레벨 울타리는 `axisWorkAreaLimitNegative` 입니다.

**켜고 끄는 스위치는 없습니다.** 그래서 켜짐을 묻는 형제 주소도 없고, 유효 여부는 아래 "두 값이 같거나 뒤집혀 있을 때" 규칙처럼 값의 모양이 정합니다. 원점 복귀 후 항상 적용되는 설치 레벨 울타리입니다 (작업 영역 제한의 `axisWorkAreaLimitNegativeOn` 같은 것이 여기엔 없습니다). 기종별로 대신 있는 것이 위의 영역 선택입니다.

**두 값이 같거나 뒤집혀 있을 때의 뜻은 기종마다 다릅니다.** 값의 모양으로 "미설정" 을 판정하려면 기종을 가리세요. **Fanuc**: 양쪽이 같은 값이면 **전 영역이 금지**됩니다 (조작 설명서 CAUTION 1). 양의 값이 음의 값보다 작게 뒤집혀 있으면 검사가 걸리지 않습니다 (조작 설명서 CAUTION 2 는 영역 크기를 잘못 설정하면 스트로크 제한이 없어진다고 밝힙니다). 테스트 환경에서 원점을 확립한 뒤 양의 값을 음의 값보다 작게 설정하자, 자동운전 중 축이 두 경계를 양방향으로 막힘 없이 지나갔습니다. 값을 바꾸는 순간 현재 위치에 따라 `OT0500` 알람이 한 번 뜰 수 있으며 RESET 으로 풀립니다. **Mitsubishi**: `#2013`=`#2014`(0 이 아닌 같은 값)이면 **무효**입니다. 뒤집혀 있어도 검사는 꺼지지 않고 **방향마다 자기 값으로** 걸립니다. 테스트 환경에서 원점을 확립한 뒤 두 값을 뒤집자, 자동운전 중 양의 방향 이동은 `#2014` 에서, 음의 방향 이동은 `#2013` 에서 멈췄고, 이미 그 값을 넘어선 자리에서는 그 방향으로 출발하지 못했습니다 (스트로크 끝 경고 `M01 0007`). **Siemens**: 두 값의 관계에 따른 규칙이 없습니다. 기본값이 ±1.0e8 이라 사실상 무제한이고 각 방향이 독립입니다. 같은 값이 Fanuc 에선 "잠김", Mitsubishi 에선 "무효" 로 정반대이고, 뒤집힌 값은 Fanuc 에선 검사를 없애지만 Mitsubishi 에선 그렇지 않으므로 기종 무관 규칙 하나로 읽지 마세요.

## /machine/channel/axis/axisSoftLimitAreaCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

이 제어기가 소프트 리미트(저장형 스트로크 체크 1)에 대해 가진 **영역(값 한 벌)의 수**입니다. `axisSoftLimitArea/axisSoftLimitPositive`·`…Negative` 의 `axisSoftLimitArea` 필터가 받는 상한이며, 그 폴더를 `1` 부터 이 값까지 순회하면 모든 영역의 설정값을 읽을 수 있습니다. 값은 축과 무관하지만 `axis` 필터는 다른 축 주소처럼 범위를 검증합니다.

- **Fanuc**: 영역 III 의 파라미터(`1350`)가 파라미터 표에 있으면 `8`, 없으면 `2` 입니다. 연결할 때 한 번 확인합니다. 저희가 시험한 장비에서는 영역 확장 옵션과 무관하게 `1350`~`1361` 이 표에 있어 `8` 이 나왔는데, III~VIII 를 실제로 고를 수 있는지는 그 옵션에 달려 있습니다 (옵션이 없으면 제어기가 `EXLM2`·`EXLM3` 신호를 보지 않습니다). 옵션 유무를 FOCAS2 로 가려내는 방법은 아직 확인하지 못해, 이 값은 표의 크기입니다.
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

- **Fanuc**: PLC 선택 신호에서 계산합니다. 파라미터 `1301#0`(`DLM`)이 `1` 이면 `+EXLx`(`Gn104` 의 축 비트)가 `0`=I, `1`=II 를 고르고, 아니면 `1300#2`(`LMS`)가 `1` 일 때 `EXLM3`·`EXLM2`·`EXLM`(`Gn531.7`·`Gn531.6`·`Gn007.6`)의 세 비트 값에 `1` 을 더한 것(`000`=1 … `111`=8)입니다. 둘 다 `0` 이면 `1` 입니다. 영역 확장 옵션이 없는 장비에서는 제어기가 `EXLM2`·`EXLM3` 를 보지 않는데 이 값은 신호 그대로 계산하므로, 그런 장비의 PLC 가 그 두 신호를 세워 두었다면 제어기가 실제로 쓰는 영역과 달라집니다 (31i 벤치에서 확인).
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

- **Fanuc**: PLC 선택 신호에서 계산합니다. 파라미터 `1301#0`(`DLM`)이 `1` 이면 `-EXLx`(`Gn105` 의 축 비트)가 `0`=I, `1`=II 를 고르고, 아니면 `1300#2`(`LMS`)가 `1` 일 때 `EXLM3`·`EXLM2`·`EXLM`(`Gn531.7`·`Gn531.6`·`Gn007.6`)의 세 비트 값에 `1` 을 더한 것(`000`=1 … `111`=8)입니다. 둘 다 `0` 이면 `1` 입니다. 영역 확장 옵션이 없는 장비에서는 제어기가 `EXLM2`·`EXLM3` 를 보지 않는데 이 값은 신호 그대로 계산하므로, 그런 장비의 PLC 가 그 두 신호를 세워 두었다면 제어기가 실제로 쓰는 영역과 달라집니다 (31i 벤치에서 확인).
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

- **Fanuc**: 영역 I `1320`, II `1326`, III~VIII `1350`·`1352`·…·`1360` 입니다. `/machine/channel/parameter/index/parameterValue` 로 같은 번호를 읽은 값과 항상 일치합니다. **직경 지령 축(선반 X 등)은 직경값**입니다 (파라미터 매뉴얼 B-64490EN, 각 파라미터의 NOTE 1).
- **Siemens**: 영역 `1` 은 `$MA_POS_LIMIT_PLUS`(MD 36110), `2` 는 `$MA_POS_LIMIT_PLUS2`(MD 36130) 입니다. **쓴 값의 효력은 NEW CONF 등급**입니다. 그 축이 멈추고 축이 속한 모드 그룹의 채널이 리셋 상태일 때 활성화되며(조작반 'MD 활성화'·`NEWCONF` 명령과 같은 절차), 운전 중에 쓰면 그때까지 이전 값이 강제됩니다. 채널 축과 기계축의 대응을 읽을 수 없는 드문 구성에서는 상태 `-20` 입니다.
- **Mitsubishi**: 영역 `1` 만 있고 파라미터 `#2014`(`OT+`) 입니다.

**쓰기는 장비의 쓰기 허가 상태를 따릅니다.** Fanuc 은 파라미터 쓰기가 허가되지 않은 상태면, Mitsubishi 는 그 계통이 자동운전 중(일시정지 포함)이면 상태 `-22`(기계 상태)로 거절되고, Siemens 는 머신 데이터라 보호 레벨 등으로 거절되면 상태 `-17` 에 벤더 사유가 실립니다. Mitsubishi 에서 설정 범위 밖의 값은 상태 `-16` 이고(테스트 환경에서 `#2014` 로 확인), 설정 단위 `#1003` 보다 잘게 쓴 자리는 제어기가 반올림해 저장한 뒤 상태 `0` 을 돌려주므로(`parameterValue` 참조) 무엇이 들어갔는지는 다시 읽어 확인하세요. 이 값을 잘못 쓰면 축이 필요한 곳까지 못 가거나 반대로 보호가 느슨해지므로, 실제 장비에서는 값을 먼저 읽어 두고 바꾸세요. 두 값이 같거나 뒤집혀 있을 때의 기종별 뜻은 `axisSoftLimitPositive` 에 적혀 있습니다.

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

- **Fanuc**: 영역 I `1321`, II `1327`, III~VIII `1351`·`1353`·…·`1361` 입니다. `/machine/channel/parameter/index/parameterValue` 로 같은 번호를 읽은 값과 항상 일치합니다. **직경 지령 축(선반 X 등)은 직경값**입니다 (파라미터 매뉴얼 B-64490EN, 각 파라미터의 NOTE 1).
- **Siemens**: 영역 `1` 은 `$MA_POS_LIMIT_MINUS`(MD 36100), `2` 는 `$MA_POS_LIMIT_MINUS2`(MD 36120) 입니다. **쓴 값의 효력은 NEW CONF 등급**입니다. 그 축이 멈추고 축이 속한 모드 그룹의 채널이 리셋 상태일 때 활성화되며(조작반 'MD 활성화'·`NEWCONF` 명령과 같은 절차), 운전 중에 쓰면 그때까지 이전 값이 강제됩니다. 채널 축과 기계축의 대응을 읽을 수 없는 드문 구성에서는 상태 `-20` 입니다.
- **Mitsubishi**: 영역 `1` 만 있고 파라미터 `#2013`(`OT-`) 입니다.

**쓰기는 장비의 쓰기 허가 상태를 따릅니다.** Fanuc 은 파라미터 쓰기가 허가되지 않은 상태면, Mitsubishi 는 그 계통이 자동운전 중(일시정지 포함)이면 상태 `-22`(기계 상태)로 거절되고, Siemens 는 머신 데이터라 보호 레벨 등으로 거절되면 상태 `-17` 에 벤더 사유가 실립니다. Mitsubishi 에서 설정 범위 밖의 값은 상태 `-16` 이고(테스트 환경에서 `#2014` 로 확인), 설정 단위 `#1003` 보다 잘게 쓴 자리는 제어기가 반올림해 저장한 뒤 상태 `0` 을 돌려주므로(`parameterValue` 참조) 무엇이 들어갔는지는 다시 읽어 확인하세요. 이 값을 잘못 쓰면 축이 필요한 곳까지 못 가거나 반대로 보호가 느슨해지므로, 실제 장비에서는 값을 먼저 읽어 두고 바꾸세요. 두 값이 같거나 뒤집혀 있을 때의 기종별 뜻은 `axisSoftLimitNegative` 에 적혀 있습니다.

## /machine/channel/axis/axisWorkAreaLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

그 축의 **작업 영역 제한 양의 방향 좌표**입니다. `axisSoftLimitPositive`(장비 설치 때 고정되는 한계) 안쪽에 셋업마다 조정하는 두 번째 울타리로, Fanuc·Mitsubishi 는 기계좌표, Siemens 는 기본 좌표계(BCS) 기준이고 읽기·쓰기 모두 됩니다.

**Fanuc**: 저장형 스트로크 체크 2 (파라미터 `1322`) 입니다. 이 기능은 설정에 따라 "이 안에 머물러라" 도 되고 "여기 들어가지 마라"(척 배리어 등) 도 되는데, **이 주소는 "이 안에 머물러라" 뜻만 약속합니다.** 그래서 장비가 진입 금지 상자로 설정돼 있으면(파라미터 `1300#0`=`0`) 값을 내보내지 않고 상태 `-20` 으로 거절합니다. 그 값을 "이 사이면 안전" 으로 계산하면 정확히 거꾸로가 되기 때문입니다 (그때는 `axisWorkAreaLimitOn` 도 같은 상태 `-20` 입니다). **직경 지령 축(선반 X 등)은 직경값**입니다 (파라미터 매뉴얼 B-64490EN, 각 파라미터의 NOTE 1). 축별 사용 여부(`1310#0`)와 모달(`G22`/`G23`)은 `axisWorkAreaLimitOn` 에 적었습니다.

**Mitsubishi**: 저장형 스트로크 리미트 II (파라미터 `#8205` `OT-CHECK-P`) 입니다. 매뉴얼의 소프트 리미트 I 항목이 실사용 범위를 좁히는 용도로 안내하는 파라미터입니다 (`#8204`/`#8205`). Fanuc 과 같은 판정 조건이 있되 **축별**이고 **극성이 Fanuc 과 반대**입니다: `#8210 OT INSIDE` 가 `0`(바깥 금지 = II, 머무름)이면 답하고, `1`(안쪽 금지 = IIB, 진입 금지 상자)이면 그 축만 상태 `-20` 입니다. 체크 사용 여부는 `axisWorkAreaLimitOn` 에 있고, `#8204`=`#8205` 이면 무효입니다. 선반 G코드 리스트 6·7 에서는 프로그램이 `G22 X_ Z_ I_ K_` 로 `#8204`/`#8205` 를 바꾸며 켜고 `G23` 으로 끌 수 있어(비모달, 선반 프로그래밍 매뉴얼) 이 값이 프로그램 실행 중에 바뀔 수 있습니다. 머시닝센터의 `G22`/`G23` 은 다른 기능(이동 전 스트로크 체크: 프로그램이 지정한 진입 금지 상자를 이동 전에 검사해 `P452` 에러)이라 이 주소와 무관합니다.

**Siemens**: 설정 데이터 `$SA_WORKAREA_LIMIT_PLUS` (SD 43420) 입니다. 이 기능은 언제나 머무름 뜻이라 모드 거절이 없습니다. 위반하면 알람 `10730`(블록 준비 단계)·`10630`(실행 중)·`10631`(JOG)이 뜹니다 (진단 매뉴얼 3장 NC 알람). 채널 축과 기계축의 대응을 읽을 수 없는 드문 구성에서는 상태 `-20` 입니다. **쓰기는 채널이 Reset 상태일 때만 됩니다.** 설정 데이터는 Siemens 가 채널 상태에 따라 외부 변경을 거절하는 데이터라, 프로그램 실행 중·정지 중·비상정지 중에는 제어기가 거절하고 알람 `4230`(현재 채널 상태에서는 외부에서 데이터를 바꿀 수 없다는 알람, 진단 매뉴얼 3장)을 띄우며 이 주소는 상태 `-22`(기계 상태)로 답합니다 (에러 문구에 이 사유를 함께 싣습니다). 진단 매뉴얼의 알람 `4230` 설명도 파트 프로그램 실행 중에는 이 데이터를 입력할 수 없다고 적으며 작업 영역 제한의 설정 데이터와 드라이런 이송을 예로 듭니다. 840D sl 벤치에서는 비상정지로 중단된 채널에서도 거절됐습니다. 거절된 쓰기가 띄운 `4230` 은 다음 NC 시작이나 조작반에서 지울 때까지 알람 목록에 남아, 그동안 `alarmStatus` 가 `2` 입니다 (진단 매뉴얼, 840D sl 벤치에서 확인). 머신 데이터인 소프트 리미트에는 이 제약이 없습니다. 매뉴얼(List Manual 12/2019) 기준으로 이 설정 데이터는 **기본 좌표계(BCS)** 값이고(변환이 없는 기계에서는 기계좌표와 같습니다) 효력은 즉시, 보호 레벨은 사용자(7/7)입니다. 프로그램의 `G26`(양의 방향)/`G25`(음의 방향)으로도 바뀌며, 그렇게 바뀐 값이 리셋 뒤에 남는지는 머신 데이터 `10710`(`$MN_PROG_SD_RESET_SAVE_TAB`)에 달렸습니다. `WALIMOF` 동안은 값이 있어도 무시됩니다. 감시 기준점은 **공구 선단**이라 공구 길이가 자동으로 고려되고(반경은 머신 데이터 `21020` 을 켜야), AUTO 와 JOG 양쪽에서 감시합니다 (기능 매뉴얼 A3). 워크좌표계(WCS/SZS) 기준의 좌표계별 작업 영역 제한(`WALCS0`~`WALCS10`)은 별개 기능이라 이 주소와 무관합니다 (프로그래밍 매뉴얼).

**값이 있어도 켜져 있어야 강제됩니다.** 스위치는 Fanuc·Mitsubishi 가 축 단위 `axisWorkAreaLimitOn`, Siemens 가 방향별 `axisWorkAreaLimitPositiveOn`·`axisWorkAreaLimitNegativeOn` 이고, 기종별 확인법(Fanuc `G22`/`G23` 모달, Siemens `WALIMON` + 스위치, Mitsubishi `#8202`)은 그 주소들에 적었습니다.

**거리라서 `unit` 을 붙이지 않습니다.** mm/inch 는 기계 설정입니다 (`axisSoftLimitPositive` 와 같은 처리). 쓰기는 장비의 파라미터 쓰기 허가 상태를 따르며, 막혀 있으면 상태 `-22`(기계 상태)입니다. Mitsubishi 는 그 계통이 자동운전 중(일시정지 포함)이거나 데이터 보호 키 2(PLC 신호 `*KEY2`, `Y709`. 사용자 파라미터를 보호합니다)가 꺼져 있으면 상태 `-22` 이고, 설정 범위 밖의 값은 상태 `-16` 입니다. 설정 단위 `#1003` 보다 잘게 쓴 자리는 제어기가 반올림해 저장한 뒤 상태 `0` 을 돌려주므로 무엇이 들어갔는지는 다시 읽어 확인하세요 (보호 키와 반올림은 시뮬레이터에서 `#8205` 로 확인).

**두 값이 같거나 뒤집혀 있을 때의 뜻도 기종마다 다릅니다.** **Fanuc**: 양쪽이 같은 값이면 체크 2 는 **전 영역이 이동 가능**합니다 (조작 설명서 CAUTION 1, 소프트 리미트인 체크 1 과 반대). 뒤집혀 있으면 두 점을 꼭짓점으로 하는 직육면체를 그대로 경계로 삼습니다 (CAUTION 2). **Mitsubishi**: `#8204`=`#8205`(부호·값 동일)이면 무효입니다. 뒤집혀 있으면 II(머무름)는 **전 범위 금지**, IIB(진입 금지 상자)는 두 점 사이가 금지입니다 (Alarm/Parameter Manual IB-1501279). **Siemens**: 두 값의 관계에 따른 규칙이 없고 각 방향이 독립이며, 켜고 끄는 것은 방향별 스위치 주소가 맡습니다.

## /machine/channel/axis/axisWorkAreaLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

그 축의 **작업 영역 제한 음의 방향 좌표**입니다. `axisSoftLimitNegative`(장비 설치 때 고정되는 한계) 안쪽에 셋업마다 조정하는 두 번째 울타리로, Fanuc·Mitsubishi 는 기계좌표, Siemens 는 기본 좌표계(BCS) 기준이고 읽기·쓰기 모두 됩니다.

**Fanuc**: 저장형 스트로크 체크 2 (파라미터 `1323`) 입니다. 이 기능은 설정에 따라 "이 안에 머물러라" 도 되고 "여기 들어가지 마라"(척 배리어 등) 도 되는데, **이 주소는 "이 안에 머물러라" 뜻만 약속합니다.** 그래서 장비가 진입 금지 상자로 설정돼 있으면(파라미터 `1300#0`=`0`) 값을 내보내지 않고 상태 `-20` 으로 거절합니다. 그 값을 "이 사이면 안전" 으로 계산하면 정확히 거꾸로가 되기 때문입니다 (그때는 `axisWorkAreaLimitOn` 도 같은 상태 `-20` 입니다). **직경 지령 축(선반 X 등)은 직경값**입니다 (파라미터 매뉴얼 B-64490EN, 각 파라미터의 NOTE 1). 축별 사용 여부(`1310#0`)와 모달(`G22`/`G23`)은 `axisWorkAreaLimitOn` 에 적었습니다.

**Mitsubishi**: 저장형 스트로크 리미트 II (파라미터 `#8204` `OT-CHECK-N`) 입니다. 매뉴얼의 소프트 리미트 I 항목이 실사용 범위를 좁히는 용도로 안내하는 파라미터입니다 (`#8204`/`#8205`). Fanuc 과 같은 판정 조건이 있되 **축별**이고 **극성이 Fanuc 과 반대**입니다: `#8210 OT INSIDE` 가 `0`(바깥 금지 = II, 머무름)이면 답하고, `1`(안쪽 금지 = IIB, 진입 금지 상자)이면 그 축만 상태 `-20` 입니다. 체크 사용 여부는 `axisWorkAreaLimitOn` 에 있고, `#8204`=`#8205` 이면 무효입니다. 선반 G코드 리스트 6·7 에서는 프로그램이 `G22 X_ Z_ I_ K_` 로 `#8204`/`#8205` 를 바꾸며 켜고 `G23` 으로 끌 수 있어(비모달, 선반 프로그래밍 매뉴얼) 이 값이 프로그램 실행 중에 바뀔 수 있습니다. 머시닝센터의 `G22`/`G23` 은 다른 기능(이동 전 스트로크 체크: 프로그램이 지정한 진입 금지 상자를 이동 전에 검사해 `P452` 에러)이라 이 주소와 무관합니다.

**Siemens**: 설정 데이터 `$SA_WORKAREA_LIMIT_MINUS` (SD 43430) 입니다. 이 기능은 언제나 머무름 뜻이라 모드 거절이 없습니다. 위반하면 알람 `10730`(블록 준비 단계)·`10630`(실행 중)·`10631`(JOG)이 뜹니다 (진단 매뉴얼 3장 NC 알람). 채널 축과 기계축의 대응을 읽을 수 없는 드문 구성에서는 상태 `-20` 입니다. **쓰기는 채널이 Reset 상태일 때만 됩니다.** 설정 데이터는 Siemens 가 채널 상태에 따라 외부 변경을 거절하는 데이터라, 프로그램 실행 중·정지 중·비상정지 중에는 제어기가 거절하고 알람 `4230`(현재 채널 상태에서는 외부에서 데이터를 바꿀 수 없다는 알람, 진단 매뉴얼 3장)을 띄우며 이 주소는 상태 `-22`(기계 상태)로 답합니다 (에러 문구에 이 사유를 함께 싣습니다). 진단 매뉴얼의 알람 `4230` 설명도 파트 프로그램 실행 중에는 이 데이터를 입력할 수 없다고 적으며 작업 영역 제한의 설정 데이터와 드라이런 이송을 예로 듭니다. 840D sl 벤치에서는 비상정지로 중단된 채널에서도 거절됐습니다. 거절된 쓰기가 띄운 `4230` 은 다음 NC 시작이나 조작반에서 지울 때까지 알람 목록에 남아, 그동안 `alarmStatus` 가 `2` 입니다 (진단 매뉴얼, 840D sl 벤치에서 확인). 머신 데이터인 소프트 리미트에는 이 제약이 없습니다. 매뉴얼(List Manual 12/2019) 기준으로 이 설정 데이터는 **기본 좌표계(BCS)** 값이고(변환이 없는 기계에서는 기계좌표와 같습니다) 효력은 즉시, 보호 레벨은 사용자(7/7)입니다. 프로그램의 `G26`(양의 방향)/`G25`(음의 방향)으로도 바뀌며, 그렇게 바뀐 값이 리셋 뒤에 남는지는 머신 데이터 `10710`(`$MN_PROG_SD_RESET_SAVE_TAB`)에 달렸습니다. `WALIMOF` 동안은 값이 있어도 무시됩니다. 감시 기준점은 **공구 선단**이라 공구 길이가 자동으로 고려되고(반경은 머신 데이터 `21020` 을 켜야), AUTO 와 JOG 양쪽에서 감시합니다 (기능 매뉴얼 A3). 워크좌표계(WCS/SZS) 기준의 좌표계별 작업 영역 제한(`WALCS0`~`WALCS10`)은 별개 기능이라 이 주소와 무관합니다 (프로그래밍 매뉴얼).

**값이 있어도 켜져 있어야 강제됩니다.** 스위치는 Fanuc·Mitsubishi 가 축 단위 `axisWorkAreaLimitOn`, Siemens 가 방향별 `axisWorkAreaLimitPositiveOn`·`axisWorkAreaLimitNegativeOn` 이고, 기종별 확인법(Fanuc `G22`/`G23` 모달, Siemens `WALIMON` + 스위치, Mitsubishi `#8202`)은 그 주소들에 적었습니다.

**거리라서 `unit` 을 붙이지 않습니다.** mm/inch 는 기계 설정입니다 (`axisSoftLimitNegative` 와 같은 처리). 쓰기는 장비의 파라미터 쓰기 허가 상태를 따르며, 막혀 있으면 상태 `-22`(기계 상태)입니다. Mitsubishi 는 그 계통이 자동운전 중(일시정지 포함)이거나 데이터 보호 키 2(PLC 신호 `*KEY2`, `Y709`. 사용자 파라미터를 보호합니다)가 꺼져 있으면 상태 `-22` 이고, 설정 범위 밖의 값은 상태 `-16` 입니다. 설정 단위 `#1003` 보다 잘게 쓴 자리는 제어기가 반올림해 저장한 뒤 상태 `0` 을 돌려주므로 무엇이 들어갔는지는 다시 읽어 확인하세요 (보호 키와 반올림은 시뮬레이터에서 `#8205` 로 확인).

**두 값이 같거나 뒤집혀 있을 때의 뜻도 기종마다 다릅니다.** **Fanuc**: 양쪽이 같은 값이면 체크 2 는 **전 영역이 이동 가능**합니다 (조작 설명서 CAUTION 1, 소프트 리미트인 체크 1 과 반대). 뒤집혀 있으면 두 점을 꼭짓점으로 하는 직육면체를 그대로 경계로 삼습니다 (CAUTION 2). **Mitsubishi**: `#8204`=`#8205`(부호·값 동일)이면 무효입니다. 뒤집혀 있으면 II(머무름)는 **전 범위 금지**, IIB(진입 금지 상자)는 두 점 사이가 금지입니다 (Alarm/Parameter Manual IB-1501279). **Siemens**: 두 값의 관계에 따른 규칙이 없고 각 방향이 독립이며, 켜고 끄는 것은 방향별 스위치 주소가 맡습니다.

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

- **Fanuc**: 파라미터 `1310#0`(`OT2x`: `0`=사용 안 함, `1`=사용)입니다. 비트 파라미터라 쓰기는 그 바이트를 읽어 bit 0 만 바꾸고 나머지 비트(`#1` 은 체크 3 스위치)는 그대로 되씁니다. 값 주소와 같은 판정 조건을 지납니다: 장비가 진입 금지 상자로 설정돼 있으면(파라미터 `1300#0`=`0`) 그 스위치는 작업 영역 스위치가 아니므로 읽기·쓰기 모두 상태 `-20` 입니다. 체크 2 전체의 켜짐은 모달 `G22`(켬)/`G23`(끔)이며 `gModalList` 로 보이고, 전원 투입 시 어느 쪽으로 시작하는지는 파라미터 `3402#7` 이 정합니다. 체크 2 옵션이 없는 장비는 `G22` 여도 강제하지 않습니다. 두 값이 같으면 스위치가 켜져 있어도 체크 2 는 전 영역을 이동 가능으로 다루어 사실상 제한이 없습니다 (조작 설명서 CAUTION 1). 쓰기는 장비의 파라미터 쓰기 허가 상태를 따르며, 막혀 있으면 상태 `-22`(기계 상태)입니다.
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

- **Siemens**: 이 스위치(SD 43400) **그리고** 채널 모달 `WALIMON` (`/machine/channel/gModalList` 에서 `WALIMON`/`WALIMOF`). 프로그램이 `WALIMOF` 를 내리면 스위치가 `true` 여도 그동안은 걸리지 않습니다. 이 스위치는 설정 데이터라 **쓰기는 채널이 Reset 상태일 때만** 되고, 아니면 알람 `4230` 과 함께 상태 `-22`(기계 상태)입니다. 거절된 쓰기가 띄운 `4230` 은 다음 NC 시작이나 조작반에서 지울 때까지 알람 목록에 남아, 그동안 `alarmStatus` 가 `2` 입니다 (진단 매뉴얼, 840D sl 벤치에서 확인). 매뉴얼상 `BOOLEAN`, 효력 즉시, 사용자 보호 레벨이며 조작반 '파라미터' 영역에서 작업 영역 제한을 켜고 끄는 바로 그 값입니다.
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

- **Siemens**: 이 스위치(SD 43410) **그리고** 채널 모달 `WALIMON` (`/machine/channel/gModalList` 에서 `WALIMON`/`WALIMOF`). 프로그램이 `WALIMOF` 를 내리면 스위치가 `true` 여도 그동안은 걸리지 않습니다. 이 스위치는 설정 데이터라 **쓰기는 채널이 Reset 상태일 때만** 되고, 아니면 알람 `4230` 과 함께 상태 `-22`(기계 상태)입니다. 거절된 쓰기가 띄운 `4230` 은 다음 NC 시작이나 조작반에서 지울 때까지 알람 목록에 남아, 그동안 `alarmStatus` 가 `2` 입니다 (진단 매뉴얼, 840D sl 벤치에서 확인). 매뉴얼상 `BOOLEAN`, 효력 즉시, 사용자 보호 레벨이며 조작반 '파라미터' 영역에서 작업 영역 제한을 켜고 끄는 바로 그 값입니다.
- **Fanuc**: 스위치가 방향이 아니라 **축 단위**라 이 주소는 상태 `-20` 입니다. `axisWorkAreaLimitOn` 을 읽고 쓰세요 (파라미터 `1310#0`, 모달 `G22`/`G23` 설명도 거기에 있습니다).
- **Mitsubishi**: 역시 축 단위라 상태 `-20` 입니다. `axisWorkAreaLimitOn` 을 쓰세요 (파라미터 `#8202`, 게이트 `#8210` 설명도 거기에 있습니다).

## /machine/channel/spindleCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

채널의 스핀들 수입니다. 연결 시 캐싱. `spindle` 필터의 유효 범위가 `1`~이 값입니다.

**Fanuc·Siemens 는 경로마다 다릅니다.** 스핀들이 없는 경로는 `0` 이고, 그 채널의 스핀들 주소들은 Fanuc 에서 상태 `-20`(채널 공통값인 `spindleOverride`·`spindleSpeedCommanded` 도 마찬가지), Siemens 에서 상태 `-18` 로 답합니다 (Siemens 는 이 둘도 스핀들별로 읽습니다).

**Mitsubishi 는 NC 전체의 스핀들 수입니다** (파라미터 `#1039 spinno`, 기본 공통 파라미터). 모든 채널이 같은 값을 냅니다. 디메시는 이 값을 채널마다 파라미터 `#1039` 에서 읽으며, 스핀들 번호도 NC 전역으로 보이므로 어느 채널에서든 `spindle=1`~이 값으로 모든 스핀들을 가리킵니다. 스핀들 둘·계통 둘인 시뮬레이터에서 두 계통 모두 `spindle=1`·`2` 를 받고(`3` 은 상태 `-18`), 스핀들별 지령 회전수가 두 계통에서 같게 나오는 것을 확인했습니다. 실장비에서는 확인하지 못했습니다.

**Heidenhain** 은 제어기의 스핀들 전부의 개수입니다: 연결 때 HEIDENHAIN DNC 의 `GetAxesInfo` 가 준 축 목록에서 종류가 스핀들인 것을 셉니다. 지금 채널에 배정된 스핀들만이 아니어서, 밀링·선삭을 오가는 장비에서는 밀링 스핀들과 선삭 스핀들을 모두 셉니다 (테스트 환경에서 `2`. 채널의 축 목록은 밀링 모드에서 `S1`, 선삭 모드에서 `S2` 만 줍니다). `spindle` 번호는 그 목록의 순서이고 운전 모드와 무관합니다.

## /machine/channel/spindle/spindleOverride
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

스핀들 오버라이드 (%)입니다. 반환 `float` + `unit:"%"`. 값은 **0.1% 자리까지** 냅니다. 대개 정수 퍼센트지만 0.1% 단계로 거는 장비에서는 `87.5` 처럼 소수가 나옵니다. **Fanuc 은 기본적으로 채널 공통값**(`G30` 신호 `SOV0`~`SOV7`, 2진 0~254%, 전부 켜진 상태는 제어기와 같이 `0`)이라 모든 스핀들에 같은 값이 나오고, 파라미터 `3713#3`(MSC)과 `#4`(EOV)가 모두 켜진 장비는 **스핀들별 값**(2번 `G376`, 3번 `G377`, 4번 `G378`; 5번 이상은 상태 `-20`)입니다 (Connection Manual B-64483EN-1). 다경로 장비는 그 경로의 신호(경로마다 `1000` 씩 밀린 주소)를 읽습니다. Siemens·Mitsubishi 는 스핀들별 값. **Mitsubishi 는 기계 제작사 래더가 고른 방식을 따라 읽습니다** (PLC 인터페이스 매뉴얼 IB-1501272): 스핀들별 방식 선택 신호 `SPS`(`Y188F`, 스핀들마다 `+0x60`)가 꺼져 있으면 코드 신호 `SP1`/`SP2`/`SP4`(50~120% 10% 단계)를, 켜져 있으면 레지스터 `R7008`(0~200% 1% 단위, 스핀들마다 `+50`)을 읽습니다.

Heidenhain 은 `GetOverrideInfo` 가 주는 스핀들 오버라이드(정수 퍼센트) 하나라, 어느 `spindle` 로 물어도 같은 값입니다. 테스트 환경에서 조작반 표시와 같은 값이 나왔습니다.

**쓰기는 Heidenhain 만 지원합니다** (`SetOverrideSpeed`. 다른 기종은 상태 `-20`(미지원)). 정수 퍼센트를 `{"value": 80}` 처럼 씁니다. 소수부가 `0` 이면(`80.0`) 받고, `50.5` 처럼 소수부가 있거나 음수이면 상태 `-16`(쓰기 값 오류)입니다. 받는 범위는 장비가 정하며, **테스트 환경에서는 범위 밖의 값을 제어기가 가까운 끝으로 잘라 넣고 상태 `0` 을 돌려줬습니다** (테스트 환경에서 스핀들은 `50`~`130` 이라 `0` 을 쓰면 `50`, `200` 을 쓰면 `130` 이 걸렸습니다). TNC7 사용 설명서('Cutting data')는 스핀들 오버라이드 다이얼로 0%~150% 사이를 바꿀 수 있다고 적고(무단 변속 스핀들 구동의 기계에서만 걸립니다), 최대 회전수는 기계에 따라 다르다고 합니다. 무엇이 걸렸는지는 다시 읽어 확인하세요. 테스트 환경에서는 쓴 값이 읽기에 반영되기까지 0.1초쯤 걸려, 쓴 직후에 읽으면 이전 값이 나올 수 있었습니다. 스핀들 오버라이드는 하나라 어느 `spindle` 로 써도 그 하나가 바뀝니다. 쓴 값은 자동운전 중에도 걸리고 운전이 끝난 뒤에도 남습니다. 조작반에서 오버라이드를 조작하면 그 값이 걸리고, 다시 쓰면 쓴 값이 걸립니다 (마지막에 바꾼 쪽. 테스트 환경의 가상 다이얼로 확인했고, 실제 장비의 다이얼과는 확인하지 못했습니다). **쓰기 주의**: 운전 중인 장비의 스핀들 회전수가 곧바로 바뀝니다.

## /machine/channel/spindle/spindleSpeedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

스핀들 **S 지령값**입니다. 반환 `float`. **`unit` 을 붙이지 않습니다**. 지령의 뜻이 스핀들 속도 모드에 따라 갈리기 때문입니다 (회전수 일정이면 회전수, 주속 일정이면 주속). Fanuc·Siemens·Mitsubishi 가 그렇고, Heidenhain 은 주속 일정이면 값 대신 상태 `-22` 입니다 (아래). 지금 어느 모드인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=8` 로 확인하세요. 응답의 `desc` 가 기종 무관 의미를 말합니다: `constant surface speed` 면 주속 일정(`G96`, Siemens 는 `G961`/`G962` 도), `constant spindle speed (rpm)` 이면 회전수 일정(`G97`, Siemens 는 `G971`/`G972`/`G973` 과 같은 그룹의 `G94`/`G95` 도)입니다. **Fanuc 은 채널 모달 S 값** (`spindle` 필터 무시: S 지령이 채널 단위 개념), Siemens 는 스핀들별 `cmdSpeed`, Mitsubishi 는 스핀들별 S 지령 모달값입니다. **Siemens 는 회전 방향에 따라 부호가 붙습니다** (`M4` 에서 음수, 840D sl 벤치). Fanuc 은 `M3`·`M4` 모두 양수입니다 (31i 벤치). Mitsubishi 는 확인하지 못했습니다.

이 주소의 예전 이름은 `/machine/channel/spindle/speedCommanded` 입니다. 옛 주소 그대로도 동작하지만, 이 문서는 새 이름만 안내합니다.

**Heidenhain** 은 기본 PLC 프로그램이 받아 두는 프로그램의 S 값을 PLC 데이터로 읽습니다. 스핀들이 서 있어도 마지막에 지령한 S 이고, 조작반 상태 줄의 S 와 같았습니다 (저희 테스트 환경 TNC7 프로그래밍 스테이션, 회전수 지령). **그 스핀들의 마지막 지령이 주속 일정이면 상태 `-22` 입니다**: 회전수가 지름을 따라 바뀌어 지령한 회전수가 없기 때문이고, 에러 문구에 절삭 속도를 싣습니다. 그동안의 회전수는 `/machine/channel/spindle/spindleSpeedActual` 로 읽으세요. 테스트 환경에서 주속 일정은 선삭 모드의 선삭 스핀들에서만 걸렸고, 밀링 모드로 돌아온 뒤에도 그 스핀들은 마지막 지령이 남아 상태 `-22` 였습니다. 밀링의 공구 호출에 절삭 속도를 주면(`TOOL CALL 3 Z S(VC=100)`) 공구를 부를 때의 회전수로 바뀌어 주속 일정이 아니고, 이 주소는 그 회전수를 냅니다 (테스트 환경에서 지름 6 mm 공구로 조작반 S `5305`, 이 주소 `5305.165`). PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요. `spindle` 번호는 제어기의 스핀들 목록 순서이고 운전 모드와 무관합니다 (`/machine/channel/spindleCount`).

## /machine/channel/spindle/spindleSpeedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

스핀들별 실제 회전수입니다. 반환 `float` + `unit:"rpm"` (네 기종 동일). Fanuc 은 `cnc_acts2`, Siemens 는 스핀들별 `actSpeed`, Mitsubishi 는 스핀들 모니터의 회전수 항목입니다. 세 기종 모두 `spindle` 필터로 대상 스핀들을 지정하며, **오버라이드가 반영된 실측값**입니다. **Siemens 는 회전 방향에 따라 부호가 붙습니다** (`$AA_S` 의 부호가 회전 방향, 시스템 변수 목록 매뉴얼. 840D sl 벤치에서 `M4` 가 음수로 나오는 것을 확인). Fanuc 은 회전 방향과 상관없이 크기만 냅니다 (31i 벤치와 실장비에서 `M3`·`M4` 모두 양수로 확인). Mitsubishi 의 부호 규칙은 확인하지 못했습니다.

**Mitsubishi 의 이 값은 실장비에서 확인하지 못했습니다.** 저희 테스트 환경(시뮬레이터)에서는 읽히는 것까지만 확인했고, 값은 스핀들이 도는 동안에도 늘 `0` 이었습니다.

이 주소의 예전 이름은 `/machine/channel/spindle/speedActual` 입니다. 옛 주소 그대로도 동작하지만, 이 문서는 새 이름만 안내합니다.

**Heidenhain** 은 기본 PLC 프로그램이 드라이브에서 읽어 두는 실제 회전수를 PLC 데이터로 읽습니다. 회전 방향이 없는 크기이고, 조작반 상태 줄의 S 와 드라이브 진단 표의 회전수와 같았습니다 (저희 테스트 환경 TNC7 프로그래밍 스테이션). PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요. `spindle` 번호는 제어기의 스핀들 목록 순서이고 운전 모드와 무관합니다 (`/machine/channel/spindleCount`).

## /machine/channel/spindle/spindleLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

스핀들 부하율입니다. 반환 `float` + `unit`. Siemens·Mitsubishi·Heidenhain 은 항상 `unit:"%"` 입니다. Fanuc 은 벤더 응답이 단위를 함께 실어 주어 장비 설정에 따라 `%` 또는 `rpm` 이 오고, 벤더가 그 밖의 단위 코드를 주는 드문 경우에는 `unit` 키가 생략됩니다. 호출하는 쪽은 `%` 를 가정하지 말고 `unit` 을 보세요. Mitsubishi 는 스핀들 모니터의 부하 항목입니다.

**`unit` 이 `%` 일 때는 `spindleCurrent` 와 같은 물리량입니다**. 제어기가 모터 전류를 정격으로 나눠 부하율로 내기 때문입니다. 그래서 기계가 달라도 비교되는 반면(80% 는 어디서나 80%), 절대 전류값이 필요하면 `spindleCurrent` 를 쓰세요. 환산에는 그 모터의 정격 전류가 필요한데 디메시는 그 값을 내지 않으므로 **한쪽만 지원하는 기종에서는 다른 쪽을 계산해낼 수 없습니다.** Fanuc 이 `unit:"rpm"` 을 줄 때는 부하가 아니라 회전수라 이 관계가 성립하지 않습니다.

**Mitsubishi 의 이 값은 실장비에서 확인하지 못했습니다.** 저희 테스트 환경(시뮬레이터)에서는 읽히는 것까지만 확인했고, 값은 스핀들이 도는 동안에도 늘 `0` 이었습니다.

**Heidenhain** 은 기본 PLC 프로그램이 드라이브에서 읽어 두는 모터 부하율을 PLC 데이터로 읽습니다. 조작반의 드라이브 진단 표(Utilization [%])에 보이는 값입니다. 저희 테스트 환경(TNC7 프로그래밍 스테이션)에서는 PLC 프로그램이 드라이브 대신 정해진 값을 넣어 두므로, 조작반의 드라이브 진단 표와 같은 값인 것까지 확인했고 실제 드라이브의 값은 확인하지 못했습니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요. `spindle` 번호는 제어기의 스핀들 목록 순서이고 운전 모드와 무관합니다 (`/machine/channel/spindleCount`).

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

**Siemens**: 값이 드라이브에서 오므로, 그 스핀들에 드라이브가 배정되지 않은 채널에서는 상태 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

## /machine/channel/spindle/spindleTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

스핀들 모터 온도입니다. 반환 `float` + `unit:"°C"` (네 기종 동일). Fanuc 은 진단 403번, Siemens 는 드라이브 파라미터 `R0035`, Mitsubishi 는 스핀들 드라이브 모니터의 모터 온도입니다.

**Siemens**: 값이 드라이브에서 오므로, 그 스핀들에 드라이브가 배정되지 않은 채널에서는 상태 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

**Mitsubishi 의 이 값은 실장비에서 확인하지 못했습니다.** 저희 테스트 환경(시뮬레이터)에서는 읽히는 것까지만 확인했고, 값은 늘 `0` 이었습니다.

**Heidenhain** 은 기본 PLC 프로그램이 드라이브에서 읽어 두는 모터 온도를 PLC 데이터로 읽습니다. 조작반의 드라이브 진단 표(Temperature [°C])에 보이는 값입니다. 저희 테스트 환경(TNC7 프로그래밍 스테이션)에서는 PLC 프로그램이 드라이브 대신 정해진 값을 넣어 두므로, 조작반의 드라이브 진단 표와 같은 값인 것까지 확인했고 실제 드라이브의 값은 확인하지 못했습니다. PLC 데이터라 연결 설정에 `access_password` 가 있어야 하며, 없거나 제어기가 거절하면 상태 `-20` 입니다. 디메시가 아는 심볼 이름을 그 장비에서 찾지 못해도 상태 `-20` 입니다. 심볼 이름은 장비의 PLC 프로그램마다 다를 수 있으니, 그 장비에서 이 값을 담은 심볼 이름을 알면 `/machine/plcAddress/plcType/plcValue` 로 읽으세요. `spindle` 번호는 제어기의 스핀들 목록 순서이고 운전 모드와 무관합니다 (`/machine/channel/spindleCount`).

## /machine/channel/spindle/spindlePower
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

그 스핀들이 **지금 쓰고 있는 전력**입니다. `channel` + `spindle` 필터. 반환 `float` + `unit:"W"`. 규칙은 `axisPower` 와 같습니다 (회생 중 음수, 누적은 `spindleEnergy*`).

**Fanuc**: 진단 `4902`. **Siemens**: 스핀들 전용 노드가 없어 그 스핀들의 **기계축** 값을 읽습니다 (`vaPower`). 제어기가 그 스핀들을 기계축에 대응시키지 못하는 드문 구성에서는 상태 `-20` 입니다. 머신 데이터 `36730` `$MA_DRIVE_SIGNAL_TRACKING` 이 그 기계축에서 `1` 일 때만 값이 나오고, `0` 이면 늘 `0` 입니다 (840D sl 벤치: `1` 로 바꾼 뒤 1350rpm 에서 90~110W, 감속 중 음수. 자세히는 `axisPower` 에 적었습니다).

**전체 소비 전력을 내는 주소는 없습니다.** Fanuc 에서는 제어기가 직접 재는 전체 값을 읽을 수 있지만, Siemens 에서는 그런 값을 확인하지 못해 디메시가 축·스핀들을 더해야 하는데, 그러면 같은 주소가 한쪽은 제어기가 잰 값, 다른 쪽은 디메시가 더한 값이 됩니다. 게다가 제어기가 각 값을 서로 다른 순간에 재므로 **Fanuc 에서도 전체와 부분의 합이 같지 않을 수 있습니다**. 합계가 필요하면 축·스핀들 값을 받아 직접 더하시고, 그것이 제어기가 재는 전체와 다를 수 있다는 것을 감안하세요.

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

## /machine/channel/activeWorkOffset
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

**지금 골라져 있는 워크좌표계**를 그 기종의 `workOffset` 필터 표기로 반환합니다. 반환 `string`, 읽기 전용입니다. 받은 값을 그대로 `/machine/channel/workOffset/axis/workOffsetValue` 의 `workOffset` 필터에 넣으면 그 워크좌표계의 오프셋을 읽을 수 있습니다.

기종마다 나오는 값:

- **Fanuc**: `G54`~`G59`, 추가 워크좌표계는 `G54.1P2` 처럼 P 번호까지 붙습니다. P 번호는 커스텀 매크로 시스템 변수 `#4330`(실행 중 블록의 추가 워크좌표계 번호, 운영자 매뉴얼 B-64484EN §16)에서 읽습니다. `EXT` 는 어느 좌표계에나 더해지는 공통 오프셋이라 나오지 않습니다. 커스텀 매크로 옵션이 없어 `#4330` 을 읽을 수 없는 제어기에서는 P 번호를 알 수 없습니다. 그때 제어기가 추가 워크좌표계를 `G54` 로 알려 주면(저희 시뮬레이터와 31i-B 실장비가 그랬습니다) 추가 워크좌표계가 걸려 있어도 `G54` 가 나오고, `G54.1` 로 알려 주면 상태 `-22`(기계 상태)입니다. 저희 시험 장비에는 모두 그 옵션이 있어 이 경우는 확인하지 못했습니다
- **Mitsubishi**: `G54`~`G59`. **`G54.1` 이 걸려 있으면 상태 `-22`(기계 상태)입니다.** 어느 확장 워크좌표계인지(P 번호)를 저희가 가진 EZSocket 레퍼런스로 읽는 방법을 확인하지 못했습니다. `G54.1` 이 걸려 있다는 것까지는 `/machine/channel/gModalCategory/gModal?gModalCategory=7` 이 답합니다 (시뮬레이터의 머시닝센터·선반 구성에서 `G54.1 P2` 를 걸어 이 거절과 그 `G54.1` 을 확인했습니다). 시스템 타입이 머시닝센터도 선반도 아닌 구성(`machineType`=`unknown`)에서는 상태 `-20` 입니다
- **Siemens**: 설정 프레임 `G500`, `G54`~`G57`, `G505`~`G599`. `G500` 은 설정 오프셋이 꺼진 상태이고, 이 표기도 `workOffset` 필터가 받습니다
- **Heidenhain**: 활성 프리셋의 번호(`0`, `1` …)이고, 프리셋 테이블에서 활성으로 표시된 행입니다 (테스트 환경에서 프리셋을 바꾸자 따라 바뀌는 것을 확인했습니다. TNC7 사용 설명서 'Preset table' 은 제어기가 활성 행의 `ACTNO` 칸에 `1` 을 넣는다고 적습니다). G코드가 아니라 번호인 것은 Heidenhain 의 `workOffset` 필터가 프리셋 번호를 받기 때문입니다

⚠️ **이 주소는 어느 워크좌표계가 골라져 있는지만 답합니다.** 그 오프셋이 지금 좌표에 그대로 걸려 있다는 뜻은 아닙니다. Fanuc·Mitsubishi 는 `EXT` 가 더해지고, Siemens 는 표를 고친 뒤 다시 활성화하기 전까지 옛 값이 걸려 있으며(`totalWorkOffsetValue`), Heidenhain 은 표를 고친 뒤 프리셋을 다시 활성화하기 전까지 옛 값이 걸려 있고 프리셋의 빈 칸인 축은 직전 오프셋을 유지하며, 영점 표·`TRANS DATUM` 의 영점 이동이나 (장비에 따라) 팔레트 프리셋이 그 위에 더해질 수 있습니다. 공작물 좌표가 필요하면 `/machine/channel/axis/workPosition` 을 읽으세요.

**`gModalCategory=7` 과의 차이**: G코드 기종에서는 둘이 같은 모달을 봅니다. 추가 워크좌표계가 걸려 있으면 그쪽은 `G54.1` 을 내고, 이 주소는 Fanuc 에서 거기에 P 번호를 붙여(`G54.1P2`) 필터로 되돌려 쓸 수 있게 합니다 (Mitsubishi 는 위와 같이 상태 `-22`, Siemens 에는 `G54.1` 이 없습니다). 이 주소는 Heidenhain 에서도 답합니다.

Fanuc 의 값은 **실행 중인 블록** 기준입니다. 선독한 블록이 워크좌표계를 바꿔도 그 블록이 실행되기 전까지는 이전 값입니다.

범위 확장(`channel=1-2`)을 지원합니다. 채널마다 따로 고르므로 채널마다 다른 값이 나올 수 있습니다 (840D sl 벤치에서 채널 1 `G54`, 채널 2 `G500`).

## /machine/channel/workOffset/axis/workOffsetValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**워크좌표계 오프셋**: G54 등 워크좌표계의 축별 오프셋 거리입니다 (read + write). 반환 `float`, 실거리 (장비 설정 단위 mm/inch 그대로. Fanuc 내부 정수 표현은 SDK 가 소수점 배율 정규화).

`workOffset` 필터는 **현장 표기를 직접 입력**합니다. G코드 기종은 G코드 표기, Heidenhain 은 프리셋 번호입니다 (`plcAddress` 처럼 열린 이름공간: 별도 번호 체계 없음). 대소문자 무시, 공백·별칭 불허:

- **Fanuc · Mitsubishi**: `EXT`(공통 오프셋: 전 좌표계 가산, 조작반 EXT 행), `G54`~`G59`, 확장 `G54.1P1`~`G54.1P300`. 옵션에 없는 P번호는 Fanuc 이 벤더 에러로 거절합니다. Mitsubishi 는 옵션 밖의 번호에도 상태 `0` 과 값 `0` 을 낼 수 있습니다 (시뮬레이터에서 확인: 확장 워크좌표계 48세트 옵션인 선반 구성에서 프로그램의 `G54.1 P49` 는 거절됐는데 이 주소는 `G54.1P49`~`G54.1P96` 에 값 `0` 을 냈고, `G54.1P97` 부터 상태 `-17`(벤더 에러)이었습니다). 그 장비가 몇 세트까지 쓰는지는 조작반 진단 화면의 옵션 목록에서 확인하세요 (시뮬레이터에서는 그 목록에 세트 수가 보였습니다). 두 기종이 **같은 표기를 받고 같은 문구로 거절**합니다
- **Siemens**: `G500`, `G54`~`G57`, `G505`~`G599`. 실제로 몇 개까지 있는지는 장비 설정에 달려 있어, 없는 지정자는 상태 `-18` 과 함께 **그 장비에서 허용되는 목록**을 알려줍니다
- **Heidenhain**: 프리셋 테이블의 번호(`0`, `1` …, 조작반 프리셋 테이블의 행 번호). G코드 표기는 상태 `-18`(필터 값 오류)로 거절하고, 표에 없는 번호는 상태 `-18` 과 함께 **표에 있는 번호 범위**를 알려줍니다

⚠️ **`G500` 은 Fanuc `EXT` 와 다릅니다.** `EXT` 는 어느 `G5x` 가 활성이든 **그 위에 더해지는** 공통 오프셋이지만, `G500` 은 `G54`~`G57` 과 **같은 모달 그룹의 배타적 멤버**라 그것들과 동시에 활성일 수 없습니다. `G500` 이 걸린 상태는 설정 오프셋이 꺼진 상태이고 그 자리의 값은 보통 `0` 입니다. 지금 어느 것이 골라져 있는지는 `/machine/channel/activeWorkOffset` 으로 확인하세요 (그 값을 그대로 `workOffset` 에 넣을 수 있습니다). Siemens 에서 `EXT` 처럼 전 좌표계에 가산되는 몫은 이 주소가 아니라 **별도의 프레임**(조작반의 `Basic reference`·`Total basic WO` 행)에 있고, 디메시는 그것들을 개별 주소로 내지 않습니다. 합쳐진 결과는 `/machine/channel/axis/totalWorkOffsetValue` 가 답합니다.

**Siemens 는 기본 오프셋과 미세 조정(`Fine`)의 합을 반환합니다.** 장비가 그 합을 적용하고 조작반도 한 오프셋의 두 칸으로 보여주므로, 이 주소는 **실제 적용되는 값**을 냅니다. 조작반의 `Coarse` 칸만 보고 비교하면 다르게 보일 수 있습니다. 미세 조정만 따로 보려면 `/machine/channel/workOffset/axis/workOffsetFineValue` 를 쓰세요. Fanuc·Mitsubishi 에는 미세 조정 개념이 없어 값이 하나이고, 그래서 Fanuc·Mitsubishi·Siemens 에서 이 주소의 뜻이 같습니다 (Heidenhain 은 다음 문단).

**Heidenhain 은 프리셋 테이블의 칸을 반환합니다.** X·Y·Z 축은 기본 변환 칸(`X`·`Y`·`Z`)이고, 그 밖의 축은 그 축의 오프셋 칸(`A_OFFS` 처럼 축 이름 뒤에 `_OFFS`)입니다. X·Y·Z 축의 오프셋 칸(`X_OFFS`·`Y_OFFS`·`Z_OFFS`)은 이 값에 들어 있지 않습니다. 기본 변환은 기본 좌표계의 값이고 오프셋 칸은 기계 좌표계에서 그 축을 옮기는 값이라, 둘이 함께 어떻게 작용하는지가 기계 구조에 따라 달라 한 값으로 더하지 않습니다 (TNC7 사용 설명서 'Preset table'). 디메시가 프리셋 테이블에서 그 축의 칸을 찾지 못하면 상태 `-20`(미지원)입니다. 프리셋의 회전(`SPA`·`SPB`·`SPC`, 공작물 좌표계의 기본 회전을 정하는 공간각)은 이 값에 들어 있지 않고 `workOffsetRotation` 이 냅니다. 이 값은 프리셋 테이블의 값이고, 가공 중에는 그 위에 영점 표·`TRANS DATUM` 의 영점 이동이나 (장비에 따라) 팔레트 프리셋이 더해질 수 있습니다 (설명서 'Preset table'·'Datum table').

**Heidenhain 은 빈 칸이 `null` 입니다.** 프리셋 테이블은 칸을 비워 둘 수 있고, 빈 칸은 `0` 이 아니라 **이 프리셋에 그 축이 적혀 있지 않다**는 뜻입니다. 그 프리셋을 활성화해도 그 축은 이미 걸려 있던 오프셋을 그대로 유지했습니다 (테스트 환경에서 확인: X·Y·Z 가 모두 빈 프리셋을 활성화해도 공작물 좌표가 바뀌지 않았습니다. TNC7 사용 설명서 'Preset table' 도 빈 칸은 활성화할 때 이전 값을 두고 `0` 인 칸은 덮어쓴다고 적습니다). 그래서 이 주소로는 그 축에 지금 걸린 값을 알 수 없으니, 좌표는 `workPosition` 으로 확인하세요. 단일 읽기는 이 사정을 `desc` 에 싣습니다. Fanuc·Mitsubishi·Siemens 는 칸마다 숫자가 있어 `null` 을 내지 않습니다.

⚠️ **이 값은 저장된 평행이동입니다.** 두 가지가 더 있습니다. ① 워크좌표계에는 **회전·배율·미러**가 걸릴 수 있어(`workOffsetRotation`·`workOffsetScale`·`workOffsetMirrorOn`) 걸려 있으면 좌표 변환이 이 값만으로 결정되지 않습니다. ② 실제로 걸리는 총량에는 기준 오프셋 등이 더해져 이 값과 다를 수 있습니다 (`totalWorkOffsetValue`. Siemens 전용이며, 다른 기종에서 유효 오프셋을 얻는 법은 그 절에 있습니다). 부품 좌표가 필요하면 계산하지 말고 `/machine/channel/axis/workPosition` 을 읽으세요. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 라 이 값이 곧 변환입니다.

**자동운전 중에 표를 고칠 수 있는지, 고친 값이 좌표에 언제 반영되는지가 기종에 따라 다릅니다.** 이 주소는 저장값이라 조작반에서 고치면 **즉시** 새 값을 냅니다.

| 기종 | 자동운전 중 편집 | 좌표 반영 |
|---|---|---|
| Fanuc | 가능 | 즉시 `workPosition` 에 반영 (시뮬레이터와 31i 벤치에서 확인) |
| Siemens | 가능 (조작반) | 다음 활성화(G500·G54~G599 프로그래밍, 또는 리셋 후 재시작)까지 반영 안 됨. 표는 그 시점에 채널의 활성 프레임으로 복사됩니다 (Basic Functions K2. 테스트 벤치에서 확인). 지금 걸린 값은 `totalWorkOffsetValue` 가 답합니다 |
| Mitsubishi | 자동운전 중엔 제어기가 거절합니다. 이 주소의 쓰기는 상태 `-22` 이고 조작반도 "자동운전중" 을 띄웁니다 (시뮬레이터에서 확인. 공구 보정용 파라미터 `#11017` 을 `1` 로 두어도 같았습니다) | 조작반에서 자동운전 중에 고친 값은 다음 블록 또는 몇 블록 뒤부터 유효 (Instruction Manual) |
| Heidenhain | 확인하지 못했습니다 | 프리셋을 다시 활성화할 때까지 반영 안 됨 (테스트 환경에서 활성 프리셋의 칸을 고친 뒤 확인). 활성화하면 표의 값이 걸리고, 빈 칸인 축은 그대로입니다 |

Siemens 에서 두 주소가 다른 동안이 곧 "바꿨지만 아직 안 걸린" 상태이니, 좌표를 감시하는 앱은 그것을 이상으로 보지 마세요.

`axis` 는 축 번호(1~)입니다. 축 이름은 네 기종 모두 받습니다 (`/machine/channel/axis/axisName` 이 내는 이름, 대소문자 무관). `axis=1-3`·`workOffset=G54,G55` 확장 지원합니다. Fanuc 은 같은 workOffset 의 축 확장이 FOCAS 호출 1회로 묶입니다. Heidenhain 은 `workOffset=0-24` 처럼 번호 범위로 확장하고, 한 요청의 프리셋·축을 표 한 번 읽기로 답합니다.

쓰기는 `{"value": 25.4}` (단일 축). **Fanuc·Mitsubishi 지원. Siemens 는 상태 `-20` 이고, 이것은 미구현이 아니라 의도한 제외입니다.** 이 값이 든 노드는 벤더 변수 매뉴얼에서 읽기·쓰기(`$P_UIFR`)로 되어 있지만, 같은 매뉴얼이 **설정 프레임을 활성화하려면 PI 서비스 `SETUFR` 를 불러야 한다**고 못박습니다 (변수 목록 매뉴얼, Area C Block FU). 디메시는 OPC-UA 로 그 서비스를 부르지 않으므로(서버의 `/Methods` 에서 확인한 메서드는 파일 처리와 공구 관리입니다) 값만 쓰면 받아들여지는데 실제로 걸리는 오프셋도 조작반 표시도 그대로입니다 (테스트 벤치에서 확인). 성공처럼 보이면서 아무것도 하지 않는 쓰기라 그대로 통과시키지 않고 거절합니다. Siemens 에서 오프셋을 바꾸려면 조작반에서 설정하세요. **Heidenhain 도 상태 `-20` 입니다.** 프리셋 테이블에 쓴 값은 프리셋을 다시 활성화해야 좌표에 걸리는데(위 표), 디메시는 그 활성화를 하지 않으므로 Siemens 와 같은 이유로 쓰기를 받지 않습니다. 조작반에서 설정하세요.

**Fanuc·Mitsubishi 는 쓴 값을 그 축의 자릿수로 반올림해 저장하고 상태 `0` 을 돌려줍니다.** Fanuc 은 디메시가 그 축의 소수점 자릿수로 반올림해 보내고, Mitsubishi 는 설정 단위 파라미터 `#1003` 이 자릿수를 정합니다 (테스트 환경에서 1µm 설정은 `12.345678` 을 `12.346` 으로, 1nm 설정은 그대로 저장). 무엇이 들어갔는지는 다시 읽어 확인하세요. Mitsubishi 에서 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다).

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다. Heidenhain 은 이 규칙과 달리 직선축이 언제나 mm 입니다 (디메시가 DNC 에서 mm 를 골라 읽습니다).

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
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

워크좌표계의 **축별 회전각**입니다. 필터는 `/machine/channel/workOffset/axis/workOffsetValue` 와 같습니다. 반환 `float`, `unit` 은 `deg`(도). **읽기 전용**.

`0` 이면 그 축에 회전이 걸려 있지 않습니다.

**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**Heidenhain 은 프리셋 테이블의 공간각 칸을 반환합니다.** X 축은 `SPA`, Y 축은 `SPB`, Z 축은 `SPC` 이고, 각각 그 축을 도는 각도입니다. TNC7 사용 설명서 'Preset table' 은 이 셋을 공작물 좌표계의 기본 회전(`SPC` 만 쓸 때)이나 3D 기본 회전으로 해석한다고 적습니다. 회전이 겹치는 순서와 부호는 Siemens 의 기본 설정(머신 데이터 `10600` `$MN_FRAME_ANGLE_INPUT_MODE` 가 `1`, RPY 각)과 같아, 같은 세 값이면 같은 회전입니다. 고정된 축을 기준으로 X, Y, Z 순서로 돕니다 (TNC7 사용 설명서 'PLANE SPATIAL' 은 공간각을 A, B, C 순서의 회전으로 적고, Basic Functions K2 는 RPY 각을 Z, Y', X'' 순서로 적습니다. 테스트 환경에서 세 칸에 값을 넣고 프리셋을 활성화한 뒤 `workPosition` 과 대조해 확인했습니다). 머신 데이터 `10600` 이 `2`(ZX'Z'' 오일러 각)인 Siemens 장비와는 같은 세 값이라도 회전이 다릅니다. 그 밖의 축(회전축 `A`·`C` 등)은 상태 `-18` 입니다. 회전축의 오프셋 칸(`A_OFFS` 등)은 회전이 아니라 그 축의 이동이라 `workOffsetValue` 가 냅니다. 이 값은 표에 적힌 값이라, 표를 고친 뒤 프리셋을 다시 활성화하기 전에는 지금 걸린 회전과 다를 수 있습니다.

**쓰기는 지원하지 않습니다** (상태 `-20`). Siemens 는 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다** (테스트 벤치에서 확인). 장비 쪽 활성화 절차가 따로 필요한데 디메시는 그 절차를 부르지 않습니다. Heidenhain 도 프리셋 테이블에 쓴 값은 프리셋을 다시 활성화해야 걸리는데 디메시는 그 활성화를 하지 않습니다. 변경은 조작반에서 하세요. 평행이동 `workOffsetValue` 도 같은 이유로 두 기종에서 쓰기가 상태 `-20` 입니다 (Siemens 는 `workOffsetFineValue` 도).

**Siemens·Heidenhain 에서 읽을 수 있습니다.**

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 워크 오프셋 표는 평행이동만 저장합니다 (`workOffsetValue`).

## /machine/channel/workOffset/axis/workOffsetScale
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

워크좌표계의 **축별 배율**입니다. 필터는 `/machine/channel/workOffset/axis/workOffsetValue` 와 같습니다. 반환 `float`, 무차원이라 `unit` 이 없습니다. **읽기 전용**.

`1` 이면 배율이 없습니다 (실제 크기). `2` 면 그 축 방향으로 두 배로 가공합니다.

**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요한데 디메시는 그 절차를 부르지 않습니다. 변경은 조작반에서 하세요. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 상태 `-20` 입니다.

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

워크좌표계의 **축별 미러(대칭) 여부**입니다. 필터는 `/machine/channel/workOffset/axis/workOffsetValue` 와 같습니다. 반환 `boolean`. **읽기 전용**.

`true` 면 그 축 방향이 반전됩니다. 조작반의 워크오프셋 상세 화면에서 축별 체크박스로 보이는 값입니다.

**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요한데 디메시는 그 절차를 부르지 않습니다. 변경은 조작반에서 하세요. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 상태 `-20` 입니다.

**Siemens 전용**입니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 워크 오프셋 표는 평행이동만 저장합니다 (`workOffsetValue`).

## /machine/channel/gModalCategory/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalCategory"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
filter_codes: {"gModalCategory": [{"value": 1, "name": "motion"}, {"value": 2, "name": "plane"}, {"value": 3, "name": "distanceMode"}, {"value": 4, "name": "units"}, {"value": 5, "name": "feedMode"}, {"value": 6, "name": "cutterComp"}, {"value": 7, "name": "coordinateSystem"}, {"value": 8, "name": "spindleSpeedMode"}]}
```

활성 G모달을 **기종 무관 표준 그룹 번호**로 조회합니다 (`plcType` 처럼 디메시가 정한 벤더 중립 번호: 벤더 원시 그룹 번호가 아닙니다). `gModalCategory` 필터 값:

⚠️ **중립인 것은 "무엇을 묻는가"(그룹 번호)까지입니다. 돌아오는 값은 그 기종의 G코드라 기종 무관 분기에는 쓸 수 없습니다.** 같은 상태를 Fanuc 은 `G21`, Siemens 는 `G710` 이라고 답합니다. `desc` 가 의미를 알려주지만 그것은 **사람이 읽는 문구**이지 분기용 계약이 아닙니다 (문구는 바뀔 수 있습니다). 이 주소는 **자기 기종의 G코드를 아는 호스트 앱**을 위한 것입니다. 기종을 가로지르는 판단이 필요하면 그 사실을 직접 답하는 주소를 쓰세요. 예컨대 이송 실효값은 `feedActual`, 주속 회전수는 `spindleSpeedActual` 입니다.

- `1` = motion: 이송 모드 (G00 급속 / G01 직선 / G02·G03 원호 …)
- `2` = plane: 가공 평면 (G17 XY / G18 ZX / G19 YZ)
- `3` = distanceMode: 절대/증분 (G90/G91) · **Fanuc 선반은 G코드 체계에 따라 갈립니다.** System A 에는 이 모달 그룹이 아예 없어(절대/증분을 `U`/`W` 어드레스로 표현) 상태 `-20` 으로 답하고, System B·C 에는 있어 값이 나옵니다. 체계는 파라미터 `3401` 의 `GSB`(#6)·`GSC`(#7)로 정해지며 디메시가 연결할 때 읽습니다
- `4` = units: 인치/미터 (G20·G70·G700 / G21·G71·G710). Siemens 는 `G70`/`G71` 이 좌표값만, `G700`/`G710` 이 이송·오프셋까지 바꿉니다 (프로그래밍 매뉴얼)
- `5` = feedMode: 이송 지정 (분당/회전당/역시간)
- `6` = cutterComp: 공구경 보정 (G40 해제 / G41 좌 / G42 우)
- `7` = coordinateSystem: 워크좌표계. Fanuc·Mitsubishi 는 G54~G59 이고 추가 워크좌표계는 `G54.1` 입니다 (Fanuc 은 시뮬레이터와 31i-B 실장비에서, Mitsubishi 는 시뮬레이터에서 확인했습니다). 다만 Fanuc 에서 커스텀 매크로 옵션이 없어 추가 워크좌표계 번호(`#4330`)를 읽을 수 없는 제어기는 추가 워크좌표계가 `G54` 로 나올 수 있습니다. Siemens 는 `G500`·`G54`~`G57`·`G505`~`G599` 입니다. 몇 번째 추가 워크좌표계인지(P 번호)까지는 `/machine/channel/activeWorkOffset`
- `8` = spindleSpeedMode: 주속 일정(G96) / 회전수 일정(G97)

값은 그 기종의 G코드 문자열 + 핵심 조합엔 `desc` 로 기종 무관 의미 (예: Fanuc `{"value":"G21","desc":"metric"}`, Siemens `{"value":"G710","desc":"metric"}`). 벤더 원시 그룹 접근은 `gModalGroup/gModal`(하나) 또는 `gModalList`(전체)를 쓰세요. 그 그룹에 걸린 모달이 없으면(그 기종이 지원하지 않는 조합 포함) 값이 `null` 입니다. `gModalList` 의 그 자리와 같은 표현입니다. Mitsubishi 는 제어기가 그 그룹을 거절하면 `null` 이고, 통신 오류는 `null` 이 아니라 에러로 돌아옵니다. **Siemens 는 `feedMode`(`5`)와 `spindleSpeedMode`(`8`)가 같은 그룹이라 값이 항상 같습니다.** SINUMERIK 은 이송 방식(`G93`·`G94`·`G95`)과 주속 방식(`G96`·`G97`)을 한 G그룹에 넣어 활성값이 하나뿐이고, 물어본 쪽에 따라 `desc` 만 갈립니다:

```
기계가 G94 상태
  gModalCategory=5 → {"value":"G94","desc":"feed per minute"}
  gModalCategory=8 → {"value":"G94","desc":"constant spindle speed (rpm)"}
```

`G94` 는 이송 코드이지 주속 코드가 아닙니다. `desc` 가 저렇게 붙는 것은 "`G96` 계열이 아니면 주속 일정이 아니다" 는 뜻이며, **`value == "G97"` 로 회전수 일정을 판정하면 Siemens 에서는 영영 맞지 않습니다.** Fanuc·Mitsubishi 는 두 그룹이 분리돼 있어 값이 다릅니다.

**Mitsubishi 는 머시닝센터와 선반 모두 지원합니다** (여덟 그룹 모두). 그룹 번호는 두 계열의 프로그래밍 매뉴얼(머시닝센터 · 선반 G코드 리스트 표)로 같음을 확인했습니다. 선반은 G코드 리스트(파라미터 `#1037 cmdtyp`)에 따라 **같은 뜻이 다른 G코드로 나옵니다**: 절대/증분이 `G90`/`G91`(리스트 3·5·7) 또는 `G190`/`G191`(리스트 2·4·6), 이송이 `G94`/`G95` 또는 `G98`/`G99`. 그래서 값이 아니라 `desc` 로 뜻을 읽어야 하는 자리이고, 이 표기들은 모두 `desc` 가 붙습니다. 시스템 타입이 머시닝센터도 선반도 아닌 구성(`machineType`=`unknown`)에서만 상태 `-20` 입니다. 벤더 원시 접근(`gModalGroup/gModal` 낱개, `gModalList` 전체)은 기종과 무관하게 동작합니다 (원시 번호라 의미를 약속하지 않으므로).

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

- **Fanuc**: FOCAS 가 매기는 번호 `0`~`36` (`cnc_rdgcode` 의 `type`). 값은 FOCAS 가 준 그대로라, 추가 워크좌표계(`G54.1`)가 걸려 있어도 워크좌표계 그룹이 `G54` 로 올 수 있습니다 (NC Guide 에서 확인. 중립 분류 `gModalCategory=7` 은 `G54.1`)
- **Siemens**: G-펑션 그룹 `1`~`N` (N = 장비의 그룹 수. 실측 840D sl 은 `64`)
- **Mitsubishi**: 벤더 API 의 그룹 번호 `1`~`21`

⚠️ **여기 넣는 번호는 통신 규격의 번호이지 프로그래밍 매뉴얼의 `그룹` 열이 아닙니다.** 두 체계가 같은지는 기종마다 다를 수 있으므로, 매뉴얼의 그룹 번호를 그대로 넣지 말고 `gModalList` 를 한 번 읽어 자리를 확인하세요. Fanuc 은 `0`부터 시작합니다.

범위 밖 번호는 상태 `-18` 로 거절하며 허용 범위를 함께 알려줍니다. 그 그룹에 걸린 모달이 없거나 그 기종에 없는 그룹이면 값이 **`null`** 입니다. `gModalList` 의 그 자리가 `null` 인 것과 같은 표현입니다. Mitsubishi 는 제어기가 그 그룹을 거절하면 `null` 이고, 통신 오류는 `null` 이 아니라 에러로 돌아옵니다.

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

- **Fanuc**: FOCAS 그룹 `0`~`36` 순서로 **길이는 항상 37** 입니다. 그 기종에 없는 그룹은 `null` (실측 31i: 34개가 오고 `24`·`25`·`28` 이 빈 자리). **`cnc_rdgcode` 를 쓸 수 없는 제어기에서는 `21` 부터가 전부 `null`** 입니다. 그때 쓰는 통로(`cnc_modal`)가 FOCAS2 스펙상 G코드 그룹 `0`~`20` 까지만 덮습니다. 추가 워크좌표계(`G54.1`)가 걸려 있어도 워크좌표계 그룹은 `G54` 로 올 수 있습니다 (FOCAS 가 준 값 그대로. NC Guide 에서 확인). 중립 분류 `/machine/channel/gModalCategory/gModal?gModalCategory=7` 은 이때 `G54.1` 을 냅니다
- **Siemens**: `ncFkt` G-펑션 그룹 1~N 순서 (N = 장비의 그룹 수). 활성 G-펑션이 없는 그룹은 `null` 입니다 (실측: 64개 중 7개)
- **Mitsubishi**: 벤더 그룹 1~21 순서 (`GetGCodeCommand`). **길이는 항상 21** 이고, 그 기종에 없는 그룹이나 활성 모달이 없는 그룹은 `null` 입니다. 통신 오류는 `null` 로 채우지 않습니다. 도중에 통신 오류가 나면 목록 전체가 에러로 돌아옵니다. 표기는 벤더 매뉴얼 예시대로 정수부 두 자리(`G02`·`G50.2`)라 Fanuc 과 같은 모양입니다

**배열 위치가 곧 그룹 번호입니다.** 없는 자리를 건너뛰지 않고 `null` 로 채우는 것도 그 대응을 지키기 위해서입니다. 앞으로 당겨 담으면 뒤 항목이 남의 자리에 앉습니다.

⚠️ **Mitsubishi 는 왕복이 그룹 수만큼(21회) 듭니다.** 벤더 API 는 그룹 단위 조회를 제공합니다. 모달 화면을 한 벌 그리는 용도이지 주기 폴링용이 아닙니다. 특정 그룹 하나만 필요하면 `gModalGroup/gModal`(1왕복)로 좁혀 읽으세요.

**이 가족의 세 주소 모두 값은 기종 무관이 아닙니다.** 이 목록과 `gModalGroup/gModal` 은 벤더 그룹 번호·순서 그대로이고, `gModalCategory/gModal` 도 그룹을 고르는 번호만 중립이며 값은 그 기종의 G코드입니다. **기종을 가로지르는 판단에는 셋 다 쓰지 마세요** (자세히는 `gModalCategory/gModal` 설명 참조).

길이는 그 기종의 벤더 그룹 수(위 목록)로 정해지므로 `[]` 은 나오지 않습니다. 적용된 G코드가 없는 자리가 `null` 입니다.

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
- **Fanuc·Mitsubishi 는 `0` 입니다.** 두 제어기는 "지령 없음" 을 따로 표현하지 않아, `0` 이 지령되지 않음과 `0` 지령(`T0` 은 실제로 쓰이는 지령입니다)을 함께 뜻합니다. 읽기만으로는 가를 수 없는 구분을 만들지 않으므로 `null` 로 바꾸지 않습니다.

그래서 **"지령된 게 있나" 를 `== 0` 하나로 판정하면 Siemens 에서 걸리지 않고, `== null` 하나로 판정하면 Fanuc·Mitsubishi 에서 걸리지 않습니다.** 어느 값을 "없음" 으로 볼지는 그 장비를 아는 호스트 앱이 정하세요.

## /machine/channel/partCountActual
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

지금까지 가공된 수량입니다. 작업을 바꿀 때 리셋하는 카운터입니다. `channel` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0}` 으로 리셋합니다. Fanuc 라이브러리에 이 쓰기가 쓰는 함수(`cnc_wrparam`)가 없으면 쓰기가 상태 `-20` 입니다 (들어 있는 함수는 Fanuc 라이브러리 제품에 따라 다릅니다).

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

만들어야 할 목표 수량입니다. `channel` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 100}`. `0` 이면 목표가 설정되지 않은 상태입니다. Fanuc 라이브러리에 이 쓰기가 쓰는 함수(`cnc_wrparam`)가 없으면 쓰기가 상태 `-20` 입니다 (들어 있는 함수는 Fanuc 라이브러리 제품에 따라 다릅니다).

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
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

**이번 자동운전 사이클의 실행 시간**입니다. 사이클을 새로 시작하면 `0` 부터 다시 셉니다. `channel` 필터. 반환 `int` (초) + `unit:"s"`, 읽기 전용.

누적값이 아닙니다. 장비 수명에 걸쳐 쌓이는 값(전원투입 시간·절삭 시간)과 달리 **한 번의 운전**을 잽니다. 정지 중에는 세지 않습니다. Fanuc 은 정지·홀드 시간을 빼고(파라미터 매뉴얼 B-64490EN 의 `6758`) 전원 투입 때와 **리셋 상태에서 사이클 스타트할 때** `0` 으로 돌아갑니다 (홀드에서 재개할 때는 이어서 셉니다). Siemens 는 정지 시간과 이송 오버라이드 `0` 으로 멈춘 시간을 빼고, `M30` 도착과 리셋 상태에서의 재시작에 `0` 이 됩니다 (840D sl 벤치에서 확인: 피드홀드·싱글블록 정지·이송 오버라이드 `0` 동안 멈추고 드웰 동안은 셉니다). Mitsubishi 도 리셋 상태에서 기동하면 `0` 부터 세고, 피드홀드·블록 정지(`M0`) 동안 멈췄다가 재개하면 이어서 세며, 리셋과 `M30` 뒤에도 값이 남습니다 (시뮬레이터에서 확인). Mitsubishi 의 이송 오버라이드 `0` 처리는 확인하지 못했습니다.

**초 미만은 버립니다.** Fanuc·Siemens 는 밀리초 해상도를 제공하지만 경과시간 주소는 정수 초로 통일합니다 (Mitsubishi 는 초 단위로 줍니다). 조작반의 `CYCLE TIME` 표시(Mitsubishi 는 `CYC`)와 같은 값이 나옵니다 (Fanuc·Mitsubishi 테스트 환경과 31i 벤치에서 확인). Fanuc 은 이송 오버라이드 `0` 이나 드웰 동안에도 세고, 피드홀드 동안에는 세지 않습니다 (31i 벤치). 사이클이 몇 초로 짧으면 최대 1초의 오차가 상대적으로 큽니다.

Fanuc 은 파라미터 `6758`(분) + `6757`(분 미만 ms) 를 한 번의 호출로 함께 읽고, Siemens 는 `actProgNetTime`, Mitsubishi 는 사이클 시간(`GetCycleTime`, 레퍼런스 IB-1501209 가 밝히는 범위는 `99:59:59` 까지)입니다.

**Mitsubishi 는 EZSocket `FCSB1224W100-A9` 이상이 필요합니다.** 그보다 앞선 판에서는 상태 `-20` 이고, M800 계열은 그 판 이상을 설치하면 읽힙니다. M700 계열은 판과 관계없이 읽을 수 없습니다. 레퍼런스(IB-1501209)의 지원 표가 이 기능을 M800 계열에만 두고 있기 때문이며, 제어기가 미지원으로 답하면 상태 `-20` 입니다 (M700 계열 장비에서는 확인하지 못했습니다).

## /machine/channel/operatingDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

**자동 운전 시간의 누적값**입니다. 장비가 자동 운전을 하고 있던 시간을 계속 쌓습니다. `channel` 필터. 반환 `int` (초) + `unit:"s"`, 읽기 전용.

`programRunDuration` 과 짝을 이룹니다. 그쪽은 **이번 사이클**만 재고, 이쪽은 **장비 일생**입니다. 이름의 `program` 이 그 범위를 밝히는 자리라, 접두어가 없는 이 주소는 `powerOnDuration`·`cuttingDuration` 과 같은 누적값입니다.

**홀드·정지 중에는 세지 않습니다.** 피드홀드로 세워 둔 시간은 세 기종 모두 빠집니다 (Fanuc 은 파라미터 매뉴얼이 정지·홀드 시간을 제외한다고 명시하고 테스트 환경과 31i 벤치에서도 확인, Mitsubishi 는 홀드 포함/제외 카운터를 벤더가 나눠 주며 이 주소는 제외 쪽, Heidenhain 은 테스트 환경에서 NC 정지와 `M0` 로 멈춘 시간을 세지 않았습니다). 프로그램을 걸어두고 자리를 비운 시간이 가동 시간으로 잡히지 않는다는 뜻입니다.

`/machine/powerOnDuration` 과 함께 읽으면 가동률의 재료가 됩니다. 켜져 있던 시간 중 실제로 운전한 시간의 비율. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

**쓰기는 지원하지 않습니다.** 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다.

Fanuc 은 파라미터 `6752`(분) + `6751`(분 미만 ms) 를 한 번의 호출로 함께 읽고(조작반 실적 화면의 `RUN TIME`), Mitsubishi 는 `GetStartTime` 입니다. 조작반 통합 시간 화면의 **`Auto strt`(자동 기동 시간: 자동 기동 버튼부터 피드홀드·블록 정지·리셋까지의 누적)** 와 같은 값이고, 홀드를 포함하는 `Auto oper`(자동 운전 시간: 기동부터 `M02`/`M30`/리셋까지, `GetRunTime`)는 쓰지 않습니다 (M800 조작 매뉴얼). **제어기는 `59999:59:59` 에서 누적을 멈춥니다** (`powerOnDuration` 과 같은 상한·같은 API 문서 단서).

Heidenhain 은 `GetMachineRunningTime` 입니다. 레퍼런스가 설치 뒤 자동·싱글블록 모드로 프로그램이 실행된 누적 가공 시간이라 밝힙니다. **분 해상도**라 값이 항상 60의 배수입니다. 조작반 설정 → 기계 설정 → 기계 시간의 "프로그램 실행" 과 같은 카운터였습니다 (테스트 환경에서 00:13:31 일 때 `780`).

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

**측정이 멈추는 조건이 기종마다 다릅니다.** Fanuc 은 파라미터 매뉴얼의 정의대로 **절삭 이송(`G01`·`G02`·`G03` 등) 중의 시간만** 쌓습니다. 31i 벤치에서는 이송 오버라이드 `0` 인 동안에도 쌓고(절삭 이송 블록 안이라서), 드웰(휴지)과 피드홀드 동안에는 쌓지 않았습니다. Siemens 는 `$AC_CUTTING_TIME` 의 규칙을 따릅니다: 급속이송은 빼고 드웰(휴지) 중에는 멈추며, 기본 설정에서는 **공구가 활성일 때만** 재고 드라이런·프로그램 테스트 중에는 재지 않습니다 (머신 데이터 `27860` 의 비트 7·4·5 로 바꿀 수 있습니다). Siemens 가 정지 상태와 이송 오버라이드 `0` 에서 어떻게 재는지는 확인하지 못했습니다.

**리셋 기준점이 기종마다 다릅니다.** Fanuc 은 계속 쌓이고, Siemens 는 기본값으로 제어기를 부팅하면 `0` 이 됩니다 (일반 전원 재투입은 무관). 양쪽 모두 조작반에서 작업자가 리셋할 수 있습니다.

**Siemens 는 이 측정을 꺼둘 수 있습니다.** 머신 데이터 `27860` 의 비트 2 가 `0` 이면 측정 자체가 꺼져 있어 항상 `0` 입니다.

**쓰기는 지원하지 않습니다.** 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다.

Fanuc 은 파라미터 `6754`(분) + `6753`(분 미만 ms) 를 합산하고, Siemens 는 `cuttingTime` 입니다. 쪼개진 두 파라미터는 **한 번의 호출로 함께** 읽으므로, 읽는 도중 분이 넘어가 값이 어긋나는 일은 없습니다.

**Mitsubishi 는 상태 `-20` 입니다.** 디메시가 그 기종에서 읽는 시간 값은 통합 시간 화면의 전원 투입·자동 기동 시간과 사이클 시간이고, 절삭 시간은 그 안에 없습니다.

## /machine/channel/mainProgramName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

HMI 에서 **선택된 메인 프로그램**의 이름입니다. 반환 `string`, 읽기 전용. 실행 중 서브프로그램에 들어가도 변하지 않습니다 (그게 `programName` 과의 차이).

출처는 Fanuc `cnc_pdf_rdmain`, Siemens `/Channel/ProgramInfo/selectedWorkPProg`, Mitsubishi `GetProgramNumber2`(선택된 메인) 입니다.

Heidenhain 은 `GetExecutionPoint` 가 주는 선택된 프로그램의 파일 이름(확장자 포함, 경로 없음)입니다. TNC 는 운전 모드마다 프로그램을 따로 두어, 테스트 환경에서 MDI 모드로 가면 MDI 프로그램(`$mdi.h`, TNC7 사용 설명서에 따르면 인치면 `$mdi_inch.h`)이 나왔고 수동 모드에서는 프로그램 실행 모드에서 고른 프로그램이 그대로 나왔습니다.

**선택된 프로그램이 없으면 빈 문자열입니다.** Fanuc(조작반에 `No Program` 이 보일 때)·Mitsubishi(조작반의 프로그램 번호 칸이 비어 있을 때)·Heidenhain 에서 테스트 환경에서, Siemens 는 840D sl 벤치에서 확인했습니다 (조작반의 프로그램 칸이 비어 있을 때입니다. 제어기는 그때 기본 프로그램 `MPF0` 을 가리키지만 디메시는 그것을 내지 않습니다). Fanuc 은 테스트 환경에서 조작반으로 선택된 메인을 지우자 같은 폴더의 다른 프로그램이 곧바로 선택됐고, 폴더의 마지막 프로그램을 지운 뒤에야 빈 문자열이 됐습니다. MDI 모드로 가도 Fanuc 은 선택된 메인 그대로입니다 (테스트 환경에서 확인).

## /machine/channel/mainProgramPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

HMI 에서 **선택된 메인 프로그램**의 전체 경로입니다 (`mainProgramName` 의 경로 버전). 실행 중 서브프로그램에 들어가도 변하지 않습니다. Heidenhain 은 운전 모드마다 선택된 프로그램이 따로라(`mainProgramName` 참조) MDI 모드에서는 MDI 프로그램의 경로가 나오고, 선택된 프로그램을 지우거나 이름을 바꿔도 이 값은 옛 경로를 가리킵니다 (테스트 환경에서 확인).

선택된 프로그램이 없으면 빈 문자열입니다 (Fanuc·Mitsubishi 는 테스트 환경에서 확인, Heidenhain 도 같습니다. Siemens 는 840D sl 벤치에서 확인. Fanuc 에서 그 상태가 되는 때는 `mainProgramName` 참조). MDI 모드로 가도 Fanuc 은 선택된 메인 그대로입니다 (테스트 환경에서 확인).

**쓰기 = 프로그램 선택**: 그 경로의 프로그램을 해당 채널의 실행 대상(메인 프로그램)으로 고릅니다. 값은 경로 문자열입니다: `{"value": "//CNC_MEM/USER/O0001"}`.

- 경로 표기는 기종을 따릅니다. Fanuc `//CNC_MEM/USER/O0001`(데이터 서버는 `//DATA_SV/...`), Siemens `//NC/Part programs/PART1.MPF` (`programPath`·`entryList` 가 돌려주는 표기 그대로. `Subprograms`·`Workpieces` 도 동일), Mitsubishi `//PRG/USER/O0001` (`ncMemoryRootPath` 아래 표기 그대로), Heidenhain `//TNC/nc_prog/PART1.H`. NC 파일시스템 경로는 벤더 고유라 `plcAddress` 와 같은 이유로 통일하지 않습니다
- **파일이어야 합니다.** 폴더 경로를 주면 상태 `-18`. 없는 경로도 상태 `-18`
- 선택만 할 뿐 **가공을 시작하지는 않습니다** (사이클 스타트는 조작반/PLC 몫)
- **선택이 받아들여지는 채널 상태는 제어기가 정합니다.** 그 상태가 아니라서 거절되면 상태 `-22`(기계 상태)이고, 사유에 어느 상태였는지가 실립니다. 기종별 조건:
  - Fanuc: FOCAS2 사양상 선택 함수는 MEM(자동)·EDIT 모드에서만 쓸 수 있어, MDI·JOG 같은 다른 모드에서는 어느 경로든 상태 `-22` 입니다. 자동운전이 기동된 상태(`executionStatus` 값 `3`)에서도 어느 경로든(실행 중인 프로그램 자신을 포함) 상태 `-22` 입니다. 비상정지가 걸려 있을 때도 어느 경로든 상태 `-22` 이고, 사유가 비상정지를 밝힙니다 (31i 벤치와 테스트 환경에서 확인). 블록 정지·피드홀드 같은 정지 상태에서 받아들일지는 장비 사정이며, 받아들여지면 선택된 프로그램이 곧바로 바뀌므로 리셋 상태에서 쓰는 것이 안전합니다. 없는 경로는 상태 `-18` 로 갈립니다. 파라미터 `3202#0`/`#4` 로 보호된 O8000~O8999·O9000~O9999 번 프로그램은 선택되지 않고 상태 `-22` 입니다 (보호를 풀거나 다른 프로그램을 고르세요). 조작반의 편집 금지 속성은 선택을 막지 않습니다 (테스트 환경에서 확인)
  - Siemens: 채널이 Reset 상태여야 합니다 (`Select` 메서드의 필수 조건). 그렇지 않으면 상태 `-22` 입니다
  - Mitsubishi: 프로그램 운전 중에는 운전 검색이 거절되어 상태 `-22` 입니다
  - Heidenhain: 프로그램 실행 모드에서만 받습니다. 수동 운전·MDI 모드에서는 상태 `-22` 이고, 프로그램 운전 중에도 상태 `-22` 입니다. 운전이 블록 경계에서 멈춘 동안(`executionStatus` `1`)에는 받아지고, 끝나지 않은 운전은 버려집니다 (테스트 환경에서 확인. 이어서 가공하려면 운전이 끝나거나 취소된 뒤에 쓰세요). 폴더 경로·없는 폴더 아래의 경로·없는 파일은 모두 상태 `-18` 입니다 (테스트 환경에서 확인)
- Siemens 는 서버의 파일 핸들링 `Select` 메서드를, Fanuc 은 CNC 메모리와 데이터 서버 모두 `cnc_pdf_slctmain` 을, Mitsubishi 는 운전 검색 `Search` 를, Heidenhain 은 `SelectProgram` 을 씁니다. Fanuc 의 데이터 서버 경로에서 선택이 기계 상태가 아닌 사유로 거절되면 storage 모드의 DNC 운전 파일 설정(`cnc_wrdsdncfile`)을 시도하고, 설정한 파일이 되읽혀 확인될 때만 성공으로 답합니다

## /machine/channel/programName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

**현재 실행 중인** 프로그램의 이름(파일명)입니다. 반환 `string`, 읽기 전용. 서브프로그램에 들어가면 그 서브의 이름으로 바뀝니다 (테스트 벤치에서 서브프로그램 실행 중 그 서브의 파일명이 오고, 나오면 메인으로 돌아오는 것을 확인했습니다). HMI 에서 선택된 메인은 `mainProgramName` 참조.

출처는 Fanuc `cnc_exeprgname2`, Siemens `/Channel/ProgramInfo/workPandProgName`, Mitsubishi `GetProgramNumber2`(실행 중 프로그램) 입니다.

표기는 기종을 따릅니다. Fanuc 은 `O0003` 처럼 O 번호, Siemens 는 **확장자를 포함한 파일명**(`PART1.MPF`, 서브프로그램에 들어가면 `SUB1.SPF`)이고 경로는 붙지 않습니다 (경로가 필요하면 `programPath`). Mitsubishi 는 프로그램 파일 이름입니다 (M700·M800 계열은 이 자리에 파일 이름을 줍니다, 레퍼런스 IB-1501209 의 `GetProgramNumber2`).

**프로그램이 선택돼 있지 않으면 빈 문자열입니다** (Fanuc·Mitsubishi 는 테스트 환경에서 확인, Heidenhain 도 같습니다. Siemens 는 840D sl 벤치에서 확인). Fanuc 은 MDI 모드에서 경로 없이 `O0000` 입니다 (테스트 환경에서 확인).

Heidenhain 은 `GetExecutionPoint` 가 주는 호출 목록에서 마지막 프로그램의 파일 이름(확장자 포함)입니다. 서브프로그램에 들어가면 그 서브의 이름이 됩니다 (테스트 환경에서 확인). 프로그램을 고르기만 하고 아직 시작하지 않았을 때는 선택된 메인의 이름입니다 (프로그램 포인터가 메인에 있는 것으로 봅니다. `programNestLevel` 이 `1` 인 것과 같습니다).

## /machine/channel/programPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

현재 실행 중인 프로그램의 전체 경로입니다 (예: `//CNC_MEM/USER/PATH1/O0001`). `channel` 필터. Siemens 는 NCK 내부 경로를 사용자 표기(`//NC/...`)로 변환해 돌려줍니다. Heidenhain 도 `TNC:` 드라이브의 경로를 `//TNC/...` 로 옮겨 돌려주고(다른 드라이브에서 도는 프로그램은 확인하지 못했습니다), 서브프로그램(`CALL PGM`)에 들어가면 그 서브의 경로가 됩니다. 운전을 시작하기 전에는 선택된 메인의 경로입니다 (테스트 환경에서 확인).

프로그램이 선택돼 있지 않으면 빈 문자열입니다 (Fanuc·Mitsubishi 는 테스트 환경에서 확인, Heidenhain 도 같습니다. Siemens 는 840D sl 벤치에서 확인). Fanuc 은 MDI 모드에서 경로 없이 `O0000` 입니다 (테스트 환경에서 확인).

**Mitsubishi 는 폴더 부분이 실행 중인 프로그램의 것이라는 보장이 없습니다.** 디메시는 이 기종에서 디렉터리와 파일 이름을 따로 읽어 잇는데, **실행 중인 프로그램의 디렉터리를 알려 주는 값은 찾지 못했습니다**. 다만 이 기종의 NC 메모리는 **디렉터리 구성이 고정**이라(폴더를 만들 수 없습니다. `directoryExists` 참조) 사용자 프로그램끼리는 메인과 서브가 다른 폴더에 놓이는 일이 드물어, 실무에서는 대개 맞습니다. 다만 고정 사이클(`//PRG/FIX`)이나 기계 제조사 매크로(`//PRG/MMACRO`)가 호출돼 도는 동안에는 폴더 부분이 메인의 폴더로 나올 수 있습니다 (확인하지 못했습니다). 파일 이름 쪽은 언제나 실행 중인 프로그램의 것입니다.

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

**N 번호를 쓰지 않은 프로그램에서는 이 값으로 실행 위치를 알 수 없습니다.** 위 표처럼 N 없는 블록에서는 `0` 이거나 앞선 N 이 남을 뿐, 블록을 세지 않습니다. 실행 위치는 `/machine/channel/programBlockCounter` 로 읽으세요. 그 값이 무엇을 세는지는 기종마다 다르니 그 주소의 설명을 함께 보세요.

**Mitsubishi 에서 유지되는 것은 운전 중까지입니다.** 프로그램이 끝나 리셋 상태가 되면 `0` 으로 돌아갑니다 (시뮬레이터에서 확인: `N400` 을 마지막으로 실행하고 `M30` 이후 `0`). Fanuc 은 프로그램이 끝난 뒤와 리셋 뒤에도 값이 남고 조작반의 N 표시와 같지만, 마지막 블록의 번호는 아닐 수 있습니다 (31i 벤치: `N40 M30` 으로 끝난 뒤 `30`, 조작반도 `N00030`). 어느 기종이든 이 값을 "마지막으로 실행한 N" 으로 믿지 마세요.

**서브프로그램에 들어가면 서브의 N 이 나옵니다** (`programName` 이 서브 이름으로 바뀌는 것과 같은 시점). 복귀하면 메인의 N 으로 돌아옵니다.

⚠️ **Fanuc 의 유지는 파일 경계를 넘습니다** (테스트 환경에서 확인). 서브에서 복귀한 직후의 N 없는 블록에서는 **서브의 마지막 N** 이, 서브 진입 직후의 N 없는 블록에서는 **메인의 N** 이 그대로 보입니다. 즉 Fanuc 에서는 이 값만으로 어느 파일의 N 인지 판단할 수 없습니다. `/machine/channel/programName` 을 함께 읽으세요. Siemens 는 값을 유지하지 않으므로 이 상황이 생기지 않습니다. Mitsubishi 는 매 조회를 **지금 실행 중인 쪽**(메인/서브)에 지정해 물으므로, 값은 항상 현재 실행 중인 프로그램의 N 입니다.

## /machine/channel/programBlockCounter
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

실행 블록 카운터입니다. `channel` 필터. 반환 `int`.

⚠️ **기종마다 세는 것이 다릅니다.** 값이 서로 비교되지 않으므로 **기종을 가로지르는 진척도로 쓰지 마세요** (네 기종 테스트 환경에서 확인):

| | 세는 것 | 리셋 시점 |
|---|---|---|
| Fanuc | 사이클 시작부터 실행한 블록 수 (`cnc_rdblkcount`, 서브프로그램 블록도 이어서) | Cycle Start |
| Siemens | 지금 실행 중인 **파일 안의 행 번호** (`actLineNumber`, 음수는 `0` 으로 클램프) | 파일이 바뀔 때 (서브 진입·복귀) |
| Mitsubishi | 지금 `N` 번호로부터 **몇 블록 지났는지** | `N` 번호를 만날 때마다 |
| Heidenhain | 지금 실행 중인 **파일 안의 블록 번호**(대화형 프로그램의 줄 앞 번호, `BEGIN PGM` 이 `0`) | 파일이 바뀔 때 (`CALL PGM` 진입·복귀) |

**Mitsubishi 는 `programSequenceNumber` 와 한 쌍입니다.** 이 기종은 프로그램 안의 위치를 (프로그램 이름 · `N` 번호 · 그 N 으로부터의 블록 수) 세 값으로 지정하며, 조작반의 운전 검색도 같은 세 값을 받습니다. 그래서 이 숫자만으로는 위치가 정해지지 않고 `N` 과 함께 읽어야 위치가 정해집니다 (시뮬레이터에서 확인: `N100` 구간에서 `0`→`1`→`2`→`3`, `N500` 을 만나 `0`).

Siemens 의 행 번호는 **지금 실행 중인 파일 기준**입니다. 서브프로그램에 들어가면 서브 파일의 행 번호로 바뀌고, 메인으로 복귀하면 메인 파일의 행 번호로 돌아옵니다 (테스트 환경에서 확인). 그래서 이 숫자만으로는 어느 파일의 몇 행인지 알 수 없습니다. 메인의 3행과 서브의 3행이 같은 `3` 입니다. 파일까지 특정하려면 `/machine/channel/programName` · `/machine/channel/programNestLevel` 을 함께 읽으세요. 중첩 전환 순간(1초 미만)에는 세 값의 조합이 잠시 어긋난 샘플이 나올 수 있습니다 (레벨이 이름보다 먼저 갱신됨).

Heidenhain 도 **지금 실행 중인 파일 기준**이라 Siemens 와 같은 방법으로 읽으세요. 같은 파일 안의 서브루틴(`CALL LBL`)을 도는 동안에는 그 파일의 블록 번호가 나옵니다. 블록이 실행 중일 때만 번호를 내고 그 밖에는 `0` 입니다: 운전을 시작하기 전과 운전이 끝난 뒤가 `0` 이고 (끝나면 조작반의 커서도 프로그램 처음으로 돌아갑니다), `M0`·싱글블록으로 멈춰 있으면 그 블록의 번호입니다. MDI 는 블록을 실행하는 동안에도 `0` 입니다 (테스트 환경에서 그동안 HEIDENHAIN DNC 의 프로그램 상태가 '선택된 프로그램 없음' 이었고, MDI 의 실행 상태를 읽는 다른 길은 찾지 못했습니다).

## /machine/channel/programLastBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

직전에 실행된 블록의 G코드 텍스트입니다. `channel` 필터. 직전 블록이 없으면(프로그램의 첫 블록이거나 운전 중이 아닐 때) 빈 문자열입니다.

**Siemens·Mitsubishi 지원이고 Fanuc 은 상태 `-20`** 입니다. 디메시가 Fanuc 에서 블록 텍스트를 읽는 `cnc_rdexecprog` 는 선독 버퍼, 즉 **앞으로 실행할** 블록을 돌려주는 함수라 지나간 블록은 이 통로로 읽을 수 없습니다.

## /machine/channel/programCurrentBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

지금 실행 중인 블록의 **G코드 텍스트**입니다. `channel` 필터. 없으면 **빈 문자열**이며 `null` 이 아닙니다.

**"실행 중이 아닐 때" 의 답이 기종에 따라 다릅니다.** 프로그램을 걸어 두고 리셋 상태인 제어기를 테스트 환경에서 확인한 결과입니다:

| | Siemens · Mitsubishi | Fanuc |
|---|---|---|
| `programCurrentBlock` | `""` | 프로그램의 **첫 줄** |
| `programNextBlock` | 첫 블록 | 그 **다음** 줄 |

Heidenhain 은 Siemens·Mitsubishi 와 같이 `""` 입니다. 프로그램이 운전 중이거나 정지·중단 상태일 때만(오류로 멈춘 동안 포함, `executionStatus` 가 `0` 이 아닐 때) 그 블록의 원문을 내고, 운전을 시작하기 전과 운전이 끝난 뒤에는 `""` 입니다. 원문은 대화형 프로그램의 줄 그대로라 앞의 블록 번호까지 들어 있습니다 (예: `"9 CYCL DEF 9.0 DWELL TIME"`, 테스트 환경에서 확인). `programNextBlock` 은 Heidenhain 에서 상태 `-20` 입니다.

Siemens 와 Mitsubishi 는 "실행 중인 블록 없음" 을 표현할 수단이 있습니다 (Mitsubishi 는 벤더가 실행 위치를 `0`=운전 안 함으로 알려줍니다). Fanuc 에서 디메시가 읽는 `cnc_rdexecprog` 는 선독(look-ahead) 버퍼를 돌려주므로 그 첫 줄이 "현재" 로 나가는데, 정지 중에 그 줄은 실제로는 **다음에 실행될 블록**입니다.

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
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

프로그램 호출 중첩 단계입니다 (뜻은 `desc` 로 함께 옵니다): `0` = 프로그램 없음, `1` = 메인, `2`~ = 서브프로그램 (L1, L2, …). `channel` 필터. **Siemens·Mitsubishi·Heidenhain 지원** (Fanuc 은 상태 `-20`).

**세는 것은 실행이 아니라 프로그램 포인터의 깊이입니다.** 프로그램이 걸려 있으면 운전 중이 아니어도 `1` 입니다 (테스트 환경에서 확인: Siemens·Mitsubishi 는 리셋·중단 상태에서 `1`). "지금 돌고 있나" 는 `/machine/channel/executionStatus` 가 답합니다.

Mitsubishi 는 벤더 값이 **서브프로그램을 몇 겹 파고들었는지**(메인이 `0`)라 우리 눈금과 한 칸 달라, 디메시가 맞춰서 내보냅니다. `0`(프로그램 없음)과 `1`(메인)을 가르기 위해 이 주소만 장비 조회가 한 번 더 붙습니다.

Heidenhain 은 다른 프로그램을 부르는 호출을 한 단계로 세고, 같은 파일 안의 서브루틴을 부르는 `CALL LBL` 은 세지 않습니다 (테스트 환경에서 `CALL PGM` 으로 확인. `CALL SELECTED PGM`·사이클 12 로 부른 경우는 확인하지 못했습니다). TNC7 사용 설명서('Nesting of programming techniques')는 다른 프로그램을 부르는 중첩을 19단까지로 적어, 이 값은 `20` 까지입니다. 운전을 시작하기 전에도 프로그램이 선택돼 있으면 `1` 입니다.

## /machine/channel/variable/variableValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "variable"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

**매크로 변수(Fanuc·Mitsubishi) / R 파라미터(Siemens)** 를 읽고/씁니다 (read + write). `variable` 필터에 변수 번호 (예: `variable=100` → Fanuc·Mitsubishi `#100`, Siemens `R100`). 반환 `float`, 쓰기는 `{"value": 3.14}`. **읽기는** 범위/콤마 확장을 지원합니다. `variable=100-105` 는 6개 값 배열. 쓰기는 항상 단일 변수입니다 (확장 문법은 상태 `-13` 으로 거절: 모든 쓰기 공통 규칙). Fanuc·Mitsubishi 의 **미설정(vacant) 매크로 변수는 `null`** 입니다. 조작반 커스텀 매크로 화면에 빈 칸(Fanuc 은 `DATA EMPTY`)으로 뜨는 그 상태이며 값 `0` 과 구분됩니다. 범위 확장에서도 그 자리만 `null` 이 됩니다 (예: `[3.14, null]`).

**쓸 수 있는 번호는 기종·옵션마다 다릅니다.** 그 장비에 없는 번호는 **상태 `-18`** 로 돌아옵니다 (읽기·쓰기 모두). 고칠 것은 `variable` 값 하나이며, 그 장비에 실제로 있는 번호는 조작반의 변수 화면이 알려줍니다. Fanuc·Mitsubishi 는 디메시가 목록을 들지 않고 번호를 그대로 전달하므로 에러 문자열에 벤더가 밝힌 사유가 실립니다. **Siemens 는 R 파라미터 개수를 알고 있습니다.** 연결할 때 채널별로 `numRParams`(머신 데이터 `28050`)를 읽어 두고, **R 은 `0` 부터** 세므로 `0`~개수-1 을 벗어난 번호는 장비에 묻지 않고 즉시 상태 `-18` 로 거절하며 에러 문자열에 허용 범위와 개수를 싣습니다 (예: `expected 0-99`). 그래서 쓰기 전에 읽기로 번호를 확인하는 방어가 Siemens 에서도 통합니다. 범위 확장에 없는 번호가 섞이면 요청 **전체가 상태 `-15`** 로 실패합니다 (부분 배열이 오지 않습니다). 미설정(vacant) 변수는 에러가 아니라 `null` 원소라 확장을 깨지 않는 것과 구분하세요. 문법 자체가 범위 밖인 번호(Fanuc 은 `0`~`89999`)도 같은 상태 `-18` 이되, 이쪽은 장비에 묻지 않고 즉시 거절합니다. Fanuc 의 `#10000` 이상은 P-code 매크로 변수라 Macro Executor 옵션이 있는 장비에만 있습니다. 옵션이 없으면 그 번호도 같은 상태 `-18` 이고 에러 문자열에 옵션 이름이 실리며, 배치(`/read/batch`·`deemesh_read_batch`)에 함께 넣은 다른 번호는 그대로 읽힙니다 (테스트 환경에서 확인. 범위·콤마 확장은 위처럼 요청 전체가 상태 `-15` 입니다). 옵션이 있는 장비에서 P-code 매크로 변수의 값을 읽고 쓰는 것은 아직 확인하지 못했습니다.

**Mitsubishi 에서 제어기가 보호로 막은 쓰기는 상태 `-22`(기계 상태)입니다.** 파라미터 `#12111`~`#12114` 가 정한 공통 변수 설정 보호 범위에 든 번호, 그리고 데이터 보호 키 2(PLC 신호 `*KEY2`, `Y709`)가 꺼진 동안의 쓰기가 그렇습니다 (시뮬레이터에서 확인). 번호가 없는 것이 아니므로 조작반에서 보호를 푼 뒤 다시 쓰세요. 읽기는 막히지 않습니다.

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
- `[r,c](열수)` → 2차원 배열의 (행,열) 원소 1개. 대괄호는 화면 표기 그대로 적고, 배열의 **열 개수**만 괄호로 덧붙입니다 (열 개수는 읽기만으로는 알 수 없어 함께 입력합니다)
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
- `[r,c](열수)` → 2차원 배열의 (행,열) 원소 1개. **대괄호는 화면 표기 그대로** 적고, 배열의 열 개수만 괄호로 덧붙입니다. 화면에 열이 `[0,3]` 까지 보이면 `(4)` (열 개수는 읽기만으로는 알 수 없어 함께 입력합니다)
- 예: `SGUD:MYVAR` (스칼라), `SGUD:_SC_NCK_ROU_S[1]` (1D 의 화면 표기 [1]), `SGUD:POS[0-2]` (1D 의 [0]~[2] 3개), `SGUD:_SC_C97[0,1](4)` (4열 2D 의 화면 표기 [0,1])

**type** (엔벨로프 안). 원소의 실제 타입입니다:

- `BOOL`: 참/거짓
- `CHAR`: 문자 코드 (0~255 정수)
- `INT`: 정수
- `REAL`: 실수 (Siemens R 파라미터와 같은 64비트 실수)
- `STRING`: 문자열

(구조형 GUD `AXIS` / `FRAME` 은 지원하지 않습니다. 그 타입으로 쓰면 상태 `-16` 으로 거절합니다)

**data** (엔벨로프 안). 1개(스칼라 / `[i]` / `[r,c](열수)`)면 값 하나, `[i-j]` 범위면 JSON 배열:

- 스칼라/단일: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- 범위: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**쓰기**: 읽기와 **같은 object** 를 `value` 에 담습니다 (예: `{"value":{"type":"REAL","data":42.0}}` → 화면 표기 [i] 한 칸만 변경). `type` 으로 쓸 타입을 정하므로 읽기 없이 바로 씁니다. `data` 원소 개수는 범위 크기 (단일이면 1)와 정확히 일치해야 합니다.

**주의**: GUD 영역이 없는 구형 NCK 에서는 상태 `-20`(미지원)으로 답합니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** GUD 는 Siemens 고유의 사용자 변수 체계이고, 두 기종의 사용자 변수(매크로 변수·공통 변수)는 `/machine/channel/variable/variableValue` 로 읽고 씁니다.

## /machine/plcAddress/plcType/plcValue
```yaml
value_type: "float"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
filter_codes: {"plcType": [{"value": 0, "name": "auto", "protocols": ["nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": 1, "name": "bit", "protocols": ["nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]}, {"value": 2, "name": "uint8"}, {"value": 3, "name": "int16"}, {"value": 4, "name": "int32"}, {"value": 5, "name": "float32", "protocols": ["nc_focas2_fanuc", "nc_opcua_siemens"]}, {"value": 6, "name": "float64", "protocols": ["nc_focas2_fanuc", "nc_dnc_heidenhain"]}, {"value": 7, "name": "int8"}, {"value": 8, "name": "uint16"}, {"value": 9, "name": "uint32"}]}
```

**PMC/PLC 메모리의 단일 원소**를 읽고/씁니다 (Fanuc FOCAS2 `pmc_rdpmcrng`/`pmc_wrpmcrng`, Siemens OPC-UA `/Plc/` 노드, Mitsubishi EZSocket `ReadDevice`/`WriteDevice`, HEIDENHAIN DNC PLC 데이터). 읽기와 쓰기 모두 지원하며, 반환 타입은 `float` (값 하나), 쓰기도 숫자 하나입니다 (예: `{"value": 42}`). 이 주소는 **원소 하나 전용**이며, 여러 원소를 한 번에 다루려면 같은 트리의 목록형 주소를 사용합니다. `plcAddress` 와 `plcType` 두 필터가 필요합니다.

**plcAddress**: 주소 형식이 **기종별로 다릅니다**. `plcType` 과 달리 **중립화하지 않는 의도적 예외**입니다. Fanuc 의 `D100` 과 Siemens 의 `DB10.DBB56` 는 서로 다른 메모리 아키텍처를 가리키고, 둘을 잇는 대응표는 SDK 가 알 수 있는 지식이 아니라 그 장비의 래더를 어떻게 짰는지에 달린 현장 설정이기 때문입니다. 기종 간에 통일하지 않으므로, 여러 기종에서 같은 신호를 읽어야 한다면 **호스트 앱이 기종별 주소 표를 들고** 있어야 합니다.

- **Fanuc**: 첫 글자가 PMC 영역, 나머지가 바이트 번호 (예: `R5`, `D100`). `~` 로 범위 지정. 단, 이 주소(단일)는 범위가 **정확히 `plcType` 크기 1개**여야 합니다 (예: word 면 `D100~D101`)
- **Fanuc** PMC 영역 첫 글자: `G` `F` `Y` `X` `A` `R` `T` `K` `C` `D` `M` `N` `E` `Z`: 범위는 같은 영역이어야 합니다 (`D100~D101` O, `D100~R101` X)
- **Fanuc 에서는 이 주소가 비트 주소(`R38.7` 같은 `바이트.비트` 표기)를 받지 않습니다.** 바이트를 읽어 비트를 떼어 쓰세요: `plcAddress=R38&plcType=2` 의 값은 조작반 PMC 신호 화면의 `HEX` 열과 같은 바이트 값(`0xDC` 면 `220`)이고, 비트 7 은 `(220 >> 7) & 1` 입니다. 쓰기도 바이트 단위입니다. 이유: FOCAS 의 PMC 읽기/쓰기 함수에 비트 타입이 없어, 비트 쓰기를 만들면 "바이트를 읽어 고친 뒤 다시 쓰기" 가 되고 그 사이 래더가 바꾼 다른 비트를 덮어쓸 수 있습니다. 신호 이름은 `바이트.비트`(예: `R0039.4`) 로 불러도 주소는 바이트로 넣으세요
- **Siemens**: 조작반의 **`NC/PLC variables` 화면에 보이는 표기 그대로** 씁니다. 값이 `/Plc/{주소}` 노드로 전달됩니다. 첨자 생략 시 `[1]` 자동 부착. 이 주소는 단일 원소만 되며, 다원소면 에러와 함께 목록형 주소를 안내합니다
- **Siemens** 형식: **주소가 오프셋을 품습니다**. `DB<n>.DBB<offset>`(바이트) · `DB<n>.DBW<offset>`(워드) · `DB<n>.DBX<byte>.<bit>`(비트) · `IW<n>` · `MB<n>` · `Q<byte>.<bit>`. 표기 예: `DB10.DBB56` · `DB31.DBX24.1` · `IW0` · `Q0.2`
- **Siemens** 첨자 `[N]` 은 **"몇 번째" 가 아니라 "몇 개"** 입니다. `DB10.DBB56[4]` 는 오프셋 56 부터 **연속 4개**(56·57·58·59)이지 "56번의 4번째" 가 아닙니다. 다른 자리를 짚으려면 첨자가 아니라 **주소를 옮깁니다** (`DB10.DBB61`)
- **Siemens** 문법 주의(기종 무관): 오프셋 없는 표기(`MB` 단독 · `DB<n>` 단독)는 문법이 아니고, 비트는 `DBB` 가 아니라 `DBX` 로 짚습니다
- **Siemens**: 어느 블록·바이트가 **그 장비에 실재하는지는 래더 구성에 달려 기계마다 다릅니다.** 위 표기 예도 형태를 보이기 위한 것이지 어느 장비에나 있는 주소가 아닙니다. 조작반의 같은 화면에서 확인하세요. 거기서 값이 보이면 여기서도 읽힙니다
- **Siemens** 828D 제약: 828D 는 **`DB9000` 이상의 고객 데이터 블록에만** 접근할 수 있습니다 (840D sl 은 제약 없음)
- **Mitsubishi**: 조작반의 PLC 화면 표기 그대로 `<디바이스><번호>` 입니다 (예: `R100`, `M50`, `Y8A0`). 점 수는 **`[N]` 첨자**로 붙이며, Siemens 와 같이 **"몇 번째" 가 아니라 "몇 개"** 입니다. `R100[4]` 는 `R100` 부터 연속 4점. 이 주소(단일)는 첨자 없이 쓰거나 `[1]` 이어야 합니다
- **Mitsubishi** 디바이스 번호의 진법이 계열마다 다릅니다. `M`·`R`·`D` 는 10진수, `X`·`Y`·`B` 는 **16진수**입니다 (조작반 표기와 같습니다)
- **Mitsubishi** 정렬: `M`·`X`·`Y` 처럼 **비트 단위로 번호가 매겨진 디바이스**는 byte·word·dword 로 읽을 때 시작 번호가 **8점 경계**에 있어야 합니다 (`Y890` O, `Y894` X). `R`·`D` 처럼 워드 단위 디바이스는 제약이 없습니다. 어긋나면 상태 `-18` 이며, 디메시가 표로 판정하지 않고 **장비에 직접 물어** 가르므로 그 장비가 받는 주소는 그대로 통과합니다
- **Heidenhain**: 두 가지 표기를 받습니다. **PLC 메모리** `<종류><번호>`(예: `M10`, `B10`, `W100`, `D8`)와 **PLC 전역 심볼 이름**(예: `ApiAxis[0].NN_AxDriveOn`)입니다. 메모리 표기의 모양이면 메모리로, 아니면 전역 심볼 이름으로 읽습니다. 종류 글자는 대소문자를 가리지 않습니다
- **Heidenhain** 메모리 종류는 `M`·`I`·`O`·`T`·`C`·`B`·`W`·`D`·`R`·`S`·`IB`·`IW`·`ID`·`OB`·`OW`·`OD` 입니다. 번호는 **바이트 주소**라, 칸 하나가 2바이트인 `W` 는 `W0`·`W2`·`W4`, 4바이트인 `D` 는 `D0`·`D4`, 8바이트인 `R` 은 `R0`·`R8` 처럼 칸의 폭만큼 건너뜁니다. 칸의 첫 바이트가 아닌 번호(`W1`)는 상태 `-18` 입니다 (저희 테스트 환경인 TNC7 프로그래밍 스테이션에서 확인)
- **Heidenhain** 개수 첨자 `[N]` 은 메모리 표기에만 붙고, Siemens·Mitsubishi 와 같이 **"몇 번째" 가 아니라 "몇 개"** 입니다 (`W100[4]` 는 `W100`·`W102`·`W104`·`W106`). 이 주소(단일)는 첨자 없이 쓰거나 `[1]` 이어야 합니다. 심볼 이름 안의 대괄호(`ApiAxis[0].NN_AxDriveOn`)는 개수가 아니라 이름의 일부입니다
- **Heidenhain** 심볼 이름은 그 장비의 PLC 프로그램이 정합니다. 디메시는 심볼 목록을 읽지 못하므로 **이름을 알아야** 읽을 수 있습니다. 이름은 기계 제작사에 문의하세요
- **Heidenhain**: 읽기와 쓰기에는 연결 설정의 `access_password` 가 필요합니다. 없거나 제어기가 거절하면 상태 `-20` 이고 사유에 어느 쪽인지 실립니다. **쓰기 주의**: TNC7 사용 설명서는 PLC 를 바꾸면 제어기를 쓸 수 없게 될 수도 있어 PLC 접근을 암호로 막는다고 적고, Heidenhain·기계 제작사와 협의한 뒤에 쓰라고 안내합니다

**plcType**: PLC 칸 하나를 어떤 형으로 읽고 쓸지 정하는 숫자 코드입니다. **기종 무관 통일 값**이라 `1`~`9` 는 어느 기종에서든 폭과 부호까지 같은 약속이고, 같은 비트에서 같은 값이 나옵니다:

- `1` = bit: 1비트 (0 / 1)
- `2` = uint8: 8비트 정수, 부호 없음 (0~255) · 주소 폭 1 (예 `D100`)
- `7` = int8: 8비트 정수, 부호 있음 (-128~127) · 주소 폭 1
- `3` = int16: 16비트 정수, 부호 있음 · 주소 폭 2 (예 `D100~D101`)
- `8` = uint16: 16비트 정수, 부호 없음 (0~65535) · 주소 폭 2
- `4` = int32: 32비트 정수, 부호 있음 · 주소 폭 4 (예 `D100~D103`)
- `9` = uint32: 32비트 정수, 부호 없음 · 주소 폭 4
- `5` = float32: 32비트 실수 · 주소 폭 4 (예 `D100~D103`)
- `6` = float64: 64비트 실수 · 주소 폭 8 (예 `D100~D107`)

같은 바이트 `0xA0` 은 `2` 로 읽으면 `160`, `7` 로 읽으면 `-96` 입니다. 정수 형(`1`~`4`·`7`~`9`)의 쓰기는 그 형의 범위 안에 드는 정수만 받고, 범위 밖이거나 소수부가 있으면 상태 `-16` 입니다 (잘라 넣거나 반올림하지 않습니다). 실수 형(`5`·`6`)은 소수를 받되 유한한 수여야 하고 `5` 는 float32 범위 안이어야 합니다 (아니면 `-16`).

`0` = **auto**: 그 기종이 그 칸에 주는 기본 형입니다. **값이 기종에 따라 다를 수 있는 코드는 `0` 하나입니다** (Siemens 는 부호 없음, Heidenhain 은 부호 있음, 아래). Fanuc·Mitsubishi 처럼 원시 메모리를 다루는 기종은 기본 형이 없어 `0` 이 상태 `-18` 이고 형을 지정해야 합니다.

칸의 폭을 정하는 자리는 기종마다 다릅니다. **Fanuc·Mitsubishi 는 `plcType` 이 폭을 정하고, Siemens·Heidenhain 은 주소가 폭을 정합니다.** 주소가 폭을 정하는 기종에서 `1`~`9` 는 그 칸의 폭과 맞아야 하고, 어긋나면 상태 `-18` 이며 에러 문자열에 그 칸의 폭과 받는 코드가 실립니다.

**중요(Fanuc)**: `plcAddress` 범위의 바이트 수가 `plcType` 크기와 일치해야 합니다 (예: `plcType=3`(int16, 2바이트)인데 `D100` 단일 주소면 실패 → `D100~D101` 로 지정). PMC 읽기는 바이트 단위라 `1`(bit)도 받지 않습니다. `2`~`9` 중에서 지정하세요. 결과는 `float`(JSON 숫자)로 반환됩니다.

**Siemens** 는 칸의 폭이 주소에 들어 있습니다 (`DBX`·`Qx.y`·`Ix.y`·`Mx.y` 는 비트, `DBB`·`QB`·`IB`·`MB` 는 바이트, `DBW`·`QW`·`IW`·`MW` 는 워드, `DBD`·`QD`·`ID`·`MD` 는 더블워드). 비트 주소는 `1`, 바이트 주소는 `2`·`7`, 워드 주소는 `3`·`8`, 더블워드 주소는 `4`·`9`·`5` 를 받습니다. `0`(auto)은 서버의 기본 형이라 **바이트·워드·더블워드 모두 부호 없는 정수**입니다. 부호 있는 값은 `7`·`3`·`4` 로 읽으세요. **실수(REAL)는 `plcType=5` 로 읽고 씁니다.** 더블워드 주소에서만 되며, 조작반 `NC/PLC variables` 화면의 `F` 형식에 해당합니다 (840D sl 벤치에서 읽기와 쓰기 확인). `plcType=6`(float64)은 이 PLC 접근에 8바이트 실수가 없어 상태 `-18` 입니다. 타이머·카운터처럼 폭이 정해지지 않은 자리는 `0` 만 받습니다. 쓰기는 노드를 먼저 읽어 서버 형을 확인한 뒤 같은 형으로 기록합니다.

**Mitsubishi** 는 `1`(bit)·`2`·`7`(byte)·`3`·`8`(word)·`4`·`9`(dword) 를 받습니다. `0`(auto)은 Fanuc 과 같은 이유로 안 되고(원시 메모리라 고유 타입이 없음), `5`·`6`(실수)은 이 기종의 PLC 디바이스 API 가 정수만 실어 나르기 때문입니다. 셋 다 상태 `-18` 이며 주소 자체는 정상 동작합니다. **쓰기는 byte(`2`·`7`) 한 점을 받지 않습니다** (상태 `-18`). 이 기종에서 byte 로 쓰는 호출은 블록 쓰기뿐이고 블록 쓰기는 2점 이상을 다루어 옆 디바이스까지 함께 바꾸므로, 디메시가 한 점짜리인 이 주소의 byte 쓰기를 거절합니다. word(`3`·`8`)로 쓰거나 목록형 주소 `/machine/plcAddress/plcType/plcValueList` 로 2점 이상을 지정하세요. 읽기는 byte 로도 됩니다.

**Heidenhain** 은 메모리 칸의 폭을 종류 글자가 정하고 (`M`·`I`·`O`·`T`·`C` 비트, `B`·`IB`·`OB` 바이트, `W`·`IW`·`OW` 워드, `D`·`ID`·`OD` 더블워드, `R` 8바이트 실수), 심볼은 제어기가 알려 주는 값의 범위로 정합니다. 비트 칸은 `1`, 바이트 칸은 `2`·`7`, 워드 칸은 `3`·`8`, 더블워드 칸은 `4`·`9`, `R` 칸은 `6` 을 받습니다. `5`(float32)는 이 제어기에 4바이트 실수 칸이 없어 상태 `-18` 입니다. `0`(auto)은 제어기가 주는 그대로라 **정수는 부호 있는 값**입니다 (테스트 환경에서 조작반 PLC 표의 10진 표시와 같았습니다). 비트 칸은 `0`/`1` 로 옵니다. 쓰기도 같은 형으로 합니다. `0`(auto)으로 쓰면 정수 칸은 부호 있는 범위를 받으므로(바이트 칸이면 `-128`~`127`), `128`~`255` 는 `2` 로 쓰세요. 쓰기는 테스트 환경에서 메모리 칸(`M`·`B`·`W`·`D`·`R`)으로 확인했고, 심볼 이름에 쓰는 것은 아직 확인하지 못했습니다.

**문자열인 자리는 `/machine/plcAddress/plcText` 로 읽습니다.** Heidenhain 에서 그 자리의 값이 문자열이면 이 주소는 상태 `-25` 로 답합니다. 같은 `plcAddress` 로 `/machine/plcAddress/plcText` 를 읽으세요 (그 주소는 `plcType` 을 쓰지 않습니다). Heidenhain 의 `S` 메모리와 문자열 심볼이 그런 자리입니다 (테스트 환경에서 확인). Siemens 는 이 주소가 같은 바이트를 숫자로 읽고, 문자열 변수는 `plcText` 가 바이트 주소로 읽습니다. Fanuc·Mitsubishi 에서는 이 응답이 나오지 않고, Siemens 도 조작반 표기로 적은 주소에서는 나오지 않습니다. 범위·콤마로 여러 자리를 한 번에 읽는 요청(`plcAddress=W0,S0`)에서는 한 자리라도 실패하면 요청 전체가 상태 `-15` 이므로, 이 응답도 에러 문구에만 남습니다. 그런 자리는 따로 읽으세요. 문자열로는 쓸 수 없습니다. 이 주소로 문자열 자리에 쓰면 상태 `-18` 이고(Siemens·Heidenhain), `plcText` 는 읽기만 합니다.

**쓴 값이 그대로 남지 않을 수 있습니다.** 쓰기는 제어기가 받아 상태 `0` 이 되지만, 기계 쪽에서 그 자리를 계속 자기 값으로 채우는 구성이 있는 경우가 있습니다. 그런 자리인지는 그 장비의 설정에 달려 있으니, 쓴 뒤에 읽어 확인하세요.

**에러 코드**: 그 장비에 **실재하지 않는 주소**도 상태 `-18`(필터 값 오류)입니다. 어느 블록·바이트가 있는지는 그 장비 래더 구성에 달렸으므로, 조작반의 같은 화면에서 먼저 확인하세요. Siemens 는 서버가 접근 거부(`BadUserAccessDenied`)로 답하면 상태 `-17` 에 줄 권한(`PlcRead`·`PlcReadDB<번호>`·`SinuReadAll`)을 싣습니다. 840D sl 벤치에서 PLC 읽기 권한이 없는 계정에게는 있는 주소도 없는 주소로 답했으므로, 서버가 없는 주소로 답할 때는 디메시가 연결할 때 읽은 그 계정의 권한으로 가립니다. PLC 읽기 권한이 없으면 같은 상태 `-17` 과 줄 권한을, 권한을 알 수 없으면 상태 `-18` 에 권한이 없어도 같은 모양으로 보인다는 안내를, 권한이 있으면 상태 `-18`(그 장비에 없는 주소)을 냅니다. 그 기종이 못 쓰는 `plcType` 도 상태 `-18` 입니다. 규약 밖 값(`0`~`9` 이외)도 같은 상태 `-18` 이며, 두 경우 모두 대응은 같습니다(다른 `plcType` 지정). 상태 `-20` 이 아닌 이유는 **주소 자체는 그 기종에서 정상 동작**하기 때문입니다. 상태 `-20` 은 "이 주소를 이 기종에서 못 쓴다"는 뜻으로 남겨 둡니다. 에러 문자열에 허용 값이 함께 실려 옵니다. Heidenhain 은 그 장비에 없는 칸·심볼이 상태 `-18` 이고, `access_password` 가 없거나 거절되면 그 연결에서는 주소 자체를 쓸 수 없어 상태 `-20` 입니다.

## /machine/plcAddress/plcType/plcValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
filter_codes: {"plcType": [{"value": 0, "name": "auto", "protocols": ["nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": 1, "name": "bit", "protocols": ["nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]}, {"value": 2, "name": "uint8"}, {"value": 3, "name": "int16"}, {"value": 4, "name": "int32"}, {"value": 5, "name": "float32", "protocols": ["nc_focas2_fanuc", "nc_opcua_siemens"]}, {"value": 6, "name": "float64", "protocols": ["nc_focas2_fanuc", "nc_dnc_heidenhain"]}, {"value": 7, "name": "int8"}, {"value": 8, "name": "uint16"}, {"value": 9, "name": "uint32"}]}
```

**PMC/PLC 메모리의 원소 블록**을 배열로 읽고/씁니다. 필터·주소 형식·`plcType` 규칙은 위 `plcValue`(단일)와 동일하고, **여러 원소**를 다룬다는 점만 다릅니다. 반환 타입은 `floatArray`, 쓰기 `value` 는 숫자 배열 `[1, 2, ...]` 입니다. 단일 원소도 `[42]` 처럼 배열로 적어야 합니다. `plcValue` 와 마찬가지로, 쓴 값이 그대로 남지 않는 구성이 있는 경우가 있으니 쓴 뒤에 읽어 확인하세요.

- **Fanuc**: 범위의 바이트 수가 `plcType` 크기의 **배수**여야 하고, 원소 수 = 바이트 수 ÷ 타입 크기 (예: `D100~D107` + word = 4개 → `[v1,v2,v3,v4]`)
- **Siemens**: 다원소 첨자 허용. `[N]` 은 **개수**입니다. `DB10.DBB56[4]` 는 오프셋 56 부터 **연속 4개**를 배열로 돌려줍니다. 서버가 주는 원소들이 그대로 배열이 됩니다
- **Siemens**: 첨자를 생략하거나 `[1]` 을 줘도 **결과는 배열**입니다 (`[131.0]`). 이 주소의 반환은 `floatArray` 로 고정이라 원소가 하나여도 흔들리지 않습니다. 개수가 가변이거나 미리 모를 때 이 주소를 쓰면 파싱 코드가 분기할 필요가 없습니다
- **Mitsubishi**: `[N]` 이 개수입니다 (`R100[4]` → 4개 배열). 한 번에 읽을 수 있는 최대 점 수가 타입마다 다릅니다: bit·byte `1280`, word `640`, dword `320`. 넘기면 상태 `-18`
- **Mitsubishi**: byte(`plcType` `2`·`7`)는 **한 점만 쓸 수 없습니다** (상태 `-18`). 이 기종의 단일 디바이스 쓰기 호출에 byte 타입이 없고, 블록 쓰기는 2점 이상이라 옆 디바이스까지 함께 바뀌기 때문입니다. word(`3`·`8`)로 쓰거나 이 목록형 주소로 2점 이상을 지정하세요. **읽기는 한 점도 됩니다**
- **Heidenhain**: 메모리 표기의 `[N]` 이 개수입니다 (`W100[4]` → `W100`·`W102`·`W104`·`W106` 4개 배열, 칸의 폭만큼 건너뜀). 심볼 이름은 언제나 원소 하나짜리 배열입니다. 쓰기는 칸마다 따로 보내지만 값과 칸을 모두 먼저 확인하므로, 범위 밖 값이나 없는 칸이 하나라도 있으면 아무것도 쓰지 않습니다
- **Heidenhain**: 문자열인 자리가 있으면 상태 `-25` 로 답합니다. 그런 자리는 `/machine/plcAddress/plcText` 로 읽으세요. 그 주소는 원소 하나씩 읽으므로 여러 자리는 `plcAddress` 를 콤마로 나열하세요. 다만 `plcAddress` 를 콤마로 나열한 이 주소의 요청에서는 한 자리라도 실패하면 요청 전체가 상태 `-15` 입니다 (이 응답은 에러 문구에만 남습니다)
- 쓰기는 **원소 수가 대상 범위/노드의 원소 수와 정확히 일치**해야 합니다

**에러 코드**: 그 장비에 **실재하지 않는 주소**도 상태 `-18`(필터 값 오류)입니다. 어느 블록·바이트가 있는지는 그 장비 래더 구성에 달렸으므로, 조작반의 같은 화면에서 먼저 확인하세요. Siemens 는 서버가 접근 거부(`BadUserAccessDenied`)로 답하면 상태 `-17` 에 줄 권한(`PlcRead`·`PlcReadDB<번호>`·`SinuReadAll`)을 싣습니다. 840D sl 벤치에서 PLC 읽기 권한이 없는 계정에게는 있는 주소도 없는 주소로 답했으므로, 서버가 없는 주소로 답할 때는 디메시가 연결할 때 읽은 그 계정의 권한으로 가립니다. PLC 읽기 권한이 없으면 같은 상태 `-17` 과 줄 권한을, 권한을 알 수 없으면 상태 `-18` 에 권한이 없어도 같은 모양으로 보인다는 안내를, 권한이 있으면 상태 `-18`(그 장비에 없는 주소)을 냅니다. 그 기종이 못 쓰는 `plcType` 도 상태 `-18` 입니다. 규약 밖 값(`0`~`9` 이외)도 같은 상태 `-18` 이며, 두 경우 모두 대응은 같습니다(다른 `plcType` 지정). 상태 `-20` 이 아닌 이유는 **주소 자체는 그 기종에서 정상 동작**하기 때문입니다. 상태 `-20` 은 "이 주소를 이 기종에서 못 쓴다"는 뜻으로 남겨 둡니다. 에러 문자열에 허용 값이 함께 실려 옵니다. Heidenhain 은 그 장비에 없는 칸·심볼이 상태 `-18` 이고, `access_password` 가 없거나 거절되면 그 연결에서는 주소 자체를 쓸 수 없어 상태 `-20` 입니다.

길이는 주소의 `[N]` 이 정하므로 `[]` 은 나오지 않습니다.

## /machine/plcAddress/plcText
```yaml
value_type: "string"
null_able: false
required_filters: ["plcAddress"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

**PLC 데이터 중 문자열인 자리 하나**를 읽습니다 (Siemens OPC-UA `/Plc/` 노드, HEIDENHAIN DNC PLC 데이터). 반환 타입은 `string` 이고 쓰기는 없습니다. 필터는 `plcAddress` 하나이며 표기는 `/machine/plcAddress/plcType/plcValue` 와 같습니다. 문자열은 해석할 형이 하나뿐이라 `plcType` 을 쓰지 않습니다.

- **숫자인 자리를 읽으면** 상태 `-25` 로 답합니다. 그 자리는 `/machine/plcAddress/plcType/plcValue` 가 읽습니다. 그 주소는 `plcType` 도 필요하니 `plcType=0`(auto)을 붙여 같은 `plcAddress` 로 읽으세요
- 여러 자리는 `plcAddress` 를 콤마로 나열합니다 (결과는 문자열 배열)
- 내용이 없는 자리는 빈 문자열 `""` 입니다
- **Heidenhain**: `S` 메모리와 문자열 심볼을 읽습니다. 자리 하나가 문자열 하나라 개수 첨자(`S0[2]`)는 상태 `-18` 입니다. 저희 테스트 환경(TNC7 프로그래밍 스테이션)에서 `S0` 은 `""`, 기본 PLC 프로그램의 시각 심볼은 `"01:01:54"` 로 답했습니다. 심볼 이름은 그 장비의 PLC 프로그램이 정합니다. 읽기에는 연결 설정의 `access_password` 가 필요하며, 없거나 제어기가 거절하면 상태 `-20` 입니다
- **Siemens**: 조작반 `NC/PLC variables` 화면에서 **`A` 형식으로 보이는 것과 같은 글자**를 돌려줍니다. 바이트 주소(`DBB`·`MB`·`IB`·`QB`)부터 개수 첨자 `[N]` 만큼의 바이트를 글자로 잇고(첨자가 없으면 1바이트), 화면처럼 `0` 바이트에서 멈추며, 128 이상의 바이트는 화면처럼 라틴 문자(`Ä`·`ß` 등)로 읽습니다. 화면에서 `MB302[7]` 을 `A` 로 보았을 때 `HELLO` 라면 `plcAddress=MB302[7]` 도 `"HELLO"` 입니다 (840D sl 벤치에서 조작반과 나란히 확인: `HELLO`, 가운데에 `0` 이 든 `HE`, `AÄßB`). PLC 프로그램의 문자열 변수(STRING)는 앞의 두 바이트가 최대 길이와 실제 길이라, 글자는 변수 주소에서 2바이트 뒤부터이고 개수는 최대 길이만큼 주면 됩니다. 화면과 같이 실제 길이 바이트는 보지 않으므로, PLC 가 문자열을 줄이면서 뒤를 비우지 않으면 남은 글자까지 보일 수 있습니다. 워드·더블워드·비트 주소는 글자가 아니라서 상태 `-25` 로 `plcValue` 를 가리킵니다. 접근 권한과 없는 주소에 대한 응답은 `plcValue` 와 같습니다
- **Fanuc·Mitsubishi** 어댑터에는 이 주소가 없습니다 (상태 `-20`). 두 기종의 PLC 주소는 `plcValue` 로 숫자를 읽습니다

**에러 코드**: 그 장비에 없는 자리는 상태 `-18` 입니다.

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
- 소수(real) 파라미터는 장비의 소수 자릿수가 적용된 실수로 오가며, 쓰기도 같은 자릿수로 저장됩니다. Mitsubishi 의 길이 파라미터는 그 자릿수를 설정 단위 `#1003` 이 정하고, 더 잘게 쓴 자리는 제어기가 반올림해 저장한 뒤 상태 `0` 을 돌려줍니다 (테스트 환경에서 `#8205` 로 확인). 무엇이 들어갔는지는 다시 읽어 확인하세요.
- **Mitsubishi 에는 수치가 아닌 파라미터가 있습니다** (예: 축 이름 `#1013` 이 `X`). 이 주소는 `float` 이라 표현할 수 없어 상태 `-18` 로 거절하며, 실제로 읽힌 문자열을 에러에 실어 줍니다. `index` 는 축별 파라미터면 축 번호, 아니면 `1` 뿐입니다.
- Fanuc 의 정수 파라미터에 범위를 넘는 값을 쓰면 상태 `-16` 으로 거절합니다 (허용 범위를 에러에 함께 실어). Mitsubishi 는 제어기가 값을 거절하면 상태 `-16` 이고 허용 범위는 싣지 않으므로, 파라미터 설명서의 설정 범위를 확인하세요.
- **쓰기 주의**: 파라미터는 기계 거동을 바꿉니다. Fanuc 은 장비가 파라미터 쓰기를 막아둔 상태면, Mitsubishi 는 그 계통이 자동운전 중(일시정지 포함)이거나 데이터 보호 키 2(PLC 신호 `*KEY2`, `Y709`. 사용자 파라미터를 보호합니다)가 꺼져 있으면 상태 `-22`(기계 상태)로 거절되고(Mitsubishi 는 리셋하거나 키를 켠 뒤 다시 쓰세요. 키는 디메시가 거절된 뒤 그 신호를 읽어 가립니다), 일부 파라미터는 변경 후 전원 재투입을 요구합니다. Fanuc 라이브러리에 파라미터 쓰기 함수(`cnc_wrparam`)가 없으면 쓰기는 상태 `-20` 입니다 (들어 있는 함수는 Fanuc 라이브러리 제품에 따라 다릅니다).

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

**Fanuc·Siemens 는 상태 `-20` 입니다.** 이 섹션·서브섹션 번호 체계는 Mitsubishi 소유입니다.

## /machine/ncMemorySizeTotal
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

NC 메모리 전체 용량입니다. 반환 `int` + `unit:"bytes"`, 읽기 전용.

가리키는 것은 **가공 프로그램 메모리**입니다. Fanuc 은 `cnc_rdpdf_inf` 의 드라이브 용량(필드 이름과 달리 바이트 단위), Siemens 는 NC 의 **패시브 파일 시스템 사용자 영역**(메인/서브프로그램·워크피스·GUD 정의 파일이 사는 곳, 확장 기능 매뉴얼 S7)인 `/Nck/State/usedMemDramUPassF`·`freeMemDramUPassF`, Mitsubishi 는 `GetInformation` 의 문자 수(조작반 편집 화면의 `기억용량`·`나머지` 와 같은 값)입니다. 셋 중 하나는 나머지 둘로 계산한 값입니다 (Fanuc 은 잔여 = 전체 - 사용, Siemens·Mitsubishi 는 전체 = 사용 + 잔여, Heidenhain 은 사용 = 전체 - 잔여). 그래서 한 요청에서 함께 읽은 셋은 `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree` 로 맞아떨어집니다. Siemens 에서 값을 읽지 못하면 `0` 이 아니라 상태 `-17` 입니다 (전체는 사용과 잔여가 둘 다 있어야 계산합니다).

**Mitsubishi 는 메인 루트(`//PRG`, 조작반 `Memory`)의 값입니다.** M800V/M80V 의 두 번째 프로그램 메모리(`//PRG2`, 조작반 `Memory2`)는 용량을 따로 세며 이 값에 들어가지 않습니다 (테스트 환경에서 확인).

**Heidenhain 은 `//TNC`(조작반 `TNC:` 드라이브)의 값입니다.** HEIDENHAIN DNC 의 `GetDiskSpace` 가 주는 전체·여유 바이트이고, 사용량은 그 둘의 차이입니다. 저희 테스트 환경에서 조작반 파일 관리자가 보여 주는 사용량은 `ncMemorySizeUsed` 보다 전체 용량의 약 5% 만큼 컸습니다 (파일을 더해 두 번 대조해 같은 차이였고, 전체 용량은 같았습니다). 조작반의 사용량 표시는 제어기를 다시 켤 때만 새로 계산되어, 그 사이에 파일을 더하거나 지워도 바뀌지 않았습니다.

크기는 SDK 전체에서 **바이트**로 통일되어 있습니다. `entry`/`entryList` 의 `sizeBytes` 와 같은 단위이므로 "이 파일이 남은 공간에 들어가나" 같은 계산에 변환이 필요 없습니다. 장비가 더 거친 단위로만 알려주는 경우에도 이 주소는 바이트로 환산해 내보내며, 그때 값은 그 단위의 배수가 됩니다.

## /machine/ncMemorySizeUsed
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

NC 메모리 사용량입니다. 반환 `int` + `unit:"bytes"`, 읽기 전용.

가리키는 것은 **가공 프로그램 메모리**입니다. Fanuc 은 `cnc_rdpdf_inf` 의 드라이브 용량(필드 이름과 달리 바이트 단위), Siemens 는 NC 의 **패시브 파일 시스템 사용자 영역**(메인/서브프로그램·워크피스·GUD 정의 파일이 사는 곳, 확장 기능 매뉴얼 S7)인 `/Nck/State/usedMemDramUPassF`·`freeMemDramUPassF`, Mitsubishi 는 `GetInformation` 의 문자 수(조작반 편집 화면의 `기억용량`·`나머지` 와 같은 값)입니다. 셋 중 하나는 나머지 둘로 계산한 값입니다 (Fanuc 은 잔여 = 전체 - 사용, Siemens·Mitsubishi 는 전체 = 사용 + 잔여, Heidenhain 은 사용 = 전체 - 잔여). 그래서 한 요청에서 함께 읽은 셋은 `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree` 로 맞아떨어집니다. Siemens 에서 값을 읽지 못하면 `0` 이 아니라 상태 `-17` 입니다.

**Mitsubishi 는 메인 루트(`//PRG`, 조작반 `Memory`)의 값입니다.** M800V/M80V 의 두 번째 프로그램 메모리(`//PRG2`, 조작반 `Memory2`)는 용량을 따로 세며 이 값에 들어가지 않습니다 (테스트 환경에서 확인).

**Heidenhain 은 `//TNC`(조작반 `TNC:` 드라이브)의 값입니다.** HEIDENHAIN DNC 의 `GetDiskSpace` 가 주는 전체·여유 바이트이고, 사용량은 그 둘의 차이입니다. 저희 테스트 환경에서 조작반 파일 관리자가 보여 주는 사용량은 `ncMemorySizeUsed` 보다 전체 용량의 약 5% 만큼 컸습니다 (파일을 더해 두 번 대조해 같은 차이였고, 전체 용량은 같았습니다). 조작반의 사용량 표시는 제어기를 다시 켤 때만 새로 계산되어, 그 사이에 파일을 더하거나 지워도 바뀌지 않았습니다.

크기는 SDK 전체에서 **바이트**로 통일되어 있습니다. `entry`/`entryList` 의 `sizeBytes` 와 같은 단위이므로 "이 파일이 남은 공간에 들어가나" 같은 계산에 변환이 필요 없습니다. 장비가 더 거친 단위로만 알려주는 경우에도 이 주소는 바이트로 환산해 내보내며, 그때 값은 그 단위의 배수가 됩니다.

## /machine/ncMemorySizeFree
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

NC 메모리 잔여 용량입니다. 반환 `int` + `unit:"bytes"`, 읽기 전용.

가리키는 것은 **가공 프로그램 메모리**입니다. Fanuc 은 `cnc_rdpdf_inf` 의 드라이브 용량(필드 이름과 달리 바이트 단위), Siemens 는 NC 의 **패시브 파일 시스템 사용자 영역**(메인/서브프로그램·워크피스·GUD 정의 파일이 사는 곳, 확장 기능 매뉴얼 S7)인 `/Nck/State/usedMemDramUPassF`·`freeMemDramUPassF`, Mitsubishi 는 `GetInformation` 의 문자 수(조작반 편집 화면의 `기억용량`·`나머지` 와 같은 값)입니다. 셋 중 하나는 나머지 둘로 계산한 값입니다 (Fanuc 은 잔여 = 전체 - 사용, Siemens·Mitsubishi 는 전체 = 사용 + 잔여, Heidenhain 은 사용 = 전체 - 잔여). 그래서 한 요청에서 함께 읽은 셋은 `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree` 로 맞아떨어집니다. Siemens 에서 값을 읽지 못하면 `0` 이 아니라 상태 `-17` 입니다.

**Mitsubishi 는 메인 루트(`//PRG`, 조작반 `Memory`)의 값입니다.** M800V/M80V 의 두 번째 프로그램 메모리(`//PRG2`, 조작반 `Memory2`)는 용량을 따로 세며 이 값에 들어가지 않습니다 (테스트 환경에서 확인).

**Heidenhain 은 `//TNC`(조작반 `TNC:` 드라이브)의 값입니다.** HEIDENHAIN DNC 의 `GetDiskSpace` 가 주는 전체·여유 바이트이고, 사용량은 그 둘의 차이입니다. 저희 테스트 환경에서 조작반 파일 관리자가 보여 주는 사용량은 `ncMemorySizeUsed` 보다 전체 용량의 약 5% 만큼 컸습니다 (파일을 더해 두 번 대조해 같은 차이였고, 전체 용량은 같았습니다). 조작반의 사용량 표시는 제어기를 다시 켤 때만 새로 계산되어, 그 사이에 파일을 더하거나 지워도 바뀌지 않았습니다.

크기는 SDK 전체에서 **바이트**로 통일되어 있습니다. `entry`/`entryList` 의 `sizeBytes` 와 같은 단위이므로 "이 파일이 남은 공간에 들어가나" 같은 계산에 변환이 필요 없습니다. 장비가 더 거친 단위로만 알려주는 경우에도 이 주소는 바이트로 환산해 내보내며, 그때 값은 그 단위의 배수가 됩니다.

**Mitsubishi 는 이 값이 250바이트 눈금으로 움직입니다**. 장비가 잔여를 250문자 단위로만 세기 때문입니다. 단위는 다른 기종과 같은 바이트이고, 값이 250의 배수가 될 뿐입니다.

## /machine/ncMemoryRootPath
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

메인 NC 메모리의 루트 경로입니다. `ncMemoryPath` 필터에 넣을 경로의 시작점. Fanuc 은 보통 `//CNC_MEM`, Siemens 는 `//NC`, Mitsubishi 는 보통 `//PRG`, Heidenhain 은 `//TNC` 입니다. Mitsubishi 는 연결할 때 선택된 프로그램이 두 번째 프로그램 메모리(`//PRG2`)나 SD 카드(`//IC1`)에 있어도 루트를 `//PRG` 로 잡고, 그 둘은 `/machine/ncMemoryExternalRootPathList` 에 나옵니다 (`//PRG2` 는 테스트 환경에서 확인, SD 카드는 아직 확인하지 못했습니다).

**Fanuc 의 `//CNC_MEM` 아래에는** 가공 프로그램이 사는 `USER`(경로별 `PATH1`·`PATH2`… 와 공통 `LIBRARY`)와, 시스템·기계 제조사의 매크로 폴더 `SYSTEM`·`MTB1`·`MTB2` 가 있습니다 (테스트 환경에서 확인). 뒤의 셋에는 디메시가 쓰지 않습니다 (상태 `-18`, 읽기는 됩니다).

**Mitsubishi 의 `//PRG` 는 조작반 `Memory`(NC 메모리)의 프로그램 영역이고, 가공 프로그램은 그 아래 `//PRG/USER` 에 있습니다** (조작반의 `Memory:/Program` 목록). `//PRG` 를 조회하면 폴더가 나옵니다: `USER`(가공 프로그램), `MDI`(MDI 프로그램 `MDI.PRG`), `FIX`(고정 사이클), `MMACRO`(기계 제조사 매크로). M700 계열은 `UMACRO` 도 나옵니다. M800 계열은 목록에서 `UMACRO` 를 뺍니다. 매뉴얼상 M700 의 경로이고, 테스트 환경(M800V)에서는 `USER` 와 같은 파일을 가리켰기 때문입니다 (`//PRG/UMACRO/…` 로 직접 부르면 그대로 열립니다). `FIX`·`MMACRO` 아래에는 디메시가 쓰지 않습니다 (상태 `-18`, 읽기는 됩니다). M800V/M80V 의 두 번째 프로그램 메모리(조작반 `Memory2`)는 `//PRG2` 이며 `ncMemoryExternalRootPathList` 에 나옵니다. 그 아래에는 가공 프로그램 폴더 `USER` 만 있습니다. 그곳의 프로그램이 선택돼 있어도 이 값은 `//PRG` 입니다.

**Heidenhain 의 `//TNC` 는 조작반 파일 관리자의 `TNC:` 드라이브이고, 가공 프로그램은 보통 `//TNC/nc_prog` 아래에 있습니다.** 디메시는 이 드라이브만 다룹니다. 저희 테스트 환경에서 다른 드라이브는 접근이 거절됐고, 그래서 `ncMemoryExternalRootPathList` 는 상태 `-20` 입니다 (TNC7 사용 설명서는 `PLC:` 드라이브를 기계 제작사용 사용자, `SYS:` 를 서비스용 사용자의 것으로 적습니다). 표·설정이 든 `//TNC/table`·`//TNC/system`·`//TNC/config` 아래에는 디메시가 쓰지 않습니다 (상태 `-18`, 읽기는 됩니다. `fileContent` 참조). 이름은 대소문자를 가리지 않고 찾습니다.

## /machine/ncMemoryExternalRootPathList
```yaml
value_type: "stringArray"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

메인 NC 메모리 루트 외의 **저장소** 목록입니다 (예: 데이터 서버, 메모리 카드). 반환 타입 `stringArray`.

- 각 항목은 root 처럼 앞에 `//` 가 붙습니다 (뒤 슬래시는 없음). Fanuc `//DATA_SV`·`//MEM_CARD`, Siemens `//Local drive`, Mitsubishi `//IC1` (NC 쪽 SD 카드, 조작반의 `DS`. SD 카드로는 아직 확인하지 못했습니다). 하나도 없으면 빈 배열입니다
- **Mitsubishi M800V/M80V 는 두 번째 프로그램 메모리 `//PRG2` 도 여기 나옵니다** (조작반의 `Memory2`, 프로그램은 `//PRG2/USER`). NC 안의 메모리지만 메인 루트(`//PRG`)와 프로그램 칸·용량을 따로 세는 저장소라 이 목록에 넣습니다. 모든 계통이 같은 내용을 봅니다 (테스트 환경에서 확인)
- 이름은 **장비 HMI 표기**입니다. Siemens 의 로컬 드라이브는 OPC-UA 내부 이름이 `NCExtend` 이지만 조작반과 같게 `//Local drive` 로 내보냅니다 (옛 표기 `//NCExtend` 로 요청해도 받습니다)
- 메인 루트 자신은 이 목록에서 제외됩니다
- **캐시하지 않음.** 외부 장치는 연결/해제로 바뀔 수 있어 요청마다 새로 조회
- **Mitsubishi 는 `//IC1`·`//PRG2` 를 열어 보고 제어기가 거절하면 목록에서 뺍니다.** 확인하는 중에 통신 오류가 나면 그 저장소를 빼지 않고 요청 전체를 에러로 답합니다
- 필터 없음

## /machine/ncMemoryPath/entry
```yaml
value_type: "object"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

경로의 항목 1개 정보입니다 (`object`). 키 집합은 **기종과 무관하게 항상 같습니다.** 값이 없으면 키가 빠지는 게 아니라 `null` 입니다.

| 키 | 타입 | 없을 때 |
|---|---|---|
| `name` | `string` | 없음 |
| `sizeBytes` | `int` | 폴더, 또는 크기를 못 읽은 경우 `null`. Siemens·Mitsubishi·Heidenhain 은 내용의 바이트 수, **Fanuc 은 할당 크기(500바이트 단위)** 라 22바이트 프로그램도 `500` 입니다 |
| `modifiedAt` | `string` | 폴더, 또는 수정 시각이 없는 기종에서는 `null` (Siemens 는 항상 `null`. Heidenhain 은 아래 참조) |
| `isDir` | `boolean` | 없음 |
| `comment` | `string` | 폴더, 또는 주석이 없는 기종에서는 `null` (Siemens·Heidenhain 은 항상 `null`. 아래 설명 참조) |

**`comment` 는 제어기가 목록에 함께 실어 주는 프로그램 주석**입니다. Fanuc 은 O 번호 줄의 괄호 주석, Mitsubishi 는 프로그램 목록의 코멘트 칸(첫 블록의 괄호 주석)이라 **파일을 열지 않고 목록 한 번으로 받습니다** (시뮬레이터에서 확인). **Siemens 에서는 목록에 주석이 실리지 않아 항상 `null`** 이고, 첫 줄의 `;` 주석은 파일 내용이라 `fileContent` 를 읽어야만 보입니다. 목록만으로 프로그램을 가려내야 한다면 Siemens 에서는 주석 대신 **폴더나 이름**으로 나누세요. 하위 폴더(`directoryExists` 로 생성)와 이름 접두어는 `entryList` 한 번에 보입니다. 후보마다 `fileContent` 를 읽는 방식은 느린 회선에서 파일 수만큼 비용이 듭니다.

경로 끝 `/` 로 폴더를 명시할 수 있고, 없으면 파일 우선 검색입니다. 항목이 없으면(빈 폴더 안의 이름, 없는 폴더 아래의 경로 포함) 상태 `-18` 입니다. 상태 `-18` 은 없을 때만이고, 통신 오류는 그 오류의 상태로 돌아옵니다. 다만 Mitsubishi 에서 폴더 경로가 너무 길 때도 상태 `-18` 이며, 이때 에러 문구는 항목이 없다는 것이 아니라 경로나 이름이 너무 길다고 알립니다 (시뮬레이터에서 확인).

**Heidenhain 의 `modifiedAt` 은 HEIDENHAIN DNC 가 주는 시각 그대로이고 `Z` 를 붙이지 않습니다.** 이 값은 프로그램이 도는 PC 의 시간대로 본 시각이고, 조작반 파일 관리자는 제어기에 설정된 시간대로 보여 줍니다. 그래서 두 시간대가 같으면 조작반과 같은 시각이고, 다르면 시간대 차이만큼 다릅니다 (저희 테스트 환경에서 제어기의 시간대를 PC 와 같게 바꾸자 조작반 표시가 목록을 새로 고친 즉시 이 값과 같아졌고, 이 값은 그대로였습니다). Heidenhain 은 이름을 대소문자 없이 찾고 `name` 은 제어기에 저장된 표기로 돌려줍니다. 목록에 주석이 실리지 않아 `comment` 는 항상 `null` 입니다.

## /machine/ncMemoryPath/entryList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

폴더 안의 파일/폴더 목록입니다 (**읽기 전용**, `objectArray`). 각 원소는 `entry` 와 **완전히 같은 객체**입니다. 키 집합·`null` 규약 모두 동일하므로 그쪽 표를 보세요. 폴더 우선, 이름 오름차순으로 정렬됩니다. 폴더의 생성/삭제는 `directoryExists` 를 사용하세요.

없는 폴더를 지정하면 상태 `-18` 입니다.

**Mitsubishi M800 계열은 루트(`//PRG`) 목록에서 `UMACRO` 를 뺍니다.** 그 기종에서 `USER` 와 같은 파일을 가리키는 이름이라 프로그램이 두 번 보이지 않게 한 것입니다. `//PRG/UMACRO/…` 경로는 그대로 쓸 수 있습니다 (`ncMemoryRootPath` 참조).

폴더가 비어 있으면 `[]` 입니다.

**Heidenhain 은 `.`·`..` 만 빼고 제어기의 목록을 그대로 냅니다.** 숨김 속성 항목도 나옵니다 (예: 테스트 환경에서 프로그램을 운전하면 생긴 `*.T.DEP`. 조작반 파일 관리자에도 보이고, 기계 설정의 공구 사용 파일 생성이 "안 함" 이어도 생겼습니다). 조작반이 `//TNC` 목록에서 보여 주지 않는 `lost+found`·`.nc_index` 는 나옵니다 (테스트 환경에서 확인). `lost+found` 의 목록은 제어기가 거절해 상태 `-17` 입니다.

`comment` 는 Fanuc·Mitsubishi 에서 목록 한 번으로 오고 Siemens·Heidenhain 은 항상 `null` 입니다. 이유와 대안(폴더·이름으로 나누기)은 `entry` 의 `comment` 설명에 있습니다.

## /machine/ncMemoryPath/entryName
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

`ncMemoryPath` 가 가리키는 **항목의 이름**입니다. **장비에 있는 그 항목의 이름**을 돌려주고(read), **이름 변경**(write)을 합니다. 읽기는 파일·폴더 어느 쪽이든 답하며, 그 경로에 아무것도 없으면 상태 `-18` 로 거절합니다 (`entry` 와 같은 판단입니다). 상태 `-18` 은 없을 때만이고, 통신 오류는 그 오류의 상태로 돌아옵니다. 돌려주는 것은 요청에 적은 문자열이 아니라 **장비가 가진 이름**이라, 표기가 다르면 장비 쪽 표기가 나옵니다. 쓰기는 `{"value": "새이름"}` 이며 경로 구분자는 넣을 수 없습니다 (파일/폴더 공통). **루트의 이름은 바꿀 수 없습니다**: `ncMemoryPath` 가 `//CNC_MEM`·`//NC`·`//PRG` 나 외부 드라이브처럼 한 토막이면 장비에 보내지 않고 상태 `-18`(필터 값 오류)로 거절합니다. `\\NC` 처럼 역슬래시로 적은 루트도 같습니다. Siemens 의 `//NC/Part programs`·`//NC/Subprograms`·`//NC/Workpieces` 도 이름을 바꾸지 않습니다 (`directoryExists` 의 삭제 거절과 같은 규칙). `entry`/`entryList` 와 같은 "항목" 을 가리킵니다. **Siemens 는 운전 중(채널이 Reset 이 아닐 때) 선택된 메인 프로그램의 이름을 바꾸지 못합니다.** 그 경우 상태 `-22`(기계 상태)로 돌아오니 채널을 리셋하거나 프로그램이 끝난 뒤 다시 시도하세요. 채널이 전부 Reset 이어도 제어기가 지금 상태에서는 이름 변경을 받지 않는다고 답하면(`BadInvalidState`) 역시 상태 `-22` 이고, 에러 문구는 편집기나 다른 클라이언트가 파일을 쥔 경우로 안내합니다. **선독이 열어 둔 서브프로그램은 삭제와 갈립니다**: 테스트 벤치에서 운전 중에 같은 서브프로그램의 삭제(`fileExists`)는 상태 `-22` 였는데 이름 변경은 받아들여졌습니다. 실행 중인 프로그램이 부를 서브프로그램의 이름은 채널이 Reset 일 때 바꾸세요. **Fanuc 은 선택된 메인 프로그램(운전 중이 아니어도), Mitsubishi 는 자동운전 중인 계통의 메인 프로그램의 이름을 바꾸지 못하며** 상태 `-22`(기계 상태)로 돌아옵니다. Fanuc 은 보호(파라미터 `3202#0`/`#4`, 조작반의 편집 금지 속성)가 걸린 항목, 그리고 보호 번호대로 바꾸는 이름 변경도 상태 `-22` 입니다. Mitsubishi 는 사용자 레벨 데이터 보호(파라미터 `#1391`)가 프로그램 편집을 막고 있을 때 상태 `-22` 입니다 (시뮬레이터에서 확인, `fileContent` 참조). **이름을 바꿀 항목이 없으면** 네 기종 모두 상태 `-18` 입니다. Fanuc 데이터 서버(`//DATA_SV`)도 같고, 데이터 서버에서 메인으로 선택된 파일도 상태 `-22` 입니다 (31i-B 실장비에서 확인). 새 이름이 빈 문자열이면 네 기종 모두 상태 `-16`(쓰기 값 오류)입니다. **새 이름이 지금 이름과 같으면** 장비에 보내지 않고 그 항목이 있는지만 확인합니다 (있으면 성공, 없으면 상태 `-18`). 파일 시스템의 이름 바꾸기와 같은 동작이고, 대소문자까지 같을 때만입니다. Mitsubishi 의 프로그램 메모리(`//PRG`·`//PRG2`)는 이름을 대문자로 저장하므로 대소문자만 다른 이름도 같은 이름으로 봅니다. **새 이름의 항목이 이미 있으면** 네 기종 모두 상태 `-21`(이미 있음)로 거절하고 아무것도 바꾸지 않습니다 (그 항목을 대신 지우지 않습니다). **시스템·기계 제조사 영역(Fanuc `//CNC_MEM/SYSTEM`·`MTB1`·`MTB2`, Mitsubishi `//PRG/FIX`·`//PRG/MMACRO`)과 Heidenhain 의 `//TNC/table`·`//TNC/system`·`//TNC/config` 는 이름을 바꾸지 않습니다** (장비에 보내지 않고 상태 `-18`, `fileContent` 참조). 다만 새 이름이 **대소문자까지** 지금 이름과 같으면 위의 같은 이름 확인이 먼저라 상태 `0` 입니다 (대소문자만 다른 이름은 제조사 영역 거절이 먼저라 상태 `-18` 입니다). 장비에는 아무것도 보내지 않으므로 바뀌는 것은 없습니다 (Siemens 의 세 고정 폴더도 같습니다). **Mitsubishi 의 편집 잠금(`#8105`·`#1121`)은 이름 변경을 막지 않습니다**: 잠긴 번호의 프로그램 이름을 바꾸는 것도, 다른 프로그램을 잠긴 번호로 바꾸는 것도 받아들여졌습니다 (테스트 환경에서 확인, `fileContent` 참조). **Heidenhain 에서 대소문자만 다른 이름으로 바꾸는 쓰기는 상태 `-21` 입니다.** 제어기가 대소문자만 다른 이름을 같은 이름으로 보므로 디메시가 장비에 보내지 않고 답합니다. 조작반 파일 관리자에서 쓰기 방지가 걸린 항목과 운전 중인 프로그램(메인·서브)은 상태 `-22`(기계 상태)이고, 사유에 어느 쪽인지 실립니다 (테스트 환경에서 확인).

**Fanuc 데이터 서버(`//DATA_SV`)에서는 이 쓰기가 데이터 서버의 현재 폴더를 옮깁니다.** 데이터 서버의 폴더·파일을 다루는 Fanuc 함수는 현재 폴더 안의 이름만 받아서, 디메시가 먼저 대상의 부모 폴더로 옮긴 뒤 부르고 그 폴더에 둔 채 끝냅니다. 이 현재 폴더는 장비에 하나이고 조작반 데이터 서버 화면과 같은 것이라, 조작반은 그 화면에 다시 들어갈 때 바뀐 폴더를 보입니다 (31i-B 실장비에서 확인). 그래서 **한 장비의 데이터 서버 쓰기는 한 곳에서만 하세요.** 다른 프로그램이나 조작반이 같은 때 폴더를 옮기면, 이름으로 부르는 호출이 다른 폴더의 같은 이름에 닿을 수 있습니다. 이 연결로 데이터 서버의 폴더·파일 조작을 할 수 없는 장비에서는 상태 `-18`(필터 값 오류)이고 사유에 그 뜻이 실립니다 (저희 테스트 벤치 하나가 그랬고, 그 장비의 조작반에서는 같은 조작이 되었습니다).

## /machine/ncMemoryPath/directoryExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

경로에 **폴더**가 존재하는지 확인하고(read), 상태를 선언적으로 씁니다(write):

- read → 폴더가 있으면 `true` (같은 이름의 파일만 있으면 `false`). `false` 는 **없을 때만**입니다. 통신 오류나 제어기의 다른 거절은 `false` 가 아니라 그 오류의 상태로 돌아옵니다. Mitsubishi 에서 폴더 경로가 너무 길면 `false` 가 아니라 상태 `-18` 이고, 에러 문구가 경로나 이름이 너무 길다고 알립니다 (시뮬레이터에서 확인). Fanuc 에서 `//CNC_MEM` 같은 드라이브 자신을 물으면 `true` 입니다 (Heidenhain 의 `//TNC` 도 같습니다)
- write `{"value": true}` → 폴더 생성 (이미 있으면 상태 `-21`(이미 존재). Fanuc 데이터 서버 `//DATA_SV/` 도 같습니다. 거절 사유만으로는 이미 있는지 가를 수 없어, 거절되면 디메시가 부모 폴더의 목록을 한 번 읽어 가려냅니다). Fanuc 에서 등록할 수 있는 개수가 다 차면 상태 `-23`(자리 없음)이고, 폴더와 프로그램이 같은 개수에 들어갑니다 (테스트 벤치에서 확인)
- write `{"value": false}` → 폴더 삭제. **지워지는 범위가 기종에 따라 다릅니다**: Fanuc·Heidenhain 은 빈 폴더만 지우고, 내용이 있으면 상태 `-17`(핸들러 에러)에 사유를 실어 거절합니다. **Siemens 는 폴더 안의 파일과 하위 폴더까지 함께 지웁니다**(저희 벤치에서 확인). 지우기 전에 `entryList` 로 내용을 확인하세요. 운전 중인 채널이 그 폴더 안의 프로그램을 쥐고 있으면 Siemens 는 상태 `-22`(기계 상태)로 거절합니다. **없는 폴더면 상태 `-18`(필터 값 오류)** 입니다 (네 기종 공통. Fanuc 데이터 서버 `//DATA_SV/` 는 벤더의 거절이 그대로 돌아옵니다). **루트는 지울 수 없습니다**: `ncMemoryPath` 가 `//CNC_MEM`·`//NC`·`//PRG` 나 외부 드라이브처럼 한 토막이면 장비에 보내지 않고 상태 `-18`(필터 값 오류)로 거절합니다. `\` 는 `/` 로 보므로 `\\NC` 도 루트로 거절됩니다. 경로가 `//` 로 시작하지 않거나 `.`·`..`·빈 토막이 들어 있으면 그보다 먼저 표기 오류로 상태 `-18` 입니다. **Siemens 의 `//NC/Part programs`·`//NC/Subprograms`·`//NC/Workpieces` 도 지우지 않습니다** (대소문자·끝의 `.DIR` 와 관계없이 장비에 보내지 않고 상태 `-18`). 그 안의 항목을 지정하세요
- **Fanuc 에서 조작반의 편집 금지 속성이 걸린 폴더**는 지우거나 그 안에 폴더를 만들 수 없고, 상태 `-22`(기계 상태)로 돌아옵니다
- **Fanuc 의 `//CNC_MEM/SYSTEM`·`MTB1`·`MTB2` 와 그 아래에는 폴더를 만들거나 지우지 않습니다** (장비에 보내지 않고 상태 `-18`, `fileContent` 참조)
- **Mitsubishi 의 NC 메모리 드라이브에서는 폴더 생성·삭제가 상태 `-20` 입니다.** 그 드라이브는 디렉터리 구성이 고정이라 제어기가 폴더 생성·삭제를 받지 않습니다. SD 카드(`//IC1`)에서는 이미 있는 폴더를 만들면 상태 `-21`, 자리가 없으면 상태 `-23`, 내용이 있는 폴더를 지우면 상태 `-17`(사유가 실립니다), 카드가 쓰기 금지면 상태 `-22` 입니다 (SD 카드 환경에서는 아직 확인하지 못했습니다). `//PRG/FIX`·`//PRG/MMACRO` 와 그 아래는 장비에 보내기 전에 상태 `-18` 로 거절합니다 (`fileContent` 참조). 읽기는 정상입니다
- **Heidenhain**: 이미 있는 폴더를 만들면 상태 `-21`, 만들 자리의 부모 폴더가 없으면 상태 `-18` 입니다. 조작반 파일 관리자에서 쓰기 방지가 걸린 폴더는 지우거나 그 안에 폴더를 만들 수 없고, 상태 `-22`(기계 상태)에 사유가 실립니다 (테스트 환경에서 확인). 프로그램을 운전한 폴더에는 숨김 파일 `*.T.DEP` 가 남아, 조작반에 보이는 파일을 다 지워도 폴더 삭제가 상태 `-17`(비어 있지 않음)일 수 있습니다 (`entryList` 로 확인하세요). 디메시로 지운 폴더는 조작반의 휴지통을 거치지 않아 되살릴 수 없습니다 (테스트 환경에서 파일로 확인. 조작반 파일 관리자로 지운 것은 휴지통으로 간다고 TNC7 사용 설명서 'Basic information' 이 적습니다). `//TNC/table`·`//TNC/system`·`//TNC/config` 와 그 아래에는 폴더를 만들거나 지우지 않습니다 (장비에 보내지 않고 상태 `-18`, `fileContent` 참조)

경로 끝 `/` 는 무시됩니다. 파일은 `fileExists` 사용.

**Fanuc 데이터 서버(`//DATA_SV`)에서는 이 쓰기가 데이터 서버의 현재 폴더를 옮깁니다.** 데이터 서버의 폴더·파일을 다루는 Fanuc 함수는 현재 폴더 안의 이름만 받아서, 디메시가 먼저 대상의 부모 폴더로 옮긴 뒤 부르고 그 폴더에 둔 채 끝냅니다. 이 현재 폴더는 장비에 하나이고 조작반 데이터 서버 화면과 같은 것이라, 조작반은 그 화면에 다시 들어갈 때 바뀐 폴더를 보입니다 (31i-B 실장비에서 확인). 그래서 **한 장비의 데이터 서버 쓰기는 한 곳에서만 하세요.** 다른 프로그램이나 조작반이 같은 때 폴더를 옮기면, 이름으로 부르는 호출이 다른 폴더의 같은 이름에 닿을 수 있습니다. 이 연결로 데이터 서버의 폴더·파일 조작을 할 수 없는 장비에서는 상태 `-18`(필터 값 오류)이고 사유에 그 뜻이 실립니다 (저희 테스트 벤치 하나가 그랬고, 그 장비의 조작반에서는 같은 조작이 되었습니다).

## /machine/ncMemoryPath/fileExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

경로에 **파일**이 존재하는지 확인하고(read), 상태를 선언적으로 씁니다(write):

- read → 파일이 있으면 `true` (같은 이름의 폴더만 있으면 `false`). `false` 는 **없을 때만**입니다. 통신 오류나 제어기의 다른 거절은 `false` 가 아니라 그 오류의 상태로 돌아옵니다. Mitsubishi 에서 폴더 경로가 너무 길면 `false` 가 아니라 상태 `-18` 이고, 에러 문구가 경로나 이름이 너무 길다고 알립니다 (시뮬레이터에서 확인)
- write `{"value": false}` → 파일 삭제. **없는 파일이면 상태 `-18`(필터 값 오류)** 입니다 (네 기종 공통). 경로 오타가 성공으로 숨지 않게 하려는 것이라, 이미 지워졌을 수 있는 파일을 지울 때는 이 코드를 "이미 없음" 으로 처리하세요. Fanuc 데이터 서버 `//DATA_SV/` 에서는 이 구분을 하지 않아 벤더의 거절이 그대로 돌아옵니다
- write `{"value": true}` → 상태 `-16`(쓰기 값 오류)으로 거절합니다: 빈 파일 생성은 지원하지 않습니다. 파일 생성은 내용과 함께 `fileContent` 쓰기로 하세요
- **시스템·기계 제조사 영역은 지우지 않습니다** (Fanuc `//CNC_MEM/SYSTEM`·`MTB1`·`MTB2`, Mitsubishi `//PRG/FIX`·`//PRG/MMACRO`, Heidenhain `//TNC/table`·`//TNC/system`·`//TNC/config`). 장비에 보내지 않고 상태 `-18` 로 거절합니다 (`fileContent` 참조)
- **Mitsubishi 의 편집 잠금(`#8105`·`#1121`)은 삭제를 막지 않습니다.** 잠긴 번호의 프로그램도 지워집니다 (테스트 환경에서 확인, `fileContent` 참조)
- **Mitsubishi 의 사용자 레벨 데이터 보호가 프로그램 편집을 막고 있으면 삭제는 상태 `-22`(기계 상태)입니다** (파라미터 `#1391`, 시뮬레이터에서 확인, `fileContent` 참조)
- **Mitsubishi 의 쓰기 금지된 SD 카드(`//IC1`)에서는 삭제가 상태 `-22`(기계 상태)입니다** (쓰기 금지된 SD 카드로는 아직 확인하지 못했습니다)

경로 끝 `/` 는 무시됩니다 (종류는 주소가 확정). 폴더는 `directoryExists` 사용.

**Siemens 는 채널이 Reset 이 아닐 때 그 채널이 쓰고 있는 파일을 지우지 못합니다.** 선택된 메인 프로그램, 실행 중인 서브프로그램, 그리고 **선독이 미리 열어 둔** 서브프로그램은 삭제가 상태 `-22`(기계 상태)로 돌아옵니다. 잠금 창이 "지금 실행 중" 보다 넓어서 같은 프로그램을 돌리는 동안 어느 순간엔 되고 어느 순간엔 안 되는 것처럼 보이지만 무작위가 아닙니다. 메인이 참조만 하고 아직 부르지 않은 서브는 지울 수 있습니다 (테스트 벤치에서 확인). 채널을 리셋하거나 프로그램이 끝난 뒤 다시 지우세요. 채널이 전부 Reset 이어도 제어기가 지금 상태에서는 삭제를 받지 않는다고 답하면(`BadInvalidState`) 역시 상태 `-22` 이고, 에러 문구는 편집기나 다른 클라이언트가 파일을 쥔 경우로 안내합니다.

**Fanuc 은 선택된 메인 프로그램(운전 중이 아니어도), Mitsubishi 는 자동운전 중인 계통의 메인 프로그램을 지우지 못합니다.** 삭제는 상태 `-22`(기계 상태)로 돌아옵니다. Fanuc 은 보호(파라미터 `3202#0`/`#4`, 조작반의 편집 금지 속성)가 걸린 프로그램의 삭제도 상태 `-22` 입니다. Fanuc 은 다른 프로그램을 선택한 뒤, Mitsubishi 는 계통을 리셋하거나 운전이 끝난 뒤 지우세요. 저희 테스트 환경에서 두 기종 모두 실행 중인 서브프로그램은 지워졌습니다.

**Heidenhain 은 운전 중인 프로그램(메인·서브)과, 운전이 멈춘 채 끝나지 않은 프로그램을 지우지 못합니다** (TNC7 사용 설명서 'Calling an NC program with PGM CALL' 은 부르는 프로그램이 도는 동안 그것이 부르는 프로그램을 고칠 수 없다고 적습니다). 상태 `-22`(기계 상태)입니다. 운전이 끝나거나 다른 프로그램을 선택한 뒤에는 지워졌습니다. 조작반 파일 관리자에서 쓰기 방지가 걸린 파일도 상태 `-22` 이고 사유에 쓰기 방지라고 실립니다. 선택된 프로그램도 운전하지 않았거나 운전이 끝났으면 지워지며, 그 뒤에도 `/machine/channel/mainProgramPath` 는 지운 경로를 가리킵니다 (테스트 환경에서 확인). 디메시로 지운 파일은 조작반의 휴지통에 없었습니다. 되살릴 수 없으니 지우기 전에 확인하세요 (조작반 파일 관리자로 지운 것은 휴지통으로 간다고 설명서 'Basic information' 이 적습니다).

Siemens 에서 **채널이 Reset 이면 선택된 메인 프로그램도 지워지고, 제어기가 그 선택을 풉니다.** 840D sl 벤치에서는 지운 뒤 `/machine/channel/mainProgramPath` 가 빈 문자열(선택된 프로그램 없음)이 되었습니다. 지운 뒤에도 같은 프로그램이 선택돼 있다고 가정하지 말고, 필요하면 프로그램을 다시 선택하세요.

**Fanuc 데이터 서버(`//DATA_SV`)에서는 이 쓰기가 데이터 서버의 현재 폴더를 옮깁니다.** 데이터 서버의 폴더·파일을 다루는 Fanuc 함수는 현재 폴더 안의 이름만 받아서, 디메시가 먼저 대상의 부모 폴더로 옮긴 뒤 부르고 그 폴더에 둔 채 끝냅니다. 이 현재 폴더는 장비에 하나이고 조작반 데이터 서버 화면과 같은 것이라, 조작반은 그 화면에 다시 들어갈 때 바뀐 폴더를 보입니다 (31i-B 실장비에서 확인). 그래서 **한 장비의 데이터 서버 쓰기는 한 곳에서만 하세요.** 다른 프로그램이나 조작반이 같은 때 폴더를 옮기면, 이름으로 부르는 호출이 다른 폴더의 같은 이름에 닿을 수 있습니다. 이 연결로 데이터 서버의 폴더·파일 조작을 할 수 없는 장비에서는 상태 `-18`(필터 값 오류)이고 사유에 그 뜻이 실립니다 (저희 테스트 벤치 하나가 그랬고, 그 장비의 조작반에서는 같은 조작이 되었습니다).

## /machine/ncMemoryPath/fileContent
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

NC 파일의 **내용**을 읽고(다운로드) 씁니다(업로드: 없으면 생성. 이미 있을 때의 동작은 기종에 따라 다릅니다. 아래 참조). 값은 문자열 (프로그램 텍스트).

- **Fanuc 쓰기 자동 처리**: `%` 미포함 시 자동 삽입, 맨 앞에 O번호/`<이름>` 이 없으면 경로의 파일명 기준으로 자동 삽입. 마지막 블록이 줄바꿈으로 끝나지 않으면 줄바꿈을 붙입니다 (없으면 마지막 블록이 `M30%` 처럼 저장되어 운전이 그 블록에서 `SR5010` 알람으로 멈췄습니다. 31i-B 실장비에서 확인). 저장 파일명은 **내용의 O번호/이름 기준**입니다
- **Siemens·Mitsubishi·Heidenhain 은 내용을 그대로 씁니다.** 자동 삽입이 없고, 저장 파일명은 **경로의 파일명**입니다. 내용의 O번호가 달라도 경로대로 저장됩니다 (Fanuc 과 반대). `%` 나 O번호가 필요하면 값에 직접 넣으세요
- **이미 있는 파일에 쓸 때**: Fanuc 은 파라미터 `3201#2`(REP)가 `1` 이면 기존 프로그램을 지우고 새로 등록합니다 (`0` 이면 상태 `-21`(이미 있음)이고 기존 프로그램은 그대로입니다. 바꾸려면 `fileExists` 에 `false` 를 써서 지운 뒤 올리거나 `3201#2` 를 `1` 로 두세요. 파라미터 매뉴얼 B-64490EN). Mitsubishi·Heidenhain 은 덮어씁니다 (테스트 환경에서 확인). **Siemens 는 덮어쓰지 않습니다. 상태 `-21`(이미 있음)로 거절하고 기존 내용은 그대로입니다** (테스트 벤치에서 확인). 바꾸려면 `fileExists` 에 `false` 를 써서 지운 뒤 업로드하세요. 지운 다음 만드는 순서를 디메시가 대신하지 않는 것은, 생성이 실패하면 원본이 사라지기 때문입니다. 그 판단은 호출자의 몫입니다
- **Mitsubishi 편집 잠금**: 파라미터 `#8105`(편집 잠금 B)가 `1` 이면 8000~9999 번, `#1121`(편집 잠금 C)가 `1` 이면 9000~9999 번 프로그램은 새로 만들거나 덮어쓸 수 없고, 쓰기는 상태 `-22`(기계 상태)로 거절됩니다 (레퍼런스 IB-1501209). 이미 있던 내용은 그대로입니다. 저희 테스트 환경에서 이 잠금은 쓰기에만 걸렸고, 그 번호의 읽기·선택·이름 변경·삭제는 잠금과 관계없이 받았습니다
- **Mitsubishi 사용자 레벨 데이터 보호**: 파라미터 `#1391` 이 `1` 이고 조작반 보호 설정 화면(Mainte > Protect setting)의 "Program edit" 변경 레벨이 현재 조작 레벨보다 높으면, 쓰기는 상태 `-22`(기계 상태)로 거절되고 이미 있던 내용은 그대로입니다 (시뮬레이터에서 확인). 조작반에서 그 레벨의 비밀번호로 조작 레벨을 올린 뒤 다시 쓰세요. 이름 변경(`entryName`)과 삭제(`fileExists`)도 같고, 읽기는 막히지 않았습니다
- **Mitsubishi 의 쓰기 금지된 SD 카드(`//IC1`)**: 쓰기는 상태 `-22`(기계 상태)로 거절됩니다. 쓰기 금지된 SD 카드로는 아직 확인하지 못했습니다
- **Fanuc 의 보호**: 파라미터 `3202#0`/`#4`(O8000~O8999·O9000~O9999 편집 금지)나 조작반의 편집 금지 속성(폴더·파일)이 걸린 프로그램은 쓰기(새로 만들기 포함)가 상태 `-22`(기계 상태)로 거절됩니다. `3202` 보호는 `3202#6`(PSR)이 `0` 이면 읽기도 상태 `-22` 이고(`1` 이면 읽힙니다. 31i 벤치에서 확인), 편집 금지 속성은 읽기를 막지 않습니다 (테스트 환경에서 확인). 보호를 풀 수 있는지는 장비 담당자에게 확인하세요
- **Fanuc·Mitsubishi 에서 제어기가 쓰고 있는 프로그램**: Fanuc 은 선택된 메인 프로그램(운전 중이 아니어도), Mitsubishi 는 자동운전 중인 계통의 메인 프로그램을 덮어쓰지 않고, 쓰기는 상태 `-22`(기계 상태)로 거절됩니다. Fanuc 의 이 거절은 `3201#2`(REP)가 `1` 일 때이고, `0` 이면 그보다 먼저 위의 상태 `-21`(이미 있음)입니다 (테스트 환경에서 확인). Fanuc 은 다른 프로그램을 선택한 뒤, Mitsubishi 는 계통을 리셋하거나 운전이 끝난 뒤 다시 쓰세요. 저희 테스트 환경에서 두 기종 모두 실행 중인 서브프로그램의 덮어쓰기는 거절하지 않았습니다
- **Heidenhain 에서 제어기가 쓰고 있는 프로그램과 쓰기 방지**: 운전 중인 프로그램은 메인도, 아직 부르지 않은 서브도 덮어쓰지 않고 쓰기는 상태 `-22`(기계 상태)입니다. 조작반 파일 관리자에서 쓰기 방지가 걸린 파일을 덮어쓰거나 쓰기 방지가 걸린 폴더에 새로 만들 때도 상태 `-22` 이고 사유에 쓰기 방지라고 실립니다. 넣을 폴더가 없거나 경로가 폴더면 상태 `-18` 입니다 (테스트 환경에서 확인). Heidenhain 은 HEIDENHAIN DNC 가 파일을 PC 쪽 파일로 주고받아, 디메시가 읽기·쓰기마다 PC 의 임시 폴더에 파일을 만들고 곧바로 지웁니다. 보낸 바이트가 그대로 저장되고 그대로 읽혔습니다 (UTF-8 한글과 CRLF 포함, 테스트 환경). TNC7 사용 설명서('Converting files')는 기계 제작사 설정에 따라 제어기가 들여온 파일을 고칠 수 있다고 적는데(움라우트 제거 등), HEIDENHAIN DNC 전송이 그에 해당하는지는 확인하지 못했습니다. 대화형 프로그램의 `BEGIN PGM`·`END PGM` 줄은 조작반 편집기가 자동으로 넣지만 디메시는 넣지 않으니 값에 직접 넣으세요. 파일 이름에는 영문자·숫자·`_`·`-` 를 쓰고 경로는 255자까지입니다 (설명서 'Basic information')
- **자리가 없을 때**: NC 메모리가 모자라거나 등록할 수 있는 프로그램 수가 다 차면 상태 `-23`(자리 없음)입니다 (Fanuc 은 테스트 벤치, Mitsubishi 는 시뮬레이터에서 확인). 필요 없는 프로그램을 지운 뒤 다시 올리세요. 이미 있는 프로그램을 덮어쓰는 것은 개수가 다 차도 됩니다 (Mitsubishi 에서 확인). Fanuc 은 폴더도 같은 개수에 들어갑니다. Mitsubishi 의 `//PRG2` 는 개수가 다 찼는지 가려내지 못해 상태 `-17` 로 돌아옵니다. **Fanuc 은 메모리가 모자라면 들어간 만큼만 끊어 내용의 O번호/이름으로 등록합니다.** 끝의 `M30` 만 빠져 온전한 프로그램처럼 보이므로, 디메시가 그 프로그램이 보낸 내용의 앞부분인지 확인해 지우고 에러 문구에 적습니다. `3201#2`(REP)가 `1` 이면 같은 이름의 기존 프로그램은 제어기가 이미 바꾼 뒤라 함께 없어집니다. 여유가 전혀 없으면 제어기가 아무것도 바꾸지 않아 기존 프로그램은 그대로입니다. 확인하거나 지우지 못하면 에러 문구가 그렇게 알리니, 그 프로그램을 실행하기 전에 확인하세요
- **Fanuc 에서 내용이 형식에 맞지 않을 때**: 제어기가 알람(예: `BG1090`)을 내고, 받은 데까지를 그 이름으로 등록할 수 있습니다. 디메시는 상태 `-16`(쓰기 값 오류)으로 답하고 에러 문구에 알람 코드를 싣습니다. 내용을 고쳐 다시 올리세요. 올리기 전에 그 폴더에 없던 이름이면 디메시가 제어기가 남긴 프로그램을 지우고 에러 문구에 적습니다. 이미 있던 이름이면 지우지 않습니다. `3201#2`(REP)가 `1` 이면 기존 프로그램이 받은 데까지로 바뀌었을 수 있으니 실행하기 전에 확인하세요 (`0` 이면 상태 `-21` 이고 기존 프로그램은 그대로입니다). 알람은 조작반에서 RESET 해야 풀리지만, 풀리기 전에도 다른 업로드는 됩니다. 이것을 가리려고 디메시는 Fanuc 에 올릴 때마다 먼저 그 폴더의 목록을 한 번 읽습니다 (알람과 남는 프로그램은 실장비와 시뮬레이터에서, 디메시의 뒷정리는 시뮬레이터의 CNC 메모리와 실장비의 데이터 서버에서 확인했습니다)
- **Siemens 에서 채널이 쓰는 이름으로 만들 때**: 채널이 그 이름을 쥐고 있으면(선택된 메인, 실행 중이거나 선독이 열어 둔 서브. 방금 지운 파일을 곧바로 다시 만들 때가 전형) 파일은 만들어지는데 내용을 쓰려는 `Open` 이 상태 `-22`(기계 상태)로 거절됩니다. 채널이 전부 Reset 이어도 제어기가 지금 상태에서는 `Open` 을 받지 않는다고 답하면(`BadInvalidState`) 역시 상태 `-22` 이고, 에러 문구는 편집기나 다른 클라이언트가 파일을 쥔 경우로 안내합니다. 어느 쪽이든 디메시는 **방금 만든 빈 파일을 같은 호출 안에서 지웁니다** (호출 전 상태로 되돌림). 채널이 그 빈 파일까지 잡아 못 지우면 에러 문구에 남았다고 알리니, 채널이 놓은 뒤 지우세요
- 파일 삭제는 `fileExists` 에 `false` 쓰기
- **시스템·기계 제조사 영역에는 쓰지 않습니다.** 장비에 보내지 않고 상태 `-18` 로 거절하며 읽기는 됩니다 (`fileExists`·`entryName`·`directoryExists` 도 같습니다). 잘못 바꾸면 기계 동작이 바뀌거나 장비를 쓸 수 없게 될 수 있는 영역이라 장비의 보호 설정과 관계없이 막습니다
  - Fanuc: `//CNC_MEM/SYSTEM`·`//CNC_MEM/MTB1`·`//CNC_MEM/MTB2` (시스템·기계 제조사의 매크로 폴더. G·M·T 코드 매크로 호출이 여기서 프로그램을 찾습니다, 조작 설명서 B-64484EN). `//CNC_MEM/USER/LIBRARY` 는 사용자 공통 폴더라 막지 않습니다
  - Mitsubishi: `//PRG/FIX`(고정 사이클)·`//PRG/MMACRO`(기계 제조사 매크로). 조작반도 파라미터 `#1166` 을 켜야 이 영역을 편집하게 합니다
  - Heidenhain: `//TNC/table`(공구표·프리셋 등)·`//TNC/system`·`//TNC/config` 와 그 아래 전부. 제어기가 정해 둔 이름으로 쓰는 파일이 있어, 잘못 지우거나 덮어쓰면 장비를 쓸 수 없게 될 수 있습니다. 이 폴더들에는 TNC7 사용 설명서가 사용자 파일 자리로 정한 하위 폴더도 있지만(`system/PGM-Templates`·`system/Toolkinematics`·`system/3D-ToolComp`, `table` 의 자유 정의 표 등) 디메시로는 올리지 않습니다. 조작반에서 다루세요

**쓰기의 `status` 0 은 전송이 끝났다는 뜻입니다.** 세 기종 모두 청크마다 제어기의 반환값을 확인하고 마지막에 파일을 닫는 것까지 확인한 뒤 0 을 냅니다 (Fanuc `cnc_download4`·`cnc_dwnend4`, Siemens `Write`·`Close`, Mitsubishi `WriteFile`·`CloseFile3`). Heidenhain 은 파일 전송 한 번(`TransmitFile`)이 끝난 뒤 0 을 냅니다. 중간에 실패하면 에러이고, Mitsubishi 는 파일을 버리며 Siemens 는 만들다 만 파일을, Fanuc 은 메모리가 모자라 끊긴 프로그램을 거둡니다 (위 참조). Mitsubishi 에서 마지막 닫기가 거절되면(NC 메모리가 모자랄 때 등) 제어기가 대상 폴더에 미완성 임시 파일(이름이 `~` 로 시작)을 남기는데 (시뮬레이터에서 확인), 디메시가 그 파일을 지우고 에러 문구에 적습니다. 확인하거나 지우지 못하면 에러 문구가 그렇게 알리니 목록에서 그 파일을 찾아 `fileExists` 에 `false` 를 써서 지우세요. 완성된 프로그램이 아닙니다. 그러니 **전송 확인을 위해 다시 읽을 필요는 없습니다.** 다시 읽어 바이트를 비교하면 Fanuc 에서는 `%` 와 O 번호 삽입, 줄바꿈 정규화로 보낸 것과 달라지고, 목록의 `sizeBytes` 도 Fanuc 은 500바이트 단위의 할당 크기라(22바이트를 써도 `500`) 내용 길이와 맞지 않습니다 (저희 테스트에서 확인). 이 주소가 응답에 크기나 해시를 싣지 않는 이유가 그것입니다. 크기는 기종에 따라 뜻이 갈리고, 해시는 다시 읽지 않고는 만들 수 없어 그 비용을 안으로 옮길 뿐입니다. 내용을 확인해야 한다면 `fileContent` 를 읽어 **의미**(프로그램 번호·블록)로 비교하세요.

## /machine/channel/toolOffsetCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

공구 보정 레지스터의 **사용 가능 개수**입니다 (read 전용, `int`). 오프셋 번호는 `1`~이 값까지입니다. UI 가 테이블을 순회할 때 상한으로 쓰세요.

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어 번호 표의 칸 수가 없습니다. 공구 목록은 `/machine/toolArea/toolList`, 공구 하나의 보정 세트 수는 `/machine/toolArea/tool/toolEdgeCount` 로 읽으세요.

## /machine/channel/toolOffset/toolOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

공구 보정 번호 하나의 **보정량**입니다 (read + write, `float`). `channel` + `toolOffset` 필터가 필요하고, 쓰기는 `{"value": 12.345}` 입니다.

**이 주소는 보정 메모리가 열로 나뉘지 않은 Mitsubishi 장비 전용입니다.** 그런 장비의 조작반 보정량 화면은 번호마다 값을 **하나만** 보여줍니다. 형상/마모도, 길이/반경도 나뉘지 않습니다. 그래서 이름이 `toolLength…` 가 아니라 `toolOffsetValue` 입니다. **장비가 그 값을 "길이" 라고 부르지 않기 때문**이며, 그것이 길이 보정으로 쓰일지 반경 보정으로 쓰일지는 프로그램이 그 번호를 어떻게 참조하느냐에 달려 있습니다.

열이 나뉜 Mitsubishi 장비에서는 상태 `-20` 이 반환되며, **에러 문자열에 그 장비에서 되는 리프 목록**이 실려 옵니다 (예: `toolLength{Geometry,Wear}, toolRadius{Geometry,Wear}`). 즉 한 번 요청해 보면 그 장비의 보정 트리 모양을 알 수 있으므로, 어느 모델인지 미리 묻는 주소는 따로 없습니다.

보정 번호의 상한은 `/machine/channel/toolOffsetCount` 입니다. 그 범위를 벗어난 번호는 상태 `-18` 입니다. 값의 단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요.

쓴 값은 제어기가 설정 단위 `#1003` 의 자릿수로 반올림해 저장하고 상태 `0` 을 돌려줍니다 (`workOffsetValue` 와 같습니다. 테스트 환경에서 확인). 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (같은 쓰기 호출을 선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 이 쓰기는 **자동운전 중에도 받아들여집니다** (테스트 환경에서 축이 움직이는 중에, 프로그램이 쓰고 있는 번호로도 확인. 운전 중에 거절되는 `workOffsetValue` 와 다릅니다).

**Fanuc·Siemens 는 상태 `-20` 입니다.** 두 기종의 어댑터는 이 주소를 지원하지 않으므로, 보정 메모리 구성과 무관하게 거절하고 에러 문자열에 리프 목록도 싣지 않습니다. Fanuc 은 `toolLength…`·`toolRadius…`(선반은 `toolX…` 계열) 리프로 읽습니다. 길이/반경이 나뉘지 않은 오프셋 메모리(A·B)의 값은 FOCAS2 스펙상 공구경 열로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. Siemens 는 공구 단위 표라 `/machine/toolArea/tool/toolEdge/…` 로 읽습니다.

## /machine/channel/toolOffset/toolName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 보정 번호로 불리는 **공구의 이름**입니다. `channel` + `toolOffset` 필터. 반환 `string`. 읽기·쓰기 모두 Fanuc 전용입니다.

값은 Fanuc 의 **공구 형상 크기 데이터**(조작반 `TL GEOM SIZE` 화면, 옵션 "Tool geometry size data 100/300 pairs")에서 옵니다. 그 표는 **공구 보정 번호로 색인**됩니다 (조작 설명서 B-64484EN: M 계열은 `D` 코드, T 계열은 형상 오프셋 번호와 같은 번호의 행). 그래서 이 주소는 공구관리 칸(`/machine/toolArea/tool/…`)이 아니라 `toolOffset` 폴더에 있고, 공구관리(TOOL MANAGEMENT) 옵션과는 무관합니다. 형상 크기 데이터 옵션이 없으면 상태 `-20` 입니다.

종류가 정해지지 않은 행(공구 종류 `0`)은 빈 문자열 `""` 입니다. 표의 크기는 옵션이 정하므로 `/machine/channel/toolOffsetCount` 와 같다는 보장이 없습니다 (벤치 장비는 둘 다 100). 표 밖 번호는 상태 `-18` 로 거절하며 문구에 표의 끝을 싣습니다. 여러 번호를 범위로 물으면(`toolOffset=1-100`) 한 번의 왕복으로 읽습니다.

**쓰기**는 장비에 8바이트 이내로 남는 문자열입니다 (초과는 상태 `-16`. 장비의 표시 언어 코드페이지로 옮길 수 없는 글자도 상태 `-16`). 종류가 정해지지 않은 행에는 이름을 쓸 수 없어 상태 `-18` 입니다 (Fanuc 은 종류 없는 행을 만들지 않습니다. `/machine/channel/toolOffset/toolType` 을 먼저 쓰거나 조작반 `TL GEOM SIZE` 화면에서 종류를 먼저 정하세요). 이미 같은 이름이면 아무것도 하지 않고 성공합니다.

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

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. Mitsubishi 에서는 **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. Fanuc 어댑터는 `toolOffsetValue` 를 지원하지 않아 이 안내가 없고, 길이/반경이 나뉘지 않은 오프셋 메모리(A·B)의 값은 FOCAS2 스펙상 공구경 열(`toolRadius…`)로 지정하게 되어 있으나 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 형상·마모와 길이·반경으로 나뉜 장비(type II)의 네 칸 중 하나입니다 (테스트 환경에서 조작반의 Length·L wear·Radius·R wear 칸과 대조). 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (같은 쓰기 호출을 선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (이 구성의 시뮬레이터에서 확인).

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

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. Mitsubishi 에서는 **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. Fanuc 어댑터는 `toolOffsetValue` 를 지원하지 않아 이 안내가 없고, 길이/반경이 나뉘지 않은 오프셋 메모리(A·B)의 값은 FOCAS2 스펙상 공구경 열(`toolRadius…`)로 지정하게 되어 있으나 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 형상·마모와 길이·반경으로 나뉜 장비(type II)의 네 칸 중 하나입니다 (테스트 환경에서 조작반의 Length·L wear·Radius·R wear 칸과 대조). 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (같은 쓰기 호출을 선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (이 구성의 시뮬레이터에서 확인).

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

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. Mitsubishi 에서는 **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. Fanuc 어댑터는 `toolOffsetValue` 를 지원하지 않아 이 안내가 없고, 길이/반경이 나뉘지 않은 오프셋 메모리(A·B)의 값은 FOCAS2 스펙상 공구경 열(`toolRadius…`)로 지정하게 되어 있으나 그런 구성의 장비에서는 확인하지 않았습니다. Fanuc 머시닝센터에 밀링·터닝 공구 보정(Tool offset for Milling and Turning) 기능이 켜져 있으면 이 주소는 읽기·쓰기 모두 상태 `-20` 입니다. 그 구성에서는 보정 열의 번호가 달라져(FOCAS2 스펙) 이 주소가 다른 열을 가리키게 되는데, 그런 장비에서 확인하지 못해 막아 두었습니다. `toolLengthGeometry`·`toolLengthWear` 는 영향이 없습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 **반지름**이지만, 오프셋 화면은 설정에 따라 **지름**으로 표시·입력하게 설정돼 있을 수 있습니다. 디메시는 장비가 저장한 값을 그대로 내보내며 임의로 환산하지 않습니다.

**Mitsubishi** 에서는 보정 메모리가 형상·마모와 길이·반경으로 나뉜 장비(type II)의 네 칸 중 하나입니다 (테스트 환경에서 조작반의 Length·L wear·Radius·R wear 칸과 대조). 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (같은 쓰기 호출을 선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (이 구성의 시뮬레이터에서 확인).

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

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. Mitsubishi 에서는 **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. Fanuc 어댑터는 `toolOffsetValue` 를 지원하지 않아 이 안내가 없고, 길이/반경이 나뉘지 않은 오프셋 메모리(A·B)의 값은 FOCAS2 스펙상 공구경 열(`toolRadius…`)로 지정하게 되어 있으나 그런 구성의 장비에서는 확인하지 않았습니다. Fanuc 머시닝센터에 밀링·터닝 공구 보정(Tool offset for Milling and Turning) 기능이 켜져 있으면 이 주소는 읽기·쓰기 모두 상태 `-20` 입니다. 그 구성에서는 보정 열의 번호가 달라져(FOCAS2 스펙) 이 주소가 다른 열을 가리키게 되는데, 그런 장비에서 확인하지 못해 막아 두었습니다. `toolLengthGeometry`·`toolLengthWear` 는 영향이 없습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 **반지름**이지만, 오프셋 화면은 설정에 따라 **지름**으로 표시·입력하게 설정돼 있을 수 있습니다. 디메시는 장비가 저장한 값을 그대로 내보내며 임의로 환산하지 않습니다.

**Mitsubishi** 에서는 보정 메모리가 형상·마모와 길이·반경으로 나뉜 장비(type II)의 네 칸 중 하나입니다 (테스트 환경에서 조작반의 Length·L wear·Radius·R wear 칸과 대조). 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (같은 쓰기 호출을 선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (이 구성의 시뮬레이터에서 확인).

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

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. Fanuc 에서 형상/마모가 나뉘지 않은 오프셋 메모리(A)의 값은 FOCAS2 스펙상 마모 열(`…Wear`)로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 선반 배치인 장비의 열입니다. 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (같은 쓰기 호출을 머시닝센터 구성의 시뮬레이터에서 확인).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있고, 그 표에는 X·Y·Z 방향 열 대신 길이1~3(`/machine/toolArea/tool/toolEdge/toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있습니다. 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/channel/toolOffset/toolXWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 X 방향 공구 치수 마모값**입니다. 가공 중 누적되는 X 방향 보정분입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. Fanuc 에서 형상/마모가 나뉘지 않은 오프셋 메모리(A)의 값은 FOCAS2 스펙상 마모 열(`…Wear`)로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 선반 배치인 장비의 열입니다. 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (같은 쓰기 호출을 머시닝센터 구성의 시뮬레이터에서 확인).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있고, 그 표에는 X·Y·Z 방향 열 대신 길이1~3(`/machine/toolArea/tool/toolEdge/toolLengthWear`·`toolLength2Wear`·`toolLength3Wear`)이 있습니다. 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/channel/toolOffset/toolZGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Z 방향 공구 치수 형상값**입니다. X 와 마찬가지로 축이 아니라 화면의 고정 열입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. Fanuc 에서 형상/마모가 나뉘지 않은 오프셋 메모리(A)의 값은 FOCAS2 스펙상 마모 열(`…Wear`)로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 선반 배치인 장비의 열입니다. 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (같은 쓰기 호출을 머시닝센터 구성의 시뮬레이터에서 확인).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있고, 그 표에는 X·Y·Z 방향 열 대신 길이1~3(`/machine/toolArea/tool/toolEdge/toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있습니다. 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/channel/toolOffset/toolZWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Z 방향 공구 치수 마모값**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. Fanuc 에서 형상/마모가 나뉘지 않은 오프셋 메모리(A)의 값은 FOCAS2 스펙상 마모 열(`…Wear`)로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 선반 배치인 장비의 열입니다. 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (같은 쓰기 호출을 머시닝센터 구성의 시뮬레이터에서 확인).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있고, 그 표에는 X·Y·Z 방향 열 대신 길이1~3(`/machine/toolArea/tool/toolEdge/toolLengthWear`·`toolLength2Wear`·`toolLength3Wear`)이 있습니다. 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/channel/toolOffset/toolYGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Y 방향 공구 치수 형상값**으로, X·Z 에 이은 **세 번째 열**입니다. Fanuc 은 Y축 오프셋 옵션이 없는 선반에서 상태 `-20` 을 돌려줍니다. Mitsubishi 에서 제3축이 없는 선반이 무엇을 답하는지는 확인하지 못했습니다.

**기계에 따라 이 열의 화면 머리글이 `Y` 가 아닐 수 있습니다.** Mitsubishi 는 이 자리를 제3축에 배정하므로 C축 선반에서는 조작반이 `공구길이 C` 로 표시합니다 (레퍼런스 IB-1501209 도 옛 판은 이 열을 `C (Y*)` 로, 최신 판은 추가 축으로 적어 축 이름을 정해 두지 않습니다). 주소가 약속하는 것은 **세 번째 오프셋 열**이며, 그 열이 어느 축인지는 조작반의 열 머리글이 알려 줍니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. Fanuc 에서 형상/마모가 나뉘지 않은 오프셋 메모리(A)의 값은 FOCAS2 스펙상 마모 열(`…Wear`)로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 선반 배치인 장비의 열입니다. 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (같은 쓰기 호출을 머시닝센터 구성의 시뮬레이터에서 확인).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있고, 그 표에는 X·Y·Z 방향 열 대신 길이1~3(`/machine/toolArea/tool/toolEdge/toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있습니다. 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/channel/toolOffset/toolYWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Y 방향 공구 치수 마모값**입니다. X·Z 에 이은 **세 번째 열**이며, 기계에 따라 화면 머리글이 `Y` 가 아닐 수 있습니다 (Mitsubishi C축 선반은 마모 `C`). 자세히는 `toolYGeometry` 를 보세요.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. Fanuc 에서 형상/마모가 나뉘지 않은 오프셋 메모리(A)의 값은 FOCAS2 스펙상 마모 열(`…Wear`)로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 선반 배치인 장비의 열입니다. 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (같은 쓰기 호출을 머시닝센터 구성의 시뮬레이터에서 확인).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있고, 그 표에는 X·Y·Z 방향 열 대신 길이1~3(`/machine/toolArea/tool/toolEdge/toolLengthWear`·`toolLength2Wear`·`toolLength3Wear`)이 있습니다. 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/channel/toolOffset/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 노즈 반경 형상값**입니다. 노즈 반경 보정(G41/G42)이 참조하며, 팁 방향(`toolTipDirection`)과 함께 날끝 궤적을 결정합니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. Fanuc 에서 형상/마모가 나뉘지 않은 오프셋 메모리(A)의 값은 FOCAS2 스펙상 마모 열(`…Wear`)로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 선반 배치인 장비의 열입니다. 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (같은 쓰기 호출을 머시닝센터 구성의 시뮬레이터에서 확인).

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

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 상태 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. Fanuc 에서 형상/마모가 나뉘지 않은 오프셋 메모리(A)의 값은 FOCAS2 스펙상 마모 열(`…Wear`)로 지정하게 되어 있으나, 그런 구성의 장비에서는 확인하지 않았습니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G71`/`G710` 이면 metric, `G20`/`G70`/`G700` 이면 inch. Siemens 의 `G70`/`G71` 은 좌표값만 바꾸고 이송·공구 오프셋·워크 오프셋은 기본 시스템(`MD10240`) 단위를 유지하며, `G700`/`G710` 은 그것들까지 바꿉니다 (프로그래밍 매뉴얼). 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음). **Fanuc 은 이 값의 소수점 자릿수를 연결할 때 정하므로, `G20`/`G21` 처럼 단위 설정을 바꾸면 다시 연결하세요** (SDK 는 `deemesh_disconnect` 뒤 `deemesh_connect`, 허브는 `POST /admin/reload`). 다시 연결하기 전에는 옛 자릿수로 읽고 써서 10배 틀릴 수 있습니다.

**Mitsubishi** 에서는 보정 메모리가 선반 배치인 장비의 열입니다. 쓴 값은 설정 단위 `#1003` 의 자릿수로 반올림돼 저장되고, 제어기가 받지 않는 값(설정 범위 밖)은 상태 `-16` 입니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터와 좌표 데이터를 보호합니다)이 꺼져 있으면 상태 `-22`(기계 상태)이니 키를 켠 뒤 다시 쓰세요 (선반 구성의 시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다). 자동운전 중에도 쓰기가 받아들여집니다 (같은 쓰기 호출을 머시닝센터 구성의 시뮬레이터에서 확인).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/toolOffset/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

선반 공구의 **가상 날끝 위치 코드**입니다 (read + write). 노즈 반경 보정(G41/G42) 때 날끝이 노즈 중심 기준 어느 방위에 있는지 판정하는 코드입니다. 각도가 아니라 위치 코드이며, 배율 없는 정수 그대로 반환/입력합니다 (`{"value": 3}`). `channel` + `toolOffset` 필터가 필요합니다. **선반 계열 오프셋 메모리에만 있는 열**이라, 기종의 오프셋 화면에 이 열이 없으면 상태 `-20`(미지원)으로 거절하고 그 채널에서 쓸 수 있는 리프 목록을 에러 문자열에 실어 줍니다. Mitsubishi 에서는 보정 메모리가 열로 나뉘지 않은 장비도 여기 해당하며, 그때는 `toolOffsetValue` 를 쓰라고 안내합니다.

- `1`~`8` = 방위, **`0`/`9` = 노즈 중심이 기준점** (가상 날끝이 아니라). 두 값은 같은 의미입니다. 노즈 중심이 기준점과 일치할 때 `0` 또는 `9` 를 쓴다고 Fanuc 0i-F 선반 매뉴얼(`B-64604EN-1/01`)이 정의합니다
- **`desc` 는 `0`·`9` 에만 붙습니다.** `1`~`8` 은 매뉴얼이 평면별 도해로 정의하고 그 도해가 평면(`G17`/`G18`/`G19`)별로 여러 벌이라, 같은 번호가 구성에 따라 다른 방위를 가리킵니다. 방위 해석은 그 기종 매뉴얼의 도해를 따르세요
- Siemens 대응 개념: cutting edge position (`toolArea/tool/toolEdge/toolTipDirection`). **두 트리는 같은 번호 체계와 같은 `desc` 어휘를 씁니다.** 주소만 다를 뿐 값은 그대로 비교·재사용할 수 있습니다
- **허용 범위가 기종마다 다릅니다**: Fanuc `0`~`9`, Siemens `1`~`9`, Mitsubishi `0`~`8`. 중심을 가리키는 코드도 각각 `0`/`9`, `9`, `0` 이라 **읽을 때는 `desc` 로 통일되지만 쓸 때는 그 장비의 범위를 지켜야 합니다** (Fanuc 에서 되는 `9` 를 Mitsubishi 에 그대로 보내면 상태 `-16`)

**Mitsubishi** 의 쓰기는 다른 선반 열과 같이 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`)이 꺼져 있으면 상태 `-22`(기계 상태)입니다 (같은 쓰기 호출을 선반 구성의 시뮬레이터에서 확인). 범위 밖의 코드는 장비에 보내기 전에 상태 `-16` 으로 거절합니다.

**Siemens 는 상태 `-20` 입니다.** Siemens 의 보정은 채널 보정 번호 표가 아니라 공구 단위 표에 있어, 같은 값은 `/machine/toolArea/tool/toolEdge/…` 의 같은 이름 리프로 읽으세요.

## /machine/channel/activeToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

그 채널에서 지금 활성인 공구의 번호(`T`)입니다. `channel` 필터. 반환 `int`.

**"활성" 이 되는 시점이 기종에 따라 다릅니다.** Fanuc·Mitsubishi 는 `T` 모달이라 **지령하는 순간** 바뀌고, Siemens 는 `actTNumber` 라 **교환이 끝난 뒤에** 바뀝니다 (Heidenhain 도 교환이 끝난 뒤이며 아래 Heidenhain 문단에 적었습니다):

| 상황 | Fanuc·Mitsubishi | Siemens |
|---|---|---|
| `T7` 만 지령 (아직 `M06` 전) | `7` | **그 장비의 교환 방식에 달렸습니다** (아래) |
| `T7 M06` 이 끝난 뒤 | `7` | `7` |

⚠️ **Siemens 는 `T` 만으로 교환이 끝나는 장비가 있습니다.** 교환을 `M06` 이 하는지 `T` 가 하는지는 기계 제작사가 정하는 설정입니다. 840D sl 벤치에서 `T="CUTTER 10"` 만 지령했더니 **`M06` 없이 교환이 완료됐습니다**: 이 값이 곧바로 바뀌었고, 그 공구의 `toolLocationType` 이 `magazine` → `buffer`(스핀들)로, 물러난 공구는 반대로 갔습니다. 그러니 이 값을 "교환 전" 의 표시로 쓰지 마세요. 교환 여부는 `/machine/toolArea/tool/toolLocationType` 이 확실히 답합니다.

앞의 두 기종은 확인했습니다. `M06` 없이 `T7` 만 준 직후 값이 `7` 이 되며, Mitsubishi 는 조작반의 공구번호 표시가 이전 공구에 그대로 있는 것까지 함께 관측했습니다. 리셋(`M30`)으로도 지워지지 않습니다. Fanuc 쪽은 **실제 공구교환 매크로가 도는 31i 벤치**에서 다시 확인했습니다: `T` 만 있는 블록 다음 드웰에서 이미 그 번호였고(`M06` 은 아직 실행 전), 교환이 끝난 뒤에도 같았으며, 두 번째 `T` 에서도 그 자리에서 바뀌었습니다. 프로그램이 `M30` 으로 끝난 뒤에도 값이 남았습니다.

**그래서 Fanuc·Mitsubishi 에서는 이 값을 "지금 깎고 있는 공구" 로 해석하면 안 됩니다**. 지령된 공구이기 때문입니다. 교환 시점이 중요한 용도라면 이 주소를 교환 신호로 쓰지 마시고 장비의 교환 완료 신호를 보세요. 실제로 물려 있는 공구는 이 두 기종에서 이름 붙은 주소로는 얻을 수 없습니다. 조작반에 뜨는 공구번호는 기계 제작사가 래더로 만드는 값이라 장비마다 다르고, `plcAddress` 와 같은 이유로 중립화가 성립하지 않습니다. 다만 그 번호가 담긴 PMC 주소를 알면 `/machine/plcAddress/plcType/plcValue` 로 읽을 수 있습니다. Fanuc 조작반의 `HD.T`(스핀들 공구)·`NX.T`(다음 공구)가 그런 값입니다. 파라미터 `3108#2` 와 `13200#1` 이 `1` 인 장비에서 T 모달 대신 보이며, 기계 제작사의 래더가 PMC 창 기능으로 넣는 번호입니다 (PMC 프로그래밍 매뉴얼 B-64513EN §5.4.26). 래더가 그 번호를 담아 두는 PMC 주소는 기계 제작사에 물으세요 (31i-B 실장비에서 그 설정과, 그때 이 주소가 T 모달 그대로 `0` 인 것을 확인).

⚠️ **Fanuc 에 공구관리(Tool Management) 옵션이 켜져 있으면 이 값은 공구 번호가 아닙니다.** 그 옵션에서는 `T` 가 공구를 직접 가리키지 않고 **공구 타입(그룹) 번호**를 지정하며, 제어기가 그 타입에 속한 실제 공구를 골라 씁니다. 31i 벤치에서 확인: 조작반의 `EACH TOOL DATA` 가 공구 `1` 의 타입을 `4` 로 두고 있을 때 `T4` 를 지령하니 이 주소가 `4` 를 냈습니다(실제 공구는 `1`). 옵션이 꺼진 장비에서는 `T` 가 곧 공구 번호라 이 문제가 없습니다. 그 옵션을 쓰는 장비라면 이 값을 아래 공구 트리 조회에 그대로 넣지 마세요. 대신 `/machine/toolArea/toolList` 에서 `toolTNumber` 가 이 값과 같은 항목들이 후보 공구이고, `/machine/toolArea/tool/toolTNumber` 로 낱개 확인할 수 있습니다.

⚠️ **Siemens 의 공구관리(WZV)에서는 프로그램의 `T` 가 이름을 가리킵니다.** 이 주소가 내는 번호(그리고 `tool` 필터가 받는 번호)는 제어기 **내부의 공구 번호**라, 프로그램에 그대로 타이핑하는 값이 아닙니다. 840D sl 벤치에서 `T3` 은 알람 `17190`(illegal T number)이었고 `T="CUTTER 10"` 이 통과했으며, 그 공구의 내부 번호가 바로 `3` 이었습니다. 번호 ↔ 이름은 `/machine/toolArea/toolList` 의 `toolNumber` 와 `toolName` 이 짝지어 알려줍니다.

**Fanuc 에서 공구수명관리(tool life management)로 그룹을 지령해도 이 값은 공구 번호입니다.** 프로그램이 파라미터 `6810` 보다 큰 값으로 그룹을 부르면(예: `6810` 이 `1000` 인 장비에서 `T1001` = 그룹 `1`), 제어기가 그 그룹에서 쓸 공구를 골라 **그 공구 번호를 이 자리에 넣습니다.** Fanuc 실장비에서 확인: 그룹 `1` 의 첫 공구가 `16` 인 장비에서 `T1001` 을 걸자 이 값이 `16` 이 됐고, 교환 후에도 `16` 이었습니다. 그룹 번호가 이 값으로 나오는 경우는 없습니다. Mitsubishi 의 공구 수명 관리로 그룹을 지령했을 때의 값은 확인하지 못했습니다.

지금 수명이 깎이는 그룹은 `/machine/channel/activeToolGroupNumber` 가, 그 그룹의 공구 목록은 `/machine/toolArea/toolGroup/toolNumberList` 가 알려줍니다.

이 번호를 공구 트리의 `tool` 필터에 넣으면 그 공구의 이름·보정 세트 수·오프셋을 조회할 수 있습니다. 함께 필요한 `toolArea` 값은 `/machine/channel/toolAreaNumber` 가 알려줍니다 (연결 시 캐싱되어 추가 통신이 없습니다).

Fanuc 은 `T` 모달, Siemens 는 `actTNumber` (`$P_TOOLNO`: 지금 유효한 D 보정이 계산된 공구의 T 번호), Mitsubishi 는 `GetCommand2` 의 T 지령 모달을 읽습니다.

**Heidenhain** 은 포켓 테이블의 스핀들 행(`0.0`, 조작반 공구 관리 화면의 `Spindle`)에 든 공구 번호입니다. Siemens 처럼 **교환이 끝난 뒤에** 바뀝니다 (시뮬레이터에서 `TOOL CALL 5` 가 끝난 뒤 `5` 가 된 것을 확인). 스핀들이 비었으면 `0` 이고, 포켓 테이블에서 스핀들 행을 찾지 못하면 상태 `-20` 입니다 (이 주소가 그 장비에서 동작하지 않습니다). 인덱스 공구(`10.1` 처럼)가 스핀들에 있어도 공구 번호(`10`)만 냅니다. 포켓 테이블의 스핀들 행이 번호만 담기 때문입니다 (시뮬레이터에서 확인, 조작반은 `10.1` 을 보였습니다).

## /machine/channel/activeToolName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

그 채널에서 지금 활성인 공구의 **이름**입니다. `channel` 필터. 반환 `string`.

**Siemens(`actToolIdent`)와 Heidenhain 이 냅니다.** Fanuc·Mitsubishi 는 상태 `-20` 입니다.

- **Fanuc**: 공구관리 레코드에 이름 칸이 없습니다 (그래서 `/machine/toolArea/toolList` 의 `toolName` 도 Fanuc 에서는 빈 문자열입니다). 공구 형상 크기 데이터에 `/machine/channel/toolOffset/toolName` 이 있지만 그쪽은 **공구 보정 번호**로 색인되는 다른 표라, 활성 공구의 이름으로 풀려면 없는 대응을 지어내야 합니다.
- **Mitsubishi**: 공구관리 표에 이름 칸은 있으나, 그 기종의 `activeToolNumber` 는 `T` 모달(지령된 번호)이라 표의 행과 같다는 보장이 없습니다.
- **Heidenhain**: 스핀들에 있는 공구(포켓 테이블의 스핀들 행)의 공구 테이블 이름입니다. `/machine/toolArea/tool/toolName` 과 같은 칸이라 이름을 바꾸면 곧바로 따라옵니다. 포켓 테이블에도 이름 칸이 있지만 공구 이름을 바꿔도 따라가지 않아 쓰지 않습니다 (테스트 환경에서 확인). **스핀들에 있는 공구에 인덱스 행(`10.1` 처럼)이 있으면 상태 `-22` 입니다.** 인덱스 행마다 이름이 따로인데 포켓 테이블의 스핀들 행은 공구 번호만 담아, 어느 행이 스핀들에 있는지 알 수 없기 때문입니다 (시뮬레이터에서 `10.1` 을 부른 뒤에도 스핀들 행은 `10` 이었고 조작반은 `10.1` 을 보였습니다). 공구 자신의 행(`10`)을 불렀을 때도 같은 까닭으로 `-22` 이고, 인덱스 행이 없는 공구가 스핀들에 오면 다시 이름을 냅니다. 번호는 `activeToolNumber` 가 냅니다.

**`activeToolNumber` 와 짝입니다. 둘 다 있는 이유가 있습니다.** 공구관리(WZV)가 켜진 SINUMERIK 은 파트 프로그램의 `T` 가 **이름**을 가리킵니다. 즉 번호를 받아 `T3` 이라고 쓰면 제어기가 거절합니다 (840D sl 벤치에서 알람 `17190` illegal T number). 프로그램에 그대로 쓸 수 있는 값은 이쪽이고, 우리 공구 트리(`/machine/toolArea/tool/…` 의 `tool` 필터)를 조회할 값은 `activeToolNumber` 입니다.

```
activeToolName   -> "CUTTER 10"   프로그램에 T="CUTTER 10"
activeToolNumber -> 3             toolArea/tool/*?tool=3
```

두 주소는 같은 묶음이라 함께 요청하면 왕복 한 번입니다 (Heidenhain 은 이름을 읽으러 공구 테이블의 행 목록과 그 공구의 행을 더 묻습니다).

**이름만으로는 공구가 유일하지 않을 수 있습니다.** SINUMERIK 의 공구 정체는 이름과 자매공구 번호(duplo)의 쌍이라, 같은 이름의 공구가 여럿일 수 있습니다. 그때 어느 것을 쓸지는 제어기가 정합니다. 하나를 정확히 지목해야 하면 `activeToolNumber` 를 쓰세요.

활성 공구가 없을 때는 **빈 문자열**입니다 (`activeToolNumber` 는 그 상태에서 `0`). 840D sl 벤치에서 채널의 공구를 내려 확인했습니다. 그 자리에는 값이 없는데, 내용이 없는 텍스트를 `null` 로 내지 않는 것이 이 SDK 의 규칙이라 빈 문자열로 맞춥니다. Heidenhain 도 스핀들이 비면 빈 문자열입니다 (그 상태는 확인하지 못했습니다).

## /machine/channel/activeToolEdgeNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens"]
write: []
```

활성 공구에서 지금 **보정이 적용되고 있는 보정 세트**의 번호(`D`)입니다. `channel` 필터. 반환 `int`. Siemens 의 `actDNumber` 입니다. 이 주소가 답하는 것은 "몇 번 보정 세트냐" 이지 "공구가 걸렸냐" 가 아닙니다 (그건 `activeToolNumber` 가 `0` 으로 답합니다).

**Siemens 전용**입니다 (Fanuc·Mitsubishi·Heidenhain 은 상태 `-20`. Heidenhain 은 지금 스핀들에 있는 인덱스 공구의 인덱스를 묻는 길을 디메시가 찾지 못했습니다). Fanuc·Mitsubishi 의 오프셋 모델에는 공구에 딸린 날(보정 세트) 계층이 없어 "몇 번째 날" 이라는 물음 자체가 성립하지 않습니다. 예전 판은 그 두 기종에서 고정 `1` 을 냈는데, 없는 차원을 있다고 답하는 값이라 뺐습니다. Fanuc 에서 프로그램이 부르는 보정 번호는 공구 단위의 `/machine/toolArea/tool/toolHNumber`·`toolDNumber`(공구관리 옵션) 로 읽으세요.

## /machine/channel/activeToolGroupNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다. 이름이 비슷한 공구관리(tool management)와는 **다른 옵션**이며, 지금까지 본 장비에서는 둘이 함께 켜져 있지 않았습니다.

이 채널에서 **지금 수명 카운트가 돌고 있는 공구그룹** 번호입니다. `channel` 필터. 반환 `int`, 읽기 전용입니다.

공구수명관리를 쓰는 장비에서 프로그램이 그룹을 걸면 그 그룹의 수명이 깎이기 시작하는데, 그 그룹의 번호입니다. **쓰는 그룹이 없으면 `0`** 입니다.

`/machine/channel/activeToolNumber` 가 "지금 걸린 공구" 라면 이 값은 "지금 수명이 깎이는 그룹" 입니다. 둘은 다른 개념이라 함께 보셔야 합니다.

**Siemens·Mitsubishi·Heidenhain 은 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

`/machine/channel/activeToolGroupNumber` 와는 **아무것도 안 돌 때 마지막 그룹을 유지하느냐**로 갈립니다:

| | 그룹이 도는 중 | 아무것도 안 돌 때 |
|---|---|---|
| `activeToolGroupNumber` | 그 그룹 | `0` |
| 이 주소 | 그 그룹 (같은 값) | **마지막으로 돌았던 그룹** |

기계가 쉬고 있을 때 "직전에 어느 그룹을 썼나" 를 아는 통로입니다. 전원을 껐다 켜면 초기화되어 `0` 이 됩니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

## /machine/toolArea/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

그 공구 영역에 **등록된 공구의 수**입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기 전용. 등록된 공구가 없으면 `0` 입니다.

**Mitsubishi 는 이 주소가 느립니다** (시뮬레이터 NC Trainer2 plus 에서 2초 남짓. 장비와 통신 환경에 따라 다릅니다). 그 제어기의 공구관리 표는 999칸이고 한 칸씩 물어야 하는데, 지운 자리에 **빈 칸이 남을 수 있어** 중간에서 멈출 수 없기 때문입니다. **주기 폴링에 쓰지 마세요** - 화면을 한 벌 그리는 용도입니다. 공구 하나만 필요하면 `/machine/toolArea/tool/…` 주소가 훨씬 빠릅니다 (그쪽은 찾으면 멈춥니다). 표를 훑는 도중 통신 오류가 나면 거기까지 읽은 결과를 내지 않고 에러로 답합니다.

목록(`toolList`)이 돌려주는 항목 수와 같고 **장비의 같은 값**을 봅니다. 개수만 필요할 때 목록 전체를 받지 않아도 되도록 따로 둔 주소입니다. 공구 17개가 등록된 Siemens 840D sl 벤치에서 재 보면 목록보다 훨씬 빠릅니다 (81ms 대 684ms).

없는 공구 영역을 지정하면 상태 `-18` 로 거절됩니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 옵션이 없으면 "공구" 라는 객체 자체가 없고, 보정 레지스터의 개수는 공구 수와 다른 값이라 대신 쓰지 않습니다. 공구관리 표(조작반 TOOL MANAGER 화면의 `NO.` 행, 칸 수는 파라미터 `13220`)에서 **등록 표시가 선 칸**만 셉니다 (공구 정보의 RGS 비트, 조작반 `T-INFO` 의 마지막 글자 `R`). 등록이 풀린 칸은 값이 남아 있어도 제어기가 무효 데이터로 보므로 세지 않습니다. 칸 수 `13220` 은 옵션의 상한이 아니라 기계 제작사 설정이며(64/240/1000 pairs 옵션 범위 안), 벤치 실측으로 칸 10개 중 등록 5개였습니다.

**Mitsubishi 는 공구관리 표를 읽을 수 없는 구성에서 상태 `-20` 입니다** (표를 쓰지 않는 장비·프로젝트에서 제어기가 읽기를 거절합니다. 에러 문구에 벤더 코드가 함께 실립니다).

**Heidenhain** 은 공구 테이블(조작반의 공구 관리 화면)의 공구 번호를 하나씩 셉니다. 인덱스 공구(`5.1` 처럼 공구 번호 뒤에 붙는 행)는 공구를 늘리지 않고 그 공구의 날로 셉니다 (`/machine/toolArea/tool/toolEdgeCount`). 테이블 맨 앞의 `0` 번 행은 공구가 아닌 자리로 보아 세지 않습니다. `toolArea` 는 `1` 뿐입니다.

## /machine/toolArea/toolList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
field_codes: {"toolLocationType": [{"value": "magazine", "name": "Magazine", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": "buffer", "name": "Spindle or tool changer", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": "loading", "name": "Load/unload position", "read": ["nc_opcua_siemens"]}, {"value": "none", "name": "No physical place", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]}]}
```

그 공구 영역에 **등록된 공구 전부**의 목록입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `objectArray`, 등록된 공구가 없으면 빈 배열 `[]`.

**Mitsubishi 는 이 주소가 느립니다** (시뮬레이터 NC Trainer2 plus 에서 2초 남짓. 장비와 통신 환경에 따라 다릅니다). 그 제어기의 공구관리 표는 999칸이고 한 칸씩 물어야 하는데, 지운 자리에 **빈 칸이 남을 수 있어** 중간에서 멈출 수 없기 때문입니다. **주기 폴링에 쓰지 마세요** - 화면을 한 벌 그리는 용도입니다. 공구 하나만 필요하면 `/machine/toolArea/tool/…` 주소가 훨씬 빠릅니다 (그쪽은 찾으면 멈춥니다). 표를 훑는 도중 통신 오류가 나면 거기까지 읽은 결과를 내지 않고 에러로 답합니다.

항목: `{"toolNumber": 16, "toolTNumber": null, "toolName": "BALLNOSE_D8", "toolEdgeCount": 4, "sisterToolNumber": 9, "magazineNumber": 0, "pocketNumber": 0, "toolLocationType": "buffer", "toolTeethCount": null, "toolBodyLength": null, "toolBodyDiameter": null, "toolOffsetNumber": null}`

공구 번호는 **연속되지 않습니다.** 공구 17개가 2번~18번을 쓰고 1번은 없는 식이라, 번호를 1부터 넣어보는 것으로는 무엇이 있는지 알 수 없습니다. 이 목록이 그 답이며, 항목의 `toolNumber` 를 그대로 `tool` 필터에 넣어 공구별 주소를 조회하는 것이 용법입니다.

**순서는 기종에 따라 다르되, 매번 같은 순서가 보장됩니다.** Fanuc·Siemens·Heidenhain 은 `toolNumber` 오름차순입니다. Siemens 의 공구 목록 화면은 보통 이름순이고 작업자가 정렬 기준을 바꿀 수 있어 맞출 수 있는 하나의 "화면 순서" 가 없으므로 번호순으로 고정했습니다 (화면과 같은 순서로 보여주려면 `toolName` 으로 정렬하세요). **Mitsubishi 는 공구관리 표의 행 순서 그대로**입니다. 그 기종의 화면은 표 행 순서가 그대로 보이므로 이쪽이 화면과 일치하며, 지운 행이 나중에 새 공구로 채워지면 번호 오름차순이 아닐 수 있습니다.

**`toolNumber` 는 장비 화면의 `Loc.`(자리 번호)이 아닙니다.** 공구 관리를 쓰는 장비에서는 공구를 이름과 자매번호로 식별하므로 이 번호가 목록 화면에 나오지 않습니다 (공구 상세 화면의 `Tool number` 항목이 이 값입니다). 화면의 `Loc.` 을 `tool` 필터에 넣으면 **다른 공구를 조회하고도 성공으로 보입니다.** 두 번호가 우연히 같은 공구가 많아 알아채기 어렵습니다. 그 값은 `pocketNumber` 이며, 이 목록이 둘을 함께 담고 있어 대응을 확인할 수 있습니다.

- **toolNumber**: 공구 번호. `tool` 필터에 넣는 값
- **toolTNumber**: 프로그램이 `T` 로 이 공구를 부르는 번호 (Fanuc 공구관리의 `TYPE NO.`, 여러 공구가 같은 값을 가질 수 있음). 이름으로 부르는 Siemens 와, 이 개념이 없는 Mitsubishi 는 `null`
- **toolName**: 공구 이름. 이름을 쓰지 않는 Fanuc 은 빈 문자열, 디메시가 표에서 이름을 읽지 않는 Mitsubishi 는 `null`
- **toolEdgeCount**: 보정 세트 개수 (인선 수가 아니고, **가장 큰 D 번호도 아닙니다**. 중간 삭제로 구멍이 나면 번호가 개수보다 클 수 있습니다: `/machine/toolArea/tool/toolEdgeCount` 참조)
- **sisterToolNumber**: 자매공구 번호 (이름이 같은 공구들을 구분하는 번호. 조작반의 `ST` 열)
- **magazineNumber**: 지금 꽂혀 있는 매거진(공구 저장고) 번호. 매거진 밖이면 `0`
- **pocketNumber**: 그 매거진 안의 포켓 번호. 매거진 밖이면 `0`
- **toolLocationType**: 자리의 종류. `"magazine"`(매거진에 있음) · `"buffer"`(스핀들 또는 교환기) · `"loading"`(반입·반출 위치) · `"none"`(실물 자리 없음)
- **toolTeethCount** · **toolBodyLength** · **toolBodyDiameter** · **toolOffsetNumber**: Mitsubishi 공구관리 표의 공구 단위 열 (뜻은 같은 이름의 단독 주소 참조). Fanuc·Siemens·Heidenhain 은 `null` 입니다 (Siemens·Heidenhain 의 날 수는 날마다 따로라 `/machine/toolArea/tool/toolEdge/toolTeethCount` 가 답합니다)

이 목록은 **무엇이 있고 · 어떻게 부르고 · 어디 있나** 까지 답합니다. 오프셋·마모 같은 측정값은 보정 세트 단위라 담지 않습니다.

값이 없으면 키를 빼지 않고 `null` 입니다 (위치 세 필드는 위치를 아는 기종에서는 예외로 같은 이름의 단독 주소와 같은 값을 내고, 위치를 볼 수 없는 Mitsubishi 에서는 `null` 입니다). **매번 같은 순서**로 돌려주므로 두 번 읽어 비교하는 것이 의미를 갖습니다. 없는 공구 영역을 지정하면 상태 `-18` 로 거절됩니다.

**위치 세 필드는 공구가 움직일 때마다 바뀌고**, 나머지 필드는 잘 바뀌지 않습니다. 이 목록은 화면을 그릴 때 한 벌 받아오는 용도이며, 지금 활성인 공구만 알고 싶다면 목록을 반복해 읽는 대신 `/machine/channel/activeToolNumber` 를 쓰세요 (`activeToolNumber` 는 모든 기종이 지원하며, Siemens·Heidenhain 에서 그 값은 교환이 끝난 공구이고, 공구관리 옵션이 켜진 Fanuc 에서는 이 목록의 `toolNumber` 가 아니라 공구의 타입 번호입니다).

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 표에서 등록 표시(공구 정보의 RGS 비트, 조작반 `T-INFO` 끝의 `R`)가 선 칸만 담고, `toolNumber` 는 그 칸의 번호(조작반 `NO.` 열, 파라미터 `13220` 까지)입니다. `toolName` 은 빈 문자열이고, `toolEdgeCount` 와 `sisterToolNumber` 는 Siemens 공구 관리의 개념(날 계층·자매공구)이라 Fanuc 에서는 `null` 입니다. 프로그램이 `T` 로 부르는 번호는 이 번호가 아니라 공구의 **타입 번호**(조작반 `TYPE NO.`)이며 항목의 `toolTNumber` 가 그것입니다. 위치 세 필드는 단독 주소와 같은 값입니다 (`1`~`8` 매거진, 스핀들·대기 위치는 `"buffer"`, 미장착은 `"none"`. `/machine/toolArea/tool/toolLocationType` 참조). 옵션이 없는 Fanuc 은 오프셋 테이블이 `1` 부터 촘촘히 채워져 있어 열거할 대상이 없습니다.

**Mitsubishi 는 공구관리 표의 등록 행**을 담습니다. 표는 계통(`toolArea`)마다 따로입니다 (기계 전체에 딸린 매거진과 다릅니다. 시뮬레이터에서 확인). 공통 키 중 이 기종이 갖지 않거나 디메시가 읽지 않는 개념(`toolTNumber`·`toolName`·`toolEdgeCount`·`sisterToolNumber`, 그리고 공구 레코드가 자기 위치를 담지 않아 위치 세 필드)은 `null` 이고, 반대로 표의 열 넷(`toolTeethCount`·`toolBodyLength`·`toolBodyDiameter`·`toolOffsetNumber`)은 이 기종만 값을 채웁니다 (제어기가 거절한 열도 키는 남고 `null` 입니다. 통신 오류는 `null` 이 아니라 에러로 돌아옵니다). 공구가 어느 포켓에 있는지는 이 목록이 아니라 `/machine/toolArea/magazine/pocket/toolNumber` 로 매거진 쪽에서 확인하세요.

**Mitsubishi 는 공구관리 표를 읽을 수 없는 구성에서 상태 `-20` 입니다** (표를 쓰지 않는 장비·프로젝트에서 제어기가 읽기를 거절합니다. 에러 문구에 벤더 코드가 함께 실립니다).

**Heidenhain** 은 공구 테이블(조작반의 공구 관리 화면)의 공구를 담습니다. `toolNumber` 는 테이블의 공구 번호이고 `toolTNumber` 도 같은 값입니다 (프로그램의 `TOOL CALL` 이 이 번호로 부릅니다. 기계 설정에 따라 이름으로도 부를 수 있다고 TNC7 사용 설명서 'Tool call by TOOL CALL' 이 적습니다). `toolName` 은 공구 이름(`NAME`), `toolEdgeCount` 는 그 공구의 행 수(공구 자신의 행에 인덱스 공구 행을 더한 수, `/machine/toolArea/tool/toolEdgeCount` 참조)입니다. `sisterToolNumber` 는 `null` 이고, 대체 공구는 `/machine/toolArea/tool/sisterTool` 이 답합니다. 위치 세 필드는 포켓 테이블에서 옵니다 (`/machine/toolArea/tool/toolLocationType` 참조). Mitsubishi 공구관리 표의 열 넷은 `null` 입니다. 테이블 맨 앞의 `0` 번 행은 공구가 아닌 자리로 보아 담지 않습니다. 테이블 전체를 한 번에 읽으며, 시뮬레이터(공구 222개)에서 0.5초 안팎이었습니다.

## /machine/toolArea/tool/toolExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

그 공구 번호가 **공구표에 등록되어 있는지** 여부입니다. `toolArea` + `tool` 필터. 반환 `boolean`. 읽기는 네 기종 모두, 쓰기는 Heidenhain 을 뺀 세 기종이 지원합니다.

없는 공구를 물어도 에러가 아니라 `false` 입니다. 존재 여부를 묻는 주소이기 때문입니다. 다만 Siemens 에서 공구가 없어서가 아니라 다른 이유로 읽지 못하면(연결 계정에 공구 데이터 읽기 권한이 없어 `BadUserAccessDenied` 가 오는 경우 등) 상태 `-17` 이고, 쓰기도 상태 `-17` 입니다.

**쓰기가 공구를 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. **이미 있는 공구에 `true` 를 쓰면 상태 `-21`(이미 존재)로 거절합니다.** 조용히 성공시키면 빈 새 공구인 줄 알고 남의 수명·오프셋·자리 위에 값을 이어 쓰게 되기 때문입니다. 지우고 다시 만들거나, 그대로 각 주소로 값을 쓰거나, 건너뛰세요. 없는 공구에 `false` 를 쓰는 것은 성공입니다 (사후 조건 "없다" 가 그대로 성립하고, 응답을 못 받아 다시 보내도 안전합니다).

**공구 번호는 `tool` 필터에 넣은 값 그대로입니다.** 장비가 다음 번호를 자동으로 붙여 주지 않습니다. 번호 공간에는 구멍이 있고(840D sl 벤치는 공구 20개가 `2`~`18` 과 `100`~`102` 를 쓰며 `1` 번과 `19`~`99` 가 비어 있습니다) **그 빈 번호를 지정해 채울 수 있습니다.** 어느 번호가 비었는지는 `/machine/toolArea/toolList` 의 `toolNumber` 들을 보고 고르세요. 공구를 지우면 그 번호가 다시 비고, 나중에 같은 번호로 다시 만들 수 있습니다.

Siemens 에서 만들어지는 공구는 **날 1개짜리 빈 공구**입니다. 이름은 공구 번호 문자열이고, **자리 종류**는 표준값으로 채워집니다. 자리 종류는 그 공구가 들어갈 수 있는 매거진 자리를 정하는 값이라 비워 두면 조작반이 적재할 자리를 찾지 못하므로, 디메시가 조작반이 새 공구에 넣는 값과 같게 채웁니다. **사용 허가**도 조작반처럼 켠 채로 만듭니다 (허가가 없는 공구는 제어기가 고르지 않아 `T` 지령이 알람으로 거절됩니다. 840D sl 벤치에서 확인). 공구 타입은 채우기 전까지 `9999`(미지정)이고 조작반도 종류 칸을 비워 둔 것으로 보여 주는데, **자리 종류와 달리 적재나 사용을 막지는 않습니다** (두 칸 모두 `9999` 를 쓰기 때문에 혼동하기 쉽습니다). 이어서 `/machine/toolArea/tool/toolName` 으로 이름을, `/machine/toolArea/tool/toolEdge/*` 로 오프셋을 넣고, 날을 더 붙이려면 `/machine/toolArea/tool/toolEdge/toolEdgeExists` 를 쓰세요. 공구 준비실에서 잰 값을 조작반을 거치지 않고 그대로 등록하는 흐름이 이것입니다. 만들기가 거절되면 공구 수가 최대에 이르렀을 때 상태 `-23`(자리 없음), 제어기가 그 공구 번호를 받지 않을 때 상태 `-18` 입니다. 공구를 만든 뒤 마무리 단계(자리 종류나 사용 허가 설정)가 실패하면 디메시가 방금 만든 공구를 지우고 에러를 돌려줍니다 (지우지 못하면 공구가 남았다는 것과 지우는 방법을 에러 문구에 싣습니다).

**Siemens·Fanuc 에서는 어딘가에 실려 있는 공구를 지울 수 없습니다** (상태 `-18`). 매거진 포켓뿐 아니라 스핀들·그리퍼 같은 버퍼 자리도 해당하고, Siemens 는 반입출 위치, Fanuc 은 대기 위치도 같습니다. 실물은 매거진에 남는데 등록만 사라지면 다음 공구 교환이 어긋나고, 되돌리려 해도 **디메시에는 공구를 그 포켓에 다시 배정하는 주소가 없습니다.** 먼저 조작반에서 공구를 빼세요. 지금 어디에 있는지는 `/machine/toolArea/tool/toolLocationType` 이 답합니다. **Siemens 에서 채널이 Reset 이 아니거나 공구가 사용 중이라 삭제가 거절되면 디메시는 상태 `-22`(기계 상태)로 답합니다** (840D sl 벤치: 방금 스핀들에 올렸다 내린 공구를 채널이 Interrupted 인 채 지우려 할 때). 채널을 리셋하고 다시 지우세요. 그 밖의 거절은 대개 상태 `-17` 이고, 에러 문구에 반환 코드가 실립니다 (매뉴얼에 뜻이 있는 코드는 그 뜻도). **Mitsubishi 에서는 디메시가 이 검사를 하지 않습니다** (그 기종은 공구 레코드가 자기 위치를 담지 않아, 매거진 전체를 훑어야 알 수 있습니다). 매거진을 쓰는 장비라면 지우기 전에 `/machine/toolArea/magazine/pocket/toolNumber` 로 그 공구가 어느 포켓에 있는지 직접 확인하세요.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 그 칸의 등록 표시(공구 정보의 RGS 비트, 조작반 `T-INFO` 끝의 `R`)입니다. 파라미터 `13220`(칸 수)을 넘는 번호도 읽기는 에러가 아니라 `false` 입니다 (다만 `32767` 을 넘는 번호는 상태 `-13`, `0` 이하는 상태 `-18` 입니다). 등록이 풀린 칸은 값이 남아 있어도 제어기가 무효로 보므로(Connection Manual B-64483EN-1 이 RGS 가 0 이면 다른 항목에 값이 있어도 미등록으로 취급한다고 밝힘), 그 칸에 대한 다른 공구 주소(`toolHNumber`·수명·위치 등)는 상태 `-18` 로 거절됩니다 (벤치 실측: 등록 표시만 켜면 같은 칸이 곧바로 `true` 가 되고 `toolCount` 도 하나 늘었습니다).

Mitsubishi 의 표는 **행의 목록**이라 공구 번호가 곧 행이 아닙니다. `true` 는 **첫 번째 빈 행**에 그 번호를 써 넣고, `false` 는 그 행을 비웁니다. 행 번호는 주소 표면에 나오지 않으므로 어느 자리에 들어가는지 신경 쓸 필요가 없습니다. 만들어진 공구는 날 수·치수가 `0` 이고 **보정 번호만 공구 번호와 같은 값으로 제어기가 채워 줍니다** (시뮬레이터에서 확인). 나머지는 각 주소로 채우세요. 지운 행은 제어기가 통째로 비우므로(조작반의 `공구 클리어` 와 같습니다) 나중에 그 자리에 만들어진 공구가 옛 값을 물려받지 않습니다.

**Mitsubishi 에서 만들기와 없는 공구 지우기는 표 전체를 훑습니다** (시뮬레이터에서 2초 남짓). 표가 999행이고 중간을 지우면 구멍이 남아, "이 번호가 없다" 를 증명하려면 끝까지 봐야 하기 때문입니다. 있는 공구를 지우는 것은 그 행을 찾으면 멈춥니다 (시뮬레이터에서 0.2초가량). **되풀이해 부르는 용도가 아닙니다.** 중복 번호는 제어기도 거절하지만 디메시가 먼저 보고 상태 `-21` 로 답합니다. 표에 빈 행이 하나도 없으면 상태 `-23`(자리 없음)으로 거절합니다. 값이 잘못된 것이 아니라 넣을 자리가 없는 것이므로, 공구 하나를 지워 자리를 비운 뒤 다시 요청하세요. 표를 읽는 사이 다른 쪽(조작반 등)이 그 빈 행을 차지하면 아무것도 쓰지 않고 상태 `-24`(바쁨)로 답하니, 같은 요청을 다시 보내세요. 표를 훑는 도중 통신 오류가 나면 읽기는 `false` 가 아니라 에러로, 쓰기는 아무것도 쓰지 않고 에러로 답합니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`)이 꺼진 장비에서 등록·삭제가 거절되면 디메시가 그 신호를 읽어 상태 `-22`(기계 상태)로 답합니다 (키를 끄면 등록·삭제가 거절되는 것을 시뮬레이터에서 확인했습니다).

Fanuc 의 쓰기: `true` 는 그 칸을 **등록**합니다 (`cnc_regtool`). 만들어지는 칸은 **등록 표시만 켜진 빈 레코드**(타입 번호 `0`, 수명 관리 안 함, H/D/S/F `0`)라 어떤 `T` 지령에도 걸리지 않습니다. 제어기가 대신 넣어 주는 기본값은 없고 디메시도 지어내지 않으니, 이어서 `toolTNumber`·`toolHNumber`·`toolDNumber`·`toolLifeMonitorType`·수명 주소로 채우세요. 등록이 풀렸는데 값이 남은 칸(조작반 `T-INFO` 가 `-` 인데 다른 열에 값이 보이는 칸)은 제어기가 그대로 등록을 거절하므로 디메시가 먼저 비우고(`cnc_deltool`) 등록합니다. 남은 값은 버려집니다. `13220` 을 넘는 칸은 만들 수 없어 상태 `-18` 입니다. `false` 는 그 칸을 **삭제**합니다 (`cnc_deltool`): 레코드가 통째로 비워지고, 제어기가 매거진 관리표에서도 그 공구 번호를 지웁니다 (Connection Manual B-64483EN-1). 공구 데이터 잠금(`toolDataLockedOn`)은 이 삭제를 막지 않습니다 (벤치 실측). 지운 칸 뒤의 칸은 당겨지지 않고 그대로입니다 (31i 벤치에서 확인). `13220` 밖에 `false` 를 쓰는 것은 읽기와 같이 에러가 아닙니다.

**Mitsubishi 는 공구관리 표를 읽을 수 없는 구성에서 상태 `-20` 입니다** (표를 쓰지 않는 장비·프로젝트에서 제어기가 읽기를 거절합니다. 에러 문구에 벤더 코드가 함께 실립니다).

**Heidenhain** 은 공구 테이블(조작반의 공구 관리 화면)에 그 번호의 행이 있는지입니다. 읽기만 지원하며 쓰기는 상태 `-20` 입니다 (디메시는 Heidenhain 에서 공구를 만들거나 지우지 않습니다). 테이블 맨 앞의 `0` 번 행은 공구가 아닌 자리로 보아 `false` 입니다. `toolArea` 는 `1` 뿐입니다.

## /machine/toolArea/tool/toolName
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

공구의 이름입니다 (SINUMERIK `toolIdent`). `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호).

반환 `string`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": "DRILL 10"}`. **Siemens·Heidenhain** 에서 지원합니다. Siemens 에서 공구 관리 기능을 쓰는 장비에서는 이름과 자매공구 번호(duplo)의 조합이 공구의 정체이므로 같은 이름을 가진 공구가 여럿 있을 수 있습니다. 이름을 쓰지 않는 장비에서는 빈 문자열이 정상입니다.

없는 공구를 지정하면 상태 `-18` 로 거절됩니다. 이 주소는 공구를 만들지 않습니다. 이름의 길이·문자 제약은 장비가 판단하며 위반하면 에러가 돌아옵니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 디메시는 두 기종의 공구관리 표에서 이름을 읽지 않습니다 (Fanuc 의 공구 이름은 공구 형상 크기 데이터에 있어 `/machine/channel/toolOffset/toolName` 이 답합니다).

**Heidenhain** 은 공구 테이블의 공구 이름(`NAME`)이고 쓰기도 지원합니다. 제어기가 받지 않는 이름은 상태 `-16` 입니다. TNC7 사용 설명서('Tool name')는 32자까지, 영문 대문자·숫자와 `#` `$` `%` `&` `,` `-` `_` `.` 를 쓸 수 있고 소문자는 저장할 때 대문자로 바뀐다고 적습니다. 테스트 환경에서도 33자 이상, 공백(`DRILL 10`), `/`·`:`, 한글·움라우트가 든 이름은 거절됐고 `abc_def` 는 `ABC_DEF` 로 저장됐습니다. 위의 쓰기 예시는 공백이 있어 Heidenhain 에서는 `DRILL_10` 처럼 쓰세요. 이름은 공구마다 유일하지 않을 수 있습니다 (설명서. 공구는 번호로 지목하세요). 인덱스 공구(`5.1` 처럼 공구 번호 뒤에 붙는 행)의 이름은 이 주소가 다루지 않습니다. 테이블 맨 앞의 `0` 번 행은 공구가 아닌 자리로 보아 상태 `-18` 입니다.

## /machine/toolArea/tool/toolUseStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
codes: [{"value": 0, "name": "not managed", "read": ["nc_focas2_fanuc"]}, {"value": 1, "name": "unused"}, {"value": 2, "name": "in use"}, {"value": 3, "name": "life expired"}, {"value": 4, "name": "broken", "read": ["nc_focas2_fanuc"]}, {"value": 5, "name": "locked", "read": ["nc_opcua_siemens", "nc_dnc_heidenhain"]}]
```

그 공구의 **사용 상태**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int` + `desc`. 읽기·쓰기 모두 Siemens·Fanuc·Heidenhain 이 지원합니다. **공구 단위**라 `toolEdge` 필터를 받지 않습니다 (Siemens 는 보정 세트가 여럿인 공구도 상태가 공구 전체에 걸립니다. Heidenhain 의 인덱스 공구는 행마다 따로라 `/machine/toolArea/tool/toolEdge/toolUseStatus` 가 답합니다).

값은 디메시가 정한 기종 무관 코드입니다 (벤더 번호가 아닙니다). 각 값은 **그 공구가 지금 어떤 상태인가**로 정의하고, 기종이 달라도 같은 상태일 때만 같은 값을 냅니다:

| 값 | 뜻 |
|---|---|
| `0` | 수명 관리 밖: 수명 상태가 "관리 안 함" 이라 타입 번호 검색에서 빠지는 공구 (Fanuc 전용. `/machine/toolArea/tool/toolSearchedWhenUnmanagedOn` 이 그 예외) |
| `1` | 미사용: 아직 절삭한 적 없고 잠기지 않음 |
| `2` | 사용 중: 쓰인 적 있고 잠기지 않음 |
| `3` | 수명 초과: 수명이 소진되어 제어기가 쓰지 않음 |
| `4` | 파손: 파손으로 제어기가 쓰지 않음 (Fanuc 전용) |
| `5` | 잠금: 수명 말고 다른 이유로 쓸 수 없음. 수명이 남았는데 잠겨 있거나(조작반·PLC·NC 프로그램 등), 사용 허가가 없어 제어기가 고르지 않는 공구 (Siemens·Heidenhain) |

`3`·`4`·`5` 면 제어기가 그 공구를 쓰지 않습니다. 프로그램이 부르면 거절하거나, 자매공구가 등록되어 있으면 그쪽으로 넘어갑니다 (`/machine/toolArea/tool/sisterToolNumber`, Heidenhain 은 대체 공구 `/machine/toolArea/tool/sisterTool`). 한 기종에서만 나오는 값이 있어도 그 값이 나올 때의 뜻은 같습니다. `desc` 는 사람이 읽는 문구라 분기는 값으로 하세요.

**Fanuc** 은 공구관리 데이터의 수명 상태(조작반 `L-STATE`)를 그대로 옮깁니다: 관리 안 함 `0` · 미사용 `1` · 사용 가능 `2` · 수명 초과 `3` · 파손 `4`. 작업자가 조작반에서 손으로 잠근 공구도 Fanuc 제어기는 `수명 초과`(`3`)로 부르므로 `5` 는 나오지 않습니다. 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만 지원하며, 없으면 상태 `-20` 입니다. 공구 정보의 LOC 비트(`/machine/toolArea/tool/toolDataLockedOn`)는 데이터 편집 잠금이라 여기 섞지 않습니다.

**Siemens** 는 공구 상태 비트(`toolState`)와 잔여 수명에서 도출합니다: 잠금 비트(Disabled)가 꺼져 있으면 "쓰인 적 있음" 비트에 따라 `1`/`2` 이되 사용 허가(Enabled) 비트까지 꺼져 있으면 `5`(제어기가 고르지 않는 공구입니다), 잠금 비트가 켜져 있으면 수명 감시 중이고 어느 날이든 잔여 수명이 `0` 이하면 `3`, 아니면 `5` 입니다. 잠금 비트는 하나라 누가 왜 잠갔는지는 남지 않으므로, `5` 는 "수명 때문이 아닌 잠금" 까지만 말합니다. 잠긴 공구는 잔여 수명을 읽기 위해 왕복이 보통 한 번 더 듭니다. 날 번호에 구멍이 있어도(`D2` 를 지워 `D1`·`D3` 만 남은 경우 등) 실재하는 날을 찾아 모두 봅니다. 날 번호 `1`~`240` 안에서 날 개수만큼을 찾지 못하면 값을 지어내지 않고 상태 `-17` 입니다. SINUMERIK 의 공구 상태에는 파손 구분이 없어 `4` 는 나오지 않고, 수명 감시가 꺼진 공구도 선택되므로 `0` 도 나오지 않습니다.

**쓰기**는 원하는 상태를 값으로 지정합니다. 이미 그 상태면 아무것도 하지 않고 성공합니다 (그 기종이 쓰기로 받는 값일 때. 예외: Siemens 에서 사용 허가가 없어서만 `5` 로 읽히는 공구에 `5` 를 쓰면 잠금 비트가 켜집니다. Heidenhain 에서 사용 시간이 `TIME2` 에 닿은 공구는 이미 그 상태여도 상태 `-16` 입니다). 기종마다 쓸 수 있는 값이 다릅니다:

- Fanuc: `1`~`4` 를 수명 상태에 씁니다 (`cnc_wrtool2`). `3` 으로 바꾸면 같은 타입 번호의 공구가 전부 수명 초과가 되는 순간 공구 교환 신호(`TLCH`)가 켜질 수 있습니다. 수명 상태가 "관리 안 함" 인 공구는 상태 `-18` 로 거절합니다 (먼저 `/machine/toolArea/tool/toolLifeMonitorType` 을 `1`/`2` 로). `0` 은 `toolLifeMonitorType` 으로 다루고, `5` 는 Fanuc 에 없는 상태라 상태 `-16` 입니다 (`3` 을 쓰세요).
- Siemens: `5` 는 잠금 비트를 켜고, `1`/`2` 는 잠금 비트를 끄면서 "쓰인 적 있음" 비트를 각각 끄고 켜며 사용 허가 비트를 켭니다. `3` 은 제어기가 잔여 수명에서 도출하는 사실이라 직접 쓸 수 없어 상태 `-16` 입니다 (`/machine/toolArea/tool/toolEdge/toolLifeRemaining` 을 `0` 으로 쓰거나, 잠그려면 `5`). `4`·`0` 도 상태 `-16` 입니다. **수명 값을 쓰면 잠금이 다시 매겨집니다**: SINUMERIK 은 감시 값(`toolLifeTotal`·`toolLifeRemaining`·`toolLifeWarnLimit`)이 바뀌면 공구 상태를 다시 매기므로(Siemens 공구관리 기능 매뉴얼 §8.11), `5` 로 잠근 공구도 수명 값을 쓰면 풀립니다. 잠금을 유지하려면 쓴 뒤 `5` 를 다시 쓰세요 (벤치에서 확인). 사용 허가가 없어서만 `5` 인 공구도 수명 값을 쓰면 풀리는지는 확인하지 못했습니다.
- Heidenhain: `5` 는 공구 테이블의 잠금(`TL`)을 켜고, `1`·`2` 는 잠금을 풉니다. 풀린 공구는 쓴 시간이 있으면 `2`, 없으면 `1` 로 읽히므로 그 상태와 맞는 값만 받고, 맞지 않으면 상태 `-16` 입니다 (미사용으로 되돌리려면 먼저 `/machine/toolArea/tool/toolLifeUsed` 에 `0` 을 쓰세요). `3` 은 수명에서 매기는 값이라, `0`·`4` 는 이 제어기에 없는 상태라 상태 `-16` 입니다. 사용 시간이 `TIME2` 에 닿은 공구는 잠금과 무관하게 `3` 으로 읽히므로 `1`·`2`·`5` 도 상태 `-16` 입니다 (먼저 사용 시간을 고치세요). 최대 수명(`TIME1`)을 넘긴 공구에 `5` 를 쓰면 잠금이 걸리고 수명 초과 `3` 으로 읽힙니다 (잠긴 공구의 수명이 다했으므로). 테스트 환경에서 잠그자 조작반 공구 테이블의 그 행이 곧바로 잠김 표시로 바뀌었습니다.

**수명이 다해 `3` 이 된 공구를 `2` 로만 되돌리면 잔여 수명(Fanuc 은 카운터)은 그대로입니다.** 그래서 제어기가 다시 수명을 볼 때 `3` 으로 돌아갈 수 있습니다 (SINUMERIK 벤치에서는 `2` 를 쓰고 3초 뒤까지 `2` 였습니다). 인서트를 갈았다면 수명을 되돌리세요: Fanuc 은 `/machine/toolArea/tool/toolLifeUsed` 를 먼저 되돌리고, Siemens 는 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 만 되돌리면 잠금도 함께 풀립니다. 인서트를 갈지 않은 채 상태만 되돌리는 것은 다 쓴 날로 깎는다는 뜻이기도 합니다.

없는 공구를 지정하면 상태 `-18` 로 거절됩니다.

이 주소는 1.1.0 의 `/machine/toolArea/tool/toolDisabledOn`(`boolean`)을 대체합니다. 옛 주소는 상태 `-12` 로 거절되며, `toolDisabledOn` 이 `true` 였던 공구는 이 주소의 `3`·`4`·`5` 에 해당합니다.

**Mitsubishi 는 상태 `-20` 입니다.**

**Heidenhain** 은 공구 테이블의 잠금(`TL`)과 수명 칸(최대 수명 `TIME1`, 공구를 부를 때의 한계 `TIME2`, 사용 시간 `CUR_TIME`)에서 도출합니다. 사용 시간이 `TIME2` 에 닿았으면 잠금과 무관하게 `3` 입니다: 테스트 환경에서 그 공구를 부르자 제어기가 "공구 수명이 종료됨" 오류로 넣지 않았고 (사용 시간이 `TIME2` 와 같을 때도 그랬습니다), 공구 테이블의 잠금은 켜지지 않았습니다 (TNC7 사용 설명서 'Tool table tool.t' 도 `TIME2` 를 넘은 공구는 부를 때 넣지 않는다고 적습니다). 그 밖에는 잠겨 있으면 최대 수명이 있고 사용 시간이 그에 이르렀을 때 `3`, 아니면 `5` 이고, 잠겨 있지 않으면 사용 시간이 있으면 `2`, 없으면 `1` 입니다. `0`·`4` 는 나오지 않습니다. **`TIME1` 을 넘긴 것은 제어기의 잠금을 따릅니다**: 사용 시간이 최대 수명을 넘었어도 잠기기 전에는 `2` 입니다 (테스트 환경에서는 넘긴 공구도 그대로 넣었습니다. 설명서는 이 동작이 기계에 따라 다르다고 적습니다). 설명서에 따르면 제어기는 자동 공구 측정의 허용치를 넘은 공구도 잠그는데, 그 까닭은 공구 테이블에 남지 않아 `5` 로 나옵니다. 수명을 넘겼는지는 `/machine/toolArea/tool/toolLifeUsed` 와 `/machine/toolArea/tool/toolLifeTotal` 을 비교해 가려내세요. 쓰기는 위 목록에 있습니다. 이 주소는 공구 자신의 행입니다. 인덱스 공구(`320.1` 처럼)는 행마다 잠금과 수명이 따로 있어 `/machine/toolArea/tool/toolEdge/toolUseStatus` 가 답합니다 (`toolEdge=0` 은 이 주소와 같은 값).

## /machine/toolArea/tool/toolTNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc"]
```

프로그램이 **`T` 로 이 공구를 부르는 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`. 읽기는 Fanuc·Heidenhain, 쓰기는 Fanuc 이 지원합니다. 쓰기는 `{"value": 10}`.

`T10 M06` 의 그 `10` 입니다. `/machine/toolArea/tool/toolHNumber`(`H`)·`toolDNumber`(`D`) 와 같은 식구로, 프로그램의 글자에 대응하는 번호입니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 공구관리 데이터의 타입 번호(조작반 TOOL MANAGER 화면의 `TYPE NO.`)입니다. **공구 번호가 아니라 묶음 꼬리표입니다**: 여러 공구가 같은 번호를 가질 수 있고, 프로그램이 그 번호를 부르면 제어기가 그 번호를 가진 공구 중 잔여 수명이 가장 적은 유효한 공구를 골라 씁니다 (같으면 스핀들 위치, 대기 위치, 매거진 순, 그다음 공구 번호가 작은 것. Connection Manual B-64483EN-1). 테스트 벤치는 공구 `1` 이 `4`, 공구 `2`~`5` 가 전부 `10` 이었습니다. 공구마다 다른 번호를 매긴 장비에서는 공구 번호처럼 보이지만 그건 운용 방식일 뿐입니다. 쓰기는 이 공구의 타입 번호를 바꿉니다 (`0`~`99999999` 의 정수, 밖이면 상태 `-16`). 그 번호를 가진 다른 공구들과 묶이거나 풀리므로 프로그램의 `T` 가 고르는 후보가 달라집니다. 등록되지 않은 공구는 상태 `-18` 입니다.

**`/machine/channel/activeToolNumber` 와 짝입니다.** 공구관리 장비에서 그 주소가 내는 값은 이 번호이므로, `/machine/toolArea/toolList` 에서 `toolTNumber` 가 그 값과 같은 항목들이 후보 공구입니다. 어느 것이 실제로 스핀들에 물렸는지는 `/machine/toolArea/tool/toolLocationType` 의 `"buffer"` 로 좁힐 수 있습니다.

공구의 **종류**(드릴·엔드밀 등)가 아닙니다. 종류는 Fanuc 에서는 `/machine/channel/toolOffset/toolType`(별도 옵션인 공구 형상 크기 데이터), Siemens 에서는 `/machine/toolArea/tool/toolEdge/toolType` 입니다.

**Heidenhain** 은 공구 테이블의 공구 번호 그대로입니다 (`TOOL CALL 10` 의 그 `10`, `/machine/toolArea/toolList` 원소의 `toolTNumber` 와 같은 값). Fanuc 과 달리 공구마다 하나뿐인 번호이고, 번호가 곧 테이블의 행이라 쓰기는 상태 `-20` 입니다. `/machine/channel/activeToolNumber` 도 같은 번호를 냅니다. TNC7 사용 설명서('Tool call by TOOL CALL')에 따르면 기계 설정에 따라 프로그램이 공구를 이름으로 부를 수도 있는데, 이름은 여러 공구가 같을 수 있으니('Tool name') 공구를 가리킬 때는 이 번호를 쓰세요. 인덱스 공구(`10.1` 처럼)도 이 주소는 공구 자신의 번호를 냅니다 (인덱스는 `toolEdge` 로 가립니다). 없는 공구와 테이블 맨 앞의 `0` 번 행은 상태 `-18` 입니다 (테스트 환경에서 확인).

**Siemens 는 상태 `-20` 입니다.** 프로그램이 공구를 이름으로 부르는 제어기라 같은 자리는 `/machine/toolArea/tool/toolName` 입니다.

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

그 공구가 쓰는 **보정 번호**입니다 (**Mitsubishi 전용**. 공구관리 표의 열이라 다른 세 기종은 상태 `-20` 입니다). `toolArea` + `tool` 필터. 반환 `int`, 읽기와 쓰기가 됩니다.

**공구 번호로 그 공구의 보정값에 닿는 고리입니다.** 이 값을 `/machine/channel/toolOffset/…` 의 `toolOffset` 필터에 그대로 넣으면 형상·마모·노즈R을 읽을 수 있습니다.

```
toolList -> 공구 5  ->  toolOffsetNumber?tool=5 -> 5  ->  toolOffset/toolXGeometry?toolOffset=5
```

**공구 번호와 다를 수 있습니다.** 우연히 같은 장비가 많지만 별개 값입니다 (시뮬레이터에서 확인: 공구번호를 `7` 로 바꿔도 보정 번호는 `5` 로 남았습니다).

⚠️ **쓰면 그 공구에 적용되는 보정값이 통째로 바뀝니다.** 값 하나를 고치는 것이 아니라 어느 보정을 볼지를 갈아 끼우는 것이라, 가공 중인 공구에 쓰면 그 자리부터 다른 치수로 움직입니다. `0` 도 받습니다 (제어기가 받아들이는 값이며, 뜻은 그 기계의 설정을 따릅니다). 상한은 `/machine/channel/toolOffsetCount` 이고 그보다 큰 번호는 제어기가 거절해 상태 `-16`(잘못된 쓰기 값)이 나갑니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터를 보호합니다)이 꺼져 있으면 제어기가 이 칸의 쓰기를 거절하고 상태 `-22`(기계 상태)가 나가니 키를 켠 뒤 다시 쓰세요 (시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다. 같은 표의 `toolBodyLength`·`toolBodyDiameter` 는 키가 꺼져도 받아들여졌습니다). 없는 공구 번호는 상태 `-18`(잘못된 필터 값)입니다.

Mitsubishi 조작반의 공구관리 표에는 보정 열이 두 벌인데(`X5` / `Y5`) **한 번 쓰면 두 열이 함께 바뀝니다** (시뮬레이터에서 확인). 그 화면은 자동으로 갱신되지 않으므로 확인하려면 다른 화면에 갔다가 돌아와야 합니다.

**Fanuc·Siemens 는 상태 `-20` 입니다.** 두 기종은 공구 번호와 보정 번호를 잇는 열이 따로 없습니다 (Fanuc 은 `toolHNumber`·`toolDNumber`, Siemens 는 공구 자체가 보정값을 가집니다).

**Mitsubishi 는 공구관리 표를 읽을 수 없는 구성에서 상태 `-20` 입니다** (표를 쓰지 않는 장비·프로젝트에서 제어기가 읽기를 거절합니다. 에러 문구에 벤더 코드가 함께 실립니다).

## /machine/toolArea/tool/sisterToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**자매공구 번호**입니다 (SINUMERIK `duploNo`, 조작반의 `ST` 열). 이름이 같은 공구들을 구분하는 번호이며, 앞선 공구의 수명이 다했을 때 어느 것이 대체 투입될지를 정합니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호).

**`1` 부터 이어지는 순번이 아닙니다.** 840D sl 벤치에서 그 이름의 공구가 하나뿐인데 이 값이 `2` 인 공구, `5` 인 공구, `101` 인 공구가 있었습니다. 번호로 정렬하거나 `1` 이 반드시 있다고 가정하지 마세요.

**새로 만든 공구는 이 값이 공구 번호와 같게 시작합니다** (실측: `50` 번으로 만든 공구의 자매번호가 `50`). 자매공구를 쓸 생각이면 만든 뒤 이 주소로 원하는 값을 넣으세요. 그대로 두어도 동작에는 지장이 없지만, 같은 이름의 공구를 나중에 추가할 때 번호가 뒤죽박죽으로 보입니다.

⚠️ **조작반에서 바로 옆 `D` 열과 헷갈리기 쉽습니다.** `ST` 는 **어느 공구**인가, `D` 는 그 공구의 **어느 날**인가입니다. 이름이 같은 줄이 여러 개 보일 때 `ST` 가 같으면 공구 한 자루의 날 여러 개이고, `ST` 가 다르면 서로 다른 공구입니다.

반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 2}`. **Siemens 전용**입니다. 공구 관리 기능을 쓰는 장비에서는 공구 이름과 이 번호의 조합이 공구의 정체이므로, 이름이 같은 공구가 여럿일 때 이 번호로 구분합니다.

없는 공구를 지정하면 상태 `-18` 로 거절됩니다. 값이 정수가 아니거나 `0`~`65535` 를 벗어나면 상태 `-16` 입니다. 실제 유효 상한은 장비 설정이 정하며, 그보다 좁은 범위를 벗어난 값은 장비가 거절합니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 이름이 같은 공구를 번호로 구분하는 자매공구는 Siemens 공구 관리의 개념이고, Fanuc 의 대체 공구는 공구수명관리 그룹(`/machine/toolArea/toolGroup/…`)이 맡습니다.

**Heidenhain 은 상태 `-20` 입니다.** Heidenhain 의 대체 공구는 이름이 같은 공구 사이의 번호가 아니라 다른 공구를 가리키는 값이고 인덱스 공구까지 가리킬 수 있어, `/machine/toolArea/tool/sisterTool` 이 답합니다.

## /machine/toolArea/tool/sisterTool
```yaml
value_type: "object"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

그 공구 대신 쓸 **대체 공구**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `object`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": {"toolNumber": 320, "toolEdgeNumber": 1}}`.

값은 `{"toolNumber": 320, "toolEdgeNumber": 1}` 처럼 **대체 공구를 가리키는 번호 둘**입니다. 두 값을 그대로 `tool`·`toolEdge` 필터에 넣으면 그 공구를 조회할 수 있습니다. 대체 공구가 없으면 `{"toolNumber": 0, "toolEdgeNumber": 0}` 입니다.

**Heidenhain 전용**입니다. 공구 테이블의 대체 공구(`RT`, 조작반 공구 관리 화면의 수명 항목)이고, 인덱스 공구(`320.1` 처럼 공구 번호 뒤에 붙는 행)도 가리킬 수 있어 `toolEdgeNumber` 가 그 인덱스입니다 (`320` 은 `0`, `320.1` 은 `1`. `toolEdge` 필터와 같은 번호). 제어기는 이 값을 소수 하나(`320.1`)로 두지만 디메시는 번호 둘로 나눠 냅니다. 소수로 받으면 공구 번호와 인덱스를 떼어 내는 계산이 이진 소수의 오차로 틀릴 수 있기 때문입니다 (`5.3` 의 소수부가 `0.2999…` 가 됩니다). 이 주소는 공구 자신의 행의 값입니다. 인덱스 공구의 행에도 대체 공구 칸이 따로 있고, 그 값은 `/machine/toolArea/tool/toolEdge/sisterTool` 이 답합니다 (`toolEdge=0` 은 이 주소와 같은 값).

**쓰기**: 두 키를 모두 정수로 주세요. 다른 키가 섞이거나 하나가 빠지면 상태 `-16` 입니다 (잘못 쓴 키 이름을 무시하면 다른 공구를 가리키게 되므로 받지 않습니다). `{"toolNumber": 0, "toolEdgeNumber": 0}` 을 쓰면 대체 공구를 비웁니다. 대체 공구는 공구 테이블에 있어야 하고, 없는 공구는 제어기가 받지 않아 상태 `-16` 입니다. 제어기가 소수 하나로 두기 때문에 그 모양으로 구분되지 않는 인덱스(`10`·`20`·`100` 처럼 끝자리가 `0` 인 인덱스)는 보내지 않고 상태 `-16` 입니다. 테이블 맨 앞의 `0` 번 행은 공구가 아닌 자리로 보아 상태 `-18` 입니다.

Siemens·Fanuc·Mitsubishi 는 상태 `-20` 입니다. Siemens 의 자매공구는 이름이 같은 공구들 사이에서 그 공구 자신의 번호라 `/machine/toolArea/tool/sisterToolNumber` 가 답합니다.

## /machine/toolArea/tool/toolTeethCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

공구의 **날 수**입니다 ("4날 엔드밀" 이라 할 때의 그 수). `toolArea` + `tool` 필터. 반환 `int`, 읽기와 쓰기가 됩니다.

`toolEdge` 밑에도 같은 이름이 있습니다. 그쪽은 **날마다 값을 갖는** 제어기(Siemens)의 자리이고, 이쪽은 **공구 하나에 하나**인 제어기(Mitsubishi)의 자리입니다. 쓰는 기종이 갈리므로 한 장비에서 둘 다 답하는 일은 없습니다 (Fanuc 은 둘 다, Siemens 는 이 주소가 상태 `-20` 입니다).

Mitsubishi 는 조작반 `절차 > 툴관리` 의 `Num. of teeth` 입니다. 그 화면은 자동으로 갱신되지 않으므로, 쓴 값을 화면에서 확인하려면 다른 화면에 갔다가 돌아와야 합니다. 쓰기는 그 표에 **이미 등록된 공구**에만 됩니다. 없는 공구 번호는 상태 `-18`(잘못된 필터 값)로 거절하고, 어떤 번호가 있는지는 `/machine/toolArea/toolList` 가 알려줍니다. 값이 그 칸이 받는 범위나 자릿수를 넘으면 제어기가 거절하고 상태 `-16`(잘못된 쓰기 값)이 나갑니다. 데이터 보호 키 1(PLC 신호 `*KEY1`, `Y708`. 공구 데이터를 보호합니다)이 꺼져 있으면 제어기가 이 칸의 쓰기를 거절하고 상태 `-22`(기계 상태)가 나가니 키를 켠 뒤 다시 쓰세요 (시뮬레이터에서 확인. 디메시가 거절된 뒤 그 신호를 읽어 가립니다. 같은 표의 `toolBodyLength`·`toolBodyDiameter` 는 키가 꺼져도 받아들여졌습니다).

**Mitsubishi 는 공구관리 표를 읽을 수 없는 구성에서 상태 `-20` 입니다** (표를 쓰지 않는 장비·프로젝트에서 제어기가 읽기를 거절합니다. 에러 문구에 벤더 코드가 함께 실립니다).

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

Mitsubishi 는 조작반 `절차 > 툴관리` 의 `Length :A` 입니다. 그 화면은 자동으로 갱신되지 않으므로, 쓴 값을 화면에서 확인하려면 다른 화면에 갔다가 돌아와야 합니다. 쓰기 규칙은 `toolTeethCount` 와 같습니다 (등록된 공구만, 범위 밖은 상태 `-16`). 다만 데이터 보호 키 1(`*KEY1`)은 이 칸을 막지 않아(시뮬레이터에서 키를 끈 채 받아들여졌습니다), 이 칸의 거절은 키와 관계없이 상태 `-16` 입니다. 이 칸은 설정 단위(`#1003`)와 관계없이 소수 셋째 자리까지 담고 조작반에서 입력해도 같으므로, 디메시도 셋째 자리로 반올림해 보내고 상태 `0` 을 돌려줍니다 (테스트 환경의 1nm 설정에서 `12.345678` 이 `12.346` 으로 저장됨). 무엇이 들어갔는지는 다시 읽어 확인하세요.

**Fanuc·Siemens 는 상태 `-20` 입니다.** 이 열은 Mitsubishi 공구 관리 표 고유의 것입니다.

**Mitsubishi 는 공구관리 표를 읽을 수 없는 구성에서 상태 `-20` 입니다** (표를 쓰지 않는 장비·프로젝트에서 제어기가 읽기를 거절합니다. 에러 문구에 벤더 코드가 함께 실립니다).

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

**Mitsubishi 는 공구관리 표를 읽을 수 없는 구성에서 상태 `-20` 입니다** (표를 쓰지 않는 장비·프로젝트에서 제어기가 읽기를 거절합니다. 에러 문구에 벤더 코드가 함께 실립니다).

## /machine/toolArea/tool/toolSpindleSpeed
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

공구관리 데이터에 **공구별로 적어 둔 주축 회전수 `S`** 입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int` + `unit:"rpm"`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 1500}` (정수).

**제어기가 `T` 를 불렀을 때 자동으로 적용하는 값이 아닙니다.** 조작 설명서(B-64484EN)는 이 값을 공구관리 데이터에 등록한 가공 조건으로 설명하며, 공구 교환 매크로(예: `M06`)에서 `S#8411` 처럼 코딩해 직접 지정할 수 있다고 안내합니다. 즉 기계 제작사나 작업자가 교환 매크로·가공 프로그램을 "이 공구의 등록 조건으로 돌리도록" 짜 두었을 때만 쓰이는 **참고값**이고, 그렇게 짜지 않은 장비에서는 적혀 있어도 아무 효과가 없습니다. 지금 실제 회전수는 `/machine/channel/spindle/spindleSpeedActual`, 지령값은 `spindleSpeedCommanded` 를 읽으세요.

실측 사례도 있습니다. 실제 교환 매크로가 있는 31i 벤치에서 공구에 `1234` 를 적고 `M06` 교환을 돌렸는데, 교환 뒤에도 지령 S 는 값 `0` 그대로였습니다. 이 값이 실려 가는지는 전적으로 그 장비의 매크로에 달렸으니, 의존하기 전에 그 장비에서 확인하세요.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 조작반 EACH TOOL DATA 화면의 `S` 칸이며 FOCAS 스펙의 유효 범위는 `1`~`99999` 라 `0` 은 적어 두지 않은 칸입니다 (테스트 벤치는 전부 `0`). 등록되지 않은 공구는 상태 `-18` 입니다. 쓰기는 `0`~`99999` 의 정수만 받고(밖이면 상태 `-16`), 그 밖의 거절은 벤더 사유를 싣고 돌아오며, 조작반 쪽 사정(쓰기 금지·모드·실행 중)이면 상태 `-22`, 다른 호출이 진행 중이면 상태 `-24`, 그 밖은 상태 `-17` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 이 칸은 Fanuc 공구관리 데이터의 항목이라 디메시가 두 기종에는 대응시키지 않습니다.

## /machine/toolArea/tool/toolFeed
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

공구관리 데이터에 **공구별로 적어 둔 절삭 이송 `F`** 입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 250}` (정수).

**제어기가 `T` 를 불렀을 때 자동으로 적용하는 값이 아닙니다.** `/machine/toolArea/tool/toolSpindleSpeed` 와 같은 부류로, 기계 제작사의 교환 매크로가 `F#8412` 로 읽어 쓰도록 적어 두는 **참고값**입니다 (조작 설명서 B-64484EN). 그렇게 짜지 않은 장비에서는 적혀 있어도 효과가 없습니다. 지금 실제 이송은 `/machine/channel/feedActual`, 지령값은 `feedCommanded` 를 읽으세요.

실측 사례도 있습니다. 실제 교환 매크로가 있는 31i 벤치에서 공구에 `567` 을 적고 `M06` 교환을 돌렸는데, 교환 뒤에도 지령 F 는 이 값으로 바뀌지 않았습니다 (이전 모달 그대로). 이 값이 실려 가는지는 전적으로 그 장비의 매크로에 달렸으니, 의존하기 전에 그 장비에서 확인하세요.

**`unit` 을 붙이지 않습니다.** FOCAS 스펙이 이 칸의 단위를 mm/min·inch/min·deg/min·mm/rev·inch/rev 로 다 허용하고, 어느 단위로 적었는지는 그것을 읽는 매크로가 정하기 때문입니다 (다른 이송 주소가 기계 설정 때문에 단위를 붙이지 않는 것과 같습니다).

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 조작반 EACH TOOL DATA 화면의 `F` 칸이며 FOCAS 스펙의 범위는 `0`~`99999999` 입니다 (테스트 벤치는 전부 `0`). 등록되지 않은 공구는 상태 `-18` 입니다. 쓰기는 `0`~`99999999` 의 정수만 받고(밖이면 상태 `-16`), 단위는 쓰는 쪽이 그 장비의 매크로와 맞춰야 합니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 이 칸은 Fanuc 공구관리 데이터의 항목이라 디메시가 두 기종에는 대응시키지 않습니다.

## /machine/toolArea/tool/toolDataLockedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 공구의 **공구관리 데이터가 편집 잠금되어 있는지** 여부입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `boolean`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": true}` 로 잠그고 `{"value": false}` 로 풉니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 공구 정보의 LOC 비트(조작반 `T-INFO` 의 `L`/`U`, 벤더 문서의 "Data access: Locked/Unlocked")입니다. **공구를 쓰지 말라는 뜻이 아닙니다** (그건 `/machine/toolArea/tool/toolUseStatus`): 벤더 문서의 이름대로 데이터 접근의 잠금이며, 제어기의 공구 검색·교환에는 영향이 없습니다. 실제로 무엇이 막히는지는 아래를 보세요.

**이 잠금은 SDK 쓰기를 막지 않습니다.** 31i 벤치에서 잠근 공구의 수명 카운터·예고 수명·H 번호·주축 회전수를 FOCAS 로 썼더니 모두 성공했습니다 (파라미터 `13204#0` 과 조작반의 메모리 보호 키를 어느 쪽으로 두어도). **조작반 편집도 잠금만으로는 막히지 않습니다.** `13204#0`(TDL)이 `0` 이면 잠긴 공구도 조작반에서 편집되고, `1` 이면 잠금과 상관없이 공구관리 데이터 보호 신호 `TKEY0`~`TKEY5`(`G330`)가 조작반 입력을 항목별로 허락합니다 (연결 기능 매뉴얼 B-64483EN-1. 벤치에서 그 신호가 모두 `0` 일 때 `WRITE PROTECT`). 조작반에서 그 공구의 데이터를 편집 중이면 이 잠금을 바꾸는 쓰기를 제어기가 거절하고, 디메시는 상태 `-17` 에 그 사유를 싣습니다. 편집을 닫고 다시 쓰세요.

쓰기는 공구 정보 워드의 이 비트 하나만 바꾸고 나머지 비트는 읽은 그대로 다시 씁니다. 이미 그 상태면 아무것도 하지 않고 성공합니다. 등록되지 않은 공구는 읽기·쓰기 모두 상태 `-18` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 이 잠금은 Fanuc 공구관리 데이터의 항목이라 디메시가 두 기종에는 대응시키지 않습니다.

## /machine/toolArea/tool/toolSearchedWhenUnmanagedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**수명 관리를 하지 않는 공구라도 `T` 검색 대상에 넣을지** 여부입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `boolean`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": true}` / `{"value": false}`.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 값은 공구 정보의 SEN 비트(조작반 `T-INFO` 의 `S`/`-`)입니다. 제어기는 프로그램이 `T` 로 타입 번호를 부르면 그 번호의 공구 중 하나를 고르는데, **수명 상태가 관리 안 함(`L-STATE` `NO-MNG`)인 공구는 원래 후보에서 빠집니다.** 이 값이 `true` 면 그런 공구도 잔여 수명을 보지 않고 후보에 넣습니다 (Connection Manual B-64483EN-1). 수명 관리 중인 공구(`/machine/toolArea/tool/toolLifeMonitorType` 이 `0` 이 아닌 공구)에는 영향이 없습니다.

이 비트의 뜻은 Connection Manual 과 조작 설명서에서 가져왔고, 테스트 벤치(조작반이 이 비트를 `S` 로 표시하고 `cnc_wrtool2` 로 켜고 끌 수 있음)로 확인했습니다.

쓰기는 공구 정보 워드의 이 비트 하나만 바꾸고 나머지 비트는 읽은 그대로 다시 씁니다. 이미 그 상태면 아무것도 하지 않고 성공합니다. 등록되지 않은 공구는 읽기·쓰기 모두 상태 `-18` 입니다. 조작반에서 그 공구의 데이터를 편집 중이면 이 쓰기도 `/machine/toolArea/tool/toolDataLockedOn` 쓰기처럼 거절될 수 있고, 그때는 에러 문구가 그 사유를 알립니다 (같은 공구 정보 항목에 든 비트라서이며, 거절은 31i 벤치에서 잠금 비트로만 확인했습니다). 기계 제작사의 운용 방침에 속하는 플래그이므로, 바꾸기 전에 그 장비의 공구 교환 절차를 확인하세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 이 플래그는 Fanuc 공구관리 데이터의 항목이라 디메시가 두 기종에는 대응시키지 않습니다.

## /machine/toolArea/tool/toolOversizedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

그 공구가 **포켓 하나보다 큰지** 여부입니다. 양옆 포켓을 비워 둬야 하는 굵은 공구입니다. `toolArea` + `tool` 필터. 반환 `boolean`, **읽기 전용**.

Siemens 의 매거진 화면에서 `Z` 열이 이 값입니다. 몇 포켓을 차지하는지가 아니라 **하나를 넘는지 여부**만 답합니다.

**쓰기는 지원하지 않습니다.** Siemens 는 초과크기를 하나의 플래그가 아니라 위·아래·좌·우로 몇 칸을 차지하는지로 저장하고 있어, `true` 를 받아도 어느 방향으로 몇 칸인지 정할 수 없습니다. 초과크기 지정은 조작반에서 하세요.

없는 공구는 상태 `-18` 로 거절됩니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구 정보의 대형 공구 비트(BDT)이며 읽기 전용입니다. 대형 공구가 차지하는 이웃 포켓은 `/machine/toolArea/magazine/pocketList` 에서 `toolNumber` `0` 으로 나옵니다 (벤치 실측: 비트를 켜니 `true`).

**Heidenhain** 은 포켓 테이블의 `ST` 칸으로, 대형 공구 같은 특수 공구의 표시입니다 (TNC7 사용 설명서 'Pocket table tool_p.tch' 는 그 이웃 포켓을 잠금 칸 `L` 로 막는다고 설명합니다). 그 공구가 든 포켓의 값이고, 스핀들에 있는 공구는 비워 둔 원래 포켓의 값입니다. 이 표시가 포켓에 있어서 **매거진에 없는 공구는 `false`** 입니다. 테스트 환경에서 포켓 하나의 `ST` 를 켜자 그 포켓의 공구만 `true` 가 되었습니다. 읽기만 합니다. 디메시가 포켓 테이블에서 `ST` 칸을 찾지 못하면 상태 `-20` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

## /machine/toolArea/tool/toolFixedLocationOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

그 공구가 **고정 자리로 지정되어 있는지** 여부입니다. 늘 같은 포켓으로 돌아갑니다. `toolArea` + `tool` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원합니다. `{"value": true}` 로 지정하고 `false` 로 해제합니다.

`true` 면 공구 교환 후 원래 포켓으로 돌아가고, `false` 면 장비가 빈 포켓을 골라 넣습니다. Siemens 는 매거진 화면의 `L` 열이 이 값입니다.

이미 그 상태면 아무것도 하지 않고 성공합니다. 없는 공구는 상태 `-18` 로 거절됩니다.

**Siemens·Heidenhain** 이 지원합니다.

**Heidenhain** 은 이 표시를 공구가 아니라 포켓 테이블의 `F` 칸(고정 포켓. TNC7 사용 설명서 'Pocket table tool_p.tch' 는 공구를 늘 같은 포켓으로 돌려놓는 표시로 설명합니다)에 둡니다. 그래서 그 공구가 든 포켓의 값을 읽고 쓰며, 스핀들에 있는 공구는 비워 둔 원래 포켓의 값입니다. **포켓 테이블에 자리가 없는 공구는 `false`** 이고 (매거진에 없거나, 스핀들에 있는데 원래 포켓이 예약돼 있지 않은 공구, 고정 자리를 둘 포켓이 없습니다), 쓰면 상태 `-22` 입니다. 공구를 매거진에 넣은 뒤 쓰세요. 포켓 테이블에서 `F` 칸을 찾지 못하면 상태 `-20` 입니다. 포켓 테이블을 다루는 방식은 기계에 따라 다릅니다 (TNC7 사용 설명서 'Configuring a tool': 기계 제작사의 기능이나 외부 공구 관리 시스템이 다루기도 합니다). 그런 장비에서는 쓴 값이 바뀌거나 그 체계와 어긋날 수 있으니 기계 설명서를 확인하세요. 테스트 환경에서 매거진의 공구와 스핀들의 공구로 읽기·쓰기를 확인했습니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 디메시는 두 기종의 공구 데이터에서 이 항목을 읽지 않습니다.

## /machine/toolArea/tool/toolLifeMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
codes: [{"value": 0, "name": "no monitoring", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": 1, "name": "time"}, {"value": 2, "name": "count", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]}, {"value": 3, "name": "wear", "read": ["nc_opcua_siemens"]}]
```

그 공구의 **수명 감시 방식**입니다. `toolArea` + `tool` 필터. 반환 `int`. 읽기·쓰기 모두 Siemens·Fanuc 에서 지원하고, Mitsubishi·Heidenhain 은 읽기만 지원합니다. 쓰기는 `{"value": 2}`.

| 값 | 뜻 | 수명 값의 단위 |
|---|---|---|
| `0` | 감시 없음 | 없음 |
| `1` | 시간(실제로 깎은 시간을 센다) | 벤더가 주는 단위 그대로: Siemens·Mitsubishi·Heidenhain 은 분(`unit` 은 `"min"`), Fanuc 은 초(`unit` 은 `"s"`) |
| `2` | 횟수(무엇을 세는지는 기종이 정한다: Siemens 는 완성한 가공물 수, Mitsubishi 는 장착 횟수 또는 절삭 횟수) | 횟수 (`unit` 은 `"count"`) |
| `3` | 마모(오프셋이 한계까지 밀렸는지 본다) | 기계 설정 (mm/inch) 이라 `unit` 을 붙이지 않는다 |

**어느 기종이든 이 넷입니다.** 장비 고유의 번호가 아니라 디메시가 정한 값이라 어느 기종에 붙었는지 몰라도 그대로 분기할 수 있습니다.

Siemens 에서는 방식은 **공구가 하나 고르고, 값은 날마다 따로**입니다. 그래서 이 주소는 `toolEdge` 를 받지 않고, 날 단위 수명 값 3종은 받습니다. Fanuc 은 수명도 공구 단위라 `/machine/toolArea/tool/toolLifeTotal`·`toolLifeUsed`·`toolLifeWarnLimit` 이 짝입니다.

Siemens·Fanuc 은 `0` 이면 수명 값 3종이 상태 `-18` 로 거절됩니다. 그 공구에는 잴 것이 없습니다. 감시를 켜려면 이 주소에 방식을 먼저 쓰고, 그 다음 수명 총량을 넣으세요 (Siemens 는 `/machine/toolArea/tool/toolEdge/toolLifeTotal`, Fanuc 은 `/machine/toolArea/tool/toolLifeTotal`. Fanuc 은 조작반의 공구관리 화면에서 수명 상태(`L-STATE`)를 켜도 같습니다). Heidenhain 은 방식을 쓰지 않고 최대 수명으로 켭니다 (아래).

Siemens 는 여러 방식을 **동시에** 켜 둘 수도 있습니다. 그 경우 이 주소는 시간 → 개수 → 마모 순으로 하나를 골라 답하고, 수명 값 3종도 같은 순서를 따르므로 방식과 값이 어긋나지 않습니다. "갈아야 하나" 의 답인 `/machine/toolArea/tool/toolLifeWarnOn` 과 `/machine/toolArea/tool/toolUseStatus` 는 방식과 무관하게 항상 정확합니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터(`cnc_rdtool`)에서 읽습니다: 수명 상태가 `NO-MNG`(관리 안 함)인 공구는 수명 값이 표시돼 있어도 제어기가 세지 않으므로 `0`, 관리 중이면 공구 정보의 수명 종류 비트에 따라 `1`(시간) 또는 `2`(횟수)입니다. `3`(마모)은 Fanuc 공구관리에 없어 쓰기에서 상태 `-16` 입니다. **쓰기 규칙**: `0` 은 수명 상태를 관리 안 함으로 바꾸고(수명 종류 비트와 값은 그대로 남음), `1`/`2` 는 수명 종류 비트를 시간/횟수로 놓고, 관리 안 함이던 공구는 수명 카운터가 `0` 이면 미사용, 아니면 잔여 있음 상태로 켭니다. 이미 관리 중이면 종류만 바꿉니다 (종류를 바꿔도 숫자는 환산되지 않으니 수명 값을 다시 넣으세요). 조작반에서 그 공구의 데이터를 편집 중이면 `1`·`2` 로 종류를 바꾸는 쓰기도 `/machine/toolArea/tool/toolDataLockedOn` 쓰기처럼 거절될 수 있고, 그때는 에러 문구가 그 사유를 알립니다 (같은 공구 정보 항목을 쓰기 때문이며, 거절은 31i 벤치에서 잠금 비트로만 확인했습니다).

이 주소의 예전 이름은 `/machine/toolArea/tool/toolMonitorType` 입니다. 옛 주소 그대로도 동작하지만, 이 문서는 새 이름만 안내합니다.

**Mitsubishi** 는 공구 수명 관리에 등록된 공구의 방식(조작반 `툴수명` 그룹 화면의 `방식`, 세 자리 가운데 맨 오른쪽 자리)을 옮깁니다. 누적 절삭 시간은 `1`, 누적 장착 횟수와 누적 절삭 횟수는 `2` 이고, 어느 쪽인지 `desc` 가 밝힙니다 (`"Cutting time"`·`"Mounting count"`·`"Cutting count"`, 조작 매뉴얼 IB-1501274). `0`·`3` 은 나오지 않습니다. 짝이 되는 수명 값은 `/machine/toolArea/tool/toolLifeTotal`·`toolLifeUsed` 입니다. `toolArea` 는 파트 시스템 번호이고, 수명 관리에 등록되지 않은 공구는 상태 `-18`, 공구 수명 그룹을 읽을 수 없는 파트 시스템은 상태 `-20` 입니다. EZSocket `FCSB1224W100-A9` 이상이 필요합니다. 그보다 앞선 판에서는 다른 판정보다 먼저 늘 상태 `-20` 이고, 그 판 이상을 설치하면 읽힙니다. 제어기가 돌려준 수명 값이 디메시가 읽는 배치가 아니면(다른 배치로 답하는 파트 시스템) 역시 상태 `-20` 입니다 (시뮬레이터에서 조작반과 대조).

**Heidenhain** 은 시간 감시 하나입니다. 공구 테이블의 최대 수명(`TIME1`)이나 공구를 부를 때의 한계(`TIME2`)가 `0` 보다 크면 `1`, 둘 다 `0` 이면 `0` 입니다 (TNC7 사용 설명서 'Tool table tool.t' 는 둘 다 넘으면 공구를 잠그는 한계로 적습니다). 이 주소의 쓰기는 상태 `-20` 이고, 감시를 켜고 끄는 것은 최대 수명을 쓰는 일입니다: `/machine/toolArea/tool/toolLifeTotal` 에 분 단위 값을 쓰면 켜지고 `0` 을 쓰면 꺼집니다 (`TIME2` 는 디메시가 쓰지 않아, 그 칸에 값이 있으면 `0` 을 써도 `1` 로 남습니다). 짝이 되는 수명 값은 `/machine/toolArea/tool/toolLifeTotal`·`toolLifeUsed` 입니다. 이 주소는 공구 자신의 행이고, 인덱스 공구(`320.1` 처럼)가 감시되는지는 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 이 상태 `-18` 인지로 가려내세요.

## /machine/toolArea/tool/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_dnc_heidenhain"]
```

그 공구에 배정된 **수명 총량**(최대 수명)입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `float`, 읽기·쓰기 모두 지원합니다 (Mitsubishi 는 읽기만). 쓰기는 `{"value": 20}`.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 최대 수명(조작반 TOOL MANAGER 화면의 `MAX-LIFE`)이고, 제어기는 사용량 카운터(`/machine/toolArea/tool/toolLifeUsed`)를 `0` 에서 이 값까지 세어 올라갑니다. 잔여는 이 값에서 사용량을 뺀 것입니다. Fanuc 의 수명은 날이 아니라 **공구에 하나** 붙으므로 `toolEdge` 필터 없이 공구 단위로 둡니다.

**단위는 벤더가 주는 그대로입니다.** 감시 방식(`/machine/toolArea/tool/toolLifeMonitorType`)이 시간이면 **초**(`unit` 은 `"s"`, 조작반은 `4H 5M 6S` 처럼 시·분·초로 표시), 횟수면 `count`. 분으로 환산하지 않습니다 (Siemens 날 단위 수명이 분인 것과 다르니 응답의 `unit` 을 보세요). 수명 상태가 관리 안 함(`toolLifeMonitorType` 이 `0`)이면 값이 남아 있어도 상태 `-18` 입니다 (방식을 먼저 쓰세요).

쓰기는 읽기와 같은 단위의 정수만 받으며(소수는 상태 `-16`), 관리 안 함 상태면 상태 `-18` 입니다. 등록되지 않은 공구는 상태 `-18` 입니다.

**Mitsubishi** 는 공구 수명 관리 데이터의 수명(조작반 `툴수명` 그룹 화면의 `수명`)입니다. 단위는 방식(`toolLifeMonitorType`)을 따라 절삭 시간이면 **분**(`unit` 은 `"min"`), 횟수면 `count` 이고, `0` 은 수명 제한이 없다는 뜻입니다 (조작 매뉴얼 IB-1501274). `toolArea` 는 파트 시스템 번호이고, 수명 관리에 등록되지 않은 공구는 상태 `-18`, 공구 수명 그룹을 읽을 수 없는 파트 시스템은 상태 `-20` 입니다. EZSocket `FCSB1224W100-A9` 이상이 필요합니다. 그보다 앞선 판에서는 다른 판정보다 먼저 늘 상태 `-20` 이고, 그 판 이상을 설치하면 읽힙니다. 제어기가 돌려준 수명 값이 디메시가 읽는 배치가 아니면(다른 배치로 답하는 파트 시스템) 역시 상태 `-20` 입니다 (시뮬레이터에서 조작반과 대조).

**Siemens 는 상태 `-20` 입니다.** Siemens 의 수명은 날별이라 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 에 있습니다.

**Heidenhain** 은 공구 테이블의 최대 수명(`TIME1`)입니다. 단위는 분입니다 (`unit` 은 `"min"`, 조작반 공구 관리 화면의 `TIME1 (min)`). `0` 이면 최대 수명이 없다는 뜻이라 읽기는 상태 `-18` 입니다 (공구를 부를 때의 한계 `TIME2` 만 있는 공구도 그렇고, 그때는 에러 문구에 `TIME2` 가 실립니다. `toolLifeMonitorType` 참조). **쓰기는 감시가 꺼져 있어도 받습니다**: 값을 쓰면 감시가 켜지고 `0` 을 쓰면 꺼집니다. 정수 분만 받습니다 (소수는 상태 `-16`. 제어기가 정수로 반올림해 담아서입니다: 테스트 환경에서 `1.5` 가 `2` 가 되었습니다). 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 허용 범위가 실립니다 (시뮬레이터에서 `0`~`99999`). 테이블 맨 앞의 `0` 번 행은 공구가 아닌 자리로 보아 상태 `-18` 입니다. 이 주소는 공구 자신의 행입니다. 인덱스 공구(`320.1` 처럼)는 행마다 수명이 따로 있어 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 가 답합니다 (`toolEdge=0` 은 이 주소와 같은 값).

## /machine/toolArea/tool/toolLifeUsed
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_dnc_heidenhain"]
```

그 공구가 **지금까지 쓴 수명**(수명 카운터)입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `float`, 읽기·쓰기 모두 지원합니다 (Mitsubishi 는 읽기만). 쓰기는 `{"value": 0}`.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 수명 카운터(조작반 `L-COUNT`)이며, 제어기가 그 공구가 스핀들에 있는 동안 **`0` 에서 최대 수명(`/machine/toolArea/tool/toolLifeTotal`)을 향해 올려 셉니다** (Connection Manual B-64483EN-1: 증가 카운터, 잔여 = 최대 − 카운터). 잔여가 필요하면 `toolLifeTotal` 에서 이 값을 빼세요. 잔여 주소를 따로 두지 않는 것은 Fanuc 이 주는 값이 이것이고, 화면(`L-COUNT`)과 같은 숫자를 내기 위해서입니다.

단위는 `toolLifeTotal` 과 같습니다 (시간 감시 초 `"s"`, 횟수 감시 `count`). 수명 상태가 관리 안 함(`toolLifeMonitorType` 이 `0`)이면 상태 `-18` 입니다.

**인서트를 갈고 카운터를 되돌릴 때 이 주소에 씁니다** (보통 `0`). 쓰기는 읽기와 같은 단위의 정수만 받으며(소수는 상태 `-16`), 관리 안 함 상태면 상태 `-18` 입니다. 수명 초과로 잠긴 공구는 카운터를 되돌린 뒤 상태도 되돌려야 쓰입니다 (`/machine/toolArea/tool/toolUseStatus` 참조). 등록되지 않은 공구는 상태 `-18` 입니다.

**Mitsubishi** 는 그 공구의 사용량(조작반 `툴수명` 그룹 화면의 `사용`)입니다. 단위는 `toolLifeTotal` 과 같고(절삭 시간 분 `"min"`, 횟수 `count`), 사용량이 수명을 넘으면 그 공구의 상태가 수명 도달이 됩니다 (`/machine/toolArea/toolGroup/toolLifeStatusList` 의 `2`, 조작 매뉴얼 IB-1501274). `toolArea` 는 파트 시스템 번호이고, 수명 관리에 등록되지 않은 공구는 상태 `-18`, 공구 수명 그룹을 읽을 수 없는 파트 시스템은 상태 `-20` 입니다. EZSocket `FCSB1224W100-A9` 이상이 필요합니다. 그보다 앞선 판에서는 다른 판정보다 먼저 늘 상태 `-20` 이고, 그 판 이상을 설치하면 읽힙니다. 제어기가 돌려준 수명 값이 디메시가 읽는 배치가 아니면(다른 배치로 답하는 파트 시스템) 역시 상태 `-20` 입니다 (시뮬레이터에서 조작반과 대조).

**Siemens 는 상태 `-20` 입니다.** Siemens 는 잔여를 내려 세므로 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 을 보세요.

**Heidenhain** 은 공구 테이블의 현재 사용 시간(`CUR_TIME`)입니다. 단위는 분이고 (`unit` 은 `"min"`, 조작반 공구 관리 화면의 `CUR_TIME (min)`), 제어기가 최대 수명(`/machine/toolArea/tool/toolLifeTotal`) 쪽으로 올려 셉니다 (시뮬레이터에서 이송 블록 동안 늘었습니다). 수명 한계가 없으면(`TIME1`·`TIME2` 둘 다 `0`, `toolLifeMonitorType` 이 `0`) 읽기·쓰기 모두 상태 `-18` 입니다. 먼저 `toolLifeTotal` 을 쓰세요. 쓰기는 소수도 받되, 제어기가 소수 둘째 자리로 반올림해 담습니다 (테스트 환경에서 `1.234` 는 `1.23`, `1.235` 는 `1.24` 가 되었습니다). 무엇이 담겼는지는 다시 읽어 확인하세요. TNC7 사용 설명서 'Tool table tool.t' 는 입력 범위를 `0`~`99999.99` 로 적고, 운전 중에 고쳐도 곧바로 수명 감시에 반영된다고 적습니다. 사용 시간이 `TIME2` 에 닿으면(같아도) `/machine/toolArea/tool/toolUseStatus` 가 `3` 이고, 최대 수명(`TIME1`)을 넘은 것은 제어기의 잠금을 따르므로 곧바로 `3` 이 되지는 않습니다 (그 주소 참조). 테이블 맨 앞의 `0` 번 행은 공구가 아닌 자리로 보아 상태 `-18` 입니다. 이 주소는 공구 자신의 행입니다. 인덱스 공구(`320.1` 처럼)는 행마다 수명이 따로 있어 `/machine/toolArea/tool/toolEdge/toolLifeUsed` 가 답합니다 (`toolEdge=0` 은 이 주소와 같은 값).

## /machine/toolArea/tool/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

그 공구의 **예고 수명**(경고선)입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 30}`.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 예고 수명(조작반 `NOTICE-L`, FOCAS 스펙의 "predictive tool life")으로, 잔여 수명(최대 수명에서 카운터를 뺀 값)이 이 값 이하가 되면 제어기가 수명 도달 예고 신호를 냅니다 (Connection Manual B-64483EN-1. 예고를 공구 타입 단위로 낼지 공구 단위로 낼지는 파라미터 `13200#3` 이 정하고, 타입 단위일 때 마지막 공구의 잔여를 볼지 같은 타입 공구들의 잔여 합을 볼지는 `13200#2` 가 정합니다). `0` 이면 예고 신호를 내지 않습니다.

단위는 `toolLifeTotal` 과 같습니다. 수명 상태가 관리 안 함(`toolLifeMonitorType` 이 `0`)이면 상태 `-18` 입니다. 쓰기는 읽기와 같은 단위의 정수만 받으며(소수는 상태 `-16`), 관리 안 함 상태면 상태 `-18`, 등록되지 않은 공구는 상태 `-18` 입니다.

Fanuc 에서는 `/machine/toolArea/tool/toolLifeWarnOn` 이 상태 `-20` 입니다 (디메시가 읽을 공구별 예고 도달 플래그를 찾지 못했습니다). 필요하면 `toolLifeTotal − toolLifeUsed` 와 이 값을 비교하세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** Siemens 의 경고선은 날별이라 `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit` 에 있고, Mitsubishi 어댑터는 공구 단위로는 공구 수명 값 가운데 방식·총량·사용량(`toolLifeMonitorType`·`toolLifeTotal`·`toolLifeUsed`)만 답합니다 (그룹 단위로는 `/machine/toolArea/toolGroup/toolLifeStatusList` 도 답합니다).

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

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 디메시는 Fanuc 공구 레코드에서 예고 상태를 읽지 않고(예고값은 `toolLifeWarnLimit`, 예고 신호는 PMC 쪽), Mitsubishi 어댑터는 공구 단위로는 공구 수명 값 가운데 방식·총량·사용량(`toolLifeMonitorType`·`toolLifeTotal`·`toolLifeUsed`)만 답합니다 (그룹 단위로는 `/machine/toolArea/toolGroup/toolLifeStatusList` 도 답합니다).

## /machine/toolArea/tool/toolLocationType
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
codes: [{"value": "magazine", "name": "Magazine"}, {"value": "buffer", "name": "Spindle or tool changer"}, {"value": "loading", "name": "Load/unload position", "read": ["nc_opcua_siemens"]}, {"value": "none", "name": "No physical place"}]
```

그 공구가 **어떤 종류의 자리에 있는지** 나타냅니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `string`, **읽기 전용**. 값이 곧 뜻입니다.

| 값 | 뜻 |
|---|---|
| `"magazine"` | 매거진(공구 저장고)에 꽂혀 있음 |
| `"buffer"` | 스핀들 또는 교환기(가공 중이거나 옮겨지는 중) |
| `"loading"` | 반입·반출 위치 |
| `"none"` | 실물 자리 없음(공구 데이터만 등록되어 있음) |

**어느 기종이든 이 넷입니다.** 장비는 스핀들·교환기·반입출 위치에도 자기 고유의 매거진 번호(Siemens 는 내부 버퍼 매거진 `9998`·로딩 매거진 `9999`, Fanuc 은 스핀들 위치 `11`~`14`·대기 위치 `21`~`24`, 2경로 이상에서는 경로 번호를 백의 자리에 붙인 `211`·`221` 같은 번호)를 붙이지만 그것은 벤더 상수라 호출하는 쪽이 알아야 할 이유가 없습니다. 디메시가 이 넷으로 묶어 내보내므로, 어느 기종에 붙었는지 몰라도 값으로 분기할 수 있습니다.

`"buffer"` 는 스핀들과 교환기 그리퍼를 **구분하지 않습니다.** 장비가 둘을 같은 자리로 취급하기 때문입니다. 구분이 필요하면 `/machine/channel/activeToolNumber` 와 겹쳐 보되, 기종별로 대조 방법이 다릅니다. Siemens 는 그 값이 교환 완료 후의 공구 번호라 그대로 비교하면 되고, Fanuc 공구관리 장비는 그 값이 타입 번호라 `/machine/toolArea/tool/toolTNumber` 로 후보를 좁힌 뒤 비교하세요.

**쓰기는 지원하지 않습니다.** 자리를 바꾸는 것은 공구 이동의 몫이고, 이 값만 고치면 장부와 실물이 어긋납니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 매거진 번호(카트리지 관리 표의 `1`~`8`. Connection Manual B-64483EN-1 은 최대 8개를 허용하고, 파라미터로 구성하는 것은 그중 `1`~`4` 입니다)는 `"magazine"`, 스핀들 위치와 대기 위치는 `"buffer"`, 어디에도 실려 있지 않으면 `"none"` 이고, Fanuc 에는 반입출 위치가 없어 `"loading"` 은 나오지 않습니다. 스핀들·대기 위치는 테스트 벤치에 설정되어 있지 않아 벤더 문서 기준입니다. Siemens 에서 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `"none"` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

**Heidenhain** 은 포켓 테이블에서 찾습니다: 스핀들 행(`0.0`)에 있으면 `"buffer"`, 매거진 행에 있으면 `"magazine"`, 표에 없으면 `"none"` 입니다. 스핀들에 올라간 공구는 원래 포켓에도 번호가 남지만(자리를 잡아 둔다) `"buffer"` 로 답합니다. 디메시는 Heidenhain 에서 반입·반출 위치를 가리지 않아 `"loading"` 은 나오지 않습니다.

## /machine/toolArea/tool/magazineNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

그 공구가 지금 꽂혀 있는 **매거진(공구 저장고)의 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

매거진에 없으면 `0` 입니다. 스핀들에 물려 가공 중이거나, 교환기가 옮기는 중이거나, 반입·반출 위치에 있거나, 공구 데이터만 등록되고 실물 자리가 없는 경우입니다. 장비는 이런 자리에도 자기 고유 번호(Siemens 는 내부 버퍼 매거진 `9998`·로딩 매거진 `9999`, Fanuc 은 스핀들 위치 `11`~`14`·대기 위치 `21`~`24`, 2경로 이상에서는 경로 번호를 백의 자리에 붙인 `211`·`221` 같은 번호)를 붙이지만 디메시는 그 번호를 내보내지 않고 `0` 으로 뭉뚱그립니다. 벤더 상수라 호출하는 쪽이 알아야 할 이유가 없습니다.

**지금 있는 곳이지 원래 자리가 아닙니다.** 공구가 스핀들에 물리면 이 값이 `0` 으로 바뀌고 매거진으로 돌아가면 다시 번호가 붙습니다. 원래 어느 자리에서 나왔는지는 이 주소가 답하지 않습니다.

**쓰기는 지원하지 않습니다.** 이 값은 실물 위치의 기록이라 디메시는 쓰기를 열지 않습니다. 장부와 실물이 어긋나면 이후 공구 교환에 영향을 줄 수 있으므로, 위치 변경은 장비의 공구 관리 절차로 하세요.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 매거진 번호(조작반 `MG`)를 그대로 내며, 스핀들 위치와 대기 위치는 매거진이 아니라 `0` 으로 뭉뚱그립니다 (`/machine/toolArea/tool/toolLocationType` 이 `"buffer"`). 스핀들·대기 위치는 테스트 벤치에 설정되어 있지 않아 벤더 문서 기준입니다. Siemens 에서 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `0` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

**Heidenhain** 은 포켓 테이블에서 그 공구가 든 매거진 행의 매거진 번호입니다. 스핀들에 있거나(`toolLocationType` 이 `"buffer"`) 표에 없으면 `0` 입니다. 스핀들에 있는 동안 원래 포켓에 남는 번호는 지금 자리가 아니라 `/machine/toolArea/tool/originalMagazineNumber` 가 답합니다.

## /machine/toolArea/tool/pocketNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

그 공구가 꽂혀 있는 **매거진 안의 포켓 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

매거진 번호가 아파트의 동이라면 이 값은 호수입니다. 둘을 함께 읽어야 위치가 정해집니다. 매거진에 없으면 `0` 입니다 (스핀들에서 가공 중이거나, 교환기가 옮기는 중이거나, 반입·반출 위치에 있거나, 실물 자리가 없는 경우). 장비가 그런 자리에 붙이는 고유 번호(예: `9998`)는 내보내지 않습니다.

**터렛(선반)의 스테이션도 포켓으로 나타납니다.** 장비가 터렛 위치를 매거진의 포켓으로 모델링하기 때문입니다. 현장에서 "3번 스테이션" 이라 부르는 자리가 이 주소에서는 포켓 `3` 입니다.

**지금 있는 곳이지 원래 자리가 아닙니다.** 공구가 스핀들에 물리면 이 값이 `0` 으로 바뀌고 매거진으로 돌아가면 다시 포켓 번호가 붙습니다.

**쓰기는 지원하지 않습니다.** 이 값은 실물 위치의 기록이라 디메시는 쓰기를 열지 않습니다. 장부와 실물이 어긋나면 이후 공구 교환에 영향을 줄 수 있으므로, 위치 변경은 장비의 공구 관리 절차로 하세요.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 공구관리 데이터의 포트 번호(조작반 `POT`)이며, 공구가 스핀들 위치나 대기 위치에 있으면 `0` 입니다. Siemens 에서 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `0` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

**Heidenhain** 은 포켓 테이블에서 그 공구가 든 매거진 행의 포켓 번호입니다. 스핀들에 있거나(`toolLocationType` 이 `"buffer"`) 표에 없으면 `0` 입니다. 스핀들에 있는 동안 원래 포켓에 남는 번호는 지금 자리가 아니라 `/machine/toolArea/tool/originalPocketNumber` 가 답합니다.

## /machine/toolArea/tool/originalMagazineNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
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

**Siemens·Heidenhain** 이 답합니다 (Siemens 는 `toolMyMag`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 디메시는 두 기종에서 원래 자리를 읽지 않습니다 (Fanuc 은 스핀들이나 대기 자리에 오르면 `magazineNumber`·`pocketNumber` 가 `0` 이 되고 `toolLocationType` 만 그 사실을 말합니다).

**Heidenhain** 은 포켓 테이블에서 찾습니다. 스핀들에 올라간 공구는 포켓 테이블이 원래 포켓에 그 번호를 남겨 자리를 잡아 두므로 그 자리의 매거진 번호이고 (Siemens 의 실측 표와 같은 모양, 시뮬레이터에서 `TOOL CALL` 로 확인), 매거진에 있는 공구는 `/machine/toolArea/tool/magazineNumber` 와 같고, 표에 없으면 `0` 입니다. TNC7 사용 설명서('Pocket table tool_p.tch')는 공구가 스핀들에 있는 동안 그 포켓을 예약해 두는 것을 상자형 매거진의 동작으로 설명합니다. 예약하지 않는 구성이나 손으로 넣은 공구는 스핀들에 있는 동안 `0` 일 수 있습니다 (테스트 환경에서는 확인하지 못했습니다).

## /machine/toolArea/tool/originalPocketNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

그 공구가 **돌아갈 포켓의 번호**입니다. `toolArea` + `tool` 필터. 반환 `int`.

`originalMagazineNumber` 와 한 쌍이라 동·호수를 이룹니다. 규칙은 그쪽과 같습니다: 매거진에 있는 동안에는 `pocketNumber` 와 값이 같고, 스핀들·그리퍼에 올라가면 `pocketNumber` 는 `0` 이 되는데 이 값은 원래 포켓을 유지합니다. 조작반 공구 상세의 `Orig. location` 입니다.

자리가 배정되지 않은 공구는 `0`, 없는 공구는 상태 `-18` 입니다.

**Siemens·Heidenhain** 이 답합니다 (Siemens 는 `toolMyPlace`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 디메시는 두 기종에서 원래 자리를 읽지 않습니다 (Fanuc 은 스핀들이나 대기 자리에 오르면 `magazineNumber`·`pocketNumber` 가 `0` 이 되고 `toolLocationType` 만 그 사실을 말합니다).

**Heidenhain** 은 포켓 테이블에서 찾습니다. 스핀들에 올라간 공구는 포켓 테이블이 원래 포켓에 그 번호를 남겨 자리를 잡아 두므로 그 자리의 포켓 번호이고 (Siemens 의 실측 표와 같은 모양, 시뮬레이터에서 `TOOL CALL` 로 확인), 매거진에 있는 공구는 `/machine/toolArea/tool/pocketNumber` 와 같고, 표에 없으면 `0` 입니다. TNC7 사용 설명서('Pocket table tool_p.tch')는 공구가 스핀들에 있는 동안 그 포켓을 예약해 두는 것을 상자형 매거진의 동작으로 설명합니다. 예약하지 않는 구성이나 손으로 넣은 공구는 스핀들에 있는 동안 `0` 일 수 있습니다 (테스트 환경에서는 확인하지 못했습니다).

## /machine/toolArea/magazineCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

그 공구 영역의 **매거진 수**입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

**실물 공구 저장고만 셉니다.** Siemens 는 내부에서 스핀들·교환기와 반입출 위치도 매거진으로 취급하지만(그렇게 세면 840D sl 벤치가 `3`) 디메시는 그 자리들을 매거진으로 보지 않습니다. 공구가 거기 있으면 `/machine/toolArea/tool/toolLocationType` 이 `"buffer"` / `"loading"` 으로 답합니다.

**개수는 알려주지만 번호는 알려주지 않습니다.** 번호가 연속이라는 보장이 없어 이 값으로부터 유효한 매거진 번호를 유추할 수 없습니다. 번호가 필요하면 `/machine/toolArea/magazineList` 를 쓰세요. 개수만 필요할 때 목록 전체를 받지 않아도 되게 이 주소를 따로 둡니다.

Siemens 에서 공구 관리 기능이 없는 장비는 매거진 자체가 없어 `0` 입니다.

**Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. 옵션 없이는 매거진 데이터 자체가 없으므로 `0` 이라 답하지 않습니다. Fanuc 의 매거진 구성은 파라미터 `13222`/`13227`/`13232`/`13237`(매거진 `1`~`4` 의 포트 수)로 정해지며 포트 수가 `0` 이 아닌 것만 셉니다 (카트리지 관리 표는 번호 `1`~`8` 을 허용하지만 파라미터로 구성되는 것은 이 넷입니다). 스핀들 위치(`11`~`14`)와 대기 위치(`21`~`24`)는 매거진 번호를 갖지만 매거진이 아니라 세지 않습니다. `toolArea` 는 경로 번호지만 Fanuc 의 공구관리 표는 CNC 전역이라 어느 경로로 물어도 같은 값입니다.

**Mitsubishi 는 매거진 번호가 `1`~`5` 로 고정 범위**이고, 그 중 포켓이 하나라도 있는 것만 셉니다. 스핀들과 대기 자리는 이 기종에서 매거진이 아니라 별도 개념이라 애초에 세어지지 않습니다. `toolArea` 는 `1` 부터 채널 수까지 받으며, 매거진은 계통에 딸리지 않고 기계 전체라 어느 값으로 물어도 같은 답입니다.

**Heidenhain** 은 포켓 테이블(조작반 공구 관리 화면의 `MAGAZIN`·`P` 열)의 행 이름 `매거진.포켓` 에서 매거진 번호(`1` 이상)의 개수입니다. 스핀들 행(`0.0`)은 매거진이 아니라 세지 않습니다. 포켓 테이블이 없다고 답하는 제어기는 `0` 입니다. `toolArea` 는 `1` 뿐입니다.

## /machine/toolArea/magazineList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

그 공구 영역의 **매거진 목록**입니다. `toolArea` 필터. 반환 `objectArray`, **읽기 전용**.

**매거진 번호가 연속이라는 보장이 없으므로 이 목록으로 확인하세요.** Siemens 는 840D sl 벤치의 내부 매거진 번호가 `1`·`9998`·`9999` 였고 이 목록에는 그중 실물 저장고인 `1` 만 나옵니다 (Mitsubishi 는 `1`~`5` 중 포켓이 있는 매거진만 나오므로 번호가 건너뛸 수 있습니다. Mitsubishi 의 `toolArea` 는 `1` 부터 채널 수까지 받으며, 매거진은 계통에 딸리지 않고 기계 전체라 어느 값으로 물어도 같은 답입니다.) Fanuc 은 파라미터로 구성되는 `1`~`4` 중 설정된 것만 나옵니다 (파라미터 `13222`/`13227`/`13232`/`13237` 의 포트 수가 `0` 이 아닌 매거진. 매트릭스형 매거진(파라미터 `13240`)은 `pocketCount` 가 행×열, `13241`×`13242` 등). **Fanuc 은 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만** 지원하며, 없으면 상태 `-20` 입니다. `1` 부터 `/machine/toolArea/magazineCount` 까지 세어 올라가면 찾지 못합니다.

각 항목:

| 필드 | 뜻 |
|---|---|
| `magazineNumber` | 매거진 번호. `/machine/toolArea/magazine/pocketCount` 의 `magazine` 필터에 그대로 넣습니다 |
| `pocketCount` | 그 매거진의 포켓 수 |

**실물 공구 저장고만 담습니다.** Siemens 는 내부에서 스핀들·교환기와 반입출 위치도 매거진으로 취급하고 고정 번호(버퍼 `9998`·로딩 `9999`)를 붙이지만, 디메시에서 "매거진" 은 공구 저장고 하나만 뜻합니다. 공구가 그런 자리에 있으면 `/machine/toolArea/tool/toolLocationType` 이 `"buffer"` / `"loading"` 으로 답하고 `/machine/toolArea/tool/magazineNumber` 는 `0` 을 줍니다.

**공구와 그대로 맞물립니다.** `/machine/toolArea/tool/magazineNumber` 가 돌려준 값으로 이 목록의 항목을 찾고, 그 번호를 `pocketCount` 에 그대로 넣을 수 있습니다.

**매거진 이름은 담지 않습니다.** 장비가 이름 필드를 갖고 있지만 현장에서 설정하지 않으면 뜻이 없습니다. 840D sl 벤치에서는 40자리 매거진과 버퍼와 반입출 위치가 **모두 같은 문자열**을 돌려줬습니다. 세 항목에 같은 이름이 붙으면 목록이 고장난 것처럼 보이므로 넣지 않았습니다.

매거진이 하나도 구성되지 않은 장비는 `[]` 이고, 공구관리 옵션 자체가 없는 Fanuc 장비는 상태 `-20` 입니다.

**Heidenhain** 은 포켓 테이블의 매거진 번호마다 한 항목이고 `pocketCount` 는 그 매거진의 행 수입니다 (시뮬레이터: `[{"magazineNumber": 1, "pocketCount": 50}]`). 스핀들 행(`0.0`)은 담지 않습니다.

## /machine/toolArea/magazine/pocketCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

그 매거진의 **포켓 수**입니다. `toolArea` + `magazine` 필터 (`magazine` 은 매거진 번호). 반환 `int`, **읽기 전용**.

**매거진 번호는 연속이 아닙니다.** 유효한 번호는 `/machine/toolArea/magazineList` 가 알려줍니다. `1` 부터 `/machine/toolArea/magazineCount` 까지 세어 올라가는 방식으로는 찾을 수 없습니다.

없는 번호는 상태 `-18` 로 거절됩니다. **스핀들·교환기·반입출 위치의 번호도 거절됩니다.** 장비는 그 자리들에도 매거진 번호를 붙이지만 디메시는 매거진으로 보지 않습니다. 범위·콤마 확장을 지원합니다.

**Fanuc**: 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만 지원하며 없으면 상태 `-20`, 설정되지 않은 매거진 번호는 상태 `-18`(설정된 번호 목록 동봉)입니다. 값은 파라미터 `13222`/`13227`/`13232`/`13237` 이고, 매트릭스형(파라미터 `13240`)은 행×열입니다. 포켓 번호는 `1` 이 아니라 **시작 포트 번호**(파라미터 `13223` 등)부터 이어지므로 포켓 번호의 범위는 `/machine/toolArea/magazine/pocketList` 로 확인하세요.

**Mitsubishi 주의**: 매거진 번호가 고정 범위 `1`~`5` 라 그 밖은 상태 `-18` 이지만, **범위 안의 실재하지 않는 매거진은 거절 대신 `0` 으로 옵니다**. 이 기종의 매거진 존재 판정이 곧 포켓 수 조회라서입니다. 실재 여부가 필요하면 `/machine/toolArea/magazineList` 를 보세요. `toolArea` 는 `1` 부터 채널 수까지 받으며, 매거진은 계통에 딸리지 않고 기계 전체라 어느 값으로 물어도 같은 답입니다.

**Heidenhain** 은 포켓 테이블에서 그 매거진의 행 수입니다. 포켓 번호는 행 이름 그대로라 `1` 부터 이 수까지 이어진다는 보장이 없습니다 (시뮬레이터: 행 50개가 `1`~`41`·`50`~`58`). 번호는 `/machine/toolArea/magazine/pocketList` 로 확인하세요. `magazine=0` 은 스핀들 행이라 상태 `-18` 입니다.

## /machine/toolArea/magazine/pocketList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

그 매거진의 **포켓을 전부, 포켓마다 무엇이 들어 있는지**입니다. `toolArea` + `magazine` 필터. 반환 `objectArray`, **읽기 전용**.

각 항목:

| 필드 | 뜻 |
|---|---|
| `pocketNumber` | 포켓 번호. 빠짐없이 나옵니다. Siemens·Mitsubishi 는 `1` 부터 `/machine/toolArea/magazine/pocketCount` 까지, Fanuc 은 시작 포트 번호(파라미터 `13223` 등, 보통 `1`)부터 포켓 수만큼, Heidenhain 은 포켓 테이블의 행 그대로(건너뛸 수 있음) |
| `toolNumber` | 그 포켓에 든 공구 번호. **`0` 이면 빈 포켓** |

**다른 주소들이 답하지 못하는 방향입니다.** `/machine/toolArea/tool/pocketNumber` 는 "이 공구가 몇 번 포켓에 있나" 를 답하지만, "몇 번 포켓에 뭐가 있나" 와 "빈 포켓이 어디인가" 는 이 목록만 답합니다.

공구 이름·오프셋은 담지 않습니다. `/machine/toolArea/toolList` 가 번호로 그것들을 주므로 번호로 이어 붙이세요. 포켓마다 이름을 함께 읽으면 40포켓 매거진에서 읽는 값이 두 배가 됩니다.

버퍼(스핀들·교환기)와 반입출 위치의 번호는 상태 `-18` 로 거절됩니다. 디메시는 그 자리들을 매거진으로 보지 않습니다.

**Fanuc**: 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만 지원하며(없으면 상태 `-20`), 매거진 관리 테이블(`cnc_rdmagazine`)을 매거진 통째로 한 번에 읽습니다. `toolNumber` 는 공구관리 데이터 번호(조작반 `NO.` 열, `tool` 필터에 넣는 값)입니다. 대형 공구가 차지한 이웃 포켓과 예약된 원위치 포켓(각각 대형 공구 지원 옵션·확장 B 옵션)도 공구가 든 것은 아니라 `0` 으로 나오므로, 공구를 넣을 빈 자리를 고를 때는 조작반의 매거진 화면도 함께 확인하세요. 매트릭스형 매거진(파라미터 `13240`)도 같은 시작 포트 번호부터 행×열 개수만큼 이어집니다 (Fanuc Connection Manual B-64483EN-1: 매거진 앞에서 보아 왼쪽 위에서 오른쪽 아래로 번호가 매겨집니다). 테스트 벤치는 체인형이라 매트릭스형은 문서 기준입니다.

**Mitsubishi**: 매거진 번호는 고정 범위 `1`~`5` 이며, 실재하지 않는 매거진은 상태 `-18` 로 거절됩니다. `toolArea` 는 `1` 부터 채널 수까지 받으며, 매거진은 계통에 딸리지 않고 기계 전체라 어느 값으로 물어도 같은 답입니다.

포켓이 없는 매거진은 Siemens 에서 `[]` 입니다. Fanuc·Mitsubishi 는 포켓이 있는 매거진만 매거진으로 보므로, 그런 번호는 상태 `-18` 로 거절합니다.

**Heidenhain** 은 포켓 테이블(조작반 공구 관리 화면의 `MAGAZIN`·`P` 열)에서 그 매거진의 행을 포켓 번호 순으로 담습니다. `toolNumber` 는 그 행의 공구 번호(`T`)이고, **스핀들에 올라간 공구의 원래 자리는 `0`** 입니다. 공구를 스핀들에 올려도 포켓 테이블은 원래 포켓에 그 번호를 남겨 자리를 잡아 두는데(시뮬레이터에서 `TOOL CALL` 로 확인), 실제로 공구가 든 것은 아니기 때문입니다. 그 자리는 공구 쪽의 `/machine/toolArea/tool/originalPocketNumber` 가 알려 줍니다. 포켓 번호는 행 이름 그대로라 건너뛸 수 있습니다 (시뮬레이터: `1`~`41`·`50`~`58`). `magazine=0` 은 스핀들 행이라 상태 `-18` 입니다. `toolNumber` 가 `0` 이어도 공구를 넣을 수 있는 자리라는 뜻은 아닙니다: 잠긴 포켓(`/machine/toolArea/magazine/pocket/pocketDisabledOn`)이거나, 설명서('Pocket table tool_p.tch')에 따르면 특수 공구의 이웃 포켓이나 상자형 매거진에서 위·아래·좌·우 포켓을 막는 칸(`LOCKED_*`)으로 막힌 자리일 수 있습니다.

## /machine/toolArea/magazine/pocket/toolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

그 포켓에 든 **공구 번호**입니다. `toolArea` + `magazine` + `pocket` 필터. 반환 `int`, **읽기 전용**.

**`0` 은 빈 포켓**입니다 (공구 번호는 `1` 부터). 없는 포켓은 `0` 이 아니라 상태 `-18` 로 거절되므로 둘이 섞이지 않습니다. 유효한 포켓 범위는 `/machine/toolArea/magazine/pocketCount` 가 알려줍니다. Fanuc 은 포켓 번호가 시작 포트(파라미터 `13223` 등)부터 이어지므로 범위는 `/machine/toolArea/magazine/pocketList` 로 확인하세요.

포켓을 하나만 볼 때 쓰고, 매거진 전체를 훑을 때는 `/machine/toolArea/magazine/pocketList` 가 요청 한 번으로 끝냅니다.

버퍼(스핀들·교환기)와 반입출 위치의 번호는 상태 `-18` 로 거절됩니다.

**Fanuc**: 공구관리(TOOL MANAGEMENT) 옵션이 있는 장비에서만 지원하며 없으면 상태 `-20` 입니다. 값은 공구관리 데이터 번호(`tool` 필터에 넣는 값)이고, 대형 공구가 차지한 이웃 포켓과 예약된 원위치 포켓도 `0` 으로 나옵니다 (`/machine/toolArea/magazine/pocketList` 참조). 매트릭스형 매거진도 같은 방식입니다 (포켓 번호 체계는 `/machine/toolArea/magazine/pocketList` 참조).

**Mitsubishi**: `toolArea` 는 `1` 부터 채널 수까지 받으며, 매거진은 계통에 딸리지 않고 기계 전체라 어느 값으로 물어도 같은 답입니다.

**쓰기는 지원하지 않습니다.** 포켓의 공구를 고쳐 쓰면 실물은 그대로인 채 장부만 바뀌어, 다음 공구 교환 때 교환기가 엉뚱한 포켓을 집습니다. 공구 이동은 매거진 명령의 몫이며 디메시는 그 명령을 노출하지 않습니다.

**Heidenhain** 은 포켓 테이블에서 그 행의 공구 번호(`T`)입니다. 스핀들에 올라간 공구의 원래 자리는 `0` 입니다 (`/machine/toolArea/magazine/pocketList` 참조). 표에 없는 포켓 번호는 상태 `-18` 이고, `magazine=0` 은 스핀들 행이라 상태 `-18` 입니다. `toolNumber` 가 `0` 이어도 공구를 넣을 수 있는 자리라는 뜻은 아닙니다: 잠긴 포켓(`/machine/toolArea/magazine/pocket/pocketDisabledOn`)이거나, 설명서('Pocket table tool_p.tch')에 따르면 특수 공구의 이웃 포켓이나 상자형 매거진에서 위·아래·좌·우 포켓을 막는 칸(`LOCKED_*`)으로 막힌 자리일 수 있습니다.

## /machine/toolArea/magazine/pocket/pocketDisabledOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

그 포켓이 **쓰지 말라고 표시되어 있는지** 여부입니다. 손상되었거나 비워 둬야 하는 자리입니다. `toolArea` + `magazine` + `pocket` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원합니다. `{"value": true}` 로 잠그고 `false` 로 풉니다.

`true` 면 장비가 공구를 넣을 자리를 고를 때 이 포켓을 건너뜁니다. Siemens 는 매거진 화면의 `D` 열이 이 값이며, 그 화면에서 이 칸은 **포켓을 차지한 공구 행에만** 나타납니다. 포켓의 속성이지 공구의 속성이 아니기 때문입니다. Heidenhain 은 포켓 테이블의 잠금 칸(`L`)입니다 (아래).

공구 쪽의 잠금은 `/machine/toolArea/tool/toolUseStatus`(값 `5`) 이고 별개입니다. 포켓이 잠겨도 그 안의 공구는 잠긴 것이 아니며, 다른 자리로 옮기면 다시 쓸 수 있습니다.

이미 그 상태면 아무것도 하지 않고 성공합니다. 없는 포켓과 버퍼·반입출 위치의 번호는 상태 `-18` 로 거절됩니다.

**Siemens·Heidenhain** 에서 지원합니다. Fanuc 은 이 정보가 공구관리 확장 B 옵션(`cnc_rdpot_property`)에 있고 디메시는 그 통로를 쓰지 않아 상태 `-20` 입니다.

**Mitsubishi 는 상태 `-20` 입니다.**

**Heidenhain** 은 포켓 테이블의 잠금 칸(`L`)이고 읽기·쓰기 모두 지원합니다 (시뮬레이터에서 쓰고 다시 읽어 확인). 이 칸만 봅니다: 상자형 매거진에서 이웃 포켓의 `LOCKED_*` 칸으로 막힌 자리는 `false` 입니다 (테스트 환경에는 없어 확인하지 못했습니다). 포켓 테이블을 다루는 방식은 기계에 따라 다릅니다 (TNC7 사용 설명서 'Configuring a tool': 기계 제작사의 기능이나 외부 공구 관리 시스템이 다루기도 합니다). 그런 장비에서는 쓴 값이 바뀌거나 그 체계와 어긋날 수 있으니 기계 설명서를 확인하세요. 표에 없는 포켓 번호는 상태 `-18` 이고, `magazine=0` 은 스핀들 행이라 상태 `-18` 입니다.

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

**칸 수이지 쓰이고 있는 그룹의 수가 아닙니다.** 대부분의 칸은 비어 있고, 쓰이는 번호도 띄엄띄엄합니다. 실장비에서 이 값이 `64` 인데 공구가 등록된 그룹은 `1` 과 `60` 둘뿐이었습니다. 어느 그룹이 쓰이는지는 `/machine/toolArea/registeredToolGroupList` 가 한 번에 알려 줍니다.

**Fanuc 에서는 이 주소로 옵션 유무를 알 수 있습니다.** 상태 `-20`(미지원)이 아니면 그 장비에 공구수명관리가 있는 것입니다. `/machine/toolArea/toolCount` 가 공구관리(Tool Management)에 대해 같은 구실을 하므로, 두 주소를 한 번씩 읽으면 그 Fanuc 장비가 어느 공구 기능을 갖췄는지 정해집니다. Mitsubishi 에서는 이 주소가 늘 상태 `-20` 이라 이 방법을 쓸 수 없고, 공구 수명 관리는 `registeredToolGroupList` 가 답합니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 의 공구 수명 그룹 번호는 정해진 칸이 아니라 `1`~`99999999` 에서 고르는 번호라 칸 단위로 답할 수 없습니다. 쓰이는 그룹은 `registeredToolGroupList` 로 확인하세요.

## /machine/toolArea/registeredToolGroupList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
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

**그룹 칸이 할당되지 않은 장비도 옵션이 없는 장비와 같이 상태 `-20`** 입니다. `[]` 는 기능이 있고 등록된 그룹이 없다는 뜻이라 구분됩니다. `cnc_rdgrpinfo4` 를 쓰므로 쓰고 있는 FOCAS 라이브러리에 그 함수가 없으면 상태 `-20` 입니다. 제어기가 그 호출을 거절하면 그 거절 사유에 맞는 상태로 돌아옵니다.

**Siemens 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다.

**Mitsubishi** 는 공구 수명 관리의 그룹 번호를 번호가 작은 순으로 답합니다 (시뮬레이터에서 조작반 `툴수명` 의 등록 그룹 일람과 대조). 공구 수명 그룹은 파트 시스템마다 따로라 `toolArea` 가 파트 시스템 번호입니다 (`1` 부터 채널 수까지. 기계 전체를 답하는 매거진 주소와 다릅니다). 공구가 없는 그룹은 담기지 않습니다. 그룹 번호는 정해진 칸이 아니라 `1`~`99999999` 에서 고르는 번호라 `toolGroupCount`·`toolGroupToolCountList` 는 상태 `-20` 이고, 쓰이는 그룹은 이 목록으로 확인합니다. 그 파트 시스템의 공구 수명 그룹을 읽을 수 없으면 상태 `-20` 입니다.

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

**그룹 칸이 할당되지 않은 장비도 옵션이 없는 장비와 같이 상태 `-20`** 입니다. `[]` 는 기능이 있고 지금 교환 요구가 없다는 뜻이라 구분됩니다. 이 함수를 지원하지 않는 제어기도 상태 `-20` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

`0` 은 **그 그룹에 등록된 공구가 없다**는 뜻입니다. 실장비에서는 그런 칸이 조작반에 `타입: 데이터 없음` 으로 떴으므로 새 그룹을 만들 자리로 보면 되지만, 이 값 자체가 약속하는 것은 공구 수뿐입니다.

그룹 하나만 필요하면 `/machine/toolArea/toolGroup/toolCount` 를 쓰세요. 이 목록은 전체를 한 번에 받는 쪽입니다.

**`cnc_rdgrpinfo4` 를 쓰므로 쓰고 있는 FOCAS 라이브러리에 그 함수가 없으면 상태 `-20`** 입니다 (제어기가 그 호출을 거절하면 그 거절 사유에 맞는 상태로 돌아옵니다). 그 경우에도 `/machine/toolArea/toolGroupCount` 는 정상입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 의 공구 수명 그룹 번호는 정해진 칸이 아니라 `1`~`99999999` 에서 고르는 번호라 칸 단위로 답할 수 없습니다. 쓰이는 그룹은 `registeredToolGroupList` 로 확인하세요.

길이는 `toolGroupCount`(그룹 칸 수)로 고정이라 빈 배열은 나오지 않습니다. 비어 있는 그룹 칸은 `0` 입니다.

## /machine/toolArea/toolGroup/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹에 **등록된 공구의 수**입니다. `toolArea` + `toolGroup` 필터. 반환 `int`, 읽기 전용입니다.

**빈 그룹은 `0`** 입니다. 그룹 번호 자체가 장비의 범위를 벗어나면 상태 `-18` 로 거절되므로, `0` 은 "그런 그룹이 없다" 가 아니라 "그 그룹에 공구가 없다" 는 뜻입니다.

그룹 번호의 상한은 `/machine/toolArea/toolGroupCount` 가 알려줍니다. 다만 그 칸들이 다 쓰이는 것은 아니라, 훑으면 대부분 `0` 이 나옵니다.

**쓸 그룹이 없는 장비는 상태 `-20`** 입니다. 공구수명관리(tool life management) 옵션이 없거나, 옵션은 있으나 그룹 칸이 할당되지 않은 경우입니다.

**Siemens 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다.

**Mitsubishi** 는 공구 수명 관리 그룹에 등록된 공구의 수입니다 (`toolNumberList` 의 길이, 시뮬레이터에서 확인). `toolArea` 는 파트 시스템 번호입니다. 등록되지 않은 그룹 번호는 `0`, `1`~`99999999` 밖의 번호는 상태 `-18` 입니다. 그룹 번호의 상한을 알려 주는 `toolGroupCount` 는 Mitsubishi 에서 상태 `-20` 이니(그룹 번호가 정해진 칸이 아닙니다), 쓰이는 그룹은 `registeredToolGroupList` 로 확인하세요.

## /machine/toolArea/toolGroup/toolNumberList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹에 등록된 **공구 번호 목록**입니다. `toolArea` + `toolGroup` 필터. 반환 `intArray`, 읽기 전용입니다.

Fanuc 은 **쓰이는 순서대로** 담습니다. 번호순이 아닙니다. 실장비의 한 그룹이 `[16,13,2]` 였는데, `16` 번을 먼저 쓰고 수명이 다하면 `13` 번, 그다음 `2` 번으로 넘어간다는 뜻입니다.

빈 그룹은 `[]` 입니다. 항목 수는 `/machine/toolArea/toolGroup/toolCount` 와 같고, 두 주소를 함께 요청하면 장비 왕복 한 번으로 처리됩니다.

같은 그룹의 `/machine/toolArea/toolGroup/toolHNumberList`·`/machine/toolArea/toolGroup/toolDNumberList`·`/machine/toolArea/toolGroup/toolLifeStatusList` 도 **길이와 순서가 언제나 이 목록과 같습니다.** 같은 자리끼리 짝지으면 한 공구의 정보가 됩니다. Fanuc 에서는 넷을 함께 요청해도 왕복은 하나입니다.

**쓸 그룹이 없는 장비는 상태 `-20`** 입니다. 공구수명관리(tool life management) 옵션이 없거나, 옵션은 있으나 그룹 칸이 할당되지 않은 경우입니다.

**Siemens 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다.

**Mitsubishi** 는 공구 수명 관리 그룹에 등록된 공구 번호를 **등록 순서**로 답합니다. 예비 공구를 고르는 공구 수명 관리 II(파라미터 `#1096 T_Ltyp` 가 `2`)에서 `#1105 T_sel2` 가 `0` 이면 예비 공구가 이 순서로 선택되어 쓰이는 순서와 같고(프로그래밍 매뉴얼 머시닝센터편 IB-1501278), `1` 이면 제어기가 그룹에서 남은 수명이 가장 긴 공구를 고르므로 쓰이는 순서가 아닙니다 (알람/파라미터 매뉴얼 IB-1501279). 시뮬레이터에서 조작반 `툴수명` 그룹 화면의 `#` 순서와 대조했습니다 (`[3, 2, 4]`). `toolArea` 는 파트 시스템 번호입니다. 등록되지 않은 그룹 번호는 `[]`, `1`~`99999999` 밖의 번호는 상태 `-18` 입니다. 같은 그룹의 `toolLifeStatusList` 는 이 목록과 같은 순서라 짝지을 수 있고(EZSocket `FCSB1224W100-A9` 이상), `toolHNumberList`·`toolDNumberList` 는 Mitsubishi 에서 상태 `-20` 입니다.

## /machine/toolArea/toolGroup/toolLifeMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
codes: [{"value": 0, "name": "no monitoring"}, {"value": 1, "name": "time"}, {"value": 2, "name": "count"}]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹의 **수명 감시 방식**입니다. `toolArea` + `toolGroup` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 2}`.

| 값 | 뜻 | 수명 값의 단위 |
|---|---|---|
| `0` | 감시 없음 (공구가 등록되지 않은 칸) | 없음 |
| `1` | 시간 | 분 (`unit` 은 `"min"`) |
| `2` | 개수 | 회 (`unit` 은 `"count"`) |

`/machine/toolArea/tool/toolLifeMonitorType` 과 **같은 어휘**입니다. 그쪽은 공구마다, 이쪽은 그룹마다 정해집니다. 이 제어기에는 `3`(마모)이 없습니다.

`0` 은 공구가 등록되지 않은 칸일 때만입니다. 그룹 정보(`cnc_rdgrpinfo4`)를 읽지 못하면 `0` 이 아니라 그 에러로 답하고, 쓰고 있는 FOCAS 라이브러리에 그 함수가 없으면 읽기·쓰기 모두 상태 `-20` 입니다.

쓰기는 `1`·`2` 만 받습니다. `0` 으로 되돌리는 것은 그룹을 지우는 일이라 이 주소가 하는 일이 아닙니다.

⚠️ **방식을 바꾸면 제어기가 그 그룹의 수명 총량과 사용량을 `0` 으로 되돌립니다** (실장비에서 확인). 디메시는 두 값을 그대로 실어 보내며, 되돌리는 것은 제어기입니다. 값이 환산되지도 남지도 않으므로, 방식을 바꾼 뒤 `/machine/toolArea/toolGroup/toolGroupLifeTotal` 과 `/machine/toolArea/toolGroup/toolGroupLifeUsed` 를 다시 쓰세요.

**공구가 등록된 그룹만 쓸 수 있습니다.** 빈 그룹에 쓰면 상태 `-18` 입니다. 벤더가 받아준다면 그룹 자체가 생기는 것이라, 이 주소가 약속한 "수정" 밖입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

**단위는 그룹마다 다를 수 있어 응답의 `unit` 에 실립니다.** 시간 방식이면 `"min"`, 횟수 방식이면 `"count"`. 값은 **조작반 화면에 뜨는 그대로**이며 초로 환산하지 않습니다. 기계 전체 기본값은 파라미터 `6800#2` 가 정하지만, FOCAS2 스펙은 M 계열에서 그룹마다 따로 지정할 수 있다고 밝히므로, 디메시가 그룹에 직접 물어 붙입니다.

감시 방식(`/machine/toolArea/toolGroup/toolLifeMonitorType`)을 바꾸면 제어기가 이 값을 `0` 으로 되돌리고(31i-B 실장비에서 확인), 단위도 바뀝니다(시간 방식은 `min`, 횟수 방식은 `count`). 방식을 바꾼 뒤에 새 방식의 단위로 쓰세요.

빈 그룹은 `0` 이고 그 `0` 에는 `unit` 이 붙지 않습니다. 감시 종류를 읽지 못한 채 값이 `0` 이 아니면 단위를 붙일 수 없어 상태 `-17` 입니다.

**쓰기는 정수만 받습니다.** 장비 필드가 정수라 소수는 상태 `-16` 으로 거절합니다. 상한도 장비가 정하며(실장비에서 읽은 상한은 횟수 `65535`, 분 `4300`) 넘으면 상태 `-16` 입니다. **공구가 등록된 그룹만** 쓸 수 있습니다. 쓰기 전에 그룹 정보(`cnc_rdgrpinfo4`)를 읽으며, 그 읽기가 실패하면 그 에러로, 쓰고 있는 FOCAS 라이브러리에 그 함수가 없으면 상태 `-20` 으로 답합니다.

**쓸 그룹이 없는 장비는 상태 `-20`** 입니다. 공구수명관리(tool life management) 옵션이 없거나, 옵션은 있으나 그룹 칸이 할당되지 않은 경우입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

남은 수명은 `/machine/toolArea/toolGroup/toolGroupLifeTotal` 에서 이 값을 빼면 됩니다. 단위 규칙과 상태 `-20` 조건, 빈 그룹의 `0` 에 `unit` 이 붙지 않는 것과 감시 종류를 읽지 못했을 때의 상태 `-17` 도 그 주소와 같습니다.

감시 방식(`/machine/toolArea/toolGroup/toolLifeMonitorType`)을 바꾸면 제어기가 이 값을 `0` 으로 되돌리고(31i-B 실장비에서 확인), 단위도 바뀝니다(시간 방식은 `min`, 횟수 방식은 `count`). 방식을 바꾸기 전에 읽어 둔 값을 다시 쓰려면 새 방식의 단위로 적어야 합니다.

**쓰기 제약**은 `toolGroupLifeTotal` 과 같습니다. 정수만, 장비 상한 이내, 공구가 등록된 그룹만. 쓰기 전에 그룹 정보(`cnc_rdgrpinfo4`)를 읽으며, 그 읽기가 실패하면 그 에러로, 쓰고 있는 FOCAS 라이브러리에 그 함수가 없으면 상태 `-20` 으로 답합니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

## /machine/toolArea/toolGroup/toolLifeStatusList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
codes: [{"value": 0, "name": "no tool", "read": ["nc_focas2_fanuc"]}, {"value": 1, "name": "usable"}, {"value": 2, "name": "life expired"}, {"value": 3, "name": "skipped"}]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 공구그룹 각 공구의 **수명 상태**입니다. `toolArea` + `toolGroup` 필터. 반환 `intArray`, 읽기 전용입니다.

| 값 | 뜻 |
|---|---|
| `1` | 아직 쓸 수 있음 |
| `2` | 수명 다함 |
| `3` | 건너뜀 또는 공구 이상 |
| `0` | 그 자리에 쓸 공구가 없음 |

**이 주소는 "지금 쓰이는 공구" 를 알려주지 않습니다.** 차례를 기다리는 공구와 지금 도는 공구가 둘 다 `1` 입니다. `1` 은 "쓸 수 있음" 이지 "쓰는 중" 이 아닙니다. Fanuc 에서 어느 그룹이 쓰이는지는 `/machine/channel/activeToolGroupNumber` 가 답합니다 (Mitsubishi 는 그 주소가 상태 `-20` 입니다).

**Siemens 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다.

**Mitsubishi** 는 공구 수명 관리 그룹의 공구마다 조작반 `툴수명` 그룹 화면의 `ST` 칸을 옮깁니다. 그 아랫자리가 미사용(`0`)·사용 중(`1`)이면 `1`, 수명 도달(`2`)이면 `2`, 공구 이상 1·2(`3`·`4`, 어떤 이상인지는 기계 제조사가 정합니다)면 `3` 이고, 윗자리(기계 제조사 사양)는 보지 않습니다 (조작 매뉴얼 IB-1501274). `0` 은 나오지 않습니다. 순서는 `/machine/toolArea/toolGroup/toolNumberList` 와 같고, 공구마다 장비 왕복이 한 번씩 더 듭니다. `toolArea` 는 파트 시스템 번호이고, 등록되지 않은 그룹 번호는 `[]`, `1`~`99999999` 밖의 번호는 상태 `-18`, 공구 수명 그룹을 읽을 수 없는 파트 시스템은 상태 `-20` 입니다. EZSocket `FCSB1224W100-A9` 이상이 필요합니다. 그보다 앞선 판에서는 다른 판정보다 먼저 늘 상태 `-20` 이고, 그 판 이상을 설치하면 읽힙니다. 제어기가 돌려준 수명 값이 디메시가 읽는 배치가 아니면(다른 배치로 답하는 파트 시스템) 역시 상태 `-20` 입니다 (시뮬레이터에서 조작반과 대조).

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

이 번호로 `/machine/channel/toolOffset/toolLengthGeometry` 와 `/machine/channel/toolOffset/toolLengthWear` 를 찾아가면 실제 보정값이 나옵니다. FOCAS2 스펙은 이 칸이 T 계열에서 항상 `0` 이라고 밝힙니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

이 번호로 `/machine/channel/toolOffset/toolRadiusGeometry` 와 `/machine/channel/toolOffset/toolRadiusWear` 를 찾아가면 실제 보정값이 나옵니다. FOCAS2 스펙은 이 칸이 T 계열에서 항상 `0` 이라고 밝힙니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

그 공구그룹이 **지금 가리키고 있는 공구의 사용 순번**입니다. `toolArea` + `toolGroup` 필터. 반환 `int`, 읽기 전용이며 `1`부터 셉니다. 아직 그 그룹을 한 번도 안 썼으면 `0` 입니다.

이 번호는 조작반 편집 화면의 `번호` 열(`01`·`02`·`03`)과 같고, `/machine/toolArea/toolGroup/toolNumberList` 의 **같은 자리**를 가리킵니다 (`2` 면 목록의 두 번째).

⚠️ **"그 그룹이 지금 돌고 있다" 는 뜻이 아닙니다.** 그룹이 들고 있는 포인터라 오래 남습니다. 실측에서 공구 교환 직후 `1` 이 된 뒤 리셋에도, 전원을 껐다 켜도 `1` 이었습니다. 조작반의 `@`(사용중) 표시를 그대로 재현하려면 `/machine/channel/activeToolGroupNumber` 가 그 그룹일 때만 찍으세요.

**공구수명관리(tool life management) 옵션이 없거나 그룹 칸이 할당되지 않은 장비는 상태 `-20`** 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

**빈 그룹에 자리 `1` 을 만들면 그룹 자체가 생깁니다** (실장비에서 확인). 공구그룹을 만드는 별도 주소는 없고 이것이 그 방법입니다. 어느 번호가 비어 있는지는 `/machine/toolArea/toolGroupToolCountList` 의 `0` 인 자리가 알려줍니다. 새 그룹은 수명이 `0` 이고 감시 방식은 장비 기본값이므로, 이어서 `/machine/toolArea/toolGroup/toolLifeMonitorType` 을 먼저 쓰고 그다음 `/machine/toolArea/toolGroup/toolGroupLifeTotal` 을 쓰세요. 방식을 바꾸면 수명 총량이 `0` 으로 돌아가기 때문입니다.

**마지막 공구를 지우면 그룹도 사라집니다.** 수명과 감시 방식도 함께 지워집니다 (실측 확인).

⚠️ **자리가 밀립니다.** 지우면 뒤 공구들이 한 칸씩 당겨집니다 (실장비에서 확인). 그래서 **한 번의 조작으로 다른 자리 주소가 가리키는 대상이 전부 바뀝니다** (`toolNumber`·`toolHNumber`·`toolDNumber`·`toolLifeStatus`·`/machine/toolArea/toolGroup/currentToolUseOrder`). 여러 자리를 다룰 때는 조작 사이에 목록을 다시 읽으세요.

**만들 수 있는 자리는 맨 뒤 하나뿐입니다.** 공구가 3개인 그룹이면 자리 `4` 만 만들 수 있고, 그보다 뒤는 구멍이 생기므로 상태 `-18` 입니다. 중간에 밀어 넣는 조작은 이 주소로 표현되지 않습니다. 중간 자리는 **이미 존재**하므로 `true` 를 쓰면 위와 같이 상태 `-21` 입니다. 순서를 바꾸려면 맨 뒤에 만들고 번호들을 다시 쓰세요.

그룹이 가득 차면(`/machine/toolArea/toolGroupToolCountList` 의 그 칸이 장비 상한에 닿으면) 상태 `-23`(자리 없음) 입니다. 그 그룹에서 공구를 지운 뒤 다시 만드세요. 모든 그룹을 합쳐 등록할 수 있는 공구 수가 다 차서 제어기가 거절해도 상태 `-23` 입니다. 이때는 그 그룹에 자리가 남아 있어도 생기므로, 어느 그룹에서든 쓰지 않는 공구를 지운 뒤 다시 만드세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

`toolUseOrder` 는 그룹 안의 자리(`1`부터)이고 조작반 편집 화면의 `번호` 열과 같습니다. 그룹이 지금 가리키는 자리는 `/machine/toolArea/toolGroup/currentToolUseOrder` 가 알려줍니다.

**목록으로 한 번에 받으려면** `/machine/toolArea/toolGroup/toolNumberList` 를 쓰세요. 같은 값이고 왕복도 같습니다. 이 주소는 **한 자리를 지목해 쓰기 위한** 형태입니다.

**없는 자리는 상태 `-18`** 입니다 (그 그룹의 공구 수가 상한). 벤더가 받아준다면 그건 없던 공구가 생기는 것이라, 이 주소가 약속한 "수정" 밖입니다.

**가공 중에는 제어기가 거절할 수 있습니다.** 자동운전 중이거나 그 그룹을 지금 쓰고 있거나 다음 차례로 잡아 둔 상태면 상태 `-22`(기계 상태)에 사유가 실려 돌아오므로, 가공이 끝나거나 그룹이 바뀐 뒤 다시 시도하세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

이 번호로 `/machine/channel/toolOffset/toolLengthGeometry` 와 `/machine/channel/toolOffset/toolLengthWear` 를 찾아가면 실제 보정값이 나옵니다. FOCAS2 스펙은 이 값이 T 계열에서 항상 `0` 이라고 밝힙니다.

**가공 중에는 제어기가 거절할 수 있습니다.** 자동운전 중이거나 그 그룹을 지금 쓰고 있거나 다음 차례로 잡아 둔 상태면 상태 `-22`(기계 상태)에 사유가 실려 돌아오므로, 가공이 끝나거나 그룹이 바뀐 뒤 다시 시도하세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

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

이 번호로 `/machine/channel/toolOffset/toolRadiusGeometry` 와 `/machine/channel/toolOffset/toolRadiusWear` 를 찾아가면 실제 보정값이 나옵니다. FOCAS2 스펙은 이 값이 T 계열에서 항상 `0` 이라고 밝힙니다.

**가공 중에는 제어기가 거절할 수 있습니다.** 자동운전 중이거나 그 그룹을 지금 쓰고 있거나 다음 차례로 잡아 둔 상태면 상태 `-22`(기계 상태)에 사유가 실려 돌아오므로, 가공이 끝나거나 그룹이 바뀐 뒤 다시 시도하세요.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

## /machine/toolArea/toolGroup/toolUseOrder/toolLifeStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
codes: [{"value": 0, "name": "no tool"}, {"value": 1, "name": "usable"}, {"value": 2, "name": "life expired"}, {"value": 3, "name": "skipped"}]
```

**Fanuc 은 공구수명관리(Tool Life Management) 옵션이 있어야 합니다.** 없는 장비에서는 상태 `-20`(미지원)으로 거절합니다.

그 자리 공구의 **수명 상태**입니다. `toolArea` + `toolGroup` + `toolUseOrder` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 1}`.

| 값 | 뜻 |
|---|---|
| `1` | 아직 쓸 수 있음 |
| `2` | 수명 다함 |
| `3` | 건너뜀 |
| `0` | 그 자리에 쓸 공구가 없음 (읽기에서만) |

**인서트를 갈고 다시 쓰려면 `1` 을 쓰세요.** `2` 인 공구를 `1` 로 되돌리는 것이 그 조작입니다.

쓰기는 `1`·`2`·`3` 만 받습니다. `0`(공구 없음)은 **삭제**를 뜻하는데 그건 이 주소가 약속한 일이 아니라 상태 `-16` 으로 거절합니다.

**가공 중에는 제어기가 거절할 수 있습니다.** 자동운전 중이거나 그 그룹을 지금 쓰고 있거나 다음 차례로 잡아 둔 상태면 상태 `-22`(기계 상태)에 사유가 실려 돌아오므로, 가공이 끝나거나 그룹이 바뀐 뒤 다시 시도하세요.

**읽기 전용 목록**은 `/machine/toolArea/toolGroup/toolLifeStatusList` 입니다.

**Siemens·Mitsubishi 는 상태 `-20` 입니다.** 공구 그룹은 공구 수명 관리(Fanuc·Mitsubishi)의 계층이라 Siemens 에는 같은 표가 없고, 이름이 같은 공구를 `sisterToolNumber` 로 묶어 대체합니다. Mitsubishi 어댑터는 공구 수명 관리 중 그룹 목록과 그룹 안 공구(`registeredToolGroupList`·`toolGroup/toolNumberList`·`toolGroup/toolCount`), 공구의 수명 값(`tool/toolLifeMonitorType`·`tool/toolLifeTotal`·`tool/toolLifeUsed`), 그룹 공구의 수명 상태(`toolGroup/toolLifeStatusList`)만 답합니다.

## /machine/toolArea/tool/toolEdgeCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

공구가 가진 **보정 세트(cutting edge)의 개수**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). Siemens 의 `numCuttEdges` 입니다.

**개수이지 가장 큰 번호가 아닙니다.** 보통은 `1` 부터 이어지지만, 조작반에서 중간 날을 지우면 **번호에 구멍이 생기고 뒤 번호는 밀리지 않습니다.** 예를 들어 `1`·`2`·`3` 중 `2` 를 지우면 남는 것은 `1` 과 `3` 이고 이 값은 `2` 가 됩니다. 그래서 `toolEdge` 를 `1`~이 값으로 가정하면 안 됩니다.

없는 날을 가리키는 `toolEdge/…` 주소는 읽기·쓰기 모두 상태 `-18` 로 거절되므로, 어떤 번호가 실재하는지는 읽어 보면 알 수 있습니다 (이 주소 자체는 읽기 전용입니다). 예외는 `toolEdge/toolEdgeExists` 로, 없는 날을 물으면 `false` 이고 없는 날에 `false` 를 쓰는 것은 성공입니다.

**인선 개수와는 다른 값입니다.** "2날 볼엔드밀", "4날 엔드밀" 이라 할 때의 그 날은 물리적 인선 수이고, 이 값은 제어기가 그 공구에 대해 갖고 있는 보정 세트의 수입니다. 인선이 여럿이어도 모두 같은 높이·반경이면 보정 세트는 하나면 됩니다. 실측 예로 4날 커터가 `1`, 2날 볼엔드밀이 `3` 을 답했습니다.

없는 공구를 지정하면 상태 `-18` 로 거절됩니다. 보정 세트가 `0` 개라고 답하지 않습니다.

**Siemens·Heidenhain** 에서 지원합니다 (Fanuc·Mitsubishi 는 상태 `-20`). Fanuc·Mitsubishi 의 오프셋 모델은 오프셋(세트) 번호 하나가 곧 보정값 한 벌이라 공구에 딸린 날이라는 계층이 없습니다. 예전 판은 고정 `1` 을 냈는데, 없는 차원을 있다고 답하는 값이라 뺐습니다. Fanuc 공구관리의 공구별 데이터(`H`·`D`·수명)는 `/machine/toolArea/tool/*` 의 공구 단위 주소로 읽으세요.

**Heidenhain** 은 그 공구 번호의 행 수입니다: 공구 자신의 행 하나에 인덱스 공구(`5.1`·`5.2` 처럼 공구 번호 뒤에 붙는 행)를 더한 수이고, 인덱스가 없는 공구는 `1` 입니다. **`toolEdge` 는 그 인덱스이고 `0` 부터입니다**: 공구 자신의 행이 `toolEdge=0`, `5.1` 이 `toolEdge=1` 입니다 (조작반에 보이는 번호 그대로). **인덱스 번호에는 구멍이 생길 수 있습니다**: 테스트 환경의 조작반에서 `10.2` 없이 `10.3` 을 넣을 수 있었고 (TNC7 사용 설명서 'Indexed tool' 도 번호가 이어지지 않아도 된다고 적고, 공구 하나에 인덱스 공구는 9개까지라 이 값은 `10` 까지입니다), 그때 이 값은 `3` 이지만 `toolEdge=2` 는 없습니다. 어떤 번호가 있는지는 `/machine/toolArea/tool/toolEdge/toolEdgeExists` 로 확인하세요. 없는 공구와 테이블 맨 앞의 `0` 번 행은 상태 `-18` 입니다.

## /machine/toolArea/tool/toolEdge/toolEdgeExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens"]
```

그 날 번호가 **그 공구에 있는지** 여부입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원.

없는 날을 물어도 에러가 아니라 `false` 입니다. `/machine/toolArea/tool/toolEdgeCount` 는 개수만 알려주고 번호에 구멍이 있을 수 있으므로, **어떤 번호가 실재하는지는 이 주소가 답합니다.**

**쓰기가 날을 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. 이미 있는 날에 `true` 를 쓰면 상태 `-21`(이미 존재)로 거절하고, 없는 날에 `false` 를 쓰는 것은 성공입니다. 채널이 Reset 이 아니거나 공구가 사용 중이라 날 삭제가 거절되면 디메시는 상태 `-22`(기계 상태)로 답합니다. 채널을 리셋하고 다시 지우세요. 제어기의 그 밖의 거절은 대개 상태 `-17` 입니다.

**날은 순서대로만 만들어집니다.** 장비는 **비어 있는 가장 작은 번호**에 날을 만듭니다 (번호 지정 없음). 그래서 그 번호가 아닌 것을 요청하면 만들지 않고 상태 `-18` 로 거절하며, 다음에 만들어질 번호를 에러 문구에 실어 보냅니다. `D5` 가 필요하면 `3`·`4`·`5` 를 차례로 만드세요. 구멍이 있으면 그 구멍부터 채워집니다.

**1번 날은 지울 수 없습니다** (상태 `-18`). 공구가 있는 한 남습니다. 공구째 지우려면 `/machine/toolArea/tool/toolExists` 를 쓰세요.

공구 자체가 없으면 쓰기는 `true`·`false` 모두 상태 `-18` 입니다. 공구가 없어서가 아니라 다른 이유로 읽지 못하면(연결 계정에 공구 데이터 읽기 권한이 없어 `BadUserAccessDenied` 가 오는 경우 등) 읽기·쓰기 모두 상태 `-17` 입니다.

읽기는 Siemens·Heidenhain, 쓰기는 Siemens 가 지원합니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 에서 `toolEdge` 는 공구 테이블의 인덱스입니다: `toolEdge=0` 은 공구 자신의 행이라 공구가 있으면 언제나 `true` 이고, `toolEdge=1` 부터는 `5.1` 처럼 공구 번호 뒤에 붙는 인덱스 공구의 행입니다. 없는 인덱스는 `false`, 공구 자체가 없으면(테이블 맨 앞의 `0` 번 행 포함) 상태 `-18` 입니다. 쓰기는 상태 `-20` 입니다 (디메시는 Heidenhain 에서 행을 만들거나 지우지 않습니다).

## /machine/toolArea/tool/toolEdge/toolType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

공구의 **타입 코드**입니다 (반환 `int`). **Siemens 는 SINUMERIK DP1 코드를 그대로 사용**합니다 (read + write, `desc` 동반. Heidenhain 은 다른 코드 공간이고 `desc` 가 없습니다, 아래). 코드 체계는 Siemens 소유의 열린 분류라 디메시가 번역하지 않으며, 정본은 SINUMERIK 공구 관리 매뉴얼입니다 (벤더가 코드를 추가해도 값은 그대로 전달). 쓰기는 정수 코드 `{"value": 500}` 입니다. 공구 셋업 자동화용이며 코드 유효성은 NCK 가 판정합니다.

| 계열 | 의미 | 예 |
|---|---|---|
| `1xx` | 밀링 공구 | `120` 엔드밀, `140` 페이스밀, `145` 나사 밀링 |
| `2xx` | 드릴 계열 | `200` 트위스트드릴, `240` 탭, `250` 리머 |
| `4xx` | 연삭 공구 | |
| `5xx` | 선삭 공구 | `500` 황삭, `510` 정삭, `530` 절단, `540` 나사 |
| `7xx` | 특수 | `711` 프로브, `730` 스톱 |

위 표는 **정본이 아니라 길잡이**입니다. 이 코드 체계는 Siemens 가 소유하므로, 정확한 목록은 그 기종 매뉴얼에서 확인하세요: 840D sl 은 *Tool Management Function Manual* "List of tool types", 828D 는 *Tools Function Manual*. `desc` 의 이름은 그 목록(07/2021 판)을 따르고, 조작반의 공구 목록 화면에서도 같은 번호가 보입니다.

알려진 코드는 `desc` 로 의미가 함께 오고 (`{"value": 500, "desc": "turning roughing tool"}`), 미등재 코드는 첫 자리 계열 desc 를 대신 씁니다 (`{"value": 573, "desc": "turning tool family"}`). **위 표의 계열 밖 코드는 `desc` 키 자체가 빠집니다** (`{"value": 300}`). 없는 뜻을 지어내지 않기 위해서입니다. `desc` 가 항상 있다고 가정하지 마세요.

선삭 공구(5xx)의 길이1/2 축 배정, 반경 해석(커터/노즈)을 판별하는 기준값이기도 합니다.

**쓰기 주의 (Siemens)**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 은 공구 테이블의 공구 종류(`TYP`) 번호를 그대로 냅니다. Siemens 의 코드와는 다른 코드 공간이고, 디메시는 번역하지 않고 기종 간에 통일하지 않으며 `desc` 도 붙이지 않습니다. 번호의 뜻은 TNC7 사용 설명서 'Tool types' 의 번호와 같습니다 (테스트 환경에서 밀링 공구 `9`, 드릴 `1`, NC 센터 드릴 `4`, 모따기 밀 `24`, 선삭 공구 `29` 였습니다). 조작반 공구 관리 화면의 공구 종류 표시로도 확인할 수 있습니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`5.1` 처럼)의 행 값입니다. 쓰기도 지원합니다. 번호를 그대로 쓰고, 제어기가 받지 않는 값은 상태 `-16` 이며 에러 문구에 허용 범위가 실립니다 (테스트 환경에서는 `0`~`99`). 쓴 번호가 어떤 종류인지는 조작반 공구 관리 화면의 표시로 확인하세요.

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
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

그 보정 세트의 **인선 개수**입니다. "4날 엔드밀" 이라 할 때의 그 수. `toolArea` + `tool` + `toolEdge` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 4}`.

**`toolEdgeCount` 와 다른 값입니다.** 저쪽은 제어기가 그 공구에 대해 갖고 있는 보정 세트의 수이고, 이 값은 한 세트가 기술하는 절삭날이 물리적으로 몇 개인가입니다. 실측 예로 4날 커터가 보정 세트 `1` 개에 인선 `4` 개였습니다.

**보정 세트마다 따로 저장됩니다.** 한 자루에 지름이 다른 절삭부가 둘이면 인선 수도 다를 수 있어, 공구가 아니라 날에 붙습니다.

**Siemens 장비 화면의 `N` 열과 항상 같지는 않습니다.** 그 열은 밀링 공구면 인선 수를, 드릴류면 선단각을 보여주는 겸용 칸입니다. 실측에서 드릴은 이 주소가 `0` 이고 화면엔 `118.0`(선단각)이 떴습니다. 디메시는 한 주소가 공구 종류에 따라 다른 물리량이 되지 않도록 둘을 섞지 않습니다.

**Siemens·Heidenhain** 에서 지원합니다. 없는 공구/날(D)을 지정하면 상태 `-18` 로 거절됩니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 은 공구 테이블의 날 수(`CUT`)입니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`5.1` 처럼)의 행 값입니다. 쓰기도 지원하며, 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 허용 범위가 실립니다 (테스트 환경에서는 `0`~`99`).

## /machine/toolArea/tool/toolEdge/toolLengthGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

**길이1 형상값**입니다. Siemens 에서는 SINUMERIK `DP3` 이고, 선삭 공구에서는 통상 X 방향에 대응하지만 축 대응은 공구 타입과 활성 평면이 정하는 규칙이라 SDK 는 번역하지 않습니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens·Heidenhain** 에서 지원하며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의 (Siemens)**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 은 공구 테이블의 길이(`L`)입니다 (마모는 `DL`). 표 안의 값은 형상 + 마모이고, 가공에는 NC 프로그램(`TOOL CALL` 의 델타)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Tool compensation for tool length and tool radius'). `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`5.1` 처럼)의 행입니다. **단위는 위 규칙과 달리 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (inch 로 만든 공구 테이블에서는 확인하지 못했습니다). 쓰기를 지원하며, 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (시뮬레이터에서 `-99999.9999`~`99999.9999`, 설명서 'Tool table tool.t' 의 입력 범위와 같습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행은 상태 `-18` 입니다. 조작반의 공구 테이블에서 길이 칸이 비어 있으면(공구 삽입으로 막 만든 행처럼. 설명서 'Tool management' 도 새 공구의 길이·반지름 칸은 처음에 비어 있다고 적습니다) 상태 `-22` 이고, 칸을 채우면(조작반에서, 또는 이 주소에 쓰면) 읽힙니다. **선삭·연삭·드레싱 공구는 상태 `-18` 입니다** (읽기·쓰기 모두). 디메시는 공구 종류 칸(`TYP`)으로 가립니다: TNC7 사용 설명서 'Tool types' 의 번호로 선삭 `29`·연삭 `30`·드레싱 `31` 이고, 설명서는 그 공구에는 공구 테이블의 길이·반지름이 효과가 없다고 적습니다. 테스트 환경의 선삭 공구는 공구 테이블의 `L`·`R`·`DL`·`DR` 이 모두 `0` 이었고 형상은 선삭 공구 표(`toolturn.trn`)의 `ZL`·`XL`·`YL`·`RS` 와 그 마모 칸에 있었습니다 (연삭·드레싱 공구는 시험하지 못했습니다). 선삭 공구의 형상은 `toolXGeometry`·`toolZGeometry`·`toolYGeometry`·`toolNoseRadiusGeometry` 와 그 마모 주소가 냅니다.

## /machine/toolArea/tool/toolEdge/toolLengthWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

**길이1 마모값**입니다 (SINUMERIK `DP12`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens·Heidenhain** 에서 지원하며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의 (Siemens)**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 은 공구 테이블의 길이 마모(`DL`)입니다 (형상은 `L`). 표 안의 값은 형상 + 마모이고, 가공에는 NC 프로그램(`TOOL CALL` 의 델타)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Tool compensation for tool length and tool radius'). 이 칸은 측정 사이클이 값을 써 넣기도 합니다 (설명서). `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`5.1` 처럼)의 행입니다. **단위는 위 규칙과 달리 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (inch 로 만든 공구 테이블에서는 확인하지 못했습니다). 쓰기를 지원하며, 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (시뮬레이터에서 `-999.9999`~`999.9999`). 스핀들에 있는 공구의 값도 받았습니다 (시뮬레이터). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행은 상태 `-18` 입니다. 칸이 비어 있으면 상태 `-22` 이고, 칸을 채우면 읽힙니다. **선삭·연삭·드레싱 공구는 상태 `-18` 입니다** (읽기·쓰기 모두). 디메시는 공구 종류 칸(`TYP`)으로 가립니다: TNC7 사용 설명서 'Tool types' 의 번호로 선삭 `29`·연삭 `30`·드레싱 `31` 이고, 설명서는 그 공구에는 공구 테이블의 길이·반지름이 효과가 없다고 적습니다. 테스트 환경의 선삭 공구는 공구 테이블의 `L`·`R`·`DL`·`DR` 이 모두 `0` 이었고 형상은 선삭 공구 표(`toolturn.trn`)의 `ZL`·`XL`·`YL`·`RS` 와 그 마모 칸에 있었습니다 (연삭·드레싱 공구는 시험하지 못했습니다). 선삭 공구의 형상은 `toolXGeometry`·`toolZGeometry`·`toolYGeometry`·`toolNoseRadiusGeometry` 와 그 마모 주소가 냅니다.

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

**커터 반경 형상값**입니다. Siemens 에서는 SINUMERIK `DP6`(밀링 공구 관점)이고 `toolNoseRadiusGeometry` 와 **같은 저장소**를 가리켜, 어느 주소를 쓰는지가 곧 호출하는 쪽의 의도 선언입니다. 그래서 Siemens 에서는 SDK 가 공구 타입을 검사하지 않습니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens·Heidenhain** 에서 지원하며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의 (Siemens)**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 이름 그대로 **반지름**인데, 공구 목록/오프셋 화면은 흔히 **지름(Ø)** 으로 표시합니다. 실측(2026-07): `BALLNOSE_D8` 의 저장값이 `4.0` 인데 HMI 는 `8.000` 으로 보여줍니다. 디메시는 장비가 저장한 값을 그대로 내보내며 2를 곱하지 않습니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 은 공구 테이블의 반지름(`R`)입니다 (마모는 `DR`. 조작반도 반지름으로 보여 줍니다). 표 안의 값은 형상 + 마모이고, 가공에는 NC 프로그램(`TOOL CALL` 의 델타)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Tool compensation for tool length and tool radius'). `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`5.1` 처럼)의 행입니다. **단위는 위 규칙과 달리 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (inch 로 만든 공구 테이블에서는 확인하지 못했습니다). 쓰기를 지원하며, 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다. 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행은 상태 `-18` 입니다. 조작반의 공구 테이블에서 반지름 칸이 비어 있으면(공구 삽입으로 막 만든 행처럼. 설명서 'Tool management' 도 새 공구의 길이·반지름 칸은 처음에 비어 있다고 적습니다) 상태 `-22` 이고, 칸을 채우면(조작반에서, 또는 이 주소에 쓰면) 읽힙니다. **선삭·연삭·드레싱 공구는 상태 `-18` 입니다** (읽기·쓰기 모두). 디메시는 공구 종류 칸(`TYP`)으로 가립니다: TNC7 사용 설명서 'Tool types' 의 번호로 선삭 `29`·연삭 `30`·드레싱 `31` 이고, 설명서는 그 공구에는 공구 테이블의 길이·반지름이 효과가 없다고 적습니다. 테스트 환경의 선삭 공구는 공구 테이블의 `L`·`R`·`DL`·`DR` 이 모두 `0` 이었고 형상은 선삭 공구 표(`toolturn.trn`)의 `ZL`·`XL`·`YL`·`RS` 와 그 마모 칸에 있었습니다 (연삭·드레싱 공구는 시험하지 못했습니다). 선삭 공구의 형상은 `toolXGeometry`·`toolZGeometry`·`toolYGeometry`·`toolNoseRadiusGeometry` 와 그 마모 주소가 냅니다.

## /machine/toolArea/tool/toolEdge/toolRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

**커터 반경 마모값**입니다. Siemens 에서는 SINUMERIK `DP15` 이고 `toolNoseRadiusWear` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens·Heidenhain** 에서 지원하며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의 (Siemens)**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 이름 그대로 **반지름**인데, 공구 목록/오프셋 화면은 흔히 **지름(Ø)** 으로 표시합니다. 실측(2026-07): `BALLNOSE_D8` 의 저장값이 `4.0` 인데 HMI 는 `8.000` 으로 보여줍니다. 디메시는 장비가 저장한 값을 그대로 내보내며 2를 곱하지 않습니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 은 공구 테이블의 반지름 마모(`DR`)입니다 (형상은 `R`). 표 안의 값은 형상 + 마모이고, 가공에는 NC 프로그램(`TOOL CALL` 의 델타)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Tool compensation for tool length and tool radius'). 이 칸은 측정 사이클이 값을 써 넣기도 합니다 (설명서). `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`5.1` 처럼)의 행입니다. **단위는 위 규칙과 달리 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (inch 로 만든 공구 테이블에서는 확인하지 못했습니다). 쓰기를 지원하며, 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다. 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행은 상태 `-18` 입니다. 칸이 비어 있으면 상태 `-22` 이고, 칸을 채우면 읽힙니다. **선삭·연삭·드레싱 공구는 상태 `-18` 입니다** (읽기·쓰기 모두). 디메시는 공구 종류 칸(`TYP`)으로 가립니다: TNC7 사용 설명서 'Tool types' 의 번호로 선삭 `29`·연삭 `30`·드레싱 `31` 이고, 설명서는 그 공구에는 공구 테이블의 길이·반지름이 효과가 없다고 적습니다. 테스트 환경의 선삭 공구는 공구 테이블의 `L`·`R`·`DL`·`DR` 이 모두 `0` 이었고 형상은 선삭 공구 표(`toolturn.trn`)의 `ZL`·`XL`·`YL`·`RS` 와 그 마모 칸에 있었습니다 (연삭·드레싱 공구는 시험하지 못했습니다). 선삭 공구의 형상은 `toolXGeometry`·`toolZGeometry`·`toolYGeometry`·`toolNoseRadiusGeometry` 와 그 마모 주소가 냅니다.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

**노즈 반경 형상값**입니다 (SINUMERIK `DP6`, 선삭 공구 관점). Siemens 에서는 `toolRadiusGeometry` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens·Heidenhain** 에서 지원하며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **그 표 안의 값은 형상 + 마모** 이고, 가공에는 NC 프로그램(`FUNCTION TURNDATA CORR`)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)').

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의 (Siemens)**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 은 선삭 공구 표(`toolturn.trn`)의 `RS` 칸입니다 (마모는 `toolNoseRadiusWear`(`DRS`), 실제 적용값은 형상 + 마모). Siemens 와 달리 `toolRadiusGeometry` 와 같은 저장소가 아닙니다: 선삭 공구의 노즈 반경은 이 주소로만 읽고, 선삭 공구 표에 행이 없는 공구(밀링 공구 등)는 상태 `-18` 입니다 (그 공구의 반지름은 `toolRadiusGeometry`). `toolRadiusGeometry` 는 공구 종류(`TYP`)가 선삭·연삭·드레싱인 공구에 상태 `-18` 입니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼)의 행입니다. **단위는 위 규칙과 달리 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다. 쓰기를 지원하며, 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다. 선삭 공구 표를 찾지 못하면 상태 `-20` 입니다 (TNC7 사용 설명서는 이 표를 소프트웨어 옵션 50 의 기능으로 설명합니다. 테스트 환경에는 표가 있어 그 경우는 확인하지 못했습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행은 상태 `-18` 입니다. TNC7 프로그래밍 스테이션에서 읽기와 쓰기를 확인했습니다.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

**노즈 반경 마모값**입니다 (SINUMERIK `DP15`). Siemens 에서는 `toolRadiusWear` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens·Heidenhain** 에서 지원하며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다. **그 표 안의 값은 형상 + 마모** 이고, 가공에는 NC 프로그램(`FUNCTION TURNDATA CORR`)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). 이 마모 칸은 공작물을 재는 터치 프로브 사이클이 값을 써 넣기도 합니다 (설명서 'Turning tool table toolturn.trn (option 50)').

단위는 기계 설정을 따릅니다 (mm 또는 inch). 공구 오프셋의 단위를 바꾸는 것은 `G700`/`G710` 뿐입니다 (프로그래밍 매뉴얼): `/machine/channel/gModalCategory/gModal?gModalCategory=4` 가 `G710` 이면 metric, `G700` 이면 inch 입니다. `G70`/`G71` 은 좌표값만 바꾸므로, 그 모달일 때나 둘 다 아닐 때 공구 오프셋은 기본 시스템(`MD10240`)의 단위입니다. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의 (Siemens)**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

**Heidenhain** 은 선삭 공구 표(`toolturn.trn`)의 `DRS` 칸입니다 (형상은 `toolNoseRadiusGeometry`(`RS`), 실제 적용값은 형상 + 마모). Siemens 와 달리 `toolRadiusWear` 와 같은 저장소가 아닙니다: 선삭 공구의 노즈 반경은 이 주소로만 읽고, 선삭 공구 표에 행이 없는 공구(밀링 공구 등)는 상태 `-18` 입니다 (그 공구의 반지름은 `toolRadiusGeometry`). `toolRadiusGeometry` 는 공구 종류(`TYP`)가 선삭·연삭·드레싱인 공구에 상태 `-18` 입니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼)의 행입니다. **단위는 위 규칙과 달리 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다. 쓰기를 지원하며, 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다. 선삭 공구 표를 찾지 못하면 상태 `-20` 입니다 (TNC7 사용 설명서는 이 표를 소프트웨어 옵션 50 의 기능으로 설명합니다. 테스트 환경에는 표가 있어 그 경우는 확인하지 못했습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행은 상태 `-18` 입니다. TNC7 프로그래밍 스테이션에서 읽기와 쓰기를 확인했습니다.

## /machine/toolArea/tool/toolEdge/toolXGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

**선삭 공구의 X 방향 길이 형상값**입니다. Heidenhain 선삭 공구 표(`toolturn.trn`)의 `XL` 칸이고, 마모는 `toolXWear`(`DXL`)입니다. 이 값은 공구 캐리어 기준점에서 잰 X 방향 길이로, 조작반 대화상자의 "공구 길이 2" 입니다 (설명서 'Turning tool table toolturn.trn (option 50)'). **그 표 안의 값은 형상 + 마모** 이고, 가공에는 NC 프로그램(`FUNCTION TURNDATA CORR`)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). X 는 축 이름이 아니라 그 표의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 45.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼)의 행입니다. **단위는 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (그래서 `unit` 필드를 붙이지 않습니다).

**선삭 공구만 답합니다.** 선삭 공구 표에 행이 없는 공구(밀링 공구 등)는 상태 `-18` 이고, 그 공구의 길이·반지름은 `toolLengthGeometry`·`toolRadiusGeometry` 로 읽습니다. 그 두 주소는 공구 종류(`TYP`)가 선삭·연삭·드레싱인 공구에 상태 `-18` 입니다. 선삭 공구 표를 찾지 못하면 상태 `-20` 입니다 (TNC7 사용 설명서는 이 표를 소프트웨어 옵션 50 의 기능으로 설명합니다. 테스트 환경에는 표가 있어 그 경우는 확인하지 못했습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행도 상태 `-18` 입니다. 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (시뮬레이터에서 `-99999.9999`~`99999.9999`). TNC7 프로그래밍 스테이션에서 읽기와 쓰기를 확인했습니다.

**Heidenhain 전용**입니다. Fanuc·Mitsubishi 는 상태 `-20` 이고, 선반의 X 방향 보정은 채널 보정 표의 `/machine/channel/toolOffset/toolXGeometry` 로 읽습니다. Siemens 도 상태 `-20` 입니다: Siemens 의 공구 단위 표에는 X·Y·Z 방향 열 대신 길이1~3(`toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있고, 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/toolArea/tool/toolEdge/toolXWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

**선삭 공구의 X 방향 길이 마모값**입니다. Heidenhain 선삭 공구 표(`toolturn.trn`)의 `DXL` 칸이고, 형상은 `toolXGeometry`(`XL`)입니다. **그 표 안의 값은 형상 + 마모** 이고, 가공에는 NC 프로그램(`FUNCTION TURNDATA CORR`)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). 이 마모 칸은 공작물을 재는 터치 프로브 사이클이 값을 써 넣기도 합니다 (설명서 'Turning tool table toolturn.trn (option 50)'). X 는 축 이름이 아니라 그 표의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0.05}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼)의 행입니다. **단위는 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (그래서 `unit` 필드를 붙이지 않습니다).

**선삭 공구만 답합니다.** 선삭 공구 표에 행이 없는 공구(밀링 공구 등)는 상태 `-18` 이고, 그 공구의 길이·반지름은 `toolLengthGeometry`·`toolRadiusGeometry` 로 읽습니다. 그 두 주소는 공구 종류(`TYP`)가 선삭·연삭·드레싱인 공구에 상태 `-18` 입니다. 선삭 공구 표를 찾지 못하면 상태 `-20` 입니다 (TNC7 사용 설명서는 이 표를 소프트웨어 옵션 50 의 기능으로 설명합니다. 테스트 환경에는 표가 있어 그 경우는 확인하지 못했습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행도 상태 `-18` 입니다. 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (시뮬레이터에서 `-99999.9999`~`99999.9999`). TNC7 프로그래밍 스테이션에서 읽기와 쓰기를 확인했습니다.

**Heidenhain 전용**입니다. Fanuc·Mitsubishi 는 상태 `-20` 이고, 선반의 X 방향 보정은 채널 보정 표의 `/machine/channel/toolOffset/toolXWear` 로 읽습니다. Siemens 도 상태 `-20` 입니다: Siemens 의 공구 단위 표에는 X·Y·Z 방향 열 대신 길이1~3(`toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있고, 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/toolArea/tool/toolEdge/toolZGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

**선삭 공구의 Z 방향 길이 형상값**입니다. Heidenhain 선삭 공구 표(`toolturn.trn`)의 `ZL` 칸이고, 마모는 `toolZWear`(`DZL`)입니다. 이 값은 공구 캐리어 기준점에서 잰 Z 방향 길이로, 조작반 대화상자의 "공구 길이 1" 입니다 (설명서 'Turning tool table toolturn.trn (option 50)'). **그 표 안의 값은 형상 + 마모** 이고, 가공에는 NC 프로그램(`FUNCTION TURNDATA CORR`)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Z 는 축 이름이 아니라 그 표의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 70.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼)의 행입니다. **단위는 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (그래서 `unit` 필드를 붙이지 않습니다).

**선삭 공구만 답합니다.** 선삭 공구 표에 행이 없는 공구(밀링 공구 등)는 상태 `-18` 이고, 그 공구의 길이·반지름은 `toolLengthGeometry`·`toolRadiusGeometry` 로 읽습니다. 그 두 주소는 공구 종류(`TYP`)가 선삭·연삭·드레싱인 공구에 상태 `-18` 입니다. 선삭 공구 표를 찾지 못하면 상태 `-20` 입니다 (TNC7 사용 설명서는 이 표를 소프트웨어 옵션 50 의 기능으로 설명합니다. 테스트 환경에는 표가 있어 그 경우는 확인하지 못했습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행도 상태 `-18` 입니다. 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (시뮬레이터에서 `-99999.9999`~`99999.9999`). TNC7 프로그래밍 스테이션에서 읽기와 쓰기를 확인했습니다.

**Heidenhain 전용**입니다. Fanuc·Mitsubishi 는 상태 `-20` 이고, 선반의 Z 방향 보정은 채널 보정 표의 `/machine/channel/toolOffset/toolZGeometry` 로 읽습니다. Siemens 도 상태 `-20` 입니다: Siemens 의 공구 단위 표에는 X·Y·Z 방향 열 대신 길이1~3(`toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있고, 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/toolArea/tool/toolEdge/toolZWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

**선삭 공구의 Z 방향 길이 마모값**입니다. Heidenhain 선삭 공구 표(`toolturn.trn`)의 `DZL` 칸이고, 형상은 `toolZGeometry`(`ZL`)입니다. **그 표 안의 값은 형상 + 마모** 이고, 가공에는 NC 프로그램(`FUNCTION TURNDATA CORR`)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). 이 마모 칸은 공작물을 재는 터치 프로브 사이클이 값을 써 넣기도 합니다 (설명서 'Turning tool table toolturn.trn (option 50)'). Z 는 축 이름이 아니라 그 표의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0.05}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼)의 행입니다. **단위는 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (그래서 `unit` 필드를 붙이지 않습니다).

**선삭 공구만 답합니다.** 선삭 공구 표에 행이 없는 공구(밀링 공구 등)는 상태 `-18` 이고, 그 공구의 길이·반지름은 `toolLengthGeometry`·`toolRadiusGeometry` 로 읽습니다. 그 두 주소는 공구 종류(`TYP`)가 선삭·연삭·드레싱인 공구에 상태 `-18` 입니다. 선삭 공구 표를 찾지 못하면 상태 `-20` 입니다 (TNC7 사용 설명서는 이 표를 소프트웨어 옵션 50 의 기능으로 설명합니다. 테스트 환경에는 표가 있어 그 경우는 확인하지 못했습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행도 상태 `-18` 입니다. 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (시뮬레이터에서 `-99999.9999`~`99999.9999`). TNC7 프로그래밍 스테이션에서 읽기와 쓰기를 확인했습니다.

**Heidenhain 전용**입니다. Fanuc·Mitsubishi 는 상태 `-20` 이고, 선반의 Z 방향 보정은 채널 보정 표의 `/machine/channel/toolOffset/toolZWear` 로 읽습니다. Siemens 도 상태 `-20` 입니다: Siemens 의 공구 단위 표에는 X·Y·Z 방향 열 대신 길이1~3(`toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있고, 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/toolArea/tool/toolEdge/toolYGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

**선삭 공구의 Y 방향 길이 형상값**입니다. Heidenhain 선삭 공구 표(`toolturn.trn`)의 `YL` 칸이고, 마모는 `toolYWear`(`DYL`)입니다. 이 값은 공구 캐리어 기준점에서 잰 Y 방향 길이로, 조작반 대화상자의 "공구 길이 3" 입니다 (설명서 'Turning tool table toolturn.trn (option 50)'). **그 표 안의 값은 형상 + 마모** 이고, 가공에는 NC 프로그램(`FUNCTION TURNDATA CORR`)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Y 는 축 이름이 아니라 그 표의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼)의 행입니다. **단위는 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (그래서 `unit` 필드를 붙이지 않습니다).

**선삭 공구만 답합니다.** 선삭 공구 표에 행이 없는 공구(밀링 공구 등)는 상태 `-18` 이고, 그 공구의 길이·반지름은 `toolLengthGeometry`·`toolRadiusGeometry` 로 읽습니다. 그 두 주소는 공구 종류(`TYP`)가 선삭·연삭·드레싱인 공구에 상태 `-18` 입니다. 선삭 공구 표를 찾지 못하면 상태 `-20` 입니다 (TNC7 사용 설명서는 이 표를 소프트웨어 옵션 50 의 기능으로 설명합니다. 테스트 환경에는 표가 있어 그 경우는 확인하지 못했습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행도 상태 `-18` 입니다. 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (시뮬레이터에서 `-99999.9999`~`99999.9999`). TNC7 프로그래밍 스테이션에서 읽기와 쓰기를 확인했습니다.

**Heidenhain 전용**입니다. Fanuc·Mitsubishi 는 상태 `-20` 이고, 선반의 Y 방향 보정은 채널 보정 표의 `/machine/channel/toolOffset/toolYGeometry` 로 읽습니다. Siemens 도 상태 `-20` 입니다: Siemens 의 공구 단위 표에는 X·Y·Z 방향 열 대신 길이1~3(`toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있고, 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/toolArea/tool/toolEdge/toolYWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

**선삭 공구의 Y 방향 길이 마모값**입니다. Heidenhain 선삭 공구 표(`toolturn.trn`)의 `DYL` 칸이고, 형상은 `toolYGeometry`(`YL`)입니다. **그 표 안의 값은 형상 + 마모** 이고, 가공에는 NC 프로그램(`FUNCTION TURNDATA CORR`)이나 보정 표가 준 델타가 더해질 수 있습니다 (TNC7 사용 설명서 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). 이 마모 칸은 공작물을 재는 터치 프로브 사이클이 값을 써 넣기도 합니다 (설명서 'Turning tool table toolturn.trn (option 50)'). Y 는 축 이름이 아니라 그 표의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0.05}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼)의 행입니다. **단위는 언제나 mm 입니다**: 디메시가 DNC 에서 mm 를 골라 읽고 씁니다 (그래서 `unit` 필드를 붙이지 않습니다).

**선삭 공구만 답합니다.** 선삭 공구 표에 행이 없는 공구(밀링 공구 등)는 상태 `-18` 이고, 그 공구의 길이·반지름은 `toolLengthGeometry`·`toolRadiusGeometry` 로 읽습니다. 그 두 주소는 공구 종류(`TYP`)가 선삭·연삭·드레싱인 공구에 상태 `-18` 입니다. 선삭 공구 표를 찾지 못하면 상태 `-20` 입니다 (TNC7 사용 설명서는 이 표를 소프트웨어 옵션 50 의 기능으로 설명합니다. 테스트 환경에는 표가 있어 그 경우는 확인하지 못했습니다). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행도 상태 `-18` 입니다. 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (시뮬레이터에서 `-99999.9999`~`99999.9999`). TNC7 프로그래밍 스테이션에서 읽기와 쓰기를 확인했습니다.

**Heidenhain 전용**입니다. Fanuc·Mitsubishi 는 상태 `-20` 이고, 선반의 Y 방향 보정은 채널 보정 표의 `/machine/channel/toolOffset/toolYWear` 로 읽습니다. Siemens 도 상태 `-20` 입니다: Siemens 의 공구 단위 표에는 X·Y·Z 방향 열 대신 길이1~3(`toolLengthGeometry`·`toolLength2Geometry`·`toolLength3Geometry`)이 있고, 어느 길이가 어느 방향인지는 공구 타입과 활성 평면이 정하므로 디메시는 번역하지 않습니다.

## /machine/toolArea/tool/toolEdge/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**날끝 위치 코드**입니다 (SINUMERIK `DP2`, `1`~`9`). 노즈 반경 보정 때 날끝이 노즈 중심 기준 어느 방위에 있는지를 나타내며, 각도가 아니라 위치 코드입니다. Fanuc 의 공구 보정 트리에도 같은 개념이 있고 (`0`~`9`), **번호 체계와 `desc` 어휘를 공유**하므로 기종이 달라도 값을 그대로 비교·재사용할 수 있습니다 (`desc` 는 `9` 에만: `1`~`8` 방위는 매뉴얼이 도해로만 정의하고 가공 구성별로 세 벌이라 싣지 않습니다). 유효 범위는 `1`~`9` 입니다 (공구 관리 기능 매뉴얼이 날끝 위치를 `1`~`8` 과 `9` 로 정의합니다). 쓰기는 이 범위만 받고, **`0` 을 포함한 그 밖의 값은 상태 `-16` 입니다**.

반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 3}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다.

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 상태 `-18` 로 거절됩니다 (에러 문구가 없는 것이 공구인지 날인지 구분해 알려 줍니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 두 기종의 날끝 위치는 보정 번호 쪽의 `/machine/channel/toolOffset/toolTipDirection` 으로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolTipAngle
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

그 보정 세트의 **날끝 각**입니다. 드릴이면 선단각(`118.0`), 센터드릴이면 `90.0` 같은 값. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 118.0}`.

**`toolTipDirection` 과 다른 값입니다.** 이름이 한 글자 차이인데 성격이 다릅니다. 저쪽은 날끝이 노즈 중심 기준 **어느 방위**인지를 나타내는 코드이고, 이 값은 날끝의 **각도**입니다.

**각도를 쓰지 않는 공구는 `0.0`** 입니다. 밀링 공구가 그렇습니다. Siemens 실측에서 드릴은 `118.0`, 페이스밀은 `0.0` 이었습니다. 출처는 SINUMERIK `DP24` 로, 드릴류에서는 조작반(Operate)이 선단각으로 쓰는 칸이지만 **선삭 공구에서는 같은 칸이 여유각(clearance angle)** 입니다 (공구 관리 매뉴얼의 날 데이터 표). 선삭 공구에서 이 값을 선단각으로 읽지 마세요.

**장비 화면의 `N` 열은 이 값과 `toolTeethCount` 를 겸용합니다.** 드릴류면 각도를, 밀링이면 인선 수를 그 한 칸에 보여줍니다. 디메시는 한 주소가 공구 종류에 따라 다른 물리량이 되지 않도록 둘을 따로 냅니다. 화면의 그 숫자를 찾으려면 둘 중 값이 있는 쪽을 보세요.

**Siemens·Heidenhain** 이 지원합니다. Siemens 는 없는 공구/날(D)을 지정하면 상태 `-18` 로 거절합니다.

**Heidenhain** 은 공구 테이블의 선단각(`T-ANGLE`)입니다. TNC7 사용 설명서('Tool table tool.t')는 이 칸을 드릴 같은 공구의 선단각(시뮬레이션·사이클·충돌 감시에 쓰입니다)으로 설명하고 입력 범위를 -180~+180 으로 적습니다. 테스트 환경에서 드릴은 `118.0`, 스팟 드릴은 `90.0`, 밀링 공구는 `0.0` 이었습니다. `toolEdge=0` 은 공구 자신의 행, `toolEdge=1` 부터는 인덱스 공구(`5.1` 처럼)의 행입니다. 쓰기를 지원하며, 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 그 칸의 허용 범위가 실립니다 (테스트 환경에서 `200.0` 을 쓰면 `-180`~`180`). 없는 공구·인덱스와 테이블 맨 앞의 `0` 번 행은 상태 `-18`, 조작반의 공구 테이블에서 이 칸이 비어 있으면 상태 `-22`, 공구 테이블에서 `T-ANGLE` 칸을 찾지 못하면 상태 `-20` 입니다 (이 주소가 그 장비에서 동작하지 않습니다). **선삭·연삭·드레싱 공구(공구 종류 칸 `TYP` 이 `29`·`30`·`31`)는 상태 `-18` 입니다** (읽기·쓰기 모두). 선삭 공구의 각도는 그 표의 `T-ANGLE`(공구각)·`P-ANGLE`(선단각) 칸에 따로 있습니다 (TNC7 사용 설명서 'Turning tool table toolturn.trn'). 디메시는 그 두 칸을 내지 않습니다.

**Fanuc·Mitsubishi 는 상태 `-20` 입니다.** 두 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조). 보정값은 `/machine/channel/toolOffset/…` 의 채널 보정 표로 읽으세요.

## /machine/toolArea/tool/toolEdge/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

그 날에 배정된 **수명 총량**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 6}`.

Siemens 제어기는 잔여를 이 값에서 시작해 깎아 내려갑니다 (Heidenhain 은 쓴 시간을 올려 셉니다. 아래 문단). 단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 마모 감시일 때는 값이 거리라 단위가 기계 설정(mm/inch)에 달려 있어 `unit` 을 붙이지 않습니다. `/machine/toolArea/tool/toolLifeMonitorType` 참조. 감시가 꺼져 있으면 상태 `-18` 로 거절됩니다 (Heidenhain 의 쓰기는 예외로 받습니다. 아래 문단).

**값은 조작반 화면에 뜨는 그대로입니다.** 시간 감시면 분(`unit` 은 `"min"`)이고 초로 환산하지 않습니다. 다른 시간 값들(`…Duration`)이 초인 것과 다릅니다. 수명은 작업자가 화면을 보며 판단하는 값이라 숫자가 화면과 같아야 합니다.

개수 감시일 때는 **정수만 받습니다.** 소수를 보내면 장비가 성공을 답하고 값은 바뀌지 않으므로 디메시가 상태 `-16` 으로 먼저 거절합니다.

**이 값을 쓰면 SINUMERIK 이 공구의 잠금을 다시 매깁니다** (Siemens 공구관리 기능 매뉴얼 §8.11, 저희 벤치에서도 확인). 잔여가 한계 안이면 잠금이 풀리고(`/machine/toolArea/tool/toolUseStatus` 가 `1`/`2`), 잔여가 `0` 이하면 잠깁니다(`3`). 수명 말고 다른 이유로 잠금 비트가 켜진 공구(`5`)도 풀리므로, 잠금을 유지하려면 쓴 뒤 `toolUseStatus` 에 `5` 를 다시 쓰세요. 사용 허가가 없어서만 `5` 인 공구도 풀리는지는 확인하지 못했습니다.

**Siemens·Heidenhain** 에서 지원합니다. Fanuc 의 수명은 날이 아니라 공구에 붙으므로 `/machine/toolArea/tool/toolLifeTotal` 에 있습니다 (그쪽은 초·횟수 단위).

**Mitsubishi 는 상태 `-20` 입니다.** 그 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조).

**Heidenhain** 은 그 행의 최대 수명(`TIME1`)입니다. `toolEdge=0` 은 공구 자신의 행이라 `/machine/toolArea/tool/toolLifeTotal` 과 같은 값이고, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼 공구 번호 뒤에 붙는 행)의 값입니다. 인덱스 공구는 행마다 수명이 따로 있습니다. 규칙은 공구 단위 주소와 같습니다: 단위는 분(`unit` 은 `"min"`)이고, `0` 이면 최대 수명이 없다는 뜻이라 읽기는 상태 `-18` 입니다 (`TIME2` 만 있는 행도 그렇습니다). 쓰기는 감시가 꺼져 있어도 받습니다 (값을 쓰면 켜지고 `0` 을 쓰면 꺼집니다). 정수 분만 받습니다 (소수는 상태 `-16`. 제어기가 정수로 반올림해 담습니다). 제어기가 받지 않는 값은 상태 `-16` 이고 에러 문구에 허용 범위가 실립니다. 제어기는 잔여가 아니라 쓴 시간을 올려 세므로 그 값은 `/machine/toolArea/tool/toolEdge/toolLifeUsed` 가 답합니다. 없는 공구·인덱스는 상태 `-18` 이고, 테이블 맨 앞의 `0` 번 행도 공구가 아닌 자리로 보아 상태 `-18` 입니다.

## /machine/toolArea/tool/toolEdge/toolLifeUsed
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

그 날(인덱스 공구)이 **지금까지 쓴 수명**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0}`.

**Heidenhain 전용**입니다. 공구 테이블에서 그 행의 현재 사용 시간(`CUR_TIME`, 조작반 공구 관리 화면의 `CUR_TIME (min)`)이고 단위는 분(`unit` 은 `"min"`)입니다. `toolEdge=0` 은 공구 자신의 행이라 `/machine/toolArea/tool/toolLifeUsed` 와 같은 값이고, `toolEdge=1` 부터는 인덱스 공구(`320.1` 처럼 공구 번호 뒤에 붙는 행)의 값입니다. 인덱스 공구는 행마다 수명이 따로 있습니다. 제어기는 이 값을 최대 수명(`/machine/toolArea/tool/toolEdge/toolLifeTotal`) 쪽으로 올려 셉니다 (테스트 환경에서 공구 자신의 행과 밀링 인덱스 공구 `10.1` 의 행이 각각 이송 블록 동안 늘었고, `10.1` 을 쓰는 동안 공구 자신의 행은 그대로였습니다. TNC7 사용 설명서 'Indexed tool' 도 제어기가 사용 시간을 행마다 따로 적는다고 합니다).

**인서트를 갈고 수명을 되돌릴 때 이 주소에 씁니다** (보통 `0`). 소수도 받되, 제어기가 소수 둘째 자리로 반올림해 담습니다 (`/machine/toolArea/tool/toolLifeUsed` 참조). 그 행의 수명 한계(`TIME1`·`TIME2`)가 둘 다 `0` 이면 읽기·쓰기 모두 상태 `-18` 입니다. 먼저 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 을 쓰세요. 없는 공구·인덱스는 상태 `-18` 이고, 테이블 맨 앞의 `0` 번 행도 공구가 아닌 자리로 보아 상태 `-18` 입니다.

Siemens·Fanuc·Mitsubishi 는 상태 `-20` 입니다. Siemens 는 남은 수명을 내려 세므로 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 을, Fanuc·Mitsubishi 는 공구 단위의 `/machine/toolArea/tool/toolLifeUsed` 를 보세요.

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

**인서트를 갈고 수명을 되돌릴 때 이 주소에 씁니다.** 보통 `toolLifeTotal` 과 같은 값을 넣습니다.

**이 값을 쓰면 SINUMERIK 이 공구의 잠금을 다시 매깁니다** (Siemens 공구관리 기능 매뉴얼 §8.11, 저희 벤치에서도 확인). 수명이 다해 잠긴 공구(`/machine/toolArea/tool/toolUseStatus` `3`)는 이 쓰기 하나로 풀리고(`1`/`2`), 반대로 `0` 을 쓰면 제어기가 잠급니다(`3`). 수명 말고 다른 이유로 잠금 비트가 켜진 공구(`5`)도 풀리므로, 잠금을 유지하려면 쓴 뒤 `toolUseStatus` 에 `5` 를 다시 쓰세요. 사용 허가가 없어서만 `5` 인 공구도 풀리는지는 확인하지 못했습니다.

단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 마모 감시일 때는 값이 거리라 단위가 기계 설정(mm/inch)에 달려 있어 `unit` 을 붙이지 않습니다. 감시가 꺼져 있으면 상태 `-18`, 개수 감시에 소수를 쓰면 상태 `-16` 으로 거절됩니다.

**Siemens 전용**입니다. Fanuc 은 잔여가 아니라 **올라가는 사용량 카운터**를 주므로 공구 단위의 `/machine/toolArea/tool/toolLifeUsed` 로 읽고, 잔여는 `/machine/toolArea/tool/toolLifeTotal` 에서 빼서 구하세요.

**Mitsubishi 는 상태 `-20` 입니다.** 그 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조).

**Heidenhain 은 상태 `-20` 입니다.** Heidenhain 은 쓴 시간을 올려 세므로 `/machine/toolArea/tool/toolEdge/toolLifeUsed` 로 읽고, 최대 수명(`TIME1`)까지 남은 시간은 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 에서 빼서 구하세요. 공구를 부를 때의 한계 `TIME2` 가 더 작으면 그쪽이 먼저 걸립니다 (`/machine/toolArea/tool/toolEdge/toolUseStatus` 참조).

## /machine/toolArea/tool/toolEdge/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**경고선**입니다. 잔여가 이 값 밑으로 내려오면 제어기가 경고를 올립니다 (`/machine/toolArea/tool/toolLifeWarnOn`, **공구 단위**). `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 3}`.

교체 공구를 준비할 시간을 벌기 위한 값이라 `toolLifeTotal` 보다 작게 잡습니다. 단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 마모 감시일 때는 값이 거리라 단위가 기계 설정(mm/inch)에 달려 있어 `unit` 을 붙이지 않습니다. 감시가 꺼져 있으면 상태 `-18`, 개수 감시에 소수를 쓰면 상태 `-16` 으로 거절됩니다.

**이 값을 쓰면 SINUMERIK 이 공구의 잠금을 다시 매깁니다** (Siemens 공구관리 기능 매뉴얼 §8.11, 저희 벤치에서도 확인). 잔여가 한계 안이면 잠금이 풀리고(`/machine/toolArea/tool/toolUseStatus` 가 `1`/`2`), 잔여가 `0` 이하면 잠깁니다(`3`). 수명 말고 다른 이유로 잠금 비트가 켜진 공구(`5`)도 풀리므로, 잠금을 유지하려면 쓴 뒤 `toolUseStatus` 에 `5` 를 다시 쓰세요. 사용 허가가 없어서만 `5` 인 공구도 풀리는지는 확인하지 못했습니다.

**Siemens 전용**입니다. Fanuc 의 예고 수명은 공구에 붙으므로 `/machine/toolArea/tool/toolLifeWarnLimit` 에 있습니다.

**Mitsubishi 는 상태 `-20` 입니다.** 그 기종의 오프셋 모델에는 공구에 딸린 날 계층이 없습니다 (`toolEdgeCount` 참조).

**Heidenhain 은 상태 `-20` 입니다.**

## /machine/toolArea/tool/toolEdge/toolUseStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
codes: [{"value": 1, "name": "unused"}, {"value": 2, "name": "in use"}, {"value": 3, "name": "life expired"}, {"value": 5, "name": "locked"}]
```

그 날(인덱스 공구)의 **사용 상태**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `int` + `desc`, 읽기·쓰기 모두 지원합니다. 값의 뜻은 공구 단위의 `/machine/toolArea/tool/toolUseStatus` 와 같습니다 (`1` 미사용 · `2` 사용 중 · `3` 수명 초과 · `5` 잠금).

**Heidenhain 전용**입니다. 인덱스 공구는 행마다 잠금과 수명이 따로 있어, 그 행의 잠금(`TL`)과 수명(최대 `TIME1`, 공구를 부를 때의 한계 `TIME2`, 사용 `CUR_TIME`)에 공구 단위와 같은 규칙을 적용합니다: 사용 시간이 `TIME2` 에 닿았으면 잠금과 무관하게 `3`, 그 밖에는 잠겨 있으면 최대 수명이 있고 사용 시간이 그에 이르렀을 때 `3`, 아니면 `5` 이고, 잠겨 있지 않으면 사용 시간이 있으면 `2`, 없으면 `1` 입니다. `0`·`4` 는 나오지 않습니다. `toolEdge=0` 은 공구 자신의 행이라 공구 단위 주소와 같은 값입니다. `TIME1` 을 넘긴 것은 제어기의 잠금을 따르므로, 수명을 넘겼는지는 `/machine/toolArea/tool/toolEdge/toolLifeUsed` 와 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 을 비교해 가려내세요. 없는 공구·인덱스는 상태 `-18` 이고, 테이블 맨 앞의 `0` 번 행도 공구가 아닌 자리로 보아 상태 `-18` 입니다.

**쓰기**는 그 행의 잠금(`TL`)을 바꿉니다. 규칙은 공구 단위 주소와 같습니다: `5` 는 잠그고, `1`·`2` 는 잠금을 풀되 풀린 뒤 그 값으로 읽히는 경우만 받습니다 (쓴 시간이 있으면 `2`, 없으면 `1`. 맞지 않으면 상태 `-16` 이고, 미사용으로 되돌리려면 먼저 `/machine/toolArea/tool/toolEdge/toolLifeUsed` 에 `0` 을 쓰세요). `3`·`0`·`4` 는 상태 `-16` 이고, 그 행의 사용 시간이 `TIME2` 에 닿았으면 잠금과 무관하게 `3` 으로 읽히므로 `1`·`2`·`5` 도 상태 `-16` 입니다 (먼저 사용 시간을 고치세요). 그 밖에 이미 그 상태면 아무것도 하지 않고 성공합니다. 인덱스 행을 잠가도 공구 자신의 행은 그대로였습니다 (테스트 환경에서 확인).

Siemens·Fanuc·Mitsubishi 는 상태 `-20` 입니다. Siemens·Fanuc 의 사용 상태는 공구 단위라 `/machine/toolArea/tool/toolUseStatus` 가 답합니다.

## /machine/toolArea/tool/toolEdge/sisterTool
```yaml
value_type: "object"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

그 날(인덱스 공구) 대신 쓸 **대체 공구**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `object`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": {"toolNumber": 320, "toolEdgeNumber": 2}}`.

값의 모양과 규칙은 공구 단위의 `/machine/toolArea/tool/sisterTool` 과 같습니다. `{"toolNumber": 320, "toolEdgeNumber": 2}` 처럼 대체 공구를 가리키는 번호 둘이고, 대체 공구가 없으면 `{"toolNumber": 0, "toolEdgeNumber": 0}` 입니다. 두 값을 그대로 `tool`·`toolEdge` 필터에 넣으면 그 공구를 조회할 수 있습니다.

**Heidenhain 전용**입니다. 인덱스 공구(`320.1` 처럼 공구 번호 뒤에 붙는 행)는 행마다 대체 공구 칸(`RT`)이 따로 있어, 이 주소가 그 행의 값을 다룹니다. `toolEdge=0` 은 공구 자신의 행이라 공구 단위 주소와 같은 값입니다.

**쓰기**: 두 키를 모두 정수로 주세요. 다른 키가 섞이거나 하나가 빠지면 상태 `-16` 입니다. `{"toolNumber": 0, "toolEdgeNumber": 0}` 을 쓰면 비웁니다. 대체 공구는 공구 테이블에 있어야 하고, 없는 공구는 제어기가 받지 않아 상태 `-16` 입니다. 제어기가 소수 하나로 두기 때문에 그 모양으로 구분되지 않는 인덱스(`10`·`20`·`100` 처럼 끝자리가 `0` 인 인덱스)는 보내지 않고 상태 `-16` 입니다. 없는 공구·인덱스는 상태 `-18` 이고, 테이블 맨 앞의 `0` 번 행도 공구가 아닌 자리로 보아 상태 `-18` 입니다.

Siemens·Fanuc·Mitsubishi 는 상태 `-20` 입니다.
