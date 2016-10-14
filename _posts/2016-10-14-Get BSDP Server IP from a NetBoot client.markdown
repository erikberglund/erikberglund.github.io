---
title:  "Get BSDP Server IP from a NetBoot client"
date:   2016-08-14 16:00:00
description: Acessing information about the server a NetBoot client is booted from.
---

A question on the [MacAdmins Slack](https://macadmins.slack.com/archives/imagr/p1476426586000654) got me to investigate how a NetBoot client can get the IP of the BSDP server it's booted from.

I could not find any documented command to query the server IP, but the client should have this information stored somewhere as the BSDP ACK[SELECT] packet returned from the BSDP server should be cached on the system.

From the [Boot Server Discovery Protocol Documentation](http://opensource.apple.com//source/bootp/bootp-314.50.1/Documentation/BSDP.doc) Page 5:

> <cite> [The BSDP server] replies with an ACK[SELECT] that contains a path to the boot file to be downloaded using TFTP plus other options required to boot the selected boot image.  Note: Client firmware must save this ACK[SELECT] in a location accessible by the loaded operating system. </cite>

# NetBoot Variables

The only accessable NetBoot variables I could find were from looking in **/etc/rc.netboot**.

```console
ipconfig netbootoption shadow_mount_path
ipconfig netbootoption shadow_file_path
ipconfig netbootoption machine_name
```

The option **netbootoption** is not included in the [man page](x-man-page://8/ipconfig) or usage output for ipconfig, but running it without any options returns a usage message:

```console
$ ipconfig netbootoption
usage: ipconfig netbootoption <option> [<vendor option>] | shadow_mount_path | shadow_file_path | machine_name
```

There is a **\<vendor option\>** which should print the value of DHCP vendor options.

# BSDP DHCP Options

On page 35 in the [Boot Server Discovery Protocol Documentation](http://opensource.apple.com//source/bootp/bootp-314.50.1/Documentation/BSDP.doc), it describes the contents of the ACK[SELECT] packet, including it's DHCP options. I also inspected an ACK[SELECT] packet and by combining that with the documentation, these were the options I found:

| Tag                         | Code | Description                                         |
|:----------------------------|:-----|:----------------------------------------------------|
| Root Path                   | 17   | The client's root disk                              |
| Vendor Specific Information | 43   | BSDP options: BSDP Message Type, BSDP Selected Boot Image ID, Other boot image specific options |
| Option Overload             | 52   | optional; value 1, 2 or 3: see RFC 2132 section 9.3 |
| Message Type                | 53   | 5=ACK                                               |
| Server Identifier           | 54   | IP address of this DHCP server                      |
| Vendor Class Identifier     | 60   | value AAPLBSDPC                                     |
| TFTP server name            | 66   | optional; server’s host name if sname is overloaded |
| Bootfile name               | 67   | optional; boot file name if file is overloaded      |

And, when testing these options on my NetBoot client, I got the result I was looking for:

```console
$ ipconfig netbootoption 17
http://10.2.0.10/NetBoot/NetBootSP0/10.12-16A323_Imagr.nbi/NetInstall.dmg

$ ipconfig netbootoption 43
0000  01 01 02 08 04 81 00 09  29 82 0a 4e 65 74 42 6f  ........)..NetBo
0010  6f 74 30 30 33                                    ot003 

$ ipconfig netbootoption 53
0x5

$ ipconfig netbootoption 54
10.2.0.1

$ ipconfig netbootoption 60
AAPLBSDPC

$ ipconfig netbootoption 66


$ ipconfig netbootoption 67

```

# BSDP Server IP

To get the BSDP server IP from a NetBoot client, extract the IP from root path:

```console
ipconfig netbootoption 17 | awk -F'/' '{ print $3 }'
10.2.0.10
```

# Alternative Method

Because the client is booted from a remote disk image, you can also get the path from [hdiutil](x-man-page://1/hdiutil):

```bash
# Get device id for the boot volume ( example: /dev/disk1s1 )
boot_device_id=$( diskutil info / | awk '/Device Node:/ { print $NF }' )

# Get path for the disk image mounted at "${boot_device_id}"
boot_disk_image_path=$( hdiutil info | awk -v device_id="${boot_device_id}" '{ 
                        if ( $1 == "image-path" ) { 
                            disk_image_path=$NF;
                        } else if ( $1 == device_id ) { 
                            print disk_image_path; exit 0 
                        } }' )

# Print the IP for the BSDP Server
awk -F'/' '{ print $3 }' <<< "${boot_disk_image_path}"
10.2.0.10
```