## /machine/channelCount
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

CNC 의 채널(계통) 수입니다. 연결 시 캐싱된 값이라 추가 통신 없이 즉시 반환됩니다.

## /machine/cncModel
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

CNC 모델 문자열입니다.

- **Fanuc**: 시리즈 번호 문자열입니다. `"15"`, `"16"`, `"18"`, `"21"`, `"30"`, `"31"`, `"32"`, `"35"`, `"0"`(0i), `"PD"`/`"PH"`(Power Mate i), `"PM"`(Power Motion i). `desc` 에 시리즈명이 함께 옵니다 (예: `"31"` → `Series 31i`)
- **Siemens**: 모델명 그대로 (예: `"840D sl"`)
- **Mitsubishi**: NC 시스템 S/W 번호·이름 문자열입니다 (벤더 `GetVersion`). 이 항목을 제공하지 않는 장비에서는 `-20` 으로 답합니다. 실물 하드웨어 정보가 없는 시뮬레이터가 그렇습니다. `desc` 는 없습니다

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
- **장비 헬스체크 권장 주소**: 모든 프로토콜에서 실제 NC 왕복을 일으키는 부담 적은 읽기라, 주기 폴링 후 `status` 판정(`0`=정상, `-10`/`-14`=링크 이상)으로 장비별 통신 상태 감시에 쓰세요. (`machineType` 등 캐시 서빙 주소는 링크가 죽어도 성공할 수 있어 부적합)

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
- **Mitsubishi**: `GetAliveTime`. **초 해상도**입니다. 다만 **9999시간 59분 59초에서 누적이 멈추고 그 값을 유지**하므로(장비 사양), 그 지점 이후로는 차분이 계속 `0` 이 됩니다

경과시간 계열 주소는 항상 **초 정규화 int** 입니다 (`…Duration` 접미사 규칙). **초 미만은 버립니다**: `59.9`초는 `59` 입니다. 조작반의 경과시간 표시와 같은 방식이고, 아직 지나지 않은 초를 세지 않습니다. 모든 기종·모든 `…Duration` 주소가 같습니다.

## /machine/configuredMachineName
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

`config.json`(허브) 또는 `deemesh_create` 설정의 `machine_name` 을 그대로 돌려줍니다. 장비가 보고하는 이름이 아니라 **설정에서 온 값**입니다. 이름에 `configured` 를 넣은 것도 그 때문입니다. 연결이 의도한 장비로 갔는지 확인하거나 응답을 식별할 때 씁니다.

## /machine/configuredProtocol
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

이 연결이 쓰는 **프로토콜 식별자**입니다. `"nc_focas2_fanuc"`·`"nc_opcua_siemens"`·`"nc_ezsocket_mitsubishi"` 중 하나. 필터 없음. 반환 `string`, 읽기 전용. 연결 시 정해지는 값이라 연결된 뒤에는 NC 통신 없이 즉시 응답합니다 (미연결 상태에서는 다른 주소처럼 연결 확인이 먼저라 `-10` 입니다).

`configuredMachineName` 과 마찬가지로 **설정에서 온 값**입니다. 기계에 물어본 결과가 아니라 `deemesh_create` 의 `protocol` 필드(또는 `config.json` 의 머신 설정)를 그대로 돌려줍니다. 그래서 주소에 `configured` 가 붙습니다.

**용도는 좁습니다.** 대부분의 주소는 기종을 감추도록 설계되어 있어 분기가 필요 없습니다. 이 값이 필요한 곳은 **값 공간이 기종 소유인 소수의 자리**입니다. PLC 주소 문법(`D100` 대 `DB10.DBB56`), 진단 번호 체계, 공구 타입 코드처럼 카탈로그가 "기종에 따라 다르다" 고 명시한 곳들입니다.

**지원 여부 판단에는 쓰지 마세요.** "이 기종은 이 주소를 못 쓰니 건너뛰자" 는 판단은 `-20` 으로 해야 합니다. 이 값으로 분기해 두면 나중에 그 기종 지원이 추가돼도 코드가 계속 건너뜁니다.

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

## /machine/channel/spindleCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

채널의 스핀들 수입니다. 연결 시 캐싱. `spindle` 필터의 유효 범위가 `1`~이 값입니다.

## /machine/channel/toolAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

채널이 사용하는 공구 영역(tool area) 번호입니다. 공구 트리 주소들의 `toolArea` 필터에 넣는 값입니다. Fanuc·Mitsubishi 는 고정 `1`(공구 영역이라는 계층이 없습니다), Siemens 는 NCK 설정(`toNo`)값.

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

**Mitsubishi 에만 "스톱 코드" 라는 채널이 있기 때문입니다.** 제어기가 *자동 운전의 상태*를 알리는 자리로, 벤더 매뉴얼도 알람(빨강)이 아니라 경고와 같은 노랑으로 칠합니다. 담기는 것은 `T03`(싱글블록 스위치 ON)·`T10`(M 코드 완료 대기)·`T02`(소프트 리밋에 걸린 축 있음) 같은 것들이라 **고장이 아니라 "지금 이걸 기다리는 중"** 입니다. 위 실측에서 올라와 있던 것도 `T10` 이었고, 프로그램이 끝나자 사라졌습니다. Fanuc·Siemens 는 이런 상태를 알람 목록에 싣지 않아 `0` 이 유지됩니다.

**그래서 `1` 을 운영자 호출 신호로 쓰지 마세요**. Mitsubishi 장비에서는 정상 자동 운전 중에도 `1` 이 됩니다. 호출에는 `2` 를 쓰거나 `alarmList` 항목의 `severity` 와 `category` 를 보고 판단하세요.

**기준은 "장비에 이상이 있느냐" 이지 "가공이 멈췄느냐" 가 아닙니다.** 둘은 대개 함께 가지만 갈릴 때가 있습니다. `M0`(프로그램 정지)나 싱글블록으로 선 기계는 멀쩡하므로 `1` 입니다. 지금 가공이 멈췄는지는 `/machine/channel/executionStatus` 로 판단하세요.

- **숫자는 기종이 늘어도 이 셋뿐입니다.** 벤더 코드를 그대로 내보내지 않으므로, 어느 기종에 붙였는지 몰라도 `value` 로 바로 분기할 수 있습니다.
- **원인은 `desc` 로 옵니다.** Fanuc 은 원인 계열(`{"value": 1, "desc": "Memory backup battery voltage low (CNC or Amplifier)"}`), Siemens 는 가장 무거운 알람의 본문(`{"value": 2, "desc": "Emergency stop"}`), Mitsubishi 는 알람 종류(`{"value": 2, "desc": "NC alarm"}`). `desc` 는 사람이 읽는 문자열이므로 **분기 조건으로 쓰지 마세요.** 분기는 `value` 로.
- **Mitsubishi**: 벤더의 알람 종류로 판정합니다. NC 알람·PLC 알람은 `2`, 오퍼레이터 메시지와 **스톱 코드는 `1`** 입니다. 스톱 코드에는 `M0`·싱글블록 같은 정상 정지가 들어 있어 장비 이상이 아니기 때문입니다 (고장으로 멈춘 경우에는 NC 알람이 함께 올라와 `2` 가 됩니다). NC 알람 계열 안의 워닝 하위 종류는 `GetAlarm2` 가 구분하지 않으므로 함께 `2` 로 분류됩니다 (비상정지는 NC 알람이라 `2`)
- 판정이 애매한 벤더 코드는 **보수적으로 `2`** 로 분류합니다. 정지를 경고로 낮춰 부르는 쪽이 그 반대보다 위험하기 때문입니다. Fanuc 은 SDK 가 모르는 코드를 새로 내보내도 `2` 로 분류합니다. Siemens 는 서버가 이벤트마다 주는 심각도를 씁니다. 오류(`1000`)만 `2` 이고 벤더가 **경고(`500`)라 답한 것은 `1`** 입니다. `alarmList` 의 `severity` 와 **같은 기준**입니다.
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

- **비용 주의 (Fanuc)**: count 전용 벤더 API 가 없어 내부적으로 `alarmList` 와 **같은 비용**입니다. 둘을 함께 배치 요청하면 fetch 1회로 합쳐집니다. 저비용 존재 판정만 필요하면 `alarmStatus` 를 쓰세요
- **비용 주의 (Siemens)**: 위 둘과 같습니다. 개수를 `alarmList` 와 **같은 이벤트 스냅샷**에서 세므로 같은 비용이고, 함께 배치 요청하면 fetch 1회로 합쳐집니다. NCK 전역 개수라 `channel` 값은 무시됩니다 (`alarmList` 와 동일 방침)
- **비용 주의 (Mitsubishi)**: Fanuc 과 같습니다. count 전용 API 가 없어 `alarmList` 와 같은 비용이고, 함께 배치 요청하면 fetch 1회로 합쳐집니다

## /machine/channel/alarmList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

채널의 **활성 알람 + 오퍼레이터/매크로 메시지** 목록입니다. 반환 타입 `objectArray`, 없으면 빈 배열 `[]`. Siemens 는 알람이 NCK 전역이라 `channel` 값은 무시됩니다.

원소: `{"number": 1234, "message": "SPINDLE OVERHEAT", "category": "Overheat", "severity": "alarm", "raisedAt": "2026-07-29T11:11:03Z"}`

키 집합은 **기종과 무관하게 항상 같습니다.** 값이 없으면 키가 빠지는 게 아니라 `null` 입니다 (`entry` 와 같은 규약). `severity` 는 `"alarm"` / `"warning"` 두 값뿐입니다.

