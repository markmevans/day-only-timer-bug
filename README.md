# day-only-timer-bug
If you create a snap with a timer string with only a weekday in it,
the resulting snap will put snapd in an unrecoverable state where it
is stuck in a cycle of consuming all memory and restarting.

## Duplicate Bug
THIS WILL PUT SNAPD IN A STATE THAT IT DOES NOT RECOVER FROM.
EVEN AFTER A REBOOT.
So don't run it unless you can wipe and reinstall your system.

### Install the snap

    admin@xxxxxxx:~$ snap install destroy-snapd_0.1_amd64.snap --devmode
    error: cannot communicate with server: timeout exceeded while waiting for response

The install "hangs" for a long time. Eventually it will error out. `snapd` will have
run out of memory. If `snap list` works (rather than times out), `destroy-snapd`
is not listed which makes sense. But `snapd` will continue to consume memory
until it auto restarts and the cycle will continue.

#### Install logs

Complete log here [snapd-intall.log](logs/snapd-install.log)
```
Mar 18 22:18:25 xxxxxxx snapd[3566]: fatal error: runtime: out of memory
Mar 18 22:18:25 xxxxxxx snapd[3566]: runtime stack:
Mar 18 22:18:25 xxxxxxx snapd[3566]: runtime.throw(0x561d5b97be30, 0x16)
. . .
Mar 18 22:47:09 xxxxxxx systemd[1]: snapd.service: Main process exited, code=exited, status=2/INVALIDARGUMENT
Mar 18 22:47:09 xxxxxxx systemd[1]: snapd.service: Unit entered failed state.
Mar 18 22:47:09 xxxxxxx systemd[1]: snapd.service: Triggering OnFailure= dependencies.
Mar 18 22:47:09 xxxxxxx systemd[1]: snapd.service: Failed with result 'exit-code'.
Mar 18 22:47:09 xxxxxxx systemd[1]: snapd.service: Service hold-off time over, scheduling restart.
Mar 18 22:47:09 xxxxxxx systemd[1]: Stopped Snappy daemon.
Mar 18 22:47:09 xxxxxxx systemd[1]: Starting Snappy daemon...
Mar 18 22:47:11 xxxxxxx snapd[2500]: AppArmor status: apparmor is enabled and all features are available
Mar 18 22:47:11 xxxxxxx snapd[2500]: daemon.go:346: started snapd/2.43.3 (series 16) ubuntu-core/16 (amd64) linux/4.4.0-176-generic.
Mar 18 22:47:11 xxxxxxx snapd[2500]: daemon.go:439: adjusting startup timeout by 1m40s (pessimistic estimate of 30s plus 5s per snap)
Mar 18 22:47:12 xxxxxxx systemd[1]: Started Snappy daemon.
Mar 18 22:51:54 xxxxxxx snapd[2500]: fatal error: runtime: out of memory
Mar 18 22:51:54 xxxxxxx snapd[2500]: runtime stack:
. . .
```

### Reboot the Machine

Reboot the machine, `snapd` continues the same behavior.

```
admin@xxxxxxx:~$ snap list
error: cannot list snaps: cannot communicate with server: timeout exceeded while waiting for response
admin@xxxxxxx:~$ snap list
error: cannot list snaps: cannot communicate with server: timeout exceeded while waiting for response
```

#### snapd post reboot log

Complete log here [snapd-after-reboot.log](logs/snapd-after-reboot.log)

```
admin@xxxxxxx:~$ sudo journalctl -n 10000 -f -u snapd
-- Logs begin at Wed 2020-03-18 22:43:32 UTC. --
Mar 18 22:43:44 xxxxxxx systemd[1]: Starting Snappy daemon...
Mar 18 22:43:47 xxxxxxx snapd[1645]: AppArmor status: apparmor is enabled and all features are available
Mar 18 22:43:47 xxxxxxx snapd[1645]: daemon.go:346: started snapd/2.43.3 (series 16) ubuntu-core/16 (amd64) linux/4.4.0-176-generic.
Mar 18 22:43:47 xxxxxxx snapd[1645]: daemon.go:439: adjusting startup timeout by 1m40s (pessimistic estimate of 30s plus 5s per snap)
Mar 18 22:43:48 xxxxxxx systemd[1]: Started Snappy daemon.
Mar 18 22:47:09 xxxxxxx snapd[1645]: fatal error: runtime: out of memory
Mar 18 22:47:09 xxxxxxx snapd[1645]: runtime stack:
Mar 18 22:47:09 xxxxxxx snapd[1645]: runtime.throw(0x556209804e30, 0x16)
```
## Tested Build Environment

### MacOS
 * Catalina (10.15.3 (19D76))
 * snapcraft (3.9.1)
 * multipass (1.1.0+mac)
 
 ## Tested Runtime Environment
 
 Ubuntu Core 16
 
    caracalla         16.04-1.38     52    stable  canonical✓  gadget
    caracalla-kernel  4.4.0-176.206  127   stable  canonical✓  kernel
    core              16-2.43.3      8689  stable  canonical✓  core
