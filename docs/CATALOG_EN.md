## /machine/configuredProtocol
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The **protocol identifier** this connection uses: one of `"nc_focas2_fanuc"`, `"nc_opcua_siemens"`, `"nc_ezsocket_mitsubishi"`, `"nc_dnc_heidenhain"`. No filters. Returns `string`, read-only. It is fixed at connection time, so once connected it answers immediately with no NC communication (while disconnected it returns status `-10` like any other address; the connection check comes first).

Like `configuredMachineName`, this is a **value from the configuration**, not something the machine reports. It returns the `protocol` field of `deemesh_create` (or of the machine entry in the hub's `machines.json`) as-is, which is why the address says `configured`.

**Its purpose is narrow.** Most addresses are designed to hide the machine type, so no branching is needed. This value is for the **few places where the value space belongs to the machine type**: PLC address syntax (`D100` vs `DB10.DBB56`), diagnosis numbering, tool type codes, and the like, which the catalog explicitly marks as machine-dependent.

**Do not use it to decide whether something is supported.** "This machine type cannot do that address, so skip it" must be decided from status `-20`. Branching on this value means your code keeps skipping even after support is added for that machine type.

## /machine/configuredMachineName
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

Returns the `machine_name` from the hub's `machines.json` or the `deemesh_create` configuration as-is. It is a **value from the configuration**, not a name the machine reports; hence `configured` in the address. Use it to confirm a connection reached the intended machine, or to label a response.

## /machine/cncModel
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The CNC model string.

- **Fanuc**: series number string: `"15"`, `"16"`, `"18"`, `"21"`, `"30"`, `"31"`, `"32"`, `"35"`, `"0"` (0i), `"PD"`/`"PH"` (Power Mate i), `"PM"` (Power Motion i). `desc` also carries the series name (e.g. `"31"` → `Series 31i`). **Where the control reports its model generation, that letter is appended to `desc`** (e.g. `Series 31i-B`, `Series 0i-F`). Controls without generation information get no letter (0i-A/B/C, 30i-A and earlier series). `value` is the same either way. `desc` is a display string whose wording may change, so do not branch on it for the generation
- **Siemens**: the model name as-is (e.g. `"840D sl"`). When the control's NCK type could not be read at connection, or is a type deemesh does not know, the value is `"UNKNOWN"`
- **Mitsubishi**: the NC system S/W number and name string (vendor `GetVersion`). A control without it answers with status `-20`; that is the case on simulators, which carry no real hardware identity. No `desc`
- **Heidenhain**: the model name the control reports for itself, spelled as given (e.g. `"TNC7"`; the NC software entry of the software list HEIDENHAIN DNC provides). It is read from the control, not taken from the connection's `system_type`, once at connection. `desc` carries the NC software number (e.g. `NC software 817625 17 SP4` on the programming station of our test environment). It is the same information as Control model and NC-SW under General information in the control's settings (the TNC7 User's Manual, 'Software', lists `817625` as the programming station's number). When that entry is not found the answer is status `-20`

## /machine/machineType
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
codes: [{"value": "machiningCenter", "name": "Machining center"}, {"value": "lathe", "name": "Lathe", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]}, {"value": "punchPress", "name": "Punch press", "read": ["nc_focas2_fanuc"]}, {"value": "laser", "name": "Laser", "read": ["nc_focas2_fanuc"]}, {"value": "wireCut", "name": "Wire cut", "read": ["nc_focas2_fanuc"]}, {"value": "unknown", "name": "Unknown"}]
```

The machine type. Returns a `string` whose value describes itself. The full set of possible values:

- `"machiningCenter"`: machining center (Fanuc M/MM, Siemens M, Mitsubishi `…M` series, Heidenhain TNC7, TNC 640 and TNC 620)
- `"lathe"`: lathe (Fanuc T/TT/MT, Siemens T, Mitsubishi `…L` series)
- `"punchPress"`: punch press (Fanuc only)
- `"laser"`: laser (Fanuc only)
- `"wireCut"`: wire cut (Fanuc only)
- `"unknown"`: could not be determined; it can occur on every control whenever the machine type does not map to one of the values above (on Mitsubishi, a configuration whose `system_type` token carries no `…M` or `…L`, so we cannot tell which it is; on Heidenhain, see the paragraph below)

**This address describes the machine as a whole.** On a Fanuc mill-turn, where paths differ, it reports the name the control gives the whole machine (the control has a fixed name for each kind of machine: a milling series with two paths is `MM`, so `"machiningCenter"`, while a turning series with two or three paths is `TT` and a turning series with compound machining is `MT`, both `"lathe"`). Behaviour that differs per path, such as the G modal tables and the tool offset columns, is handled by deemesh with **that path's own type** and therefore moves independently of this value.

On Mitsubishi this value comes from the configured `system_type`. That is not the same as trusting the configuration: the vendor defines `…M` as a machining center system and `…L` as a lathe system, and **validates that distinction at connect time**: pointing a `…L` type at a mill is refused outright. So a successful connection is the control confirming this value.

On Heidenhain this value is decided from **the model name the control reports** (the value of `cncModel`). It does not use the configured `system_type`, because the TNC7 in our test environment also accepted connections with other `system_type` values, so the setting alone does not identify the machine. TNC7, TNC 640 and TNC 620 are `"machiningCenter"` because Heidenhain's documentation (document ID 1080370-04, §1.1) classifies them as milling controls. For TNC 320 and TNC 128 we have not confirmed that classification in Heidenhain's documentation, so the answer is `"unknown"`, as it is for any other model we have not tested; use `cncModel` to see which control it is. A control whose `cncModel` answers with status `-20` (where deemesh did not find the model entry in the control's software list) is also `"unknown"`.

## /machine/currentDateTime
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The machine's **current date/time**. Returns a `string`, ISO 8601 to the second (`"2026-07-11T14:30:00"`).

- **Machine-local clock** (Fanuc, Siemens, Mitsubishi, Heidenhain): since there is no timezone information, no TZ suffix (`Z`/`+09:00`) is appended. It is the ISO 8601 local-time form, and an RFC 3339 parser, which requires an offset, may reject it
- **Do not hand it straight to JavaScript's `new Date()`**: a date-and-time without an offset is interpreted in **the viewer's** time zone. This value is the **machine's** wall clock, not the viewer's
- This is the **CNC's clock**, not the server PC's clock; if the machine's clock is off, it is reflected as-is
- Fanuc: `cnc_gettimer` / Siemens: `sysTimeBCD` / Mitsubishi: `GetClockData` / Heidenhain: the basic PLC program's date and time symbols
- **Heidenhain reads the basic PLC program's date and time symbols.** It is the time set on the control, so no `Z` is appended (in our test environment the value followed when the control's time zone was changed; the time zone and time are set under Operating system, Date/Time in the control's settings, see the TNC7 User's Manual, 'Adjust system time window'). It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20` and `error` says which. This status `-20` is not a link problem (when the link is down, the answer is status `-10`, `-14` or `-17`). If deemesh finds none of the date and time symbol names it knows on the machine, the status is also `-20` (symbol names can differ between machines' PLC programs; if you know the names used on that machine, read them with `/machine/plcAddress/plcText`)
- **Recommended health-check address**: a cheap read that triggers an actual NC round-trip on all protocols, so use it for polling and then judging `status` (`0` = normal, status `-10`, status `-14` and status `-17` (connection failed) = link problem) to monitor per-machine communication state. (Cache-served addresses like `machineType` may succeed even when the link is dead, making them unsuitable)

Time-related addresses are always ISO 8601 strings (`…At` = event moment, `…DateTime` = clock reading).

## /machine/powerOnDuration
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The machine's **cumulative power-on time**, an accumulating total that keeps counting even across power cycles. Returns `int` (seconds) + `unit:"s"`.

- **Fanuc**: parameter 6750 (**minute resolution**), so the value is always a multiple of
  60. Differential calculations (e.g. utilization) carry an inherent ±60 s error
- **Siemens**: `setupTime`. On an 840D sl bench it came in whole minutes, so the value was a multiple of 60 (deemesh multiplies the minutes by 60 to get seconds and does not round to whole minutes). A normal power cycle does not reset it, but **powering the control up with default values sets it to `0`** (a rare service operation)
- **Mitsubishi**: `GetAliveTime` (**second resolution**). It is the `Power ON` item of the control's integrated-time screen (accumulated from NC power on to off), and **the control stops accumulating at `59999:59:59` and holds that value** (M800 Instruction Manual). The EZSocket manual documents the value as an 8-digit `HHHHMMSS` (up to `9999:59:59`), so we have not confirmed what the API returns beyond 9999 hours (about 416 days). Once the cap is reached every difference reads `0`
- **Heidenhain**: `GetNcUpTime`. The reference describes it as the accumulated time the control has been on and states that this counter cannot be reset. It has **minute resolution**, so the value is always a multiple of 60. It was the same counter as "Control on" under Machine times in the control's machine settings (`2114880` while that showed 587:28:21 in our test environment)

Elapsed-time addresses are always **seconds-normalized int** (the `…Duration` suffix rule). **Anything below a second is discarded** - `59.9` seconds reads as `59`. That is how the control's own elapsed-time display works, and it never counts a second that has not passed yet. Every machine type and every `…Duration` address behaves the same way.

## /machine/channelCount
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The number of channels (paths) on the CNC. Returns `int`, read-only. Cached at connection time, so it returns immediately with no additional communication. The valid range of the `channel` filter is `1` to this value.

Sources: the maximum path count from `cnc_getpath` on Fanuc and `/Nck/Configuration/numChannels` on Siemens; on Mitsubishi it is counted at connection time by opening part systems `1` to `8` in turn, and on Heidenhain it is the number of channels in the list `GetChannelInfo` returns at connection. On Siemens and Heidenhain, if that value could not be read at connection, deemesh does not make up a value and answers status `-17`.

The HEIDENHAIN DNC reference states that the channel list from `GetChannelInfo` holds a single element, so on Heidenhain the value is `1`.

## /machine/channel/toolAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The tool area number the channel uses. The value to put in the `toolArea` filter of the tool tree addresses.

**Read this value and pass it straight into `toolArea`.** Do not pick a number yourself, because the numbering differs by machine.

On Siemens it is the NCK setting (`toNo`), so several channels may get the **same number**, which means those channels share their tools.

Fanuc and Mitsubishi have no separate tool-area layer; their tool data belongs to the **path (part system)**. The channel number therefore comes back unchanged. The `/machine/toolArea/…` addresses of these two machine types take no `channel` filter, so `toolArea` is what selects the path. The exception is **the Mitsubishi magazines, which belong to the whole machine**: the magazine addresses (`magazineCount`, `magazineList`, `magazine/…`) answer the same magazines for any `toolArea` value from `1` to the channel count.

**Heidenhain** has a single tool management (one set of tool numbers), so this is always `1`. The tool addresses accept only `1` for `toolArea`; any other value is status `-18`.

## /machine/channel/executionStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
codes: [{"value": 0, "name": "Reset"}, {"value": 1, "name": "Stop"}, {"value": 2, "name": "Hold"}, {"value": 3, "name": "Run"}, {"value": 4, "name": "MSTR (retraction/recovery/JOG MDI)", "read": ["nc_focas2_fanuc"]}, {"value": 5, "name": "Interrupted", "read": ["nc_opcua_siemens"]}, {"value": 99, "name": "Unknown", "read": ["nc_focas2_fanuc"]}]
```

The program **execution status** code (with `desc`). The "is it running now" counterpart to `operateMode` (which mode it is in):

- `0` = Reset · `1` = Stop · `2` = Hold · `3` = Run (running)
- `4` = MSTR (Fanuc: retraction/recovery) · `5` = Interrupted (Siemens: see below) · `99` = Unknown (Fanuc only; Siemens and Heidenhain surface unlisted values as a status `-17` error)

`1` and `2` are different **kinds** of pause (who halted it, and where):

- `Stop` = halted **by the program at a planned point**. A block finished in single-block mode (confirmed on all four machine types), or it hit M0/M1, always standing at a block boundary.
  - ⚠️ **On Fanuc, M0/M1 may not read as `Stop`.** The control sends the M code and then waits for the PLC to acknowledge it, and it treats that wait as a cycle still in progress, so the value stays **`3` (Run)**. On our 31i bench (with a PLC that does not act on M0) the program stood at the `M00` block with the control's cycle-start lamp blinking, this address read `3` the whole time, and a second cycle start resumed it. A machine whose PLC does act on M0 can read differently: on one 31i-B machine tool this address read `2` (Hold) while the program stood at `M00`. The machine's ladder decides which value appears, so if you need to recognize an M0/M1 stop, check it once on that machine. Siemens reads `1` (Stop) in the same situation.
- `Hold` = halted **by the operator at an arbitrary moment**. The stop key (feed hold) on the panel was pressed; it can stop mid-block.

⚠️ **The button name and the state name disagree** (an industry convention): pressing the panel's **Stop button puts the machine in `Hold`**. The `Stop` state is produced by the program (M0/M1, single block), not by a button. Either way, Cycle Start resumes.

**What you get while an alarm is up differs by machine type.** The same situation - automatic operation halted by a call to a subprogram that does not exist - measured in the test environments of three machine types:

| | `executionStatus` | `alarmStatus` |
|---|---|---|
| Fanuc | `1` (Stop) | `2` |
| Mitsubishi | `3` (Run), until reset | `2` |
| Heidenhain | `0` (Reset), briefly `2` first | `2` |

Each control expresses its own automatic-operation state differently. deemesh does not override this: whether an alarm halts machining depends on the alarm (informational ones do not), and we have no per-alarm knowledge of that, so demoting the value would be wrong in the cases that are fine.

**What an emergency stop gives also differs by machine type.** On Fanuc, stopping automatic operation with the emergency stop read `0` (Reset) in our test environment (NC Guide) and on a 31i bench, and Mitsubishi read `0` (Reset) in its test environment too; on Siemens it is `5` (Interrupted). On Heidenhain, an emergency stop raised during a run by writing `emergencyStatus` read `0` (Reset), briefly passing through `2`, in our test environment. **To detect the emergency stop itself use `/machine/channel/emergencyStatus`, not this address** - that one absorbs the difference between machine types.

**So do not read `3` (Run) as "it is cutting right now".** The value means automatic operation has not ended, not that an axis is moving. Two situations that read `3` while the machine stands still are confirmed: **an alarm is up** (`alarmStatus` is `2`) and **the control is waiting for an M code to be acknowledged** (no alarm; Mitsubishi raises stop code `T10` then, so `alarmStatus` reads `1`). If you need to know whether it is really halted, watch whether `/machine/channel/programCurrentBlock` stops changing, or read `alarmStatus` alongside.

**Mitsubishi reports only `0`-`3`.** This control gives a set of automatic-operation flags (in operation / executing / paused) rather than a status code, and deemesh combines them into the vocabulary above. The `Stop` / `Hold` split maps exactly onto the vendor's own definitions - what it calls "pause" means *halted while executing a command*, which is the `Hold` state above, and the remaining case (in automatic operation but neither executing nor paused) is `Stop`, standing at a block boundary.

`5` (Siemens only) means **the machine stopped because something is wrong**. Two causes are confirmed on our 840D sl bench: an **emergency stop** and an **alarm stop**. Normal stops (M0, single block, an operator stop) all resolve to `1`/`2`, so when you see `5`, tell the two apart with `/machine/channel/alarmStatus` and `/machine/channel/emergencyStatus`. The control reports stop reasons in finer detail than this, and every stop outside the reasons deemesh has confirmed (M0, single block, an operator stop) lands here. So `5` is a stop of unspecified kind, and its `desc` is `Interrupted`. The fact that it is paused is certain, so a caller that does not care about the kind may treat `1`/`2`/`5` together as "paused".

**On Heidenhain** it carries the part-program status (`GetProgramStatus`). Values seen in our test environment (the TNC7 programming station): `3` while running; `2` when the operator pressed NC stop during a move (the axis stops mid-block); `1` at `M0` and after one block in single block (on a block boundary); `0` at the end of the program (also when it ends with `M30`). **A program stopped by an error reads `2`, and what follows depends on the class the control gives the error** (the error stays in `alarmList`). An error of a class that aborts the program passes through `2` briefly (under 0.3 seconds in our test environment) and then reads `0`: so did a collision-monitoring error, an error raised by the program (`FN 14`) and a call to a subprogram that does not exist. An error of a class that only holds the program stays at `2`: in our test environment, running a feed block without the spindle turning read `2` until the error was cleared, and after that the same `2` as when the operator presses NC stop. The TNC keeps a program per operating mode, so outside program run (manual, MDI) the status refers to that mode's program and usually reads `0`. It read `0` even while a block ran in MDI (in our test environment the program status from HEIDENHAIN DNC stayed 'no program selected' meanwhile, and we have not found another way to read MDI execution). The panel's words and this address's names are the other way round: the TNC7 User's Manual calls stopping on a block boundary by `M0` or single block "interrupting" and stopping with the NC stop key "stopping", while this address follows the situation, so the former is `1` (Stop) and the latter `2` (Hold).

## /machine/channel/operateMode
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
codes: [{"value": 0, "name": "Jog"}, {"value": 1, "name": "MDI"}, {"value": 2, "name": "Memory (Auto)"}, {"value": 5, "name": "No mode", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]}, {"value": 6, "name": "Edit", "read": ["nc_focas2_fanuc"]}, {"value": 7, "name": "Handle", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]}, {"value": 8, "name": "Teach in Jog", "read": ["nc_focas2_fanuc"]}, {"value": 9, "name": "Teach in Handle", "read": ["nc_focas2_fanuc"]}, {"value": 10, "name": "INC feed", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]}, {"value": 11, "name": "Reference"}, {"value": 12, "name": "Remote", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]}, {"value": 13, "name": "Jog-REPOS", "read": ["nc_opcua_siemens"]}, {"value": 14, "name": "MDI-Reference", "read": ["nc_opcua_siemens"]}, {"value": 15, "name": "MDI-Teach in", "read": ["nc_opcua_siemens"]}, {"value": 16, "name": "MDI-Teach in-Reference", "read": ["nc_opcua_siemens"]}, {"value": 17, "name": "Auto-Teach in-Reference", "read": ["nc_opcua_siemens"]}, {"value": 18, "name": "MDI-REPOS", "read": ["nc_opcua_siemens"]}, {"value": 19, "name": "MDI-Teach in-REPOS", "read": ["nc_opcua_siemens"]}, {"value": 20, "name": "Auto-Teach in", "read": ["nc_opcua_siemens"]}, {"value": 99, "name": "Unknown", "read": ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]}]
```

The current operating mode code (with `desc`). A unified code regardless of machine:

- `0` = Jog · `1` = MDI · `2` = Memory (automatic) · `5` = no mode · `6` = Edit · `7` = Handle
- `8` = Teach in Jog · `9` = Teach in Handle · `10` = INC feed · `11` = Reference (return to origin) · `12` = Remote (DNC)
- `13` = Jog-REPOS · `14` = MDI-Reference · `15` = MDI-Teach in · `16` = MDI-Teach in-Reference · `17` = Auto-Teach in-Reference · `18` = MDI-REPOS · `19` = MDI-Teach in-REPOS · `20` = Auto-Teach in
- `99` = Unknown

`13`–`20` are **Siemens-only**: a basic mode (Jog/MDI/Auto) with an auxiliary function (REPOS, reference, teach-in) layered on top, which appears when that combination is selected on the operator panel (Basic Functions manual, K1: JOG can carry REF or REPOS, MDI can carry REF, REPOS or teach-in; `20` is TEACH IN pressed in AUTO, which the operator panel of an 840D sl bench showed as the mode `TEACH IN`). Fanuc reports the same situations using the basic mode code alone, so these values never occur there. `8` (Teach in Jog) is a Fanuc mode. `17` (Auto-Teach in-Reference) is a combination we did not find in the K1 function manual, and it has never appeared on the 840D sl bench, so we have not confirmed that it occurs.

On Mitsubishi, **RAPID** (manual rapid traverse) comes out as `0` (Jog); it is manual continuous feed, the same family as Jog, and we do not mint a new number every time a machine type is added. The panel's STEP is `10` (INC feed) and TAPE is `12` (Remote).

`5` (no mode) comes from **Fanuc and Mitsubishi**: the state where no basic mode is selected. The operator panel shows `****` in the mode field on Fanuc and "no mode" on Mitsubishi. On Mitsubishi it is common on a multi-part-system machine, where **each part system has its own mode**: while you work in one, the other reads this, and that part system's `alarmList` carries `M01 0101` (operation mode not selected) alongside it. It differs from `99` (Unknown): here the machine positively reported "no mode", whereas `99` means we could not interpret the value it gave. Siemens has no equivalent state. On Siemens, a mode combination deemesh cannot interpret answers status `-17` rather than `99`.

**On Heidenhain** it carries the control's execution mode (`GetExecutionMode`). In our test environment (the TNC7 programming station) manual operation read `0`, MDI `1` and program run `2`; turning on Single block in program run still reads `2` (whether single block is on is what `/machine/channel/singleBlockOn` tells). The TNC7 User's Manual ('Overview of operating modes') also calls the editor (Editor), the files (Files) and the tables (Tables) operating modes, but the execution mode HEIDENHAIN DNC reports covers the side that moves the machine (manual, MDI, program run and so on), so while one of those screens is open the last value is still reported; `6` (Edit) therefore does not appear. The Setup application inside the manual operating mode also reads `0`. Handwheel (`7`) appeared when the handwheel was switched on with its activation key in manual operation, and switching it off again read `0` (with the virtual handwheel of our test environment). Switching the handwheel on in program run keeps `2`, and in MDI keeps `1`. Reference (`11`) appeared in our test environment when the mode was switched through DNC; entering it from the control's own panel has not been confirmed (according to the manual, a machine with incremental encoders stays in reference mode after power-on until all axes are referenced; our test environment always had its reference established). These values were seen on a TNC7, and the manual ('Operating elements of the keyboard unit') notes that the TNC7 arranges its operating modes differently from the TNC 640 (some keys switch a function on instead of changing the mode). Whether a TNC 640 gives the same values has not been confirmed. A value the reference classes as an execution other than the known ones reads `99`.

**Writing is supported on Heidenhain only** (`SetExecutionMode`; the other controls answer status `-20` (not supported)). The accepted values are `0` (manual operation), `1` (MDI) and `2` (program run); any other code is status `-16` (invalid write value). Handwheel (`7`) and reference (`11`) are not accepted because in our test environment switching to handwheel mode set the overrides to the virtual handwheel's values (feed and rapid `0`%, spindle `50`%), which stayed after leaving the mode (the TNC7 User's Manual says that switching the handwheel on hands the feed rate over to the handwheel's own potentiometer), and the reference mode could not go straight back to program run. **If the control is already in that mode, nothing is sent and the answer is status `0`.** Program run with single block on also reads `2`, so writing `2` again leaves single block as it is (single block is written through `/machine/channel/singleBlockOn`). When the control does not switch now the answer is status `-22` (machine state); in our test environment that was the case when switching to manual operation while a program was running. The operator panel screen switched to the mode at once. **Write caution**: this changes the mode the operator at the panel is working in.

## /machine/channel/emergencyStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
codes: [{"value": 0, "name": "Not emergency"}, {"value": 1, "name": "Emergency"}, {"value": 2, "name": "Reset", "read": ["nc_focas2_fanuc"]}]
```

The emergency-stop status (with `desc`): `0` = normal, `1` = emergency stop. On Fanuc a transient `2` (Reset) flashes by: on a 31i bench it lasted about 1.4 s and 2.3 s in two measurements when the emergency stop was released, and under a second when RESET was pressed; treating anything non-`0` as "not normal" is the safe reading.

**Use this address to detect an emergency stop.** Which channel an emergency stop surfaces on differs by machine type, so looking for it in the alarm list yourself gives different answers on different controls. This address absorbs that difference and answers with the same meaning whatever the control.

On Mitsubishi it is `1` whenever the alarm list carries an `EMG` class; it is caught **regardless of the cause** of the emergency stop.

**Siemens 840D sl is decided by a single signal the NC sets on the PLC interface, emergency stop active (`DB10.DBX106.1`).** In the emergency-stop sequence of the function manual, the NC sets this signal and raises the emergency-stop alarm `3000` in consecutive steps and clears both together, so this one read carries the same meaning as the two-step method below. Both come on when the machine builder's PLC program passes the emergency-stop button to the NC. Reading this signal needs read access to the PLC `DB10` for the account used to connect (`SinuReadAll` includes it); without it the signal cannot be read and the address answers with status `-17`. **After the emergency stop is released it stays `1` until a RESET acknowledges it** (840D sl bench: the signal and alarm `3000` stayed until RESET); Fanuc goes to `0` on release, passing briefly through `2` (31i bench).

**Other Siemens controls (828D and others) decide in two steps.** While the mode-group-ready signal (`readyActive`, PLC interface DB11 DBX6.3) is on the value is `0` with no extra communication; only when it is off does deemesh fetch the alarm snapshot and answer **`1` if the emergency-stop alarm `3000` is present, `0` otherwise**. The ready signal alone would not do: every alarm whose reaction includes "mode group not ready" (drive, encoder and referencing faults among others) clears it, which would make the value broader than the Fanuc emergency-stop signal or the Mitsubishi `EMG` class (Basic Functions manual, A2). In those cases the value is `0` and the cause is told by `alarmStatus` (`2`) and `/machine/channel/alarmList`. After the emergency stop is released, `3000` stays until it is acknowledged and reset, so `1` lasts that much longer (the 840D sl signal behaves the same); if the alarm snapshot cannot be fetched while the mode group is not ready (on 840D sl, if the signal cannot be read), the address answers with status `-17` rather than guessing. The alarm snapshot this reads is the one `alarmList` reads, so right after the control starts the wait for late alarms described under `alarmList` applies here too.

**On Heidenhain it reads the PLC API's CNC emergency-stop symbol.** In the PLC API definition Heidenhain places on the control, this symbol means the CNC is in emergency stop. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`. In our test environment (the TNC7 programming station) we could not trigger an emergency stop from a panel button; `1` was confirmed with an emergency stop raised by the write below.

**On Heidenhain it can also be written (raising and removing an emergency stop).** Writing `1` makes deemesh raise an error of the emergency-stop class on the control through HEIDENHAIN DNC, which puts the control into emergency stop. The message line on the operator panel shows "Emergency stop requested through deemesh", and a running program is aborted. ⚠️ **On a machine tool this stops the machine at once, even in the middle of a cut.** If the control is already in emergency stop, whatever the cause, nothing is sent and the status is `0`. In our test environment this address became `1` within a second of the write, and a running `executionStatus` went from `3` to `0`, briefly passing through `2`.

Writing `0` **removes only the emergency stop that this connection raised with `1`.** This address returns to `0` within a second and the aborted program does not start again (confirmed in our test environment). If another cause, such as the emergency-stop button on the panel, is active as well, the emergency stop stays. With nothing raised by this connection and the control in emergency stop, the status is `-22`: release it at the machine. **An emergency stop that deemesh raised stays on the control when the connection drops or deemesh restarts.** It can then no longer be removed with `0` (status `-22`); clear its message with CE in the control's message window (confirmed in our test environment). Clearing it with CE at the panel first is fine too. Because the write looks at the current state, it also needs `access_password`. The HEIDENHAIN DNC reference lists this function from DNC 1.7.1 on the TNC7 and from DNC 1.6.1 on the TNC 640.

## /machine/channel/motionStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
codes: [{"value": 0, "name": "None (Idle)", "read": ["nc_focas2_fanuc"]}, {"value": 1, "name": "Motion", "read": ["nc_focas2_fanuc"]}, {"value": 2, "name": "Dwell"}, {"value": 3, "name": "Wait (Multi-path Synchronization)", "read": ["nc_focas2_fanuc"]}, {"value": 4, "name": "Not dwelling", "read": ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]}]
```

The axis motion status code (with `desc`):

- `0` = None/Idle · `1` = Motion (moving) · `2` = Dwell (dwelling)
- `3` = multi-path synchronization wait (Fanuc) · `4` = Not dwelling (not in a dwell; nothing more is known)

**A dwell keeps counting down while the program is stopped** (confirmed in the Siemens test environment). While an operator stop holds the program and `executionStatus` reads `2` (Hold), the remaining `G4` time still runs to `0`, after which this address changes from `2` (Dwell) to `4` (Not dwelling). A stop does not freeze the dwell, so on resume that block is already finished and execution continues with the next one.

**`4` is a superset of `0`, `1` and `3`** - it means the state is one of those three but cannot be narrowed further. On Siemens and Mitsubishi, deemesh judges from the remaining dwell time, so those two report only `2` or `4` (`0`, `1` and `3` are Fanuc only).

If you need to know whether an axis is actually moving and the control reports `4`, this address cannot tell you. Use `/machine/channel/executionStatus` to see whether automatic operation is running (neither address answers for motion during manual operation).

## /machine/channel/alarmStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
codes: [{"value": 0, "name": "No alarm"}, {"value": 1, "name": "Warning"}, {"value": 2, "name": "Alarm"}]
```

The **alarm severity**. Regardless of machine it returns **only the three values `0` / `1` / `2`**, and what is actually wrong on that machine comes alongside in `desc`.

| Value | Meaning |
|---|---|
| `0` | normal - no alarm and no message |
| `1` | warning - what the control shows as a warning or notice (an informational message, a normal program stop, a warning such as low battery, and so on) |
| `2` | alarm - **what the control shows as an alarm** (a red alarm on the operator panel) |

**Picture it as a signal light**: `0` green, `1` yellow, `2` red. The three values map straight onto a stack light or a status indicator on screen. That works because the scale matches **how the control itself colours its own messages**: the Mitsubishi manual paints NC alarms and PLC alarms on a red background, and warnings, stop codes and operator messages on a yellow one. So yellow is what the control shows as a warning or notice. It covers machine warnings such as low battery (Fanuc) or servo warnings (Mitsubishi `S52`, `S53`), and it also covers signs that **the machine is waiting on something**, such as the Mitsubishi stop codes.

💡 **The most robust use is `0` versus non-`0`.** `0` means exactly the same thing on every machine type ("no alarm and no message") and lines up with `alarmCount` being `0`. The boundary between `1` and `2`, by contrast, rests on **how that control classifies its own alarms**, so it can differ slightly between machine types. If you must branch on severity, read the per-item `severity` and text in `alarmList` alongside it.

⚠️ **`1` does not always mean "something has gone wrong"; it can show up during perfectly normal machining, and how often depends on the machine type.** An untroubled automatic run (a dwell-only program) measured in the test environments of two machine types:

| | `alarmStatus` | what the list held |
|---|---|---|
| Fanuc | `0` | nothing |
| Mitsubishi | `1` | a stop code |

**Only Mitsubishi has a "stop code" channel.** It is where the control reports *the state of automatic operation*, and the vendor's own manual colours it yellow like a warning, not red like an alarm. What lands there is `T03 0301` (a single block stop), `T10` (waiting for an M-code completion), `T02 0202` (an axis is at a soft limit) and the like: not faults, but "waiting on this right now". The one standing there in the check above was `T10`, and it went away when the program ended. On Mitsubishi, an automatic run with the axes moving read `0` (confirmed on the simulator).

**The letter is the family and the four digits are the reason** (`T01` cycle start not possible, `T02` automatic operation stopped, `T03` block stop, `T04` collation stop, `T10` waiting for a completion signal). So within `T02`, `0202` is a soft limit and `0204` is feed hold. **An operator simply pressing feed hold or single block raises a stop code, which makes this address read `1`** (confirmed in our test environment: feed hold `T02 0204`, single block stop `T03 0301`). When you need the reason, read the full four digits from the `code` of the `alarmList` entry. Fanuc does not put such states in the alarm list, so it stays at `0`. On Siemens the NC does not list such states either, but a PLC message from the machine builder can make this address `1` (on an 840D sl bench, `M0` raised PLC message `700355` as a warning).

**So do not use `1` as a call-the-operator signal**: on a Mitsubishi machine it can read `1` even during normal automatic operation. Use `2` for that, or decide from the `severity` and `category` of the entries in `alarmList`.

**The criterion is the grade the control gives, not whether machining stopped.** The two usually coincide but not always - a machine halted by `M0` or by single block has no alarm, so it does not become `2` (Mitsubishi reports the value `1` through a stop code, Fanuc keeps the value `0` since it does not list the state. On Siemens the NC does not list it either, but a PLC message from the machine builder can make it the value `1`). The other way round, a Fanuc background edit alarm (`BG`, for example `BG1090` after uploading a program that is not in a valid format) is `2` because the operator panel shows it as a red alarm, yet automatic operation can still start and run (confirmed on the simulator; a reset or `M30` clears it). Use `/machine/channel/executionStatus` to find out whether machining is stopped right now.

- **The number is one of these three on every machine type.** Vendor codes are not passed through, so you can branch on `value` without knowing which machine it is attached to.
- **The cause comes in `desc`**: Fanuc gives the cause family (`{"value": 1, "desc": "Memory backup battery voltage low (CNC or Amplifier)"}`), Siemens gives the text of the heaviest alarm (`{"value": 2, "desc": "Emergency stop"}`), Mitsubishi gives the alarm kind (`{"value": 2, "desc": "NC alarm"}`). `desc` is a human-readable string, so **do not use it as a branch condition**; branch on `value`.
- **Mitsubishi**: **exactly the colour the control paints the message** - red (NC alarm, PLC alarm message) is `2`, yellow (NC warning, stop code, operator message) is `1`. The vendor API delivers NC alarms and NC warnings as one kind, so the colour is restored line by line. With EZSocket `FCSB1224W100-A9` or later the control also says whether it treats each line as a warning (`GetAlarm3`), and that is followed; with earlier versions **the category code** decides (`GetAlarm2`): operation errors `M00`/`M01`, operation warnings `M50`, servo warnings `S52`/`S53` and smart-safety warnings `V5x` (anything starting with `V5`) are yellow (`1`); every other NC alarm (`S01`-`S05`, `S51`, `Y`, `Z`, `Z7x`, `Z8x`, `EMG`, `L`, `U`, `N`, `P`, `V01`-`V07`, ...) and PLC alarms are red (`2`). An unknown category is `2`. The two methods gave the same grade for `M01`, `P114` and `EMG` on the simulator. So waiting states such as no operation mode (`M01 0101`) or override at zero (`M01 0102`) read `1`, while an emergency stop (`EMG`) reads `2`. Note that a soft stroke end is operation error `M01 0007` on Mitsubishi (yellow, `1`) but an OT alarm on Fanuc (`2`) - one situation the two controls grade differently, and this address follows each control's own grading
- Vendor codes whose severity is ambiguous are classified **conservatively as `2`**, because calling a stop a mere warning is more dangerous than the reverse. On Fanuc, a code the control emits that the SDK does not know is classified as `2` as well. On Siemens the per-event severity the server reports is used directly: only an error (`1000`) is `2`, while anything the control itself calls a **warning (`500`) is `1`**. This is the **same criterion** as `alarmList`'s `severity`.
- For the alarm **list, numbers and messages** use `alarmList`; for the **count** alone use `alarmCount`. This address is the summary meant for high-frequency polling. **Whether it is cheaper than the list depends on the machine type.** Fanuc decides it from the status information without reading the list (it costs one extra round trip to check for operator messages, and only that one more when you ask for it alongside the other status addresses). Mitsubishi, when this address is asked for on its own, only checks whether each kind is present, so the usual no-alarm case is one round trip; asked together with `alarmList`, `alarmCount` or `emergencyStatus` it reads the full list. On Siemens all three addresses come from the same snapshot, so it costs the same as `alarmList`.
- ⚠️ **`alarmStatus` can be non-zero while `alarmCount` is `0`.** On Fanuc the summary also reads the operator panel's **status line**, while the list carries only alarms and messages, so there are states that reach the summary alone (low battery, power-supply and insulation warnings, and the like). The opposite - `alarmStatus` of `0` while the list has entries - does not happen.
- **Siemens does not use the `channel` value** (same policy as `alarmList` and `alarmCount`). All three come from the same NCK-global snapshot, so one request answers them together and they cannot disagree. They cannot be split per channel because the alarm events carry no channel information (the alarm origin areas the OPC-UA manual defines are `HMI`, `NCK` and `PLC`). On every machine type the `channel` value is range-validated.
- **Siemens also sees the machine builder's PLC alarms** (hydraulics, lubrication, door interlocks and the like). It used to read a node covering NCK alarms only, so such an alarm could be active while this address reported `0` (normal).
- **On Siemens, alarms can arrive late right after the control starts** (the same snapshot as `alarmList`). When the first read after the alarm subscription is created (a new connection or a reconnection) finds no alarm, deemesh waits up to 1 second for alarms arriving late before it answers (only within what is left of the request's `timeout`), so that read takes that much longer, and an alarm arriving later than that shows in the next read. On the 840D sl, use `emergencyStatus` to detect an emergency stop right after the control starts (there it reads a PLC signal). On other Siemens controls such as the 828D, `emergencyStatus` reads the same alarm snapshot.

**On Heidenhain** it follows the grade the control gives each `GetErrorList` entry: `2` if any entry has the Error grade, `1` if there are only Warning, Info or Note entries, with the grade name in `desc`. This follows the split in the TNC7 User's Manual ('Message menu on the information bar'): an error must be cleared before work can continue (some need a restart), while a warning, info or note lets work continue without clearing it. An entry with a number or text but no grade, or a grade deemesh does not know, is treated as an error to be safe, so `2`. Two things to know. First, in our test environment (the TNC7 programming station) the control recorded an Info entry (`130-07e2`) every time an unsecured DNC connection was made, so once deemesh had connected this address did not return to `0` until that entry was cleared in the panel's message menu (an info entry can be cleared at any time), and `alarmCount` grew by one with each new connection. A secure connection (`RPC secure`, `connection_name`) did not leave this entry. Second, in the same test environment `M0` raised the PLC message `PLC00050` with the Error grade, so the value was `2` (it cleared on resuming). Which message a machine raises, and with which grade, is decided by that machine's PLC program. An empty entry with no grade, number or text is not counted (see `alarmList`).

## /machine/channel/alarmCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The **number** of active alarms/messages (= the count of `alarmList` items, regardless of severity). Returns `int`. Use it when you only need the count, like a dashboard badge.

- **Cost note (Fanuc)**: the list is fetched and counted, so internally it costs the **same** as `alarmList`. If you only need a low-cost presence check, use `alarmStatus`
- **Cost note (Siemens)**: the count is taken from the **same event snapshot** as `alarmList`, so it costs the same, and `alarmStatus` comes from that snapshot too, so it is no cheaper. Being an NCK-global count, the `channel` value is ignored (same policy as `alarmList`)
- **Cost note (Mitsubishi)**: same as Fanuc; the list is fetched and counted, so it costs the same as `alarmList`
- **On Siemens, alarms can arrive late right after the control starts** (the same snapshot as `alarmList`). When the first read after the alarm subscription is created (a new connection or a reconnection) finds no alarm, deemesh waits up to 1 second for alarms arriving late before it answers (only within what is left of the request's `timeout`), so that read takes that much longer, and an alarm arriving later than that is counted in the next read. On the 840D sl, use `emergencyStatus` to detect an emergency stop right after the control starts (there it reads a PLC signal). On other Siemens controls such as the 828D, `emergencyStatus` reads the same alarm snapshot

- **Cost note (Heidenhain)**: counts the same list as `alarmList` (`GetErrorList`), so it costs the same. Info-grade entries are counted too. Unlike the Group view of the panel's message menu (one row per number), it counts every entry. In our test environment each unsecured DNC connection left an Info entry, so the value grew with every new connection. A secure connection (`RPC secure`, `connection_name`) left none, and an empty entry with no grade, number or text is not counted (see `alarmStatus` and `alarmList`)

## /machine/channel/alarmList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
field_codes: {"severity": [{"value": "alarm", "name": "Alarm"}, {"value": "warning", "name": "Warning"}]}
```

The channel's **active alarms + operator/macro messages** list. Return type `objectArray`, or an empty array `[]` if none. For Siemens, alarms are NCK-global so the `channel` value is ignored.

**On Siemens this list carries the control's alarms and the messages of the machine's PLC (the `700000` range).** The text of a part program's `MSG()` is not an alarm and does not appear (unlike a Fanuc macro message `#3006`; confirmed on an 840D sl bench). PLC messages come as `warning` (on the 840D sl bench, for feed override `0` and an `M0` stop), and which messages appear is up to that machine's PLC program.

**On Siemens, alarms can arrive late right after the control starts.** On an 840D sl bench restarted with the emergency stop on, a first read less than 10 seconds after the OPC-UA port opened received two alarms only after the mark that ends the list (from 17 seconds on they were in the list from the start). So when the first read after the alarm subscription is created (a new connection or a reconnection) finds the list empty, deemesh waits up to 1 second for alarms arriving late before it answers (only within what is left of the request's `timeout`). That read takes that much longer, and an alarm arriving later than that is in the next read. On the 840D sl, use `emergencyStatus` to detect an emergency stop right after the control starts (there it reads a PLC signal, not an alarm). On other Siemens controls such as the 828D, `emergencyStatus` reads the same alarm snapshot as this list.

Element: `{"code": "OH0700", "message": "SPINDLE OVERHEAT", "category": "Overheat", "severity": "alarm", "raisedAt": "2026-07-29T11:11:03Z"}`

The key set is **always the same regardless of machine**: when a value is unavailable the key is not dropped, it is `null` (same convention as `entry`). `severity` has only the two values `"alarm"` / `"warning"`.

- **code**: **the identifier exactly as the control's panel shows it, as a string** - Fanuc: alarm type letters plus the 4-digit number (`"OT0501"`, `"PS0010"`; operator/macro messages carry the message number, `"2000"`), Siemens: the alarm number (`"4230"`, `"700015"`; the number range tells the origin: `0`-`9999` general, `10000`-`19999` channel, `20000`-`29999` axis/spindle, `60000`-`69999` cycles, `100000`-`199999` HMI, `200000`-`299999` drive (SINAMICS), `300000`-`399999` drive and I/O, `400000`-`899999` PLC, of which `500000`-`899999` are defined by the machine builder; Diagnostics Manual), Mitsubishi: class plus detail (`"M01 0101"`, `"S01 0051"`, `"EMG EXIN"`). Letters and leading zeros are significant, so **keep it as text and look it up as-is** in the manual. When no identifier can be obtained it is `""`, never `null` (in practice Fanuc, Siemens and Mitsubishi always fill it; on Heidenhain an entry without a number gives `""`). Replaces the integer `number` as of 1.2.0
- **message**: display text
- **category**: on Fanuc, for alarms, the cause family (`Servo`, `Overheat`, `Spindle`, `PLC`, etc.: undefined types come as a numeric string); for messages, the source (`Operator message` = PMC/external input, `Macro message` = part-program #3006). Siemens: the source name the server puts on the event (`SourceName`), as is (e.g. `NCU`: falls back to `Alarm` when empty). Mitsubishi: the alarm class as shown on the operator panel (`EMG`, `S01`, `M01`, etc.)
- **severity**: **the grade the control gives the entry**. `"alarm"` = what the control shows as an alarm (red on the operator panel) / `"warning"` = what it shows as a warning or notice. It does not say whether machining stopped. Fanuc: every alarm in the alarm list is alarm, background edit alarms (`BG`) included (the operator panel shows them as red alarms; automatic operation can still start while a `BG` alarm is up); operator/macro messages are all warnings. Siemens: translates the server's severity (1–1000) at a 500 boundary. Mitsubishi: what the control paints red (NC alarms, PLC alarms) is alarm, what it paints yellow (NC warnings, stop codes, operator messages) is warning - the same criterion as `alarmStatus`. Whether an NC alarm line is a warning follows the control's own judgement (`GetAlarm3`) with EZSocket `FCSB1224W100-A9` or later; with earlier versions the category code decides (`M00`/`M01`/`M50`/`S52`/`S53`/`V5x` are warnings). Whether "machining is currently stopped" should be judged by `executionStatus` rather than this field, but note that **while an alarm is up that value too can read `3` (Run) on some machine types** (see that address). A warning in a stopped state means it is waiting for operator intervention such as macro `#3006`
- Fanuc carries at most **100 active alarms** (the read size deemesh uses) and **17 operator/macro messages** (the count the FOCAS2 specification sets for reading them all: 16 operator messages plus the macro message) per read. Mitsubishi carries **10 per alarm kind**, so at most **40** in total (vendor API limit). The limits apply per kind (Fanuc reads alarms and messages separately, Mitsubishi takes 10 per kind), so an overflow is cut only within the kind that overflowed.
- **raisedAt**: the time raised, in **UTC with a trailing `Z`** (`"2026-07-29T11:11:03Z"`). Siemens gives the actual time, and `null` for an entry without a time (shown as `---` on the operator panel; on an 840D sl bench this happened after restarting with the emergency stop on); Heidenhain also gives the actual time (see below); Fanuc and Mitsubishi are always `null` (active alarms have no time information)
  - The number **differs from what the machine's own screen (HMI) shows**: the HMI renders the machine's local time while this is UTC. They are the same instant written differently; converting is for whoever knows the machine's time zone (the host application). OPC-UA events do have a field for the local-time offset, but it was empty on the control we measured
  - **Do not subtract it from `/machine/currentDateTime`.** Beyond the time zone, the two come from **different clocks**; on one machine they were measured about 18 minutes apart (clock setup varies by site)

**On Heidenhain** it lists the entries of `GetErrorList`. `code` is the number exactly as the operator panel shows it (`130-07e2`, `PLC00050`), `category` is the control's error group (`Operating`, `Programming`, `PLC`, `General`, `Remote`, `Python`), `""` for an entry without a group and the group's number as a string for a group deemesh does not know. `severity` is the grade the control gives: the Error family is `alarm`, while Warning, Info and Note are `warning`, and an entry without a grade, or with a grade deemesh does not know, is `alarm` to be safe. `message` comes in the control's display language. `raisedAt` is UTC (`Z`): we did not find the time zone stated in the reference, but in our test environment it matched the PC's UTC to the second and differed from the operator panel's message window (the control's local time) by the time-zone offset. Info entries are listed too, so in our test environment the Info entries (`130-07e2`) left by each unsecured DNC connection had piled up. With a secure connection (`RPC secure`, `connection_name`) no such entry was left. An empty entry with no grade, number or text is not listed (in our test environment one such entry came after all messages were cleared on the operator panel, while the operator panel showed nothing). The channel is not used to filter (there is one channel).

## /machine/channel/singleBlockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The single-block switch state (`true` = on). Fanuc reads the F4 signal bit, Siemens `singleBlockActive`, and Mitsubishi a PLC output (Y) bit in the operator-panel signal block.

Heidenhain decides it from the execution mode (`GetExecutionMode`) rather than a switch signal: in program run with Single block on it reads `true` even while stopped (confirmed in our test environment), and in manual or MDI mode it reads `false` because the execution mode then reports manual or MDI. According to the TNC7 User's Manual the Single block switch exists in program run only, and MDI always runs one block at a time without a switch; this address reports the switch, so it reads `false` in MDI as well.

**Writing is supported on Heidenhain only** (`SetExecutionMode`; the other controls answer status `-20` (not supported)). Single block is a switch inside the program run operating mode, so it is accepted in that mode only; in another mode such as manual or MDI the answer is status `-22` (machine state), because switching it on would change the operating mode, so write `2` to `/machine/channel/operateMode` first. If it is already in that state, nothing is sent and the answer is status `0`. In our test environment it could be switched on and off during a run as well, and the operator panel showed Single block at once.

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

**Fanuc answers status `-20`.** The optional-stop switch is an input from the machine operator's panel into the PMC, so its state lives in that machine's ladder devices (Connection Manual B-64483EN-1 states that for M00/M01 only the code, strobe and decode signals are sent, and that program stop and optional stop are designed on the PMC side). The `DM01` signal the control does output (`F9.6`) is a decode signal meaning the program has commanded an `M01` block, unrelated to the switch (measured on NC Guide: `F9` stays `0` with the switch on). If you know the device the ladder uses, read it through `/machine/plcAddress/plcType/plcValue`.

**Mitsubishi answers status `-20` as well.** In the arrangement the PLC Interface Manual (IB-1501272) describes, when `M01` arrives **the machine builder's PLC checks its own switch input** and raises the single-block signal (`SBK`) to stop, so the switch state lives only in that machine's ladder devices and cannot be read through a machine-independent address (the same class as `plcAddress`). If you know the device the ladder uses, read it directly through `/machine/plcAddress/plcType/plcValue`.

## /machine/channel/blockSkipOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The block-skip (`/`) switch state (`true` = on). `channel` filter. Supported on Fanuc, Siemens and Mitsubishi. On Heidenhain deemesh has not found a way to read this switch, so the status is `-20` (according to the TNC7 User's Manual, program run has a skip-block switch).

**On controls with several skip levels, this address still looks only at the plain `/`.** Some controls let you number the prefix (`/2`, `/3`, …) so different sections are skipped by different switches (Siemens documents levels `0`–`9`, Mitsubishi `BDT1`–`BDT9`), but what this address reports is always the **unnumbered `/`**. There is no address for the numbered levels.

## /machine/channel/machineLockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The machine-lock (axis-motion lock) state (`true` = on). `channel` filter. Supported on Fanuc, Siemens and Mitsubishi (Heidenhain answers status `-20`); for Siemens this is the program-test (`progTestActive`) state.

**Mitsubishi publishes this signal per axis, while this address is one value per channel.** It therefore reads `true` **only when every axis in that channel is locked**. Calling a partial lock "on" would read as "nothing is moving" and lead a consumer to mistake real machining for a test run. The operator-panel switch drives all axes together, so on an ordinary machine the distinction never surfaces.

## /machine/channel/rapidOverride
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The rapid-traverse override (%). Returns `float` + `unit:"%"`, the value **to the 0.1% digit**. It is usually a whole percent, but a control set to 0.1% steps reports a fraction such as `87.5`. **Fanuc depends on the method the machine builder's ladder selects** (Connection Manual B-64483EN-1): the default `ROV1`/`ROV2` method (`G14`) is stepped, so only `100`/`50`/`25`/`0` appear (`0` is F0, the speed in parameter `1421`); the 1% step method (`HROV`, `G96`) gives integers `0` to `100`; the 0.1% step method (`HROV` and `FHROV` both on, `G96` and `G353`) reports the value down to the 0.1% digit, such as `87.5`. In every method a value above 100% is capped at `100`. On a multi-path control the path's own signals are read (addresses shift by `1000` per path). Siemens is continuous and, like the feed override, it is **the effective value** rather than the switch value: while the PLC enable signal `DB21.DBX6.6` is off, the value `100` is reported regardless of the dial (when it is dropped is up to the machine builder's ladder; the machine we measured kept it off from emergency stop until the panel's ready button, MC READY, was pressed). Machines without a dedicated rapid dial commonly have the ladder apply the feed dial to rapid as well; when it is applied that way, feed override settings above 100% are capped by the control at 100% for rapid (Basic Functions Manual). Measured: with the feed dial at 110, the feed override address read the value `110` and this address read the value `100`. **Mitsubishi depends on the method the machine builder's ladder selects** (PLC Interface Manual IB-1501272): with the method-selection signal `ROVS` (`YC6F`) off, the code signals `ROV1`/`ROV2` (`YC68`/`YC69`) give the four values `100`/`50`/`25`/`0` (the manual's `1%` step is reported as `0`, like Fanuc); with it on, the per-part-system register `R2502` (0-100% in 1% units) gives a continuous value.

On those two, `0` does not mean "stopped" but **the slowest rapid step that machine defines** (the lowest position on the panel). What speed that actually is depends on the machine's settings and cannot be read from this address.

Heidenhain gives the rapid-traverse override from `GetOverrideInfo` (a whole percent). In our test environment a written value read back as it was.

**Writing is supported on Heidenhain only** (`SetOverrideRapid`; the other controls answer status `-20` (not supported)). Write a whole percentage, as in `{"value": 80}`. A value whose fractional part is `0` (`80.0`) is accepted; a value with a fractional part such as `50.5`, or a negative value, is status `-16` (invalid write value). The machine decides the range it accepts, and **in our test environment the control clamped a value outside that range to the nearest end and answered status `0`** (in our test environment the range for rapid traverse was from `0` to `100`, so writing `150` set `100`). Read the value back to see what was set. In our test environment a written value took about 0.1 s to show up in a read, so a read right after the write could still return the previous value. A written value takes effect during automatic operation as well and stays after the run ends. Operating the override on the operator panel sets the panel's value, and writing again sets the written one (whichever changed last; confirmed with the virtual dial in our test environment, not with the dial of a real machine). **Write caution**: the rapid-traverse rate of a running machine changes at once.

## /machine/channel/feedOverride
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The feed override (%). Returns `float` + `unit:"%"`, the value **to the 0.1% digit**. It is usually a whole percent, but a control set to 0.1% steps reports a fraction such as `87.5`. Fanuc reads the PMC `G12` signals (`*FV0` to `*FV7`, inverted binary, 0 to 254%), and all signals off reads `0` as the control treats it (Connection Manual B-64483EN-1). It is the switch value: while the override cancel signal (`OVC`) forces the effective rate to 100% the switch value is still reported, and the second feed override (`G13`) is not applied. On a multi-path control the path's own signals are read (addresses shift by `1000` per path, such as `G1012` for path 2). Siemens reads the `feedRateIpoOvr` node, which is not the switch value but **the effective value applied at the interpolator**. While the PLC enable signal `DB21.DBX6.7` is off, the control internally treats the override as 100% (Basic Functions Manual), so the value `100` is reported regardless of the dial. When that signal is dropped is up to the machine builder's ladder. On the machine we measured it went off the moment the emergency stop was pressed and stayed off through releasing the stop and resetting, so the value `100` was reported until the panel's ready button (labeled MC READY on that machine) was pressed, at which point the dial value returned. The dial position itself kept arriving at the PLC the whole time. **Mitsubishi is read the way the machine builder's ladder selects it** (PLC Interface Manual IB-1501272): with the method-selection signal `FVS` (`YC67`) off, the override code signals (`YC60`-`YC64`, 0-300% in 10% steps) are decoded; with it on, the per-part-system register `R2500` (0-300% in 1% units) is read. When all code signals are off the control keeps the previous value, so there is nothing to read and the address answers with status `-17`.

Heidenhain gives the feed override from `GetOverrideInfo` (a whole percent). In our test environment it matched the value on the operator panel.

**Writing is supported on Heidenhain only** (`SetOverrideFeed`; the other controls answer status `-20` (not supported)). Write a whole percentage, as in `{"value": 80}`. A value whose fractional part is `0` (`80.0`) is accepted; a value with a fractional part such as `50.5`, or a negative value, is status `-16` (invalid write value). The machine decides the range it accepts, and **in our test environment the control clamped a value outside that range to the nearest end and answered status `0`** (in our test environment the range for feed was from `0` to `150`, so writing `200` set `150`). The TNC7 User's Manual ('Cutting data') gives the feed-rate override potentiometer a range of 0% to 150%, and says that while the handwheel is on, the handwheel's own feed-rate potentiometer applies ('Fundamentals' of the electronic handwheel). Read the value back to see what was set. In our test environment a written value took about 0.1 s to show up in a read, so a read right after the write could still return the previous value. A written value takes effect during automatic operation as well and stays after the run ends. Operating the override on the operator panel sets the panel's value, and writing again sets the written one (whichever changed last; confirmed with the virtual dial in our test environment, not with the dial of a real machine). **Write caution**: the feed rate of a running machine changes at once.

## /machine/channel/feedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The commanded feedrate (F command value). Returns `float`. Fanuc uses the modal F, Siemens `cmdFeedRateIpo`, and Mitsubishi the `F command feed speed` (FA).

The unit follows the machine setting (mm/min or inch/min in per-minute feed). Fanuc returns the programmed F as it is, so in per-revolution feed (`G95`, `G99`) the value is per revolution (check the feed mode with `/machine/channel/gModalCategory/gModal?gModalCategory=5`). We have not confirmed the unit of this value in per-revolution feed on Siemens and Mitsubishi. For mm or inch, read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address.

**Heidenhain** reads, as PLC data, the value the PLC API definition Heidenhain places on the control describes as the programmed feed rate per minute. It is the last F commanded, so it stays the same in a rapid (FMAX) block, and it equalled the program's F in our test environment, the TNC7 programming station (mm/min). With the control's unit of measure switched to inch and `F100` (10 inch/min) commanded in an inch program, it still came in mm/min (`254`). In an inch program F is in units of 0.1 inch/min (TNC7 User's Manual, 'Cutting data'). A feed per revolution command (`FU`, the distance in mm per spindle revolution, used mainly for turning, according to the same manual) gives the per-minute value converted with the commanded spindle speed: in our test environment `FU0.3` at `S1000` read `300`, and still `300` with the spindle override lowered to 50% so that the actual speed was 500. A negative value answers status `-17`, because what it would mean is not known. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`.

## /machine/channel/feedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The channel's actual feedrate, **the speed at which the tool tip travels along the programmed path**. Returns `float`.

This is the value the `F` command sets. It holds at the commanded rate as the direction changes, and drops when override, acceleration/deceleration, corner slowdown or feed hold apply. Use it to answer "is the machine actually cutting at the programmed feed?"

Fanuc uses `actf` and Siemens `actFeedRateIpo`. Mitsubishi splits the effective feedrate into **an automatic-operation value and a manual one**, so both are read and combined - the value is there while you move an axis by jog or handwheel too.

The unit follows the machine setting (mm/min or inch/min). **On Fanuc this address carries `unit`** (`mm/min`, `inch/min`). The value is always the actual feed per minute. Parameters `3107#3` and `3191#5` can switch the panel's F display to feed per revolution (`MM/REV`, parameter manual); the deemesh value then stays per minute and `unit` says so (confirmed on a 31i bench and on a 0i-F lathe in our test environment: panel `0.10 MM/REV`, deemesh `50.0` `mm/min`). For feed per revolution, divide by `/machine/channel/spindle/spindleSpeedActual`, keeping in mind that the two values are read at different moments. It is the unit the control reported at connect; on older series without the function that reports it (`cnc_rdspeed`; per the support table in the FOCAS2 manual, Series 16/18/21, 0i-A, 15 and 15i lathes) the value is the control's as given, with no `unit`. **Fanuc fixes the unit and the decimal places at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then values come with the old decimal places and can be off by a factor of 10. Other controls carry no `unit`, because the unit is not fixed per address. Without `unit`, read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual).

**Heidenhain** reads, as PLC data, the value the PLC API definition Heidenhain places on the control describes as the current contouring feed rate. It equalled the F of the control's status line (270 and 3 mm/min at overrides of 90% and 1% in our test environment, the TNC7 programming station). With the control's unit of measure switched to inch it still came in mm/min (`270` while the panel showed F `10.6` inch/min). A feed per revolution command (`FU0.3` at `S1000`) also came per minute and matched the panel's F (`270` at a feed override of 90%). The TNC7 User's Manual ('Positions workspace') also states that the F of that display is converted to a per-minute value whatever unit it was programmed in. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`.

## /machine/channel/axisCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The channel's **user axis count**. Cached at connection time. The valid range of the `axis` filter is `1` to this value.

It counts geometry axes together with **non-spindle auxiliary axes** (indexing rotary tables, tailstocks, and the like), and excludes spindles; those are covered by `spindleCount` and the `spindle` filter.

**It differs per path.** On a multi-path control each channel has its own axis configuration, and a path with no axes reads `0`; the axis addresses on that channel then answer with status `-20` on Fanuc and status `-18` on the other controls.

**On Heidenhain** it counts the axes whose type is not spindle (main and auxiliary, linear and rotary) in the channel's axis list that `GetChannelInfo` returns at connection. The HEIDENHAIN DNC reference states that the axis names and types in this list can change during operation; deemesh uses the list read at connection, so later changes take effect when it reconnects. In our test environment a rotary axis served as the turning spindle during turning mode and dropped out of this list (connecting in milling mode gave `X Y Z A C` and in turning mode `X Y Z A`). What changes with the mode is decided by the machine manufacturer (TNC7 User's Manual, 'Switching the operating mode with FUNCTION MODE': a mode change runs the manufacturer's macro and activates the kinematic model it defines). Read the axis list by connecting in milling mode, and reconnect after a mode change to refresh it.

## /machine/channel/axis/axisName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The axis name (e.g. `"X"`, `"Z1"`). Returns `string`, read-only. Used to confirm the correspondence between the `axis` number and the actual axis.

**A name from here can be used directly in the `axis` filter** (`axis=Z1`, `axis=X,Z`). Letter case is ignored, and a name the channel does not have is status `-18`. Names are meaningful within a channel, so each channel is looked up on its own.

Sources: the axis names in the servo load meter data on Fanuc (`cnc_rdsvmeter`, cached at connection), `/Channel/GeometricAxis/name` on Siemens, axis parameter `#1013` on Mitsubishi, and on Heidenhain the names (the programmable axis names) in the channel's axis list that `GetChannelInfo` returns at connection. On Heidenhain the `axis` number follows that list with the spindles left out. The HEIDENHAIN DNC reference states that these names can change during operation; deemesh uses the names read at connection (also when it turns a name given in the `axis` filter into a number), so later changes take effect when it reconnects. In our test environment a rotary axis served as the turning spindle during turning mode and dropped out of this list (connecting in milling mode gave `X Y Z A C` and in turning mode `X Y Z A`). What changes with the mode is decided by the machine manufacturer (TNC7 User's Manual, 'Switching the operating mode with FUNCTION MODE': a mode change runs the manufacturer's macro and activates the kinematic model it defines). The number and order of axes in the panel's position display are set by the machine configuration (manual, 'Positions workspace'), so the `axis` number may differ from the row order there; check by name. Read the axis list by connecting in milling mode, and reconnect after a mode change to refresh it.

## /machine/channel/axis/machinePosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The axis's machine coordinate. Specify the axis with the `axis` filter; a range (`axis=1-3`) or multiple selection (`axis=1,2`) is possible. The return type is `float` (64-bit double precision).

The four position addresses (machinePosition/workPosition/distanceToGo/relativePosition) are all **real distances**, in the machine's configured unit (mm/inch) as-is and matching the operator-panel display (Fanuc's internal integer representation is normalized by the SDK using each axis's decimal scaling). On Heidenhain they stay in mm when the panel is switched to inch, so they differ from the panel display then (see below). All four are only valid once the axis has established its reference point; right after power-on, check `/machine/channel/axis/axisReferencedOn` first (before establishment, plausible-looking numbers arrive silently). On Mitsubishi that address returns status `-20`, so check the panel or fall back to `/machine/channel/axis/axisAtReferencePositionOn`, which only tells whether the axis is at the reference position right now.

⚠️ **This value is the coordinate of the tool reference point (the spindle face).** `workPosition` is at the tool tip, so the difference between the two includes **tool length compensation**; `machinePosition − work offset = workPosition` does not hold (the active work coordinate system's rotation, scaling and mirroring bear on it too). If you need workpiece coordinates, read `workPosition` instead of computing them yourself.

The unit follows the machine setting (mm or inch, degrees on a rotary axis). **On Fanuc this address carries `unit`** (`mm`, `inch` or `deg`): the unit the control reported at connect. It is absent only on older series without the function that reports it (`cnc_rdposition`; per the support table in the FOCAS2 manual, Series 16/18/21, 0i-A, 15 and 15i lathes). **Fanuc fixes the unit and the decimal places at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then values come with the old decimal places and can be off by a factor of 10. Other controls carry no `unit`, because the unit is not fixed per address. Without `unit`, read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual).

**On Fanuc the machine position may not follow G20/G21.** With parameter `3104#0` at `0` (the default) it comes in the machine's own unit (parameter `1001#0`) whatever the input unit is, and with `1` it follows the input unit (parameter manual, `3104`). So on a mm machine used with inch input, this address is in mm while `workPosition`, `relativePosition` and `distanceToGo` are in inch (confirmed on the simulator). `unit` tells the two apart; on an older series without `unit`, read the two parameters through `/machine/channel/parameter/index/parameterValue` instead of `gModalCategory=4`.

**Heidenhain** reads, as PLC data, the axis position the basic PLC program copies from the NC. It equalled the "actual reference position (RFACTL)" of the control's position display (compared at rest and while moving in our test environment, the TNC7 programming station), in mm (degrees for rotary axes): in our test environment it stayed in mm both with the control's unit of measure switched to inch, so that the panel showed inch, and with an inch program (`BEGIN PGM … INCH`) running (`24.7763` while the panel showed `0.9754` inch). RFACTL is the tool position measured in the machine coordinate system M-CS (TNC7 User's Manual, 'Position displays'), and in turning mode the value also matched the panel's RFACTL (the X axis carries a diameter sign `⌀` there, but the number was the same as in milling mode). **An axis without reference information answers status `-22`**: the basic PLC program updates this value only for axes that have reference information, so on an axis that lost it (`axisReferencedOn` is `false`) the value would be stale (our test environment always has its reference established, so that case has not been confirmed). **An axis not assigned to the channel now also answers status `-22`**: on a machine that switches between milling and turning, after connecting in milling mode and switching to turning mode, the rotary axis serves as the turning spindle and drops out of the channel, and in our test environment this value stayed at `0` meanwhile even while the turning spindle was running. The value comes back on returning to milling mode. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`.

## /machine/channel/axis/workPosition
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The axis's workpiece coordinate (absolute coordinate). Returns `float`.

This value is measured at the **tool tip** and is the result after the active work offset, rotation, scaling, mirroring and tool length compensation have **all been applied**; it is the final coordinate the machine computed, so there is no need to derive it from `machinePosition` (subtraction does not give the right answer).

**When an edited table value reaches this coordinate differs by control.** Siemens fixes work offsets and tool offsets **at activation time**. Programming G500 or G54 to G599 copies the table (`$P_UIFR`) into the channel's active frame (`$P_IFRAME`) (Basic Functions K2), and a change to tool offset data takes effect the next time a T or D number is programmed (Programming Fundamentals; immediate effect only on a machine with `MD9440` set, a setting the manual flags as a collision risk). So editing G54 or a tool length at the panel while a program runs leaves this value unchanged until the next activation (or a restart after reset) (confirmed on our 840D sl bench: G54 X 80.4 to 95.0 and tool length 100 to 105, neither reflected). A coordinate-watching app should not treat "the table changed but the coordinate did not move" as a fault. On Fanuc, parameter `5001#6` (EVO: `0` for the next G43/H block, `1` for the next buffered block) decides when a tool length change applies and `5001#4` (EVR) the radius, while a work offset change is reflected in this value at once (confirmed on the simulator). On Mitsubishi, a tool compensation amount or work coordinate offset changed during automatic operation (including a single block stop) is valid from the next block or after several subsequent blocks (Instruction Manual). A `workOffsetValue` write during automatic operation, however, is refused by the control with status `-22` (confirmed on the simulator; the same with the tool compensation parameter `#11017` set to `1`). Tool offset writes were accepted during automatic operation (confirmed on the simulator).

The unit follows the machine setting (mm or inch, degrees on a rotary axis). **On Fanuc this address carries `unit`** (`mm`, `inch` or `deg`): the unit the control reported at connect. It is absent only on older series without the function that reports it (`cnc_rdposition`; per the support table in the FOCAS2 manual, Series 16/18/21, 0i-A, 15 and 15i lathes). **Fanuc fixes the unit and the decimal places at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then values come with the old decimal places and can be off by a factor of 10. Other controls carry no `unit`, because the unit is not fixed per address. Without `unit`, read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual).

On Heidenhain it is `GetCutterLocation`, which the reference describes as the tool-tip position in the workpiece coordinate system; it comes by coordinate name, so deemesh matches it to the names of `axisName`. In our test environment it matched the tool position in the panel's position display (NOML) each time a datum shift (`TRANS DATUM`), a rotation (cycle 10) and a scaling (cycle 11) were applied one after another, and the TNC7 User's Manual ('Position displays') calls that display the position in the input coordinate system (I-CS). Switching the panel's position display to the actual reference position (RFACTL) left this value as it was. **In turning mode X is a diameter value, as on the panel** (`-65.247` while the panel showed `X ⌀ -65.247` in our test environment; per the manual, the X coordinate in turning describes the workpiece diameter). deemesh returns the value HEIDENHAIN DNC gives, unchanged. In our test environment, with the control's unit of measure switched to inch and an inch program (`BEGIN PGM … INCH`) running, this value still came in mm (rotary axes in degrees). The HEIDENHAIN DNC reference says the value can also come in inch, but we could not produce that case, so we have not confirmed it. deemesh does not read `gModalCategory` on Heidenhain (status `-20`), so the method above cannot tell you the unit there. An axis that was in the channel at connection but is not among the control's coordinates now answers status `-22`: in our test environment that happened to the rotary axis `C` after connecting in milling mode and switching to turning mode, where it serves as the turning spindle, and the value came back on returning to milling mode (`/machine/channel/axisCount`).

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

The unit follows the machine setting (mm or inch, degrees on a rotary axis). **On Fanuc this address carries `unit`** (`mm`, `inch` or `deg`): the unit the control reported at connect. It is absent only on older series without the function that reports it (`cnc_rdposition`; per the support table in the FOCAS2 manual, Series 16/18/21, 0i-A, 15 and 15i lathes). **Fanuc fixes the unit and the decimal places at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then values come with the old decimal places and can be off by a factor of 10. Other controls carry no `unit`, because the unit is not fixed per address. Without `unit`, read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual).

## /machine/channel/axis/distanceToGo
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The axis's **remaining travel** in the current block. Returns `float`.

The unit follows the machine setting (mm or inch, degrees on a rotary axis). **On Fanuc this address carries `unit`** (`mm`, `inch` or `deg`): the unit the control reported at connect. It is absent only on older series without the function that reports it (`cnc_rdposition`; per the support table in the FOCAS2 manual, Series 16/18/21, 0i-A, 15 and 15i lathes). **Fanuc fixes the unit and the decimal places at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then values come with the old decimal places and can be off by a factor of 10. Other controls carry no `unit`, because the unit is not fixed per address. Without `unit`, read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual).

**Heidenhain** reads, as PLC data, the distance to go the basic PLC program copies from the NC. It equalled the distance to go (Δ) of the control's position display (compared while feeding in our test environment, the TNC7 programming station), in mm: in our test environment it stayed in mm both with the control's unit of measure switched to inch, so that the panel showed inch, and with an inch program (`BEGIN PGM … INCH`) running. **An axis without reference information answers status `-22`**: the basic PLC program updates this value only for axes that have reference information, so on an axis that lost it (`axisReferencedOn` is `false`) the value would be stale (our test environment always has its reference established, so that case has not been confirmed). **An axis not assigned to the channel now also answers status `-22`**: on a machine that switches between milling and turning, after connecting in milling mode and switching to turning mode, the rotary axis serves as the turning spindle and drops out of the channel, and in our test environment this value stayed at `0` meanwhile even while the turning spindle was running. The value comes back on returning to milling mode. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`.

## /machine/channel/axis/totalWorkOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

The **total work offset actually in effect** right now, per axis (translation). Returns `float`. **Read-only.** This corresponds to the `Total WO` row on the operator panel.

Where `workOffsetValue` is the value **stored in the table**, this address is the value **being applied**. The total is built up in layers (values measured on our 840D sl bench):

```
stored in the table      workOffsetValue?workOffset=G54     80.400   the selected system is in activeWorkOffset
+ shifts outside it      basic reference (set actual value, scratching)   20.000   the panel's Basic reference row
                         base frames, a program TRANS, cycle frames
= total in effect        totalWorkOffsetValue              100.400
```

The basic reference is the share that goes in when the operator sets the zero point in JOG with "set actual value", by scratching or with a measuring cycle (the Siemens system frame `$P_SETFRAME`), so it is added whichever coordinate system is selected. Its role matches `EXT` on Fanuc and Mitsubishi, but on Siemens it lives outside the table, so `workOffsetValue` does not show it. Compare the two addresses when diagnosing "the setting is unchanged but the part is off"; reading only the stored value hides the share added outside the table.

**This value is fixed at activation time.** Programming G500 or G54 to G599 copies the table (`$P_UIFR`) into the channel's active frame (`$P_IFRAME`); editing the table afterwards leaves this value unchanged until the next activation (or a restart after reset) (Basic Functions K2; confirmed on our 840D sl bench: editing G54 X from 80.4 to 95.0 during a run left this value at what it was before the edit). Meanwhile `workOffsetValue` reports the new value, so when the two differ it means "the table changed but is not yet in effect". A coordinate-watching app should not treat that difference as a fault.

⚠️ **Tool compensation is not in this layer.** This value covers "where the workpiece origin was moved to"; "how long the tool is" is the next layer. That is why, on any control, subtracting `workPosition` from `machinePosition` does not give this value: that difference also contains the tool length compensation, so it is off on the tool axis. Rotation, scaling and mirroring are not in it either. If you need workpiece coordinates, read `/machine/channel/axis/workPosition`; that is the result after the machine has applied all of them.

This address takes no `workOffset` filter; it is "whatever is in effect", so there is no designator to choose. `axis=1-3` expansion is supported, and writing is not (a summed result is not something you write back).

**Siemens only, because this value is not computed by deemesh: the control itself holds it.** SINUMERIK keeps the sum of the active frames (`$P_ACTFRAME`) as one value and exposes it over OPC-UA. On Fanuc (FOCAS2) and Mitsubishi (EZSocket) what deemesh reads is the **table** of `EXT` and `G54` to `G59`, and we have not confirmed a call that reads the amount of a program-set `G52` (local coordinate system) or `G92` (coordinate system setting) shift. A sum of the table entries would lack those two shifts and be **plausible yet possibly wrong**, so deemesh does not produce it (the rule against inventing a derived value the control does not hold as one value). On those two controls the address therefore answers status `-20` (not supported). Heidenhain answers status `-20` (not supported) too: what deemesh reads is the preset **table**, an edit to it reaches the coordinates only when the preset is activated again, and an axis whose cell is empty keeps the offset it had (`workOffsetValue`), so the table cannot give what is in effect now.

**How to get what you need on Fanuc and Mitsubishi**: ① If you need coordinates, read `/machine/channel/axis/workPosition`; the control computed it with `G52`, `G92` and tool compensation all applied, so this address is not needed. ② If you need the settable offset itself, read `workOffsetValue` for `EXT` and for the selected coordinate system (check it with `/machine/channel/activeWorkOffset`) and add them. On Fanuc a table edit applies at once, so that sum is the settable offset in effect. On Mitsubishi a value changed during automatic operation is valid from the next block or after several subsequent blocks (Instruction Manual), so in between the sum can be ahead of what is in effect. ③ Whether the program has set `G52` or `G92` is visible through `gModalList` and `gModalCategory`, but we have not confirmed a way to read the amount. If you need a total that includes them, judge it from the relation between `workPosition` and `machinePosition`, bearing in mind that tool compensation is mixed into that difference.

**How to get what you need on Heidenhain**: ① If you need coordinates, read `/machine/channel/axis/workPosition`. ② If you need the preset itself, put the number `/machine/channel/activeWorkOffset` returns into `workOffset` and read `workOffsetValue` (the rotation from `workOffsetRotation`). An axis whose cell is empty (`null`) cannot be known from the table, and after a table edit the table can be ahead of what is in effect until the preset is activated again. The control's status screen shows the active preset and transformations (TNC7 User's Manual, 'Status workspace'), but we have not found a way to read their sum through HEIDENHAIN DNC.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address.

## /machine/channel/axis/axisFeedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

The **component** of the actual feed along this axis (**Siemens only**). Returns `float`. Fanuc answers status `-20` (the FOCAS2 calls deemesh uses do not yield a per-axis feed component).

It is not the speed at which the tool travels along the path, but that speed resolved onto one axis. So **it keeps changing with direction even under the same command**: with `F1000` in XY, this reads 1000 while only X moves, and about 707 on a 45° diagonal.

Adding the axis components **does not give the path speed** (it is a vector magnitude: not 707+707 but √(707²+707²)=1000). Use this value to see whether a particular axis is hitting its own velocity limit and holding the path back.

The unit follows the machine setting (mm/min or inch/min). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address.

**Mitsubishi answers status `-20` as well.**

## /machine/channel/axis/axisLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The axis (servo) load rate. Returns `float` + `unit:"%"` (identical on all four). Fanuc reads the servo load meter, Siemens the drive load (`$VA_LOAD`, available for PROFIdrive drives only), and Mitsubishi the load current of the servo monitor (a ratio of the rated current). All three report the **present measured value**.

**On Siemens a value appears only on axes where machine datum `36730` `$MA_DRIVE_SIGNAL_TRACKING` is `1`.** The machine data list manual states that the control receives this value only while that datum is on and the drive sends it. On an axis where it is `0` the value is always `0` (840D sl bench: a value that was always `0` with `0` appeared while the axes and the spindle moved, once we set `1` and switched the power off and on).

**So check machine datum `36730` and confirm a value actually appears before you rely on this address on Siemens.** Reading alone cannot tell `0` from "no value". Move an axis and read `axisCurrent` alongside: if the current moves while this stays `0`, this address cannot monitor load there.

**This is the same physical quantity as `axisCurrent`.** On a servo, torque is proportional to current, so measuring load is measuring current; this address divides that value by the **motor's rated continuous current**. That makes it comparable across machines and axes (80% means 80% anywhere), while `axisCurrent` gives you the absolute figure. Converting between the two needs that motor's rated current, which deemesh does not expose - so **on a control that supports only one of them, the other cannot be derived**.

**The Mitsubishi value has not been confirmed on a machine tool.** In our test environment (a simulator) we confirmed only that it can be read; the value was always `0`.

**Heidenhain** reads, as PLC data, the motor utilization the basic PLC program reads from the drive. It is the value the control's drive diagnosis table shows as Utilization [%]. In our test environment (the TNC7 programming station) the PLC program puts fixed values there instead of the drive's, so what was confirmed is that the value equals the control's drive diagnosis table; values from a real drive have not been confirmed. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`.

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

**Fanuc and Siemens answer with status `-20`.** deemesh reads this value from the Mitsubishi drive monitor, and we have not confirmed a way to read the same value on the other two machine types.

**The Mitsubishi value has not been confirmed on a machine tool.** In our test environment (a simulator) we confirmed only that it can be read; the value was always `0`.

## /machine/channel/axis/axisCurrent
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The axis motor current. Returns `float` + `unit:"Ampere"` on both protocols. Fanuc reads the servo axis load current in amperes, Siemens the drive parameter `R0078`.

**This is the same physical quantity as `axisLoad`, in a different unit** (that one is a percentage of the motor's rating). **Mitsubishi answers status `-20`** there; the same measurement arrives through `axisLoad` as a `%` value.

**Siemens**: the value comes from the drive, so on a channel whose axis has no drive assigned this is status `-20`, a property of the machine's configuration, not a fault.

## /machine/channel/axis/axisTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The axis motor temperature. Returns `float` + `unit:"°C"` (identical on all four). For Fanuc this is diagnosis 308; for Siemens the drive parameter `R0035`; for Mitsubishi the motor temperature of the servo drive monitor.

**Siemens**: the value comes from the drive, so on a channel whose axis has no drive assigned this is status `-20`, a property of the machine's configuration, not a fault.

**The Mitsubishi value has not been confirmed on a machine tool.** In our test environment (a simulator) we confirmed only that it can be read; the value was always `0`.

**Heidenhain** reads, as PLC data, the motor temperature the basic PLC program reads from the drive. It is the value the control's drive diagnosis table shows as Temperature [°C]. In our test environment (the TNC7 programming station) the PLC program puts fixed values there instead of the drive's, so what was confirmed is that the value equals the control's drive diagnosis table; values from a real drive have not been confirmed. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`.

## /machine/channel/axis/axisPower
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The power that axis is **drawing right now**. `channel` + `axis` filters. Returns `float` + `unit:"W"`.

**It goes negative during regeneration**: a decelerating axis feeds power back, which flips the sign.

**It pairs with the cumulative `axisEnergy*` (Wh).** Those count up from power-on, so getting the usage over an interval means reading twice and subtracting; this address gives you the value at this moment directly.

**Fanuc**: diagnosis `4901`. **Siemens**: `$VA_POWER` (`vaPower`), which **only carries a value on PROFIdrive drives** (the same restriction as `axisLoad`); other axes read `0`.

**On Siemens a value appears only on axes where machine datum `36730` `$MA_DRIVE_SIGNAL_TRACKING` is `1`.** The machine data list manual states that the control receives this value only while that datum is on and the drive sends it. On an axis where it is `0` the value is always `0` (840D sl bench: a value that was always `0` with `0` appeared while the axes and the spindle moved, once we set `1` and switched the power off and on).

**So check machine datum `36730` and confirm a value actually appears before you rely on this address on Siemens.** Reading alone cannot tell `0` from "no value". Move an axis and read `axisCurrent` alongside: if the current moves while this stays `0`, this address cannot monitor power there.

**Mitsubishi answers status `-20`.**

## /machine/channel/axis/axisEnergyNet
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

The axis's net **energy** (cumulative consumed − cumulative regenerated). Returns `float` + `unit:"Wh"`, read-only.

Energy (Wh) is not power (W): W is an instantaneous rate, Wh is an accumulated amount. This value keeps climbing like an odometer, so to get the energy used by one job, **read it before and after and subtract**. For average power in W, divide by the elapsed time: `ΔWh ÷ Δhours`.

**Fanuc only** (diagnosis 4920). Siemens has no cumulative energy counter among the nodes deemesh reads, so status `-20` (instantaneous power is `axisPower`). Mitsubishi answers status `-20`.

## /machine/channel/axis/axisEnergyConsumed
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

The axis's cumulative consumed **energy**. Returns `float` + `unit:"Wh"`, read-only.

Energy (Wh) is not power (W): W is an instantaneous rate, Wh is an accumulated amount. This value keeps climbing like an odometer, so to get the energy used by one job, **read it before and after and subtract**. For average power in W, divide by the elapsed time: `ΔWh ÷ Δhours`.

**Fanuc only** (diagnosis 4921). Siemens has no cumulative energy counter among the nodes deemesh reads, so status `-20` (instantaneous power is `axisPower`). Mitsubishi answers status `-20`.

## /machine/channel/axis/axisEnergyRegenerated
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

The axis's cumulative regenerated **energy** (recovered while decelerating). Returns `float` + `unit:"Wh"`, read-only.

Energy (Wh) is not power (W): W is an instantaneous rate, Wh is an accumulated amount. This value keeps climbing like an odometer, so to get the energy used by one job, **read it before and after and subtract**. For average power in W, divide by the elapsed time: `ΔWh ÷ Δhours`.

**Fanuc only** (diagnosis 4922). Siemens has no cumulative energy counter among the nodes deemesh reads, so status `-20` (instantaneous power is `axisPower`). Mitsubishi answers status `-20`.

## /machine/channel/axis/axisReferencedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

Whether the axis has **established its machine reference point**. Returns `boolean`. **Read-only.**

On an axis where this is `false` the coordinate system is not yet established: **the position addresses (`machinePosition`, `workPosition`, `relativePosition`, `distanceToGo`) may return plausible numbers that mean nothing.** No error is raised; coordinates without a reference are returned as they are, so anything that uses positions right after power-on should check this value first. Machines with incremental encoders need a reference return before coordinates are valid.

**Machines with absolute encoders do not lose the reference when powered down.** Such an axis reads `true` from the moment the control comes up, so you may never see `false` on them - that is normal, not a fault. You do not need to know which kind you are connected to: checking this value before using a position is the one rule that works for both.

Once established it stays `true` wherever the axis moves; this is **coordinate-system validity**, not the momentary "is the axis at the reference position right now". That momentary state has its own sibling address, `/machine/channel/axis/axisAtReferencePositionOn`.

Fanuc reads the standard CNC→PMC signal ZRF (per-axis bits of `F120`) and Siemens reads `refPtStatus` (both confirmed in our test environments). **Mitsubishi returns status `-20`**: what deemesh reads on that control is "is the axis at the reference position right now" (the panel's `#1` mark, the PLC `ZP1n` signals, `GetAxisStatus`) and the zero-point initialization completion of absolute-position systems (`ZSF`), and a latched coordinate-system-established state cannot be built from those two (in our test environment the bit dropped as soon as the axis was moved after a reference return). That momentary state is what `/machine/channel/axis/axisAtReferencePositionOn` reports, with the same meaning on Fanuc and Mitsubishi. On Fanuc up to 16 axes are covered, and on a multi-path machine the signals of that path are read.

Heidenhain reads the PLC API's per-axis reference-information symbol. In the PLC API definition Heidenhain places on the control, this symbol means reference information is available for the axis, which is the meaning above. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`. Our test environment (the TNC7 programming station) always has its reference established, so only `true` has been confirmed. According to the TNC7 User's Manual a machine with absolute encoders needs no referencing, while on a machine with incremental encoders the reference screen opens after power-on, and program run cannot be selected until all axes are referenced.

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

**It is a different question from `axisReferencedOn`.** That one asks "is the coordinate system established" (once established it stays `true` wherever the axis goes); this one asks "is the axis at the reference point now" (it turns `false` when the axis moves). Use `axisReferencedOn` to decide whether positions right after power-on can be trusted, and this address when waiting for an axis to come back to its reference point - a tool-change position or a start-up condition, for instance. The Fanuc test environment (NC Guide) reporting `ZRF=7` (three axes established) together with `ZP=0` (none at the reference point) shows the difference between the two addresses in one picture.

Fanuc reads the standard CNC→PMC signal ZP (per-axis bits of `F094`, "reference position return completion": `1` while the axis is at the reference position, `0` once it has left), and Mitsubishi reads the 1st-reference-position return-completion bits of `GetAxisStatus` (the same value as the PLC `ZP1n` signals and the panel's `#1` mark); both are confirmed in our test environments. **Siemens returns status `-20`**: among the NC variables deemesh reads there is none that means "at reference point" (`refPtStatus` is the established state; `refPtBusy`, `refPtCamNo` and `refPtPhase` describe a return in progress). On Fanuc up to 16 axes are covered and on a multi-path machine the signals of that path are read; the 2nd and later reference positions (Fanuc `ZP2`..., Mitsubishi `ZP2n`...) are outside this address.

## /machine/channel/axis/axisInterlockOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc"]
write: []
```

The axis interlock state (`true` = interlock engaged). Returns `boolean`, read-only. An interlock is the ladder holding that axis still, so while this is `true` the axis does not move even when commanded.

**Fanuc only.** It reads bit 2 (Interlock state) of the status flag in the per-axis data (`cnc_rdaxisdata`). Siemens and Mitsubishi answer with status `-20`: on those controls deemesh does not use a channel that reports this state as one bit of axis data, and the interlock is a PLC signal defined by the machine builder's ladder, so read that machine's signal through `plcAddress` if you need it.

## /machine/channel/axis/axisSoftLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **positive-direction coordinate of the axis soft limit (stored stroke check 1) that is in force right now**. It is in machine coordinates and is read-only.

"Soft" means the limit is not a physical device such as a limit switch but **a boundary the control enforces by coordinate**. A command beyond this value is blocked by the control with a stroke-limit alarm.

**A soft limit can have several areas, and which one is in force is chosen by the PLC at run time.** This address follows that choice and returns **the value blocking this direction at this moment**. To read or write the setting of a given area, use `axisSoftLimitArea/axisSoftLimitPositive` with the `axisSoftLimitArea` filter; `axisSoftLimitAreaCount` tells how many areas the control has and `axisSoftLimitPositiveAreaNumber` which one is selected now. That is why this address has no write: at the instant the PLC changes the selection, a value written to "the current one" could land in either area, so settings are written only through the folder address that names the area.

**No `unit` is attached, because this is a distance.** Whether the machine works in millimetres or inches is a machine setting, so check `/machine/channel/gModalCategory/gModal?gmodalcategory=4` (`G21`/`G71`/`G710` = metric, `G20`/`G70`/`G700` = inch). Other distance values such as `machinePosition` are handled the same way.

**Fanuc**: area I is parameter `1320`, area II `1326`, areas III to VIII `1350` to `1360` (even numbers). When parameter `1301#0` (`DLM`) is `1`, the per-axis, per-direction signal `+EXLx` selects I or II; otherwise, when `1300#2` (`LMS`) is `1`, the combination of the `EXLM`, `EXLM2` and `EXLM3` signals selects I to VIII. With both at `0` it is always I. This address returns the value of the area chosen by that rule. Areas III to VIII are used by the control only with the area-expansion option: on a control without it, the control ignores a PLC that sets `EXLM2` or `EXLM3`, while this address returns the value of the area the signals select, so it differs from what is in force (confirmed on a 31i bench). The number of decimal places comes from the machine itself, so the value always matches what `/machine/channel/parameter/index/parameterValue` returns for the same number. **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (Parameter Manual B-64490EN, NOTE 1 of each parameter). The check applies after reference return, and only on a machine set to check immediately after power-on (`1311#0`=`1`) does `1300#6` decide whether it runs before reference return as well.

**Siemens**: whichever of the 1st software limit switch `$MA_POS_LIMIT_PLUS` (MD 36110) and the 2nd `$MA_POS_LIMIT_PLUS2` (MD 36130) the axis interface signal `DBX12.3` selects (`1` selects the 2nd). It is a machine-axis coordinate and applies in every mode once the axis is referenced (after `PRESET` it is off until the axis is referenced again, and modulo rotary axes are not monitored - Basic Functions manual, A3). A violation raises alarm `10720` (during block preparation), `10620` (during motion) or `10621` (resting on the switch in JOG) per the Diagnostics Manual (chapter 3, NC alarms). In the rare configuration where the mapping of channel axes to machine axes cannot be read, this address answers with status `-20`.

**Mitsubishi**: parameter `#2014` (`OT+`). There is only one area, so this equals the setting and always matches what `/machine/channel/parameter/index/parameterValue?parameter=2014` returns. If `#2013` and `#2014` hold the same non-zero value the control treats this limit as disabled (Alarm/Parameter Manual IB-1501279). The setup-level fence that narrows the range further is `axisWorkAreaLimitPositive`.

**There is no on/off switch**, so there is no sibling address asking whether it is on; whether it is effective follows from the shape of the values (see "what equal or reversed values mean" below). This is the installation-level fence that applies at all times once the axis is referenced (nothing like the working area limit's `axisWorkAreaLimitPositiveOn` exists here). What each control has instead is the area selection described above.

**What equal or reversed values mean differs by control.** If you judge "not set" from the shape of the values, do it per control. **Fanuc**: identical values make the **entire area forbidden** (Operator's Manual, CAUTION 1). Reversed values (positive smaller than negative) impose no check (the Operator's Manual, CAUTION 2, notes that an incorrectly set area removes the stroke limit). In our test environment, with the reference position established and the positive value set below the negative one, the axis crossed both boundaries in both directions during automatic operation without being stopped. Changing the values can raise alarm `OT0500` once, depending on the current position; RESET clears it. **Mitsubishi**: `#2013` equal to `#2014` (the same non-zero value) makes the limit **invalid**. Reversed values do not switch the check off; **each direction is checked against its own value**. In our test environment, with the reference position established and the two values reversed, motion in the positive direction during automatic operation stopped at `#2014` and motion in the negative direction at `#2013`, and from a position already beyond a value the axis could not start in that direction (stroke-end warning `M01 0007`). **Siemens**: there is no rule tied to the relation of the two values; the defaults are ±1.0e8, effectively unlimited, and each direction is independent. Equal values mean "locked" on Fanuc and "invalid" on Mitsubishi, the exact opposite, and reversed values remove the check on Fanuc but not on Mitsubishi, so do not read them with one control-independent rule.

## /machine/channel/axis/axisSoftLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **negative-direction coordinate of the axis soft limit (stored stroke check 1) that is in force right now**. It is in machine coordinates and is read-only.

"Soft" means the limit is not a physical device such as a limit switch but **a boundary the control enforces by coordinate**. A command below this value is blocked by the control with a stroke-limit alarm.

**A soft limit can have several areas, and which one is in force is chosen by the PLC at run time.** This address follows that choice and returns **the value blocking this direction at this moment**. To read or write the setting of a given area, use `axisSoftLimitArea/axisSoftLimitNegative` with the `axisSoftLimitArea` filter; `axisSoftLimitAreaCount` tells how many areas the control has and `axisSoftLimitNegativeAreaNumber` which one is selected now. That is why this address has no write: at the instant the PLC changes the selection, a value written to "the current one" could land in either area, so settings are written only through the folder address that names the area.

**No `unit` is attached, because this is a distance.** Whether the machine works in millimetres or inches is a machine setting, so check `/machine/channel/gModalCategory/gModal?gmodalcategory=4` (`G21`/`G71`/`G710` = metric, `G20`/`G70`/`G700` = inch). Other distance values such as `machinePosition` are handled the same way.

**Fanuc**: area I is parameter `1321`, area II `1327`, areas III to VIII `1351` to `1361` (odd numbers). When parameter `1301#0` (`DLM`) is `1`, the per-axis, per-direction signal `-EXLx` selects I or II; otherwise, when `1300#2` (`LMS`) is `1`, the combination of the `EXLM`, `EXLM2` and `EXLM3` signals selects I to VIII. With both at `0` it is always I. This address returns the value of the area chosen by that rule. Areas III to VIII are used by the control only with the area-expansion option: on a control without it, the control ignores a PLC that sets `EXLM2` or `EXLM3`, while this address returns the value of the area the signals select, so it differs from what is in force (confirmed on a 31i bench). The number of decimal places comes from the machine itself, so the value always matches what `/machine/channel/parameter/index/parameterValue` returns for the same number. **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (Parameter Manual B-64490EN, NOTE 1 of each parameter). The check applies after reference return, and only on a machine set to check immediately after power-on (`1311#0`=`1`) does `1300#6` decide whether it runs before reference return as well.

**Siemens**: whichever of the 1st software limit switch `$MA_POS_LIMIT_MINUS` (MD 36100) and the 2nd `$MA_POS_LIMIT_MINUS2` (MD 36120) the axis interface signal `DBX12.2` selects (`1` selects the 2nd). It is a machine-axis coordinate and applies in every mode once the axis is referenced (after `PRESET` it is off until the axis is referenced again, and modulo rotary axes are not monitored - Basic Functions manual, A3). A violation raises alarm `10720` (during block preparation), `10620` (during motion) or `10621` (resting on the switch in JOG) per the Diagnostics Manual (chapter 3, NC alarms). In the rare configuration where the mapping of channel axes to machine axes cannot be read, this address answers with status `-20`.

**Mitsubishi**: parameter `#2013` (`OT-`). There is only one area, so this equals the setting and always matches what `/machine/channel/parameter/index/parameterValue?parameter=2013` returns. If `#2013` and `#2014` hold the same non-zero value the control treats this limit as disabled (Alarm/Parameter Manual IB-1501279). The setup-level fence that narrows the range further is `axisWorkAreaLimitNegative`.

**There is no on/off switch**, so there is no sibling address asking whether it is on; whether it is effective follows from the shape of the values (see "what equal or reversed values mean" below). This is the installation-level fence that applies at all times once the axis is referenced (nothing like the working area limit's `axisWorkAreaLimitNegativeOn` exists here). What each control has instead is the area selection described above.

**What equal or reversed values mean differs by control.** If you judge "not set" from the shape of the values, do it per control. **Fanuc**: identical values make the **entire area forbidden** (Operator's Manual, CAUTION 1). Reversed values (positive smaller than negative) impose no check (the Operator's Manual, CAUTION 2, notes that an incorrectly set area removes the stroke limit). In our test environment, with the reference position established and the positive value set below the negative one, the axis crossed both boundaries in both directions during automatic operation without being stopped. Changing the values can raise alarm `OT0500` once, depending on the current position; RESET clears it. **Mitsubishi**: `#2013` equal to `#2014` (the same non-zero value) makes the limit **invalid**. Reversed values do not switch the check off; **each direction is checked against its own value**. In our test environment, with the reference position established and the two values reversed, motion in the positive direction during automatic operation stopped at `#2014` and motion in the negative direction at `#2013`, and from a position already beyond a value the axis could not start in that direction (stroke-end warning `M01 0007`). **Siemens**: there is no rule tied to the relation of the two values; the defaults are ±1.0e8, effectively unlimited, and each direction is independent. Equal values mean "locked" on Fanuc and "invalid" on Mitsubishi, the exact opposite, and reversed values remove the check on Fanuc but not on Mitsubishi, so do not read them with one control-independent rule.

## /machine/channel/axis/axisSoftLimitAreaCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **number of areas (sets of values)** this control has for the soft limit (stored stroke check 1). It is the upper bound of the `axisSoftLimitArea` filter of `axisSoftLimitArea/axisSoftLimitPositive` and `…Negative`; iterating that folder from `1` to this value reads the settings of every area. The value does not depend on the axis, but the `axis` filter is range-checked like on every other axis address.

- **Fanuc**: `8` when the parameter of area III (`1350`) exists in the parameter table, `2` otherwise; checked once at connection. On the controls we tested, `1350` to `1361` were in the table regardless of the area-expansion option, so `8` was returned, while whether areas III to VIII can actually be selected depends on that option (without it the control ignores the `EXLM2` and `EXLM3` signals). We have not yet confirmed how to detect the option through FOCAS2, so this value is the size of the table.
- **Siemens**: always `2` (the 1st and 2nd software limit switches).
- **Mitsubishi**: always `1` (the single set `#2013`/`#2014`).

## /machine/channel/axis/axisSoftLimitPositiveAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **number of the area in force right now for the positive-direction soft limit** of the axis (`1` to `axisSoftLimitAreaCount`). It says which area's value `axisSoftLimitPositive` is returning; to change that setting, put this number into the `axisSoftLimitArea` filter of `axisSoftLimitArea/axisSoftLimitPositive`. It exists per direction because both Fanuc and Siemens can select a different area for each direction.

- **Fanuc**: computed from the PLC selection signals. When parameter `1301#0` (`DLM`) is `1`, `+EXLx` (the axis bit of `Gn104`) selects `0` = I or `1` = II; otherwise, when `1300#2` (`LMS`) is `1`, it is the three-bit value of `EXLM3`, `EXLM2`, `EXLM` (`Gn531.7`, `Gn531.6`, `Gn007.6`) plus `1` (`000` = 1 ... `111` = 8). With both at `0` it is `1`. On a control without the area-expansion option the control ignores `EXLM2` and `EXLM3`, while this value is computed from the signals as they are, so if such a machine's PLC holds those two signals on, this value differs from the area the control actually uses (confirmed on a 31i bench).
- **Siemens**: `2` when the axis interface signal `DBX12.3` (2nd software limit switch plus) is `1`, otherwise `1`.
- **Mitsubishi**: always `1`, there is only one area.

**There is no write; the machine decides which area applies.** Several areas exist so that the usable stroke can follow the machine's situation (tailstock position, an attachment), and that decision is made every scan by the machine builder's ladder through PLC signals. This value reports the outcome, so it is an observation of the same kind as `executionStatus`. Writing those signals from outside either changes nothing (the ladder overwrites them on the next scan) or, on a machine whose ladder does not drive them, changes the protected area regardless of the machine's situation. To switch areas, use the ladder or the operator panel switch. The setting of an area itself is written through `axisSoftLimitArea/axisSoftLimitPositive`.

## /machine/channel/axis/axisSoftLimitNegativeAreaNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **number of the area in force right now for the negative-direction soft limit** of the axis (`1` to `axisSoftLimitAreaCount`). It says which area's value `axisSoftLimitNegative` is returning; to change that setting, put this number into the `axisSoftLimitArea` filter of `axisSoftLimitArea/axisSoftLimitNegative`. It exists per direction because both Fanuc and Siemens can select a different area for each direction.

- **Fanuc**: computed from the PLC selection signals. When parameter `1301#0` (`DLM`) is `1`, `-EXLx` (the axis bit of `Gn105`) selects `0` = I or `1` = II; otherwise, when `1300#2` (`LMS`) is `1`, it is the three-bit value of `EXLM3`, `EXLM2`, `EXLM` (`Gn531.7`, `Gn531.6`, `Gn007.6`) plus `1` (`000` = 1 ... `111` = 8). With both at `0` it is `1`. On a control without the area-expansion option the control ignores `EXLM2` and `EXLM3`, while this value is computed from the signals as they are, so if such a machine's PLC holds those two signals on, this value differs from the area the control actually uses (confirmed on a 31i bench).
- **Siemens**: `2` when the axis interface signal `DBX12.2` (2nd software limit switch minus) is `1`, otherwise `1`.
- **Mitsubishi**: always `1`, there is only one area.

**There is no write; the machine decides which area applies.** Several areas exist so that the usable stroke can follow the machine's situation (tailstock position, an attachment), and that decision is made every scan by the machine builder's ladder through PLC signals. This value reports the outcome, so it is an observation of the same kind as `executionStatus`. Writing those signals from outside either changes nothing (the ladder overwrites them on the next scan) or, on a machine whose ladder does not drive them, changes the protected area regardless of the machine's situation. To switch areas, use the ladder or the operator panel switch. The setting of an area itself is written through `axisSoftLimitArea/axisSoftLimitNegative`.

## /machine/channel/axis/axisSoftLimitArea/axisSoftLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis", "axisSoftLimitArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **positive-direction setting of the soft limit (stored stroke check 1) of the area named by the `axisSoftLimitArea` filter**. It is in machine coordinates and supports both read and write. Area numbers run from `1` to `axisSoftLimitAreaCount`; anything else answers with status `-18`.

Which area is in force is chosen by the PLC, so the value read here is not necessarily the one being enforced. The enforced value is `axisSoftLimitPositive` and the selected area number is `axisSoftLimitPositiveAreaNumber`; to change the setting in force, put that number into this filter.

**No `unit` is attached, because this is a distance.** Whether the machine works in millimetres or inches is a machine setting, so check `/machine/channel/gModalCategory/gModal?gmodalcategory=4` (`G21`/`G71`/`G710` = metric, `G20`/`G70`/`G700` = inch).

- **Fanuc**: area I is `1320`, II `1326`, III to VIII `1350`, `1352`, ... `1360`. The value always matches what `/machine/channel/parameter/index/parameterValue` returns for the same number. **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (Parameter Manual B-64490EN, NOTE 1 of each parameter).
- **Siemens**: area `1` is `$MA_POS_LIMIT_PLUS` (MD 36110), area `2` is `$MA_POS_LIMIT_PLUS2` (MD 36130). **A written value takes effect at NEW CONF level**: it is activated once the axis has stopped and the channels of the mode group the axis belongs to are in Reset (the same procedure as the panel's "Activate MD" or the `NEWCONF` command), so a value written during operation leaves the previous one enforced until then. In the rare configuration where the mapping of channel axes to machine axes cannot be read, this address answers with status `-20`.
- **Mitsubishi**: only area `1` exists, parameter `#2014` (`OT+`).

**Writing follows the machine's write-enable state.** On Fanuc a parameter write that is not enabled, and on Mitsubishi a write while the part system is in automatic operation (including a pause), is refused with status `-22` (machine state); on Siemens the value is machine data, and a refusal (protection level and the like) comes back as status `-17` with the vendor's reason. On Mitsubishi a value outside the setting range answers status `-16` (confirmed in our test environment with `#2014`), and places finer than the setting unit `#1003` are rounded by the control, which then answers status `0` (see `parameterValue`), so read the value back to see what was stored. Setting this value incorrectly can stop an axis short of where it needs to travel, or loosen the protection, so on a real machine read the current value before changing it. What equal or reversed values mean on each control is described under `axisSoftLimitPositive`.

## /machine/channel/axis/axisSoftLimitArea/axisSoftLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis", "axisSoftLimitArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **negative-direction setting of the soft limit (stored stroke check 1) of the area named by the `axisSoftLimitArea` filter**. It is in machine coordinates and supports both read and write. Area numbers run from `1` to `axisSoftLimitAreaCount`; anything else answers with status `-18`.

Which area is in force is chosen by the PLC, so the value read here is not necessarily the one being enforced. The enforced value is `axisSoftLimitNegative` and the selected area number is `axisSoftLimitNegativeAreaNumber`; to change the setting in force, put that number into this filter.

**No `unit` is attached, because this is a distance.** Whether the machine works in millimetres or inches is a machine setting, so check `/machine/channel/gModalCategory/gModal?gmodalcategory=4` (`G21`/`G71`/`G710` = metric, `G20`/`G70`/`G700` = inch).

- **Fanuc**: area I is `1321`, II `1327`, III to VIII `1351`, `1353`, ... `1361`. The value always matches what `/machine/channel/parameter/index/parameterValue` returns for the same number. **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (Parameter Manual B-64490EN, NOTE 1 of each parameter).
- **Siemens**: area `1` is `$MA_POS_LIMIT_MINUS` (MD 36100), area `2` is `$MA_POS_LIMIT_MINUS2` (MD 36120). **A written value takes effect at NEW CONF level**: it is activated once the axis has stopped and the channels of the mode group the axis belongs to are in Reset (the same procedure as the panel's "Activate MD" or the `NEWCONF` command), so a value written during operation leaves the previous one enforced until then. In the rare configuration where the mapping of channel axes to machine axes cannot be read, this address answers with status `-20`.
- **Mitsubishi**: only area `1` exists, parameter `#2013` (`OT-`).

**Writing follows the machine's write-enable state.** On Fanuc a parameter write that is not enabled, and on Mitsubishi a write while the part system is in automatic operation (including a pause), is refused with status `-22` (machine state); on Siemens the value is machine data, and a refusal (protection level and the like) comes back as status `-17` with the vendor's reason. On Mitsubishi a value outside the setting range answers status `-16` (confirmed in our test environment with `#2014`), and places finer than the setting unit `#1003` are rounded by the control, which then answers status `0` (see `parameterValue`), so read the value back to see what was stored. Setting this value incorrectly can stop an axis short of where it needs to travel, or loosen the protection, so on a real machine read the current value before changing it. What equal or reversed values mean on each control is described under `axisSoftLimitNegative`.

## /machine/channel/axis/axisWorkAreaLimitPositive
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **positive-direction coordinate of the axis working area limit**. It is a second, setup-level boundary inside `axisSoftLimitPositive` (the fixed limit set at machine installation), in machine coordinates on Fanuc and Mitsubishi and in the basic coordinate system (BCS) on Siemens, and supports both read and write.

**Fanuc**: stored stroke check 2 (parameter `1322`). Depending on configuration this feature means either "stay within" or "keep out" (a chuck barrier, for example), and **this address promises only the stay-within meaning.** On a control configured as a no-entry box (parameter `1300#0` = `0`) the address refuses with status `-20` instead of returning a value - computing "safe if between these" from it would be exactly backwards (`axisWorkAreaLimitOn` answers the same status `-20` then). **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (Parameter Manual B-64490EN, NOTE 1 of each parameter). The per-axis enable (`1310#0`) and the modal (`G22`/`G23`) are described under `axisWorkAreaLimitOn`.

**Mitsubishi**: stored stroke limit II (parameter `#8205` `OT-CHECK-P`) - the parameter the manual's soft limit I entry points to for narrowing the range in actual use (`#8204`/`#8205`). It has the same mode check as Fanuc, but **per axis** and with the **opposite polarity**: `#8210 OT INSIDE` = `0` (inhibits outside = limit II, stay-within) answers, `1` (inhibits inside = limit IIB, a no-entry box) makes that axis answer with status `-20`. Whether the check is in use is `axisWorkAreaLimitOn`, and `#8204` equal to `#8205` disables it. In lathe G-code lists 6 and 7 a program can change `#8204`/`#8205` and switch the check on with `G22 X_ Z_ I_ K_` and off with `G23` (non-modal, lathe Programming Manual), so this value can change while a program runs. On machining centres `G22`/`G23` is a different feature (stroke check before travel: a program-specified no-entry box checked before the move, error `P452`) and unrelated to this address.

**Siemens**: the setting datum `$SA_WORKAREA_LIMIT_PLUS` (SD 43420). This feature always means stay-within, so there is no mode refusal. A violation raises alarm `10730` (during block preparation), `10630` (during motion) or `10631` (in JOG) per the Diagnostics Manual (chapter 3, NC alarms). In the rare configuration where the mapping of channel axes to machine axes cannot be read, this address answers with status `-20`. **Writing works only while the channel is in Reset.** Setting data is something Siemens refuses to alter from outside depending on the channel state: while a program is running, stopped, or the control is in emergency stop, the control refuses, raises alarm `4230` (which reports that data cannot be changed from outside in the channel's current state; Diagnostics Manual chapter 3), and this address answers with status `-22` (machine state) (the error text carries that reason). The Diagnostics Manual's entry for alarm `4230` likewise states that this data cannot be entered while a part program runs, naming working area limitation setting data and dry run feedrate as examples; on our 840D sl bench a channel interrupted by an emergency stop refused as well. The `4230` raised by a refused write stays in the alarm list until the next NC start or until it is cleared on the operator panel, and `alarmStatus` is `2` meanwhile (Diagnostics Manual; confirmed on an 840D sl bench). The soft limit, which is machine data, has no such restriction. Per the List Manual (12/2019) this setting datum is a value in the **basic coordinate system (BCS)** (identical to machine coordinates on a machine without a transformation), takes effect immediately, and has user protection level (7/7). A program can also change it with `G26` (positive direction) / `G25` (negative direction); whether such a change survives a reset depends on machine datum `10710` (`$MN_PROG_SD_RESET_SAVE_TAB`). Under `WALIMOF` the value is ignored even when set. The monitored point is the **tool tip**, so the tool length is taken into account automatically (the radius only with machine datum `21020`), and the check runs in both AUTO and JOG (Basic Functions manual, A3). The coordinate-system-specific working area limitation in WCS/SZS (`WALCS0`-`WALCS10`) is a separate feature and unrelated to this address (Programming Manual).

**A value is only enforced while the check is on.** The switch is the per-axis `axisWorkAreaLimitOn` on Fanuc and Mitsubishi and the per-direction `axisWorkAreaLimitPositiveOn` and `axisWorkAreaLimitNegativeOn` on Siemens; how to tell, per control (Fanuc `G22`/`G23` modal, Siemens `WALIMON` plus the switch, Mitsubishi `#8202`), is described under those addresses.

**No `unit` is attached, because this is a distance.** Millimetres or inches is a machine setting (handled like `axisSoftLimitPositive`). Writing follows the machine's parameter-write enable state; if writing is blocked, status `-22` (machine state). On Mitsubishi a write while the part system is in automatic operation (including a pause) or while data protect key 2 (PLC signal `*KEY2`, `Y709`, which protects user parameters) is off answers status `-22`, and a value outside the setting range answers status `-16`. Places finer than the setting unit `#1003` are rounded by the control, which then answers status `0`, so read the value back to see what was stored (the protect key and the rounding were confirmed on the simulator with `#8205`).

**What equal or reversed values mean also differs by control.** **Fanuc**: identical values make the **entire area movable** for check 2 (Operator's Manual, CAUTION 1, the opposite of check 1, the soft limit). Reversed values are taken as they are, the rectangular parallelepiped having the two points as vertices becoming the boundary (CAUTION 2). **Mitsubishi**: `#8204` equal to `#8205` (same sign and value) makes the limit invalid; reversed values make limit II (stay-within) **prohibit the entire range**, while IIB (a no-entry box) prohibits the range between the two points (Alarm/Parameter Manual IB-1501279). **Siemens**: there is no rule tied to the relation of the two values, each direction is independent, and switching on and off is done by the per-direction switch addresses.

## /machine/channel/axis/axisWorkAreaLimitNegative
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

The **negative-direction coordinate of the axis working area limit**. It is a second, setup-level boundary inside `axisSoftLimitNegative` (the fixed limit set at machine installation), in machine coordinates on Fanuc and Mitsubishi and in the basic coordinate system (BCS) on Siemens, and supports both read and write.

**Fanuc**: stored stroke check 2 (parameter `1323`). Depending on configuration this feature means either "stay within" or "keep out" (a chuck barrier, for example), and **this address promises only the stay-within meaning.** On a control configured as a no-entry box (parameter `1300#0` = `0`) the address refuses with status `-20` instead of returning a value - computing "safe if between these" from it would be exactly backwards (`axisWorkAreaLimitOn` answers the same status `-20` then). **On a diameter-programmed axis (lathe X, typically) the value is a diameter** (Parameter Manual B-64490EN, NOTE 1 of each parameter). The per-axis enable (`1310#0`) and the modal (`G22`/`G23`) are described under `axisWorkAreaLimitOn`.

**Mitsubishi**: stored stroke limit II (parameter `#8204` `OT-CHECK-N`) - the parameter the manual's soft limit I entry points to for narrowing the range in actual use (`#8204`/`#8205`). It has the same mode check as Fanuc, but **per axis** and with the **opposite polarity**: `#8210 OT INSIDE` = `0` (inhibits outside = limit II, stay-within) answers, `1` (inhibits inside = limit IIB, a no-entry box) makes that axis answer with status `-20`. Whether the check is in use is `axisWorkAreaLimitOn`, and `#8204` equal to `#8205` disables it. In lathe G-code lists 6 and 7 a program can change `#8204`/`#8205` and switch the check on with `G22 X_ Z_ I_ K_` and off with `G23` (non-modal, lathe Programming Manual), so this value can change while a program runs. On machining centres `G22`/`G23` is a different feature (stroke check before travel: a program-specified no-entry box checked before the move, error `P452`) and unrelated to this address.

**Siemens**: the setting datum `$SA_WORKAREA_LIMIT_MINUS` (SD 43430). This feature always means stay-within, so there is no mode refusal. A violation raises alarm `10730` (during block preparation), `10630` (during motion) or `10631` (in JOG) per the Diagnostics Manual (chapter 3, NC alarms). In the rare configuration where the mapping of channel axes to machine axes cannot be read, this address answers with status `-20`. **Writing works only while the channel is in Reset.** Setting data is something Siemens refuses to alter from outside depending on the channel state: while a program is running, stopped, or the control is in emergency stop, the control refuses, raises alarm `4230` (which reports that data cannot be changed from outside in the channel's current state; Diagnostics Manual chapter 3), and this address answers with status `-22` (machine state) (the error text carries that reason). The Diagnostics Manual's entry for alarm `4230` likewise states that this data cannot be entered while a part program runs, naming working area limitation setting data and dry run feedrate as examples; on our 840D sl bench a channel interrupted by an emergency stop refused as well. The `4230` raised by a refused write stays in the alarm list until the next NC start or until it is cleared on the operator panel, and `alarmStatus` is `2` meanwhile (Diagnostics Manual; confirmed on an 840D sl bench). The soft limit, which is machine data, has no such restriction. Per the List Manual (12/2019) this setting datum is a value in the **basic coordinate system (BCS)** (identical to machine coordinates on a machine without a transformation), takes effect immediately, and has user protection level (7/7). A program can also change it with `G26` (positive direction) / `G25` (negative direction); whether such a change survives a reset depends on machine datum `10710` (`$MN_PROG_SD_RESET_SAVE_TAB`). Under `WALIMOF` the value is ignored even when set. The monitored point is the **tool tip**, so the tool length is taken into account automatically (the radius only with machine datum `21020`), and the check runs in both AUTO and JOG (Basic Functions manual, A3). The coordinate-system-specific working area limitation in WCS/SZS (`WALCS0`-`WALCS10`) is a separate feature and unrelated to this address (Programming Manual).

**A value is only enforced while the check is on.** The switch is the per-axis `axisWorkAreaLimitOn` on Fanuc and Mitsubishi and the per-direction `axisWorkAreaLimitPositiveOn` and `axisWorkAreaLimitNegativeOn` on Siemens; how to tell, per control (Fanuc `G22`/`G23` modal, Siemens `WALIMON` plus the switch, Mitsubishi `#8202`), is described under those addresses.

**No `unit` is attached, because this is a distance.** Millimetres or inches is a machine setting (handled like `axisSoftLimitNegative`). Writing follows the machine's parameter-write enable state; if writing is blocked, status `-22` (machine state). On Mitsubishi a write while the part system is in automatic operation (including a pause) or while data protect key 2 (PLC signal `*KEY2`, `Y709`, which protects user parameters) is off answers status `-22`, and a value outside the setting range answers status `-16`. Places finer than the setting unit `#1003` are rounded by the control, which then answers status `0`, so read the value back to see what was stored (the protect key and the rounding were confirmed on the simulator with `#8205`).

**What equal or reversed values mean also differs by control.** **Fanuc**: identical values make the **entire area movable** for check 2 (Operator's Manual, CAUTION 1, the opposite of check 1, the soft limit). Reversed values are taken as they are, the rectangular parallelepiped having the two points as vertices becoming the boundary (CAUTION 2). **Mitsubishi**: `#8204` equal to `#8205` (same sign and value) makes the limit invalid; reversed values make limit II (stay-within) **prohibit the entire range**, while IIB (a no-entry box) prohibits the range between the two points (Alarm/Parameter Manual IB-1501279). **Siemens**: there is no rule tied to the relation of the two values, each direction is independent, and switching on and off is done by the per-direction switch addresses.

## /machine/channel/axis/axisWorkAreaLimitOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "axis"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **switch of the axis working area limit, per axis**. `true` means both values, `axisWorkAreaLimitPositive` and `axisWorkAreaLimitNegative`, are effective settings; `false` means both are ignored. Read and write.

The switch of the working area limit comes in different units per control. Fanuc and Mitsubishi have **one per axis**, read and written through this address; Siemens has **one per axis and direction**, so this address answers with status `-20` (not supported) there and the per-direction addresses `axisWorkAreaLimitPositiveOn` and `axisWorkAreaLimitNegativeOn` are used instead. Whichever of the two answers something other than status `-20` is that machine's switch. Either way this is a **setting state** in the same sense as the other `…On` addresses (`singleBlockOn` and friends): the switch is on, which is not the same as the limit being enforced at this instant, and whether it is actually enforced needs one more look that differs by control.

- **Fanuc**: parameter `1310#0` (`OT2x`: `0` = disabled, `1` = enabled). It is a bit parameter, so a write reads that byte, changes bit 0 only, and writes the other bits back as they were (`#1` is the check 3 switch). The same mode check as the value addresses applies: on a control configured as a no-entry box (parameter `1300#0` = `0`) that switch is not a working-area switch, so both read and write answer with status `-20`. Stroke check 2 as a whole is on or off by the `G22` (on) / `G23` (off) modal, visible in `gModalList`, and parameter `3402#7` decides which one the control starts in at power-on; a control without the check 2 option enforces nothing even under `G22`. With both values identical, check 2 treats the entire area as movable even while the switch is on, so there is effectively no limit (Operator's Manual, CAUTION 1). Writing follows the machine's parameter-write enable state; if writing is blocked, status `-22` (machine state).
- **Mitsubishi**: the inverse of parameter `#8202 OT-CHECK OFF` (`0` = in use reads as `true`). The same gate as Fanuc exists **per axis**: with `#8210 OT INSIDE` at `1` (inhibits inside = limit IIB, a no-entry box) that axis answers with status `-20` for both read and write. With `#8204` equal to `#8205` the check is void even while the switch is on.
- **Siemens**: there is no per-axis switch, so status `-20`. Use the per-direction addresses.

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

- **Siemens**: this switch (SD 43400) **and** the channel modal `WALIMON` (`WALIMON`/`WALIMOF` in `/machine/channel/gModalList`). While a program has issued `WALIMOF`, nothing is enforced even with the switch `true`. The switch is setting data, so **writing works only while the channel is in Reset**; otherwise alarm `4230` and status `-22` (machine state). The `4230` raised by a refused write stays in the alarm list until the next NC start or until it is cleared on the operator panel, and `alarmStatus` is `2` meanwhile (Diagnostics Manual; confirmed on an 840D sl bench). The manual lists it as `BOOLEAN`, effective immediately, user protection level - it is the very value the operator toggles under the panel's "Parameters" area to switch the working area limitation on and off.
- **Fanuc**: the switch is **per axis**, not per direction, so this address answers with status `-20`. Read and write `axisWorkAreaLimitOn` instead (parameter `1310#0` and the `G22`/`G23` modal are described there).
- **Mitsubishi**: per axis as well, so status `-20`. Use `axisWorkAreaLimitOn` (parameter `#8202` and the `#8210` gate are described there).

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

- **Siemens**: this switch (SD 43410) **and** the channel modal `WALIMON` (`WALIMON`/`WALIMOF` in `/machine/channel/gModalList`). While a program has issued `WALIMOF`, nothing is enforced even with the switch `true`. The switch is setting data, so **writing works only while the channel is in Reset**; otherwise alarm `4230` and status `-22` (machine state). The `4230` raised by a refused write stays in the alarm list until the next NC start or until it is cleared on the operator panel, and `alarmStatus` is `2` meanwhile (Diagnostics Manual; confirmed on an 840D sl bench). The manual lists it as `BOOLEAN`, effective immediately, user protection level - it is the very value the operator toggles under the panel's "Parameters" area to switch the working area limitation on and off.
- **Fanuc**: the switch is **per axis**, not per direction, so this address answers with status `-20`. Read and write `axisWorkAreaLimitOn` instead (parameter `1310#0` and the `G22`/`G23` modal are described there).
- **Mitsubishi**: per axis as well, so status `-20`. Use `axisWorkAreaLimitOn` (parameter `#8202` and the `#8210` gate are described there).

## /machine/channel/spindleCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The channel's spindle count. Cached at connection time. The valid range of the `spindle` filter is `1` to this value.

**On Fanuc and Siemens it differs per path.** A path with no spindle reads `0`, and the spindle addresses on that channel then answer with status `-20` on Fanuc (including `spindleOverride` and `spindleSpeedCommanded`, which are channel-wide values there) and status `-18` on Siemens (Siemens reads those two per spindle as well).

**On Mitsubishi it is the spindle count of the whole NC** (parameter `#1039 spinno`, a base common parameter), so every channel reports the same value. deemesh reads it from parameter `#1039` for each channel, and spindles appear to be numbered NC-wide, so `spindle=1` up to this value addresses every spindle from any channel. Confirmed on a simulator with two spindles and two part systems: both part systems accept `spindle=1` and `2` (`3` answers status `-18`), and the commanded speed of each spindle reads the same from both part systems. We have not confirmed this on a machine tool.

**On Heidenhain** it is the number of all the control's spindles: at connection deemesh counts the entries of spindle type in the axis list that `GetAxesInfo` of HEIDENHAIN DNC returns. It is not only the spindles assigned to the channel now, so on a machine that switches between milling and turning it counts both the milling spindle and the turning spindle (`2` in our test environment; the channel's axis list gives only `S1` in milling mode and only `S2` in turning mode). The `spindle` number follows that list and does not depend on the operating mode.

## /machine/channel/spindle/spindleOverride
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The spindle override (%). Returns `float` + `unit:"%"`, the value **to the 0.1% digit**. It is usually a whole percent, but a control set to 0.1% steps reports a fraction such as `87.5`. **Fanuc is a channel-common value by default** (the `G30` signals `SOV0` to `SOV7`, binary 0 to 254%, all on reads `0` as the control treats it), so every spindle shows the same value; on a machine with parameter `3713#3` (MSC) and `#4` (EOV) both set it is **per spindle** (spindle 2 `G376`, 3 `G377`, 4 `G378`; 5 and above are status `-20`) (Connection Manual B-64483EN-1). On a multi-path control the path's own signals are read (addresses shift by `1000` per path). Siemens and Mitsubishi give a per-spindle value. **Mitsubishi is read the way the machine builder's ladder selects it** (PLC Interface Manual IB-1501272): with the per-spindle method-selection signal `SPS` (`Y188F`, `+0x60` per spindle) off, the code signals `SP1`/`SP2`/`SP4` (50-120% in 10% steps) are decoded; with it on, the register `R7008` (0-200% in 1% units, `+50` per spindle) is read.

Heidenhain gives a single spindle override from `GetOverrideInfo` (a whole percent), so every `spindle` reads the same value. In our test environment it matched the value on the operator panel.

**Writing is supported on Heidenhain only** (`SetOverrideSpeed`; the other controls answer status `-20` (not supported)). Write a whole percentage, as in `{"value": 80}`. A value whose fractional part is `0` (`80.0`) is accepted; a value with a fractional part such as `50.5`, or a negative value, is status `-16` (invalid write value). The machine decides the range it accepts, and **in our test environment the control clamped a value outside that range to the nearest end and answered status `0`** (in our test environment the range for the spindle was from `50` to `130`, so writing `0` set `50` and writing `200` set `130`). The TNC7 User's Manual ('Cutting data') gives the spindle override potentiometer a range of 0% to 150% (effective only on machines with an infinitely variable spindle drive) and says that the maximum spindle speed depends on the machine. Read the value back to see what was set. In our test environment a written value took about 0.1 s to show up in a read, so a read right after the write could still return the previous value. There is a single spindle override, so writing through any `spindle` changes that one value. A written value takes effect during automatic operation as well and stays after the run ends. Operating the override on the operator panel sets the panel's value, and writing again sets the written one (whichever changed last; confirmed with the virtual dial in our test environment, not with the dial of a real machine). **Write caution**: the spindle speed of a running machine changes at once.

## /machine/channel/spindle/spindleSpeedCommanded
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The spindle **S command value**. Returns `float`. **No `unit` is attached**: what the command means depends on the spindle speed mode (a rotational speed under constant-speed mode, a surface speed under constant-surface-speed mode). That holds on Fanuc, Siemens and Mitsubishi; Heidenhain answers status `-22` instead of a value under constant surface speed (see below). Which mode is active is `/machine/channel/gModalCategory/gModal?gModalCategory=8`: the response's `desc` carries the machine-independent meaning - `constant surface speed` means a surface speed (`G96`, on Siemens also `G961`/`G962`), `constant spindle speed (rpm)` means a rotational speed (`G97`, on Siemens also `G971`/`G972`/`G973` and `G94`/`G95` from the same group). **Fanuc is the channel modal S value** (the `spindle` filter is ignored; the S command is a channel-level concept); Siemens is the per-spindle `cmdSpeed`, and Mitsubishi the per-spindle S command modal value. **On Siemens the sign follows the direction of rotation** (negative under `M4`, 840D sl bench); Fanuc reads positive under both `M3` and `M4` (31i bench); we have not confirmed Mitsubishi.

This address was previously named `/machine/channel/spindle/speedCommanded`; the old address keeps working as-is, but the documentation describes only this name.

**Heidenhain** reads, as PLC data, the programmed S value the basic PLC program receives. It is the last S commanded even while the spindle is stopped, and it equalled the S of the control's status line (in our test environment, the TNC7 programming station, with a rotational speed command). **When the spindle's last command is a constant surface speed it answers status `-22`**: the speed follows the diameter, so there is no commanded speed, and the error text carries the cutting speed. Read the speed meanwhile with `/machine/channel/spindle/spindleSpeedActual`. In our test environment constant surface speed applied only to the turning spindle in turning mode, and after returning to milling mode that spindle still answered status `-22` because its last command stays. A cutting speed given in a milling tool call (`TOOL CALL 3 Z S(VC=100)`) is turned into a spindle speed when the tool is called, so it is not constant surface speed and this address gives that speed (with a 6 mm tool in our test environment the panel showed S `5305` and this address `5305.165`). It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`. The `spindle` number follows the control's spindle list and does not depend on the operating mode (`/machine/channel/spindleCount`).

## /machine/channel/spindle/spindleSpeedActual
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The per-spindle actual speed. Returns `float` + `unit:"rpm"` on all four protocols: Fanuc uses `cnc_acts2`, Siemens the per-spindle `actSpeed`, and Mitsubishi the speed item of the spindle monitor. All three specify the target spindle with the `spindle` filter, and the value is the **measured speed with override applied**. **On Siemens the sign follows the direction of rotation** (the System Variables List Manual defines the sign of `$AA_S` that way; on an 840D sl bench `M4` read negative); Fanuc gives the magnitude whatever the direction (both `M3` and `M4` read positive on a 31i bench and on a machine tool); we have not confirmed the sign convention of Mitsubishi.

**The Mitsubishi value has not been confirmed on a machine tool.** In our test environment (a simulator) we confirmed only that it can be read; the value was always `0`, even while the spindle turned.

This address was previously named `/machine/channel/spindle/speedActual`; the old address keeps working as-is, but the documentation describes only this name.

**Heidenhain** reads, as PLC data, the actual speed the basic PLC program reads from the drive. It is a magnitude without direction, and it equalled the S of the control's status line and the speed in the drive diagnosis table (in our test environment, the TNC7 programming station). It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`. The `spindle` number follows the control's spindle list and does not depend on the operating mode (`/machine/channel/spindleCount`).

## /machine/channel/spindle/spindleLoad
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The spindle load rate. Returns `float` + `unit`. Siemens, Mitsubishi and Heidenhain always carry `unit:"%"`. On Fanuc the vendor response carries the unit itself, so `%` or `rpm` arrives depending on the machine configuration, and in the rare case the vendor reports some other unit code the `unit` key is omitted. Do not assume `%`; read `unit`. On Mitsubishi this is the load item of the spindle monitor.

**When `unit` is `%` this is the same physical quantity as `spindleCurrent`** - the control divides the motor current by its rating to produce a load rate. That makes it comparable across machines (80% means 80% anywhere), while `spindleCurrent` gives you the absolute figure. Converting between the two needs that motor's rated current, which deemesh does not expose, so **on a control that supports only one of them, the other cannot be derived**. When Fanuc reports `unit:"rpm"` the value is a speed rather than a load and this relationship does not hold.

**The Mitsubishi value has not been confirmed on a machine tool.** In our test environment (a simulator) we confirmed only that it can be read; the value was always `0`, even while the spindle turned.

**Heidenhain** reads, as PLC data, the motor utilization the basic PLC program reads from the drive. It is the value the control's drive diagnosis table shows as Utilization [%]. In our test environment (the TNC7 programming station) the PLC program puts fixed values there instead of the drive's, so what was confirmed is that the value equals the control's drive diagnosis table; values from a real drive have not been confirmed. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`. The `spindle` number follows the control's spindle list and does not depend on the operating mode (`/machine/channel/spindleCount`).

## /machine/channel/spindle/spindleCurrent
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_opcua_siemens"]
write: []
```

The spindle motor current. **Siemens only** (drive parameter `R0078`). Returns `float` + `unit:"Ampere"`.

**This is the same physical quantity as `spindleLoad`, in a different unit** (that one is a `%` of the motor's rating). Fanuc and Mitsubishi answer status `-20` there; the same measurement is available through `spindleLoad` - though on Fanuc check `unit` first, since it may report `rpm` depending on the machine's configuration.

**Siemens**: the value comes from the drive, so on a channel whose spindle has no drive assigned this is status `-20`, a property of the machine's configuration, not a fault.

## /machine/channel/spindle/spindleTemperature
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The spindle motor temperature. Returns `float` + `unit:"°C"` (identical on all four). For Fanuc this is diagnosis 403; for Siemens the drive parameter `R0035`; for Mitsubishi the motor temperature of the spindle drive monitor.

**Siemens**: the value comes from the drive, so on a channel whose spindle has no drive assigned this is status `-20`, a property of the machine's configuration, not a fault.

**The Mitsubishi value has not been confirmed on a machine tool.** In our test environment (a simulator) we confirmed only that it can be read; the value was always `0`.

**Heidenhain** reads, as PLC data, the motor temperature the basic PLC program reads from the drive. It is the value the control's drive diagnosis table shows as Temperature [°C]. In our test environment (the TNC7 programming station) the PLC program puts fixed values there instead of the drive's, so what was confirmed is that the value equals the control's drive diagnosis table; values from a real drive have not been confirmed. It is PLC data, so the connection needs `access_password`; when it is missing or the control rejects it, the status is `-20`. If deemesh finds none of the symbol names it knows for this value on the machine, the status is also `-20`. Symbol names can differ between machines' PLC programs; if you know the name that holds this value on that machine, read it with `/machine/plcAddress/plcType/plcValue`. The `spindle` number follows the control's spindle list and does not depend on the operating mode (`/machine/channel/spindleCount`).

## /machine/channel/spindle/spindlePower
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens"]
write: []
```

The power that spindle is **drawing right now**. `channel` + `spindle` filters. Returns `float` + `unit:"W"`. The rules are those of `axisPower` (negative while regenerating, cumulative in `spindleEnergy*`).

**Fanuc**: diagnosis `4902`. **Siemens**: there is no spindle-area node for this, so the value of that spindle's **machine axis** is read (`vaPower`). On the rare machine where the control does not map the spindle to a machine axis, the address answers status `-20`. A value appears only while machine datum `36730` `$MA_DRIVE_SIGNAL_TRACKING` is `1` on that machine axis; with `0` it is always `0` (840D sl bench: after setting `1`, 90 to 110 W at 1350 rpm and negative while decelerating; see `axisPower` for the detail).

**There is no address for total machine power.** On Fanuc the total the control measures itself can be read, but on Siemens we have not confirmed such a value, so deemesh would have to add up the axes and spindles: the same address would then be a measured value on one control and a sum of ours on the other. **Because the control samples each value at a different instant, the total need not equal the sum of the parts even on Fanuc.** If you need a total, add the axis and spindle values yourself, and bear in mind it may differ from what the control measures.

**Mitsubishi answers status `-20`.**

## /machine/channel/spindle/spindleEnergyNet
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

The spindle's net **energy** (cumulative consumed − cumulative regenerated). Returns `float` + `unit:"Wh"`, read-only.

Energy (Wh) is not power (W): W is an instantaneous rate, Wh is an accumulated amount. This value keeps climbing like an odometer, so to get the energy used by one job, **read it before and after and subtract**. For average power in W, divide by the elapsed time: `ΔWh ÷ Δhours`.

**Fanuc only** (diagnosis 4930). Siemens has no cumulative energy counter among the nodes deemesh reads, so status `-20` (instantaneous power is `spindlePower`). Mitsubishi answers status `-20`.

## /machine/channel/spindle/spindleEnergyConsumed
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

The spindle's cumulative consumed **energy**. Returns `float` + `unit:"Wh"`, read-only.

Energy (Wh) is not power (W): W is an instantaneous rate, Wh is an accumulated amount. This value keeps climbing like an odometer, so to get the energy used by one job, **read it before and after and subtract**. For average power in W, divide by the elapsed time: `ΔWh ÷ Δhours`.

**Fanuc only** (diagnosis 4931). Siemens has no cumulative energy counter among the nodes deemesh reads, so status `-20` (instantaneous power is `spindlePower`). Mitsubishi answers status `-20`.

## /machine/channel/spindle/spindleEnergyRegenerated
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "spindle"]
read: ["nc_focas2_fanuc"]
write: []
```

The spindle's cumulative regenerated **energy** (recovered while decelerating). Returns `float` + `unit:"Wh"`, read-only.

Energy (Wh) is not power (W): W is an instantaneous rate, Wh is an accumulated amount. This value keeps climbing like an odometer, so to get the energy used by one job, **read it before and after and subtract**. For average power in W, divide by the elapsed time: `ΔWh ÷ Δhours`.

**Fanuc only** (diagnosis 4932). Siemens has no cumulative energy counter among the nodes deemesh reads, so status `-20` (instantaneous power is `spindlePower`). Mitsubishi answers status `-20`.

## /machine/channel/activeWorkOffset
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

Returns **the work coordinate system selected now**, in the notation that machine's `workOffset` filter takes. Returns `string`; read only. Put the value as it is into the `workOffset` filter of `/machine/channel/workOffset/axis/workOffsetValue` to read that work coordinate system's offset.

The values by control:

- **Fanuc**: `G54`–`G59`, and an additional work coordinate system with its P number, as in `G54.1P2`. The P number is read from the custom macro system variable `#4330` (the additional work coordinate system number of the block being executed, Operator's Manual B-64484EN §16). `EXT` is the common offset added to every coordinate system, so it never appears. On a control where `#4330` cannot be read because it has no custom macro option, the P number is unknown: when the control reports the additional work coordinate system as `G54` (as our simulator and a 31i-B machine tool did), `G54` comes out even while an additional work coordinate system is in effect, and when it reports `G54.1` the read answers status `-22` (machine state). All our test machines have that option, so this case is not confirmed
- **Mitsubishi**: `G54`–`G59`. **While `G54.1` is in effect the read answers status `-22` (machine state).** We have not confirmed a way to read which extended work coordinate system it is (the P number) with the EZSocket reference we have. That `G54.1` is in effect is answered by `/machine/channel/gModalCategory/gModal?gModalCategory=7` (confirmed on the simulator in both a machining-centre and a lathe configuration: with `G54.1 P2` in effect we saw this refusal and that `G54.1`). A configuration that is neither machining centre nor lathe (`machineType` = `unknown`) returns status `-20`
- **Siemens**: the settable frames `G500`, `G54`–`G57`, `G505`–`G599`. `G500` is the state with the settable offset switched off, and the `workOffset` filter takes that notation too
- **Heidenhain**: the number of the active preset (`0`, `1` …), the row marked active in the preset table (confirmed in our test environment: it followed when the preset was changed; the TNC7 User's Manual, 'Preset table', says the control enters `1` in the `ACTNO` column of the active row). It is a number rather than a G code because the Heidenhain `workOffset` filter takes the preset number

⚠️ **This address only says which work coordinate system is selected.** It does not mean that offset is what reaches the coordinates now: on Fanuc and Mitsubishi `EXT` is added, on Siemens the old value stays in effect after a table edit until the next activation (`totalWorkOffsetValue`), and on Heidenhain the old value stays in effect after a table edit until the preset is activated again, an axis whose preset cell is empty keeps the offset it had before, and a datum shift from the datum table or `TRANS DATUM` and (depending on the machine) a pallet preset can be added on top. If you need workpiece coordinates, read `/machine/channel/axis/workPosition`.

**How it differs from `gModalCategory=7`**: on the G-code controls both look at the same modal. While an additional work coordinate system is in effect that address answers `G54.1`, and on Fanuc this address adds the P number (`G54.1P2`) so the value can go back into the filter (Mitsubishi answers status `-22` as above, and Siemens has no `G54.1`). This address answers on Heidenhain as well.

On Fanuc the value follows **the block being executed**. When a block read ahead changes the work coordinate system, the previous value holds until that block runs.

Range expansion (`channel=1-2`) is supported. Each channel selects on its own, so the channels can answer differently (on the 840D sl bench, channel 1 `G54` and channel 2 `G500`).

## /machine/channel/workOffset/axis/workOffsetValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

**Work coordinate system offset**: the per-axis offset distance of a work coordinate system such as G54 (read + write). Returns `float`, a real distance (in the machine's configured unit mm/inch as-is; the SDK normalizes Fanuc's internal integer representation by the decimal scaling).

The `workOffset` filter **takes the shop-floor notation directly**: the G-code notation on the G-code controls, the preset number on Heidenhain (an open namespace like `plcAddress`, no separate numbering system). Case-insensitive; whitespace and aliases are not allowed:

- **Fanuc and Mitsubishi**: `EXT` (the common offset added to all coordinate systems, the operator-panel EXT row), `G54`–`G59`, extended `G54.1P1`–`G54.1P300`. Fanuc rejects a P number not in the option with a vendor error. Mitsubishi can answer status `0` with the value `0` for a number beyond the option too (confirmed on the simulator: in a lathe configuration with the 48-set extended work coordinate system option, a program was refused at `G54.1 P49`, yet this address answered the value `0` for `G54.1P49`–`G54.1P96`, and status `-17` (vendor error) from `G54.1P97` on). To see how many sets a machine has, check the option list on the operator panel's diagnosis screen (on the simulator the number of sets appeared in that list). The two accept **the same notation and reject with the same message**
- **Siemens**: `G500`, `G54`–`G57`, `G505`–`G599`. How many actually exist depends on the machine configuration, so a designator that does not exist answers with status `-18` together with **the list this machine accepts**
- **Heidenhain**: the preset table number (`0`, `1` …, the row number of the preset table on the operator panel). G-code notation is rejected with status `-18` (filter value error), and a number that is not in the table answers status `-18` together with **the range of numbers the table has**

⚠️ **`G500` is not the same as Fanuc's `EXT`.** `EXT` is a common offset that is **added on top** of whichever `G5x` is active, whereas `G500` is an **exclusive member of the same modal group** as `G54`–`G57` and therefore cannot be active alongside them; when `G500` is in effect the settable offset is switched off, and the value in that slot is normally `0`. Read `/machine/channel/activeWorkOffset` to see which one is selected (its value goes into `workOffset` as it is). On Siemens the part that adds to every coordinate system the way `EXT` does lives in **separate frames** (the `Basic reference` and `Total basic WO` rows on the operator panel), and deemesh does not expose those individually; for the combined result, read `/machine/channel/axis/totalWorkOffsetValue`.

**On Siemens the value is the coarse offset plus the fine offset.** The machine applies that sum and the operator panel shows them as two cells of one offset (`Coarse` and `Fine`), so this address gives you **the offset actually in effect**; comparing it against the panel's `Coarse` cell alone can look like a mismatch. To read the fine part on its own, use `/machine/channel/workOffset/axis/workOffsetFineValue`. Fanuc and Mitsubishi have no fine offset, so their value is single; that is what makes this address mean the same thing on Fanuc, Mitsubishi and Siemens (Heidenhain in the next paragraph).

**On Heidenhain the value is a cell of the preset table.** For the X, Y and Z axes it is the basic transformation cell (`X`, `Y`, `Z`); for any other axis it is that axis's offset cell (the axis name followed by `_OFFS`, as in `A_OFFS`). The offset cells of the X, Y and Z axes (`X_OFFS`, `Y_OFFS`, `Z_OFFS`) are not included: the basic transformation is a value in the basic coordinate system, while an offset cell shifts that axis in the machine coordinate system, and how the two act together depends on the machine kinematics, so deemesh does not add them into one value (TNC7 User's Manual, 'Preset table'). If deemesh does not find that axis's cell in the preset table, the status is `-20` (not supported). The preset's rotation (`SPA`, `SPB`, `SPC`, the spatial angles that set the basic rotation of the workpiece coordinate system) is not part of this value; `workOffsetRotation` reports it. This value is what the preset table holds; during machining a datum shift from the datum table or `TRANS DATUM` and (depending on the machine) a pallet preset can be added on top of it (manual, 'Preset table' and 'Datum table').

**On Heidenhain an empty cell is `null`.** The preset table allows empty cells, and an empty cell does not mean `0`: it means **this preset does not set that axis**. Activating such a preset left that axis at the offset already in effect (confirmed in our test environment: activating a preset whose X, Y and Z were all empty did not change the workpiece coordinates; the TNC7 User's Manual, 'Preset table', also says an empty cell keeps the previous value on activation while a cell holding `0` overwrites it). This address therefore cannot tell you what is in effect on that axis; read `workPosition` for the coordinates. A single read carries this explanation in `desc`. Fanuc, Mitsubishi and Siemens have a number in every cell and never produce `null` here.

⚠️ **This value is the stored translation.** Two more things bear on it. ① A work coordinate system can also carry **rotation, scaling and mirroring** (`workOffsetRotation`, `workOffsetScale`, `workOffsetMirrorOn`), and where those are set the coordinate transform is not determined by this value alone. ② The total actually in effect can differ from this value, because a basic reference and other frames add to it (`totalWorkOffsetValue`; Siemens only, and that section explains how to get the offset in effect on the other controls). If you need part coordinates, do not compute them; read `/machine/channel/axis/workPosition`. On an ordinary setup that only translates, rotation is `0`, scaling is `1` and mirroring is `false`, so this value *is* the transform.

**Whether the table can be edited during automatic operation, and when an edit reaches the coordinates, differ by control.** This address is the stored value, so it reports a panel edit **at once**.

| Control | Editing during automatic operation | Effect on coordinates |
|---|---|---|
| Fanuc | Allowed | Reflected in `workPosition` at once (confirmed on the simulator and a 31i bench) |
| Siemens | Allowed (at the panel) | Not until the next activation (programming G500 or G54 to G599, or a restart after reset); the table is copied into the channel's active frame at that moment (Basic Functions K2; confirmed on the test bench). `totalWorkOffsetValue` reports what is in effect now |
| Mitsubishi | Refused by the control during automatic operation: a write to this address answers status `-22`, and the panel shows "Executing automatic operation" (confirmed on the simulator; the same with the tool compensation parameter `#11017` set to `1`) | A value changed at the panel during automatic operation is valid from the next block or after several subsequent blocks (Instruction Manual) |
| Heidenhain | Not confirmed | Not until the preset is activated again (confirmed in our test environment by editing a cell of the active preset); activating it applies the table values, and an axis whose cell is empty stays as it was |

On Siemens, the time the two addresses differ is exactly the "changed but not yet applied" state; a coordinate-watching app should not treat it as a fault.

`axis` is the axis number (1–). All four controls take the axis name as well (the name `/machine/channel/axis/axisName` returns, letter case ignored). `axis=1-3` · `workOffset=G54,G55` expansion is supported; for Fanuc, axis expansion of the same workOffset is bundled into a single FOCAS call. Heidenhain expands a number range such as `workOffset=0-24` and answers all the presets and axes of one request from a single read of the table.

Writes take `{"value": 25.4}` (a single axis). **Supported on Fanuc and Mitsubishi. On Siemens the write answers status `-20`, and that is a deliberate exclusion, not something unimplemented.** The node holding this value is read/write in the vendor variable manual (`$P_UIFR`), but the same manual states that **the PI service `SETUFR` has to be called to activate the settable frames** (NC Variables List Manual, Area C Block FU). deemesh does not call that service over OPC-UA (the methods we found under the server's `/Methods` are for file handling and tool management), so writing the value alone is accepted while the offset actually in effect and the operator panel display both stay as they were (observed on our test bench). A write that looks like it succeeded and does nothing is exactly what deemesh refuses to pass through. To change an offset on Siemens, set it at the operator panel. **Heidenhain answers status `-20` too.** A value written to the preset table reaches the coordinates only when the preset is activated again (table above), and deemesh does not activate presets, so for the same reason as Siemens it does not accept the write. Set the offset at the operator panel.

**Fanuc and Mitsubishi round a written value to that axis's decimal places and answer status `0`.** On Fanuc deemesh rounds to the axis's decimal places before sending; on Mitsubishi the setting unit parameter `#1003` decides those places (in our test environment a 1 µm setting stored `12.345678` as `12.346`, and a 1 nm setting stored it unchanged). Read the value back to see what was stored. On Mitsubishi, a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on the simulator; deemesh tells this case apart by reading that signal after the refusal).

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10. Unlike this rule, Heidenhain linear axes are always in mm (deemesh selects mm when it reads through DNC).

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

**Mitsubishi answers status `-20` as well.**

## /machine/channel/workOffset/axis/workOffsetRotation
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

The **per-axis rotation angle** of a work coordinate system. Filters are the same as `/machine/channel/workOffset/axis/workOffsetValue`. Returns `float` with `unit` `deg` (degrees). **Read-only**.

`0` means no rotation is set on that axis.

**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five; on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**On Heidenhain the value is a spatial-angle cell of the preset table.** The X axis is `SPA`, the Y axis `SPB` and the Z axis `SPC`, each the angle of rotation around that axis. The TNC7 User's Manual, 'Preset table', says the control interprets these three as a basic rotation (when only `SPC` is used) or a 3D basic rotation of the workpiece coordinate system. The order and sign of the rotations are the same as the Siemens default setting (machine datum `10600` `$MN_FRAME_ANGLE_INPUT_MODE` at `1`, RPY angles), so the same three values mean the same rotation: around the fixed axes in the order X, Y, Z (the TNC7 User's Manual, 'PLANE SPATIAL', describes the spatial angles as rotations in the order A, B, C, and Basic Functions K2 gives RPY angles in the order Z, Y', X''; confirmed in our test environment by entering values in the three cells, activating the preset and comparing against `workPosition`). On a Siemens machine with machine datum `10600` at `2` (ZX'Z'' Euler angles) the same three values mean a different rotation. Any other axis (a rotary axis such as `A` or `C`) answers status `-18`. The offset cell of a rotary axis (`A_OFFS` and so on) is not a rotation but a shift of that axis, and `workOffsetValue` reports it. This is the value written in the table, so after the table is edited it can differ from the rotation in effect until the preset is activated again.

**Writing is not supported** (status `-20`). On Siemens, writing to these values directly makes the machine **accept the request and change nothing** (confirmed on our test bench). A separate activation step is required on the machine side, and deemesh does not call it. On Heidenhain too, a value written to the preset table takes effect only when the preset is activated again, and deemesh does not activate presets. Make changes at the operator panel. The translation `workOffsetValue` answers status `-20` to a write on both controls for the same reason (on Siemens, `workOffsetFineValue` as well).

**Readable on Siemens and Heidenhain.**

**Fanuc and Mitsubishi answer with status `-20`.** Their work offset tables store translation only (`workOffsetValue`).

## /machine/channel/workOffset/axis/workOffsetScale
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

The **per-axis scale factor** of a work coordinate system. Filters are the same as `/machine/channel/workOffset/axis/workOffsetValue`. Returns `float`; it is dimensionless, so there is no `unit`. **Read-only**.

`1` means no scaling (true size). `2` machines at double size along that axis.

**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five; on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**Writing is not supported.** Writing to these values directly makes the machine **accept the request and change nothing** (measured). A separate activation step is required on the machine side, and deemesh does not call it; make changes at the operator panel. The translation (`workOffsetValue` and `workOffsetFineValue`) is status `-20` on Siemens for the same reason.

**Siemens only.**

**Fanuc and Mitsubishi answer with status `-20`.** Their work offset tables store translation only (`workOffsetValue`).

## /machine/channel/workOffset/axis/workOffsetMirrorOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["channel", "workOffset", "axis"]
read: ["nc_opcua_siemens"]
write: []
```

Whether the work coordinate system **mirrors** that axis. Filters are the same as `/machine/channel/workOffset/axis/workOffsetValue`. Returns `boolean`. **Read-only**.

`true` reverses the direction of that axis. This is the per-axis checkbox on the work-offset detail screen of the operator panel.

**These three are components of the same coordinate frame as the translation.** The actual coordinate transform is translation + rotation + scale + mirror, so computing part coordinates means reading all five; on an ordinary setup that only translates, rotation is `0`, scale is `1` and mirror is `false`, so `workOffsetValue` alone is enough.

**Writing is not supported.** Writing to these values directly makes the machine **accept the request and change nothing** (measured). A separate activation step is required on the machine side, and deemesh does not call it; make changes at the operator panel. The translation (`workOffsetValue` and `workOffsetFineValue`) is status `-20` on Siemens for the same reason.

**Siemens only.**

**Fanuc and Mitsubishi answer with status `-20`.** Their work offset tables store translation only (`workOffsetValue`).

## /machine/channel/gModalCategory/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalCategory"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
filter_codes: {"gModalCategory": [{"value": 1, "name": "motion"}, {"value": 2, "name": "plane"}, {"value": 3, "name": "distanceMode"}, {"value": 4, "name": "units"}, {"value": 5, "name": "feedMode"}, {"value": 6, "name": "cutterComp"}, {"value": 7, "name": "coordinateSystem"}, {"value": 8, "name": "spindleSpeedMode"}]}
```

Queries the active G modal by a **machine-independent standard group number** (a vendor-neutral number defined by deemesh like `plcType`, not the vendor's raw group number). `gModalCategory` filter values:

⚠️ **What is neutral is the question you ask (the group number), not the answer.** The value that comes back is that machine's G code, so it cannot carry a machine-independent branch. The same state reads `G21` on Fanuc and `G710` on Siemens. `desc` tells you the meaning, but it is **prose for a human**, not a contract to branch on; the wording can change. This address is for a host application that knows its own machine's G codes. When you need a decision that spans machine types, use an address that answers the question directly: `feedActual` for the effective feed, `spindleSpeedActual` for spindle rpm, and so on.

- `1` = motion: feed mode (G00 rapid / G01 linear / G02·G03 circular …)
- `2` = plane: machining plane (G17 XY / G18 ZX / G19 YZ)
- `3` = distanceMode: absolute/incremental (G90/G91) · **on a Fanuc lathe this depends on the G code system.** System A has no such modal group at all (it expresses absolute/incremental with the `U`/`W` addresses) and the address answers status `-20`; systems B and C do have it and return a value. The system is set by `GSB` (bit 6) and `GSC` (bit 7) of parameter `3401`, which deemesh reads when it connects
- `4` = units: inch/metric (G20·G70·G700 / G21·G71·G710). On Siemens, `G70`/`G71` switch coordinates only, `G700`/`G710` also feedrates and offsets (Programming Manual)
- `5` = feedMode: feed specification (per-minute/per-rev/inverse-time)
- `6` = cutterComp: tool-radius compensation (G40 cancel / G41 left / G42 right)
- `7` = coordinateSystem: work coordinate system. On Fanuc and Mitsubishi G54–G59, and `G54.1` for an additional work coordinate system (confirmed for Fanuc on the simulator and on a 31i-B machine tool, and for Mitsubishi on the simulator). On a Fanuc control that cannot read the additional work coordinate system number (`#4330`) because it has no custom macro option, an additional work coordinate system can come out as `G54`. On Siemens `G500`, `G54`–`G57`, `G505`–`G599`. For which additional work coordinate system it is (the P number), read `/machine/channel/activeWorkOffset`
- `8` = spindleSpeedMode: constant surface speed (G96) / constant rpm (G97)

The value is that machine's G-code string, plus for key combinations a machine-independent meaning in `desc` (e.g. Fanuc `{"value":"G21","desc":"metric"}`, Siemens `{"value":"G710","desc":"metric"}`). For access to the raw vendor groups, use `gModalGroup/gModal` (one group) or `gModalList` (all). When no modal is in effect for that group (including combinations the machine type does not support), the value is `null`, the same representation as that slot of `gModalList`. On Mitsubishi a group the control refuses reads `null`, while a communication error answers an error, not `null`. **On Siemens, `feedMode` (`5`) and `spindleSpeedMode` (`8`) are the same group, so their values are always identical.** SINUMERIK puts the feed types (`G93`, `G94`, `G95`) and the spindle-speed types (`G96`, `G97`) in one G group, so only one value is ever active and only `desc` distinguishes which question you asked:

```
machine in G94
  gModalCategory=5 → {"value":"G94","desc":"feed per minute"}
  gModalCategory=8 → {"value":"G94","desc":"constant spindle speed (rpm)"}
```

`G94` is a feed code, not a spindle code. The `desc` reads that way because "not in the G96 family" means the spindle is not in constant surface speed - so **testing `value == "G97"` for constant-rpm never matches on Siemens.** On Fanuc and Mitsubishi the two groups are separate, so the values differ.

**On Mitsubishi this is supported on machining centres and lathes alike** (all eight groups). The group numbers were verified to be the same in both programming manuals (the machining-centre and lathe G-code list tables). On a lathe **the same meaning appears as different G codes** depending on the G-code list (parameter `#1037 cmdtyp`): absolute/incremental is `G90`/`G91` (lists 3, 5, 7) or `G190`/`G191` (lists 2, 4, 6), feed is `G94`/`G95` or `G98`/`G99`, so read the meaning from `desc`, not from the value; all of these carry a `desc`. Only a configuration that is neither machining centre nor lathe (`machineType` = `unknown`) returns status `-20`. Raw vendor access (`gModalGroup/gModal` for one group, `gModalList` for all) works regardless of machine type, since it promises raw vendor numbering and no meaning.

This address was previously named `/machine/channel/gGroup/gModal` (filter `gGroup`); the old address and filter keep working as-is, but the documentation describes only this name.

## /machine/channel/gModalGroup/gModal
```yaml
value_type: "string"
null_able: true
required_filters: ["channel", "gModalGroup"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

Reads **one slot of `gModalList`, picked by group number** (`string`). The `gModalGroup` filter takes **the group number that control's communication interface uses**, in the same numbering as the array positions of `gModalList`:

- **Fanuc**: the number FOCAS uses, `0`-`36` (the `type` of `cnc_rdgcode`). The value is what FOCAS reports, so while an additional work coordinate system (`G54.1`) is in effect the work coordinate system group can arrive as `G54` (confirmed on NC Guide; the neutral category `gModalCategory=7` answers `G54.1`)
- **Siemens**: G-function groups `1`-`N` (N = the machine's group count; `64` on a measured 840D sl)
- **Mitsubishi**: the vendor API's group numbers, `1`-`21`

⚠️ **These are interface numbers, not the `Group` column of the programming manual.** Whether the two agree may differ by machine type, so rather than feeding a manual's group number straight in, read `gModalList` once and confirm the position. Fanuc's numbering starts at `0`.

A number outside the range is rejected with status `-18` carrying the valid range. When no modal is in effect for that group, or the machine does not have that group, the value is **`null`**, exactly as that slot of `gModalList` reads. On Mitsubishi a group the control refuses reads `null`, while a communication error answers an error, not `null`.

The meaning of the group numbers belongs to each vendor and is **not translated and not unified across machine types** (a raw pass-through, like `plcAddress`). To pick a group machine-independently, use `/machine/channel/gModalCategory/gModal`, which takes neutral numbers. The value is that control's G-code string either way, so it cannot be used for machine-independent branching (see `gModalCategory/gModal` for details).

**This address is particularly useful on Mitsubishi.** That control costs one round trip per group, so `gModalList` is 21 round trips while this address is one; and on a configuration where `gModalCategory/gModal` answers status `-20` (`machineType` = `unknown`), raw numbers are still readable one at a time here.

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

- **Fanuc**: FOCAS group order `0`-`36`, so **the length is always 37**. Groups the machine does not have read `null` (measured on a 31i: 34 arrive and `24`, `25`, `28` are empty). **On a control where `cnc_rdgcode` cannot be used, everything from `21` on is `null`**: the fallback (`cnc_modal`) covers only G code groups `0` to `20` per the FOCAS2 specification. While an additional work coordinate system (`G54.1`) is in effect, the work coordinate system group can still arrive as `G54` (the value FOCAS reports, as it is; confirmed on NC Guide). The neutral category `/machine/channel/gModalCategory/gModal?gModalCategory=7` answers `G54.1` then
- **Siemens**: `ncFkt` G-function group order 1–N (N = the machine's group count). A group with no G function in effect reads `null` (measured: 7 of 64)
- **Mitsubishi**: vendor group order 1–21 (`GetGCodeCommand`). **The length is always 21**; a group the machine does not have, or one with no modal in effect, reads `null`. A communication error is not filled in as `null`: when one occurs partway, the whole list answers the error. Codes are formatted as the vendor manual's examples show, with a two-digit integer part (`G02`, `G50.2`), so they look the same as Fanuc's

**The position in the array is the group number.** Missing entries are filled with `null` rather than skipped precisely to keep that correspondence; packing them forward would seat later entries in someone else's slot.

⚠️ **On Mitsubishi this costs one round trip per group (21).** The vendor API reads one group per call. It is meant for drawing a modal screen once, not for periodic polling; when you need just one group, read it through `gModalGroup/gModal` (one round trip).

**None of the three addresses in this family gives you a machine-independent value.** This list and `gModalGroup/gModal` are raw vendor numbering and order, and even `gModalCategory/gModal` neutralizes only the number you use to select a group; its value is still that machine's G code. **Use none of them for decisions that span machine types** (see the `gModalCategory/gModal` entry for details).

The length is set by that machine type's vendor group count (listed above), so `[]` never appears; a slot with no G-code in effect is `null`.

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

- **Siemens answers `null`.** The control reports "nothing is assigned to this letter" as a distinct state (source `/Channel/SelectedFunctions/`; the vendor manual says an unassigned entry yields a negative number), and that state is returned as `null`. Measured on an idle machine, `T`, `S`, `H` and `M` were `null`, `D` was `1` and `F` was `0` (where the letter has a real value you get that value). **Measured while running, the values behaved as modal values, not per-block ones**: in the block after `M3 S500 T="CUTTER 10"` (a `G4` dwell), `S` still read `500` and `M` still read `3`. So `S` and `M` are modal here, as they are on Fanuc and Mitsubishi. **They do go back to `null` when the program ends (`M30`)**, however, and on those two controls a reset does not clear them, so this is where the two differ. In the same measurement **`T` stayed `null` even though it had been programmed** (that machine performs the tool change on `T` itself). Read the commanded tool from `/machine/channel/activeToolNumber` rather than from this address. `H` can be commanded negative on Siemens, and since the control also marks "unassigned" with a negative number, a negative `H` command cannot be told apart from `null` at this address.
- **Fanuc and Mitsubishi answer `0`.** Those controls have no separate "not commanded" representation: `0` covers both "nothing commanded" and a commanded `0` (`T0` is a command people actually use). We do not invent a distinction the machine does not make, so it is not turned into `null`.

So **testing for "has anything been commanded" with `== 0` alone will not catch it on Siemens, and `== null` alone will not catch it on Fanuc or Mitsubishi.** Which value counts as "none" is for the host application, which knows the machine, to decide.

## /machine/channel/partCountActual
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

The number of parts machined so far, the counter you reset when switching jobs. `channel` filter. Returns `int`; both read and write are supported; write `{"value": 0}` to reset it. If the Fanuc library does not include the function this write uses (`cnc_wrparam`), the write is status `-20` (which functions a Fanuc library includes depends on the package).

**This is not "how many parts were actually produced".** It is a counter the control increments in response to program end (`M02`/`M30`), and whether it reacts at all depends on the machine configuration. With that setting off it never moves; a dry run increments it too; running the same program twice makes it 2. An operator can change it at the panel. **The control does not know whether a part is good.** To use it for production reporting, the host application must overlay program and timing information.

**On Fanuc a read right after a write may still return the old value**: the control takes a moment to apply a parameter (measured: reading immediately gave the old value 6 times out of 8; after `50` ms all reads were correct). Read again shortly afterwards to confirm what was written. On the same Fanuc, macro variable writes apply immediately; only parameters behave this way. Siemens applies immediately.

Fanuc uses parameter `6711` (incremented on `M02`/`M30` or the M code set in parameter `6710`; with `6700#0`=`1` only that M code counts), Siemens `actParts` (`$AC_ACTUAL_PARTS`: counts only when bit 8 of machine datum `27880` is set, by default on `M02`/`M30`, or on the M code set in `27882` when bit 9 is set; **with the target comparison enabled (`27880` bit 0) and `partCountRequired` above `0`, it is automatically reset to `0` the moment it reaches the target** - unlike Fanuc, which keeps counting and only raises a signal), and Mitsubishi parameter `8002` (counted when the M code set in parameter `8001` executes; with `8001` at `0` nothing is counted and the panel's work-count display is off, M800 Instruction Manual). **On Mitsubishi this is read-only**, so a reset write returns status `-20`.

## /machine/channel/partCountRequired
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
```

The target quantity to be produced. `channel` filter. Returns `int`; both read and write are supported; write `{"value": 100}`. A `0` means no target is set. If the Fanuc library does not include the function this write uses (`cnc_wrparam`), the write is status `-20` (which functions a Fanuc library includes depends on the package).

The machine can be configured to signal or stop once the machined quantity reaches this value, but whether it does so depends on the machine configuration; deemesh only carries the value.

**On Fanuc a read right after a write may still return the old value**: the control takes a moment to apply a parameter (measured: reading immediately gave the old value 6 times out of 8; after `50` ms all reads were correct). Read again shortly afterwards to confirm what was written. Siemens applies immediately.

Fanuc uses parameter `6713` (`0` is treated as infinite and the arrival signal `PRTSF` is never output), Siemens `reqParts` (`$AC_REQUIRED_PARTS`: the target comparison, alarm and PLC signal need bit 0 of machine datum `27880`; with that set, reaching a value above `0` resets `partCountActual` to `0`), and Mitsubishi parameter `8003`. **On Mitsubishi this is read-only.**

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

Fanuc uses parameter `6712` (incremented together with `partCountActual`, under the same conditions) and Siemens `totalParts` (`$AC_TOTAL_PARTS`: counts only when bit 4 of machine datum `27880` is set; by default it increments on `M02`/`M30`, or on the M code of `27882` when bit 5 is set). **Mitsubishi answers status `-20`**: the standard value deemesh can read on that control is the resettable job counter (`partCountActual`); a lifetime total is something the machine builder implements in the PLC, which the SDK cannot know.

## /machine/channel/programRunDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The **run time of the current automatic cycle**, which restarts from `0` when a new cycle begins. `channel` filter. Returns `int` (seconds) + `unit:"s"`, read-only.

It is not a running total. Unlike the values that accumulate over the machine's life (power-on time, cutting time), this measures **one run**. It does not count while stopped - Fanuc excludes stop and hold time (Parameter Manual B-64490EN, `6758`) and returns to `0` at power-on and **on a cycle start from the reset state** (resuming from feed hold continues the count); Siemens excludes stopped time and time halted by feed override `0`, and returns to `0` when `M30` is reached or the program is restarted from reset (confirmed on an 840D sl bench: it stops during feed hold, a single-block stop and feed override `0`, and counts during a dwell). Mitsubishi likewise restarts from `0` on a cycle start from the reset state, stops during feed hold and block stop (`M0`) and continues on resume, and keeps its value after reset and `M30` (confirmed on the simulator). We have not confirmed how Mitsubishi treats feed override `0`.

**Sub-second precision is discarded.** Fanuc and Siemens offer millisecond resolution, but elapsed-time addresses are uniformly whole seconds (Mitsubishi gives whole seconds). You get the same value the control shows as `CYCLE TIME` (`CYC` on Mitsubishi) (confirmed in the Fanuc and Mitsubishi test environments and on a 31i bench). Fanuc keeps counting while the feed override is `0` or during a dwell, and does not count during a feed hold (31i bench). For a cycle only a few seconds long, the up-to-one-second difference is relatively large.

Fanuc reads parameters `6758` (minutes) and `6757` (milliseconds below a minute) together in a single call; Siemens uses `actProgNetTime`, and Mitsubishi the cycle time (`GetCycleTime`; the range the reference IB-1501209 gives ends at `99:59:59`).

**On Mitsubishi this needs EZSocket `FCSB1224W100-A9` or later.** Earlier versions answer status `-20`; on M800-series controls, installing that version or later makes it readable. M700-series controls cannot read it whatever the version, because the support table of the reference (IB-1501209) lists this function for the M800 series only; when the control answers that it is not supported, the status is `-20` (we have not confirmed this on an M700-series machine).

## /machine/channel/operatingDuration
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

**The accumulated automatic-operation time.** It keeps adding up the time the machine spends running a program automatically. `channel` filter. Returns `int` (seconds) + `unit:"s"`, read-only.

It is the counterpart to `programRunDuration`: that one measures **the current cycle**, this one the **life of the machine**. The `program` in that name is what marks its scope, so an address without that prefix accumulates, like `powerOnDuration` and `cuttingDuration`.

**It does not count while held or stopped.** Time parked in feed hold is excluded on all three controls (on Fanuc the parameter manual states that stop and hold time are not included, and our test environment and a 31i bench agree; Mitsubishi exposes separate counters that include and exclude hold, and this address uses the excluding one; on Heidenhain our test environment did not count time stopped by NC stop or `M0`) - so a program left paused while nobody is at the machine does not register as operating time.

Read together with `/machine/powerOnDuration` it gives you the ingredients for a utilization figure - how much of the powered-on time was actually spent running. It accumulates, so measure an interval by reading twice and subtracting.

**Writing is not supported.** This is the machine's history; changing it makes production figures quietly wrong.

Fanuc reads parameters `6752` (minutes) + `6751` (milliseconds below the minute) in a single call - this is the `RUN TIME` on the control's production screen - and Mitsubishi uses `GetStartTime`. That is the **`Auto strt` item of the control's integrated-time screen (automatic start time: accumulated from the cycle-start button to a feed hold, block stop or reset)**; the `Auto oper` item (automatic operation time: from start to `M02`/`M30` or reset, `GetRunTime`), which includes hold, is deliberately not used (M800 Instruction Manual). **The control stops accumulating at `59999:59:59`** (the same cap, and the same API-documentation caveat, as `powerOnDuration`).

On Heidenhain it is `GetMachineRunningTime`, which the reference describes as the accumulated machining time since installation with a program running in Automatic or Single Block mode. It has **minute resolution**, so the value is always a multiple of 60. It was the same counter as "Program Run" under Machine times in the control's machine settings (`780` while that showed 00:13:31 in our test environment).

**Siemens answers status `-20`** (deemesh does not provide this value there).

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

**The pause conditions differ per control.** Fanuc accumulates, as its parameter manual defines it, **only time spent in cutting feed (`G01`, `G02`, `G03` and the like)**; on a 31i bench it kept accumulating while the feed override was `0` (still inside a cutting-feed block) and did not accumulate during a dwell or a feed hold. Siemens follows the rules of `$AC_CUTTING_TIME`: rapid traverse is excluded, the timer pauses during a dwell, and in the default configuration it counts **only while a tool is active** and not during dry run or program test (machine datum `27860`, bits 7, 4 and 5, can change that). For Siemens, we have not confirmed how it treats the stopped state or feed override `0`.

**The reset point differs by machine type.** Fanuc keeps accumulating; on Siemens, powering the control up with default values sets it to `0` (a normal power cycle does not). On both, an operator can reset it at the panel.

**Siemens can have this measurement switched off**: when bit 2 of machine datum `27860` is `0` the measurement itself is off and the value stays `0`.

**Writing is not supported.** This is the machine's history, and changing it would silently distort production reporting.

Fanuc sums parameters `6754` (minutes) and `6753` (milliseconds below a minute); Siemens uses `cuttingTime`. The two split parameters are read **in a single call**, so the value cannot be skewed by the minute rolling over between reads.

**Mitsubishi answers with status `-20`.** The time values deemesh reads on that control are the power-on and automatic-start times of the integrated time screen and the cycle time; cutting time is not among them.

## /machine/channel/mainProgramName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The name of the **main program selected in the HMI**. Returns `string`, read-only. It does not change even when a subprogram is entered during execution (that is the difference from `programName`).

Sources: `cnc_pdf_rdmain` on Fanuc, `/Channel/ProgramInfo/selectedWorkPProg` on Siemens, and `GetProgramNumber2` (the selected main) on Mitsubishi.

On Heidenhain it is the file name (with extension, without path) of the selected program from `GetExecutionPoint`. The TNC keeps a program per operating mode: in our test environment MDI mode reported the MDI program (`$mdi.h`; per the TNC7 User's Manual `$mdi_inch.h` in inch), and manual mode kept reporting the program chosen in program run.

**With no program selected it is an empty string.** We confirmed this in our test environment on Fanuc (with the panel showing `No Program`), Mitsubishi (with the panel's program number field blank) and Heidenhain, and on an 840D sl bench for Siemens (with the panel's program field blank; the control then points at its default program `MPF0`, which deemesh does not report). On Fanuc in our test environment, deleting the selected main on the panel made the control select another program in the same folder at once, and the value became empty only after the folder's last program was deleted. In MDI mode Fanuc keeps reporting the selected main (confirmed in our test environment).

## /machine/channel/mainProgramPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

The full path of the **main program selected on the HMI** (the path form of `mainProgramName`). It does not change when execution descends into a subprogram. Heidenhain keeps a selected program per operating mode (see `mainProgramName`), so MDI mode reports the path of the MDI program, and after the selected program is deleted or renamed this value still names the old path (confirmed in our test environment).

With no program selected it is an empty string (confirmed in our test environment on Fanuc and Mitsubishi; Heidenhain does the same; confirmed on an 840D sl bench for Siemens. For when Fanuc ends up with no program selected, see `mainProgramName`). In MDI mode Fanuc keeps reporting the selected main (confirmed in our test environment).

**Writing selects the program**: it makes the program at that path the channel's main program (the one to be executed). The value is a path string: `{"value": "//CNC_MEM/USER/O0001"}`.

- Path notation follows the machine: Fanuc `//CNC_MEM/USER/O0001` (data server: `//DATA_SV/...`), Siemens `//NC/Part programs/PART1.MPF` (the same notation `programPath` and `entryList` return; `Subprograms` and `Workpieces` likewise), Mitsubishi `//PRG/USER/O0001` (the notation below `ncMemoryRootPath`), Heidenhain `//TNC/nc_prog/PART1.H`. NC file-system paths are vendor-specific and are not normalized, for the same reason as `plcAddress`
- **It must be a file**: passing a folder path returns status `-18`, as does a path that does not exist
- Selecting does **not start machining** (cycle start remains the operator panel's / PLC's job)
- **The control decides in which channel state a selection is accepted.** A refusal for that reason answers status `-22` (machine state), and the reason names the state. Per control:
  - Fanuc: per the FOCAS2 specification the selection function can be used only in MEM (auto) and EDIT mode, so any path answers status `-22` in other modes such as MDI or JOG. While automatic operation is started (`executionStatus` value `3`) any path answers status `-22` as well, including the program that is running. The same holds while the emergency stop is on: any path answers status `-22`, and the reason says so (confirmed on a 31i bench and in our test environment). Whether a stopped state such as block stop or feed hold accepts a selection is up to the machine; when it does, the selected program changes at once, so writing in the reset state is the safe choice. A path that does not exist answers status `-18` instead. A program in O8000 to O8999 or O9000 to O9999 protected by parameter `3202#0`/`#4` is not selected and answers status `-22` (lift the protection or choose another program). An edit-disable attribute set on the panel does not prevent selection (confirmed in our test environment)
  - Siemens: the channel must be in the Reset state (a precondition of the `Select` method); otherwise status `-22`
  - Mitsubishi: while a program is running the operation search is refused, status `-22`
  - Heidenhain: accepted in the Program Run operating mode only. Manual operation and MDI mode answer status `-22`, and so does a running program. While a run is stopped on a block boundary (`executionStatus` `1`) it is accepted, and the unfinished run is dropped (confirmed in our test environment; write it after the run has ended or been cancelled if the machining is to continue). A folder path, a path under a folder that does not exist and a file that does not exist all answer status `-18` (confirmed in our test environment)
- Siemens calls the server's file-handling `Select` method, Fanuc uses `cnc_pdf_slctmain` for both CNC memory and data-server paths, Mitsubishi uses the operation search `Search`, and Heidenhain uses `SelectProgram`. On a Fanuc data-server path, when the selection is refused for a reason other than the machine's state, deemesh tries setting the storage-mode DNC operation file (`cnc_wrdsdncfile`) and answers success only when that file reads back as the one it set

## /machine/channel/programName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The name (file name) of the program **currently executing**. Returns `string`, read-only. When entering a subprogram, it changes to that subprogram's name (confirmed on our test bench: while a subprogram was executing the value was its file name, and it returned to the main program on exit). For the main selected in the HMI, see `mainProgramName`.

Sources: `cnc_exeprgname2` on Fanuc, `/Channel/ProgramInfo/workPandProgName` on Siemens, and `GetProgramNumber2` (the running program) on Mitsubishi.

The notation follows the control. Fanuc gives the O number (`O0003`); Siemens gives the **file name with its extension** (`PART1.MPF`, and `SUB1.SPF` once a subprogram is entered) without a path (use `programPath` for the path); Mitsubishi gives the program file name (M700- and M800-series controls return the file name here; `GetProgramNumber2` in the reference IB-1501209).

**With no program selected it is an empty string** (confirmed in our test environment on Fanuc and Mitsubishi; Heidenhain does the same; confirmed on an 840D sl bench for Siemens). On Fanuc, MDI mode gives `O0000` without a path (confirmed in our test environment).

On Heidenhain it is the file name (with extension) of the last program in the call list from `GetExecutionPoint`; entering a subprogram gives that subprogram's name (confirmed in our test environment). With a program selected but not yet started it is the selected main program's name (the program pointer is taken to be on the main program, as `programNestLevel` reading `1` says).

## /machine/channel/programPath
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The full path of the currently executing program (e.g. `//CNC_MEM/USER/PATH1/O0001`). `channel` filter. Siemens converts the NCK-internal path into the user notation (`//NC/...`) before returning it. Heidenhain likewise turns a path on the `TNC:` drive into `//TNC/...` (we have not confirmed a program running from another drive); inside a subprogram (`CALL PGM`) it is that subprogram's path, and before a run starts it is the selected main program's path (confirmed in our test environment).

With no program selected it is an empty string (confirmed in our test environment on Fanuc and Mitsubishi; Heidenhain does the same; confirmed on an 840D sl bench for Siemens). On Fanuc, MDI mode gives `O0000` without a path (confirmed in our test environment).

**On Mitsubishi the folder part is not guaranteed to belong to the program currently executing.** deemesh fetches the directory and the file name separately on that control, and **has not found a value that gives the directory of the program currently executing**. In practice it is usually right, because this control's NC memory has a fixed directory layout (you cannot create folders - see `directoryExists`), so a user's main program and its subprogram rarely sit in different places. While a fixed cycle (`//PRG/FIX`) or a machine tool builder macro (`//PRG/MMACRO`) is called and running, however, the folder part may come out as the main program's folder (we have not confirmed this). The file name part always belongs to the program currently executing.

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

**In a program written without N numbers this value cannot tell you where execution is.** As the table shows, a block with no N gives `0` or keeps an earlier N; it never counts blocks. Read the position from `/machine/channel/programBlockCounter`, and see that address's description too, since what it counts differs by machine type.

**On Mitsubishi the retention lasts only while the program runs.** Once it ends and the control is in reset, the value goes back to `0` (seen on the simulator: `N400` executed last, then `0` after `M30`). On Fanuc a value stays after the program ends and after a reset and matches the N shown on the operator panel, but it need not be the number of the last block (31i bench: `30` after ending with `N40 M30`, with the panel showing `N00030` too). On no machine type take it as the last executed N.

**Entering a subprogram gives you the subprogram's N** (the same moment `programName` switches to the subprogram's name). On return it goes back to the main program's N.

⚠️ **Fanuc's retention crosses file boundaries** (confirmed in our test environment): on an N-less block right after returning from a subprogram you see **the subprogram's last N**, and right after entering one you see **the main's N**. On Fanuc this value alone therefore cannot tell you which file the N belongs to; read `/machine/channel/programName` alongside it. On Siemens the value is not retained, so the situation does not arise. Mitsubishi directs every query at **the program currently running** (main or sub), so the value is always the N of the running program.

## /machine/channel/programBlockCounter
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The executed-block counter. `channel` filter. Returns `int`.

⚠️ **Each machine type counts something different.** The values are not comparable, so **do not use this as a progress figure across machine types** (all four confirmed in our test environments):

| | What it counts | Resets |
|---|---|---|
| Fanuc | blocks executed since the cycle started (`cnc_rdblkcount`, counting through subprogram blocks) | at each Cycle Start |
| Siemens | the **line number within the file currently executing** (`actLineNumber`, negatives clamped to `0`) | when the file changes (entering or leaving a subprogram) |
| Mitsubishi | how many blocks have passed **since the current `N` number** | at every `N` number |
| Heidenhain | the **block number within the file currently executing** (the number that opens each line of a conversational program; `BEGIN PGM` is `0`) | when the file changes (entering or leaving a `CALL PGM`) |

**On Mitsubishi it is one half of a pair with `programSequenceNumber`.** That control addresses a position inside a program with three values - program name, `N` number, and the block count from that `N` - and the operation search on the control takes the same three. So this number alone does not fix a position; read it together with `N` to have a position (seen on the simulator: `0`, `1`, `2`, `3` through the `N100` stretch, then `0` on reaching `N500`).

On Siemens the line number is **relative to the file currently executing**: entering a subprogram switches it to the subprogram's line numbers, and returning switches it back to the main's (confirmed in our test environment). The number alone therefore cannot tell you which file it counts; line 3 of the main and line 3 of the subprogram are both `3`. To pin down the file, read `/machine/channel/programName` and `/machine/channel/programNestLevel` alongside it. During the nesting transition (under a second) a sample can briefly combine mismatched values (the level updates before the name).

Heidenhain is also **relative to the file currently executing**, so read it the same way as on Siemens. While a subroutine in the same file (`CALL LBL`) runs, the block numbers are that file's. It gives a number only while a block is executing, and `0` otherwise: before a run starts and after it ends it is `0` (when a run ends, the control's cursor also goes back to the start of the program), and while stopped by `M0` or single block it is that block's number. In MDI it is `0` even while a block runs (in our test environment the program status from HEIDENHAIN DNC stayed 'no program selected' meanwhile, and we have not found another way to read MDI execution).

## /machine/channel/programLastBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The G-code text of the block executed just before. `channel` filter. When there is no preceding block (the program's first block, or not in operation) the value is an empty string.

**Supported on Siemens and Mitsubishi; Fanuc returns status `-20`.** The `cnc_rdexecprog` deemesh uses for block text on Fanuc returns the look-ahead buffer (blocks still to come), so a block already executed cannot be read through it.

## /machine/channel/programCurrentBlock
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The **G-code text** of the block currently executing. `channel` filter. When there is none the value is an **empty string**, never `null`.

**What you get "when nothing is executing" differs by machine type.** Observed in our test environments on controls holding a selected program in reset:

| | Siemens, Mitsubishi | Fanuc |
|---|---|---|
| `programCurrentBlock` | `""` | the **first line** of the program |
| `programNextBlock` | the first block | the line **after** that |

Heidenhain answers `""` like Siemens and Mitsubishi. It gives the block's text only while a program is running, stopped or interrupted (including while it is stopped by an error, that is while `executionStatus` is not `0`), and `""` before a run starts and after it ends. The text is the conversational program's line as it is, including the leading block number (for example `"9 CYCL DEF 9.0 DWELL TIME"`; confirmed in our test environment). `programNextBlock` answers status `-20` on Heidenhain.

Siemens and Mitsubishi can express "no block is executing" (on Mitsubishi the control reports the execution position as `0`, meaning not in operation). On Fanuc, the `cnc_rdexecprog` deemesh reads returns the look-ahead buffer, so its first line comes out as "current"; while stopped, that line is really the block that will run **next**.

**Comparing against the control makes this visible.** On the program screen, the execution marker (the highlight bar) sits **above** the first line while in reset, meaning no block has run yet, and the empty string is that state written down faithfully.

**So do not decide "what is executing right now" from this address alone.** Read `/machine/channel/executionStatus` alongside it and treat this value as meaningful only when that is `3` (Run).

**If you need more than one of them, ask for them in a single request.** `programLastBlock`, `programCurrentBlock`, `programNextBlock` and `programLookAhead` are served from **one query to the machine** when requested together, so they agree with each other. Read separately, the program advances in between and you get a combination where previous, current and next are not consecutive at all (seen in our test environment: read one at a time they came back as `G04 X8.`, `G04 X7.` and `G04 X5.`, a block skipped, while the same stretch read together never disagreed). It is faster too, but consistency is the reason.

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

Line endings are normalized to a single LF (`\n`) on every protocol; CR is stripped, so splitting on `\n` is safe.

## /machine/channel/programNestLevel
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The program-call nesting level (with `desc`): `0` = no program, `1` = main, `2`+ = subprogram (L1, L2, …). `channel` filter. **Supported on Siemens, Mitsubishi and Heidenhain** (Fanuc answers status `-20`).

**What it counts is the depth of the program pointer, not execution.** With a program loaded it reads `1` even when nothing is running (confirmed in our test environments: Siemens and Mitsubishi report `1` while in reset or interrupted). For "is it running right now", use `/machine/channel/executionStatus`.

On Mitsubishi the vendor value counts **how many subprograms deep** you are (main is `0`), one step off our scale, so deemesh shifts it. Telling `0` (no program) apart from `1` (main) costs this address one extra query to the machine.

Heidenhain counts a level for a call of another program, while `CALL LBL`, which calls a subroutine in the same file, does not add one (confirmed in our test environment with `CALL PGM`; calls through `CALL SELECTED PGM` or cycle 12 have not been confirmed). The TNC7 User's Manual ('Nesting of programming techniques') allows external programs to be nested 19 deep, so this value goes up to `20`. With a program selected it reads `1` before a run starts too.

## /machine/channel/variable/variableValue
```yaml
value_type: "float"
null_able: true
required_filters: ["channel", "variable"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

Reads/writes a **macro variable (Fanuc, Mitsubishi) / R parameter (Siemens)** (read + write). Put the variable number in the `variable` filter (e.g. `variable=100` → Fanuc and Mitsubishi `#100`, Siemens `R100`). Returns `float`; writes take `{"value": 3.14}`. **Reads** support range/comma expansion: `variable=100-105` is an array of 6 values. Writes always target a single variable (expansion syntax is rejected with status `-13`: the rule shared by every write). A **vacant macro variable on Fanuc or Mitsubishi reads as `null`**; this is the state the control's custom-macro screen shows as an empty cell (`DATA EMPTY` on Fanuc), and it is distinct from the value `0`. In a range expansion only that slot becomes `null` (e.g. `[3.14, null]`).

**Which numbers exist depends on the machine and its options.** A number that machine does not have comes back as **status `-18`** (on reads and writes alike). The only thing to fix is the `variable` value, and the control's variable screen tells you which numbers that machine actually has. On Fanuc and Mitsubishi, deemesh keeps no list and passes the number through, so the error string carries the vendor's own reason. **On Siemens, deemesh knows the number of R parameters.** It reads `numRParams` (machine data `28050`) per channel when it connects, and since **R numbering starts at `0`**, a number outside `0` to count-1 is rejected immediately with status `-18` without asking the machine, and the error string carries the allowed range and the count (e.g. `expected 0-99`). So checking a number with a read before writing works on Siemens too. If a missing number is mixed into a range expansion, the **whole request fails with status `-15`** (no partial array is returned). Distinguish this from vacant variables, which are not errors but `null` elements and do not break the expansion. A number outside the syntactic range (`0`-`89999` on Fanuc) is the same status `-18`, except that one is rejected immediately without asking the machine. On Fanuc, `#10000` and up are P-code macro variables, which exist only on a machine with the Macro Executor option. Without the option such a number is the same status `-18` with the option named in the error string, and other numbers in the same batch (`/read/batch`, `deemesh_read_batch`) are still read (confirmed in our test environment; a range or comma expansion fails as a whole with status `-15`, as above). Reading and writing P-code macro variable values on a machine with the option is not yet confirmed.

**On Mitsubishi, a write the control refuses because of a protection answers status `-22` (machine state).** That covers a number inside the common variable setting protection ranges set by parameters `#12111`–`#12114`, and any write while data protect key 2 (PLC signal `*KEY2`, `Y709`) is off (confirmed on the simulator). The number does exist, so remove the protection on the operator panel and write again. Reads are not blocked.

**Writing a variable back to vacant is not supported**: the value is a single number, and clearing a variable that has one requires the operator panel.

**Writing an R parameter on Siemens has no channel-state gate.** Alarm `4230` (data alteration from external not possible in current channel state), which blocks setting-data writes, does not apply to R parameters, and we have confirmed on our test bench that writes are accepted both while the channel is interrupted by an emergency stop and while automatic operation is running (`executionStatus` Run). In the same test, GUD (`userDataValue`), PLC (`plcValue`) and tool offset writes (`toolEdge/toolLengthWear` and the like, including the active edge of the active tool) were accepted during automatic operation as well.

## /machine/channel/userData/userDataValue
```yaml
value_type: "object"
null_able: false
required_filters: ["channel", "userData"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Reads/writes Siemens **channel GUD (per-channel SGUD)** user variables (**OPC-UA (Siemens) only**). Both the `channel` and `userData` filters are required, and only variables of the channel named by `channel` are addressed; NC-wide shared variables are not reachable through this address.

The return type is `object`: GUD variables differ in type, so the value arrives in a **self-describing envelope** `{"type":..,"data":..}` that also tells you what type it is.

**userData**: the `SGUD:<name>` or `SGUD:<name>[<index>]` format. **Indices are exactly as shown on the machine screen (HMI)**, 0-based (converted to the internal numbering automatically).

- no index → a scalar variable
- `[i]` → one element of a 1D array
- `[i-j]` → the i..j range of a 1D array (both ends inclusive)
- `[r,c](column count)` → one (row, column) element of a 2D array. Write the brackets exactly as shown on screen and append only the array's **column count** in parentheses (the column count cannot be determined from the read alone, so supply it)
- Example: `?channel=1&userData=SGUD:_SC_C97[0,1](4)` (channel 1, screen notation [0,1] of a 4-column 2D)

**type**: `BOOL` (true/false) · `CHAR` (character code 0-255) · `INT` (integer) · `REAL` (64-bit float) · `STRING`. The structured `AXIS`/`FRAME` types are not supported (writing with such a type is rejected with status `-16`).

**data**: one value for a single element (scalar / `[i]` / `[r,c](column count)`), a JSON array for an `[i-j]` range.

- single: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- range: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**Writing**: put the **same object** as the read returns into `value` (e.g. `{"value":{"type":"REAL","data":42.0}}`). `type` decides the type written, so no read is needed first. The number of `data` elements must match the range size exactly (1 for a single element).

**Note**: on older NCKs without a GUD area the address answers status `-20` (not supported).

**Fanuc and Mitsubishi answer with status `-20`.** GUD is Siemens' own user-variable system; on those two controls user variables (macro and common variables) are read and written through `/machine/channel/variable/variableValue`.

## /machine/userData/userDataValue
```yaml
value_type: "object"
null_able: false
required_filters: ["userData"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

Reads/writes Siemens **global GUD (SGUD)** user variables (**OPC-UA (Siemens) only**, NC-wide shared variables). Both read and write are supported. Since GUD variables have different types, the return type is `object`; it comes as a **self-describing envelope** `{"type":..,"data":..}` that also tells you what type the value is. The only filter is `userData`; this address covers **NC-wide shared** variables only, so no channel is specified.

**userData**: the format is `SGUD:<name>` or `SGUD:<name>[<index>]`. The prefix is the GUD definition block name (only `SGUD` is supported; other blocks such as `MGUD` are not). **All indices are exactly as shown on the machine screen (HMI)**: enter the number you see on screen as-is (0-based, automatically converted to the OPC-UA internal number).

- no index → scalar variable
- `[i]` → a single i-th element of a 1D array (if the screen shows `_ARR[3]`, use `[3]`)
- `[i-j]` → the i–j range of a 1D array (both ends inclusive, the same convention as `1-3` in filter expansion)
- `[r,c](column count)` → a single (row, column) element of a 2D array. **Write the brackets as shown on screen** and append only the array's column count in parentheses; if the screen shows columns up to `[0,3]`, use `(4)` (the column count cannot be determined from the read alone, so enter it as well)
- Examples: `SGUD:MYVAR` (scalar), `SGUD:_SC_NCK_ROU_S[1]` (screen notation [1] of a 1D), `SGUD:POS[0-2]` (the 3 elements [0]–[2] of a 1D), `SGUD:_SC_C97[0,1](4)` (screen notation [0,1] of a 4-column 2D)

**type** (inside the envelope) is the element's actual type:

- `BOOL`: true/false
- `CHAR`: character code (0–255 integer)
- `INT`: integer
- `REAL`: real number (the same 64-bit real as Siemens R parameters)
- `STRING`: string

(Structured GUD `AXIS` / `FRAME` are not supported; writing with such a type is rejected with status `-16`)

**data** (inside the envelope): if one element (scalar / `[i]` / `[r,c](column count)`), a single value; if an `[i-j]` range, a JSON array:

- scalar/single: `{"status":0,"value":{"type":"REAL","data":3.14}}`
- range: `{"status":0,"value":{"type":"INT","data":[1,2,3]}}`

**Write**: put the **same object** as the read into `value` (e.g. `{"value":{"type":"REAL","data":42.0}}` → changes only the single cell at screen notation [i]). Since `type` decides the type to write, you write directly without reading first. The number of `data` elements must exactly match the range size (1 if single).

**Note**: on older NCKs without a GUD area the address answers status `-20` (not supported).

**Fanuc and Mitsubishi answer with status `-20`.** GUD is Siemens' own user-variable system; on those two controls user variables (macro and common variables) are read and written through `/machine/channel/variable/variableValue`.

## /machine/plcAddress/plcType/plcValue
```yaml
value_type: "float"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
filter_codes: {"plcType": [{"value": 0, "name": "auto", "protocols": ["nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": 1, "name": "bit", "protocols": ["nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]}, {"value": 2, "name": "uint8"}, {"value": 3, "name": "int16"}, {"value": 4, "name": "int32"}, {"value": 5, "name": "float32", "protocols": ["nc_focas2_fanuc", "nc_opcua_siemens"]}, {"value": 6, "name": "float64", "protocols": ["nc_focas2_fanuc", "nc_dnc_heidenhain"]}, {"value": 7, "name": "int8"}, {"value": 8, "name": "uint16"}, {"value": 9, "name": "uint32"}]}
```

Reads/writes a **single element of PMC/PLC memory** (Fanuc FOCAS2 `pmc_rdpmcrng`/`pmc_wrpmcrng`, Siemens OPC-UA `/Plc/` node, Mitsubishi EZSocket `ReadDevice`/`WriteDevice`, HEIDENHAIN DNC PLC data). Both read and write are supported; the return type is `float` (one value), and writes are also a single number (e.g. `{"value": 42}`). This address handles **a single element only**; to work with several elements at once, use the list-form address in the same tree. Both the `plcAddress` and `plcType` filters are required.

**plcAddress**: the address format **differs by machine**. Unlike `plcType` this is a **deliberate exception that is not normalized**: Fanuc's `D100` and Siemens's `DB10.DBB56` point into different memory architectures, and the table that maps one to the other is not SDK knowledge but site configuration that depends on how that machine's ladder was written. It is not unified across machine types either, so if you need to read the same signal across machines, **the host application must keep a per-machine address table**.

- **Fanuc**: the first character is the PMC area, the rest is the byte number (e.g. `R5`, `D100`). Specify a range with `~`, but for this (single) address the range must be **exactly one `plcType` size** (e.g. for word, `D100~D101`)
- **Fanuc** PMC area first characters: `G` `F` `Y` `X` `A` `R` `T` `K` `C` `D` `M` `N` `E` `Z`. The range must be within the same area (`D100~D101` OK, `D100~R101` NG)
- **On Fanuc this address does not take bit addresses (`byte.bit` notation such as `R38.7`).** Read the byte and take the bit out of it: the value of `plcAddress=R38&plcType=2` is the same byte the panel's PMC signal screen shows in its `HEX` column (`0xDC` reads `220`), and bit 7 is `(220 >> 7) & 1`. Writes are byte-wise as well. Reason: FOCAS's PMC read/write functions have no bit type, so a bit write would have to read the byte, change it and write it back, overwriting any other bit the ladder changed in between. Signals are named `byte.bit` (e.g. `R0039.4`), but give the address as the byte
- **Siemens**: write it **exactly as it appears on the operator panel's `NC/PLC variables` screen**. The value is passed to the `/Plc/{address}` node. If the subscript is omitted, `[1]` is attached automatically. This address takes a single element only. For multiple elements it errors and points you to the list-form address
- **Siemens** forms (**the address carries its own offset**): `DB<n>.DBB<offset>` (byte) · `DB<n>.DBW<offset>` (word) · `DB<n>.DBX<byte>.<bit>` (bit) · `IW<n>` · `MB<n>` · `Q<byte>.<bit>`. Notation examples: `DB10.DBB56` · `DB31.DBX24.1` · `IW0` · `Q0.2`
- **Siemens** the subscript `[N]` is a **count, not an index**. `DB10.DBB56[4]` means **4 consecutive elements** starting at offset 56 (56·57·58·59), not "the 4th of 56". To reach a different location, **move the address**, not the subscript (`DB10.DBB61`)
- **Siemens** syntax caution (machine-independent): a form with no offset (`MB` alone · `DB<n>` alone) is not valid syntax, and a bit is addressed with `DBX`, not `DBB`
- **Siemens**: **which blocks and bytes actually exist is a property of that machine's ladder** and differs from machine to machine. The notation examples above show the shape only. They are not addresses every machine has. Check on that same panel screen: if a value shows there, it reads here
- **Siemens** 828D limit: 828D can only reach **customer data blocks from `DB9000` upward** (840D sl has no such limit)
- **Mitsubishi**: `<device><number>` exactly as the control's PLC screen shows it (e.g. `R100`, `M50`, `Y8A0`). A point count is added as an **`[N]` subscript** and, as on Siemens, `[N]` is **"how many", not "which one"** - `R100[4]` is 4 consecutive points starting at `R100`. This (single) address takes no subscript, or `[1]`
- **Mitsubishi**: the number base differs by device family - `M`, `R` and `D` are decimal while `X`, `Y` and `B` are **hexadecimal** (the same as on the control's screen)
- **Mitsubishi** alignment: devices numbered **by bit**, such as `M`, `X` and `Y`, need the head number on an **8-point boundary** for byte, word and dword access (`Y890` yes, `Y894` no). Word-addressed devices such as `R` and `D` have no such constraint. A misaligned address returns status `-18`; deemesh does not decide this from a table of its own but **asks the machine**, so any address that machine accepts goes through
- **Heidenhain**: takes two notations, **PLC memory** as `<kind><number>` (e.g. `M10`, `B10`, `W100`, `D8`) and **global PLC symbol names** (e.g. `ApiAxis[0].NN_AxDriveOn`). A value shaped like memory notation is read as memory, anything else as a global symbol name. The kind letters are case-insensitive
- **Heidenhain** memory kinds are `M`, `I`, `O`, `T`, `C`, `B`, `W`, `D`, `R`, `S`, `IB`, `IW`, `ID`, `OB`, `OW` and `OD`. The number is a **byte address**, so it steps by the width of a cell: `W` (2 bytes a cell) goes `W0`, `W2`, `W4`, `D` (4 bytes) goes `D0`, `D4`, and `R` (8 bytes) goes `R0`, `R8`. A number that is not the first byte of a cell (`W1`) is status `-18` (confirmed in our test environment, the TNC7 programming station)
- **Heidenhain** the count subscript `[N]` goes on memory notation only and, as on Siemens and Mitsubishi, is **"how many", not "which one"** (`W100[4]` is `W100`, `W102`, `W104`, `W106`). This (single) address takes no subscript, or `[1]`. Brackets inside a symbol name (`ApiAxis[0].NN_AxDriveOn`) are part of the name, not a count
- **Heidenhain** symbol names are set by the machine's PLC program. deemesh cannot read the symbol list, so you **need to know the name** to read it. Ask the machine manufacturer for the names
- **Heidenhain**: reading and writing need `access_password` in the connection settings. When it is missing or the control rejects it, the status is `-20` and the reason says which. **Write caution**: the TNC7 User's Manual says PLC access is protected by a password because changes to the PLC can make the control inoperable, and advises writing only after consulting Heidenhain or the machine manufacturer

**plcType**: a numeric code that decides which type one PLC cell is read and written as. It is a **machine-independent unified value**: `1`–`9` are the same promise on every machine, down to the width and the sign, so the same bits give the same value:

- `1` = bit: 1 bit (0 / 1)
- `2` = uint8: 8-bit integer, unsigned (0–255) · address width 1 (e.g. `D100`)
- `7` = int8: 8-bit integer, signed (-128–127) · address width 1
- `3` = int16: 16-bit integer, signed · address width 2 (e.g. `D100~D101`)
- `8` = uint16: 16-bit integer, unsigned (0–65535) · address width 2
- `4` = int32: 32-bit integer, signed · address width 4 (e.g. `D100~D103`)
- `9` = uint32: 32-bit integer, unsigned · address width 4
- `5` = float32: 32-bit real · address width 4 (e.g. `D100~D103`)
- `6` = float64: 64-bit real · address width 8 (e.g. `D100~D107`)

The same byte `0xA0` reads `160` with `2` and `-96` with `7`. A write to an integer type (`1`–`4`, `7`–`9`) takes only a whole number inside that type's range; a value outside it, or one with a fractional part, answers status `-16` (it is neither cut down nor rounded). The floating point types (`5`, `6`) take fractions, but the value must be finite and, for `5`, within the float32 range (otherwise `-16`).

`0` = **auto**: the type the machine itself gives that cell. **`0` is the only code whose value may differ by machine** (Siemens unsigned, Heidenhain signed, below). Machines that address raw memory, like Fanuc and Mitsubishi, have no intrinsic type, so `0` answers status `-18` there and the type must be given.

Where the width of a cell comes from differs by machine. **On Fanuc and Mitsubishi `plcType` sets the width; on Siemens and Heidenhain the address does.** Where the address sets the width, `1`–`9` must match that cell's width; a mismatch answers status `-18`, and the error string names the cell's width and the codes it takes.

**Important (Fanuc)**: the byte count of the `plcAddress` range must match the `plcType` size (e.g. `plcType=3` (int16, 2 bytes) with a single `D100` address fails → specify `D100~D101`). PMC reads go in byte units, so `1` (bit) is not taken either. Specify one of `2`–`9`. The result is returned as `float` (a JSON number).

**Siemens** has the width in the address (`DBX`, `Qx.y`, `Ix.y`, `Mx.y` are bits; `DBB`, `QB`, `IB`, `MB` bytes; `DBW`, `QW`, `IW`, `MW` words; `DBD`, `QD`, `ID`, `MD` double words). A bit address takes `1`, a byte address `2`/`7`, a word address `3`/`8`, and a double word address `4`/`9`/`5`. `0` (auto) is the server's base type, so **bytes, words and double words are all unsigned integers**; read signed values with `7`, `3` or `4`. **Read and write a floating-point value (REAL) with `plcType=5`.** It works on double word addresses only and corresponds to the `F` format of the operator panel's `NC/PLC variables` screen (reading and writing confirmed on our 840D sl bench). `plcType=6` (float64) answers status `-18` because this PLC access has no 8-byte floating point. A location with no fixed width, such as a timer or counter, takes `0` only. Writes read the node first to confirm the server type, then write with the same type.

**Mitsubishi** takes `1` (bit), `2`/`7` (byte), `3`/`8` (word) and `4`/`9` (dword). `0` (auto) is out for the same reason as on Fanuc (raw memory has no intrinsic type), and `5`/`6` (floating point) because this control's PLC device API carries integers only. All three return status `-18`, and the address itself still works. **Writes do not take a single byte point (`2`/`7`)** (status `-18`). On this control the only call that writes bytes is the block write, which covers 2 points or more and would change the neighbouring device as well, so deemesh refuses a byte write to this single-point address. Write it as a word (`3`/`8`), or use the list address `/machine/plcAddress/plcType/plcValueList` with 2 points or more. Reading as a byte works.

**Heidenhain** takes the width of a memory cell from its kind letter (`M`, `I`, `O`, `T`, `C` bits; `B`, `IB`, `OB` bytes; `W`, `IW`, `OW` words; `D`, `ID`, `OD` double words; `R` 8-byte reals), and the width of a symbol from the value range the control reports. A bit cell takes `1`, a byte cell `2`/`7`, a word cell `3`/`8`, a double word cell `4`/`9`, and an `R` cell `6`. `5` (float32) answers status `-18` because this control has no 4-byte floating point cells. `0` (auto) is what the control gives, so **integers are signed** (in our test environment they matched the decimal view of the operator panel's PLC table). A bit cell comes as `0`/`1`. Writes use the same type. Writing with `0` (auto) takes the signed range on an integer cell (`-128`–`127` on a byte cell), so write `128`–`255` with `2`. Writes were confirmed in our test environment on memory cells (`M`, `B`, `W`, `D`, `R`); writing to a symbol name has not been confirmed yet.

**Text locations are read with `/machine/plcAddress/plcText`.** On Heidenhain, when the value at that location is text, this address answers status `-25`. Read `/machine/plcAddress/plcText` with the same `plcAddress` (it does not use `plcType`). Heidenhain's `S` memory and text symbols are such locations (confirmed in our test environment). On Siemens this address reads the same byte as a number, and `plcText` reads a text variable from its byte address. Fanuc and Mitsubishi never give this answer, and neither does Siemens for an address written in the operator panel's notation. In a request that reads several locations at once with a range or commas (`plcAddress=W0,S0`), one failed location fails the whole request with status `-15`, so this answer too is left only in the error text; read such a location on its own. Text cannot be written: writing this address to a text location answers status `-18` (Siemens, Heidenhain), and `plcText` only reads.

**A written value may not stay.** The write is accepted and the status is `0`, but on some machines the value at that address keeps being refilled by the machine itself. Whether an address behaves that way depends on how that machine is configured, so read the value back after writing to see whether it stayed.

**Error codes**: an address that **does not exist on the machine** also returns status `-18` (invalid filter value); which blocks and bytes exist depends on that machine's ladder, so check the same screen on the operator panel first. On Siemens, when the server answers access denied (`BadUserAccessDenied`), deemesh answers status `-17` naming the rights to give (`PlcRead`, `PlcReadDB<n>` or `SinuReadAll`). On an 840D sl bench an account without PLC read rights got existing addresses reported as unknown, so when the server reports the address as unknown, deemesh checks the account's rights, read when it connects: without PLC read rights it answers the same status `-17` with the rights; if the rights are unknown, status `-18` with a note that missing rights look the same; with the rights present, status `-18` (the address is not on this machine). A `plcType` the machine cannot use returns status `-18` too. A value outside the spec (other than `0`~`9`) returns the same status `-18`, and both call for the same fix: pick another `plcType`. It is not status `-20` because **the address itself works on that machine**. status `-20` is reserved for "this address cannot be used on this machine". The error string carries the accepted values. On Heidenhain a cell or symbol that is not on the machine is status `-18`, and when `access_password` is missing or rejected the address itself cannot be used on that connection, so the status is `-20`.

## /machine/plcAddress/plcType/plcValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["plcAddress", "plcType"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
filter_codes: {"plcType": [{"value": 0, "name": "auto", "protocols": ["nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": 1, "name": "bit", "protocols": ["nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]}, {"value": 2, "name": "uint8"}, {"value": 3, "name": "int16"}, {"value": 4, "name": "int32"}, {"value": 5, "name": "float32", "protocols": ["nc_focas2_fanuc", "nc_opcua_siemens"]}, {"value": 6, "name": "float64", "protocols": ["nc_focas2_fanuc", "nc_dnc_heidenhain"]}, {"value": 7, "name": "int8"}, {"value": 8, "name": "uint16"}, {"value": 9, "name": "uint32"}]}
```

Reads/writes a **block of PMC/PLC memory elements** as an array. The filters, address format, and `plcType` rules are the same as `plcValue` (single) above; the only difference is that it handles **multiple elements**. The return type is `floatArray`, and the write `value` is a number array `[1, 2, ...]`. Even a single element must be written as an array like `[42]`. As with `plcValue`, some machines are configured so that a written value does not stay, so read it back after writing.

- **Fanuc**: the range's byte count must be a **multiple** of the `plcType` size, and the element count = byte count ÷ type size (e.g. `D100~D107` + word = 4 → `[v1,v2,v3,v4]`)
- **Siemens**: multi-element subscripts are allowed. `[N]` is a **count**. `DB10.DBB56[4]` returns **4 consecutive elements** starting at offset 56 as an array. The elements the server gives become the array as-is
- **Siemens**: omitting the subscript or giving `[1]` still returns **an array** (`[131.0]`). This address always returns `floatArray`, so a single element does not change the shape. Use it whenever the count varies or is not known in advance, and your parsing code never has to branch
- **Mitsubishi**: `[N]` is a count (`R100[4]` -> an array of 4). The maximum number of points in one read depends on the type - bit and byte `1280`, word `640`, dword `320`. Beyond that is status `-18`
- **Mitsubishi**: a byte (`plcType` `2`/`7`) **cannot write a single point** (status `-18`). This control's single-device write call has no byte type, and its block write takes 2 points or more, which would change the neighbouring device as well. Use a word (`3`/`8`), or this list address with 2 points or more. **Reading a single point is fine**
- **Heidenhain**: the `[N]` of memory notation is the count (`W100[4]` -> an array of 4: `W100`, `W102`, `W104`, `W106`, stepping by the width of a cell). A symbol name always gives a one-element array. A write is sent cell by cell, but every value and cell is checked first, so one value out of range or one missing cell writes nothing
- **Heidenhain**: when a location holds text, the answer is status `-25`. Read such a location with `/machine/plcAddress/plcText`. That address reads one element at a time, so for several locations list `plcAddress` with commas. When a request to this address lists `plcAddress` with commas, however, one failed location fails the whole request with status `-15` (this answer is left only in the error text)
- Writes require the **element count to exactly match the target range/node's element count**

**Error codes**: an address that **does not exist on the machine** also returns status `-18` (invalid filter value); which blocks and bytes exist depends on that machine's ladder, so check the same screen on the operator panel first. On Siemens, when the server answers access denied (`BadUserAccessDenied`), deemesh answers status `-17` naming the rights to give (`PlcRead`, `PlcReadDB<n>` or `SinuReadAll`). On an 840D sl bench an account without PLC read rights got existing addresses reported as unknown, so when the server reports the address as unknown, deemesh checks the account's rights, read when it connects: without PLC read rights it answers the same status `-17` with the rights; if the rights are unknown, status `-18` with a note that missing rights look the same; with the rights present, status `-18` (the address is not on this machine). A `plcType` the machine cannot use returns status `-18` too. A value outside the spec (other than `0`~`9`) returns the same status `-18`, and both call for the same fix: pick another `plcType`. It is not status `-20` because **the address itself works on that machine**. status `-20` is reserved for "this address cannot be used on this machine". The error string carries the accepted values. On Heidenhain a cell or symbol that is not on the machine is status `-18`, and when `access_password` is missing or rejected the address itself cannot be used on that connection, so the status is `-20`.

The length is set by the `[N]` in the address, so `[]` never appears.

## /machine/plcAddress/plcText
```yaml
value_type: "string"
null_able: false
required_filters: ["plcAddress"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

Reads **a single PLC data location whose value is text** (Siemens OPC-UA `/Plc/` node, HEIDENHAIN DNC PLC data). The return type is `string`, and there is no write. The only filter is `plcAddress`, written the same way as for `/machine/plcAddress/plcType/plcValue`. Text has only one way to be read, so this address does not use `plcType`.

- **Reading a location that holds a number** answers status `-25`. Such a location is read by `/machine/plcAddress/plcType/plcValue`. That address also needs `plcType`, so add `plcType=0` (auto) and read it with the same `plcAddress`
- For several locations, list `plcAddress` with commas (the result is an array of strings)
- A location with no content reads as the empty string `""`
- **Heidenhain**: reads `S` memory and text symbols. Each location is one string, so a count subscript (`S0[2]`) is status `-18`. In our test environment (the TNC7 programming station) `S0` answered `""` and the basic PLC program's time symbol answered `"01:01:54"`. Symbol names are set by the machine's PLC program. Reading needs `access_password` in the connection settings; when it is missing or the control rejects it, the status is `-20`
- **Siemens**: returns **the same characters the operator panel's `NC/PLC variables` screen shows in the `A` format**. It joins the bytes from a byte address (`DBB`, `MB`, `IB` or `QB`) for the count in the subscript `[N]` (one byte without a subscript) into text, stops at a `0` byte as the screen does, and reads bytes of 128 and above as Latin characters (`Ä`, `ß` and so on), as the screen does. If the screen shows `HELLO` for `MB302[7]` in the `A` format, `plcAddress=MB302[7]` reads `"HELLO"` too (checked side by side with the operator panel on our 840D sl bench: `HELLO`, `HE` for text with a `0` in the middle, and `AÄßB`). A text variable (STRING) of a PLC program holds its maximum length and actual length in its first two bytes, so its characters start two bytes after the variable's address; give its maximum length as the count. Like the screen, this does not look at the actual-length byte, so if the PLC shortens a string without clearing the rest, leftover characters can show. A word, double word or bit address is not text and answers status `-25`, pointing to `plcValue`. Answers about access rights and missing addresses are the same as for `plcValue`
- The **Fanuc and Mitsubishi** adapters do not have this address (status `-20`). On those two, read PLC addresses as numbers with `plcValue`

**Error codes**: a location that is not on the machine is status `-18`.

## /machine/channel/parameter/index/parameterValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "parameter", "index"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The value of **one row** of a CNC parameter (**Fanuc and Mitsubishi**, `float`, read/write). Specify it with `channel=<channel>&parameter=<number>&index=<n>`, where the number is taken straight from that control's parameter manual. It is **not translated and not unified across machine types** (a vendor-owned numbering scheme, the same class as the `diagnosis` filter, so no cross-vendor mapping can exist). Parameters can differ per path (channel), so this address is **channel-scoped**. Frequently used parameters are exposed as named addresses such as `partCountActual` and `powerOnDuration`; this address is the general-purpose corridor for everything else.

**Every parameter is treated as an array of rows**. `index` is the row number: the axis number for axis-type parameters, the spindle number for spindle-type ones (same as the row order on the parameter screen, 1-based; e.g. X1=1, Y1=2), and **single-value parameters have exactly one row, so use `index=1`** (the same model as `parameterValueList`, which returns single values as one-element arrays). An out-of-range index is refused with status `-18` carrying the actual row count.

- Bit-type parameters travel as the **whole byte** (a packed integer). Decomposing/composing bits is the caller's job. To change a single bit, do a read-modify-write (which can race with concurrent changes from the operator panel).
- Decimal (real) parameters travel as real numbers with the machine's decimal places applied, and writes are stored with the same number of places. On Mitsubishi the setting unit `#1003` decides those places for length parameters; finer digits are rounded by the control, which then answers status `0` (confirmed in our test environment with `#8205`). Read the value back to see what was stored.
- **Mitsubishi has parameters that are not numeric** (for example the axis name `#1013` reads `X`). This address is `float` and cannot represent them, so it refuses with status `-18` and carries the string that was actually read. `index` is the axis number for axis parameters and only `1` otherwise.
- On Fanuc, writing an out-of-range value to an integer parameter is refused with status `-16` (the allowed range is included in the error). On Mitsubishi a value the control rejects answers status `-16` without the allowed range, so check the setting range in the parameter manual.
- **Write caution**: parameters change machine behavior. A write is refused with status `-22` (machine state) on Fanuc when the machine blocks parameter writes, and on Mitsubishi while the part system is in automatic operation (including a pause) or while data protect key 2 (PLC signal `*KEY2`, `Y709`, which protects user parameters) is off (reset the part system or turn the key on, then write again; deemesh tells the key case apart by reading that signal after the refusal). Some parameters require a power cycle after the change. If the Fanuc library does not include the parameter write function (`cnc_wrparam`), writes return status `-20` (which functions a Fanuc library includes depends on the package).

**Siemens answers with status `-20`.** That control uses machine data addressed by name rather than a numbered parameter system, and deemesh does not open that channel through this address.

## /machine/channel/parameter/parameterValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["channel", "parameter"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

The **all-axes value array** of a CNC parameter (**Fanuc and Mitsubishi**, `floatArray`, read-only). The array length is the **row count** established by vendor validation: axis-type parameters get one element per axis, spindle-type per spindle, and non-axis parameters a one-element array (the same row model as `diagnosisValueList` and the sibling `parameterValue`). Whether a parameter is row-indexed, and how many rows it has, is determined on the first query with vendor validation and cached per channel, so repeated polling is light, and range/comma expansion such as `parameter=6711-6713` returns nested arrays with per-parameter boundaries preserved.

**Siemens answers with status `-20`.** That control uses machine data addressed by name rather than a numbered parameter system, and deemesh does not open that channel through this address.

Every parameter has at least one row, so `[]` never appears.

## /machine/channel/diagnosis/index/diagnosisValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "diagnosis", "index"]
read: ["nc_focas2_fanuc"]
write: []
```

The value of **one row** of diagnosis data (**Fanuc only**, `float`, read-only). `channel=<channel>&diagnosis=<number>&index=<n>`, the same row model as the parameter corridor: for axis/spindle-dependent diagnoses `index` is the axis/spindle number, and **single-value diagnoses have exactly one row, so use `index=1`** (the same model as `diagnosisValueList`, which returns single values as one-element arrays). An out-of-range index is refused with status `-18` carrying the actual row count. When polling just one axis periodically, this is lighter than reading every row via the List (one call).

**Siemens and Mitsubishi answer with status `-20`.** The Fanuc diagnosis numbering belongs to that control and has no counterpart. Mitsubishi NC internal data is read through `/machine/channel/diagnosisSection/diagnosisSubsection/index/diagnosisValue`.

## /machine/channel/diagnosis/diagnosisValueList
```yaml
value_type: "floatArray"
null_able: false
required_filters: ["channel", "diagnosis"]
read: ["nc_focas2_fanuc"]
write: []
```

The **value** of an arbitrary diagnosis number (**Fanuc only**, `floatArray`). Axis/spindle-dependent diagnoses return an array of axis-count length; non-dependent diagnoses return a single-element array. Diagnosis data can differ per path (channel), so this address is **channel-scoped**: specify the path with `channel=`; path-common numbers read the same on any channel. Each diagnosis's format (whether it is row-indexed, and how many rows) is determined on the first query with vendor validation and cached per channel, so repeated polling is light. With comma/range expansion like `diagnosis=301,308`, it comes as a nested array with per-diagnosis boundaries preserved.

**Siemens and Mitsubishi answer with status `-20`.** The Fanuc diagnosis numbering belongs to that control and has no counterpart. Mitsubishi NC internal data is read through `/machine/channel/diagnosisSection/diagnosisSubsection/index/diagnosisValue`.

Every diagnosis has at least one row, so `[]` never appears.

## /machine/channel/diagnosisSection/diagnosisSubsection/index/diagnosisValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "diagnosisSection", "diagnosisSubsection", "index"]
read: ["nc_ezsocket_mitsubishi"]
write: []
```

The value of **one row** of the NC's internal data (**Mitsubishi only**, `float`, read-only). Address it with a **section number, sub-section number and axis number**: `channel=<channel>&diagnosisSection=<section>&diagnosisSubsection=<subsection>&index=<n>`.

**This corridor is for when you already know the numbers.** The numbering is vendor-owned, so it is **not translated and not unified across machine types** (the same class as `plcAddress` and `diagnosis`). Fanuc's `diagnosis` takes a single number while this takes two, which is why it is a separate address. The two machines' diagnosis data cannot share one address.

- Frequently used values are exposed as **named addresses** such as `axisLoad` and `spindleLoad`. Prefer those where they exist. This address is the general-purpose corridor for everything else
- `index` is the **row number** (1-based). Depending on the section it is an axis number or a spindle number, and data with no rows has exactly one row, so use `index=1`. An out-of-range index is refused with status `-18` carrying the actual row count
- **Some data is not numeric.** This corridor carries decimal, hexadecimal, real and string data, but this address is `float`, so strings are refused with status `-18` carrying the value that was actually read (hexadecimal display data is an integer underneath and arrives as-is)
- An unknown section or sub-section returns status `-18`

**Fanuc and Siemens answer with status `-20`.** This section and subsection numbering belongs to Mitsubishi.

## /machine/ncMemorySizeTotal
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The total NC memory capacity. Returns `int` + `unit:"bytes"`, read-only.

What it refers to is the **machining-program memory**. On Fanuc it is the drive capacity from `cnc_rdpdf_inf` (in bytes, despite the field names); on Siemens the user area of the NC's **passive file system** (where main and subprograms, workpieces and GUD definition files live; Extended Functions manual, S7), `/Nck/State/usedMemDramUPassF` and `freeMemDramUPassF`; on Mitsubishi the character counts from `GetInformation` (the same capacity and remaining-space figures shown on the control's edit screen). One of the three is calculated from the other two (Fanuc: free = total - used; Siemens and Mitsubishi: total = used + free; Heidenhain: used = total - free), so the three read together in one request satisfy `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree`. On Siemens, when the value cannot be read the answer is status `-17`, not `0` (the total needs both the used and the free value).

**On Mitsubishi the value is that of the main root (`//PRG`, `Memory` on the operator panel).** The second program memory of M800V/M80V controls (`//PRG2`, `Memory2` on the panel) counts its capacity separately and is not included (confirmed in our test environment).

**On Heidenhain the value is that of `//TNC` (the `TNC:` drive on the control).** It is the total and free bytes from HEIDENHAIN DNC's `GetDiskSpace`, and the used space is their difference. In our test environment the used space shown by the control's file manager was larger than `ncMemorySizeUsed` by about 5% of the total capacity (the same difference at two comparisons with files added in between; the total was the same). The control's figure was recalculated only when the control restarted, so adding or deleting files in the meantime did not change it.

All sizes in the SDK are in **bytes**, the same unit as `sizeBytes` in `entry`/`entryList`, so a question like "does this file fit in the free space" needs no conversion. When a machine only reports a coarser unit, this address still returns bytes, and the value is then a multiple of that unit.

## /machine/ncMemorySizeUsed
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The NC memory in use. Returns `int` + `unit:"bytes"`, read-only.

What it refers to is the **machining-program memory**. On Fanuc it is the drive capacity from `cnc_rdpdf_inf` (in bytes, despite the field names); on Siemens the user area of the NC's **passive file system** (where main and subprograms, workpieces and GUD definition files live; Extended Functions manual, S7), `/Nck/State/usedMemDramUPassF` and `freeMemDramUPassF`; on Mitsubishi the character counts from `GetInformation` (the same capacity and remaining-space figures shown on the control's edit screen). One of the three is calculated from the other two (Fanuc: free = total - used; Siemens and Mitsubishi: total = used + free; Heidenhain: used = total - free), so the three read together in one request satisfy `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree`. On Siemens, when the value cannot be read the answer is status `-17`, not `0`.

**On Mitsubishi the value is that of the main root (`//PRG`, `Memory` on the operator panel).** The second program memory of M800V/M80V controls (`//PRG2`, `Memory2` on the panel) counts its capacity separately and is not included (confirmed in our test environment).

**On Heidenhain the value is that of `//TNC` (the `TNC:` drive on the control).** It is the total and free bytes from HEIDENHAIN DNC's `GetDiskSpace`, and the used space is their difference. In our test environment the used space shown by the control's file manager was larger than `ncMemorySizeUsed` by about 5% of the total capacity (the same difference at two comparisons with files added in between; the total was the same). The control's figure was recalculated only when the control restarted, so adding or deleting files in the meantime did not change it.

All sizes in the SDK are in **bytes**, the same unit as `sizeBytes` in `entry`/`entryList`, so a question like "does this file fit in the free space" needs no conversion. When a machine only reports a coarser unit, this address still returns bytes, and the value is then a multiple of that unit.

## /machine/ncMemorySizeFree
```yaml
value_type: "int"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The free NC memory capacity. Returns `int` + `unit:"bytes"`, read-only.

What it refers to is the **machining-program memory**. On Fanuc it is the drive capacity from `cnc_rdpdf_inf` (in bytes, despite the field names); on Siemens the user area of the NC's **passive file system** (where main and subprograms, workpieces and GUD definition files live; Extended Functions manual, S7), `/Nck/State/usedMemDramUPassF` and `freeMemDramUPassF`; on Mitsubishi the character counts from `GetInformation` (the same capacity and remaining-space figures shown on the control's edit screen). One of the three is calculated from the other two (Fanuc: free = total - used; Siemens and Mitsubishi: total = used + free; Heidenhain: used = total - free), so the three read together in one request satisfy `ncMemorySizeTotal` = `ncMemorySizeUsed` + `ncMemorySizeFree`. On Siemens, when the value cannot be read the answer is status `-17`, not `0`.

**On Mitsubishi the value is that of the main root (`//PRG`, `Memory` on the operator panel).** The second program memory of M800V/M80V controls (`//PRG2`, `Memory2` on the panel) counts its capacity separately and is not included (confirmed in our test environment).

**On Heidenhain the value is that of `//TNC` (the `TNC:` drive on the control).** It is the total and free bytes from HEIDENHAIN DNC's `GetDiskSpace`, and the used space is their difference. In our test environment the used space shown by the control's file manager was larger than `ncMemorySizeUsed` by about 5% of the total capacity (the same difference at two comparisons with files added in between; the total was the same). The control's figure was recalculated only when the control restarted, so adding or deleting files in the meantime did not change it.

All sizes in the SDK are in **bytes**, the same unit as `sizeBytes` in `entry`/`entryList`, so a question like "does this file fit in the free space" needs no conversion. When a machine only reports a coarser unit, this address still returns bytes, and the value is then a multiple of that unit.

**On Mitsubishi this value moves in steps of 250 bytes**, because the control counts the remainder in units of 250 characters. The unit is bytes as on every other machine type; only the granularity differs.

## /machine/ncMemoryRootPath
```yaml
value_type: "string"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The root path of the main NC memory, the starting point for paths to put in the `ncMemoryPath` filter. Fanuc is usually `//CNC_MEM`, Siemens `//NC`, Mitsubishi usually `//PRG`, and Heidenhain `//TNC`. On Mitsubishi the root stays `//PRG` even when the program selected at connection time is in the second program memory (`//PRG2`) or on the SD card (`//IC1`); those two appear in `/machine/ncMemoryExternalRootPathList` (`//PRG2` confirmed in our test environment; the SD card has not been confirmed yet).

**Under Fanuc's `//CNC_MEM`** there is `USER`, where machining programs live (per-path `PATH1`, `PATH2`, … and the shared `LIBRARY`), and the system and machine tool builder macro folders `SYSTEM`, `MTB1` and `MTB2` (confirmed in our test environment). deemesh does not write to the last three (status `-18`; reading works).

**On Mitsubishi, `//PRG` is the program area of `Memory` (the NC memory) on the operator panel, and machining programs live below it in `//PRG/USER`** (the `Memory:/Program` list on the panel). Listing `//PRG` shows folders: `USER` (machining programs), `MDI` (the MDI program `MDI.PRG`), `FIX` (fixed cycles), `MMACRO` (machine tool builder macros). M700-series controls also show `UMACRO`. On M800-series controls deemesh leaves `UMACRO` out of the list: the manual gives that path for M700 only, and in our test environment (M800V) it pointed at the same files as `USER` (paths under `//PRG/UMACRO/…` still open when you give them directly). deemesh does not write under `FIX` or `MMACRO` (status `-18`; reading works). The second program memory of M800V/M80V controls (`Memory2` on the panel) is `//PRG2` and appears in `ncMemoryExternalRootPathList`. It holds only the machining program folder `USER`. This value stays `//PRG` even when a program there is selected.

**On Heidenhain, `//TNC` is the `TNC:` drive of the control's file manager, and machining programs usually live under `//TNC/nc_prog`.** deemesh works with this drive only. In our test environment access to the other drives was refused, so `ncMemoryExternalRootPathList` answers status `-20` (the TNC7 User's Manual assigns the `PLC:` drive to the machine manufacturer's user and `SYS:` to the service user). deemesh does not write under `//TNC/table`, `//TNC/system` or `//TNC/config`, which hold tables and settings (status `-18`; reading works; see `fileContent`). Names are found regardless of letter case.

## /machine/ncMemoryExternalRootPathList
```yaml
value_type: "stringArray"
null_able: false
required_filters: []
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
write: []
```

The list of **storage locations** other than the main NC memory root (e.g. data server, memory card). Return type `stringArray`.

- Each item, like the root, has a leading `//` (no trailing slash): Fanuc `//DATA_SV`/`//MEM_CARD`, Siemens `//Local drive`, Mitsubishi `//IC1` (the NC-side SD card, shown as `DS` on the operator panel; not yet confirmed with an SD card). The array is empty when there is none
- **On Mitsubishi M800V/M80V controls the second program memory `//PRG2` appears here as well** (`Memory2` on the operator panel; programs live in `//PRG2/USER`). It is memory inside the NC, but it counts program slots and capacity separately from the main root (`//PRG`), so it is listed here. Every part system sees the same contents (confirmed in our test environment)
- Names are the **machine HMI wording**. The Siemens local drive is called `NCExtend` internally in OPC-UA, but it is reported as `//Local drive` to match the operator panel (requests using the old `//NCExtend` are still accepted)
- The main root itself is excluded from this list
- **Not cached**: external devices can change by connect/disconnect, so it is re-queried on each request
- **On Mitsubishi, deemesh opens `//IC1` and `//PRG2` and leaves one out when the control refuses it.** A communication error during that check does not leave the storage out: the whole request answers the error
- No filter

## /machine/ncMemoryPath/entry
```yaml
value_type: "object"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

Information about a single entry at the path (`object`). The key set is **always the same regardless of machine**: when a value is unavailable the key is not dropped, it is `null`.

| Key | Type | When unavailable |
|---|---|---|
| `name` | `string` | n/a |
| `sizeBytes` | `int` | `null` for a folder, or when the size could not be read. Siemens, Mitsubishi and Heidenhain report the content's byte count; **Fanuc reports the allocated size (in 500-byte units)**, so a 22-byte program lists `500` |
| `modifiedAt` | `string` | `null` for a folder, or on a machine without modification times (always `null` on Siemens; for Heidenhain see below) |
| `isDir` | `boolean` | n/a |
| `comment` | `string` | `null` for a folder, or on a machine without comments (always `null` on Siemens and Heidenhain; see below) |

**`comment` is the program comment the control delivers together with the listing.** On Fanuc it is the parenthesized comment on the O-number line, on Mitsubishi the comment column of the program list (the parenthesized comment in the first block), so **one listing gives it without opening any file** (confirmed on the simulators). **On Siemens it is always `null`, because the listing carries no comment**; a `;` comment on the first line is file content and shows only when you read `fileContent`. If you must tell programs apart from the listing alone, separate them on Siemens by **folder or name** instead of a comment: a subfolder (created with `directoryExists`) and a name prefix both show up in a single `entryList`. Reading `fileContent` for every candidate costs one transfer per file, which adds up on a slow link.

A trailing `/` on the path forces a folder; without it files are searched first. A missing entry answers status `-18` (including a name inside an empty folder and a path below a folder that does not exist). Status `-18` means only that the entry is not there; a communication error answers its own status. On Mitsubishi, however, a folder path that is too long also answers status `-18`, and then the error text says the path or name is too long rather than that the entry is missing (confirmed on the simulator).

**On Heidenhain, `modifiedAt` is the time HEIDENHAIN DNC gives, unchanged and without a `Z`.** It is the time as seen in the time zone of the PC your program runs on, while the control's file manager shows times in the time zone set on the control. So it matches the control when the two time zones are the same, and differs by the difference between them otherwise (in our test environment, setting the control's time zone to that of the PC made the file manager match this value as soon as the listing was refreshed, and this value did not change). Heidenhain finds names regardless of letter case, and `name` comes back as the control stores it. The listing carries no comment, so `comment` is always `null`.

## /machine/ncMemoryPath/entryList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The list of files and folders inside a folder (**read-only**, `objectArray`). Each element is **exactly the same object** as `entry`: same key set and same `null` convention, so see that table. Sorted folders first, then by name ascending. To create or delete a folder use `directoryExists`.

A folder that does not exist returns status `-18`.

**On Mitsubishi M800-series controls the root (`//PRG`) list leaves out `UMACRO`.** On those controls it is a name for the same files as `USER`, so it is left out to keep programs from appearing twice. Paths under `//PRG/UMACRO/…` still work (see `ncMemoryRootPath`).

An empty folder answers `[]`.

**On Heidenhain the control's listing is returned as it is, leaving out only `.` and `..`.** Entries with the hidden attribute appear too (for example the `*.T.DEP` files that appeared in our test environment when a program ran; the control's file manager shows them as well, and they appeared even with the machine setting for creating tool usage files set to never). `lost+found` and `.nc_index`, which the control's screen does not show in the `//TNC` list, do appear (confirmed in our test environment). The control refuses the listing of `lost+found`, which answers status `-17`.

`comment` comes with the single listing on Fanuc and Mitsubishi and is always `null` on Siemens and Heidenhain. The reason and the alternative (separating programs by folder or name) are in the `comment` note of `entry`.

## /machine/ncMemoryPath/entryName
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

The **name of the entry** that `ncMemoryPath` points at. Reading returns **the name the machine holds for that entry**, and writing performs a **rename**. The read answers for a file or a folder alike, and rejects a path with nothing at it with status `-18`, the same judgement `entry` makes. Status `-18` means only that nothing is there; a communication error answers its own status. What comes back is the machine's own name, not the string you asked with, so a difference in spelling shows the machine's version. Writes take `{"value": "new name"}`, and path separators are not allowed, common to files/folders. **A root cannot be renamed**: if `ncMemoryPath` is a single segment such as `//CNC_MEM`, `//NC`, `//PRG` or an external drive, the request is refused with status `-18` (filter value error) without touching the machine. A root written with backslashes, such as `\\NC`, is refused the same way. On Siemens, `//NC/Part programs`, `//NC/Subprograms` and `//NC/Workpieces` are not renamed either (the same rule as the delete refusal of `directoryExists`). It refers to the same "entry" as `entry`/`entryList`. **Siemens does not rename the selected main program while the channel is running (not in Reset).** That case answers status `-22` (machine state); reset the channel or try again after the program has ended. Even with every channel in Reset, a refusal of the rename in the current state (`BadInvalidState`) answers status `-22` as well, and the error text then points to an editor or another client holding the file. **A subprogram the look-ahead has opened differs from the delete**: on our test bench, during a run, deleting that subprogram (`fileExists`) answered status `-22` while renaming it was accepted. Rename a subprogram the running program will call while the channel is in Reset. **Fanuc does not rename its selected main program (even when not running), and Mitsubishi does not rename the main program of a part system in automatic operation**; that answers status `-22` (machine state). On Fanuc an entry under protection (parameter `3202#0`/`#4`, or an edit-disable attribute set on the panel), and a rename into a protected number range, also answer status `-22`. On Mitsubishi the rename answers status `-22` while the data protection by operation level (parameter `#1391`) protects program editing (confirmed on the simulator; see `fileContent`). **If the entry to rename does not exist**, all four controls answer status `-18`. The Fanuc data server (`//DATA_SV`) answers the same way, and a file selected there as the main program also answers status `-22` (confirmed on a 31i-B machine tool). An empty new name answers status `-16` (invalid write value) on all four controls. **If the new name is the entry's current name**, nothing is sent to the control; deemesh only checks that the entry exists (success if it does, status `-18` if not), as a file system rename does, and only when the letter case matches too. Mitsubishi's program memory (`//PRG`, `//PRG2`) stores names in upper case, so there a name that differs only in letter case counts as the same name. **If an entry with the new name already exists**, all four controls refuse the request with status `-21` (already exists) and nothing is changed (deemesh does not delete that entry for you). **deemesh does not rename anything in the system and machine tool builder areas (Fanuc `//CNC_MEM/SYSTEM`, `MTB1` and `MTB2`; Mitsubishi `//PRG/FIX` and `//PRG/MMACRO`) or in Heidenhain's `//TNC/table`, `//TNC/system` and `//TNC/config`** (refused with status `-18` without reaching the machine; see `fileContent`). If the new name matches the entry's current name **including letter case**, though, the same-name check above comes first and the answer is status `0` (a name differing only in case is refused with status `-18`, because the builder-area refusal comes first); nothing is sent to the machine, so nothing changes (the same holds for the three fixed Siemens folders). **The Mitsubishi edit lock (`#8105`, `#1121`) does not prevent renaming**: renaming a program in the locked range and renaming another program into that range were both accepted (confirmed in our test environment; see `fileContent`). **On Heidenhain, a rename to a name that differs only in letter case answers status `-21`**: the control treats such names as the same name, so deemesh does not send it. An entry write-protected in the control's file manager and a program that is running (main or subprogram) answer status `-22` (machine state), and the reason says which (confirmed in our test environment).

**On the Fanuc data server (`//DATA_SV`), this write moves the data server's current folder.** The Fanuc functions for data server folders and files take only a name within the current folder, so deemesh first moves to the target's parent folder, calls the function, and leaves the current folder there. That current folder is one per machine and is the same one the operator panel's data server screen uses; the panel shows the changed folder when that screen is opened again (confirmed on a 31i-B machine tool). So **write to one machine's data server from one place only**: if another program or the operator panel moves the folder at the same time, a call that uses a name can reach the same name in a different folder. On a machine where folder and file operations on the data server cannot be done over this connection, the answer is status `-18` (filter value error) and the reason says so (one of our test benches is like this, while its operator panel can do the same operations).

## /machine/ncMemoryPath/directoryExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

Checks whether a **folder** exists at the path (read) and declaratively writes the state (write):

- read → `true` if the folder exists (`false` if only a file of the same name exists). `false` means **only** that it is not there: a communication error or another refusal by the control answers its own status, not `false`. On Mitsubishi a folder path that is too long answers status `-18`, not `false`, with an error text saying the path or name is too long (confirmed on the simulator). On Fanuc, asking about a drive itself such as `//CNC_MEM` answers `true` (so does Heidenhain's `//TNC`)
- write `{"value": true}` → create the folder (status `-21` (already exists) if it is already there. The Fanuc data server `//DATA_SV/` answers the same: the refusal alone does not tell whether it already exists, so after a refusal deemesh reads the parent folder's list once to tell). On Fanuc, when the number of entries that can be registered is used up, the answer is status `-23` (no room); folders and programs count toward the same number (confirmed on the test bench)
- write `{"value": false}` → delete the folder. **What is removed differs by machine type**: Fanuc and Heidenhain delete empty folders only and refuse a folder that has contents with status `-17` (handler error) and the reason. **Siemens deletes the files and subfolders the folder holds along with it** (confirmed on our bench). Check the contents with `entryList` before you delete. If a running channel holds a program inside that folder, Siemens refuses with status `-22` (machine state). **If the folder does not exist the answer is status `-18` (filter value error)** (all four machine types; on the Fanuc data server `//DATA_SV/` the vendor's refusal is returned as it is). **A root cannot be deleted**: if `ncMemoryPath` is a single segment such as `//CNC_MEM`, `//NC`, `//PRG` or an external drive, the request is refused with status `-18` (filter value error) without touching the machine. `\` counts as `/`, so `\\NC` is refused as a root too. A path that does not start with `//`, or that contains `.`, `..` or an empty segment, is refused before that as a spelling error, also with status `-18`. **On Siemens, `//NC/Part programs`, `//NC/Subprograms` and `//NC/Workpieces` are not deleted either** (refused with status `-18` without reaching the machine, whatever the letter case and with or without a trailing `.DIR`). Give an entry inside them instead
- **On Fanuc, a folder with an edit-disable attribute set on the panel** cannot be deleted and no folder can be created in it; that answers status `-22` (machine state)
- **On Fanuc, deemesh does not create or delete folders at or below `//CNC_MEM/SYSTEM`, `MTB1` or `MTB2`** (refused with status `-18` without reaching the machine; see `fileContent`)
- **On the Mitsubishi NC memory drive, creating or deleting a folder answers status `-20`.** That drive has a fixed directory layout, so the control accepts neither. On the SD card (`//IC1`), creating a folder that is already there answers status `-21`, no room answers status `-23`, deleting a folder that is not empty answers status `-17` (with the reason), and a write-protected card answers status `-22` (not yet confirmed with an SD card). `//PRG/FIX`, `//PRG/MMACRO` and anything below them are refused with status `-18` before reaching the machine (see `fileContent`). Reading works normally
- **Heidenhain**: creating a folder that is already there answers status `-21`, and a missing parent folder for the new one answers status `-18`. A folder write-protected in the control's file manager cannot be deleted and no folder can be created in it; that answers status `-22` (machine state) with the reason (confirmed in our test environment). A folder in which a program has run keeps the hidden `*.T.DEP` files, so deleting it can answer status `-17` (not empty) even after every file the panel shows is gone (check with `entryList`). What deemesh deletes does not pass through the control's recycle bin and cannot be restored (confirmed with a file in our test environment; the TNC7 User's Manual, 'Basic information', says that what the control's file manager deletes goes to the recycle bin). deemesh does not create or delete folders at or below `//TNC/table`, `//TNC/system` or `//TNC/config` (refused with status `-18` without reaching the machine; see `fileContent`)

A trailing `/` in the path is ignored. For files, use `fileExists`.

**On the Fanuc data server (`//DATA_SV`), this write moves the data server's current folder.** The Fanuc functions for data server folders and files take only a name within the current folder, so deemesh first moves to the target's parent folder, calls the function, and leaves the current folder there. That current folder is one per machine and is the same one the operator panel's data server screen uses; the panel shows the changed folder when that screen is opened again (confirmed on a 31i-B machine tool). So **write to one machine's data server from one place only**: if another program or the operator panel moves the folder at the same time, a call that uses a name can reach the same name in a different folder. On a machine where folder and file operations on the data server cannot be done over this connection, the answer is status `-18` (filter value error) and the reason says so (one of our test benches is like this, while its operator panel can do the same operations).

## /machine/ncMemoryPath/fileExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

Checks whether a **file** exists at the path (read) and declaratively writes the state (write):

- read → `true` if the file exists (`false` if only a folder of the same name exists). `false` means **only** that it is not there: a communication error or another refusal by the control answers its own status, not `false`. On Mitsubishi a folder path that is too long answers status `-18`, not `false`, with an error text saying the path or name is too long (confirmed on the simulator)
- write `{"value": false}` → delete the file. **If the file does not exist the answer is status `-18` (filter value error)** (all four machine types). This keeps a mistyped path from passing as a success, so when deleting a file that may already be gone, treat this code as "already gone". The Fanuc data server `//DATA_SV/` does not make this distinction and returns the vendor's refusal as it is
- write `{"value": true}` → refused with status `-16` (invalid write value): creating an empty file is not supported. Create a file with its content via a `fileContent` write
- **deemesh does not delete in the system and machine tool builder areas** (Fanuc `//CNC_MEM/SYSTEM`, `MTB1` and `MTB2`; Mitsubishi `//PRG/FIX` and `//PRG/MMACRO`; Heidenhain `//TNC/table`, `//TNC/system` and `//TNC/config`). The request is refused with status `-18` without reaching the machine (see `fileContent`)
- **The Mitsubishi edit lock (`#8105`, `#1121`) does not prevent deletion.** Programs in the locked number range are deleted as well (confirmed in our test environment; see `fileContent`)
- **When Mitsubishi's data protection by operation level protects program editing, the delete answers status `-22` (machine state)** (parameter `#1391`; confirmed on the simulator; see `fileContent`)
- **On a write-protected SD card on Mitsubishi (`//IC1`), the delete answers status `-22` (machine state)** (not yet confirmed with a write-protected SD card)

A trailing `/` in the path is ignored (the kind is fixed by the address). For folders, use `directoryExists`.

**Siemens does not delete a file a channel is using while that channel is not in Reset.** The selected main program, the running subprogram and a subprogram **the look-ahead has already opened** answer status `-22` (machine state) on delete. Because the lock window is wider than "executing right now", a delete can seem to succeed at one moment and fail at another while the same program runs, but it is not random. A subprogram the main only references and has not yet called can be deleted (confirmed on the test bench). Reset the channel or delete again after the program has ended. Even with every channel in Reset, a refusal of the delete in the current state (`BadInvalidState`) answers status `-22` as well, and the error text then points to an editor or another client holding the file.

**Fanuc does not delete its selected main program (even when not running), and Mitsubishi does not delete the main program of a part system in automatic operation.** The delete answers status `-22` (machine state). On Fanuc a program under protection (parameter `3202#0`/`#4`, or an edit-disable attribute set on the panel) also answers status `-22` on delete. On Fanuc, select another program first; on Mitsubishi, reset the part system or delete after the program has ended. In our test environment both controls deleted a subprogram that was running.

**Heidenhain does not delete a program that is running (main or subprogram) or one whose run stopped and has not ended** (the TNC7 User's Manual, 'Calling an NC program with PGM CALL', says the programs a running program calls cannot be edited while it runs). That answers status `-22` (machine state). Once the run ended or another program was selected, it was deleted. A file write-protected in the control's file manager also answers status `-22`, and the reason says it is write-protected. A selected program that has not run, or whose run ended, is deleted too, and `/machine/channel/mainProgramPath` still names the deleted path afterwards (confirmed in our test environment). A file deleted through deemesh was not in the control's recycle bin; it cannot be restored, so check before you delete (the manual, 'Basic information', says that what the control's file manager deletes goes to the recycle bin).

On Siemens, **with the channel in Reset the selected main program can be deleted, and the control then clears the selection.** On an 840D sl bench `/machine/channel/mainProgramPath` read an empty string (no program selected) after the delete. Do not assume the same program is still selected after deleting it; select a program again if you need one.

**On the Fanuc data server (`//DATA_SV`), this write moves the data server's current folder.** The Fanuc functions for data server folders and files take only a name within the current folder, so deemesh first moves to the target's parent folder, calls the function, and leaves the current folder there. That current folder is one per machine and is the same one the operator panel's data server screen uses; the panel shows the changed folder when that screen is opened again (confirmed on a 31i-B machine tool). So **write to one machine's data server from one place only**: if another program or the operator panel moves the folder at the same time, a call that uses a name can reach the same name in a different folder. On a machine where folder and file operations on the data server cannot be done over this connection, the answer is status `-18` (filter value error) and the reason says so (one of our test benches is like this, while its operator panel can do the same operations).

## /machine/ncMemoryPath/fileContent
```yaml
value_type: "string"
null_able: false
required_filters: ["ncMemoryPath"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
```

Reads the **content** of an NC file (download) and writes it (upload: creates the file if absent; what happens when it already exists depends on the control, see below). The value is a string (program text).

- **Fanuc write auto-handling**: if `%` is absent, it is inserted automatically; if there is no leading O number/`<name>`, it is inserted automatically based on the path's file name; if the last block does not end with a line break, one is added (without it the last block was stored like `M30%` and the program stopped at that block with alarm `SR5010`; seen on a 31i-B machine tool). The saved file name is **based on the O number/name in the content**
- **Siemens, Mitsubishi and Heidenhain write the content verbatim.** Nothing is inserted, and the saved file name is **the file name in the path**: the file is stored under the path even when the O number in the content differs (the opposite of Fanuc). Include `%` or an O number yourself if you need them
- **Writing to a file that already exists**: on Fanuc, when parameter `3201#2` (REP) is `1` the existing program is deleted and the new one registered (with `0` the answer is status `-21` (already exists) and the existing program is untouched; to replace it, write `false` to `fileExists` first and then upload, or set `3201#2` to `1`; parameter manual B-64490EN). Mitsubishi and Heidenhain overwrite (confirmed in our test environments). **Siemens does not overwrite: the write answers status `-21` (already exists) and the existing content is untouched** (confirmed on the test bench). To replace it, write `false` to `fileExists` first and then upload. deemesh does not delete and recreate on your behalf, because if the creation failed the original would be gone. That decision belongs to the caller
- **Mitsubishi edit lock**: with parameter `#8105` (edit lock B) at `1`, programs 8000 to 9999, and with `#1121` (edit lock C) at `1`, programs 9000 to 9999, cannot be created or overwritten, and the write is refused with status `-22` (machine state) (reference manual IB-1501209). The existing content is untouched. In our test environment the lock applied to writing only: reading, selecting, renaming and deleting those programs were accepted regardless of it
- **Mitsubishi data protection by operation level**: when parameter `#1391` is `1` and the change level set for "Program edit" on the operator panel's protection setting screen (Mainte > Protect setting) is above the current operation level, the write is refused with status `-22` (machine state) and the existing content is untouched (confirmed on the simulator). Raise the operation level on the operator panel with that level's password, then write again. Renames (`entryName`) and deletes (`fileExists`) behave the same, and reads were not blocked
- **A write-protected SD card on Mitsubishi (`//IC1`)**: the write is refused with status `-22` (machine state). This is not yet confirmed with a write-protected SD card
- **Protection on Fanuc**: a program protected by parameter `3202#0`/`#4` (editing of O8000 to O8999 and O9000 to O9999 inhibited) or by an edit-disable attribute set on the panel (folder or file) is refused for writing (including creating it) with status `-22` (machine state). The `3202` protection also refuses reading with status `-22` when `3202#6` (PSR) is `0` (with `1` the program can be read, as confirmed on a 31i bench), while the edit-disable attribute does not block reading (confirmed in our test environment). Ask the person responsible for the machine whether the protection may be lifted
- **A program the control is using, on Fanuc and Mitsubishi**: Fanuc does not overwrite its selected main program (even when not running) and Mitsubishi does not overwrite the main program of a part system in automatic operation; the write is refused with status `-22` (machine state). On Fanuc this refusal applies when `3201#2` (REP) is `1`; with `0` the status `-21` (already exists) above comes first (confirmed in our test environment). On Fanuc, select another program first; on Mitsubishi, reset the part system or write again after the program has ended. In our test environment neither control refused overwriting a subprogram that was running
- **A program the control is using, and write protection, on Heidenhain**: a running program, whether the main or a subprogram not yet called, is not overwritten, and the write answers status `-22` (machine state). Overwriting a file write-protected in the control's file manager, or creating one in a write-protected folder, also answers status `-22`, and the reason says it is write-protected. A missing folder, or a path that is a folder, answers status `-18` (confirmed in our test environment). HEIDENHAIN DNC transfers files through files on the PC, so for every read and write deemesh creates a file in the PC's temporary folder and deletes it right after. The bytes sent were stored and read back unchanged (including UTF-8 Korean text and CRLF, in our test environment). The TNC7 User's Manual ('Converting files') says the control can adapt imported files depending on the machine manufacturer's settings (removing umlauts, for example); whether a HEIDENHAIN DNC transfer counts as such an import has not been confirmed. The `BEGIN PGM` and `END PGM` lines of a conversational program are inserted by the control's editor but not by deemesh, so include them in the value. Use letters, digits, `_` and `-` in file names, and keep the path within 255 characters (manual, 'Basic information')
- **When there is no room**: if the NC memory is short or the number of programs that can be registered is used up, the write answers status `-23` (no room) (confirmed on the test bench for Fanuc and on the simulator for Mitsubishi). Delete programs you no longer need and upload again. Overwriting an existing program works even when that number is used up (confirmed on Mitsubishi). On Fanuc, folders count toward the same number. On Mitsubishi `//PRG2`, a full count cannot be told apart and the write answers status `-17`. **When Fanuc runs out of memory, it registers as much as fit, cut at a block, under the O number/name in the content.** Only the final `M30` is missing, so it looks like a complete program; deemesh checks that the program is the start of what was sent, deletes it and says so in the error text. If `3201#2` (REP) is `1`, an existing program of that name had already been replaced by the control and is gone as well. With no free space at all the control changes nothing, and an existing program stays as it was. If deemesh cannot check or delete the program, the error text says so; check it before running it
- **On Fanuc, when the content is not in a valid format**: the control raises an alarm (for example `BG1090`) and may register what it accepted so far under that name. deemesh answers status `-16` (invalid write value) and puts the alarm code in the error text; fix the content and upload again. If the name was not in the folder before the upload, deemesh deletes the program the control kept and says so in the error text. If the name was already there, deemesh does not delete it: with `3201#2` (REP) at `1` the existing program may have been replaced by what the control accepted, so check it before running it (with `0` the answer is status `-21` and the existing program stays as it was). The alarm needs RESET on the operator panel, but other uploads are accepted before it is cleared. To tell these cases apart, deemesh reads the folder's list once before each upload to Fanuc (the alarm and the leftover program were seen on a machine tool and on the simulator, and deemesh's cleanup was confirmed in CNC memory on the simulator and on a machine tool's data server)
- **Creating a file under a name a channel is using, on Siemens**: if a channel holds that name (the selected main program, or a subprogram that is running or that the look-ahead has opened; typically when re-creating a file right after deleting it), the file is created but the `Open` that would write its content answers status `-22` (machine state). Even with every channel in Reset, a refusal of that `Open` in the current state (`BadInvalidState`) answers status `-22` as well, and the error text then points to an editor or another client holding the file. In either case deemesh **deletes the empty file it just created within the same call** (back to the state before the call). If the channel holds even that empty file so it cannot be removed, the error text says so; delete it once the channel releases it
- To delete a file, write `false` to `fileExists`
- **deemesh does not write to the system and machine tool builder areas.** The request is refused with status `-18` without reaching the machine, and reading works (the same applies to `fileExists`, `entryName` and `directoryExists`). A wrong change there can alter how the machine behaves or leave it unusable, so these areas are blocked whatever the machine's protection settings are
  - Fanuc: `//CNC_MEM/SYSTEM`, `//CNC_MEM/MTB1` and `//CNC_MEM/MTB2` (the system and machine tool builder macro folders; G, M and T code macro calls look programs up there, Operator's Manual B-64484EN). `//CNC_MEM/USER/LIBRARY` is a shared user folder and is not blocked
  - Mitsubishi: `//PRG/FIX` (fixed cycles) and `//PRG/MMACRO` (machine tool builder macros). The operator panel also opens this area for editing only with parameter `#1166` switched on
  - Heidenhain: `//TNC/table` (tool table, presets and the like), `//TNC/system` and `//TNC/config`, and everything below them. They hold files the control uses under names it relies on, so a wrong delete or overwrite can leave the machine unusable. These folders also hold subfolders the TNC7 User's Manual names as places for user files (`system/PGM-Templates`, `system/Toolkinematics`, `system/3D-ToolComp`, freely definable tables in `table` and so on), but deemesh does not upload there; handle those on the control

**A write's `status` 0 means the transfer is complete.** On all three controls deemesh checks the control's return value for every chunk and then confirms the close before answering 0 (Fanuc `cnc_download4` and `cnc_dwnend4`, Siemens `Write` and `Close`, Mitsubishi `WriteFile` and `CloseFile3`). Heidenhain answers 0 after its single file transfer (`TransmitFile`) has finished. A failure midway is an error; Mitsubishi discards the file, Siemens removes the partly created one, and Fanuc removes a program cut short by a memory shortage (see above). On Mitsubishi, when the final close is refused (for example, when the NC memory is short), the control leaves an incomplete temporary file in the target folder (its name starts with `~`; confirmed on the simulator); deemesh deletes it and says so in the error text. If it cannot check or delete it, the error text says so: find that file in the listing and delete it by writing `false` to `fileExists`. It is not a finished program. So **there is no need to read the file back to confirm the transfer.** A byte comparison after reading back differs on Fanuc anyway, because of the inserted `%` and O number and the line-ending normalization, and the listing's `sizeBytes` on Fanuc is an allocation size in 500-byte units (writing 22 bytes lists `500`), not the content length (confirmed in our tests). That is why this address puts no size or hash in the write response: a size means different things per control, and a hash cannot be produced without reading back, which only moves that cost inside. If you need to verify content, read `fileContent` and compare by **meaning** (program number, blocks).

## /machine/channel/toolOffsetCount
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

The **number of available** tool compensation registers (read only, `int`). Offset numbers run `1` to this value; use it as the upper bound when a UI iterates the table.

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table, so there is no register count. Read the tools from `/machine/toolArea/toolList` and a tool's number of cutting edges from `/machine/toolArea/tool/toolEdgeCount`.

## /machine/channel/toolOffset/toolOffsetValue
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

The **compensation amount** of one tool compensation number (read + write, `float`). Needs the `channel` and `toolOffset` filters; writes take `{"value": 12.345}`.

**This address is for Mitsubishi machines whose compensation memory is not split into columns.** On such a machine the control's compensation screen shows exactly **one** value per number: no geometry/wear split and no length/radius split. That is why the name is `toolOffsetValue` and not `toolLength…`: **the control does not call this value a length**, and whether it acts as a length or a radius compensation depends on how the program references that number.

On a Mitsubishi machine whose memory is split, this returns status `-20`, and **the error string carries the list of leaves that do work there** (for example `toolLength{Geometry,Wear}, toolRadius{Geometry,Wear}`). One request therefore tells you the shape of that machine's compensation tree, so there is no separate address to ask which model it uses.

The upper bound for the compensation number is `/machine/channel/toolOffsetCount`; a number beyond it returns status `-18`. The unit follows the machine's configuration (mm or inch); check with `/machine/channel/gModalCategory/gModal?gModalCategory=4`.

The control rounds a written value to the places of its setting unit `#1003` and answers status `0` (the same as `workOffsetValue`; confirmed in our test environment). A value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed for the same write call on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). This write is **accepted during automatic operation as well** (confirmed in our test environment while the axes were moving, including for the number the program was using; unlike `workOffsetValue`, which is refused during operation).

**Fanuc and Siemens answer with status `-20`.** The adapters for those two do not support this address, so they refuse it whatever the compensation memory looks like, and the error text carries no list of leaves. On Fanuc read the `toolLength…` and `toolRadius…` leaves (the `toolX…` family on a lathe). The FOCAS2 specification has the value of an offset memory without a length/radius split (memory A or B) addressed through the cutter-radius columns, but we have not confirmed this on a machine with that configuration. Siemens keeps compensation per tool; read it under `/machine/toolArea/tool/toolEdge/…`.

## /machine/channel/toolOffset/toolName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

The **name of the tool** called by that offset number. `channel` + `toolOffset` filters. Returns `string`. Reading and writing are Fanuc only.

The value comes from Fanuc's **tool geometry size data** (the panel's `TL GEOM SIZE` screen, option "Tool geometry size data 100/300 pairs"). That table is **indexed by the tool offset number** (Operator's Manual B-64484EN: the row with the same number as the `D` code on machining centres, or the geometry offset number on lathes). That is why this address lives in the `toolOffset` folder rather than under a tool management slot (`/machine/toolArea/tool/…`), and it does not depend on the TOOL MANAGEMENT option. Without the geometry size data option the address returns status `-20`.

A row whose tool type is not set (type `0`) reads as the empty string `""`. The table's size is set by the option, so it is not guaranteed to equal `/machine/channel/toolOffsetCount` (both are 100 on our bench machine). A number beyond the table is rejected with status `-18`, and the message names where the table ends. Asking for a range (`toolOffset=1-100`) costs one round trip.

**Writing** takes a string that fits in 8 bytes on the machine (longer is status `-16`; characters the machine's display-language code page cannot store are status `-16` as well). A row without a tool type cannot take a name, status `-18` (Fanuc does not keep a row without a type; write `/machine/channel/toolOffset/toolType` first, or set the type on the panel's `TL GEOM SIZE` screen). If the name is already the same nothing happens and the write succeeds.

On Siemens the tool name belongs to the tool number, so it is `/machine/toolArea/tool/toolName`. Same meaning (the tool's name), different key, hence different addresses.

**Mitsubishi answers with status `-20`.** This column belongs to Fanuc tool geometry size data and does not exist in the Mitsubishi offset table.

## /machine/channel/toolOffset/toolType
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

The **kind of the tool** called by that offset number (the `TK` icon on the panel's `TL GEOM SIZE` screen). `channel` + `toolOffset` filters. Returns `int` + `desc`. Reading and writing are Fanuc only.

The value is **Fanuc's raw code, passed through**. It is a vendor-owned open classification, so it is not translated and not unified across machine types (Siemens' `/machine/toolArea/tool/toolEdge/toolType` carries the DP1 code, a **different code space** with a different key). `desc` is prose for a human; branch on the value.

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

**Mitsubishi answers with status `-20`.** This column belongs to Fanuc tool geometry size data and does not exist in the Mitsubishi offset table.

## /machine/channel/toolOffset/toolLengthGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **M-type (machining center) tool length geometry** value (the H column on the offset screen). The reference value entered from tool measurement, forming the basis of length compensation (H code).

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; on Mitsubishi that includes a machine whose compensation memory is **not split into columns**, where the error string points you at `toolOffsetValue` instead. The Fanuc adapter does not support `toolOffsetValue`, so no such pointer appears there; the FOCAS2 specification has the value of an offset memory without a length/radius split (memory A or B) addressed through the cutter-radius columns (`toolRadius…`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is one of the four columns of a machine whose compensation memory is split into geometry/wear and length/radius (type II; checked in our test environment against the Length, L wear, Radius and R wear cells on the operator panel). A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed for the same write call on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed on a simulator in this configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table; read the same value from the leaf of the same name under `/machine/toolArea/tool/toolEdge/…`.

## /machine/channel/toolOffset/toolLengthWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **M-type tool length wear** value (the H column on the offset screen). The fine compensation accumulated during machining; the usual practice is to adjust this alone and leave the geometry value untouched.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; on Mitsubishi that includes a machine whose compensation memory is **not split into columns**, where the error string points you at `toolOffsetValue` instead. The Fanuc adapter does not support `toolOffsetValue`, so no such pointer appears there; the FOCAS2 specification has the value of an offset memory without a length/radius split (memory A or B) addressed through the cutter-radius columns (`toolRadius…`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is one of the four columns of a machine whose compensation memory is split into geometry/wear and length/radius (type II; checked in our test environment against the Length, L wear, Radius and R wear cells on the operator panel). A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed for the same write call on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed on a simulator in this configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table; read the same value from the leaf of the same name under `/machine/toolArea/tool/toolEdge/…`.

## /machine/channel/toolOffset/toolRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **M-type tool radius geometry** value (the D column on the offset screen). The radius reference that cutter compensation (G41/G42) consults.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; on Mitsubishi that includes a machine whose compensation memory is **not split into columns**, where the error string points you at `toolOffsetValue` instead. The Fanuc adapter does not support `toolOffsetValue`, so no such pointer appears there; the FOCAS2 specification has the value of an offset memory without a length/radius split (memory A or B) addressed through the cutter-radius columns (`toolRadius…`), but we have not confirmed this on a machine with that configuration. On a Fanuc machining center with the tool offset for milling and turning function active, this address answers status `-20` for both read and write: in that configuration the offset columns are numbered differently (FOCAS2 specification), so this address would point at another column, and since we have not confirmed it on such a machine the address is blocked there. `toolLengthGeometry` and `toolLengthWear` are unaffected. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**The number can differ from what the machine's screen shows.** This value is a **radius**, but an offset screen may be configured to display and accept **diameters**. deemesh emits what the machine stores and does not convert.

**On Mitsubishi** this is one of the four columns of a machine whose compensation memory is split into geometry/wear and length/radius (type II; checked in our test environment against the Length, L wear, Radius and R wear cells on the operator panel). A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed for the same write call on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed on a simulator in this configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table; read the same value from the leaf of the same name under `/machine/toolArea/tool/toolEdge/…`.

## /machine/channel/toolOffset/toolRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **M-type tool radius wear** value (the D column on the offset screen). Reflects the radius reduction caused by tool wear.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; on Mitsubishi that includes a machine whose compensation memory is **not split into columns**, where the error string points you at `toolOffsetValue` instead. The Fanuc adapter does not support `toolOffsetValue`, so no such pointer appears there; the FOCAS2 specification has the value of an offset memory without a length/radius split (memory A or B) addressed through the cutter-radius columns (`toolRadius…`), but we have not confirmed this on a machine with that configuration. On a Fanuc machining center with the tool offset for milling and turning function active, this address answers status `-20` for both read and write: in that configuration the offset columns are numbered differently (FOCAS2 specification), so this address would point at another column, and since we have not confirmed it on such a machine the address is blocked there. `toolLengthGeometry` and `toolLengthWear` are unaffected. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**The number can differ from what the machine's screen shows.** This value is a **radius**, but an offset screen may be configured to display and accept **diameters**. deemesh emits what the machine stores and does not convert.

**On Mitsubishi** this is one of the four columns of a machine whose compensation memory is split into geometry/wear and length/radius (type II; checked in our test environment against the Length, L wear, Radius and R wear cells on the operator panel). A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed for the same write call on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed on a simulator in this configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table; read the same value from the leaf of the same name under `/machine/toolArea/tool/toolEdge/…`.

## /machine/channel/toolOffset/toolXGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type (lathe) X-direction tool dimension geometry** value. Here X is not an axis name but a fixed column on the offset screen, that is, a **directional component of the tool dimensions**.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. On Fanuc, the FOCAS2 specification has the value of an offset memory without a geometry/wear split (memory A) addressed through the wear columns (`…Wear`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is a column of a machine whose compensation memory is laid out for a lathe. A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed for the same write call on a simulator in a machining center configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table, and that table has lengths 1 to 3 (`/machine/toolArea/tool/toolEdge/toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns. Which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/channel/toolOffset/toolXWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type X-direction tool dimension wear** value. The X-direction compensation accumulated during machining.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. On Fanuc, the FOCAS2 specification has the value of an offset memory without a geometry/wear split (memory A) addressed through the wear columns (`…Wear`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is a column of a machine whose compensation memory is laid out for a lathe. A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed for the same write call on a simulator in a machining center configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table, and that table has lengths 1 to 3 (`/machine/toolArea/tool/toolEdge/toolLengthWear`, `toolLength2Wear`, `toolLength3Wear`) instead of X, Y and Z columns. Which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/channel/toolOffset/toolZGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type Z-direction tool dimension geometry** value. Like X, this is a fixed column on the screen, not an axis.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. On Fanuc, the FOCAS2 specification has the value of an offset memory without a geometry/wear split (memory A) addressed through the wear columns (`…Wear`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is a column of a machine whose compensation memory is laid out for a lathe. A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed for the same write call on a simulator in a machining center configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table, and that table has lengths 1 to 3 (`/machine/toolArea/tool/toolEdge/toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns. Which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/channel/toolOffset/toolZWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type Z-direction tool dimension wear** value.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. On Fanuc, the FOCAS2 specification has the value of an offset memory without a geometry/wear split (memory A) addressed through the wear columns (`…Wear`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is a column of a machine whose compensation memory is laid out for a lathe. A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed for the same write call on a simulator in a machining center configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table, and that table has lengths 1 to 3 (`/machine/toolArea/tool/toolEdge/toolLengthWear`, `toolLength2Wear`, `toolLength3Wear`) instead of X, Y and Z columns. Which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/channel/toolOffset/toolYGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type Y-direction tool dimension geometry** value: the **third column** after X and Z. Fanuc answers status `-20` on a lathe without the Y-axis offset option; we have not confirmed what a Mitsubishi lathe without a third axis answers.

**The screen heading for this column is not `Y` on every machine.** Mitsubishi assigns this slot to the third axis, so a C-axis lathe shows it as `C` on the control (the reference IB-1501209 likewise labels this column `C (Y*)` in an older edition and an additional axis in the latest one, without fixing the axis name). What the address promises is the **third offset column**; which axis that is, the column heading on the control tells you.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. On Fanuc, the FOCAS2 specification has the value of an offset memory without a geometry/wear split (memory A) addressed through the wear columns (`…Wear`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is a column of a machine whose compensation memory is laid out for a lathe. A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed for the same write call on a simulator in a machining center configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table, and that table has lengths 1 to 3 (`/machine/toolArea/tool/toolEdge/toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns. Which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/channel/toolOffset/toolYWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type Y-direction tool dimension wear** value. The **third column** after X and Z; the screen heading is not `Y` on every machine (a Mitsubishi C-axis lathe shows wear `C`). See `toolYGeometry` for details.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. On Fanuc, the FOCAS2 specification has the value of an offset memory without a geometry/wear split (memory A) addressed through the wear columns (`…Wear`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is a column of a machine whose compensation memory is laid out for a lathe. A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed for the same write call on a simulator in a machining center configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table, and that table has lengths 1 to 3 (`/machine/toolArea/tool/toolEdge/toolLengthWear`, `toolLength2Wear`, `toolLength3Wear`) instead of X, Y and Z columns. Which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/channel/toolOffset/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type nose radius geometry** value. Consulted by nose-radius compensation (G41/G42); together with the tip direction (`toolTipDirection`) it determines the tool-tip path.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. On Fanuc, the FOCAS2 specification has the value of an offset memory without a geometry/wear split (memory A) addressed through the wear columns (`…Wear`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is a column of a machine whose compensation memory is laid out for a lathe. A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed for the same write call on a simulator in a machining center configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table; read the same value from the leaf of the same name under `/machine/toolArea/tool/toolEdge/…`.

## /machine/channel/toolOffset/toolNoseRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The **T-type nose radius wear** value.

Returns `float` (a real distance); both read and write are supported; write `{"value": 125.0}`. Requires the `channel` + `toolOffset` filters. If the machine's offset screen has no such column, status `-20` is returned; that includes a machine whose compensation memory is not laid out for a lathe, where the error string names the leaves that do work there. On Fanuc, the FOCAS2 specification has the value of an offset memory without a geometry/wear split (memory A) addressed through the wear columns (`…Wear`), but we have not confirmed this on a machine with that configuration. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Read `/machine/channel/gModalCategory/gModal?gModalCategory=4` to find out which: `G21`/`G71`/`G710` means metric, `G20`/`G70`/`G700` means inch. On Siemens, `G70`/`G71` switch only coordinates while feedrates, tool offsets and work offsets stay in the basic system (`MD10240`); `G700`/`G710` switch those as well (Programming Manual). This address carries no `unit` field, because the unit is not fixed per address. **Fanuc fixes the decimal places of this value at connect, so reconnect after changing a unit setting such as `G20`/`G21`** (in the SDK `deemesh_disconnect` then `deemesh_connect`; on the hub `POST /admin/reload`). Until then it is read and written with the old decimal places and can be off by a factor of 10.

**On Mitsubishi** this is a column of a machine whose compensation memory is laid out for a lathe. A written value is rounded to the places of the setting unit `#1003`, and a value the control does not accept (outside its setting range) answers status `-16`. While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data and coordinate data) is off, the write answers status `-22` (machine state); turn the key on and write again (confirmed on a simulator in a lathe configuration; deemesh tells this case apart by reading that signal after the refusal). Writes are accepted during automatic operation as well (confirmed for the same write call on a simulator in a machining center configuration).

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table; read the same value from the leaf of the same name under `/machine/toolArea/tool/toolEdge/…`.

## /machine/channel/toolOffset/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["channel", "toolOffset"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
```

The lathe tool's **virtual tool-tip position code** (read + write). A code that determines, during nose-radius compensation (G41/G42), where the tool tip lies relative to the nose center; it is a position code, not an angle, and is returned/entered as an integer with no scaling (`{"value": 3}`). Requires the `channel` + `toolOffset` filters. **This column exists only in lathe-style offset memory**, so if the machine's offset screen has no such column the address refuses with status `-20` (not supported) and the error string lists the leaves that channel does have. On Mitsubishi that includes a machine whose compensation memory is not split into columns, where the error points you at `toolOffsetValue` instead.

- `1`–`8` = orientation, **`0`/`9` = the tool nose center is the reference point** (rather than the imaginary tip). The two values mean the same thing: the Fanuc 0i-F lathe manual (`B-64604EN-1/01`) defines `0` and `9` as the codes used when the tool nose center coincides with the reference point
- **`desc` is attached only to `0` and `9`**: the manual defines `1`–`8` by per-plane diagrams, and there is more than one diagram (per plane: `G17`/`G18`/`G19`), so the same number denotes different orientations depending on the setup. For the per-orientation reading follow that machine's manual
- Siemens equivalent concept: cutting edge position (`toolArea/tool/toolEdge/toolTipDirection`). **The two trees share the same numbering and the same `desc` vocabulary**; only the address differs, so the values can be compared and reused as-is
- **The accepted range differs by machine type** - Fanuc `0`-`9`, Siemens `1`-`9`, Mitsubishi `0`-`8`. The code for the center differs too (`0`/`9`, `9` and `0` respectively), so **reads are unified by `desc` but writes must stay inside that machine's range** (the `9` that Fanuc accepts is status `-16` on Mitsubishi)

**On Mitsubishi** a write, like the other lathe columns, answers status `-22` (machine state) while data protect key 1 (PLC signal `*KEY1`, `Y708`) is off (confirmed for the same write call on a simulator in a lathe configuration). A code outside the range is refused with status `-16` before reaching the machine.

**Siemens answers with status `-20`.** On Siemens compensation lives in the per-tool table, not in a channel offset-number table; read the same value from the leaf of the same name under `/machine/toolArea/tool/toolEdge/…`.

## /machine/channel/activeToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The number (`T`) of the tool that is currently active in that channel. `channel` filter. Returns `int`.

**What makes a tool "active" differs by machine type.** On Fanuc and Mitsubishi this is the `T` modal, so it changes **the moment `T` is programmed**; on Siemens it is `actTNumber`, so it changes **only once the change has completed** (so does Heidenhain; see the Heidenhain paragraph below):

| Situation | Fanuc, Mitsubishi | Siemens |
|---|---|---|
| `T7` programmed, `M06` not yet reached | `7` | **depends on how that machine changes tools** (below) |
| after `T7 M06` has completed | `7` | `7` |

⚠️ **On Siemens some machines complete the change on `T` alone.** Whether `M06` or `T` performs the change is set by the machine builder. On our 840D sl bench, programming only `T="CUTTER 10"` **completed the change without any `M06`**: this value changed at once, that tool's `toolLocationType` went from `magazine` to `buffer` (the spindle), and the tool it replaced went the other way. So do not read this value as "before the change". Whether a change happened is answered for certain by `/machine/toolArea/tool/toolLocationType`.

The first two were confirmed - the value reads `7` immediately after programming `T7` with no `M06`, and on Mitsubishi the control's own tool display was observed still showing the previous tool. A reset (`M30`) does not clear it either. On Fanuc this was checked again on **our 31i bench, which runs a real tool-change macro**: the dwell after a block holding only `T` already showed that number (with `M06` not yet executed), it was the same after the change completed, and a second `T` changed it at that block too. The value survived the program's `M30`.

**So on Fanuc and Mitsubishi this value must not be read as "the tool that is cutting right now"**; it is the commanded tool. If the moment of the change matters, do not use this address as a change signal. Watch the machine's own change-complete signal. The tool actually held in the spindle is not available from a named address on those two: the tool number shown on the control is produced by the machine builder's ladder, so it differs per machine, and it resists neutralization for the same reason `plcAddress` does. If you know the PMC address that holds the number, though, `/machine/plcAddress/plcType/plcValue` reads it. The `HD.T` (spindle tool) and `NX.T` (next tool) on a Fanuc control's screen are such values: they are shown instead of the T modal on a machine where parameters `3108#2` and `13200#1` are `1`, and the machine builder's ladder puts the numbers there through a PMC window function (PMC Programming Manual B-64513EN §5.4.26). Ask the machine builder for the PMC address where its ladder keeps them (confirmed on a 31i-B machine tool with that setting, where this address stayed the T modal, `0`).

⚠️ **With the Fanuc Tool Management option enabled, this value is not a tool number.** Under that option `T` does not name a tool: it names a **tool type (group) number**, and the control picks an actual tool of that type. Confirmed on our 31i bench: with the control's `EACH TOOL DATA` listing tool `1` as type `4`, programming `T4` made this address read `4` while the actual tool was `1`. On a machine without the option `T` is the tool number and the question does not arise. If your machine uses the option, do not feed this value straight into the tool tree lookup below. Instead, the entries of `/machine/toolArea/toolList` whose `toolTNumber` equals this value are the candidate tools, and `/machine/toolArea/tool/toolTNumber` answers it per tool.

⚠️ **Under Siemens tool management (WZV) the `T` in a part program names the tool.** The number this address reports (and the number the `tool` filter takes) is the control's **internal tool number**, not the text you type into a program. On our 840D sl bench `T3` raised alarm `17190` (illegal T number) while `T="CUTTER 10"` was accepted, and that tool's internal number was exactly `3`. The number-to-name pairing is given by `toolNumber` and `toolName` in `/machine/toolArea/toolList`.

**On Fanuc, even when a program selects a tool group through tool life management, this value is a tool number.** A program calls a group with a value greater than parameter `6810` (on a machine where `6810` is `1000`, `T1001` is group `1`), and the control picks the tool to use from that group and puts **that tool's number** here. Confirmed on a Fanuc machine tool: on a machine whose group `1` starts with tool `16`, commanding `T1001` made this read `16`, and it still read `16` after the tool change. A group number never appears in this value. We have not confirmed the value when a group is commanded through Mitsubishi tool life management.

The group whose life is currently being counted is reported by `/machine/channel/activeToolGroupNumber`, and that group's tools by `/machine/toolArea/toolGroup/toolNumberList`.

Put this number into the `tool` filter of the tool tree to look up that tool's name, edge count and offsets. The `toolArea` value you also need comes from `/machine/channel/toolAreaNumber` (cached at connection time, so it costs no extra communication).

Fanuc reads the `T` modal, Siemens `actTNumber` (`$P_TOOLNO`: the T number of the tool with which the active D offset was calculated), and Mitsubishi the T command modal from `GetCommand2`.

**Heidenhain** returns the tool number in the pocket table's spindle row (`0.0`, shown as `Spindle` on the control's tool management screen). Like Siemens, it changes **after the change has completed** (on the simulator it became `5` once `TOOL CALL 5` finished). An empty spindle is `0`, and when the spindle row is not found in the pocket table the status is `-20` (this address does not work on that machine). With an index tool (such as `10.1`) in the spindle it still gives only the tool number (`10`), because the spindle row of the pocket table holds only the number (confirmed in our test environment; the control showed `10.1`).

## /machine/channel/activeToolName
```yaml
value_type: "string"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

The **name** of the tool that is currently active in that channel. `channel` filter. Returns `string`.

**Siemens (`actToolIdent`) and Heidenhain report it.** Fanuc and Mitsubishi answer status `-20`.

- **Fanuc**: the tool management record has no name field (which is also why `toolName` in `/machine/toolArea/toolList` is an empty string on Fanuc). The tool geometry size data does carry `/machine/channel/toolOffset/toolName`, but that table is indexed by **tool offset number**, so resolving it to the active tool's name would mean inventing a correspondence that does not exist.
- **Mitsubishi**: the tool management table does have a name field, but that control's `activeToolNumber` is the `T` modal (the number that was commanded), which is not guaranteed to be a row in that table.
- **Heidenhain**: the tool table name of the tool in the spindle (the spindle row of the pocket table). It is the same field as `/machine/toolArea/tool/toolName`, so a renamed tool shows its new name at once. The pocket table has a name field too, but it did not follow a renamed tool, so it is not used (confirmed in our test environment). **When the tool in the spindle has index rows (such as `10.1`), the status is `-22`.** Each index row has its own name, but the spindle row of the pocket table holds only the tool number, so which row is in the spindle cannot be told (on the simulator the spindle row still read `10` after `10.1` was called, while the control showed `10.1`). The same applies when the tool's own row (`10`) was called, and the name comes back once a tool without index rows is in the spindle. `activeToolNumber` gives the number.

**It is the partner of `activeToolNumber`, and there is a reason for having both.** On a SINUMERIK with tool management (WZV) the `T` in a part program names the tool. Take the number and write `T3` and the control refuses it (alarm `17190`, illegal T number, on our 840D sl bench). This address gives the value you can put in a program; `activeToolNumber` gives the value that addresses our tool tree (the `tool` filter of `/machine/toolArea/tool/…`).

```
activeToolName   -> "CUTTER 10"   in a program: T="CUTTER 10"
activeToolNumber -> 3             toolArea/tool/*?tool=3
```

The two are in the same group, so asking for both costs one round trip (for the name Heidenhain also asks the tool table for its row list and for the tool's row).

**A name need not identify a tool uniquely.** A SINUMERIK tool is identified by its name together with its duplo (sister tool) number, so several tools can share a name; which one is used is the control's decision. When you need to point at exactly one, use `activeToolNumber`.

When no tool is active the value is an **empty string** (`activeToolNumber` answers `0` in that state). This was checked by unloading the tool from a channel on our 840D sl bench. There is no value there, and since this SDK does not report text with no content as `null`, it is normalised to an empty string. Heidenhain gives an empty string too when the spindle is empty (that state has not been confirmed).

## /machine/channel/activeToolEdgeNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_opcua_siemens"]
write: []
```

The number (`D`) of the edge whose compensation is **currently in effect** on the active tool. `channel` filter. Returns `int`. It is Siemens `actDNumber`. This address answers "which compensation set", not "is a tool loaded" - that is what `activeToolNumber` answers with `0`.

**Siemens only** (Fanuc, Mitsubishi and Heidenhain return status `-20`; on Heidenhain, deemesh has not found a way to ask which index of an index tool is in the spindle now). Neither the Fanuc nor the Mitsubishi offset model has a per-tool edge (compensation set) layer, so the question "which edge" does not arise. Earlier versions returned a fixed `1` on those two; that asserted a dimension that does not exist, so it was removed. On Fanuc the compensation numbers a program calls are read per tool from `/machine/toolArea/tool/toolHNumber` and `toolDNumber` (tool management option).

## /machine/channel/activeToolGroupNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["channel"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported). That is a **different option** from tool management, despite the similar name, and on every machine seen so far the two were not enabled together.

The tool group whose **life is currently being counted** in this channel. `channel` filter. Returns `int`, read only.

On a machine with tool life management, a program selects a group and that group's life starts counting down; this is that group's number. **`0` when no group is in use.**

Where `/machine/channel/activeToolNumber` is "the tool currently loaded", this is "the group whose life is being spent". They are different things, so read both.

**Siemens, Mitsubishi and Heidenhain answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

It differs from `/machine/channel/activeToolGroupNumber` in that it **keeps its value when nothing is running**:

| | A group is running | Nothing is running |
|---|---|---|
| `activeToolGroupNumber` | that group | `0` |
| this address | that group (same value) | **the last group that ran** |

That makes it the way to ask "which group did we use last" while the machine sits idle. It resets to `0` over a power cycle.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

## /machine/toolArea/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The **number of tools registered** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `int`, read-only. It is `0` when no tool is registered.

**On Mitsubishi this address is slow** (a little over 2 s on the NC Trainer2 plus simulator; it varies with the machine and the network). That control's tool management table has 999 slots that must be asked for one at a time, and a cleared slot leaves a **gap**, so the walk cannot stop early. **Do not poll it** - it is for drawing a screen once. When you need a single tool, the `/machine/toolArea/tool/…` addresses are far faster because they stop at the match. A communication error during the walk answers the error rather than what was read so far.

It equals the number of elements `toolList` returns and reads **the same value from the machine**; it exists so that you need not fetch the whole list when only the count matters. Measured on our Siemens 840D sl bench with 17 tools it is far cheaper than the list (81 ms vs 684 ms).

A nonexistent tool area is rejected with status `-18`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. Without the option there is no "tool" object at all, and the number of compensation registers is a different quantity, so it is not used instead. It counts only the **registered slots** of the tool management table (the `NO.` rows of the TOOL MANAGER screen; the slot count is parameter `13220`): the RGS bit of the tool information, the last letter `R` in the panel's `T-INFO` column. A slot whose registration is off is not counted even if values remain in it, because the control treats it as void data. The slot count `13220` is not the option ceiling but a machine-builder setting (within the 64/240/1000 pairs option range); on our bench 5 of 10 slots were registered.

**On Mitsubishi, a configuration whose tool management table cannot be read answers status `-20`** (the control refuses the read on a machine or project that does not use the table; the error carries the vendor code).

**Heidenhain** counts each tool number in the tool table (the tool management screen on the control) once. Index tools (rows such as `5.1` that follow a tool number) do not add tools; they count as that tool's edges (`/machine/toolArea/tool/toolEdgeCount`). Row `0` at the top of the table is not treated as a tool and is not counted. `toolArea` is `1` only.

## /machine/toolArea/toolList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
field_codes: {"toolLocationType": [{"value": "magazine", "name": "Magazine", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": "buffer", "name": "Spindle or tool changer", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": "loading", "name": "Load/unload position", "read": ["nc_opcua_siemens"]}, {"value": "none", "name": "No physical place", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]}]}
```

The list of **every tool registered** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `objectArray`, or an empty array `[]` when no tool is registered.

**On Mitsubishi this address is slow** (a little over 2 s on the NC Trainer2 plus simulator; it varies with the machine and the network). That control's tool management table has 999 slots that must be asked for one at a time, and a cleared slot leaves a **gap**, so the walk cannot stop early. **Do not poll it** - it is for drawing a screen once. When you need a single tool, the `/machine/toolArea/tool/…` addresses are far faster because they stop at the match. A communication error during the walk answers the error rather than what was read so far.

Element: `{"toolNumber": 16, "toolTNumber": null, "toolName": "BALLNOSE_D8", "toolEdgeCount": 4, "sisterToolNumber": 9, "magazineNumber": 0, "pocketNumber": 0, "toolLocationType": "buffer", "toolTeethCount": null, "toolBodyLength": null, "toolBodyDiameter": null, "toolOffsetNumber": null}`

Tool numbers are **sparse**: 17 tools may occupy numbers 2 through 18 with no number 1, so trying numbers from 1 upward tells you nothing about what exists. This list is the answer: take an element's `toolNumber` and put it straight into the `tool` filter to query the per-tool addresses.

**The order differs by machine type, but is the same on every read.** Fanuc, Siemens and Heidenhain sort by ascending `toolNumber`. The Siemens tool list screen is usually sorted by name and the operator can change the sort, so there is no single "screen order" to match, which is why we fixed the number order (to display it the way the screen does, sort by `toolName`). **Mitsubishi keeps the row order of the tool management table**: on that control the screen shows the table rows as they are, so this order matches the screen, and it may not be ascending tool number once a cleared row is refilled by a new tool later.

**`toolNumber` is not the `Loc.` (place number) on the machine's screen.** A machine using tool management identifies tools by name and sister number, so this number does not appear in the list screen (it is the `Tool number` field in the tool detail screen). Putting the screen's `Loc.` into the `tool` filter **queries a different tool and still looks successful**. The two numbers coincide for many tools, which makes it hard to notice. That value is `pocketNumber`, and this list carries both so you can check the correspondence.

- **toolNumber**: the tool number; the value to put in the `tool` filter
- **toolTNumber**: the number a program calls the tool with via `T` (Fanuc tool management's `TYPE NO.`; several tools may share it). `null` on Siemens, which calls tools by name, and on Mitsubishi, which has no such concept
- **toolName**: the tool's name; an empty string on Fanuc, which does not use names, and `null` on Mitsubishi, where deemesh does not read a name from the table
- **toolEdgeCount**: the number of offset data sets (not of physical teeth, and **not the highest D number either**, because deleting an edge in the middle leaves a gap, so numbers can exceed the count: see `/machine/toolArea/tool/toolEdgeCount`)
- **sisterToolNumber**: the sister-tool number (what tells apart tools sharing one name; the panel's `ST` column)
- **magazineNumber**: the magazine (tool store) it currently sits in; `0` when outside a magazine
- **pocketNumber**: the pocket inside that magazine; `0` when outside a magazine
- **toolLocationType** is the kind of place: `"magazine"` (in a magazine), `"buffer"` (spindle or tool changer), `"loading"` (load/unload position), `"none"` (no physical place)
- **toolTeethCount** · **toolBodyLength** · **toolBodyDiameter** · **toolOffsetNumber**: the per-tool columns of the Mitsubishi tool management table (see the identically named single addresses for their meaning). `null` on Fanuc, Siemens and Heidenhain (on Siemens and Heidenhain the tooth count is per cutting edge and is answered by `/machine/toolArea/tool/toolEdge/toolTeethCount`)

The list answers **what exists, what to call it, and where it is**. Measured values such as offsets and wear are per-edge and are not included.

When a value is unavailable the key is not dropped, it is `null` (on machine types that know the locations, the three location fields are the exception and carry the same values as the identically named single addresses; on Mitsubishi, which cannot see them, they are `null`). The list comes back in **the same order on every read**, so reading it twice and comparing is meaningful. A nonexistent tool area is rejected with status `-18`.

**The three location fields change whenever a tool moves**; the rest rarely change. This list is meant to be fetched once to draw a screen; if all you need is the currently active tool, use `/machine/channel/activeToolNumber` rather than polling the list (`activeToolNumber` is supported on every machine type; on Siemens and Heidenhain that value is the tool whose change has completed, and on a Fanuc with the tool management option it is the tool's type number, not a `toolNumber` of this list).

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It lists only the registered slots of the tool management table (RGS bit of the tool information, the last letter `R` of the panel's `T-INFO`); `toolNumber` is the slot number (the panel's `NO.` column, up to parameter `13220`). `toolName` is an empty string, and `toolEdgeCount` and `sisterToolNumber` are `null` because they are Siemens tool management concepts (edge layer, sister tool). The number a program calls with `T` is not this number but the tool's **type number** (the panel's `TYPE NO.`), carried as `toolTNumber` in each item. The three location fields carry the same values as the single addresses (`1` to `8` magazines, spindle and standby positions `"buffer"`, not loaded `"none"`; see `/machine/toolArea/tool/toolLocationType`). On a Fanuc without the option the offset table is filled densely from `1`, so there is nothing to enumerate.

**Mitsubishi lists the registered rows of its tool management table.** The table is separate for each part system (`toolArea`), unlike the magazines, which belong to the whole machine (confirmed on the simulator). Among the common keys, the concepts this control does not have or deemesh does not read (`toolTNumber`, `toolName`, `toolEdgeCount`, `sisterToolNumber`, and the three location fields, since its tool record does not carry its own location) are `null`; the other way round, the table's four columns (`toolTeethCount`, `toolBodyLength`, `toolBodyDiameter`, `toolOffsetNumber`) are filled only on this control (a column the control refuses keeps its key and is `null`, while a communication failure comes back as an error rather than `null`). To find which pocket a tool sits in, ask the magazine side through `/machine/toolArea/magazine/pocket/toolNumber` rather than this list.

**On Mitsubishi, a configuration whose tool management table cannot be read answers status `-20`** (the control refuses the read on a machine or project that does not use the table; the error carries the vendor code).

**Heidenhain** lists the tools in the tool table (the tool management screen on the control). `toolNumber` is the table's tool number and `toolTNumber` is the same value (a program's `TOOL CALL` calls the tool by this number; depending on the machine settings it can also call by name, according to the TNC7 User's Manual, 'Tool call by TOOL CALL'). `toolName` is the tool name (`NAME`) and `toolEdgeCount` is the tool's row count (its own row plus its index tool rows; see `/machine/toolArea/tool/toolEdgeCount`). `sisterToolNumber` is `null`; the replacement tool is answered by `/machine/toolArea/tool/sisterTool`. The three location fields come from the pocket table (see `/machine/toolArea/tool/toolLocationType`). The four Mitsubishi table columns are `null`. Row `0` at the top of the table is not treated as a tool and is not listed. The whole table is read at once; on the simulator (222 tools) it took around 0.5 seconds.

## /machine/toolArea/tool/toolExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]
```

Whether that tool number is **registered in the tool table**. `toolArea` + `tool` filters. Returns `boolean`. Reading is supported on all four controls; writing on all but Heidenhain.

Asking about a tool that does not exist is not an error. It answers `false`, because the address exists to ask that question. On Siemens, however, when the tool cannot be read for a reason other than being absent (for example the account used to connect may not read tool data, so `BadUserAccessDenied` comes back), the read answers status `-17`, and so does a write.

**Writing creates and deletes the tool.** `{"value": true}` creates it, `{"value": false}` deletes it. **Writing `true` to a tool that already exists is rejected with status `-21` (already exists)**: a silent success would let you believe you got an empty new tool and write over someone else's life data, offsets and location. Delete and recreate it, write its values through the individual addresses as-is, or skip it. Writing `false` to a tool that does not exist succeeds (the postcondition "absent" holds as it is, so resending after a lost response is safe).

**The tool number is exactly what you put in the `tool` filter**; the machine does not hand out the next free number. The number space has gaps (on our 840D sl bench, 20 tools occupied `2`-`18` and `100`-`102`, leaving `1` and `19`-`99` free) and **you can fill a gap by naming that number.** Pick one by looking at the `toolNumber` values in `/machine/toolArea/toolList`. Deleting a tool frees its number again, and the same number can be created later.

On Siemens a newly created tool is **empty and has one cutting edge**. Its name is the tool number as a string, and the **location type** is filled with the standard value: that field decides which magazine places the tool can go into, so leaving it unset means the operator panel finds no place to load it into, and deemesh therefore fills it with the same value the panel gives a new tool. The tool is also created with its **enable** flag on, as the operator panel does (the control does not pick a tool without it, so a `T` command for it is refused with an alarm; confirmed on an 840D sl bench). Until you set the tool type, it reads `9999` (not set) and the operator panel shows the type column as empty, but **unlike the location type this blocks neither loading nor use** (the two fields are easy to confuse because both use `9999`). Follow up with `/machine/toolArea/tool/toolName` for the name and `/machine/toolArea/tool/toolEdge/*` for the offsets; to add more edges use `/machine/toolArea/tool/toolEdge/toolEdgeExists`. This is the flow that registers presetter measurements without going to the machine panel. When creation is refused, the answer is status `-23` (no room) if the maximum number of tools is reached, and status `-18` if the control does not accept that tool number. If finishing the new tool fails (setting its location type or its enable flag), deemesh deletes the tool it has just created and returns the error (if it cannot delete it, the error text says the tool is still there and how to delete it).

**On Siemens and Fanuc a tool that is loaded anywhere cannot be deleted** (status `-18`). That covers not only a magazine pocket but also a buffer place such as the spindle or a gripper, plus the load/unload position on Siemens and the standby position on Fanuc. The physical tool would stay in the magazine while its record disappeared, throwing off the next tool change, and **deemesh has no address that would put a recreated tool back into that pocket.** Unload it at the machine first. Where it is right now is answered by `/machine/toolArea/tool/toolLocationType`. **On Siemens, when a deletion is refused because a channel is not in Reset or the tool is in use, deemesh answers status `-22` (machine state)** (840D sl bench: deleting a tool that had just been in the spindle while the channel was interrupted). Reset the channel and delete again. Any other refusal is usually status `-17`, with the return code in the error text (and its meaning where the manual gives one). **On Mitsubishi deemesh does not make this check** (that control's tool record does not carry its own location, so finding it means walking the whole magazine). On a machine with magazines, check which pocket the tool is in yourself with `/machine/toolArea/magazine/pocket/toolNumber` before deleting it.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the slot's registration mark (RGS bit of the tool information, the last letter `R` of the panel's `T-INFO`). Reading a number beyond parameter `13220` (the slot count) is `false`, not an error (a number above `32767`, however, answers status `-13`, and `0` or below status `-18`). A slot whose registration is off is void to the control even if values remain in it (Connection Manual B-64483EN-1: with RGS at 0 the data is regarded as not registered even when other items are set), so the other per-tool addresses (`toolHNumber`, life, location and so on) reject that slot with status `-18` (measured on our bench: switching the registration mark on made the same slot `true` at once and raised `toolCount` by one).

The Mitsubishi table is **a list of rows**, so a tool number is not a row number. `true` writes the number into the **first empty row** and `false` empties that row. The row number never appears in the address, so you do not have to care which slot it lands in. A newly created tool has `0` for the teeth count and the body dimensions, and **only the offset number is filled in by the control, with the tool number itself** (seen on the simulator). Set the rest through their own addresses. A deleted row is cleared in full by the control (the same as the panel's tool clear), so a tool created there later inherits no old values.

**On Mitsubishi, creating and deleting an absent tool walk the whole table** (a little over two seconds on the simulator). The table has 999 rows and clearing a middle row leaves a hole, so proving that a number is not there means reading to the end. Deleting a tool that is there stops once its row is found (about 0.2 s on the simulator). **These are not calls to repeat in a loop.** A duplicate number is refused by the control as well, but deemesh checks first and answers with status `-21`. If the table has no free row at all, the write is refused with status `-23` (no room). The value is not wrong; there is nowhere to put it, so delete a tool to free a row and request again. If someone else (the operator panel, for instance) takes that free row while the table is being read, nothing is written and the answer is status `-24` (busy); send the same request again. A communication error during the walk answers an error rather than `false` for a read, and an error with nothing written for a write. On a machine where data protect key 1 (PLC signal `*KEY1`, `Y708`) is off, a refused registration or deletion is answered with status `-22` (machine state) after deemesh reads that signal (we confirmed on a simulator that registration and deletion are refused with the key off).

Writing on Fanuc: `true` **registers** the slot (`cnc_regtool`). The slot comes out as an **empty record with only the registration mark set** (type number `0`, no life management, H/D/S/F `0`), so no `T` command can pick it. The control supplies no defaults and deemesh invents none; fill it in afterwards through `toolTNumber`, `toolHNumber`, `toolDNumber`, `toolLifeMonitorType` and the life addresses. A slot whose registration is off but still holds values (the panel's `T-INFO` shows `-` while other columns show data) is refused by the control as it stands, so deemesh clears it first (`cnc_deltool`) and then registers it; the leftover values are discarded. A slot beyond `13220` cannot be created (status `-18`). `false` **deletes** the slot (`cnc_deltool`): the whole record is cleared and the control also removes that tool number from the magazine table (Connection Manual B-64483EN-1). The tool data lock (`toolDataLockedOn`) does not block this deletion (measured on our bench). The slots after it are not pulled up; they stay where they are (confirmed on a 31i bench). Writing `false` to a slot beyond `13220` is not an error, as with reading.

**On Mitsubishi, a configuration whose tool management table cannot be read answers status `-20`** (the control refuses the read on a machine or project that does not use the table; the error carries the vendor code).

**Heidenhain** reports whether the tool table (the tool management screen on the control) has a row for that number. Reading only; writing is status `-20` (deemesh does not create or delete tools on Heidenhain). Row `0` at the top of the table is not treated as a tool, so it reads `false`. `toolArea` is `1` only.

## /machine/toolArea/tool/toolName
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The tool's name (SINUMERIK `toolIdent`). `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses).

Returns `string`; both read and write are supported. Write `{"value": "DRILL 10"}`. **Siemens and Heidenhain.** On Siemens machines that use tool management, a tool's identity is its name plus its sister-tool (duplo) number, so several tools may share one name. On machines that do not use names, an empty string is normal.

A nonexistent tool is rejected with status `-18`. This address does not create tools. Length and character limits are the machine's to judge, and violations surface as an error.

**Fanuc and Mitsubishi answer with status `-20`.** deemesh does not read a name from either control's tool management table (on Fanuc the tool name lives in the tool geometry size data and is answered by `/machine/channel/toolOffset/toolName`).

**Heidenhain** uses the tool name in the tool table (`NAME`), and writing is supported. A name the control does not accept is status `-16`. The TNC7 User's Manual ('Tool name') allows up to 32 characters, using capital letters, digits and `#` `$` `%` `&` `,` `-` `_` `.`, and says lowercase letters are turned into capitals when saved. In our test environment, too, names of 33 characters or more and names with a space (`DRILL 10`), `/`, `:`, Korean letters or umlauts were refused, and `abc_def` was stored as `ABC_DEF`. The write example above has a space, so on Heidenhain write it as `DRILL_10`, for example. Names need not be unique (manual; point at a tool by its number). This address does not cover the names of index tools (rows such as `5.1` that follow a tool number). Row `0` at the top of the table is not treated as a tool, so it is status `-18`.

## /machine/toolArea/tool/toolUseStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
codes: [{"value": 0, "name": "not managed", "read": ["nc_focas2_fanuc"]}, {"value": 1, "name": "unused"}, {"value": 2, "name": "in use"}, {"value": 3, "name": "life expired"}, {"value": 4, "name": "broken", "read": ["nc_focas2_fanuc"]}, {"value": 5, "name": "locked", "read": ["nc_opcua_siemens", "nc_dnc_heidenhain"]}]
```

The tool's **use status**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int` + `desc`. Reading and writing are supported on Siemens, Fanuc and Heidenhain. **This is per tool**: it takes no `toolEdge` filter (on Siemens a tool with several offset data sets has one status for the whole tool; a Heidenhain index tool has a status per row, which `/machine/toolArea/tool/toolEdge/toolUseStatus` answers).

The value is a machine-independent code defined by deemesh (not a vendor number). Each value is defined by **what state the tool is in right now**, and different machine types return the same value only when the tool is in the same state:

| Value | Meaning |
|---|---|
| `0` | Outside life management: the tool's life state is "not managed", so it is skipped by the type-number search (Fanuc only; `/machine/toolArea/tool/toolSearchedWhenUnmanagedOn` is the exception) |
| `1` | Unused: has never cut and is not locked |
| `2` | In use: has been used and is not locked |
| `3` | Life expired: the life is used up and the control will not use it |
| `4` | Broken: the control will not use it because it is broken (Fanuc only) |
| `5` | Locked: cannot be used for a reason other than its life. Life remains but the tool is locked (from the operator panel, the PLC, an NC program and so on), or the tool has no use permission, so the control will not choose it (Siemens and Heidenhain) |

With `3`, `4` or `5` the control will not use that tool: it rejects a program that calls for it, or switches to the sister tool if one is registered (`/machine/toolArea/tool/sisterToolNumber`; on Heidenhain the replacement tool, `/machine/toolArea/tool/sisterTool`). Some values occur on one machine type only, but whenever a value occurs it means the same thing. `desc` is prose for a human; branch on the value.

**Fanuc** carries the life state of the tool management data (the panel's `L-STATE`) over as it is: not managed `0`, unused `1`, usable `2`, life expired `3`, broken `4`. A tool the operator locks by hand at the panel is also called `life expired` (`3`) by the Fanuc control, so `5` never occurs. Supported only on machines with the tool management (TOOL MANAGEMENT) option; without it the address returns status `-20`. The LOC bit of the tool information (`/machine/toolArea/tool/toolDataLockedOn`) is a data edit lock and is not mixed in here.

**Siemens** derives it from the tool state bits (`toolState`) and the remaining life: with the lock bit (Disabled) clear, the "was in use" bit decides `1`/`2`, except that a tool whose enable bit (Enabled) is also clear is `5` (the control does not pick such a tool); with the lock bit set, the value is `3` when life monitoring is on and any cutting edge has `0` or less remaining, otherwise `5`. There is a single lock bit and it does not record who set it or why, so `5` says no more than "locked for a reason other than the tool's life". Reading a locked tool usually costs one extra round trip for the remaining life. Even when the edge numbers have a gap (only `D1` and `D3` left after `D2` was deleted, for example), deemesh finds the edges that exist and checks them all; if it cannot find as many edges as the tool has within edge numbers `1`-`240`, it does not make up a value and answers status `-17`. SINUMERIK's tool state has no broken flag, so `4` never occurs, and a tool with monitoring switched off is still selectable, so `0` never occurs either.

**Writing** names the state you want. If the tool is already in that state nothing happens and the write succeeds (for a value that machine type accepts for writing; the exceptions: on Siemens, writing `5` to a tool that reads `5` only because it has no use permission sets its lock bit, and on Heidenhain a tool whose used time has reached `TIME2` answers status `-16` even when it is already in that state). The writable values differ per machine type:

- Fanuc: `1`-`4` are written to the life state (`cnc_wrtool2`). Setting `3` may raise the tool change signal (`TLCH`) the moment every tool of the same type number has expired. A tool whose life state is "not managed" is rejected with status `-18` (set `/machine/toolArea/tool/toolLifeMonitorType` to `1`/`2` first). `0` is handled through `toolLifeMonitorType`, and `5` is status `-16` because Fanuc has no such state (write `3`).
- Siemens: `5` sets the lock bit; `1`/`2` clear the lock bit, clear/set the "was in use" bit respectively, and set the enable bit. `3` is a fact the control derives from the remaining life and cannot be written directly, so it is status `-16` (write `0` to `/machine/toolArea/tool/toolEdge/toolLifeRemaining`, or `5` to lock the tool). `4` and `0` are status `-16` as well. **Writing a life value re-evaluates the lock**: SINUMERIK re-evaluates the tool state whenever a monitoring value (`toolLifeTotal`, `toolLifeRemaining`, `toolLifeWarnLimit`) changes (Siemens Tool management Function Manual §8.11), so a tool locked with `5` is released by writing a life value. To keep the lock, write `5` again afterwards (confirmed on our bench). Whether writing a life value also releases a tool that is `5` only because it has no use permission is not yet confirmed.
- Heidenhain: `5` sets the tool table's lock (`TL`); `1`/`2` clear it. An unlocked tool reads `2` when it has used time and `1` when it has none, so only the value that matches is accepted, and a mismatch is status `-16` (to mark a tool unused, write `0` to `/machine/toolArea/tool/toolLifeUsed` first). `3` follows from the tool life, and `0` and `4` are states this control does not have, so they are status `-16`. A tool whose used time has reached `TIME2` reads `3` whatever the lock, so `1`, `2` and `5` are status `-16` for it as well (correct the used time first). Writing `5` to a tool past its maximum life (`TIME1`) sets the lock, and the tool then reads `3` (life expired), since a locked tool whose life is used up is expired. In our test environment locking a tool turned its row in the operator panel's tool table to the locked display at once.

**If a tool that became `3` because its life ran out is only written back to `2`, its remaining life (the counter on Fanuc) stays as it was**, so it can return to `3` when the control next looks at its life (on the SINUMERIK bench it still read `2` three seconds after the write). If you changed the insert, reset the life: on Fanuc reset `/machine/toolArea/tool/toolLifeUsed` first; on Siemens resetting `/machine/toolArea/tool/toolEdge/toolLifeRemaining` alone also releases the lock. Restoring the state without changing the insert also means cutting with a worn-out edge.

Specifying a nonexistent tool is rejected with status `-18`.

This address replaces `/machine/toolArea/tool/toolDisabledOn` (`boolean`) of 1.1.0. The old address is rejected with status `-12`; a tool whose `toolDisabledOn` read `true` corresponds to `3`, `4` or `5` here.

**Mitsubishi answers status `-20`.**

**Heidenhain** derives the status from the tool table's lock (`TL`) and life columns (maximum life `TIME1`, the limit at tool call `TIME2`, used time `CUR_TIME`). A tool whose used time has reached `TIME2` is `3` whatever the lock: in our test environment calling such a tool made the control refuse it with a "tool life expired" error (also when the used time equaled `TIME2`), and the lock in the tool table was not set (the TNC7 User's Manual, 'Tool table tool.t', also says that a tool past `TIME2` is not inserted when called). Otherwise, when locked, it is `3` if a maximum life is set and the used time has reached it, otherwise `5`; when not locked, it is `2` if there is used time and `1` if not. `0` and `4` do not occur. **Being past `TIME1` follows the control's lock**: a tool whose used time is past its maximum life reads `2` until it is locked (in our test environment such a tool was still inserted; the manual says this behaviour depends on the machine). According to the manual the control also locks a tool that exceeded a tolerance of automatic tool measurement; that cause is not kept in the tool table, so the status is `5`. To tell whether the life has run out, compare `/machine/toolArea/tool/toolLifeUsed` with `/machine/toolArea/tool/toolLifeTotal`. Writing is in the list above. This address carries the tool's own row. Each index tool (such as `320.1`) has a lock and a life of its own, which `/machine/toolArea/tool/toolEdge/toolUseStatus` answers (`toolEdge=0` gives the same value as this address).

## /machine/toolArea/tool/toolTNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc"]
```

The number a program uses to **call this tool with `T`**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`. Reading is supported on Fanuc and Heidenhain, writing on Fanuc. Write `{"value": 10}`.

It is the `10` in `T10 M06`, a sibling of `/machine/toolArea/tool/toolHNumber` (`H`) and `toolDNumber` (`D`): the numbers that correspond to the program's letters.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the tool type number of the tool management data (`TYPE NO.` on the TOOL MANAGER screen). **It is not a tool number but a group tag**: several tools may carry the same number, and when a program calls that number the control picks, among the valid tools carrying it, the one with the least life left (ties go to the spindle position, then the standby position, then the magazine, then the smaller tool number; Connection Manual B-64483EN-1). On our test bench tool `1` was `4` and tools `2` to `5` were all `10`. On a machine that gives every tool a different number it looks like a tool number, but that is only how that shop runs it. Writing changes this tool's type number (a whole number `0` to `99999999`, otherwise status `-16`): the tool joins or leaves the group of tools carrying that number, so the candidates a program's `T` picks from change. An unregistered tool is status `-18`.

**It pairs with `/machine/channel/activeToolNumber`.** On a tool-management machine the value that address reports is this number, so the entries of `/machine/toolArea/toolList` whose `toolTNumber` equals it are the candidate tools; `/machine/toolArea/tool/toolLocationType` answering `"buffer"` narrows down which one is actually in the spindle.

It is not the **kind** of tool (drill, end mill and so on). The kind is `/machine/channel/toolOffset/toolType` on Fanuc (the tool geometry size data of a separate option) and `/machine/toolArea/tool/toolEdge/toolType` on Siemens.

**Heidenhain** returns the tool number of the tool table as it is (the `10` in `TOOL CALL 10`, the same value as `toolTNumber` in the elements of `/machine/toolArea/toolList`). Unlike Fanuc it is one number per tool, and the number is the table row itself, so writing is status `-20`. `/machine/channel/activeToolNumber` gives the same number. According to the TNC7 User's Manual ('Tool call by TOOL CALL') a program can also call a tool by name, depending on the machine settings, but several tools can share a name ('Tool name'), so point at a tool by this number. For an index tool (such as `10.1`) this address also gives the tool's own number (the index is told by `toolEdge`). A nonexistent tool and row `0` at the top of the table are status `-18` (confirmed in our test environment).

**Siemens answers status `-20`**: that control calls tools by name, so the same place is `/machine/toolArea/tool/toolName`.

**Mitsubishi answers status `-20`.**

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

**Siemens and Mitsubishi answer status `-20`.** The Siemens ISO-dialect `H` is a number assigned to a cutting edge and lives at `/machine/toolArea/tool/toolEdge/toolHNumber`.

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

**Siemens and Mitsubishi answer status `-20`.** On Siemens the `D` number is not a radius compensation but the cutting edge number, so this address does not apply; read the radius there directly from `/machine/toolArea/tool/toolEdge/toolRadiusGeometry`.

## /machine/toolArea/tool/toolOffsetNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

The **offset number** this tool uses (**Mitsubishi only**: it is a column of that control's tool management table, so the other three answer status `-20`). `toolArea` + `tool` filters. Returns `int`, readable and writable.

**This is the link from a tool number to that tool's compensation values.** Put it straight into the `toolOffset` filter of `/machine/channel/toolOffset/…` and you get the geometry, wear and nose radius.

```
toolList -> tool 5  ->  toolOffsetNumber?tool=5 -> 5  ->  toolOffset/toolXGeometry?toolOffset=5
```

**It can differ from the tool number.** They match on many machines by coincidence, but they are separate values (seen on the simulator: changing the tool number to `7` left the offset number at `5`).

⚠️ **Writing this changes which compensation values apply to the tool.** It does not edit a value; it swaps which set of values is used, so writing it for a tool that is cutting moves the machine to different dimensions from that point on. `0` is accepted (the control takes it, and what it means follows the machine's setting). The upper bound is `/machine/channel/toolOffsetCount`; a larger number is refused by the control with status `-16` (invalid write value). While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data) is off, the control refuses writes to this field and the answer is status `-22` (machine state); turn the key on and write again (confirmed on the simulator; deemesh tells this case apart by reading that signal after the refusal. In the same table `toolBodyLength` and `toolBodyDiameter` were accepted with the key off). An unregistered tool number gives status `-18` (invalid filter value).

The Mitsubishi panel's tool management table carries two offset columns (`X5` / `Y5`), and **one write changes both** (seen on the simulator). That screen does not refresh on its own, so to check it you have to leave the screen and come back.

**Fanuc and Siemens answer with status `-20`.** Neither has a separate column linking tool number to offset number (Fanuc uses `toolHNumber` and `toolDNumber`; on Siemens the tool itself carries its compensation).

**On Mitsubishi, a configuration whose tool management table cannot be read answers status `-20`** (the control refuses the read on a machine or project that does not use the table; the error carries the vendor code).

## /machine/toolArea/tool/sisterToolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **sister-tool number** (SINUMERIK `duploNo`, the panel's `ST` column). It tells apart tools that share one name and decides which one is brought in once the preceding tool's life runs out. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses).

**It is not a sequence starting at `1`.** On our 840D sl bench, tools whose name was unique still carried `2`, `5` and `101` here. Do not sort by it or assume a `1` exists.

**A newly created tool starts with this equal to its tool number** (measured: a tool created as `50` came out with sister number `50`). If you intend to use sister tools, write the value you want here after creating it. Leaving it does no harm, but the numbers look haphazard once another tool of the same name is added later.

⚠️ **It is easy to confuse with the `D` column beside it on the panel.** `ST` says **which tool**, `D` says **which cutting edge** of that tool. Where several rows share a name, the same `ST` means several edges of one tool while different `ST` values mean different tools.

Returns `int`; both read and write are supported. Write `{"value": 2}`. **Siemens only.** On machines that use tool management, a tool's identity is its name plus this number, so tools sharing a name are told apart by it.

A nonexistent tool is rejected with status `-18`. A value that is not an integer, or outside `0`~`65535`, gives status `-16`. The effective upper limit comes from the machine configuration, and values outside that narrower range are rejected by the machine.

**Fanuc and Mitsubishi answer with status `-20`.** Sister tools, same-name tools told apart by number, are a Siemens tool management concept; on Fanuc replacement tools are handled by tool life management groups (`/machine/toolArea/toolGroup/…`).

**Heidenhain answers with status `-20`.** A Heidenhain replacement tool is not a number among same-named tools but a reference to another tool, which can be an index tool, so `/machine/toolArea/tool/sisterTool` answers it.

## /machine/toolArea/tool/sisterTool
```yaml
value_type: "object"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The **replacement tool** used in this tool's place. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `object`; both read and write are supported. Write `{"value": {"toolNumber": 320, "toolEdgeNumber": 1}}`.

The value is **two numbers that point at the replacement tool**, such as `{"toolNumber": 320, "toolEdgeNumber": 1}`. Put them as they are into the `tool` and `toolEdge` filters to read that tool. With no replacement tool it is `{"toolNumber": 0, "toolEdgeNumber": 0}`.

**Heidenhain only.** It is the tool table's replacement tool (`RT`, a tool-life field on the control's tool management screen). It can point at an index tool (a row such as `320.1` that follows a tool number), and `toolEdgeNumber` is that index (`320` is `0` and `320.1` is `1`, the same number as the `toolEdge` filter). The control keeps this as one decimal number (`320.1`), but deemesh returns it as two numbers: splitting a decimal into the tool number and the index can go wrong through binary rounding (the fraction of `5.3` becomes `0.2999…`). This address carries the tool's own row. Index tool rows have replacement tool fields of their own, which `/machine/toolArea/tool/toolEdge/sisterTool` answers (`toolEdge=0` gives the same value as this address).

**Writing**: give both keys as integers. Any other key, or a missing one, is status `-16` (ignoring a misspelled key would point at another tool, so it is not accepted). Writing `{"toolNumber": 0, "toolEdgeNumber": 0}` clears the replacement tool. The replacement tool must be in the tool table; the control does not accept a tool that is not there, which is status `-16`. Because the control keeps one decimal number, an index that cannot be told apart in that form (an index ending in `0`, such as `10`, `20` or `100`) is not sent and is status `-16`. Row `0` at the top of the table is not treated as a tool, so it is status `-18`.

Siemens, Fanuc and Mitsubishi answer with status `-20`. On Siemens the sister tool is the tool's own number among same-named tools, which `/machine/toolArea/tool/sisterToolNumber` answers.

## /machine/toolArea/tool/toolTeethCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

The tool's **number of teeth** (the count in "a 4-flute end mill"). `toolArea` + `tool` filters. Returns `int`, readable and writable.

The same name also exists under `toolEdge`. That one belongs to controls that hold the value **per cutting edge** (Siemens); this one belongs to controls that hold **one per tool** (Mitsubishi). The two are answered by different controls, so no machine answers both (on Fanuc both answer status `-20`, and on Siemens this address does).

On Mitsubishi this is `Num. of teeth` on the operator panel under `Setup > Tool management`. That screen does not refresh on its own, so to see a written value there you have to leave the screen and come back. Writing works only for a tool **already registered** in that table. An unregistered tool number is refused with status `-18` (invalid filter value), and `/machine/toolArea/toolList` tells you which numbers are there. A value beyond the range or the digit count the field accepts is refused by the control with status `-16` (invalid write value). While data protect key 1 (PLC signal `*KEY1`, `Y708`, which protects tool data) is off, the control refuses writes to this field and the answer is status `-22` (machine state); turn the key on and write again (confirmed on the simulator; deemesh tells this case apart by reading that signal after the refusal. In the same table `toolBodyLength` and `toolBodyDiameter` were accepted with the key off).

**On Mitsubishi, a configuration whose tool management table cannot be read answers status `-20`** (the control refuses the read on a machine or project that does not use the table; the error carries the vendor code).

## /machine/toolArea/tool/toolBodyLength
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

The **length of the tool body**. `toolArea` + `tool` filters. Returns `float`, readable and writable.

**This is not a compensation value.** It is a physical dimension of the tool, so it is used differently from `toolLengthGeometry`, which is added to coordinates; this one feeds interference checking and magazine allocation. The unit follows the machine's setting, so none is attached.

On Mitsubishi this is `Length :A` on the operator panel under `Setup > Tool management`. That screen does not refresh on its own, so to see a written value there you have to leave the screen and come back. Writing follows the same rules as `toolTeethCount` (registered tools only, out of range gives status `-16`). Data protect key 1 (`*KEY1`), however, does not guard this column (it was accepted with the key off on the simulator), so a refusal here answers status `-16` whatever the key. This column holds three decimal places whatever the setting unit (`#1003`), including values entered at the operator panel, so deemesh rounds to three places before sending and answers status `0` (with a 1 nm setting in our test environment, `12.345678` was stored as `12.346`). Read the value back to see what was stored.

**Fanuc and Siemens answer with status `-20`.** This column is specific to the Mitsubishi tool management table.

**On Mitsubishi, a configuration whose tool management table cannot be read answers status `-20`** (the control refuses the read on a machine or project that does not use the table; the error carries the vendor code).

## /machine/toolArea/tool/toolBodyDiameter
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_ezsocket_mitsubishi"]
write: ["nc_ezsocket_mitsubishi"]
```

The **diameter of the tool body** (not the radius). `toolArea` + `tool` filters. Returns `float`, readable and writable.

Same family as `toolBodyLength`, and no unit for the same reason.

On Mitsubishi this is `Diameter :B` on the operator panel under `Setup > Tool management`. That screen does not refresh on its own, so to see a written value there you have to leave the screen and come back. Writing follows the same rules as `toolBodyLength`.

**Fanuc and Siemens answer with status `-20`.** This column is specific to the Mitsubishi tool management table.

**On Mitsubishi, a configuration whose tool management table cannot be read answers status `-20`** (the control refuses the read on a machine or project that does not use the table; the error carries the vendor code).

## /machine/toolArea/tool/toolSpindleSpeed
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

The **spindle speed `S` stored per tool** in the tool management data. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int` + `unit:"rpm"`; both read and write are supported. Write `{"value": 1500}` (a whole number).

**The control does not apply it automatically when `T` is called.** The Operator's Manual (B-64484EN) describes it as a machining condition registered in the tool management data, one that can be specified directly by coding `S#8411` in a tool change macro (such as `M06`). It is a **reference value** that takes effect only where the machine builder or the operator wrote the change macro or the part program to run the tool under its registered conditions; on a machine not programmed that way the value sits there without effect. For the actual speed read `/machine/channel/spindle/spindleSpeedActual`, for the commanded one `spindleSpeedCommanded`.

We have a measured example. On our 31i bench, which does have a real change macro, we stored `1234` on a tool and ran an `M06` change, and the commanded S stayed at value `0` afterwards. Whether this value travels anywhere is entirely up to that machine's macro, so confirm it on the machine before relying on it.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the `S` field of the EACH TOOL DATA screen; the FOCAS valid range is `1` to `99999`, so `0` is a field nobody filled in (all `0` on our test bench). An unregistered tool is status `-18`. Writing accepts whole numbers `0` to `99999` only (otherwise status `-16`); any other rejection comes back with the vendor's reason: status `-22` when it is the control's own state (write protection, mode, running), status `-24` when another call is in progress, and status `-17` otherwise.

**Siemens and Mitsubishi answer status `-20`**: this field is an item of Fanuc tool management data, and deemesh does not map it to those two controls.

## /machine/toolArea/tool/toolFeed
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

The **cutting feed `F` stored per tool** in the tool management data. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`; both read and write are supported. Write `{"value": 250}` (a whole number).

**The control does not apply it automatically when `T` is called.** Like `/machine/toolArea/tool/toolSpindleSpeed`, it is a **reference value** the machine builder's change macro can read through `F#8412` (Operator's Manual B-64484EN); on a machine not programmed that way it has no effect. For the actual feed read `/machine/channel/feedActual`, for the commanded one `feedCommanded`.

We have a measured example. On our 31i bench, which does have a real change macro, we stored `567` on a tool and ran an `M06` change, and the commanded F did not change to it afterwards (the previous modal value stayed). Whether this value travels anywhere is entirely up to that machine's macro, so confirm it on the machine before relying on it.

**No `unit` is attached.** The FOCAS specification allows mm/min, inch/min, deg/min, mm/rev and inch/rev for this field, and which one a value means is decided by the macro that reads it (the same reason the other feed addresses carry no unit: it depends on machine settings).

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the `F` field of the EACH TOOL DATA screen; the FOCAS range is `0` to `99999999` (all `0` on our test bench). An unregistered tool is status `-18`. Writing accepts whole numbers `0` to `99999999` only (otherwise status `-16`); the unit is whatever the machine's macro expects.

**Siemens and Mitsubishi answer status `-20`**: this field is an item of Fanuc tool management data, and deemesh does not map it to those two controls.

## /machine/toolArea/tool/toolDataLockedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

Whether the tool's **tool management data is locked against editing**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `boolean`; both read and write are supported. Write `{"value": true}` to lock and `{"value": false}` to unlock.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the LOC bit of the tool information (`L`/`U` in the panel's `T-INFO`, "Data access: Locked/Unlocked" in the vendor documentation). **It does not mean the tool must not be used** (that is `/machine/toolArea/tool/toolUseStatus`): as the vendor documentation names it, it locks data access, and has no effect on the control's tool search or tool change; see below for what it blocks in practice.

**This lock does not block SDK writes.** On a 31i bench the life counter, the notice life, the H number and the spindle speed of a locked tool were all written successfully through FOCAS (with parameter `13204#0` and the operator panel's memory protect key in either position). **The lock alone does not block panel editing either.** With `13204#0` (TDL) at `0` a locked tool can still be edited on the panel; with `1` the tool management data protection signals `TKEY0` to `TKEY5` (`G330`) permit panel input item by item, whether the tool is locked or not (Connection Manual B-64483EN-1; on the bench, with all those signals at `0`, the panel showed `WRITE PROTECT`). While that tool's data is open for editing on the panel, the control refuses a write that changes this lock, and deemesh answers status `-17` with that reason; close the edit and write again.

The write changes only this bit of the tool information word and writes the other bits back as read. If the tool is already in that state nothing is written and the call succeeds. An unregistered tool is status `-18` for both read and write.

**Siemens and Mitsubishi answer status `-20`**: this lock is an item of Fanuc tool management data, and deemesh does not map it to those two controls.

## /machine/toolArea/tool/toolSearchedWhenUnmanagedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

Whether a tool **whose life is not managed is still included in the `T` search**. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `boolean`; both read and write are supported. Write `{"value": true}` / `{"value": false}`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. The value is the SEN bit of the tool information (`S`/`-` in the panel's `T-INFO`). When a program calls a type number with `T` the control picks one of the tools carrying that number, and **a tool whose life state is not managed (`L-STATE` `NO-MNG`) is normally left out of the candidates.** With this value `true` such a tool is included without its remaining life being checked (Connection Manual B-64483EN-1). It has no effect on a tool under life management (`/machine/toolArea/tool/toolLifeMonitorType` other than `0`).

The bit's meaning is taken from the Connection Manual and the Operator's Manual and was confirmed on our test bench (the panel shows the bit as `S`, and `cnc_wrtool2` sets and clears it).

The write changes only this bit of the tool information word and writes the other bits back as read. If the tool is already in that state nothing is written and the call succeeds. An unregistered tool is status `-18` for both read and write. While that tool's data is open for editing on the operator panel, this write may be refused like a `/machine/toolArea/tool/toolDataLockedOn` write, and the error text then says so (the bit sits in the same tool information item; the refusal has been confirmed on the 31i bench for the lock bit only). The flag belongs to the machine builder's operating policy, so check the machine's tool change procedure before changing it.

**Siemens and Mitsubishi answer status `-20`**: this flag is an item of Fanuc tool management data, and deemesh does not map it to those two controls.

## /machine/toolArea/tool/toolOversizedOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

Whether the tool is **bigger than one pocket**: a wide tool that needs the neighbouring pockets kept free. `toolArea` + `tool` filters. Returns `boolean`, **read-only**.

On Siemens this is the `Z` column on the machine's magazine screen. It answers only **whether** the tool exceeds one pocket, not how many it occupies.

**Writing is not supported.** Siemens stores the oversize not as one flag but as how many pockets the tool takes up to the left, right, above and below, so a bare `true` could not decide which direction and how far. Set oversize at the machine panel.

A nonexistent tool is rejected with status `-18`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the big-tool bit (BDT) of the tool information and is read-only. The neighbouring pockets an oversize tool occupies appear as `toolNumber` `0` in `/machine/toolArea/magazine/pocketList` (measured on our bench: setting the bit made it `true`).

**Heidenhain** reads the `ST` column of the pocket table, which marks a special tool such as an oversize tool (the TNC7 User's Manual, 'Pocket table tool_p.tch', has the pockets next to it locked through the lock column `L`). It is the value of the pocket that holds the tool, and for the tool in the spindle, of the pocket kept free for it. Because the flag lives with the pocket, **a tool that is not in a magazine reads `false`**. In our test environment, turning `ST` on for one pocket made only the tool in that pocket read `true`. Read only. If deemesh does not find the `ST` column in the pocket table, the status is `-20`.

**Mitsubishi answers status `-20`.**

## /machine/toolArea/tool/toolFixedLocationOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

Whether the tool is **assigned to a fixed location**: it always returns to the same pocket. `toolArea` + `tool` filters. Returns `boolean`; both read and write are supported (`{"value": true}` assigns it, `false` releases it).

When `true` the tool goes back to its own pocket after a tool change; when `false` the machine picks a free pocket. On Siemens this is the `L` column on the magazine screen.

If it is already in that state, nothing happens and the write succeeds. A nonexistent tool is rejected with status `-18`.

**Siemens and Heidenhain support it.**

**Heidenhain** keeps this flag not with the tool but in the `F` column (fixed pocket, which the TNC7 User's Manual, 'Pocket table tool_p.tch', describes as returning the tool to the same pocket every time) of the pocket table. deemesh therefore reads and writes it on the pocket that holds the tool; for the tool in the spindle it is the pocket kept free for it. **A tool without a pocket in the pocket table reads `false`** (a tool not in a magazine, or one in the spindle whose original pocket is not reserved; there is no pocket to keep a fixed location in), and a write answers status `-22`: put the tool into a magazine first. When the `F` column is not found in the pocket table the status is `-20`. How the pocket table is handled depends on the machine (TNC7 User's Manual, 'Configuring a tool': a machine manufacturer's function or an external tool management system may handle it). On such a machine a written value may be changed or may conflict with that system, so check the machine manual. Reading and writing were confirmed in our test environment with a tool in the magazine and with the tool in the spindle.

**Fanuc and Mitsubishi answer with status `-20`.** deemesh does not read this item from their tool data.

## /machine/toolArea/tool/toolLifeMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_opcua_siemens"]
codes: [{"value": 0, "name": "no monitoring", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]}, {"value": 1, "name": "time"}, {"value": 2, "name": "count", "read": ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi"]}, {"value": 3, "name": "wear", "read": ["nc_opcua_siemens"]}]
```

How the tool's life is **monitored**. `toolArea` + `tool` filters. Returns `int`. Reading and writing are supported on Siemens and Fanuc; Mitsubishi and Heidenhain support reading only. Write `{"value": 2}`.

| Value | Meaning | Unit of the life values |
|---|---|---|
| `0` | no monitoring | n/a |
| `1` | time: counts how long the tool actually cut | the control's own unit: minutes on Siemens, Mitsubishi and Heidenhain (`unit` is `"min"`), seconds on Fanuc (`unit` is `"s"`) |
| `2` | count: what is counted is up to the control (finished workpieces on Siemens, tool mountings or cuttings on Mitsubishi) | counts (`unit` is `"count"`) |
| `3` | wear: watches whether the offset has drifted to its limit | machine setting (mm/inch), so no `unit` is attached |

**These four are the same on every machine type**: they are deemesh's own values, not the machine's, so you can branch on them without knowing which machine you are attached to.

On Siemens the **tool picks one method, and the values are per cutting edge**. That is why this address takes no `toolEdge` filter while the three per-edge life values do. On Fanuc the life values are per tool as well: `/machine/toolArea/tool/toolLifeTotal`, `toolLifeUsed` and `toolLifeWarnLimit` are its partners.

On Siemens and Fanuc, when it is `0` the three life values are rejected with status `-18`: there is nothing to measure on that tool. To switch monitoring on, write the method here first, then put a budget in (Siemens `/machine/toolArea/tool/toolEdge/toolLifeTotal`, Fanuc `/machine/toolArea/tool/toolLifeTotal`; on Fanuc switching the life state (`L-STATE`) on in the tool management screen of the operator panel has the same effect). Heidenhain switches it on through the maximum life instead (below).

A Siemens machine may have several methods on **at once**. In that case this address answers with the first of time → count → wear, and the three life values follow the same order, so the method and the values never disagree. The answers to "does this need replacing" (`/machine/toolArea/tool/toolLifeWarnOn` and `/machine/toolArea/tool/toolUseStatus`) are always exact regardless of method.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is read from the tool management data (`cnc_rdtool`): a tool whose life state is `NO-MNG` (not managed) answers `0`, because the control does not count it even if life values are displayed; a managed tool answers `1` (time) or `2` (count) according to the life-type bit of its tool information. `3` (wear) does not exist in Fanuc tool management, so writing it is status `-16`. **Write rule**: `0` sets the life state to not managed (the life-type bit and the values stay); `1`/`2` set the life-type bit to time/count and, for a tool that was not managed, switch the state on as unused when the life counter is `0`, otherwise as life remaining. For a tool already managed only the type changes (the numbers are not converted when the type changes, so write the life values again). While that tool's data is open for editing on the operator panel, a write of `1` or `2` that changes the type may be refused like a `/machine/toolArea/tool/toolDataLockedOn` write, and the error text then says so (it writes the same tool information item; the refusal has been confirmed on the 31i bench for the lock bit only).

This address was previously named `/machine/toolArea/tool/toolMonitorType`; the old address keeps working as-is, but the documentation describes only this name.

**Mitsubishi** carries the method of a tool registered in tool life management (the `Mthd` column of the operator panel's `T-life group` screen, the rightmost of its three digits). Cumulative cutting time is `1`; cumulative mounting count and cumulative cutting count are both `2`, and `desc` says which (`"Cutting time"`, `"Mounting count"`, `"Cutting count"`; Instruction Manual IB-1501274). `0` and `3` do not occur. The matching life values are `/machine/toolArea/tool/toolLifeTotal` and `toolLifeUsed`. `toolArea` is the part system number; a tool not registered in tool life management answers status `-18`, and a part system whose tool life groups cannot be read answers status `-20`. EZSocket `FCSB1224W100-A9` or later is required: with an earlier version the answer is always status `-20`, before any other check, and installing that version or later makes it readable. When the life values the control returns are not laid out the way deemesh reads them (a part system that answers in a different layout), the answer is status `-20` as well (checked against the operator panel on the simulator).

**Heidenhain** has time monitoring only. It is `1` when the tool table's maximum life (`TIME1`) or the limit at tool call (`TIME2`) is greater than `0`, and `0` when both are `0` (the TNC7 User's Manual, 'Tool table tool.t', describes both as limits past which the tool is locked). Writing this address is status `-20`; monitoring is switched by writing the maximum life: write a value in minutes to `/machine/toolArea/tool/toolLifeTotal` to turn it on, and `0` to turn it off (deemesh does not write `TIME2`, so while that column holds a value, writing `0` leaves this at `1`). The matching life values are `/machine/toolArea/tool/toolLifeTotal` and `toolLifeUsed`. This address carries the tool's own row; whether an index tool (such as `320.1`) is monitored shows as `/machine/toolArea/tool/toolEdge/toolLifeTotal` answering status `-18` or not.

## /machine/toolArea/tool/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_dnc_heidenhain"]
```

The **whole life budget** (maximum life) allotted to that tool. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `float`; both read and write are supported (Mitsubishi reads only). Write `{"value": 20}`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the maximum tool life of the tool management data (`MAX-LIFE` on the TOOL MANAGER screen); the control counts the usage counter (`/machine/toolArea/tool/toolLifeUsed`) up from `0` towards this value, and the remainder is this value minus the usage. On Fanuc tool life belongs to the **tool, one per tool**, not to a cutting edge, so the address is per tool with no `toolEdge` filter.

**The unit is the control's own.** With the monitoring method (`/machine/toolArea/tool/toolLifeMonitorType`) set to time it is **seconds** (`unit` is `"s"`; the panel shows it as hours, minutes and seconds such as `4H 5M 6S`), with count it is `count`. It is not converted to minutes (the Siemens per-edge life is in minutes, so check the response's `unit`). A tool whose life is not managed (`toolLifeMonitorType` `0`) returns status `-18` even if values remain (write the method first).

Writing accepts only whole numbers in the unit of the read (a fractional value is status `-16`); a tool whose life is not managed is status `-18`. An unregistered tool is status `-18`.

**Mitsubishi** gives the life of the tool life management data (the `Life` column of the operator panel's `T-life group` screen). The unit follows the method (`toolLifeMonitorType`): **minutes** for cutting time (`unit` is `"min"`), `count` for the counts; `0` means no life limit (Instruction Manual IB-1501274). `toolArea` is the part system number; a tool not registered in tool life management answers status `-18`, and a part system whose tool life groups cannot be read answers status `-20`. EZSocket `FCSB1224W100-A9` or later is required: with an earlier version the answer is always status `-20`, before any other check, and installing that version or later makes it readable. When the life values the control returns are not laid out the way deemesh reads them (a part system that answers in a different layout), the answer is status `-20` as well (checked against the operator panel on the simulator).

**Siemens answers status `-20`.** The Siemens life is per cutting edge and lives at `/machine/toolArea/tool/toolEdge/toolLifeTotal`.

**Heidenhain** uses the tool table's maximum life (`TIME1`), in minutes (`unit` is `"min"`, shown as `TIME1 (min)` on the control's tool management screen). `0` means there is no maximum life, so reading is status `-18` (also for a tool that has only the limit at tool call `TIME2`, and then the error message names `TIME2`; see `toolLifeMonitorType`). **Writing is accepted even while monitoring is off**: writing a value turns monitoring on, and writing `0` turns it off. Only whole minutes are accepted (a fraction is status `-16`, because the control rounds it to a whole number: `1.5` became `2` in our test environment). A value the control does not accept is status `-16`, with the allowed range in the error message (`0` to `99999` on the simulator). Row `0` at the top of the table is not treated as a tool, so it is status `-18`. This address carries the tool's own row. Each index tool (such as `320.1`) has a life of its own, which `/machine/toolArea/tool/toolEdge/toolLifeTotal` answers (`toolEdge=0` gives the same value as this address).

## /machine/toolArea/tool/toolLifeUsed
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: ["nc_focas2_fanuc", "nc_dnc_heidenhain"]
```

The life the tool **has used so far** (the life counter). `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `float`; both read and write are supported (Mitsubishi reads only). Write `{"value": 0}`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the life counter of the tool management data (`L-COUNT` on the operator panel): while the tool is in the spindle the control **counts it up from `0` towards the maximum life** (`/machine/toolArea/tool/toolLifeTotal`) (Connection Manual B-64483EN-1: an incrementing counter, remainder = maximum minus counter). Subtract it from `toolLifeTotal` when you need the remainder. There is no separate remainder address because this is the value Fanuc provides, and it keeps the number identical to the screen (`L-COUNT`).

The unit is the same as `toolLifeTotal` (seconds `"s"` under time monitoring, `count` under count monitoring). A tool whose life is not managed (`toolLifeMonitorType` `0`) returns status `-18`.

**This is the address you write after changing an insert to reset the counter** (usually `0`). Writing accepts only whole numbers in the unit of the read (a fractional value is status `-16`); a tool whose life is not managed is status `-18`. A tool taken out of service for life-over needs its state restored as well as the counter (see `/machine/toolArea/tool/toolUseStatus`). An unregistered tool is status `-18`.

**Mitsubishi** gives the tool's usage (the `Used` column of the operator panel's `T-life group` screen). The unit is that of `toolLifeTotal` (minutes `"min"` for cutting time, `count` for the counts), and when the usage exceeds the life the tool's status becomes life reached (`2` of `/machine/toolArea/toolGroup/toolLifeStatusList`; Instruction Manual IB-1501274). `toolArea` is the part system number; a tool not registered in tool life management answers status `-18`, and a part system whose tool life groups cannot be read answers status `-20`. EZSocket `FCSB1224W100-A9` or later is required: with an earlier version the answer is always status `-20`, before any other check, and installing that version or later makes it readable. When the life values the control returns are not laid out the way deemesh reads them (a part system that answers in a different layout), the answer is status `-20` as well (checked against the operator panel on the simulator).

**Siemens answers status `-20`.** Siemens counts the remainder down, so read `/machine/toolArea/tool/toolEdge/toolLifeRemaining` there.

**Heidenhain** uses the tool table's current used time (`CUR_TIME`), in minutes (`unit` is `"min"`, shown as `CUR_TIME (min)` on the control's tool management screen); the control counts it up toward the maximum life (`/machine/toolArea/tool/toolLifeTotal`) (on the simulator it grew during feed blocks). When there is no life limit (`TIME1` and `TIME2` both `0`, `toolLifeMonitorType` is `0`), both reading and writing are status `-18`; write `toolLifeTotal` first. Writing accepts fractions, and the control rounds them to two decimal places (in our test environment `1.234` became `1.23` and `1.235` became `1.24`). Read the value back to see what was stored. The TNC7 User's Manual, 'Tool table tool.t', gives the input range as `0` to `99999.99` and says a change during program run applies to tool life monitoring at once. Used time that has reached `TIME2` (equal counts) makes `/machine/toolArea/tool/toolUseStatus` `3`; being past the maximum life (`TIME1`) follows the control's lock, so it does not by itself make it `3` (see that address). Row `0` at the top of the table is not treated as a tool, so it is status `-18`. This address carries the tool's own row. Each index tool (such as `320.1`) has a life of its own, which `/machine/toolArea/tool/toolEdge/toolLifeUsed` answers (`toolEdge=0` gives the same value as this address).

## /machine/toolArea/tool/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
```

The tool's **notice life** (warning limit). `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `float`; both read and write are supported. Write `{"value": 30}`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the notice life of the tool management data (`NOTICE-L` on the operator panel, "predictive tool life" in the FOCAS specification): when the remaining life (maximum life minus the counter) reaches this value or less, the control raises the tool life arrival notice signal (Connection Manual B-64483EN-1; whether that notice is per tool type or per tool is set by parameter `13200#3`, and, when it is per type, whether the last tool's remaining life or the sum over the tools of that type is compared is set by `13200#2`). With `0` no notice signal is output.

The unit is the same as `toolLifeTotal`. A tool whose life is not managed (`toolLifeMonitorType` `0`) returns status `-18`. Writing accepts only whole numbers in the unit of the read (a fractional value is status `-16`); a tool whose life is not managed is status `-18`, and an unregistered tool is status `-18`.

On Fanuc `/machine/toolArea/tool/toolLifeWarnOn` returns status `-20` (deemesh has not found a per-tool notice-reached flag to read); if you need it, compare `toolLifeTotal − toolLifeUsed` with this value.

**Siemens and Mitsubishi answer status `-20`.** The Siemens warning limit is per cutting edge and lives at `/machine/toolArea/tool/toolEdge/toolLifeWarnLimit`; per tool, the Mitsubishi adapter answers only the method, the total and the usage of the tool life values (`toolLifeMonitorType`, `toolLifeTotal`, `toolLifeUsed`; per group it also answers `/machine/toolArea/toolGroup/toolLifeStatusList`).

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

**Fanuc and Mitsubishi answer with status `-20`.** deemesh does not read a warning state from the Fanuc tool record (the warning value is `toolLifeWarnLimit`, the warning signal lives on the PMC side), and per tool, the Mitsubishi adapter answers only the method, the total and the usage of the tool life values (`toolLifeMonitorType`, `toolLifeTotal`, `toolLifeUsed`; per group it also answers `/machine/toolArea/toolGroup/toolLifeStatusList`).

## /machine/toolArea/tool/toolLocationType
```yaml
value_type: "string"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
codes: [{"value": "magazine", "name": "Magazine"}, {"value": "buffer", "name": "Spindle or tool changer"}, {"value": "loading", "name": "Load/unload position", "read": ["nc_opcua_siemens"]}, {"value": "none", "name": "No physical place"}]
```

**What kind of place** the tool is in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `string`, **read-only**. The value is its own meaning.

| Value | Meaning |
|---|---|
| `"magazine"` | sitting in a magazine (tool store) |
| `"buffer"` | in the spindle or the tool changer, cutting or being moved |
| `"loading"` | at the load/unload position |
| `"none"` | no physical place, only the tool data is registered |

**These four are the same on every machine type.** The machine gives the spindle, the changer and the load/unload position their own magazine numbers too (Siemens: the internal buffer magazine `9998` and loading magazine `9999`; Fanuc: spindle positions `11` to `14` and standby positions `21` to `24`, and on multi-path controls numbers such as `211` and `221` with the path number in the hundreds place), but those are vendor constants a caller has no reason to know. deemesh folds them into these four, so you can branch on the value without knowing which machine it is attached to.

`"buffer"` does **not** distinguish the spindle from a changer gripper. The machine treats both as the same place. If you need to tell them apart, overlay `/machine/channel/activeToolNumber`, but the comparison differs by machine type: on Siemens that value is the tool number after the change completed, so compare it directly; on a Fanuc machine with tool management it is a type number, so narrow the candidates through `/machine/toolArea/tool/toolTNumber` first and compare then.

**Writing is not supported.** Moving a tool is the job of a tool move, and changing this value alone would put the bookkeeping out of step with reality.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. Magazine numbers (`1` to `8` in the cartridge management table; Connection Manual B-64483EN-1 allows up to eight, of which `1` to `4` are the ones configured by parameters) are `"magazine"`, the spindle and standby positions are `"buffer"`, a tool loaded nowhere is `"none"`, and `"loading"` never occurs because Fanuc has no load/unload position. The spindle and standby positions are not configured on our test bench, so that part rests on the vendor documentation. On Siemens a machine without tool management has no magazine at all, so the value is always `"none"`.

**Mitsubishi answers status `-20`.**

**Heidenhain** looks the tool up in the pocket table: `"buffer"` in the spindle row (`0.0`), `"magazine"` in a magazine row, and `"none"` when it is not in the table. A tool in the spindle also keeps its number in its original pocket (the spot is held), but it answers `"buffer"`. deemesh does not tell loading positions apart on Heidenhain, so `"loading"` does not occur.

## /machine/toolArea/tool/magazineNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

The number of the **magazine (tool store)** the tool currently sits in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

When the tool is not in a magazine the value is `0`: it is cutting in the spindle, being carried by the changer, sitting at the load/unload position, or registered as data with no physical place. The machine gives such places its own numbers too (Siemens: the internal buffer magazine `9998` and loading magazine `9999`; Fanuc: spindle positions `11` to `14` and standby positions `21` to `24`, and on multi-path controls numbers such as `211` and `221` with the path number in the hundreds place), but deemesh does not emit those and folds them to `0`. They are vendor constants a caller has no reason to know.

**This is where the tool is now, not where it belongs.** Once the tool is loaded into the spindle the value becomes `0`, and it gets a number again when it returns to the magazine. Which place it originally came from is not what this address answers.

**Writing is not supported.** This value records the physical location, so deemesh does not open it for writing. A record that disagrees with the physical tool could affect later tool changes; move tools through the machine's own tool-management procedure.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the magazine number of the tool management data (`MG` on the operator panel); the spindle and standby positions are not magazines and fold to `0` (`/machine/toolArea/tool/toolLocationType` is `"buffer"`). The spindle and standby positions are not configured on our test bench, so that part rests on the vendor documentation. On Siemens a machine without tool management has no magazine at all, so the value is always `0`.

**Mitsubishi answers status `-20`.**

**Heidenhain** returns the magazine number of the magazine row holding the tool in the pocket table. In the spindle (`toolLocationType` is `"buffer"`) or not in the table it is `0`. The number kept in the original pocket while the tool is in the spindle is not where the tool is now; `/machine/toolArea/tool/originalMagazineNumber` answers it.

## /machine/toolArea/tool/pocketNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

The number of the **pocket inside the magazine** the tool sits in. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

If the magazine number is the apartment building, this is the unit number. Both are needed to pin down a location. When the tool is not in a magazine the value is `0` (cutting in the spindle, being carried by the changer, at the load/unload position, or with no physical place). Numbers the machine gives to such places (e.g. `9998`) are not emitted.

**A turret's stations appear as pockets too.** The machine models turret positions as magazine pockets, so the position a machinist calls "station 3" is pocket `3` here.

**This is where the tool is now, not where it belongs.** Once the tool is loaded into the spindle the value becomes `0`, and it gets a place number again when it returns to the magazine.

**Writing is not supported.** This value records the physical location, so deemesh does not open it for writing. A record that disagrees with the physical tool could affect later tool changes; move tools through the machine's own tool-management procedure.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. It is the pot number of the tool management data (`POT` on the operator panel), and `0` while the tool is in a spindle or standby position. On Siemens a machine without tool management has no magazine at all, so the value is always `0`.

**Mitsubishi answers status `-20`.**

**Heidenhain** returns the pocket number of the magazine row holding the tool in the pocket table. In the spindle (`toolLocationType` is `"buffer"`) or not in the table it is `0`. The number kept in the original pocket while the tool is in the spindle is not where the tool is now; `/machine/toolArea/tool/originalPocketNumber` answers it.

## /machine/toolArea/tool/originalMagazineNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

The number of the magazine the tool **returns to**. `toolArea` + `tool` filters. Returns `int`.

**It is the partner of `magazineNumber`.** That one gives where the tool **is now**, this one where it **belongs**. While the tool sits in a magazine the two agree; **they part when it is in a spindle or a gripper**: `magazineNumber` then reads `0` (meaning it is outside a real magazine, and `toolLocationType` says where it is instead) while this address still points at its magazine. This is `Orig. magazine` on the operator panel's tool detail.

Measured, for one tool in a magazine and then in the spindle:

```
                          in magazine    in spindle
magazineNumber                  1              0
pocketNumber                    3              0
originalMagazineNumber          1              1
originalPocketNumber            3              3
toolLocationType           "magazine"      "buffer"
```

**What it is for**: it tells you where a tool now in the spindle will go back to, which is what tool-change planning and magazine housekeeping need. Where it is right now does not answer that.

A tool with no place assigned reads `0` (the same rule as `magazineNumber`). Asking about a tool that does not exist is status `-18`.

**Siemens and Heidenhain** (`toolMyMag` on Siemens).

**Fanuc and Mitsubishi answer with status `-20`.** deemesh does not read an original place on either control (on Fanuc, once the tool is in the spindle or a standby place, `magazineNumber` and `pocketNumber` read `0` and only `toolLocationType` says so).

**Heidenhain** looks it up in the pocket table. For a tool in the spindle the pocket table keeps its number in the original pocket to hold the spot, so this is that spot's magazine number (the same shape as the Siemens table above, confirmed with `TOOL CALL` on the simulator); for a tool in the magazine it equals `/machine/toolArea/tool/magazineNumber`, and a tool not in the table gives `0`. The TNC7 User's Manual ('Pocket table tool_p.tch') describes reserving that pocket while the tool is in the spindle as the behaviour of a box magazine; with a setup that does not reserve it, or for a tool inserted by hand, the value can be `0` while the tool is in the spindle (not confirmed in our test environment).

## /machine/toolArea/tool/originalPocketNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

The number of the pocket the tool **returns to**. `toolArea` + `tool` filters. Returns `int`.

It pairs with `originalMagazineNumber` to give the full place. The rules are the same: while the tool is in a magazine this matches `pocketNumber`, and once it moves to a spindle or gripper `pocketNumber` becomes `0` while this keeps the original pocket. This is `Orig. location` on the operator panel's tool detail.

A tool with no place assigned reads `0`; a tool that does not exist is status `-18`.

**Siemens and Heidenhain** (`toolMyPlace` on Siemens).

**Fanuc and Mitsubishi answer with status `-20`.** deemesh does not read an original place on either control (on Fanuc, once the tool is in the spindle or a standby place, `magazineNumber` and `pocketNumber` read `0` and only `toolLocationType` says so).

**Heidenhain** looks it up in the pocket table. For a tool in the spindle the pocket table keeps its number in the original pocket to hold the spot, so this is that spot's pocket number (the same shape as the Siemens table above, confirmed with `TOOL CALL` on the simulator); for a tool in the magazine it equals `/machine/toolArea/tool/pocketNumber`, and a tool not in the table gives `0`. The TNC7 User's Manual ('Pocket table tool_p.tch') describes reserving that pocket while the tool is in the spindle as the behaviour of a box magazine; with a setup that does not reserve it, or for a tool inserted by hand, the value can be `0` while the tool is in the spindle (not confirmed in our test environment).

## /machine/toolArea/magazineCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The **number of magazines** in that tool area. `toolArea` filter (`toolArea` is the tool area number the channel uses). Returns `int`, **read-only**.

**Only real tool stores are counted.** Internally Siemens treats the spindle, the changer and the load/unload position as magazines too (counted that way our 840D sl bench answers `3`), but deemesh does not treat those places as magazines. When a tool sits in one, `/machine/toolArea/tool/toolLocationType` answers `"buffer"` / `"loading"`.

**It gives you the count but not the numbers.** The numbers need not be contiguous, so you cannot infer valid magazine numbers from this value. Use `/machine/toolArea/magazineList` when you need the numbers. This address exists so you do not have to fetch the whole list just to get a count.

On Siemens a machine without tool management has no magazine at all, so the value is `0`.

**On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. No magazine data exists without the option, so the address does not answer `0`. The Fanuc magazine configuration is defined by parameters `13222`/`13227`/`13232`/`13237` (pocket counts of magazines `1` to `4`), and only those with a non-zero count are counted (the cartridge management table allows numbers `1` to `8`, but these four are the ones configured by parameters). The spindle positions (`11` to `14`) and standby positions (`21` to `24`) carry magazine numbers but are not magazines, so they are not counted. `toolArea` is a path number, but the Fanuc tool management table is CNC-wide, so any path gives the same value.

**On Mitsubishi the magazine numbers are a fixed `1`-`5` range**, and only those with at least one pocket are counted. The spindle and the standby positions are a separate concept on that control, not magazines, so they never enter this count. `toolArea` accepts `1` up to the channel count; magazines belong to the whole machine, not to a part system, so every value gives the same answer.

**Heidenhain** counts the magazine numbers (`1` and up) in the row names `magazine.pocket` of the pocket table (the `MAGAZIN` and `P` columns on the control's tool management screen). The spindle row (`0.0`) is not a magazine and is not counted. A control that answers it has no pocket table gives `0`. `toolArea` is `1` only.

## /machine/toolArea/magazineList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The **list of magazines** in that tool area. `toolArea` filter. Returns `objectArray`, **read-only**.

**Do not assume magazine numbers are contiguous: use this list.** On Siemens the internal magazine numbers on our 840D sl bench were `1`, `9998` and `9999`, and only the real tool store `1` appears in this list (on Mitsubishi only the magazines among `1`-`5` that have pockets appear, so numbers can be skipped. On Mitsubishi `toolArea` accepts `1` up to the channel count; magazines belong to the whole machine, not to a part system, so every value gives the same answer.) On Fanuc only the configured ones among the parameter-defined `1` to `4` appear (magazines whose pocket count in parameters `13222`/`13227`/`13232`/`13237` is non-zero; for a matrix-type magazine, parameter `13240`, `pocketCount` is rows x columns, `13241` x `13242` and so on). **On Fanuc this is supported only on machines with the tool management (TOOL MANAGEMENT) option**; without it the address returns status `-20`. Counting from `1` up to `/machine/toolArea/magazineCount` will not find them.

Each entry:

| Field | Meaning |
|---|---|
| `magazineNumber` | the magazine number; pass it straight to the `magazine` filter of `/machine/toolArea/magazine/pocketCount` |
| `pocketCount` | how many pockets that magazine holds |

**Only real tool stores are listed.** Internally Siemens treats the spindle, the changer and the load/unload position as magazines too and gives them fixed numbers (`9998` for the buffer, `9999` for loading), but in deemesh "magazine" means a tool store and nothing else. When a tool sits in one of those places, `/machine/toolArea/tool/toolLocationType` answers `"buffer"` / `"loading"` and `/machine/toolArea/tool/magazineNumber` gives `0`.

**It joins directly with tools.** Look up the entry by whatever `/machine/toolArea/tool/magazineNumber` returned, and pass that number straight to `pocketCount`.

**Magazine names are not included.** The machine has a name field, but it means nothing unless it is configured on site. On our 840D sl bench the 40-pocket magazine, the buffer and the load position all returned **the same string**. Three entries with identical names would make the list look broken, so it is left out.

A machine with no magazine configured answers `[]`; a Fanuc control without the tool management option answers with status `-20`.

**Heidenhain** gives one entry per magazine number in the pocket table, and `pocketCount` is that magazine's row count (on the simulator: `[{"magazineNumber": 1, "pocketCount": 50}]`). The spindle row (`0.0`) is not listed.

## /machine/toolArea/magazine/pocketCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The number of **places (pockets)** in that magazine. `toolArea` + `magazine` filters (`magazine` is the magazine number). Returns `int`, **read-only**.

**Magazine numbers are not contiguous**: `/machine/toolArea/magazineList` tells you the valid ones. Counting up from `1` to `/machine/toolArea/magazineCount` will not find them.

A number that does not exist is rejected with status `-18`. **The numbers of the spindle, the changer and the load/unload position are rejected too**. The machine gives those places magazine numbers, but deemesh does not treat them as magazines. Range and comma expansion are supported.

**Fanuc**: supported only on machines with the tool management (TOOL MANAGEMENT) option, otherwise status `-20`; a magazine number that is not configured is status `-18` (with the list of configured numbers). The value is parameter `13222`/`13227`/`13232`/`13237`, or rows x columns for a matrix-type magazine (parameter `13240`). Pocket numbers do not start at `1` but at the magazine's **start pot number** (parameter `13223` and so on), so take the pocket range from `/machine/toolArea/magazine/pocketList`.

**Mitsubishi note**: magazine numbers are a fixed `1`-`5` range, so anything outside it is status `-18`, but **a magazine inside the range that does not actually exist answers `0` instead of being rejected**, because on this control the existence probe is the pocket-count query itself. When you need existence, read `/machine/toolArea/magazineList`. `toolArea` accepts `1` up to the channel count; magazines belong to the whole machine, not to a part system, so every value gives the same answer.

**Heidenhain** returns the magazine's row count in the pocket table. Pocket numbers are the row names as they are, so there is no guarantee they run from `1` to this count (on the simulator the 50 rows are `1` to `41` and `50` to `58`). Check the numbers with `/machine/toolArea/magazine/pocketList`. `magazine=0` is the spindle row, so it is status `-18`.

## /machine/toolArea/magazine/pocketList
```yaml
value_type: "objectArray"
null_able: false
required_filters: ["toolArea", "magazine"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

**Every pocket in that magazine and what sits in each one.** `toolArea` + `magazine` filters. Returns `objectArray`, **read-only**.

Each entry:

| Field | Meaning |
|---|---|
| `pocketNumber` | the pocket number; every pocket appears. On Siemens and Mitsubishi they run from `1` to `/machine/toolArea/magazine/pocketCount`, on Fanuc from the magazine's start pot number (parameter `13223` and so on, usually `1`) for as many pockets as it has, on Heidenhain as the pocket table's rows (which can skip) |
| `toolNumber` | the tool in that pocket. **`0` means the pocket is empty** |

**This is the direction the other addresses cannot answer.** `/machine/toolArea/tool/pocketNumber` tells you which pocket a tool is in, but "what is in pocket N" and "where are the empty pockets" are answered only by this list.

Tool names and offsets are not included: `/machine/toolArea/toolList` gives those by number, so join on the number. Reading a name for every pocket would double the values fetched for a 40-pocket magazine.

The numbers of the buffer (spindle and changer) and the load/unload position are rejected with status `-18`: deemesh does not treat those places as magazines.

**Fanuc**: supported only on machines with the tool management (TOOL MANAGEMENT) option (otherwise status `-20`); the magazine management table (`cnc_rdmagazine`) is read one whole magazine at a time. `toolNumber` is the tool management data number (the `NO.` column on the operator panel, the value you pass as the `tool` filter). A neighbouring pocket occupied by an oversize tool and a reserved home pocket (oversize-tool support option and extension B option respectively) hold no tool either and appear as `0`, so when picking an empty place for a tool also check the magazine screen on the operator panel. A matrix-type magazine (parameter `13240`) is numbered the same way, from the same start pot number for rows x columns pockets (Fanuc Connection Manual B-64483EN-1: numbered from the upper-left to the lower-right corner as seen from the front of the magazine). Our test bench is chain-type, so the matrix case rests on the documentation.

**Mitsubishi**: magazine numbers are a fixed `1`-`5` range, and a magazine that does not actually exist is rejected with status `-18`. `toolArea` accepts `1` up to the channel count; magazines belong to the whole machine, not to a part system, so every value gives the same answer.

A magazine with no pockets answers `[]` on Siemens. Fanuc and Mitsubishi count only magazines that have pockets, so such a number is refused with status `-18`.

**Heidenhain** lists the magazine's rows of the pocket table (the `MAGAZIN` and `P` columns on the control's tool management screen) in pocket number order. `toolNumber` is the row's tool number (`T`), and **the original spot of a tool now in the spindle is `0`**: when a tool goes into the spindle, the pocket table keeps its number in the original pocket to hold that spot (confirmed with `TOOL CALL` on the simulator), but no tool is actually in it. The tool side gives that spot through `/machine/toolArea/tool/originalPocketNumber`. Pocket numbers are the row names as they are and can skip (on the simulator `1` to `41` and `50` to `58`). `magazine=0` is the spindle row, so it is status `-18`. A `toolNumber` of `0` does not mean a tool can go there: the pocket may be locked (`/machine/toolArea/magazine/pocket/pocketDisabledOn`) or, according to the manual ('Pocket table tool_p.tch'), blocked as the neighbour of a special tool or, in a box magazine, by the columns that lock the pockets above, below, left or right (`LOCKED_*`).

## /machine/toolArea/magazine/pocket/toolNumber
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_focas2_fanuc", "nc_opcua_siemens", "nc_ezsocket_mitsubishi", "nc_dnc_heidenhain"]
write: []
```

The **tool number** in that pocket. `toolArea` + `magazine` + `pocket` filters. Returns `int`, **read-only**.

**`0` means the pocket is empty** (tool numbers start at `1`). A pocket that does not exist is rejected with status `-18` rather than answering `0`, so the two never blur together. The valid range is given by `/machine/toolArea/magazine/pocketCount`. On Fanuc pocket numbers run from the magazine's start pot (parameter `13223` and so on), so take the range from `/machine/toolArea/magazine/pocketList`.

Use it for a single pocket; to sweep a whole magazine, `/machine/toolArea/magazine/pocketList` does it in one request.

The numbers of the buffer (spindle and changer) and the load/unload position are rejected with status `-18`.

**Fanuc**: supported only on machines with the tool management (TOOL MANAGEMENT) option, otherwise status `-20`. The value is the tool management data number (what you pass as the `tool` filter); a neighbouring pocket occupied by an oversize tool and a reserved home pocket appear as `0` too (see `/machine/toolArea/magazine/pocketList`). A matrix-type magazine works the same way (see `/machine/toolArea/magazine/pocketList` for the pocket numbering).

**Mitsubishi**: `toolArea` accepts `1` up to the channel count; magazines belong to the whole machine, not to a part system, so every value gives the same answer.

**Writing is not supported.** Overwriting the tool in a pocket would change the bookkeeping while the physical tool stayed put, and the changer would then reach for the wrong pocket at the next tool change. Moving a tool is the job of a magazine command, and deemesh does not expose one.

**Heidenhain** returns the row's tool number (`T`) in the pocket table. The original spot of a tool now in the spindle is `0` (see `/machine/toolArea/magazine/pocketList`). A pocket number that is not in the table is status `-18`, and `magazine=0` is the spindle row, so it is status `-18`. A `toolNumber` of `0` does not mean a tool can go there: the pocket may be locked (`/machine/toolArea/magazine/pocket/pocketDisabledOn`) or, according to the manual ('Pocket table tool_p.tch'), blocked as the neighbour of a special tool or, in a box magazine, by the columns that lock the pockets above, below, left or right (`LOCKED_*`).

## /machine/toolArea/magazine/pocket/pocketDisabledOn
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "magazine", "pocket"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

Whether the pocket is **marked as not to be used**: damaged, or a place that must stay empty. `toolArea` + `magazine` + `pocket` filters. Returns `boolean`; both read and write are supported (`{"value": true}` disables it, `false` enables it).

When `true` the machine skips this pocket when choosing where to put a tool. On Siemens this is the `D` column on the magazine screen, and on that screen the cell appears **only on rows for tools that occupy a pocket**, because it is a property of the pocket, not of the tool. On Heidenhain it is the pocket table's lock field (`L`, below).

Locking a tool is `/machine/toolArea/tool/toolUseStatus` (value `5`), which is separate. A disabled pocket does not disable the tool sitting in it; move that tool elsewhere and it is usable again.

If it is already in that state, nothing happens and the write succeeds. A nonexistent pocket, and the numbers of the buffer and load/unload positions, are rejected with status `-18`.

**Siemens and Heidenhain.** On Fanuc this lives in the tool management extension B option (`cnc_rdpot_property`), which deemesh does not use, so the address returns status `-20`.

**Mitsubishi answers status `-20`.**

**Heidenhain** uses the pocket table's lock field (`L`), for both reading and writing (written and read back on the simulator). Only this column is read: a spot blocked in a box magazine by a neighbour's `LOCKED_*` column reads `false` (our test environment has none, so this has not been confirmed). How the pocket table is handled depends on the machine (TNC7 User's Manual, 'Configuring a tool': a machine manufacturer's function or an external tool management system may handle it). On such a machine a written value may be changed or may conflict with that system, so check the machine manual. A pocket number that is not in the table is status `-18`, and `magazine=0` is the spindle row, so it is status `-18`.

## /machine/toolArea/toolGroupCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **number of available** tool groups. `toolArea` filter. Returns `int`, read only. Group numbers run `1` to this value, so use it as the upper bound when walking the groups.

**This is how many slots exist, not how many groups are in use.** Most slots are empty and the numbers actually used are sparse: on a machine tool where this read `64`, only groups `1` and `60` had tools registered. Which ones are in use is answered in one read by `/machine/toolArea/registeredToolGroupList`.

**On Fanuc this address tells you whether the option is there.** Anything other than status `-20` (not supported) means the control has tool life management. `/machine/toolArea/toolCount` does the same job for Tool Management, so reading the two once each settles which tool features a Fanuc control has. On Mitsubishi this address always answers status `-20`, so the method does not apply; tool life management is answered by `registeredToolGroupList` there.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Mitsubishi tool life group numbers are not fixed slots but numbers chosen from `1` to `99999999`, so there is no slot to answer for; use `registeredToolGroupList` for the groups in use.

## /machine/toolArea/registeredToolGroupList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
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

**A machine with no group capacity allocated answers status `-20`, the same as one without the option.** `[]` means the function is there and no group is registered, so the two stay distinct. It uses `cnc_rdgrpinfo4`, so a FOCAS library without that function also answers status `-20`; when the control refuses the call, the answer is the status of that reason.

**Siemens answers with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`.

**Mitsubishi** answers the group numbers of tool life management, smallest first (checked on the simulator against the registered group list on the operator panel's `T-life` screen). Tool life groups are kept per part system, so `toolArea` is the part system number (`1` up to the channel count; unlike the magazine addresses, which answer for the whole machine). A group with no tools is not listed. Group numbers are not fixed slots but numbers chosen from `1` to `99999999`, so `toolGroupCount` and `toolGroupToolCountList` answer status `-20` there; use this list for the groups in use. A part system whose tool life groups cannot be read answers status `-20`.

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

**A machine with no group capacity allocated answers status `-20`, the same as one without the option.** `[]` means the function is there and nothing needs replacing right now, so the two stay distinct. A control that does not support the underlying function also returns status `-20`.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

A `0` means **no tools are registered in that group**. On a machine tool we measured, such slots showed on the panel as `no data`, so they are where a new group would go, but what this value promises is the tool count and nothing more.

For a single group use `/machine/toolArea/toolGroup/toolCount`; this list is the read-them-all form.

**It uses `cnc_rdgrpinfo4`, so a FOCAS library without that function answers status `-20`** (when the control refuses the call, the answer is the status of that reason). `/machine/toolArea/toolGroupCount` still works on those.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Mitsubishi tool life group numbers are not fixed slots but numbers chosen from `1` to `99999999`, so there is no slot to answer for; use `registeredToolGroupList` for the groups in use.

The length is fixed at `toolGroupCount` (the number of group slots), so it is never empty; an unused group slot is `0`.

## /machine/toolArea/toolGroup/toolCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

How many tools are **registered in that tool group**. `toolArea` + `toolGroup` filters. Returns `int`, read only.

**An empty group reads `0`.** A group number outside the machine's range is rejected with status `-18`, so `0` means "that group has no tools", not "no such group".

The upper bound on group numbers is reported by `/machine/toolArea/toolGroupCount`. Not every slot is in use, so walking them mostly yields `0`.

**A machine with no groups to work with returns status `-20`**: either the tool life management option is absent, or it is present with no group capacity allocated.

**Siemens answers with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`.

**Mitsubishi** answers the number of tools registered in a tool life management group (the length of `toolNumberList`; checked on the simulator). `toolArea` is the part system number. A group number that is not registered gives `0`, and a number outside `1` to `99999999` answers status `-18`. `toolGroupCount`, which gives the upper bound of group numbers, answers status `-20` on Mitsubishi (group numbers there are not fixed slots), so use `registeredToolGroupList` for the groups in use.

## /machine/toolArea/toolGroup/toolNumberList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **tool numbers registered in that tool group**. `toolArea` + `toolGroup` filters. Returns `intArray`, read only.

On Fanuc the list is in **use order**, not sorted by number. One group on a machine tool read `[16,13,2]`, meaning tool `16` is used first and, once its life runs out, the machine moves on to `13` and then `2`.

An empty group reads `[]`. The item count matches `/machine/toolArea/toolGroup/toolCount`, and asking for both together costs a single round trip to the machine.

The same group's `/machine/toolArea/toolGroup/toolHNumberList`, `/machine/toolArea/toolGroup/toolDNumberList` and `/machine/toolArea/toolGroup/toolLifeStatusList` **always have the same length and the same order as this list.** Line up the same positions and you have one tool's information. On Fanuc, asking for all four still costs a single round trip.

**A machine with no groups to work with returns status `-20`**: either the tool life management option is absent, or it is present with no group capacity allocated.

**Siemens answers with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`.

**Mitsubishi** answers the tool numbers registered in a tool life management group, **in registration order**. Under tool life management II, the type that selects spare tools (parameter `#1096 T_Ltyp` at `2`), with `#1105 T_sel2` at `0` the spare tools are selected in that order, so it is the order of use (Programming Manual, Machining Center System, IB-1501278); with `1` the control picks the tool with the longest remaining life in the group, so it is not (Alarm/Parameter Manual IB-1501279). We checked it on the simulator against the `#` order on the operator panel's `T-life group` screen (`[3, 2, 4]`). `toolArea` is the part system number. A group number that is not registered gives `[]`, and a number outside `1` to `99999999` answers status `-18`. `toolLifeStatusList` of the same group pairs with this list in the same order (EZSocket `FCSB1224W100-A9` or later), while `toolHNumberList` and `toolDNumberList` answer status `-20` on Mitsubishi.

## /machine/toolArea/toolGroup/toolLifeMonitorType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
codes: [{"value": 0, "name": "no monitoring"}, {"value": 1, "name": "time"}, {"value": 2, "name": "count"}]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **life monitoring mode** of that tool group. `toolArea` + `toolGroup` filters. Returns `int`; both read and write are supported. Write `{"value": 2}`.

| Value | Meaning | Unit of the life values |
|---|---|---|
| `0` | no monitoring (a slot with no tools registered) | none |
| `1` | time | minutes (`unit` is `"min"`) |
| `2` | count | uses (`unit` is `"count"`) |

It is the **same vocabulary** as `/machine/toolArea/tool/toolLifeMonitorType`; that one is per tool, this one per group. This control has no `3` (wear).

`0` means only a slot with no tools registered. When the group information (`cnc_rdgrpinfo4`) cannot be read, the answer is that error rather than `0`, and when the FOCAS library in use lacks the function, both read and write answer status `-20`.

Writes take `1` and `2` only. Going back to `0` means deleting the group, which is not what this address does.

⚠️ **Changing the mode makes the control reset the group's life and used count to `0`** (confirmed on a machine tool). deemesh sends both values unchanged; the reset is the control's. The numbers are neither converted nor kept, so write `/machine/toolArea/toolGroup/toolGroupLifeTotal` and `/machine/toolArea/toolGroup/toolGroupLifeUsed` again after changing the mode.

**Only a group with tools registered can be written.** Writing to an empty group returns status `-18`: if the control accepted it, the group itself would be created, which is outside what this address promises.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

**The unit can differ per group, so it rides along in the response's `unit`**: `"min"` under time monitoring, `"count"` under count monitoring. The value is exactly what the operator panel shows and is not converted to seconds. Parameter `6800#2` sets the machine-wide default, but the FOCAS2 specification states that on the M series it can be set per group, so deemesh asks the group itself.

Changing the monitoring mode (`/machine/toolArea/toolGroup/toolLifeMonitorType`) makes the control reset this value to `0` (confirmed on a 31i-B machine tool) and changes its unit (`min` for the time mode, `count` for the count mode). Write it after changing the mode, in the new mode's unit.

An empty group reads `0`, and that `0` carries no `unit`. When the counter type cannot be read and the value is not `0`, it cannot be given a unit and the answer is status `-17`.

**Writes take whole numbers only** - the control's field is an integer, so a fractional value is rejected with status `-16`. The upper limit is the control's own (a machine tool we measured reports `65535` uses or `4300` minutes); going over is status `-16`. **Only a group with tools registered** can be written. A write first reads the group information (`cnc_rdgrpinfo4`): if that read fails the write answers that error, and if the FOCAS library in use lacks the function it answers status `-20`.

**A machine with no groups to work with returns status `-20`**: either the tool life management option is absent, or it is present with no group capacity allocated.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

For the life left, subtract this from `/machine/toolArea/toolGroup/toolGroupLifeTotal`. The unit rule, the status `-20` condition, the `0` of an empty group carrying no `unit` and the status `-17` when the counter type cannot be read are all the same as for that address.

Changing the monitoring mode (`/machine/toolArea/toolGroup/toolLifeMonitorType`) makes the control reset this value to `0` (confirmed on a 31i-B machine tool) and changes its unit (`min` for the time mode, `count` for the count mode). A value read before the change must be written back in the new mode's unit.

**The write constraints** match `toolGroupLifeTotal`: whole numbers only, within the control's limit, and only for a group that has tools registered. A write first reads the group information (`cnc_rdgrpinfo4`): if that read fails the write answers that error, and if the FOCAS library in use lacks the function it answers status `-20`.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

## /machine/toolArea/toolGroup/toolLifeStatusList
```yaml
value_type: "intArray"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc", "nc_ezsocket_mitsubishi"]
write: []
codes: [{"value": 0, "name": "no tool", "read": ["nc_focas2_fanuc"]}, {"value": 1, "name": "usable"}, {"value": 2, "name": "life expired"}, {"value": 3, "name": "skipped"}]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **life status of each tool** in that tool group. `toolArea` + `toolGroup` filters. Returns `intArray`, read only.

| Value | Meaning |
|---|---|
| `1` | still usable |
| `2` | life used up |
| `3` | skipped, or a tool error |
| `0` | no usable tool in that slot |

**This address does not tell you which tool is running.** A tool waiting its turn and the tool currently cutting both read `1`, which means "usable", not "in use". On Fanuc, which group is in use is answered by `/machine/channel/activeToolGroupNumber` (on Mitsubishi that address answers status `-20`).

**Siemens answers with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`.

**Mitsubishi** carries the `ST` column of the operator panel's `T-life group` screen for each tool of a tool life management group. When its low digit is unused (`0`) or in use (`1`) the value is `1`, when the life is reached (`2`) it is `2`, and for tool error 1 or 2 (`3`, `4`; which error that is, the machine tool builder decides) it is `3`; the high digit (a machine tool builder setting) is ignored (Instruction Manual IB-1501274). `0` does not occur. The order is that of `/machine/toolArea/toolGroup/toolNumberList`, and each tool costs one more round trip to the machine. `toolArea` is the part system number; a group number that is not registered gives `[]`, a number outside `1` to `99999999` answers status `-18`, and a part system whose tool life groups cannot be read answers status `-20`. EZSocket `FCSB1224W100-A9` or later is required: with an earlier version the answer is always status `-20`, before any other check, and installing that version or later makes it readable. When the life values the control returns are not laid out the way deemesh reads them (a part system that answers in a different layout), the answer is status `-20` as well (checked against the operator panel on the simulator).

A group with no tools answers `[]`.

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

Follow the number to `/machine/channel/toolOffset/toolLengthGeometry` and `/machine/channel/toolOffset/toolLengthWear` for the compensation values themselves. The FOCAS2 specification states that this field is always `0` on the T series.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

A group with no tools answers `[]`.

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

Follow the number to `/machine/channel/toolOffset/toolRadiusGeometry` and `/machine/channel/toolOffset/toolRadiusWear` for the compensation values themselves. The FOCAS2 specification states that this field is always `0` on the T series.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

A group with no tools answers `[]`.

## /machine/toolArea/toolGroup/currentToolUseOrder
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup"]
read: ["nc_focas2_fanuc"]
write: []
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **use-order position of the tool that group currently points at**. `toolArea` + `toolGroup` filters. Returns `int`, read only, counting from `1`. It reads `0` for a group that has never been used.

The number matches the panel's position column (`01`, `02`, `03`) and points at the **same position** in `/machine/toolArea/toolGroup/toolNumberList` (a `2` means the second entry).

⚠️ **It does not mean the group is running.** It is a pointer the group holds and it persists: measured, it became `1` at the tool change and was still `1` after a reset and after a power cycle. To reproduce the panel's `@` (in use) marker, show it only while `/machine/channel/activeToolGroupNumber` names that group.

**A machine without the tool life management option, or with no group capacity allocated, returns status `-20`.**

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

**Creating position `1` in an empty group creates the group itself** (confirmed on a machine tool). There is no separate address for making a tool group; this is how it is done. The `0` entries in `/machine/toolArea/toolGroupToolCountList` show which numbers are free. A new group starts with no life and the control's default monitoring mode, so follow up by writing `/machine/toolArea/toolGroup/toolLifeMonitorType` first and then `/machine/toolArea/toolGroup/toolGroupLifeTotal`, because changing the mode resets the life total to `0`.

**Removing the last tool removes the group.** Its life and monitoring mode go with it (measured).

⚠️ **Positions shift.** Removing one pulls every later tool up a place (confirmed on a machine tool), so **one operation changes what every other position address refers to** (`toolNumber`, `toolHNumber`, `toolDNumber`, `toolLifeStatus`, `/machine/toolArea/toolGroup/currentToolUseOrder`). Re-read the lists between operations when working through several positions.

**Only the position just past the end can be created.** In a group holding three tools that is position `4`; anything beyond would leave a gap and returns status `-18`. Inserting in the middle is not expressible here, because a middle position **already exists**, so writing `true` there is status `-21`, as above. To reorder, create at the end and rewrite the numbers.

A group that is full (its entry in `/machine/toolArea/toolGroupToolCountList` has reached the control's limit) returns status `-23` (no room). Remove a tool from that group first, then create again. When the control refuses because the tools registered across all groups have reached its limit, the answer is also status `-23`. This can happen while the group itself still has room, so remove a tool you no longer use from any group, then create again.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

`toolUseOrder` is the position within the group, counting from `1`, and matches the panel's position column. `/machine/toolArea/toolGroup/currentToolUseOrder` says which position the group currently points at.

**To read them all at once** use `/machine/toolArea/toolGroup/toolNumberList`; same values, same round trip. This address exists so that **one position can be written**.

**A position that does not exist returns status `-18`** (the group's tool count is the bound). If the control accepted it, a tool that was not there would appear, which is outside what this address promises.

**The control may refuse the write while machining.** During automatic operation, or when the group is the one in use or lined up next, the answer is status `-22` (machine state) with the reason; try again once the job is done or the group has changed.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

Follow the number to `/machine/channel/toolOffset/toolLengthGeometry` and `/machine/channel/toolOffset/toolLengthWear` for the compensation values. The FOCAS2 specification states that this value is always `0` on the T series.

**The control may refuse the write while machining.** During automatic operation, or when the group is the one in use or lined up next, the answer is status `-22` (machine state) with the reason; try again once the job is done or the group has changed.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

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

Follow the number to `/machine/channel/toolOffset/toolRadiusGeometry` and `/machine/channel/toolOffset/toolRadiusWear` for the compensation values. The FOCAS2 specification states that this value is always `0` on the T series.

**The control may refuse the write while machining.** During automatic operation, or when the group is the one in use or lined up next, the answer is status `-22` (machine state) with the reason; try again once the job is done or the group has changed.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

## /machine/toolArea/toolGroup/toolUseOrder/toolLifeStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "toolGroup", "toolUseOrder"]
read: ["nc_focas2_fanuc"]
write: ["nc_focas2_fanuc"]
codes: [{"value": 0, "name": "no tool"}, {"value": 1, "name": "usable"}, {"value": 2, "name": "life expired"}, {"value": 3, "name": "skipped"}]
```

**On Fanuc this needs the Tool Life Management option.** A control without it refuses with status `-20` (not supported).

The **life status** of the tool at that position. `toolArea` + `toolGroup` + `toolUseOrder` filters. Returns `int`; both read and write are supported. Write `{"value": 1}`.

| Value | Meaning |
|---|---|
| `1` | still usable |
| `2` | life used up |
| `3` | skipped |
| `0` | no usable tool in that slot (reads only) |

**Write `1` after fitting a fresh insert**: turning a `2` back into a `1` is that operation.

Writes take `1`, `2` or `3` only. A `0` means "no tool", which is a deletion rather than anything this address promises, so it is rejected with status `-16`.

**The control may refuse the write while machining.** During automatic operation, or when the group is the one in use or lined up next, the answer is status `-22` (machine state) with the reason; try again once the job is done or the group has changed.

**The read-only list** is `/machine/toolArea/toolGroup/toolLifeStatusList`.

**Siemens and Mitsubishi answer with status `-20`.** Tool groups are a layer of tool life management (Fanuc, Mitsubishi); Siemens has no such table and replaces tools by grouping same-name tools through `sisterToolNumber`. Of tool life management, the Mitsubishi adapter answers only the group list and the tools in a group (`registeredToolGroupList`, `toolGroup/toolNumberList`, `toolGroup/toolCount`), the life values of a tool (`tool/toolLifeMonitorType`, `tool/toolLifeTotal`, `tool/toolLifeUsed`) and the life status of a group's tools (`toolGroup/toolLifeStatusList`).

## /machine/toolArea/tool/toolEdgeCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: []
```

The number of **cutting edges (offset data sets)** the tool has. `toolArea` + `tool` filters (`toolArea` is the tool area number the channel uses). It is Siemens `numCuttEdges`.

**This is a count, not the highest number.** Edges normally run from `1` upwards, but deleting a middle edge at the machine panel **leaves a gap and does not renumber the ones after it**: delete `2` out of `1`, `2`, `3` and what remains is `1` and `3` while this value becomes `2`. So do not assume `toolEdge` runs from `1` to this value.

A `toolEdge/…` address that points at a nonexistent edge is rejected with status `-18` on both read and write, so reading tells you which numbers are real (this address itself is read-only). The exception is `toolEdge/toolEdgeExists`: asking about a nonexistent edge answers `false`, and writing `false` to one succeeds.

**This is not the number of teeth.** The flutes or inserts a cutter physically carries ("a 2-flute ball nose", "a 4-flute end mill") are a different quantity. When several teeth sit at the same length and radius one edge covers them all; measured here, a 4-flute cutter answered `1` while a 2-flute ball nose answered `3`.

A nonexistent tool is rejected with status `-18`. It does not answer that the tool has `0` edges.

**Siemens and Heidenhain** (Fanuc and Mitsubishi return status `-20`). In the Fanuc and Mitsubishi offset models one offset (set) number *is* one set of compensation values, so there is no per-tool edge layer. Earlier versions returned a fixed `1`; that asserted a dimension that does not exist, so it was removed. Fanuc's per-tool tool management data (`H`, `D`, life) is read from the per-tool addresses under `/machine/toolArea/tool/*`.

**Heidenhain** returns the number of rows for that tool number: the tool's own row plus its index tools (rows such as `5.1` and `5.2` that follow the tool number), so a tool without indexes is `1`. **`toolEdge` is that index and starts at `0`**: the tool's own row is `toolEdge=0` and `5.1` is `toolEdge=1` (the number as shown on the control). **Index numbers can have gaps**: in our test environment the operator panel accepted `10.3` without `10.2` (the TNC7 User's Manual, 'Indexed tool', also says the numbers need not be sequential, and allows up to nine index tools per tool, so this value goes up to `10`), and this value was then `3` while `toolEdge=2` did not exist. Check which numbers exist with `/machine/toolArea/tool/toolEdge/toolEdgeExists`. A nonexistent tool and row `0` at the top of the table are status `-18`.

## /machine/toolArea/tool/toolEdge/toolEdgeExists
```yaml
value_type: "boolean"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens"]
```

Whether that edge number **exists on that tool**. `toolArea` + `tool` + `toolEdge` filters. Returns `boolean`; both read and write are supported.

Asking about an edge that does not exist is not an error. It answers `false`. Since `/machine/toolArea/tool/toolEdgeCount` gives only a count and the numbers can have gaps, **this address is what tells you which numbers are real.**

**Writing creates and deletes the edge.** `{"value": true}` creates it, `{"value": false}` deletes it. Writing `true` to an edge that already exists is rejected with status `-21` (already exists); writing `false` to an edge that does not exist succeeds. When deleting an edge is refused because a channel is not in Reset or the tool is in use, deemesh answers status `-22` (machine state); reset the channel and delete again. Any other refusal by the control is usually status `-17`.

**Edges are only created in order.** The machine creates the edge at the **lowest free number** (the number is not selectable). So asking for any other number creates nothing and is rejected with status `-18`, and the error text carries the number that would be created next. If you need `D5`, create `3`, `4` and `5` in turn. Where there is a gap, that gap is filled first.

**The first cutting edge cannot be deleted** (status `-18`). It stays as long as the tool does. To remove the whole tool use `/machine/toolArea/tool/toolExists`.

When the tool itself does not exist, every write (`true` or `false`) is status `-18`. When the tool cannot be read for a reason other than being absent (for example the account used to connect may not read tool data, so `BadUserAccessDenied` comes back), both reads and writes answer status `-17`.

Reading is supported on Siemens and Heidenhain, writing on Siemens.

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

On **Heidenhain**, `toolEdge` is the tool table index: `toolEdge=0` is the tool's own row, so it is always `true` while the tool exists, and from `toolEdge=1` on it is the row of an index tool such as `5.1` that follows the tool number. A missing index is `false`; if the tool itself does not exist (including row `0` at the top of the table) it is status `-18`. Writing is status `-20` (deemesh does not create or delete rows on Heidenhain).

## /machine/toolArea/tool/toolEdge/toolType
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The tool's **type code** (returns `int`). **On Siemens it uses the SINUMERIK DP1 code as-is** (read + write, with `desc`; Heidenhain uses a different code space without `desc`, below): the code scheme is an open classification owned by Siemens, so deemesh does not translate it, and the authority is the SINUMERIK tool-management manual (even if the vendor adds codes, the value is passed through as-is). Writes take an integer code `{"value": 500}`, for tool-setup automation, and the NCK judges code validity.

| Family | Meaning | Examples |
|---|---|---|
| `1xx` | milling tools | `120` end mill, `140` face mill, `145` thread milling |
| `2xx` | drill family | `200` twist drill, `240` tap, `250` reamer |
| `4xx` | grinding tools | |
| `5xx` | turning tools | `500` roughing, `510` finishing, `530` cutoff, `540` threading |
| `7xx` | special | `711` probe, `730` stop |

The table above is **a guide, not the authority**. The code scheme is owned by Siemens, so check the exact list in the manual for your model: for 840D sl the *Tool Management Function Manual* "List of tool types", for 828D the *Tools Function Manual*. The `desc` names follow that list (07/2021 edition), and the same numbers appear on the control's tool list screen.

Known codes come with their meaning in `desc` (`{"value": 500, "desc": "turning roughing tool"}`), and unregistered codes fall back to the first-digit family desc (`{"value": 573, "desc": "turning tool family"}`). **For a code outside the families listed above, the `desc` key is absent entirely** (`{"value": 300}`). We do not invent a meaning we do not have. Do not assume `desc` is always present.

It is also the reference value that determines the length1/2 axis assignment and radius interpretation (cutter/nose) for turning tools (5xx).

**Write caution (Siemens)**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

**Heidenhain** returns the tool type number from the tool table (`TYP`) as is. It is a different code space from the Siemens codes; deemesh does not translate it, does not unify it across controls, and attaches no `desc`. The numbers mean what the TNC7 User's Manual, 'Tool types', lists for them (in our test environment a milling tool was `9`, a drill `1`, an NC center drill `4`, a chamfer mill `24` and a turning tool `29`). The tool type shown on the control's tool management screen tells the same. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is the value of an index tool row (such as `5.1`). Writing is supported as well: the number is written as it is, and a value the control does not accept is status `-16` with the allowed range in the error message (`0` to `99` in our test environment). Check what type a written number means against the tool type shown on the control's tool management screen.

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

**Mitsubishi answers with status `-20`.** That control's offset model has no edge layer under the tool (see `toolEdgeCount`).

## /machine/toolArea/tool/toolEdge/toolTeethCount
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **number of teeth** of that cutting edge, the count you mean by "a 4-flute end mill". `toolArea` + `tool` + `toolEdge` filters. Returns `int`; both read and write are supported. Write `{"value": 4}`.

**It is not `toolEdgeCount`.** That one counts the offset data sets the control holds for the tool; this one counts how many physical cutting teeth one such set describes. Measured here, a 4-flute cutter had `1` edge and `4` teeth.

**It is stored per edge.** A single body can carry cutting sections of different diameters, whose tooth counts may differ, so the value belongs to the edge rather than to the tool.

**On Siemens it does not always match the `N` column on the machine's screen.** That column doubles up: it shows the tooth count for milling tools and the point angle for drills. Measured, a drill returned `0` here while the screen showed `118.0` (the point angle). deemesh keeps the two apart so that one address never means a different physical quantity depending on the tool type.

**Siemens and Heidenhain.** A nonexistent tool or edge (D) is rejected with status `-18`.

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

**Heidenhain** uses the number of cutting edges in the tool table (`CUT`). `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is the value of an index tool row (such as `5.1`). Writing is supported as well; a value the control does not accept is status `-16` with the allowed range in the error message (`0` to `99` in our test environment).

## /machine/toolArea/tool/toolEdge/toolLengthGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **length1 geometry** value. On Siemens it is SINUMERIK `DP3`; for turning tools it usually corresponds to the X direction, but the axis correspondence is a rule set by the tool type and active plane, so the SDK does not translate it.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens and Heidenhain**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution (Siemens)**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

**Heidenhain** uses the tool table's length (`L`) (wear is `DL`). The table holds geometry plus wear, and during machining a delta from the NC program (`TOOL CALL`) or from a compensation table can be added (TNC7 User's Manual, 'Tool compensation for tool length and tool radius'). `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `5.1`). **Unlike the unit rule above, the unit is always mm**: deemesh selects mm when it reads and writes through DNC (not confirmed with a tool table created in inch). Writing is supported; a value the control does not accept is status `-16`, with the field's allowed range in the error message (`-99999.9999` to `99999.9999` on the simulator, the input range in the manual's 'Tool table tool.t'). A nonexistent tool or index, and row `0` at the top of the table, are status `-18`. If the length cell is blank in the control's tool table (as in a row just added with tool insert; the manual's 'Tool management' also says these cells of a new tool start out empty), it is status `-22`; it reads once the cell is filled, on the control or by writing this address. **Turning, grinding and dressing tools answer status `-18`** (both read and write). deemesh tells them by the tool type column (`TYP`): turning `29`, grinding `30` and dressing `31`, numbered as in the TNC7 User's Manual, 'Tool types', which says the tool table's length and radius have no effect on those tools. In our test environment the turning tools had `L`, `R`, `DL` and `DR` all `0` in the tool table, and their geometry was in the `ZL`, `XL`, `YL` and `RS` columns of the turning tool table (`toolturn.trn`) and in its wear columns (grinding and dressing tools were not tested). The geometry of a turning tool comes from `toolXGeometry`, `toolZGeometry`, `toolYGeometry`, `toolNoseRadiusGeometry` and their wear addresses.

## /machine/toolArea/tool/toolEdge/toolLengthWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **length1 wear** value (SINUMERIK `DP12`).

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens and Heidenhain**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution (Siemens)**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

**Heidenhain** uses the tool table's length wear (`DL`) (geometry is `L`). The table holds geometry plus wear, and during machining a delta from the NC program (`TOOL CALL`) or from a compensation table can be added (TNC7 User's Manual, 'Tool compensation for tool length and tool radius'). Measuring cycles can also write this cell (manual). `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `5.1`). **Unlike the unit rule above, the unit is always mm**: deemesh selects mm when it reads and writes through DNC (not confirmed with a tool table created in inch). Writing is supported; a value the control does not accept is status `-16`, with the field's allowed range in the error message (`-999.9999` to `999.9999` on the simulator). It also accepted a value for the tool in the spindle (simulator). A nonexistent tool or index, and row `0` at the top of the table, are status `-18`. An empty cell is status `-22`; it reads once the cell is filled. **Turning, grinding and dressing tools answer status `-18`** (both read and write). deemesh tells them by the tool type column (`TYP`): turning `29`, grinding `30` and dressing `31`, numbered as in the TNC7 User's Manual, 'Tool types', which says the tool table's length and radius have no effect on those tools. In our test environment the turning tools had `L`, `R`, `DL` and `DR` all `0` in the tool table, and their geometry was in the `ZL`, `XL`, `YL` and `RS` columns of the turning tool table (`toolturn.trn`) and in its wear columns (grinding and dressing tools were not tested). The geometry of a turning tool comes from `toolXGeometry`, `toolZGeometry`, `toolYGeometry`, `toolNoseRadiusGeometry` and their wear addresses.

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

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

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

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

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

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

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

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

## /machine/toolArea/tool/toolEdge/toolRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **cutter radius geometry** value. On Siemens it is SINUMERIK `DP6` (from the milling-tool viewpoint) and points at the **same storage** as `toolNoseRadiusGeometry`, so which address you use is your declaration of intent; on Siemens the SDK therefore does not inspect the tool type.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens and Heidenhain**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution (Siemens)**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**The number can differ from what the machine's screen shows.** This value is a **radius**, as the name says, while tool-list and offset screens commonly display the **diameter (Ø)**. Measured (2026-07): `BALLNOSE_D8` stores `4.0` and the HMI shows `8.000`. deemesh emits what the machine stores and does not multiply by two.

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

**Heidenhain** uses the tool table's radius (`R`) (wear is `DR`; the control shows it as a radius too). The table holds geometry plus wear, and during machining a delta from the NC program (`TOOL CALL`) or from a compensation table can be added (TNC7 User's Manual, 'Tool compensation for tool length and tool radius'). `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `5.1`). **Unlike the unit rule above, the unit is always mm**: deemesh selects mm when it reads and writes through DNC (not confirmed with a tool table created in inch). Writing is supported; a value the control does not accept is status `-16`, with the field's allowed range in the error message. A nonexistent tool or index, and row `0` at the top of the table, are status `-18`. If the radius cell is blank in the control's tool table (as in a row just added with tool insert; the manual's 'Tool management' also says these cells of a new tool start out empty), it is status `-22`; it reads once the cell is filled, on the control or by writing this address. **Turning, grinding and dressing tools answer status `-18`** (both read and write). deemesh tells them by the tool type column (`TYP`): turning `29`, grinding `30` and dressing `31`, numbered as in the TNC7 User's Manual, 'Tool types', which says the tool table's length and radius have no effect on those tools. In our test environment the turning tools had `L`, `R`, `DL` and `DR` all `0` in the tool table, and their geometry was in the `ZL`, `XL`, `YL` and `RS` columns of the turning tool table (`toolturn.trn`) and in its wear columns (grinding and dressing tools were not tested). The geometry of a turning tool comes from `toolXGeometry`, `toolZGeometry`, `toolYGeometry`, `toolNoseRadiusGeometry` and their wear addresses.

## /machine/toolArea/tool/toolEdge/toolRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **cutter radius wear** value. On Siemens it is SINUMERIK `DP15`, the same storage as `toolNoseRadiusWear`.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens and Heidenhain**; specifying a nonexistent tool/edge surfaces an error. The **applied value is geometry + wear**.

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution (Siemens)**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**The number can differ from what the machine's screen shows.** This value is a **radius**, as the name says, while tool-list and offset screens commonly display the **diameter (Ø)**. Measured (2026-07): `BALLNOSE_D8` stores `4.0` and the HMI shows `8.000`. deemesh emits what the machine stores and does not multiply by two.

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

**Heidenhain** uses the tool table's radius wear (`DR`) (geometry is `R`). The table holds geometry plus wear, and during machining a delta from the NC program (`TOOL CALL`) or from a compensation table can be added (TNC7 User's Manual, 'Tool compensation for tool length and tool radius'). Measuring cycles can also write this cell (manual). `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `5.1`). **Unlike the unit rule above, the unit is always mm**: deemesh selects mm when it reads and writes through DNC (not confirmed with a tool table created in inch). Writing is supported; a value the control does not accept is status `-16`, with the field's allowed range in the error message. A nonexistent tool or index, and row `0` at the top of the table, are status `-18`. An empty cell is status `-22`; it reads once the cell is filled. **Turning, grinding and dressing tools answer status `-18`** (both read and write). deemesh tells them by the tool type column (`TYP`): turning `29`, grinding `30` and dressing `31`, numbered as in the TNC7 User's Manual, 'Tool types', which says the tool table's length and radius have no effect on those tools. In our test environment the turning tools had `L`, `R`, `DL` and `DR` all `0` in the tool table, and their geometry was in the `ZL`, `XL`, `YL` and `RS` columns of the turning tool table (`toolturn.trn`) and in its wear columns (grinding and dressing tools were not tested). The geometry of a turning tool comes from `toolXGeometry`, `toolZGeometry`, `toolYGeometry`, `toolNoseRadiusGeometry` and their wear addresses.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **nose radius geometry** value (SINUMERIK `DP6`, from the turning-tool viewpoint). On Siemens, same storage as `toolRadiusGeometry`.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens and Heidenhain**; specifying a nonexistent tool/edge surfaces an error. The **table holds geometry plus wear**, and during machining a delta from the NC program (`FUNCTION TURNDATA CORR`) or from a compensation table can be added (TNC7 User's Manual, 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)').

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution (Siemens)**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

**Heidenhain** uses the `RS` field of the turning tool table (`toolturn.trn`) (the wear is `toolNoseRadiusWear` (`DRS`); the applied value is geometry + wear). Unlike Siemens it is not the same storage as `toolRadiusGeometry`: the nose radius of a turning tool is read only through this address, and a tool with no row in the turning tool table (a milling tool, for example) answers status `-18` (its radius is `toolRadiusGeometry`); `toolRadiusGeometry` answers status `-18` for a tool whose tool type (`TYP`) is turning, grinding or dressing. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `320.1`). **Unlike the unit rule above, the unit is always mm**: deemesh selects mm when it reads and writes through DNC. Writing is supported; a value the control does not accept is status `-16`, with the field's allowed range in the error message. When the turning tool table is not found the status is `-20` (the TNC7 User's Manual describes this table as a feature of software option 50; our test environment has the table, so that case has not been confirmed). A nonexistent tool or index, and row `0` at the top of the table, are status `-18`. Reading and writing were confirmed on the TNC7 programming station.

## /machine/toolArea/tool/toolEdge/toolNoseRadiusWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **nose radius wear** value (SINUMERIK `DP15`). On Siemens, same storage as `toolRadiusWear`.

Returns `float`; both read and write are supported; write `{"value": 125.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens and Heidenhain**; specifying a nonexistent tool/edge surfaces an error. The **table holds geometry plus wear**, and during machining a delta from the NC program (`FUNCTION TURNDATA CORR`) or from a compensation table can be added (TNC7 User's Manual, 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Touch probe cycles that measure the workpiece can also write this wear column (manual, 'Turning tool table toolturn.trn (option 50)').

The unit follows the machine setting (mm or inch). Only `G700`/`G710` change the unit of tool offsets (Programming Manual): `/machine/channel/gModalCategory/gModal?gModalCategory=4` reading `G710` means metric and `G700` means inch. `G70`/`G71` switch only coordinates, so under them (or under neither) tool offsets are in the unit of the basic system (`MD10240`). This address carries no `unit` field, because the unit is not fixed per address.

**Write caution (Siemens)**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

**Heidenhain** uses the `DRS` field of the turning tool table (`toolturn.trn`) (the geometry is `toolNoseRadiusGeometry` (`RS`); the applied value is geometry + wear). Unlike Siemens it is not the same storage as `toolRadiusWear`: the nose radius of a turning tool is read only through this address, and a tool with no row in the turning tool table (a milling tool, for example) answers status `-18` (its radius is `toolRadiusGeometry`); `toolRadiusGeometry` answers status `-18` for a tool whose tool type (`TYP`) is turning, grinding or dressing. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `320.1`). **Unlike the unit rule above, the unit is always mm**: deemesh selects mm when it reads and writes through DNC. Writing is supported; a value the control does not accept is status `-16`, with the field's allowed range in the error message. When the turning tool table is not found the status is `-20` (the TNC7 User's Manual describes this table as a feature of software option 50; our test environment has the table, so that case has not been confirmed). A nonexistent tool or index, and row `0` at the top of the table, are status `-18`. Reading and writing were confirmed on the TNC7 programming station.

## /machine/toolArea/tool/toolEdge/toolXGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The **X-direction length geometry** value of a turning tool. It is the `XL` field of the Heidenhain turning tool table (`toolturn.trn`); the wear is `toolXWear` (`DXL`). It is the length in the X direction from the tool carrier preset, "tool length 2" in the control's dialog (manual, 'Turning tool table toolturn.trn (option 50)'). The **table holds geometry plus wear**, and during machining a delta from the NC program (`FUNCTION TURNDATA CORR`) or from a compensation table can be added (TNC7 User's Manual, 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Here X is not an axis name but a fixed column of that table, that is, a **directional component of the tool dimensions**.

Returns `float`; both read and write are supported; write `{"value": 45.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `320.1`). **The unit is always mm**: deemesh selects mm when it reads and writes through DNC (so no `unit` field is attached).

**Only turning tools answer.** A tool with no row in the turning tool table (a milling tool, for example) answers status `-18`, and its length and radius are read through `toolLengthGeometry` and `toolRadiusGeometry`; those two addresses answer status `-18` for a tool whose tool type (`TYP`) is turning, grinding or dressing. When the turning tool table is not found the status is `-20` (the TNC7 User's Manual describes this table as a feature of software option 50; our test environment has the table, so that case has not been confirmed). A nonexistent tool or index, and row `0` at the top of the table, are status `-18` as well. A value the control does not accept is status `-16`, with the field's allowed range in the error message (`-99999.9999` to `99999.9999` on the simulator). Reading and writing were confirmed on the TNC7 programming station.

**Heidenhain only.** Fanuc and Mitsubishi answer status `-20`; read a lathe's X-direction compensation from the channel offset table at `/machine/channel/toolOffset/toolXGeometry`. Siemens answers status `-20` too: its per-tool table has lengths 1 to 3 (`toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns, and which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/toolArea/tool/toolEdge/toolXWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The **X-direction length wear** value of a turning tool. It is the `DXL` field of the Heidenhain turning tool table (`toolturn.trn`); the geometry is `toolXGeometry` (`XL`). The **table holds geometry plus wear**, and during machining a delta from the NC program (`FUNCTION TURNDATA CORR`) or from a compensation table can be added (TNC7 User's Manual, 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Touch probe cycles that measure the workpiece can also write this wear column (manual, 'Turning tool table toolturn.trn (option 50)'). Here X is not an axis name but a fixed column of that table, that is, a **directional component of the tool dimensions**.

Returns `float`; both read and write are supported; write `{"value": 0.05}`. Requires the `toolArea` + `tool` + `toolEdge` filters. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `320.1`). **The unit is always mm**: deemesh selects mm when it reads and writes through DNC (so no `unit` field is attached).

**Only turning tools answer.** A tool with no row in the turning tool table (a milling tool, for example) answers status `-18`, and its length and radius are read through `toolLengthGeometry` and `toolRadiusGeometry`; those two addresses answer status `-18` for a tool whose tool type (`TYP`) is turning, grinding or dressing. When the turning tool table is not found the status is `-20` (the TNC7 User's Manual describes this table as a feature of software option 50; our test environment has the table, so that case has not been confirmed). A nonexistent tool or index, and row `0` at the top of the table, are status `-18` as well. A value the control does not accept is status `-16`, with the field's allowed range in the error message (`-99999.9999` to `99999.9999` on the simulator). Reading and writing were confirmed on the TNC7 programming station.

**Heidenhain only.** Fanuc and Mitsubishi answer status `-20`; read a lathe's X-direction compensation from the channel offset table at `/machine/channel/toolOffset/toolXWear`. Siemens answers status `-20` too: its per-tool table has lengths 1 to 3 (`toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns, and which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/toolArea/tool/toolEdge/toolZGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The **Z-direction length geometry** value of a turning tool. It is the `ZL` field of the Heidenhain turning tool table (`toolturn.trn`); the wear is `toolZWear` (`DZL`). It is the length in the Z direction from the tool carrier preset, "tool length 1" in the control's dialog (manual, 'Turning tool table toolturn.trn (option 50)'). The **table holds geometry plus wear**, and during machining a delta from the NC program (`FUNCTION TURNDATA CORR`) or from a compensation table can be added (TNC7 User's Manual, 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Here Z is not an axis name but a fixed column of that table, that is, a **directional component of the tool dimensions**.

Returns `float`; both read and write are supported; write `{"value": 70.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `320.1`). **The unit is always mm**: deemesh selects mm when it reads and writes through DNC (so no `unit` field is attached).

**Only turning tools answer.** A tool with no row in the turning tool table (a milling tool, for example) answers status `-18`, and its length and radius are read through `toolLengthGeometry` and `toolRadiusGeometry`; those two addresses answer status `-18` for a tool whose tool type (`TYP`) is turning, grinding or dressing. When the turning tool table is not found the status is `-20` (the TNC7 User's Manual describes this table as a feature of software option 50; our test environment has the table, so that case has not been confirmed). A nonexistent tool or index, and row `0` at the top of the table, are status `-18` as well. A value the control does not accept is status `-16`, with the field's allowed range in the error message (`-99999.9999` to `99999.9999` on the simulator). Reading and writing were confirmed on the TNC7 programming station.

**Heidenhain only.** Fanuc and Mitsubishi answer status `-20`; read a lathe's Z-direction compensation from the channel offset table at `/machine/channel/toolOffset/toolZGeometry`. Siemens answers status `-20` too: its per-tool table has lengths 1 to 3 (`toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns, and which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/toolArea/tool/toolEdge/toolZWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The **Z-direction length wear** value of a turning tool. It is the `DZL` field of the Heidenhain turning tool table (`toolturn.trn`); the geometry is `toolZGeometry` (`ZL`). The **table holds geometry plus wear**, and during machining a delta from the NC program (`FUNCTION TURNDATA CORR`) or from a compensation table can be added (TNC7 User's Manual, 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Touch probe cycles that measure the workpiece can also write this wear column (manual, 'Turning tool table toolturn.trn (option 50)'). Here Z is not an axis name but a fixed column of that table, that is, a **directional component of the tool dimensions**.

Returns `float`; both read and write are supported; write `{"value": 0.05}`. Requires the `toolArea` + `tool` + `toolEdge` filters. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `320.1`). **The unit is always mm**: deemesh selects mm when it reads and writes through DNC (so no `unit` field is attached).

**Only turning tools answer.** A tool with no row in the turning tool table (a milling tool, for example) answers status `-18`, and its length and radius are read through `toolLengthGeometry` and `toolRadiusGeometry`; those two addresses answer status `-18` for a tool whose tool type (`TYP`) is turning, grinding or dressing. When the turning tool table is not found the status is `-20` (the TNC7 User's Manual describes this table as a feature of software option 50; our test environment has the table, so that case has not been confirmed). A nonexistent tool or index, and row `0` at the top of the table, are status `-18` as well. A value the control does not accept is status `-16`, with the field's allowed range in the error message (`-99999.9999` to `99999.9999` on the simulator). Reading and writing were confirmed on the TNC7 programming station.

**Heidenhain only.** Fanuc and Mitsubishi answer status `-20`; read a lathe's Z-direction compensation from the channel offset table at `/machine/channel/toolOffset/toolZWear`. Siemens answers status `-20` too: its per-tool table has lengths 1 to 3 (`toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns, and which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/toolArea/tool/toolEdge/toolYGeometry
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The **Y-direction length geometry** value of a turning tool. It is the `YL` field of the Heidenhain turning tool table (`toolturn.trn`); the wear is `toolYWear` (`DYL`). It is the length in the Y direction from the tool carrier preset, "tool length 3" in the control's dialog (manual, 'Turning tool table toolturn.trn (option 50)'). The **table holds geometry plus wear**, and during machining a delta from the NC program (`FUNCTION TURNDATA CORR`) or from a compensation table can be added (TNC7 User's Manual, 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Here Y is not an axis name but a fixed column of that table, that is, a **directional component of the tool dimensions**.

Returns `float`; both read and write are supported; write `{"value": 0.0}`. Requires the `toolArea` + `tool` + `toolEdge` filters. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `320.1`). **The unit is always mm**: deemesh selects mm when it reads and writes through DNC (so no `unit` field is attached).

**Only turning tools answer.** A tool with no row in the turning tool table (a milling tool, for example) answers status `-18`, and its length and radius are read through `toolLengthGeometry` and `toolRadiusGeometry`; those two addresses answer status `-18` for a tool whose tool type (`TYP`) is turning, grinding or dressing. When the turning tool table is not found the status is `-20` (the TNC7 User's Manual describes this table as a feature of software option 50; our test environment has the table, so that case has not been confirmed). A nonexistent tool or index, and row `0` at the top of the table, are status `-18` as well. A value the control does not accept is status `-16`, with the field's allowed range in the error message (`-99999.9999` to `99999.9999` on the simulator). Reading and writing were confirmed on the TNC7 programming station.

**Heidenhain only.** Fanuc and Mitsubishi answer status `-20`; read a lathe's Y-direction compensation from the channel offset table at `/machine/channel/toolOffset/toolYGeometry`. Siemens answers status `-20` too: its per-tool table has lengths 1 to 3 (`toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns, and which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/toolArea/tool/toolEdge/toolYWear
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The **Y-direction length wear** value of a turning tool. It is the `DYL` field of the Heidenhain turning tool table (`toolturn.trn`); the geometry is `toolYGeometry` (`YL`). The **table holds geometry plus wear**, and during machining a delta from the NC program (`FUNCTION TURNDATA CORR`) or from a compensation table can be added (TNC7 User's Manual, 'Compensating turning tools with FUNCTION TURNDATA CORR (option 50)'). Touch probe cycles that measure the workpiece can also write this wear column (manual, 'Turning tool table toolturn.trn (option 50)'). Here Y is not an axis name but a fixed column of that table, that is, a **directional component of the tool dimensions**.

Returns `float`; both read and write are supported; write `{"value": 0.05}`. Requires the `toolArea` + `tool` + `toolEdge` filters. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `320.1`). **The unit is always mm**: deemesh selects mm when it reads and writes through DNC (so no `unit` field is attached).

**Only turning tools answer.** A tool with no row in the turning tool table (a milling tool, for example) answers status `-18`, and its length and radius are read through `toolLengthGeometry` and `toolRadiusGeometry`; those two addresses answer status `-18` for a tool whose tool type (`TYP`) is turning, grinding or dressing. When the turning tool table is not found the status is `-20` (the TNC7 User's Manual describes this table as a feature of software option 50; our test environment has the table, so that case has not been confirmed). A nonexistent tool or index, and row `0` at the top of the table, are status `-18` as well. A value the control does not accept is status `-16`, with the field's allowed range in the error message (`-99999.9999` to `99999.9999` on the simulator). Reading and writing were confirmed on the TNC7 programming station.

**Heidenhain only.** Fanuc and Mitsubishi answer status `-20`; read a lathe's Y-direction compensation from the channel offset table at `/machine/channel/toolOffset/toolYWear`. Siemens answers status `-20` too: its per-tool table has lengths 1 to 3 (`toolLengthGeometry`, `toolLength2Geometry`, `toolLength3Geometry`) instead of X, Y and Z columns, and which length is which direction depends on the tool type and the active plane, so deemesh does not translate it.

## /machine/toolArea/tool/toolEdge/toolTipDirection
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **tool-tip position code** (SINUMERIK `DP2`, `1`-`9`). Indicates where the tip lies relative to the nose center during nose-radius compensation; it is a position code, not an angle. Fanuc's tool offset tree has the same concept (`0`-`9`), and **the numbering and `desc` vocabulary are shared**, so values can be compared and reused as-is across machine types (`desc` is attached to `9` only: the manual defines the `1`-`8` orientations by diagram alone, in three variants by machining setup, so they are not shipped). The valid range is `1`-`9` (Tool Management Function Manual defines the cutting edge positions as `1`-`8` and `9`). Writes take only this range; **any other value, `0` included, answers status `-16`**.

Returns `int`; both read and write are supported. Write `{"value": 3}`. Requires the `toolArea` + `tool` + `toolEdge` filters. **Siemens only**; specifying a nonexistent tool/edge surfaces an error.

**Write caution**: a nonexistent tool, or an edge that tool does not have, is rejected with status `-18` (the message says whether it is the tool or the edge that is missing). The machine itself would create a new edge when writing to edge count + 1, but a single typo would leave an unintended edge behind, so deemesh allows **modifying existing edges only** (create/delete via `toolEdgeExists`).

**Fanuc and Mitsubishi answer with status `-20`.** Their offset models have no edge layer under the tool (see `toolEdgeCount`). On those two, read the tip position on the offset-number side, `/machine/channel/toolOffset/toolTipDirection`.

## /machine/toolArea/tool/toolEdge/toolTipAngle
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **tip angle** of that cutting edge: the point angle of a drill (`118.0`), `90.0` for a centre drill, and so on. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported. Write `{"value": 118.0}`.

**It is not `toolTipDirection`.** The names differ by one word but the values do not match up: that one is a code for **which direction** the tip sits relative to the nose centre, while this one is the **angle** of the tip.

**Tools that carry no angle report `0.0`.** Milling tools are such tools. Measured on Siemens, a drill gave `118.0` and a face mill `0.0`. The source is SINUMERIK `DP24`: for drills it is the field the panel (Operate) uses for the tip angle, but **for turning tools the same field is the clearance angle** (cutting edge data table of the tool management manual), so do not read it as a tip angle on a turning tool.

**The `N` column on the machine's screen doubles as this value and `toolTeethCount`.** It shows the angle for drills and the tooth count for milling tools in that one cell. deemesh keeps them apart so that one address never means a different physical quantity depending on the tool type. To find the number from the screen, look at whichever of the two carries a value.

**Siemens and Heidenhain support it.** On Siemens a nonexistent tool or edge (D) is rejected with status `-18`.

**Heidenhain** uses the tool table's point angle (`T-ANGLE`). The TNC7 User's Manual ('Tool table tool.t') describes this column as the point angle of tools such as drills (used for the simulation, in cycles and for collision monitoring) and gives its input range as -180 to +180. In our test environment a drill read `118.0`, a spot drill `90.0` and a milling tool `0.0`. `toolEdge=0` is the tool's own row and from `toolEdge=1` on it is an index tool row (such as `5.1`). Writing is supported; a value the control does not accept is status `-16`, with the field's allowed range in the error message (writing `200.0` in our test environment gave `-180` to `180`). A nonexistent tool or index, and row `0` at the top of the table, are status `-18`; a blank cell in the control's tool table is status `-22`; when the `T-ANGLE` column is not found in the tool table the status is `-20` (this address does not work on that machine). **Turning, grinding and dressing tools (tool type column `TYP` `29`, `30` or `31`) answer status `-18`** (both read and write). The turning tool table (`toolturn.trn`) keeps the angles of a turning tool in its own `T-ANGLE` (tool angle) and `P-ANGLE` (point angle) columns (TNC7 User's Manual, 'Turning tool table toolturn.trn'). deemesh does not expose those two.

**Fanuc and Mitsubishi answer with status `-20`.** Neither control's offset model has an edge layer under the tool (see `toolEdgeCount`); read compensation values from the channel offset table under `/machine/channel/toolOffset/…`.

## /machine/toolArea/tool/toolEdge/toolLifeTotal
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
write: ["nc_opcua_siemens", "nc_dnc_heidenhain"]
```

The **whole life budget** allotted to that cutting edge. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported. Write `{"value": 6}`.

The Siemens control starts the remaining life at this value and counts down (Heidenhain counts the used time up; see its paragraph below). The unit is decided by the monitoring method and is carried in the response's `unit` field (see `/machine/toolArea/tool/toolLifeMonitorType`). Under wear monitoring the value is a distance, so its unit depends on the machine setting (mm/inch) and no `unit` field is attached. With monitoring off the request is rejected with status `-18` (except a Heidenhain write, which is accepted; see below).

**The value is exactly what the operator panel shows.** Under time monitoring that means minutes (`unit` is `"min"`), not converted to seconds. This differs from the other time values (`…Duration`, always seconds): tool life is a number the operator reads off the screen, so it has to match the screen.

When counting pieces, **only whole numbers are accepted.** A fractional value makes the machine answer success while leaving the value unchanged, so deemesh refuses it first with status `-16`.

**Writing this value makes SINUMERIK re-evaluate the tool's lock** (Siemens Tool management Function Manual §8.11, and confirmed on our bench). With the remaining life within its limits the lock is released (`/machine/toolArea/tool/toolUseStatus` reads `1`/`2`); with `0` or less remaining the tool is locked (`3`). A tool whose lock bit was set for a reason other than its life (`5`) is released as well, so to keep it locked, write `5` to `toolUseStatus` again after this write. Whether a tool that is `5` only because it has no use permission is released too is not yet confirmed.

**Siemens and Heidenhain.** On Fanuc tool life belongs to the tool, not to a cutting edge, so it lives at `/machine/toolArea/tool/toolLifeTotal` (in seconds or counts there).

**Mitsubishi answers with status `-20`.** That control's offset model has no edge layer under the tool (see `toolEdgeCount`).

**Heidenhain** uses the row's maximum life (`TIME1`). `toolEdge=0` is the tool's own row, so it gives the same value as `/machine/toolArea/tool/toolLifeTotal`, and from `toolEdge=1` on it is the value of an index tool (a row such as `320.1` that follows a tool number). Each index tool has a life of its own. The rules match the tool-level address: the unit is minutes (`unit` is `"min"`), and `0` means there is no maximum life, so reading is status `-18` (also for a row that has only `TIME2`). Writing is accepted even while monitoring is off (writing a value turns it on, writing `0` turns it off). Only whole minutes are accepted (a fraction is status `-16`, because the control rounds it to a whole number). A value the control does not accept is status `-16`, with the allowed range in the error message. The control counts the used time up rather than a remainder down, so that value is answered by `/machine/toolArea/tool/toolEdge/toolLifeUsed`. A nonexistent tool or index, and row `0` at the top of the table, are status `-18`.

## /machine/toolArea/tool/toolEdge/toolLifeUsed
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The life that edge (index tool) **has used so far**. `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported. Write `{"value": 0}`.

**Heidenhain only.** It is the current used time of that row in the tool table (`CUR_TIME`, shown as `CUR_TIME (min)` on the control's tool management screen), in minutes (`unit` is `"min"`). `toolEdge=0` is the tool's own row, so it gives the same value as `/machine/toolArea/tool/toolLifeUsed`, and from `toolEdge=1` on it is the value of an index tool (a row such as `320.1` that follows a tool number). Each index tool has a life of its own. The control counts this value up toward the maximum life (`/machine/toolArea/tool/toolEdge/toolLifeTotal`) (in our test environment the tool's own row and the row of the milling index tool `10.1` each grew during feed blocks, and while `10.1` was in use the tool's own row stayed as it was; the TNC7 User's Manual, 'Indexed tool', also says the control writes the used time separately for each row).

**Write this address when you change the insert and reset the life** (usually `0`). Fractions are accepted, and the control rounds them to two decimal places (see `/machine/toolArea/tool/toolLifeUsed`). When the row has no life limit (`TIME1` and `TIME2` both `0`), both reading and writing are status `-18`; write `/machine/toolArea/tool/toolEdge/toolLifeTotal` first. A nonexistent tool or index, and row `0` at the top of the table, are status `-18`.

Siemens, Fanuc and Mitsubishi answer with status `-20`. Siemens counts the remaining life down, so see `/machine/toolArea/tool/toolEdge/toolLifeRemaining`; on Fanuc and Mitsubishi see the tool-level `/machine/toolArea/tool/toolLifeUsed`.

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

**This is the address you write after changing an insert to reset the life**, usually to the same value as `toolLifeTotal`.

**Writing this value makes SINUMERIK re-evaluate the tool's lock** (Siemens Tool management Function Manual §8.11, and confirmed on our bench). A tool locked because its life ran out (`/machine/toolArea/tool/toolUseStatus` `3`) is released by this write alone (`1`/`2`), and writing `0` makes the control lock it (`3`). A tool whose lock bit was set for a reason other than its life (`5`) is released as well, so to keep it locked, write `5` to `toolUseStatus` again after this write. Whether a tool that is `5` only because it has no use permission is released too is not yet confirmed.

The unit is decided by the monitoring method and is carried in the response's `unit` field. Under wear monitoring the value is a distance, so its unit depends on the machine setting (mm/inch) and no `unit` field is attached. With monitoring off the request is rejected with status `-18`; a fractional value while counting pieces is rejected with status `-16`.

**Siemens only.** Fanuc provides an **up-counting usage counter** rather than a remainder, so read `/machine/toolArea/tool/toolLifeUsed` (per tool) and subtract it from `/machine/toolArea/tool/toolLifeTotal` for the remainder.

**Mitsubishi answers with status `-20`.** That control's offset model has no edge layer under the tool (see `toolEdgeCount`).

**Heidenhain answers with status `-20`.** Heidenhain counts the used time up, so read `/machine/toolArea/tool/toolEdge/toolLifeUsed` and get the time left to the maximum life (`TIME1`) by subtracting it from `/machine/toolArea/tool/toolEdge/toolLifeTotal`. If the limit at tool call `TIME2` is smaller, that one takes effect first (see `/machine/toolArea/tool/toolEdge/toolUseStatus`).

## /machine/toolArea/tool/toolEdge/toolLifeWarnLimit
```yaml
value_type: "float"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_opcua_siemens"]
write: ["nc_opcua_siemens"]
```

The **warning limit**: once the remaining life drops below this value the control raises a warning (`/machine/toolArea/tool/toolLifeWarnOn`, which is **per tool**). `toolArea` + `tool` + `toolEdge` filters. Returns `float`; both read and write are supported. Write `{"value": 3}`.

It buys time to prepare a replacement tool, so it is set lower than `toolLifeTotal`. The unit is decided by the monitoring method and is carried in the response's `unit` field. Under wear monitoring the value is a distance, so its unit depends on the machine setting (mm/inch) and no `unit` field is attached. With monitoring off the request is rejected with status `-18`; a fractional value while counting pieces is rejected with status `-16`.

**Writing this value makes SINUMERIK re-evaluate the tool's lock** (Siemens Tool management Function Manual §8.11, and confirmed on our bench). With the remaining life within its limits the lock is released (`/machine/toolArea/tool/toolUseStatus` reads `1`/`2`); with `0` or less remaining the tool is locked (`3`). A tool whose lock bit was set for a reason other than its life (`5`) is released as well, so to keep it locked, write `5` to `toolUseStatus` again after this write. Whether a tool that is `5` only because it has no use permission is released too is not yet confirmed.

**Siemens only.** On Fanuc the notice life belongs to the tool and lives at `/machine/toolArea/tool/toolLifeWarnLimit`.

**Mitsubishi answers with status `-20`.** That control's offset model has no edge layer under the tool (see `toolEdgeCount`).

**Heidenhain answers with status `-20`.**

## /machine/toolArea/tool/toolEdge/toolUseStatus
```yaml
value_type: "int"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
codes: [{"value": 1, "name": "unused"}, {"value": 2, "name": "in use"}, {"value": 3, "name": "life expired"}, {"value": 5, "name": "locked"}]
```

The **use status** of that edge (index tool). `toolArea` + `tool` + `toolEdge` filters. Returns `int` + `desc`; both read and write are supported. The values mean the same as the tool-level `/machine/toolArea/tool/toolUseStatus` (`1` unused, `2` in use, `3` life expired, `5` locked).

**Heidenhain only.** Each index tool has a lock and a life of its own, so the tool-level rule is applied to that row's lock (`TL`) and life (maximum `TIME1`, the limit at tool call `TIME2`, used `CUR_TIME`): it is `3` whatever the lock when the used time has reached `TIME2`; otherwise, when locked, it is `3` if a maximum life is set and the used time has reached it, otherwise `5`; when not locked, it is `2` if there is used time and `1` if not. `0` and `4` do not occur. `toolEdge=0` is the tool's own row, so it gives the same value as the tool-level address. Being past `TIME1` follows the control's lock, so to tell whether the life has run out, compare `/machine/toolArea/tool/toolEdge/toolLifeUsed` with `/machine/toolArea/tool/toolEdge/toolLifeTotal`. A nonexistent tool or index, and row `0` at the top of the table, are status `-18`.

**Writing** changes that row's lock (`TL`), with the same rules as the tool-level address: `5` locks, and `1`/`2` unlock but are accepted only when the row then reads that value (`2` with used time, `1` without; a mismatch is status `-16`, and to mark it unused write `0` to `/machine/toolArea/tool/toolEdge/toolLifeUsed` first). `3`, `0` and `4` are status `-16`; when the row's used time has reached `TIME2` it reads `3` whatever the lock, so `1`, `2` and `5` are status `-16` too (correct the used time first). Otherwise, if the row is already in that state nothing happens and the write succeeds. Locking an index row left the tool's own row as it was (confirmed in our test environment).

Siemens, Fanuc and Mitsubishi answer with status `-20`. On Siemens and Fanuc the use status is per tool, which `/machine/toolArea/tool/toolUseStatus` answers.

## /machine/toolArea/tool/toolEdge/sisterTool
```yaml
value_type: "object"
null_able: false
required_filters: ["toolArea", "tool", "toolEdge"]
read: ["nc_dnc_heidenhain"]
write: ["nc_dnc_heidenhain"]
```

The **replacement tool** used in place of that edge (index tool). `toolArea` + `tool` + `toolEdge` filters. Returns `object`; both read and write are supported. Write `{"value": {"toolNumber": 320, "toolEdgeNumber": 2}}`.

The value's shape and rules match the tool-level `/machine/toolArea/tool/sisterTool`: two numbers that point at the replacement tool, such as `{"toolNumber": 320, "toolEdgeNumber": 2}`, or `{"toolNumber": 0, "toolEdgeNumber": 0}` with no replacement tool. Put them as they are into the `tool` and `toolEdge` filters to read that tool.

**Heidenhain only.** Each index tool (a row such as `320.1` that follows a tool number) has a replacement tool field (`RT`) of its own, and this address carries that row's value. `toolEdge=0` is the tool's own row, so it gives the same value as the tool-level address.

**Writing**: give both keys as integers. Any other key, or a missing one, is status `-16`. Writing `{"toolNumber": 0, "toolEdgeNumber": 0}` clears it. The replacement tool must be in the tool table; the control does not accept a tool that is not there, which is status `-16`. Because the control keeps one decimal number, an index that cannot be told apart in that form (an index ending in `0`, such as `10`, `20` or `100`) is not sent and is status `-16`. A nonexistent tool or index, and row `0` at the top of the table, are status `-18`.

Siemens, Fanuc and Mitsubishi answer with status `-20`.
