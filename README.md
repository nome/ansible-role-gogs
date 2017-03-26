[![Travis CI](https://travis-ci.org/nome/ansible-role-gogs.png)](https://travis-ci.org/nome/ansible-role-gogs)

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

    - hosts: gogs
      roles:
      - role: nome.gogs
        gogs_admin: gogsadmin
        gogs_admin_password: secret

Note that calling the admin user `admin` won't work, since this user id is reserved in Gogs.


Typical playbook example
------------------------

As a more typical example, we'll consider a setup with a reverse proxy
configured via the
[jdauphant.nginx](https://galaxy.ansible.com/jdauphant/nginx/) role.

    - hosts: gogs

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

For demonstration purposes, all variables are given as role parameters. In a
real-world setup, you may want to put them in a separate file, e.g.
`vars/main.yml`.


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


Role variables
--------------

See [`defaults/main.yml`](defaults/main.yml).

License
-------
MIT / BSD
