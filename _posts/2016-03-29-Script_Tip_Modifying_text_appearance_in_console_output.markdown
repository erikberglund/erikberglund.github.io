---
title:  "Script Tip: Modifying text appearance in console output"
description: Adding color and attributes to text in the terminal
date: 2016-03-29 20:00:00
---

When writing user-facing scripts you sometimes want to emphasize a word or a line in the text printed to the console to make it easier to read (or harder to miss). 

Common use cases include changing the text <font color="red">c</font><font color="green">o</font><font color="cyan">l</font><font color="blue">o</font><font color="magenta">r</font> or setting attributes like **bold** or <u>underlined</u>.

This is made possible through the ISO/IEC 6429 SGR (Select Graphic Rendition) escape sequences ([^1]) (commonly referred to as the [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code#CSI_codes)).

When an SGR escape sequence is sent, the terminal application interprets the code and modifies the appearance of all subsequent characters printed to match the command until a new escape sequence is sent that either updates or clears the current modifications.

# tput	

There are many different terminal control languages. Browse the following directory to see all language files ([_terminfo_](x-man-page://5/terminfo)) recognized by your system: 

```console
/usr/share/terminfo
```

Because of the lack of standardization there was a need to create a tool that could translate what you intended the terminal to display to the correct escape code for the current terminal.

[tput](x-man-page://1/tput) handles that translation for most of the escape sequences needed in a portable way.

This is the recommended way of passing escape sequences to the terminal (instead of hardcoding the  values for a specific terminal type).

# My setup

In BASH, I use a function to set up variables for the escape sequences I'm going to use in the script. Then I use the variable name (instead of the tput command or hardcoded escape sequence) directly inline with the text I want to output.

By referencing the escape sequences by their variable names, it's easier to understand what will be printed to the screen, but also easier (and faster) to write correct code.

Here is a link to my reference function containing the variables I often use: [SGR variables](https://github.com/erikberglund/Scripts/blob/master/Functions/sgr_escape_sequence_variables/function_sgr_escape_sequence_variables.function)

# Color (Foreground)

When adding this to a script, I create a new function and add the ones I am going to use instead of copying and pasting all escape sequences (to avoid unused variables).

This function is setting up the variables for the foreground color escape sequences I'll need:

```
sgr_variables() {
  # Foreground text colors
  red=$(tput setaf 1) # Red
  grn=$(tput setaf 2) # Green
  blu=$(tput setaf 4) # Blue
  def=$'\e[39m'       # Default foreground color
  
  # Deactivate ALL sgr attributes and colors.
  clr=$(tput sgr0)
}
```
The variables can then be used inline with the text to be printed:

```bash
# Call the function ONCE to set up the variables before using them.
sgr_variables
printf "%s\n" "${red}Red${def},${grn}Green${def},${blu}Blue${clr}"
```
![sgr_color]({{ site.url }}/assets/screenshots/sgr_color.png)

# Color (Background)

Using the example from above, but this time change the text background color instead:

_I have updated the **sgr_variables** function to include the new variables._

```bash
# Call the function ONCE to set up the variables before using them.
sgr_variables
printf "%s\n" "${bgred}Red${bgdef},${bggrn}Green${bgdef},${bgblu}Blue${clr}"
```
![sgr_bgcolor]({{ site.url }}/assets/screenshots/sgr_bgcolor.png)

# Attributes

The following attributes can also be sent as escape sequences to modify text appearance:

* Bold
* Dim
* Underscore
* Blink
* Reverse (Swap foreground and background color)
* Hidden

<br>
Using the first example, but also adding some text attributes:

   1. Add **Bold** to <font color="red">Red</font>
   2. Add **Dim** to <font color="green">Green</font> (bold is still active)
   3. Remove Bold and Dim by using _${nobld}_
   4. Add **Underline** to <font color="blue">Blue</font>

<br>
_I have updated the **sgr_variables** function to include the new variables._

```bash
# Call the function ONCE to set up the variables before using them.
sgr_variables
printf "%s\n" \
"${red}${bld}Red${def},${grn}${dim}Green${nobld}${def},${blu}${uln}Blue${clr}"
```
![sgr_attributes]({{ site.url }}/assets/screenshots/sgr_attributes.png)

# Example Use Case

**Colorized Status**  
This is an example that will print three status messages in different colors:

```bash
printf "%-15s\t%s\n" "StatusOk:"   "[${grn}${bld}OK${clr}]"
printf "%-15s\t%s\n" "StatusWarn:"   "[${yel}${bld}WARN${clr}]"
printf "%-15s\t%s\n" "StatusError:" "[${red}${bld}ERROR${clr}]"
```
![sgr_status]({{ site.url }}/assets/screenshots/sgr_status.png)

# Final Notes

The escape sequence variables are not tied to [printf](x-man-page://1/printf), they can just as easily be used with [echo](x-man-page://1/echo):

```bash
# Call the function ONCE to set up the variables before using them.
sgr_variables
echo "${red}Red${def},${grn}Green${def},${blu}Blue${clr}"
```
![sgr_color]({{ site.url }}/assets/screenshots/sgr_color.png)

To print your current terminal type, use this command:

```console
echo $TERM
```

To print your current terminal [_terminfo_](x-man-page://5/terminfo) file, use this command:

```console
infocmp -L -1
```

___
[^1]: Section **8.3.117** of the [Standard ECMA-48 (Control Functions for Coded Character Sets)](http://www.ecma-international.org/publications/standards/Ecma-048.htm)