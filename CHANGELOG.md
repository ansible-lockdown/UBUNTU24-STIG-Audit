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
