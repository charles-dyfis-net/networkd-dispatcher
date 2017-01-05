# networkd-dispatcher
Dispatcher service for systemd-networkd connection status changes


## Usage

Application expects that scripts are executable and owned by root (gid = uid = 0), and will not execute scripts that are otherwise.

Scripts can be installed into these directories under ```/etc/networkd-dispatcher```:

```
routable.d/

dormant.d/

no-carrier.d/

off.d/
```

networkd-dispatcher will execute any valid scripts in the directory that reflects the new state. 

Scripts are executed in the alpha-numeric order in which they are named, starting with 0 and ending with z. For example, a script named ```50runme``` would run before ```99runmenext```.


## Installation

### Arch Linux

*Stay tuned for a package in AUR*

### Other Linux Folks

Copy networkd-dispatcher to /usr/bin.

Create the appropriate directory structure:

```$ sudo mkdir -p /etc/networkd-dispatcher/{routable,dormant,no-carrier,off}.d```

Install networkd-dispatcher.service and start it. If networkd-dispatcher was not copied to /usr/bin, then edit service file to reflect the appropriate path.


## Credits

A large portion of the code was leveraged from [networkd-notify](https://github.com/wavexx/networkd-notify), which was written by wave++ "Yuri D'Elia" <wavexx@thregr.org>
