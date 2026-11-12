# Lab plan

A study plan for learning Linux administration by doing it. Nils Rönnbäck drafted the plan. The labs are done by Mae on a test machine, and each lab ends with a note in this repository.

The labs follow the courses and guides on [beginlinux.org](https://beginlinux.org) and then work through [vps-baseline](https://github.com/maenobleza/vps-baseline) and [selfhost-stack](https://github.com/maenobleza/selfhost-stack).

## Rules for the notes

- Do the lab yourself, on your own test machine. Write the note in your own words.
- Paste the command and the output you actually got. Remove passwords, keys and public IP addresses first.
- One note per lab: `category/YYYY-MM-DD-short-title.md`. The date is the day you did the lab. Never backdate a note.
- If something failed, write that down. A note about a failure is as useful as one about a success.
- If someone helped you, or you used an AI assistant, say so in the note.
- Run `bash scripts/index.sh` before you commit, and commit from your own account.
- `templates/example-note.md` shows the format.

## Test machine

Use a virtual machine with Ubuntu 24.04 or Debian 12. Multipass, UTM or VirtualBox on a laptop works, and so does a small rented VPS. Take a snapshot before labs 4, 5 and 7 so you can go back. Never practise on a machine that matters.

## Labs

| # | Lab | Note goes in | Done |
|---|---|---|---|
| 0 | First login | `linux/` | |
| 1 | Command line basics | `linux/` | |
| 2 | Where things live | `linux/` | |
| 3 | Users, groups and permissions | `linux/` | |
| 4 | SSH keys and sshd | `networking/` | |
| 5 | Firewall with nftables | `networking/` | |
| 6 | Services and logs | `linux/` | |
| 7 | First Ansible run | `ansible/` | |
| 8 | Docker and a first stack | `docker/` | |
| 9 | Reverse proxy | `docker/` | |
| 10 | Backup and restore | `docker/` | |

### 0. First login

- Goal: a working test machine you can log in to.
- Do: create the VM, log in, update the packages.
- Check: `cat /etc/os-release` and `uname -m`. Write down the architecture (`x86_64` or `aarch64`). You need it in lab 7.
- Note: how you created the machine and what was confusing.

### 1. Command line basics

- Read: [35 Linux commands](https://beginlinux.org/courses/linux-command-line/) and [Bash essentials](https://beginlinux.org/guides/bash-essentials/).
- Do: make a `practice` directory, create and move files, search them with `grep` and `find`, and connect two commands with a pipe.
- Check: `history | tail -20` shows what you ran.
- Note: three commands that were new to you and one mistake you made.

### 2. Where things live

- Read: [The Linux filesystem](https://beginlinux.org/guides/linux-filesystem-basics/).
- Do: find where logs, configuration files and home directories are. Run `df -h` and `sudo du -sh /var/log/*`.
- Check: you can say what `/etc`, `/var`, `/home` and `/usr` are for without looking it up.
- Note: the output of `df -h` and which directory was the biggest.

### 3. Users, groups and permissions

- Read: [Users, groups and permissions](https://beginlinux.org/guides/users-groups-permissions/).
- Do: create a user `labuser` and a group `web`. Make a shared directory that only the group can write to. Try to write there as a user outside the group.
- Check: `id labuser` and `ls -ld` on the directory.
- Note: the permission string of the directory and what each part means.

### 4. SSH keys and sshd

- Read: [SSH from zero](https://beginlinux.org/guides/ssh-basics/).
- Do: create an ed25519 key on your laptop, copy the public key to the test machine and log in with it. Add a host alias in `~/.ssh/config`.
- Then: read `roles/baseline/templates/sshd_config.j2` in vps-baseline and explain each line in your own words. Turn off password login by hand in a file under `/etc/ssh/sshd_config.d/`. Keep a second session open while you restart sshd.
- Check: `sudo sshd -t` reports nothing, and `ssh -o PubkeyAuthentication=no <host>` is refused.
- Note: your explanation of the template and what happened when you tested the password login.

### 5. Firewall with nftables

- Do: read `roles/baseline/templates/nftables.conf.j2`. Look at the current rules with `sudo nft list ruleset`. Write a small ruleset that allows SSH only. Start a test server with `python3 -m http.server 8080`.
- Check: from your laptop, `nc -zv <host> 22` works and `nc -zv <host> 8080` does not. Open port 8080 and test again.
- Note: your ruleset and the `nc` results before and after.

### 6. Services and logs

- Do: `systemctl status ssh`, `journalctl -u ssh --since today`, `journalctl --disk-usage`. Cap the journal size with a drop-in file as in `templates/example-note.md`.
- Check: `journalctl --disk-usage` before and after.
- Note: the two numbers and which directory you had to create first. That is the first open issue in vps-baseline.

### 7. First Ansible run

- Do: install Ansible on your laptop and write an inventory for the test machine. Run `ansible all -i inventory.ini -m ping`. Run vps-baseline with `--check --diff`, then for real on a fresh machine or a snapshot.
- Expect: some of the issues listed in the vps-baseline README. Write down every error exactly as it appeared.
- Then: fix one issue in a pull request to vps-baseline. Run the playbook a second time and write down the `changed=` number in the recap.
- Check: add a line under "Tested on" in the vps-baseline README.
- Note: what failed, what you fixed and the recap of the second run.

### 8. Docker and a first stack

- Do: `docker run hello-world`, `docker ps -a`, `docker volume ls`. Create the network with `docker network create proxy`. Start `uptime-kuma/` from selfhost-stack. The compose file publishes no port, so decide how to reach it and write down what you chose.
- Check: you can open Uptime Kuma and add a monitor for a website.
- Note: how you reached it and what `docker compose ps` shows.

### 9. Reverse proxy

- Do: put Caddy from `proxy/` in front of Uptime Kuma on the test machine. Without a public domain, change the Caddyfile to listen on `:80`.
- Check: `curl -I http://<host>/` returns a response from Uptime Kuma. If the firewall from lab 5 or 7 is active, save `sudo nft list ruleset` before and after `sudo systemctl reload nftables` and test again.
- Note: whether the firewall and Docker worked together. That answers one of the open issues in vps-baseline.

### 10. Backup and restore

- Do: create a local restic repository with `restic init --repo /srv/restic-test`. Back up the Uptime Kuma volume the way `backup/restic-backup.sh` does. Remove the volume. Restore it by following `backup/RESTORE.md`.
- Check: the monitor from lab 8 is back after the restore.
- Then: fill in "Last tested" in `backup/RESTORE.md` through a pull request, with the date and the real result.
- Note: each restore step and anything in RESTORE.md that was wrong or missing.
