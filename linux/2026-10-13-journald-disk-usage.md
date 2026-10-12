# journald quietly eats disk on a small VPS

A 20 GB box hit 95 % disk. `du -sh /var/log/*` pointed at `/var/log/journal` (3.8 GB).

Check:
```
journalctl --disk-usage
```
Cap it permanently:
```
mkdir -p /etc/systemd/journald.conf.d
printf '[Journal]\nSystemMaxUse=200M\n' > /etc/systemd/journald.conf.d/size.conf
systemctl restart systemd-journald
```
Vacuum what is already there:
```
journalctl --vacuum-size=200M
```
Added this to my baseline role so it never happens again.
