## /machine/channelCount
The number of channels (paths) on the CNC. Cached at connection time, so it returns immediately with no additional communication.

## /machine/cncModel
The CNC model string.

- **Fanuc**: series number string — `"15"`, `"16"`, `"18"`, `"21"`, `"30"`, `"31"`, `"32"`, `"35"`, `"0"` (0i), `"PD"`/`"PH"` (Power Mate i), `"PM"` (Power Motion i). `desc` also carries the series name (e.g. `"31"` → `Series 31i`)
- **Siemens**: the model name as-is (e.g. `"840D sl"`)

## /machine/machineType
The machine type. Returns a `string` — a self-describing enum, so no separate code table is needed. The full set of possible values:

- `"machiningCenter"` — machining center (Fanuc M/MM, Siemens M)
- `"lathe"` — lathe (Fanuc T/TT/MT, Siemens T)
- `"punchPress"` — punch press (Fanuc only)
- `"laser"` — laser (Fanuc only)
- `"wireCut"` — wire cut (Fanuc only)
- `"unknown"` — could not be determined

## /machine/currentDateTime
The machine's **current date/time**. Returns a `string`, ISO 8601 to the second (`"2026-07-11T14:30:00"`).

- **Machine-local clock** — since there is no timezone information, no TZ suffix (`Z`/`+09:00`) is appended. It is the ISO 8601 local-time form, and an RFC 3339 parser, which requires an offset, may reject it
- **Do not hand it straight to JavaScript's `new Date()`** — a date-and-time without an offset is interpreted in **the viewer's** time zone. This value is the **machine's** wall clock, not the viewer's
- This is the **CNC's clock**, not the server PC's clock — if the machine's clock is off, it is reflected as-is
- Fanuc: `cnc_gettimer` / Siemens: `sysTimeBCD`
- **Recommended health-check address** — a cheap read that triggers an actual NC round-trip on all protocols, so use it for polling and then judging `status` (`0` = normal, `-10`/`-14` = link problem) to monitor per-machine communication state. (Cache-served addresses like `machineType` may succeed even when the link is dead, making them unsuitable)

Time-related addresses are always ISO 8601 strings (`…At` = event moment, `…DateTime` = clock reading).

## /machine/powerOnDuration
The machine's **cumulative power-on time** — an accumulating total that keeps counting even across power cycles. Returns `int` (seconds) + `unit:"s"`.

- **Fanuc**: parameter 6750 — **minute resolution**, so the value is always a multiple of
  60. Differential calculations (e.g. utilization) carry an inherent ±60 s error
- **Siemens**: `setupTime` — reflects sub-minute resolution (converted to seconds and rounded). A normal power cycle does not reset it, but **powering the control up with default values sets it to `0`** (a rare service operation)

Elapsed-time addresses are always **seconds-normalized int** (the `…Duration` suffix rule).

## /machine/configuredMachineName
Returns the `machine_name` from `config.json` (hub) or the `deemesh_create` configuration as-is — a **value from the configuration**, not a name the machine reports; hence `configured` in the address. Use it to confirm a connection reached the intended machine, or to label a response.

## /machine/configuredProtocol
The **protocol identifier** this connection uses — `"nc_focas2_fanuc"` or `"nc_opcua_siemens"`. No filters. Returns `string`, read-only. It is fixed at connection time, so it answers immediately with no NC communication.

Like `configuredMachineName`, this is a **value from the configuration** — not something the machine reports. It returns the `protocol` field of `deemesh_create` (or of the machine entry in `config.json`) as-is, which is why the address says `configured`.

**Its purpose is narrow.** Most addresses are designed to hide the machine type, so no branching is needed. This value is for the **few places where the value space belongs to the machine type** — PLC address syntax (`D100` vs `DB99:DBB[5]`), diagnosis numbering, tool type codes, and the like, which the catalog explicitly marks as machine-dependent.

**Do not use it to decide whether something is supported.** "This machine type cannot do that address, so skip it" must be decided from `-20`. Branching on this value means your code keeps skipping even after support is added for that machine type.

## /machine/channel/axisCount
The channel's **user axis count** (excluding spindles). Cached at connection time. The valid range of the `axis` filter is `1` to this value.

## /machine/channel/spindleCount
The channel's spindle count. Cached at connection time. The valid range of the `spindle` filter is `1` to this value.

## /machine/channel/toolAreaNumber
The tool area number the channel uses. The value to put in the `toolArea` filter of the tool tree addresses. Fixed at `1` for Fanuc; the NCK setting (`toNo`) value for Siemens.

## /machine/channel/alarmStatus
The **alarm severity**. Regardless of machine it returns **only the three values `0` / `1` / `2`**, and what is actually wrong on that machine comes alongside in `desc`.

| Value | Meaning | Machining |
|---|---|---|
| `0` | normal — no alarm | continues |
| `1` | warning — an alarm exists but machining can continue | continues |
| `2` | alarm — machining stopped | stopped |

- **The number stays these three no matter how many machine types are added.** Vendor codes are not passed through, so you can branch on `value` without knowing which machine it is attached to.
- **The cause comes in `desc`** — Fanuc gives the cause family (`{"value": 1, "desc": "Memory backup battery voltage low (CNC or Amplifier)"}`), Siemens gives whether it stopped (`{"value": 2, "desc": "Alarm with stop"}`). `desc` is a human-readable string, so **do not use it as a branch condition** — branch on `value`.
- Vendor codes whose severity is ambiguous are classified **conservatively as `2`**, because calling a stop a mere warning is more dangerous than the reverse. On Fanuc, a code the SDK newly emits and we do not know is classified as `2` as well. On Siemens the server value is a closed `0`/`1`/`2` set, so anything else is treated as a broken assumption and surfaces as a `-17` error.
- For the alarm **list, numbers and messages** use `alarmList`; for the **count** alone use `alarmCount`. This address is the cheap summary meant for high-frequency polling.
- On Siemens this address reads the per-channel alarm-state node (`chanAlarm`), so `channel` is actually used (unlike the NCK-global `alarmList`/`alarmCount`). On either machine type the `channel` value is range-validated.

## /machine/channel/alarmCount
The **number** of active alarms/messages (= the count of `alarmList` items, regardless of severity). Returns `int`. Use it when you only need the count, like a dashboard badge.

- **Cost note (Fanuc)** — there is no count-only vendor API, so internally it costs the **same** as `alarmList`. Requesting both together in a batch merges them into a single fetch. If you only need a low-cost presence check, use `alarmStatus`
- **Siemens** — a single-node read (`numAlarms`), so it is much cheaper than `alarmList` (an event subscription). Being an NCK-global count, the `channel` value is ignored (same policy as `alarmList`)

## /machine/channel/alarmList
The channel's **active alarms + operator/macro messages** list. Return type `objectArray`, or an empty array `[]` if none. For Siemens, alarms are NCK-global so the `channel` value is ignored.

Element: `{"number": 1234, "message": "SPINDLE OVERHEAT", "category": "Overheat", "severity": "alarm", "raisedAt": "2026-07-29T11:11:03Z"}`

The key set is **always the same regardless of machine** — when a value is unavailable the key is not dropped, it is `null` (same convention as `entry`). `severity` has only the two values `"alarm"` / `"warning"`.

