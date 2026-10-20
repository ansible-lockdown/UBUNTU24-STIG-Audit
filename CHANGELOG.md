# UBUNTU24-STIG-Audit

## 1.7.0 - Based on DISA STIG Ubuntu 24.04 LTS V1R7 (30 September 2026) - benchmark_v1.7.0

Benchmark bump V1R6 -> V1R7 (194 -> 195; 2 added, 1 removed, 1 severity change, 53 SV-rev bumps). Version strings: vars/STIG.yml v1.7.0, run_audit.sh BENCHMARK_VER 1.7.0, README banner + DISA zip V1R7.

Rule delta:

- **UBTU-24-101100 (V-288819, CAT II) is new: "must not assign an interactive login shell for system accounts".** `cat_2/UBTU-24-10xxxx/UBTU-24-101100.yml` runs the check-content `awk` over `/etc/passwd` (UID 1 to 999, excluding `halt`, `sync` and `shutdown`, shell not ending in `nologin` or `false`) and fails on any output. Toggle `ubtu24stig_101100`
- **UBTU-24-900315 (V-288818, CAT II) is new: "must generate audit records for successful/unsuccessful uses of the useradd command".** `cat_2/UBTU-24-900xxx/UBTU-24-900315.yml` follows the `usermod` check: the `-a always,exit -F path=/usr/sbin/useradd -F perm=x -F auid>=1000 -F auid!=-1` rule in a file under `/etc/audit/rules.d/`, and the loaded rule in `auditctl -l`. The key is not fixed, as the check-content allows. Toggle `ubtu24stig_900315`
- **UBTU-24-200680 (V-270694), the SSH acknowledgement of the Standard Mandatory Notice and Consent Banner, is withdrawn.** Its check file and the `ubtu24stig_200680` toggle are removed
- **UBTU-24-600090 (V-270747), cryptographic protection of data at rest, is now CAT I.** The check moves to `cat_1/`, is gated by `ubtu24stig_cat1`, carries `Cat: 1`, and its toggle moves to the CAT 1 list. The check logic is unchanged

Check changes:

- **UBTU-24-909000, "audit system must protect auditing rules from unauthorized change", requires `-e 2` at the start of a line.** The config test was `grep "\e 2"` with an unanchored pattern, so a commented `#-e 2` passed. It now greps `^-e 2` across `/etc/audit/rules.d/*.rules`, accepting a `99-finalize.rules` file as the check-content shows, and the live test matches `enabled 2` as a whole line
- **UBTU-24-909890, "must use cryptographic mechanisms to protect the integrity of audit tools", reads the AIDE drop-in directory.** It searches `/etc/aide/aide.conf` and every file under `/etc/aide/aide.conf.d/` and requires `auditctl`, `auditd`, `ausearch`, `aureport`, `autrace` and `augenrules` under both `/usr/sbin` and `/sbin`, each with exactly `p+i+n+u+g+s+b+acl+xattrs+sha512`. Commented lines no longer count. The 12 lines follow the fix text; the check-content regex omits `augenrules` and lists ten
- **UBTU-24-700020, UBTU-24-700060, UBTU-24-700080 and UBTU-24-700090 expect the systemd default journal permissions.** The V1R7 values are `z /run/log/journal 2755`, `Z /run/log/journal/%m ~2750`, `z /var/log/journal 2755`, `z /var/log/journal/%m 2755` and `z /var/log/journal/%m/system.journal 0640`, owner `root`, group `systemd-journal`, which is what Ubuntu 24.04 ships. The checks read the effective configuration from `systemd-tmpfiles --cat-config` and take the last line for each path, so the fix text's `/etc/tmpfiles.d/zzz-systemd-stig.conf` drop-in and an edited `/usr/lib/tmpfiles.d/systemd.conf` are both judged by what systemd applies. UBTU-24-700060 checks the group and UBTU-24-700080 and UBTU-24-700090 the owner, each with the V1R7 mode. The UBTU-24-700020 live test now fails a journal directory above `2755` or a journal file above `0640`; it previously flagged any directory with the owner execute bit, which every journal directory has, so a default `2755` directory failed
- **UBTU-24-100050, "must not have the nfs-kernel-server package installed", no longer fails on `nfs-common`.** Only `nfs-kernel-server` is checked
- **UBTU-24-400310 is "must be able to enforce a 180-day maximum password lifetime restriction".** The check accepts `PASS_MAX_DAYS` of 1 to 180 with any whitespace separator; it previously accepted 10 to 60, and only after a single space, so a tab-separated line, as Ubuntu ships it, never passed. `ubtu24stig_pass.max_days` moves from 60 to 180. V1R7 drops `CCI-004065` and the `SRG-OS-000730-GPOS-00190` Satisfies entry from the rule; the revision history does not list this change. The check-content calls a value "less than 180" a finding, which reverses its own title; this check treats a value above 180 as the finding
- **UBTU-24-600060, "must use DoW PKI-established certificate authorities (CAs)", accepts a DoD or a DoW CA file.** V1R7 greps for `DoW` and shows `DoW_PKE_CA_chain.pem` while still naming the "DOD ROOT CA" subject; the check passes on a file name containing `dod`, as before, or `dow` as a separate word, so `Windows` or `Shadow` do not match. The revision history does not list this change
- **UBTU-24-200270, "must audit any script or executable called by cron as root or by any privileged user", is unchanged.** V1R7 moves the fix to `/etc/audit/rules.d/stig.rules`, which the check already reads

