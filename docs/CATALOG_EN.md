## /machine/channelCount
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The number of channels (paths) on the CNC. Cached at connection time, so it returns immediately with no additional communication.

## /machine/cncModel
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The CNC model string.

- **Fanuc**: series number string: `"15"`, `"16"`, `"18"`, `"21"`, `"30"`, `"31"`, `"32"`, `"35"`, `"0"` (0i), `"PD"`/`"PH"` (Power Mate i), `"PM"` (Power Motion i). `desc` also carries the series name (e.g. `"31"` → `Series 31i`)
- **Siemens**: the model name as-is (e.g. `"840D sl"`)
- **Mitsubishi**: the NC system S/W number and name string (vendor `GetVersion`). A control that does not provide it answers with status `-20`; that is the case on simulators, which carry no real hardware identity. No `desc`

## /machine/machineType
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The machine type. Returns a `string`, a self-describing enum, so no separate code table is needed. The full set of possible values:

- `"machiningCenter"`: machining center (Fanuc M/MM, Siemens M, Mitsubishi `…M` series)
- `"lathe"`: lathe (Fanuc T/TT/MT, Siemens T, Mitsubishi `…L` series)
- `"punchPress"`: punch press (Fanuc only)
- `"laser"`: laser (Fanuc only)
- `"wireCut"`: wire cut (Fanuc only)
- `"unknown"`: could not be determined (on Mitsubishi, the series that do not split into mill/lathe: C70, C80, C6/C64, and the PC-card types)

On Mitsubishi this value comes from the configured `system_type`. That is not the same as trusting the configuration: the vendor defines `…M` as a machining center system and `…L` as a lathe system, and **validates that distinction at connect time**: pointing a `…L` type at a mill is refused outright. So a successful connection is the control confirming this value.

## /machine/currentDateTime
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The machine's **current date/time**. Returns a `string`, ISO 8601 to the second (`"2026-07-11T14:30:00"`).

- **Machine-local clock**: since there is no timezone information, no TZ suffix (`Z`/`+09:00`) is appended. It is the ISO 8601 local-time form, and an RFC 3339 parser, which requires an offset, may reject it
- **Do not hand it straight to JavaScript's `new Date()`**: a date-and-time without an offset is interpreted in **the viewer's** time zone. This value is the **machine's** wall clock, not the viewer's
- This is the **CNC's clock**, not the server PC's clock; if the machine's clock is off, it is reflected as-is
- Fanuc: `cnc_gettimer` / Siemens: `sysTimeBCD` / Mitsubishi: `GetClockData`
- **Recommended health-check address**: a cheap read that triggers an actual NC round-trip on all protocols, so use it for polling and then judging `status` (`0` = normal, status `-10`/status `-14` = link problem) to monitor per-machine communication state. (Cache-served addresses like `machineType` may succeed even when the link is dead, making them unsuitable)

Time-related addresses are always ISO 8601 strings (`…At` = event moment, `…DateTime` = clock reading).

## /machine/powerOnDuration
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The machine's **cumulative power-on time**, an accumulating total that keeps counting even across power cycles. Returns `int` (seconds) + `unit:"s"`.

- **Fanuc**: parameter 6750 (**minute resolution**), so the value is always a multiple of
  60. Differential calculations (e.g. utilization) carry an inherent ±60 s error
- **Siemens**: `setupTime`, which reflects sub-minute resolution (converted to seconds). A normal power cycle does not reset it, but **powering the control up with default values sets it to `0`** (a rare service operation)
- **Mitsubishi**: `GetAliveTime` (**second resolution**). It is the `Power ON` item of the control's integrated-time screen (accumulated from NC power on to off), and **the control stops accumulating at `59999:59:59` and holds that value** (M800 Instruction Manual section 9.3.1). The EZSocket manual documents the value only as an 8-digit `HHHHMMSS` (up to `9999:59:59`), so what the API returns beyond 9999 hours (about 416 days) has not been confirmed. Once the cap is reached every difference reads `0`

Elapsed-time addresses are always **seconds-normalized int** (the `…Duration` suffix rule). **Anything below a second is discarded** - `59.9` seconds reads as `59`. That is how the control's own elapsed-time display works, and it never counts a second that has not passed yet. Every machine type and every `…Duration` address behaves the same way.

## /machine/configuredMachineName
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

Returns the `machine_name` from `config.json` (hub) or the `deemesh_create` configuration as-is. It is a **value from the configuration**, not a name the machine reports; hence `configured` in the address. Use it to confirm a connection reached the intended machine, or to label a response.

## /machine/configuredProtocol
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **protocol identifier** this connection uses: one of `"nc_focas2_fanuc"`, `"nc_opcua_siemens"`, `"nc_ezsocket_mitsubishi"`. No filters. Returns `string`, read-only. It is fixed at connection time, so once connected it answers immediately with no NC communication (while disconnected it returns status `-10` like any other address; the connection check comes first).

Like `configuredMachineName`, this is a **value from the configuration**, not something the machine reports. It returns the `protocol` field of `deemesh_create` (or of the machine entry in `config.json`) as-is, which is why the address says `configured`.

**Its purpose is narrow.** Most addresses are designed to hide the machine type, so no branching is needed. This value is for the **few places where the value space belongs to the machine type**: PLC address syntax (`D100` vs `DB10.DBB56`), diagnosis numbering, tool type codes, and the like, which the catalog explicitly marks as machine-dependent.

**Do not use it to decide whether something is supported.** "This machine type cannot do that address, so skip it" must be decided from status `-20`. Branching on this value means your code keeps skipping even after support is added for that machine type.

## /machine/channel/axisCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The channel's **user axis count**. Cached at connection time. The valid range of the `axis` filter is `1` to this value.

It counts geometry axes together with **non-spindle auxiliary axes** (indexing rotary tables, tailstocks, and the like), and excludes spindles; those are covered by `spindleCount` and the `spindle` filter.

**It differs per path.** On a multi-path control each channel has its own axis configuration, and a path with no axes reads `0`; the axis addresses on that channel then answer with status `-20`.

## /machine/channel/spindleCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The channel's spindle count. Cached at connection time. The valid range of the `spindle` filter is `1` to this value.

**On Fanuc and Siemens it differs per path.** A path with no spindle reads `0`, and the spindle addresses on that channel then answer with status `-20` - including `spindleOverride` and `spindleSpeedCommanded`, which are channel-wide values.

**On Mitsubishi it is the spindle count of the whole NC** (parameter `#1039 spinno`, a base common parameter), so every channel reports the same value. EZSocket has no call for a per-part-system spindle count and appears to number spindles NC-wide, so `spindle=1` up to this value addresses every spindle from any channel. This has not yet been checked on a multi-spindle, multi-part-system machine.

## /machine/channel/toolAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The tool area number the channel uses. The value to put in the `toolArea` filter of the tool tree addresses.

**Read this value and pass it straight into `toolArea`.** Do not pick a number yourself, because the numbering differs by machine.

On Siemens it is the NCK setting (`toNo`), so several channels may get the **same number**, which means those channels share their tools.

Fanuc and Mitsubishi have no separate tool-area layer; their tool data belongs to the **path (part system)**. The channel number therefore comes back unchanged. The tool addresses of these two machine types take no `channel` filter, so `toolArea` is what selects the path.

## /machine/channel/alarmStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **alarm severity**. Regardless of machine it returns **only the three values `0` / `1` / `2`**, and what is actually wrong on that machine comes alongside in `desc`.

| Value | Meaning |
|---|---|
| `0` | normal - no alarm and no message |
| `1` | warning - nothing is wrong with the machine (an informational message, a normal program stop, and so on) |
| `2` | alarm - **something is wrong with the machine** |

**Picture it as a signal light**: `0` green, `1` yellow, `2` red. The three values map straight onto a stack light or a status indicator on screen. That works because the scale matches **how the control itself colours its own messages**: the Mitsubishi manual paints NC alarms and PLC alarms on a red background, and warnings, stop codes and operator messages on a yellow one. So yellow coming on is not a fault notification but a sign that **the machine is waiting on something**.

💡 **The most robust use is `0` versus non-`0`.** `0` means exactly the same thing on every machine type ("no alarm and no message") and lines up with `alarmCount` being `0`. The boundary between `1` and `2`, by contrast, rests on **how that control classifies its own alarms**, so it can differ slightly between machine types. If you must branch on severity, read the per-item `severity` and text in `alarmList` alongside it.

⚠️ **`1` does not mean "something has gone wrong"; it shows up during perfectly normal machining, and how often depends on the machine type.** An untroubled automatic run (a dwell-only program) measured in the test environments of two machine types:

| | `alarmStatus` | what the list held |
|---|---|---|
| Fanuc | `0` | nothing |
| Mitsubishi | `1` | a stop code |

**Only Mitsubishi has a "stop code" channel.** It is where the control reports *the state of automatic operation*, and the vendor's own manual colours it yellow like a warning, not red like an alarm. What lands there is `T03` (single block switch ON), `T10` (waiting for an M-code completion), `T02` (an axis is at a soft limit) and the like: not faults, but "waiting on this right now". The one standing in the measurement above was `T10`, and it went away when the program ended. Fanuc and Siemens do not put such states in the alarm list, so they stay at `0`.

**So do not use `1` as a call-the-operator signal**: on a Mitsubishi machine it would be `1` during every normal automatic run. Use `2` for that, or decide from the `severity` and `category` of the entries in `alarmList`.

**The criterion is whether something is wrong with the machine, not whether machining stopped.** The two usually coincide but not always - a machine halted by `M0` or by single block is perfectly healthy, so it reports `1`. Use `/machine/channel/executionStatus` to find out whether machining is stopped right now.

- **The number stays these three no matter how many machine types are added.** Vendor codes are not passed through, so you can branch on `value` without knowing which machine it is attached to.
- **The cause comes in `desc`**: Fanuc gives the cause family (`{"value": 1, "desc": "Memory backup battery voltage low (CNC or Amplifier)"}`), Siemens gives the text of the heaviest alarm (`{"value": 2, "desc": "Emergency stop"}`), Mitsubishi gives the alarm kind (`{"value": 2, "desc": "NC alarm"}`). `desc` is a human-readable string, so **do not use it as a branch condition**; branch on `value`.
- **Mitsubishi**: **exactly the colour the control paints the message** - red (NC alarm, PLC alarm message) is `2`, yellow (NC warning, stop code, operator message) is `1`. The vendor API (`GetAlarm2`) delivers NC alarms and NC warnings as one kind, so **the category code restores the colour**: operation errors `M00`/`M01`, operation warnings `M50`, servo warnings `S52`/`S53` and smart-safety warnings `V50`-`V54` are yellow (`1`); every other NC alarm (`S01`-`S05`, `S51`, `Y`, `Z`, `Z7x`, `Z8x`, `EMG`, `L`, `U`, `N`, `P`, `V01`-`V07`, ...) and PLC alarms are red (`2`). An unknown category is `2`. So waiting states such as no operation mode (`M01 0101`) or override at zero (`M01 0102`) read `1`, while an emergency stop (`EMG`) reads `2`. Note that a soft stroke end is operation error `M01 0007` on Mitsubishi (yellow, `1`) but an OT alarm on Fanuc (`2`) - one situation the two controls grade differently, and this address follows each control's own grading
- Vendor codes whose severity is ambiguous are classified **conservatively as `2`**, because calling a stop a mere warning is more dangerous than the reverse. On Fanuc, a code the SDK newly emits and we do not know is classified as `2` as well. On Siemens the per-event severity the server reports is used directly: only an error (`1000`) is `2`, while anything the control itself calls a **warning (`500`) is `1`**. This is the **same criterion** as `alarmList`'s `severity`.
- For the alarm **list, numbers and messages** use `alarmList`; for the **count** alone use `alarmCount`. This address is the cheap summary meant for high-frequency polling (on Fanuc it costs one extra round trip to check for operator messages, and only that one more when you ask for it alongside the other status addresses).
- ⚠️ **`alarmStatus` can be non-zero while `alarmCount` is `0`.** On Fanuc the summary also reads the operator panel's **status line**, while the list carries only alarms and messages, so there are states that reach the summary alone (low battery, power-supply and insulation warnings, and the like). The opposite - `alarmStatus` of `0` while the list has entries - does not happen.
- **Siemens does not use the `channel` value** (same policy as `alarmList` and `alarmCount`). All three come from the same NCK-global snapshot, so one request answers them together and they cannot disagree. They cannot be split per channel because the alarm events carry no channel information - the alarm source is only `HMI`, `NCK` or `PLC`. On every machine type the `channel` value is range-validated.
- **Siemens also sees the machine builder's PLC alarms** (hydraulics, lubrication, door interlocks and the like). It used to read a node covering NCK alarms only, so such an alarm could be active while this address reported `0` (normal).

## /machine/channel/alarmCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **number** of active alarms/messages (= the count of `alarmList` items, regardless of severity). Returns `int`. Use it when you only need the count, like a dashboard badge.

- **Cost note (Fanuc)**: there is no count-only vendor API, so internally it costs the **same** as `alarmList`. Requesting both together in a batch merges them into a single fetch. If you only need a low-cost presence check, use `alarmStatus`
- **Cost note (Siemens)**: same as the two above; the count is taken from the **same event snapshot** as `alarmList`, so it costs the same, and requesting both together merges them into a single fetch. Being an NCK-global count, the `channel` value is ignored (same policy as `alarmList`)
- **Cost note (Mitsubishi)**: same as Fanuc; there is no count-only API, so it costs the same as `alarmList`, and requesting both together merges them into a single fetch

## /machine/channel/alarmList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The channel's **active alarms + operator/macro messages** list. Return type `objectArray`, or an empty array `[]` if none. For Siemens, alarms are NCK-global so the `channel` value is ignored.

Element: `{"code": "OH0700", "message": "SPINDLE OVERHEAT", "category": "Overheat", "severity": "alarm", "raisedAt": "2026-07-29T11:11:03Z"}`

The key set is **always the same regardless of machine**: when a value is unavailable the key is not dropped, it is `null` (same convention as `entry`). `severity` has only the two values `"alarm"` / `"warning"`.

