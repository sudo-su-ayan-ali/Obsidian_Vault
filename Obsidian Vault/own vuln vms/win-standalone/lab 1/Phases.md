We’ll develop OPMGR-QPATH-01 as a sequence of detailed implementation phases, with prerequisites, procedures, validation checks, evidence requirements, and exit criteria for each phase. We’ll use VMware Workstation and preserve the isolated, authorized training-lab scope.

The existing guide establishes two important constraints: OpManager 12.4.034 is still a candidate vulnerable build, and the precise vulnerability, plugin state, exploit compatibility, and resulting account identity must be validated before the lab is approved.

OPMGR-QPATH-01.txt

# OPMGR-QPATH-01 — Phased implementation plan

01Requirements and researchResolve prerequisites, validate CVE references, identify installer provenance, and establish release gates.Stage: Pre-build02VMware Workstation foundationCreate the isolated VMnet, define addressing, configure VM resources, and document isolation.03Windows Server 2019 baselineInstall the operating system, apply the selected baseline, configure networking, and capture a clean snapshot.04OpManager deploymentInstall the approved application/plugin combination and record versions, services, logs, and ports.05Vulnerability validationCheck the affected build range, prerequisites, public exploit provenance, and repeatable lab-only test results.06Foothold account and flagsEstablish least-privilege account behavior, flag ACLs, and per-run flag handling.07Unquoted service pathConfigure the single intended local privilege-escalation weakness and rule out accidental alternatives.08Isolation and QATest network boundaries, attack-path repeatability, flags, service behavior, and clean recovery.09Hints and documentationPrepare progressive hints, instructor evidence, solution walkthrough, and trainee brief.10Golden snapshot and releaseFreeze the validated VM, test restore/export procedures, and establish the cohort reset process.

## Phase 1 — Requirements and vulnerability research

Objective: Establish an evidence-based, reproducible build specification before creating the VM.

### Tasks

1. Confirm Windows Server 2019 Standard with Desktop Experience.
    
2. Confirm VMware Workstation version and host resource availability.
    
3. Obtain an authorized source for the intended OpManager installer.
    
4. Verify the vendor's CVE-2019-15106 advisory and the reported affected build range.
    
5. Identify the required APM plugin and establish its version independently of the main product.
    
6. Record the public PoC or Metasploit module's provenance, target support, prerequisites, and expected execution context.
    
7. Define the approved isolated subnet and instructor access method.
    
8. Establish the test evidence folder and release checklist.
    

### Evidence register

|Artifact|Required record|
|---|---|
|Installer|Source URL, filename, acquisition date, SHA-256|
|OpManager|Exact version/build and source of version evidence|
|APM plugin|Exact version/build and installation status|
|Vulnerability|CVE, vendor advisory, affected range and prerequisites|
|Exploit reference|Repository, revision, target compatibility and behavior|
|Lab network|VMnet, subnet, target IP and isolation tests|

### Exit criteria

- The installer is from an authorized source.
    
- The application and plugin versions are recorded separately.
    
- The version combination is consistent with the vendor advisory and intended scenario.
    
- The exploit's prerequisites and expected identity are documented.
    
- Any unresolved compatibility question is explicitly marked as a blocker.
    

Important: If the exact historical build cannot be sourced or the exploit assumptions cannot be validated, pause here and revise the design through an approved change. Do not substitute an unverified build and claim equivalence.

## Phase 2 — VMware Workstation foundation

Objective: Create a dedicated virtual network that prevents the target from reaching production systems or the public internet.

### Tasks

1. Open VMware Workstation's Virtual Network Editor with administrative privileges.
    
2. Create a dedicated host-only or custom internal VMnet.
    
3. Disable the virtual network's DHCP service if static addressing is used.
    
4. Check that the subnet does not overlap with the host's LAN, VPN, or other virtual networks.
    
5. Attach the target VM to this network only.
    
6. Do not use Bridged networking. Do not add a NAT adapter for convenience.
    
7. Leave the target's default gateway unset unless the approved lab architecture requires one.
    
8. Use the VMware console for administration rather than exposing RDP or WinRM to trainees.
    
9. Record the VMnet settings and capture screenshots as build evidence.
    

### Suggested starting configuration

|Setting|Initial value|
|---|---|
|Guest OS|Windows Server 2019 x64|
|vCPU|2–4|
|RAM|8 GB, subject to application requirements|
|Disk|100–120 GB, subject to actual install and data requirements|
|Network|One dedicated host-only/custom VMnet|
|Addressing|Static private IPv4|
|Internet egress|None|
|Administration|VMware console|

