# Nameservers

The main purpose of this script is to ensure that nameservers in resolv.conf matches those of the host interface.
It goes about doing this in a few (admittedly ignorant and naive) steps.

Check for privileges - This script is likely to need elevated privileges, as such, it will exit if run without.

Do a naive check for init system - more specifically systemd.
  If found - check if WSL interoperability is enabled.
    If interoperability is enabled - proceed to next step.
    If not enabled - ask if user want the script to try and enable it.
      If yes - create the file /usr/lib/binfmt.d/WSLInterop.conf and redirect config contents into the file.
      If no - exit.

Ask to continue (assuming no).
  If no - exit.
  If yes - continue.

Generate resolv.conf file
  Call host system powershell with -command switch to get a temporary shell, get all DNS addresses and select those associated with WSL interface.
  Pipe result from powershell command to awk in order to create structure of the resolv.conf file.
  Pipe from awk to tr to remove carriage return characters redirect the result in to resolv.conf file.

Check default interface in WSL with ip route.

Check if there is a carrier signal on interface.
  If no - exit.
  If yes - attempt to get WSLs current nameserver by querying cloudflare. Read resolv.conf file and extract nameserver addresses.
    print addresses from resolv.conf and compare to address queried from cloudflare.
  If match is found - print success and matched DNS server, then exit.
  If no match is found - exit

