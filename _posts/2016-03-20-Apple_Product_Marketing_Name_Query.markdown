---
title:  "Apple Product Marketing Name Query"
date:   2016-03-20 11:00:00
description: Get marketing name from serial number or os version.
---

Apple has a web service you can query for an Apple product marketing name.

Use the following base URL: [http://support-sp.apple.com/sp/product](http://support-sp.apple.com/sp/product) and add a query for the product marketing name you're looking for.

# OS X marketing name (10.7+)

Use the query string **edid=_\<os version\>_** to return the marketing name of an OS X Version.

Example using os version: **10.11.4**

```bash
curl -s http://support-sp.apple.com/sp/product?edid=10.11.4
```

_Response:_

```xml
<?xml version="1.0" encoding="utf-8" ?>
<root>
    <name>CPU Name</name>
    <configCode>OS X El Capitan</configCode>
    <locale>en_US</locale>
</root>
```

To get the marketing name of a running os, you can use `sw_vers` to fill the query value:

```bash
curl -s http://support-sp.apple.com/sp/product?edid=$( sw_vers -productVersion )
```

# Hardware marketing name

Use the query string **cc=_\<last x characters of serial number\>_** to return the marketing name for a hardware product.

* **If serial number is 12 characters, use the last 4 characters.**  
* **If serial number is 11 characters, use the last 3 characters.**
<br>

Example using a Mac serial number: **C02PRAR2G8WP**

```bash
curl -s http://support-sp.apple.com/sp/product?cc=G8WP
```

_Response:_

```xml
<?xml version="1.0" encoding="utf-8" ?>
<root>
    <name>CPU Name</name>
    <configCode>MacBook Pro (Retina, 15-inch, Mid 2015)</configCode>
    <locale>en_US</locale>
</root>
```

Example using an Airport Express serial number: **F12HQHPADV2R**

```bash
curl -s http://support-sp.apple.com/sp/product?cc=DV2R
```

_Response:_

```xml
<?xml version="1.0" encoding="utf-8" ?>
<root>
    <name>CPU Name</name>
    <configCode>AirPort Express 802.11n (2nd Generation)</configCode>
    <locale>en_US</locale>
</root>
```

To get the marketing name of a running computer, you can use `ioreg` to fill the query value:

```bash
curl -s http://support-sp.apple.com/sp/product?cc=$( ioreg -c IOPlatformExpertDevice -d 2 | awk -F\" '/IOPlatformSerialNumber/{ sn=$(NF-1); if (length(sn) == 12) count=3; else if (length(sn) == 11) count=2; print substr(sn, length(sn) - count, length(sn))}' )
```

I use awk to print the x last characters depending on the serial number length.

Here is a version of the awk program used by itself on the Airport Express serial number:

```bash
awk '{ if (length($0) == 12) count=3; else if (length($0) == 11) count=2; print substr($0, length($0) - count, length($0))}' <<< "F12HQHPADV2R"
```

_Output:_

```console
DV2R
```


# Cleaning output

If you only want the marketing name as output, you can use the following xpath query to print the _configCode_ node content from the response:

```bash
'/root/configCode/text()'
```

**OS X marketing name example:**

```bash
curl -s http://support-sp.apple.com/sp/product?edid=$( sw_vers -productVersion ) | xpath '/root/configCode/text()' 2>/dev/null
```

_Output:_

```console
OS X El Capitan
```

**Hardware product marketing name example:**

```bash
curl -s http://support-sp.apple.com/sp/product?cc=$( ioreg -c IOPlatformExpertDevice -d 2 | awk -F\" '/IOPlatformSerialNumber/{ sn=$(NF-1); if (length(sn) == 12) count=3; else if (length(sn) == 11) count=2; print substr(sn, length(sn) - count, length(sn))}' ) | xpath '/root/configCode/text()' 2>/dev/null
```

_Output:_

```console
MacBook Pro (Retina, 15-inch, Mid 2015)
```

_NOTE: xpath doesn't add a newline at the end of the output._
