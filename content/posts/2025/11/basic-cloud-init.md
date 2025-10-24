+++
title = "Basic Cloud-Init script for most VPSs"
date = "2025-10-24T19:16:01-04:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Ema"
tags = ["tech-projects", "automation"]
keywords = ["yaml", "cloud"]
description = ""
showFullContent = false
readingTime = false
hideComments = true
draft = true
+++

Look at this cool code!!!

```yaml
#cloud-config

# Disable password authentication for SSH
ssh_pwauth: false

# Create users
users:
  - name: ema
    gecos: Ema User
    sudo: ALL=(ALL) ALL
    groups: users, sudo
    shell: /bin/bash
    lock_passwd: false
    passwd: #HAshed password here
    ssh_authorized_keys:
      - "ssh-pubkey-here"
  - name: apps
    groups: users, docker
    shell: /bin/bash
    lock_passwd: false
    passwd: # your hashed password here


# Configure SSH daemon
write_files:
  - path: /etc/ssh/sshd_config.d/99-custom.conf
    content: |
      # Custom SSH configuration
      PasswordAuthentication no
      ChallengeResponseAuthentication no
      UsePAM yes
      PubkeyAuthentication yes
      AuthorizedKeysFile .ssh/authorized_keys
    permissions: '0644'

# Restart SSH service to apply configuration changes
runcmd:

  - sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
  - sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
  - sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
  - sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
  - sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no' /etc/ssh/sshd_config
  - sed -i 's/PasswordAuthentication yes/PasswordAuthentication no' /etc/ssh/sshd_config
  - sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config
  - systemctl restart sshd
  - systemctl enable sshd

package_update: true
package_upgrade: true

# Reboot after package upgrades
power_state:
  mode: reboot
  message: "Rebooting after package upgrades"
```