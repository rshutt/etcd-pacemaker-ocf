# etcd-pacemaker-ocf
This is what I use in my home 2-node physical kubernetes cluster.  It monitors etcd and handles start/stop and migration.  It's far from OCF compliant, but here it is..

## Usage

Just drop `etcd` into `/usr/lib/ocf/resources/heartbeat/etcd`.  Of course, do the obligatory chmod/chown/restorecon thing to ensure that the DAC/MAC is in order for your system.

All of the args common to `/etc/etcd/etcd.conf` are available as OCF env vars (e.g. `OCF_RESKEY_etcd_advertise_client_urls_default`) and are settable via resource options.

A resource definition that uses the defaults that worked for me are as such:

```
pcs resource create etcd ocf:heartbeat:etcd op monitor interval=30s \
    timeout=20s start interval=0s timeout=20s stop interval=0s timeout=20s
```

Furthermore, I created this to be a single node etcd, but given that I had GFS2 already set up to use as PVCs for pods (local-storage... meh... need a provisioner), I went ahead and leveraged that.  Since I was already into `clvmd` with GFS2, HA LVM wasn't an option.  I also manage the VIP for both etcd and the api.  I saw no reason to go active/active despite the fact that both nodes run an apiserver and may be master for scheduler and controller manager.  Furthermore, from the SDN API calls could very well hit the other node due to iptables.

### TODOs

1. Make this script a proper OCF resource with required test harnesses and such
1. Consider active/passive scripting to prevent the controller manager from living on server01 while etcd is on server02?  Does this actually matter?  Sure, the 1GbE network will add latency to etcd which is bad, but is it that bad?  If this were a production system, I'd care... but then again who would deploy a 2 node production system when 3 nodes is clearly far more resilient with 2 etcds to achieve quorum.
1. I was bored, should i have done this or just run peg legged with some failover process?  Probably :P




