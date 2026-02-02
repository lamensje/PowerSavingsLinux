# PowerSavingsLinux
PCIe ASPM &amp; USB Suspend tweaks for Linux

usbAutoSuspendDelay.sh
```
path="/sys/bus/usb/devices/"
usb=$(ls -A $path)
for device in $usb
do
 dev=$path$device/product
 if [[ -f $dev ]]
 then 
  if [[ $(cat $dev) =~ "Keychron" ]]
  then
   echo "Increasing autosuspend to 1 minute for $(cat $dev)"
   echo 60 > "$path$device/power/autosuspend"
  fi
  if [[ $(cat $dev) =~ "Receiver" ]]
  then
   echo "Increasing autosuspend to 1 minute for $(cat $dev)"
   echo 60 > "$path$device/power/autosuspend"
  fi
  if [[ $(cat $dev) =~ "Scarlett" ]]
  then
   echo "Increasing autosuspend to 1 minute for $(cat $dev)"
   echo 60 > "$path$device/power/autosuspend"
  fi
 fi
done
```

powertopExclusions.sh
```
path="/sys/bus/usb/devices/"
usb=$(ls -A $path)
for device in $usb
do
 dev=$path$device/product
 if [[ -f $dev ]]
 then 
  if [[ $(cat $dev) =~ "Keychron" ]]
  then
   echo "Disabling powersavings for $(cat $dev)"
   echo on > "$path$device/power/control"
  fi
  if [[ $(cat $dev) =~ "Receiver" ]]
  then
   echo "Disabling powersavings for $(cat $dev)"
   echo on > "$path$device/power/control"
  fi
  if [[ $(cat $dev) =~ "Scarlett" ]]
  then
   echo "Disabling powersavings for $(cat $dev)"
   echo on > "$path$device/power/control"
  fi
 fi
done
```

realtekASPM.sh
```
#!/bin/bash

# Warning: Use at your own risk. Created to aid with the Realtek C-State issues mentioned at:
# https://mattgadient.com/7-watts-idle-on-intel-12th-13th-gen-the-foundation-for-building-a-low-power-server-nas/

# INSTRUCTIONS:

# Rename this to end in .sh, then run it, maybe like this:
#     mv RTL8125-ASPM.sh.txt RTL8125-ASPM.sh
#     chmod +x RTL8125-ASPM.sh
#     ./RTL8125-ASPM.sh

# This will attempt to enable ASPM L1 on a Realtek RTL8125 and/or RTL8168, as recent Linux kernels
# disable L1 on most Realtek Ethernet adapters due to issues experienced by some
# users. This must be run as sudo/root. If it doesn't cause issues, run after each boot.

# You may need to change RTLDEVID from 0x8125 to the device ID of your Realtek network card.
# One way to find it is to run lspci, find the PCI ID of your Realtek Ethernet (example: 03:00.0),
# and then (using the 03:00.0 example above) type:
#    cat /sys/bus/pci/devices/0000\:03\:00.0/device
RTLVENID="0x10ec"
RTLDEVID="0x8125"
RTLDEVIDB="0x8168"

PCIDEVICES=$(ls /sys/bus/pci/devices/ | sort -u)
RTLFOUND=0
echo ""
echo "Checking for Realtek LAN and enabling ASPM L1"
for i in $PCIDEVICES; do
	DEVICEID=$(cat /sys/bus/pci/devices/$i/device)
	if [[ $DEVICEID == *"$RTLDEVID"* || $DEVICEID == *"$RTLDEVIDB"* ]]; then
		VENDORID=$(cat /sys/bus/pci/devices/$i/vendor)
		if [[ $VENDORID == *"$RTLVENID"* ]]; then
			echo " -Enabling L1 on /sys/bus/pci/devices/$i/link/l1_aspm"
			echo 1 > /sys/bus/pci/devices/$i/link/l1_aspm
			RTLFOUND=1
		fi
	fi
done
if [[ $RTLFOUND -eq 0 ]]; then
	echo " -Unable to find a matching PCI device with vendor id $RTLVENID and device id $RTLDEVID."
fi
```

/usr/lib/systemd/system/powertop.service
```
[Unit]
Description=PowerTOP autotuner

[Service]
Type=oneshot
ExecStart=/usr/sbin/powertop --auto-tune
ExecStart=/usr/bin/sh /root/powertopExclusions.sh
ExecStart=/usr/bin/sh /root/realtekASPM.sh

[Install]
WantedBy=multi-user.target
```
