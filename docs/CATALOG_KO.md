## /machine/channelCount
CNC 의 채널(계통) 수입니다. 연결 시 캐싱된 값이라 추가 통신 없이 즉시 반환됩니다.

## /machine/cncModel
CNC 모델 문자열입니다.

- **Fanuc**: 시리즈 번호 문자열입니다. `"15"`, `"16"`, `"18"`, `"21"`, `"30"`, `"31"`, `"32"`, `"35"`, `"0"`(0i), `"PD"`/`"PH"`(Power Mate i), `"PM"`(Power Motion i). `desc` 에 시리즈명이 함께 옵니다 (예: `"31"` → `Series 31i`)
- **Siemens**: 모델명 그대로 (예: `"840D sl"`)

## /machine/machineType
장비 종류입니다. 반환 `string`. 자기 기술 enum 이라 별도 코드표가 필요 없습니다. 나올 수 있는 값 전체:

- `"machiningCenter"`: 머시닝센터 (Fanuc M/MM, Siemens M)
- `"lathe"`: 선반 (Fanuc T/TT/MT, Siemens T)
- `"punchPress"`: 펀치 프레스 (Fanuc 전용)
- `"laser"`: 레이저 (Fanuc 전용)
- `"wireCut"`: 와이어 컷 (Fanuc 전용)
- `"unknown"`: 판별 불가

## /machine/currentDateTime
장비의 **현재 날짜/시각**입니다. 반환 `string`, ISO 8601 초 단위 (`"2026-07-11T14:30:00"`).

- **장비 로컬 시계**: 타임존 정보가 없으므로 TZ 접미사(`Z`/`+09:00`)를 붙이지 않습니다. ISO 8601 의 로컬 시각 형식이며, 오프셋을 필수로 요구하는 RFC 3339 파서는 이 값을 거부할 수 있습니다
- **자바스크립트 `new Date()` 에 그대로 넣지 마십시오**: 오프셋 없는 날짜+시각을 **보는 사람의 시간대**로 해석합니다. 이 값은 보는 사람이 아니라 **장비의 벽시계**입니다
- 서버 PC 시계가 아니라 **CNC 의 시계**입니다. 장비 시계가 틀어져 있으면 그대로 반영
- Fanuc: `cnc_gettimer` / Siemens: `sysTimeBCD`
- **장비 헬스체크 권장 주소**: 모든 프로토콜에서 실제 NC 왕복을 일으키는 값싼 읽기라, 주기 폴링 후 `status` 판정(`0`=정상, `-10`/`-14`=링크 이상)으로 장비별 통신 상태 감시에 쓰세요. (`machineType` 등 캐시 서빙 주소는 링크가 죽어도 성공할 수 있어 부적합)

시각 계열 주소는 항상 ISO 8601 문자열입니다 (`…At` = 이벤트 시점, `…DateTime` = 시계 읽기).

## /machine/powerOnDuration
장비의 **누적 전원투입 시간**입니다. 전원을 껐다 켜도 계속 누적되는 적산값. 반환 `int` (초) + `unit:"s"`.

- **Fanuc**: 파라미터 6750. **분 해상도**라 값이 항상 60의 배수입니다. 차분 계산 (가동률 등) 시 ±60초 오차 내재
- **Siemens**: `setupTime` 이라 분 미만 해상도까지 반영합니다 (초로 환산해 반올림). 일반 전원 재투입에는 리셋되지 않지만, **기본값으로 제어기를 부팅하면 `0`** 이 됩니다 (드문 정비 작업)

경과시간 계열 주소는 항상 **초 정규화 int** 입니다 (`…Duration` 접미사 규칙).

## /machine/configuredMachineName
`config.json`(허브) 또는 `deemesh_create` 설정의 `machine_name` 을 그대로 돌려줍니다. 장비가 보고하는 이름이 아니라 **설정에서 온 값**입니다. 이름에 `configured` 를 넣은 것도 그 때문입니다. 연결이 의도한 장비로 갔는지 확인하거나 응답을 식별할 때 씁니다.

## /machine/configuredProtocol
이 연결이 쓰는 **프로토콜 식별자**입니다. `"nc_focas2_fanuc"` 또는 `"nc_opcua_siemens"`. 필터 없음. 반환 `string`, 읽기 전용. 접속 시 정해지는 값이라 NC 통신 없이 즉시 응답합니다.

`configuredMachineName` 과 마찬가지로 **설정에서 온 값**입니다. 기계에 물어본 결과가 아니라 `deemesh_create` 의 `protocol` 필드(또는 `config.json` 의 머신 설정)를 그대로 돌려줍니다. 그래서 주소에 `configured` 가 붙습니다.

**용도는 좁습니다.** 대부분의 주소는 기종을 감추도록 설계되어 있어 분기가 필요 없습니다. 이 값이 필요한 곳은 **값 공간이 기종 소유인 소수의 자리**입니다. PLC 주소 문법(`D100` 대 `DB10.DBB56`), 진단 번호 체계, 공구 타입 코드처럼 카탈로그가 "기종에 따라 다르다" 고 명시한 곳들입니다.

**지원 여부 판단에는 쓰지 마십시오.** "이 기종은 이 주소를 못 쓰니 건너뛰자" 는 판단은 `-20` 으로 해야 합니다. 이 값으로 분기해 두면 나중에 그 기종 지원이 추가돼도 코드가 계속 건너뜁니다.

## /machine/channel/axisCount
채널의 **사용자 축 수**입니다. 연결 시 캐싱. `axis` 필터의 유효 범위가 `1`~이 값입니다.

기하축과 **비스핀들 보조축**(인덱싱 로터리 테이블·심압대 등)을 함께 세고 스핀들은 제외합니다. 스핀들은 `spindleCount` 와 `spindle` 필터가 담당합니다.

## /machine/channel/spindleCount
채널의 스핀들 수입니다. 연결 시 캐싱. `spindle` 필터의 유효 범위가 `1`~이 값입니다.

## /machine/channel/toolAreaNumber
채널이 사용하는 공구 영역(tool area) 번호입니다. 공구 트리 주소들의 `toolArea` 필터에 넣는 값입니다. Fanuc 은 고정 `1`, Siemens 는 NCK 설정(`toNo`)값.

## /machine/channel/alarmStatus
**알람 심각도**입니다. 기종과 무관하게 **`0` / `1` / `2` 세 값**만 반환하며, 그 기종에서 무엇이 문제인지는 `desc` 로 함께 옵니다.

| 값 | 의미 | 가공 |
|---|---|---|
| `0` | 정상(알람 없음) | 계속 |
| `1` | 경고(알람이 있으나 가공은 계속 가능) | 계속 |
| `2` | 알람(가공 정지) | 정지 |

- **숫자는 기종이 늘어도 이 셋뿐입니다.** 벤더 코드를 그대로 내보내지 않으므로, 어느 기종에 붙였는지 몰라도 `value` 로 바로 분기할 수 있습니다.
- **원인은 `desc` 로 옵니다.** Fanuc 은 원인 계열(`{"value": 1, "desc": "Memory backup battery voltage low (CNC or Amplifier)"}`), Siemens 는 정지 여부(`{"value": 2, "desc": "Alarm with stop"}`). `desc` 는 사람이 읽는 문자열이므로 **분기 조건으로 쓰지 마세요.** 분기는 `value` 로.
- 판정이 애매한 벤더 코드는 **보수적으로 `2`** 로 분류합니다. 정지를 경고로 낮춰 부르는 쪽이 그 반대보다 위험하기 때문입니다. Fanuc 은 SDK 가 모르는 코드를 새로 내보내도 `2` 로 분류합니다. Siemens 는 서버 값이 `0`/`1`/`2` 의 닫힌 집합이라 그 밖의 값은 가정이 깨졌다는 신호로 보고 `-17` 에러로 표면화합니다.
- 알람 **목록·번호·메시지**가 필요하면 `alarmList`, **개수**만 필요하면 `alarmCount` 를 쓰세요. 이 주소는 고빈도 폴링용 요약이라 가장 쌉니다.
- Siemens 는 채널별 알람 상태 노드(`chanAlarm`)를 읽으므로 `channel` 이 실제로 쓰입니다 (NCK 전역인 `alarmList`/`alarmCount` 와 다른 지점). 어느 기종이든 `channel` 값은 범위 검증됩니다.

## /machine/channel/alarmCount
활성 알람/메시지 **개수**입니다 (= `alarmList` 항목 수, severity 불문). 반환 `int`. 대시보드 배지처럼 개수만 필요할 때 쓰세요.

- **비용 주의 (Fanuc)**: count 전용 벤더 API 가 없어 내부적으로 `alarmList` 와 **같은 비용**입니다. 둘을 함께 배치 요청하면 fetch 1회로 합쳐집니다. 저비용 존재 판정만 필요하면 `alarmStatus` 를 쓰세요
- **Siemens**: 단일 노드 read (`numAlarms`) 라 `alarmList`(이벤트 구독)보다 훨씬 쌉니다. NCK 전역 개수라 `channel` 값은 무시됩니다 (`alarmList` 와 동일 방침)

## /machine/channel/alarmList
채널의 **활성 알람 + 오퍼레이터/매크로 메시지** 목록입니다. 반환 타입 `objectArray`, 없으면 빈 배열 `[]`. Siemens 는 알람이 NCK 전역이라 `channel` 값은 무시됩니다.

원소: `{"number": 1234, "message": "SPINDLE OVERHEAT", "category": "Overheat", "severity": "alarm", "raisedAt": "2026-07-29T11:11:03Z"}`

키 집합은 **기종과 무관하게 항상 같습니다.** 값이 없으면 키가 빠지는 게 아니라 `null` 입니다 (`entry` 와 같은 규약). `severity` 는 `"alarm"` / `"warning"` 두 값뿐입니다.

- **number**: 알람/메시지 번호
- **message**: 표시 텍스트
- **category**: Fanuc: 알람은 원인 계열 (`Servo`, `Overheat`, `Spindle`, `PLC` 등: 미정의 타입은 숫자 문자열), 메시지는 출처 (`Operator message` = PMC/외부입력, `Macro message` = 파트프로그램 #3006). Siemens: 서버가 알려주는 소스 (예: `NCU`: 비어 있으면 `Alarm`)
- **severity**: `"alarm"` = 장비 이상(가공 불가) / `"warning"` = 정보성(가공 가능). Fanuc: 알람은 백그라운드 편집 에러(BG)만 warning 이고 나머지 alarm, 오퍼레이터/매크로 메시지는 전부 warning. Siemens: 서버의 심각도(1~1000)를 500 경계로 번역. "지금 가공이 멈췄는가"는 이 필드가 아니라 `executionStatus` 로 판단하세요. warning 인데 정지 상태면 매크로 `#3006` 등 오퍼레이터 개입 대기입니다
- Fanuc 은 한 번에 활성 **알람 최대 100건**, 오퍼레이터/매크로 **메시지 최대 17건**까지 실어 옵니다 (벤더 API 버퍼 한도).
- **raisedAt**: 발생 시각. **`Z` 로 끝나는 UTC** 입니다 (`"2026-07-29T11:11:03Z"`). Siemens 는 실제 시각, Fanuc 은 항상 `null` (활성 알람에 시각 정보가 없음)
  - 장비 화면(HMI)이 보여주는 시각과 **숫자가 다릅니다.** HMI 는 장비 시간대로 표시하고 이 값은 UTC 입니다. 같은 순간을 다르게 표기한 것이며, 변환은 장비의 시간대를 아는 쪽(호스트 앱)이 합니다. 시간대를 알려주는 표준 필드가 있지만 이 장비는 비워서 보냅니다
  - **`/machine/currentDateTime` 과 직접 빼지 마십시오.** 시간대가 다를 뿐 아니라 **출처 시계가 다릅니다.** 한 장비에서 두 시계가 18분가량 어긋나 있는 것을 실측했습니다

## /machine/channel/operateMode
현재 운전 모드 코드입니다 (`desc` 동봉). 기종 무관 통일 코드:

- `0` = Jog · `1` = MDI · `2` = Memory (자동) · `5` = 모드 없음 · `6` = Edit · `7` = Handle (핸들)
- `8` = Teach in Jog · `9` = Teach in Handle · `10` = INC feed · `11` = Reference (원점복귀) · `12` = Remote (DNC)
- `13` = Jog-REPOS · `14` = MDI-Reference · `15` = MDI-Teach in · `16` = MDI-Teach in-Reference · `17` = Auto-Teach in-Reference
- `99` = Unknown

`13`~`17` 은 **Siemens 전용**입니다 — 기본 모드(Jog/MDI/Auto)에 보조 기능(REPOS·원점복귀·Teach in)이 겹쳐진 상태로, 조작반에서 그 조합을 고르면 나옵니다. Fanuc 은 같은 상황을 기본 모드 코드로만 내보내므로 이 값이 나오지 않습니다.

`5`(모드 없음)는 **Fanuc 전용**입니다 — 조작반이 모드 자리에 `****` 를 표시하는, 어느 기본 모드도 선택돼 있지 않은 상태입니다. `99`(Unknown)와 다릅니다: 이쪽은 장비가 "모드 없음" 이라고 분명히 답한 것이고, `99` 는 우리가 그 값을 해석하지 못한 것입니다. Siemens 엔 대응 상태가 없습니다.

## /machine/channel/executionStatus
프로그램 **실행 상태** 코드입니다 (`desc` 동봉). `operateMode`(무슨 모드인가)와 짝을 이루는 "지금 돌고 있는가":

- `0` = Reset · `1` = Stop · `2` = Hold · `3` = Run (실행 중)
- `4` = MSTR (Fanuc: 리트랙션/복구) · `5` = Interrupted (Siemens: 아래 참조) · `99` = Unknown (Fanuc 한정 — Siemens 는 미등재 값을 `-17` 에러로 표면화)

`1` 과 `2` 는 정지의 **종류**가 다릅니다. 누가, 어디서 세웠는가로 갈립니다:

- `Stop` = **프로그램이 예정된 지점에서** 세운 것. M0/M1 을 만났거나 싱글블록 모드로 블록이 끝난 경우입니다. 항상 블록 경계에 서 있습니다.
- `Hold` = **조작자가 임의 시점에** 세운 것. 조작반의 정지 키(피드홀드)를 누른 경우입니다. 블록 중간에서도 멈춥니다.

⚠️ **버튼 이름과 상태 이름이 어긋납니다** (업계 관례). 조작반의 **정지(Stop) 버튼을 누르면 상태는 `Hold`** 가 됩니다. `Stop` 상태는 버튼이 아니라 프로그램(M0/M1·싱글블록)이 만듭니다. 재개는 둘 다 Cycle Start 입니다.

`5` (Siemens 전용) 는 **정지됐지만 위 둘로 분류되지 않은 사유**입니다 (비상정지, 스핀들 대기 등 그 외 정지 사유). "멈췄다" 는 사실은 확실하니, 종류가 중요하지 않은 소비자는 `1`/`2`/`5` 를 묶어 "정지" 로 다뤄도 됩니다.

⚠️ **비상정지 중의 값은 기종별로 갈립니다** (실기 검증): Fanuc 은 `0`(Reset), Siemens 는 `5`(Interrupted). 이 주소가 같은 상황에서 다른 코드를 내는 유일한 지점이므로, E-stop 여부는 이 주소가 아니라 `/machine/channel/emergencyStatus` 로 판단하십시오.

## /machine/channel/motionStatus
축 이동 상태 코드입니다 (`desc` 동봉):

- `0` = None/Idle (Fanuc) · `1` = Motion (이동 중) · `2` = Dwell (드웰 중)
- `3` = 다계통 동기 대기 (Fanuc)

Siemens 는 잔여 드웰 시간으로 판정하므로 `1`/`2` 만 나옵니다.

## /machine/channel/emergencyStatus
비상정지 상태입니다 (`desc` 동봉): `0` = 정상, `1` = 비상정지. Fanuc 은 `2`(Reset: E-stop 을 해제하는 순간의 과도값, 1초 미만)가 스칠 수 있습니다 (실기 검증). `0` 이 아니면 "정상 아님" 으로 다루면 안전합니다.

## /machine/channel/partCountActual
지금까지 가공된 수량입니다. 작업을 바꿀 때 리셋하는 카운터입니다. `channel` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 0}` 으로 리셋합니다. Linux 용 Fanuc 라이브러리는 이 쓰기가 쓰는 함수(`cnc_wrparam`)를 제공하지 않아 Linux 에서는 쓰기가 `-20` 입니다.

**이 값은 "실제로 생산한 개수" 가 아닙니다.** 제어기가 프로그램 종료(`M02`/`M30`)에 반응해 올리는 카운터이고, 그 반응 여부부터 장비 설정에 달렸습니다. 설정이 꺼져 있으면 올라가지 않고, 드라이런으로 돌려도 올라가며, 같은 프로그램을 두 번 돌리면 2가 됩니다. 작업자가 조작반에서 바꿀 수도 있습니다. **양품인지 아닌지는 제어기가 알지 못합니다.** 실적 집계에 쓰려면 호스트 앱이 프로그램·시간 정보를 겹쳐 판단해야 합니다.

**Fanuc 은 쓰기 직후 되읽으면 이전 값이 올 수 있습니다.** 제어기가 파라미터를 반영하는 데 시간이 걸립니다 (실측: 즉시 읽으면 8회 중 6회가 이전 값, `50`ms 뒤에는 모두 정상). 쓴 값을 확인하려면 잠시 뒤에 읽으십시오. 같은 Fanuc 이라도 매크로 변수 쓰기는 즉시 반영되고, 파라미터만 그렇습니다. Siemens 는 즉시 반영됩니다.

Fanuc 은 파라미터 `6711`, Siemens 는 `actParts` 입니다.

## /machine/channel/partCountRequired
만들어야 할 목표 수량입니다. `channel` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 100}`. `0` 이면 목표가 설정되지 않은 상태입니다. Linux 용 Fanuc 라이브러리는 이 쓰기가 쓰는 함수(`cnc_wrparam`)를 제공하지 않아 Linux 에서는 쓰기가 `-20` 입니다.

