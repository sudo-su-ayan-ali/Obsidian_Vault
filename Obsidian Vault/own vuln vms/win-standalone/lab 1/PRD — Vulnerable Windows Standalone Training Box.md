# PRD — Vulnerable Windows Standalone Training Box

_Codename: OPMGR-QPATH-01_

_Internal training lab — OSCP+-style standalone Windows box_
  

# 1. Overview

This PRD defines a single, standalone, intentionally vulnerable Windows Server 2019 VM for internal offensive-security training. The box models a realistic two-stage compromise: an unauthenticated remote code execution foothold against an outdated, real-world monitoring product, followed by local privilege escalation via a classic Windows service misconfiguration. The design deliberately avoids Active Directory, chained/gamey CTF logic, and multiple unrelated privesc vectors, in favor of a clean, enumeration-driven path consistent with OSCP / OSCP+ standalone machine conventions.

## 1.1 Goals

- Train foothold-to-root workflow: service enumeration → CVE identification → public exploit usage → local enumeration → privesc → root flag.
    
- Reinforce version fingerprinting and CVE research discipline (not just "run a known script").
    
- Reinforce Windows service-permission enumeration (PowerUp / winPEAS / manual icacls, sc qc) as the sole privesc path, taught cleanly rather than diluted by a second vector.
    
- Produce a reusable, documented, re-deployable lab artifact (VM + build doc + solution guide) for repeated internal training cohorts.
    

## 1.2 Non-goals

- No Active Directory / domain component.
    
- No second, parallel privesc vector on this box (see Section 3.3 for rationale).
    
- Not intended for public CTF platform submission — internal use only, per scoping decision.
    
- Not a red-team infra exercise — no C2, no persistence stage, no lateral movement.
    

# 2. Target Audience & Use Case


 

| **Attribute**   | **Detail**                                                                                           |
| --------------- | ---------------------------------------------------------------------------------------------------- |
| Audience        | Internal security team (junior–intermediate pentesters, SOC analysts cross-training offense)         |
| Delivery        | Isolated internal lab network (host-only / internal vSwitch, no internet egress from target VM)      |
| Format          | Standalone VM, single flag per stage (user.txt, root.txt), OSCP-style writeup expected from trainees |
| Duration target | 45–90 min for intermediate trainee; used as a timed or untimed exercise per cohort                   |

# 3. Scenario Design

## 3.1 Attack Chain Summary

| **Stage**      | **Technique**                                                                                                                    | **Outcome**                                                    |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 1 — Recon      | Port/service scan, HTTP fingerprinting of OpManager instance                                                                     | Identify product + exact vulnerable build                      |
| 2 — Foothold   | Unauthenticated RCE — OpManager v12.4.034 auth-bypass RCE (DEFCON AppSec Village disclosure, Aug 2019; public Metasploit module) | Command execution as the OpManager service account (non-admin) |
| 3 — Local enum | Manual + tool-assisted service enumeration (sc qc, icacls, PowerUp/winPEAS)                                                      | Discover unquoted service path on a custom auto-start service  |
| 4 — Privesc    | Unquoted service path binary planting + service restart/reboot trigger                                                           | SYSTEM-level code execution                                    |


## 3.2 Foothold: Web Vulnerability Selection

Selected: ManageEngine OpManager v12.4.034 (or earlier), unauthenticated remote command execution via password-check bypass in the login flow, exploitable through a public Metasploit module and documented manual PoC (DEFCON AppSec Village disclosure, 2019). Deployed on IIS is not literal — OpManager ships its own bundled Java/Tomcat web stack on Windows; "IIS-hosted" framing in the original ask is replaced with the more realistic "bundled app server on Windows" pattern, which is the actual real-world shape of this bug class.

**Why a real CVE over a custom-built vulnerable app:**

- Trains version fingerprinting and CVE research, not puzzle-solving specific to a bespoke app.
    
- Real product footprint (install paths, service names, log formats) adds believable enumeration noise that is hard to fabricate convincingly.
    
- A mature public exploit (Metasploit module + manual PoC) mirrors real engagement workflow: adapt a public exploit rather than write one from scratch.
    
- Trade-off accepted: version pinning must be exact and documented, since this CVE's public tooling is version-sensitive (fails silently on the wrong build).
    

## 3.3 Privilege Escalation: Single-Vector Decision

**Decision: one privesc vector only — unquoted service path — not both unquoted-path and writable-service-binary as originally scoped.**

