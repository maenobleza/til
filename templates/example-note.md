# Example note: capping the journald log size

> This file shows the note format. Nils Rönnbäck wrote it as a template when the repository was set up. It is not one of Mae's notes and it is not listed in the index.

journald keeps logs under `/var/log/journal` and can use a lot of space on a small disk.

Check how much it uses:
```
journalctl --disk-usage
```
Cap it permanently:
```
mkdir -p /etc/systemd/journald.conf.d
printf '[Journal]\nSystemMaxUse=200M\n' > /etc/systemd/journald.conf.d/size.conf
systemctl restart systemd-journald
```
Shrink what is already there:
```
journalctl --vacuum-size=200M
```

A real note also says what you saw on your own machine: the numbers before and after, and anything that did not work the first time.
