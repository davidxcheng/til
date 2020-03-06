# Capturing network traffic with `netsh`

`netsh` is a CLI tool on Windows that lets you do a lot of network related stuff. It uses "contexts" to scope "stuff" and the context for tracing network traffic is aptly named `trace`.

## How to trace

Use `netsh trace show capturefilterhelp` to get some great examples of how to start a trace with a filter that caputres the packages you are interested in. The instructions are very well written and goes into the details that matter when used in real scenarios:

> Filters need to be explicitly stated when required. If a filter is
        not specified, it is treated as "don't-care".
         e.g. 'netsh trace start capture=yes IPv4.SourceAddress=157.59.136.1'
              This will capture IPv4 packets only from 157.59.136.1, and it
              will also capture packets with non-IPv4 Ethernet Types, since
              the Ethernet.Type filter is not explicitly specified.
         e.g. 'netsh trace start capture=yes IPv4.SourceAddress=157.59.136.1
               Ethernet.Type=IPv4'
              This will capture IPv4 packets only from 157.59.136.1. Packets
              with other Ethernet Types will be discarded since an explicit
              filter has been specified.
              
## Tales from the real world

I was tasked with investigating/verifiying that JWT:s were indeed reused until they came close to expiring. Our service acquired its JWT from `login.microsoftonline.com` and the JWT expired after 1 hour.

A `nslookup` of `login.microsoftonline.com` reveals that it has 5 IP addresses:

```terminal
nslookup login.microsoftonline.com
Server:  XXXXXXXX.XXXXX.se
Address:  XXX.XXX.XXX.XXX

Non-authoritative answer:
Name:    www.tm.a.prd.aadg.akadns.net
Addresses:  40.126.9.5
          20.190.137.97
          40.126.9.65
          40.126.9.67
          40.126.9.7
Aliases:  login.microsoftonline.com
          prda.aadg.msidentity.com
```

To make the tracing easier I added this line to my local `hosts` file (located at `C:\Windows\System32\drivers\etc`):

> 20.190.137.97 login.microsoftonline.com

I then started a trace with: `netsh trace start capture=yes IPv4.SourceAddress=20.190.137.97 Ethernet.Type=IPv4` and when I was done I viewed the trace using `Microsoft Message Analyzed`.

Or at least that is what I wish I did, I was lazy and just started a local trace in said tool and then added the `IPv4.Datagram.DestinationAddress=20.190.137.97` filter. The upside with that is that I could see the log in real time and the down side is that the filter was just a filter for the view. When I saved the file it was 227 MB so it seems to have captured lots of unrelated network packages.
