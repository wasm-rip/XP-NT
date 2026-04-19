# Windows XP Source Code

This repository contains the source tree for the Windows NT 5.1 / XP codebase (codename **Whistler**; this drop is the **Amarillo** / XP SP1 branch). It is a complete operating-system source tree — kernel, drivers, core subsystems, shell, Internet Explorer, IIS, networking stack, Terminal Services, Active Directory, DirectX, and the NT SDK/DDK public headers.

## Scale

| Metric | Count |
|---|---|
| Total files | ~278,000 |
| C / C++ / ASM source + headers | ~107,000 |
| Lines of C / C++ / ASM | ~3.78 million |
| `.rc` resource files | ~6,574 |
| `.def` export files | ~3,661 |
| Makefiles / `sources` files | ~551 |
| Top-level subsystems | 20 |

## Languages

- **C** — kernel, drivers, services (~24,900 files)
- **C++** — user-mode apps, COM components, shell, IE (~29,400 files)
- **x86 / AMD64 / IA64 / Alpha assembly** — context switching, interrupts, trap handlers, low-level math (~1,430 files)

## Directory Layout

| Directory | Contents |
|---|---|
| `admin/` | Active Directory admin tools, DCPromo, DNS admin, Group Policy, event log utilities, MMC consoles |
| `base/` | NT kernel (`ntos`), executive subsystems, HAL, boot code, base runtime, clustering, core drivers |
| `com/` | COM / OLE32, RPC runtime, COM+ (MTS), mobile services, COM test harnesses |
| `developer/` | Developer / build environment configuration (Anvil) |
| `drivers/` | Device drivers: input, network, serial, sound, smart card, dot4, parallel, APM |
| `ds/` | Directory Services: ADSI, Active Directory, DNS, JET/ESE, Novell interop, network APIs |
| `enduser/` | End-user applications: NetMeeting, Windows Media, Speech, troubleshooters, downlevel clients |
| `inetcore/` | Internet Explorer, MSHTML, URLMon, WinInet, WinHTTP, Outlook Express, Passport, Digest auth |
| `inetsrv/` | IIS, MSMQ, Indexing Service / Query, server-side internationalization |
| `loc/` | Localization resources and binaries for multilingual builds |
| `mergedcomponents/` | Combined components shipped as integrated units (advapi32, uuid, WMI stubs, setup) |
| `multimedia/` | DirectX, DirectShow, Direct3D, OpenGL, DANIM, audio/video stack |
| `net/` | TCP/IP, NDIS, NetBIOS, NetBEUI, DLC, ATM, IrDA, IPsec, DHCP, DNS, routing protocols |
| `printscan/` | Print subsystem, print drivers, FAX services, WIA scanner stack, print UI |
| `public/` | Public SDK / DDK / OAK / HAL kit headers and import libraries |
| `published/` | Generated / exported header sets (`genxddk`, `genxwin`) |
| `sdktools/` | ~240 subdirs of build tools, debuggers, profilers, API monitors, verifiers |
| `shell/` | Explorer, taskbar, Start menu, common controls, `browseui`, IE shell integration |
| `termsrv/` | Terminal Services / Remote Desktop: session manager, licensing, admin, RDP filter |
| `tools/` | Top-level build infrastructure, Perl build scripts, post-build `.cmd` scripts, machine configs |
| `windows/` | Win32 subsystem (USER, GDI, console, CSRSS) |

## Kernel (`base/ntos/`)

The full NT executive and kernel source, organized by subsystem:

| Subsystem | Role |
|---|---|
| `ke/` | Kernel Executive — scheduler, dispatcher, traps, context switch (per-arch: `i386`, `amd64`, `ia64`, `alpha`) |
| `mm/` | Memory Manager — virtual memory, paging, working sets, section objects |
| `io/` | I/O Manager — IRP dispatch, Plug-and-Play, driver stack |
| `ob/` | Object Manager — namespace, handles, reference counting |
| `se/` | Security Reference Monitor — tokens, SIDs, ACLs, access checks |
| `ps/` | Process / thread management |
| `ex/` | Executive support — pools, work items, fast mutexes, resources |
| `config/` | Registry / configuration manager, hive load and sync |
| `cache/` | Cache Manager |
| `rtl/` | Kernel runtime library |
| `lpc/` | Local Procedure Call (inter-subsystem IPC) |
| `dbgk/` | Kernel debugging support |
| `po/` | Power Manager |
| `wmi/` | Windows Management Instrumentation kernel side |
| `vdm/` | Virtual DOS Machine for 16-bit compatibility |

