# Ubuntu 24.04 Goss config

## Overview

### Based on STIG Ubuntu Linux 24.04 LTS Benchmark v1.1.0 [Release](https://dl.dod.cyber.mil/wp-content/uploads/stigs/zip/U_CAN_Ubuntu_24-04_LTS_V1R1_STIG.zip)

Ability to audit a system using a lightweight binary to check the current state.

This is:

- very small < 14MB
- lightweight
- self contained

It works using a set of configuration files and directories to audit STIG of Ubuntu22 servers. These files/directories correlate to the STIG Level and STIG_ID

feedback on any differences between OSs please raise an issue

## Requirements

You must have [goss](https://github.com/goss-org/goss/) available to your host you would like to test.

You must have sudo/root access to the system as some commands require privilege information.

Assuming you have already clone this repository you can run goss from where you wish.

Please refer to the audit documentation for usage.

- [readthedocs](https://ansible-lockdown.readthedocs.io/en/latest/)

This also works alongside the [Ansible Lockdown Ubuntu24-STIG role](https://github.com/ansible-lockdown/UBUNTU24-STIG)

Which will:

- install
- audit
- remediate
- audit

## Join us

On our [Discord Server](https://www.lockdownenterprise.com/discord) to ask questions, discuss features, or just chat with other Ansible-Lockdown users

Set of configuration files and directories to run the first stages of STIG RHEL9 based servers

This is configured in a directory structure level.

Goss is run based on the goss.yml file in the top level directory. This specifies the configuration.

## further information

- [goss documentation](https://github.com/aelsabbahy/goss/blob/master/docs/manual.md#patterns)
- [STIG standards](https://public.cyber.mil/stigs/)
