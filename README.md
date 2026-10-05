# deemesh

**A unified CNC interface: one address model over Fanuc, Siemens, Mitsubishi and Heidenhain controls.**

[![License: Freeware](https://img.shields.io/badge/license-Freeware-brightgreen.svg)](LICENSE)

> This repository hosts deemesh's **documentation and releases**. deemesh is free but **not
> open source**: this repo does not contain its source code. The "Source code" archives that
> GitHub attaches to each release contain only the files you see here.

> 🇰🇷 **한국어 사용자**: 제품명은 **디메시**(deemesh), 서버 제품은 **디메시 허브**(deemesh-hub)로 표기합니다.
> 다운로드 패키지에는 설정 가이드·명령 매뉴얼·주소 카탈로그 등
> **모든 문서가 한국어와 영어로 포함**되어 있습니다 (`*_KO.md` 한국어 원본 + `*_EN.md` 영어판).

---

## What is deemesh?

On a shop floor, the same value sits behind different interfaces: FOCAS2 on one control, OPC-UA on another, EZSocket on a third. And a shared protocol is not the whole answer: where two controls both offer OPC-UA, their data models still differ, so the node holding an axis position on one is not the node on the other. Connecting a machine means learning its interface *and* its address space; the next one starts over.

**deemesh puts one address layer on top.** Whether the machine runs a Fanuc, Siemens, Mitsubishi Electric or Heidenhain control, you read and write axis positions, alarms, programs and tool data through the same command structure:

```
/machine/channel/axis/machinePosition?channel=1&axis=1
```

That address is the same on every supported control, and deemesh maps it onto each vendor's interface underneath. Where controls behave differently, or an address is not available on one of them or needs an option installed on the machine, the catalog says so for that address.

**The interface itself is public.** Browse the complete address catalog here: **[English](docs/CATALOG_EN.md)** · **[한국어](docs/CATALOG_KO.md)**

---

## Two ways to use it

| | What it is | Who it's for |
|---|---|---|
| **deemesh-sdk** | A native library (`.dll` / `.so`) callable from C#, Python, C++ and Rust over the C ABI | Developers **integrating** equipment communication into their own application |
| **deemesh-hub** | A standalone HTTP server (plain GET/POST, no code required) | Anyone who wants to use it **over HTTP without writing code**, or to **try it first** |

Both are free. Grab them from **Releases** below.

---

## Try it in 5 minutes: no code (deemesh-hub)

The fastest way to see what deemesh does:

1. Download **deemesh-hub** from Releases. On Windows, take the portable archive (`-win-portable.zip`) and unzip it. To run the hub as a Windows service instead, take the installer (`-win-setup.exe`): it asks for an admin code and a port, starts the service, and you continue at step 3.
2. Run `deemesh-hub-x64.exe` (Windows will ask once whether to allow it through the firewall. Click **Allow** if other PCs will call it; it is not a virus warning). It starts with default settings and writes `deemesh-hub.toml` (where the hub listens and who may call it) and `deemesh-data/hub/machines.json` (the machine list).
3. Open **`http://localhost:8080/admin`** in a browser, **Log in** with the admin code the hub wrote in `deemesh-hub.toml` (`admin_code`), press **Edit**, add your machine with **+ Add machine** (its IP and protocol) and press **Save**.
4. Open the **address catalog** (`/admin/address`) to:
   - browse a tree of **every address deemesh supports**, with which protocols read and write each one, and
   - select any address and **run a GET (read) right there**, or a POST (write) once that machine's `write_enabled` is turned on (writes are off by default).

Connect it to a real machine and you can try, right away and in the browser, what reads and writes on that machine. If you want to evaluate before committing, this catalog is the front door.

```bash
# Calling it directly is just HTTP
curl "http://localhost:8080/machine/channel/axis/machineposition?machine=1&channel=1&axis=1"
# → {"status":0,"value":123.456}
```

> The hub is designed for a **trusted network inside the plant**. Access control is three IP whitelists (read, write and admin), a write switch per machine and an admin code, with HTTPS available; there is no built-in user login. To expose it beyond that, put it behind an authenticating reverse proxy. Details in the hub guide.

---

## Integrate it into your app: the SDK

The download package is the native library plus its C header, a **standard C ABI**, so you call it with whatever FFI your language already has. No wrapper to install, nothing to keep in sync with us.

| Language | How you call it |
|---|---|
| **C / C++** | Include `include/deemesh.h`, link the `.dll` / `.so` |
| **C#** | `[DllImport]` / P/Invoke |
| **Python** | `ctypes` (standard library) |
| **Rust** | `extern "C"`, or load at runtime with `libloading` |
| **Java, Go, others** | JNI / Panama, cgo, … |

There are nine functions in total. `deemesh.h` is the authoritative declaration (signatures, the `DeemeshResult` layout, and the error-code macros) and `FFI_EN.md` / `FFI_KO.md` in the download walk through calling them, with worked examples in C#, Python and C/C++.

```c
DeemeshHandle h = NULL;
deemesh_create("{\"protocol\":\"nc_focas2_fanuc\",\"host\":\"192.168.1.100\","
               "\"port\":8193,\"library_path\":\"fwlib32.dll\"}", &h);
deemesh_connect(h);
DeemeshResult r = deemesh_read(h, "/machine/channel/motionstatus?channel=1", 10000);
printf("%s\n", r.data);          // {"status":0,"value":1,"desc":"Motion"}
deemesh_free_string(r.data);
/* create/connect also return a DeemeshResult - free their .data the same way
   (omitted here for brevity; the FFI guide shows the full pattern) */
```

> Configuration, commands and results are all JSON, so the same config object and the same
> response envelope apply in every language. The full address list lives in the catalog
> ([English](docs/CATALOG_EN.md) / [한국어](docs/CATALOG_KO.md)); command grammar, calling
> conventions and deployment guides are in the `docs/` folder of the download. Every
> document ships in both English (`*_EN.md`) and Korean (`*_KO.md`).

---

## Command structure at a glance

```
GET  /machine/channel/axis/machinePosition?channel=1&axis=1              ← read
POST /machine/channel/variable/variableValue?channel=1&variable=100      ← write
     Body: {"value": 3.14}
     └─────────── address (what) ───────────┘ └──── filter (which) ────┘
```

- **GET reads, POST writes.** Responses are always JSON: a read answers `{"status":0,"value":...}`, a write `{"status":0}`.
- Every write takes the same `{"value": ...}` envelope.
- Addresses and filters are vendor-neutral: the same address is used on Fanuc, Siemens, Mitsubishi and Heidenhain, and where the controls differ the catalog states it for that address.

> ⚠️ **Writes change the real state of the machine.** Verify the target and the value, and
> test in a safe environment before using it on production equipment. See the package
> documentation and the LICENSE for details.

---

## Supported protocols

| Identifier | Equipment |
|---|---|
| `nc_focas2_fanuc` | Fanuc CNC (FOCAS2: Windows and Linux) |
| `nc_opcua_siemens` | Siemens SINUMERIK CNC (OPC-UA) |
| `nc_ezsocket_mitsubishi` | Mitsubishi Electric CNC (EZSocket: M800/M700 series, mill and lathe; Windows only) |
| `nc_dnc_heidenhain` | Heidenhain CNC (HEIDENHAIN DNC: TNC7, TNC 640, TNC 620, TNC 320, TNC 128; Windows only. The catalog's `read` and `write` lists show the addresses that work on it) |

> **Each control has prerequisites of its own. Check them before you download.**
>
> - **Fanuc** needs Fanuc's own FOCAS2 library: `fwlib32.dll` / `fwlib64.dll` on Windows,
>   the 64-bit library on Linux (for example `libfwlib64.so.1.0.5`; your file name may differ); some packages name the Windows
>   library `FOCAS32.dll` / `FOCAS64.dll`, and the functions it includes vary by package: a write that needs a
>   function your library lacks answers status `-20`, and the error text names the function.
>   It is covered by Fanuc's license and is **not bundled with
>   deemesh**, so obtain it from Fanuc and point `library_path` at it. The hub takes
>   **either bitness on Windows**: it reads the DLL and hosts it in a matching process by itself.
> - **Siemens** needs the **OPC-UA server option licensed and enabled** on the machine
>   (a separately purchased Siemens option). A machine without it refuses the
>   connection outright. Support has been tested on **SINUMERIK
>   840D sl**; 828D has not been tested, so some addresses may not be available there.
> - **Mitsubishi** needs **Mitsubishi Electric's EZSocket (`FCSB1224W100`) installed** on the PC (obtained
>   from Mitsubishi Electric; a 32-bit Windows component), and the machine's control
>   series in the `system_type` setting. The hub hosts the Mitsubishi adapters in a
>   32-bit child process on its own, so the hub itself stays 64-bit. Mills and lathes of
>   different series can be connected together, on the hub and in one SDK process alike.
> - **Heidenhain** needs **HEIDENHAIN DNC installed** on the PC (obtained from Heidenhain; a Windows
>   component with 32-bit and 64-bit variants), **software option 18 (HEIDENHAIN DNC)** enabled on the
>   control, and either the control model in the `system_type` setting or the name of a connection
>   registered with HEIDENHAIN DNC on the PC in `connection_name` (secure connections, RPC over SSH, use
>   this). Without registration the connection is unencrypted RPC to port `19003`, and the control
>   connects back to the PC for events, so the PC's firewall must allow that.
>   The hub adapts to whichever variant is installed.
>
> The vendor pieces (the FOCAS2 library, the OPC-UA option, EZSocket, HEIDENHAIN DNC) are not something
> deemesh can supply; they come from the machine's vendor. One note for **SDK** users:
> the child-process conveniences above belong to the **hub**. The SDK library itself must
> match your process (bitness for `fwlib*` and HEIDENHAIN DNC, Mitsubishi on the 32-bit Windows library only).
> The FFI guide in the download spells this out.
>
> **Correction:** this page and the documentation of releases 1.1.0 through 1.4.1 said that
> EZSocket lets one process use only one Mitsubishi control series. **That was a mistake in
> our testing, not a behavior of Mitsubishi's EZSocket**; repeating the test with the same
> EZSocket version did not reproduce it.

> **Test environment**: this release was tested in three kinds of environment: a **simulator** (control software running on a PC), a **bench** (a real control with no machine attached) and a **machine tool** (a complete machine). For Siemens, a SINUMERIK 840D sl bench; for Fanuc, a FANUC Series 31i bench, a 31i-B machine tool and the FANUC NC Guide simulator; for Mitsubishi, the NC Trainer2 plus simulator (M800 series); for Heidenhain, the TNC7 programming station simulator. On other models some addresses may not be available; the catalog states support per address.

> If you need a specific machine or protocol, get in touch.

---

## Download

Browse **[all releases](https://github.com/spoonhasi/deemesh/releases)**, or jump straight to the latest:

| Product | Latest release | Contents |
|---|---|---|
| **deemesh-sdk** | **[⬇ deemesh-sdk v1.8.0](https://github.com/spoonhasi/deemesh/releases/tag/deemesh-sdk-v1.8.0)** | Windows (32/64) · Linux (64) native libraries + C header + docs (KO/EN) |
| **deemesh-hub** | **[⬇ deemesh-hub v2.0.0](https://github.com/spoonhasi/deemesh/releases/tag/deemesh-hub-v2.0.0)** | Standalone HTTP server: `win-setup` (Windows service installer) · `win-portable` · `linux-x64`. Each download is self-contained (no separate SDK needed) |

Each download contains a `README`, documentation, `LICENSE` and `THIRD-PARTY-NOTICES`. The Windows hub comes two ways with the same files: the **installer** installs it as a Windows service (the way to run it as a service: it locks the install folder so that only administrators can modify it), and the **portable** archive is unzipped and run. Both carry both bitnesses: run `deemesh-hub-x64.exe` and keep the two files together; bitness (yours, and your Fanuc DLL's) is the hub's problem, not yours. The installer and the executables are not code-signed, so Windows may show "Windows protected your PC": choose **More info**, then **Run anyway**, and check the download against its `.sha256`. The SDK and the hub are versioned independently; take the latest of each.

---

## License

**Freeware.** Free to use and redistribute, **including commercially**. The source is not published, and the software is provided **without warranty or liability**. See [LICENSE](LICENSE) for the full terms.

## Trademarks

FANUC, FOCAS, SIEMENS, SINUMERIK, Mitsubishi Electric, EZSocket, HEIDENHAIN, TNC, OPC-UA and all other product and company names mentioned here are trademarks or registered trademarks of their respective owners and are used for identification only. deemesh is an independent project; it is not affiliated with, sponsored by, or endorsed by those companies, and it does not distribute their software.

---

## About this project

A machine tool exposes a great many properties, and each control describes them in its own way. deemesh exists to make those properties **easy to find and easy to reach**: one address model that a person can read and a program can call, whichever control is behind it. Which addresses a particular machine answers depends on its control and its installed options, and the catalog records this for each address. Such a model depends on **consistency**: an address means the same thing on every control that supports it, and where a value is the vendor's own numbering that cannot be unified, such as PLC addresses or parameter numbers, the catalog says so. To protect that, the address model is curated by a single maintainer under one set of principles rather than by open contribution. The interface itself is public, and suggestions are welcome.

It is distributed free of charge in order to spread that interface as widely as possible.

---

## Custom development

If you would like this interface adapted to your site or your product, or support for a particular machine or protocol, please get in touch:

**spoonhasi@gmail.com**

---

Copyright © 2026 spoonhasi. Freeware (see [LICENSE](LICENSE)).
