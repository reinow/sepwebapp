### 1.3.6

Added support for java applications

Added tunables:

webapp_id_fsetid

Minor bugfixes.

### 1.3.5

Added types:

webapp_id_var_lib_t

Thanks to pharasyte for pointing out a few typos which prevent the policy to build.

### 1.3.4

Added support for Python bytecode files, systemd unit files, and connections to unreserved ports.

Added tunables:

webapp_id_connect_unreserved_port

Added types:

webapp_id_unit_file_t, webapp_id_pyc_script_t

Minor bugfixes.

### 1.3.3

Added tunables:

webapp_id_block_suspend, webapp_id_exec_ping, webapp_id_manage_cgroups, webapp_id_rw_sysctls_net, webapp_id_sys_admin, webapp_id_sys_nice, and webapp_id_sys_tty_config

Minor bugfixes.

### 1.3.2

Added tunables:

webapp_id_dbus_sssd, webapp_id_connect_sssd, webapp_id_read_public_sssd

Minor bugfixes.

### 1.3.1

Changed name of tunables:

From webapp_id_enable_chown to webapp_id_chown
From webapp_id_enable_ptrace to webapp_id_ptrace
From webapp_id_enable_chroot to webapp_id_chroot

Added tunables:

webapp_id_block_suspend, webapp_id_net_admin

### 1.3.0

Merged to RHEL/CentOS 7

Added support for web applications started with systemd.

Added tunables:
webapp_id_auth_pam, webapp_id_setgid, webapp_id_setuid, webapp_id_read_passwd_file

Added types: webapp_id_passwd_file_t

### 1.2.0
Initial release
