# Fix Intel CPU Throttling on Linux
Workaround for Linux throttling issues on Lenovo T480 / T480s / X1C6 notebooks as described [here](https://www.reddit.com/r/thinkpad/comments/870u0a/t480s_linux_throttling_bug/).

This script forces the CPU package power limit (PL1/2) to **44 W** (29 W on battery) and the temperature trip point to **95 'C** (85 'C on battery) by overriding default values in MSR and MCHBAR every 5 seconds (30 on battery) to block the Embedded Controller from resetting these values to default.

Tested and working also for **T580**!

### Is this script really doing something on my PC??
I suggest you to use the excellent **[s-tui](https://github.com/amanusk/s-tui)** tool to check and monitor the CPU usage, frequency, power and temperature under load! 

### Undervolt:
The script now also supports **undervolting** the CPU by configuring voltage offsets for CPU, cache, GPU, System Agent and Analog I/O planes. The script will re-apply undervolt on resume from standby and hibernate by listening to DBus signals.

### HWP override (EXPERIMENTAL):
I have found that under load my CPU was not always hitting max turbo frequency, in particular when using one/two cores only. For instance, when running [prime95](https://www.mersenne.org/download/) (1 core, test #1) my CPU is limited to about 3500 MHz over the theoretical 4000 MHz maximum. The reason is the value for the HWP energy performance [hints](http://manpages.ubuntu.com/manpages/artful/man8/x86_energy_perf_policy.8.html). By default TLP sets this value to "balance_performance" on AC in order to reduce the power consumption/heat in idle. By setting this value to "performance" I was able to reach 3900 MHz in the prime95 single core test, achieving a +400 MHz boost. Since this value forces the CPU to full speed even during idle, a new experimental feature allows to automatically set HWP to performance under load and revert it to balanced when idle. This feature can be enabled (in AC mode *only*) by setting to `True` the `HWP_Mode` parameter in the config file. 

I have run **[Geekbench 4](https://browser.geekbench.com/v4/cpu/8656840)** and now I can get a score of 5391/17265! On balance_performance I can reach only 4672/16129, so **15% improvement** in single core and 7% in multicore, not bad ;)

## Requirements
The python module `python-periphery` is used for accessing the MCHBAR register by memory mapped I/O. You also need `dbus` and `gobject` python bindings for listening to dbus signals on resume from sleep/hibernate.

### Secure Boot:
Right now it is mandatory to **disable Secure Boot** (in BIOS) in order to avoid [Kernel Lockdown](https://lwn.net/Articles/706637/). In particular Lockdown restricts access to MSR and PCI BAR (via /dev/mem) which are required by this script.

### Update:
The scripts is now running with Python3 by default (tested w/ 3.6) and a virtualenv is automatically created in `/opt/lenovo_fix`. Python2 should probably still work.

## Installation
```
sudo apt install git virtualenv build-essential python3-dev libdbus-glib-1-dev libgirepository1.0-dev libcairo2-dev
git clone https://github.com/erpalma/lenovo-throttling-fix.git
sudo ./install.sh
```

If you own a X1C6 you can also check a tutorial for Ubuntu 18.04 [here](https://mensfeld.pl/2018/05/lenovo-thinkpad-x1-carbon-6th-gen-2018-ubuntu-18-04-tweaks/).

## Configuration
The configuration has moved to `/etc/lenovo_fix.conf`. Makefile does not overwrite your previous config file, so you need to manually check for differences in config file structure when updating the tool. If you want to overwrite the config with new defaults just issue `sudo cp etc/lenovo_fix.conf /etc`. There exist two profiles `AC` and `BATTERY` and the script can be totally disabled by setting `Enabled: False` in the `GENERAL` section. Undervolt is applied if any voltage plane in the config file (section UNDERVOLT) was set. Notice that the offset is in *mV* and only undervolting (*i.e.* negative values) is supported.  
All fields accept floating point values as well as integers. 

My T480s with i7-8550u is stable with:
```
[UNDERVOLT]
# CPU core voltage offset (mV)
CORE: -110
# Integrated GPU voltage offset (mV)
GPU: -90
# CPU cache voltage offset (mV)
CACHE: -110
# System Agent voltage offset (mV)
UNCORE: -90
# Analog I/O voltage offset (mV)
ANALOGIO: 0
```

## Disclaimer
This script overrides the default values set by Lenovo. I'm using it without any problem, but it is still experimental so use it at your own risk. This script can be probably adapted/used on other notebooks too.
