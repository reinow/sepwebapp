##Overview

Security Enhanced Linux (SELinux) is an implementation of a Mandatory Access Control (MAC) mechanism in the Linux kernel. In computer security, Mandatory Access Control refers to a type of access control where the operating system controls the ability of a subject to access an object. MAC is typically used in military and highly secure government installations. A subject is usually a process or thread. Objects are typically directories, files, shared memory segments, TCP/UDP ports, or other processes. Subjects and objects each have security attributes. With the principle of least privilege and through a security policy, SELinux prevents compromising an entire system due to the compromise of a single application running with what would otherwise be elevated privileges. *SELinux policy module for web applications* is a flexible SELinux security policy for web applications. With a few lines of code you can create a fine grained security policy for each of your web applications and make them run in separate domains, thereby strongly isolate them from one another and from the underlying operating system. For each web application, there is a type defined for the process, and optionally a type is defined for each web application's subprocesses. Furthermore, for each web application, a set of file types are also defined. Types are also defined for, packets, port, netif, and node. A set of tunables are defined for each web application to allow fine grained control of permissions without modifying the policy source.

##How to

Each web application is identified with an identity and created with a template. The identity is a keyword used to name the domain and its objects.

To define a web application with identity *cms*, add the line below to you webapp.te file:

```
webapp_base(cms)
```

To define a Python web application with identity *cms*:

```
webapp_base(cms)
webapp_py_files(cms)
```

To define a Python web application with identity *cms* started with Supervisord:

```
webapp_base(cms)
webapp_supervisord_domain(cms)
webapp_py_files(cms)
```

Of course, replace cms with whatever you wish, but keep the identity rather short.

To define a Ruby web application with identity *family* started with an init script:

```
webapp_base(family)
webapp_init_domain(family)
webapp_rb_files(family)
```

To define a Perl web application with identity *cms*, controlled by Supervisord, which forks children into its own domain:

```
webapp_base(cms)
webapp_supervisord_domain(cms)
webapp_base(cmschild)
webapp_pl_files(childcms)
webapp_child_domain(cms, childcms)
webapp_list_content_dirs(childcms, webapp_cms_t)
```

To define a Python web application with identity *emperor*, controlled by Systemd, which forks children into its own domain:

```
webapp_base(emperor)
webapp_systemd_domain(emperor)
webapp_base(oribnet)
webapp_py_files(oribnet)
webapp_child_domain(emperor, oribnet)
webapp_list_content_dirs(oribnet, webapp_emperor_t)
```

It could be useful that the worker processes have its own security context, rather then sharing the security with the parent process, thus fullfilling necessary minimum privileges for the worker processes as well as for the parent process. Of course, the web application and application server must support forking children into its own domain.

This is an example of a Python uWSGI web application with identity *uwsgi* started with Systemd, where emperors run in a separate domain, and the web application's worker processes run in their own domains.

```
webapp_base(uwsgi)
webapp_systemd_domain(uwsgi)
webapp_base(emperor)
webapp_child_domain(uwsgi, emperor)
webapp_base(oribnet)
webapp_py_files(oribnet)
webapp_child_domain(emperor, oribnet)
webapp_list_content_dirs(oribnet, webapp_emperor_t)
```

Below is the output of *ps axZ|grep uwsgi* on a CentOS 6.6 box, running some web applications, where the main process, the emperor processes, and the working processes have their own security context.