- **number**: 알람/메시지 번호. **`null` 이거나 `0` 일 수 있습니다.** Fanuc·Siemens 는 이 값이 매뉴얼에서 찾아보는 알람 번호이지만, **Mitsubishi 는 식별자가 `category`** 입니다 (조작반에 뜨는 `P232`·`S01`·`EMG` 같은 코드 전체). 그쪽의 `number` 는 코드 옆에 붙는 보조 상세라 (서보 알람 번호·축 비트·키 코드) 없으면 `0`, 숫자가 아닌 기호면 `null` 입니다 (비상정지의 `EXIN` 등). **Mitsubishi 에서 알람을 식별할 때는 `number` 가 아니라 `category` 를 쓰세요**
- **message**: 표시 텍스트
- **category**: Fanuc: 알람은 원인 계열 (`Servo`, `Overheat`, `Spindle`, `PLC` 등: 미정의 타입은 숫자 문자열), 메시지는 출처 (`Operator message` = PMC/외부입력, `Macro message` = 파트프로그램 #3006). Siemens: 서버가 알려주는 소스 (예: `NCU`: 비어 있으면 `Alarm`). Mitsubishi: 조작반에 뜨는 알람 구분 (`EMG`, `S01`, `M01` 등)
- **severity**: `"alarm"` = 장비 이상(가공 불가) / `"warning"` = 정보성(가공 가능). Fanuc: 알람은 백그라운드 편집 에러(BG)만 warning 이고 나머지 alarm, 오퍼레이터/매크로 메시지는 전부 warning. Siemens: 서버의 심각도(1~1000)를 500 경계로 번역. Mitsubishi: NC 알람·PLC 알람은 alarm, 오퍼레이터 메시지와 스톱 코드는 warning (스톱 코드에는 `M0`·싱글블록 같은 정상 정지가 들어 있습니다). "지금 가공이 멈췄는가"는 이 필드가 아니라 `executionStatus` 로 판단하되, **알람 중에는 그 값도 기종에 따라 `3`(Run)일 수 있습니다** (그 주소 설명 참조). warning 인데 정지 상태면 매크로 `#3006` 등 오퍼레이터 개입 대기입니다
- Fanuc 은 한 번에 활성 **알람 최대 100건**, 오퍼레이터/매크로 **메시지 최대 17건**까지 실어 옵니다 (벤더 API 버퍼 한도). Mitsubishi 는 알람 종류별로 **10건씩**이라 합계 **최대 40건**입니다 (벤더 API 상한). 목록은 무거운 종류부터 담기므로, 넘칠 때 잘리는 쪽은 가벼운 메시지입니다.
- **raisedAt**: 발생 시각. **`Z` 로 끝나는 UTC** 입니다 (`"2026-07-29T11:11:03Z"`). Siemens 는 실제 시각, Fanuc·Mitsubishi 는 항상 `null` (활성 알람에 시각 정보가 없음)
  - 장비 화면(HMI)이 보여주는 시각과 **숫자가 다릅니다.** HMI 는 장비 시간대로 표시하고 이 값은 UTC 입니다. 같은 순간을 다르게 표기한 것이며, 변환은 장비의 시간대를 아는 쪽(호스트 앱)이 합니다. OPC-UA 이벤트에는 시간대 오프셋 필드가 있지만, 실측한 장비에서는 비어 있었습니다
  - **`/machine/currentDateTime` 과 직접 빼지 마세요.** 시간대가 다를 뿐 아니라 **출처 시계가 다릅니다.** 한 장비에서 두 시계가 18분가량 어긋나 있는 것을 실측했습니다 (시계 설정은 현장마다 다릅니다)

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
- `13` = Jog-REPOS · `14` = MDI-Reference · `15` = MDI-Teach in · `16` = MDI-Teach in-Reference · `17` = Auto-Teach in-Reference
- `99` = Unknown

`13`~`17` 은 **Siemens 전용**입니다. 기본 모드(Jog/MDI/Auto)에 보조 기능(REPOS·원점복귀·Teach in)이 겹쳐진 상태로, 조작반에서 그 조합을 고르면 나옵니다. Fanuc 은 같은 상황을 기본 모드 코드로만 내보내므로 이 값이 나오지 않습니다.

Mitsubishi 의 **RAPID**(수동 급속이송)는 `0`(Jog) 로 나옵니다. 수동 연속 이송이라는 점에서 Jog 와 같은 부류이고, 기종이 늘 때마다 번호를 새로 만들지 않는다는 규칙을 따릅니다. 조작반의 STEP 은 `10`(INC feed), TAPE 는 `12`(Remote)입니다.

`5`(모드 없음)는 **Fanuc 전용**입니다. 조작반이 모드 자리에 `****` 를 표시하는, 어느 기본 모드도 선택돼 있지 않은 상태입니다. `99`(Unknown)와 다릅니다: 이쪽은 장비가 "모드 없음" 이라고 분명히 답한 것이고, `99` 는 우리가 그 값을 해석하지 못한 것입니다. Siemens 엔 대응 상태가 없습니다.

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
- `4` = MSTR (Fanuc: 리트랙션/복구) · `5` = Interrupted (Siemens: 아래 참조) · `99` = Unknown (Fanuc 한정. Siemens 는 미등재 값을 `-17` 에러로 돌려줍니다)

`1` 과 `2` 는 정지의 **종류**가 다릅니다. 누가, 어디서 세웠는가로 갈립니다:

- `Stop` = **프로그램이 예정된 지점에서** 세운 것. M0/M1 을 만났거나 싱글블록 모드로 블록이 끝난 경우입니다. 항상 블록 경계에 서 있습니다.
- `Hold` = **조작자가 임의 시점에** 세운 것. 조작반의 정지 키(피드홀드)를 누른 경우입니다. 블록 중간에서도 멈춥니다.

⚠️ **버튼 이름과 상태 이름이 어긋납니다** (업계 관례). 조작반의 **정지(Stop) 버튼을 누르면 상태는 `Hold`** 가 됩니다. `Stop` 상태는 버튼이 아니라 프로그램(M0/M1·싱글블록)이 만듭니다. 재개는 둘 다 Cycle Start 입니다.

**알람이 걸렸을 때의 답이 기종에 따라 다릅니다.** 같은 상황(없는 서브프로그램을 불러 자동운전이 멎음)을 두 기종의 테스트 환경에서 밟은 결과입니다:

| | `executionStatus` | `alarmStatus` |
|---|---|---|
| Fanuc | `1` (Stop) | `2` |
| Mitsubishi | `3` (Run), 리셋할 때까지 | `2` |

제어기마다 자기 자동운전 상태를 표현하는 방식이 달라서입니다. 디메시는 이것을 일괄로 뒤집지 않습니다. 알람이 가공을 멈추는지는 알람마다 다르고(경고성 알람은 안 멈춥니다), 우리에겐 알람별로 그걸 아는 지식이 없어 강등하면 멀쩡한 경우를 틀리게 만듭니다.

**그래서 "지금 가공이 진행될 수 있는가" 는 `alarmStatus` 와 함께 보세요.** `alarmStatus` 가 `0` 이 아니면 `executionStatus` 가 `3` 이어도 실제로는 서 있을 수 있습니다.

**Mitsubishi 는 `0`~`3` 만 냅니다.** 이 기종은 상태 코드가 아니라 자동 운전 플래그 셋(운전 중 · 진행 중 · 일시정지)을 주므로 디메시가 위 어휘로 합칩니다. `Stop` 과 `Hold` 의 구분은 벤더 정의와 그대로 맞아떨어집니다. 벤더가 말하는 "일시정지" 가 *명령을 실행하던 도중 멈춤* 이라 위 `Hold` 와 같은 상태이고, 자동 운전 중이면서 진행도 일시정지도 아닌 자리가 블록 경계에 선 `Stop` 입니다.

`5` (Siemens 전용) 는 **정지됐지만 위 둘로 분류되지 않은 사유**입니다 (비상정지, 스핀들 대기 등 그 외 정지 사유). "멈췄다" 는 사실은 확실하니, 종류가 중요하지 않은 소비자는 `1`/`2`/`5` 를 묶어 "정지" 로 다뤄도 됩니다.

⚠️ **비상정지 중의 값은 기종별로 갈립니다** (테스트 환경에서 확인): Fanuc 과 Mitsubishi 는 `0`(Reset), Siemens 는 `5`(Interrupted). 이 주소가 같은 상황에서 다른 코드를 내는 지점이므로, E-stop 여부는 이 주소가 아니라 `/machine/channel/emergencyStatus` 로 판단하세요.

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

**`4` 는 `0`·`1`·`3` 의 상위집합입니다**. 그 셋 중 하나인데 어느 것인지 좁힐 수 없다는 뜻입니다. Siemens·Mitsubishi 에서는 디메시가 잔여 드웰 시간으로 판정하므로 그 두 기종에서는 `2` 아니면 `4` 만 나옵니다 (`0`·`1`·`3` 은 Fanuc 전용).

축이 실제로 움직이는지가 필요한데 기종이 `4` 를 낸다면 이 주소로는 알 수 없습니다. 자동 운전 중인지는 `/machine/channel/executionStatus` 로 확인하세요 (수동 조작 중의 이동은 두 주소 모두 답하지 않습니다).

## /machine/channel/emergencyStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

비상정지 상태입니다 (뜻은 `desc` 로 함께 옵니다): `0` = 정상, `1` = 비상정지. Fanuc 은 `2`(Reset: E-stop 을 해제하는 순간의 과도값, 1초 미만)가 스칠 수 있습니다 (테스트 환경에서 확인). `0` 이 아니면 "정상 아님" 으로 다루면 안전합니다.

Mitsubishi 는 알람 목록에 `EMG` 구분이 있으면 `1` 입니다. 비상정지의 **원인과 무관하게** 잡습니다.

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

**홀드·정지 중에는 세지 않습니다.** 피드홀드로 세워 둔 시간은 양 기종 모두 빠집니다 (Fanuc 은 실측 확인, Mitsubishi 는 홀드 포함/제외 카운터를 벤더가 나눠 주며 이 주소는 제외 쪽입니다). 프로그램을 걸어두고 자리를 비운 시간이 가동 시간으로 잡히지 않는다는 뜻입니다.

`/machine/powerOnDuration` 과 함께 읽으면 가동률의 재료가 됩니다. 켜져 있던 시간 중 실제로 운전한 시간의 비율. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

**쓰기는 지원하지 않습니다.** 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다.

Fanuc 은 파라미터 `6752`(분) + `6751`(분 미만 ms) 를 한 번의 호출로 함께 읽고(조작반 실적 화면의 `RUN TIME`), Mitsubishi 는 `GetStartTime` 입니다. **Mitsubishi 는 9999시간 59분 59초에서 누적이 멈춥니다** (`powerOnDuration` 과 같은 장비 사양).

## /machine/channel/partCountActual
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

지금까지 가공된 수량입니다. 작업을 바꿀 때 리셋하는 카운터입니다. `channel` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0}` 으로 리셋합니다. Linux 용 Fanuc 라이브러리는 이 쓰기가 쓰는 함수(`cnc_wrparam`)를 제공하지 않아 Linux 에서는 쓰기가 `-20` 입니다.

**이 값은 "실제로 생산한 개수" 가 아닙니다.** 제어기가 프로그램 종료(`M02`/`M30`)에 반응해 올리는 카운터이고, 그 반응 여부부터 장비 설정에 달렸습니다. 설정이 꺼져 있으면 올라가지 않고, 드라이런으로 돌려도 올라가며, 같은 프로그램을 두 번 돌리면 2가 됩니다. 작업자가 조작반에서 바꿀 수도 있습니다. **양품인지 아닌지는 제어기가 알지 못합니다.** 실적 집계에 쓰려면 호스트 앱이 프로그램·시간 정보를 겹쳐 판단해야 합니다.

**Fanuc 은 쓰기 직후 다시 읽으면 이전 값이 올 수 있습니다.** 제어기가 파라미터를 반영하는 데 시간이 걸립니다 (실측: 즉시 읽으면 8회 중 6회가 이전 값, `50`ms 뒤에는 모두 정상). 쓴 값을 확인하려면 잠시 뒤에 읽으세요. 같은 Fanuc 이라도 매크로 변수 쓰기는 즉시 반영되고, 파라미터만 그렇습니다. Siemens 는 즉시 반영됩니다.

Fanuc 은 파라미터 `6711`, Siemens 는 `actParts`, Mitsubishi 는 파라미터 `8002` 입니다. **Mitsubishi 는 현재 읽기 전용**이라 리셋 쓰기가 `-20` 입니다.

## /machine/channel/partCountRequired
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

만들어야 할 목표 수량입니다. `channel` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 100}`. `0` 이면 목표가 설정되지 않은 상태입니다. Linux 용 Fanuc 라이브러리는 이 쓰기가 쓰는 함수(`cnc_wrparam`)를 제공하지 않아 Linux 에서는 쓰기가 `-20` 입니다.

가공된 수량이 이 값에 도달하면 장비가 신호를 내거나 정지하도록 설정할 수 있는데, 그 동작 여부는 장비 설정에 달렸습니다. 디메시는 값만 전달합니다.

**Fanuc 은 쓰기 직후 다시 읽으면 이전 값이 올 수 있습니다.** 제어기가 파라미터를 반영하는 데 시간이 걸립니다 (실측: 즉시 읽으면 8회 중 6회가 이전 값, `50`ms 뒤에는 모두 정상). 쓴 값을 확인하려면 잠시 뒤에 읽으세요. Siemens 는 즉시 반영됩니다.

Fanuc 은 파라미터 `6713`, Siemens 는 `reqParts`, Mitsubishi 는 파라미터 `8003` 입니다. **Mitsubishi 는 현재 읽기 전용**입니다.

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

**측정이 멈추는 조건이 있습니다.** 프로그램이 정지 상태이거나 이송 오버라이드가 `0` 이면 세지 않습니다. Siemens 는 여기에 더해 급속이송 중, 공구가 활성이 아닐 때, 드웰(휴지) 중에도 세지 않습니다. Fanuc 도 절삭 이송을 기준으로 하지만 드웰 처리가 같은지는 확인되지 않았습니다.

**리셋 기준점이 기종마다 다릅니다.** Fanuc 은 계속 쌓이고, Siemens 는 기본값으로 제어기를 부팅하면 `0` 이 됩니다 (일반 전원 재투입은 무관). 양쪽 모두 조작반에서 작업자가 리셋할 수 있습니다.

**Siemens 는 이 측정을 꺼둘 수 있습니다.** 머신데이터 `27860` 으로 비활성화되어 있으면 항상 `0` 입니다.

**쓰기는 지원하지 않습니다.** 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다.

Fanuc 은 파라미터 `6754`(분) + `6753`(분 미만 ms) 를 합산하고, Siemens 는 `cuttingTime` 입니다. 쪼개진 두 파라미터는 **한 번의 호출로 함께** 읽으므로, 읽는 도중 분이 넘어가 값이 어긋나는 일은 없습니다.

## /machine/channel/programRunDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

**이번 자동운전 사이클의 실행 시간**입니다. 사이클을 새로 시작하면 `0` 부터 다시 셉니다. `channel` 필터. 반환 `int` (초) + `unit:"s"`, 읽기 전용.

누적값이 아닙니다. 장비 수명에 걸쳐 쌓이는 값(전원투입 시간·절삭 시간)과 달리 **한 번의 운전**을 잽니다. 프로그램이 정지 상태이거나 이송 오버라이드가 `0` 이면 세지 않습니다.

**초 미만은 버립니다.** 두 기종 다 밀리초 해상도를 제공하지만 경과시간 주소는 정수 초로 통일합니다. 조작반의 `CYCLE TIME` 표시와 같은 값이 나옵니다 (Fanuc 테스트 환경에서 확인). 사이클이 몇 초로 짧으면 최대 1초의 오차가 상대적으로 큽니다.

Fanuc 은 파라미터 `6758`(분) + `6757`(분 미만 ms) 를 한 번의 호출로 함께 읽고, Siemens 는 `actProgNetTime` 입니다.

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

Fanuc 은 파라미터 `6712`, Siemens 는 `totalParts` 입니다. **Mitsubishi 는 미지원**(`-20`)입니다. 그 제어기는 리셋되는 작업 카운터(`partCountActual`)만 표준으로 갖고, 누적을 세는 값은 기계 제작사가 PLC 로 구현하는 현장 설정이라 SDK 가 알 수 없습니다.

## /machine/channel/feedOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

이송 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. Fanuc 은 PMC G12 신호에서, Siemens 는 `feedRateIpoOvr` 노드에서, Mitsubishi 는 채널별 PLC 레지스터에서 읽습니다.

## /machine/channel/rapidOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

급속이송 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. **Fanuc·Mitsubishi 는 단계식**: `0` / `25` / `50` / `100` 만 나옵니다. Siemens 는 연속값.

두 기종의 `0` 은 "멈춤" 이 아니라 **그 장비가 정한 가장 느린 급속이송 단계**입니다 (조작반의 최저 단계). 실제 속도는 장비 설정에 달려 있어 이 주소로는 알 수 없습니다.

## /machine/channel/feedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

지령 이송속도 (F 지령값)입니다. 반환 `float`. Fanuc 은 모달 F, Siemens 는 `cmdFeedRateIpo`, Mitsubishi 는 `F command feed speed`(FA)입니다.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

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

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

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

**유지되는 것은 운전 중까지입니다.** 프로그램이 끝나 리셋 상태가 되면 `0` 으로 돌아갑니다 (Mitsubishi 실측: `N400` 을 마지막으로 실행하고 `M30` 이후 `0`). 마지막 N 이 계속 남아 있다고 가정하지 마세요.

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

**Mitsubishi 는 `programSequenceNumber` 와 한 쌍입니다.** 이 기종은 프로그램 안의 위치를 (프로그램 이름 · `N` 번호 · 그 N 으로부터의 블록 수) 세 값으로 지정하며, 조작반의 운전 검색도 같은 세 값을 받습니다. 그래서 이 숫자만으로는 위치가 정해지지 않고 `N` 과 함께 읽어야 위치가 정해집니다 (실측: `N100` 구간에서 `0`→`1`→`2`→`3`, `N500` 을 만나 `0`).

Siemens 의 행 번호는 **지금 실행 중인 파일 기준**입니다. 서브프로그램에 들어가면 서브 파일의 행 번호로 바뀌고, 메인으로 복귀하면 메인 파일의 행 번호로 돌아옵니다 (테스트 환경에서 확인). 그래서 이 숫자만으로는 어느 파일의 몇 행인지 알 수 없습니다. 메인의 3행과 서브의 3행이 같은 `3` 입니다. 파일까지 특정하려면 `/machine/channel/programName` · `/machine/channel/programNestLevel` 을 함께 읽으세요. 중첩 전환 순간(1초 미만)에는 세 값의 조합이 잠시 어긋난 샘플이 나올 수 있습니다 (레벨이 이름보다 먼저 갱신됨).

## /machine/channel/programName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

**현재 실행 중인** 프로그램의 이름(파일명)입니다. 서브프로그램에 들어가면 그 서브의 이름으로 바뀝니다. HMI 에서 선택된 메인은 `mainProgramName` 참조.

## /machine/channel/programPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

현재 실행 중인 프로그램의 전체 경로입니다 (예: `//CNC_MEM/USER/PATH1/O0001`). `channel` 필터. Siemens 는 NCK 내부 경로를 사용자 표기(`//NC/...`)로 변환해 돌려줍니다.

**Mitsubishi 는 폴더 부분이 실행 중인 프로그램의 것이라는 보장이 없습니다.** 이 기종엔 전체 경로를 주는 호출이 없어 디렉터리와 파일 이름을 따로 물어 잇는데, 벤더 API 에는 **실행 중인 프로그램의 디렉터리를 돌려주는 호출이 없습니다**. 다만 이 기종의 NC 메모리는 **디렉터리 구성이 고정**이라(폴더를 만들 수 없습니다. `directoryExists` 참조) 메인과 서브가 다른 폴더에 놓이는 일이 드물어, 실무에서는 대개 맞습니다. 파일 이름 쪽은 언제나 실행 중인 프로그램의 것입니다.

## /machine/channel/mainProgramName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

HMI 에서 **선택된 메인 프로그램**의 이름입니다. 실행 중 서브프로그램에 들어가도 변하지 않습니다 (그게 `programName` 과의 차이).

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
- **파일이어야 합니다.** 폴더 경로를 주면 `-18`. 없는 경로도 `-18`
- 선택만 할 뿐 **가공을 시작하지는 않습니다** (사이클 스타트는 조작반/PLC 몫)
- Siemens 는 서버의 파일 핸들링 `Select` 메서드를, Fanuc 은 CNC 메모리면 `cnc_pdf_slctmain`, 데이터 서버 경로면 `cnc_wrdsdncfile`, Mitsubishi 는 운전 검색 `Search` 를 씁니다

## /machine/channel/programCurrentBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

지금 실행 중인 블록의 **G코드 텍스트**입니다. `channel` 필터. 없으면 **빈 문자열**이며 `null` 이 아닙니다.

**"실행 중이 아닐 때" 의 답이 기종에 따라 다릅니다.** 프로그램을 걸어 두고 리셋 상태인 장비를 실측한 결과입니다:

| | Siemens · Mitsubishi | Fanuc |
|---|---|---|
| `programCurrentBlock` | `""` | 프로그램의 **첫 줄** |
| `programNextBlock` | 첫 블록 | 그 **다음** 줄 |

Siemens 와 Mitsubishi 는 "실행 중인 블록 없음" 을 표현할 수단이 있습니다 (Mitsubishi 는 벤더가 실행 위치를 `0`=운전 안 함으로 알려줍니다). Fanuc 의 `cnc_rdexecprog` 에는 그 표시가 없어 선독(look-ahead) 버퍼의 첫 줄이 그대로 "현재" 로 나가는데, 정지 중에 그 줄은 실제로는 **다음에 실행될 블록**입니다.

**조작반과 대조하면 바로 보입니다.** 프로그램 화면의 실행 위치 표시(강조 막대)가 리셋 상태에서는 첫 줄 **위**에 있습니다. 아직 아무 블록도 실행하지 않았다는 뜻이고, 그 상태를 그대로 옮긴 것이 빈 문자열입니다.

