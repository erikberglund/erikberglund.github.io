---
title:  "Script Tip: Split line into variables"
description: Assigning multiple variables with values from a single input line
date: 2016-03-28 20:00:00
---

This is a BASH script tip for improving multi-variable assignments from a single input line.

# Example

On OS X, **/usr/bin/sw_vers -productVersion** will output the current computer's OS version:

```
$ /usr/bin/sw_vers -productVersion
10.11.4
```
When assign the components from the version string into separate variables I often see this:

```bash
major=$( /usr/bin/sw_vers -productVersion | awk -F'.' '{print $1}' )
minor=$( /usr/bin/sw_vers -productVersion | awk -F'.' '{print $2}' )
patch=$( /usr/bin/sw_vers -productVersion | awk -F'.' '{print $3}' )
```

This works fine, but calls **sw_vers** and **awk** three times to parse the same string into variables.

We could remove the repeated calls to **sw_vers** by assigning it's output to a variable and then use that variable as input to the awk commands:

```bash
version=$( /usr/bin/sw_vers -productVersion )
major=$( awk -F'.' '{print $1}' <<< "${version}" )
minor=$( awk -F'.' '{print $2}' <<< "${version}" )
patch=$( awk -F'.' '{print $3}' <<< "${version}" )
```

Better, but still executes **awk** three times in order to parse the string and assign the variables.

# read

BASH has a builtin command named **read** designed for this situation.

This is what the same variable assignment looks like when using **read**:

```bash
IFS='.' read -r major minor patch < <( /usr/bin/sw_vers -productVersion )
```

**Command explained:**

I start by defining the separator I want to use to split the input string:

```
IFS='.'
```

Then, I call the **read** command followed by the variable names I want to assign each value to: 

```
read -r major minor patch
```

I use process substitution to pass the output from **sw_vers** to the input for the **read** command:

```bash
< <( /usr/bin/sw_vers -productVersion )
```

When executed, **read** will use the separator to split the input line and assign each value one by one to each of the variable names defined.

# Matching number of variables and values

If I define <u>fewer</u> variable names than there are available values, **read** will assign one value to each variable until there are no more variables to assign. After that it will just add the remainder of the line to the last variable (including separators).

Consider this from the above example, but here I've defined <u>fewer</u> variable names than values:

```bash
IFS='.' read -r major minor < <( /usr/bin/sw_vers -productVersion )

# Now the variables hold the following values:
(major=10)
(minor=11.4)
```

If I define <u>more</u> variable names than there are values, **read** will just fill each variable until there are no more values to assign and then assign empty values to the remaining variables:

```bash
IFS='.' read -r major minor patch beta < <( /usr/bin/sw_vers -productVersion )

# Now the variables hold the following values:
(major=10)
(minor=11)
(patch=4)
(beta=)
```

# Only assign a subset of values from the line

If you don't want want the entire line assigned to variables you should parse the line before passing it to **read**. Then you can control what variables and values will be associated:

```bash
read -r minor patch < <( /usr/bin/sw_vers -productVersion | awk -F'.' '{print $2"\ "$3}' )

# Now the variables hold the following values:
(minor=11)
(patch=4)
```

# man page

Because **read** is a BASH [builtin](http://tldp.org/LDP/abs/html/internal.html) command it doesn't have it's own man page.

You can use this command to jump to the **read** section in the _bash_ man page:

```bash
man bash | less -p "^       read "
```

Here are more links to Apple's _bash_ man page (search for "**read [**" to find the correct section.):
  
* [Open in Terminal.app](x-man-page://1/bash)
* [Open in Browser](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/bash.1.html)  
<br>

# Final Notes

By saving computer resources (removing unnecessary subshells and pipes), we've reduced the execution time of the variable assignment in our example by **55%** (real time).

This is the time each example took:

* **sw_vers and awk each called three times:**

      real	0m0.031s  
      user	0m0.016s  
       sys	0m0.016s

* **sw_vers output to variable and awk called three times:**

      real	0m0.022s
      user	0m0.009s
       sys	0m0.011s

* **read:**

      real	0m0.014s
      user	0m0.002s
       sys	0m0.003s