```
ps axZ|grep uwsgi
system_u:system_r:webapp_uwsgi_t:s0:c0.c1023 3295 ? S   0:17 /usr/local/sbin/uwsgi --dlopen /lib64/libselinux.so.1 --pidfile /var/run/uwsgi/uwsgi-uwsgi.pid --daemonize /var/log/uwsgi/uwsgi/uwsgi-uwsgi.log --ini /usr/local/etc/uwsgi/uwsgi.ini
system_u:system_r:webapp_emperor_t:s0:c0.c1023 3311 ? S   0:12 /usr/local/sbin/uwsgi --ini wsgi.ini
system_u:system_r:webapp_emperor_t:s0:c0.c1023 3312 ? S   0:12 /usr/local/sbin/uwsgi --ini rack.ini
system_u:system_r:webapp_emperor_t:s0:c0.c1023 3316 ? S   0:12 /usr/local/sbin/uwsgi --ini php.ini
system_u:system_r:webapp_emperor_t:s0:c0.c1023 3317 ? S   0:14 /usr/local/sbin/uwsgi --ini rack.ini
system_u:system_r:webapp_emperor_t:s0:c0.c1023 3318 ? S   0:14 /usr/local/sbin/uwsgi --ini wsgi.ini
system_u:system_r:webapp_emperor_t:s0:c0.c1023 3319 ? S   0:15 /usr/local/sbin/uwsgi --ini php.ini
system_u:system_r:webapp_emperor_t:s0:c0.c1023 3320 ? S   0:12 /usr/local/sbin/uwsgi --ini psgi.ini
system_u:system_r:webapp_emperor_t:s0:c0.c1023 3321 ? S   0:10 /usr/local/sbin/uwsgi --ini psgi.ini
system_u:system_r:webapp_sevhost_t:s0:c0.c1023 3323 ? S   0:30 /usr/local/sbin/uwsgi --ini sevhost.oribium.net.ini
system_u:system_r:webapp_sussiep_t:s0:c0.c1023 3324 ? S   0:58 /usr/local/sbin/uwsgi --ini sussiep.oribium.net.ini
system_u:system_r:webapp_family_t:s0:c0.c1023 3327 ? Sl   0:22 /usr/local/sbin/uwsgi --ini slakt.oribium.net.ini
system_u:system_r:webapp_ponpshop_t:s0:c0.c1023 3328 ? S  0:59 /usr/local/sbin/uwsgi --ini prestashop.pontuswahlgren.se.ini
system_u:system_r:webapp_sussiep_t:s0:c0.c1023 3587 ? Sl  0:00 /usr/local/sbin/uwsgi --ini sussiep.oribium.net.ini
system_u:system_r:webapp_sussiep_t:s0:c0.c1023 3588 ? Sl  0:00 /usr/local/sbin/uwsgi --ini sussiep.oribium.net.ini
system_u:system_r:webapp_family_t:s0:c0.c1023 3604 ? S    0:00 /usr/local/sbin/uwsgi --ini slakt.oribium.net.ini
system_u:system_r:webapp_family_t:s0:c0.c1023 3605 ? S    0:00 /usr/local/sbin/uwsgi --ini slakt.oribium.net.ini
system_u:system_r:webapp_ponpshop_t:s0:c0.c1023 3615 ? S  0:00 /usr/local/sbin/uwsgi --ini prestashop.pontuswahlgren.se.ini
system_u:system_r:webapp_ponpshop_t:s0:c0.c1023 3616 ? S  0:00 /usr/local/sbin/uwsgi --ini prestashop.pontuswahlgren.se.ini
system_u:system_r:webapp_sevhost_t:s0:c0.c1023 3641 ? Sl  0:00 /usr/local/sbin/uwsgi --ini sevhost.oribium.net.ini
system_u:system_r:webapp_sevhost_t:s0:c0.c1023 3642 ? Sl  0:00 /usr/local/sbin/uwsgi --ini sevhost.oribium.net.ini
```

