# README

This repository contains Ansible scripts for setupping my home servers. For developing said scripts it also contains configuration for Vagrant.

NOTE: this is a work in progress. All roles may not have been fully tested.

### How do I get set up?

Pre-requirements and assumptions:

- You are using MacOS
- You have MacPorts installed
- You are familiar with SSH and are using SSH config (because it's awesome)

#### Summary of set up

- Install Vagrant
  -- Download Vagrant from their website and install it
- Install Ansible
  -- `sudo port install py310-ansible`

#### Configuration for development

You need to add Vagrant boxes into your SSH config:

```
# ~/.ssh/config
Host vg-suksiboksi-rpi3
    HostName 127.0.0.1
    User vagrant
    Port 2222
    StrictHostKeyChecking no
    PasswordAuthentication no
    IdentityFile <PATH TO THIS REPOSITORY>/suksiboksi-ansible/.vagrant/machines/vg-suksiboksi-rpi3/virtualbox/private_key
    IdentitiesOnly yes
    LogLevel FATAL
Host vg-suksiboksi-rpi4
    HostName 127.0.0.1
    User vagrant
    Port 2200
    StrictHostKeyChecking no
    PasswordAuthentication no
    IdentityFile <PATH TO THIS REPOSITORY>/suksiboksi-ansible/.vagrant/machines/vg-suksiboksi-rpi4/virtualbox/private_key
    IdentitiesOnly yes
    LogLevel FATAL
```

#### Ansible Vault

Note: Ansible Vault files are excluded from version control. You need to create files with `ansible-vault` command:

```
ansible-vault create --vault-id vault@prompt inventories/<inventory>/group_vars/all/secrets.yml
```

Template for Ansible Vault secrets file:

```
pihole:
  admin_password: password
  private_domains:
    - example.com
    - someelse.com
caddy:
  grafana:
    duckdns_token: <your token>
    duckdns_domain: <your domain>
  pihole:
    domain: http://pi.hole
  localhost_name: localhost:80
  localhost_ip: 192.168.50.11
  email: somebody@example.com
pivpn:
  ipv4_dev: enp0s3
  ipv4_addr: 192.168.1.206/24
  ipv4_gw: 192.168.1.1
  pivpn_port: 12345
  pivpn_dns1: 192.168.1.100
  pivpn_dns2: 192.168.1.101
  pivpn_host: example.com
  allowed_ips: 0.0.0.0/0, ::0/0
docker:
  network: some_network
  pihole:
    service_address: pihole:80
  grafana:
    service_address: grafana:3000
  influxdb:
    service_address: influxdb:8086
ssh:
  allowed_users: vagrant
ruuvi:
  external_configuration:
    AA_2C_6A_1E_59_3D: "Balcony"
    CC_2C_6A_1E_59_3D: "Living room"
```

#### Ansible setup structure

This eetup currently both supports developing Ansible scripts in "staging" environment and running changes to "production" (NOTE: this is a work in progress and not all production configuration is available at the time of writing). The setup does not require Ansible to be installed in the target machine: SSH connection to the machine is sufficient.

Staging environment mimics what would be the production: this is done with the help of Vagrant based Ubuntu boxes `vg-suksiboksi-rpi3` and `vg-suksiboksi-rpi4`. To create/start these boxes, run `vagrant up` in the root of this repository.

These environments are defined under `inventories` directory. Under each inventory there's host definitions and variables used by roles. Under `group_vars/all` there's also Ansible Vault encrypted file `secrets.yml` which holds sensitive values such as passwords.

Under `roles` directory there's a subdirectory for each tool/software to be installed. These roles contain tasks to that are needed to install and configure the installable. There IS an order in which these roles should be installed: this is not enforced by any mechanism but it is assumed whoever is running these roles knows what are the pre-requirements (hint: install `common` first, then `docker`). Some of these roles are helpers to other roles, e.g. `git_installer` helps with cloning wanted GIT repository and copying its contents to wanted destination.

Each installable has its own playbook file. These files tell Ansible which role to install into which host.

#### Command cheat-sheet

Go to `ansible` directory.

- Install Galaxies: `ansible-galaxy collection install -r requirements.yml`
- Run playbook: `ansible-playbook -i inventories/staging -D <playbook>.yml --ask-vault-pass`
- Run playbook with GoPass provided Vault password: `ansible-playbook -i inventories/staging -D <playbook>.yml --vault-password-file ./scripts/staging-vault-pass`
- Edit secrets: `EDITOR='code --wait' ansible-vault edit inventories/<inventory>/group_vars/all/secrets.yml`