- **code**: **the identifier exactly as the control's panel shows it, as a string** - Fanuc: alarm type letters plus the 4-digit number (`"OT0501"`, `"PS0010"`; operator/macro messages carry the message number, `"2000"`), Siemens: the alarm number (`"4230"`, `"700015"`; the number range tells the origin: `0`-`9999` general, `10000`-`19999` channel, `20000`-`29999` axis/spindle, `60000`-`69999` cycles, `100000`-`199999` HMI, `200000`-`299999` drive (SINAMICS), `300000`-`399999` drive and I/O, `400000`-`899999` PLC, of which `500000`-`899999` are defined by the machine builder; Diagnostics Manual section 2.3), Mitsubishi: class plus detail (`"M01 0101"`, `"S01 0051"`, `"EMG EXIN"`). Letters and leading zeros are significant, so **keep it as text and look it up as-is** in the manual. When no identifier can be obtained it is `""`, never `null` (in practice all three controls always fill it). Replaces the integer `number` as of 1.2.0
- **message**: display text
- **category**: on Fanuc, for alarms, the cause family (`Servo`, `Overheat`, `Spindle`, `PLC`, etc.: undefined types come as a numeric string); for messages, the source (`Operator message` = PMC/external input, `Macro message` = part-program #3006). Siemens: the source reported by the server (e.g. `NCU`: falls back to `Alarm` when empty). Mitsubishi: the alarm class as shown on the operator panel (`EMG`, `S01`, `M01`, etc.)
- **severity**: `"alarm"` = machine fault (machining not possible) / `"warning"` = informational (machining possible). Fanuc: among alarms only background-edit errors (BG) are warnings and the rest are alarms; operator/macro messages are all warnings. Siemens: translates the server's severity (1–1000) at a 500 boundary. Mitsubishi: what the control paints red (NC alarms, PLC alarms) is alarm, what it paints yellow (NC warnings `M00`/`M01`/`M50`/`S52`/`S53`/`V5x`, stop codes, operator messages) is warning - the same criterion as `alarmStatus`. Whether "machining is currently stopped" should be judged by `executionStatus` rather than this field, but note that **while an alarm is up that value too can read `3` (Run) on some machine types** (see that address). A warning in a stopped state means it is waiting for operator intervention such as macro `#3006`
- Fanuc carries at most **100 active alarms** and **17 operator/macro messages** per read (vendor API buffer limits). Mitsubishi carries **10 per alarm kind**, so at most **40** in total (vendor API limit). The list is filled heaviest kind first, so what gets cut when it overflows is the lighter messages.
- **raisedAt**: the time raised, in **UTC with a trailing `Z`** (`"2026-07-29T11:11:03Z"`). Siemens gives the actual time; Fanuc and Mitsubishi are always `null` (active alarms have no time information)
  - The number **differs from what the machine's own screen (HMI) shows**: the HMI renders the machine's local time while this is UTC. They are the same instant written differently; converting is for whoever knows the machine's time zone (the host application). OPC-UA events do have a field for the local-time offset, but it was empty on the control we measured
  - **Do not subtract it from `/machine/currentDateTime`.** Beyond the time zone, the two come from **different clocks**; on one machine they were measured about 18 minutes apart (clock setup varies by site)

## /machine/channel/operateMode
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The current operating mode code (with `desc`). A unified code regardless of machine:

- `0` = Jog · `1` = MDI · `2` = Memory (automatic) · `5` = no mode · `6` = Edit · `7` = Handle
- `8` = Teach in Jog · `9` = Teach in Handle · `10` = INC feed · `11` = Reference (return to origin) · `12` = Remote (DNC)
- `13` = Jog-REPOS · `14` = MDI-Reference · `15` = MDI-Teach in · `16` = MDI-Teach in-Reference · `17` = Auto-Teach in-Reference · `18` = MDI-REPOS · `19` = MDI-Teach in-REPOS
- `99` = Unknown

`13`–`19` are **Siemens-only**: a basic mode (Jog/MDI/Auto) with an auxiliary function (REPOS, reference, teach-in) layered on top, which appears when that combination is selected on the operator panel (Basic Functions manual, K1: JOG can carry REF or REPOS, MDI can carry REF, REPOS or teach-in). Fanuc reports the same situations using the basic mode code alone, so these values never occur there.

On Mitsubishi, **RAPID** (manual rapid traverse) comes out as `0` (Jog); it is manual continuous feed, the same family as Jog, and we do not mint a new number every time a machine type is added. The panel's STEP is `10` (INC feed) and TAPE is `12` (Remote).

`5` (no mode) is **Fanuc-only**: the state where no basic mode is selected, which the operator panel shows as `****` in the mode field. It differs from `99` (Unknown): here the machine positively reported "no mode", whereas `99` means we could not interpret the value it gave. Siemens has no equivalent state.

## /machine/channel/executionStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The program **execution status** code (with `desc`). The "is it running now" counterpart to `operateMode` (which mode it is in):

- `0` = Reset · `1` = Stop · `2` = Hold · `3` = Run (running)
- `4` = MSTR (Fanuc: retraction/recovery) · `5` = Interrupted (Siemens: see below) · `99` = Unknown (Fanuc only; Siemens surfaces unlisted values as a status `-17` error)

`1` and `2` are different **kinds** of pause (who halted it, and where):

- `Stop` = halted **by the program at a planned point**. A block finished in single-block mode (confirmed on all three machine types), or it hit M0/M1, always standing at a block boundary.
  - ⚠️ **On Fanuc, M0/M1 may not read as `Stop`.** The control sends the M code and then waits for the PLC to acknowledge it, and it treats that wait as a cycle still in progress, so the value stays **`3` (Run)**. In a test environment (31i with a PLC that does not act on M0) the program stood at the `M00` block with the control's cycle-start lamp blinking, this address read `3` the whole time, and a second cycle start resumed it. A machine whose PLC does act on M0 may differ, so only what was observed is stated here. Siemens reads `1` (Stop) in the same situation.
- `Hold` = halted **by the operator at an arbitrary moment**. The stop key (feed hold) on the panel was pressed; it can stop mid-block.

⚠️ **The button name and the state name disagree** (an industry convention): pressing the panel's **Stop button puts the machine in `Hold`**. The `Stop` state is produced by the program (M0/M1, single block), not by a button. Either way, Cycle Start resumes.

**What you get while an alarm is up differs by machine type.** The same situation - automatic operation halted by a call to a subprogram that does not exist - measured in the test environments of two machine types:

| | `executionStatus` | `alarmStatus` |
|---|---|---|
| Fanuc | `1` (Stop) | `2` |
| Mitsubishi | `3` (Run), until reset | `2` |

Each control expresses its own automatic-operation state differently. deemesh does not override this: whether an alarm halts machining depends on the alarm (informational ones do not), and we have no per-alarm knowledge of that, so demoting the value would be wrong in the cases that are fine.

**So do not read `3` (Run) as "it is cutting right now".** The value means automatic operation has not ended, not that an axis is moving. Two situations that read `3` while the machine stands still are confirmed: **an alarm is up** (`alarmStatus` is not `0`) and **the control is waiting for an M code to be acknowledged** (no alarm at all). If you need to know whether it is really halted, watch whether `/machine/channel/programCurrentBlock` stops changing, or read `alarmStatus` alongside.

**Mitsubishi reports only `0`-`3`.** This control gives a set of automatic-operation flags (in operation / executing / paused) rather than a status code, and deemesh combines them into the vocabulary above. The `Stop` / `Hold` split maps exactly onto the vendor's own definitions - what it calls "pause" means *halted while executing a command*, which is the `Hold` state above, and the remaining case (in automatic operation but neither executing nor paused) is `Stop`, standing at a block boundary.

`5` (Siemens only) means **the machine stopped because something is wrong**. Two causes are confirmed in the test environment: an **emergency stop** and an **alarm stop**. Normal stops (M0, single block, an operator stop) all resolve to `1`/`2`, so when you see `5`, tell the two apart with `/machine/channel/alarmStatus` and `/machine/channel/emergencyStatus`. The control reports stop reasons in finer detail than this, so other reasons may land here too, which is why the name is still "a stop that was not classified". The fact that it is paused is certain, so a consumer that does not care about the kind may treat `1`/`2`/`5` together as "paused".

⚠️ **During an emergency stop the value differs by machine type** (confirmed in our test environments): Fanuc and Mitsubishi report `0` (Reset), Siemens `5` (Interrupted). This is a spot where this address gives different codes for the same situation; detect an E-stop with `/machine/channel/emergencyStatus`, not this address.

## /machine/channel/motionStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The axis motion status code (with `desc`):

- `0` = None/Idle · `1` = Motion (moving) · `2` = Dwell (dwelling)
- `3` = multi-path synchronization wait (Fanuc) · `4` = Not dwelling (not in a dwell; nothing more is known)

**A dwell keeps counting down while the program is stopped** (confirmed in the Siemens test environment). While an operator stop holds the program and `executionStatus` reads `2` (Hold), the remaining `G4` time still runs to `0`, after which this address changes from `2` (Dwell) to `4` (Not dwelling). A stop does not freeze the dwell, so on resume that block is already finished and execution continues with the next one.

**`4` is a superset of `0`, `1` and `3`** - it means the state is one of those three but cannot be narrowed further. On Siemens and Mitsubishi, deemesh judges from the remaining dwell time, so those two report only `2` or `4` (`0`, `1` and `3` are Fanuc only).

If you need to know whether an axis is actually moving and the control reports `4`, this address cannot tell you. Use `/machine/channel/executionStatus` to see whether automatic operation is running (neither address answers for motion during manual operation).

## /machine/channel/emergencyStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The emergency-stop status (with `desc`): `0` = normal, `1` = emergency stop. On Fanuc a transient `2` (Reset: the moment the E-stop is being released, under a second) can flash by (confirmed in our test environment); treating anything non-`0` as "not normal" is the safe reading.

On Mitsubishi it is `1` whenever the alarm list carries an `EMG` class; it is caught **regardless of the cause** of the emergency stop.

**Siemens decides in two steps.** While the mode-group-ready signal (`readyActive`, PLC interface DB11 DBX6.3) is on the value is `0` with no extra communication; only when it is off does deemesh fetch the alarm snapshot and answer **`1` if the emergency-stop alarm `3000` is present, `0` otherwise**. The ready signal alone would not do: every alarm whose reaction includes "mode group not ready" (drive, encoder and referencing faults among others) clears it, which would make the value broader than the Fanuc emergency-stop signal or the Mitsubishi `EMG` class (Basic Functions manual, A2). In those cases the value is `0` and the cause is told by `alarmStatus` (`2`) and `/machine/channel/alarmList`. After the emergency stop is released, `3000` stays until it is acknowledged and reset, so `1` lasts that much longer; if the alarm snapshot cannot be fetched while the mode group is not ready, the address answers with status `-17` rather than guessing.

## /machine/channel/operatingDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```


**The accumulated automatic-operation time.** It keeps adding up the time the machine spends running a program automatically. `channel` filter. Returns `int` (seconds) + `unit:"s"`, read-only.

It is the counterpart to `programRunDuration`: that one measures **the current cycle**, this one the **life of the machine**. The `program` in that name is what marks its scope, so an address without that prefix accumulates, like `powerOnDuration` and `cuttingDuration`.

**It does not count while held or stopped.** Time parked in feed hold is excluded on both controls (on Fanuc the parameter manual states "neither stop nor hold time included" and the test environment agrees; Mitsubishi exposes separate counters that include and exclude hold, and this address uses the excluding one) - so a program left paused while nobody is at the machine does not register as operating time.

Read together with `/machine/powerOnDuration` it gives you the ingredients for a utilization figure - how much of the powered-on time was actually spent running. It accumulates, so measure an interval by reading twice and subtracting.

**Writing is not supported.** This is the machine's history; changing it makes production figures quietly wrong.

Fanuc reads parameters `6752` (minutes) + `6751` (milliseconds below the minute) in a single call - this is the `RUN TIME` on the control's production screen - and Mitsubishi uses `GetStartTime`. That is the **`Auto strt` item of the control's integrated-time screen (automatic start time: accumulated from the cycle-start button to a feed hold, block stop or reset)**; the `Auto oper` item (automatic operation time: from start to `M02`/`M30` or reset, `GetRunTime`), which includes hold, is deliberately not used (M800 Instruction Manual section 9.3.1). **The control stops accumulating at `59999:59:59`** (the same cap, and the same API-documentation caveat, as `powerOnDuration`).

## /machine/channel/partCountActual
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

The number of parts machined so far, the counter you reset when switching jobs. `channel` filter. Returns `int`; both read and write are supported; write `{"value": 0}` to reset it. The Linux Fanuc library does not provide the function this write uses (`cnc_wrparam`), so on Linux the write is status `-20`.

**This is not "how many parts were actually produced".** It is a counter the control increments in response to program end (`M02`/`M30`), and whether it reacts at all depends on the machine configuration. With that setting off it never moves; a dry run increments it too; running the same program twice makes it 2. An operator can change it at the panel. **The control does not know whether a part is good.** To use it for production reporting, the host application must overlay program and timing information.

**On Fanuc a read right after a write may still return the old value**: the control takes a moment to apply a parameter (measured: reading immediately gave the old value 6 times out of 8; after `50` ms all reads were correct). Read again shortly afterwards to confirm what was written. On the same Fanuc, macro variable writes apply immediately; only parameters behave this way. Siemens applies immediately.

Fanuc uses parameter `6711` (incremented on `M02`/`M30` or the M code set in parameter `6710`; with `6700#0`=`1` only that M code counts), Siemens `actParts` (`$AC_ACTUAL_PARTS`: counts only when bit 8 of machine datum `27880` is set, by default on `M02`/`M30`, or on the M code set in `27882` when bit 9 is set; **with the target comparison enabled (`27880` bit 0) and `partCountRequired` above `0`, it is automatically reset to `0` the moment it reaches the target** - unlike Fanuc, which keeps counting and only raises a signal), and Mitsubishi parameter `8002` (counted when the M code set in parameter `8001` executes; with `8001` at `0` nothing is counted and the panel's work-count display is off, M800 Instruction Manual). **On Mitsubishi this is read-only for now**, so a reset write returns status `-20`.

## /machine/channel/partCountRequired
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

The target quantity to be produced. `channel` filter. Returns `int`; both read and write are supported; write `{"value": 100}`. A `0` means no target is set. The Linux Fanuc library does not provide the function this write uses (`cnc_wrparam`), so on Linux the write is status `-20`.

The machine can be configured to signal or stop once the machined quantity reaches this value, but whether it does so depends on the machine configuration; deemesh only carries the value.

**On Fanuc a read right after a write may still return the old value**: the control takes a moment to apply a parameter (measured: reading immediately gave the old value 6 times out of 8; after `50` ms all reads were correct). Read again shortly afterwards to confirm what was written. Siemens applies immediately.

Fanuc uses parameter `6713` (`0` is treated as infinite and the arrival signal `PRTSF` is never output), Siemens `reqParts` (`$AC_REQUIRED_PARTS`: the target comparison, alarm and PLC signal need bit 0 of machine datum `27880`; with that set, reaching a value above `0` resets `partCountActual` to `0`), and Mitsubishi parameter `8003`. **On Mitsubishi this is read-only for now.**

## /machine/channel/cuttingDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The **cumulative cutting time**, the accumulated time the tool was actually engaged and cutting. `channel` filter. Returns `int` (seconds) + `unit:"s"`, read-only.

It forms a layer together with the power-on time: how much of the time the machine was on was actually spent cutting is the raw material for a utilization figure. Being a running total, the usage over an interval is the difference between two readings.

**The pause conditions differ per control.** Fanuc accumulates, as its parameter manual defines it, **only time spent in cutting feed (`G01`, `G02`, `G03` and the like)**; how it treats feed override `0`, dwell and feed hold is not stated in the manual and has not been confirmed. Siemens follows the rules of `$AC_CUTTING_TIME`: rapid traverse is excluded, the timer pauses during a dwell, and in the default configuration it counts **only while a tool is active** and not during dry run or program test (machine datum `27860`, bits 7, 4 and 5, can change that). The manual does not say how it treats the stopped state or feed override `0`.

**The reset point differs by machine type.** Fanuc keeps accumulating; on Siemens, powering the control up with default values sets it to `0` (a normal power cycle does not). On both, an operator can reset it at the panel.

**Siemens can have this measurement switched off**: when bit 2 of machine datum `27860` is `0` the measurement itself is off and the value stays `0`.

**Writing is not supported.** This is the machine's history, and changing it would silently distort production reporting.

Fanuc sums parameters `6754` (minutes) and `6753` (milliseconds below a minute); Siemens uses `cuttingTime`. The two split parameters are read **in a single call**, so the value cannot be skewed by the minute rolling over between reads.

## /machine/channel/programRunDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The **run time of the current automatic cycle**, which restarts from `0` when a new cycle begins. `channel` filter. Returns `int` (seconds) + `unit:"s"`, read-only.

It is not a running total. Unlike the values that accumulate over the machine's life (power-on time, cutting time), this measures **one run**. It does not count while stopped - Fanuc excludes stop and hold time (stated in the parameter manual) and returns to `0` at power-on and **on a cycle start from the reset state** (resuming from feed hold continues the count); Siemens excludes stopped time and time halted by feed override `0`, and returns to `0` when `M30` is reached or the program is restarted from reset. How Fanuc treats feed override `0` is not in the manual and has not been confirmed.

**Sub-second precision is discarded.** Both machine types offer millisecond resolution, but elapsed-time addresses are uniformly whole seconds. You get the same value the control shows as `CYCLE TIME` (confirmed in the Fanuc test environment). For a cycle only a few seconds long, the up-to-one-second difference is relatively large.

Fanuc reads parameters `6758` (minutes) and `6757` (milliseconds below a minute) together in a single call; Siemens uses `actProgNetTime`.

## /machine/channel/partCountTotal
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The machine's lifetime part count, a running total that is not reset when jobs change. `channel` filter. Returns `int`, **read-only**.

Writing is deliberately not supported: this is the machine's history, and changing it would silently distort production reporting. A resettable counter is available separately.

This value is also a counter driven by program end, so dry runs and re-runs are counted as well; **do not use it as "how many parts were actually produced".**

Fanuc uses parameter `6712` (incremented together with `partCountActual`, under the same conditions) and Siemens `totalParts` (`$AC_TOTAL_PARTS`: counts only when bit 4 of machine datum `27880` is set; by default it increments on `M02`/`M30`, or on the M code of `27882` when bit 5 is set). **Mitsubishi does not support it** (status `-20`) - that control only provides the resettable job counter (`partCountActual`) as a standard value; a lifetime total is something the machine builder implements in the PLC, which the SDK cannot know.

## /machine/channel/feedOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The feed override (%). Returns `int` + `unit:"%"`. Fanuc reads the PMC `G12` signals (`*FV0` to `*FV7`, inverted binary, 0 to 254%), and all signals off reads `0` as the control treats it (Connection Manual B-64483EN-1 section 7.1.6). It is the switch value: while the override cancel signal (`OVC`) forces the effective rate to 100% the switch value is still reported, and the second feed override (`G13`) is not applied. On a multi-path control the path's own signals are read (addresses shift by `1000` per path, such as `G1012` for path 2). Siemens reads the `feedRateIpoOvr` node. **Mitsubishi is read the way the machine builder's ladder selects it** (PLC Interface Manual): with the method-selection signal `FVS` (`YC67`) off, the override code signals (`YC60`-`YC64`, 0-300% in 10% steps) are decoded; with it on, the per-part-system register `R2500` (0-300% in 1% units) is read. When all code signals are off the control keeps the previous value, so there is nothing to read and the address answers with status `-17`.

## /machine/channel/rapidOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The rapid-traverse override (%). Returns `int` + `unit:"%"`. **Fanuc depends on the method the machine builder's ladder selects** (Connection Manual B-64483EN-1 section 7.1.7): the default `ROV1`/`ROV2` method (`G14`) is stepped, so only `100`/`50`/`25`/`0` appear (`0` is F0, the speed in parameter `1421`); the 1% step method (`HROV`, `G96`) gives integers `0` to `100`; the 0.1% step method (`FHROV`, `G353`) reports values that fall on whole percents as they are, and a value such as `87.5` is status `-17` (this address is `int`). On a multi-path control the path's own signals are read (addresses shift by `1000` per path). Siemens is continuous. **Mitsubishi depends on the method the machine builder's ladder selects** (PLC Interface Manual): with the method-selection signal `ROVS` (`YC6F`) off, the code signals `ROV1`/`ROV2` (`YC68`/`YC69`) give the four values `100`/`50`/`25`/`0` (the manual's `1%` step is reported as `0`, like Fanuc); with it on, the per-part-system register `R2502` (0-100% in 1% units) gives a continuous value.

On those two, `0` does not mean "stopped" but **the slowest rapid step that machine defines** (the lowest position on the panel). What speed that actually is depends on the machine's settings and cannot be read from this address.

## /machine/channel/feedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The commanded feedrate (F command value). Returns `float`. Fanuc uses the modal F, Siemens `cmdFeedRateIpo`, and Mitsubishi the `F command feed speed` (FA).

The unit follows the machine setting (mm/min or inch/min). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/feedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The channel's actual feedrate, **the speed at which the tool tip travels along the programmed path**. Returns `float`.

This is the value the `F` command sets. It holds at the commanded rate as the direction changes, and drops when override, acceleration/deceleration, corner slowdown or feed hold apply. Use it to answer "is the machine actually cutting at the programmed feed?"

Fanuc uses `actf` and Siemens `actFeedRateIpo`. Mitsubishi splits the effective feedrate into **an automatic-operation value and a manual one**, so both are read and combined - the value is there while you move an axis by jog or handwheel too.

The unit follows the machine setting (mm/min or inch/min). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/programSequenceNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **sequence number (N number)** of the block currently executing. `channel` filter. Returns `int`.

**On blocks without an N number the machines differ** (all three confirmed in our test environments).

| | A block with no N |
|---|---|
| Fanuc, Mitsubishi | the last executed N number is **retained** |
| Siemens | **always `0`**: it never retains the previous N, not even within the same file |

Only on Siemens can `0` be read as "currently in a block with no N number".

**The retention lasts only while the program runs.** Once it ends and the control is in reset, the value goes back to `0` (measured on Mitsubishi: `N400` executed last, then `0` after `M30`). Do not assume the final N stays there.

**Entering a subprogram gives you the subprogram's N** (the same moment `programName` switches to the subprogram's name). On return it goes back to the main program's N.

⚠️ **Fanuc's retention crosses file boundaries** (confirmed in our test environment): on an N-less block right after returning from a subprogram you see **the subprogram's last N**, and right after entering one you see **the main's N**. On Fanuc this value alone therefore cannot tell you which file the N belongs to; read `/machine/channel/programName` alongside it. On Siemens the value is not retained, so the situation does not arise. Mitsubishi directs every query at **the program currently running** (main or sub), so the value is always the N of the running program.

## /machine/channel/programBlockCounter
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The executed-block counter. `channel` filter. Returns `int`.

⚠️ **Each machine type counts something different.** The three values are not comparable, so **do not use this as a progress figure across machine types** (all three confirmed in our test environments):

| | What it counts | Resets |
|---|---|---|
| Fanuc | blocks executed since the cycle started (`cnc_rdblkcount`, counting through subprogram blocks) | at each Cycle Start |
| Siemens | the **line number within the file currently executing** (`actLineNumber`, negatives clamped to `0`) | when the file changes (entering or leaving a subprogram) |
| Mitsubishi | how many blocks have passed **since the current `N` number** | at every `N` number |

**On Mitsubishi it is one half of a pair with `programSequenceNumber`.** That control addresses a position inside a program with three values - program name, `N` number, and the block count from that `N` - and the operation search on the control takes the same three. So this number alone does not fix a position; read it together with `N` to have a position (measured: `0`, `1`, `2`, `3` through the `N100` stretch, then `0` on reaching `N500`).

On Siemens the line number is **relative to the file currently executing**: entering a subprogram switches it to the subprogram's line numbers, and returning switches it back to the main's (confirmed in our test environment). The number alone therefore cannot tell you which file it counts; line 3 of the main and line 3 of the subprogram are both `3`. To pin down the file, read `/machine/channel/programName` and `/machine/channel/programNestLevel` alongside it. During the nesting transition (under a second) a sample can briefly combine mismatched values (the level updates before the name).

## /machine/channel/programName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The name (file name) of the program **currently executing**. When entering a subprogram, it changes to that subprogram's name. For the main selected in the HMI, see `mainProgramName`.

## /machine/channel/programPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The full path of the currently executing program (e.g. `//CNC_MEM/USER/PATH1/O0001`). `channel` filter. Siemens converts the NCK-internal path into the user notation (`//NC/...`) before returning it.

**On Mitsubishi the folder part is not guaranteed to belong to the program currently executing.** That control has no call returning a full path, so the directory and the file name are fetched separately - and the vendor API **has no call that returns the directory of the program currently executing**. In practice it is usually right, because this control's NC memory has a fixed directory layout (you cannot create folders - see `directoryExists`), so a main program and its subprogram rarely sit in different places. The file name part always belongs to the program currently executing.

## /machine/channel/mainProgramName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The name of the **main program selected in the HMI**. It does not change even when a subprogram is entered during execution (that is the difference from `programName`).

## /machine/channel/mainProgramPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The full path of the **main program selected on the HMI** (the path form of `mainProgramName`). It does not change when execution descends into a subprogram.

**Writing selects the program**: it makes the program at that path the channel's main program (the one to be executed). The value is a path string: `{"value": "//CNC_MEM/USER/O0001"}`.

- Path notation follows the machine: Fanuc `//CNC_MEM/USER/O0001` (data server: `//DATA_SV/...`), Siemens `//NC/Part programs/PART1.MPF` (the same notation `programPath` and `entryList` return; `Subprograms` and `Workpieces` likewise), Mitsubishi `//PRG/USER/O0001` (the notation below `ncMemoryRootPath`). NC file-system paths are vendor-specific and are not normalized, for the same reason as `plcAddress`
- **It must be a file**: passing a folder path returns status `-18`, as does a path that does not exist
- Selecting does **not start machining** (cycle start remains the operator panel's / PLC's job)
- Siemens calls the server's file-handling `Select` method; Fanuc uses `cnc_pdf_slctmain` for CNC memory paths and `cnc_wrdsdncfile` for data-server paths; Mitsubishi uses the operation search `Search`

## /machine/channel/programCurrentBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **G-code text** of the block currently executing. `channel` filter. When there is none the value is an **empty string**, never `null`.

**What you get "when nothing is executing" differs by machine type.** Measured on controls holding a selected program in reset:

| | Siemens, Mitsubishi | Fanuc |
|---|---|---|
| `programCurrentBlock` | `""` | the **first line** of the program |
| `programNextBlock` | the first block | the line **after** that |

Siemens and Mitsubishi can express "no block is executing" (on Mitsubishi the control reports the execution position as `0`, meaning not in operation). Fanuc's `cnc_rdexecprog` carries no such marker, so the first line of the look-ahead buffer comes out as "current"; while stopped, that line is really the block that will run **next**.

**Comparing against the control makes this visible.** On the program screen, the execution marker (the highlight bar) sits **above** the first line while in reset, meaning no block has run yet, and the empty string is that state written down faithfully.

**So do not decide "what is executing right now" from this address alone.** Read `/machine/channel/executionStatus` alongside it and treat this value as meaningful only when that is `3` (Run).

**If you need more than one of them, ask for them in a single request.** `programLastBlock`, `programCurrentBlock`, `programNextBlock` and `programLookAhead` are served from **one query to the machine** when requested together, so they agree with each other. Read separately, the program advances in between and you get a combination where previous, current and next are not consecutive at all (measured: read one at a time they came back as `G04 X8.`, `G04 X7.` and `G04 X5.`, a block skipped, while the same stretch read together never disagreed). It is faster too, but consistency is the reason.

## /machine/channel/programNextBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The G-code text of the block to be executed next. `channel` filter. When there is no next block (the last block) the value is an empty string.

The **machine-type difference described under `programCurrentBlock` applies here too**: while stopped, Fanuc returns a line that is off by one.

## /machine/channel/programLastBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The G-code text of the block executed just before. `channel` filter. When there is no preceding block (the program's first block, or not in operation) the value is an empty string.

**Supported on Siemens and Mitsubishi; Fanuc returns status `-20`.** Fanuc's `cnc_rdexecprog` holds the look-ahead buffer (blocks still to come), so a block already executed is not retained.

## /machine/channel/programLookAhead
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

**The program text around the current execution point**: a multi-line string holding the current block and what lies ahead of it. Returns `string`.

How much you get differs by machine: Fanuc returns the whole look-ahead buffer (`cnc_rdexecprog`), Siemens a chunk around the execution point (`actPartProgram`), Mitsubishi up to ten blocks starting at the current one (`CurrentBlockRead`). **The whole program cannot be obtained from this address**; for that, read the program file from the NC file system.

Line endings are normalized to a single LF (`
`) on every protocol; CR is stripped, so splitting on `
` is safe.

## /machine/channel/programNestLevel
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The program-call nesting level (with `desc`): `0` = no program, `1` = main, `2`+ = subprogram (L1, L2, …). `channel` filter. **Supported on Siemens and Mitsubishi** (Fanuc unsupported).

**What it counts is the depth of the program pointer, not execution.** With a program loaded it reads `1` even when nothing is running (measured: both machine types report `1` while in reset or interrupted). For "is it running right now", use `/machine/channel/executionStatus`.

On Mitsubishi the vendor value counts **how many subprograms deep** you are (main is `0`), one step off our scale, so deemesh shifts it. Telling `0` (no program) apart from `1` (main) costs this address one extra query to the machine.

## /machine/channel/auxModal/auxModalValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "auxModal"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The auxiliary-function modal value: specify a **letter** in the `auxModal` filter (e.g. `auxModal=M`, `S`, `T`, `D`, `H`, `F`). Returns `float`. Example: `auxModal=T` → the commanded tool number, `auxModal=S` → the commanded spindle speed.

**When a block commands several M codes, read them by suffixing the letter with its position**: `M` is the first, `M2` the second (as in a line like `M8 M42 M13;`). How many are available is a control specification: **Fanuc goes up to `M3`, Mitsubishi up to `M4`.** Mitsubishi also gives `B` a position, up to `B4`.

**When a block holds several M codes, the two machine types answer differently.** The same `M8 M5` block, observed in our test environments:

| | `M` | `M2` |
|---|---|---|
| Mitsubishi | `8` | `5` |
| Fanuc (the control we tested) | `5` | `0` |

Mitsubishi fills the positions in **the order written in the block** (not numeric order: `8` was written first, so it takes the first position). The Fanuc control we tested carried only one of the two in `M` and left the positional slots empty. Commanding several M codes in one block is an optional function on Fanuc, so on a machine without it `M2` and `M3` are always `0`.

**So do not use this address to ask "is this M code in effect" portably.** How many positions there are, which one a code lands in, and whether they are filled at all depend on the machine. Unused positions read `0`.

**Which letters are accepted differs by machine type.**

- **Fanuc**: a fixed set of `B`, `D`, `F`, `H`, `L`, `M`, `M2`, `M3`, `P`, `Q`, `R`, `S`, `T`
- **Siemens**: any letter the control has a modal for (in our test environment `E` and `A` answered too). Positional forms are not accepted
- **Mitsubishi**: `B`, `B2`, `B3`, `B4`, `M`, `M2`, `M3`, `M4`, `S`, `T`. The vendor API covers M/S/T/B, so `D`, `F`, `H` and so on are not available

A letter that is not accepted returns status `-18`, and the error string tells you what that machine type can use.

**`S` never takes a position.** Mitsubishi's vendor index for `S` is the spindle number, but this address has no `spindle` filter, so it is fixed to the first spindle. Per-spindle commanded speed is what `/machine/channel/spindle/spindleSpeedCommanded` is for.

**You get the modal value the control reports, verbatim - it is not translated.** This is a general-purpose channel, like `parameter` and `diagnosis`. However, **how a control represents "this letter has nothing commanded" differs by machine type.**

- **Siemens answers `null`.** The control reports "nothing is assigned to this letter" as a distinct state (source `/Channel/SelectedFunctions/`; the vendor manual says an unassigned entry yields a negative number), and that state is returned as `null`. Measured on an idle machine, `T`, `S`, `H` and `M` were `null`, `D` was `1` and `F` was `0` (where the letter has a real value you get that value). The vendor manual describes this source as the list of auxiliary functions **programmed in the current block**, so during program execution `null` can also appear in a block that does not contain that letter (this has not been measured while running yet). `H` can be commanded negative on Siemens, and since the control also marks "unassigned" with a negative number, a negative `H` command cannot be told apart from `null` at this address.
- **Fanuc and Mitsubishi answer `0`.** Those controls have no separate "not commanded" representation: `0` covers both "nothing commanded" and a commanded `0` (`T0` is a command people actually use). We do not invent a distinction the machine does not make, so it is not turned into `null`.

So **testing for "has anything been commanded" with `== 0` alone will not catch it on Siemens, and `== null` alone will not catch it on Fanuc or Mitsubishi.** Which value counts as "none" is for the host application, which knows the machine, to decide.

## /machine/channel/singleBlockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The single-block switch state (`true` = on). Fanuc reads the F4 signal bit, Siemens `singleBlockActive`, and Mitsubishi a PLC output (Y) bit in the operator-panel signal block.

## /machine/channel/dryRunOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The dry-run switch state (`true` = on). `channel` filter. Supported on Mitsubishi as well as Fanuc and Siemens; on Mitsubishi it is a PLC output (Y) bit in the same operator-panel signal block as `singleBlockOn`, so asking for both costs a single query.

## /machine/channel/optionalStopOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens"]
write: []
```

The optional-stop (M01 enabled) switch state (`true` = on). `channel` filter. **Siemens only.**

**Not supported on Fanuc** (status `-20`). The optional-stop switch is an input from the machine operator's panel into the PMC, and the control exposes no signal for its state (Connection Manual B-64483EN-1 section 10: "for M00/M01 only the code, strobe and decode signals are sent; program stop and optional stop are designed on the PMC side"). The `DM01` signal the control does output (`F9.6`) is a decode signal meaning the program has commanded an `M01` block, unrelated to the switch (measured on NC Guide: `F9` stays `0` with the switch on). If you know the device the ladder uses, read it through `/machine/plcAddress/plcType/plcValue`.

**Not supported on Mitsubishi either** (status `-20`). Per the PLC Interface Manual that control has no NC signal for the optional-stop switch: when `M01` arrives, **the machine builder's PLC checks its own switch input** and raises the single-block signal (`SBK`) to stop, so the switch state lives only in that machine's ladder devices and cannot be read through a machine-independent address (the same class as `plcAddress`). If you know the device the ladder uses, read it directly through `/machine/plcAddress/plcType/plcValue`.

## /machine/channel/blockSkipOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The block-skip (`/`) switch state (`true` = on). `channel` filter. Supported on all three machine types.

**On controls with several skip levels, this address still looks only at the plain `/`.** Some controls let you number the prefix (`/2`, `/3`, …) so different sections are skipped by different switches (Siemens documents levels `0`–`9`, Mitsubishi `BDT1`–`BDT9`), but what this address reports is always the **unnumbered `/`**. There is no address for the numbered levels.

## /machine/channel/machineLockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The machine-lock (axis-motion lock) state (`true` = on). `channel` filter. Supported on all three machine types; for Siemens this is the program-test (`progTestActive`) state.

**Mitsubishi publishes this signal per axis, while this address is one value per channel.** It therefore reads `true` **only when every axis in that channel is locked**. Calling a partial lock "on" would read as "nothing is moving" and lead a consumer to mistake real machining for a test run. The operator-panel switch drives all axes together, so on an ordinary machine the distinction never surfaces.

## /machine/channel/variable/variableValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "variable"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

Reads/writes a **macro variable (Fanuc, Mitsubishi) / R parameter (Siemens)** (read + write). Put the variable number in the `variable` filter (e.g. `variable=100` → Fanuc and Mitsubishi `#100`, Siemens `R100`). Returns `float`; writes take `{"value": 3.14}`. **Reads** support range/comma expansion: `variable=100-105` is an array of 6 values. Writes always target a single variable (expansion syntax is rejected with status `-13`: the rule shared by every write). A **vacant macro variable on Fanuc or Mitsubishi reads as `null`**; this is the state the control's custom-macro screen shows as an empty cell (`DATA EMPTY` on Fanuc), and it is distinct from the value `0`. In a range expansion only that slot becomes `null` (e.g. `[3.14, null]`).

**Which numbers exist depends on the machine and its options.** deemesh keeps no list and passes the number through, so a number that machine does not have comes back as **status `-18`** (on reads and writes alike, with the vendor's own reason in the error string). The only thing to fix is the `variable` value, and the control's variable screen tells you which numbers that machine actually has. If a missing number is mixed into a range expansion, the **whole request fails with `-15`** (no partial array is returned). Distinguish this from vacant variables, which are not errors but `null` elements and do not break the expansion. A number outside the syntactic range (`0`-`89999` on Fanuc) is the same status `-18`, except that one is rejected immediately without asking the machine.

**Writing a variable back to vacant is not supported**: the value is a single number, and clearing a variable that has one requires the operator panel.

## /machine/channel/toolOffset/toolName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

The **name of the tool** called by that offset number. `channel` + `toolOffset` filters. Returns `string`. Reading and writing are Fanuc only.

The value comes from Fanuc's **tool geometry size data** (the panel's `TL GEOM SIZE` screen, option "Tool geometry size data 100/300 pairs"). That table is **indexed by the tool offset number** (Operator's Manual B-64484EN section 13.2.2.2: the row with the same number as the `D` code on machining centres, or the geometry offset number on lathes). That is why this address lives in the `toolOffset` folder rather than under a tool management slot (`/machine/toolArea/tool/…`), and it does not depend on the TOOL MANAGEMENT option. Without the geometry size data option the address returns status `-20`.

A row whose tool type is not set (type `0`) reads as the empty string `""`. The table's size is set by the option, so it is not guaranteed to equal `/machine/channel/toolOffsetCount` (both are 100 on our bench machine). A number beyond the table is rejected with status `-18`, and the message names where the table ends. Asking for a range (`toolOffset=1-100`) costs one round trip.

**Writing** takes a string that fits in 8 bytes on the machine (longer is status `-16`; characters the machine's display-language code page cannot store are status `-16` as well). A row without a tool type cannot take a name, status `-18` (Fanuc does not keep a row without a type; set the type on the panel's `TL GEOM SIZE` screen first). If the name is already the same nothing happens and the write succeeds.

On Siemens the tool name belongs to the tool number, so it is `/machine/toolArea/tool/toolName`. Same meaning (the tool's name), different key, hence different addresses.

## /machine/channel/toolOffset/toolType
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

The **kind of the tool** called by that offset number (the `TK` icon on the panel's `TL GEOM SIZE` screen). `channel` + `toolOffset` filters. Returns `int` + `desc`. Reading and writing are Fanuc only.

The value is **Fanuc's raw code, passed through**. It is a vendor-owned open classification, so it is not translated and will not be unified across machine types (Siemens' `/machine/toolArea/tool/toolEdge/toolType` carries the DP1 code, a **different code space** with a different key). If a machine-independent classification is ever needed it will be a separate address with a smaller promise (`toolTypeCategory`). `desc` is prose for a human; branch on the value.

| Value | Meaning |
|---|---|
| `0` | Row not defined |
| `10` | General-purpose turning tool |
| `11` | Threading tool |
| `12` | Grooving tool |
| `13` | Round-nose tool |
| `14` | Point nose straight tool |
| `15` | Versatile tool |
| `20` | Drill |
| `21` | Counter sink tool |
| `22` | Flat end mill |
| `23` | Ball end mill |
| `24` | Tap |
| `25` | Reamer |
| `26` | Boring tool |
| `27` | Face mill |

Source, table size and option are as for `/machine/channel/toolOffset/toolName`: the tool geometry size data (option "Tool geometry size data 100/300 pairs"), indexed by the tool offset number, status `-20` without the option, status `-18` beyond the table.

**Writing** accepts only the codes in the table (anything else is status `-16`). Writing a non-zero kind to an undefined row **creates that row**; follow up with `toolName`. **Writing `0` deletes the row** (the name and the dimensions go with it; the control defines writing kind `0` as deletion). If the value is already the same nothing happens and the write succeeds. **Changing a defined row to another kind makes the control redefine the row, and the name and dimensions are cleared** (measured on our bench: changing `20` to `27` left the name as `""`). Set the kind first, then the name and the dimensions.

## /machine/channel/toolOffset/toolLengthGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **M-type (machining center) tool length geometry** value (the H column on the offset screen). The reference value entered from tool measurement, forming the basis of length compensation (H code).

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is **not split into columns**, where the error string points you at `toolOffsetValue` instead. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolLengthWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **M-type tool length wear** value (the H column on the offset screen). The fine compensation accumulated during machining; the usual practice is to adjust this alone and leave the geometry value untouched.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is **not split into columns**, where the error string points you at `toolOffsetValue` instead. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **M-type tool radius geometry** value (the D column on the offset screen). The radius reference that cutter compensation (G41/G42) consults.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is **not split into columns**, where the error string points you at `toolOffsetValue` instead. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.


**The number can differ from what the machine's screen shows.** This value is a **radius**, but an offset screen may be configured to display and accept **diameters**. deemesh emits what the machine stores and does not convert.

## /machine/channel/toolOffset/toolRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **M-type tool radius wear** value (the D column on the offset screen). Reflects the radius reduction caused by tool wear.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is **not split into columns**, where the error string points you at `toolOffsetValue` instead. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.


**The number can differ from what the machine's screen shows.** This value is a **radius**, but an offset screen may be configured to display and accept **diameters**. deemesh emits what the machine stores and does not convert.

## /machine/channel/toolOffset/toolXGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type (lathe) X-direction tool dimension geometry** value. Here X is not an axis name but a fixed column on the offset screen, that is, a **directional component of the tool dimensions**.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolXWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type X-direction tool dimension wear** value. The X-direction compensation accumulated during machining.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolZGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type Z-direction tool dimension geometry** value. Like X, this is a fixed column on the screen, not an axis.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolZWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type Z-direction tool dimension wear** value.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolYGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type Y-direction tool dimension geometry** value: the **third column** after X and Z; a lathe without the option returns status `-20`.

**The screen heading for this column is not `Y` on every machine.** Mitsubishi assigns this slot to the third axis, so a C-axis lathe shows it as `C` on the control (the vendor manual writes the column as `C (Y*)`). What the address promises is the **third offset column**; which axis that is, the column heading on the control tells you.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolYWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type Y-direction tool dimension wear** value (Y-axis-option machines only). The **third column** after X and Z; the screen heading is not `Y` on every machine (a Mitsubishi C-axis lathe shows `C`). See `toolYGeometry` for details.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type nose radius geometry** value. Consulted by nose-radius compensation (G41/G42); together with the tip direction (`toolTipDirection`) it determines the tool-tip path.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolNoseRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type nose radius wear** value.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/toolOffset/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The lathe tool's **virtual tool-tip position code** (read + write). A code that determines, during nose-radius compensation (G41/G42), where the tool tip lies relative to the nose center; it is a position code, not an angle, and is returned/entered as an integer with no scaling (`{"value": 3}`).

- `1`–`8` = orientation, **`0`/`9` = the tool nose center is the reference point** (rather than the imaginary tip). The two values mean the same thing: the Fanuc 0i-F lathe manual (`B-64604EN-1/01` §5.2.2) defines `0` and `9` as the codes used when the tool nose center coincides with the reference point
- **`desc` is attached only to `0` and `9`**: the manual defines `1`–`8` by per-plane diagrams, and there is more than one diagram (per plane: `G17`/`G18`/`G19`), so the same number denotes different orientations depending on the setup. For the per-orientation reading follow that machine's manual
- Siemens equivalent concept: cutting edge position (`toolArea/tool/toolEdge/toolTipDirection`). **The two trees share the same numbering and the same `desc` vocabulary**; only the address differs, so the values can be compared and reused as-is
- **The accepted range differs by machine type** - Fanuc `0`-`9`, Siemens `1`-`9`, Mitsubishi `0`-`8`. The code for the center differs too (`0`/`9`, `9` and `0` respectively), so **reads are unified by `desc` but writes must stay inside that machine's range** (the `9` that Fanuc accepts is status `-16` on Mitsubishi)

## /machine/channel/toolOffset/toolOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

The **compensation amount** of one tool compensation number (read + write, `float`). Needs the `channel` and `toolOffset` filters; writes take `{"value": 12.345}`.

**This address is for machines whose compensation memory is not split into columns.** On such a machine the control's compensation screen shows exactly **one** value per number: no geometry/wear split and no length/radius split. That is why the name is `toolOffsetValue` and not `toolLength…`: **the control does not call this value a length**, and whether it acts as a length or a radius compensation depends on how the program references that number.

On a machine whose memory is split, this returns status `-20`, and **the error string carries the list of leaves that do work there** (for example `toolLength{Geometry,Wear}, toolRadius{Geometry,Wear}`). One request therefore tells you the shape of that machine's compensation tree, so there is no separate address to ask which model it uses.

The upper bound for the compensation number is `/machine/channel/toolOffsetCount`; a number beyond it returns status `-18`. The unit follows the machine's configuration (mm or inch); check with `/machine/channel/gModalCategory/gModal?gModalCategory=4`.

## /machine/channel/toolOffsetCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

The **number of available** tool compensation registers (read only, `int`). Offset numbers run `1` to this value; use it as the upper bound when a UI iterates the table.

## /machine/channel/workOffset/axis/workOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**Work coordinate system offset**: the per-axis offset distance of a work coordinate system such as G54 (read + write). Returns `float`, a real distance (in the machine's configured unit mm/inch as-is; the SDK normalizes Fanuc's internal integer representation by the decimal scaling).

The `workOffset` filter **takes the shop-floor G-code notation directly** (an open namespace like `plcAddress`, no separate numbering system). Case-insensitive; whitespace and aliases are not allowed:

- **Fanuc and Mitsubishi**: `EXT` (the common offset added to all coordinate systems, the operator-panel EXT row), `G54`–`G59`, extended `G54.1P1`–`G54.1P300` (a P number not in the option returns a vendor error). The two accept **the same notation and reject with the same message**
- **Siemens**: `G500`, `G54`–`G57`, `G505`–`G599`. How many actually exist depends on the machine configuration, so a designator that does not exist answers with status `-18` together with **the list this machine accepts**

⚠️ **`G500` is not the same as Fanuc's `EXT`.** `EXT` is a common offset that is **added on top** of whichever `G5x` is active, whereas `G500` is an **exclusive member of the same modal group** as `G54`–`G57` and therefore cannot be active alongside them; when `G500` is in effect the settable offset is switched off, and the value in that slot is normally `0`. Read `/machine/channel/gModalCategory/gModal?gModalCategory=7` to see which one is active. On Siemens the part that adds to every coordinate system the way `EXT` does lives in **separate frames** (the `Basic reference` and `Total basic WO` rows on the operator panel), and deemesh does not expose those individually; for the combined result, read `/machine/channel/axis/totalWorkOffsetValue`.

**On Siemens the value is the coarse offset plus the fine offset.** The machine applies that sum and the operator panel shows them as two cells of one offset (`Coarse` and `Fine`), so this address gives you **the offset actually in effect**; comparing it against the panel's `Coarse` cell alone can look like a mismatch. To read the fine part on its own, use `/machine/channel/workOffset/axis/workOffsetFineValue`. Fanuc and Mitsubishi have no fine offset, so their value is single; that is what makes this address mean the same thing on all three machine types.

⚠️ **This value is the stored translation.** Two more things bear on it. ① A work coordinate system can also carry **rotation, scaling and mirroring** (`workOffsetRotation`, `workOffsetScale`, `workOffsetMirrorOn`), and where those are set the coordinate transform is not determined by this value alone. ② The total actually in effect can differ from this value, because a basic reference and other frames add to it (`totalWorkOffsetValue`). If you need part coordinates, do not compute them; read `/machine/channel/axis/workPosition`. On an ordinary setup that only translates, rotation is `0`, scaling is `1` and mirroring is `false`, so this value *is* the transform.

`axis` is the axis number (1–) or the axis name. `axis=1-3` · `workOffset=G54,G55` expansion is supported; for Fanuc, axis expansion of the same workOffset is bundled into a single FOCAS call.

Writes take `{"value": 25.4}` (a single axis). **Supported on Fanuc and Mitsubishi**: on Siemens, writing this value directly **is accepted by the machine yet changes nothing** (observed in our test environment); a separate machine-side activation procedure is required and this protocol does not expose it, so the write is rejected with status `-20`.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/workOffset/axis/workOffsetFineValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

The **fine (`Fine`) part** of a work coordinate system offset. Filters and type are the same as `/machine/channel/workOffset/axis/workOffsetValue`. **Read-only**.

A fine offset is **a small correction laid on top without touching the base offset**. What you measure when first establishing the coordinate system goes into the coarse value (`Coarse`); if the first part then measures `0.02mm` out, that `0.02` goes into the fine offset; the original setup value stays intact and traceable, and zeroing the fine offset returns you to the setup state.

**The offset in effect is the coarse value plus this one**, and that sum is what `workOffsetValue` answers. If you need the coarse value alone, subtract this from `workOffsetValue`; there is no separate address for it.

On a machine that does not use fine offsets (or has them switched off in machine data) it is `0`.

**Siemens only**: a Fanuc work coordinate system offset is a single value with no fine part.
## /machine/channel/workOffset/axis/workOffsetRotation
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

The **per-axis rotation angle** of a work coordinate system. Filters are the same as `/machine/channel/workOffset/axis/workOffsetValue`. Returns `float` with `unit` `deg` (degrees). **Read-only**.

`0` means no rotation is set on that axis.

**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five; on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**Writing is not supported.** Writing to these values directly makes the machine **accept the request and change nothing** (measured). A separate activation step is required on the machine side and this protocol does not expose it; make changes at the operator panel. The translation (`workOffsetValue` and `workOffsetFineValue`) is status `-20` on Siemens for the same reason.



## /machine/channel/workOffset/axis/workOffsetScale
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

The **per-axis scale factor** of a work coordinate system. Filters as above. Returns `float`; it is dimensionless, so there is no `unit`. **Read-only**.

`1` means no scaling (true size). `2` machines at double size along that axis.

**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five; on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**Writing is not supported.** Writing to these values directly makes the machine **accept the request and change nothing** (measured). A separate activation step is required on the machine side and this protocol does not expose it; make changes at the operator panel. The translation (`workOffsetValue` and `workOffsetFineValue`) is status `-20` on Siemens for the same reason.

**Siemens only.**

## /machine/channel/workOffset/axis/workOffsetMirrorOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

Whether the work coordinate system **mirrors** that axis. Filters as above. Returns `boolean`. **Read-only**.

`true` reverses the direction of that axis. This is the per-axis checkbox on the work-offset detail screen of the operator panel.
**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five; on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**Writing is not supported.** Writing to these values directly makes the machine **accept the request and change nothing** (measured). A separate activation step is required on the machine side and this protocol does not expose it; make changes at the operator panel. The translation (`workOffsetValue` and `workOffsetFineValue`) is status `-20` on Siemens for the same reason.

**Siemens only.**

## /machine/channel/axis/totalWorkOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

The **total zero offset actually in effect** right now, per axis (translation). Returns `float`. **Read-only.** This corresponds to the `Total WO` row on the operator panel.

Where `workOffsetValue` is the value **stored in the table**, this address is the value **being applied**. When the two differ, that difference is the clue you are looking for:

```
workOffsetValue?workOffset=G54     80.400    what you set
totalWorkOffsetValue              100.400    what is actually in effect
                                   ↑ 20.000 was added somewhere
```

Besides the stored offset, the total includes the **`Basic reference`, base frames, a `TRANS` set by the program, and frames set by cycles**. Compare the two addresses when diagnosing "the setting is unchanged but the part is off"; reading only the stored value hides the part that was added.

⚠️ **Do not compute part coordinates from this value.** It is only the total translation; **rotation, scaling, mirroring and tool length compensation are not in it.** If you need workpiece coordinates, read `/machine/channel/axis/workPosition`; that is the result after the machine has applied all of them.

This address takes no `workOffset` filter; it is "whatever is in effect", so there is no designator to choose. `axis=1-3` expansion is supported, and writing is not (a summed result is not something you write back).

**Siemens only.** On Fanuc and Mitsubishi, no call that reports this total directly was found within the vendor API range deemesh uses, so it is currently unsupported there. On those controls, read the common offset separately with `workOffset=EXT` and add it to the active coordinate system's value, but that sum **does not mean the same thing as this address**: it omits any shift set by the program (`G92`, `G52` and the like), which is precisely the difference this address exists to reveal. Subtracting `workPosition` from `machinePosition` is not the answer either; that difference also contains the **tool length compensation**, so it is off on the tool axis.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/gModalCategory/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalCategory"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

Queries the active G modal by a **machine-independent standard group number** (a vendor-neutral number defined by deemesh like `plcType`, not the vendor's raw group number). `gModalCategory` filter values:

⚠️ **What is neutral is the question you ask (the group number), not the answer.** The value that comes back is that machine's G code, so it cannot carry a machine-independent branch. The same state reads `G21` on Fanuc and `G710` on Siemens. `desc` tells you the meaning, but it is **prose for a human**, not a contract to branch on; the wording can change. This address is for a host application that knows its own machine's G codes. When you need a decision that spans machine types, use an address that answers the question directly: `feedActual` for the effective feed, `spindleSpeedActual` for spindle rpm, and so on.

- `1` = motion: feed mode (G00 rapid / G01 linear / G02·G03 circular …)
- `2` = plane: machining plane (G17 XY / G18 ZX / G19 YZ)
- `3` = distanceMode: absolute/incremental (G90/G91) · **not supported on Fanuc T-series (System A)** (which uses the U/W address method)
- `4` = units: inch/metric (G20·G70·G700 / G21·G71·G710). On Siemens, `G70`/`G71` switch coordinates only, `G700`/`G710` also feedrates and offsets (Programming Manual section 9.3.5)
- `5` = feedMode: feed specification (per-minute/per-rev/inverse-time)
- `6` = cutterComp: tool-radius compensation (G40 cancel / G41 left / G42 right)
- `7` = coordinateSystem: work coordinate system (G54–G59)
- `8` = spindleSpeedMode: constant surface speed (G96) / constant rpm (G97)

The value is that machine's G-code string, plus for key combinations a machine-independent meaning in `desc` (e.g. Fanuc `{"value":"G21","desc":"metric"}`, Siemens `{"value":"G710","desc":"metric"}`). For access to the raw vendor groups, use `gModalGroup/gModal` (one group) or `gModalList` (all). When no modal is in effect for that group (including combinations the machine type does not support), the value is `null`, the same representation as that slot of `gModalList`. **On Siemens, `feedMode` (`5`) and `spindleSpeedMode` (`8`) are the same group, so their values are always identical.** SINUMERIK puts the feed types (`G93`, `G94`, `G95`) and the spindle-speed types (`G96`, `G97`) in one G group, so only one value is ever active and only `desc` distinguishes which question you asked:

```
machine in G94
  gModalCategory=5 → {"value":"G94","desc":"feed per minute"}
  gModalCategory=8 → {"value":"G94","desc":"constant spindle speed (rpm)"}
```

`G94` is a feed code, not a spindle code. The `desc` reads that way because "not in the G96 family" means the spindle is not in constant surface speed - so **testing `value == "G97"` for constant-rpm never matches on Siemens.** On Fanuc and Mitsubishi the two groups are separate, so the values differ.

**On Mitsubishi this is supported on machining centres and lathes alike** (all eight groups). The group numbers were verified to be the same in both programming manuals (machining centre section 3.4.2, lathe section 3.4.3, the G-code list tables). On a lathe **the same meaning appears as different G codes** depending on the G-code list (parameter `#1037 cmdtyp`): absolute/incremental is `G90`/`G91` (lists 3, 5, 7) or `G190`/`G191` (lists 2, 4, 6), feed is `G94`/`G95` or `G98`/`G99`, so read the meaning from `desc`, not from the value; all of these carry a `desc`. Only a configuration that is neither machining centre nor lathe (`machineType` = `unknown`) returns status `-20`. Raw vendor access (`gModalGroup/gModal` for one group, `gModalList` for all) works regardless of machine type, since it promises raw vendor numbering and no meaning.

This address was previously named `/machine/channel/gGroup/gModal` (filter `gGroup`); the old address and filter keep working permanently as-is, but the documentation and the dashboard describe only this name.

## /machine/channel/gModalGroup/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalGroup"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

Reads **one slot of `gModalList`, picked by group number** (`string`). The `gModalGroup` filter takes **the group number that control's communication interface uses**, in the same numbering as the array positions of `gModalList`:

- **Fanuc**: the number FOCAS uses, `0`-`36` (the `type` of `cnc_rdgcode`)
- **Siemens**: G-function groups `1`-`N` (N = the machine's group count; `64` on a measured 840D sl)
- **Mitsubishi**: the vendor API's group numbers, `1`-`21`

⚠️ **These are interface numbers, not the `Group` column of the programming manual.** Whether the two agree may differ by machine type, so rather than feeding a manual's group number straight in, read `gModalList` once and confirm the position. Fanuc's numbering starts at `0`.

A number outside the range is rejected with status `-18` carrying the valid range. When no modal is in effect for that group, or the machine does not have that group, the value is **`null`**, exactly as that slot of `gModalList` reads.

The meaning of the group numbers belongs to each vendor and is **not translated; it will not be unified across machine types** (a raw pass-through, like `plcAddress`). To pick a group machine-independently, use `/machine/channel/gModalCategory/gModal`, which takes neutral numbers. The value is that control's G-code string either way, so it cannot be used for machine-independent branching (see `gModalCategory/gModal` for details).

**This address is particularly useful on Mitsubishi.** That control costs one round trip per group, so `gModalList` is 21 round trips while this address is one; and since its group numbering table is M-system only, on a lathe (where `gModalCategory/gModal` returns status `-20`), raw numbers are still readable one at a time here.

## /machine/channel/gModalList
```yaml
value_type: "stringArray"
null_able: true
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

Returns the **full list of modal G codes reported by the machine in vendor order** (`stringArray`). For reproducing a machine-specific HMI's modal screen; the meaning of each group index is per that vendor's manual.

⚠️ **Elements can be `null`**, on every machine type. It means no modal is in effect in that slot, or the machine does not have that group. Code that expects a string will break on it, so check each element.

- **Fanuc**: FOCAS group order `0`-`36`, so **the length is always 37**. Groups the machine does not have read `null` (measured on a 31i: 34 arrive and `24`, `25`, `28` are empty). **On an older control everything from `21` on is `null`**, those groups having been added with the 30i generation
- **Siemens**: `ncFkt` G-function group order 1–N (N = the machine's group count). A group with no G function in effect reads `null` (measured: 7 of 64)
- **Mitsubishi**: vendor group order 1–21 (`GetGCodeCommand`). **The length is always 21**; a group the machine does not have (15, 16 and 21 are M-system only, for instance) or one with no modal in effect reads `null`. Codes are formatted as the vendor manual's examples show, with a two-digit integer part (`G02`, `G50.2`), so they look the same as Fanuc's

**The position in the array is the group number.** Missing entries are filled with `null` rather than skipped precisely to keep that correspondence; packing them forward would seat later entries in someone else's slot.

⚠️ **On Mitsubishi this costs one round trip per group (21).** The vendor API reads one group per call. It is meant for drawing a modal screen once, not for periodic polling; when you need just one group, read it through `gModalGroup/gModal` (one round trip).

**None of the three addresses in this family gives you a machine-independent value.** This list and `gModalGroup/gModal` are raw vendor numbering and order, and even `gModalCategory/gModal` neutralizes only the number you use to select a group; its value is still that machine's G code. **Use none of them for decisions that span machine types** (see the `gModalCategory/gModal` entry for details).

## /machine/channel/axis/machinePosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The axis's machine coordinate. Specify the axis with the `axis` filter; a range (`axis=1-3`) or multiple selection (`axis=1,2`) is possible. The return type is `float` (64-bit double precision).

The four position addresses (machinePosition/workPosition/distanceToGo/relativePosition) are all **real distances**, in the machine's configured unit (mm/inch) as-is and matching the operator-panel display (Fanuc's internal integer representation is normalized by the SDK using each axis's decimal scaling). All four are only valid once the axis has established its reference point; right after power-on, check `/machine/channel/axis/axisReferencedOn` first (before establishment, plausible-looking numbers arrive silently). On Mitsubishi that address returns status `-20`, so check the panel or fall back to `/machine/channel/axis/axisAtReferencePositionOn`, which only tells whether the axis is at the reference position right now.

⚠️ **This value is the coordinate of the tool reference point (the spindle face).** `workPosition` is at the tool tip, so the difference between the two includes **tool length compensation**; `machinePosition − zero offset = workPosition` does not hold (the active work coordinate system's rotation, scaling and mirroring bear on it too). If you need workpiece coordinates, read `workPosition` instead of computing them yourself.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/workPosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The axis's workpiece coordinate (absolute coordinate). Returns `float`.

This value is measured at the **tool tip** and is the result after the active zero offset, rotation, scaling, mirroring and tool length compensation have **all been applied**; it is the final coordinate the machine computed, so there is no need to derive it from `machinePosition` (subtraction does not give the right answer).

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/distanceToGo
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The axis's **remaining travel** in the current block. Returns `float`.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/relativePosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The axis's relative coordinate. Returns `float`.

⚠️ **Its origin is not fixed.** This is a counter the operator can zero at any time (origin set, counter set, or a `G92` preset), so the value alone does not tell you where the machine is. Use `machinePosition` when you need a fixed reference, or `workPosition` for machining coordinates.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/axisName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The axis name (e.g. `"X"`, `"Z1"`). Used to confirm the correspondence between the `axis` number and the actual axis.

## /machine/channel/axis/axisLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The axis (servo) load rate. Returns `float` + `unit:"%"` (identical on all three). Fanuc reads the servo load meter, Siemens the drive load (`$VA_LOAD`, available for PROFIdrive drives only), and Mitsubishi the load current of the servo monitor (a ratio of the rated current). All three report the **present measured value**.

**This is the same physical quantity as `axisCurrent`.** On a servo, torque is proportional to current, so measuring load is measuring current; this address divides that value by the **motor's rated continuous current**. That makes it comparable across machines and axes (80% means 80% anywhere), while `axisCurrent` gives you the absolute figure. Converting between the two needs that motor's rated current, which deemesh does not expose - which is why **some controls support only one of them**.

## /machine/channel/axis/axisLoadCommandedPeak
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_ezsocket_mitsubishi"]
write: []
```

The peak of the axis motor's **current command** over the most recent 2 seconds. **Mitsubishi only.** Returns `float` + `unit:"%"` (converted to continuous current, so the same scale as `axisLoad`).

Two things differ from `axisLoad`: this is the **command**, not the measurement, and it is a **peak**, not the present value. It therefore reads higher than `axisLoad` under the same load; do not compare the two as if they were the same number.

It is useful when polling at a slow interval. `axisLoad` is a present value, so a sample can land low even during a cut, whereas this value is the maximum within the preceding 2 seconds and will not miss the load in between.

## /machine/channel/axis/axisFeedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

The **component** of the actual feed along this axis (**Siemens only**). Returns `float`. Fanuc (FOCAS2) does not provide per-axis feed, so it is unsupported (status `-20`).

It is not the speed at which the tool travels along the path, but that speed resolved onto one axis. So **it keeps changing with direction even under the same command**: with `F1000` in XY, this reads 1000 while only X moves, and about 707 on a 45° diagonal.

Adding the axis components **does not give the path speed** (it is a vector magnitude: not 707+707 but √(707²+707²)=1000). Use this value to see whether a particular axis is hitting its own velocity limit and holding the path back.

The unit follows the machine setting (mm/min or inch/min). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/axisCurrent
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The axis motor current. Returns `float` + `unit:"Ampere"` on both protocols. Fanuc reads the servo axis load current in amperes, Siemens the drive parameter `R0078`.

**This is the same physical quantity as `axisLoad`, in a different unit** (that one is a percentage of the motor's rating). **Mitsubishi does not provide this value in amperes, so it returns status `-20`** there; the same measurement arrives through `axisLoad` as a `%` value.

**Siemens**: the value comes from the drive, so on a channel whose axis has no drive assigned this is status `-20`, a property of the machine's configuration, not a fault.

## /machine/channel/axis/axisWorkAreaLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **positive-direction coordinate of the axis working area limit**. It is a second, setup-level boundary inside `axisSoftLimitPositive` (the fixed limit set at machine installation), in machine coordinates on Fanuc and Mitsubishi and in the basic coordinate system (BCS) on Siemens, and supports both read and write.

**Fanuc**: stored stroke check 2 (parameter `1322`). Depending on configuration this feature means either "stay within" or "keep out" (a chuck barrier, for example), and **this address promises only the stay-within meaning.** On a control configured as a no-entry box (parameter `1300#0` = `0`) the address refuses with status `-20` instead of returning a value - computing "safe if between these" from it would be exactly backwards. If you need the raw boundary, read `/machine/channel/parameter/index/parameterValue?parameter=1322`. **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (stated in the manual). The per-axis enable (`1310#0`) and the modal (`G22`/`G23`) are described under the `...On` addresses below.

**Mitsubishi**: stored stroke limit II (parameter `#8205` `OT-CHECK-P`) - the parameter the manual points to from soft limit I with "to narrow the available range in actual use, use #8204/#8205". It has the same predicate gate as Fanuc, but **per axis** and with the **opposite polarity**: `#8210 OT INSIDE` = `0` (inhibits outside = limit II, stay-within) answers, `1` (inhibits inside = limit IIB, a no-entry box) makes that axis answer with status `-20`. Whether the check is in use is `#8202 OT-CHECK OFF` (`0` = in use), and `#8204` equal to `#8205` disables it - both readable through `/machine/channel/parameter/index/parameterValue`. In lathe G-code lists 6 and 7 a program can change `#8204`/`#8205` and switch the check on with `G22 X_ Z_ I_ K_` and off with `G23` (non-modal, lathe Programming Manual section 21.2), so this value can change while a program runs. On machining centres `G22`/`G23` is a different feature (stroke check before travel: a program-specified no-entry box checked before the move, error `P452`, section 21.1) and unrelated to this address.

**Siemens**: the setting datum `$SA_WORKAREA_LIMIT_PLUS` (SD 43420). This feature always means stay-within, so there is no mode refusal. A violation raises alarm `10730` (during block preparation), `10630` (during motion) or `10631` (in JOG) per the Diagnostics Manual. On the rare control that does not report how channel axes map to machine axes, this address answers with status `-20`. **Writing works only while the channel is in Reset.** Setting data is something Siemens refuses to alter from outside depending on the channel state: while a program is running, stopped, or the control is in emergency stop, the control refuses, raises alarm `4230` "Data alteration from external not possible in current channel state", and this address answers with status `-17` (the error text carries that reason). The Diagnostics Manual explains it as "it is not allowed to enter this data while the part program is being executed (e.g. setting data for working area limitation or for dry run feedrate)"; on the test machine a channel interrupted by an emergency stop refused as well. The soft limit, which is machine data, has no such restriction. Per the List Manual (12/2019) this setting datum is a value in the **basic coordinate system (BCS)** (identical to machine coordinates on a machine without a transformation), takes effect immediately, and has user protection level (7/7). A program can also change it with `G26` (positive direction) / `G25` (negative direction); whether such a change survives a reset depends on machine datum `10710` (`$MN_PROG_SD_RESET_SAVE_TAB`). Under `WALIMOF` the value is ignored even when set. The monitored point is the **tool tip**, so the tool length is taken into account automatically (the radius only with machine datum `21020`), and the check runs in both AUTO and JOG (Basic Functions manual, A3). The coordinate-system-specific working area limitation in WCS/SZS (`WALCS0`-`WALCS10`) is a separate feature and unrelated to this address (Programming Manual section 15.3.2).

**A value is only enforced while the check is on** - how to tell, per control, is collected under the per-direction switch addresses `axisWorkAreaLimitPositiveOn` and `axisWorkAreaLimitNegativeOn` (Fanuc `G22`/`G23` modal, Siemens `WALIMON` plus the switch, Mitsubishi `#8202`).

**No `unit` is attached, because this is a distance.** Millimetres or inches is a machine setting (handled like `axisSoftLimitPositive`). Writing follows the machine's parameter-write enable state; a refusal is status `-17`.

## /machine/channel/axis/axisWorkAreaLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **negative-direction coordinate of the axis working area limit**. It is a second, setup-level boundary inside `axisSoftLimitNegative` (the fixed limit set at machine installation), in machine coordinates on Fanuc and Mitsubishi and in the basic coordinate system (BCS) on Siemens, and supports both read and write.

**Fanuc**: stored stroke check 2 (parameter `1323`). Depending on configuration this feature means either "stay within" or "keep out" (a chuck barrier, for example), and **this address promises only the stay-within meaning.** On a control configured as a no-entry box (parameter `1300#0` = `0`) the address refuses with status `-20` instead of returning a value - computing "safe if between these" from it would be exactly backwards. If you need the raw boundary, read `/machine/channel/parameter/index/parameterValue?parameter=1323`. **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (stated in the manual). The per-axis enable (`1310#0`) and the modal (`G22`/`G23`) are described under the `...On` addresses below.

**Mitsubishi**: stored stroke limit II (parameter `#8204` `OT-CHECK-N`) - the parameter the manual points to from soft limit I with "to narrow the available range in actual use, use #8204/#8205". It has the same predicate gate as Fanuc, but **per axis** and with the **opposite polarity**: `#8210 OT INSIDE` = `0` (inhibits outside = limit II, stay-within) answers, `1` (inhibits inside = limit IIB, a no-entry box) makes that axis answer with status `-20`. Whether the check is in use is `#8202 OT-CHECK OFF` (`0` = in use), and `#8204` equal to `#8205` disables it - both readable through `/machine/channel/parameter/index/parameterValue`. In lathe G-code lists 6 and 7 a program can change `#8204`/`#8205` and switch the check on with `G22 X_ Z_ I_ K_` and off with `G23` (non-modal, lathe Programming Manual section 21.2), so this value can change while a program runs. On machining centres `G22`/`G23` is a different feature (stroke check before travel: a program-specified no-entry box checked before the move, error `P452`, section 21.1) and unrelated to this address.

**Siemens**: the setting datum `$SA_WORKAREA_LIMIT_MINUS` (SD 43430). This feature always means stay-within, so there is no mode refusal. A violation raises alarm `10730` (during block preparation), `10630` (during motion) or `10631` (in JOG) per the Diagnostics Manual. On the rare control that does not report how channel axes map to machine axes, this address answers with status `-20`. **Writing works only while the channel is in Reset.** Setting data is something Siemens refuses to alter from outside depending on the channel state: while a program is running, stopped, or the control is in emergency stop, the control refuses, raises alarm `4230` "Data alteration from external not possible in current channel state", and this address answers with status `-17` (the error text carries that reason). The Diagnostics Manual explains it as "it is not allowed to enter this data while the part program is being executed (e.g. setting data for working area limitation or for dry run feedrate)"; on the test machine a channel interrupted by an emergency stop refused as well. The soft limit, which is machine data, has no such restriction. Per the List Manual (12/2019) this setting datum is a value in the **basic coordinate system (BCS)** (identical to machine coordinates on a machine without a transformation), takes effect immediately, and has user protection level (7/7). A program can also change it with `G26` (positive direction) / `G25` (negative direction); whether such a change survives a reset depends on machine datum `10710` (`$MN_PROG_SD_RESET_SAVE_TAB`). Under `WALIMOF` the value is ignored even when set. The monitored point is the **tool tip**, so the tool length is taken into account automatically (the radius only with machine datum `21020`), and the check runs in both AUTO and JOG (Basic Functions manual, A3). The coordinate-system-specific working area limitation in WCS/SZS (`WALCS0`-`WALCS10`) is a separate feature and unrelated to this address (Programming Manual section 15.3.2).

**A value is only enforced while the check is on** - how to tell, per control, is collected under the per-direction switch addresses `axisWorkAreaLimitPositiveOn` and `axisWorkAreaLimitNegativeOn` (Fanuc `G22`/`G23` modal, Siemens `WALIMON` plus the switch, Mitsubishi `#8202`).

**No `unit` is attached, because this is a distance.** Millimetres or inches is a machine setting (handled like `axisSoftLimitNegative`). Writing follows the machine's parameter-write enable state; a refusal is status `-17`.

## /machine/channel/axis/axisWorkAreaLimitPositiveOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **positive-direction switch of the axis working area limit**. `true` means the value in `axisWorkAreaLimitPositive` is an effective setting; `false` means it is ignored. Read and write.

Like the other `…On` addresses (`singleBlockOn` and friends) this is a **setting state** - the switch is on, which is not the same as the limit being enforced at this instant. **Whether it is actually enforced needs one more look, which differs by control**:

- **Siemens**: this switch (SD 43400) **and** the channel modal `WALIMON` (`WALIMON`/`WALIMOF` in `/machine/channel/gModalList`). While a program has issued `WALIMOF`, nothing is enforced even with the switch `true`. The switch is setting data, so **writing works only while the channel is in Reset**; otherwise alarm `4230` and status `-17`. The manual lists it as `BOOLEAN`, effective immediately, user protection level - it is the very value the operator toggles under the panel's "Parameters" area to switch the working area limitation on and off.
- **Fanuc**: there is no per-direction switch, so this address answers with status `-20`. There is a **per-axis** enable instead, parameter `1310#0` (`OT2x`: `0` = disabled, `1` = enabled) - a bit parameter, so read the byte with `/machine/channel/parameter/index/parameterValue?parameter=1310&index=<axis>` and look at bit 0. Stroke check 2 as a whole is on or off by the `G22` (on) / `G23` (off) modal, visible in `gModalList`, and parameter `3402#7` decides which one the control starts in at power-on; a control without the check 2 option enforces nothing even under `G22`.
- **Mitsubishi**: no per-direction switch either, so status `-20`. Per-axis use is parameter `#8202 OT-CHECK OFF` (`0` = in use), readable via `/machine/channel/parameter/index/parameterValue?parameter=8202`, and `#8204` equal to `#8205` disables the limit.

## /machine/channel/axis/axisWorkAreaLimitNegativeOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **negative-direction switch of the axis working area limit**. `true` means the value in `axisWorkAreaLimitNegative` is an effective setting; `false` means it is ignored. Read and write.

Like the other `…On` addresses (`singleBlockOn` and friends) this is a **setting state** - the switch is on, which is not the same as the limit being enforced at this instant. **Whether it is actually enforced needs one more look, which differs by control**:

- **Siemens**: this switch (SD 43410) **and** the channel modal `WALIMON` (`WALIMON`/`WALIMOF` in `/machine/channel/gModalList`). While a program has issued `WALIMOF`, nothing is enforced even with the switch `true`. The switch is setting data, so **writing works only while the channel is in Reset**; otherwise alarm `4230` and status `-17`. The manual lists it as `BOOLEAN`, effective immediately, user protection level - it is the very value the operator toggles under the panel's "Parameters" area to switch the working area limitation on and off.
- **Fanuc**: there is no per-direction switch, so this address answers with status `-20`. There is a **per-axis** enable instead, parameter `1310#0` (`OT2x`: `0` = disabled, `1` = enabled) - a bit parameter, so read the byte with `/machine/channel/parameter/index/parameterValue?parameter=1310&index=<axis>` and look at bit 0. Stroke check 2 as a whole is on or off by the `G22` (on) / `G23` (off) modal, visible in `gModalList`, and parameter `3402#7` decides which one the control starts in at power-on; a control without the check 2 option enforces nothing even under `G22`.
- **Mitsubishi**: no per-direction switch either, so status `-20`. Per-axis use is parameter `#8202 OT-CHECK OFF` (`0` = in use), readable via `/machine/channel/parameter/index/parameterValue?parameter=8202`, and `#8204` equal to `#8205` disables the limit.

## /machine/channel/axis/axisSoftLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **positive-direction coordinate of the axis soft limit** (stored stroke check 1). It is in machine coordinates and supports both read and write.

"Soft" means the limit is not a physical device such as a limit switch but **a boundary the control enforces by coordinate**. A command beyond this value is blocked by the control with a stroke-limit alarm.

**No `unit` is attached, because this is a distance.** Whether the machine works in millimetres or inches is a machine setting, so check `/machine/channel/gModalCategory/gModal?gmodalcategory=4` (`G21`/`G71`/`G710` = metric, `G20`/`G70`/`G700` = inch). Other distance values such as `machinePosition` are handled the same way.

**Fanuc**: parameter `1320`. The number of decimal places comes from the machine itself, so the value always matches what `/machine/channel/parameter/index/parameterValue?parameter=1320` returns. **What this address returns is area I of stored stroke check 1.** On a machine whose PLC selects area II through the `EXLM` signal (parameters `1326`/`1327`, up to VIII with the area-expansion option), the value in force at that moment is the other one - read it through `parameterValue`. **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (stated in the manual).

**Siemens**: the axis machine datum `$MA_POS_LIMIT_PLUS` (MD 36110, the 1st software limit switch). It is a machine-axis coordinate and applies in every mode once the axis is referenced (after `PRESET` it is off until the axis is referenced again, and modulo rotary axes are not monitored - Basic Functions manual, A3). A violation raises alarm `10720` (during block preparation), `10620` (during motion) or `10621` (resting on the switch in JOG) per the Diagnostics Manual. **While the PLC selects the 2nd switch (MD 36130 `$MA_POS_LIMIT_PLUS2`) through the axis interface signal `DBX12.3`, this value is not the one in force**, and that 2nd value cannot be read through this SDK. **A written value takes effect at NEW CONF level**: it is activated once the axis has stopped and the channels of the mode group the axis belongs to are in Reset (the same procedure as the panel's "Activate MD" or the `NEWCONF` command), so a value written during operation leaves the previous one enforced until then. On the rare control that does not report how channel axes map to machine axes, this address answers with status `-20`.

**Mitsubishi**: parameter `#2014` (`OT+`). The value always matches what `/machine/channel/parameter/index/parameterValue?parameter=2014` returns. If `#2013` and `#2014` hold the same non-zero value the control treats this limit as disabled (stated in the manual). The setup-level fence that narrows the range further is `axisWorkAreaLimitPositive`.

**There is no on/off switch** - this is the installation-level fence that applies at all times once the axis is referenced (nothing like the working area limit's `axisWorkAreaLimitPositiveOn` exists here). What each control has instead: Fanuc can only **select** among check 1 areas I/II (up to VIII with the area-expansion option) through PLC signals (`1300#2` enables the `EXLM` signal, `1301#0` the per-axis, per-direction `+EXLx`/`-EXLx` signals); the check applies after reference return, and only on a machine set to check immediately after power-on (`1311#0`=`1`) does `1300#6` decide whether it runs before reference return as well. Siemens selects the 1st (MD 36100/36110) or 2nd (MD 36120/36130) software limit switch through per-axis PLC signals (`DBX12.2`/`DBX12.3`); Mitsubishi treats the limit as disabled when `#2013` equals `#2014` (same non-zero value).

**Writing follows the machine's parameter-write enable state.** If writing is not enabled, the machine rejects it and you get status `-17`. Setting this value incorrectly can stop an axis short of where it needs to travel, or loosen the protection, so on a real machine read the current value before changing it.

## /machine/channel/axis/axisSoftLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **negative-direction coordinate of the axis soft limit** (stored stroke check 1). It is in machine coordinates and supports both read and write.

"Soft" means the limit is not a physical device such as a limit switch but **a boundary the control enforces by coordinate**. A command below this value is blocked by the control with a stroke-limit alarm.

**No `unit` is attached, because this is a distance.** Whether the machine works in millimetres or inches is a machine setting, so check `/machine/channel/gModalCategory/gModal?gmodalcategory=4` (`G21`/`G71`/`G710` = metric, `G20`/`G70`/`G700` = inch). Other distance values such as `machinePosition` are handled the same way.

**Fanuc**: parameter `1321`. The number of decimal places comes from the machine itself, so the value always matches what `/machine/channel/parameter/index/parameterValue?parameter=1321` returns. **What this address returns is area I of stored stroke check 1.** On a machine whose PLC selects area II through the `EXLM` signal (parameters `1326`/`1327`, up to VIII with the area-expansion option), the value in force at that moment is the other one - read it through `parameterValue`. **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (stated in the manual).

**Siemens**: the axis machine datum `$MA_POS_LIMIT_MINUS` (MD 36100, the 1st software limit switch). It is a machine-axis coordinate and applies in every mode once the axis is referenced (after `PRESET` it is off until the axis is referenced again, and modulo rotary axes are not monitored - Basic Functions manual, A3). A violation raises alarm `10720` (during block preparation), `10620` (during motion) or `10621` (resting on the switch in JOG) per the Diagnostics Manual. **While the PLC selects the 2nd switch (MD 36120 `$MA_POS_LIMIT_MINUS2`) through the axis interface signal `DBX12.2`, this value is not the one in force**, and that 2nd value cannot be read through this SDK. **A written value takes effect at NEW CONF level**: it is activated once the axis has stopped and the channels of the mode group the axis belongs to are in Reset (the same procedure as the panel's "Activate MD" or the `NEWCONF` command), so a value written during operation leaves the previous one enforced until then. On the rare control that does not report how channel axes map to machine axes, this address answers with status `-20`.

**Mitsubishi**: parameter `#2013` (`OT-`). The value always matches what `/machine/channel/parameter/index/parameterValue?parameter=2013` returns. If `#2013` and `#2014` hold the same non-zero value the control treats this limit as disabled (stated in the manual). The setup-level fence that narrows the range further is `axisWorkAreaLimitNegative`.

**There is no on/off switch** - this is the installation-level fence that applies at all times once the axis is referenced (nothing like the working area limit's `axisWorkAreaLimitPositiveOn` exists here). What each control has instead: Fanuc can only **select** among check 1 areas I/II (up to VIII with the area-expansion option) through PLC signals (`1300#2` enables the `EXLM` signal, `1301#0` the per-axis, per-direction `+EXLx`/`-EXLx` signals); the check applies after reference return, and only on a machine set to check immediately after power-on (`1311#0`=`1`) does `1300#6` decide whether it runs before reference return as well. Siemens selects the 1st (MD 36100/36110) or 2nd (MD 36120/36130) software limit switch through per-axis PLC signals (`DBX12.2`/`DBX12.3`); Mitsubishi treats the limit as disabled when `#2013` equals `#2014` (same non-zero value).

**Writing follows the machine's parameter-write enable state.** If writing is not enabled, the machine rejects it and you get status `-17`. Setting this value incorrectly can stop an axis short of where it needs to travel, or loosen the protection, so on a real machine read the current value before changing it.

## /machine/channel/axis/axisTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The axis motor temperature. Returns `float` + `unit:"°C"` (identical on all three). For Fanuc this is diagnosis 308; for Mitsubishi the motor temperature of the servo drive monitor.

**Siemens**: the value comes from the drive, so on a channel whose axis has no drive assigned this is status `-20`, a property of the machine's configuration, not a fault.

## /machine/channel/axis/axisInterlockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

The axis interlock state (`true` = interlock engaged). **Fanuc only**.

## /machine/channel/axis/axisReferencedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

Whether the axis has **established its machine reference point**. Returns `boolean`. **Read-only.**

On an axis where this is `false` the coordinate system is not yet established: **the position addresses (`machinePosition`, `workPosition`, `relativePosition`, `distanceToGo`) can return numbers without a reference; they are not an error.** This is not an error you would notice; the coordinates are returned without a reference, so anything that consumes positions right after power-on should check this value first. Machines with incremental encoders need a reference return before coordinates are valid.

**Machines with absolute encoders do not lose the reference when powered down.** Such an axis reads `true` from the moment the control comes up, so you may never see `false` on them - that is normal, not a fault. You do not need to know which kind you are connected to: checking this value before using a position is the one rule that works for both.

Once established it stays `true` wherever the axis moves; this is **coordinate-system validity**, not the momentary "is the axis at the reference position right now". That momentary state has its own sibling address, `/machine/channel/axis/axisAtReferencePositionOn`.

Fanuc reads the standard CNC→PMC signal ZRF (per-axis bits of `F120`) and Siemens reads `refPtStatus` (both confirmed in our test environments). **Mitsubishi returns status `-20`**: all that control exposes is "is the axis at the reference position right now" (the panel's `#1` mark, the PLC `ZP1n` signals, `GetAxisStatus`) and the zero-point initialization completion of absolute-position systems (`ZSF`), so a latched coordinate-system-established state cannot be produced (in our test environment the bit dropped as soon as the axis was moved after a reference return). That momentary state is what `/machine/channel/axis/axisAtReferencePositionOn` reports, with the same meaning on every control. The Fanuc implementation currently covers up to 16 axes and the first path of a multi-path machine.

## /machine/channel/axis/axisAtReferencePositionOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

Whether the axis is **standing at its 1st reference position right now**. Returns `boolean`. **Read-only.**

It is `true` once a reference return has completed and the axis is still there, and `false` as soon as a move command takes the axis away. It is a **momentary state**, so during machining it is usually `false`, which is normal. It has the same meaning as the reference mark the operator panel puts next to the machine position (the `#1` on Mitsubishi, for example).

**It is a different question from `axisReferencedOn`.** That one asks "is the coordinate system established" (once established it stays `true` wherever the axis goes); this one asks "is the axis at the reference point now" (it turns `false` when the axis moves). Use `axisReferencedOn` to decide whether positions right after power-on can be trusted, and this address when waiting for an axis to come back to its reference point - a tool-change position or a start-up condition, for instance. The Fanuc test machine reporting `ZRF=7` (three axes established) together with `ZP=0` (none at the reference point) is the difference between the two addresses in one picture.

Fanuc reads the standard CNC→PMC signal ZP (per-axis bits of `F094`, "reference position return completion": `1` while the axis is at the reference position, `0` once it has left), and Mitsubishi reads the 1st-reference-position return-completion bits of `GetAxisStatus` (the same value as the PLC `ZP1n` signals and the panel's `#1` mark); both are confirmed in our test environments. **Siemens returns status `-20`**: the NC variables carry no "at reference point" flag (`refPtStatus` is the established state; `refPtBusy`, `refPtCamNo` and `refPtPhase` describe a return in progress). The Fanuc implementation currently covers up to 16 axes and the first path of a multi-path machine, and the 2nd and later reference positions (Fanuc `ZP2`..., Mitsubishi `ZP2n`...) are outside this address.

## /machine/channel/axis/axisEnergyNet
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

The axis's net **energy** (cumulative consumed − cumulative regenerated). **Fanuc only** (diagnosis 4920), `float` + `unit:"Wh"`.

Energy (Wh) is not power (W): W is an instantaneous rate, Wh is an accumulated amount. This value keeps climbing like an odometer, so to get the energy used by one job, **read it before and after and subtract**. For average power in W, divide by the elapsed time: `ΔWh ÷ Δhours`.

## /machine/channel/axis/axisEnergyConsumed
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

The axis's cumulative consumed **energy**. **Fanuc only** (diagnosis 4921), `unit:"Wh"`. Cumulative, so subtract two readings for an interval.

## /machine/channel/axis/axisEnergyRegenerated
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

The axis's cumulative regenerated **energy** (recovered while decelerating). **Fanuc only** (diagnosis 4922), `unit:"Wh"`.

## /machine/channel/spindle/spindleLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The spindle load rate. Returns `float` + `unit`. Siemens and Mitsubishi always carry `unit:"%"`. On Fanuc the vendor response carries the unit itself, so `%` or `rpm` arrives depending on the machine configuration, and in the rare case the vendor reports some other unit code the `unit` key is omitted. Do not assume `%`; read `unit`. On Mitsubishi this is the load item of the spindle monitor.

**When `unit` is `%` this is the same physical quantity as `spindleCurrent`** - the control divides the motor current by its rating to produce a load rate. That makes it comparable across machines (80% means 80% anywhere), while `spindleCurrent` gives you the absolute figure. Converting between the two needs that motor's rated current, which deemesh does not expose, so **some controls support only one of them**. When Fanuc reports `unit:"rpm"` the value is a speed rather than a load and this relationship does not hold.

## /machine/channel/spindle/spindleOverride
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The spindle override (%). Returns `int` + `unit:"%"`. **Fanuc is a channel-common value by default** (the `G30` signals `SOV0` to `SOV7`, binary 0 to 254%, all on reads `0` as the control treats it), so every spindle shows the same value; on a machine with parameter `3713#3` (MSC) and `#4` (EOV) both set it is **per spindle** (spindle 2 `G376`, 3 `G377`, 4 `G378`; 5 and above are status `-20`) (Connection Manual B-64483EN-1 section 11.11). On a multi-path control the path's own signals are read (addresses shift by `1000` per path). Siemens and Mitsubishi give a per-spindle value. **Mitsubishi is read the way the machine builder's ladder selects it** (PLC Interface Manual): with the per-spindle method-selection signal `SPS` (`Y188F`, `+0x60` per spindle) off, the code signals `SP1`/`SP2`/`SP4` (50-120% in 10% steps) are decoded; with it on, the register `R7008` (0-200% in 1% units, `+50` per spindle) is read.

## /machine/channel/spindle/spindleCurrent
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_opcua_siemens"]
write: []
```

The spindle motor current. **Siemens only** (drive parameter `R0078`). Returns `float` + `unit:"Ampere"`.

**This is the same physical quantity as `spindleLoad`, in a different unit** (that one is a `%` of the motor's rating). Fanuc and Mitsubishi do not provide this value in amperes, so it returns status `-20` there; the same measurement is available through `spindleLoad` - though on Fanuc check `unit` first, since it may report `rpm` depending on the machine's configuration.

## /machine/channel/spindle/spindleTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The spindle motor temperature. Returns `float` + `unit:"°C"` (identical on all three). For Fanuc this is diagnosis 403; for Siemens the drive parameter `R0035`; for Mitsubishi the motor temperature of the spindle drive monitor.

## /machine/channel/spindle/spindleSpeedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The spindle **S command value**. Returns `float`. **No `unit` is attached**: what the command means depends on the spindle speed mode (a rotational speed under constant-speed mode, a surface speed under constant-surface-speed mode), and that holds on all three machine types. **Fanuc is the channel modal S value** (the `spindle` filter is ignored; the S command is a channel-level concept); Siemens is the per-spindle `cmdSpeed`, and Mitsubishi the per-spindle S command modal value.

This address was previously named `/machine/channel/spindle/speedCommanded`; the old address keeps working permanently as-is, but the documentation and the dashboard describe only this name.

## /machine/channel/spindle/spindleSpeedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The per-spindle actual speed. Returns `float` + `unit:"rpm"` on all three protocols: Fanuc uses `cnc_acts2`, Siemens the per-spindle `actSpeed`, and Mitsubishi the speed item of the spindle monitor. All three specify the target spindle with the `spindle` filter, and the value is the **measured speed with override applied**. **On Siemens the sign follows the direction of rotation** (the manual defines the sign of `$AA_S` that way); the sign convention of Fanuc and Mitsubishi is not documented and has not been confirmed.

This address was previously named `/machine/channel/spindle/speedActual`; the old address keeps working permanently as-is, but the documentation and the dashboard describe only this name.

## /machine/channel/activeToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The number (`T`) of the tool that is currently active in that channel. `channel` filter. Returns `int`.

**What makes a tool "active" differs by machine type.** On Fanuc and Mitsubishi this is the `T` modal, so it changes **the moment `T` is programmed**; on Siemens it is `actTNumber`, so it changes **only once the change has completed**:

| Situation | Fanuc, Mitsubishi | Siemens |
|---|---|---|
| `T7` programmed, `M06` not yet reached | `7` | still the previous tool |
| after `T7 M06` has completed | `7` | `7` |

The first two were confirmed in our test environments - the value reads `7` immediately after programming `T7` with no `M06`, and on Mitsubishi the control's own tool display was observed still showing the previous tool. A reset (`M30`) does not clear it either.

**So on Fanuc and Mitsubishi this value must not be read as "the tool that is cutting right now"**; it is the commanded tool. If the moment of the change matters, do not use this address as a change signal. Watch the machine's own change-complete signal. The tool actually held in the spindle is not reachable through the SDK on those two: the tool number shown on the control is produced by the machine builder's ladder, so it differs per machine, and it resists neutralization for the same reason `plcAddress` does.

⚠️ **With the Fanuc Tool Management option enabled, this value is not a tool number.** Under that option `T` does not name a tool: it names a **tool type (group) number**, and the control picks an actual tool of that type. Confirmed in a test environment: with the control's `EACH TOOL DATA` listing tool `1` as type `4`, programming `T4` made this address read `4` while the actual tool was `1`. On a machine without the option `T` is the tool number and the question does not arise. If your machine uses the option, do not feed this value straight into the tool tree lookup below. Instead, the entries of `/machine/toolArea/toolList` whose `toolTNumber` equals this value are the candidate tools, and `/machine/toolArea/tool/toolTNumber` answers it per tool.

**Even when a program selects a tool group through tool life management, this value is a tool number.** A program calls a group with a value greater than parameter `6810` (on a machine where `6810` is `1000`, `T1001` is group `1`), and the control picks the tool to use from that group and puts **that tool's number** here. Measured: on a machine whose group `1` starts with tool `16`, commanding `T1001` made this read `16`, and it still read `16` after the tool change. A group number never appears in this value.

The group whose life is currently being counted is reported by `/machine/channel/activeToolGroupNumber`, and that group's tools by `/machine/toolArea/toolGroup/toolNumberList`.

Put this number into the `tool` filter of the tool tree to look up that tool's name, edge count and offsets. The `toolArea` value you also need comes from `/machine/channel/toolAreaNumber` (cached at connection time, so it costs no extra communication).

Fanuc reads the `T` modal, Siemens `actTNumber` (`$P_TOOLNO`: the T number of the tool with which the active D offset was calculated), and Mitsubishi the T command modal from `GetCommand2`.

## /machine/channel/activeToolEdgeNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens"]
write: []
```


The number (`D`) of the edge whose compensation is **currently in effect** on the active tool. `channel` filter. Returns `int`. It is Siemens `actDNumber`. This address answers "which compensation set", not "is a tool loaded" - that is what `activeToolNumber` answers with `0`.

**Siemens only** (Fanuc and Mitsubishi return status `-20`). Neither of those offset models has a per-tool edge (compensation set) layer, so the question "which edge" does not arise. Earlier versions returned a fixed `1` on those two; that asserted a dimension that does not exist, so it was removed. On Fanuc the compensation numbers a program calls are read per tool from `/machine/toolArea/tool/toolHNumber` and `toolDNumber` (tool management option).

## /machine/channel/activeToolGroupNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The tool group whose **life is currently being counted** in this channel. `channel` filter. Returns `int`, read only.

On a machine with tool life management, a program selects a group and that group's life starts counting down; this is that group's number. **`0` when no group is in use.**

Where `/machine/channel/activeToolNumber` is "the tool currently loaded", this is "the group whose life is being spent". They are different things, so read both.

**Requires the tool life management option**; otherwise the address returns status `-20`. That is a **different option** from tool management, despite the similar name, and on every machine seen so far the two were not enabled together.

## /machine/channel/selectedToolGroupNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The tool group whose life is **currently being counted in this channel, or the last one that was counted** if none is running. `channel` filter. Returns `int`, read only.

It differs from `/machine/channel/activeToolGroupNumber` in that it **is sticky**:

| | A group is running | Nothing is running |
|---|---|---|
| `activeToolGroupNumber` | that group | `0` |
| this address | that group (same value) | **the last group that ran** |

That makes it the way to ask "which group did we use last" while the machine sits idle. It resets to `0` over a power cycle.

**Requires the tool life management option**; otherwise the address returns status `-20`.

## /machine/toolArea/toolGroup/currentToolUseOrder
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

**Which tool is next in turn** in that tool group. `toolArea` + `toolGroup` filters. Returns `int`, read only, counting from `1`. It reads `0` for a group that has never been used.

The number matches the panel's position column (`01`, `02`, `03`) and points at the **same position** in `/machine/toolArea/toolGroup/toolNumberList` (a `2` means the second entry).

⚠️ **It does not mean the group is running.** It is a pointer the group holds and it persists: measured, it became `1` at the tool change and was still `1` after a reset and after a power cycle. To reproduce the panel's `@` (in use) marker, show it only while `/machine/channel/activeToolGroupNumber` names that group.

**A machine without the tool life management option, or with no group capacity allocated, returns status `-20`.**

## /machine/channel/nextToolGroupNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The tool group whose life count **starts next** in this channel. `channel` filter. Returns `int`, read only.

When a program selects a group with `T`, the control holds it here; once the group is actually put to use it moves to `/machine/channel/activeToolGroupNumber`. So this is the group that has been **chosen but not yet started**, not a value someone reserves in advance.

**`0` when no group is waiting.**

It comes from the same vendor call as `/machine/channel/activeToolGroupNumber`, so reading both costs one round trip.

**Requires the tool life management option**; otherwise the address returns status `-20`.

## /machine/channel/spindle/spindleEnergyNet
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

The spindle's net **energy** (cumulative consumed − cumulative regenerated). **Fanuc only** (diagnosis 4930), `unit:"Wh"`. Cumulative, so subtract two readings for an interval.

## /machine/channel/spindle/spindleEnergyConsumed
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

The spindle's cumulative consumed **energy**. **Fanuc only** (diagnosis 4931), `unit:"Wh"`.

## /machine/channel/spindle/spindleEnergyRegenerated
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

The spindle's cumulative regenerated **energy**. **Fanuc only** (diagnosis 4932), `unit:"Wh"`.

## /machine/userData/userDataValue
```yaml
value_type: "object"
null_able: false
required_filters: ["userData"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Reads/writes Siemens **global GUD (SGUD)** user variables (currently **OPC-UA (Siemens) only**, NC-wide shared variables). Both read (GET) and write (POST) are supported. Since GUD variables have different types, the return type is `object`; it comes as a **self-describing envelope** `{"type":..,"data":..}` that also tells you what type the value is. The only filter is `userData`; this address covers **NC-wide shared** variables only, so no channel is specified.

**userData**: the format is `SGUD:<name>` or `SGUD:<name>[<index>]`. The prefix is the GUD definition block name (currently `SGUD` is supported; MGUD etc. later). **All indices are exactly as shown on the machine screen (HMI)**: enter the number you see on screen as-is (0-based, automatically converted to the OPC-UA internal number).

- no index → scalar variable
- `[i]` → a single i-th element of a 1D array (if the screen shows `_ARR[3]`, use `[3]`)
- `[i-j]` → the i–j range of a 1D array (both ends inclusive, the same convention as `1-3` in filter expansion)
- `[r,c](column count)` → a single (row, column) element of a 2D array. **Write the brackets as shown on screen** and append only the array's column count in parentheses; if the screen shows columns up to `[0,3]`, use `(4)` (the machine does not report the column count, so it must be entered as well)
- Examples: `SGUD:MYVAR` (scalar), `SGUD:_SC_NCK_ROU_S[1]` (screen notation [1] of a 1D), `SGUD:POS[0-2]` (the 3 elements [0]–[2] of a 1D), `SGUD:_SC_C97[0,1](4)` (screen notation [0,1] of a 4-column 2D)

**type** (inside the envelope) is the element's actual type:

- `BOOL`: true/false
- `CHAR`: character code (0–255 integer)
- `INT`: integer
- `REAL`: real number (the same 64-bit real as Fanuc R/macro variables)
- `STRING`: string

(Structured GUD `AXIS` / `FRAME` are planned for later)

**data** (inside the envelope): if one element (scalar / `[i]` / `[r,c](column count)`), a single value; if an `[i-j]` range, a JSON array:

- scalar/single: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- range: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**Write (POST)**: put the **same object** as the read into the body's `value` (e.g. `{"value":{"type":"REAL","data":42.0}}` → changes only the single cell at screen notation [i]). Since `type` decides the type to write, you write directly without reading first. The number of `data` elements must exactly match the range size (1 if single).

**Note**: on older NCKs without a GUD area, an unsupported error is returned.

## /machine/channel/userData/userDataValue
```yaml
value_type: "object"
null_able: false
required_filters: ["channel", "userData"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Reads/writes Siemens **channel GUD (per-channel SGUD)** user variables (currently **OPC-UA (Siemens) only**). Both the `channel` and `userData` filters are required, and only variables of the channel named by `channel` are addressed; NC-wide shared variables are not reachable through this address.

The return type is `object`: GUD variables differ in type, so the value arrives in a **self-describing envelope** `{"type":..,"data":..}` that also tells you what type it is.

**userData**: the `SGUD:<name>` or `SGUD:<name>[<index>]` format. **Indices are exactly as shown on the machine screen (HMI)**, 0-based (converted to the internal numbering automatically).

- no index → a scalar variable
- `[i]` → one element of a 1D array
- `[i-j]` → the i..j range of a 1D array (both ends inclusive)
- `[r,c](column count)` → one (row, column) element of a 2D array. Write the brackets exactly as shown on screen and append only the array's **column count** in parentheses (the machine does not report it, so you must supply it)
- Example: `?channel=1&userData=SGUD:_SC_C97[0,1](4)` (channel 1, screen notation [0,1] of a 4-column 2D)

**type**: `BOOL` (true/false) · `CHAR` (character code 0-255) · `INT` (integer) · `REAL` (64-bit float) · `STRING`. The structured `AXIS`/`FRAME` types are planned.

**data**: one value for a single element (scalar / `[i]` / `[r,c](column count)`), a JSON array for an `[i-j]` range.

- single: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- range: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**Writing (POST)**: put the **same object** as the read returns into the body's `value` (e.g. `{"value":{"type":"REAL","data":42.0}}`). `type` decides the type written, so no read is needed first. The number of `data` elements must match the range size exactly (1 for a single element).

**Note**: on older NCKs without a GUD area, an unsupported error is returned.

## /machine/toolArea/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The **number of tools registered** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `int`, read-only. It is `0` when no tool is registered.

It equals the number of elements `toolList` returns and reads **the same value from the machine**; it exists so that you need not fetch the whole list when only the count matters. Measured on a machine with 17 tools it is far cheaper than the list (81 ms vs 684 ms).

A nonexistent tool area is rejected with status `-18`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. Without the option there is no "tool" object at all, and the number of compensation registers is a different quantity, so it is not used instead. It counts only the **registered slots** of the tool management table (the `NO.` rows of the TOOL MANAGER screen; the slot count is parameter `13220`): the RGS bit of the tool information, the last letter `R` in the panel's `T-INFO` column. A slot whose registration is off is not counted even if values remain in it, because the control treats it as void data. The slot count `13220` is not the option ceiling but a machine-builder setting (within the 64/240/1000 pairs option range); on our bench 5 of 10 slots were registered.

## /machine/toolArea/toolGroupCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it returns the value `0` (nothing to scan).

The **number of available** tool groups. `toolArea` filter. Returns `int`, read only. Group numbers run `1` to this value, so use it as the upper bound when walking the groups.

**This is how many slots exist, not how many groups are in use.** Most slots are empty and the numbers actually used are sparse: on a machine where this read `64`, only groups `1` and `60` had tools registered. To find which ones are in use, walk the numbers and check whether `/machine/toolArea/toolGroup/toolCount` is greater than `0`.

**It reads `0` when there are no groups to work with.** That covers both a machine without the tool life management option and a machine that has the option but no group capacity allocated. Neither leaves anything to walk, so they are not given separate values.

## /machine/toolArea/toolGroupToolCountList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

How many tools are registered in **each tool group slot**. `toolArea` filter. Returns `intArray`, read only.

**Position `i` is group number `i+1`** (the array starts at `0`, groups start at `1`), and the **length always equals `/machine/toolArea/toolGroupCount`** since both come from the same source.

One request answers two questions:

| | How to read it |
|---|---|
| Which groups hold tools | the positions that are not `0` |
| Where a new group can go | the positions that are `0` |

A `0` means **no tools are registered in that group**. On the machine measured, such slots showed on the panel as `no data`, so they are where a new group would go, but what this value promises is the tool count and nothing more.

For a single group use `/machine/toolArea/toolGroup/toolCount`; this list is the read-them-all form.

**It uses `cnc_rdgrpinfo4`, so older controls return status `-20`.** `/machine/toolArea/toolGroupCount` still works on those.

## /machine/toolArea/exchangeRequiredToolGroupList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The tool groups **whose tools need replacing**, by number. `toolArea` filter. Returns `intArray`, read only, in ascending order.

When every tool in a group has used up its life, the control raises an exchange signal; the groups with that signal show up here. **`[]` when nothing needs replacing**, which is the usual state. It is the same value the panel shows as the tool group exchange request.

Its length deliberately does not match `toolGroupCount`. This address carries only what is **currently raised**, not a property of every slot, much like `/machine/channel/alarmList`. For the whole set of slots use `/machine/toolArea/toolGroupToolCountList`.

Once a group turns up here, `/machine/toolArea/toolGroup/toolLifeStatusList` says which tools are the problem: replace the ones reading `2` (life used up).

**A machine without the tool life management option, or with no group capacity allocated, reads `[]`** since it has no groups to replace either way. A control that does not support the underlying function returns status `-20`.

## /machine/toolArea/registeredToolGroupList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The numbers of the tool groups that **have tools registered**. `toolArea` filter. Returns `intArray`, read only, in ascending order.

**Feed these numbers straight into the `toolGroup` filter**; no arithmetic needed.

`/machine/toolArea/toolGroupToolCountList` carries the same fact but expresses it **by position** (position `i` is group `i+1`). Use this address when you are picking one group to dig into, that one when you want the whole set of slots at a glance. Asking for both costs a single round trip to the machine.

```
registeredToolGroupList   [1, 60]
toolGroupToolCountList    [3, 0, 0, ... , 1, 0, 0, 0, 0]
```

**A machine without the tool life management option, or with no group capacity allocated, reads `[]`.** It uses `cnc_rdgrpinfo4`, so controls without that function return status `-20`.

## /machine/toolArea/toolGroup/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

How many tools are **registered in that tool group**. `toolArea` + `toolGroup` filters. Returns `int`, read only.

**An empty group reads `0`.** A group number outside the machine's range is rejected with status `-18`, so `0` means "that group has no tools", not "no such group".

The upper bound on group numbers is reported by `/machine/toolArea/toolGroupCount`. Not every slot is in use, so walking them mostly yields `0`.

**A machine with no groups to work with returns status `-20`**: either the tool life management option is absent, or it is present with no group capacity allocated.

## /machine/toolArea/toolGroup/toolNumberList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **tool numbers registered in that tool group**. `toolArea` + `toolGroup` filters. Returns `intArray`, read only.

The list is in **use order**, not sorted by number. One group on a real machine read `[16,13,2]`, meaning tool `16` is used first and, once its life runs out, the machine moves on to `13` and then `2`.

An empty group reads `[]`. The item count matches `/machine/toolArea/toolGroup/toolCount`, and asking for both together costs a single round trip to the machine.

The same group's `/machine/toolArea/toolGroup/toolHNumberList`, `/machine/toolArea/toolGroup/toolDNumberList` and `/machine/toolArea/toolGroup/toolLifeStatusList` **always have the same length and the same order as this list.** Line up the same positions and you have one tool's information. Asking for all four still costs a single round trip.

**A machine with no groups to work with returns status `-20`**: either the tool life management option is absent, or it is present with no group capacity allocated.

## /machine/toolArea/toolGroup/toolGroupLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **life allotted to that tool group**. `toolArea` + `toolGroup` filters. Returns `float`; both read and write are supported. Write `{"value": 50}`.

**Life belongs to the group, not to a tool.** A group is a queue of interchangeable tools of the same kind, and only one of them runs at a time, so one limit covers them all. When the running tool reaches the limit, the machine moves to the next tool and the counter restarts from `0`.

**The unit can differ per group, so it rides along in the response's `unit`**: `"min"` under time monitoring, `"count"` under count monitoring. The value is exactly what the operator panel shows and is not converted to seconds. Parameter `6800#2` sets the machine-wide default, but M series can set it per group from the part program, so deemesh asks the group itself.

An empty group reads `0`.

**Writes take whole numbers only** - the control's field is an integer, so a fractional value is rejected with status `-16`. The upper limit is the control's own (the machine measured allows `65535` uses or `4300` minutes); going over is status `-16`. **Only a group with tools registered** can be written.

**A machine with no groups to work with returns status `-20`**: either the tool life management option is absent, or it is present with no group capacity allocated.

## /machine/toolArea/toolGroup/toolGroupLifeUsed
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

How much of that tool group's life has **been used so far**. `toolArea` + `toolGroup` filters. Returns `float`; both read and write are supported. Write `{"value": 0}` to reset the counter.

This is the usage of the **single tool currently running**, not a total across the group. A tool whose turn has not come reads `0`, and a tool that expired reached the limit before the machine moved on.

For the life left, subtract this from `/machine/toolArea/toolGroup/toolGroupLifeTotal`. The unit rule and the status `-20` condition are the same as for that address.

**The write constraints** match `toolGroupLifeTotal`: whole numbers only, within the control's limit, and only for a group that has tools registered.

## /machine/toolArea/toolGroup/toolLifeMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **life monitoring mode** of that tool group. `toolArea` + `toolGroup` filters. Returns `int`; both read and write are supported. Write `{"value": 2}`.

| Value | Meaning | Unit of the life values |
|---|---|---|
| `0` | no monitoring (a slot with no tools registered) | none |
| `1` | time | minutes (`unit` is `"min"`) |
| `2` | count | uses (`unit` is `"count"`) |

It is the **same vocabulary** as `/machine/toolArea/tool/toolLifeMonitorType`; that one is per tool, this one per group. This control has no `3` (wear).

Writes take `1` and `2` only. Going back to `0` means deleting the group, which is not what this address does.

⚠️ **Changing the mode does not convert the numbers.** A group set to `50` uses becomes a group set to `50` minutes. Write `/machine/toolArea/toolGroup/toolGroupLifeTotal` again after changing the mode.

**Only a group with tools registered can be written.** Writing to an empty group returns status `-18`: if the control accepted it, the group itself would be created, which is outside what this address promises.

## /machine/toolArea/toolGroup/toolLifeStatusList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **life status of each tool** in that tool group. `toolArea` + `toolGroup` filters. Returns `intArray`, read only.

| Value | Meaning |
|---|---|
| `1` | still usable |
| `2` | life used up |
| `3` | skipped |
| `0` | no usable tool in that slot |

**This address does not tell you which tool is running.** A tool waiting its turn and the tool currently cutting both read `1`, which means "usable", not "in use". Which group is in use is answered by `/machine/channel/activeToolGroupNumber`.

## /machine/toolArea/toolGroup/toolHNumberList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **length compensation number (H)** of each tool in that tool group. `toolArea` + `toolGroup` filters. Returns `intArray`, read only.

Follow the number to `/machine/channel/toolOffset/toolLengthGeometry` and `/machine/channel/toolOffset/toolLengthWear` for the compensation values themselves. On a lathe this is always `0`.

## /machine/toolArea/toolGroup/toolDNumberList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **radius compensation number (D)** of each tool in that tool group. `toolArea` + `toolGroup` filters. Returns `intArray`, read only.

Follow the number to `/machine/channel/toolOffset/toolRadiusGeometry` and `/machine/channel/toolOffset/toolRadiusWear` for the compensation values themselves. On a lathe this is always `0`.

## /machine/toolArea/toolGroup/toolUseOrder/toolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **tool number at one position** in that group. `toolArea` + `toolGroup` + `toolUseOrder` filters. Returns `int`; both read and write are supported. Write `{"value": 16}`.

`toolUseOrder` is the position within the group, counting from `1`, and matches the panel's position column. `/machine/toolArea/toolGroup/currentToolUseOrder` says which position is next in turn.

**To read them all at once** use `/machine/toolArea/toolGroup/toolNumberList`; same values, same round trip. This address exists so that **one position can be written**.

**A position that does not exist returns status `-18`** (the group's tool count is the bound). If the control accepted it, a tool that was not there would appear, which is outside what this address promises.

## /machine/toolArea/toolGroup/toolUseOrder/toolHNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **length compensation number (H)** of the tool at that position. `toolArea` + `toolGroup` + `toolUseOrder` filters. Returns `int`; both read and write are supported. Write `{"value": 16}`.

Follow the number to `/machine/channel/toolOffset/toolLengthGeometry` and `/machine/channel/toolOffset/toolLengthWear` for the compensation values. On a lathe this is always `0`.

## /machine/toolArea/toolGroup/toolUseOrder/toolDNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **radius compensation number (D)** of the tool at that position. `toolArea` + `toolGroup` + `toolUseOrder` filters. Returns `int`; both read and write are supported. Write `{"value": 16}`.

Follow the number to `/machine/channel/toolOffset/toolRadiusGeometry` and `/machine/channel/toolOffset/toolRadiusWear` for the compensation values. On a lathe this is always `0`.

## /machine/toolArea/toolGroup/toolUseOrder/toolLifeStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **life status** of the tool at that position. `toolArea` + `toolGroup` + `toolUseOrder` filters. Returns `int`; both read and write are supported. Write `{"value": 1}`.

| Value | Meaning |
|---|---|
| `1` | still usable |
| `2` | life used up |
| `3` | skipped |

**Write `1` after fitting a fresh insert**: turning a `2` back into a `1` is that operation.

Writes take `1`, `2` or `3` only. A `0` means "no tool", which is a deletion rather than anything this address promises, so it is rejected with status `-16`.

**The control may refuse the write while it is machining.** If that group is the one in use or the one lined up next, the machine protects itself; the vendor error comes straight through, so try again once the job is done.

**The read-only list** is `/machine/toolArea/toolGroup/toolLifeStatusList`.

## /machine/toolArea/toolGroup/toolUseOrder/toolExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

Whether **a tool occupies that position** in the group. `toolArea` + `toolGroup` + `toolUseOrder` filters. Returns `boolean`; both read and write are supported.

Asking about a position that does not exist is not an error, it reads `false`: this address asks about existence.

**Writing creates and removes the position.** `{"value": true}` creates it, `{"value": false}` removes it. **Writing `true` to a position that already holds a tool is rejected with status `-21` (already exists).** Writing `false` to a position that does not exist succeeds (so resending a removal after a lost response is safe).

A freshly created position has tool number, H and D all `0`. Fill in the number with `/machine/toolArea/toolGroup/toolUseOrder/toolNumber`.

**Creating position `1` in an empty group creates the group itself.** There is no separate address for making a tool group; this is how it is done. The `0` entries in `/machine/toolArea/toolGroupToolCountList` show which numbers are free. A new group starts with no life and the control's default monitoring mode, so follow up with `/machine/toolArea/toolGroup/toolLifeMonitorType` and `/machine/toolArea/toolGroup/toolGroupLifeTotal`.

**Removing the last tool removes the group.** Its life and monitoring mode go with it (measured).

⚠️ **Positions shift.** Removing one pulls every later tool up a place, so **one operation changes what every other position address refers to** (`toolNumber`, `toolHNumber`, `toolDNumber`, `toolLifeStatus`, `/machine/toolArea/toolGroup/currentToolUseOrder`). Re-read the lists between operations when working through several positions.

**Only the position just past the end can be created.** In a group holding three tools that is position `4`; anything beyond would leave a gap and returns status `-18`. Inserting in the middle is not expressible here, because a middle position **already exists**, so writing `true` has nothing to do. To reorder, create at the end and rewrite the numbers.

A group that is full (its entry in `/machine/toolArea/toolGroupToolCountList` has reached the control's limit) returns status `-18`.

## /machine/toolArea/toolList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The list of **every tool registered** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `objectArray`, or an empty array `[]` when no tool is registered.

Element: `{"toolNumber": 16, "toolTNumber": null, "toolName": "BALLNOSE_D8", "toolEdgeCount": 4, "sisterToolNumber": 9, "magazineNumber": 0, "pocketNumber": 0, "toolLocationType": "buffer"}`

Tool numbers are **sparse**: 17 tools may occupy numbers 2 through 18 with no number 1, so trying numbers from 1 upward tells you nothing about what exists. This list is the answer: take an element's `toolNumber` and put it straight into the `tool` filter to query the per-tool addresses.

**The order is ascending `toolNumber`**, the same order on every read. **It is not the order on the machine's screen**: the tool list screen is usually sorted by name and the operator can change the sort, so there is no single "screen order" to match. To display it the way the screen does, sort by `toolName`.

**`toolNumber` is not the `Loc.` (place number) on the machine's screen.** A machine using tool management identifies tools by name and sister number, so this number does not appear in the list screen (it is the `Tool number` field in the tool detail screen). Putting the screen's `Loc.` into the `tool` filter **queries a different tool and still looks successful**. The two numbers coincide for many tools, which makes it hard to notice. That value is `pocketNumber`, and this list carries both so you can check the correspondence.

- **toolNumber**: the tool number; the value to put in the `tool` filter
- **toolTNumber**: the number a program calls the tool with via `T` (Fanuc tool management's `TYPE NO.`; several tools may share it). `null` on Siemens, which calls tools by name
- **toolName**: the tool's name; an empty string on machines that do not use names
- **toolEdgeCount**: the number of offset data sets (not of physical teeth, and **not the highest D number either**, because deleting an edge in the middle leaves a gap, so numbers can exceed the count: see `/machine/toolArea/tool/toolEdgeCount`)
- **sisterToolNumber**: the sister-tool number (what tells apart tools sharing one name; the panel's `ST` column)
- **magazineNumber**: the magazine (tool store) it currently sits in; `0` when outside a magazine
- **pocketNumber**: the pocket inside that magazine; `0` when outside a magazine
- **toolLocationType** is the kind of place: `"magazine"` (in a magazine), `"buffer"` (spindle or tool changer), `"loading"` (load/unload position), `"none"` (no physical place)

The list answers **what exists, what to call it, and where it is**. Measured values such as offsets and wear are per-edge and are not included.

When a value is unavailable the key is not dropped, it is `null` (the three location fields are the exception: they carry the same values as the identically named single addresses). The list is sorted in **ascending tool number**, so reading it twice and comparing is meaningful. A nonexistent tool area is rejected with status `-18`.

The first four fields rarely change, but **the three location fields change whenever a tool moves.** This list is meant to be fetched once to draw a screen; if all you need is the currently active tool, use `/machine/channel/activeToolNumber` rather than polling the list (`activeToolNumber` is supported on every machine type; on Siemens that value is the tool whose change has completed, and on a Fanuc with the tool management option it is the tool's type number, not a `toolNumber` of this list).

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It lists only the registered slots of the tool management table (RGS bit of the tool information, the last letter `R` of the panel's `T-INFO`); `toolNumber` is the slot number (the panel's `NO.` column, up to parameter `13220`). `toolName` is an empty string, and `toolEdgeCount` and `sisterToolNumber` are `null` because Fanuc has neither concept (edge layer, sister tool). The number a program calls with `T` is not this number but the tool's **type number** (the panel's `TYPE NO.`), carried as `toolTNumber` in each item. The three location fields carry the same values as the single addresses (`1` to `4` magazines, spindle and standby positions `"buffer"`, not loaded `"none"`). On a Fanuc without the option the offset table is filled densely from `1`, so there is nothing to enumerate.

## /machine/toolArea/tool/toolName
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The tool's name (SINUMERIK `toolIdent`). `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses).

Returns `string`; both read and write are supported. Write `{"value": "DRILL 10"}`. **Siemens only.** On machines that use tool management, a tool's identity is its name plus its sister-tool (duplo) number, so several tools may share one name. On machines that do not use names, an empty string is normal.

A nonexistent tool is rejected with status `-18`. This address does not create tools. Length and character limits are the machine's to judge, and violations surface as an error.

## /machine/toolArea/tool/sisterToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **sister-tool number** (SINUMERIK `duploNo`, the panel's `ST` column). It tells apart tools that share one name and decides which one is brought in once the preceding tool's life runs out. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses).

**It is not a sequence starting at `1`.** On a measured machine, tools whose name was unique still carried `2`, `5` and `101` here. Do not sort by it or assume a `1` exists.

**A newly created tool starts with this equal to its tool number** (measured: a tool created as `50` came out with sister number `50`). If you intend to use sister tools, write the value you want here after creating it. Leaving it does no harm, but the numbers look haphazard once another tool of the same name is added later.

⚠️ **It is easy to confuse with the `D` column beside it on the panel.** `ST` says **which tool**, `D` says **which cutting edge** of that tool. Where several rows share a name, the same `ST` means several edges of one tool while different `ST` values mean different tools.

Returns `int`; both read and write are supported. Write `{"value": 2}`. **Siemens only.** On machines that use tool management, a tool's identity is its name plus this number, so tools sharing a name are told apart by it.

A nonexistent tool is rejected with status `-18`. A value that is not an integer, or outside `0`~`65535`, gives status `-16`. The effective upper limit comes from the machine configuration, and values outside that narrower range are rejected by the machine.

## /machine/toolArea/tool/magazineNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The number of the **magazine (tool store)** the tool currently sits in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

When the tool is not in a magazine the value is `0`: it is cutting in the spindle, being carried by the changer, sitting at the load/unload position, or registered as data with no physical place. The machine gives such places its own numbers too (Siemens: the internal buffer magazine `9998` and loading magazine `9999`; Fanuc: spindle positions `11` to `14` and standby positions `21` to `24`, and on multi-path controls numbers such as `211` and `221` with the path number in the hundreds place), but deemesh does not emit those and folds them to `0`. They are vendor constants a consumer has no reason to know.

**This is where the tool is now, not where it belongs.** Once the tool is loaded into the spindle the value becomes `0`, and it gets a number again when it returns to the magazine. Which place it originally came from is not what this address answers.

**Writing is not supported.** This value records the physical location, so deemesh does not open it for writing. A record that disagrees with the physical tool could affect later tool changes; move tools through the machine's own tool-management procedure.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the magazine number of the tool management data (`MG` on the operator panel); the spindle and standby positions are not magazines and fold to `0` (`/machine/toolArea/tool/toolLocationType` is `"buffer"`). The spindle and standby positions are not configured on our test bench, so that part rests on the vendor documentation. On Siemens a machine without tool management has no magazine at all, so the value is always `0`.

## /machine/toolArea/tool/pocketNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The number of the **pocket inside the magazine** the tool sits in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

If the magazine number is the apartment building, this is the unit number. Both are needed to pin down a location. When the tool is not in a magazine the value is `0` (cutting in the spindle, being carried by the changer, at the load/unload position, or with no physical place). Numbers the machine gives to such places (e.g. `9998`) are not emitted.

**A turret's stations appear as pockets too.** The machine models turret positions as magazine pockets, so the position a machinist calls "station 3" is pocket `3` here.

**This is where the tool is now, not where it belongs.** Once the tool is loaded into the spindle the value becomes `0`, and it gets a place number again when it returns to the magazine.

**Writing is not supported.** This value records the physical location, so deemesh does not open it for writing. A record that disagrees with the physical tool could affect later tool changes; move tools through the machine's own tool-management procedure.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the pot number of the tool management data (`POT` on the operator panel), and `0` while the tool is in a spindle or standby position. On Siemens a machine without tool management has no magazine at all, so the value is always `0`.

## /machine/toolArea/tool/toolLocationType
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

**What kind of place** the tool is in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `string`, **read-only**. The value is its own meaning, so no separate code table is needed.

| Value | Meaning |
|---|---|
| `"magazine"` | sitting in a magazine (tool store) |
| `"buffer"` | in the spindle or the tool changer, cutting or being moved |
| `"loading"` | at the load/unload position |
| `"none"` | no physical place, only the tool data is registered |

**These four stay the same no matter how many machine types are added.** The machine gives the spindle, the changer and the load/unload position their own magazine numbers too (Siemens: the internal buffer magazine `9998` and loading magazine `9999`; Fanuc: spindle positions `11` to `14` and standby positions `21` to `24`, and on multi-path controls numbers such as `211` and `221` with the path number in the hundreds place), but those are vendor constants a consumer has no reason to know. deemesh folds them into these four, so you can branch on the value without knowing which machine it is attached to.

`"buffer"` does **not** distinguish the spindle from a changer gripper. The machine treats both as the same place. Which tool is in the spindle is answered by `/machine/channel/activeToolNumber`, and that address is supported on every machine type, so overlaying the two tells them apart.

**Writing is not supported.** Moving a tool is the job of a tool move, and changing this value alone would put the bookkeeping out of step with reality.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. Magazine numbers (`1` to `8`, up to eight per Connection Manual B-64483EN-1 section 12.3.1) are `"magazine"`, the spindle and standby positions are `"buffer"`, a tool loaded nowhere is `"none"`, and `"loading"` never occurs because Fanuc has no load/unload position. The spindle and standby positions are not configured on our test bench, so that part rests on the vendor documentation. On Siemens a machine without tool management has no magazine at all, so the value is always `"none"`.

## /machine/toolArea/tool/toolEdgeCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```


The number of **cutting edges (offset data sets)** the tool has. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). It is Siemens `numCuttEdges`.

**This is a count, not the highest number.** Edges normally run from `1` upwards, but deleting a middle edge at the machine panel **leaves a gap and does not renumber the ones after it**: delete `2` out of `1`, `2`, `3` and what remains is `1` and `3` while this value becomes `2`. So do not assume `toolEdge` runs from `1` to this value.

A nonexistent edge is rejected with status `-18` on both read and write, so reading tells you which numbers are real.

**This is not the number of teeth.** The flutes or inserts a cutter physically carries ("a 2-flute ball nose", "a 4-flute end mill") are a different quantity. When several teeth sit at the same length and radius one edge covers them all; measured here, a 4-flute cutter answered `1` while a 2-flute ball nose answered `3`.

A nonexistent tool is rejected with status `-18`. It does not answer that the tool has `0` edges.

**Siemens only** (Fanuc and Mitsubishi return status `-20`). In both of those offset models one offset (set) number *is* one set of compensation values, so there is no per-tool edge layer. Earlier versions returned a fixed `1`; that asserted a dimension that does not exist, so it was removed. Fanuc's per-tool tool management data (`H`, `D`, life) is read from the per-tool addresses under `/machine/toolArea/tool/*`.

## /machine/toolArea/magazineList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **list of magazines** in that tool area. `toolArea` filter. Returns `objectArray`, **read-only**.

**Do not assume magazine numbers are contiguous: use this list.** On Siemens they were measured as `1`, `9998` and `9999` (on Mitsubishi they fall in a `1`-`5` range and are contiguous, but this list works the same either way). On Fanuc only the configured ones among `1` to `4` appear (magazines whose pocket count in parameters `13222`/`13227`/`13232`/`13237` is non-zero; for a matrix-type magazine, parameter `13240`, `pocketCount` is rows x columns, `13241` x `13242` and so on). **On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. Counting from `1` up to `/machine/toolArea/magazineCount` will not find them.

Each entry:

| Field | Meaning |
|---|---|
| `magazineNumber` | the magazine number; pass it straight to the `magazine` filter of `/machine/toolArea/magazine/pocketCount` |
| `pocketCount` | how many pockets that magazine holds |

**Only real tool stores are listed.** Internally the spindle, the changer and the load/unload position are magazines too and carry fixed numbers (`9998` for the buffer, `9999` for loading), but in deemesh "magazine" means a tool store and nothing else. When a tool sits in one of those places, `/machine/toolArea/tool/toolLocationType` answers `"buffer"` / `"loading"` and `/machine/toolArea/tool/magazineNumber` gives `0`.

**It joins directly with tools.** Look up the entry by whatever `/machine/toolArea/tool/magazineNumber` returned, and pass that number straight to `pocketCount`.

**Magazine names are not included.** The machine has a name field, but it means nothing unless it is configured on site. On the machine measured here the 40-pocket magazine, the buffer and the load position all returned **the same string**. Three entries with identical names would make the list look broken, so it is left out.

## /machine/toolArea/magazineCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **number of magazines** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

**Only real tool stores are counted.** Internally the spindle, the changer and the load/unload position are magazines too (counted that way the machine measured here answers `3`), but deemesh does not treat those places as magazines. When a tool sits in one, `/machine/toolArea/tool/toolLocationType` answers `"buffer"` / `"loading"`.

**It gives you the count but not the numbers.** The numbers are not contiguous, so you cannot infer valid magazine numbers from this value. Use `/machine/toolArea/magazineList` when you need the numbers. This address exists so you do not have to fetch the whole list just to get a count.

On Siemens a machine without tool management has no magazine at all, so the value is `0`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. No magazine data exists without the option, so the address does not answer `0`. Fanuc magazines are defined by parameters `13222`/`13227`/`13232`/`13237` (pocket counts of magazines `1` to `4`), and only those with a non-zero count are counted. The spindle positions (`11` to `14`) and standby positions (`21` to `24`) carry magazine numbers but are not magazines, so they are not counted. `toolArea` is a path number, but the Fanuc tool management table is CNC-wide, so any path gives the same value.

**On Mitsubishi the magazine numbers are a fixed `1`-`5` range**, and only those with at least one pocket are counted. The spindle and the standby positions are a separate concept on that control, not magazines, so they never enter this count.

## /machine/toolArea/magazine/pocketCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The number of **places (pockets)** in that magazine. `toolArea` + `magazine` filters (`magazine` is the magazine number). Returns `int`, **read-only**.

**Magazine numbers are not contiguous**: `/machine/toolArea/magazineList` tells you the valid ones. Counting up from `1` to `/machine/toolArea/magazineCount` will not find them.

A number that does not exist is rejected with status `-18`. **The numbers of the spindle, the changer and the load/unload position are rejected too**. The machine gives those places magazine numbers, but deemesh does not treat them as magazines. Range and comma expansion are supported.

**Fanuc**: supported only on machines with the tool management (TOOL MANAGEMENT) option, otherwise status `-20`; a magazine number that is not configured is status `-18` (with the list of configured numbers). The value is parameter `13222`/`13227`/`13232`/`13237`, or rows x columns for a matrix-type magazine (parameter `13240`). Pocket numbers do not start at `1` but at the magazine's **start pot number** (parameter `13223` and so on), so take the pocket range from `/machine/toolArea/magazine/pocketList`.

**Mitsubishi note**: magazine numbers are a fixed `1`-`5` range, so anything outside it is status `-18`, but **a magazine inside the range that does not actually exist answers `0` instead of being rejected**, because on this control the existence probe is the pocket-count query itself. When you need existence, read `/machine/toolArea/magazineList`.

## /machine/toolArea/magazine/pocketList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

**Every pocket in that magazine and what sits in each one.** `toolArea` + `magazine` filters. Returns `objectArray`, **read-only**.

Each entry:

| Field | Meaning |
|---|---|
| `pocketNumber` | the pocket number; every pocket appears. On Siemens and Mitsubishi they run from `1` to `/machine/toolArea/magazine/pocketCount`, on Fanuc from the magazine's start pot number (parameter `13223` and so on, usually `1`) for as many pockets as it has |
| `toolNumber` | the tool in that pocket. **`0` means the pocket is empty** |

**This is the direction the other addresses cannot answer.** `/machine/toolArea/tool/pocketNumber` tells you which pocket a tool is in, but "what is in pocket N" and "where are the empty pockets" are answered only by this list.

Tool names and offsets are not included: `/machine/toolArea/toolList` gives those by number, so join on the number. Reading a name for every pocket would double the values fetched for a 40-pocket magazine.

The numbers of the buffer (spindle and changer) and the load/unload position are rejected with status `-18`: deemesh does not treat those places as magazines.

**Fanuc**: supported only on machines with the tool management (TOOL MANAGEMENT) option (otherwise status `-20`); the magazine management table (`cnc_rdmagazine`) is read one whole magazine at a time. `toolNumber` is the tool management data number (the `NO.` column on the operator panel, the value you pass as the `tool` filter). A neighbouring pocket occupied by an oversize tool and a reserved home pocket (oversize-tool support option and extension B option respectively) hold no tool either and appear as `0`, so when picking an empty place for a tool also check the magazine screen on the operator panel. A matrix-type magazine (parameter `13240`) is numbered the same way, from the same start pot number for rows x columns pockets (Fanuc Connection Manual B-64483EN-1 section 12.3.3: numbered from the upper-left to the lower-right corner as seen from the front of the magazine). Our test bench is chain-type, so the matrix case rests on the documentation.

**Mitsubishi**: magazine numbers are a fixed `1`-`5` range, and a magazine that does not actually exist is rejected with an error.

## /machine/toolArea/magazine/pocket/toolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **tool number** in that pocket. `toolArea` + `magazine` + `pocket` filters. Returns `int`, **read-only**.

**`0` means the pocket is empty** (tool numbers start at `1`). A pocket that does not exist is rejected with status `-18` rather than answering `0`, so the two never blur together. The valid range is given by `/machine/toolArea/magazine/pocketCount`. On Fanuc pocket numbers run from the magazine's start pot (parameter `13223` and so on), so take the range from `/machine/toolArea/magazine/pocketList`.

Use it for a single pocket; to sweep a whole magazine, `/machine/toolArea/magazine/pocketList` does it in one round trip.

The numbers of the buffer (spindle and changer) and the load/unload position are rejected with status `-18`.

**Fanuc**: supported only on machines with the tool management (TOOL MANAGEMENT) option, otherwise status `-20`. The value is the tool management data number (what you pass as the `tool` filter); a neighbouring pocket occupied by an oversize tool and a reserved home pocket appear as `0` too (see `/machine/toolArea/magazine/pocketList`). A matrix-type magazine works the same way (see `/machine/toolArea/magazine/pocketList` for the pocket numbering).

**Writing is not supported.** Overwriting the tool in a pocket would change the bookkeeping while the physical tool stayed put, and the changer would then reach for the wrong pocket at the next tool change. Moving a tool is the job of a magazine command, and deemesh does not expose one.

## /machine/toolArea/tool/toolExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

Whether that tool number is **registered in the tool table**. `toolArea` + `tool` filters. Returns `boolean`. Reading and writing are supported on Siemens and Fanuc.

Asking about a tool that does not exist is not an error. It answers `false`, because the address exists to ask that question.

**Writing creates and deletes the tool** (Siemens and Fanuc). `{"value": true}` creates it, `{"value": false}` deletes it. **Writing `true` to a tool that already exists is rejected with status `-21` (already exists)**: a silent success would let you believe you got an empty new tool and write over someone else's life data, offsets and location. Delete and recreate it, write its values through the individual addresses as-is, or skip it. Writing `false` to a tool that does not exist succeeds (the postcondition "absent" holds as it is, so resending after a lost response is safe).

**The tool number is exactly what you put in the `tool` filter**; the machine does not hand out the next free number. The number space has gaps (on a measured machine, 20 tools occupied `2`-`18` and `100`-`102`, leaving `1` and `19`-`99` free) and **you can fill a gap by naming that number.** Pick one by looking at the `toolNumber` values in `/machine/toolArea/toolList`. Deleting a tool frees its number again, and the same number can be created later.

On Siemens the tool is created with **one cutting edge** and the **standard location type**. The location type decides which magazine places the tool can go into, so leaving it unset means the operator panel finds no place to load it into. deemesh therefore fills it with the same value the panel gives a new tool. Set the name, tool type and offsets afterwards through their own addresses. Until you do, the tool type reads `9999` (not set) and the operator panel shows the type column as empty, but **unlike the location type this blocks neither loading nor use** (the two fields are easy to confuse because both use `9999`).

On Siemens a newly created tool is **empty and has one cutting edge**, and its name is the tool number as a string. Follow up with `/machine/toolArea/tool/toolName` for the name and `/machine/toolArea/tool/toolEdge/*` for the offsets; to add more edges use `/machine/toolArea/tool/toolEdge/toolEdgeExists`. This is the flow that registers presetter measurements without going to the machine panel.

**A tool that occupies a magazine pocket cannot be deleted** (status `-18`; on Fanuc this also covers a tool at a spindle or standby position). The physical tool would stay in the magazine while its record disappeared, throwing off the next tool change, and **deemesh has no address that would put a recreated tool back into that pocket.** Unload it at the machine first. Where it is right now is answered by `/machine/toolArea/tool/toolLocationType`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the slot's registration mark (RGS bit of the tool information, the last letter `R` of the panel's `T-INFO`). Reading a number beyond parameter `13220` (the slot count) is `false`, not an error. A slot whose registration is off is void to the control even if values remain in it (Connection Manual B-64483EN-1 section 12.3.1: with RGS at 0 the data is regarded as not registered even when other items are set), so the other per-tool addresses (`toolHNumber`, life, location and so on) reject that slot with status `-18` (measured on our bench: switching the registration mark on made the same slot `true` at once and raised `toolCount` by one).

Writing on Fanuc: `true` **registers** the slot (`cnc_regtool`). The slot comes out as an **empty record with only the registration mark set** (type number `0`, no life management, H/D/S/F `0`), so no `T` command can pick it. The control supplies no defaults and deemesh invents none; fill it in afterwards through `toolTNumber`, `toolHNumber`, `toolDNumber`, `toolLifeMonitorType` and the life addresses. A slot whose registration is off but still holds values (the panel's `T-INFO` shows `-` while other columns show data) is refused by the control as it stands, so deemesh clears it first (`cnc_deltool`) and then registers it; the leftover values are discarded. A slot beyond `13220` cannot be created (status `-18`). `false` **deletes** the slot (`cnc_deltool`): the whole record is cleared and the control also removes that tool number from the magazine table (Connection Manual B-64483EN-1 section 12.3). The panel's edit lock (`toolDataLockedOn`) does not block this deletion (measured on our bench). Writing `false` to a slot beyond `13220` is not an error, as with reading.

## /machine/toolArea/tool/toolEdge/toolEdgeExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Whether that edge number **exists on that tool**. `toolArea` + `tool` + `toolEdge` filters. Returns `boolean`; both read and write are supported.

Asking about an edge that does not exist is not an error. It answers `false`. Since `/machine/toolArea/tool/toolEdgeCount` gives only a count and the numbers can have gaps, **this address is what tells you which numbers are real.**

**Writing creates and deletes the edge.** `{"value": true}` creates it, `{"value": false}` deletes it. Writing `true` to an edge that already exists is rejected with status `-21` (already exists); writing `false` to an edge that does not exist succeeds.

**Edges are only created in order.** The machine creates the edge at the **lowest free number** (the number is not selectable). So asking for any other number creates nothing and is rejected with status `-18`, and the error text carries the number that would be created next. If you need `D5`, create `3`, `4` and `5` in turn. Where there is a gap, that gap is filled first.

**The first cutting edge cannot be deleted** (status `-18`). It stays as long as the tool does. To remove the whole tool use `/machine/toolArea/tool/toolExists`.

Creating an edge on a tool that does not exist is status `-18`.

**Siemens only.**

## /machine/toolArea/tool/toolUseStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

The tool's **use status**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int` + `desc`. Reading and writing are supported on Siemens and Fanuc. **This is per tool**: it takes no `toolEdge` filter (a tool with several offset data sets has one status for the whole tool).

The value is a machine-independent code defined by deemesh (not a vendor number). Each value is defined by a predicate, **what state the tool is in right now**, and the two machine types return the same value only when that predicate holds:

| Value | Meaning |
|---|---|
| `0` | Outside life management: the tool's life state is "not managed", so it is skipped by the type-number search (Fanuc only; `/machine/toolArea/tool/toolSearchedWhenUnmanagedOn` is the exception) |
| `1` | Unused: has never cut and is not locked |
| `2` | In use: has been used and is not locked |
| `3` | Life expired: the life is used up and the control will not use it |
| `4` | Broken: the control will not use it because it is broken (Fanuc only) |
| `5` | Locked: life remains but someone marked it as not to be used (Siemens only) |

With `3`, `4` or `5` the control will not use that tool: it rejects a program that calls for it, or switches to the sister tool if one is registered (`/machine/toolArea/tool/sisterToolNumber`). Some values occur on one machine type only, but whenever a value occurs it means the same thing. `desc` is prose for a human; branch on the value.

**Fanuc** carries the life state of the tool management data (the panel's `L-STATE`) over as it is: not managed `0`, unused `1`, usable `2`, life expired `3`, broken `4`. A tool the operator locks by hand at the panel is also called `life expired` (`3`) by the Fanuc control, so `5` never occurs. Supported only on machines with the tool management (TOOL MANAGEMENT) option; without it the address returns status `-20`. The LOC bit of the tool information (`/machine/toolArea/tool/toolDataLockedOn`) is a data edit lock and is not mixed in here.

**Siemens** derives it from the tool state bits (`toolState`) and the remaining life: with the lock bit (Disabled) clear, the "was in use" bit decides `1`/`2`; with it set, the value is `3` when life monitoring is on and any cutting edge has `0` or less remaining, otherwise `5`. Reading a locked tool costs one extra round trip for the remaining life. Sinumerik has no broken state, so `4` never occurs, and a tool with monitoring switched off is still selectable, so `0` never occurs either.

**Writing** names the state you want. If the tool is already in that state nothing happens and the write succeeds. The writable values differ per machine type:

- Fanuc: `1`-`4` are written to the life state (`cnc_wrtool2`). Setting `3` may raise the tool change signal (`TLCH`) the moment every tool of the same type number has expired. A tool whose life state is "not managed" is rejected with status `-18` (set `/machine/toolArea/tool/toolLifeMonitorType` to `1`/`2` first). `0` is handled through `toolLifeMonitorType`, and `5` is status `-16` because Fanuc has no such state (write `3`).
- Siemens: `5` sets the lock bit; `1`/`2` clear the lock bit and clear/set the "was in use" bit respectively. `3` is a fact the control derives from the remaining life and cannot be written directly, so it is status `-16` (write `0` to `/machine/toolArea/tool/toolEdge/toolLifeRemaining`, or `5` to lock the tool). `4` and `0` are status `-16` as well.

**A tool that became `3` because its life ran out will return to `3` shortly if you only write `2` back**, since the remaining life (the counter on Fanuc) is unchanged. If you changed the insert, reset `/machine/toolArea/tool/toolLifeUsed` (Fanuc) or `/machine/toolArea/tool/toolEdge/toolLifeRemaining` (Siemens) first. Restoring the state without changing the insert also means cutting with a worn-out edge.

Specifying a nonexistent tool is rejected with status `-18`.

This address replaces `/machine/toolArea/tool/toolDisabledOn` (`boolean`) of 1.1.0. The old address is rejected with status `-12`; a tool whose `toolDisabledOn` read `true` corresponds to `3`, `4` or `5` here.

## /machine/toolArea/tool/toolFixedLocationOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Whether the tool is **assigned to a fixed location**: it always returns to the same pocket. `toolArea` + `tool` filters. Returns `boolean`; both read and write are supported (`{"value": true}` assigns it, `false` releases it).

When `true` the tool goes back to its own pocket after a tool change; when `false` the machine picks a free pocket. This is the `L` column on the machine's magazine screen.

If it is already in that state, nothing happens and the write succeeds. A nonexistent tool is rejected with status `-18`.

**Siemens only.**

## /machine/toolArea/tool/toolOversizedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

Whether the tool is **bigger than one pocket**: a wide tool that needs the neighbouring pockets kept free. `toolArea` + `tool` filters. Returns `boolean`, **read-only**.

On Siemens this is the `Z` column on the machine's magazine screen. It answers only **whether** the tool exceeds one pocket, not how many it occupies.

**Writing is not supported.** Siemens has no dedicated oversized flag. How many pockets the tool takes up to the left, right, above and below is stored separately, so a bare `true` could not decide which direction and how far. Set oversize at the machine panel.

A nonexistent tool is rejected with status `-18`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the big-tool bit (BDT) of the tool information and is read-only. The neighbouring pockets an oversize tool occupies appear as `toolNumber` `0` in `/machine/toolArea/magazine/pocketList` (measured on our bench: setting the bit made it `true`).

## /machine/toolArea/magazine/pocket/pocketDisabledOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Whether the pocket is **marked as not to be used**: damaged, or a place that must stay empty. `toolArea` + `magazine` + `pocket` filters. Returns `boolean`; both read and write are supported (`{"value": true}` disables it, `false` enables it).

When `true` the machine skips this pocket when choosing where to put a tool. This is the `D` column on the machine's magazine screen, and on that screen the cell appears **only on rows for tools that occupy a pocket**, because it is a property of the pocket, not of the tool.

Locking a tool is `/machine/toolArea/tool/toolUseStatus` (value `5`), which is separate. A disabled pocket does not disable the tool sitting in it; move that tool elsewhere and it is usable again.

If it is already in that state, nothing happens and the write succeeds. A nonexistent pocket, and the numbers of the buffer and load/unload positions, are rejected with status `-18`.

**Siemens only.** On Fanuc this lives in the tool management extension B option (`cnc_rdpot_property`) and is not supported yet; the address returns status `-20`.

## /machine/toolArea/tool/toolLifeMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

How the tool's life is **monitored**. `toolArea` + `tool` filters. Returns `int`. Reading and writing are supported on Siemens and Fanuc. Write `{"value": 2}`.

| Value | Meaning | Unit of the life values |
|---|---|---|
| `0` | no monitoring | n/a |
| `1` | time: counts how long the tool actually cut | the control's own unit: minutes on Siemens (`unit` is `"min"`), seconds on Fanuc (`unit` is `"s"`) |
| `2` | count: counts finished workpieces | pieces (`unit` is `"count"`) |
| `3` | wear: watches whether the offset has drifted to its limit | machine setting (mm/inch) |

**These four stay the same no matter how many machine types are added**: they are deemesh's own values, not the machine's, so you can branch on them without knowing which machine you are attached to.

On Siemens the **tool picks one method, and the values are per cutting edge**. That is why this address takes no `toolEdge` filter while the three per-edge life values do. On Fanuc the life values are per tool as well: `/machine/toolArea/tool/toolLifeTotal`, `toolLifeUsed` and `toolLifeWarnLimit` are its partners.

When it is `0` the three life values are rejected with status `-18`: there is nothing to measure on that tool. To switch monitoring on, write the method here first, then put a budget in (Siemens `/machine/toolArea/tool/toolEdge/toolLifeTotal`, Fanuc `/machine/toolArea/tool/toolLifeTotal`; on Fanuc switching the life state (`L-STATE`) on in the tool management screen of the operator panel has the same effect).

A Siemens machine may have several methods on **at once**. In that case this address answers with the first of time → count → wear, and the three life values follow the same order, so the method and the values never disagree. The answers to "does this need replacing" (`/machine/toolArea/tool/toolLifeWarnOn` and `/machine/toolArea/tool/toolUseStatus`) are always exact regardless of method.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is read from the tool management data (`cnc_rdtool`): a tool whose life state is `NO-MNG` (not managed) answers `0`, because the control does not count it even if life values are displayed; a managed tool answers `1` (time) or `2` (count) according to the life-type bit of its tool information. `3` (wear) does not exist in Fanuc tool management, so writing it is status `-16`. **Write rule**: `0` sets the life state to not managed (the life-type bit and the values stay); `1`/`2` set the life-type bit to time/count and, for a tool that was not managed, switch the state on as unused when the life counter is `0`, otherwise as life remaining. For a tool already managed only the type changes (the numbers are not converted when the type changes, so write the life values again).

This address was previously named `/machine/toolArea/tool/toolMonitorType`; the old address keeps working permanently as-is, but the documentation and the dashboard describe only this name.

## /machine/toolArea/tool/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


The **whole life budget** (maximum life) allotted to that tool. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `float`; both read and write are supported. Write `{"value": 20}`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the maximum tool life of the tool management data (`MAX-LIFE` on the TOOL MANAGER screen); the control counts the usage counter (`/machine/toolArea/tool/toolLifeUsed`) up from `0` towards this value, and the remainder is this value minus the usage. On Fanuc tool life belongs to the **tool, one per tool**, not to a cutting edge, so the address is per tool with no `toolEdge` filter.

**The unit is the control's own.** With the monitoring method (`/machine/toolArea/tool/toolLifeMonitorType`) set to time it is **seconds** (`unit` is `"s"`; the panel shows it as hours, minutes and seconds such as `4H 5M 6S`), with count it is `count`. It is not converted to minutes (the Siemens per-edge life is in minutes, so check the response's `unit`). A tool whose life is not managed (`toolLifeMonitorType` `0`) returns status `-18` even if values remain (write the method first).

Writing accepts only whole numbers in the unit of the read (a fractional value is status `-16`); a tool whose life is not managed is status `-18`. An unregistered tool is status `-18`.

**Not supported on Siemens or Mitsubishi** (status `-20`). The Siemens life is per cutting edge and lives at `/machine/toolArea/tool/toolEdge/toolLifeTotal`.

## /machine/toolArea/tool/toolLifeUsed
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


The life the tool **has used so far** (the life counter). `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `float`; both read and write are supported. Write `{"value": 0}`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the life counter of the tool management data (`L-COUNT` on the operator panel): while the tool is in the spindle the control **counts it up from `0` towards the maximum life** (`/machine/toolArea/tool/toolLifeTotal`) (Connection Manual B-64483EN-1 section 12.3.1: an incrementing counter, remainder = maximum minus counter). Subtract it from `toolLifeTotal` when you need the remainder. There is no separate remainder address because this is the value Fanuc provides, and it keeps the number identical to the screen (`L-COUNT`).

The unit is the same as `toolLifeTotal` (seconds `"s"` under time monitoring, `count` under count monitoring). A tool whose life is not managed (`toolLifeMonitorType` `0`) returns status `-18`.

**This is the address you write after changing an insert to reset the counter** (usually `0`). Writing accepts only whole numbers in the unit of the read (a fractional value is status `-16`); a tool whose life is not managed is status `-18`. A tool taken out of service for life-over needs its state restored as well as the counter (see `/machine/toolArea/tool/toolUseStatus`). An unregistered tool is status `-18`.

**Not supported on Siemens or Mitsubishi** (status `-20`). Siemens counts the remainder down, so read `/machine/toolArea/tool/toolEdge/toolLifeRemaining` there.

## /machine/toolArea/tool/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


The tool's **notice life** (warning limit). `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `float`; both read and write are supported. Write `{"value": 30}`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the notice life of the tool management data (`NOTICE-L` on the operator panel, "predictive tool life" in the FOCAS specification): when the remaining life (maximum life minus the counter) reaches this value or less, the control raises the tool life arrival notice signal (Connection Manual B-64483EN-1 section 12.3.1; whether that notice is per tool type or per tool is set by parameter `13200#3`, and, when it is per type, whether the last tool's remaining life or the sum over the tools of that type is compared is set by `13200#2`). With `0` no notice signal is output.

The unit is the same as `toolLifeTotal`. A tool whose life is not managed (`toolLifeMonitorType` `0`) returns status `-18`. Writing accepts only whole numbers in the unit of the read (a fractional value is status `-16`); a tool whose life is not managed is status `-18`, and an unregistered tool is status `-18`.

Fanuc provides no per-tool notice-reached flag, so `/machine/toolArea/tool/toolLifeWarnOn` returns status `-20`; if you need it, compare `toolLifeTotal − toolLifeUsed` with this value.

**Not supported on Siemens or Mitsubishi** (status `-20`). The Siemens warning limit is per cutting edge and lives at `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit`.

## /machine/toolArea/tool/toolLifeWarnOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: []
```

Whether the tool has **reached its warning limit**. `toolArea` + `tool` filters. Returns `boolean`, **read-only**.

When `true` the remaining life has dropped below the warning limit. **The tool is still usable.** Becoming unusable is `3`/`5` of `/machine/toolArea/tool/toolUseStatus`; the two are independent, so the warning can stay `true` while the tool is locked (a tool locked because its life ran out).

Use it to pick out the tools that will need replacing soon. How much is left is answered by `/machine/toolArea/tool/toolEdge/toolLifeRemaining`.

**The life values are per cutting edge while this flag is per tool.** Measuring and acting happen at different levels: an edge does the cutting, but replacement takes the whole tool, and the sister tool the control switches to (`/machine/toolArea/tool/sisterToolNumber`) is per tool as well. So **any** edge crossing its own limit makes this `true`.

**It does not tell you which edge crossed.** If you need that, compare `toolLifeRemaining` against `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit` for each edge. `/machine/toolArea/tool/toolEdgeCount` gives you how many there are.

**The control raises this value, so writing is not supported.** Overwriting it would leave the remaining life untouched, and the next evaluation would put it back.

With monitoring off it is always `false`. A nonexistent tool is rejected with status `-18`.

**Siemens only.**

## /machine/toolArea/tool/toolEdge/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```


The **whole life budget** allotted to that cutting edge. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported. Write `{"value": 6}`.

The control starts the remaining life at this value and counts down. The unit is decided by the monitoring method and is carried in the response's `unit` field (see `/machine/toolArea/tool/toolLifeMonitorType`). With monitoring off the request is rejected with status `-18`.

**The value is exactly what the operator panel shows.** Under time monitoring that means minutes (`unit` is `"min"`), not converted to seconds. This differs from the other time values (`…Duration`, always seconds): tool life is a number the operator reads off the screen, so it has to match the screen.

When counting pieces, **only whole numbers are accepted.** A fractional value makes the machine answer success while leaving the value unchanged, so the SDK rejects it first.

**Siemens only.** On Fanuc tool life belongs to the tool, not to a cutting edge, so it lives at `/machine/toolArea/tool/toolLifeTotal` (in seconds or counts there).

## /machine/toolArea/tool/toolEdge/toolLifeRemaining
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```


The life **left** on that cutting edge. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported. Write `{"value": 6}`.

SINUMERIK counts down, so this value is the edge's life right now. How much has been used is `toolLifeTotal` minus this value.

**This is the address you write after changing an insert to reset the life**, usually to the same value as `toolLifeTotal`. If the tool was locked because its life ran out, this write is the correct way to release it.

The unit is decided by the monitoring method and is carried in the response's `unit` field. With monitoring off the request is rejected with status `-18`; a fractional value while counting pieces is rejected with status `-16`.

**Siemens only.** Fanuc provides an **up-counting usage counter** rather than a remainder, so read `/machine/toolArea/tool/toolLifeUsed` (per tool) and subtract it from `/machine/toolArea/tool/toolLifeTotal` for the remainder.

## /machine/toolArea/tool/toolEdge/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```


The **warning limit**: once the remaining life drops below this value the control raises a warning (`/machine/toolArea/tool/toolLifeWarnOn`, which is **per tool**). `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported. Write `{"value": 3}`.

It buys time to prepare a replacement tool, so it is set lower than `toolLifeTotal`. The unit is decided by the monitoring method and is carried in the response's `unit` field. With monitoring off the request is rejected with status `-18`; a fractional value while counting pieces is rejected with status `-16`.

**Siemens only.** On Fanuc the notice life belongs to the tool and lives at `/machine/toolArea/tool/toolLifeWarnLimit`.

## /machine/toolArea/tool/toolEdge/toolLengthGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **length1 geometry** value (SINUMERIK `DP3`). For turning tools it usually corresponds to the X direction, but the axis correspondence is a rule set by the tool type and active plane, so the SDK does not translate it.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolLengthWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **length1 wear** value (SINUMERIK `DP12`).

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolLength2Geometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **length2 geometry** value (SINUMERIK `DP4`). For turning tools it usually corresponds to the Z direction.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolLength2Wear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **length2 wear** value (SINUMERIK `DP13`).

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolLength3Geometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **length3 geometry** value (SINUMERIK `DP5`).

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolLength3Wear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **length3 wear** value (SINUMERIK `DP14`).

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **cutter radius geometry** value (SINUMERIK `DP6`, from the milling-tool viewpoint). It points at the **same storage** as `toolNoseRadiusGeometry`; which address you use is your declaration of intent. The SDK does not inspect the tool type.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).


**The number can differ from what the machine's screen shows.** This value is a **radius**, as the name says, while tool-list and offset screens commonly display the **diameter (Ø)**. Measured (2026-07): `BALLNOSE_D8` stores `4.0` and the HMI shows `8.000`. deemesh emits what the machine stores and does not multiply by two.

## /machine/toolArea/tool/toolEdge/toolRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **cutter radius wear** value (SINUMERIK `DP15`). Same storage as `toolNoseRadiusWear`.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).


**The number can differ from what the machine's screen shows.** This value is a **radius**, as the name says, while tool-list and offset screens commonly display the **diameter (Ø)**. Measured (2026-07): `BALLNOSE_D8` stores `4.0` and the HMI shows `8.000`. deemesh emits what the machine stores and does not multiply by two.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **nose radius geometry** value (SINUMERIK `DP6`, from the turning-tool viewpoint). Same storage as `toolRadiusGeometry`.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolNoseRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **nose radius wear** value (SINUMERIK `DP15`). Same storage as `toolRadiusWear`.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual section 9.3.5). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **tool-tip position code** (SINUMERIK `DP2`, `1`-`9`). Indicates where the tip lies relative to the nose center during nose-radius compensation; it is a position code, not an angle. Fanuc's tool offset tree has the same concept (`0`-`9`), and **the numbering and `desc` vocabulary are shared**, so values can be compared and reused as-is across machine types (`desc` is attached to `9` only: the manual defines the `1`-`8` orientations by diagram alone, in three variants by machining setup, so they are not shipped). The valid range is `1`-`9` and **`0` is not permitted**: *"The identifier 0 (zero) is not permitted as a cutting edge position"* (SINUMERIK 828D Tools Function Manual).

Returns `int`; both read and write are supported. Write `{"value": 3}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The tool's **type code** (read + write, `int` + `desc`). **Uses the SINUMERIK DP1 code as-is**: the code scheme is an open classification owned by Siemens, so deemesh does not translate it, and the authority is the SINUMERIK tool-management manual (even if the vendor adds codes, the value is passed through as-is). Writes take an integer code `{"value": 500}`, for tool-setup automation, and the NCK judges code validity.

| Family | Meaning | Examples |
|---|---|---|
| `1xx` | milling tools | `120` end mill, `140` face mill, `145` thread milling |
| `2xx` | drill family | `200` twist drill, `240` tap, `250` reamer |
| `4xx` | grinding tools | |
| `5xx` | turning tools | `500` roughing, `510` finishing, `530` cutoff, `540` threading |
| `7xx` | special | `711` probe, `730` stop |

XX "List of tool types" for 840D sl, the *Tools Function Manual* for 828D. The `desc` names follow that list (07/2021 edition), and the same numbers appear on the control's tool list screen.

Known codes come with their meaning in `desc` (`{"value": 500, "desc": "turning roughing tool"}`), and unregistered codes fall back to the first-digit family desc (`{"value": 573, "desc": "turning tool family"}`). **For a code outside the families listed above, the `desc` key is absent entirely** (`{"value": 300}`). We do not invent a meaning we do not have. Do not assume `desc` is always present.

It is also the reference value that determines the length1/2 axis assignment and radius interpretation (cutter/nose) for turning tools (5xx).

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message carries that tool's edge count). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

## /machine/toolArea/tool/toolEdge/toolTipAngle
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **tip angle** of that cutting edge: the point angle of a drill (`118.0`), `90.0` for a centre drill, and so on. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported. Write `{"value": 118.0}`.

**It is not `toolTipDirection`.** The names differ by one word but the values do not match up: that one is a code for **which direction** the tip sits relative to the nose centre, while this one is the **angle** of the tip.

**Tools that carry no angle report `0.0`.** Milling tools are such tools. Measured here, a drill gave `118.0` and a face mill `0.0`. The source is SINUMERIK `DP24`: for drills it is the field the panel (Operate) uses for the tip angle, but **for turning tools the same field is the clearance angle** (cutting edge data table of the tool management manual), so do not read it as a tip angle on a turning tool.

**The `N` column on the machine's screen doubles as this value and `toolTeethCount`.** It shows the angle for drills and the tooth count for milling tools in that one cell. deemesh keeps them apart so that one address never means a different physical quantity depending on the tool type. To find the number from the screen, look at whichever of the two carries a value.

**Siemens only.** A nonexistent tool or edge (D) is rejected with status `-18`.

## /machine/toolArea/tool/toolEdge/toolHNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```


The `H` number a program uses to call up this cutting edge's **length compensation** (ISO dialect). `toolArea` + `tool` + `toolEdge` filters. Returns `int`; both read and write are supported, write `{"value": 5}`.

It is the `5` in a program line such as `G43 H5`. On Siemens it is the number **assigned to** that cutting edge. An `H5` in the program applies the compensation of whichever edge carries the number `5`, independently of the tool. Numbers are meant to be unique and **the machine enforces that** - writing a number already in use surfaces as an error. Only whole numbers are accepted; a negative value is rejected with status `-16`.

`0` means **not assigned**. On a Siemens machine that does not use the ISO dialect every edge is `0`, and there you can read the compensation directly from `/machine/toolArea/tool/toolEdge/toolLengthGeometry`.

**Siemens only.** On Fanuc the `H` number belongs to the **tool**, not to a cutting edge, so it lives at `/machine/toolArea/tool/toolHNumber` (several tools may share a number, and writing it moves the offset row the tool points at).

## /machine/toolArea/tool/toolHNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


The `H` number a program uses to call up this tool's **length compensation**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`; both read and write are supported, write `{"value": 5}`.

It is the `5` in a program line such as `G43 H5`, and it is the **row number in the tool offset table** that the tool points at. To read that row's length geometry and wear, pass this number as the `toolOffset` filter of `/machine/channel/toolOffset/toolLengthGeometry` and `/machine/channel/toolOffset/toolLengthWear`. **Several tools may share the same number** (on our test bench tools `2`, `3` and `5` all use `H=3`). `0` means not assigned.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the `H` of the tool management data (the `H` field of the EACH TOOL DATA screen). On Fanuc this number belongs to the **tool, one per tool**, not to a cutting edge, so the address is per tool with no `toolEdge` filter. An unregistered tool is status `-18`.

**Writing moves the offset row the tool points at, so the compensation actually applied changes** (on our test bench changing `D` from `5` to `6` changed the panel's `GEOM(D)/RAD` from `35.000` to `45.000`; `H` behaves the same). Only whole numbers `0` to `999` are accepted, otherwise status `-16`. To change the compensation values themselves, write to `/machine/channel/toolOffset/*`.

**Not supported on Siemens or Mitsubishi** (status `-20`). The Siemens ISO-dialect `H` is a number assigned to a cutting edge and lives at `/machine/toolArea/tool/toolEdge/toolHNumber`.

## /machine/toolArea/tool/toolDNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


The `D` number a program uses to call up this tool's **radius compensation**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`; both read and write are supported, write `{"value": 6}`.

It is the `6` in a program line such as `G41 D6`, and it is the **row number in the tool offset table**. To read that row's radius geometry and wear, pass this number as the `toolOffset` filter of `/machine/channel/toolOffset/toolRadiusGeometry` and `/machine/channel/toolOffset/toolRadiusWear`. **A tool's `H` and `D` may be different numbers** (the length comes from the row `H` points at and the radius from the row `D` points at). Several tools sharing one number is also normal. `0` means not assigned, and a tool that does not use cutter compensation is commonly `0`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the `D` of the tool management data (the `D` field of the EACH TOOL DATA screen). The number belongs to the **tool, one per tool**, not to a cutting edge, so the address is per tool with no `toolEdge` filter. An unregistered tool is status `-18`.

**Writing moves the offset row this tool points at, so the radius compensation actually applied changes.** Only whole numbers `0` to `999` are accepted, otherwise status `-16`.

**Not supported on Siemens or Mitsubishi** (status `-20`). On Siemens the `D` number is not a radius compensation but the cutting edge number, so this address does not apply; read the radius there directly from `/machine/toolArea/tool/toolEdge/toolRadiusGeometry`.

## /machine/toolArea/tool/toolTNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


The number a program uses to **call this tool with `T`**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`; both read and write are supported. Write `{"value": 10}`.

It is the `10` in `T10 M06`, a sibling of `/machine/toolArea/tool/toolHNumber` (`H`) and `toolDNumber` (`D`): the numbers that correspond to the program's letters.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the tool type number of the tool management data (`TYPE NO.` on the TOOL MANAGER screen). **It is not a tool number but a group tag**: several tools may carry the same number, and when a program calls that number the control picks, among the valid tools carrying it, the one with the least life left (ties go to the spindle position, then the standby position, then the magazine, then the smaller tool number; Connection Manual B-64483EN-1 section 12.3.1). On our test bench tool `1` was `4` and tools `2` to `5` were all `10`. On a machine that gives every tool a different number it looks like a tool number, but that is only how that shop runs it. Writing changes this tool's type number (a whole number `0` to `99999999`, otherwise status `-16`): the tool joins or leaves the group of tools carrying that number, so the candidates a program's `T` picks from change.

**It pairs with `/machine/channel/activeToolNumber`.** On a tool-management machine the value that address reports is this number, so the entries of `/machine/toolArea/toolList` whose `toolTNumber` equals it are the candidate tools; `/machine/toolArea/tool/toolLocationType` answering `"buffer"` narrows down which one is actually in the spindle.

It is not the **kind** of tool (drill, end mill and so on). That is `/machine/toolArea/tool/toolEdge/toolType`, which on Fanuc lives in the geometry table of a separate option.

**Not supported on Siemens** (status `-20`): that control calls tools by name, so the same place is `/machine/toolArea/tool/toolName`. An unregistered tool is status `-18`.

## /machine/toolArea/tool/toolSpindleSpeed
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


The **spindle speed `S` stored per tool** in the tool management data. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int` + `unit:"rpm"`; both read and write are supported. Write `{"value": 1500}` (a whole number).

**The control does not apply it automatically when `T` is called.** The Operator's Manual (B-64484EN section 10) describes it as a machining condition registered in the tool management data that "can be specified directly by coding `S#8411` in a tool change macro (such as `M06`)". It is a **reference value** that takes effect only where the machine builder or the operator wrote the change macro or the part program to run the tool under its registered conditions; on a machine not programmed that way the value sits there without effect. For the actual speed read `/machine/channel/spindle/spindleSpeedActual`, for the commanded one `spindleSpeedCommanded`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the `S` field of the EACH TOOL DATA screen; the FOCAS valid range is `1` to `99999`, so `0` is a field nobody filled in (all `0` on our test bench). An unregistered tool is status `-18`. Writing accepts whole numbers `0` to `99999` only (otherwise status `-16`); any other rejection comes back as status `-17` with the vendor's reason.

**Not supported on Siemens or Mitsubishi** (status `-20`): their tool data has no such field.

## /machine/toolArea/tool/toolFeed
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


The **cutting feed `F` stored per tool** in the tool management data. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`; both read and write are supported. Write `{"value": 250}` (a whole number).

**The control does not apply it automatically when `T` is called.** Like `/machine/toolArea/tool/toolSpindleSpeed`, it is a **reference value** the machine builder's change macro can read through `F#8412` (Operator's Manual B-64484EN section 10); on a machine not programmed that way it has no effect. For the actual feed read `/machine/channel/feedActual`, for the commanded one `feedCommanded`.

**No `unit` is attached.** The FOCAS specification allows mm/min, inch/min, deg/min, mm/rev and inch/rev for this field, and which one a value means is decided by the macro that reads it (the same reason the other feed addresses carry no unit: it depends on machine settings).

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the `F` field of the EACH TOOL DATA screen; the FOCAS range is `0` to `99999999` (all `0` on our test bench). An unregistered tool is status `-18`. Writing accepts whole numbers `0` to `99999999` only (otherwise status `-16`); the unit is whatever the machine's macro expects.

**Not supported on Siemens or Mitsubishi** (status `-20`): their tool data has no such field.

## /machine/toolArea/tool/toolDataLockedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


Whether the tool's **tool management data is locked against editing**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `boolean`; both read and write are supported. Write `{"value": true}` to lock and `{"value": false}` to unlock.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the LOC bit of the tool information (`L`/`U` in the panel's `T-INFO`, "Data access: Locked/Unlocked" in the vendor documentation). **It does not mean the tool must not be used** (that is `/machine/toolArea/tool/toolUseStatus`): it protects the tool's management data from being **changed**, and has no effect on the control's tool search or tool change.

**This lock does not block SDK writes.** On our test bench the life counter, the notice life and the H number were all written successfully through FOCAS while the tool was locked. What the lock blocks is another path such as editing on the operator panel, and whether that is actually blocked could not be confirmed because the bench cannot be operated. On a machine with key protection on (parameter `13204#0`) the write itself may come back as a vendor rejection (status `-17`, reason included).

The write changes only this bit of the tool information word and writes the other bits back as read. If the tool is already in that state nothing is written and the call succeeds. An unregistered tool is status `-18` for both read and write.

**Not supported on Siemens or Mitsubishi** (status `-20`): their tool data has no such lock.

## /machine/toolArea/tool/toolSearchedWhenUnmanagedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```


Whether a tool **whose life is not managed is still included in the `T` search**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `boolean`; both read and write are supported. Write `{"value": true}` / `{"value": false}`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the SEN bit of the tool information (`S`/`-` in the panel's `T-INFO`). When a program calls a type number with `T` the control picks one of the tools carrying that number, and **a tool whose life state is not managed (`L-STATE` `NO-MNG`) is normally left out of the candidates.** With this value `true` such a tool is included without its remaining life being checked (Connection Manual B-64483EN-1 section 12.3.1). It has no effect on a tool under life management (`/machine/toolArea/tool/toolLifeMonitorType` other than `0`).

The FOCAS specification only marks this bit as reserved; its meaning was confirmed from the vendor's Connection Manual and Operator's Manual and on our test bench (the panel shows the bit as `S`, and `cnc_wrtool2` sets and clears it).

The write changes only this bit of the tool information word and writes the other bits back as read. If the tool is already in that state nothing is written and the call succeeds. An unregistered tool is status `-18` for both read and write. The flag belongs to the machine builder's operating policy, so check the machine's tool change procedure before changing it.

**Not supported on Siemens or Mitsubishi** (status `-20`): their tool data has no such flag.

## /machine/toolArea/tool/toolEdge/toolTeethCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **number of teeth** of that cutting edge, the count you mean by "a 4-flute end mill". `toolArea` + `tool` + `toolEdge` filters. Returns `int`; both read and write are supported. Write `{"value": 4}`.

**It is not `toolEdgeCount`.** That one counts the offset data sets the control holds for the tool; this one counts how many physical cutting teeth one such set describes. Measured here, a 4-flute cutter had `1` edge and `4` teeth.

**It is stored per edge.** A single body can carry cutting sections of different diameters, whose tooth counts may differ, so the value belongs to the edge rather than to the tool.

**It does not always match the `N` column on the machine's screen.** That column doubles up: it shows the tooth count for milling tools and the point angle for drills. Measured, a drill returned `0` here while the screen showed `118.0` (the point angle). deemesh keeps the two apart so that one address never means a different physical quantity depending on the tool type.

**Siemens only.** A nonexistent tool or edge (D) is rejected with status `-18`.

## /machine/channel/diagnosis/index/diagnosisValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "diagnosis", "index"]
read: ["nc_focas2_fanuc"]
write: []
```

The value of **one row** of diagnosis data (**Fanuc only**, `float`, read-only). `channel=<channel>&diagnosis=<number>&index=<n>`, the same row model as the parameter corridor: for axis/spindle-dependent diagnoses `index` is the axis/spindle number, and **single-value diagnoses have exactly one row, so use `index=1`** (the same model as `diagnosisValueList`, which returns single values as one-element arrays). An out-of-range index is refused with status `-18` carrying the actual row count. When polling just one axis periodically, this is lighter than reading every row via the List (one call).

## /machine/channel/diagnosis/diagnosisValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["channel", "diagnosis"]
read: ["nc_focas2_fanuc"]
write: []
```

The **value** of an arbitrary diagnosis number (**Fanuc only**, `floatArray`). Axis/spindle-dependent diagnoses return an array of axis-count length; non-dependent diagnoses return a single-element array. Diagnosis data can differ per path (channel), so this address is **channel-scoped**: specify the path with `channel=`; path-common numbers read the same on any channel. Each diagnosis's format (whether it is row-indexed, and how many rows) is determined on the first query with vendor validation and cached per channel, so repeated polling is light. With comma/range expansion like `diagnosis=301,308`, it comes as a nested array with per-diagnosis boundaries preserved.

## /machine/channel/diagnosisSection/diagnosisSubsection/index/diagnosisValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "diagnosisSection", "diagnosisSubsection", "index"]
read: ["nc_ezsocket_mitsubishi"]
write: []
```

The value of **one row** of the NC's internal data (**Mitsubishi only**, `float`, read-only). Address it with a **section number, sub-section number and axis number**: `channel=<channel>&diagnosisSection=<section>&diagnosisSubsection=<subsection>&index=<n>`.

**This corridor is for when you already know the numbers.** The numbering is vendor-owned, so it is **not translated and never will be unified** (the same class as `plcAddress` and `diagnosis`). Fanuc's `diagnosis` takes a single number while this takes two, which is why it is a separate address. The two machines' diagnosis data cannot share one address.

- Frequently used values are exposed as **named addresses** such as `axisLoad` and `spindleLoad`. Prefer those where they exist. This address is the general-purpose corridor for everything else
- `index` is the **row number** (1-based). Depending on the section it is an axis number or a spindle number, and data with no rows has exactly one row, so use `index=1`. An out-of-range index is refused with status `-18` carrying the actual row count
- **Some data is not numeric.** This corridor carries decimal, hexadecimal, real and string data, but this address is `float`, so strings are refused with status `-18` carrying the value that was actually read (hexadecimal display data is an integer underneath and arrives as-is)
- An unknown section or sub-section returns status `-18`

## /machine/channel/parameter/index/parameterValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "parameter", "index"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The value of **one row** of a CNC parameter (**Fanuc only**, `float`, read/write). Specify it with `channel=<channel>&parameter=<number>&index=<n>`, where the number is taken straight from the Fanuc parameter manual. It is **not translated and never will be unified** (a vendor-owned numbering scheme, the same class as the `diagnosis` filter, so no cross-vendor mapping can exist). Parameters can differ per path (channel), so this address is **channel-scoped**. Frequently used parameters are exposed as named addresses such as `partCountActual` and `powerOnDuration`; this address is the general-purpose corridor for everything else.

**Every parameter is treated as an array of rows**. `index` is the row number: the axis number for axis-type parameters, the spindle number for spindle-type ones (same as the row order on the parameter screen, 1-based; e.g. X1=1, Y1=2), and **single-value parameters have exactly one row, so use `index=1`** (the same model as `parameterValueList`, which returns single values as one-element arrays). An out-of-range index is refused with status `-18` carrying the actual row count.

- Bit-type parameters travel as the **whole byte** (a packed integer). Decomposing/composing bits is the caller's job. To change a single bit, do a read-modify-write (which can race with concurrent changes from the operator panel).
- Decimal (real) parameters travel as real numbers with the machine's decimal places applied, and writes are stored with the same number of places.
- **Mitsubishi has parameters that are not numeric** (for example the axis name `#1013` reads `X`). This address is `float` and cannot represent them, so it refuses with status `-18` and carries the string that was actually read. `index` is the axis number for axis parameters and only `1` otherwise.
- Writing an out-of-range value to an integer parameter is refused with status `-16` (the allowed range is included in the error).
- **Write caution**: parameters change machine behavior. If the machine blocks parameter writes, the write is refused as a vendor error (status `-17`), and some parameters require a power cycle after the change. The Fanuc library for Linux does not provide parameter write, so writes return status `-20` there.

## /machine/channel/parameter/parameterValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["channel", "parameter"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

The **all-axes value array** of a CNC parameter (**Fanuc and Mitsubishi**, `floatArray`, read-only). The array length is the **row count** established by vendor validation: axis-type parameters get one element per axis, spindle-type per spindle, and non-axis parameters a one-element array (the same row model as `diagnosisValueList` and the sibling `parameterValue`). Whether a parameter is row-indexed, and how many rows it has, is determined on the first query with vendor validation and cached per channel, so repeated polling is light, and range/comma expansion such as `parameter=6711-6713` returns nested arrays with per-parameter boundaries preserved.

## /machine/ncMemoryPath/entry
```yaml
value_type: "object"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

Information about a single entry at the path (`object`). The key set is **always the same regardless of machine**: when a value is unavailable the key is not dropped, it is `null`.

| Key | Type | When unavailable |
|---|---|---|
| `name` | `string` | n/a |
| `sizeBytes` | `int` | `null` for a folder, or when the size could not be read |
| `modifiedAt` | `string` | `null` for a folder, or when the machine does not provide a modification time (always `null` on Siemens) |
| `isDir` | `boolean` | n/a |
| `comment` | `string` | `null` for a folder, or when the machine does not provide comments (always `null` on Siemens) |

A trailing `/` on the path forces a folder; without it files are searched first. Missing entries are an error.

## /machine/ncMemoryPath/entryList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The list of files and folders inside a folder (**read-only**, `objectArray`). Each element is **exactly the same object** as `entry`: same key set and same `null` convention, so see that table. Sorted folders first, then by name ascending. To create or delete a folder use `directoryExists`.

A folder that does not exist returns status `-18`.

## /machine/ncMemoryPath/entryName
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **name of the entry** that `ncMemoryPath` points at. Reading returns **the name the machine holds for that entry**, and writing performs a **rename**. The read answers for a file or a folder alike, and rejects a path with nothing at it with status `-18`, the same judgement `entry` and `fileExists` make. What comes back is the machine's own name, not the string you asked with, so a difference in spelling shows the machine's version. Writes take `{"value": "new name"}`, and path separators are not allowed, common to files/folders. **A root cannot be renamed**: if `ncMemoryPath` is a single `//name` segment such as `//CNC_MEM`, `//NC`, `//PRG` or an external drive, the request is refused with status `-18` (filter value error) without touching the machine (the same rule as the root-delete refusal of `directoryExists`). It refers to the same "entry" as `entry`/`entryList`.

## /machine/ncMemoryPath/fileExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

Checks whether a **file** exists at the path (read) and declaratively writes the state (write):

- read → `true` if the file exists (`false` if only a folder of the same name exists)
- write `{"value": false}` → delete the file
- write `{"value": true}` → **error**: creating an empty file is unsupported. Create a file with its content via a `fileContent` write

A trailing `/` in the path is ignored (the kind is fixed by the address). For folders, use `directoryExists`.

## /machine/ncMemoryPath/directoryExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

Checks whether a **folder** exists at the path (read) and declaratively writes the state (write):

- read → `true` if the folder exists (`false` if only a file of the same name exists)
- write `{"value": true}` → create the folder (status `-21` (already exists) if it is already there. On the Fanuc data server `//DATA_SV/` the vendor does not report that reason separately, so the write fails with a vendor error)
- write `{"value": false}` → delete the folder (**empty folders only**: an error if it has contents, no recursion). **A root cannot be deleted**: if `ncMemoryPath` is a single `//name` segment such as `//CNC_MEM`, `//NC`, `//PRG` or an external drive, the request is refused with status `-18` (filter value error) without touching the machine (a trailing `/` makes no difference)
- **Mitsubishi does not support creating or deleting folders** (status `-20`). That drive has a fixed directory layout, so folders can be neither created nor deleted. Reading works normally

A trailing `/` in the path is ignored. For files, use `fileExists`.

## /machine/ncMemoryPath/fileContent
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

Reads the **content** of an NC file (download) and writes it (upload: creates if absent, overwrites if present). The value is a string (program text).

- **Fanuc write auto-handling**: if `%` is absent, it is inserted automatically; if there is no leading O number/`<name>`, it is inserted automatically based on the path's file name. The saved file name is **based on the O number/name in the content**
- **Siemens and Mitsubishi write the content verbatim.** Nothing is inserted, and the saved file name is **the file name in the path**: the file is stored under the path even when the O number in the content differs (the opposite of Fanuc). Include `%` or an O number yourself if you need them
- To delete a file, write `false` to `fileExists`

## /machine/plcAddress/plcType/plcValue
```yaml
value_type: "float"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

Reads/writes a **single element of PMC/PLC memory** (Fanuc FOCAS2 `pmc_rdpmcrng`/`pmc_wrpmcrng`, Siemens OPC-UA `/Plc/` node, Mitsubishi EZSocket `ReadDevice`/`WriteDevice`). Both read (GET) and write (POST) are supported; the return type is `float` (one value), and writes are also a single number (e.g. `{"value": 42}`). This address handles **a single element only**; to work with several elements at once, use the list-form address in the same tree. Both the `plcAddress` and `plcType` filters are required.

**plcAddress**: the address format **differs by machine**. Unlike `plcType` this is a **deliberate exception that is not normalized**: Fanuc's `D100` and Siemens's `DB10.DBB56` point into different memory architectures, and the table that maps one to the other is not SDK knowledge but site configuration that depends on how that machine's ladder was written. It will not be unified later either, so if you need to read the same signal across machines, **the host application must keep a per-machine address table**.

- **Fanuc**: the first character is the PMC area, the rest is the byte number (e.g. `R5`, `D100`). Specify a range with `~`, but for this (single) address the range must be **exactly one `plcType` size** (e.g. for word, `D100~D101`)
- **Fanuc** PMC area first characters: `G` `F` `Y` `X` `A` `R` `T` `K` `C` `D` `M` `N` `E` `Z`. The range must be within the same area (`D100~D101` OK, `D100~R101` NG)
- **Fanuc does not take bit addresses (`byte.bit` notation such as `R38.7`).** Read the byte and take the bit out of it: the value of `plcAddress=R38&plcType=2` is the same byte the panel's PMC signal screen shows in its `HEX` column (`0xDC` reads `220`), and bit 7 is `(220 >> 7) & 1`. Writes are byte-wise as well. Reason: FOCAS's PMC read/write functions have no bit type, so a bit write would have to read the byte, change it and write it back, overwriting any other bit the ladder changed in between. Signals are named `byte.bit` (e.g. `R0039.4`), but give the address as the byte
- **Siemens**: write it **exactly as it appears on the operator panel's `NC/PLC variables` screen**. The value is passed to the `/Plc/{address}` node. If the subscript is omitted, `[1]` is attached automatically. This address takes a single element only. For multiple elements it errors and points you to the list-form address
- **Siemens** forms (**the address carries its own offset**): `DB<n>.DBB<offset>` (byte) · `DB<n>.DBW<offset>` (word) · `DB<n>.DBX<byte>.<bit>` (bit) · `IW<n>` · `MB<n>` · `Q<byte>.<bit>`. Notation examples: `DB10.DBB56` · `DB31.DBX24.1` · `IW0` · `Q0.2`
- **Siemens** the subscript `[N]` is a **count, not an index**. `DB10.DBB56[4]` means **4 consecutive elements** starting at offset 56 (56·57·58·59), not "the 4th of 56". To reach a different location, **move the address**, not the subscript (`DB10.DBB61`)
- **Siemens** syntax caution (machine-independent): a form with no offset (`MB` alone · `DB<n>` alone) is not valid syntax, and a bit is addressed with `DBX`, not `DBB`
- **Siemens**: **which blocks and bytes actually exist is a property of that machine's ladder** and differs from machine to machine. The notation examples above show the shape only. They are not addresses every machine has. Check on that same panel screen: if a value shows there, it reads here
- **Siemens** 828D limit: 828D can only reach **customer data blocks from `DB9000` upward** (840D sl has no such limit)
- **Mitsubishi**: `<device><number>` exactly as the control's PLC screen shows it (e.g. `R100`, `M50`, `Y8A0`). A point count is added as an **`[N]` subscript** and, as on Siemens, `[N]` is **"how many", not "which one"** - `R100[4]` is 4 consecutive points starting at `R100`. This (single) address takes no subscript, or `[1]`
- **Mitsubishi**: the number base differs by device family - `M`, `R` and `D` are decimal while `X`, `Y` and `B` are **hexadecimal** (the same as on the control's screen)
- **Mitsubishi** alignment: devices numbered **by bit**, such as `M`, `X` and `Y`, need the head number on an **8-point boundary** for byte, word and dword access (`Y890` yes, `Y894` no). Word-addressed devices such as `R` and `D` have no such constraint. A misaligned address returns status `-18`; deemesh does not decide this from a table of its own but **asks the machine**, so any address that machine accepts goes through

**plcType**: a numeric code that decides how to interpret the raw bytes. It is a **machine-independent unified value**, so any vendor uses the same number (the adapter translates it to each vendor's code):

- `1` = bit: 1 bit (0 / 1)
- `2` = byte: 8-bit integer (unsigned, 0–255) · address width 1 (e.g. `D100`)
- `3` = word: 16-bit integer (signed) · address width 2 (e.g. `D100~D101`)
- `4` = dword: 32-bit integer (signed) · address width 4 (e.g. `D100~D103`)
- `5` = float32: 32-bit real · address width 4 (e.g. `D100~D103`)
- `6` = float64: 64-bit real · address width 8 (e.g. `D100~D107`)

`0` = **auto**: the source decides the type. Protocols where the node knows its type, like Siemens (OPC-UA), read with that native type. In contrast, protocols that address raw memory, like **Fanuc**, have no intrinsic type, so `0` (auto) is an error and it must be specified. **Fanuc (FOCAS2)** reads PMC memory in byte units, so `1` (bit) is unsupported as well. Specify one of `2` (byte)–`6` (float64).

**Important (Fanuc)**: the byte count of the `plcAddress` range must match the `plcType` size (e.g. `plcType=3` (word, 2 bytes) with a single `D100` address fails → specify `D100~D101`). `plcType` decides **only the interpretation**, and the result is returned as `float` (a JSON number).

**Siemens** has the type encoded in the address itself (`DBB`/`DBW`/`DBD`, etc.), so `plcType=0` (auto) is recommended, and putting in `1`–`6` behaves the same (it reads with the type the server reports). Writes read the node first to confirm the server type, then write with the same type.
**Mitsubishi** accepts four types only: `1` (bit), `2` (byte), `3` (word) and `4` (dword). `0` (auto) is out for the same reason as on Fanuc (raw memory has no intrinsic type), and `5`/`6` (floating point) because this control's PLC device API carries integers only. All three return status `-18`, and the address itself still works. `3` (word) and `4` (dword) are read as **signed** integers: a word with every bit set is `-1`, not `65535`.

**Error codes**: an address that **does not exist on the machine** also returns status `-18` (invalid filter value); which blocks and bytes exist depends on that machine's ladder, so check the same screen on the operator panel first. A `plcType` the machine cannot use returns status `-18` too. A value outside the spec (other than `0`~`6`) returns the same status `-18`, and both call for the same fix: pick another `plcType`. It is not status `-20` because **the address itself works on that machine**. status `-20` is reserved for "this address cannot be used on this machine". The error string carries the accepted values.

## /machine/plcAddress/plcType/plcValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

Reads/writes a **block of PMC/PLC memory elements** as an array. The filters, address format, and `plcType` rules are the same as `plcValue` (single) above; the only difference is that it handles **multiple elements**. The return type is `floatArray`, and the write `value` is a number array `[1, 2, ...]`. Even a single element must be written as an array like `[42]`.

- **Fanuc**: the range's byte count must be a **multiple** of the `plcType` size, and the element count = byte count ÷ type size (e.g. `D100~D107` + word = 4 → `[v1,v2,v3,v4]`)
- **Siemens**: multi-element subscripts are allowed. `[N]` is a **count**. `DB10.DBB56[4]` returns **4 consecutive elements** starting at offset 56 as an array. The elements the server gives become the array as-is
- **Siemens**: omitting the subscript or giving `[1]` still returns **an array** (`[131.0]`). This address always returns `floatArray`, so a single element does not change the shape. Use it whenever the count varies or is not known in advance, and your parsing code never has to branch
- **Mitsubishi**: `[N]` is a count (`R100[4]` -> an array of 4). The maximum number of points in one read depends on the type - bit and byte `1280`, word `640`, dword `320`. Beyond that is status `-18`
- **Mitsubishi**: `plcType=2` (byte) **cannot write a single point** (status `-18`). This control's single-device write call has no byte type, and its block write takes 2 points or more, which would change the neighbouring device as well. Use `3` (word), or this list address with 2 points or more. **Reading a single point is fine**
- Writes require the **element count to exactly match the target range/node's element count**

**Error codes**: an address that **does not exist on the machine** also returns status `-18` (invalid filter value); which blocks and bytes exist depends on that machine's ladder, so check the same screen on the operator panel first. A `plcType` the machine cannot use returns status `-18` too. A value outside the spec (other than `0`~`6`) returns the same status `-18`, and both call for the same fix: pick another `plcType`. It is not status `-20` because **the address itself works on that machine**. status `-20` is reserved for "this address cannot be used on this machine". The error string carries the accepted values.

## /machine/ncMemorySizeTotal
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The total NC memory capacity. Returns `int` + `unit:"bytes"`.

All sizes in the SDK are in **bytes**, the same unit as `sizeBytes` in `entry`/`entryList`, so a question like "does this file fit in the free space" needs no conversion. When a machine only reports a coarser unit (KB, say), this address still returns bytes, and the value is then a multiple of that unit.

What it refers to is the **machining-program memory**. On Mitsubishi it is the same figure as `기억용량` + `나머지` on the control's edit screen; on Siemens it is the user area of the NC's **passive file system** (where main and subprograms, workpieces and GUD definition files live; Extended Functions manual, S7).

## /machine/ncMemorySizeUsed
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The NC memory usage. Returns `int` + `unit:"bytes"`.

## /machine/ncMemorySizeFree
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The free NC memory capacity. Returns `int` + `unit:"bytes"`.

**On Mitsubishi this value moves in steps of 250 bytes**, because the control counts the remainder in units of 250 characters. The unit is bytes as on every other machine type; only the granularity differs.

## /machine/ncMemoryRootPath
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The root path of the main NC memory, the starting point for paths to put in the `ncMemoryPath` filter. Fanuc is usually `//CNC_MEM`, Siemens `//NC`, and Mitsubishi usually `//PRG`.

**On Mitsubishi this value lists nothing on its own.** Programs live one level below, so browse `{root}/USER` (the MDI buffer is `{root}/MDI`).

## /machine/ncMemoryExternalRootPathList
```yaml
value_type: "stringArray"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The list of **external storage drives** other than the main NC memory root (e.g. data server, memory card). Return type `stringArray`.

- Each item, like the root, has a leading `//` (no trailing slash): Fanuc `//DATA`/`//MEMCARD`, Siemens `//Local drive`, Mitsubishi `//IC1` (the NC-side SD card, shown as `DS` on the operator panel). The array is empty when none is fitted
- Names are the **machine HMI wording**. The Siemens local drive is called `NCExtend` internally in OPC-UA, but it is reported as `//Local drive` to match the operator panel (requests using the old `//NCExtend` are still accepted)
- The main root itself is excluded from this list
- **Not cached**: external devices can change by connect/disconnect, so it is re-queried on each request
- No filter