## Build System

This tree uses the Microsoft internal **Razzle** build environment, driven by `nmake`:

- **`makefil0`** (repo root) — top-level build coordinator. Phases: `echobldmsg`, `set_builddate`, `set_buildnum`, `cleanup`, `scorch`.
- **`dirs`** files — each directory declares a `DIRS=` list that defines traversal order.
- **`sources`** files — per-leaf-directory manifests that declare sources, target type, dependencies, and compilation flags.
- **`binplace`** — utility that lays out built binaries into the distribution tree.
- **Perl scripts** under `tools/` — `updtnum.pl`, `genbldno.pl`, `scorch.pl` manage build numbers and clean state.
- **Post-build `.cmd` scripts** — handle signing, compression, and packaging.

Required environment variables, set up by Razzle: `NTMAKEENV`, `RAZZLETOOLPATH`, `_NTDRIVE`, `_NTROOT`, `_NTBINDIR`, `BUILD_ALT_DIR`.

Typical flow:

```
setenv.bat                # initialize Razzle environment
build -cZ                 # clean, then full build
build -M 4                # parallel build with 4 workers
```

Individual components can be built by `cd`-ing into the component directory and running `build` there.

## Target Architectures

The tree carries per-architecture code paths for:

- **i386** — primary 32-bit x86 target
- **amd64** — 64-bit x86 (early XP-era support)
- **ia64** — Intel Itanium
- **alpha** — DEC Alpha (legacy paths present in some subsystems)

Architecture-specific assembly and C lives under `i386/`, `amd64/`, `ia64/`, `alpha/` subdirectories inside each component.

## Notable Components

- **Shell** (`shell/`) — Explorer, taskbar, Start menu, common controls, `browseui`, trident integration
- **Internet Explorer** (`inetcore/`, `shell/iexplore`) — MSHTML layout engine, URLMon, WinInet
- **DirectX / Multimedia** (`multimedia/`) — DirectShow, Direct3D, OpenGL, DANIM
- **Terminal Services** (`termsrv/`) — RDP server, session manager, licensing
- **Active Directory** (`ds/`, `admin/`) — DS engine, JET/ESE database, replication, admin UI
- **Networking** (`net/`) — TCP/IP v4, NDIS 5.x, DHCP / DNS clients and servers, IPsec, routing
- **IIS** (`inetsrv/`) — web server, MSMQ, indexing/query
- **Print / Imaging** (`printscan/`) — spooler, print drivers, FAX, WIA
- **Security** — Kerberos, SChannel, CryptoAPI, Passport, Digest, LSA
- **Clustering** (`base/cluster`) — failover clustering

## Repository Layout Notes

- `public/` holds the headers and import libraries that are shipped externally as the SDK / DDK / OAK.
- `published/` holds machine-generated export sets derived from `public/` and internal sources.
- `mergedcomponents/` contains components whose source is combined from multiple upstream trees into a single shippable unit.
- `sdktools/` is the largest directory by subdirectory count — it carries the internal toolchain used across the rest of the tree.

## Working With the Tree

Because the tree is built top-down through `dirs` files, edits should:

1. Modify source and `sources` files in the leaf directory.
2. Run `build` inside that directory for incremental compilation.
3. If adding or removing leaf directories, update the parent `dirs` file.
4. Headers exported to other components live under `public/sdk/inc/` or `public/internal/` — changes there trigger wide rebuilds.

## Layout Summary

```
amarillo/
├── makefil0              # top-level build driver
├── dirs                  # top-level directory list
├── admin/   base/   com/       developer/  drivers/
├── ds/      enduser/ inetcore/ inetsrv/    loc/
├── mergedcomponents/     multimedia/       net/
├── printscan/ public/ published/ sdktools/
├── shell/   termsrv/ tools/   windows/
└── empty/                # placeholder for build-tree scaffolding
```
