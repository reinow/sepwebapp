###1.3.1

Changed name of tunables:

From webapp_id_enable_chown to webapp_id_chown
From webapp_id_enable_ptrace to webapp_id_ptrace
From webapp_id_enable_chroot to webapp_id_chroot

Added tunables:

webapp_id_block_suspend, webapp_id_net_admin

###1.3.0

Merged to RHEL/CentOS 7

Added support for web applications started with systemd.

Added tunables:
webapp_id_auth_pam, webapp_id_setgid, webapp_id_setuid, webapp_id_read_passwd_file

Added types: webapp_id_passwd_file_t

###1.2.0
Initial release