Metadata:

- **Titles carry the V1R7 rule titles.** Seven changed in V1R7: the five DOD -> DoW renames (UBTU-24-100410, UBTU-24-200640, UBTU-24-200650, UBTU-24-200660, UBTU-24-600060), UBTU-24-400310, and UBTU-24-600090 (`data-at-rest`). Sixteen more still differed from V1R6: quotes the benchmark does not use (`"telnet"`, `"rsh-server"`, `"root"` in UBTU-24-300007, UBTU-24-300008, UBTU-24-300010, UBTU-24-300012, UBTU-24-300013, UBTU-24-901270), a missing final period (UBTU-24-100650, UBTU-24-100910, UBTU-24-200250) or an extra one (UBTU-24-700090, whose benchmark title has none), a missing space before the `|` suffix (UBTU-24-300026, UBTU-24-100820, UBTU-24-200610), and the `Personal Identity Verification` capitals in UBTU-24-400060
- **UBTU-24-100110, "must configure AIDE to perform file integrity checking", emitted nothing when `ubtu24stig_allow_aide_check` was false.** The `command:` key sat inside the toggle, so the manual placeholder landed at the top level and was ignored with an unknown-key warning. The key now opens before the toggle

SV-rev bumps (goss meta only; V1R7 changes nothing else in 17 of these rules):

