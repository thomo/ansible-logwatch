Ansible Role: thomo.logwatch [![Build Status](https://app.travis-ci.com/thomo/ansible-logwatch.svg?branch=main)](https://app.travis-ci.com/thomo/ansible-logwatch)
========

Ansible role to install and configure logwatch.

Installation
------------
### Ansible Galaxy

If you want to install it in the local ansible project dir:

    ansible-galaxy install -p roles thomo.logwatch

To install it on system level

    ansible-galaxy install thomo.logwatch

### Manual

Alternative you can install it by downloading it from github.

    wget https://github.com/thomo/ansible-logwatch/archive/master.zip
    unzip -d roles master.zip
    mv roles/ansible-logwatch-master roles/thomo.logwatch

Role Variables
--------------

    # Mail transport agent package (e.g. postfix) installed before logwatch, so that
    # apt does not pull in the distribution default (exim4 on Debian). Empty: do
    # nothing. Only installed, not configured by this role.
    logwatch_mta: ''

    # Bundled custom services to install (restic, restic-rest-server), see "Custom services"
    logwatch_custom_services: []

    # Per-service configuration overrides, written to /etc/logwatch/conf/services/<name>.conf
    # (each value is a list of lines), e.g. systemd: ['LogFile = syslog']
    logwatch_service_conf: {}

    # Per-logfile-group overrides, written to /etc/logwatch/conf/logfiles/<name>.conf
    # (each value is a list of lines), e.g. kernel from the journal, see "Custom services"
    logwatch_logfile_conf: {}

    # Packages to install; add rsyslog on nodes that only have the journal
    logwatch_packages: [logwatch]

    # default is root
    logwatch_mail_to: 'root'

    # default is Logwatch
    logwatch_mail_from: 'Logwatch'

    # Output: stdout, file, mail
    logwatch_output: 'stdout'

    # Format: text, html
    logwatch_format: 'html'

    # default is 'all'
    logwatch_services: [all]

    # specifiy service names without prepending a hyphen, default is []
    logwatch_disabled_services: []

    # valid ranges are: yesterday (yesterday), today, all
    logwatch_range: 'yesterday'

    # search through the archives, default is Yes
    logwatch_archive: 'Yes'

    # valid values are: Low, Med, High or a number (0 - 10), default is Low(0)
    logwatch_detail: 'Low'

    # Limit report to hosts
    logwatch_limit_to_hosts: []

    # Use hostname for the reports instead of this system's hostname.
    logwatch_hostname: ''

    # Number of characters that html output should be wrapped to. Default is 80.
    logwatch_html_wrap: 80

    #Inhibits additional name lookups, displaying IP addresses numerically.
    logwatch_numeric: 'No'


Custom services
---------------

Logwatch has no parser for restic. The role bundles generic services (only the messages of the
tools themselves, not how they are started); enable them with `logwatch_custom_services`:

| Name | Reads | Reports |
|------|-------|---------|
| `restic` | syslog tag `restic` | fatal errors, errors, warnings (e.g. repository missing, wrong password, 401/403 from a rest-server, damaged packs, retries), unknown lines; at detail 5 also counts of checks without errors, snapshots saved, removed, copied. Normal output of backup, check, forget/prune and copy is silent |
| `restic-rest-server` | syslog tag `restic-rest-server` | failed logins per user, TLS handshake errors per reason, unknown lines; at detail 5 starts and clean shutdowns |

A failed systemd unit (also a restic one) is reported by the stock `systemd` service. On Debian
that service reads the log group `messages` (`/var/log/messages`), which rsyslog does not write.
Point it to syslog with `logwatch_service_conf`:

    logwatch_custom_services: [restic, restic-rest-server]
    logwatch_service_conf:
      systemd: ['LogFile = syslog']

Debian trixie has only the journal. Add rsyslog to `logwatch_packages` so that `/var/log/syslog`
and `auth.log` exist. Kernel messages written after boot (OOM, disk errors) do not reach
`kern.log` through rsyslog on a VM, so let the `kernel` log group read the journal instead
(`*JournalCtl` of the shared logwatch script):

    logwatch_logfile_conf:
      kernel:
        - 'LogFile ='
        - 'LogFile = /dev/null'
        - '*JournalCtl = "--no-pager --output=short --dmesg"'
        - '*ApplyStdDate'

Example Playbook
----------------

    - hosts: servers
      roles:
         - role: thomo.logwatch
           logwatch_detail: 'Med'

License
-------

**MIT** - see LICENSE file for details.

Author Information
------------------

Thomas Mohaupt https://github.com/thomo/ansible-logwatch
