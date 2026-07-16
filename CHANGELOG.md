# UBUNTU24-STIG-Audit

## 1.5.0 - 2026 July QA

QA pass at the existing V1R5 release (no benchmark version change):

- UBTU-24-200270: corrected the cron-audit goss file test to check `/etc/audit/rules.d/stig.rules` (the file the remediation writes) instead of `/etc/audit/rules.d/audit.rules`, and fixed the `contents` regex to match audit rule-file syntax (`-w /etc/cron.d/ ...`) rather than `auditctl -w ...` (which only appears in `auditctl -l` output). The companion `command:` subtest was already correct.
- UBTU-24-400220: corrected the goss test `title:` identifier from `UBTU-24-611045` to `UBTU-24-400220`; the filename, `STIG_ID`, `Rule_ID`, and toggle were already correct, so this was a report-label fix only.

## 1.5.0 - 2026 June QA

QA pass at the existing V1R5 release (no benchmark version change):

- Added `ubtu24stig_300019` and `ubtu24stig_300020` toggles to `vars/STIG.yml` (was 192 of 194). The goss tests already existed and gate on these toggles, so standalone `run_audit.sh` runs now evaluate both rules correctly.
- `run_audit.sh`: derive the audit content directory directly from `BENCHMARK_OS` (`audit_content_version=$BENCHMARK_OS-$BENCHMARK-Audit`) instead of runtime OS detection (uname/hostnamectl/os-release greps), removing the now-redundant detection-fallback block. Resolves to the same `UBUNTU24-STIG-Audit` path but deterministically, avoiding wrong paths on minimal containers / non-GNU greps. Fixed two message typos.

## Based on STIG v1r5

## 1.5.0

2026 May Updates

Audit-side counterpart to the V1R3 -> V1R5 cumulative upgrade. ALD did NOT cut a V1R4 release branch - the deltas below are the union of V1R3 -> V1R4 and V1R4 -> V1R5 changes.

Benchmark version strings:
- vars/STIG.yml: `benchmark_version` `v1.3.0` -> `v1.5.0`.
- run_audit.sh: `BENCHMARK_VER` `1.3.0` -> `1.5.0`. `BENCHMARK_OS=UBUNTU24` unchanged.

Goss test additions (V1R3 -> V1R5):
- cat_1/UBTU-24-100050.yml - NEW for the V1R5 NFS package rule. Verifies `nfs-common` and `nfs-kernel-server` are not installed.
- cat_2/UBTU-24-40xxxx/UBTU-24-400360.yml - companion goss for the newly-added remediation task. Uses a command-based check that treats missing `/etc/sssd/sssd.conf` as a pass (SSSD not configured), and otherwise verifies `pam_cert_auth` and `services` entries.
- cat_1/UBTU-24-700400.yml - pre-existing audit-side gap. Verifies Ubuntu 24.04 LTS is within the Canonical standard support window (through June 2029).

Goss test removals (V1R3 -> V1R5):
- cat_3/UBTU-24-30xxxx/UBTU-24-300024.yml - removed alongside the remediation task; rule was withdrawn in V1R5.

Rule_ID drift (8 files, V1R3 -> V1R5 cumulative):
- cat_1/UBTU-24-300025.yml - SV-270711r1101772 -> SV-270711r1184069
- cat_2/UBTU-24-10xxxx/UBTU-24-100110.yml - SV-270650r1134802 -> SV-270650r1155241 (both meta blocks)
- cat_2/UBTU-24-60xxxx/UBTU-24-600150.yml - SV-270750r1117267 -> SV-270750r1137695
- cat_2/UBTU-24-70xxxx/UBTU-24-700020.yml - SV-270757r1066760 -> SV-270757r1184072 (both meta blocks)
- cat_2/UBTU-24-70xxxx/UBTU-24-700060.yml - SV-270761r1067180 -> SV-270761r1184074
- cat_2/UBTU-24-70xxxx/UBTU-24-700070.yml - SV-270762r1066775 -> SV-270762r1184076
- cat_2/UBTU-24-70xxxx/UBTU-24-700080.yml - SV-270763r1066778 -> SV-270763r1184078
- cat_2/UBTU-24-70xxxx/UBTU-24-700090.yml - SV-270764r1066781 -> SV-270764r1184080

V1R5 Fix-text content updates (journal permissions aligned to Canonical):
- UBTU-24-700020 - hardcoded `2640` -> `0640` for `/run/log/journal`, `/run/log/journal/%m`, `/var/log/journal`, and `/var/log/journal/%m` (the `system.journal` entry was already `0640`).
- UBTU-24-700060 - hardcoded `2640` -> `0640` for `/run/log/journal` and `/var/log/journal`.
- UBTU-24-700070 - regex pattern `~2640` -> `~0640` and `2640` -> `0640` for the journal-by-machine-id entries.
- UBTU-24-700080 - hardcoded `2640` -> `0640` for `/run/log/journal` and `/var/log/journal`.
- UBTU-24-700090 - regex pattern `~2640` -> `~0640` and `2640` -> `0640` for the journal-by-machine-id entries.

Toggle alignment (vars/STIG.yml):
- Added `ubtu24stig_100050` (new V1R5 rule) to CAT1 list.
- Removed `ubtu24stig_300024` (rule withdrawn) from CAT3 list.
- Moved `ubtu24stig_700040` from CAT1 list to CAT2 list (correct severity per V1R5 XCCDF).
- Moved `ubtu24stig_700400` from CAT2 list to CAT1 list (correct severity per V1R5 XCCDF, HIGH).

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