**그래서 "지금 무엇을 실행 중인가" 를 이 주소만으로 판단하지 마세요.** `/machine/channel/executionStatus` 를 함께 읽어 `3`(Run)일 때만 의미가 있다고 보는 것이 안전합니다.

**여러 개가 필요하면 한 요청으로 묶어 읽으세요.** `programLastBlock`·`programCurrentBlock`·`programNextBlock`·`programLookAhead` 는 함께 요청하면 **한 번의 장비 조회**로 처리되어 서로 아귀가 맞습니다. 따로 읽으면 그 사이에 블록이 넘어가 **직전·현재·다음이 연속이 아닌 조합**을 받게 됩니다 (실측: 따로 읽어 `G04 X8.`·`G04 X7.`·`G04 X5.` 라는 한 블록을 건너뛴 조합이 나왔고, 같은 구간을 묶어 읽으면 한 번도 어긋나지 않았습니다). 빠르기도 하지만 그보다 **일관성** 때문입니다.

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

## /machine/channel/programLastBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

직전에 실행된 블록의 G코드 텍스트입니다. `channel` 필터. 직전 블록이 없으면(프로그램의 첫 블록이거나 운전 중이 아닐 때) 빈 문자열입니다.

**Siemens·Mitsubishi 지원이고 Fanuc 은 `-20`** 입니다. Fanuc 의 `cnc_rdexecprog` 는 선독 버퍼, 즉 **앞으로 실행할** 블록만 담아 지나간 블록이 남지 않습니다.

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

줄바꿈은 기종에 상관없이 LF(`
`) 하나로 정규화됩니다. CR 은 제거되므로 `
` 으로 split 하면 됩니다.

## /machine/channel/programNestLevel
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

프로그램 호출 중첩 단계입니다 (뜻은 `desc` 로 함께 옵니다): `0` = 프로그램 없음, `1` = 메인, `2`~ = 서브프로그램 (L1, L2, …). `channel` 필터. **Siemens·Mitsubishi 지원** (Fanuc 은 미지원).

**세는 것은 실행이 아니라 프로그램 포인터의 깊이입니다.** 프로그램이 걸려 있으면 운전 중이 아니어도 `1` 입니다 (실측: 두 기종 모두 리셋·중단 상태에서 `1`). "지금 돌고 있나" 는 `/machine/channel/executionStatus` 가 답합니다.

Mitsubishi 는 벤더 값이 **서브프로그램을 몇 겹 파고들었는지**(메인이 `0`)라 우리 눈금과 한 칸 달라, 디메시가 맞춰서 내보냅니다. `0`(프로그램 없음)과 `1`(메인)을 가르기 위해 이 주소만 장비 조회가 한 번 더 붙습니다.

## /machine/channel/auxModal/auxModalValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "auxModal"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

보조 기능 모달 값입니다. `auxModal` 필터에 **레터**를 지정합니다 (예: `auxModal=M`, `S`, `T`, `D`, `H`, `F`). 반환 `float`. 예: `auxModal=T` → 지령된 공구번호, `auxModal=S` → 지령 회전수.

**한 블록에 M 을 여러 개 지령한 경우는 레터에 순번을 붙여 읽습니다**. `M` 이 첫 번째, `M2` 가 두 번째입니다 (`M8 M42 M13;` 처럼 한 줄에 거는 경우). 받는 개수는 제어기 사양이라 **Fanuc 은 `M3` 까지, Mitsubishi 는 `M4` 까지**입니다. Mitsubishi 는 `B` 에도 순번이 있어 `B4` 까지 받습니다.

**한 블록에 여러 M 이 있을 때 두 기종이 다르게 냅니다.** 같은 `M8 M5` 블록을 실측한 결과입니다:

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

안 받는 레터는 `-18` 이고 에러 문자열이 그 기종에서 쓸 수 있는 것을 알려줍니다.

**`S` 에는 순번을 붙이지 않습니다.** Mitsubishi 의 벤더 인덱스는 스핀들 번호지만 이 주소엔 `spindle` 필터가 없어 첫 스핀들로 고정합니다. 스핀들별 지령 회전수는 `/machine/channel/spindle/speedCommanded` 가 담당합니다.

**장비가 주는 모달 값을 그대로 냅니다. 번역하지 않으며 앞으로도 통일되지 않습니다.** `parameter`·`diagnosis` 와 같은 범용 통로입니다. 특히 **그 레터가 아직 지령되지 않은 상태의 표현이 기종마다 다릅니다**: 실측에서 Fanuc·Mitsubishi 는 `0`, Siemens 는 `-1` 이었습니다 (그 레터에 실제 값이 있으면 그 값이 나옵니다. 유휴 상태의 Siemens 도 `D` 는 `1`, `F` 는 `0`). **"지령된 게 있나" 를 `== 0` 으로 판정하면 Siemens 에서 걸리지 않습니다.**

통일하지 않는 것은 모달 값 자체가 "지령 없음" 을 따로 표현하지 않는 기종이 있기 때문입니다. Fanuc·Mitsubishi 의 `0` 은 지령되지 않음과 `0` 지령(`T0` 은 실제로 쓰이는 지령입니다)을 같은 값으로 표현합니다. 어느 값을 "없음" 으로 볼지는 그 장비를 아는 호스트 앱이 정하세요.

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
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

옵셔널 스톱(M01 유효) 스위치 상태입니다 (`true` = 켜짐). `channel` 필터.

**Mitsubishi 는 미지원**입니다 (`-20`). 이 기종은 조작반 스위치를 PLC 신호로 내는데, 이 스위치의 신호 자리를 아직 확인하지 못했습니다. 확인되면 추가합니다.

## /machine/channel/blockSkipOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

블록 스킵(`/`) 스위치 상태입니다 (`true` = 켜짐). `channel` 필터. 세 기종 모두 지원합니다.

**스킵 레벨이 여러 개인 기종에서도 이 주소는 평범한 `/` 하나만 봅니다.** 블록 앞에 번호를 붙여(`/2`·`/3` …) 구간마다 다른 스위치로 건너뛰게 하는 기능이 있는 제어기들이 있는데 (Siemens 는 레벨 `0`~`9` 로 문서화합니다), 이 주소가 답하는 것은 언제나 **번호 없는 `/`** 입니다. 번호 붙은 레벨을 읽는 주소는 없습니다.

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

## /machine/channel/variable/variableValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "variable"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

**매크로 변수(Fanuc·Mitsubishi) / R 파라미터(Siemens)** 를 읽고/씁니다 (read + write). `variable` 필터에 변수 번호 (예: `variable=100` → Fanuc·Mitsubishi `#100`, Siemens `R100`). 반환 `float`, 쓰기는 `{"value": 3.14}`. **읽기는** 범위/콤마 확장을 지원합니다. `variable=100-105` 는 6개 값 배열. 쓰기는 항상 단일 변수입니다 (확장 문법은 `-13` 으로 거절: 모든 쓰기 공통 규칙). Fanuc·Mitsubishi 의 **미설정(vacant) 매크로 변수는 `null`** 입니다. 조작반 커스텀 매크로 화면에 빈 칸(Fanuc 은 `DATA EMPTY`)으로 뜨는 그 상태이며 값 `0` 과 구분됩니다. 범위 확장에서도 그 자리만 `null` 이 됩니다 (예: `[3.14, null]`).

**쓸 수 있는 번호는 기종·옵션마다 다릅니다.** 디메시는 목록을 들지 않고 그대로 전달하므로, 그 장비에 없는 번호는 **`-18`** 로 돌아옵니다 (읽기·쓰기 모두. 에러 문자열에 벤더가 밝힌 사유가 함께 실립니다). 고칠 것은 `variable` 값 하나이며, 그 장비에 실제로 있는 번호는 조작반의 변수 화면이 알려줍니다. 범위 확장에 없는 번호가 섞이면 요청 **전체가 `-15`** 로 실패합니다 (부분 배열이 오지 않습니다). 미설정(vacant) 변수는 에러가 아니라 `null` 원소라 확장을 깨지 않는 것과 구분하세요. 문법 자체가 범위 밖인 번호(Fanuc 은 `0`~`89999`)도 같은 `-18` 이되, 이쪽은 장비에 묻지 않고 즉시 거절합니다.

**미설정으로 되돌리는 쓰기는 지원하지 않습니다**. 값은 숫자 하나이고, 한 번 값을 넣은 변수를 다시 빈 칸으로 만들려면 조작반에서 지워야 합니다.

## /machine/channel/toolOffset/toolLengthGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**M계(머시닝센터) 공구 길이 형상값**입니다 (오프셋 화면의 H 열). 공구를 측정해 넣는 기준값으로, 길이 보정(H 코드)의 바탕이 됩니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolLengthWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**M계 공구 길이 마모값**입니다 (오프셋 화면의 H 열). 가공 중 쌓이는 미세 보정분으로, 형상값은 그대로 두고 이쪽만 조정하는 것이 일반적입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**M계 공구경 형상값**입니다 (오프셋 화면의 D 열). 공구경 보정(G41/G42)이 참조하는 반경 기준값입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 **반지름**이지만, 오프셋 화면은 설정에 따라 **지름**으로 표시·입력하게 설정돼 있을 수 있습니다. 디메시는 장비가 저장한 값을 그대로 내보내며 임의로 환산하지 않습니다.

## /machine/channel/toolOffset/toolRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**M계 공구경 마모값**입니다 (오프셋 화면의 D 열). 공구 마모에 따른 반경 감소를 반영합니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 열로 나뉘지 않은 장비**도 여기 해당하며, 그때는 에러 문자열이 `toolOffsetValue` 를 쓰라고 안내합니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 **반지름**이지만, 오프셋 화면은 설정에 따라 **지름**으로 표시·입력하게 설정돼 있을 수 있습니다. 디메시는 장비가 저장한 값을 그대로 내보내며 임의로 환산하지 않습니다.

## /machine/channel/toolOffset/toolXGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계(선반) X 방향 공구 치수 형상값**입니다. 여기서 X 는 축 이름이 아니라 오프셋 화면의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolXWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 X 방향 공구 치수 마모값**입니다. 가공 중 누적되는 X 방향 보정분입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolZGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Z 방향 공구 치수 형상값**입니다. X 와 마찬가지로 축이 아니라 화면의 고정 열입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolZWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Z 방향 공구 치수 마모값**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolYGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Y 방향 공구 치수 형상값**입니다. X·Z 에 이은 **세 번째 열**이며, 옵션이 없는 선반에서는 `-20` 이 반환됩니다.

**기계에 따라 이 열의 화면 머리글이 `Y` 가 아닐 수 있습니다.** Mitsubishi 는 이 자리를 제3축에 배정하므로 C축 선반에서는 조작반이 `공구길이 C` 로 표시합니다 (벤더 매뉴얼도 이 열을 `C (Y*)` 로 적습니다). 주소가 약속하는 것은 **세 번째 오프셋 열**이며, 그 열이 어느 축인지는 조작반의 열 머리글이 알려 줍니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolYWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 Y 방향 공구 치수 마모값**입니다 (Y축 옵션 장비 전용). X·Z 에 이은 **세 번째 열**이며, 기계에 따라 화면 머리글이 `Y` 가 아닐 수 있습니다 (Mitsubishi C축 선반은 `마모 C`). 자세히는 `toolYGeometry` 를 보세요.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 노즈 반경 형상값**입니다. 노즈 반경 보정(G41/G42)이 참조하며, 팁 방향(`toolTipDirection`)과 함께 날끝 궤적을 결정합니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolNoseRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**T계 노즈 반경 마모값**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **보정 메모리가 선반 배치가 아닌 장비**도 여기 해당하며, 그때는 에러 문자열이 그 장비에서 되는 리프를 알려 줍니다. **실제 적용값 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

선반 공구의 **가상 날끝 위치 코드**입니다 (read + write). 노즈 반경 보정(G41/G42) 때 날끝이 노즈 중심 기준 어느 방위에 있는지 판정하는 코드입니다. 각도가 아니라 위치 코드이며, 배율 없는 정수 그대로 반환/입력합니다 (`{"value": 3}`).

- `1`~`8` = 방위, **`0`/`9` = 노즈 중심이 기준점** (가상 날끝이 아니라). 두 값은 같은 의미입니다. 노즈 중심이 기준점과 일치할 때 `0` 또는 `9` 를 쓴다고 Fanuc 0i-F 선반 매뉴얼(`B-64604EN-1/01` §5.2.2)이 정의합니다
- **`desc` 는 `0`·`9` 에만 붙습니다.** `1`~`8` 은 매뉴얼이 평면별 도해로 정의하고 그 도해가 평면(`G17`/`G18`/`G19`)별로 여러 벌이라, 같은 번호가 구성에 따라 다른 방위를 가리킵니다. 방위 해석은 그 기종 매뉴얼의 도해를 따르세요
- Siemens 대응 개념: cutting edge position (`toolArea/tool/toolEdge/toolTipDirection`). **두 트리는 같은 번호 체계와 같은 `desc` 어휘를 씁니다.** 주소만 다를 뿐 값은 그대로 비교·재사용할 수 있습니다
- **허용 범위가 기종마다 다릅니다**: Fanuc `0`~`9`, Siemens `1`~`9`, Mitsubishi `0`~`8`. 중심을 가리키는 코드도 각각 `0`/`9`, `9`, `0` 이라 **읽을 때는 `desc` 로 통일되지만 쓸 때는 그 장비의 범위를 지켜야 합니다** (Fanuc 에서 되는 `9` 를 Mitsubishi 에 그대로 보내면 `-16`)

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

열이 나뉜 장비에서는 `-20` 이 반환되며, **에러 문자열에 그 장비에서 되는 리프 목록**이 실려 옵니다 (예: `toolLength{Geometry,Wear}, toolRadius{Geometry,Wear}`). 즉 한 번 요청해 보면 그 장비의 보정 트리 모양을 알 수 있으므로, 어느 모델인지 미리 묻는 주소는 따로 없습니다.

보정 번호의 상한은 `/machine/channel/toolOffsetCount` 입니다. 그 범위를 벗어난 번호는 `-18` 입니다. 값의 단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요.

## /machine/channel/toolOffsetCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

공구 보정 레지스터의 **사용 가능 개수**입니다 (read 전용, `int`). 오프셋 번호는 `1`~이 값까지입니다. UI 가 테이블을 순회할 때 상한으로 쓰세요.

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
- **Siemens**: `G500`, `G54`~`G57`, `G505`~`G599`. 실제로 몇 개까지 있는지는 장비 설정에 달려 있어, 없는 지정자는 `-18` 과 함께 **그 장비에서 허용되는 목록**을 알려줍니다

