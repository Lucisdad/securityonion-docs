BPF
===

| BPF stands for Berkeley Packet Filter:
| https://en.wikipedia.org/wiki/Berkeley_Packet_Filter
| http://biot.com/capstats/bpf.html

Configuration
-------------

Global bpf.conf
~~~~~~~~~~~~~~~

You can specify your BPF in ``/etc/nsm/rules/bpf.conf`` on your master
server and, by default, it will apply to
Snort/Suricata/Bro/netsniff-ng/prads on all interfaces in your entire
deployment. If you have separate sensors reporting to that master
server, they will copy ``/etc/nsm/rules/bpf.conf`` as part of the daily
rule-update cron job (or you can run it manually) which will also
restart Snort/Suricata so that the BPF change will take effect. Bro
automatically monitors ``bpf.conf`` for changes and will update itself
as needed. Other services (such as prads and netsniff-ng) will need to
be restarted manually for the change to take effect.

Granular bpf.conf
~~~~~~~~~~~~~~~~~

Each process on each interface has its own bpf file, but by default the
per-process bpf files are symlinked to the interface bpf and the
interface bpf is then symlinked to the global ``bpf.conf``:

::

    lrwxrwxrwx 1 root  root     8 Jan 13 21:47 bpf-bro.conf -> bpf.conf
    lrwxrwxrwx 1 root  root    23 Jan 13 21:47 bpf.conf -> /etc/nsm/rules/bpf.conf
    lrwxrwxrwx 1 root  root     8 Jan 13 21:47 bpf-ids.conf -> bpf.conf
    lrwxrwxrwx 1 root  root     8 Jan 13 21:47 bpf-pcap.conf -> bpf.conf
    lrwxrwxrwx 1 root  root     8 Jan 13 21:47 bpf-prads.conf -> bpf.conf

If you don't want your sensors to inherit ``bpf.conf`` from the master
server and/or you need to specify a bpf per-interface or per-process,
you can simply replace the default symlink(s) with the desired bpf
file(s) and restart service(s) as necessary. For example, suppose you
want to apply a BPF to NIDS (Snort/Suricata) only:

::

    # Remove the default NIDS BPF symlink
    sudo rm bpf-ids.conf
    # Create a new NIDS BPF file and add your custom BPF
    sudo vi bpf-ids.conf
    # Restart NIDS
    sudo so-nids-restart


BPF Examples
~~~~~~~~~~~~

From Phillip Wang:

Just to contribute, and for others to reference, here are some examples
of what I've got working

::

    #Nothing from src host to dst port
    !(src host xxx.xxx.xxx.xxx && dst port 161) &&

    #Nothing from src host to dst host and dst port
    !(src host xxx.xxx.xxx.xxx && dst host xxx.xxx.xxx.xxx && dst port 80) &&

    #Nothing to or from:
    !(host xxx.xxx.xxx.xxx) &&

    #Last entry has no final &&
    !(host xxx.xxx.xxx.xxx)

VLAN
~~~~
From Seth Hall regarding VLAN tags:

::

    (not (host 192.168.53.254 or host 192.168.53.60 or host 192.168.53.69 or host 192.168.53.234)) or (vlan and (not (host 192.168.53.254 or host 192.168.53.60 or host 192.168.53.69 or host 192.168.53.234)))

This amazingly works if you are only using it to restrict the traffic
passing through the filter. The basic template is…

::

    <your filter> and (vlan and <your filter>)

Once the ``vlan`` tag is included in the filter, all subsequent
expressions to the right are shifted by four bytes so you need to
duplicate the filter on both sides of the vlan keyword. There are edge
cases where this will no longer work and probably edge cases where a few
undesired packets will make it though, but it should work in the example
case that you've given.

Also, I'm assuming that any tools you are running will support vlan tags
and no tags simultaneously. Bro 2.0 should work fine at least.

Troubleshooting BPF using tcpdump
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If you need to troubleshoot BPF, you can use ``tcpdump`` as shown in the following articles.

http://taosecurity.blogspot.com/2004/09/understanding-tcpdumps-d-option-have.html

http://taosecurity.blogspot.com/2004/12/understanding-tcpdumps-d-option-part-2.html

http://taosecurity.blogspot.com/2008/12/bpf-for-ip-or-vlan-traffic.html