가공된 수량이 이 값에 도달하면 장비가 신호를 내거나 정지하도록 설정할 수 있는데, 그 동작 여부는 장비 설정에 달렸습니다. 디메시는 값만 전달합니다.

**Fanuc 은 쓰기 직후 되읽으면 이전 값이 올 수 있습니다.** 제어기가 파라미터를 반영하는 데 시간이 걸립니다 (실측: 즉시 읽으면 8회 중 6회가 이전 값, `50`ms 뒤에는 모두 정상). 쓴 값을 확인하려면 잠시 뒤에 읽으십시오. Siemens 는 즉시 반영됩니다.

Fanuc 은 파라미터 `6713`, Siemens 는 `reqParts` 입니다.

## /machine/channel/cuttingDuration
**절삭 누적 시간**입니다. 공구가 실제로 물려 깎고 있던 시간의 적산값. `channel` 필터. 반환 `int` (초) + `unit:"s"`, 읽기 전용.

전원투입 시간과 함께 층을 이룹니다. 켜져 있던 시간 중 실제로 깎은 시간이 얼마인지가 가동률 계산의 재료입니다. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

**측정이 멈추는 조건이 있습니다.** 프로그램이 정지 상태이거나 이송 오버라이드가 `0` 이면 세지 않습니다. Siemens 는 여기에 더해 급속이송 중, 공구가 활성이 아닐 때, 드웰(휴지) 중에도 세지 않습니다. Fanuc 도 절삭 이송을 기준으로 하지만 드웰 처리가 같은지는 확인되지 않았습니다.

**리셋 기준점이 기종마다 다릅니다.** Fanuc 은 계속 쌓이고, Siemens 는 기본값으로 제어기를 부팅하면 `0` 이 됩니다 (일반 전원 재투입은 무관). 양쪽 모두 조작반에서 작업자가 리셋할 수 있습니다.

**Siemens 는 이 측정을 꺼둘 수 있습니다.** 머신데이터 `27860` 으로 비활성화되어 있으면 항상 `0` 입니다.

**쓰기는 지원하지 않습니다.** 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다.

Fanuc 은 파라미터 `6754`(분) + `6753`(분 미만 ms) 를 합산하고, Siemens 는 `cuttingTime` 입니다. 쪼개진 두 파라미터는 **한 번의 호출로 함께** 읽으므로, 읽는 도중 분이 넘어가 값이 어긋나는 일은 없습니다.

## /machine/channel/programRunDuration
**이번 자동운전 사이클의 실행 시간**입니다. 사이클을 새로 시작하면 `0` 부터 다시 셉니다. `channel` 필터. 반환 `int` (초) + `unit:"s"`, 읽기 전용.

누적값이 아닙니다. 장비 수명에 걸쳐 쌓이는 값(전원투입 시간·절삭 시간)과 달리 **한 번의 운전**을 잽니다. 프로그램이 정지 상태이거나 이송 오버라이드가 `0` 이면 세지 않습니다.

**초 미만은 반올림됩니다.** 두 기종 다 밀리초 해상도를 제공하지만 경과시간 주소는 정수 초로 통일합니다. 사이클이 몇 초로 짧으면 그만큼 오차가 큽니다.

Fanuc 은 파라미터 `6758`(분) + `6757`(분 미만 ms) 를 한 번의 호출로 함께 읽고, Siemens 는 `actProgNetTime` 입니다.

## /machine/channel/partCountTotal
장비 통산 가공 수량입니다. 작업을 바꿔도 리셋하지 않는 누적값입니다. `channel` 필터. 반환 `int`, **읽기 전용**.

쓰기를 지원하지 않는 것은 의도된 제한입니다. 장비의 이력이라 고치면 실적 집계가 조용히 어긋납니다. 리셋이 필요한 카운터는 별도로 있습니다.

이 값도 제어기가 프로그램 종료에 반응해 올리는 카운터라, 드라이런이나 재실행도 함께 셉니다. **"실제로 생산한 개수" 로 쓰면 안 됩니다.**

Fanuc 은 파라미터 `6712`, Siemens 는 `totalParts` 입니다.

## /machine/channel/feedOverride
이송 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. Fanuc 은 PMC G12 신호에서, Siemens 는 `feedRateIpoOvr` 노드에서 읽습니다.

## /machine/channel/rapidOverride
급속이송 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. **Fanuc 은 단계식**: `0` / `25` / `50` / `100` 만 나옵니다. Siemens 는 연속값.

## /machine/channel/feedCommanded
지령 이송속도 (F 지령값)입니다. 반환 `float`. Fanuc 은 모달 F, Siemens 는 `cmdFeedRateIpo`.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/feedActual
채널의 실제 이송속도입니다. **공구 끝이 프로그램 경로를 따라가는 속도**. 반환 `float`. Fanuc 은 `actf`, Siemens 는 `actFeedRateIpo`.

`F` 지령이 정하는 값이 이것입니다. 이동 방향이 바뀌어도 이 속도는 지령대로 유지되고, 오버라이드·가감속·코너 감속·피드홀드가 걸리면 그만큼 떨어집니다. "지금 지령대로 깎이고 있나" 는 이 값으로 판단합니다.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/programSequenceNumber
현재 실행 중인 블록의 **시퀀스 번호 (N 번호)** 입니다. 반환 `int`.

**N 번호가 없는 블록에서는 기종별로 다릅니다** (양쪽 실기 검증). Siemens 는 **항상 `0`** 입니다 (직전 N 을 유지하지 않으며, 같은 파일 안에서도 마찬가지). Fanuc 은 마지막으로 실행된 N 번호가 유지됩니다. `0` 을 "N 번호 없는 블록 실행 중" 으로 읽을 수 있는 건 Siemens 쪽뿐입니다.

N 이 있는 블록에서는 두 기종 모두 서브프로그램에 들어가면 **서브의 N 번호**가, 복귀하면 **메인의 N 번호**가 나옵니다.

⚠️ **Fanuc 의 유지는 파일 경계를 넘습니다** (실기 검증). 서브에서 복귀한 직후의 N 없는 블록에서는 **서브의 마지막 N** 이, 서브 진입 직후의 N 없는 블록에서는 **메인의 N** 이 그대로 보입니다. 즉 Fanuc 에서는 이 값만으로 어느 파일의 N 인지 판단할 수 없습니다. `/machine/channel/programName` 을 함께 읽으십시오. Siemens 는 값을 유지하지 않으므로 이 상황이 생기지 않습니다.

## /machine/channel/programBlockCounter
실행 블록 카운터입니다. **의미가 기종별로 다릅니다**: Fanuc 은 **사이클 시작부터** 실행한 블록 수 (`cnc_rdblkcount`: Cycle Start 마다 리셋되고 서브프로그램 블록도 이어서 셉니다, 실기 검증), Siemens 는 현재 프로그램 내 실행 중인 **행 번호**(`actLineNumber`, 음수는 0 으로 클램프). 반환 `int`.

Siemens 의 행 번호는 **지금 실행 중인 파일 기준**입니다. 서브프로그램에 들어가면 서브 파일의 행 번호로 바뀌고, 메인으로 복귀하면 메인 파일의 행 번호로 돌아옵니다 (실기 검증). 그래서 이 숫자만으로는 어느 파일의 몇 행인지 알 수 없습니다. 메인의 3행과 서브의 3행이 같은 `3` 입니다. 파일까지 특정하려면 `/machine/channel/programName` · `/machine/channel/programNestLevel` 을 함께 읽으십시오. 중첩 전환 순간(1초 미만)에는 네 값의 조합이 잠시 어긋난 샘플이 나올 수 있습니다 (레벨이 이름보다 먼저 갱신됨).

## /machine/channel/programName
**현재 실행 중인** 프로그램의 이름(파일명)입니다. 서브프로그램에 들어가면 그 서브의 이름으로 바뀝니다. HMI 에서 선택된 메인은 `mainProgramName` 참조.

## /machine/channel/programPath
현재 실행 중인 프로그램의 전체 경로입니다 (예: `//CNC_MEM/USER/PATH1/O0001`). Siemens 는 NCK 내부 경로를 사용자 표기(`//NC/...`)로 변환해 돌려줍니다.

## /machine/channel/mainProgramName
HMI 에서 **선택된 메인 프로그램**의 이름입니다. 실행 중 서브프로그램에 들어가도 변하지 않습니다 (그게 `programName` 과의 차이).

## /machine/channel/mainProgramPath
HMI 에서 **선택된 메인 프로그램**의 전체 경로입니다 (`mainProgramName` 의 경로 버전). 실행 중 서브프로그램에 들어가도 변하지 않습니다.

**쓰기 = 프로그램 선택**: 그 경로의 프로그램을 해당 채널의 실행 대상(메인 프로그램)으로 고릅니다. 값은 경로 문자열입니다: `{"value": "//CNC_MEM/USER/O0001"}`.

- 경로 표기는 기종을 따릅니다. Fanuc `//CNC_MEM/USER/O0001`(데이터 서버는 `//DATA_SV/...`), Siemens `//NC/Part programs/PART1.MPF` (`programPath`·`entryList` 가 돌려주는 표기 그대로. `Subprograms`·`Workpieces` 도 동일). NC 파일시스템 경로는 벤더 고유라 `plcAddress` 와 같은 이유로 통일하지 않습니다
- **파일이어야 합니다.** 폴더 경로를 주면 `-18`. 없는 경로도 `-18`
- 선택만 할 뿐 **가공을 시작하지는 않습니다** (사이클 스타트는 조작반/PLC 몫)
- Siemens 는 서버의 파일 핸들링 `Select` 메서드를, Fanuc 은 CNC 메모리면 `cnc_pdf_slctmain`, 데이터 서버 경로면 `cnc_wrdsdncfile` 을 씁니다

## /machine/channel/programCurrentBlock
지금 실행 중인 블록의 **G코드 텍스트**입니다. 실행 중이 아니면 **빈 문자열**입니다 (두 기종 공통, `null` 이 아닙니다).

## /machine/channel/programNextBlock
다음에 실행될 블록의 G코드 텍스트입니다. 다음 블록이 없으면(마지막 블록·정지 중) 빈 문자열입니다.

## /machine/channel/programLastBlock
직전에 실행된 블록의 G코드 텍스트입니다. 직전 블록이 없으면 빈 문자열입니다. **Siemens 전용** (Fanuc 은 미지원).

## /machine/channel/programLookAhead
**현재 실행 지점 주변의 프로그램 텍스트**입니다. 현재 블록과 그 앞쪽(아직 실행하지 않은 부분)을 담은 여러 줄 문자열. 반환 `string`.

분량은 기종마다 다릅니다. Fanuc 은 선독 버퍼 전체(`cnc_rdexecprog`), Siemens 는 실행 지점 주변의 조각(`actPartProgram`)입니다. **전체 프로그램은 이 주소로 얻을 수 없습니다.** 그건 NC 파일 시스템에서 해당 프로그램 파일을 읽어야 합니다.

줄바꿈은 기종에 상관없이 LF(`
`) 하나로 정규화됩니다. CR 은 제거되므로 `
` 으로 split 하면 됩니다.

## /machine/channel/programNestLevel
프로그램 호출 중첩 단계입니다 (`desc` 동봉): `0` = 프로그램 없음, `1` = 메인 실행 중, `2`~ = 서브프로그램 (L1, L2, …). **Siemens 전용**.

