# Staytus - Ansible role

This role installs staytus on either an Ubuntu 18.04 or CentOS 7 host.
Updating staytus is also possible.

"[Staytus](https://staytus.co/) is a free, open source & beautiful status site that you can install on your own servers. It is a fully-loaded with all the features you'd expect from any site you might pay for."

Prerequisites
-------------

 * Ensure [MySQL](https://github.com/geerlingguy/ansible-role-mysql) is installed
 * Ensure [Nginx](https://github.com/nginxinc/ansible-role-nginx) is installed
 * On Ubuntu; move /etc/nginx/sites-available/default to another location
 * If SElinux is enabled, allow nginx networking `sudo setsebool -P httpd_can_network_connect true`

I've provided the vars I've used for the nginx and mysql roles below.


Important
---------

 * This role is tested to be used with Ansible 2.8 and 2.9

 * The role installs ruby from source when using CentOS,
   because at least Ruby 2.2 is required, that RPM isn't present in the yum repo's

 * The role installs python3 with python3-pip, to install `PyMySQL`


Updating
--------

To update staytus set the variable and run the role again:

```yaml
staytus_update: True
```

Example playbook
----------------

```yaml
---
- hosts: staytus_host
  vars:
    staytus_db_pass: a_password
    staytus_update: True
  roles:
    # - ansible-role-mysql
    # - ansible-role-nginx
    - staytus-ansible-role
```


Vars of other playbooks
-----------------------

When using in production, please update the required vars.

```yaml
mysql_root_password: a_password
mysql_databases:
  - name: staytus
    encoding: utf8mb4
    collation: utf8mb4_unicode_ci
mysql_users:
  - name: staytus
    host: 127.0.0.1
    password: a_password
    priv: "staytus.*:ALL"
    state: present

nginx_remove_default_vhost: True
nginx_client_max_body_size: "50m"
nginx_vhosts:
  - listen: "0.0.0.0:80 default"
    server_name: "status.example.com"
    filename: "staytus.conf"
    root: "/opt/staytus/staytus/public"
    extra_parameters: |
      client_max_body_size 50M;

      location /assets {
        add_header Cache-Control max-age=3600;
      }

      location / {
          try_files $uri $uri/index.html $uri.html @staytus;
      }

      location @staytus {
        proxy_intercept_errors on;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto http;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://127.0.0.1:8787;
      }
```