⚠️ **`G500` 은 Fanuc `EXT` 와 다릅니다.** `EXT` 는 어느 `G5x` 가 활성이든 **그 위에 더해지는** 공통 오프셋이지만, `G500` 은 `G54`~`G57` 과 **같은 모달 그룹의 배타적 멤버**라 그것들과 동시에 활성일 수 없습니다. `G500` 이 걸린 상태는 설정 오프셋이 꺼진 상태이고 그 자리의 값은 보통 `0` 입니다. 지금 어느 것이 활성인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=7` 로 확인하세요. Siemens 에서 `EXT` 처럼 전 좌표계에 가산되는 몫은 이 주소가 아니라 **별도의 프레임**(조작반의 `Basic reference`·`Total basic WO` 행)에 있고, 디메시는 그것들을 개별 주소로 내지 않습니다. 합쳐진 결과는 `/machine/channel/axis/totalWorkOffsetValue` 가 답합니다.

**Siemens 는 기본 오프셋과 미세 조정(`Fine`)의 합을 반환합니다.** 장비가 그 합을 적용하고 조작반도 한 오프셋의 두 칸으로 보여주므로, 이 주소는 **실제 적용되는 값**을 냅니다. 조작반의 `Coarse` 칸만 보고 비교하면 다르게 보일 수 있습니다. 미세 조정만 따로 보려면 `/machine/channel/workOffset/axis/workOffsetFineValue` 를 쓰세요. Fanuc·Mitsubishi 에는 미세 조정 개념이 없어 값이 하나이고, 그래서 세 기종에서 이 주소의 뜻이 같습니다.

⚠️ **이 값은 저장된 평행이동입니다.** 두 가지가 더 있습니다. ① 워크좌표계에는 **회전·배율·미러**가 걸릴 수 있어(`workOffsetRotation`·`workOffsetScale`·`workOffsetMirrorOn`) 걸려 있으면 좌표 변환이 이 값만으로 결정되지 않습니다. ② 실제로 걸리는 총량에는 기준 오프셋 등이 더해져 이 값과 다를 수 있습니다 (`totalWorkOffsetValue`). 부품 좌표가 필요하면 계산하지 말고 `/machine/channel/axis/workPosition` 을 읽으세요. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 라 이 값이 곧 변환입니다.

`axis` 는 축 번호(1~) 또는 축 이름. `axis=1-3`·`workOffset=G54,G55` 확장 지원합니다. Fanuc 은 같은 workOffset 의 축 확장이 FOCAS 호출 1회로 묶입니다.

쓰기는 `{"value": 25.4}` (단일 축). **Fanuc·Mitsubishi 지원**: Siemens 는 이 값에 직접 쓰면 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요하고 이 프로토콜이 그 절차를 노출하지 않아 `-20` 으로 거절합니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

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

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하세요. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 `-20` 입니다.



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

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하세요. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 `-20` 입니다.

**Siemens 전용**입니다.

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

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비 쪽 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하세요. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 `-20` 입니다.

**Siemens 전용**입니다.

## /machine/channel/axis/totalWorkOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

지금 **실제로 걸려 있는 영점이동 총량**입니다 (축별 평행이동). 반환 `float`. **읽기 전용**. 조작반의 `Total WO` 행에 해당합니다.

`workOffsetValue` 가 **표에 저장된 값**이라면 이 주소는 **지금 적용되고 있는 값**입니다. 둘이 다르면 그 차이가 어디서 왔는지가 곧 문제의 단서입니다:

```
workOffsetValue?workOffset=G54     80.400    설정한 값
totalWorkOffsetValue              100.400    실제로 걸린 값
                                   ↑ 20.000 이 어디선가 더해졌다
```

총량에는 저장된 오프셋 말고도 **기준 오프셋(`Basic reference`)·기본 프레임·프로그램의 `TRANS`·사이클이 설정한 프레임**이 함께 들어갑니다. "설정은 그대로인데 부품이 어긋난다" 를 진단할 때 두 주소를 비교하세요. 저장된 값만 읽으면 더해진 몫이 보이지 않습니다.

⚠️ **이 값으로 부품 좌표를 계산하지 마세요.** 평행이동 총량일 뿐이고 **회전·배율·미러와 공구 길이 보정은 여기에 없습니다.** 공작물 좌표가 필요하면 `/machine/channel/axis/workPosition` 을 읽으세요. 장비가 그 모두를 적용한 결과입니다.

`workOffset` 필터를 받지 않습니다. "지금 걸린 것" 이라 지정자를 고를 대상이 없습니다. `axis=1-3` 확장을 지원하고, 쓰기는 지원하지 않습니다 (합산 결과라 되돌려 쓸 대상이 아닙니다).

**Siemens 전용**입니다. Fanuc·Mitsubishi 는 디메시가 사용하는 벤더 API 범위에서 이 총량을 직접 내주는 호출을 찾지 못해 현재 미지원입니다. 그쪽에서는 `workOffset=EXT` 로 공통 오프셋을 따로 읽어 활성 좌표계 값과 더하세요. 다만 그 합은 **이 주소와 뜻이 다릅니다**. 프로그램이 건 시프트(`G92`·`G52` 등)가 빠지는데, 그것이 바로 이 주소가 드러내려는 차이입니다. `machinePosition` 에서 `workPosition` 을 빼는 것도 답이 아닙니다. 그 차이에는 **공구 길이 보정이 섞여** 공구축 값이 어긋납니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/gModalCategory/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalCategory"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

활성 G모달을 **기종 무관 표준 그룹 번호**로 조회합니다 (`plcType` 처럼 디메시가 정한 벤더 중립 번호: 벤더 원시 그룹 번호가 아닙니다). `gModalCategory` 필터 값:

⚠️ **중립인 것은 "무엇을 묻는가"(그룹 번호)까지입니다. 돌아오는 값은 그 기종의 G코드라 기종 무관 분기에는 쓸 수 없습니다.** 같은 상태를 Fanuc 은 `G21`, Siemens 는 `G710` 이라고 답합니다. `desc` 가 의미를 알려주지만 그것은 **사람이 읽는 문구**이지 분기용 계약이 아닙니다 (문구는 바뀔 수 있습니다). 이 주소는 **자기 기종의 G코드를 아는 호스트 앱**을 위한 것입니다. 기종을 가로지르는 판단이 필요하면 그 사실을 직접 답하는 주소를 쓰세요. 예컨대 이송 실효값은 `feedActual`, 주속 회전수는 `speedActual` 입니다.

- `1` = motion: 이송 모드 (G00 급속 / G01 직선 / G02·G03 원호 …)
- `2` = plane: 가공 평면 (G17 XY / G18 ZX / G19 YZ)
- `3` = distanceMode: 절대/증분 (G90/G91) · **Fanuc T 계열(System A) 미지원** (U/W 어드레스 방식)
- `4` = units: 인치/미터 (G20·G70·G700 / G21·G71·G710)
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

**Mitsubishi 는 머시닝센터에서만 지원합니다** (여덟 그룹 모두). 벤더가 그룹 번호표를 **M 계열 기준으로만** 문서화하고 다른 기종에서는 같은 번호가 다른 기능일 수 있다고 밝히고 있어, 선반 등에서는 `-20` 으로 거절합니다. 엉뚱한 그룹의 값이 G코드처럼 생겨서 나가면 알아챌 방법이 없습니다. 그 기종에서도 벤더 원시 접근(`gModalGroup/gModal` 낱개, `gModalList` 전체)은 그대로 동작합니다 (원시 번호라 의미를 약속하지 않으므로).

이 주소의 예전 이름은 `/machine/channel/gGroup/gModal`(필터 `gGroup`)입니다. 옛 주소·옛 필터 그대로도 영구히 동작하지만, 문서와 대시보드는 이 이름만 안내합니다.

## /machine/channel/gModalGroup/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalGroup"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

`gModalList` 의 **한 자리를 그룹 번호로 집어** 읽습니다 (`string`). `gModalGroup` 필터는 **그 기종의 원시 그룹 번호**를 그대로 받으며, 번호 체계는 `gModalList` 의 배열 위치와 같습니다:

- **Fanuc**: FOCAS modal 그룹 `0`~`20`
- **Siemens**: G-펑션 그룹 `1`~`N` (N = 장비의 그룹 수. 실측 840D sl 은 `64`)
- **Mitsubishi**: 벤더 그룹 `1`~`21`

범위 밖 번호는 `-18` 로 거절하며 허용 범위를 함께 알려줍니다. 그 그룹에 걸린 모달이 없거나 그 기종에 없는 그룹이면 값이 **`null`** 입니다. `gModalList` 의 그 자리가 `null` 인 것과 같은 표현입니다.

그룹 번호의 의미는 각 벤더 소유라 **번역하지 않으며 앞으로도 기종 간에 통일되지 않습니다** (`plcAddress` 와 같은 원시 통과). 기종 무관하게 그룹을 고르려면 중립 번호를 받는 `/machine/channel/gModalCategory/gModal` 을 쓰세요. 값 역시 그 기종의 G코드 문자열이라 기종을 가로지르는 분기에는 쓸 수 없습니다 (자세히는 `gModalCategory/gModal` 설명 참조).

**Mitsubishi 에서 특히 유용합니다.** 이 기종은 왕복이 그룹당 1회라 `gModalList` 가 21왕복인데 이 주소는 1왕복이고, 그룹 번호표가 M 계열 전용이라 `gModalCategory/gModal` 이 `-20` 인 선반 등에서도 원시 번호로는 낱개 조회가 됩니다.

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

- **Fanuc**: FOCAS modal 그룹 0~20 순서. 그 기종에 정의 안 된 그룹은 `null`
- **Siemens**: `ncFkt` G-펑션 그룹 1~N 순서 (N = 장비의 그룹 수). 활성 G-펑션이 없는 그룹은 `null` 입니다 (실측: 64개 중 7개)
- **Mitsubishi**: 벤더 그룹 1~21 순서 (`GetGCodeCommand`). **길이는 항상 21** 이고, 그 기종에 없는 그룹(예: 15·16·21 은 M 계열 전용)이나 활성 모달이 없는 그룹은 `null` 입니다. 표기는 벤더 매뉴얼 예시대로 정수부 두 자리(`G02`·`G50.2`)라 Fanuc 과 같은 모양입니다

**배열 위치가 곧 그룹 번호입니다.** 없는 자리를 건너뛰지 않고 `null` 로 채우는 것도 그 대응을 지키기 위해서입니다. 앞으로 당겨 담으면 뒤 항목이 남의 자리에 앉습니다.

⚠️ **Mitsubishi 는 왕복이 그룹 수만큼(21회) 듭니다.** 벤더 API 는 그룹 단위 조회를 제공합니다. 모달 화면을 한 벌 그리는 용도이지 주기 폴링용이 아닙니다. 특정 그룹 하나만 필요하면 `gModalGroup/gModal`(1왕복)로 좁혀 읽으세요.

**이 가족의 세 주소 모두 값은 기종 무관이 아닙니다.** 이 목록과 `gModalGroup/gModal` 은 벤더 그룹 번호·순서 그대로이고, `gModalCategory/gModal` 도 그룹을 고르는 번호만 중립이며 값은 그 기종의 G코드입니다. **기종을 가로지르는 판단에는 셋 다 쓰지 마세요** (자세히는 `gModalCategory/gModal` 설명 참조).

## /machine/channel/axis/machinePosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축의 기계 좌표(machine coordinate)입니다. `axis` 필터로 축을 지정하며, 범위(`axis=1-3`)나 복수 지정(`axis=1,2`)이 가능합니다. 반환 타입은 `float` (64비트 배정밀도).

위치 계열 4종(machinePosition/workPosition/distanceToGo/relativePosition)은 모두 **실거리**: 장비 설정 단위(mm/inch) 그대로이며 조작반 표시와 일치합니다 (Fanuc 의 내부 정수 표현은 SDK 가 축별 소수점 배율로 정규화). 네 값 모두 축이 원점을 확립한 뒤에만 유효합니다. 전원 투입 직후라면 `/machine/channel/axis/axisReferencedOn` 을 먼저 확인하세요 (미확립 상태에서도 좌표값은 에러 없이 반환됩니다).

Mitsubishi 어댑터는 `axisReferencedOn` 을 아직 구현하지 않았으므로(그 주소는 `-20`), 원점 확립 여부는 조작반에서 확인하세요.

⚠️ **이 값은 공구 기준점(스핀들 끝단)의 좌표입니다.** `workPosition` 은 공구 선단이라 두 값의 차이에 **공구 길이 보정**이 들어갑니다. `machinePosition − 영점이동 = workPosition` 은 성립하지 않습니다 (활성 워크좌표계의 회전·배율·미러도 함께 걸립니다). 워크 좌표가 필요하면 직접 계산하지 말고 `workPosition` 을 읽으세요.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/distanceToGo
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

현재 블록에서 축의 **잔여 이동량**입니다. 반환 `float`.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/axisName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축 이름입니다 (예: `"X"`, `"Z1"`). `axis` 번호 ↔ 실제 축 대응을 확인할 때 사용.

## /machine/channel/axis/axisLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축(서보) 부하율입니다. 반환 `float` + `unit:"%"` (세 기종 동일). Fanuc 은 서보 부하 미터, Siemens 는 드라이브 부하, Mitsubishi 는 서보 모니터의 부하 전류(정격 대비 비율)입니다. 셋 다 **실측값의 현재값**입니다.

**`axisCurrent` 와 같은 물리량입니다.** 서보는 토크가 전류에 비례하므로 부하를 재는 것이 곧 전류를 재는 것이고, 이쪽은 그 값을 **모터 정격 연속전류로 나눈 비율**입니다. 그래서 기계·축이 달라도 비교되는 반면(80% 는 어디서나 80%), 절대 전류값이 필요하면 `axisCurrent` 를 쓰세요. 둘 사이 환산에는 그 모터의 정격 전류가 필요한데 디메시는 그 값을 내지 않으므로, **한쪽만 지원하는 기종이 있습니다.**

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

## /machine/channel/axis/axisFeedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

그 축 방향의 실제 이송 **성분**입니다 (**Siemens 전용**). 반환 `float`. Fanuc(FOCAS2)은 축별 이송을 제공하지 않아 미지원(`-20`)입니다.

공구가 경로를 따라가는 속도 자체가 아니라, 그 속도를 축 방향으로 분해한 몫입니다. 그래서 **같은 지령에서도 이동 방향에 따라 계속 변합니다.** XY 평면에서 `F1000` 을 지령했을 때 X 축만 움직이는 구간에서는 이 값이 1000 이지만, 45° 대각선 구간에서는 약 707 입니다.

축 성분들을 **더해도 경로 속도가 되지 않습니다** (벡터 크기라 707+707 이 아니라 √(707²+707²)=1000). 이 값은 해당 축이 자기 속도 한계에 걸려 경로를 제한하고 있는지 보는 용도입니다.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/axisCurrent
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

축 모터 전류입니다. 반환 `float` + `unit:"Ampere"` (양 기종 동일). Fanuc 은 진단값, Siemens 는 드라이브 파라미터 `R0078`.

**`axisLoad` 와 같은 물리량이며 단위만 다릅니다** (그쪽은 모터 정격 대비 `%`). **Mitsubishi 는 이 값을 암페어로 제공하지 않아 `-20`** 이고, 같은 측정이 `axisLoad` 로 `%` 로 나옵니다.

**Siemens**: 값이 드라이브에서 오므로, 그 축에 드라이브가 배정되지 않은 채널에서는 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

## /machine/channel/axis/axisTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

축 모터 온도입니다. 반환 `float` + `unit:"°C"` (세 기종 동일). Fanuc 은 진단 308번, Mitsubishi 는 서보 드라이브 모니터의 모터 온도입니다.

**Siemens**: 값이 드라이브에서 오므로, 그 축에 드라이브가 배정되지 않은 채널에서는 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

## /machine/channel/axis/axisInterlockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

축 인터록 상태입니다 (`true` = 인터록 걸림). **Fanuc 전용**.

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

한 번 확립되면 축이 어디로 움직여도 `true` 로 유지됩니다. "지금 원점 위치에 있는가" 라는 순간 상태가 아니라 **좌표계 유효성**입니다.

Fanuc 은 CNC→PMC 표준 신호 ZRF(`F120` 의 축별 비트)를, Siemens 는 `refPtStatus` 를 읽습니다 (둘 다 테스트 환경에서 확인). Fanuc 쪽 구현은 현재 축 16개까지, 멀티패스 장비는 1계통만 지원합니다.

## /machine/channel/axis/axisEnergyNet
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

축의 순소비 **전력량**(누적 소비 − 누적 회생)입니다. **Fanuc 전용** (진단 4920), `float` + `unit:"Wh"`.

전력량(Wh)은 전력(W)과 다릅니다. W 는 순간값, Wh 는 누적량입니다. 이 값은 주행거리계처럼 계속 쌓이므로, 특정 구간의 사용량은 **앞뒤로 두 번 읽어 빼세요**. 평균 전력(W)이 필요하면 `ΔWh ÷ Δ시간(h)` 로 구합니다.

## /machine/channel/axis/axisEnergyConsumed
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

축의 누적 소비 **전력량**입니다. **Fanuc 전용** (진단 4921), `unit:"Wh"`. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

## /machine/channel/axis/axisEnergyRegenerated
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

축의 누적 회생 **전력량**입니다 (감속 시 돌려받은 몫). **Fanuc 전용** (진단 4922), `unit:"Wh"`.

## /machine/channel/spindle/spindleLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들 부하율입니다. 반환 `float` + `unit`. Siemens 와 Mitsubishi 는 항상 `unit:"%"` 입니다. Fanuc 은 벤더 응답이 단위를 함께 실어 주어 장비 설정에 따라 `%` 또는 `rpm` 이 오고, 벤더가 그 밖의 단위 코드를 주는 드문 경우에는 `unit` 키가 생략됩니다. 소비자는 `%` 를 가정하지 말고 `unit` 을 보세요. Mitsubishi 는 스핀들 모니터의 부하 항목입니다.

**`unit` 이 `%` 일 때는 `spindleCurrent` 와 같은 물리량입니다**. 제어기가 모터 전류를 정격으로 나눠 부하율로 내기 때문입니다. 그래서 기계가 달라도 비교되는 반면(80% 는 어디서나 80%), 절대 전류값이 필요하면 `spindleCurrent` 를 쓰세요. 환산에는 그 모터의 정격 전류가 필요한데 디메시는 그 값을 내지 않으므로 **한쪽만 지원하는 기종이 있습니다.** Fanuc 이 `unit:"rpm"` 을 줄 때는 부하가 아니라 회전수라 이 관계가 성립하지 않습니다.

## /machine/channel/spindle/spindleOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. **Fanuc 은 채널 공통값** (G30 신호: `spindle` 필터 무시, 모든 스핀들에 같은 값), Siemens·Mitsubishi 는 스핀들별 값.

## /machine/channel/spindle/spindleCurrent
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_opcua_siemens"]
write: []
```