Rationale: on a single standalone box, two unrelated local privesc bugs tend to either (a) surface each other during the same enumeration sweep (e.g. a full icacls/sc qc pass for one candidate reveals the other by accident), cheapening both, or (b) require artificial separation via distinct service accounts that starts to feel like two boxes stapled together rather than one coherent scenario. Real OSCP/OSCP+ standalone boxes consistently favor one clean, well-signposted vector reinforced by decoy services and enumeration hygiene, over vector-stacking. If two techniques need to be taught, the recommended path is two separate boxes (or a linked "part 2" box), not one box with both.

Selected vector: unquoted service path on a custom auto-start service (e.g. "C:\Program Files\Backup Agent\backup service.exe" with no quotes), writable by the low-privilege foothold account. Chosen over writable-binary ACL because unquoted-path bugs test a subtler enumeration skill (recognizing the vulnerable pattern in `sc qc` / WMI output) versus writable-binary bugs, which are usually trivially obvious the moment icacls output is read. This is a judgment call, not a hard rule — flag it if your team's training goal is specifically ACL-misconfiguration literacy, in which case swap this for writable-binary instead.

## 3.4 Decoy / Noise Elements

- 2–3 additional installed services with normal (correct) permissions, to force genuine enumeration rather than "first weird-looking thing wins."
    
- A second, unrelated, non-exploitable outdated software install (patched/irrelevant CVE) to test CVE-research discipline (confirm exploitability, don't just pattern-match a product name).
    
- Standard Windows Server 2019 baseline noise (default services, scheduled tasks) left untouched — no artificial cleanup that would make the box fingerprint as "obviously a CTF box."
    

# 4. Technical Build Spec


| **Component** | **Spec** |
|---|---|
| Base OS | Windows Server 2019 (Standard, Desktop Experience), fully de-domained, static IP on isolated lab subnet |
| Vulnerable app | ManageEngine OpManager v12.4.034 (pin exact build), default install path, default service account (non-admin, e.g. `svc_opmanager`) |
| Network exposure | OpManager web/API port only (8060 or product default) + WinRM/RDP disabled from foothold's reachable scope; no outbound internet from VM |
| Privesc service | Custom Windows service, auto-start, running as SYSTEM, unquoted binary path, install directory writable by `svc_opmanager` or Authenticated Users |
| Flags | `C:\Users\svc_opmanager\Desktop\user.txt` (foothold-readable); `C:\Users\Administrator\Desktop\root.txt` (SYSTEM-readable only) |
| Flag format | Random UUID per deployment/cohort, regenerated per training run to prevent flag-sharing between cohorts |
| Snapshot policy | Golden snapshot post-build, revert-to-snapshot between every trainee run |

# 5. Deliverables


  
| **Deliverable** | **Description** | **Owner** |
|---|---|---|
| Build/deploy doc | Step-by-step VM build instructions (OS install → OpManager install/downgrade → service misconfig injection → snapshot) | TBD |
| Solution walkthrough | Full attacker-path writeup: recon → exploit → enum → privesc → flags, screenshots included | TBD |
| Trainee-facing brief | Scope, rules of engagement, objectives, flag submission process — no spoilers | TBD |
| Difficulty calibration notes | Expected time-to-foothold and time-to-root for junior vs intermediate trainees, from pilot run | TBD |
| VM export / snapshot | Packaged VM (OVA or Hyper-V export) + golden snapshot | TBD |

# 6. Risks & Mitigations


 
| **Risk** | **Mitigation** |
|---|---|
| Public exploit / Metasploit module version-sensitivity causes false negatives in training | Pin exact vulnerable build; validate exploit success during build QA before first cohort use |
| VM accidentally exposed outside isolated lab network | No outbound internet from VM; network isolation enforced at hypervisor/vSwitch level, not just firewall rules on the guest |
| Box becomes "solved knowledge" and loses training value after first cohort | Flag regeneration per run; consider minor service-name/path rotation between cohorts if reused long-term |
| Difficulty miscalibration (too easy/hard) | Pilot with 2–3 internal testers pre-rollout; adjust decoy density and hint availability based on pilot timing data |

# 7. Open Questions

- Hypervisor/export format — VMware, Hyper-V, or VirtualBox OVA? (affects build doc specifics)
    
- Should hints be provided in-platform, or fully unassisted per OSCP convention?
    
- Cohort cadence — single fixed box reused, or periodic minor variation to prevent solution-sharing across cohorts?
    
- Who owns build/deploy doc and solution walkthrough authorship (Section 5 owners are currently unassigned)?
    

  
  

_Note: OpManager v12.4.034 and the associated CVE/exploit tooling should be re-verified against current public sources at build time — exploit availability and version details can shift._