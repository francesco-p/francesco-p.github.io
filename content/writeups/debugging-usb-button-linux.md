---
title: "Debugging a USB Button on Linux: From USB Detection to HID Failure"
date: 2026-02-26
summary: "Linux detected the USB button, but HID init failed because the report descriptor was empty."
tags: ["linux", "usb", "hid", "debugging", "writeup"]
---

`lsusb` detected the Delcom button (`0fc5:b080`), but no `/dev/input` or `/dev/hidraw` node appeared.

![Delcom USB FS IO button used in the debugging walkthrough](/imgs/posts/usb_button.jpg)

## Key Finding

The HID report descriptor was empty:

```bash
wc -c /sys/bus/hid/devices/*0FC5:B080*/report_descriptor
# 0
```

If the report descriptor is `0`, Linux cannot parse inputs, so it will not create usable HID/input device nodes.

## Why This Happens

USB visibility is not enough. A HID device must pass all stages:

1. USB enumeration (`lsusb`, `dmesg`)
2. HID driver binding (`usbhid`)
3. Valid report descriptor
4. `/dev/hidrawX` or `/dev/input/eventX` creation

Failure at step 3 stops step 4.

## Most Likely Causes

- unstable USB path (port/cable/hub/power)
- device firmware or hardware failure

## Fast Next Check

Try the button through a USB 2.0 hub or another machine.  
If descriptor size stays `0`, the device is likely faulty.