## /machine/channel/auxModal/auxModalValue
보조 기능 모달 값입니다. `auxModal` 필터에 **레터**를 지정합니다 (예: `auxModal=M`, `S`, `T`, `D`, `H`, `F`). 반환 `float`. 예: `auxModal=T` → 지령된 공구번호, `auxModal=S` → 지령 회전수.

## /machine/channel/singleBlockOn
싱글블록 스위치 상태입니다 (`true` = 켜짐). Fanuc 은 F4 신호 비트, Siemens 는 `singleBlockActive`.

## /machine/channel/dryRunOn
드라이런 스위치 상태입니다 (`true` = 켜짐).

## /machine/channel/optionalStopOn
옵셔널 스톱(M01 유효) 스위치 상태입니다 (`true` = 켜짐).

## /machine/channel/blockSkipOn
블록 스킵(`/`) 스위치 상태입니다 (`true` = 켜짐). Siemens 는 스킵 레벨이 0~9 로 여러 개인데 이 주소는 **레벨 0** 을 봅니다 (Fanuc 의 `/` 과 동일 의미).

## /machine/channel/machineLockOn
머신 록(축 이동 잠금) 상태입니다 (`true` = 켜짐). Siemens 는 프로그램 테스트 (`progTestActive`) 상태.

## /machine/channel/variable/variableValue
**매크로 변수(Fanuc) / R 파라미터(Siemens)** 를 읽고/씁니다 (read + write). `variable` 필터에 변수 번호 (예: `variable=100` → Fanuc `#100`, Siemens `R100`). 반환 `float`, 쓰기는 `{"value": 3.14}`. **읽기는** 범위/콤마 확장을 지원합니다 — `variable=100-105` 는 6개 값 배열. 쓰기는 항상 단일 변수입니다 (확장 문법은 `-13` 으로 거절: 모든 쓰기 공통 규칙). Fanuc 의 **미설정(vacant) 매크로 변수는 `null`** 입니다 — 조작반 커스텀 매크로 화면에 `DATA EMPTY` 로 뜨는 그 상태이며 값 `0` 과 구분됩니다. 범위 확장에서도 그 자리만 `null` 이 됩니다 (예: `[3.14, null]`).

## /machine/channel/toolOffset/toolLengthGeometry
**M계(머시닝센터) 공구 길이 형상값**입니다 (오프셋 화면의 H 열). 공구를 측정해 넣는 기준값으로, 길이 보정(H 코드)의 바탕이 됩니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolLengthWear
**M계 공구 길이 마모값**입니다 (오프셋 화면의 H 열). 가공 중 쌓이는 미세 보정분으로, 형상값은 그대로 두고 이쪽만 조정하는 것이 일반적입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolRadiusGeometry
**M계 공구경 형상값**입니다 (오프셋 화면의 D 열). 공구경 보정(G41/G42)이 참조하는 반경 기준값입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 **반지름**이지만, 오프셋 화면은 설정에 따라 **지름**으로 표시·입력하도록 되어 있을 수 있습니다. 디메시는 장비가 저장한 값을 그대로 내보내며 임의로 환산하지 않습니다.

## /machine/channel/toolOffset/toolRadiusWear
**M계 공구경 마모값**입니다 (오프셋 화면의 D 열). 공구 마모에 따른 반경 감소를 반영합니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 **반지름**이지만, 오프셋 화면은 설정에 따라 **지름**으로 표시·입력하도록 되어 있을 수 있습니다. 디메시는 장비가 저장한 값을 그대로 내보내며 임의로 환산하지 않습니다.

## /machine/channel/toolOffset/toolXGeometry
**T계(선반) X 방향 공구 치수 형상값**입니다. 여기서 X 는 축 이름이 아니라 오프셋 화면의 고정 열, 즉 **공구 치수의 방향 성분**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolXWear
**T계 X 방향 공구 치수 마모값**입니다. 가공 중 누적되는 X 방향 보정분입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolZGeometry
**T계 Z 방향 공구 치수 형상값**입니다. X 와 마찬가지로 축이 아니라 화면의 고정 열입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolZWear
**T계 Z 방향 공구 치수 마모값**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolYGeometry
**T계 Y 방향 공구 치수 형상값**입니다. **Y축 옵션 장비 전용**이라 옵션이 없는 선반에서는 `-20` 이 반환됩니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolYWear
**T계 Y 방향 공구 치수 마모값**입니다 (Y축 옵션 장비 전용).

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolNoseRadiusGeometry
**T계 노즈 반경 형상값**입니다. 노즈 반경 보정(G41/G42)이 참조하며, 팁 방향(`toolTipDirection`)과 함께 날끝 궤적을 결정합니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolNoseRadiusWear
**T계 노즈 반경 마모값**입니다.

반환 `float` (실거리), 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `channel` + `toolOffset` 필터가 필요합니다. **Fanuc 전용**이며, 기종의 오프셋 화면에 없는 열이면 `-20` 이 반환됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/toolOffset/toolTipDirection
선반 공구의 **가상 날끝 위치 코드**입니다 (0~9, read + write). 노즈 반경 보정(G41/G42) 때 날끝이 노즈 중심 기준 어느 방위에 있는지 판정하는 코드입니다. 각도가 아니라 위치 코드이며, 배율 없는 정수 그대로 반환/입력합니다 (`{"value": 3}`).

- `1`~`8` = 방위, **`0`/`9` = 노즈 중심이 기준점** (가상 날끝이 아니라). 두 값은 같은 의미입니다. *"Imaginary tool nose numbers 0 and 9 are used when the tool nose center coincides with the start point"* (FANUC 0i-F 선반 매뉴얼 `B-64604EN-1/01` §5.2.2)
- **`desc` 는 `0`·`9` 에만 붙습니다.** `1`~`8` 은 매뉴얼이 **도해로만** 정의하고 그 도해가 평면(`G17`/`G18`/`G19`)별로 여러 벌이라, 같은 번호가 구성에 따라 다른 방위를 가리킵니다. 방위 해석은 그 기종 매뉴얼의 도해를 따르세요
- Siemens 대응 개념: cutting edge position (`toolArea/tool/toolEdge/toolTipDirection`). **두 트리는 같은 번호 체계와 같은 `desc` 어휘를 씁니다.** 주소만 다를 뿐 값은 그대로 비교·재사용할 수 있습니다

## /machine/channel/toolOffsetCount
공구 보정 레지스터의 **사용 가능 개수**입니다 (read 전용, `int`). 오프셋 번호는 `1`~이 값까지입니다. UI 가 테이블을 순회할 때 상한으로 쓰세요.

## /machine/channel/workOffset/axis/workOffsetValue
**워크좌표계 오프셋**: G54 등 워크좌표계의 축별 오프셋 거리입니다 (read + write). 반환 `float`, 실거리 (장비 설정 단위 mm/inch 그대로. Fanuc 내부 정수 표현은 SDK 가 소수점 배율 정규화).

`workOffset` 필터는 **현장 G코드 표기를 직접 입력**합니다 (`plcAddress` 처럼 열린 이름공간: 별도 번호 체계 없음). 대소문자 무시, 공백·별칭 불허:

- **Fanuc**: `EXT`(공통 오프셋: 전 좌표계 가산, 조작반 EXT 행), `G54`~`G59`, 확장 `G54.1P1`~`G54.1P300` (옵션에 없는 P번호는 벤더 에러)
- **Siemens**: `G500`, `G54`~`G57`, `G505`~`G599`. 실제로 몇 개까지 있는지는 장비 설정에 달려 있어, 없는 지정자는 `-18` 과 함께 **그 장비에서 허용되는 목록**을 알려줍니다

⚠️ **`G500` 은 Fanuc `EXT` 와 다릅니다.** `EXT` 는 어느 `G5x` 가 활성이든 **그 위에 더해지는** 공통 오프셋이지만, `G500` 은 `G54`~`G57` 과 **같은 모달 그룹의 배타적 멤버**라 그것들과 동시에 활성일 수 없습니다. `G500` 이 걸린 상태는 설정 오프셋이 꺼진 상태이고 그 자리의 값은 보통 `0` 입니다. 지금 어느 것이 활성인지는 `/machine/channel/gGroup/gModal?gGroup=7` 로 확인하십시오. Siemens 에서 `EXT` 처럼 전 좌표계에 가산되는 몫은 이 주소가 아니라 **별도의 프레임**(조작반의 `Basic reference`·`Total basic WO` 행)에 있고, 디메시는 그것들을 개별 주소로 내지 않습니다. 합쳐진 결과는 `/machine/channel/axis/totalWorkOffsetValue` 가 답합니다.

**Siemens 는 굵은 값과 미세 조정(`Fine`)의 합을 반환합니다.** 장비가 그 합을 적용하고 조작반도 한 오프셋의 두 칸으로 보여주므로, 이 주소는 **실제 적용되는 값**을 냅니다. 조작반의 `Coarse` 칸만 보고 비교하면 다르게 보일 수 있습니다. 미세 조정만 따로 보려면 `/machine/channel/workOffset/axis/workOffsetFineValue` 를 쓰십시오. Fanuc 에는 미세 조정 개념이 없어 값이 하나이고, 그래서 두 기종에서 이 주소의 뜻이 같습니다.

⚠️ **이 값은 저장된 평행이동입니다.** 두 가지가 더 있습니다. ① 워크좌표계에는 **회전·배율·미러**가 걸릴 수 있어(`workOffsetRotation`·`workOffsetScale`·`workOffsetMirrorOn`) 걸려 있으면 좌표 변환이 이 값만으로 결정되지 않습니다. ② 실제로 걸리는 총량에는 기준 오프셋 등이 더해져 이 값과 다를 수 있습니다 (`totalWorkOffsetValue`). 부품 좌표가 필요하면 계산하지 말고 `/machine/channel/axis/workPosition` 을 읽으십시오. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 라 이 값이 곧 변환입니다.

`axis` 는 축 번호(1~) 또는 축 이름. `axis=1-3`·`workOffset=G54,G55` 확장 지원합니다. Fanuc 은 같은 workOffset 의 축 확장이 FOCAS 호출 1회로 묶입니다.

쓰기는 `{"value": 25.4}` (단일 축). **현재 Fanuc 전용**: Siemens 는 이 값에 직접 쓰면 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비측 활성화 절차가 따로 필요하고 이 프로토콜이 그 절차를 노출하지 않아 `-20` 으로 거절합니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/workOffset/axis/workOffsetFineValue
워크좌표계 오프셋의 **미세 조정(`Fine`) 부분**입니다. 필터와 타입은 `/machine/channel/workOffset/axis/workOffsetValue` 와 같습니다. **읽기 전용**.

미세 조정은 **기본 오프셋을 건드리지 않고 얹는 작은 보정**입니다. 처음 워크좌표계를 잡을 때 측정한 값이 굵은 값(`Coarse`)으로 들어가고, 이후 첫 가공품을 재보니 `0.02mm` 어긋났다면 그 `0.02` 를 미세 조정에 넣습니다. 원래 셋업 값이 그대로 남아 추적되고, 미세 조정을 `0` 으로 되돌리면 셋업 상태로 복귀합니다.

**적용되는 오프셋은 굵은 값 + 이 값**이고, 그 합은 `workOffsetValue` 가 답합니다. 굵은 값만 필요하면 `workOffsetValue` 에서 이 값을 빼십시오. 굵은 값을 위한 별도 주소는 두지 않았습니다.

미세 조정을 쓰지 않는(또는 장비 설정으로 끈) 장비에서는 `0` 입니다.

**Siemens 전용**입니다. Fanuc 의 워크좌표계 오프셋은 값이 하나이고 미세 조정이라는 개념이 없습니다.
## /machine/channel/workOffset/axis/workOffsetRotation
워크좌표계의 **축별 회전각**입니다. 필터는 `/machine/channel/workOffset/axis/workOffsetValue` 와 같습니다. 반환 `float`, `unit` 은 `deg`(도). **읽기 전용**.

`0` 이면 그 축에 회전이 걸려 있지 않습니다.

**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비측 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하십시오. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 `-20` 입니다.

**Siemens 전용**입니다.

## /machine/channel/workOffset/axis/workOffsetScale
워크좌표계의 **축별 배율**입니다. 필터는 위와 같습니다. 반환 `float`, 무차원이라 `unit` 이 없습니다. **읽기 전용**.

`1` 이면 배율이 없습니다 (실제 크기). `2` 면 그 축 방향으로 두 배로 가공합니다.

**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비측 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하십시오. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 `-20` 입니다.

**Siemens 전용**입니다.

## /machine/channel/workOffset/axis/workOffsetMirrorOn
워크좌표계의 **축별 미러(대칭) 여부**입니다. 필터는 위와 같습니다. 반환 `boolean`. **읽기 전용**.

`true` 면 그 축 방향이 반전됩니다. 조작반의 워크오프셋 상세 화면에서 축별 체크박스로 보이는 값입니다.
**이 셋은 평행이동과 같은 좌표계의 성분입니다.** 실제 좌표 변환은 평행이동 + 회전 + 배율 + 미러이므로, 부품 좌표를 계산하려면 다섯을 함께 읽어야 합니다. 평행이동만 쓰는 통상적인 장비에서는 회전 `0`, 배율 `1`, 미러 `false` 이므로 `workOffsetValue` 만으로 충분합니다.

**쓰기는 지원하지 않습니다.** 이 값들에 직접 쓰면 장비가 **요청을 받아들이고도 값을 바꾸지 않습니다**(실측). 장비측 활성화 절차가 따로 필요한데 이 프로토콜이 그 절차를 노출하지 않습니다. 변경은 조작반에서 하십시오. 평행이동(`workOffsetValue`·`workOffsetFineValue`)도 같은 이유로 Siemens 에서는 쓰기가 `-20` 입니다.

**Siemens 전용**입니다.

## /machine/channel/axis/totalWorkOffsetValue
지금 **실제로 걸려 있는 영점이동 총량**입니다 (축별 평행이동). 반환 `float`. **읽기 전용**. 조작반의 `Total WO` 행에 해당합니다.

`workOffsetValue` 가 **표에 저장된 값**이라면 이 주소는 **지금 적용되고 있는 값**입니다. 둘이 다르면 그 차이가 어디서 왔는지가 곧 문제의 단서입니다:

```
workOffsetValue?workOffset=G54     80.400    설정한 값
totalWorkOffsetValue              100.400    실제로 걸린 값
                                   ↑ 20.000 이 어디선가 더해졌다
```

