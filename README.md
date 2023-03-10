ssid-changer.sh
===============

*WARNING: this is the old "master" branch of the script and it contains the 
ssid-changer version for the gluon 2016.2.x. For newer releses use the branches
"2017.1.x" and "2018.1.x.""*

This script changes the SSID when there is no connection to the selected Gateway.

Once a minute it checks if there's still a gateway reachable with 

site.conf
=========

Adapt and add this block to your site.conf: 

```
ssid_changer = {
  enabled = true,
  switch_timeframe = 30,  -- only once every timeframe (in minutes) the SSID will change to the Offline-SSID
                          -- set to 1440 to change once a day
                          -- set to 1 minute to change every time the router gets offline
  first = 5,              -- the first few minutes directly after reboot within which an Offline-SSID always may be activated (must be <= switch_timeframe)
  prefix = 'FF_Offline_', -- use something short to leave space for the nodename (no '~' allowed!)
  suffix = 'nodename',    -- generate the SSID with either 'nodename', 'mac' or to use only the prefix: 'none'
  
  tq_limit_enabled = false, -- if false, the offline SSID will only be set if there is no gateway reacheable
                            -- upper and lower limit to turn the offline_ssid on and off
                            -- in-between these two values the SSID will never be changed to prevent it from toggeling every minute.
  tq_limit_max = 45,        -- upper limit, above that the online SSID will be used
  tq_limit_min = 35         -- lower limit, below that the offline SSID will be used
},
```

if tq_limit is disabled, then it will be only checked, if the gateway is reachable with

    batctl gwl -H


Depending on the connectivity, it will be decided if a change of the SSID is 
necessary: There is a variable `switch_timeframe` (for ex.  1440 = 24h) that 
defines a time interval after which a successful check that detects an offline
state will result in a single change of the SSID to "FF_OFFLINE_$node_hostname".
Only the first few (also definable in a variable `first`) minutes the 
OFFLINE_SSID may also be set. All other minutes a checks will just be reported
in the log and whenever an online state is detected the SSID will be set back
immediately back to normal. 

Commandline options
===================

You can configure the ssid-changer on the commandline with `uci`, for example 
disable it with:

    uci set ssid-changer.settings.enabled='0'

Manual installation
===================

If you don't have ssid-changer in your firmware, you can still install it
manually on a node and set the desired settings that should differ from default:

```
ROUTER_IP='your:node::ip6'
LOGIN="root@[$ROUTER_IP]"
git clone https://github.com/Freifunk-Nord/gluon-ssid-changer.git ssid-changer
cd ssid-changer/gluon-ssid-changer/
git checkout master
scp -r files/* $LOGIN:/
ssh $ROUTER_IP "/lib/gluon/upgrade/500-ssid-changer;" \
  "uci set ssid-changer.settings.switch_timeframe='3';" \
  "uci set ssid-changer.settings.first='3';" \
  "uci commit ssid-changer;" \
  "uci show ssid-changer;" \
  "/etc/init.d/micrond reload;"
```

Alternative: gluon-ssid-notifier
================================
If you just need the Offline-SSID for administrative purposes, there is a better solution,
that will just add an extra SSID if a node is offline:
https://github.com/freifunk-kiel/gluon-ssid-notifier/


Gluon versions
==============
This branch of the script contains the ssid-changer version for the gluon 2016.2.x.


Implement this package in your firmware
=======================================
Create a file "modules" with the following content in your site directory:

```
GLUON_SITE_FEEDS="ssidchanger"
PACKAGES_SSIDCHANGER_REPO=https://github.com/freifunk-nord/gluon-ssid-changer.git
PACKAGES_SSIDCHANGER_COMMIT=384715c09b94f4b35099eeca27c2af0ed54a8a5d # <-- set the newest commit ID here
PACKAGES_SSIDCHANGER_BRANCH=master
```

With this done you can add the package `gluon-ssid-changer` to your `site.mk`


*This is a fork of https://github.com/ffac/gluon-ssid-changer that doesn't check
the tx value any more. It is now in use in Freifunk Nord*