These are starting values, not verified minimum requirements for every OpManager build.

### Validation

- Confirm the guest has one virtual NIC.
    
- Confirm that NIC is connected to the intended VMnet.
    
- Inspect the guest's route table and ensure no unintended default route exists.
    
- Test application-port reachability only from the authorized lab client.
    
- Verify that external connectivity is blocked using routing, firewall, and virtual-network configuration evidence—not just a failed ping.
    

### Exit criteria

The VM network is documented, isolated, and tested before the application is installed.

## Phase 3 — Windows Server 2019 baseline

Objective: Establish a clean, repeatable operating-system baseline.

### Tasks

1. Install Windows Server 2019 Standard with Desktop Experience from approved installation media.
    
2. Set a unique local Administrator credential and store it in the organization's approved secret store.
    
3. Install VMware Tools if required.
    
4. Rename the host to a neutral lab hostname, such as `QPATH-WIN01`.
    
5. Configure its static IP address.
    
6. Keep it out of Active Directory and do not join a domain.
    
7. Record the Windows edition, build number, architecture, timezone, and patch state.
    
8. Enable Windows Defender Firewall.
    
9. Keep RDP, WinRM, and unrelated remote-management services outside the trainee-reachable scope.
    
10. Reboot and verify that the machine starts consistently.
    

Useful local verification commands:

```
Get-ComputerInfo |
    Select-Object WindowsProductName, WindowsVersion, OsBuildNumber, OsArchitecture

Get-NetIPConfiguration

Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction

Get-NetRoute -AddressFamily IPv4
```

These commands are for inventory and configuration validation; run them in an administrative PowerShell session on the guest.

### Snapshot checkpoint

After the operating-system baseline is complete, shut down cleanly and create a preliminary snapshot named:

`OPMGR-QPATH-01-OS-BASELINE`

This is a recovery point, not yet the final golden image.

## Phase 4 — OpManager deployment

Objective: Install the exact application and plugin combination selected in Phase 1.

### Tasks

1. Verify the installer's source and SHA-256 hash.
    
2. Install OpManager using the vendor's documented Windows procedure.
    
3. Record the running product's actual build.
    
4. Install the required APM plugin only if the validated vulnerability scenario requires it.
    
5. Record the plugin's actual build independently.
    
6. Confirm that the application starts and its web interface is reachable from the instructor workstation.
    
7. Enumerate listening ports and map them to their owning processes.
    
8. Record the application service names, service accounts, installation paths, and relevant log locations.
    
9. Configure the guest firewall to permit only the necessary application access from the authorized lab subnet.
    
10. Verify that no unnecessary application, database, or management ports are exposed.
    

Useful inventory commands:

```
Get-NetTCPConnection -State Listen |
    Sort-Object LocalPort |
    Format-Table LocalAddress, LocalPort, OwningProcess

Get-CimInstance Win32_Service |
    Where-Object {
        $_.Name -match 'OpManager|ManageEngine|AdventNet'
    } |
    Select-Object Name, State, StartMode, StartName, PathName
```

Do not assume port `8060` is correct for the chosen build; record the port observed in the actual installation.

### Exit criteria

The application operates normally, its version and plugin state are recorded, and its network exposure matches the lab scope.

## Phase 5 — Vulnerability validation

Objective: Determine whether the selected installation actually supports the intended initial-access exercise.

This phase is a mandatory release gate, not a formality.

### Validation sequence

1. Compare the installed OpManager build with the vendor's CVE-2019-15106 advisory.
    
2. Verify whether the APM plugin is present and which build it uses.
    
3. Compare the exact plugin/build combination with the public PoC's stated prerequisites.
    
4. Inspect the exploit source and its provenance before execution.
    
5. Determine which target platforms and execution contexts the exploit supports.
    
6. Perform the test only from the instructor-controlled client on the isolated VMnet.
    
7. If authorized exploit validation is required, use a benign proof-of-execution test approved by the lab owner.
    
8. Record the actual Windows identity and token privileges of the resulting process.
    
9. Review application logs and confirm that the test did not leave the application in an unstable state.
    
10. Restore the pre-test snapshot and repeat the test to establish reproducibility.
    

A version string, HTTP response, or tool success message is not sufficient evidence of successful command execution.

### Release decision

- Pass: The exact application/plugin combination is validated and yields the intended low-privilege identity.
    
- Fail: The test contradicts the expected vulnerability or execution context.
    
- Blocked: The exact build, plugin, exploit provenance, or prerequisite cannot be established.
    

Do not continue to final image approval while the vulnerability validation is blocked.

## Phase 6 — Foothold account and flags

