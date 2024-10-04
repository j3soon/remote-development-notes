# DHCP Lease Renewal

This is useful when setting up a server in an intranet to ensure it has a static IP address. After configuring the router's DHCP settings, you may need to manually renew the DHCP lease to apply changes.

## Linux

Run the following commands according to [this post](https://serverfault.com/a/42804):

```sh
# Set interface name, some examples:
#   INTERFACE=wlan0
#   INTERFACE=eth0
sudo dhclient -r $INTERFACE && sudo dhclient $INTERFACE
```

## Windows

Run the following commands according to [this post](https://computing.cs.cmu.edu/desktop/ip-renew):

```
ipconfig /release
ipconfig /renew
```
