# Description

Stay online.  Use ifupdown configuration.


# Features

- Continually polices online status.
- Checks status by observing traffic.
- Sends probes only when there is no other traffic.  


# Dependencies

- iproute2
- ifupdown
- tcpdump
- wireless-tools
- iputils-ping


# Assumptions

- All desirable network configurations are specified in Debian's ifupdown
system.  (/etc/network/interfaces and /etc/network/interfaces.d/*)
- Each desirable wired interface name is the name of an appropriate
specification.
- Each desirable wireless network name is the name of an appropriate
specification.
- Only ipv4 is in use.


# Issues

## dhclient pile up

### pattern

`ifup` obtains and displays an IP address, but then shows a message that
`wpa_supplicant` failed to start, and then fails.  `dhclient` is left
running.  `ifdown` says the interface is not configured, and refuses to act.
Further `ifup` invocations create further instances of dhclient.  

The dhclient instances may interact poorly.  I think it was causing me grief
around address refresh time, once.

### analysis

`wpa_supplicant` failing to start seems like a separate problem.  Even if it
fails, `dhclient` shouldn't pile up, and this issue could presumably happen
even if `ifup` failed for some other reason.

`dhclient` shouldn't be running after `ifup` failure, because `ifup` should
have made sure it exited.  In order to handle the case when `dhclient` is
left running after `ifup` failure, it is possible to just kill all instances
of `dhclient`, but that would also kill any `dhclient` instance running an
interface not controlled by this script.  No such instances are generally
anticipated, so this script contains code to just kill off any `dhclient`
intances.  That code may or may not be active; research is ongoing.

### plan

Observe that this still happens.

Read up on `ifupdown` and `dhclient`, and make sure `ifup` should kill
`dhclient` after a failure.  Either way, creating *new* instances of
`dhclient` seems distinctly incorrect.

I've started to favor dhcpcd for unrelated reasons (better capacity to retain
an old address when refresh fails), so research may stall.
