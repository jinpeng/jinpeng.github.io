title: Open edX devstack on Ubuntu without Vagrant
date: 2016-04-21 18:21:33
tags:
---
## Install open edX devstack on Ubuntu 14.04 directly ##

Launch EC2 instance with 2 vCPUs 4G memory

```bash
$ ssh -i .ssh/aws-es.pem ubuntu@54.199.222.211
$ sudo apt-get update -y
$ sudo apt-get upgrade -y
$ sudo reboot
```

```bash
$ ssh -i .ssh/aws-es.pem ubuntu@54.199.222.211
$ export OPENEDX_RELEASE=named-release/dogwood
$ wget https://raw.githubusercontent.com/edx/configuration/master/util/install/ansible-bootstrap.sh
$ chmod +x ansible-bootstrap.sh
$ sudo ./ansible-bootstrap.sh
$ grep -rl "vagrant" /edx/app/edx_ansible/edx_ansible/playbooks/roles/local_dev/
$ grep -rl "vagrant" /edx/app/edx_ansible/edx_ansible/playbooks/roles/local_dev/ | xargs sudo sed -i 's/vagrant/ubuntu/g'
$ sudo sed -i '/edxapp_user: edxapp/ a\edxapp_ruby_version: "1.9.3-p551"' /edx/app/edx_ansible/edx_ansible/playbooks/roles/edxapp/defaults/main.yml
$ sudo sed -i '/- name: git checkout edx_ansible repo into edx_ansible_code_dir/ a\  ignore_errors: yes' /edx/app/edx_ansible/edx_ansible/playbooks/roles/edx_ansible/tasks/deploy.yml
$ sudo sed -i '/- edxapp_common/ a\  - role: rbenv\n    rbenv_user: "{{ edxapp_user }}"\n    rbenv_dir: "{{ edxapp_app_dir }}"\n    rbenv_ruby_version: "{{ edxapp_ruby_version }}"' /edx/app/edx_ansible/edx_ansible/playbooks/roles/edxapp/meta/main.yml
```

```bash
$ vim devstack.sh
```

```bash
#!/bin/bash

if [ ! -d /edx/app/edx_ansible ]; then
    echo "Error: Base box is missing provisioning scripts." 1>&2
    exit 1
fi
OPENEDX_RELEASE=named-release/dogwood
export PYTHONUNBUFFERED=1
source /edx/app/edx_ansible/venvs/edx_ansible/bin/activate
export PIP_REQUIRE_VIRTUALENV=true
export PIP_RESPECT_VIRTUALENV=true
cd /edx/app/edx_ansible/edx_ansible/playbooks

# Did we specify an openedx release?
if [ -n "$OPENEDX_RELEASE" ]; then
  EXTRA_VARS="-e edx_platform_version=$OPENEDX_RELEASE \
    -e certs_version=$OPENEDX_RELEASE \
    -e forum_version=$OPENEDX_RELEASE \
    -e xqueue_version=$OPENEDX_RELEASE \
  "
  CONFIG_VER=$OPENEDX_RELEASE
  # Need to ensure that the configuration repo is updated
  # The vagrant-devstack.yml playbook will also do this, but only
  # after loading the playbooks into memory.  If these are out of date,
  # this can cause problems (e.g. looking for templates that no longer exist).
  /edx/bin/update configuration $CONFIG_VER
else
  CONFIG_VER="master"
fi

ansible-playbook -i localhost, -c local run_role.yml -e role=edx_ansible -e configuration_version=$CONFIG_VER $EXTRA_VARS
ansible-playbook -i localhost, -c local vagrant-devstack.yml -e configuration_version=$CONFIG_VER $EXTRA_VARS
```

```bash
$ chmod +x devstack.sh
$ sudo ./devstack.sh

```

After installation, try to run studio:

```bash
$ sudo su edxapp
$ paver devstack studio
```
Faild due to bundle not installed, looks like Ruby/Gems/Bundle files installed under /edx/app/edxapp/.gems, but not installed properly. Then reinstall them.

Set edxapp user environment varibles, execute following commands as ubuntu user.

```bash
$ sudo sed -i '/SKIP_WS_MIGRATIONS="1"/ a\export GEM_HOME="/edx/app/edxapp/.gem"\nexport CONFIG_ROOT="/edx/app/edxapp"\nexport GEM_PATH="/edx/app/edxapp/.gem"\nexport RBENV_ROOT="/edx/app/edxapp/.rbenv"' /edx/app/edxapp/edxapp_env
$ sudo sed -i '/export PATH=/ c\export PATH="/edx/app/edxapp/venvs/edxapp/bin:/edx/app/edxapp/edx-platform/bin:/edx/app/edxapp/.rbenv/bin:/edx/app/edxapp/.rbenv/shims:/edx/app/edxapp/.gem/bin:/edx/app/edxapp/edx-platform/node_modules/.bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"' /edx/app/edxapp/edxapp_env
$ sudo sed -i '/1.9.3/ c\1.9.3-p551' /edx/app/edxapp/edx-platform/.ruby-version
```

The following commands executed with user edxapp, refer to steps in /edx/app/edx_ansible/edx_ansible/playbooks/roles/rbenv/tasks/main.yml.

```bash
$ sudo su edxapp
$ rbenv versions | grep 1.9.3-p551
$ rbenv install 1.9.3-p551
$ rbenv global 1.9.3-p551
$ gem install bundler -v 1.11.2
$ gem install rake -v 10.4.2
$ gem install rubygems-update && update_rubygems
$ rbenv rehash

```

Then run studio again, it works but could not be accessed from remote, the firewall of EC2 blocked the inbound request. Just add new rule with port 8001 to security group, then everything looks fine.

To run studio without pip install requirements every time, set environment varible:
```bash
export NO_PREREQ_INSTALL=True
```
