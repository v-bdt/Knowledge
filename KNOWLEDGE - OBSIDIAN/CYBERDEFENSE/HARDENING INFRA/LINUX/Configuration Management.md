

This is by no means an exhaustive list, but some simple hardening measures are to:

- Audit writable files and directories and any binaries set with the SUID bit.
- Ensure that any cron jobs and sudo privileges specify any binaries using the absolute path.
- Do not store credentials in cleartext in world-readable files.
- Clean up home directories and bash history.
- Ensure that low-privileged users cannot modify any custom libraries called by programs.
- Remove any unnecessary packages and services that potentially increase the attack surface.
- Consider implementing [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux), which provides additional access controls on the system.
