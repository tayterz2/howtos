# Replace Harvester Automation Builds' Certificates

## Intro

We'll be replacing user-facing (aka browser-visible) certificates in the Harvester and Rancher systems, such that a user's browser will believe all CAs are consistent with the browser's view of the world (trusts these certificates).

Background info here:<br>
[Rancher SSL Certificate Replacement](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/resources/update-rancher-certificate)
<br>
[Harvester SSL Certificate Setting](https://docs.harvesterhci.io/v1.4/advanced/index#ssl-certificates)

## Steps to Update Harvester SSL

1. Utilize either the GUI or the commandline to update the Certificate Authority (CA) setting section of Harvester, to include the new Certificate's Authority chain (Root CA and all Intermediate CA's, if applicable).  Harvester calls this the 'additional-ca' setting.  It is recommended that the current setting be _appended_ to, rather than replaced:
   ```
   # kubectl edit setting additional-ca

   # Please edit the object below. Lines beginning with a '#' will be ignored,
   # and an empty file will abort the edit. If an error occurs while saving this file will be
   # reopened with the relevant failures.
   #
   apiVersion: harvesterhci.io/v1beta1
   kind: Setting
   metadata:
     annotations:
       harvesterhci.io/hash: a136b67ba0c4cd81afe17c0160864ace95a793c452ccafbdf77825cd
     creationTimestamp: "2025-03-31T19:29:25Z"
     generation: 3
     name: additional-ca
     resourceVersion: "6531"
     uid: 78ddf812-7efd-4003-97f4-49e8160d0249
   status:
     conditions:
     - lastUpdateTime: "2025-03-31T19:29:30Z"
       status: "True"
       type: configured
   value: |-
     -----BEGIN CERTIFICATE-----
     MIIF1DCCA7ygAwIBAgIQd8wRr8GxesUr2Mz7NFXpuDANBgkqhkiG9w0BAQsFADCB
     gzELMAkGA1UEBhMCVVMxETAPBgNVBAgTCFZJUkdJTklBMQ8wDQYDVQQHEwZSRVNU
     T04xDzANBgNVBAoTBkhBVUxFUjETMBEGA1UECxMKSEFVTEVSIERFVjEqMCgGA1UE
     ...
     -----END CERTIFICATE-----
     -----BEGIN CERTIFICATE-----
     <your new pem-encoded certificate #1>
     -----END CERTIFICATE-----
     -----BEGIN CERTIFICATE-----
     <your new pem-encoded certificate #2>
     -----END CERTIFICATE-----
     ```
     On the GUI, you can find the "Settings" section under the "Advanced" menu item on the left of the Harvester main page, after login.

1. Utilize either the GUI or the commandline to update the SSL Certificate section of Harvester, to include the new Certificate's Authority chain (Root CA and all Intermediate CA's, if applicable).  Harvester calls this the 'ssl-certificates' setting.  Unlike the additional-ca setting, this setting will have the original value _replaced_ by the new certificate and key:
   ```
   # kubectl get setting ssl-certificates -ojsonpath='{.value}' | yq -pjson > /usr/local/etc/certs.txt
   # vi /usr/local/etc/certs.txt
   publicCertificate: |-
     -----BEGIN CERTIFICATE-----
     <public cert goes here>
     -----END CERTIFICATE-----
   ca: |-
     -----BEGIN CERTIFICATE-----
     <complete ca cert goes here (may be multipble BEGIN/END blocks)>
     -----END CERTIFICATE-----
   privateKey: |-
     -----BEGIN RSA PRIVATE KEY-----
     <private key file goes here, rsa format>
     -----END RSA PRIVATE KEY-----

   # export JSONCERTS=$(yq -ojson /usr/local/etc/certs.txt) ; kubectl get setting ssl-certificates -o yaml | yq ".value = strenv(JSONCERTS)" | kubectl apply -f -
   setting.harvesterhci.io/ssl-certificates configured
   ```
   Note that this CLI example is a bit convoluted since Harvester expects the YAML object to contain the Certifiate, CA, and Key within a JSON object ".value".  Editing via the "Settings" page (under "Advanced") in the GUI is much more straightforward - you'll be prompted for the three files as uploads from your browser.


## Steps to Update Rancher vCluster SSL

