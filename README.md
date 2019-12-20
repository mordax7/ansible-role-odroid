# ansible-role-odroid

Slightly modified [hannseman/ansible-raspbian](https://github.com/hannseman/ansible-raspbian) roles to be compatible
with a Ordoird(Ubuntu 18.04) device. It should also work with any other Debian based system.

### It will:

 * Install specified system packages.
 * Configure hostname.
 * Configure locale.
 * Mount tmpfs on write-intensive directories to increase the lifespan of SD-card. 
 * Change the password on default user.
 * Set the default editor.
 * Setup a secure SSH configuration.
 * Configure Postfix to send email through an SMTP relay.
 * Enable unattended-upgrades.
 * Configure Logwatch to send weekly reports.

### It will not:

 * Update system packages.
 * Run `apt-get update`. Please do this in a pre_task. See [Example Playbook](#example-playbook).
 * Install security patches but unattended-upgrades should take care of that. 

## Setup
* Flash SD-card with odroid Ubuntu.
* Run playbook.

## Variables

```yaml
system_hostname: "odroid"
ssh_user: "test"
system_locale: "en_US.UTF-8"
system_timezone: "Europe/Stockholm"
system_tmpfs_mounts:
  - { src: "/run", size: "10%", options: "nodev,noexec,nosuid" }
  - { src: "/tmp", size: "10%", options: "nodev,nosuid" }
  - { src: "/var/log", size: "10%", options: "nodev,noexec,nosuid" }

system_packages: []
system_default_editor_path: "/usr/bin/vi"
logwatch_tmp_dir: /var/cache/logwatch
logwatch_mailto: "root"
logwatch_detail: "Low"
logwatch_interval: "weekly"

postfix_hostname: "{{ ansible_hostname }}"
postfix_mailname: "{{ ansible_hostname }}"
postfix_mydestination:
  - "{{ postfix_hostname }}"
  - localdomain
  - localhost
  - localhost.localdomain
postfix_relayhost: smtp.gmail.com
postfix_relayhost_port: 587
postfix_sasl_user:
postfix_sasl_password:
postfix_smtp_tls_cafile: /etc/ssl/certs/ca-certificates.crt

ssh_sshd_config: "/etc/ssh/sshd_config"
ssh_public_keys: []
ssh_banner:

inteerfaces: ""

unattended_upgrades_email_address: root

bash_aliases: ""
```

## Example Playbook
```yaml
- hosts: servers
  become: true
  pre_tasks:
    - name: update apt cache
      apt:
        cache_valid_time: 600
  roles:
    - role: xmordax.odroid
  vars:
    system_packages:
      - apt-transport-https
      - vim
    system_default_editor_path: "/usr/bin/vim.basic"
    system_ssh_user_password: hunter2
    system_ssh_user_salt: pepper
    postfix_sasl_user: root@example.com
    postfix_sasl_password: hunter2

    ssh_public_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJXTGInmtpoG9rYmT/3DpL+0o/sH2shys+NwJLo8NnCj
```