- **number** — alarm/message number
- **message** — display text
- **category** — Fanuc: for alarms, the cause family (`Servo`, `Overheat`, `Spindle`, `PLC`, etc.: undefined types come as a numeric string); for messages, the source (`Operator message` = PMC/external input, `Macro message` = part-program #3006). Siemens: the source reported by the server (e.g. `NCU`: falls back to `Alarm` when empty)
- **severity** — `"alarm"` = machine fault (machining not possible) / `"warning"` = informational (machining possible). Fanuc: among alarms only background-edit errors (BG) are warnings and the rest are alarms; operator/macro messages are all warnings. Siemens: translates the server's severity (1–1000) at a 500 boundary. Whether "machining is currently stopped" should be judged by `executionStatus`, not this field — a warning in a stopped state means it is waiting for operator intervention such as macro `#3006`
- Fanuc carries at most **100 active alarms** and **17 operator/macro messages** per read (vendor API buffer limits).
- **raisedAt** — the time raised, in **UTC with a trailing `Z`** (`"2026-07-29T11:11:03Z"`). Siemens gives the actual time; Fanuc is always `null` (active alarms have no time information)
  - The number **differs from what the machine's own screen (HMI) shows** — the HMI renders the machine's local time while this is UTC. They are the same instant written differently; converting is for whoever knows the machine's time zone (the host application). The standard field that would carry that offset is left empty by this machine
  - **Do not subtract it from `/machine/currentDateTime`.** Beyond the time zone, the two come from **different clocks** — on one machine they were measured about 18 minutes apart

## /machine/channel/operateMode
The current operating mode code (with `desc`). A unified code regardless of machine:

- `0` = Jog · `1` = MDI · `2` = Memory (automatic) · `6` = Edit · `7` = Handle
- `8` = Teach in Jog · `9` = Teach in Handle · `10` = INC feed · `11` = Reference (return to origin)
- `12` = Remote (DNC) · `13`–`17` = combined modes (Jog-REPOS, MDI-Reference, etc.) · `99` = Unknown

## /machine/channel/executionStatus
The program **execution status** code (with `desc`). The "is it running now" counterpart to `operateMode` (which mode it is in):

- `0` = Reset · `1` = Stop · `2` = Hold · `3` = Run (running)
- `4` = MSTR (Fanuc — retraction/recovery) · `5` = Interrupted (Siemens — see below) · `99` = Unknown (Fanuc only — Siemens surfaces unlisted values as a `-17` error)

`1` and `2` are different **kinds** of pause — who halted it, and where:

- `Stop` = halted **by the program at a planned point**. It hit M0/M1, or a block finished in single-block mode — always standing at a block boundary.
- `Hold` = halted **by the operator at an arbitrary moment**. The stop key (feed hold) on the panel was pressed — it can stop mid-block.

⚠️ **The button name and the state name disagree** (an industry convention) — pressing the panel's **Stop button puts the machine in `Hold`**. The `Stop` state is produced by the program (M0/M1, single block), not by a button. Either way, Cycle Start resumes.

`5` (Siemens only) means **paused for a reason not classified as either of the above** (emergency stop, waiting for the spindle, and other stop reasons). The fact that it is paused is certain, so a consumer that does not care about the kind may treat `1`/`2`/`5` together as "paused".

⚠️ **During an emergency stop the value differs by machine type** (verified on live controls): Fanuc reports `0` (Reset), Siemens `5` (Interrupted). This is the one spot where this address gives different codes for the same situation — detect an E-stop with `/machine/channel/emergencyStatus`, not this address.

## /machine/channel/motionStatus
The axis motion status code (with `desc`):

- `0` = None/Idle (Fanuc) · `1` = Motion (moving) · `2` = Dwell (dwelling)
- `3` = multi-path synchronization wait (Fanuc)

Siemens judges by remaining dwell time, so only `1`/`2` appear.

## /machine/channel/emergencyStatus
The emergency-stop status (with `desc`): `0` = normal, `1` = emergency stop. On Fanuc a transient `2` (Reset — the moment the E-stop is being released, under a second) can flash by (verified on a live control) — treating anything non-`0` as "not normal" is the safe reading.

## /machine/channel/partCountActual
The number of parts machined so far — the counter you reset when switching jobs. `channel` filter. Returns `int`; both read and write are supported — write `{"value": 0}` to reset it. The Linux Fanuc library does not provide the function this write uses (`cnc_wrparam`), so on Linux the write is `-20`.

**This is not "how many parts were actually produced".** It is a counter the control increments in response to program end (`M02`/`M30`), and whether it reacts at all depends on the machine configuration. With that setting off it never moves; a dry run increments it too; running the same program twice makes it 2. An operator can change it at the panel. **The control does not know whether a part is good.** To use it for production reporting, the host application must overlay program and timing information.

**On Fanuc a read right after a write may still return the old value** — the control takes a moment to apply a parameter (measured: reading immediately gave the old value 6 times out of 8; after `50` ms all reads were correct). Read again shortly afterwards to confirm what was written. On the same Fanuc, macro variable writes apply immediately — only parameters behave this way. Siemens applies immediately.

Fanuc uses parameter `6711`, Siemens `actParts`.

## /machine/channel/partCountRequired
The target quantity to be produced. `channel` filter. Returns `int`; both read and write are supported — write `{"value": 100}`. A `0` means no target is set. The Linux Fanuc library does not provide the function this write uses (`cnc_wrparam`), so on Linux the write is `-20`.

The machine can be configured to signal or stop once the machined quantity reaches this value, but whether it does so depends on the machine configuration — deemesh only carries the value.

**On Fanuc a read right after a write may still return the old value** — the control takes a moment to apply a parameter (measured: reading immediately gave the old value 6 times out of 8; after `50` ms all reads were correct). Read again shortly afterwards to confirm what was written. Siemens applies immediately.

Fanuc uses parameter `6713`, Siemens `reqParts`.

## /machine/channel/cuttingDuration
The **cumulative cutting time** — the accumulated time the tool was actually engaged and cutting. `channel` filter. Returns `int` (seconds) + `unit:"s"`, read-only.

It forms a layer together with the power-on time: how much of the time the machine was on was actually spent cutting is the raw material for a utilization figure. Being a running total, the usage over an interval is the difference between two readings.

**Measurement pauses under some conditions.** It does not count while the program is in the stopped state or while the feed override is `0`. Siemens additionally does not count during rapid traverse, while no tool is active, or during a dwell — Fanuc also counts on cutting feed, but whether it treats dwell the same way has not been confirmed.

**The reset point differs by machine type.** Fanuc keeps accumulating; on Siemens, powering the control up with default values sets it to `0` (a normal power cycle does not). On both, an operator can reset it at the panel.

**Siemens can have this measurement switched off** — if machine data `27860` disables it, the value stays `0`.

**Writing is not supported.** This is the machine's history, and changing it would silently distort production reporting.

Fanuc sums parameters `6754` (minutes) and `6753` (milliseconds below a minute); Siemens uses `cuttingTime`. The two split parameters are read **in a single call**, so the value cannot be skewed by the minute rolling over between reads.

## /machine/channel/programRunDuration
The **run time of the current automatic cycle** — it restarts from `0` when a new cycle begins. `channel` filter. Returns `int` (seconds) + `unit:"s"`, read-only.

It is not a running total. Unlike the values that accumulate over the machine's life (power-on time, cutting time), this measures **one run**. It does not count while the program is stopped or while the feed override is `0`.

**Sub-second precision is rounded away.** Both machine types offer millisecond resolution, but elapsed-time addresses are uniformly whole seconds — for a cycle only a few seconds long the error is correspondingly large.

Fanuc reads parameters `6758` (minutes) and `6757` (milliseconds below a minute) together in a single call; Siemens uses `actProgNetTime`.

## /machine/channel/partCountTotal
The machine's lifetime part count — a running total that is not reset when jobs change. `channel` filter. Returns `int`, **read-only**.

Writing is deliberately not supported: this is the machine's history, and changing it would silently distort production reporting. A resettable counter is available separately.

This value is also a counter driven by program end, so dry runs and re-runs are counted as well — **do not use it as "how many parts were actually produced".**

Fanuc uses parameter `6712`, Siemens `totalParts`.

## /machine/channel/feedOverride
The feed override (%). Returns `int` + `unit:"%"`. Fanuc reads it from the PMC G12 signal, Siemens from the `feedRateIpoOvr` node.

## /machine/channel/rapidOverride
The rapid-traverse override (%). Returns `int` + `unit:"%"`. **Fanuc is stepped** — only `0` / `25` / `50` / `100` appear. Siemens is continuous.

## /machine/channel/feedCommanded
The commanded feedrate (F command value). Returns `float`. Fanuc uses the modal F, Siemens `cmdFeedRateIpo`.

The unit follows the machine setting (mm/min or inch/min). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/feedActual
The channel's actual feedrate — **the speed at which the tool tip travels along the programmed path**. Returns `float`. Fanuc uses `actf`, Siemens `actFeedRateIpo`.

This is the value the `F` command sets. It holds at the commanded rate as the direction changes, and drops when override, acceleration/deceleration, corner slowdown or feed hold apply. Use it to answer "is the machine actually cutting at the programmed feed?"

The unit follows the machine setting (mm/min or inch/min). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/programSequenceNumber
The **sequence number (N number)** of the block currently executing. Returns `int`.

**On blocks without an N number the machines differ** (both verified on live controls) — Siemens is **always `0`** (it never retains the previous N, not even within the same file). Fanuc retains the last executed N number. Only on Siemens can `0` be read as "currently in a block with no N number".

On blocks that do carry an N, both machine types show **the subprogram's N numbers** inside a subprogram and **the main's N numbers** after the return.

⚠️ **Fanuc's retention crosses file boundaries** (verified on a live control) — on an N-less block right after returning from a subprogram you see **the subprogram's last N**, and right after entering one you see **the main's N**. On Fanuc this value alone therefore cannot tell you which file the N belongs to — read `/machine/channel/programName` alongside it. On Siemens the leak cannot occur, since nothing is retained.

## /machine/channel/programBlockCounter
The executed-block counter. **Its meaning differs by machine**: Fanuc is the number of blocks executed **since the cycle started** (`cnc_rdblkcount` — it resets at each Cycle Start and keeps counting through subprogram blocks; verified on a live control); Siemens is the **line number** currently executing within the program (`actLineNumber`, negatives clamped to 0). Returns `int`.

On Siemens the line number is **relative to the file currently executing** — entering a subprogram switches it to the subprogram's line numbers, and returning switches it back to the main's (verified on a live control). The number alone therefore cannot tell you which file it counts — line 3 of the main and line 3 of the subprogram are both `3`. To pin down the file, read `/machine/channel/programName` and `/machine/channel/programNestLevel` alongside it. During the nesting transition (under a second) a sample can briefly combine mismatched values (the level updates before the name).

## /machine/channel/programName
The name (file name) of the program **currently executing**. When entering a subprogram, it changes to that subprogram's name. For the main selected in the HMI, see `mainProgramName`.

## /machine/channel/programPath
The full path of the currently executing program (e.g. `//CNC_MEM/USER/PATH1/O0001`). Siemens converts the NCK-internal path into the user notation (`//NC/...`) before returning it.

## /machine/channel/mainProgramName
The name of the **main program selected in the HMI**. It does not change even when a subprogram is entered during execution (that is the difference from `programName`).

## /machine/channel/mainProgramPath
The full path of the **main program selected on the HMI** (the path form of `mainProgramName`). It does not change when execution descends into a subprogram.

**Writing selects the program** — it makes the program at that path the channel's main program (the one to be executed). The value is a path string: `{"value": "//CNC_MEM/USER/O0001"}`.

- Path notation follows the machine — Fanuc `//CNC_MEM/USER/O0001` (data server: `//DATA_SV/...`), Siemens `//NC/Part programs/PART1.MPF` (the same notation `programPath` and `entryList` return; `Subprograms` and `Workpieces` likewise). NC file-system paths are vendor-specific and are not normalized, for the same reason as `plcAddress`
- **It must be a file** — passing a folder path returns `-18`, as does a path that does not exist
- Selecting does **not start machining** (cycle start remains the operator panel's / PLC's job)
- Siemens calls the server's file-handling `Select` method; Fanuc uses `cnc_pdf_slctmain` for CNC memory paths and `cnc_wrdsdncfile` for data-server paths

## /machine/channel/programCurrentBlock
The **G-code text** of the block currently executing. When nothing is executing the value is an **empty string** (the same on both controls, never `null`).

## /machine/channel/programNextBlock
The G-code text of the block to be executed next. When there is no next block (the last block, or execution stopped) the value is an empty string.

## /machine/channel/programLastBlock
The G-code text of the block executed just before, or an empty string when there is no preceding block. **Siemens only** (Fanuc unsupported).

## /machine/channel/programLookAhead
**The program text around the current execution point** — a multi-line string holding the current block and what lies ahead of it. Returns `string`.

How much you get differs by machine: Fanuc returns the whole look-ahead buffer (`cnc_rdexecprog`), Siemens a chunk around the execution point (`actPartProgram`). **The whole program cannot be obtained from this address** — for that, read the program file from the NC file system.

Line endings are normalized to a single LF (`
`) on every protocol — CR is stripped, so splitting on `
` is safe.

## /machine/channel/programNestLevel
The program-call nesting level (with `desc`): `0` = no program, `1` = main executing, `2`+ = subprogram (L1, L2, …). **Siemens only**.

## /machine/channel/auxModal/auxModalValue
The auxiliary-function modal value — specify a **letter** in the `auxmodal` filter (e.g. `auxmodal=M`, `S`, `T`, `D`, `H`, `F`). Returns `float`. Example: `auxmodal=T` → the commanded tool number, `auxmodal=S` → the commanded spindle speed.

## /machine/channel/singleBlockOn
The single-block switch state (`true` = on). Fanuc reads the F4 signal bit, Siemens `singleBlockActive`.

## /machine/channel/dryRunOn
The dry-run switch state (`true` = on).

## /machine/channel/optionalStopOn
The optional-stop (M01 enabled) switch state (`true` = on).

## /machine/channel/blockSkipOn
The block-skip (`/`) switch state (`true` = on). Siemens has multiple skip levels 0–9, and this address looks at **level 0** (the same meaning as Fanuc's `/`).

## /machine/channel/machineLockOn
The machine-lock (axis-motion lock) state (`true` = on). For Siemens this is the program-test (`progTestActive`) state.

## /machine/channel/variable/variableValue
Reads/writes a **macro variable (Fanuc) / R parameter (Siemens)** (read + write). Put the variable number in the `variable` filter (e.g. `variable=100` → Fanuc `#100`, Siemens `R100`). Returns `float`; writes take `{"value": 3.14}`. **Reads** support range/comma expansion — `variable=100-105` is an array of 6 values. Writes always target a single variable (expansion syntax is rejected with `-13`: the rule shared by every write). A **vacant Fanuc macro variable reads as `null`** — this is the state the control's custom-macro screen shows as `DATA EMPTY`, and it is distinct from the value `0`. In a range expansion only that slot becomes `null` (e.g. `[3.14, null]`).

## /machine/channel/toolOffset/toolLengthGeometry
The **M-type (machining center) tool length geometry** value (the H column on the offset screen). The reference value entered from tool measurement, forming the basis of length compensation (H code).

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolLengthWear
The **M-type tool length wear** value (the H column on the offset screen). The fine compensation accumulated during machining; the usual practice is to adjust this alone and leave the geometry value untouched.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolRadiusGeometry
The **M-type tool radius geometry** value (the D column on the offset screen). The radius reference that cutter compensation (G41/G42) consults.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.


**The number can differ from what the machine's screen shows.** This value is a **radius**, but an offset screen may be configured to display and accept **diameters**. deemesh emits what the machine stores and does not convert.

## /machine/channel/toolOffset/toolRadiusWear
The **M-type tool radius wear** value (the D column on the offset screen). Reflects the radius reduction caused by tool wear.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.


**The number can differ from what the machine's screen shows.** This value is a **radius**, but an offset screen may be configured to display and accept **diameters**. deemesh emits what the machine stores and does not convert.

## /machine/channel/toolOffset/toolXGeometry
The **T-type (lathe) X-direction tool dimension geometry** value. Here X is not an axis name but a fixed column on the offset screen — that is, a **directional component of the tool dimensions**.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolXWear
The **T-type X-direction tool dimension wear** value. The X-direction compensation accumulated during machining.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolZGeometry
The **T-type Z-direction tool dimension geometry** value. Like X, this is a fixed column on the screen, not an axis.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolZWear
The **T-type Z-direction tool dimension wear** value.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolYGeometry
The **T-type Y-direction tool dimension geometry** value — **only on Y-axis-option machines**, so a lathe without the option returns `-20`.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolYWear
The **T-type Y-direction tool dimension wear** value (Y-axis-option machines only).

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolNoseRadiusGeometry
The **T-type nose radius geometry** value. Consulted by nose-radius compensation (G41/G42); together with the tip direction (`toolTipDirection`) it determines the tool-tip path.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolNoseRadiusWear
The **T-type nose radius wear** value.

Returns `float` (a real distance); both read and write are supported — write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. **Fanuc only**; if the machine's offset screen has no such column, `-20` is returned. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolTipDirection
The lathe tool's **virtual tool-tip position code** (0–9, read + write). A code that determines, during nose-radius compensation (G41/G42), where the tool tip lies relative to the nose center — it is a position code, not an angle, and is returned/entered as an integer with no scaling (`{"value": 3}`).

- `1`–`8` = orientation, **`0`/`9` = the tool nose center is the reference point** (rather than the imaginary tip). The two values mean the same thing — *"Imaginary tool nose numbers 0 and 9 are used when the tool nose center coincides with the start point"* (FANUC 0i-F lathe manual `B-64604EN-1/01` §5.2.2)
- **`desc` is attached only to `0` and `9`** — the manual defines `1`–`8` **only by diagram**, and there is more than one diagram (per plane: `G17`/`G18`/`G19`), so the same number denotes different orientations depending on the setup. For the per-orientation reading follow that machine's manual
- Siemens equivalent concept: cutting edge position (`toolArea/tool/toolEdge/toolTipDirection`). **The two trees share the same numbering and the same `desc` vocabulary** — only the address differs, so the values can be compared and reused as-is

## /machine/channel/toolOffsetCount
The **number of available** tool compensation registers (read only, `int`). Offset numbers run `1` to this value — use it as the upper bound when a UI iterates the table.

## /machine/channel/workOffset/axis/workOffsetValue
**Work coordinate system offset** — the per-axis offset distance of a work coordinate system such as G54 (read + write). Returns `float`, a real distance (in the machine's configured unit mm/inch as-is; the SDK normalizes Fanuc's internal integer representation by the decimal scaling).

The `workOffset` filter **takes the shop-floor G-code notation directly** (an open namespace like `plcAddress` — no separate numbering system). Case-insensitive; whitespace and aliases are not allowed:

- **Fanuc**: `EXT` (common offset — added to all coordinate systems, the operator-panel EXT row), `G54`–`G59`, extended `G54.1P1`–`G54.1P300` (a P number not in the option returns a vendor error)
- **Siemens**: `G500`, `G54`–`G57`, `G505`–`G599`. How many actually exist depends on the machine configuration, so a designator that does not exist answers `-18` together with **the list this machine accepts**

⚠️ **`G500` is not the same as Fanuc's `EXT`.** `EXT` is a common offset that is **added on top** of whichever `G5x` is active, whereas `G500` is an **exclusive member of the same modal group** as `G54`–`G57` and therefore cannot be active alongside them — when `G500` is in effect the settable offset is switched off, and the value in that slot is normally `0`. Read `/machine/channel/gGroup/gModal?gGroup=7` to see which one is active. On Siemens the part that adds to every coordinate system the way `EXT` does lives in **separate frames** (the `Basic reference` and `Total basic WO` rows on the operator panel), and deemesh does not expose those individually — for the combined result, read `/machine/channel/axis/totalWorkOffsetValue`.

**On Siemens the value is the coarse offset plus the fine offset.** The machine applies that sum and the operator panel shows them as two cells of one offset (`Coarse` and `Fine`), so this address gives you **the offset actually in effect** — comparing it against the panel's `Coarse` cell alone can look like a mismatch. To read the fine part on its own, use `/machine/channel/workOffset/axis/workOffsetFineValue`. Fanuc has no fine offset, so its value is single; that is what makes this address mean the same thing on both machine types.

⚠️ **This value is the stored translation.** Two more things bear on it. ① A work coordinate system can also carry **rotation, scaling and mirroring** (`workOffsetRotation`, `workOffsetScale`, `workOffsetMirrorOn`), and where those are set the coordinate transform is not determined by this value alone. ② The total actually in effect can differ from this value, because a basic reference and other frames add to it (`totalWorkOffsetValue`). If you need part coordinates, do not compute them — read `/machine/channel/axis/workPosition`. On an ordinary setup that only translates, rotation is `0`, scaling is `1` and mirroring is `false`, so this value *is* the transform.

`axis` is the axis number (1–) or the axis name. `axis=1-3` · `workOffset=G54,G55` expansion is supported — for Fanuc, axis expansion of the same workoffset is bundled into a single FOCAS call.

Writes take `{"value": 25.4}` (a single axis). **Currently Fanuc only** — Siemens write is planned after the machine-side activation procedure (SETUFR PI service) is verified (for now it is `-20`).

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/workOffset/axis/workOffsetFineValue
The **fine (`Fine`) part** of a work coordinate system offset. Filters and type are the same as `/machine/channel/workOffset/axis/workOffsetValue`. **Read-only**.

A fine offset is **a small correction laid on top without touching the base offset**. What you measure when first establishing the coordinate system goes into the coarse value (`Coarse`); if the first part then measures `0.02mm` out, that `0.02` goes into the fine offset — the original setup value stays intact and traceable, and zeroing the fine offset returns you to the setup state.

**The offset in effect is the coarse value plus this one**, and that sum is what `workOffsetValue` answers. If you need the coarse value alone, subtract this from `workOffsetValue` — there is no separate address for it.

On a machine that does not use fine offsets (or has them switched off in machine data) it is `0`.

**Siemens only** — a Fanuc work coordinate system offset is a single value with no fine part.
## /machine/channel/workOffset/axis/workOffsetRotation
The **per-axis rotation angle** of a work coordinate system. Filters are the same as `/machine/channel/workOffset/axis/workOffsetValue`. Returns `float` with `unit` `deg` (degrees). **Read-only**.

`0` means no rotation is set on that axis.

**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five — on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**Writing is not supported.** Writing to these values directly makes the machine **accept the request and change nothing** (measured). A separate activation step is required on the machine side and this protocol does not expose it — make changes at the operator panel. The translation (`workOffsetValue` and `workOffsetFineValue`) is `-20` on Siemens for the same reason.

**Siemens only.**

## /machine/channel/workOffset/axis/workOffsetScale
The **per-axis scale factor** of a work coordinate system. Filters as above. Returns `float`; it is dimensionless, so there is no `unit`. **Read-only**.

`1` means no scaling (true size). `2` machines at double size along that axis.

**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five — on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**Writing is not supported.** Writing to these values directly makes the machine **accept the request and change nothing** (measured). A separate activation step is required on the machine side and this protocol does not expose it — make changes at the operator panel. The translation (`workOffsetValue` and `workOffsetFineValue`) is `-20` on Siemens for the same reason.

**Siemens only.**

## /machine/channel/workOffset/axis/workOffsetMirrorOn
Whether the work coordinate system **mirrors** that axis. Filters as above. Returns `boolean`. **Read-only**.

`true` reverses the direction of that axis. This is the per-axis checkbox on the work-offset detail screen of the operator panel.
**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five — on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**Writing is not supported.** Writing to these values directly makes the machine **accept the request and change nothing** (measured). A separate activation step is required on the machine side and this protocol does not expose it — make changes at the operator panel. The translation (`workOffsetValue` and `workOffsetFineValue`) is `-20` on Siemens for the same reason.

**Siemens only.**

## /machine/channel/axis/totalWorkOffsetValue
The **total zero offset actually in effect** right now, per axis (translation). Returns `float`. **Read-only.** This corresponds to the `Total WO` row on the operator panel.

Where `workOffsetValue` is the value **stored in the table**, this address is the value **being applied**. When the two differ, that difference is the clue you are looking for:

```
workOffsetValue?workOffset=G54     80.400    what you set
totalWorkOffsetValue              100.400    what is actually in effect
                                   ↑ 20.000 was added somewhere
```

Besides the stored offset, the total includes the **`Basic reference`, base frames, a `TRANS` set by the program, and frames set by cycles**. Compare the two addresses when diagnosing "the setting is unchanged but the part is off" — reading only the stored value hides the part that was added.

⚠️ **Do not compute part coordinates from this value.** It is only the total translation; **rotation, scaling, mirroring and tool length compensation are not in it.** If you need workpiece coordinates, read `/machine/channel/axis/workPosition` — that is the result after the machine has applied all of them.

This address takes no `workOffset` filter — it is "whatever is in effect", so there is no designator to choose. `axis=1-3` expansion is supported, and writing is not (a summed result is not something you write back).

**Siemens only.** On Fanuc, read the common offset separately with `workOffset=EXT` and add it to the active coordinate system's value — but that sum omits any shift set by the program, so it does not mean the same thing as this address.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/gGroup/gModal
Queries the active G modal by a **machine-independent standard group number** (a vendor-neutral number defined by deemesh like `plcType` — not the vendor's raw group number). `gGroup` filter values:

- `1` = motion — feed mode (G00 rapid / G01 linear / G02·G03 circular …)
- `2` = plane — machining plane (G17 XY / G18 ZX / G19 YZ)
- `3` = distanceMode — absolute/incremental (G90/G91) · **not supported on Fanuc T-series (System A)** (which uses the U/W address method)
- `4` = units — inch/metric (G20·G70 / G21·G71)
- `5` = feedMode — feed specification (per-minute/per-rev/inverse-time)
- `6` = cutterComp — tool-radius compensation (G40 cancel / G41 left / G42 right)
- `7` = coordinateSystem — work coordinate system (G54–G59)
- `8` = spindleSpeedMode — constant surface speed (G96) / constant rpm (G97)

The value is that machine's G-code string, plus for key combinations a machine-independent meaning in `desc` (e.g. Fanuc `{"value":"G21","desc":"metric"}`, Siemens `{"value":"G710","desc":"metric"}`). For access to the raw vendor group, use `gModalList`. **Siemens note**: `feedMode` and `spindleSpeedMode` are the same group on Sinumerik, so the value is the same and the meaning is distinguished by `desc` (the G96 family = constant surface speed).

## /machine/channel/gModalList
Returns the **full list of modal G codes reported by the machine in vendor order** (`stringArray`). For reproducing a machine-specific HMI's modal screen — the meaning of each group index is per that vendor's manual.

- **Fanuc**: FOCAS modal group order 0–20. Groups not defined on that machine are `null`
- **Siemens**: `ncFkt` G-function group order 1–N (N = the machine's group count)

For machine-independent logic, use `gModal`'s symbolic keys instead of this list.

## /machine/channel/axis/machinePosition
The axis's machine coordinate. Specify the axis with the `axis` filter; a range (`axis=1-3`) or multiple selection (`axis=1,2`) is possible. The return type is `float` (64-bit double precision).

The four position addresses (machinePosition/workPosition/distanceToGo/relativePosition) are all **real distances** — in the machine's configured unit (mm/inch) as-is and matching the operator-panel display (Fanuc's internal integer representation is normalized by the SDK using each axis's decimal scaling). All four are only valid once the axis has established its reference point — right after power-on, check `/machine/channel/axis/axisReferencedOn` first (before establishment, plausible-looking numbers arrive silently).

⚠️ **This value is the coordinate of the tool reference point (the spindle face).** `workPosition` is at the tool tip, so the difference between the two includes **tool length compensation** — `machinePosition − zero offset = workPosition` does not hold (the active work coordinate system's rotation, scaling and mirroring bear on it too). If you need workpiece coordinates, read `workPosition` instead of computing them yourself.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/workPosition
The axis's workpiece coordinate (absolute coordinate). Returns `float`.

This value is measured at the **tool tip** and is the result after the active zero offset, rotation, scaling, mirroring and tool length compensation have **all been applied** — it is the final coordinate the machine computed, so there is no need to derive it from `machinePosition` (subtraction does not give the right answer).

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/distanceToGo
The axis's **remaining travel** in the current block. Returns `float`.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/relativePosition
The axis's relative coordinate. Returns `float`.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/axisName
The axis name (e.g. `"X"`, `"Z1"`). Used to confirm the correspondence between the `axis` number and the actual axis.

## /machine/channel/axis/axisLoad
The axis (servo) load rate. Returns `float` + `unit:"%"`.

## /machine/channel/axis/axisFeedActual
The **component** of the actual feed along this axis (**Siemens only**). Returns `float`. Fanuc (FOCAS2) does not provide per-axis feed, so it is unsupported (`-20`).

It is not the speed at which the tool travels along the path, but that speed resolved onto one axis. So **it keeps changing with direction even under the same command** — with `F1000` in XY, this reads 1000 while only X moves, and about 707 on a 45° diagonal.

Adding the axis components **does not give the path speed** (it is a vector magnitude: not 707+707 but √(707²+707²)=1000). Use this value to see whether a particular axis is hitting its own velocity limit and holding the path back.

The unit follows the machine setting (mm/min or inch/min). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/axisCurrent
The axis motor current. Returns `float` + `unit:"Ampere"` on both protocols. Fanuc reads a diagnosis value, Siemens the drive parameter `R0078`.

## /machine/channel/axis/axisTemperature
The axis motor temperature. Returns `float` + `unit:"°C"`. For Fanuc this is diagnosis 308.

## /machine/channel/axis/axisInterlockOn
The axis interlock state (`true` = interlock engaged). **Fanuc only**.

## /machine/channel/axis/axisReferencedOn
Whether the axis has **established its machine reference point**. Returns `boolean`. **Read-only.**

On an axis where this is `false` the coordinate system is not yet established — **the position addresses (`machinePosition`, `workPosition`, `relativePosition`, `distanceToGo`) can return plausible-looking numbers that mean nothing.** This is not an error you would notice; wrong values arrive silently, so anything that consumes positions right after power-on should check this value first. Machines with incremental encoders need a reference return before coordinates are valid; machines with absolute encoders are always `true`.

Once established it stays `true` wherever the axis moves — this is **coordinate-system validity**, not the momentary "is the axis at the reference position right now".

Fanuc reads the standard CNC→PMC signal ZRF (per-axis bits of `F120`), Siemens reads `refPtStatus` (both verified on live controls). On Fanuc up to 16 axes are supported, and paths beyond the first on a multi-path machine are not.

## /machine/channel/axis/axisEnergyNet
The axis's net **energy** (cumulative consumed − cumulative regenerated). **Fanuc only** (diagnosis 4920), `float` + `unit:"Wh"`.

Energy (Wh) is not power (W) — W is an instantaneous rate, Wh is an accumulated amount. This value keeps climbing like an odometer, so to get the energy used by one job, **read it before and after and subtract**. For average power in W, divide by the elapsed time: `ΔWh ÷ Δhours`.

## /machine/channel/axis/axisEnergyConsumed
The axis's cumulative consumed **energy**. **Fanuc only** (diagnosis 4921), `unit:"Wh"`. Cumulative, so subtract two readings for an interval.

## /machine/channel/axis/axisEnergyRegenerated
The axis's cumulative regenerated **energy** (recovered while decelerating). **Fanuc only** (diagnosis 4922), `unit:"Wh"`.

## /machine/channel/spindle/spindleLoad
The spindle load rate. Returns `float` + `unit`. Siemens always carries `unit:"%"`. On Fanuc the vendor response carries the unit itself, so `%` or `rpm` arrives depending on the machine configuration, and in the rare case the vendor reports some other unit code the `unit` key is omitted. Do not assume `%` — read `unit`.

## /machine/channel/spindle/spindleOverride
The spindle override (%). Returns `int` + `unit:"%"`. **Fanuc is a channel-common value** (the G30 signal — the `spindle` filter is ignored, same value for all spindles); Siemens is a per-spindle value.

## /machine/channel/spindle/spindleCurrent
The spindle motor current. **Siemens only** (drive parameter `R0078`). Returns `float` + `unit:"Ampere"`.

## /machine/channel/spindle/spindleTemperature
The spindle motor temperature. Returns `float` + `unit:"°C"`. For Fanuc this is diagnosis 403; for Siemens the drive parameter R0035.

## /machine/channel/spindle/speedCommanded
The commanded spindle speed (S command value). Returns `float` + `unit:"rpm"`. **Fanuc is the channel modal S value** (the `spindle` filter is ignored — the S command is a channel-level concept); Siemens is the per-spindle `cmdSpeed`.

## /machine/channel/spindle/speedActual
The per-spindle actual speed. Returns `float` + `unit:"rpm"` on both protocols — Fanuc uses `cnc_acts2`, Siemens the per-spindle `actSpeed`. Both machines specify the target spindle with the `spindle` filter.

## /machine/channel/activeToolNumber
The number (`T`) of the tool whose compensation is **currently in effect** in that channel. `channel` filter. Returns `int`.

This is channel state — it is neither the tool physically **held in the spindle** (a location), nor a tool that has only been **selected by `T` and not yet changed in**.

Put this number into the `tool` filter of the tool tree to look up that tool's name, edge count and offsets. The `toolArea` value you also need comes from `/machine/channel/toolAreaNumber` (cached at connection time, so it costs no extra communication).

Fanuc reads the `T` modal, Siemens `actTNumber`.

**Fanuc verification pending**: Fanuc reads the `T` modal, and whether that value changes at tool-change completion (`M06`) or already at the `T` word is still awaiting verification on a real machine (in the latter case there are moments when it is indistinguishable from the "merely selected" tool). Until verified, do not use this value as a tool-change signal where the timing matters. Siemens is definitive (change-completion based, `actTNumber`).

## /machine/channel/activeToolEdgeNumber
The number (`D`) of the edge whose compensation is **currently in effect** on the active tool. `channel` filter. Returns `int`. **Fanuc has no per-tool edge concept, so it is fixed at `1`**; Siemens uses `actDNumber`.

## /machine/channel/spindle/spindleEnergyNet
The spindle's net **energy** (cumulative consumed − cumulative regenerated). **Fanuc only** (diagnosis 4930), `unit:"Wh"`. Cumulative, so subtract two readings for an interval.

## /machine/channel/spindle/spindleEnergyConsumed
The spindle's cumulative consumed **energy**. **Fanuc only** (diagnosis 4931), `unit:"Wh"`.

## /machine/channel/spindle/spindleEnergyRegenerated
The spindle's cumulative regenerated **energy**. **Fanuc only** (diagnosis 4932), `unit:"Wh"`.

## /machine/userData/userDataValue
Reads/writes Siemens **global GUD (SGUD)** user variables (currently **OPC-UA (Siemens) only**, NC-wide shared variables). Both read (GET) and write (POST) are supported. Since GUD variables have different types, the return type is `object` — it comes as a **self-describing envelope** `{"type":..,"data":..}` that also tells you what type the value is. The only filter is `userData` — this address covers **NC-wide shared** variables only, so no channel is specified.

**userData** — the format is `SGUD:<name>` or `SGUD:<name>[<index>]`. The prefix is the GUD definition block name (currently `SGUD` is supported — MGUD etc. later). **All indices are exactly as shown on the machine screen (HMI)** — enter the number you see on screen as-is (0-based, automatically converted to the OPC-UA internal number).

- no index → scalar variable
- `[i]` → a single i-th element of a 1D array (if the screen shows `_ARR[3]`, use `[3]`)
- `[i-j]` → the i–j range of a 1D array (both ends inclusive — the same convention as `1-3` in filter expansion)
- `[r,c](column count)` → a single (row, column) element of a 2D array. **Write the brackets as shown on screen** and append only the array's column count in parentheses — if the screen shows columns up to `[0,3]`, use `(4)` (the machine does not report the column count, so it must be entered as well)
- Examples: `SGUD:MYVAR` (scalar), `SGUD:_SC_NCK_ROU_S[1]` (screen notation [1] of a 1D), `SGUD:POS[0-2]` (the 3 elements [0]–[2] of a 1D), `SGUD:_SC_C97[0,1](4)` (screen notation [0,1] of a 4-column 2D)

**type** (inside the envelope) — the element's actual type:

- `BOOL` — true/false
- `CHAR` — character code (0–255 integer)
- `INT` — integer
- `REAL` — real number (the same 64-bit real as Fanuc R/macro variables)
- `STRING` — string

(Structured GUD `AXIS` / `FRAME` are planned for later)

**data** (inside the envelope) — if one element (scalar / `[i]` / `[r,c](column count)`), a single value; if an `[i-j]` range, a JSON array:

- scalar/single: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- range: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**Write (POST)** — put the **same object** as the read into the body's `value` (e.g. `{"value":{"type":"REAL","data":42.0}}` → changes only the single cell at screen notation [i]). Since `type` decides the type to write, you write directly without reading first. The number of `data` elements must exactly match the range size (1 if single).

**Note** — on older NCKs without a GUD area, an unsupported error is returned.

## /machine/channel/userData/userDataValue
Reads/writes Siemens **channel GUD (per-channel SGUD)** user variables (currently **OPC-UA (Siemens) only**). Both the `channel` and `userData` filters are required, and only variables of the channel named by `channel` are addressed — NC-wide shared variables are not reachable through this address.

The return type is `object` — GUD variables differ in type, so the value arrives in a **self-describing envelope** `{"type":..,"data":..}` that also tells you what type it is.

**userData** — the `SGUD:<name>` or `SGUD:<name>[<index>]` format. **Indices are exactly as shown on the machine screen (HMI)**, 0-based (converted to the internal numbering automatically).

- no index → a scalar variable
- `[i]` → one element of a 1D array
- `[i-j]` → the i..j range of a 1D array (both ends inclusive)
- `[r,c](column count)` → one (row, column) element of a 2D array. Write the brackets exactly as shown on screen and append only the array's **column count** in parentheses (the machine does not report it, so you must supply it)
- Example: `?channel=1&userData=SGUD:_SC_C97[0,1](4)` (channel 1, screen notation [0,1] of a 4-column 2D)

**type** — `BOOL` (true/false) · `CHAR` (character code 0-255) · `INT` (integer) · `REAL` (64-bit float) · `STRING`. The structured `AXIS`/`FRAME` types are planned.

**data** — one value for a single element (scalar / `[i]` / `[r,c](column count)`), a JSON array for an `[i-j]` range.

- single: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- range: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**Writing (POST)** — put the **same object** as the read returns into the body's `value` (e.g. `{"value":{"type":"REAL","data":42.0}}`). `type` decides the type written, so no read is needed first. The number of `data` elements must match the range size exactly (1 for a single element).

**Note** — on older NCKs without a GUD area, an unsupported error is returned.

## /machine/toolArea/toolCount
The **number of tools registered** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `int`, read-only. It is `0` when no tool is registered.

It equals the number of elements `toolList` returns and reads **the same value from the machine** — it exists so that you need not fetch the whole list when only the count matters. Measured on a machine with 17 tools it is far cheaper than the list (81 ms vs 684 ms).

A nonexistent tool area is rejected with `-18`.

**Siemens only** — without tool management Fanuc has no "tool" object at all. The number of compensation registers is a different quantity and cannot stand in for it.

## /machine/toolArea/toolList
The list of **every tool registered** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `objectArray`, or an empty array `[]` when no tool is registered.

Element: `{"toolNumber": 16, "toolName": "BALLNOSE_D8", "toolEdgeCount": 4, "sisterToolNumber": 9, "magazineNumber": 0, "pocketNumber": 0, "toolLocationType": "buffer"}`

Tool numbers are **sparse** — 17 tools may occupy numbers 2 through 18 with no number 1, so trying numbers from 1 upward tells you nothing about what exists. This list is the answer: take an element's `toolNumber` and put it straight into the `tool` filter to query the per-tool addresses.

**The order is ascending `toolNumber`** — the same order on every read. **It is not the order on the machine's screen**: the tool list screen is usually sorted by name and the operator can change the sort, so there is no single "screen order" to match. To display it the way the screen does, sort by `toolName`.

**`toolNumber` is not the `Loc.` (place number) on the machine's screen.** A machine using tool management identifies tools by name and sister number, so this number does not appear in the list screen (it is the `Tool number` field in the tool detail screen). Putting the screen's `Loc.` into the `tool` filter **queries a different tool and still looks successful** — the two numbers coincide for many tools, which makes it hard to notice. That value is `pocketNumber`, and this list carries both so you can check the correspondence.

- **toolNumber** — the tool number; the value to put in the `tool` filter
- **toolName** — the tool's name; an empty string on machines that do not use names
- **toolEdgeCount** — the number of cutting edges, not of teeth; the upper limit of the `toolEdge` filter
- **sisterToolNumber** — the sister-tool number (the sequence among tools sharing one name)
- **magazineNumber** — the magazine (tool store) it currently sits in; `0` when outside a magazine
- **pocketNumber** — the pocket inside that magazine; `0` when outside a magazine
- **toolLocationType** — the kind of place: `"magazine"` (in a magazine), `"buffer"` (spindle or tool changer), `"loading"` (load/unload position), `"none"` (no physical place)

The list answers **what exists, what to call it, and where it is**. Measured values such as offsets and wear are per-edge and are not included.

When a value is unavailable the key is not dropped, it is `null` (the three location fields are the exception — they carry the same values as the identically named single addresses). The list is sorted in **ascending tool number**, so reading it twice and comparing is meaningful. A nonexistent tool area is rejected with `-18`.

The first four fields rarely change, but **the three location fields change whenever a tool moves.** This list is meant to be fetched once to draw a screen; if all you need is which tool is in the spindle, use `/machine/channel/activeToolNumber` rather than polling the list.

**Siemens only** — on Fanuc the offset table is filled densely from `1`, so there is nothing to enumerate.

## /machine/toolArea/tool/toolName
The tool's name (Sinumerik `toolIdent`). `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses).

Returns `string`; both read and write are supported — write `{"value": "DRILL 10"}`. **Siemens only.** On machines that use tool management, a tool's identity is its name plus its sister-tool (duplo) number, so several tools may share one name. On machines that do not use names, an empty string is normal.

A nonexistent tool is rejected with `-18` — this address does not create tools. Length and character limits are the machine's to judge, and violations surface as an error.

## /machine/toolArea/tool/sisterToolNumber
The **sister-tool number** (Sinumerik `duploNo`). It is the sequence among tools that share one name, deciding which one is brought in once the preceding tool's life runs out. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses).

Returns `int`; both read and write are supported — write `{"value": 2}`. **Siemens only.** On machines that use tool management, a tool's identity is its name plus this number, so tools sharing a name are told apart by it.

A nonexistent tool is rejected with `-18`. A value that is not an integer, or outside `0`~`65535`, gives `-16` — the effective upper limit comes from the machine configuration, and values outside that narrower range are rejected by the machine.

## /machine/toolArea/tool/magazineNumber
The number of the **magazine (tool store)** the tool currently sits in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

When the tool is not in a magazine the value is `0` — it is cutting in the spindle, being carried by the changer, sitting at the load/unload position, or registered as data with no physical place. The machine gives such places its own numbers too (e.g. `9998`), but deemesh does not emit those and folds them to `0` — they vary with the machine configuration, so consumers cannot rely on them.

**This is where the tool is now, not where it belongs.** Once the tool is loaded into the spindle the value becomes `0`, and it gets a number again when it returns to the magazine. Which place it originally came from is not what this address answers.

**Writing is not supported.** Changing this value would not move the tool; it would only change the bookkeeping, and the changer would then reach for the wrong pocket at the next tool change.

**Siemens only.** A machine without tool management has no magazine at all, so the value is always `0`.

## /machine/toolArea/tool/pocketNumber
The number of the **pocket inside the magazine** the tool sits in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

If the magazine number is the apartment building, this is the unit number — both are needed to pin down a location. When the tool is not in a magazine the value is `0` (cutting in the spindle, being carried by the changer, at the load/unload position, or with no physical place). Numbers the machine gives to such places (e.g. `9998`) are not emitted.

**A turret's stations appear as pockets too.** The machine models turret positions as magazine pockets, so the position a machinist calls "station 3" is pocket `3` here.

**This is where the tool is now, not where it belongs.** Once the tool is loaded into the spindle the value becomes `0`, and it gets a place number again when it returns to the magazine.

**Writing is not supported.** Changing this value would not move the tool; it would only change the bookkeeping, and the changer would then reach for the wrong pocket at the next tool change.

**Siemens only.** A machine without tool management has no magazine at all, so the value is always `0`.

## /machine/toolArea/tool/toolLocationType
**What kind of place** the tool is in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `string`, **read-only**. The value is its own meaning, so no separate code table is needed.

| Value | Meaning |
|---|---|
| `"magazine"` | sitting in a magazine (tool store) |
| `"buffer"` | in the spindle or the tool changer — cutting or being moved |
| `"loading"` | at the load/unload position |
| `"none"` | no physical place — only the tool data is registered |

**These four stay the same no matter how many machine types are added.** The machine gives the spindle, the changer and the load/unload position their own magazine numbers too (e.g. `9998`), but those vary with the machine configuration, so consumers cannot rely on them. deemesh folds them into these four, so you can branch on the value without knowing which machine it is attached to.

`"buffer"` does **not** distinguish the spindle from a changer gripper — the machine treats both as the same place. Which tool is in the spindle is answered by `/machine/channel/activeToolNumber`, and that address is supported on every machine type, so overlaying the two tells them apart.

**Writing is not supported.** Moving a tool is the job of a tool move, and changing this value alone would put the bookkeeping out of step with reality.

**Siemens only.** A machine without tool management has no magazine at all, so the value is always `"none"`.

## /machine/toolArea/tool/toolEdgeCount
The number of **cutting edges (offset data sets)** the tool has. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). **Fanuc has no per-tool edge concept, so it is fixed at `1`**; Siemens uses `numCuttEdges`.

**This is a count, not the highest number.** Edges normally run from `1` upwards, but deleting a middle edge at the machine panel **leaves a gap and does not renumber the ones after it** — delete `2` out of `1`, `2`, `3` and what remains is `1` and `3` while this value becomes `2`. So do not assume `toolEdge` runs from `1` to this value.

A nonexistent edge is rejected with `-18` on both read and write, so reading tells you which numbers are real.

**This is not the number of teeth.** The flutes or inserts a cutter physically carries — "a 2-flute ball nose", "a 4-flute end mill" — are a different quantity. When several teeth sit at the same length and radius one edge covers them all; measured here, a 4-flute cutter answered `1` while a 2-flute ball nose answered `3`.

On Siemens a nonexistent tool is rejected with `-18` — it does not answer that the tool has `0` edges.

## /machine/toolArea/magazineList
The **list of magazines** in that tool area. `toolArea` filter. Returns `objectArray`, **read-only**.

**Magazine numbers are not contiguous, so there is no other way to learn them** — measured here they were `1`, `9998` and `9999`. Counting from `1` up to `/machine/toolArea/magazineCount` will not find them.

Each entry:

| Field | Meaning |
|---|---|
| `magazineNumber` | the magazine number; pass it straight to the `magazine` filter of `/machine/toolArea/magazine/pocketCount` |
| `pocketCount` | how many pockets that magazine holds |

**Only real tool stores are listed.** Internally the spindle, the changer and the load/unload position are magazines too and carry high numbers (`9998`, `9999` and the like), but in deemesh "magazine" means a tool store and nothing else — when a tool sits in one of those places, `/machine/toolArea/tool/toolLocationType` answers `"buffer"` / `"loading"` and `/machine/toolArea/tool/magazineNumber` gives `0`.

**It joins directly with tools.** Look up the entry by whatever `/machine/toolArea/tool/magazineNumber` returned, and pass that number straight to `pocketCount`.

**Magazine names are not included.** The machine has a name field, but it means nothing unless it is configured on site — on the machine measured here the 40-pocket magazine, the buffer and the load position all returned **the same string**. Three entries with identical names would make the list look broken, so it is left out.

**Siemens only.**

## /machine/toolArea/magazineCount
The **number of magazines** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

**Only real tool stores are counted.** Internally the spindle, the changer and the load/unload position are magazines too (counted that way the machine measured here answers `3`), but deemesh does not treat those places as magazines — when a tool sits in one, `/machine/toolArea/tool/toolLocationType` answers `"buffer"` / `"loading"`.

**It gives you the count but not the numbers.** The numbers are not contiguous, so you cannot infer valid magazine numbers from this value — use `/machine/toolArea/magazineList` when you need the numbers. This address exists so you do not have to fetch the whole list just to get a count.

**Siemens only.** A machine without tool management has no magazine at all, so the value is `0`.

## /machine/toolArea/magazine/pocketCount
The number of **places (pockets)** in that magazine. `toolArea` + `magazine` filters (`magazine` is the magazine number). Returns `int`, **read-only**.

**Magazine numbers are not contiguous** — `/machine/toolArea/magazineList` tells you the valid ones. Counting up from `1` to `/machine/toolArea/magazineCount` will not find them.

A number that does not exist is rejected with `-18`. **The numbers of the spindle, the changer and the load/unload position are rejected too** — the machine gives those places magazine numbers, but deemesh does not treat them as magazines. Range and comma expansion are supported.

**Siemens only.**

## /machine/toolArea/magazine/pocketList
**Every pocket in that magazine and what sits in each one.** `toolArea` + `magazine` filters. Returns `objectArray`, **read-only**.

Each entry:

| Field | Meaning |
|---|---|
| `pocketNumber` | the pocket number; every one from `1` to `/machine/toolArea/magazine/pocketCount` appears |
| `toolNumber` | the tool in that pocket. **`0` means the pocket is empty** |

**This is the direction the other addresses cannot answer.** `/machine/toolArea/tool/pocketNumber` tells you which pocket a tool is in, but "what is in pocket N" and "where are the empty pockets" are answered only by this list.

Tool names and offsets are not included — `/machine/toolArea/toolList` gives those by number, so join on the number. Reading a name for every pocket would double the values fetched for a 40-pocket magazine.

The numbers of the buffer (spindle and changer) and the load/unload position are rejected with `-18` — deemesh does not treat those places as magazines.

**Siemens only.**

## /machine/toolArea/magazine/pocket/toolNumber
The **tool number** in that pocket. `toolArea` + `magazine` + `pocket` filters. Returns `int`, **read-only**.

**`0` means the pocket is empty** (tool numbers start at `1`). A pocket that does not exist is rejected with `-18` rather than answering `0`, so the two never blur together — the valid range is given by `/machine/toolArea/magazine/pocketCount`.

Use it for a single pocket; to sweep a whole magazine, `/machine/toolArea/magazine/pocketList` does it in one round trip.

The numbers of the buffer (spindle and changer) and the load/unload position are rejected with `-18`.

**Writing is not supported.** Overwriting the tool in a pocket would change the bookkeeping while the physical tool stayed put, and the changer would then reach for the wrong pocket at the next tool change. Moving a tool is the job of a magazine command, and this protocol does not expose one.

**Siemens only.**

## /machine/toolArea/tool/toolExists
Whether that tool number is **registered in the tool table**. `toolArea` + `tool` filters. Returns `boolean`; both read and write are supported.

Asking about a tool that does not exist is not an error — it answers `false`, because the address exists to ask that question.

**Writing creates and deletes the tool.** `{"value": true}` creates it, `{"value": false}` deletes it. If it is already in that state, nothing happens and the write succeeds.

A newly created tool is **empty and has one cutting edge**, and its name is the tool number as a string. Follow up with `/machine/toolArea/tool/toolName` for the name and `/machine/toolArea/tool/toolEdge/*` for the offsets; to add more edges use `/machine/toolArea/tool/toolEdge/toolEdgeExists`. This is the flow that registers presetter measurements without going to the machine panel.

**A tool that occupies a magazine pocket cannot be deleted** (`-18`). The physical tool would stay in the magazine while its record disappeared, throwing off the next tool change, and there would be **no way to recreate the tool and put it back in that pocket.** Unload it at the machine first — where it is right now is answered by `/machine/toolArea/tool/toolLocationType`.

**Siemens only.**

## /machine/toolArea/tool/toolEdge/toolEdgeExists
Whether that edge number **exists on that tool**. `toolArea` + `tool` + `toolEdge` filters. Returns `boolean`; both read and write are supported.

Asking about an edge that does not exist is not an error — it answers `false`. Since `/machine/toolArea/tool/toolEdgeCount` gives only a count and the numbers can have gaps, **this address is what tells you which numbers are real.**

**Writing creates and deletes the edge.** `{"value": true}` creates it, `{"value": false}` deletes it. If it is already in that state, nothing happens and the write succeeds.

**Edges are only created in order.** The machine does not let you pick the number — it fills the **lowest free one**. So asking for any other number creates nothing and is rejected with `-18`, and the error text carries the number that would be created next. If you need `D5`, create `3`, `4` and `5` in turn. Where there is a gap, that gap is filled first.

**The first cutting edge cannot be deleted** (`-18`). It stays as long as the tool does — to remove the whole tool use `/machine/toolArea/tool/toolExists`.

Creating an edge on a tool that does not exist is `-18`.

**Siemens only.**

## /machine/toolArea/tool/toolDisabledOn
Whether the tool is **marked as not to be used**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `boolean`, **read-only**.

When `true` the control will not use that tool — it rejects a program that calls for it, or switches to the sister tool if one is registered (`/machine/toolArea/tool/sisterToolNumber`).

**This is per tool** — it takes no `toolEdge` filter. A tool with several offset data sets is locked as a whole.

Two paths raise the mark, but the **value does not distinguish them**: an operator locking the tool at the machine panel, and the control locking it automatically when tool life monitoring runs the remaining life out. Either way it means "a tool you cannot use right now", so there is no reason to tell them apart.

Specifying a nonexistent tool is rejected with `-18`.

Writing is supported too — `{"value": true}` locks the tool and `{"value": false}` releases it, so you can take a tool out of service or bring it back without going to the machine panel.

**A tool locked because its life ran out will lock again shortly** if you only clear the lock, since the remaining life is still `0`. In that case write the remaining life back via `/machine/toolArea/tool/toolEdge/toolLifeRemaining`. Clearing the lock without changing the insert also means cutting with a worn-out edge.

**Siemens only.**

## /machine/toolArea/tool/toolFixedLocationOn
Whether the tool is **assigned to a fixed location** — it always returns to the same pocket. `toolArea` + `tool` filters. Returns `boolean`; both read and write are supported — `{"value": true}` assigns it, `false` releases it.

When `true` the tool goes back to its own pocket after a tool change; when `false` the machine picks a free pocket. This is the `L` column on the machine's magazine screen.

If it is already in that state, nothing happens and the write succeeds. A nonexistent tool is rejected with `-18`.

**Siemens only.**

## /machine/toolArea/tool/toolOversizedOn
Whether the tool is **bigger than one pocket** — a wide tool that needs the neighbouring pockets kept free. `toolArea` + `tool` filters. Returns `boolean`, **read-only**.

This is the `Z` column on the machine's magazine screen. It answers only **whether** the tool exceeds one pocket, not how many it occupies.

**Writing is not supported.** The machine has no dedicated oversized flag — how many pockets the tool takes up to the left, right, above and below is stored separately, so a bare `true` could not decide which direction and how far. Set oversize at the machine panel.

A nonexistent tool is rejected with `-18`.

**Siemens only.**

## /machine/toolArea/magazine/pocket/pocketDisabledOn
Whether the pocket is **marked as not to be used** — damaged, or a place that must stay empty. `toolArea` + `magazine` + `pocket` filters. Returns `boolean`; both read and write are supported — `{"value": true}` disables it, `false` enables it.

When `true` the machine skips this pocket when choosing where to put a tool. This is the `D` column on the machine's magazine screen, and on that screen the cell appears **only on rows for tools that occupy a pocket** — because it is a property of the pocket, not of the tool.

Locking a tool is `/machine/toolArea/tool/toolDisabledOn`, which is separate. A disabled pocket does not disable the tool sitting in it; move that tool elsewhere and it is usable again.

If it is already in that state, nothing happens and the write succeeds. A nonexistent pocket, and the numbers of the buffer and load/unload positions, are rejected with `-18`.

**Siemens only.**

## /machine/toolArea/tool/toolMonitorType
How the tool's life is **monitored**. `toolArea` + `tool` filters. Returns `int`; both read and write are supported — write `{"value": 2}`.

| Value | Meaning | Unit of the life values |
|---|---|---|
| `0` | no monitoring | — |
| `1` | time — counts how long the tool actually cut | seconds |
| `2` | count — counts finished workpieces | pieces |
| `3` | wear — watches whether the offset has drifted to its limit | machine setting (mm/inch) |

**These four stay the same no matter how many machine types are added** — they are deemesh's own values, not the machine's, so you can branch on them without knowing which machine you are attached to.

The **tool picks one method, and the values are per cutting edge**. That is why this address takes no `toolEdge` filter while the three life values do.

When it is `0` the three life values are rejected with `-18` — there is nothing to measure on that tool. To switch monitoring on, write the method here first, then put a budget into `/machine/toolArea/tool/toolEdge/toolLifeTotal`.

A machine may have several methods on **at once**. In that case this address answers with the first of time → count → wear, and the three life values follow the same order, so the method and the values never disagree. The answers to "does this need replacing" — `/machine/toolArea/tool/toolLifeWarnOn` and `/machine/toolArea/tool/toolDisabledOn` — are always exact regardless of method.

**Siemens only.**

## /machine/toolArea/tool/toolLifeWarnOn
Whether the tool has **reached its warning limit**. `toolArea` + `tool` filters. Returns `boolean`, **read-only**.

When `true` the remaining life has dropped below the warning limit — **the tool is still usable.** Becoming unusable is `/machine/toolArea/tool/toolDisabledOn`; the two are independent and can both be `true` (a tool locked because its life ran out).

Use it to pick out the tools that will need replacing soon. How much is left is answered by `/machine/toolArea/tool/toolEdge/toolLifeRemaining`.

**The life values are per cutting edge while this flag is per tool.** Measuring and acting happen at different levels — an edge does the cutting, but replacement takes the whole tool, and the sister tool the control switches to (`/machine/toolArea/tool/sisterToolNumber`) is per tool as well. So **any** edge crossing its own limit makes this `true`.

**It does not tell you which edge crossed.** If you need that, compare `toolLifeRemaining` against `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit` for each edge — `/machine/toolArea/tool/toolEdgeCount` gives you how many there are.

**The control raises this value, so writing is not supported.** Overwriting it would leave the remaining life untouched, and the next evaluation would put it back.

With monitoring off it is always `false`. A nonexistent tool is rejected with `-18`.

**Siemens only.**

## /machine/toolArea/tool/toolEdge/toolLifeTotal
The **whole life budget** allotted to that cutting edge. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported — write `{"value": 6}`.

The control starts the remaining life at this value and counts down. The unit is decided by the monitoring method and is carried in the response's `unit` field — see `/machine/toolArea/tool/toolMonitorType`. With monitoring off the request is rejected with `-18`.

When counting pieces, **only whole numbers are accepted.** A fractional value makes the machine answer success while leaving the value unchanged, so the SDK rejects it first.

**Siemens only.**

## /machine/toolArea/tool/toolEdge/toolLifeRemaining
The life **left** on that cutting edge. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported — write `{"value": 6}`.

Sinumerik counts down, so this value is the edge's life right now. How much has been used is `toolLifeTotal` minus this value.

**This is the address you write after changing an insert to reset the life** — usually to the same value as `toolLifeTotal`. If the tool was locked because its life ran out, this write is the correct way to release it.

The unit is decided by the monitoring method and is carried in the response's `unit` field. With monitoring off the request is rejected with `-18`; a fractional value while counting pieces is rejected with `-16`.

**Siemens only.**

## /machine/toolArea/tool/toolEdge/toolLifeWarnLimit
The **warning limit** — once the remaining life drops below this value the control raises a warning (`/machine/toolArea/tool/toolLifeWarnOn`, which is **per tool**). `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported — write `{"value": 3}`.

It buys time to prepare a replacement tool, so it is set lower than `toolLifeTotal`. The unit is decided by the monitoring method and is carried in the response's `unit` field. With monitoring off the request is rejected with `-18`; a fractional value while counting pieces is rejected with `-16`.

**Siemens only.**

## /machine/toolArea/tool/toolEdge/toolLengthGeometry
The **length1 geometry** value (Sinumerik `DP3`). For turning tools it usually corresponds to the X direction, but the axis correspondence is a rule set by the tool type and active plane, so the SDK does not translate it.

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolLengthWear
The **length1 wear** value (Sinumerik `DP12`).

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolLength2Geometry
The **length2 geometry** value (Sinumerik `DP4`). For turning tools it usually corresponds to the Z direction.

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolLength2Wear
The **length2 wear** value (Sinumerik `DP13`).

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolLength3Geometry
The **length3 geometry** value (Sinumerik `DP5`).

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolLength3Wear
The **length3 wear** value (Sinumerik `DP14`).

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolRadiusGeometry
The **cutter radius geometry** value (Sinumerik `DP6`, from the milling-tool viewpoint). It points at the **same storage** as `toolNoseRadiusGeometry`; which address you use is your declaration of intent — the SDK does not inspect the tool type.

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.


**The number can differ from what the machine's screen shows.** This value is a **radius**, as the name says, while tool-list and offset screens commonly display the **diameter (Ø)** — measured (2026-07): `BALLNOSE_D8` stores `4.0` and the HMI shows `8.000`. deemesh emits what the machine stores and does not multiply by two.

## /machine/toolArea/tool/toolEdge/toolRadiusWear
The **cutter radius wear** value (Sinumerik `DP15`). Same storage as `toolNoseRadiusWear`.

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.


**The number can differ from what the machine's screen shows.** This value is a **radius**, as the name says, while tool-list and offset screens commonly display the **diameter (Ø)** — measured (2026-07): `BALLNOSE_D8` stores `4.0` and the HMI shows `8.000`. deemesh emits what the machine stores and does not multiply by two.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusGeometry
The **nose radius geometry** value (Sinumerik `DP6`, from the turning-tool viewpoint). Same storage as `toolRadiusGeometry`.

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusWear
The **nose radius wear** value (Sinumerik `DP15`). Same storage as `toolRadiusWear`.

Returns `float`; both read and write are supported — write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gGroup/gModal?gGroup=4` to find out which — `G21`/`G710` means metric, `G20`/`G700` means inch. This address carries no `unit` field, because the unit is not fixed per address.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolTipDirection
The **tool-tip position code** (Sinumerik `DP2`, `1`-`9`). Indicates where the tip lies relative to the nose center during nose-radius compensation; it is a position code, not an angle. Fanuc's tool offset tree has the same concept (`0`-`9`), and **the numbering and `desc` vocabulary are shared**, so values can be compared and reused as-is across machine types (`desc` is attached to `9` only — the manual defines the `1`-`8` orientations by diagram alone, in three variants by machining setup, so they are not shipped). The valid range is `1`-`9` and **`0` is not permitted** — *"The identifier 0 (zero) is not permitted as a cutting edge position"* (SINUMERIK 828D Tools Function Manual).

Returns `int`; both read and write are supported — write `{"value": 3}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolType
The tool's **type code** (read + write, `int` + `desc`). **Uses the Sinumerik DP1 code as-is** — the code scheme is an open classification owned by Siemens, so deemesh does not translate it, and the authority is the Sinumerik tool-management manual (even if the vendor adds codes, the value is passed through as-is). Writes take an integer code `{"value": 500}` — for tool-setup automation, and the NCK judges code validity.

| Family | Meaning | Examples |
|---|---|---|
| `1xx` | milling tools | `120` end mill, `140` face mill, `145` thread milling |
| `2xx` | drill family | `200` twist drill, `240` tap, `250` reamer |
| `4xx` | grinding tools | |
| `5xx` | turning tools | `500` roughing, `510` finishing, `530` cutoff, `540` threading |
| `7xx` | special | `711` probe, `730` stop |

Known codes come with their meaning in `desc` (`{"value": 500, "desc": "turning roughing tool"}`), and unregistered codes fall back to the first-digit family desc. It is also the reference value that determines the length1/2 axis assignment and radius interpretation (cutter/nose) for turning tools (5xx).

**Write caution** — a nonexistent tool, or an edge that tool does not have, is rejected with `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an edge removable only at the operator panel, so deemesh allows **modifying existing edges only**.

## /machine/toolArea/tool/toolEdge/toolTipAngle
The **tip angle** of that cutting edge — the point angle of a drill (`118.0`), `90.0` for a centre drill, and so on. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported — write `{"value": 118.0}`.

**It is not `toolTipDirection`.** The names differ by one word but the values do not match up — that one is a code for **which direction** the tip sits relative to the nose centre, while this one is the **angle** of the tip.

**Tools that carry no angle report `0.0`.** Milling tools are such tools — measured here, a drill gave `118.0` and a face mill `0.0`.

**The `N` column on the machine's screen doubles as this value and `toolTeethCount`.** It shows the angle for drills and the tooth count for milling tools in that one cell. deemesh keeps them apart so that one address never means a different physical quantity depending on the tool type — to find the number from the screen, look at whichever of the two carries a value.

**Siemens only.** A nonexistent tool or edge (D) is rejected with `-18`.

## /machine/toolArea/tool/toolEdge/toolHNumber
The **`H` number** assigned to that cutting edge (for the ISO dialect). `toolArea` + `tool` + `toolEdge` filters. Returns `int`; both read and write are supported — write `{"value": 5}`.

When machining with the ISO dialect (Fanuc-style programs), an `H5` in the program applies the compensation of whichever edge carries the number `5`, **independently of the tool**. The tool is selected with `T` and the compensation separately with `H`, and this address manages that number.

`0` means **not assigned**. The machine's tool list screen has an `H` column showing the same value.

Numbers are meant to be unique, but **the machine enforces that** — writing a number already in use surfaces as an error. Only whole numbers are accepted; a negative value is rejected with `-16`.

On a machine that does not use the ISO dialect every edge is `0`. In the Siemens-native scheme the `D` number selects the edge within a tool, so this value is not needed.

**Siemens only.**

## /machine/toolArea/tool/toolEdge/toolTeethCount
The **number of teeth** of that cutting edge — the count you mean by "a 4-flute end mill". `toolArea` + `tool` + `toolEdge` filters. Returns `int`; both read and write are supported — write `{"value": 4}`.

**It is not `toolEdgeCount`.** That one counts the offset data sets the control holds for the tool; this one counts how many physical cutting teeth one such set describes. Measured here, a 4-flute cutter had `1` edge and `4` teeth.

**It is stored per edge.** A single body can carry cutting sections of different diameters, whose tooth counts may differ, so the value belongs to the edge rather than to the tool.

**It does not always match the `N` column on the machine's screen.** That column doubles up: it shows the tooth count for milling tools and the point angle for drills — measured, a drill returned `0` here while the screen showed `118.0` (the point angle). deemesh keeps the two apart so that one address never means a different physical quantity depending on the tool type.

**Siemens only.** A nonexistent tool or edge (D) is rejected with `-18`.

## /machine/channel/diagnosis/index/diagnosisValue
The value of **one row** of diagnosis data (**Fanuc only**, `float`, read-only). `channel=<channel>&diagnosis=<number>&index=<n>` — the same row model as the parameter corridor: for axis/spindle-dependent diagnoses `index` is the axis/spindle number, and **single-value diagnoses have exactly one row, so use `index=1`** (the same model as `diagnosisValueList`, which returns single values as one-element arrays). An out-of-range index is refused with `-18` carrying the actual row count. When polling just one axis periodically, this is lighter than reading every row via the List (one call).

## /machine/channel/diagnosis/diagnosisValueList
The **value** of an arbitrary diagnosis number (**Fanuc only**, `floatArray`). Axis/spindle-dependent diagnoses return an array of axis-count length; non-dependent diagnoses return a single-element array. Diagnosis data can differ per path (channel), so this address is **channel-scoped** — specify the path with `channel=`; path-common numbers read the same on any channel. Each diagnosis's format (whether it is row-indexed, and how many rows) is determined on the first query with vendor validation and cached per channel, so repeated polling is light. With comma/range expansion like `diagnosis=301,308`, it comes as a nested array with per-diagnosis boundaries preserved.

## /machine/channel/parameter/index/parameterValue
The value of **one row** of a CNC parameter (**Fanuc only**, `float`, read/write). Specify it with `channel=<channel>&parameter=<number>&index=<n>`, where the number is taken straight from the Fanuc parameter manual — it is **not translated and never will be unified** (a vendor-owned numbering scheme, the same class as the `diagnosis` filter, so no cross-vendor mapping can exist). Parameters can differ per path (channel), so this address is **channel-scoped**. Frequently used parameters are exposed as named addresses such as `partCountActual` and `powerOnDuration`; this address is the general-purpose corridor for everything else.

**Every parameter is treated as an array of rows** — `index` is the row number: the axis number for axis-type parameters, the spindle number for spindle-type ones (same as the row order on the parameter screen, 1-based; e.g. X1=1, Y1=2), and **single-value parameters have exactly one row, so use `index=1`** (the same model as `parameterValueList`, which returns single values as one-element arrays). An out-of-range index is refused with `-18` carrying the actual row count.

- Bit-type parameters travel as the **whole byte** (a packed integer) — decomposing/composing bits is the caller's job. To change a single bit, do a read-modify-write (which can race with concurrent changes from the operator panel).
- Decimal (real) parameters travel as real numbers with the machine's decimal places applied, and writes are stored with the same number of places.
- Writing an out-of-range value to an integer parameter is refused with `-16` (the allowed range is included in the error).
- **Write caution**: parameters change machine behavior. If the machine blocks parameter writes, the write is refused as a vendor error (`-17`), and some parameters require a power cycle after the change. The Fanuc library for Linux does not provide parameter write, so writes return `-20` there.

## /machine/channel/parameter/parameterValueList
The **all-axes value array** of a CNC parameter (**Fanuc only**, `floatArray`, read-only). The array length is the **row count** established by vendor validation — axis-type parameters get one element per axis, spindle-type per spindle, and non-axis parameters a one-element array (the same row model as `diagnosisValueList` and the sibling `parameterValue`). Whether a parameter is row-indexed, and how many rows it has, is determined on the first query with vendor validation and cached per channel, so repeated polling is light, and range/comma expansion such as `parameter=6711-6713` returns nested arrays with per-parameter boundaries preserved.

## /machine/ncMemoryPath/entry
Information about a single entry at the path (`object`). The key set is **always the same regardless of machine** — when a value is unavailable the key is not dropped, it is `null`.

| Key | Type | When unavailable |
|---|---|---|
| `name` | `string` | — |
| `sizeBytes` | `int` | `null` for a folder, or when the size could not be read |
| `modifiedAt` | `string` | `null` for a folder, or when the machine does not provide a modification time (always `null` on Siemens) |
| `isDir` | `boolean` | — |
| `comment` | `string` | `null` for a folder, or when the machine does not provide comments (always `null` on Siemens) |

A trailing `/` on the path forces a folder; without it files are searched first. Missing entries are an error.

## /machine/ncMemoryPath/entryList
The list of files and folders inside a folder (**read-only**, `objectArray`). Each element is **exactly the same object** as `entry` — same key set and same `null` convention, so see that table. Sorted folders first, then by name ascending. To create or delete a folder use `directoryExists`.

## /machine/ncMemoryPath/entryName
The **name of the entry** that `ncMemoryPath` points at — reading returns the last segment of the path (a pure string operation, no machine communication) and writing performs a **rename**. Writes take `{"value": "new name"}` — path separators are not allowed, common to files/folders. It refers to the same "entry" as `entry`/`entryList`.

## /machine/ncMemoryPath/fileExists
Checks whether a **file** exists at the path (read) and declaratively writes the state (write):

- read → `true` if the file exists (`false` if only a folder of the same name exists)
- write `{"value": false}` → delete the file
- write `{"value": true}` → **error** — creating an empty file is unsupported. Create a file with its content via a `fileContent` write

A trailing `/` in the path is ignored (the kind is fixed by the address). For folders, use `directoryExists`.

## /machine/ncMemoryPath/directoryExists
Checks whether a **folder** exists at the path (read) and declaratively writes the state (write):

- read → `true` if the folder exists (`false` if only a file of the same name exists)
- write `{"value": true}` → create the folder
- write `{"value": false}` → delete the folder (**empty folders only** — an error if it has contents, no recursion)

A trailing `/` in the path is ignored. For files, use `fileExists`.

## /machine/ncMemoryPath/fileContent
Reads the **content** of an NC file (download) and writes it (upload — creates if absent, overwrites if present). The value is a string (program text).

- **Fanuc write auto-handling**: if `%` is absent, it is inserted automatically; if there is no leading O number/`<name>`, it is inserted automatically based on the path's file name. The saved file name is **based on the O number/name in the content**
- To delete a file, write `false` to `fileExists`

## /machine/plcAddress/plcType/plcValue
Reads/writes a **single element of PMC/PLC memory** (Fanuc FOCAS2 `pmc_rdpmcrng`/`pmc_wrpmcrng`, Siemens OPC-UA `/Plc/` node). Both read (GET) and write (POST) are supported; the return type is `float` (one value), and writes are also a single number (e.g. `{"value": 42}`). This address handles **a single element only**; to work with several elements at once, use the list-form address in the same tree. Both the `plcAddress` and `plcType` filters are required.

**plcAddress** — the address format **differs by machine**. Unlike `plcType` this is a **deliberate exception that is not normalized** — Fanuc's `D100` and Siemens's `DB99:DBB[5]` point into different memory architectures, and the table that maps one to the other is not SDK knowledge but site configuration that depends on how that machine's ladder was written. It will not be unified later either, so if you need to read the same signal across machines, **the host application must keep a per-machine address table**.

- **Fanuc**: the first character is the PMC area, the rest is the byte number (e.g. `R5`, `D100`). Specify a range with `~` — but for this (single) address the range must be **exactly one `plcType` size** (e.g. for word, `D100~D101`)
- **Fanuc** PMC area first characters: `G` `F` `Y` `X` `A` `R` `T` `K` `C` `D` `M` `N` `E` `Z` — the range must be within the same area (`D100~D101` OK, `D100~R101` NG)
- **Siemens**: the Sinumerik OPC-UA PLC-variable syntax as-is (e.g. `DB99:DBB[5]`) — the value is passed to the `/Plc/{address}` node. If `:` is omitted it is inserted automatically; if the `[..]` subscript is omitted `[1]` is attached automatically. A single element only — for multiple elements it errors and points you to the list-form address

**plcType** — a numeric code that decides how to interpret the raw bytes. It is a **machine-independent unified value**, so any vendor uses the same number (the adapter translates it to each vendor's code):

- `1` = bit — 1 bit (0 / 1)
- `2` = byte — 8-bit integer (unsigned, 0–255) · address width 1 (e.g. `D100`)
- `3` = word — 16-bit integer (signed) · address width 2 (e.g. `D100~D101`)
- `4` = dword — 32-bit integer (signed) · address width 4 (e.g. `D100~D103`)
- `5` = float32 — 32-bit real · address width 4 (e.g. `D100~D103`)
- `6` = float64 — 64-bit real · address width 8 (e.g. `D100~D107`)

`0` = **auto** — the source decides the type. Protocols where the node knows its type, like Siemens (OPC-UA), read with that native type. In contrast, raw-memory vendors like **Fanuc/Mitsubishi** have no intrinsic type, so `0` (auto) is an error and it must be specified. **Fanuc (FOCAS2)** also has no bit type, so `1` (bit) is unsupported too — specify one of `2` (byte)–`6` (float64).

**Important (Fanuc)** — the byte count of the `plcAddress` range must match the `plcType` size (e.g. `plcType=3` (word, 2 bytes) with a single `D100` address fails → specify `D100~D101`). `plcType` decides **only the interpretation**, and the result is returned as `float` (a JSON number).

**Siemens** has the type encoded in the address itself (`DBB`/`DBW`/`DBD`, etc.), so `plcType=0` (auto) is recommended — putting in `1`–`6` behaves the same (it reads with the type the server reports). Writes read the node first to confirm the server type, then write with the same type.

**Error codes** — a `plcType` the machine cannot use returns `-18` (invalid filter value). A value outside the spec (other than `0`~`6`) returns the same `-18`, and both call for the same fix: pick another `plcType`. It is not `-20` because **the address itself works on that machine** — `-20` is reserved for "this address cannot be used on this machine". The error string carries the accepted values.

## /machine/plcAddress/plcType/plcValueList
Reads/writes a **block of PMC/PLC memory elements** as an array. The filters, address format, and `plcType` rules are the same as `plcValue` (single) above; the only difference is that it handles **multiple elements**. The return type is `floatArray`, and the write `value` is a number array `[1, 2, ...]` — even a single element must be written as an array like `[42]`.

- **Fanuc**: the range's byte count must be a **multiple** of the `plcType` size, and the element count = byte count ÷ type size (e.g. `D100~D107` + word = 4 → `[v1,v2,v3,v4]`)
- **Siemens**: multi-element subscripts are allowed — the elements the server gives become the array as-is
- Writes require the **element count to exactly match the target range/node's element count**

**Error codes** — a `plcType` the machine cannot use returns `-18` (invalid filter value). A value outside the spec (other than `0`~`6`) returns the same `-18`, and both call for the same fix: pick another `plcType`. It is not `-20` because **the address itself works on that machine** — `-20` is reserved for "this address cannot be used on this machine". The error string carries the accepted values.

## /machine/ncMemorySizeTotal
The total NC memory capacity. Returns `int` + `unit:"bytes"`.

All sizes in the SDK are in **bytes** — the same unit as `sizeBytes` in `entry`/`entryList`, so a question like "does this file fit in the free space" needs no conversion. When a machine only reports a coarser unit (KB, say), this address still returns bytes, and the value is then a multiple of that unit.

## /machine/ncMemorySizeUsed
The NC memory usage. Returns `int` + `unit:"bytes"`.

## /machine/ncMemorySizeFree
The free NC memory capacity. Returns `int` + `unit:"bytes"`.

## /machine/ncMemoryRootPath
The root path of the main NC memory — the starting point for paths to put in the `ncMemoryPath` filter. Fanuc is usually `//CNC_MEM`, Siemens `//NC`.

## /machine/ncMemoryExternalRootPathList
The list of **external storage drives** other than the main NC memory root (e.g. data server, memory card). Return type `stringArray`.

- Each item, like the root, has a leading `//` (no trailing slash) — Fanuc `//DATA`/`//MEMCARD`, Siemens `//Local drive`
- Names are the **machine HMI wording**. The Siemens local drive is called `NCExtend` internally in OPC-UA, but it is reported as `//Local drive` to match the operator panel (requests using the old `//NCExtend` are still accepted)
- The main root itself is excluded from this list
- **Not cached** — external devices can change by connect/disconnect, so it is re-queried on each request
- No filter
