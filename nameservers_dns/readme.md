# Nameservers

## What is it good for?

The main purpose of this script is to ensure that nameservers in resolv.conf matches those of the host interface. 
 Can be used to fix DNS mismatch that may occur when connected to a VPN on host system.


## Requirements

This script is written in bash, and uses built-in functionality. A compatible shell is needed for the script to function. 
 Superuser privileges. This script must be able to operate on files in (possibly) protected folders, and restart services. 
 Currently the script is written to be used with systemd. It may, or may not, work with other init systems. 
 If systemd is not used the script will assume that it may run Windows executables, namely the host system powershell. 
 

## Usage

Disclaimer: This is written for a system with systemd and bash. It may not be current, and it may not reflect your system.

Exercise caution when using _random_ scripts. Make sure you understand what the script does before you run it. 
 ***Especially*** when the script requires superuser privileges - **like _this_ script**.

Make sure your /etc/wsl.conf includes the following lines:

```
[interop]
enabled = true
appendWindowsPath = true
```

`appendWindowsPath = true ` may not be strictly necessary, but could be useful.

If you do not have a wsl.conf in the /etc directory, create one.

Type in a terminal window: 
 `sudo $SHELL /replace/with/path/to/script/wsl_fix_dns.sh`

**Or** if in the same directory as the script, type in terminal window: 
 `sudo $SHELL wsl_fix_dns.sh`


## A somewhat abstracted view of what the script is supposed to do. 

* Check for privileges, This script is likely to need elevated privileges, as such, it will exit if run without.

* Do a naive check for init system, more specifically systemd.
  - If found, check if WSL interoperability is enabled.
    - If interoperability is enabled, proceed to next step.
    - If not enabled, ask if user want the script to try and enable it.
      - If yes, create the file /usr/lib/binfmt.d/WSLInterop.conf and redirect config contents into the file.
      - If no, exit.

* Ask to continue (assuming no).
  - If no, exit.
  - If yes, continue.

* Generate resolv.conf file
  - Call host system powershell with -command switch to get a temporary shell, get all DNS addresses and select those associated with WSL interface. 
     Pipe result from powershell command to awk in order to create structure of the resolv.conf file. 
     Pipe from awk to tr to remove carriage return characters redirect the result in to resolv.conf file.

* Check default interface in WSL with ip route.

* Check if there is a carrier signal on interface.
  - If no, exit.
  - If yes, attempt to get WSLs current nameserver by querying cloudflare. Read resolv.conf file and extract nameserver addresses.
    - print addresses from resolv.conf and compare to address queried from cloudflare.
      - If match is found, print success and matched DNS server, then exit.
      - If no match is found, exit

