# deemesh

**A single, vendor-neutral interface for CNC and industrial equipment.**

[![License: Freeware](https://img.shields.io/badge/license-Freeware-brightgreen.svg)](LICENSE)

> This repository hosts deemesh's **documentation and releases**. deemesh is free but **not
> open source** — this repo does not contain its source code. The "Source code" archives that
> GitHub attaches to each release contain only the files you see here.

> 🇰🇷 **한국어 사용자** — 제품명은 **디메시**(deemesh), 서버 제품은 **디메시 허브**(deemesh-hub)로 표기합니다.
> 다운로드 패키지에는 설정 가이드·명령 매뉴얼·주소 카탈로그 등
> **모든 문서가 한국어와 영어로 포함**되어 있습니다 (`*_KO.md` 한국어 원본 + `*_EN.md` 영어판).

---

## What is deemesh?

On a shop floor, every CNC vendor speaks its own protocol — Fanuc uses FOCAS2, Siemens uses OPC-UA, and so on. Connecting one machine means learning that vendor's SDK; connecting the next vendor means starting over. That fragmentation has been a long-standing barrier to equipment integration.

**deemesh puts one interface on top of all of them.** Whether the machine is a Fanuc or a Siemens, you read and write axis positions, alarms, programs and tool offsets through the same command structure:

```
/machine/channel/axis/machinePosition?channel=1&axis=1
```

That one address means the same thing on any supported machine. deemesh absorbs the vendor differences underneath.

**The interface itself is public** — browse the complete address catalog here: **[English](docs/CATALOG_EN.md)** · **[한국어](docs/CATALOG_KO.md)**

---

## Two ways to use it

| | What it is | Who it's for |
|---|---|---|
| **deemesh SDK** | A native library (`.dll` / `.so`) callable from C#, Python, C++ and Rust over the C ABI | Developers **integrating** equipment communication into their own application |
| **deemesh-hub** | A standalone HTTP server — plain GET/POST, no code required | Anyone who wants to use it **over HTTP without writing code**, or to **try it first** |

Both are free. Grab them from **Releases** below.

---

## Try it in 5 minutes — no code (deemesh-hub)

The fastest way to see what deemesh does:

1. Download **deemesh-hub** from Releases and unzip it.
2. Put your machine's IP and protocol in `config.json` (an example file is included).
3. Run `deemesh-hub-x64.exe`, then open **`http://localhost:8080/admin`** in a browser.
4. Open the **address catalog** (`/admin/address`) to:
   - browse a tree of **everything the machine exposes**, and
   - select any address and **run a GET (read) or POST (write) right there**.

Connect it to a real machine and you can see, immediately and visually, exactly what you can read and write. If you want to evaluate before committing, this catalog is the front door.

```bash
# Calling it directly is just HTTP
curl "http://localhost:8080/machine/channel/axis/machineposition?machine=1&channel=1&axis=1"
# → {"status":0,"value":123.456}
```

---

## Integrate it into your app — the SDK

The download package is the native library plus its C header — a **standard C ABI**, so you call it with whatever FFI your language already has. No wrapper to install, nothing to keep in sync with us.

| Language | How you call it |
|---|---|
| **C / C++** | Include `include/deemesh.h`, link the `.dll` / `.so` |
| **C#** | `[DllImport]` / P/Invoke |
| **Python** | `ctypes` (standard library) |
| **Rust** | `extern "C"`, or load at runtime with `libloading` |
| **Java, Go, others** | JNI / Panama, cgo, … |

There are nine functions in total. `deemesh.h` is the authoritative declaration — signatures, the `DeemeshResult` layout, and the error-code macros — and `FFI_EN.md` / `FFI_KO.md` in the download walk through calling them, with worked examples in C/C++ and C#.

```c
DeemeshHandle h = NULL;
deemesh_create("{\"protocol\":\"nc_focas2_fanuc\",\"host\":\"192.168.1.100\","
               "\"port\":8193,\"library_path\":\"fwlib64.dll\"}", &h);
deemesh_connect(h);
DeemeshResult r = deemesh_read(h, "/machine/channel/motionstatus?channel=1", 10000);
printf("%s\n", r.data);          // {"status":0,"value":1,"desc":"Motion"}
deemesh_free_string(r.data);
```

> Configuration, commands and results are all JSON, so the same config object and the same
> response envelope apply in every language. The full address list lives in the catalog
> ([English](docs/CATALOG_EN.md) / [한국어](docs/CATALOG_KO.md)); command grammar, calling
> conventions and deployment guides are in the `docs/` folder of the download — every
> document ships in both English (`*_EN.md`) and Korean (`*_KO.md`).

---

## Command structure at a glance

```
GET  /machine/channel/axis/machinePosition?channel=1&axis=1              ← read
POST /machine/channel/variable/variableValue?channel=1&variable=100      ← write
     Body: {"value": 3.14}
     └─────────── address (what) ───────────┘ └──── filter (which) ────┘
```

- **GET reads, POST writes.** Responses are always JSON (`{"status":0,"value":...}`).
- Every write takes the same `{"value": ...}` envelope.
- Addresses and filters are vendor-neutral — the same address means the same thing on Fanuc and on Siemens.

> ⚠️ **Writes change the real state of the machine.** Verify the target and the value, and
> test in a safe environment before using it on production equipment. See the package
> documentation and the LICENSE for details.

---

## Supported protocols

| Identifier | Equipment |
|---|---|
| `nc_focas2_fanuc` | Fanuc CNC (FOCAS2, 32/64-bit) |
| `nc_opcua_siemens` | Siemens Sinumerik CNC (OPC-UA) |

> **Each control has one prerequisite of its own — check it before you download.**
>
> - **Fanuc** needs Fanuc's own FOCAS2 library: `fwlib32.dll` / `fwlib64.dll` on Windows,
>   `libfocas64.so` on Linux. It is covered by Fanuc's license and is **not bundled with
>   deemesh**, so obtain it from Fanuc and point `library_path` at it.
> - **Siemens** needs the **OPC-UA server option licensed and enabled** on the machine
>   (SINUMERIK Access MyMachine /OPC UA — a separately purchased option). A machine
>   without it refuses the connection outright. Support is **tested against SINUMERIK
>   840D**; **on 828D some features may not work**.
>
> Neither is something deemesh can supply, and neither is a deemesh setting — they live on
> the machine and with its vendor.

> Mitsubishi and Modbus I/O are on the roadmap. If you need a specific machine or protocol,
> get in touch.

---

## Download

Browse **[all releases](https://github.com/spoonhasi/deemesh/releases)**, or jump straight to the latest:

| Product | Latest release | Contents |
|---|---|---|
| **deemesh SDK** | **[⬇ deemesh-sdk v1.0.0](https://github.com/spoonhasi/deemesh/releases/tag/deemesh-sdk-v1.0.0)** | Windows (32/64) · Linux (64) native libraries + C header + docs (KO/EN) |
| **deemesh-hub** | **[⬇ deemesh-hub v1.0.0](https://github.com/spoonhasi/deemesh/releases/tag/deemesh-hub-v1.0.0)** | Standalone HTTP server — **pick your platform**: `win-x64` · `win-x86` · `linux-x64`. Each archive is self-contained (single binary, no separate SDK needed) |

Each archive contains a `README`, documentation, `LICENSE` and `THIRD-PARTY-NOTICES`. The hub is published per platform, so download only the one you run (Windows: pick the bitness that matches your Fanuc `fwlib32/64.dll`; Siemens-only sites can use either). The SDK and the hub are versioned independently — take the latest of each.

---

## License

**Freeware.** Free to use and redistribute, **including commercially**. The source is not published, and the software is provided **without warranty or liability**. See [LICENSE](LICENSE) for the full terms.

---

## About this project

deemesh is a **working implementation** of the *unified CNC interface* that has been discussed in the literature for years. A unified interface lives or dies by its **consistency**: once several parties extend it in different directions, the very property that made it valuable is gone. So the interface — the address model — is **maintained by a single author under one consistent set of principles**, rather than by open contribution.

It is distributed free of charge in order to spread that interface as widely as possible.

---

## Custom development

If you would like this interface adapted to your site or your product, or support for a particular machine or protocol, please get in touch:

**spoonhasi@gmail.com**

---

Copyright © 2026 spoonhasi. Freeware — see [LICENSE](LICENSE).