- UBTU-24-100010 r1068357 -> r1268251: Ubuntu 24.04 LTS must not have the "systemd-timesyncd" package installed.
- UBTU-24-100020 r1068358 -> r1268251: Ubuntu 24.04 LTS must not have the "ntp" package installed.
- UBTU-24-100030 r1066430 -> r1261506: Ubuntu 24.04 LTS must not have the telnet package installed.
- UBTU-24-100050 r1156367 -> r1261502: Ubuntu 24.04 LTS must not have the nfs-kernel-server package installed.
- UBTU-24-100300 r1067143 -> r1268238: Ubuntu 24.04 LTS must have an application firewall installed in order to control remote access methods.
- UBTU-24-100310 r1067145 -> r1268238: Ubuntu 24.04 LTS must enable and run the Uncomplicated Firewall (ufw).
- UBTU-24-100410 r1066460 -> r1261509: Ubuntu 24.04 LTS must produce audit records and reports containing information to establish when, where, what type, the source, and the outcome for all DoW-defined auditable events and actions in near real time.
- UBTU-24-100650 r1067156 -> r1261510: Ubuntu 24.04 LTS must have the "SSSD" package installed.
- UBTU-24-100660 r1066478 -> r1261511: Ubuntu 24.04 LTS must use the "SSSD" package for multifactor authentication services.
- UBTU-24-100700 r1068359 -> r1268251: Ubuntu 24.04 LTS must have the "chrony" package installed.
- UBTU-24-100820 r1067107 -> r1268222: Ubuntu 24.04 LTS must configure the SSH daemon to use FIPS 140-3 approved ciphers to prevent the unauthorized disclosure of information and/or detect changes to information during transmission.
- UBTU-24-100830 r1067110 -> r1268237: Ubuntu 24.04 LTS must configure the SSH daemon to use Message Authentication Codes (MACs) employing FIPS 140-3 approved cryptographic hashes to prevent the unauthorized disclosure of information and/or detect changes to information during transmission.
- UBTU-24-100840 r1134804 -> r1268222: Ubuntu 24.04 LTS SSH server must be configured to use only FIPS 140-3 validated key exchange algorithms.
- UBTU-24-100850 r1067115 -> r1268237: Ubuntu 24.04 LTS must configure the SSH client to use FIPS 140-3 approved ciphers to prevent the unauthorized disclosure of information and/or detect changes to information during transmission.
- UBTU-24-100860 r1155244 -> r1268237: Ubuntu 24.04 LTS SSH client must be configured to use only Message Authentication Codes (MACs) employing FIPS 140-3 validated cryptographic hash algorithms.
- UBTU-24-100900 r1067161 -> r1268243: Ubuntu 24.04 LTS must accept Personal Identity Verification (PIV) credentials.
- UBTU-24-100910 r1067164 -> r1268243: Ubuntu 24.04 LTS must accept Personal Identity Verification (PIV) credentials managed through the Privileged Access Management (PAM) framework.
- UBTU-24-102000 r1137691 -> r1268226: Ubuntu 24.04 LTS when booted must require authentication upon booting into single-user and maintenance modes.
- UBTU-24-200090 r1134806 -> r1268219: Ubuntu 24.04 LTS must monitor remote access methods.
- UBTU-24-200250 r1066535 -> r1268212: Ubuntu 24.04 LTS must automatically remove or disable emergency accounts after 72 hours.
- UBTU-24-200260 r1066538 -> r1261521: Ubuntu 24.04 LTS must disable account identifiers (individuals, groups, roles, and devices) after 35 days of inactivity.
- UBTU-24-200270 r1155243 -> r1261490: Ubuntu 24.04 LTS must audit any script or executable called by cron as root or by any privileged user.
- UBTU-24-200640 r1066562 -> r1268215: Ubuntu 24.04 LTS must display the Standard Mandatory DoW Notice and Consent Banner before granting access to via an SSH logon.
- UBTU-24-200650 r1066565 -> r1268215: Ubuntu 24.04 LTS must enable the graphical user logon banner to display the Standard Mandatory DoW Notice and Consent Banner before granting local access to the system via a graphical user logon.
- UBTU-24-200660 r1066568 -> r1268215: Ubuntu 24.04 LTS must display the Standard Mandatory DoW Notice and Consent Banner before granting local access to the system via a graphical user logon.
- UBTU-24-300001 r1066574 -> r1268241: Ubuntu 24.04 LTS Advance Package Tool (APT) must be configured to prevent the installation of patches, service packs, device drivers, or Ubuntu 24.04 LTS components without verification they have been digitally signed using a certificate that is recognized and approved by the organization.
- UBTU-24-300020 r1107313 -> r1268251: Ubuntu 24.04 LTS must require users to provide a password for privilege escalation.
- UBTU-24-300021 r1101786 -> r1268251: Ubuntu 24.04 LTS must require users to reauthenticate for privilege escalation or when changing roles.
- UBTU-24-300022 r1208697 -> r1268251: Ubuntu 24.04 LTS must be configured so that remote X connections are disabled, unless to fulfill documented and validated mission requirements.
- UBTU-24-300023 r1208698 -> r1268251: Ubuntu 24.04 LTS SSH daemon must prevent remote hosts from connecting to the proxy display.
- UBTU-24-300025 r1184069 -> r1268251: Ubuntu 24.04 LTS must disable the x86 Ctrl-Alt-Delete key sequence if a graphical user interface is installed.
- UBTU-24-300026 r1068363 -> r1268251: Ubuntu 24.04 LTS must disable the x86 Ctrl-Alt-Delete key sequence.
- UBTU-24-300027 r1066628 -> r1268251: Ubuntu 24.04 LTS must not have accounts configured with blank or null passwords.
- UBTU-24-300028 r1208699 -> r1268251: Ubuntu 24.04 LTS must not allow accounts configured in Pluggable Authentication Modules (PAM) with blank or null passwords.
- UBTU-24-300029 r1066634 -> r1268251: Ubuntu 24.04 LTS must generate audit records for all events that affect the systemd journal files.
- UBTU-24-400020 r1066652 -> r1268228: Ubuntu 24.04 LTS must implement smart card logins for multifactor authentication for local and network access to privileged and nonprivileged accounts.
- UBTU-24-400030 r1067130 -> r1268228: Ubuntu 24.04 LTS must implement smart card logins for multifactor authentication for local and network access to privileged and nonprivileged accounts over SSH.
- UBTU-24-400060 r1066658 -> r1268244: Ubuntu 24.04 LTS must electronically verify Personal Identity Verification (PIV) credentials.
- UBTU-24-400310 r1066682 -> r1273634: Ubuntu 24.04 LTS must be able to enforce a 180-day maximum password lifetime restriction.
- UBTU-24-400400 r1067124 -> r1268232: Ubuntu 24.04 LTS must encrypt all stored passwords with a FIPS 140-3 approved cryptographic hashing algorithm.
- UBTU-24-600060 r1066724 -> r1268247: Ubuntu 24.04 LTS must use DoW PKI-established certificate authorities (CAs) for verification of the establishment of protected sessions.
- UBTU-24-600090 r1066730 -> r1261473: Ubuntu 24.04 LTS handling data requiring data-at-rest protections must employ cryptographic mechanisms to prevent unauthorized disclosure and modification of the information at rest.
- UBTU-24-600140 r1137695 -> r1268233: Ubuntu 24.04 LTS must restrict access to the kernel message buffer.
- UBTU-24-600150 r1137695 -> r1268233: Ubuntu 24.04 LTS must set a sticky bit on all public directories to prevent unauthorized and unintended information transferred via shared system resources.
- UBTU-24-600160 r1066742 -> r1261540: Ubuntu 24.04 LTS must compare internal information system clocks at least every 24 hours with an authoritative time server.
- UBTU-24-600230 r1066754 -> r1268254: Ubuntu 24.04 LTS must disable all wireless network adapters.
- UBTU-24-700020 r1184072 -> r1261493: Ubuntu 24.04 LTS must generate system journal entries without revealing information that could be exploited by adversaries.
- UBTU-24-700060 r1184074 -> r1261495: Ubuntu 24.04 LTS must configure the directories used by the system journal to be group-owned by "systemd-journal".
- UBTU-24-700080 r1184078 -> r1261497: Ubuntu 24.04 LTS must configure the directories used by the system journal to be owned by "root".
- UBTU-24-700090 r1184080 -> r1261499: Ubuntu 24.04 LTS must configure the files used by the system journal to be owned by "root"
- UBTU-24-700400 r1155246 -> r1268251: Ubuntu 24.04 LTS must be a vendor-supported release.
- UBTU-24-909000 r1068399 -> r1261488: Ubuntu 24.04 LTS audit system must protect auditing rules from unauthorized change.
- UBTU-24-909890 r1135002 -> r1261485: Ubuntu 24.04 LTS must use cryptographic mechanisms to protect the integrity of audit tools.

