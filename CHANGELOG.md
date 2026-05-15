# UBUNTU24-STIG-Audit

## Based on STIG v1r3

## 1.3.0

2026 May Updates

QA hygiene sweep paired with the Private-UBUNTU24-STIG `benchmark_v1r3` branch QA pass. No goss test logic changes - metadata + convention only.

- vars/STIG.yml: benchmark_version '1.3.0' -> 'v1.3.0' to align with Lockdown convention and the remediation repo's defaults/main.yml value.
- README.md: fix UB22 template leak in role description ("audit STIG of Ubuntu22 servers" -> "audit STIG of Ubuntu 24.04 servers").

Audit Rule_ID corrections - 13 files aligned with V1R3 SCAP canonical SV-* values (post-fix verification: 0 mismatches against `U_CAN_Ubuntu_24-04_LTS_V1R3_STIG_SCAP_1-3_Benchmark.xml`).

Critical metadata bugs corrected:
- UBTU-24-900110 - SV-2270782 (typo, leading 2) -> SV-270782r1066835
- UBTU-24-900140 - SV-2270785 (typo, leading 2) -> SV-270785r1068373
- UBTU-24-900120 - SV-270782 (was 900110's rule_id - copy-paste error) -> SV-270783r1066838
- UBTU-24-909000 - SV-270782 (was 900110's rule_id - copy-paste error) -> SV-270832r1068399
- UBTU-24-700130 - SV-270769 (wrong SV number) -> SV-270768r1066793
- UBTU-24-400020 - STIG_ID metadata 'UBTU-24-611045' (UB22 template artifact) -> 'UBTU-24-400020'
- UBTU-24-400030 - 3 instances of 'UBTU-22-400030' (UB22 template leak) -> 'UBTU-24-400030'

rN revision-suffix corrections (CMS-renumbering artifact - base SV-NNNNNN was already correct):
- UBTU-24-102000 - SV-270675r1117265 -> SV-270675r1137691
- UBTU-24-600030 - SV-270744r1117272 -> SV-270744r1137699
- UBTU-24-100860 - SV-270671r1067118 -> SV-270671r1155244
- UBTU-24-102010 - SV-270676r1068360 -> SV-270676r1155245
- UBTU-24-200270 - SV-274870r1107304 -> SV-274870r1155243
- UBTU-24-600070 - SV-270746r1101769 -> SV-270746r1155242
- UBTU-24-400340 - SV-270734r1066691 -> SV-270734r1155240
- UBTU-24-600140 - SV-270749r1117267 -> SV-270749r1137695

### Post-QA continuation (May 2026)

Follow-up work after the initial QA hygiene sweep above, surfaced when the paired remediation repo's molecule converge ran goss against this audit content.

Docs and metadata:
- LICENSE: copyright line was `Copyright (c) 2021 MindPoint Group` (untouched since the initial commit). Updated to `Copyright (c) 2026 MindPoint Group - A Tyto Athene Company / Ansible Lockdown` matching the canonical line in the paired remediation repo.
- README.md: replaced a stale paragraph `Set of configuration files and directories to run the first stages of STIG RHEL9 based servers` (RHEL-template leakage) with the correct `STIG Ubuntu 24.04 LTS based servers`.
- CONTRIBUTING.md: header was `Contributing to MindPoint Group Projects` with a divergent 6-rule structure left over from before the Ansible-Lockdown branding consolidation. Replaced with canonical content matching the paired remediation repo's CONTRIBUTING.rst (header `Contributing to Ansible-Lockdown Projects`, 5-rule structure, body references updated). Markdown format preserved.

Audit-test bug fixes surfaced by goss runs against a converged container:
- UBTU-24-100840 (KexAlgorithms FIPS check): `contents` pattern `/^KexAlgorithms {{ .Vars.ubtu24stig_sshd_config_kex }}/` interpolated the variable into a regex. The value contains `diffie-hellman-group-...` substrings; Go's regex parser interpreted the `n-g` substring inside `hellman-group` as an invalid character class range (`g` < `n`) and the test errored out. **Fixed by switching from slash-delimited regex to slash-less literal-substring match** — goss treats slash-less patterns under `contents:` and `stdout:` as `contains` matches, bypassing the Go regex parser entirely. Final patterns: `'KexAlgorithms {{ .Vars.ubtu24stig_sshd_config_kex }}'` and `'kexalgorithms {{ .Vars.ubtu24stig_sshd_config_kex }}'`. Preserves the full algorithm-list verification AND keeps `ubtu24stig_sshd_config_kex` actively consumed by the audit (a transitional fix that weakened the check to `/^KexAlgorithms /` was reverted — see commit history).
- UBTU-24-300028 (PAM nullok check): test name typo `nullok_commn-password` -> `nullok_common-password`. The stdout matcher `!/.*/ ` meant "stdout must NOT match anything" — fails whenever grep produced any output at all. Replaced with `!/nullok/` which is the semantic intent: no line containing 'nullok' must be present. Also collapsed an extra space between the two grep file arguments.

Dead toggle removed:
- `ubtu24stig_900500` removed from `vars/STIG.yml`. Not in V1R3 SCAP, not in V1R5 manual XCCDF — pure placeholder with no audit test referencing it. Paired removal in the remediation repo's `defaults/main.yml` and goss vars template.

run_audit.sh hardening:
- Removed `grep -w VERSION_ID=` (fragile on non-GNU greps; macOS/busybox silently miss quoted values); replaced with anchored `grep "^VERSION_ID="`.
- Added BENCHMARK_OS fallback for empty `os_vendor` / `os_maj_ver` (lesson #41): when OS detection produces empty results on minimal containers or stripped `/etc/os-release`, derive vendor/version from `BENCHMARK_OS=UBUNTU24` (-> UBUNTU + 24) instead of silently building a wrong audit path.