스핀들 모터 전류입니다. **Siemens 전용** (드라이브 파라미터 `R0078`). 반환 `float` + `unit:"Ampere"`.

**`spindleLoad` 와 같은 물리량이며 단위만 다릅니다** (그쪽은 모터 정격 대비 `%`). Fanuc·Mitsubishi 는 이 값을 암페어로 제공하지 않아 `-20` 이고, 같은 측정을 `spindleLoad` 로 받을 수 있습니다. 다만 Fanuc 은 장비 설정에 따라 `unit` 이 `rpm` 일 수 있으니 확인하고 쓰세요.

## /machine/channel/spindle/spindleTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들 모터 온도입니다. 반환 `float` + `unit:"°C"` (세 기종 동일). Fanuc 은 진단 403번, Siemens 는 드라이브 파라미터 `R0035`, Mitsubishi 는 스핀들 드라이브 모니터의 모터 온도입니다.

## /machine/channel/spindle/speedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들 **S 지령값**입니다. 반환 `float`. **`unit` 을 붙이지 않습니다**. 지령의 뜻이 스핀들 속도 모드에 따라 갈리기 때문입니다 (회전수 일정이면 회전수, 주속 일정이면 주속). 세 기종 모두 같습니다. **Fanuc 은 채널 모달 S 값** (`spindle` 필터 무시: S 지령이 채널 단위 개념), Siemens 는 스핀들별 `cmdSpeed`, Mitsubishi 는 스핀들별 S 지령 모달값입니다.

## /machine/channel/spindle/speedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

스핀들별 실제 회전수입니다. 반환 `float` + `unit:"rpm"` (세 기종 동일). Fanuc 은 `cnc_acts2`, Siemens 는 스핀들별 `actSpeed`, Mitsubishi 는 스핀들 모니터의 회전수 항목입니다. 세 기종 모두 `spindle` 필터로 대상 스핀들을 지정하며, **오버라이드가 반영된 실측값**입니다.

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
| `T7` 만 지령 (아직 `M06` 전) | `7` | 아직 이전 공구 |
| `T7 M06` 이 끝난 뒤 | `7` | `7` |

앞의 두 기종은 테스트 환경에서 확인했습니다. `M06` 없이 `T7` 만 준 직후 값이 `7` 이 되며, Mitsubishi 는 조작반의 공구번호 표시가 이전 공구에 그대로 있는 것까지 함께 관측했습니다. 리셋(`M30`)으로도 지워지지 않습니다.

**그래서 Fanuc·Mitsubishi 에서는 이 값을 "지금 깎고 있는 공구" 로 해석하면 안 됩니다**. 지령된 공구이기 때문입니다. 교환 시점이 중요한 용도라면 이 주소를 교환 신호로 쓰지 마시고 장비의 교환 완료 신호를 보세요. 실제로 물려 있는 공구는 이 두 기종의 SDK 통로로는 얻을 수 없습니다. 조작반에 뜨는 공구번호는 기계 제작사가 래더로 만드는 값이라 장비마다 다르고, `plcAddress` 와 같은 이유로 중립화가 성립하지 않습니다.

이 번호를 공구 트리의 `tool` 필터에 넣으면 그 공구의 이름·보정 세트 수·오프셋을 조회할 수 있습니다. 함께 필요한 `toolArea` 값은 `/machine/channel/toolAreaNumber` 가 알려줍니다 (연결 시 캐싱되어 추가 통신이 없습니다).

Fanuc 은 `T` 모달, Siemens 는 `actTNumber`, Mitsubishi 는 `GetCommand2` 의 T 지령 모달을 읽습니다.

## /machine/channel/activeToolEdgeNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

활성 공구에서 지금 **보정이 적용되고 있는 보정 세트**의 번호(`D`)입니다. `channel` 필터. 반환 `int`. Siemens 는 `actDNumber`, **Fanuc·Mitsubishi 는 고정 `1`** 입니다. 두 기종의 표준 오프셋 모델엔 공구에 딸린 날 계층이 없어 적용 중인 보정 세트가 언제나 하나뿐입니다 (Fanuc 의 공구 관리 기능 쪽은 현재 미지원). 공구가 아직 장착되지 않았어도 `1` 입니다. 이 주소가 답하는 것은 "몇 번 보정 세트냐" 이지 "공구가 걸렸냐" 가 아닙니다 (그건 `activeToolNumber` 가 `0` 으로 답합니다).

## /machine/channel/spindle/spindleEnergyNet
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

스핀들의 순소비 **전력량**(누적 소비 − 누적 회생)입니다. **Fanuc 전용** (진단 4930), `unit:"Wh"`. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

## /machine/channel/spindle/spindleEnergyConsumed
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

스핀들의 누적 소비 **전력량**입니다. **Fanuc 전용** (진단 4931), `unit:"Wh"`.

## /machine/channel/spindle/spindleEnergyRegenerated
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

스핀들의 누적 회생 **전력량**입니다. **Fanuc 전용** (진단 4932), `unit:"Wh"`.

