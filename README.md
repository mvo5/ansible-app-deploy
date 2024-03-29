Deploy using ansible
====================

This requires ansible 1.2+

* First time provision (use limit as needed):
```
$ printf "$(pwgen 14 1)" > non-git-files/${inventory_hostname}.db-password
$ echo "the-ldap-password-to-use" > non-git-files/ldap-password
(use certmaker to generate a ssl key)
$ cp -ar generate-keys-dir non-git-files/${inventory_hostname}.uni-trier.de

$ ansible-playbook provision.yml --limit staging
```

* Deploy new code (use limit as needed):
```
$ ansible-playbook deploy.yml --limit staging
```

Staging systems
---------------

If there is a "django_project/settings_staging.py" and the server is in the staging group the script will copy this on deploy to django_project/settings_production.py 


Create a django admin user without a browser:
---------------------------------------------
```
$ /srv/$(PROJECT_NAME)/venv/bin/activate && DJANGO_SETTINGS_MODULE=django_project.settings_production python  /srv/$(PROJECT_NAME)/application/manage.py createsuperuser
```


Test using a local LXC container
================================

* Create a 12.04 container (and disable ufw):
```
$ sudo lxc-create -t ubuntu -n webapp-devel-12.04 -- -r precise
$ sudo lxc-start -n webapp-devel-12.04
```

* Login to LXC to get IP

* Set the ip in inventory/ansible_hosts under [lxc].

* Provision the container, only needed once (but can be run multiple
times). The lxc container password is "ubuntu":
```
$ ansible-playbook -u ubuntu -k -K provision.yml --limit lxc
```

* Deploy code as needed:
```
$ ansible-playbook -u ubuntu -k -K deploy.yml --limit lxc
```