## 1.6.0 - 2026 October - Goss documentation link

- **`run_audit.sh` now runs [syver](https://github.com/krameff/syver), replacing goss.** Syver is the
  signed continuation of the krameff goss fork and reads this content unchanged. `AUDIT_BIN` defaults
  to `/usr/local/bin/syver` and `AUDIT_BIN_MIN_VER` moves from `0.4.8` to `0.13.0`, so a goss binary
  now fails the version pre-check. The paired remediation role installs syver v0.13.0 and passes it
  as `AUDIT_BIN`; it must carry that change before this one is used. The README requirements and
  links describe syver
- **Nine checks failed compliant hosts.** UBTU-24-200020 lacked the closing `/` on its `idle-delay`
  pattern, so it was matched as literal text and never passed; it now labels each `gsettings` value
  and accepts an `idle-delay` of 1 to 600. UBTU-24-200000 anchored on `*` while `grep -r` prefixes
  each match with its file name, and UBTU-24-200060 required `TMOUT=600` and then rejected the same
  line, so neither ever passed. UBTU-24-300030 required a single space in `UMASK 077`, failing the
  stock tab-separated `login.defs`, and UBTU-24-400060 and UBTU-24-400375 anchored `cert_policy` at
  the start of the line, failing the indented `pkcs11_module` block. UBTU-24-400380 carried a `(?!`
  lookahead, which RE2 rejects, so it errored on every host; it now fails when any `cert_policy` line
  lacks `crl_auto` or `crl_offline`. UBTU-24-200260 accepted only an `INACTIVE` of 10 to 35 where the
  check allows 1 to 35, and UBTU-24-600060 expected the literal text `^DOD_PKE_CA_chain.pem`, which
  `ls` output never contains; it now passes when any file name contains `dod`, case-insensitively
- **Three negated patterns never matched.** Those in UBTU-24-300030, UBTU-24-400060 and
  UBTU-24-400375 were malformed (`{2;}`, a trailing `;`, and a `(?!` lookahead that RE2 rejects), so
  they matched as literal text and rejected nothing. UBTU-24-300030 now rejects any `UMASK` whose
  group or other digit is not `7`, and UBTU-24-400060 now fails when any `cert_policy` line lacks
  `ocsp_on`, as the check text requires of every line
- **Three checks tested the wrong setting.** UBTU-24-200040 tested the screensaver lock, not the
  locked `automount-open` key the rule names. UBTU-24-400370 tested `use_mappers` in
  `pam_pkcs11.conf` rather than `ldap_user_certificate` in `sssd.conf`. UBTU-24-600070 tested
  `kdump.service`; Ubuntu ships `kdump-tools.service`, which is what the check text and the
  remediation use
- **Metadata now matches the V1R6 XCCDF.** 36 `CCI`, 11 `Vul_ID` and 11 `Group_Title` entries were
  wrong, including placeholders (`NA`, `CCI-000000`), digit typos (`V-2270781`, `CCI-00496`) and IDs
  copied from another control (UBTU-24-909000). UBTU-24-700300 carried `RRule_ID`, UBTU-24-900920
  `Cat: 2` for a low-severity rule, and UBTU-24-700040, a medium rule, sat in `cat_1`; it moves to
  `cat_2/UBTU-24-70xxxx`
- **41 check titles now carry the V1R6 rule title.** Some were copied from another rule (UBTU-24-100820
  carried the MACs title of UBTU-24-100830) or held typos (`preform`, `/etc/sudoers.d.d`,
  `/var/log/utmp`, mode `"0"`); the rest predated wording changes in the benchmark

- **the goss documentation link pointed at the upstream read-the-docs site.** `README.md`
  referenced `goss.readthedocs.io`, which documents the pre-fork project rather than the krameff
  fork this content is built against. It now points at
  `github.com/krameff/goss/blob/devel/docs/index.md`, matching the sibling audit content

## 1.6.0 - 2026 August - Company name updated to Quantum Sky

- the parent company name changed from Tyto Athene to Quantum Sky. Renamed in `LICENSE`, the only place this repository carries it
- deliberately not renamed: existing entries in this file, which record what was true when written

## 1.6.0 - 2026 August Alignment - goss regex corrections and documentation

Alignment pass at the existing V1R6 release (no benchmark version change).

Two checks could never pass, on any host, in any state:

- UBTU-24-200610: the `authfail` expectation was written as `/^auth\s+[default=die]\s+pam_faillock.so\s+authfail/`. The PAM control flag is a literal `[default=die]`, but unescaped in a regex it is a character class matching exactly one character from that set, and the text at that position is `[`, which the class does not contain. The expectation therefore returned false against a correctly hardened host while the sibling `authsucc` expectation on the same resource passed, which is why the failure read as a partial rather than an obviously broken check. Brackets are now escaped. Confirmed against a remediated container: the resource went from one passing and one failing expectation to both passing, and the suite total moved from 22 failures to 21 with no other check changing.
- UBTU-24-400020: the same defect in `/^auth\s*[success=2 default=ignore]\s*pam_pkcs11.so/`, with `\s*` where `\s+` is meant. Brackets escaped and the separators tightened. This check is gated behind `ubtu24stig_uses_smartcard` so it is inert by default, which is why it went unnoticed.

The remaining bracket expressions in the suite were reviewed in the same pass and are genuine character classes over numeric ranges. `UBTU-24-200680` matches shell text containing `[ -n "$SSH_CLIENT" ]` and passes only because the `-` inside the brackets forms a range that happens to span the literal characters; it is fragile but correct, and is left alone rather than changed speculatively.

Coverage:

- UBTU-24-400220 now asserts `rounds` as well as the hashing algorithm. The check-content is a finding when `rounds` is set below 100000, but the test only matched the algorithm, so a host at the distribution default passed. Added `'/^password.*pam_unix\.so.*rounds=[1-9][0-9]{5,}/'`, which requires at least six digits with no leading zero and therefore any value of 100000 or greater. Written as a literal range rather than a `.Vars` reference deliberately: a new `.Vars` key hard-aborts the whole audit under `missingkey=error` when a newer audit branch is run against an older remediation that does not emit it. Confirmed against a converged container - the expectation passes at `rounds=100000` and fails at `rounds=50000`, moving the suite total from 21 failures to 22 and back.

Documentation:

- README: corrected the case of the `Further Information` heading.

## 1.6.0 - Based on DISA STIG Ubuntu 24.04 LTS V1R6 (01 July 2026) - benchmark_v1.6.0

Benchmark bump V1R5 -> V1R6 (194 -> 194; 0 added/removed; 5 SV-rev bumps; all fixtext-only per the revision history, so goss check-content is unchanged and only Rule_ID metadata moves). Version strings: vars/STIG.yml v1.6.0, run_audit.sh BENCHMARK_VER 1.6.0, README banner + DISA zip V1R6.

Rule_ID SV-rev bumps (goss meta only):
- UBTU-24-300022 r1066613 -> r1208697
- UBTU-24-300023 r1066616 -> r1208698
- UBTU-24-300028 r1134808 -> r1208699 (goss already greps common-password + common-auth)
- UBTU-24-300031 r1067177 -> r1208700
- UBTU-24-600010 r1066718 -> r1208702

New Alignment (krameff goss migration):
- run_audit.sh: AUDIT_BIN_MIN_VER 0.4.4 -> 0.4.8; goss version parse awk '{print $NF}' -> awk 'NR==1{print $NF}' (krameff goss emits a 2-line banner).
- README goss link goss-org -> github.com/krameff/goss.

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
