## Installation

You can install ananicy manually by:
```
git clone https://github.com/c0d3h01/ananicy.git /tmp/ananicy
cd /tmp/ananicy
sudo make install
sudo systemctl enable --now ananicy
```

**Debian/Ubuntu:**
```
git clone https://github.com/c0d3h01/ananicy.git
./ananicy/package.sh debian
sudo dpkg -i ./ananicy/ananicy-*.deb
sudo systemctl enable --now ananicy
```

## Configuration
Rules files should be placed under `/etc/ananicy.d/` directory and have `*.rules` extension.
Inside .rules file every process is described on a separate line. General syntax is described below:

```
{ "name": "gcc", "type": "Heavy_CPU", "nice": 19, "ioclass": "best-effort", "ionice": 7, "cgroup": "cpu90" }
```

All fields except `name` are optional.

`name` used for match processes by exec bin name
```
~ basename $(sudo realpath /proc/1/exe)
systemd
```

Currently matching by other things is not supported.

You can check what Ananicy sees, by:
```
ananicy dump proc
```

Ananicy loads all rules in ram while starting, so to apply rules, you must restart the service.

Available ionice values:
```
man ionice
```

## Simple rules for writing rules
CFQ IO Scheduller also uses `nice` for internal scheduling, so it's mean processes with same IO class and IO priority, but with different nicceness will take advantages of `nice` also for IO.

1. Avoid changing `nice` of system wide process like initrd.
2. Please try to use full process name (or name with `^$` symbols like `NAME=^full_name$`)
3. When writing rule - try to only use `nice`, it must be enough in most cases.
4. Don't try set to high priority! Niceness can fix some performance problems, but can't give you more.
Example: pulseaudio uses `nice` -11 by default, if you set other cpu hungry task, with `nice` {-20..-12} you can catch a sound glitches.
5. For CPU hungry backround task like compiling, just use `NICE=19`.

About IO priority:

1. It's useful to use `{"ioclass": "idle"}` for IO hungry background tasks like: file indexers, Cloud Clients, Backups and etc.
2. It's not cool to set realtime to all tasks. The RT scheduling class is given first access to the disk, regardless of what else is going on in the system.  Thus the RT class needs to be used with some care, as it can starve other processes. So try to use ioclass first.

## Debugging
Get ananicy output with journalctl:
```
journalctl -efu ananicy.service
```

### Missing `schedtool`
If you see this error in the output
```
Jan 24 09:44:18 tony-dev ananicy[13783]: ERRO: Missing schedtool! Abort!
```
Fix it in Ubuntu with
```
sudo apt install schedtool
```

### Submitting new rules

Please use pull request, thanks
