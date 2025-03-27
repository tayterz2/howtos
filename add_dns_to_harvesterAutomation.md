# Add DNS Zones/Records to Harvester Automation Builds

## Intro
We're talking about the CoreDNS app on github.com/rancherfederal/harvesterAutomation here...

In order to edit Harvester Automation DNS records (resolvable both internal and external to the cluster), follow these steps:


## Steps
1. Sign in to the Cluster, make sure you have a working kube config logged in:
   ```
   harvester1:~ # kubectl get po -n kube-system
   NAME                                                     READY   STATUS      RESTARTS      AGE
   cloud-controller-manager-harvester1                      1/1     Running     0             37h
   ...
   helm-install-rke2-canal-gpjtv                            0/1     Completed   0             37h
   helm-install-rke2-coredns-pnmpv                          0/1     Completed   0             37h
   ...
   ```

1. Find and edit the custom-zonefile-configmap in the kube-system zone:
   ```harvester1:~ # kubectl edit configmap -n kube-system custom-zonefile-configmap
   # Please edit the object below. Lines beginning with a '#' will be ignored,
   # and an empty file will abort the edit. If an error occurs while saving this file will be
   # reopened with the relevant failures.
   #
   apiVersion: v1
   data:
     local.io.zone: |
       $ORIGIN local.io.
       $TTL 3600
       @   IN  SOA ns1.local.io. admin.local.io. (
                 2024100201 ; serial
                 7200       ; refresh (2 hours)
                 3600       ; retry (1 hour)
                 1209600    ; expire (2 weeks)
                 3600       ; minimum (1 hour)
             )
           IN  NS   ns1.local.io.
       ns1        IN A  192.168.4.210
       harvester  IN A  192.168.4.210
       rancher    IN A  192.168.4.210
       registry   IN A  192.168.4.210
       fileserver IN A  192.168.4.210
       keycloak   IN A  192.168.4.210
       apps       IN A  192.168.4.210
       git        IN A  192.168.4.210
   kind: ConfigMap
   ```
   Note the "local.io.zone" section in this example.  It was created by this automation's "Domain Name" being set to "local.io" but can be changed at this step, OR have additional zones added to it.  Since we probably want to edit this file a few times, we'll save the configuration to a file that we'll recreate the configmap from.

1. Save the configmap to a file:
   ```kubectl get configmap -n kube-system custom-zonefile-configmap -o yaml > /usr/local/zone.configmap```

1. Edit the file how you see fit.  For this example, we'll create several zones and child zones:
   ```
   *.sf.example.com
   *.dea.sf.example.com
   *.red.dea.sf.example.com
   *.yellow.dea.sf.example.com
   *.blue.dea.sf.example.com
   *.green.dea.sf.example.com
   ```

1. Our new zonefile looks like this:
   ```
   cat /usr/local/zone.configmap
   data:
     sf.example.com.zone: |
       $ORIGIN sf.example.com.
       $TTL 3600
       @   IN  SOA ns1.sf.example.com. admin.sf.example.com. (
                 2024100201 ; serial
                 7200       ; refresh (2 hours)
                 3600       ; retry (1 hour)
                 1209600    ; expire (2 weeks)
                 3600       ; minimum (1 hour)
             )
                  IN NS ns1.sf.example.com.
       ns1        IN A  192.168.1.210
       host1      IN A  192.168.1.211
       host2      IN A  192.168.1.212
     dea.sf.example.com.zone: |
       $ORIGIN dea.sf.example.com.
       $TTL 3600
       @   IN  SOA ns1.dea.sf.example.com. admin.dea.sf.example.com. (
                 2024100201 ; serial
                 7200       ; refresh (2 hours)
                 3600       ; retry (1 hour)
                 1209600    ; expire (2 weeks)
                 3600       ; minimum (1 hour)
             )
                  IN NS ns1.dea.sf.example.com.
       ns1        IN A  192.168.2.210
       host1      IN A  192.168.2.211
       host2      IN A  192.168.2.212
     red.dea.sf.example.com.zone: |
       $ORIGIN red.dea.sf.example.com.
       $TTL 3600
       @   IN  SOA ns1.red.dea.sf.example.com. admin.red.dea.sf.example.com. (
                 2024100201 ; serial
                 7200       ; refresh (2 hours)
                 3600       ; retry (1 hour)
                 1209600    ; expire (2 weeks)
                 3600       ; minimum (1 hour)
             )
                  IN NS ns1.red.dea.sf.example.com.
       ns1        IN A  192.168.3.210
       host1      IN A  192.168.3.211
       host2      IN A  192.168.3.212
     yellow.dea.sf.example.com.zone: |
       $ORIGIN yellow.dea.sf.example.com.
       $TTL 3600
       @   IN  SOA ns1.yellow.dea.sf.example.com. admin.yellow.dea.sf.example.com. (
                 2024100201 ; serial
                 7200       ; refresh (2 hours)
                 3600       ; retry (1 hour)
                 1209600    ; expire (2 weeks)
                 3600       ; minimum (1 hour)
             )
                  IN NS ns1.yellow.dea.sf.example.com.
       ns1        IN A  192.168.4.210
       host1      IN A  192.168.4.211
       host2      IN A  192.168.4.212
     green.dea.sf.example.com.zone: |
       $ORIGIN green.dea.sf.example.com.
       $TTL 3600
       @   IN  SOA ns1.green.dea.sf.example.com. admin.green.dea.sf.example.com. (
                 2024100201 ; serial
                 7200       ; refresh (2 hours)
                 3600       ; retry (1 hour)
                 1209600    ; expire (2 weeks)
                 3600       ; minimum (1 hour)
             )
                  IN NS ns1.green.dea.sf.example.com.
       ns1        IN A  192.168.5.210
       host1      IN A  192.168.5.211
       host2      IN A  192.168.5.212
     blue.dea.sf.example.com.zone: |
       $ORIGIN blue.dea.sf.example.com.
       $TTL 3600
       @   IN  SOA ns1.blue.dea.sf.example.com. admin.blue.dea.sf.example.com. (
                 2024100201 ; serial
                 7200       ; refresh (2 hours)
                 3600       ; retry (1 hour)
                 1209600    ; expire (2 weeks)
                 3600       ; minimum (1 hour)
             )
                  IN NS ns1.blue.dea.sf.example.com.
       ns1        IN A  192.168.6.210
       host1      IN A  192.168.6.211
       host2      IN A  192.168.6.212
     6.168.192.in-addr.arpa.zone: |
       $ORIGIN 6.168.192.in-addr.arpa
       $TTL 3600
       @   IN  SOA ns1.blue.dea.sf.example.com. admin.blue.dea.sf.example.com. (
                 2024100201 ; serial
                 7200       ; refresh (2 hours)
                 3600       ; retry (1 hour)
                 1209600    ; expire (2 weeks)
                 3600       ; minimum (1 hour)
             )
                  IN NS ns1.blue.dea.sf.example.com.
       210        IN PTR  ns1
       211        IN PTR  host1
       212        IN PTR  host2