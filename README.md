[![Travis CI](https://travis-ci.org/nome/ansible-role-gogs.svg?branch=master)](https://travis-ci.org/nome/ansible-role-gogs)

Ansible Role: Gogs Installation and Setup
=========================================

Installs and configures [Gogs](https://gogs.io), a self-hosted Git project
portal. All settings that in a manual setup would be entered in the first-time
run dialog are fully integrated into the playbook (including creation of the
main admin user), thus allowing for a fully automated and reproducible
configuration. Defaults are provided for most settings, which makes a
quick-shot Ansible-based installation just as painless as a manual one.

A few of the more obscure settings of the first-time run dialog, as well as a
fair number of the many [available Gogs
settings](https://gogs.io/docs/advanced/configuration_cheat_sheet), currently
cannot be set via this role. Pull requests are welcome, though.


Minimal playbook example
------------------------

    - hosts: localhost
      roles:
      - role: nome.gogs
        gogs_admin: gogsadmin
        gogs_admin_password: secret

Note that calling the admin user `admin` won't work, since this user id is
reserved in Gogs. After the playbook has run, Gogs should be reachable at
<http://localhost:3000> (as well as `http://yourhost:3000` from other hosts,
unless prevented by a local firewall).


Typical playbook example
------------------------

As a more typical example, we'll consider a setup with a reverse proxy
configured via the
[jdauphant.nginx](https://galaxy.ansible.com/jdauphant/nginx/) role.

    - hosts: gogs

      pre_tasks:
      - name: Install dependency for SELinux configuration
        package: name=libsemanage-python state=present
        when: ansible_selinux
      - name: Allow nginx to do network connects (SELinux)
        seboolean: name=httpd_can_network_connect state=yes persistent=yes
        when: ansible_selinux

      roles:
      - role: nome.gogs
        gogs_admin: gogsadmin
        gogs_admin_password: secret

        gogs_app_name: My Team's Repos
        gogs_bind_addr: 127.0.0.1
        gogs_url: http://{{ ansible_fqdn }}
        gogs_preset: shared

        gogs_smtp_host: smtp.example.com:587

      - role: jdauphant.nginx
        nginx_sites:
          gogs:
            - listen 80
            - location / { proxy_pass http://127.0.0.1:{{ gogs_http_port }}; }

In addition to the SELinux setting included in this example, you may also need
to open port 80 on the firewall, e.g. in RHEL7:

    - firewalld: service=http state=enabled permanent=true


Using a database server
-----------------------

By default, a local SQLite database file is used. A setup with a (separate)
MySQL server could look like the following, using the
[geerlingguy.mysql](https://galaxy.ansible.com/geerlingguy/mysql/) role:

    - hosts: dbserver
      roles:
      - role: geerlingguy.mysql
        mysql_root_password: secure
        mysql_databases:
        - name: gogs
        mysql_users:
        - name: gogs
          host: "%"
          password: secure
          priv: "gogs.*:ALL"

    - hosts: gogs
      roles:
      - role: nome.gogs
        gogs_admin: gogsadmin
        gogs_admin_password: secret

        gogs_db_type: mysql
        gogs_db_host: dbserver:3306
        gogs_db_user: gogs
        gogs_db_password: secure
        gogs_db_name: gogs

Supported values for `gogs_db_type` are `mysql`, `postgres`, `mssql` and `sqlite3`.


Enable sending Mails
--------------------

In order to enable Gogs to send mails, e.g. update notifications and
confirmation mails on user registration, the following settings need to be
supplied:

    gogs_smtp_host: smtp.example.com:587
    gogs_smtp_user: my_smtp_user
    gogs_smtp_password: my_smtp_password


Gogs version and installation methods
-------------------------------------

By default, the latest Gogs release is installed. This can be changed by
setting `gogs_version` to the desired release (e.g. `gogs_version=v0.10.18`).

The (possibly unreleased) "bleeding edge" version of Gogs can be pulled from
GitHub and compiled with the following settings:

    gogs_install_from_source: true
    gogs_version: master

Since distribution packages of the Go compiler are often too old to compile
Gogs, official binaries of the Go copmiler are downloaded from Google and
installed to `/home/git/.local`.

If the target machine can't access the internet (i.e. due to firewall or proxy
settings), set `gogs_install_via_localhost=true`. This will route all downloads
through the control host. When combined with `gogs_install_from_source`, the Go
compiler is *not* downloaded automatically; Git and Go >= 1.5 need to be
already installed on the control host, and the environment variables `GOROOT`
and `GOPATH` need to be set to suitable values.


Other role variables
--------------------

See [`defaults/main.yml`](defaults/main.yml).

License
-------
MIT / BSD