Objective: Define the intended account boundary and protect the two flags.

### Requirements

- The foothold identity must not be a local administrator.
    
- `user.txt` must be readable at the foothold stage.
    
- `root.txt` must remain inaccessible until the intended privilege boundary is crossed.
    
- Flag values must be unique per run or cohort.
    
- The instructor's flag manifest must be stored separately from the trainee package.
    

The actual OpManager service account must be observed during Phase 4 and Phase 5. Do not assume that changing the service to `svc_opmanager` is harmless or supported.

### Flag locations

```
C:\Users\svc_opmanager\Desktop\user.txt
C:\Users\Administrator\Desktop\root.txt
```

Create and verify the flags using an administrator-controlled process. Test effective permissions using the actual low-privilege identity rather than relying solely on the ACL display.

### Exit criteria

The user flag is readable by the intended foothold identity, the root flag is protected, and the instructor can rotate flags without exposing their values to trainees.

## Phase 7 — Unquoted service path

Objective: Implement exactly one intended local privilege-escalation weakness.

### Design requirements

- One custom auto-start service.
    
- Service execution context: LocalSystem.
    
- An executable path containing spaces that is not quoted in the service configuration.
    
- A narrowly scoped directory write permission for the intended foothold identity.
    
- A controlled, benign service executable for repeatable QA.
    
- Two or three correctly configured decoy services.
    
- No additional writable-service-binary or unrelated privilege-escalation path.
    

### Validation evidence

Inspect the service configuration and filesystem permissions with:

```
sc.exe qc QPathBackupSvc

Get-CimInstance Win32_Service `
    -Filter "Name='QPathBackupSvc'" |
    Format-List Name, StartMode, StartName, PathName

icacls 'C:\Program Files\Backup Agent'
```

The final instructor walkthrough should explain how the service configuration, path parsing, permissions, and restart conditions combine to produce the intended weakness.

Do not release the image if the trainee can achieve the same privilege escalation through an accidental alternative misconfiguration.

## Phase 8 — Isolation and quality assurance

Objective: Verify that the machine is both intentionally vulnerable and operationally contained.

Test:

- Cold boot and snapshot restoration.
    
- Application startup and intended network exposure.
    
- Vulnerability-stage repeatability.
    
- Low-privilege identity and account permissions.
    
- Service configuration and privilege boundary.
    
- User/root flag access before and after the intended stages.
    
- Lack of internet egress and production-network reachability.
    
- Absence of unintended RDP, WinRM, database, or management-port exposure.
    
- No accidental alternative local privilege-escalation paths.
    

Pilot with two or three internal testers and record time-to-foothold, time-to-root, hint usage, dead ends, and any instability.

## Phase 9 — Progressive hints and training documentation

Deliver hints in sequence, starting with methodology rather than answers.

HINT 1

Reconnaissance

Reveal

HINT 2

Foothold research

Reveal

HINT 3

Local enumeration

Reveal

HINT 4

Service path

Reveal

HINT 5

Validation

Reveal

Keep the detailed exploit walkthrough and final service-path details in instructor-only documentation until the relevant hints are requested.

## Phase 10 — Golden snapshot and release

Objective: Freeze a tested image and make it recoverable between trainees.

1. Complete every acceptance test.
    
2. Shut down the guest cleanly.
    
3. Create the final snapshot `OPMGR-QPATH-01-GOLDEN-v1.0`.
    
4. Test the exact revert or clone workflow that instructors will use.
    
5. Verify that flags can be rotated and the manifest stays synchronized with the active run.
    
6. Export or package the VM using the approved VMware Workstation workflow.
    
7. Record the VMware version, required VMnet settings, resource allocation, export filename, and export checksum.
    
8. Store the instructor solution and flag manifest separately from the trainee-facing package.
    
9. Have the designated reviewer approve the release.
    

### Final release checklist

- Application/plugin version and CVE assumptions validated.
    
- Initial foothold is repeatable and uses the intended low-privilege identity.
    
- Only one intended local privilege-escalation path exists.
    
- Flag permissions and per-run rotation are verified.
    
- Network isolation is verified.
    
- Progressive hints and instructor documentation are complete.
    
- Snapshot restore and export procedures are tested.
    
- Pilot results are recorded and release is approved.
    

## Recommended working order

I recommend we complete the phases one at a time, beginning with Phase 1: Requirements and vulnerability research. That establishes the correct application/plugin build before time is spent building a VM around an unverified exploit assumption.

The first concrete output should be a version-validation dossier containing the vendor advisory, exact affected-build evidence, installer provenance, plugin requirements, exploit references, and a formal pass/fail/block decision.