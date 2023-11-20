# **Ansible-LinuxCommon**

Ansible-LinuxCommon is an Ansible role for configuring shell environment and installing some very basic utilities on a new Linux system. It does the following on a default install:
* Change hostname.
* Replace the host file according to the hostname.
* Replace a pre-modified `/etc/bash.bashrc`, `.profile` and `.bashrc` files for root and `ansible_ssh_user`.
* Replace default `sources.list` to include all official repositories e.g. `main`, `contrib`, `non-free` (in case of Debian) and `main`, `restricted`, `universe`, `multiverse`, `partner` (in case of Ubuntu).
* Install updates.
* Install 3rd party repositories.
* Installs `htop`, `iotop`, `iftop`, `ncdu`, `ntp`, `ntpdate`, `curl`, `bash-completion`, `vim`, `mtr-tiny`, `git` and `unzip`.
* Purges `exim`.
* configures `vim` editor.
* Install `sudo`.
* Configure password-less `sudo` for `sudo` group users.
* Add `ansible_ssh_user` to sudo group.
* Installs `libpam-systemd` (only for Debian Jessie. See [this](https://serverfault.com/questions/706475/ssh-sessions-hang-on-shutdown-reboot) thread)

`bash.bashrc `is a modified version of the default file that comes with Debian.

Bits for `sudo` are added incase you have installed Debian via CD/DVD.

## **Requirements**

At the moment this role is being written for Debian based distributions e.g. Debian/Ubuntu. It will evolve to include other major distributions.

## **Role Variables**

The variable `LC_CHANGE_HOSTNAME` in `defaults/main.yml` based on which the role decides wether to change the hostname or not. By default it is set to `True`. If you don't want to change change hostname, you can set it to `False`. See the example below. The role sets `inventory_hostname_short` as the hostname.

Based on `LC_SETUP_SUDO` variable in `defaults/main.yml`, the role can decide if `sudo` related actions should be performed. By defaults its set to `True`. You can change it to `False` if you don't want the role to perform `sudo` actions. See the example below.

To modify shell environment set appropriate value for `LC_MODIFY_SYSTEM_SHELL_ENV`, `LC_MODIFY_SKEL`, `LC_MODIFY_ROOT_SHELL_ENV` and `LC_MODIFY_USER_SHELL_ENV`.

`LC_ENABLE_SRC_REPOS` controls if source repositories should be enabled or not. By default it is set to `False`.

Variables:
* `LC_DEBIAN_MIRROR`
* `LC_DEBIAN_SECURITY_MIRROR`
* `LC_UBUNTU_MIRROR`
* `LC_UBUNTU_CANONICAL_PARTNER_MIRROR`

Can be used to change mirrors for package repositories. See `defaults/main.yml` for their default values.

By default, variables:
* `LC_DEBIAN_REPOS`
* `LC_UBUNTU_REPOS`

Are used to enable repositories `main`, `contrib`, `non-free` (in case of Debian) and `main`, `restricted`, `universe`, `multiverse`, `partner` (in case of Ubuntu).

By default, the value of `LC_REBOOT` is `True`. The remote machine will reboot after performing all operations and wait for it to come back for `LC_REBOOT_TIMEOUT` which is 120 seconds. The reboot task pings SSH port defined in `LC_SSH_PORT` which defaults to 22.

You can add 3rd party package repositories to the system by using the following variables:
* `LC_SETUP_THIRD_PARTY_REPOS` (by default set to `False`).
* `LC_THIRD_PARTY_REPOS_KEYS`.
* `LC_THIRD_PARTY_REPOS`.

See example below on how to add 3rd party repositories.

To install default (as defined in `vars/main.yml`) and extra packages set `LC_INSTALL_PACKAGES` to true. You may provide a list of extra packages that you want to install via `LC_EXTRA_PACKAGES`. See example below.

To change the timezone of the system, set `LC_CHANGE_TIMEZONE` to `True` and give `LC_TIMEZONE` a value e.g. `"Asia/Karachi"`.

You can setup system locales. To do that, use the variables `LC_SET_LOCALES`, `LC_LOCALES` and `LC_DEFAULT_LOCALE` as shown in the example below.

Set kernel parameters in `sysctl.conf` by setting variable `LC_SET_KERNEL_PARAMETERS` to `True` and setting values in `LC_KERNEL_PARAMETERS`.

## **Dependencies**

This role doesn't depends on any other role for execution. The role is written to support both CD/DVD installs and pre-installed images (e.g. Amazon Machine Images) where `sudo` is already installed and configured. You can also use `su` method to become root if you don't have sudo installed.

## **Example Playbook**

Facts gathering must be enabled.

An example of running the role is as follows:
```yml
- hosts: servers
  gather_facts: True
  roles:
      - Ansible-LinuxCommon
```
If you don't want to change hostname:
```yml
- hosts: servers
  gather_facts: True
  roles:
      - { role: Ansible-LinuxCommon, LC_CHANGE_HOSTNAME: False }
```
If you don't want to add the user to password-less `sudo` or if its already there:
```yml
- hosts: servers
  gather_facts: True
  roles:
      - { role: Ansible-LinuxCommon, LC_SETUP_SUDO: False }
```
Add 3rd party repositories like Ondřej Surý's PHP repository and Syncthing:
```yml
- hosts: servers
  gather_facts: True
  roles:
    - role: Ansible-LinuxCommon
      LC_SETUP_3RD_PARTY_REPOS: True
      LC_THIRD_PARTY_REPOS_KEYS:
        - "https://packages.sury.org/php/apt.gpg"
        - "https://syncthing.net/release-key.txt"
      LC_THIRD_PARTY_REPOS:
        - NAME: "php"
          SCHEME: "https"
          URI: "packages.sury.org/php"
          RELEASE: "stretch"
          REPOS: "main"
        - NAME: "syncthing"
          SCHEME: "https"
          URI: "apt.syncthing.net"
          RELEASE: "syncthing"
          REPOS: "stable"
```
Setup locales:
```yml
- hosts: servers
  gather_facts: True
  roles:
    - role: Ansible-LinuxCommon
      LC_SET_LOCALES: True
      LC_LOCALES:
        - "en_US ISO-8859-1"
        - "en_US.ISO-8859-15 ISO-8859-15"
        - "en_US.UTF-8 UTF-8"
      LC_DEFAULT_LOCALE: "en_US.UTF-8 UTF-8"
```
Install extra packages:
```yml
- hosts: servers
  gather_facts: True
  roles:
      - role: Ansible-LinuxCommon
        LC_SETUP_SUDO: False
        LC_CHANGE_HOSTNAME: False
        LC_EXTRA_PACKAGES:
          - iptstate
          - iptables-persistent
```
Modify kernel parameters:
```yml
- hosts: servers
  gather_facts: True
  roles:
      - role: Ansible-LinuxCommon
        LC_SET_KERNEL_PARAMETERS: True
        LC_KERNEL_PARAMETERS:
          - { name: "vm.swappiness", value: 10, state: present }
          - { name: "net.core.somaxconn", value: 65535, state: present }
```
## **License**

This playbook is licensed under MIT License (See the LICENSE file).

## **Author**

[Saad Ali](https://github.com/nixknight)