1. Utilize either the GUI or the commandline to update the Certificate Authority (CA) setting section of Harvester, to include the new Certificate's Authority chain (Root CA and all Intermediate CA's, if applicable).  Rancher calls this the 'cacerts' setting.  It is recommended that the current setting be _appended_ to, rather than replaced:
   ```
   # Please edit the object below. Lines beginning with a '#' will be ignored,
   # and an empty file will abort the edit. If an error occurs while saving this file will be
   # reopened with the relevant failures.
   #
   apiVersion: management.cattle.io/v3
   customized: false
   default: ""
   kind: Setting
   metadata:
     annotations:
       kubectl.kubernetes.io/last-applied-configuration: |
         {"apiVersion":"management.cattle.io/v3","kind":"Setting","metadata":{"annotations":{},   "name":"cacerts"},"value":"-----BEGIN CERTIFICATE-----\nMIIF1DC...-----END CERTIFICATE-----"}
     creationTimestamp: "2025-03-31T19:34:03Z"
     generation: 2
     name: cacerts
     resourceVersion: "2788"
     uid: 743c1b6a-c5a5-4fe3-8d80-49aef692f2a0
   source: ""
   value: |-
     -----BEGIN CERTIFICATE-----
     MIIF1DCCA7ygAwIBAgIQd8wRr8GxesUr2Mz7NFXpuDANBgkqhkiG9w0BAQsFADCB
     gzELMAkGA1UEBhMCVVMxETAPBgNVBAgTCFZJUkdJTklBMQ8wDQYDVQQHEwZSRVNU
     T04xDzANBgNVBAoTBkhBVUxFUjETMBEGA1UECxMKSEFVTEVSIERFVjEqMCgGA1UE
     AxMhQ0VSVElGSUNBVEUgQVVUSE9SSVRZIENFUlRJRklDQVRFMB4XDTI1MDMzMTE5
     ...
     -----END CERTIFICATE-----
     -----BEGIN CERTIFICATE-----
     <your new pem-encoded certificate #1>
     -----END CERTIFICATE-----
     -----BEGIN CERTIFICATE-----
     <your new pem-encoded certificate #2>
     -----END CERTIFICATE-----
     ```
     On the GUI, you can find the "Global Settings" (Globe Icon) section at the bottom of the menu on the left of the Rancher main page, after login.

1.  Update the "tls-rancher-ingress-x-cattle-system-x-rancher-vcluster" secret in the "rancher-vcluster" namespace using the same instructions found below for [Updating Ingress Certificate Secrets](#updating-ingress-certificate-secrets).

## Updating Ingress Certificate Secrets

The remaining user-facing SSL Certificates will be contained within Kubernetes Ingress objects throughout the Harvester cluster.  Find them with the "get ingress" kubectl command, followed by -A:
   ```
   # kubectl get ingresses -A -ocustom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,SECRET:.spec.tls
   NAMESPACE          NAME                                         SECRET
   cattle-system      rancher-expose                               <none>
   hauler-system      hauler-fileserver                            [map[hosts:[fileserver.taylor.io]    secretName:fileserver-tls]]
   hauler-system      hauler-registry                              [map[hosts:[registry.taylor.io]    secretName:registry-tls]]
   keycloak-system    keycloak                                     [map[hosts:[keycloak.taylor.io]    secretName:tls-certs]]
   rancher-vcluster   rancher-x-cattle-system-x-rancher-vcluster   [map[hosts:[rancher.taylor.io]    secretName:tls-rancher-ingress-x-cattle-system-x-rancher-vcluster]]
   ```
   Note that your list will differ from this example, depending on what Addons and Apps were installed.  Also note that the custom-columns used in this example divulges what Secret contains each ingress hostnames' Certificates.

For each of the Secrets listed above, do the following (the rancher-expose Ingress has no Certificate attached, Harvester's main SSL Certificate gets updated in [this section](#steps-to-update-harvester-ssl)):

### Steps

This example will be for the hauler-fileserver Ingress above (the fileserver-tls Secret), and assumes these values are in files fileserver.cert, fileserver-ca.cert, and fileserver.key, respectively.

1. Convert the PEM-encoded Certificate and CA, and the RSA-encoded Key to Base64, and save them as environment variables:
   ```
   BASE64CA=$(cat fileserver-ca.cert | base64 -w0)
   BASE64CERT=$(cat fileserver.cert | base64 -w0)
   BASE64KEY=$(cat fileserver.key | base64 -w0)
   ```
   
1. Apply these new values to the three applicable fields of the Secret.  Pay close attention to the namespace, secret name, and .data values in this command:
   ```
   # kubectl get secret --namespace hauler-system fileserver-tls -o yaml | yq '.data.["ca.crt"] = strenv(BASE64CA) | .data.["tls.crt"] = strenv(BASE64CERT) | .data.["tls.key"] = strenv(BASE64KEY)'  | kubectl apply -f -

   Warning: resource secrets/fileserver-tls is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
   secret/fileserver-tls configured
   ```
   The warning shown in this example can safely be ignored, if observed.  The important piece is the last line - the secret was updated ("configured") correctly.

1.  Repeat steps 1 and 2 for each Ingress/Secret that you'd like to update.