## /machine/userData/userDataValue
```yaml
value_type: "object"
null_able: false
required_filters: ["userData"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Siemens **전역 GUD(SGUD)** 사용자 변수를 읽고/씁니다 (현재 **OPC-UA(Siemens) 전용**, NC 전체 공유 변수). 읽기(GET)와 쓰기(POST) 모두 지원합니다. GUD 는 변수마다 타입이 달라, 반환 타입은 `object` 입니다. 값이 무슨 타입인지 함께 알려주는 **자기 서술적(self-describing) 엔벨로프** `{"type":..,"data":..}` 로 옵니다. 필터는 `userData` 하나입니다. 이 주소는 **NC 전체 공유** 변수 전용이라 채널을 지정하지 않습니다.

**userData**: `SGUD:<이름>` 또는 `SGUD:<이름>[<인덱스>]` 형식입니다. 접두사는 GUD 정의 블록 이름 (현재 `SGUD` 지원: MGUD 등은 추후). **인덱스는 전부 장비 화면(HMI) 표기 그대로**: 화면에 보이는 번호를 그대로 입력하면 됩니다 (0부터 시작, OPC-UA 내부 번호로 자동 변환).

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

(구조형 GUD `AXIS` / `FRAME` 은 추후 지원 예정)

**data** (엔벨로프 안). 1개(스칼라 / `[i]` / `[r,c](열수)`)면 값 하나, `[i-j]` 범위면 JSON 배열:

- 스칼라/단일: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- 범위: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**쓰기(POST)**: 읽기와 **같은 object** 를 body 의 `value` 에 담습니다 (예: `{"value":{"type":"REAL","data":42.0}}` → 화면 표기 [i] 한 칸만 변경). `type` 으로 쓸 타입을 정하므로 읽기 없이 바로 씁니다. `data` 원소 개수는 범위 크기 (단일이면 1)와 정확히 일치해야 합니다.

**주의**: GUD 영역이 없는 구형 NCK 에서는 미지원 오류가 반환됩니다.

## /machine/channel/userData/userDataValue
```yaml
value_type: "object"
null_able: false
required_filters: ["channel", "userData"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Siemens **채널 GUD(채널별 SGUD)** 사용자 변수를 읽고/씁니다 (현재 **OPC-UA(Siemens) 전용**). `channel` 과 `userData` 두 필터가 필요하며, `channel` 이 가리키는 채널의 변수만 다룹니다. NC 전체가 공유하는 전역 변수는 이 주소로 접근하지 않습니다.

반환 타입은 `object` 입니다. GUD 는 변수마다 타입이 달라 값이 무슨 타입인지 함께 알려주는 **자기 서술적 엔벨로프** `{"type":..,"data":..}` 로 옵니다.

**userData**: `SGUD:<이름>` 또는 `SGUD:<이름>[<인덱스>]` 형식입니다. **인덱스는 장비 화면(HMI) 표기 그대로** 0부터 쓰면 됩니다 (내부 번호로 자동 변환).

- 인덱스 없음 → 스칼라 변수
- `[i]` → 1차원 배열의 원소 1개
- `[i-j]` → 1차원 배열의 i~j 범위 (양끝 포함)
- `[r,c](열수)` → 2차원 배열의 (행,열) 원소 1개. 대괄호는 화면 표기 그대로 적고, 배열의 **열 개수**만 괄호로 덧붙입니다 (장비가 열수를 알려주지 않아 함께 입력이 필요)
- 예: `?channel=1&userData=SGUD:_SC_C97[0,1](4)` (채널 1, 4열 2D 의 화면 표기 [0,1])

**type**: `BOOL`(참/거짓) · `CHAR`(문자 코드 0~255) · `INT`(정수) · `REAL`(64비트 실수) · `STRING`(문자열). 구조형 `AXIS`/`FRAME` 은 추후 지원 예정입니다.

**data**: 1개(스칼라 / `[i]` / `[r,c](열수)`)면 값 하나, `[i-j]` 범위면 JSON 배열입니다.

- 단일: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- 범위: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**쓰기(POST)**: 읽기와 **같은 object** 를 body 의 `value` 에 담습니다 (예: `{"value":{"type":"REAL","data":42.0}}`). `type` 으로 쓸 타입을 정하므로 읽기 없이 바로 씁니다. `data` 원소 개수는 범위 크기(단일이면 1)와 정확히 일치해야 합니다.

**주의**: GUD 영역이 없는 구형 NCK 에서는 미지원 오류가 반환됩니다.

## /machine/toolArea/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구 영역에 **등록된 공구의 수**입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기 전용. 등록된 공구가 없으면 `0` 입니다.

목록(`toolList`)이 돌려주는 항목 수와 같고 **장비의 같은 값**을 봅니다. 개수만 필요할 때 목록 전체를 받지 않아도 되도록 따로 둔 주소입니다. 공구 17개짜리 장비에서 실측하면 목록보다 훨씬 빠릅니다 (81ms 대 684ms).

없는 공구 영역을 지정하면 `-18` 로 거절됩니다.

**Siemens 전용**입니다. Fanuc 은 공구 관리 기능이 없으면 "공구" 라는 객체 자체가 없습니다. 보정 레지스터의 개수는 공구 수와 다른 값이라 대신 쓸 수 없습니다.

## /machine/toolArea/toolList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구 영역에 **등록된 공구 전부**의 목록입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `objectArray`, 등록된 공구가 없으면 빈 배열 `[]`.

항목: `{"toolNumber": 16, "toolName": "BALLNOSE_D8", "toolEdgeCount": 4, "sisterToolNumber": 9, "magazineNumber": 0, "pocketNumber": 0, "toolLocationType": "buffer"}`

공구 번호는 **연속되지 않습니다.** 공구 17개가 2번~18번을 쓰고 1번은 없는 식이라, 번호를 1부터 넣어보는 것으로는 무엇이 있는지 알 수 없습니다. 이 목록이 그 답이며, 항목의 `toolNumber` 를 그대로 `tool` 필터에 넣어 공구별 주소를 조회하는 것이 용법입니다.

**순서는 `toolNumber` 오름차순입니다.** 매번 같은 순서가 보장됩니다. **장비 화면의 순서와는 다릅니다**: 공구 목록 화면은 보통 이름순으로 정렬돼 있고 작업자가 정렬 기준을 바꿀 수 있어, 맞출 수 있는 하나의 "화면 순서" 라는 것이 없습니다. 화면과 같은 순서로 보여주려면 `toolName` 으로 정렬하세요.

**`toolNumber` 는 장비 화면의 `Loc.`(자리 번호)이 아닙니다.** 공구 관리를 쓰는 장비에서는 공구를 이름과 자매번호로 식별하므로 이 번호가 목록 화면에 나오지 않습니다 (공구 상세 화면의 `Tool number` 항목이 이 값입니다). 화면의 `Loc.` 을 `tool` 필터에 넣으면 **다른 공구를 조회하고도 성공으로 보입니다.** 두 번호가 우연히 같은 공구가 많아 알아채기 어렵습니다. 그 값은 `pocketNumber` 이며, 이 목록이 둘을 함께 담고 있어 대응을 확인할 수 있습니다.

- **toolNumber**: 공구 번호. `tool` 필터에 넣는 값
- **toolName**: 공구 이름. 이름을 쓰지 않는 장비에서는 빈 문자열
- **toolEdgeCount**: 보정 세트 개수 (인선 수가 아니고, **가장 큰 D 번호도 아닙니다**. 중간 삭제로 구멍이 나면 번호가 개수보다 클 수 있습니다: `/machine/toolArea/tool/toolEdgeCount` 참조)
- **sisterToolNumber**: 자매공구 번호 (같은 이름을 가진 공구들 사이의 순번)
- **magazineNumber**: 지금 꽂혀 있는 매거진(공구 저장고) 번호. 매거진 밖이면 `0`
- **pocketNumber**: 그 매거진 안의 포켓 번호. 매거진 밖이면 `0`
- **toolLocationType**: 자리의 종류. `"magazine"`(매거진에 있음) · `"buffer"`(스핀들 또는 교환기) · `"loading"`(반입·반출 위치) · `"none"`(실물 자리 없음)

이 목록은 **무엇이 있고 · 어떻게 부르고 · 어디 있나** 까지 답합니다. 오프셋·마모 같은 측정값은 보정 세트 단위라 담지 않습니다.

값이 없으면 키를 빼지 않고 `null` 입니다 (위치 세 필드는 예외: 같은 이름의 단독 주소와 값이 같습니다). **공구 번호 오름차순**으로 정렬해 돌려주므로 두 번 읽어 비교하는 것이 의미를 갖습니다. 없는 공구 영역을 지정하면 `-18` 로 거절됩니다.

앞의 네 필드는 잘 바뀌지 않지만 **위치 세 필드는 공구가 움직일 때마다 바뀝니다.** 이 목록은 화면을 그릴 때 한 벌 받아오는 용도이며, 지금 활성인 공구만 알고 싶다면 목록을 반복해 읽는 대신 `/machine/channel/activeToolNumber` 를 쓰세요 (이 목록 주소는 Siemens 전용이지만 `activeToolNumber` 는 모든 기종이 지원하며, Siemens 에서 그 값은 교환이 끝난 공구입니다).

**Siemens 전용**입니다. Fanuc 은 오프셋 테이블이 `1` 부터 촘촘히 채워져 있어 열거할 대상이 없습니다.

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

없는 공구를 지정하면 `-18` 로 거절됩니다. 이 주소는 공구를 만들지 않습니다. 이름의 길이·문자 제약은 장비가 판단하며 위반하면 에러가 돌아옵니다.

## /machine/toolArea/tool/sisterToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**자매공구 번호**입니다 (SINUMERIK `duploNo`). 같은 이름을 가진 공구들 사이의 순번으로, 앞선 공구의 수명이 다했을 때 어느 것이 대체 투입될지를 정합니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호).

반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 2}`. **Siemens 전용**입니다. 공구 관리 기능을 쓰는 장비에서는 공구 이름과 이 번호의 조합이 공구의 정체이므로, 이름이 같은 공구가 여럿일 때 이 번호로 구분합니다.

없는 공구를 지정하면 `-18` 로 거절됩니다. 값이 정수가 아니거나 `0`~`65535` 를 벗어나면 `-16` 입니다. 실제 유효 상한은 장비 설정이 정하며, 그보다 좁은 범위를 벗어난 값은 장비가 거절합니다.

## /machine/toolArea/tool/magazineNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구가 지금 꽂혀 있는 **매거진(공구 저장고)의 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

매거진에 없으면 `0` 입니다. 스핀들에 물려 가공 중이거나, 교환기가 옮기는 중이거나, 반입·반출 위치에 있거나, 공구 데이터만 등록되고 실물 자리가 없는 경우입니다. 장비는 이런 자리에도 자기 고유 번호(예: `9998`)를 붙이지만 디메시는 그 번호를 내보내지 않고 `0` 으로 뭉뚱그립니다. 그 번호는 장비 설정에 따라 달라져 소비자가 의존할 수 없습니다.

**지금 있는 곳이지 원래 자리가 아닙니다.** 공구가 스핀들에 물리면 이 값이 `0` 으로 바뀌고 매거진으로 돌아가면 다시 번호가 붙습니다. 원래 어느 자리에서 나왔는지는 이 주소가 답하지 않습니다.

**쓰기는 지원하지 않습니다.** 이 값은 실물 위치의 기록이라 디메시는 쓰기를 열지 않습니다. 장부와 실물이 어긋나면 이후 공구 교환에 영향을 줄 수 있으므로, 위치 변경은 장비의 공구 관리 절차로 하세요.

**Siemens 전용**입니다. 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `0` 입니다.

## /machine/toolArea/tool/pocketNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구가 꽂혀 있는 **매거진 안의 포켓 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

매거진 번호가 아파트의 동이라면 이 값은 호수입니다. 둘을 함께 읽어야 위치가 정해집니다. 매거진에 없으면 `0` 입니다 (스핀들에서 가공 중이거나, 교환기가 옮기는 중이거나, 반입·반출 위치에 있거나, 실물 자리가 없는 경우). 장비가 그런 자리에 붙이는 고유 번호(예: `9998`)는 내보내지 않습니다.

**터렛(선반)의 스테이션도 포켓으로 나타납니다.** 장비가 터렛 위치를 매거진의 포켓으로 모델링하기 때문입니다. 현장에서 "3번 스테이션" 이라 부르는 자리가 이 주소에서는 포켓 `3` 입니다.

**지금 있는 곳이지 원래 자리가 아닙니다.** 공구가 스핀들에 물리면 이 값이 `0` 으로 바뀌고 매거진으로 돌아가면 다시 포켓 번호가 붙습니다.

**쓰기는 지원하지 않습니다.** 이 값은 실물 위치의 기록이라 디메시는 쓰기를 열지 않습니다. 장부와 실물이 어긋나면 이후 공구 교환에 영향을 줄 수 있으므로, 위치 변경은 장비의 공구 관리 절차로 하세요.

**Siemens 전용**입니다. 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `0` 입니다.

## /machine/toolArea/tool/toolLocationType
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구가 **어떤 종류의 자리에 있는지** 나타냅니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `string`, **읽기 전용**. 값이 곧 뜻이라 별도 코드표가 필요 없습니다.

| 값 | 뜻 |
|---|---|
| `"magazine"` | 매거진(공구 저장고)에 꽂혀 있음 |
| `"buffer"` | 스핀들 또는 교환기(가공 중이거나 옮겨지는 중) |
| `"loading"` | 반입·반출 위치 |
| `"none"` | 실물 자리 없음(공구 데이터만 등록되어 있음) |

**기종이 늘어도 이 넷입니다.** 장비는 스핀들·교환기·반입출 위치에도 자기 고유의 매거진 번호(예: `9998`)를 붙이지만 그 번호는 장비 설정에 따라 달라져 소비자가 의존할 수 없습니다. 디메시가 이 넷으로 묶어 내보내므로, 어느 기종에 붙었는지 몰라도 값으로 분기할 수 있습니다.

`"buffer"` 는 스핀들과 교환기 그리퍼를 **구분하지 않습니다.** 장비가 둘을 같은 자리로 취급하기 때문입니다. 스핀들에 물린 공구가 무엇인지는 `/machine/channel/activeToolNumber` 가 답하고 그 주소는 모든 기종이 지원하므로, 둘을 겹쳐 보면 구분됩니다.

**쓰기는 지원하지 않습니다.** 자리를 바꾸는 것은 공구 이동의 몫이고, 이 값만 고치면 장부와 실물이 어긋납니다.

**Siemens 전용**입니다. 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `"none"` 입니다.

## /machine/toolArea/tool/toolEdgeCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

공구가 가진 **보정 세트(cutting edge)의 개수**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). Siemens 는 `numCuttEdges`, **Fanuc·Mitsubishi 는 고정 `1`** 입니다. 두 기종의 오프셋 모델은 오프셋(세트) 번호 하나가 곧 보정값 한 벌이라, 공구에 딸린 날이라는 계층이 없습니다 (공구 관리 기능을 쓰는 장비는 날 번호를 따로 갖지만 디메시는 현재 그쪽을 읽지 않습니다).

**개수이지 가장 큰 번호가 아닙니다.** 보통은 `1` 부터 이어지지만, 조작반에서 중간 날을 지우면 **번호에 구멍이 생기고 뒤 번호는 밀리지 않습니다.** 예를 들어 `1`·`2`·`3` 중 `2` 를 지우면 남는 것은 `1` 과 `3` 이고 이 값은 `2` 가 됩니다. 그래서 `toolEdge` 를 `1`~이 값으로 가정하면 안 됩니다.

없는 날은 읽기·쓰기 모두 `-18` 로 거절되므로, 어떤 번호가 실재하는지는 읽어 보면 알 수 있습니다.

**인선 개수와는 다른 값입니다.** "2날 볼엔드밀", "4날 엔드밀" 이라 할 때의 그 날은 물리적 인선 수이고, 이 값은 제어기가 그 공구에 대해 갖고 있는 보정 세트의 수입니다. 인선이 여럿이어도 모두 같은 높이·반경이면 보정 세트는 하나면 됩니다. 실측 예로 4날 커터가 `1`, 2날 볼엔드밀이 `3` 을 답했습니다.

Siemens 에서 없는 공구를 지정하면 `-18` 로 거절됩니다. 보정 세트가 `0` 개라고 답하지 않습니다.

## /machine/toolArea/magazineList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 공구 영역의 **매거진 목록**입니다. `toolArea` 필터. 반환 `objectArray`, **읽기 전용**.

**매거진 번호가 연속이라는 보장이 없으므로 이 목록으로 확인하세요.** Siemens 실측 예로 `1`·`9998`·`9999` 였습니다 (Mitsubishi 는 `1`~`5` 범위라 연속이지만, 기종에 상관없이 이 목록을 쓰면 됩니다). `1` 부터 `/machine/toolArea/magazineCount` 까지 세어 올라가면 찾지 못합니다.

각 항목:

| 필드 | 뜻 |
|---|---|
| `magazineNumber` | 매거진 번호. `/machine/toolArea/magazine/pocketCount` 의 `magazine` 필터에 그대로 넣습니다 |
| `pocketCount` | 그 매거진의 포켓 수 |

**실물 공구 저장고만 담습니다.** 장비 내부에선 스핀들·교환기와 반입출 위치도 매거진으로 취급하고 큰 번호(`9998`·`9999` 등)를 붙이지만, 디메시에서 "매거진" 은 공구 저장고 하나만 뜻합니다. 공구가 그런 자리에 있으면 `/machine/toolArea/tool/toolLocationType` 이 `"buffer"` / `"loading"` 으로 답하고 `/machine/toolArea/tool/magazineNumber` 는 `0` 을 줍니다.

**공구와 그대로 맞물립니다.** `/machine/toolArea/tool/magazineNumber` 가 돌려준 값으로 이 목록의 항목을 찾고, 그 번호를 `pocketCount` 에 그대로 넣을 수 있습니다.

**매거진 이름은 담지 않습니다.** 장비가 이름 필드를 갖고 있지만 현장에서 설정하지 않으면 뜻이 없습니다. 실측 장비에서는 40자리 매거진과 버퍼와 반입출 위치가 **모두 같은 문자열**을 돌려줬습니다. 세 항목에 같은 이름이 붙으면 목록이 고장난 것처럼 보이므로 넣지 않았습니다.

## /machine/toolArea/magazineCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 공구 영역의 **매거진 수**입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

**실물 공구 저장고만 셉니다.** 장비 내부에선 스핀들·교환기와 반입출 위치도 매거진으로 취급하지만(그렇게 세면 실측 장비가 `3`) 디메시는 그 자리들을 매거진으로 보지 않습니다. 공구가 거기 있으면 `/machine/toolArea/tool/toolLocationType` 이 `"buffer"` / `"loading"` 으로 답합니다.

**개수는 알려주지만 번호는 알려주지 않습니다.** 번호가 연속이 아니어서 이 값으로부터 유효한 매거진 번호를 유추할 수 없습니다. 번호가 필요하면 `/machine/toolArea/magazineList` 를 쓰세요. 개수만 필요할 때 목록 전체를 받지 않아도 되게 이 주소를 따로 둡니다.

공구 관리 기능이 없는 장비는 매거진 자체가 없어 `0` 입니다.

**Mitsubishi 는 매거진 번호가 `1`~`5` 로 고정 범위**이고, 그 중 포켓이 하나라도 있는 것만 셉니다. 스핀들과 대기 자리는 이 기종에서 매거진이 아니라 별도 개념이라 애초에 세어지지 않습니다.

## /machine/toolArea/magazine/pocketCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 매거진의 **포켓 수**입니다. `toolArea` + `magazine` 필터 (`magazine` 은 매거진 번호). 반환 `int`, **읽기 전용**.

**매거진 번호는 연속이 아닙니다.** 유효한 번호는 `/machine/toolArea/magazineList` 가 알려줍니다. `1` 부터 `/machine/toolArea/magazineCount` 까지 세어 올라가는 방식으로는 찾을 수 없습니다.

없는 번호는 `-18` 로 거절됩니다. **스핀들·교환기·반입출 위치의 번호도 거절됩니다.** 장비는 그 자리들에도 매거진 번호를 붙이지만 디메시는 매거진으로 보지 않습니다. 범위·콤마 확장을 지원합니다.

**Mitsubishi 주의**: 매거진 번호가 고정 범위 `1`~`5` 라 그 밖은 `-18` 이지만, **범위 안의 실재하지 않는 매거진은 거절 대신 `0` 으로 옵니다**. 이 기종의 매거진 존재 판정이 곧 포켓 수 조회라서입니다. 실재 여부가 필요하면 `/machine/toolArea/magazineList` 를 보세요.

## /machine/toolArea/magazine/pocketList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 매거진의 **포켓을 전부, 포켓마다 무엇이 들어 있는지**입니다. `toolArea` + `magazine` 필터. 반환 `objectArray`, **읽기 전용**.

각 항목:

| 필드 | 뜻 |
|---|---|
| `pocketNumber` | 포켓 번호. `1` 부터 `/machine/toolArea/magazine/pocketCount` 까지 빠짐없이 나옵니다 |
| `toolNumber` | 그 포켓에 든 공구 번호. **`0` 이면 빈 포켓** |

**다른 주소들이 답하지 못하는 방향입니다.** `/machine/toolArea/tool/pocketNumber` 는 "이 공구가 몇 번 포켓에 있나" 를 답하지만, "몇 번 포켓에 뭐가 있나" 와 "빈 포켓이 어디인가" 는 이 목록만 답합니다.

공구 이름·오프셋은 담지 않습니다. `/machine/toolArea/toolList` 가 번호로 그것들을 주므로 번호로 이어 붙이세요. 포켓마다 이름을 함께 읽으면 40포켓 매거진에서 읽는 값이 두 배가 됩니다.

버퍼(스핀들·교환기)와 반입출 위치의 번호는 `-18` 로 거절됩니다. 디메시는 그 자리들을 매거진으로 보지 않습니다.

**Mitsubishi**: 매거진 번호는 고정 범위 `1`~`5` 이며, 실재하지 않는 매거진은 에러로 거절됩니다.

## /machine/toolArea/magazine/pocket/toolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

그 포켓에 든 **공구 번호**입니다. `toolArea` + `magazine` + `pocket` 필터. 반환 `int`, **읽기 전용**.

**`0` 은 빈 포켓**입니다 (공구 번호는 `1` 부터). 없는 포켓은 `0` 이 아니라 `-18` 로 거절되므로 둘이 섞이지 않습니다. 유효한 포켓 범위는 `/machine/toolArea/magazine/pocketCount` 가 알려줍니다.

포켓을 하나만 볼 때 쓰고, 매거진 전체를 훑을 때는 `/machine/toolArea/magazine/pocketList` 가 왕복 한 번으로 끝냅니다.

버퍼(스핀들·교환기)와 반입출 위치의 번호는 `-18` 로 거절됩니다.

**쓰기는 지원하지 않습니다.** 포켓의 공구를 고쳐 쓰면 실물은 그대로인 채 장부만 바뀌어, 다음 공구 교환 때 교환기가 엉뚱한 포켓을 집습니다. 공구 이동은 매거진 명령의 몫이며 디메시는 그 명령을 노출하지 않습니다.

## /machine/toolArea/tool/toolExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 공구 번호가 **공구표에 등록되어 있는지** 여부입니다. `toolArea` + `tool` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원.

없는 공구를 물어도 에러가 아니라 `false` 입니다. 존재 여부를 묻는 주소이기 때문입니다.

**쓰기가 공구를 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. 이미 그 상태면 아무것도 하지 않고 성공합니다.

만들어지는 공구는 **날 1개짜리 빈 공구**이고 이름은 공구 번호 문자열입니다. 이어서 `/machine/toolArea/tool/toolName` 으로 이름을, `/machine/toolArea/tool/toolEdge/*` 로 오프셋을 넣으세요. 날을 더 붙이려면 `/machine/toolArea/tool/toolEdge/toolEdgeExists` 를 쓰세요. 공구 준비실에서 잰 값을 조작반을 거치지 않고 그대로 등록하는 흐름이 이것입니다.

**매거진 포켓을 차지한 공구는 지울 수 없습니다** (`-18`). 실물은 매거진에 남는데 등록만 사라지면 다음 공구 교환이 어긋나고, 되돌리려 해도 **디메시에는 공구를 그 포켓에 다시 배정하는 주소가 없습니다.** 먼저 조작반에서 공구를 빼세요. 지금 어디에 있는지는 `/machine/toolArea/tool/toolLocationType` 이 답합니다.

**Siemens 전용**입니다.

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

**쓰기가 날을 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. 이미 그 상태면 아무것도 하지 않고 성공합니다.

**날은 순서대로만 만들어집니다.** 장비는 **비어 있는 가장 작은 번호**에 날을 만듭니다 (번호 지정 없음). 그래서 그 번호가 아닌 것을 요청하면 만들지 않고 `-18` 로 거절하며, 다음에 만들어질 번호를 에러 문구에 실어 보냅니다. `D5` 가 필요하면 `3`·`4`·`5` 를 차례로 만드세요. 구멍이 있으면 그 구멍부터 채워집니다.

**1번 날은 지울 수 없습니다** (`-18`). 공구가 있는 한 남습니다. 공구째 지우려면 `/machine/toolArea/tool/toolExists` 를 쓰세요.

없는 공구에 날을 만들려 하면 `-18` 입니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolDisabledOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 공구가 **쓰지 말라고 표시되어 있는지** 여부입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `boolean`, 읽기·쓰기 모두 지원합니다.

`true` 면 제어기가 그 공구를 쓰지 않습니다. 프로그램이 부르면 거절하거나, 자매공구가 등록되어 있으면 그쪽으로 넘어갑니다 (`/machine/toolArea/tool/sisterToolNumber`).

**공구 단위입니다.** `toolEdge` 필터를 받지 않습니다. 보정 세트가 여럿인 공구도 잠금은 공구 전체에 걸립니다.

표시가 붙는 경로는 둘이지만 **값은 구분하지 않습니다**: 작업자가 조작반에서 직접 잠그는 경우와, 공구 수명 감시가 잔여를 다 쓴 시점에 제어기가 자동으로 잠그는 경우입니다. 어느 쪽이든 "지금 못 쓰는 공구" 로 뜻이 같아 갈라 볼 이유가 없습니다.

없는 공구를 지정하면 `-18` 로 거절됩니다.

쓰기도 지원합니다. `{"value": true}` 로 잠그고 `{"value": false}` 로 풉니다. 조작반에 가지 않고 원격으로 공구를 빼거나 되돌릴 수 있습니다.

**수명이 다해 잠긴 공구는 잠금만 풀면 곧 다시 잠깁니다.** 잔여 수명이 `0` 그대로이기 때문입니다. 그 경우에는 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 에 잔여를 되돌려 쓰세요. 인서트를 갈지 않은 채 잠금만 푸는 것은 다 쓴 날로 깎는다는 뜻이기도 합니다.

**Siemens 전용**입니다.

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

이미 그 상태면 아무것도 하지 않고 성공합니다. 없는 공구는 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolOversizedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구가 **포켓 하나보다 큰지** 여부입니다. 양옆 포켓을 비워 둬야 하는 굵은 공구입니다. `toolArea` + `tool` 필터. 반환 `boolean`, **읽기 전용**.

장비의 매거진 화면에서 `Z` 열이 이 값입니다. 몇 포켓을 차지하는지가 아니라 **하나를 넘는지 여부**만 답합니다.

**쓰기는 지원하지 않습니다.** 장비에 초과크기 전용 항목이 없고 위·아래·좌·우로 몇 칸을 차지하는지가 따로 저장되어 있어, `true` 를 받아도 어느 방향으로 몇 칸인지 정할 수 없습니다. 초과크기 지정은 조작반에서 하세요.

없는 공구는 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

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

공구 쪽의 잠금은 `/machine/toolArea/tool/toolDisabledOn` 이고 별개입니다. 포켓이 잠겨도 그 안의 공구는 잠긴 것이 아니며, 다른 자리로 옮기면 다시 쓸 수 있습니다.

이미 그 상태면 아무것도 하지 않고 성공합니다. 없는 포켓과 버퍼·반입출 위치의 번호는 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 공구의 **수명 감시 방식**입니다. `toolArea` + `tool` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 2}`.

| 값 | 뜻 | 수명 값의 단위 |
|---|---|---|
| `0` | 감시 없음 | 없음 |
| `1` | 시간(실제로 깎은 시간을 센다) | 초 |
| `2` | 개수(완성한 가공물 수를 센다) | 개 |
| `3` | 마모(오프셋이 한계까지 밀렸는지 본다) | 기계 설정 (mm/inch) |

**기종이 늘어도 이 넷입니다.** 장비 고유의 번호가 아니라 디메시가 정한 값이라 어느 기종에 붙었는지 몰라도 그대로 분기할 수 있습니다.

방식은 **공구가 하나 고르고, 값은 날마다 따로**입니다. 그래서 이 주소는 `toolEdge` 를 받지 않고, 수명 값 3종은 받습니다.

`0` 이면 수명 값 3종이 `-18` 로 거절됩니다. 그 공구에는 잴 것이 없습니다. 감시를 켜려면 이 주소에 방식을 먼저 쓰고, 그 다음 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 에 수명 총량을 넣으세요.

장비가 여러 방식을 **동시에** 켜 둘 수도 있습니다. 그 경우 이 주소는 시간 → 개수 → 마모 순으로 하나를 골라 답하고, 수명 값 3종도 같은 순서를 따르므로 방식과 값이 어긋나지 않습니다. "갈아야 하나" 의 답인 `/machine/toolArea/tool/toolLifeWarnOn` 과 `/machine/toolArea/tool/toolDisabledOn` 은 방식과 무관하게 항상 정확합니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolLifeWarnOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

그 공구가 **경고 한계에 닿았는지** 여부입니다. `toolArea` + `tool` 필터. 반환 `boolean`, **읽기 전용**.

`true` 면 잔여 수명이 경고선 밑으로 내려온 것입니다. **아직 쓸 수 있습니다.** 못 쓰게 된 것은 `/machine/toolArea/tool/toolDisabledOn` 이고, 둘은 독립이라 동시에 `true` 일 수 있습니다 (수명이 다해 잠긴 공구).

곧 교체할 공구를 미리 추리는 용도입니다. 잔여가 얼마나 남았는지는 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 이 답합니다.

**수명 값은 날별인데 이 플래그는 공구 단위입니다.** 재는 곳과 조치하는 곳이 다르기 때문입니다. 깎는 것은 날이지만 교체는 공구째 하고, 제어기가 넘어가는 자매공구(`/machine/toolArea/tool/sisterToolNumber`)도 공구 단위입니다. 그래서 **어느 날이든** 자기 경고선을 넘으면 이 값이 `true` 가 됩니다.

**어느 날이 넘었는지는 알려주지 않습니다.** 필요하면 날마다 `toolLifeRemaining` 과 `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit` 을 비교하세요. 날 개수는 `/machine/toolArea/tool/toolEdgeCount` 가 답합니다.

**제어기가 정하는 값이라 쓰기는 지원하지 않습니다.** 고쳐 써도 잔여 수명은 그대로라 다음 판정에서 되돌아갑니다.

감시가 꺼져 있으면 항상 `false` 입니다. 없는 공구는 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolEdge/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 날에 배정된 **수명 총량**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 6}`.

제어기는 잔여를 이 값에서 시작해 깎아 내려갑니다. 단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. `/machine/toolArea/tool/toolMonitorType` 참조. 감시가 꺼져 있으면 `-18` 로 거절됩니다.

개수 감시일 때는 **정수만 받습니다.** 소수를 보내면 장비가 성공을 답하고 값은 바뀌지 않으므로 SDK 가 먼저 거절합니다.

**Siemens 전용**입니다.

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

단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 감시가 꺼져 있으면 `-18`, 개수 감시에 소수를 쓰면 `-16` 으로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolEdge/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**경고선**입니다. 잔여가 이 값 밑으로 내려오면 제어기가 경고를 올립니다 (`/machine/toolArea/tool/toolLifeWarnOn`, **공구 단위**). `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 3}`.

교체 공구를 준비할 시간을 벌기 위한 값이라 `toolLifeTotal` 보다 작게 잡습니다. 단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 감시가 꺼져 있으면 `-18`, 개수 감시에 소수를 쓰면 `-16` 으로 거절됩니다.

**Siemens 전용**입니다.

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 이름 그대로 **반지름**인데, 공구 목록/오프셋 화면은 흔히 **지름(Ø)** 으로 표시합니다. 실측(2026-07): `BALLNOSE_D8` 의 저장값이 `4.0` 인데 HMI 는 `8.000` 으로 보여줍니다. 디메시는 장비가 저장한 값을 그대로 내보내며 2를 곱하지 않습니다.

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 이름 그대로 **반지름**인데, 공구 목록/오프셋 화면은 흔히 **지름(Ø)** 으로 표시합니다. 실측(2026-07): `BALLNOSE_D8` 의 저장값이 `4.0` 인데 HMI 는 `8.000` 으로 보여줍니다. 디메시는 장비가 저장한 값을 그대로 내보내며 2를 곱하지 않습니다.

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gModalCategory/gModal?gModalCategory=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

**날끝 위치 코드**입니다 (SINUMERIK `DP2`, `1`~`9`). 노즈 반경 보정 때 날끝이 노즈 중심 기준 어느 방위에 있는지를 나타내며, 각도가 아니라 위치 코드입니다. Fanuc 의 공구 보정 트리에도 같은 개념이 있고 (`0`~`9`), **번호 체계와 `desc` 어휘를 공유**하므로 기종이 달라도 값을 그대로 비교·재사용할 수 있습니다 (`desc` 는 `9` 에만: `1`~`8` 방위는 매뉴얼이 도해로만 정의하고 가공 구성별로 세 벌이라 싣지 않습니다). 유효 범위는 `1`~`9` 이며 **`0` 은 허용되지 않습니다.** *"The identifier 0 (zero) is not permitted as a cutting edge position"* (SINUMERIK 828D Tools Function Manual).

반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 3}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러가 돌아옵니다.

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

위 표는 **정본이 아니라 길잡이**입니다. 이 코드 체계는 Siemens 가 소유하고 계속 늘어나므로, 정확한 목록은 그 기종의 *SINUMERIK 828D Tools Function Manual*(840D sl 은 해당 공구 관리 편)에서 확인하세요. 조작반의 공구 목록 화면에서도 같은 번호가 보입니다.

알려진 코드는 `desc` 로 의미가 함께 오고 (`{"value": 500, "desc": "turning roughing tool"}`), 미등재 코드는 첫 자리 계열 desc 를 대신 씁니다 (`{"value": 573, "desc": "turning tool family"}`). **위 표의 계열 밖 코드는 `desc` 키 자체가 빠집니다** (`{"value": 300}`). 없는 뜻을 지어내지 않기 위해서입니다. `desc` 가 항상 있다고 가정하지 마세요.

선삭 공구(5xx)의 길이1/2 축 배정, 반경 해석(커터/노즈)을 판별하는 기준값이기도 합니다.

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 의도치 않은 보정 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다 (생성·삭제는 `toolEdgeExists`).

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

**각도를 쓰지 않는 공구는 `0.0`** 입니다. 밀링 공구가 그렇습니다. 실측에서 드릴은 `118.0`, 페이스밀은 `0.0` 이었습니다.

**장비 화면의 `N` 열은 이 값과 `toolTeethCount` 를 겸용합니다.** 드릴류면 각도를, 밀링이면 인선 수를 그 한 칸에 보여줍니다. 디메시는 한 주소가 공구 종류에 따라 다른 물리량이 되지 않도록 둘을 따로 냅니다. 화면의 그 숫자를 찾으려면 둘 중 값이 있는 쪽을 보세요.

**Siemens 전용**입니다. 없는 공구/날(D)을 지정하면 `-18` 로 거절됩니다.

## /machine/toolArea/tool/toolEdge/toolHNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

그 날에 배정된 **`H` 번호**입니다 (ISO 방언용). `toolArea` + `tool` + `toolEdge` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 5}`.

ISO 방언(Fanuc 식 프로그램)으로 가공할 때, 프로그램의 `H5` 는 **공구와 무관하게** 이 번호가 `5` 인 날의 보정을 적용합니다. 즉 공구를 `T` 로 고르고 보정을 `H` 로 따로 고르는 방식이며, 이 주소가 그 번호를 관리합니다.

`0` 은 **배정되지 않음**입니다. 장비의 공구 목록 화면에 `H` 열이 있고 그 값과 같습니다.

번호가 겹치지 않아야 하지만 **중복 검사는 장비가 합니다.** 이미 쓰이는 번호를 쓰면 거절이 에러로 돌아옵니다. 정수만 받고 음수는 `-16` 으로 거절합니다.

ISO 방언을 쓰지 않는 장비에서는 모든 날이 `0` 입니다. Siemens 표준 방식에서는 `D` 번호가 공구 안의 날을 고르므로 이 값이 필요하지 않습니다.

**Siemens 전용**입니다.

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

**Siemens 전용**입니다. 없는 공구/날(D)을 지정하면 `-18` 로 거절됩니다.

## /machine/channel/diagnosis/index/diagnosisValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "diagnosis", "index"]
read: ["nc_focas2_fanuc"]
write: []
```

진단 데이터 **한 행의 값**입니다 (**Fanuc 전용**, `float`, 읽기 전용). `channel=<채널>&diagnosis=<번호>&index=<순번>` 형식이며, 파라미터 통로와 같은 행 모델입니다: 축/스핀들 종속 진단이면 `index` 가 축/스핀들 번호, **단일값 진단은 행이 1개이므로 `index=1`** (`diagnosisValueList` 가 단일값을 원소 1개 배열로 주는 것과 같은 모델). 범위를 넘으면 실제 행 수를 함께 실어 `-18` 로 거절합니다. 축 하나만 주기 폴링할 때 전 행을 읽는 List 보다 가볍습니다 (호출 1회).

## /machine/channel/diagnosis/diagnosisValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["channel", "diagnosis"]
read: ["nc_focas2_fanuc"]
write: []
```

임의 진단 번호의 **값**입니다 (**Fanuc 전용**, `floatArray`). 축/스핀들 종속 진단은 축 수만큼의 배열, 비종속 진단은 원소 1개 배열. 진단에는 경로(채널)별 값이 있어 **채널 스코프**입니다. `channel=` 로 경로를 지정하며, 경로 공통 진단은 어느 채널로 읽어도 같은 값이 옵니다. 진단별 형식(행별 여부·행 수)은 첫 조회 때 벤더 검증으로 파악되어 채널별로 캐싱되므로 반복 폴링이 가볍습니다. `diagnosis=301,308` 처럼 콤마/범위 확장 시 진단별 경계가 보존된 중첩 배열로 옵니다.

## /machine/channel/diagnosisSection/diagnosisSubsection/index/diagnosisValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "diagnosisSection", "diagnosisSubsection", "index"]
read: ["nc_ezsocket_mitsubishi"]
write: []
```

NC 내부 데이터 **한 행의 값**입니다 (**Mitsubishi 전용**, `float`, 읽기 전용). **섹션 번호 · 서브섹션 번호 · 축 번호**로 지정합니다: `channel=<채널>&diagnosisSection=<섹션>&diagnosisSubsection=<서브섹션>&index=<순번>`.

**번호를 이미 알고 있는 경우를 위한 통로입니다.** 이 번호 체계는 벤더 소유라 **번역하지 않으며 · 앞으로도 통일되지 않습니다** (`plcAddress`·`diagnosis` 와 같은 부류). Fanuc 의 `diagnosis` 는 번호가 하나인데 이쪽은 둘이라 주소를 따로 두었습니다. 두 기종의 진단 데이터를 같은 주소로 부를 수 없습니다.

- 자주 쓰는 값은 `axisLoad`·`spindleLoad` 처럼 **이름 붙은 주소**로 따로 제공됩니다. 그쪽이 있으면 그쪽을 쓰세요. 이 주소는 그 밖을 위한 범용 통로입니다
- `index` 는 **행 순번**입니다 (1부터). 섹션에 따라 축 번호이거나 스핀들 번호이고, 행이 없는 데이터는 1행이므로 `index=1` 입니다. 범위를 넘으면 실제 행 수를 함께 실어 `-18`
- **수치가 아닌 데이터가 있습니다.** 이 통로는 10진·16진·실수·문자열을 모두 다루는데 이 주소는 `float` 이라 문자열은 `-18` 로 거절하고 실제로 읽힌 값을 에러에 실어 줍니다 (16진 표시 데이터는 값 자체가 정수라 그대로 옵니다)
- 없는 섹션·서브섹션은 `-18`

## /machine/channel/parameter/index/parameterValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "parameter", "index"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

CNC 파라미터 **한 행의 값**입니다 (**Fanuc·Mitsubishi**, `float`, 읽기/쓰기). `channel=<채널>&parameter=<번호>&index=<순번>` 로 지정하며, 번호는 그 기종 매뉴얼의 파라미터 번호 그대로입니다. **번역하지 않으며 · 앞으로도 통일되지 않습니다** (`diagnosis` 필터와 같은 벤더 소유 번호 체계라 기종 간 대응표가 성립하지 않음). 파라미터에는 경로(채널)별 값이 있어 **채널 스코프**입니다. 자주 쓰는 파라미터는 `partCountActual`·`powerOnDuration` 처럼 이름 붙은 주소로 따로 제공되며, 이 주소는 그 밖을 위한 범용 통로입니다.

**모든 파라미터는 행의 배열로 봅니다.** `index` 는 행 순번입니다: 축형 파라미터면 축 번호, 스핀들형이면 스핀들 번호 (화면의 행 순서 그대로, 1부터. 예: X1=1, Y1=2), **단일값 파라미터는 행이 1개이므로 `index=1`** (`parameterValueList` 가 단일값을 원소 1개 배열로 주는 것과 같은 모델). 범위를 넘으면 실제 행 수를 함께 실어 `-18` 로 거절합니다.

- 비트 단위 파라미터는 **바이트 값 그대로**(packed 정수) 오갑니다. 비트 분해/합성은 호출자 몫입니다. 특정 비트만 바꾸려면 읽고-수정-쓰기를 하세요 (그 사이 조작반 등 다른 변경과 경합할 수 있습니다).
- 소수(real) 파라미터는 장비의 소수 자릿수가 적용된 실수로 오가며, 쓰기도 같은 자릿수로 저장됩니다.
- **Mitsubishi 에는 수치가 아닌 파라미터가 있습니다** (예: 축 이름 `#1013` 이 `X`). 이 주소는 `float` 이라 표현할 수 없어 `-18` 로 거절하며, 실제로 읽힌 문자열을 에러에 실어 줍니다. `index` 는 축별 파라미터면 축 번호, 아니면 `1` 뿐입니다.
- 정수 파라미터에 범위를 넘는 값을 쓰면 `-16` 으로 거절합니다 (허용 범위를 에러에 함께 실어).
- **쓰기 주의**: 파라미터는 기계 거동을 바꿉니다. 장비가 파라미터 쓰기를 막아둔 상태면 벤더 에러(`-17`)로 거절되고, 일부 파라미터는 변경 후 전원 재투입을 요구합니다. Linux 용 Fanuc 라이브러리는 파라미터 쓰기를 제공하지 않아 쓰기는 `-20` 입니다.

## /machine/channel/parameter/parameterValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["channel", "parameter"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

CNC 파라미터의 **전 축 값 배열**입니다 (**Fanuc·Mitsubishi**, `floatArray`, 읽기 전용). 배열 길이는 벤더 검증으로 파악한 **행 수**입니다: 축형 파라미터면 축 수, 스핀들형이면 스핀들 수, 비축이면 원소 1개 (`diagnosisValueList`·형제 `parameterValue` 의 행 모델과 동일). 행별 여부·행 수는 첫 조회 때 벤더 검증으로 파악되어 채널별로 캐싱되므로 반복 폴링이 가볍고, `parameter=6711-6713` 처럼 범위/콤마 확장 시 파라미터별 경계가 보존된 중첩 배열로 옵니다.

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

없는 폴더를 지정하면 `-18` 입니다.

## /machine/ncMemoryPath/entryName
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

`ncMemoryPath` 가 가리키는 **항목의 이름**입니다. 경로의 마지막 세그먼트를 돌려주고(read: 순수 문자열 연산, 장비 통신 없음), **이름 변경**(write)을 합니다. 쓰기는 `{"value": "새이름"}` 이며 경로 구분자는 넣을 수 없습니다 (파일/폴더 공통). `entry`/`entryList` 와 같은 "항목" 을 가리킵니다.

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
- write `{"value": true}` → **에러**: 빈 파일 생성은 미지원. 파일 생성은 내용과 함께 `fileContent` 쓰기로 하세요

경로 끝 `/` 는 무시됩니다 (종류는 주소가 확정). 폴더는 `directoryExists` 사용.

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
- write `{"value": true}` → 폴더 생성
- write `{"value": false}` → 폴더 삭제 (**빈 폴더만**: 내용이 있으면 에러, 재귀 미지원)
- **Mitsubishi 는 폴더 생성·삭제를 지원하지 않습니다** (`-20`). 그 드라이브는 디렉터리 구성이 고정이라 폴더를 만들거나 지울 수 없습니다. 읽기는 정상입니다

경로 끝 `/` 는 무시됩니다. 파일은 `fileExists` 사용.

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

## /machine/plcAddress/plcType/plcValue
```yaml
value_type: "float"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

**PMC/PLC 메모리의 단일 원소**를 읽고/씁니다 (Fanuc FOCAS2 `pmc_rdpmcrng`/`pmc_wrpmcrng`, Siemens OPC-UA `/Plc/` 노드, Mitsubishi EZSocket `ReadDevice`/`WriteDevice`). 읽기(GET)와 쓰기(POST) 모두 지원하며, 반환 타입은 `float` (값 하나), 쓰기도 숫자 하나입니다 (예: `{"value": 42}`). 이 주소는 **원소 하나 전용**이며, 여러 원소를 한 번에 다루려면 같은 트리의 목록형 주소를 사용합니다. `plcAddress` 와 `plcType` 두 필터가 필요합니다.

**plcAddress**: 주소 형식이 **기종별로 다릅니다**. `plcType` 과 달리 **중립화하지 않는 의도적 예외**입니다. Fanuc 의 `D100` 과 Siemens 의 `DB10.DBB56` 는 서로 다른 메모리 아키텍처를 가리키고, 둘을 잇는 대응표는 SDK 가 알 수 있는 지식이 아니라 그 장비의 래더를 어떻게 짰는지에 달린 현장 설정이기 때문입니다. 앞으로도 통일되지 않으므로, 여러 기종에서 같은 신호를 읽어야 한다면 **호스트 앱이 기종별 주소 표를 들고** 있어야 합니다.

- **Fanuc**: 첫 글자가 PMC 영역, 나머지가 바이트 번호 (예: `R5`, `D100`). `~` 로 범위 지정. 단, 이 주소(단일)는 범위가 **정확히 `plcType` 크기 1개**여야 합니다 (예: word 면 `D100~D101`)
- **Fanuc** PMC 영역 첫 글자: `G` `F` `Y` `X` `A` `R` `T` `K` `C` `D` `M` `N` `E` `Z`: 범위는 같은 영역이어야 합니다 (`D100~D101` O, `D100~R101` X)
- **Siemens**: 조작반의 **`NC/PLC variables` 화면에 보이는 표기 그대로** 씁니다. 값이 `/Plc/{주소}` 노드로 전달됩니다. 첨자 생략 시 `[1]` 자동 부착. 이 주소는 단일 원소만 되며, 다원소면 에러와 함께 목록형 주소를 안내합니다
- **Siemens** 형식: **주소가 오프셋을 품습니다**. `DB<n>.DBB<offset>`(바이트) · `DB<n>.DBW<offset>`(워드) · `DB<n>.DBX<byte>.<bit>`(비트) · `IW<n>` · `MB<n>` · `Q<byte>.<bit>`. 표기 예: `DB10.DBB56` · `DB31.DBX24.1` · `IW0` · `Q0.2`
- **Siemens** 첨자 `[N]` 은 **"몇 번째" 가 아니라 "몇 개"** 입니다. `DB10.DBB56[4]` 는 오프셋 56 부터 **연속 4개**(56·57·58·59)이지 "56번의 4번째" 가 아닙니다. 다른 자리를 짚으려면 첨자가 아니라 **주소를 옮깁니다** (`DB10.DBB61`)
- **Siemens** 문법 주의(기종 무관): 오프셋 없는 표기(`MB` 단독 · `DB<n>` 단독)는 문법이 아니고, 비트는 `DBB` 가 아니라 `DBX` 로 짚습니다
- **Siemens**: 어느 블록·바이트가 **그 장비에 실재하는지는 래더 구성에 달려 기계마다 다릅니다.** 위 표기 예도 형태를 보이기 위한 것이지 어느 장비에나 있는 주소가 아닙니다. 조작반의 같은 화면에서 확인하세요. 거기서 값이 보이면 여기서도 읽힙니다
- **Siemens** 828D 제약: 828D 는 **`DB9000` 이상의 고객 데이터 블록에만** 접근할 수 있습니다 (840D sl 은 제약 없음)
- **Mitsubishi**: 조작반의 PLC 화면 표기 그대로 `<디바이스><번호>` 입니다 (예: `R100`, `M50`, `Y8A0`). 점 수는 **`[N]` 첨자**로 붙이며, Siemens 와 같이 **"몇 번째" 가 아니라 "몇 개"** 입니다. `R100[4]` 는 `R100` 부터 연속 4점. 이 주소(단일)는 첨자 없이 쓰거나 `[1]` 이어야 합니다
- **Mitsubishi** 디바이스 번호의 진법이 계열마다 다릅니다. `M`·`R`·`D` 는 10진수, `X`·`Y`·`B` 는 **16진수**입니다 (조작반 표기와 같습니다)
- **Mitsubishi** 정렬: `M`·`X`·`Y` 처럼 **비트 단위로 번호가 매겨진 디바이스**는 byte·word·dword 로 읽을 때 시작 번호가 **8점 경계**에 있어야 합니다 (`Y890` O, `Y894` X). `R`·`D` 처럼 워드 단위 디바이스는 제약이 없습니다. 어긋나면 `-18` 이며, 디메시가 표로 판정하지 않고 **장비에 직접 물어** 가르므로 그 장비가 받는 주소는 그대로 통과합니다

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

**Mitsubishi** 는 `1`(bit)·`2`(byte)·`3`(word)·`4`(dword) 넷만 됩니다. `0`(auto)은 Fanuc 과 같은 이유로 안 되고(원시 메모리라 고유 타입이 없음), `5`·`6`(실수)은 이 기종의 PLC 디바이스 API 가 정수만 실어 나르기 때문입니다. 셋 다 `-18` 이며 주소 자체는 정상 동작합니다. `3`(word)·`4`(dword)는 **부호 있는 정수**로 해석됩니다. 모든 비트가 1인 워드는 `65535` 가 아니라 `-1` 입니다.

**에러 코드**: 그 장비에 **실재하지 않는 주소**도 `-18`(필터 값 오류)입니다. 어느 블록·바이트가 있는지는 그 장비 래더 구성에 달렸으므로, 조작반의 같은 화면에서 먼저 확인하세요. 그 기종이 못 쓰는 `plcType` 도 `-18` 입니다. 규약 밖 값(`0`~`6` 이외)도 같은 `-18` 이며, 두 경우 모두 대응은 같습니다(다른 `plcType` 지정). `-20` 이 아닌 이유는 **주소 자체는 그 기종에서 정상 동작**하기 때문입니다. `-20` 은 "이 주소를 이 기종에서 못 쓴다"는 뜻으로 남겨 둡니다. 에러 문자열에 허용 값이 함께 실려 옵니다.

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
- **Mitsubishi**: `[N]` 이 개수입니다 (`R100[4]` → 4개 배열). 한 번에 읽을 수 있는 최대 점 수가 타입마다 다릅니다: bit·byte `1280`, word `640`, dword `320`. 넘기면 `-18`
- **Mitsubishi**: `plcType=2`(byte)는 **한 점만 쓸 수 없습니다** (`-18`). 이 기종의 단일 디바이스 쓰기 호출에 byte 타입이 없고, 블록 쓰기는 2점 이상이라 옆 디바이스까지 함께 바뀌기 때문입니다. `3`(word)을 쓰거나 이 목록형 주소로 2점 이상을 지정하세요. **읽기는 한 점도 됩니다**
- 쓰기는 **원소 수가 대상 범위/노드의 원소 수와 정확히 일치**해야 합니다

**에러 코드**: 그 장비에 **실재하지 않는 주소**도 `-18`(필터 값 오류)입니다. 어느 블록·바이트가 있는지는 그 장비 래더 구성에 달렸으므로, 조작반의 같은 화면에서 먼저 확인하세요. 그 기종이 못 쓰는 `plcType` 도 `-18` 입니다. 규약 밖 값(`0`~`6` 이외)도 같은 `-18` 이며, 두 경우 모두 대응은 같습니다(다른 `plcType` 지정). `-20` 이 아닌 이유는 **주소 자체는 그 기종에서 정상 동작**하기 때문입니다. `-20` 은 "이 주소를 이 기종에서 못 쓴다"는 뜻으로 남겨 둡니다. 에러 문자열에 허용 값이 함께 실려 옵니다.

## /machine/ncMemorySizeTotal
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

NC 메모리 전체 용량입니다. 반환 `int` + `unit:"bytes"`.

크기는 SDK 전체에서 **바이트**로 통일되어 있습니다. `entry`/`entryList` 의 `sizeBytes` 와 같은 단위이므로 "이 파일이 남은 공간에 들어가나" 같은 계산에 변환이 필요 없습니다. 장비가 더 거친 단위(예: KB)로만 알려주는 경우에도 이 주소는 바이트로 환산해 내보내며, 그때 값은 그 단위의 배수가 됩니다.

가리키는 것은 **가공 프로그램 메모리**입니다. Mitsubishi 에서는 조작반 편집 화면의 `기억용량` + `나머지` 와 같은 값입니다.

## /machine/ncMemorySizeUsed
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

NC 메모리 사용량입니다. 반환 `int` + `unit:"bytes"`.

## /machine/ncMemorySizeFree
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

NC 메모리 잔여 용량입니다. 반환 `int` + `unit:"bytes"`.

**Mitsubishi 는 이 값이 250바이트 눈금으로 움직입니다**. 장비가 잔여를 250문자 단위로만 세기 때문입니다. 단위는 다른 기종과 같은 바이트이고, 값이 250의 배수가 될 뿐입니다.

## /machine/ncMemoryRootPath
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

메인 NC 메모리의 root 경로입니다. `ncMemoryPath` 필터에 넣을 경로의 시작점. Fanuc 은 보통 `//CNC_MEM`, Siemens 는 `//NC`, Mitsubishi 는 보통 `//PRG` 입니다.

**Mitsubishi 는 이 값 자체로는 목록이 비어 있습니다.** 프로그램은 한 단계 아래에 있어 `{root}/USER` 로 조회하세요 (MDI 버퍼는 `{root}/MDI`).

## /machine/ncMemoryExternalRootPathList
```yaml
value_type: "stringArray"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

메인 NC 메모리 root 외의 **외부 저장소 드라이브** 목록입니다 (예: 데이터 서버, 메모리 카드). 반환 타입 `stringArray`.

- 각 항목은 root 처럼 앞에 `//` 가 붙습니다 (뒤 슬래시는 없음). Fanuc `//DATA`·`//MEMCARD`, Siemens `//Local drive`, Mitsubishi `//IC1` (NC 쪽 SD카드, 조작반의 `DS`). 장착되지 않았으면 빈 배열입니다
- 이름은 **장비 HMI 표기**입니다. Siemens 의 로컬 드라이브는 OPC-UA 내부 이름이 `NCExtend` 이지만 조작반과 같게 `//Local drive` 로 내보냅니다 (옛 표기 `//NCExtend` 로 요청해도 받습니다)
- 메인 root 자신은 이 목록에서 제외됩니다
- **캐시하지 않음.** 외부 장치는 연결/해제로 바뀔 수 있어 요청마다 새로 조회
- 필터 없음