총량에는 저장된 오프셋 말고도 **기준 오프셋(`Basic reference`)·기본 프레임·프로그램의 `TRANS`·사이클이 설정한 프레임**이 함께 들어갑니다. "설정은 그대로인데 부품이 어긋난다" 를 진단할 때 두 주소를 비교하십시오. 저장된 값만 읽으면 더해진 몫이 보이지 않습니다.

⚠️ **이 값으로 부품 좌표를 계산하지 마십시오.** 평행이동 총량일 뿐이고 **회전·배율·미러와 공구 길이 보정은 여기에 없습니다.** 워크 좌표가 필요하면 `/machine/channel/axis/workPosition` 을 읽으십시오. 장비가 그 모두를 적용한 결과입니다.

`workOffset` 필터를 받지 않습니다. "지금 걸린 것" 이라 지정자를 고를 대상이 없습니다. `axis=1-3` 확장을 지원하고, 쓰기는 지원하지 않습니다 (합산 결과라 되돌려 쓸 대상이 아닙니다).

**Siemens 전용**입니다. Fanuc 에서는 `workOffset=EXT` 로 공통 오프셋을 따로 읽어 활성 좌표계 값과 더하십시오. 다만 그 합에는 프로그램이 건 시프트가 빠지므로 이 주소와 뜻이 같지 않습니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/gGroup/gModal
활성 G모달을 **기종 무관 표준 그룹 번호**로 조회합니다 (`plcType` 처럼 디메시가 정한 벤더 중립 번호: 벤더 원시 그룹 번호가 아닙니다). `gGroup` 필터 값:

- `1` = motion: 이송 모드 (G00 급속 / G01 직선 / G02·G03 원호 …)
- `2` = plane: 가공 평면 (G17 XY / G18 ZX / G19 YZ)
- `3` = distanceMode: 절대/증분 (G90/G91) · **Fanuc T 계열(System A) 미지원** (U/W 어드레스 방식)
- `4` = units: 인치/미터 (G20·G70 / G21·G71)
- `5` = feedMode: 이송 지정 (분당/회전당/역시간)
- `6` = cutterComp: 공구경 보정 (G40 해제 / G41 좌 / G42 우)
- `7` = coordinateSystem: 워크좌표계 (G54~G59)
- `8` = spindleSpeedMode: 주속 일정(G96) / 회전수 일정(G97)

값은 그 기종의 G코드 문자열 + 핵심 조합엔 `desc` 로 기종 무관 의미 (예: Fanuc `{"value":"G21","desc":"metric"}`, Siemens `{"value":"G710","desc":"metric"}`). 벤더 원시 그룹 접근은 `gModalList` 를 쓰세요. **Siemens 참고**: `feedMode` 와 `spindleSpeedMode` 는 Sinumerik 에선 같은 그룹이라 값이 같고 `desc` 로 의미가 갈립니다 (G96 계열 = 주속 일정).

## /machine/channel/gModalList
장비가 보고하는 **전체 모달 G코드 목록을 벤더 순서 그대로** 반환합니다 (`stringArray`). 장비 특화 HMI 의 모달 화면 재현용입니다. 그룹 인덱스의 의미는 각 벤더 매뉴얼 기준입니다.

- **Fanuc**: FOCAS modal 그룹 0~20 순서. 그 기종에 정의 안 된 그룹은 `null`
- **Siemens**: `ncFkt` G-펑션 그룹 1~N 순서 (N = 장비의 그룹 수)

기종 무관 로직에는 이 목록 대신 `gModal` 의 심볼릭 키를 쓰세요.

## /machine/channel/axis/machinePosition
축의 기계 좌표(machine coordinate)입니다. `axis` 필터로 축을 지정하며, 범위(`axis=1-3`)나 복수 지정(`axis=1,2`)이 가능합니다. 반환 타입은 `float` (64비트 배정밀도).

위치 계열 4종(machinePosition/workPosition/distanceToGo/relativePosition)은 모두 **실거리**: 장비 설정 단위(mm/inch) 그대로이며 조작반 표시와 일치합니다 (Fanuc 의 내부 정수 표현은 SDK 가 축별 소수점 배율로 정규화). 네 값 모두 축이 원점을 확립한 뒤에만 유효합니다. 전원 투입 직후라면 `/machine/channel/axis/axisReferencedOn` 을 먼저 확인하십시오 (미확립 상태에선 그럴듯한 숫자가 조용히 나옵니다).

⚠️ **이 값은 공구 기준점(주축단)의 좌표입니다.** `workPosition` 은 공구 선단이라 두 값의 차이에 **공구 길이 보정**이 들어갑니다. `machinePosition − 영점이동 = workPosition` 은 성립하지 않습니다 (활성 워크좌표계의 회전·배율·미러도 함께 걸립니다). 워크 좌표가 필요하면 직접 계산하지 말고 `workPosition` 을 읽으십시오.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/workPosition
축의 공작물 좌표(절대 좌표)입니다. 반환 `float`.

이 값은 **공구 선단** 기준이며 활성 영점이동·회전·배율·미러·공구 길이 보정이 **모두 적용된 결과**입니다. 장비가 계산한 최종 좌표라 `machinePosition` 에서 직접 구할 필요가 없습니다 (뺄셈으로는 맞지 않습니다).

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/distanceToGo
현재 블록에서 축의 **잔여 이동량**입니다. 반환 `float`.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/relativePosition
축의 상대 좌표입니다. 반환 `float`.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/axisName
축 이름입니다 (예: `"X"`, `"Z1"`). `axis` 번호 ↔ 실제 축 대응을 확인할 때 사용.

## /machine/channel/axis/axisLoad
축(서보) 부하율입니다. 반환 `float` + `unit:"%"`.

## /machine/channel/axis/axisFeedActual
그 축 방향의 실제 이송 **성분**입니다 (**Siemens 전용**). 반환 `float`. Fanuc(FOCAS2)은 축별 이송을 제공하지 않아 미지원(`-20`)입니다.

공구가 경로를 따라가는 속도 자체가 아니라, 그 속도를 축 방향으로 분해한 몫입니다. 그래서 **같은 지령에서도 이동 방향에 따라 계속 변합니다.** XY 평면에서 `F1000` 을 지령했을 때 X 축만 움직이는 구간에서는 이 값이 1000 이지만, 45° 대각선 구간에서는 약 707 입니다.

축 성분들을 **더해도 경로 속도가 되지 않습니다** (벡터 크기라 707+707 이 아니라 √(707²+707²)=1000). 이 값은 해당 축이 자기 속도 한계에 걸려 경로를 제한하고 있는지 보는 용도입니다.

단위는 기계 설정을 따릅니다 (mm/min 또는 inch/min). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

## /machine/channel/axis/axisCurrent
축 모터 전류입니다. 반환 `float` + `unit:"Ampere"` (양 기종 동일). Fanuc 은 진단값, Siemens 는 드라이브 파라미터 `R0078`.

**Siemens**: 값이 드라이브에서 오므로, 그 축에 드라이브가 배정되지 않은 채널에서는 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

## /machine/channel/axis/axisTemperature
축 모터 온도입니다. 반환 `float` + `unit:"°C"`. Fanuc 은 진단 308번.

**Siemens**: 값이 드라이브에서 오므로, 그 축에 드라이브가 배정되지 않은 채널에서는 `-20` 입니다 (장비 구성이지 결함이 아닙니다).

## /machine/channel/axis/axisInterlockOn
축 인터록 상태입니다 (`true` = 인터록 걸림). **Fanuc 전용**.

## /machine/channel/axis/axisReferencedOn
축이 **기계 원점(레퍼런스)을 확립했는지**입니다. 반환 `boolean`. **읽기 전용**.

`false` 인 축은 좌표계가 아직 서 있지 않은 상태입니다. **위치 주소들(`machinePosition`·`workPosition`·`relativePosition`·`distanceToGo`)이 그럴듯한 숫자를 주더라도 무의미할 수 있습니다.** 값이 없다는 에러가 아니라 틀린 값이 조용히 나오는 경우이므로, 전원 투입 직후의 위치를 소비하는 쪽은 이 값을 먼저 확인하십시오. 증분형 엔코더 장비는 원점복귀를 마쳐야 좌표가 성립합니다.

한 번 확립되면 축이 어디로 움직여도 `true` 로 유지됩니다. "지금 원점 위치에 있는가" 라는 순간 상태가 아니라 **좌표계 유효성**입니다.

Fanuc 은 CNC→PMC 표준 신호 ZRF(`F120` 의 축별 비트)를, Siemens 는 `refPtStatus` 를 읽습니다 (둘 다 실기 검증). Fanuc 은 축 16개까지 지원하며, 멀티패스 장비의 2계통 이후는 지원하지 않습니다.

## /machine/channel/axis/axisEnergyNet
축의 순소비 **전력량**(누적 소비 − 누적 회생)입니다. **Fanuc 전용** (진단 4920), `float` + `unit:"Wh"`.

전력량(Wh)은 전력(W)과 다릅니다. W 는 순간값, Wh 는 누적량입니다. 이 값은 주행거리계처럼 계속 쌓이므로, 특정 구간의 사용량은 **앞뒤로 두 번 읽어 빼십시오**. 평균 전력(W)이 필요하면 `ΔWh ÷ Δ시간(h)` 로 구합니다.

## /machine/channel/axis/axisEnergyConsumed
축의 누적 소비 **전력량**입니다. **Fanuc 전용** (진단 4921), `unit:"Wh"`. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

## /machine/channel/axis/axisEnergyRegenerated
축의 누적 회생 **전력량**입니다 (감속 시 돌려받은 몫). **Fanuc 전용** (진단 4922), `unit:"Wh"`.

## /machine/channel/spindle/spindleLoad
스핀들 부하율입니다. 반환 `float` + `unit`. Siemens 는 항상 `unit:"%"` 입니다. Fanuc 은 벤더 응답이 단위를 함께 실어 주어 장비 설정에 따라 `%` 또는 `rpm` 이 오고, 벤더가 그 밖의 단위 코드를 주는 드문 경우에는 `unit` 키가 생략됩니다. 소비자는 `%` 를 가정하지 말고 `unit` 을 보세요.

## /machine/channel/spindle/spindleOverride
스핀들 오버라이드 (%)입니다. 반환 `int` + `unit:"%"`. **Fanuc 은 채널 공통값** (G30 신호: `spindle` 필터 무시, 모든 스핀들에 같은 값), Siemens 는 스핀들별 값.

## /machine/channel/spindle/spindleCurrent
스핀들 모터 전류입니다. **Siemens 전용** (드라이브 파라미터 `R0078`). 반환 `float` + `unit:"Ampere"`.

## /machine/channel/spindle/spindleTemperature
스핀들 모터 온도입니다. 반환 `float` + `unit:"°C"`. Fanuc 은 진단 403번, Siemens 는 드라이브 파라미터 R0035.

## /machine/channel/spindle/speedCommanded
스핀들 **S 지령값**입니다. 반환 `float`. **`unit` 을 붙이지 않습니다** — 지령의 뜻이 스핀들 속도 모드에 따라 갈리기 때문입니다 (회전수 일정이면 회전수, 주속 일정이면 주속). 양 기종 모두 같습니다. **Fanuc 은 채널 모달 S 값** (`spindle` 필터 무시: S 지령이 채널 단위 개념), Siemens 는 스핀들별 `cmdSpeed`.

## /machine/channel/spindle/speedActual
스핀들별 실제 회전수입니다. 반환 `float` + `unit:"rpm"` (양 기종 동일). Fanuc 은 `cnc_acts2`, Siemens 는 스핀들별 `actSpeed`. 양 기종 모두 `spindle` 필터로 대상 스핀들을 지정합니다.

## /machine/channel/activeToolNumber
그 채널에서 지금 **보정이 적용되고 있는** 공구의 번호(`T`)입니다. `channel` 필터. 반환 `int`.

채널의 상태값입니다. 스핀들에 **물려 있는** 공구(물리적 위치)와도, `T` 로 **지정만 되고 아직 교환되지 않은** 공구와도 다른 개념입니다.

**Fanuc 검증 유보**: Fanuc 은 `T` 모달을 읽는데, 이 값이 교환 완료(`M06`) 시점에 바뀌는지 `T` 지정 시점에 이미 바뀌는지는 실기 검증 대기 중입니다 (후자라면 "지정만 된 공구" 와 구분되지 않는 순간이 생깁니다). 교환 타이밍이 중요한 용도라면 검증 완료 전까지 이 값을 교환 신호로 쓰지 마세요. Siemens 는 교환 완료 기준(`actTNumber`)으로 확정입니다.

이 번호를 공구 트리의 `tool` 필터에 넣으면 그 공구의 이름·보정 세트 수·오프셋을 조회할 수 있습니다. 함께 필요한 `toolArea` 값은 `/machine/channel/toolAreaNumber` 가 알려줍니다 (접속 시 캐싱되어 추가 통신이 없습니다).

Fanuc 은 `T` 모달, Siemens 는 `actTNumber` 를 읽습니다.

## /machine/channel/activeToolEdgeNumber
활성 공구에서 지금 **보정이 적용되고 있는 보정 세트**의 번호(`D`)입니다. `channel` 필터. 반환 `int`. Siemens 는 `actDNumber`, **Fanuc 은 고정 `1`** 입니다 — 표준 오프셋 모델엔 공구에 딸린 날 계층이 없어 적용 중인 보정 세트가 언제나 하나뿐입니다 (공구 관리 기능 쪽은 현재 미지원).

## /machine/channel/spindle/spindleEnergyNet
스핀들의 순소비 **전력량**(누적 소비 − 누적 회생)입니다. **Fanuc 전용** (진단 4930), `unit:"Wh"`. 누적값이라 구간 사용량은 두 번 읽어 뺍니다.

## /machine/channel/spindle/spindleEnergyConsumed
스핀들의 누적 소비 **전력량**입니다. **Fanuc 전용** (진단 4931), `unit:"Wh"`.

## /machine/channel/spindle/spindleEnergyRegenerated
스핀들의 누적 회생 **전력량**입니다. **Fanuc 전용** (진단 4932), `unit:"Wh"`.

## /machine/userData/userDataValue
Siemens **전역 GUD(SGUD)** 사용자 변수를 읽고/씁니다 (현재 **OPC-UA(Siemens) 전용**, NC 전체 공유 변수). 읽기(GET)와 쓰기(POST) 모두 지원합니다. GUD 는 변수마다 타입이 달라, 반환 타입은 `object` 입니다. 값이 무슨 타입인지 함께 알려주는 **자기 기술(self-describing) 엔벨로프** `{"type":..,"data":..}` 로 옵니다. 필터는 `userData` 하나입니다. 이 주소는 **NC 전체 공유** 변수 전용이라 채널을 지정하지 않습니다.

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
Siemens **채널 GUD(채널별 SGUD)** 사용자 변수를 읽고/씁니다 (현재 **OPC-UA(Siemens) 전용**). `channel` 과 `userData` 두 필터가 필요하며, `channel` 이 가리키는 채널의 변수만 다룹니다. NC 전체가 공유하는 전역 변수는 이 주소로 접근하지 않습니다.