For a full list of templates and interfaces, see [templates and interfaces documentation](http://www.oribium.net/services_webapp/ "Templates and Interfaces Documentation").

##Types

###Process type

* webapp_id_t

where id is the identity of the domain. Example, foo is the identity of the webapp_foo_t domain.

###File related types

* webapp_id_cache_t
* webapp_id_config_t
* webapp_id_content_t
* webapp_id_etc_runtime_t
* webapp_id_etc_t
* webapp_id_exec_t
* webapp_id_initrc_exec_t
* webapp_id_ld_so_cache_t
* webapp_id_log_t
* webapp_id_mount_t
* webapp_id_net_conf_t
* webapp_id_php_conf_t
* webapp_id_php_script_t
* webapp_id_pl_script_t
* webapp_id_py_script_t
* webapp_id_pyc_script_t
* webapp_id_rb_script_t
* webapp_id_rw_content_t
* webapp_id_tmpfs_t
* webapp_id_tmp_t
* webapp_id_unit_file_t
* webapp_id_var_run_t

where id is the identity. Example, foo is the identity of the webapp_foo_log_t type.

###Network related types

* webapp_id_client_packet_t
* webapp_id_netif_t
* webapp_id_node_t
* webapp_id_port_t
* webapp_id_server_packet_t

where id is the identity. Example, foo is the identity of the webapp_foo_port_t type.

###Linux namespace related types

Some application servers, like uWSGI, have native support for Linux namespaces. Linux namespaces are an elegant way to detach the process from a specific layer of the kernel and assign it to a new one. You can, for instance, mount a specific etc directory to each of the web applications, but you can also create specific PIDs, IPC, and networking for each of the web applications. The types below are related to Linux namespaces. If you are using uWSGI, there is a [snippet](http://uwsgi-docs.readthedocs.org/en/latest/Snippets.html "uWSGI snippet") explaining how to run the emperor process in one domain, and each web application's worker processes in a separate domain.

* webapp_id_etc_runtime_t
* webapp_id_etc_t
* webapp_id_ld_so_cache_t
* webapp_id_mount_t
* webapp_id_net_conf_t
* webapp_id_passwd_file_t

where id is the identity. Example, foo is the identity of the webapp_foo_etc_runtime_t type.

##Tunables

For each web application the following tunables are defined:

* webapp_id_anon_read
* webapp_id_anon_write
* webapp_id_auth_pam
* webapp_id_auth_use_nsswitch
* webapp_id_bind_http_port
* webapp_id_block_suspend
* webapp_id_chown
* webapp_id_chroot
* webapp_id_connect_abrt
* webapp_id_connect_ftp
* webapp_id_connect_gopher
* webapp_id_connect_http
* webapp_id_connect_http_cache
* webapp_id_connect_memcache
* webapp_id_connect_openstack
* webapp_id_connect_pop
* webapp_id_connect_sieve
* webapp_id_connect_sssd
* webapp_id_connect_syslog
* webapp_id_connect_whois
* webapp_id_dac_override
* webapp_id_dbus_avahi
* webapp_id_dbus_sssd
* webapp_id_enable_cache
* webapp_id_enable_cifs
* webapp_id_enable_ftp_server
* webapp_id_enable_gpg
* webapp_id_enable_homedirs
* webapp_id_enable_nagios
* webapp_id_enable_nfs
* webapp_id_enable_openca
* webapp_id_enable_tuntap
* webapp_id_enable_zarafa
* webapp_id_exec_bin
* webapp_id_exec_hostname
* webapp_id_exec_ifconfig
* webapp_id_exec_mount
* webapp_id_exec_ping
* webapp_id_exec_self
* webapp_id_exec_tmp
* webapp_id_exec_tmpfs
* webapp_id_exec_uwsgi
* webapp_id_execmem
* webapp_id_install_mode
* webapp_id_manage_cgroups
* webapp_id_manage_config
* webapp_id_net_admin
* webapp_id_mod_auth_pam
* webapp_id_ptrace
* webapp_id_read_etc
* webapp_id_read_passwd_file
* webapp_id_read_public_sssd
* webapp_id_read_sysctls_net
* webapp_id_rw_sysctls_net
* webapp_id_sendmail
* webapp_id_setgid
* webapp_id_setrlimit
* webapp_id_setuid
* webapp_id_sys_admin
* webapp_id_sys_nice
* webapp_id_sys_tty_config
* webapp_id_tcp_connect
* webapp_id_tcp_connect_db
* webapp_id_tcp_listen
* webapp_id_use_tty
* webapp_id_verify_dns

All tunables defaults to false. Hopefully the names are self-explanatory.

##File context

For each web application it is required to label the related files. Below is an example on file context entries for a Python web application. This example is based on the Tornado application server, managed by Supervisord.

```
/var/www/example\.com/cms(/.*)?				gen_context(system_u:object_r:webapp_cms_content_t,s0)
/var/www/example\.com/cms/logs(/.*)?			gen_context(system_u:object_r:webapp_cms_log_t,s0)
/var/www/example\.com/cms/cache(/.*)?			gen_context(system_u:object_r:webapp_cms_cache_t,s0)
/var/www/example\.com/cms/conf(/.*)?			gen_context(system_u:object_r:webapp_cms_config_t,s0)
/var/www/example\.com/cms/static/media/uploads(/.*)?	gen_context(system_u:object_r:webapp_cms_rw_content_t,s0)
/var/www/example\.com/cms/.*\.(py|pyc|pyo)	--	gen_context(system_u:object_r:webapp_cms_py_script_t,s0)
/var/www/example\.com/cms/app\.(py|pyc|pyo)	--	gen_context(system_u:object_r:webapp_cms_exec_t,s0)
```

This is an example of a Joomla 3 web application and the uWSGI application server, with Linux namespaces in use. The directories /etc, /tmp, /var/tmp, and /var/run were mounted in the application server's namespace.

```
/var/www/example\.com/public_html(/.*)?			gen_context(system_u:object_r:webapp_cms_content_t,s0)
/var/www/example\.com/public_html/logs(/.*)?		gen_context(system_u:object_r:webapp_cms_log_t,s0)
/var/log/uwsgi/cms(/.*)?				gen_context(system_u:object_r:webapp_cms_log_t,s0)
/var/www/example\.com/public_html/cache(/.*)?		gen_context(system_u:object_r:webapp_cms_cache_t,s0)
/var/www/example\.com/public_html/config(/.*)?		gen_context(system_u:object_r:webapp_cms_config_t,s0)
/var/www/example\.com/public_html/tmp(/.*)?		gen_context(system_u:object_r:webapp_cms_tmp_t,s0)
/var/www/example\.com/mount(/.*)?			gen_context(system_u:object_r:webapp_cms_mount_t,s0)
/var/www/example\.com/mount/etc(/.*)?			gen_context(system_u:object_r:webapp_cms_etc_t,s0)
/var/www/example\.com/mount/etc/ld\.so\.cache	--	gen_context(system_u:object_r:webapp_cms_ld_so_cache_t,s0)
/var/www/example\.com/mount/etc/hosts		--	gen_context(system_u:object_r:webapp_cms_net_conf_t,s0)
/var/www/example\.com/mount/etc/mtab		--	gen_context(system_u:object_r:webapp_cms_etc_runtime_t,s0)
/var/www/example\.com/mount/etc/php\.ini	--	gen_context(system_u:object_r:webapp_cms_php_conf_t,s0)
/var/www/example\.com/mount/etc/php\.d(/.*)?		gen_context(system_u:object_r:webapp_cms_php_conf_t,s0)
/var/www/example\.com/mount/etc/resolv\.conf	--	gen_context(system_u:object_r:webapp_cms_net_conf_t,s0)
/var/www/example\.com/mount/tmp(/.*)?			gen_context(system_u:object_r:webapp_cms_tmp_t,s0)
/var/www/example\.com/mount/var/tmp(/.*)?		gen_context(system_u:object_r:webapp_cms_tmp_t,s0)
/var/www/example\.com/mount/var/run(/.*)?		gen_context(system_u:object_r:webapp_cms_var_run_t,s0)
/var/www/example\.com/public_html/.*\.php	--	gen_context(system_u:object_r:webapp_cms_php_script_t,s0)
```

##Development

This policy was developed on a CentOS 6 box, but since version 1.3.0 it is merged to CentOS 7. With some minor tweaks you may get it work on other Linux distros as well. 

##Feedback

I would appreciate feedback, and suggestions. Do you want to test the policy? Please feel free get in touch.
