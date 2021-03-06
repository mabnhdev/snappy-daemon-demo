# Snappy Daemon Demo #

This repository is used to illustrate issues/questions about Snappy daemonization bahavior.


## Test Daemon ##

All daemons use a common "daemon" implementation found in src/tests/usr/bin/test-daemon

```
src/tests/usr/bin/test-daemon --help  

usage: test-daemon [-h] [--myid MYID] [--daemonize] [--notify]  
                   [--exitafter EXITAFTER] [--exitstatus EXITSTATUS]  

optional arguments:  
  -h, --help            show this help message and exit  
  --myid MYID  
  --daemonize  
  --notify  
  --exitafter EXITAFTER exit after N minutes.  
  --exitstatus EXITSTATUS exit status to return.
```


## Test Snap ##

* To install the snap:
  * sudo snap install --devmode --dangerous snappy-daemon-demo_*.snap

* To remove the snap:
  * sudo snap remove snappy-daemon-demo


## Version 0.1 ##

This was tested on a **Ubuntu Xenial Core** system.

The following daemons are started:

* test-oneshot
  * A oneshot daemon that logs a hello message once and exits.

* test-simple
  * A simple daemon that logs a hello message every 60 seconds.

* test-notify
  * A notify daemon that logs a hello message every 60 seconds.
  * It notifies systemd as soon as it starts up.

* test-no-notify
  * A notify daemon that logs a hello message every 60 seconds.
  * It **DOES NOT** notify systemd which causes the snap installation to fail.

* test-forking
  * Currently commented out because snappy is not playing nicely with python daemonization code.

This illustrates some immediate issues.

1. Snap "hangs" trying to start up *test-no-notify* and fails to start several of the other daemons.

   Since snap/snapcraft doesn't support explicit ordering, I don' think it should exhibit any implicit ordering.  I would expect snap to start all daemons.  A hung notify daemon shouldn't be preventing other daemons from starting.

   To make matters worse, there doesn't seem to be any determinism with regards to which daemons are started and which ones are not when the *test-no-notify* hangs.

2. When a snap installation fails, cleanup is incomplete.

   This snap fails to install, but other started daemons are left running and orphaned.

3. Snappy also hangs when instatiating test-forking.

   The test-daemon uses the standard python daemon library to daemonize, but this does not seem sufficient for snappy.

   Like the "test-no-notify" hang, the snap installation failure leaves some of the other daemons running and orphaned.