반환 타입은 `object` 입니다. GUD 는 변수마다 타입이 달라 값이 무슨 타입인지 함께 알려주는 **자기 기술 엔벨로프** `{"type":..,"data":..}` 로 옵니다.

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
그 공구 영역에 **등록된 공구의 수**입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, 읽기 전용. 등록된 공구가 없으면 `0` 입니다.

목록(`toolList`)이 돌려주는 항목 수와 같고 **장비의 같은 값**을 봅니다. 개수만 필요할 때 목록 전체를 받지 않아도 되도록 따로 둔 주소입니다. 공구 17개짜리 장비에서 실측하면 목록보다 훨씬 쌉니다 (81ms 대 684ms).

없는 공구 영역을 지정하면 `-18` 로 거절됩니다.

**Siemens 전용**입니다. Fanuc 은 공구 관리 기능이 없으면 "공구" 라는 객체 자체가 없습니다. 보정 레지스터의 개수는 공구 수와 다른 값이라 대신 쓸 수 없습니다.

## /machine/toolArea/toolList
그 공구 영역에 **등록된 공구 전부**의 목록입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `objectArray`, 등록된 공구가 없으면 빈 배열 `[]`.

항목: `{"toolNumber": 16, "toolName": "BALLNOSE_D8", "toolEdgeCount": 4, "sisterToolNumber": 9, "magazineNumber": 0, "pocketNumber": 0, "toolLocationType": "buffer"}`

공구 번호는 **듬성듬성합니다.** 공구 17개가 2번~18번을 쓰고 1번은 없는 식이라, 번호를 1부터 넣어보는 것으로는 무엇이 있는지 알 수 없습니다. 이 목록이 그 답이며, 항목의 `toolNumber` 를 그대로 `tool` 필터에 넣어 공구별 주소를 조회하는 것이 용법입니다.

**순서는 `toolNumber` 오름차순입니다.** 매번 같은 순서가 보장됩니다. **장비 화면의 순서와는 다릅니다**: 공구 목록 화면은 보통 이름순으로 정렬돼 있고 작업자가 정렬 기준을 바꿀 수 있어, 맞출 수 있는 하나의 "화면 순서" 라는 것이 없습니다. 화면과 같은 순서로 보여주려면 `toolName` 으로 정렬하십시오.

**`toolNumber` 는 장비 화면의 `Loc.`(자리 번호)이 아닙니다.** 공구 관리를 쓰는 장비에서는 공구를 이름과 자매번호로 식별하므로 이 번호가 목록 화면에 나오지 않습니다 (공구 상세 화면의 `Tool number` 항목이 이 값입니다). 화면의 `Loc.` 을 `tool` 필터에 넣으면 **다른 공구를 조회하고도 성공으로 보입니다.** 두 번호가 우연히 같은 공구가 많아 알아채기 어렵습니다. 그 값은 `pocketNumber` 이며, 이 목록이 둘을 함께 담고 있어 대응을 확인할 수 있습니다.

- **toolNumber**: 공구 번호. `tool` 필터에 넣는 값
- **toolName**: 공구 이름. 이름을 쓰지 않는 장비에서는 빈 문자열
- **toolEdgeCount**: 보정 세트 개수 (인선 수가 아닙니다). `toolEdge` 필터의 유효 상한
- **sisterToolNumber**: 자매공구 번호 (같은 이름을 가진 공구들 사이의 순번)
- **magazineNumber**: 지금 꽂혀 있는 매거진(공구 저장고) 번호. 매거진 밖이면 `0`
- **pocketNumber**: 그 매거진 안의 포켓 번호. 매거진 밖이면 `0`
- **toolLocationType**: 자리의 종류. `"magazine"`(매거진에 있음) · `"buffer"`(스핀들 또는 교환기) · `"loading"`(반입·반출 위치) · `"none"`(실물 자리 없음)

이 목록은 **무엇이 있고 · 어떻게 부르고 · 어디 있나** 까지 답합니다. 오프셋·마모 같은 측정값은 보정 세트 단위라 담지 않습니다.

값이 없으면 키를 빼지 않고 `null` 입니다 (위치 세 필드는 예외: 같은 이름의 단독 주소와 값이 같습니다). **공구 번호 오름차순**으로 정렬해 돌려주므로 두 번 읽어 비교하는 것이 의미를 갖습니다. 없는 공구 영역을 지정하면 `-18` 로 거절됩니다.

앞의 네 필드는 잘 바뀌지 않지만 **위치 세 필드는 공구가 움직일 때마다 바뀝니다.** 이 목록은 화면을 그릴 때 한 벌 받아오는 용도이며, "지금 스핀들에 무엇이 물렸나" 만 알고 싶다면 목록을 반복해 읽는 대신 `/machine/channel/activeToolNumber` 를 쓰십시오.

**Siemens 전용**입니다. Fanuc 은 오프셋 테이블이 `1` 부터 촘촘히 채워져 있어 열거할 대상이 없습니다.

## /machine/toolArea/tool/toolName
공구의 이름입니다 (Sinumerik `toolIdent`). `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호).

반환 `string`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": "DRILL 10"}`. **Siemens 전용**입니다. 툴 매니지먼트를 쓰는 장비에서는 이름과 자매공구 번호(duplo)의 조합이 공구의 정체이므로 같은 이름을 가진 공구가 여럿 있을 수 있습니다. 이름을 쓰지 않는 장비에서는 빈 문자열이 정상입니다.

없는 공구를 지정하면 `-18` 로 거절됩니다. 이 주소는 공구를 만들지 않습니다. 이름의 길이·문자 제약은 장비가 판단하며 위반 시 에러로 표면화됩니다.

## /machine/toolArea/tool/sisterToolNumber
**자매공구 번호**입니다 (Sinumerik `duploNo`). 같은 이름을 가진 공구들 사이의 순번으로, 앞선 공구의 수명이 다했을 때 어느 것이 대체 투입될지를 정합니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호).

