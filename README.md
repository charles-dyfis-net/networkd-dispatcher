# networkd-dispatcher

Networkd-dispatcher is a dispatcher daemon for systemd-networkd connection status changes. This daemon is similar to [NetworkManager-dispatcher](https://developer.gnome.org/NetworkManager/unstable/NetworkManager.html), but is much more limited in the types of events it supports due to the limited nature of systemd-networkd. 

Desired actions (scripts) are placed into directories that reflect [systemd-networkd operational states](https://www.freedesktop.org/software/systemd/man/networkctl.html), and are executed when the daemon receives the relevant event from systemd-networkd.

The deamon listens for signals from systemd-networkd over dbus, so it should be very light on resources (e.g. no polling). It is meant to be run as a system-wide daemon (as root). This allows it to be used for tasks such as starting a VPN after a connection is established.

## Usage

The deamon expects that scripts are 1) executable and 2) owned by root (gid = uid = 0), and will not execute scripts that are otherwise.

Scripts can be installed into these directories under ```/etc/networkd-dispatcher```:

```
routable.d/

dormant.d/

no-carrier.d/

off.d/
```

networkd-dispatcher will execute any valid scripts in the directory that reflects the new state. 

Scripts are executed in the alpha-numeric order in which they are named, starting with 0 and ending with z. For example, a script named ```50runme``` would run before ```99runmenext```.

Scripts are executed with some environment variables set. Some of these variables may not be set or may be set to an empty value, dependent upon the type of event. These can be used by scripts to conditionally take action based on a specific interface, state, etc.

- ```IFACE``` - interface that triggered the event

- ```STATE``` - systemd-networkd state received by daemon

- ```ESSID``` - for wlan connections, the ESSID the device is connected to

- ```ADDR``` - the ipv4 address of the device

- ```IP_ADDRS``` - space-delimited string of ipv4 address(es) assigned to the device (see note below)

- ```IP6_ADDRS``` - space-delimited string of ipv6 address(es) assigned to the device (see note below)

- ```json``` - A JSON encoding of this program's interpretation of `networkctl status "$IFACE"`, when the event is one for which such information is available; for debug logs or inspection with JSON-aware tools such as `jq`. Exact structure details are implementation-defined and liable to change.

*Note: For `IP_ADDRS` and `IP6_ADDRS`, the space-delimited string can be read into a BASH array like this:

```read -r -a ip_addrs <<<"$IP_ADDRS"```

### Command-Line Options

```
usage: networkd-dispatcher [-h] [-S SCRIPT_DIR] [-T] [-v] [-q]

networkd dispatcher daemon

optional arguments:
  -h, --help            show this help message and exit
  -S SCRIPT_DIR, --script-dir SCRIPT_DIR
                        Location under which to look for scripts [default:
                        /etc/networkd-dispatcher]
  -T, --run-startup-triggers
                        Generate events reflecting preexisting state and
                        behavior on startup [default: False]
  -v, --verbose         Increment verbosity level once per call
  -q, --quiet           Decrement verbosity level once per call
```

Some further notes:

- The intended use case of `--run-startup-triggers` is race-condition avoidance: Ensuring that triggers are belatedly run even if networkd-dispatcher is invoked after systemd-networkd has already started an interface.
- The default log level is `WARNING`. Each use of `-v` will increment the log level (towards `INFO` or `DEBUG`), and each use of `-q` will decrement it (towards `ERROR` or `CRITICAL`).

## Installation

### Arch Linux

This package can be [installed from AUR](https://aur.archlinux.org/packages/networkd-dispatcher/).

### Other Linux Folks

Requirements:

- \>= python 3.4

- python-gobject

- python-dbus

- wireless_tools


Copy networkd-dispatcher to /usr/bin.

Create the appropriate directory structure:

```$ sudo mkdir -p /etc/networkd-dispatcher/{routable,dormant,no-carrier,off}.d```

Install networkd-dispatcher.service and start it. If networkd-dispatcher was not copied to /usr/bin, then edit service file to reflect the appropriate path.


## Contributors

- craftyguy (Clayton Craft)

- charles-dyfis-net (Charles Duffy)



A large portion of the code was leveraged from [networkd-notify](https://github.com/wavexx/networkd-notify), which was written by wavexx (Yuri D'Elia)