반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 2}`. **Siemens 전용**입니다. 툴 매니지먼트를 쓰는 장비에서는 공구 이름과 이 번호의 조합이 공구의 정체이므로, 이름이 같은 공구가 여럿일 때 이 번호로 구분합니다.

없는 공구를 지정하면 `-18` 로 거절됩니다. 값이 정수가 아니거나 `0`~`65535` 를 벗어나면 `-16` 입니다. 실제 유효 상한은 장비 설정이 정하며, 그보다 좁은 범위를 벗어난 값은 장비가 거절합니다.

## /machine/toolArea/tool/magazineNumber
그 공구가 지금 꽂혀 있는 **매거진(공구 저장고)의 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

매거진에 없으면 `0` 입니다. 스핀들에 물려 가공 중이거나, 교환기가 옮기는 중이거나, 반입·반출 위치에 있거나, 공구 데이터만 등록되고 실물 자리가 없는 경우입니다. 장비는 이런 자리에도 자기 고유 번호(예: `9998`)를 붙이지만 디메시는 그 번호를 내보내지 않고 `0` 으로 접습니다. 그 번호는 장비 설정에 따라 달라져 소비자가 의존할 수 없습니다.

**지금 있는 곳이지 원래 자리가 아닙니다.** 공구가 스핀들에 물리면 이 값이 `0` 으로 바뀌고 매거진으로 돌아가면 다시 번호가 붙습니다. 원래 어느 자리에서 나왔는지는 이 주소가 답하지 않습니다.

**쓰기는 지원하지 않습니다.** 이 값을 바꿔도 공구는 움직이지 않고 장부만 달라져, 다음 공구 교환 때 교환기가 엉뚱한 포켓을 집게 됩니다.

**Siemens 전용**입니다. 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `0` 입니다.

## /machine/toolArea/tool/pocketNumber
그 공구가 꽂혀 있는 **매거진 안의 포켓 번호**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

매거진 번호가 아파트의 동이라면 이 값은 호수입니다. 둘을 함께 읽어야 위치가 정해집니다. 매거진에 없으면 `0` 입니다 (스핀들에서 가공 중이거나, 교환기가 옮기는 중이거나, 반입·반출 위치에 있거나, 실물 자리가 없는 경우). 장비가 그런 자리에 붙이는 고유 번호(예: `9998`)는 내보내지 않습니다.

**터렛(선반)의 스테이션도 포켓으로 나타납니다.** 장비가 터렛 위치를 매거진의 포켓으로 모델링하기 때문입니다. 현장에서 "3번 스테이션" 이라 부르는 자리가 이 주소에서는 포켓 `3` 입니다.

**지금 있는 곳이지 원래 자리가 아닙니다.** 공구가 스핀들에 물리면 이 값이 `0` 으로 바뀌고 매거진으로 돌아가면 다시 포켓 번호가 붙습니다.

**쓰기는 지원하지 않습니다.** 이 값을 바꿔도 공구는 움직이지 않고 장부만 달라져, 다음 공구 교환 때 교환기가 엉뚱한 포켓을 집게 됩니다.

**Siemens 전용**입니다. 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `0` 입니다.

## /machine/toolArea/tool/toolLocationType
그 공구가 **어떤 종류의 자리에 있는지** 나타냅니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `string`, **읽기 전용**. 값이 곧 뜻이라 별도 코드표가 필요 없습니다.

| 값 | 뜻 |
|---|---|
| `"magazine"` | 매거진(공구 저장고)에 꽂혀 있음 |
| `"buffer"` | 스핀들 또는 교환기(가공 중이거나 옮겨지는 중) |
| `"loading"` | 반입·반출 위치 |
| `"none"` | 실물 자리 없음(공구 데이터만 등록되어 있음) |

**기종이 늘어도 이 넷입니다.** 장비는 스핀들·교환기·반입출 위치에도 자기 고유의 매거진 번호(예: `9998`)를 붙이지만 그 번호는 장비 설정에 따라 달라져 소비자가 의존할 수 없습니다. 디메시가 이 넷으로 접어 내보내므로, 어느 기종에 붙었는지 몰라도 값으로 분기할 수 있습니다.

`"buffer"` 는 스핀들과 교환기 그리퍼를 **구분하지 않습니다.** 장비가 둘을 같은 자리로 취급하기 때문입니다. 스핀들에 물린 공구가 무엇인지는 `/machine/channel/activeToolNumber` 가 답하고 그 주소는 모든 기종이 지원하므로, 둘을 겹쳐 보면 구분됩니다.

**쓰기는 지원하지 않습니다.** 자리를 바꾸는 것은 공구 이동의 몫이고, 이 값만 고치면 장부와 실물이 어긋납니다.

**Siemens 전용**입니다. 공구 관리 기능이 없는 장비는 매거진 자체가 없어 항상 `"none"` 입니다.

## /machine/toolArea/tool/toolEdgeCount
공구가 가진 **보정 세트(cutting edge)의 개수**입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). Siemens 는 `numCuttEdges`, **Fanuc 은 고정 `1`** 입니다 — 표준 오프셋 모델은 오프셋 번호 하나가 곧 보정값 한 벌이라, 공구에 딸린 날이라는 계층이 없습니다 (공구 관리 기능을 쓰는 장비는 날 번호를 따로 갖지만 디메시는 현재 그쪽을 읽지 않습니다).

**개수이지 가장 큰 번호가 아닙니다.** 보통은 `1`부터 이어지지만, 조작반에서 중간 날을 지우면 **번호에 구멍이 생기고 뒤 번호는 밀리지 않습니다.** 예를 들어 `1`·`2`·`3` 중 `2`를 지우면 남는 것은 `1`과 `3`이고 이 값은 `2`가 됩니다. 그래서 `toolEdge` 를 `1`~이 값으로 가정하면 안 됩니다.

없는 날은 읽기·쓰기 모두 `-18` 로 거절되므로, 어떤 번호가 실재하는지는 읽어 보면 알 수 있습니다.

**인선 개수와는 다른 값입니다.** "2날 볼엔드밀", "4날 엔드밀" 이라 할 때의 그 날은 물리적 인선 수이고, 이 값은 제어기가 그 공구에 대해 갖고 있는 보정 세트의 수입니다. 인선이 여럿이어도 모두 같은 높이·반경이면 보정 세트는 하나면 됩니다. 실측 예로 4날 커터가 `1`, 2날 볼엔드밀이 `3` 을 답했습니다.

Siemens 에서 없는 공구를 지정하면 `-18` 로 거절됩니다. 보정 세트가 `0` 개라고 답하지 않습니다.

## /machine/toolArea/magazineList
그 공구 영역의 **매거진 목록**입니다. `toolArea` 필터. 반환 `objectArray`, **읽기 전용**.

**매거진 번호는 연속이 아니라 이 목록 없이는 알 수 없습니다.** 실측 예로 `1`·`9998`·`9999` 였습니다. `1`부터 `/machine/toolArea/magazineCount` 까지 세어 올라가면 찾지 못합니다.

각 항목:

| 필드 | 뜻 |
|---|---|
| `magazineNumber` | 매거진 번호. `/machine/toolArea/magazine/pocketCount` 의 `magazine` 필터에 그대로 넣습니다 |
| `pocketCount` | 그 매거진의 포켓 수 |

**실물 공구 저장고만 담습니다.** 장비 내부에선 스핀들·교환기와 반입출 위치도 매거진으로 취급하고 큰 번호(`9998`·`9999` 등)를 붙이지만, 디메시에서 "매거진" 은 공구 저장고 하나만 뜻합니다. 공구가 그런 자리에 있으면 `/machine/toolArea/tool/toolLocationType` 이 `"buffer"` / `"loading"` 으로 답하고 `/machine/toolArea/tool/magazineNumber` 는 `0` 을 줍니다.

**공구와 그대로 맞물립니다.** `/machine/toolArea/tool/magazineNumber` 가 돌려준 값으로 이 목록의 항목을 찾고, 그 번호를 `pocketCount` 에 그대로 넣을 수 있습니다.

**매거진 이름은 담지 않습니다.** 장비가 이름 필드를 갖고 있지만 현장에서 설정하지 않으면 뜻이 없습니다. 실측 장비에서는 40자리 매거진과 버퍼와 반입출 위치가 **모두 같은 문자열**을 돌려줬습니다. 세 항목에 같은 이름이 붙으면 목록이 고장난 것처럼 보이므로 넣지 않았습니다.

**Siemens 전용**입니다.

## /machine/toolArea/magazineCount
그 공구 영역의 **매거진 수**입니다. `toolArea` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `int`, **읽기 전용**.

**실물 공구 저장고만 셉니다.** 장비 내부에선 스핀들·교환기와 반입출 위치도 매거진으로 취급하지만(그렇게 세면 실측 장비가 `3`) 디메시는 그 자리들을 매거진으로 보지 않습니다. 공구가 거기 있으면 `/machine/toolArea/tool/toolLocationType` 이 `"buffer"` / `"loading"` 으로 답합니다.

**개수는 알려주지만 번호는 알려주지 않습니다.** 번호가 연속이 아니어서 이 값으로부터 유효한 매거진 번호를 유추할 수 없습니다. 번호가 필요하면 `/machine/toolArea/magazineList` 를 쓰십시오. 개수만 필요할 때 목록 전체를 받지 않아도 되게 이 주소를 따로 둡니다.

**Siemens 전용**입니다. 공구 관리 기능이 없는 장비는 매거진 자체가 없어 `0` 입니다.

## /machine/toolArea/magazine/pocketCount
그 매거진의 **포켓 수**입니다. `toolArea` + `magazine` 필터 (`magazine` 은 매거진 번호). 반환 `int`, **읽기 전용**.

**매거진 번호는 연속이 아닙니다.** 유효한 번호는 `/machine/toolArea/magazineList` 가 알려줍니다. `1`부터 `/machine/toolArea/magazineCount` 까지 세어 올라가는 방식으로는 찾을 수 없습니다.

없는 번호는 `-18` 로 거절됩니다. **스핀들·교환기·반입출 위치의 번호도 거절됩니다.** 장비는 그 자리들에도 매거진 번호를 붙이지만 디메시는 매거진으로 보지 않습니다. 범위·콤마 확장을 지원합니다.

**Siemens 전용**입니다.

## /machine/toolArea/magazine/pocketList
그 매거진의 **포켓을 전부, 포켓마다 무엇이 들어 있는지**입니다. `toolArea` + `magazine` 필터. 반환 `objectArray`, **읽기 전용**.

각 항목:

| 필드 | 뜻 |
|---|---|
| `pocketNumber` | 포켓 번호. `1`부터 `/machine/toolArea/magazine/pocketCount` 까지 빠짐없이 나옵니다 |
| `toolNumber` | 그 포켓에 든 공구 번호. **`0` 이면 빈 포켓** |

**다른 주소들이 답하지 못하는 방향입니다.** `/machine/toolArea/tool/pocketNumber` 는 "이 공구가 몇 번 포켓에 있나" 를 답하지만, "몇 번 포켓에 뭐가 있나" 와 "빈 포켓이 어디인가" 는 이 목록만 답합니다.

공구 이름·오프셋은 담지 않습니다. `/machine/toolArea/toolList` 가 번호로 그것들을 주므로 번호로 이어 붙이십시오. 포켓마다 이름을 함께 읽으면 40포켓 매거진에서 읽는 값이 두 배가 됩니다.

버퍼(스핀들·교환기)와 반입출 위치의 번호는 `-18` 로 거절됩니다. 디메시는 그 자리들을 매거진으로 보지 않습니다.

**Siemens 전용**입니다.

## /machine/toolArea/magazine/pocket/toolNumber
그 포켓에 든 **공구 번호**입니다. `toolArea` + `magazine` + `pocket` 필터. 반환 `int`, **읽기 전용**.

**`0` 은 빈 포켓**입니다 (공구 번호는 `1`부터). 없는 포켓은 `0` 이 아니라 `-18` 로 거절되므로 둘이 섞이지 않습니다. 유효한 포켓 범위는 `/machine/toolArea/magazine/pocketCount` 가 알려줍니다.

포켓을 하나만 볼 때 쓰고, 매거진 전체를 훑을 때는 `/machine/toolArea/magazine/pocketList` 가 왕복 한 번으로 끝냅니다.

버퍼(스핀들·교환기)와 반입출 위치의 번호는 `-18` 로 거절됩니다.

**쓰기는 지원하지 않습니다.** 포켓의 공구를 고쳐 쓰면 실물은 그대로인 채 장부만 바뀌어, 다음 공구 교환 때 교환기가 엉뚱한 포켓을 집습니다. 공구 이동은 매거진 명령의 몫이고 이 프로토콜은 그 명령을 노출하지 않습니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolExists
그 공구 번호가 **공구표에 등록되어 있는지** 여부입니다. `toolArea` + `tool` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원.

없는 공구를 물어도 에러가 아니라 `false` 입니다. 존재 여부를 묻는 주소이기 때문입니다.

**쓰기가 공구를 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. 이미 그 상태면 아무것도 하지 않고 성공합니다.

만들어지는 공구는 **날 1개짜리 빈 공구**이고 이름은 공구 번호 문자열입니다. 이어서 `/machine/toolArea/tool/toolName` 으로 이름을, `/machine/toolArea/tool/toolEdge/*` 로 오프셋을 넣으십시오. 날을 더 붙이려면 `/machine/toolArea/tool/toolEdge/toolEdgeExists` 를 쓰십시오. 공구 준비실에서 잰 값을 조작반을 거치지 않고 그대로 등록하는 흐름이 이것입니다.

**매거진 포켓을 차지한 공구는 지울 수 없습니다** (`-18`). 실물은 매거진에 남는데 등록만 사라지면 다음 공구 교환이 어긋나고, 되돌리려 해도 **공구를 다시 만들어 그 자리에 배정할 방법이 없습니다.** 먼저 조작반에서 공구를 빼십시오. 지금 어디에 있는지는 `/machine/toolArea/tool/toolLocationType` 이 답합니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolEdge/toolEdgeExists
그 날 번호가 **그 공구에 있는지** 여부입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원.

없는 날을 물어도 에러가 아니라 `false` 입니다. `/machine/toolArea/tool/toolEdgeCount` 는 개수만 알려주고 번호에 구멍이 있을 수 있으므로, **어떤 번호가 실재하는지는 이 주소가 답합니다.**

**쓰기가 날을 만들고 지웁니다.** `{"value": true}` 로 만들고 `{"value": false}` 로 지웁니다. 이미 그 상태면 아무것도 하지 않고 성공합니다.

**날은 순서대로만 만들어집니다.** 장비가 번호를 고르게 해주지 않고 **비어 있는 가장 작은 번호**를 채웁니다. 그래서 그 번호가 아닌 것을 요청하면 만들지 않고 `-18` 로 거절하며, 다음에 만들어질 번호를 에러 문구에 실어 보냅니다. `D5` 가 필요하면 `3`·`4`·`5` 를 차례로 만드십시오. 구멍이 있으면 그 구멍부터 채워집니다.

**1번 날은 지울 수 없습니다** (`-18`). 공구가 있는 한 남습니다. 공구째 지우려면 `/machine/toolArea/tool/toolExists` 를 쓰십시오.

없는 공구에 날을 만들려 하면 `-18` 입니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolDisabledOn
그 공구가 **쓰지 말라고 표시되어 있는지** 여부입니다. `toolArea` + `tool` 필터 (`toolArea` 는 채널이 쓰는 공구 영역 번호). 반환 `boolean`, **읽기 전용**.

`true` 면 제어기가 그 공구를 쓰지 않습니다. 프로그램이 부르면 거절하거나, 자매공구가 등록되어 있으면 그쪽으로 넘어갑니다 (`/machine/toolArea/tool/sisterToolNumber`).

**공구 단위입니다.** `toolEdge` 필터를 받지 않습니다. 보정 세트가 여럿인 공구도 잠금은 공구 전체에 걸립니다.

표시가 서는 경로는 둘이지만 **값은 구분하지 않습니다**: 작업자가 조작반에서 직접 잠그는 경우와, 공구 수명 감시가 잔여를 다 쓴 시점에 제어기가 자동으로 잠그는 경우입니다. 어느 쪽이든 "지금 못 쓰는 공구" 로 뜻이 같아 갈라 볼 이유가 없습니다.

없는 공구를 지정하면 `-18` 로 거절됩니다.

쓰기도 지원합니다. `{"value": true}` 로 잠그고 `{"value": false}` 로 풉니다. 조작반에 가지 않고 원격으로 공구를 빼거나 되돌릴 수 있습니다.

**수명이 다해 잠긴 공구는 잠금만 풀면 곧 다시 잠깁니다.** 잔여 수명이 `0` 그대로이기 때문입니다. 그 경우에는 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 에 잔여를 되돌려 쓰십시오. 인서트를 갈지 않은 채 잠금만 푸는 것은 다 쓴 날로 깎는다는 뜻이기도 합니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolFixedLocationOn
그 공구가 **고정 자리로 지정되어 있는지** 여부입니다. 늘 같은 포켓으로 돌아갑니다. `toolArea` + `tool` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원합니다. `{"value": true}` 로 지정하고 `false` 로 해제합니다.

`true` 면 공구 교환 후 원래 포켓으로 돌아가고, `false` 면 장비가 빈 포켓을 골라 넣습니다. 장비의 매거진 화면에서 `L` 열이 이 값입니다.

이미 그 상태면 아무것도 하지 않고 성공합니다. 없는 공구는 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolOversizedOn
그 공구가 **포켓 하나보다 큰지** 여부입니다. 양옆 포켓을 비워 둬야 하는 굵은 공구입니다. `toolArea` + `tool` 필터. 반환 `boolean`, **읽기 전용**.

장비의 매거진 화면에서 `Z` 열이 이 값입니다. 몇 포켓을 차지하는지가 아니라 **하나를 넘는지 여부**만 답합니다.

**쓰기는 지원하지 않습니다.** 장비에 초과크기 전용 항목이 없고 위·아래·좌·우로 몇 칸을 차지하는지가 따로 저장되어 있어, `true` 를 받아도 어느 방향으로 몇 칸인지 정할 수 없습니다. 초과크기 지정은 조작반에서 하십시오.

없는 공구는 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/magazine/pocket/pocketDisabledOn
그 포켓을 **쓰지 말라고 표시되어 있는지** 여부입니다. 손상되었거나 비워 둬야 하는 자리입니다. `toolArea` + `magazine` + `pocket` 필터. 반환 `boolean`, 읽기·쓰기 모두 지원합니다. `{"value": true}` 로 잠그고 `false` 로 풉니다.

`true` 면 장비가 공구를 넣을 자리를 고를 때 이 포켓을 건너뜁니다. 장비의 매거진 화면에서 `D` 열이 이 값이며, 그 화면에서 이 칸은 **포켓을 차지한 공구 행에만** 나타납니다. 포켓의 속성이지 공구의 속성이 아니기 때문입니다.

공구 쪽의 잠금은 `/machine/toolArea/tool/toolDisabledOn` 이고 별개입니다. 포켓이 잠겨도 그 안의 공구는 잠긴 것이 아니며, 다른 자리로 옮기면 다시 쓸 수 있습니다.

이미 그 상태면 아무것도 하지 않고 성공합니다. 없는 포켓과 버퍼·반입출 위치의 번호는 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolMonitorType
그 공구의 **수명 감시 방식**입니다. `toolArea` + `tool` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 2}`.

| 값 | 뜻 | 수명 값의 단위 |
|---|---|---|
| `0` | 감시 없음 | 없음 |
| `1` | 시간(실제로 깎은 시간을 센다) | 초 |
| `2` | 개수(완성한 가공물 수를 센다) | 개 |
| `3` | 마모(오프셋이 한계까지 밀렸는지 본다) | 기계 설정 (mm/inch) |

**기종이 늘어도 이 넷입니다.** 장비 고유의 번호가 아니라 디메시가 정한 값이라 어느 기종에 붙었는지 몰라도 그대로 분기할 수 있습니다.

방식은 **공구가 하나 고르고, 값은 날마다 따로**입니다. 그래서 이 주소는 `toolEdge` 를 받지 않고, 수명 값 3종은 받습니다.

`0` 이면 수명 값 3종이 `-18` 로 거절됩니다. 그 공구에는 잴 것이 없습니다. 감시를 켜려면 이 주소에 방식을 먼저 쓰고, 그 다음 `/machine/toolArea/tool/toolEdge/toolLifeTotal` 에 예산을 넣으십시오.

장비가 여러 방식을 **동시에** 켜 둘 수도 있습니다. 그 경우 이 주소는 시간 → 개수 → 마모 순으로 하나를 골라 답하고, 수명 값 3종도 같은 순서를 따르므로 방식과 값이 어긋나지 않습니다. "갈아야 하나" 의 답인 `/machine/toolArea/tool/toolLifeWarnOn` 과 `/machine/toolArea/tool/toolDisabledOn` 은 방식과 무관하게 항상 정확합니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolLifeWarnOn
그 공구가 **경고 한계에 닿았는지** 여부입니다. `toolArea` + `tool` 필터. 반환 `boolean`, **읽기 전용**.

`true` 면 잔여 수명이 경고선 밑으로 내려온 것입니다. **아직 쓸 수 있습니다.** 못 쓰게 된 것은 `/machine/toolArea/tool/toolDisabledOn` 이고, 둘은 독립이라 동시에 `true` 일 수 있습니다 (수명이 다해 잠긴 공구).

곧 교체할 공구를 미리 추리는 용도입니다. 잔여가 얼마나 남았는지는 `/machine/toolArea/tool/toolEdge/toolLifeRemaining` 이 답합니다.

**수명 값은 날별인데 이 플래그는 공구 단위입니다.** 재는 곳과 조치하는 곳이 다르기 때문입니다. 깎는 것은 날이지만 교체는 공구째 하고, 제어기가 넘어가는 자매공구(`/machine/toolArea/tool/sisterToolNumber`)도 공구 단위입니다. 그래서 **어느 날이든** 자기 경고선을 넘으면 이 값이 `true` 가 됩니다.

**어느 날이 넘었는지는 알려주지 않습니다.** 필요하면 날마다 `toolLifeRemaining` 과 `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit` 을 비교하십시오. 날 개수는 `/machine/toolArea/tool/toolEdgeCount` 가 답합니다.

**제어기가 세우는 값이라 쓰기는 지원하지 않습니다.** 고쳐 써도 잔여 수명은 그대로라 다음 판정에서 되돌아갑니다.

감시가 꺼져 있으면 항상 `false` 입니다. 없는 공구는 `-18` 로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolEdge/toolLifeTotal
그 날에 배정된 **수명 예산 전체**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 6}`.

제어기는 잔여를 이 값에서 시작해 깎아 내려갑니다. 단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. `/machine/toolArea/tool/toolMonitorType` 참조. 감시가 꺼져 있으면 `-18` 로 거절됩니다.

개수 감시일 때는 **정수만 받습니다.** 소수를 보내면 장비가 성공을 답하고 값은 바뀌지 않으므로 SDK 가 먼저 거절합니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolEdge/toolLifeRemaining
그 날에 **남은 수명**입니다. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 6}`.

Sinumerik 은 내려세기라 이 값이 곧 "지금 이 날의 수명" 입니다. 얼마나 썼는지는 `toolLifeTotal` 에서 이 값을 빼면 됩니다.

**인서트를 갈고 수명을 되돌릴 때 이 주소에 씁니다.** 보통 `toolLifeTotal` 과 같은 값을 넣습니다. 수명이 다해 잠긴 공구라면 이 쓰기가 잠금을 푸는 올바른 경로입니다.

단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 감시가 꺼져 있으면 `-18`, 개수 감시에 소수를 쓰면 `-16` 으로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolEdge/toolLifeWarnLimit
**경고선**입니다. 잔여가 이 값 밑으로 내려오면 제어기가 경고를 세웁니다 (`/machine/toolArea/tool/toolLifeWarnOn`, **공구 단위**). `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 3}`.

교체 공구를 준비할 시간을 벌기 위한 값이라 `toolLifeTotal` 보다 작게 잡습니다. 단위는 감시 방식이 정하며 응답의 `unit` 에 실립니다. 감시가 꺼져 있으면 `-18`, 개수 감시에 소수를 쓰면 `-16` 으로 거절됩니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolEdge/toolLengthGeometry
**길이1 형상값**입니다 (Sinumerik `DP3`). 선삭 공구에서는 통상 X 방향에 대응하지만, 축 대응은 공구 타입과 활성 평면이 정하는 규칙이라 SDK 는 번역하지 않습니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolLengthWear
**길이1 마모값**입니다 (Sinumerik `DP12`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolLength2Geometry
**길이2 형상값**입니다 (Sinumerik `DP4`). 선삭 공구에서는 통상 Z 방향에 대응합니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolLength2Wear
**길이2 마모값**입니다 (Sinumerik `DP13`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolLength3Geometry
**길이3 형상값**입니다 (Sinumerik `DP5`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolLength3Wear
**길이3 마모값**입니다 (Sinumerik `DP14`).

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolRadiusGeometry
**커터 반경 형상값**입니다 (Sinumerik `DP6`, 밀링 공구 관점). `toolNoseRadiusGeometry` 와 **같은 저장소**를 가리키며, 어느 주소를 쓰는지가 곧 소비자의 의도 선언입니다. SDK 는 공구 타입을 검사하지 않습니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 이름 그대로 **반지름**인데, 공구 목록/오프셋 화면은 흔히 **지름(Ø)** 으로 표시합니다. 실측(2026-07): `BALLNOSE_D8` 의 저장값이 `4.0` 인데 HMI 는 `8.000` 으로 보여줍니다. 디메시는 장비가 저장한 값을 그대로 내보내며 2를 곱하지 않습니다.

## /machine/toolArea/tool/toolEdge/toolRadiusWear
**커터 반경 마모값**입니다 (Sinumerik `DP15`). `toolNoseRadiusWear` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.


**장비 화면과 숫자가 다를 수 있습니다.** 이 값은 이름 그대로 **반지름**인데, 공구 목록/오프셋 화면은 흔히 **지름(Ø)** 으로 표시합니다. 실측(2026-07): `BALLNOSE_D8` 의 저장값이 `4.0` 인데 HMI 는 `8.000` 으로 보여줍니다. 디메시는 장비가 저장한 값을 그대로 내보내며 2를 곱하지 않습니다.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusGeometry
**노즈 반경 형상값**입니다 (Sinumerik `DP6`, 선삭 공구 관점). `toolRadiusGeometry` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusWear
**노즈 반경 마모값**입니다 (Sinumerik `DP15`). `toolRadiusWear` 와 같은 저장소입니다.

반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 125.0}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

단위는 기계 설정을 따릅니다 (mm 또는 inch). 어느 쪽인지는 `/machine/channel/gGroup/gModal?gGroup=4` 로 확인하세요. `G21`/`G710` 이면 metric, `G20`/`G700` 이면 inch. 이 주소는 `unit` 필드를 붙이지 않습니다 (기계마다 달라 고정할 수 없음).

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolTipDirection
**날끝 위치 코드**입니다 (Sinumerik `DP2`, `1`~`9`). 노즈 반경 보정 때 날끝이 노즈 중심 기준 어느 방위에 있는지를 나타내며, 각도가 아니라 위치 코드입니다. Fanuc 의 툴오프셋 트리에도 같은 개념이 있고 (`0`~`9`), **번호 체계와 `desc` 어휘를 공유**하므로 기종이 달라도 값을 그대로 비교·재사용할 수 있습니다 (`desc` 는 `9` 에만: `1`~`8` 방위는 매뉴얼이 도해로만 정의하고 가공 구성별로 세 벌이라 싣지 않습니다). 유효 범위는 `1`~`9` 이며 **`0` 은 허용되지 않습니다.** *"The identifier 0 (zero) is not permitted as a cutting edge position"* (SINUMERIK 828D Tools Function Manual).

반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 3}`. `toolArea` + `tool` + `toolEdge` 필터가 필요합니다. **Siemens 전용**이며, 없는 공구/날(D)을 지정하면 에러로 표면화됩니다. **실제 적용치 = 형상 + 마모** 입니다.

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolType
공구의 **타입 코드**입니다 (read + write, `int` + `desc`). **Sinumerik DP1 코드를 그대로 사용**합니다. 코드 체계는 Siemens 소유의 열린 분류라 디메시가 번역하지 않으며, 정본은 Sinumerik 공구 관리 매뉴얼입니다 (벤더가 코드를 추가해도 값은 그대로 전달). 쓰기는 정수 코드 `{"value": 500}` 입니다. 공구 셋업 자동화용이며 코드 유효성은 NCK 가 판정합니다.

| 계열 | 의미 | 예 |
|---|---|---|
| `1xx` | 밀링 공구 | `120` 엔드밀, `140` 페이스밀, `145` 나사 밀링 |
| `2xx` | 드릴 계열 | `200` 트위스트드릴, `240` 탭, `250` 리머 |
| `4xx` | 연삭 공구 | |
| `5xx` | 선삭 공구 | `500` 황삭, `510` 정삭, `530` 절단, `540` 나사 |
| `7xx` | 특수 | `711` 프로브, `730` 스톱 |

위 표는 **정본이 아니라 길잡이**입니다. 이 코드 체계는 Siemens 가 소유하고 계속 늘어나므로, 정확한 목록은 그 기종의 *SINUMERIK 828D Tools Function Manual*(840D sl 은 해당 공구 관리 편)에서 확인하세요. 조작반의 공구 목록 화면에서도 같은 번호가 보입니다.

알려진 코드는 `desc` 로 의미가 동봉되고 (`{"value": 500, "desc": "turning roughing tool"}`), 미등재 코드는 첫 자리 계열 desc 로 폴백합니다 (`{"value": 573, "desc": "turning tool family"}`). **위 표의 계열 밖 코드는 `desc` 키 자체가 빠집니다** (`{"value": 300}`) — 없는 뜻을 지어내지 않기 위해서입니다. `desc` 가 항상 있다고 가정하지 마세요.

선삭 공구(5xx)의 길이1/2 축 배정, 반경 해석(커터/노즈)을 판별하는 기준값이기도 합니다.

**쓰기 주의**: 없는 공구, 또는 그 공구에 없는 날(D)을 지정하면 `-18` 로 거절됩니다 (에러 문구에 그 공구의 보정 세트 개수가 실립니다). 장비 자체는 보정 세트 개수+1 번째 쓰기로 세트를 새로 만들지만, 오타 한 글자가 조작반에서만 지울 수 있는 세트를 남기므로 디메시는 **존재하는 보정 세트의 수정만** 허용합니다.

## /machine/toolArea/tool/toolEdge/toolTipAngle
그 보정 세트의 **날끝 각**입니다. 드릴이면 선단각(`118.0`), 센터드릴이면 `90.0` 같은 값. `toolArea` + `tool` + `toolEdge` 필터. 반환 `float`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 118.0}`.

**`toolTipDirection` 과 다른 값입니다.** 이름이 한 글자 차이인데 성격이 다릅니다. 저쪽은 날끝이 노즈 중심 기준 **어느 방위**인지를 나타내는 코드이고, 이 값은 날끝의 **각도**입니다.

**각도를 쓰지 않는 공구는 `0.0`** 입니다. 밀링 공구가 그렇습니다. 실측에서 드릴은 `118.0`, 페이스밀은 `0.0` 이었습니다.

**장비 화면의 `N` 열은 이 값과 `toolTeethCount` 를 겸용합니다.** 드릴류면 각도를, 밀링이면 인선 수를 그 한 칸에 보여줍니다. 디메시는 한 주소가 공구 종류에 따라 다른 물리량이 되지 않도록 둘을 따로 냅니다. 화면의 그 숫자를 찾으려면 둘 중 값이 있는 쪽을 보십시오.

**Siemens 전용**입니다. 없는 공구/날(D)을 지정하면 `-18` 로 거절됩니다.

## /machine/toolArea/tool/toolEdge/toolHNumber
그 날에 배정된 **`H` 번호**입니다 (ISO 방언용). `toolArea` + `tool` + `toolEdge` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 5}`.

ISO 방언(Fanuc 식 프로그램)으로 가공할 때, 프로그램의 `H5` 는 **공구와 무관하게** 이 번호가 `5` 인 날의 보정을 적용합니다. 즉 공구를 `T` 로 고르고 보정을 `H` 로 따로 고르는 방식이며, 이 주소가 그 번호를 관리합니다.

`0` 은 **배정되지 않음**입니다. 장비의 공구 목록 화면에 `H` 열이 있고 그 값과 같습니다.

번호가 겹치지 않아야 하지만 **중복 검사는 장비가 합니다.** 이미 쓰이는 번호를 쓰면 거절이 에러로 표면화됩니다. 정수만 받고 음수는 `-16` 으로 거절합니다.

ISO 방언을 쓰지 않는 장비에서는 모든 날이 `0` 입니다. Siemens 표준 방식에서는 `D` 번호가 공구 안의 날을 고르므로 이 값이 필요하지 않습니다.

**Siemens 전용**입니다.

## /machine/toolArea/tool/toolEdge/toolTeethCount
그 보정 세트의 **인선 개수**입니다. "4날 엔드밀" 이라 할 때의 그 수. `toolArea` + `tool` + `toolEdge` 필터. 반환 `int`, 읽기·쓰기 모두 지원합니다. 쓰기는 `{"value": 4}`.

**`toolEdgeCount` 와 다른 값입니다.** 저쪽은 제어기가 그 공구에 대해 갖고 있는 보정 세트의 수이고, 이 값은 한 세트가 기술하는 절삭날이 물리적으로 몇 개인가입니다. 실측 예로 4날 커터가 보정 세트 `1` 개에 인선 `4` 개였습니다.

**보정 세트마다 따로 저장됩니다.** 한 자루에 지름이 다른 절삭부가 둘이면 인선 수도 다를 수 있어, 공구가 아니라 날에 붙습니다.

**장비 화면의 `N` 열과 항상 같지는 않습니다.** 그 열은 밀링 공구면 인선 수를, 드릴류면 선단각을 보여주는 겸용 칸입니다. 실측에서 드릴은 이 주소가 `0` 이고 화면엔 `118.0`(선단각)이 떴습니다. 디메시는 한 주소가 공구 종류에 따라 다른 물리량이 되지 않도록 둘을 섞지 않습니다.

**Siemens 전용**입니다. 없는 공구/날(D)을 지정하면 `-18` 로 거절됩니다.

## /machine/channel/diagnosis/index/diagnosisValue
진단 데이터 **한 행의 값**입니다 (**Fanuc 전용**, `float`, 읽기 전용). `channel=<채널>&diagnosis=<번호>&index=<순번>` 형식이며, 파라미터 통로와 같은 행 모델입니다: 축/스핀들 종속 진단이면 `index` 가 축/스핀들 번호, **단일값 진단은 행이 1개이므로 `index=1`** (`diagnosisValueList` 가 단일값을 원소 1개 배열로 주는 것과 같은 모델). 범위를 넘으면 실제 행 수를 동봉해 `-18` 로 거절합니다. 축 하나만 주기 폴링할 때 전 행을 읽는 List 보다 가볍습니다 (호출 1회).

## /machine/channel/diagnosis/diagnosisValueList
임의 진단 번호의 **값**입니다 (**Fanuc 전용**, `floatArray`). 축/스핀들 종속 진단은 축 수만큼의 배열, 비종속 진단은 원소 1개 배열. 진단에는 경로(채널)별 값이 있어 **채널 스코프**입니다. `channel=` 로 경로를 지정하며, 경로 공통 진단은 어느 채널로 읽어도 같은 값이 옵니다. 진단별 형식(행별 여부·행 수)은 첫 조회 때 벤더 검증으로 파악되어 채널별로 캐싱되므로 반복 폴링이 가볍습니다. `diagnosis=301,308` 처럼 콤마/범위 확장 시 진단별 경계가 보존된 중첩 배열로 옵니다.

## /machine/channel/parameter/index/parameterValue
CNC 파라미터 **한 행의 값**입니다 (**Fanuc 전용**, `float`, 읽기/쓰기). `channel=<채널>&parameter=<번호>&index=<순번>` 로 지정하며, 번호는 Fanuc 매뉴얼의 파라미터 번호 그대로입니다. **번역하지 않으며 · 앞으로도 통일되지 않습니다** (`diagnosis` 필터와 같은 벤더 소유 번호 체계라 기종 간 대응표가 성립하지 않음). 파라미터에는 경로(채널)별 값이 있어 **채널 스코프**입니다. 자주 쓰는 파라미터는 `partCountActual`·`powerOnDuration` 처럼 이름 붙은 주소로 따로 제공되며, 이 주소는 그 밖을 위한 범용 통로입니다.

**모든 파라미터는 행의 배열로 봅니다.** `index` 는 행 순번입니다: 축형 파라미터면 축 번호, 스핀들형이면 스핀들 번호 (화면의 행 순서 그대로, 1부터. 예: X1=1, Y1=2), **단일값 파라미터는 행이 1개이므로 `index=1`** (`parameterValueList` 가 단일값을 원소 1개 배열로 주는 것과 같은 모델). 범위를 넘으면 실제 행 수를 동봉해 `-18` 로 거절합니다.

- 비트 단위 파라미터는 **바이트 값 그대로**(packed 정수) 오갑니다. 비트 분해/합성은 호출자 몫입니다. 특정 비트만 바꾸려면 읽고-수정-쓰기를 하세요 (그 사이 조작반 등 다른 변경과 경합할 수 있습니다).
- 소수(real) 파라미터는 장비의 소수 자릿수가 적용된 실수로 오가며, 쓰기도 같은 자릿수로 저장됩니다.
- 정수 파라미터에 범위를 넘는 값을 쓰면 `-16` 으로 거절합니다 (허용 범위를 에러에 동봉).
- **쓰기 주의**: 파라미터는 기계 거동을 바꿉니다. 장비가 파라미터 쓰기를 막아둔 상태면 벤더 에러(`-17`)로 거절되고, 일부 파라미터는 변경 후 전원 재투입을 요구합니다. Linux 용 Fanuc 라이브러리는 파라미터 쓰기를 제공하지 않아 쓰기는 `-20` 입니다.

## /machine/channel/parameter/parameterValueList
CNC 파라미터의 **전 축 값 배열**입니다 (**Fanuc 전용**, `floatArray`, 읽기 전용). 배열 길이는 벤더 검증으로 파악한 **행 수**입니다 — 축형 파라미터면 축 수, 스핀들형이면 스핀들 수, 비축이면 원소 1개 (`diagnosisValueList`·형제 `parameterValue` 의 행 모델과 동일). 행별 여부·행 수는 첫 조회 때 벤더 검증으로 파악되어 채널별로 캐싱되므로 반복 폴링이 가볍고, `parameter=6711-6713` 처럼 범위/콤마 확장 시 파라미터별 경계가 보존된 중첩 배열로 옵니다.

## /machine/ncMemoryPath/entry
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
폴더 안의 파일/폴더 목록입니다 (**읽기 전용**, `objectArray`). 각 원소는 `entry` 와 **완전히 같은 객체**입니다. 키 집합·`null` 규약 모두 동일하므로 그쪽 표를 보세요. 폴더 우선, 이름 오름차순으로 정렬됩니다. 폴더의 생성/삭제는 `directoryExists` 를 사용하세요.

## /machine/ncMemoryPath/entryName
`ncMemoryPath` 가 가리키는 **항목의 이름**입니다. 경로의 마지막 세그먼트를 돌려주고(read: 순수 문자열 연산, 장비 통신 없음), **이름 변경**(write)을 합니다. 쓰기는 `{"value": "새이름"}` 이며 경로 구분자는 넣을 수 없습니다 (파일/폴더 공통). `entry`/`entryList` 와 같은 "항목" 을 가리킵니다.

## /machine/ncMemoryPath/fileExists
경로에 **파일**이 존재하는지 확인하고(read), 상태를 선언적으로 씁니다(write):

- read → 파일이 있으면 `true` (같은 이름의 폴더만 있으면 `false`)
- write `{"value": false}` → 파일 삭제
- write `{"value": true}` → **에러**: 빈 파일 생성은 미지원. 파일 생성은 내용과 함께 `fileContent` 쓰기로 하세요

경로 끝 `/` 는 무시됩니다 (종류는 주소가 확정). 폴더는 `directoryExists` 사용.

## /machine/ncMemoryPath/directoryExists
경로에 **폴더**가 존재하는지 확인하고(read), 상태를 선언적으로 씁니다(write):

- read → 폴더가 있으면 `true` (같은 이름의 파일만 있으면 `false`)
- write `{"value": true}` → 폴더 생성
- write `{"value": false}` → 폴더 삭제 (**빈 폴더만**: 내용이 있으면 에러, 재귀 미지원)

경로 끝 `/` 는 무시됩니다. 파일은 `fileExists` 사용.

## /machine/ncMemoryPath/fileContent
NC 파일의 **내용**을 읽고(다운로드) 씁니다(업로드: 없으면 생성, 있으면 덮어씀). 값은 문자열 (프로그램 텍스트).

- **Fanuc 쓰기 자동 처리**: `%` 미포함 시 자동 삽입, 선두에 O번호/`<이름>` 이 없으면 경로의 파일명 기준으로 자동 삽입. 저장 파일명은 **내용의 O번호/이름 기준**입니다
- 파일 삭제는 `fileExists` 에 `false` 쓰기

## /machine/plcAddress/plcType/plcValue
**PMC/PLC 메모리의 단일 원소**를 읽고/씁니다 (Fanuc FOCAS2 `pmc_rdpmcrng`/`pmc_wrpmcrng`, Siemens OPC-UA `/Plc/` 노드). 읽기(GET)와 쓰기(POST) 모두 지원하며, 반환 타입은 `float` (값 하나), 쓰기도 숫자 하나입니다 (예: `{"value": 42}`). 이 주소는 **원소 하나 전용**이며, 여러 원소를 한 번에 다루려면 같은 트리의 목록형 주소를 사용합니다. `plcAddress` 와 `plcType` 두 필터가 필요합니다.

**plcAddress**: 주소 형식이 **기종별로 다릅니다**. `plcType` 과 달리 **중립화하지 않는 의도적 예외**입니다. Fanuc 의 `D100` 과 Siemens 의 `DB10.DBB56` 는 서로 다른 메모리 아키텍처를 가리키고, 둘을 잇는 대응표는 SDK 가 알 수 있는 지식이 아니라 그 장비의 래더를 어떻게 짰는지에 달린 현장 설정이기 때문입니다. 앞으로도 통일되지 않으므로, 여러 기종에서 같은 신호를 읽어야 한다면 **호스트 앱이 기종별 주소 표를 들고** 있어야 합니다.

- **Fanuc**: 첫 글자가 PMC 영역, 나머지가 바이트 번호 (예: `R5`, `D100`). `~` 로 범위 지정. 단, 이 주소(단일)는 범위가 **정확히 `plcType` 크기 1개**여야 합니다 (예: word 면 `D100~D101`)
- **Fanuc** PMC 영역 첫 글자: `G` `F` `Y` `X` `A` `R` `T` `K` `C` `D` `M` `N` `E` `Z`: 범위는 같은 영역이어야 합니다 (`D100~D101` O, `D100~R101` X)
- **Siemens**: 조작반의 **`NC/PLC variables` 화면에 보이는 표기 그대로** 씁니다. 값이 `/Plc/{주소}` 노드로 전달됩니다. 첨자 생략 시 `[1]` 자동 부착. 이 주소는 단일 원소만 되며, 다원소면 에러와 함께 목록형 주소를 안내합니다
- **Siemens** 형식: **주소가 오프셋을 품습니다** — `DB<n>.DBB<offset>`(바이트) · `DB<n>.DBW<offset>`(워드) · `DB<n>.DBX<byte>.<bit>`(비트) · `IW<n>` · `MB<n>` · `Q<byte>.<bit>`. 표기 예: `DB10.DBB56` · `DB31.DBX24.1` · `IW0` · `Q0.2`
- **Siemens** 첨자 `[N]` 은 **"몇 번째" 가 아니라 "몇 개"** 입니다. `DB10.DBB56[4]` 는 오프셋 56 부터 **연속 4개**(56·57·58·59)이지 "56번의 4번째" 가 아닙니다. 다른 자리를 짚으려면 첨자가 아니라 **주소를 옮깁니다** (`DB10.DBB61`)
- **Siemens** 문법 주의(기종 무관): 오프셋 없는 표기(`MB` 단독 · `DB<n>` 단독)는 문법이 아니고, 비트는 `DBB` 가 아니라 `DBX` 로 짚습니다
- **Siemens**: 어느 블록·바이트가 **그 장비에 실재하는지는 래더 구성에 달려 기계마다 다릅니다.** 위 표기 예도 형태를 보이기 위한 것이지 어느 장비에나 있는 주소가 아닙니다. 조작반의 같은 화면에서 확인하세요 — 거기서 값이 보이면 여기서도 읽힙니다
- **Siemens** 828D 제약: 828D 는 **`DB9000` 이상의 고객 데이터 블록에만** 접근할 수 있습니다 (840D sl 은 제약 없음)

**plcType**: 원시 바이트를 어떻게 해석할지 정하는 숫자 코드입니다. **기종 무관 통일 값**이라 어느 벤더든 같은 번호를 씁니다 (어댑터가 각 벤더 코드로 번역):

- `1` = bit: 1비트 (0 / 1)
- `2` = byte: 8비트 정수 (부호없음, 0~255) · 주소 폭 1 (예 `D100`)
- `3` = word: 16비트 정수 (부호있음) · 주소 폭 2 (예 `D100~D101`)
- `4` = dword: 32비트 정수 (부호있음) · 주소 폭 4 (예 `D100~D103`)
- `5` = float32: 32비트 실수 · 주소 폭 4 (예 `D100~D103`)
- `6` = float64: 64비트 실수 · 주소 폭 8 (예 `D100~D107`)

`0` = **auto**: 소스가 타입을 결정합니다. Siemens(OPC-UA)처럼 노드가 타입을 아는 프로토콜은 그 네이티브 타입으로 읽습니다. 반면 **Fanuc** 처럼 raw 메모리를 다루는 프로토콜은 intrinsic 타입이 없어 `0`(auto)이 오류이며 명시해야 합니다. **Fanuc(FOCAS2)** 은 bit 타입도 없어 `1`(bit)도 미지원합니다. `2`(byte)~`6`(float64) 중에서 지정하세요.

**중요(Fanuc)**: `plcAddress` 범위의 바이트 수가 `plcType` 크기와 일치해야 합니다 (예: `plcType=3`(word, 2바이트)인데 `D100` 단일 주소면 실패 → `D100~D101` 로 지정). `plcType` 은 **해석 방식만** 정하며, 결과는 `float`(JSON 숫자)로 반환됩니다.

**Siemens** 는 타입이 주소 자체에 인코딩되어 있어 (`DBB`/`DBW`/`DBD` 등) `plcType=0`(auto)을 권장합니다. `1`~`6` 을 넣어도 동작은 동일합니다 (서버가 알려주는 타입으로 읽음). 쓰기는 노드를 먼저 읽어 서버 타입을 확인한 뒤 같은 타입으로 기록합니다.

**에러 코드**: 그 장비에 **실재하지 않는 주소**도 `-18`(필터 값 오류)입니다 — 어느 블록·바이트가 있는지는 그 장비 래더 구성에 달렸으므로, 조작반의 같은 화면에서 먼저 확인하세요. 그 기종이 못 쓰는 `plcType` 도 `-18` 입니다. 규약 밖 값(`0`~`6` 이외)도 같은 `-18` 이며, 두 경우 모두 대응은 같습니다(다른 `plcType` 지정). `-20` 이 아닌 이유는 **주소 자체는 그 기종에서 정상 동작**하기 때문입니다. `-20` 은 "이 주소를 이 기종에서 못 쓴다"는 뜻으로 남겨 둡니다. 에러 문자열에 허용 값이 함께 실려 옵니다.

## /machine/plcAddress/plcType/plcValueList
**PMC/PLC 메모리의 원소 블록**을 배열로 읽고/씁니다. 필터·주소 형식·`plcType` 규칙은 위 `plcValue`(단일)와 동일하고, **여러 원소**를 다룬다는 점만 다릅니다. 반환 타입은 `floatArray`, 쓰기 `value` 는 숫자 배열 `[1, 2, ...]` 입니다. 단일 원소도 `[42]` 처럼 배열로 적어야 합니다.

- **Fanuc**: 범위의 바이트 수가 `plcType` 크기의 **배수**여야 하고, 원소 수 = 바이트 수 ÷ 타입 크기 (예: `D100~D107` + word = 4개 → `[v1,v2,v3,v4]`)
- **Siemens**: 다원소 첨자 허용 — `[N]` 은 **개수**입니다. `DB10.DBB56[4]` 는 오프셋 56 부터 **연속 4개**를 배열로 돌려줍니다. 서버가 주는 원소들이 그대로 배열이 됩니다
- **Siemens**: 첨자를 생략하거나 `[1]` 을 줘도 **결과는 배열**입니다 (`[131.0]`). 이 주소의 반환은 `floatArray` 로 고정이라 원소가 하나여도 흔들리지 않습니다 — 개수가 가변이거나 미리 모를 때 이 주소를 쓰면 파싱 코드가 분기할 필요가 없습니다
- 쓰기는 **원소 수가 대상 범위/노드의 원소 수와 정확히 일치**해야 합니다

**에러 코드**: 그 장비에 **실재하지 않는 주소**도 `-18`(필터 값 오류)입니다 — 어느 블록·바이트가 있는지는 그 장비 래더 구성에 달렸으므로, 조작반의 같은 화면에서 먼저 확인하세요. 그 기종이 못 쓰는 `plcType` 도 `-18` 입니다. 규약 밖 값(`0`~`6` 이외)도 같은 `-18` 이며, 두 경우 모두 대응은 같습니다(다른 `plcType` 지정). `-20` 이 아닌 이유는 **주소 자체는 그 기종에서 정상 동작**하기 때문입니다. `-20` 은 "이 주소를 이 기종에서 못 쓴다"는 뜻으로 남겨 둡니다. 에러 문자열에 허용 값이 함께 실려 옵니다.

## /machine/ncMemorySizeTotal
NC 메모리 전체 용량입니다. 반환 `int` + `unit:"bytes"`.

크기는 SDK 전체에서 **바이트**로 통일되어 있습니다. `entry`/`entryList` 의 `sizeBytes` 와 같은 단위이므로 "이 파일이 남은 공간에 들어가나" 같은 계산에 변환이 필요 없습니다. 장비가 더 거친 단위(예: KB)로만 알려주는 경우에도 이 주소는 바이트로 환산해 내보내며, 그때 값은 그 단위의 배수가 됩니다.

## /machine/ncMemorySizeUsed
NC 메모리 사용량입니다. 반환 `int` + `unit:"bytes"`.

## /machine/ncMemorySizeFree
NC 메모리 잔여 용량입니다. 반환 `int` + `unit:"bytes"`.

## /machine/ncMemoryRootPath
메인 NC 메모리의 root 경로입니다. `ncMemoryPath` 필터에 넣을 경로의 시작점. Fanuc 은 보통 `//CNC_MEM`, Siemens 는 `//NC`.

## /machine/ncMemoryExternalRootPathList
메인 NC 메모리 root 외의 **외부 저장소 드라이브** 목록입니다 (예: 데이터 서버, 메모리 카드). 반환 타입 `stringArray`.

- 각 항목은 root 처럼 앞에 `//` 가 붙습니다 (뒤 슬래시는 없음). Fanuc `//DATA`·`//MEMCARD`, Siemens `//Local drive`
- 이름은 **장비 HMI 표기**입니다. Siemens 의 로컬 드라이브는 OPC-UA 내부 이름이 `NCExtend` 이지만 조작반과 같게 `//Local drive` 로 내보냅니다 (옛 표기 `//NCExtend` 로 요청해도 받습니다)
- 메인 root 자신은 이 목록에서 제외됩니다
- **캐시하지 않음.** 외부 장치는 연결/해제로 바뀔 수 있어 요청마다 새로 조회
- 필터 없음
