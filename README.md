# azure-p2s-powershell

````
### Azure P2S RootCert
$cert = New-SelfSignedCertificate -Type Custom -KeySpec Signature `
-Subject "CN=P2SRootCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" -KeyUsageProperty Sign -KeyUsage CertSign

### Azure P2S ChildCert
New-SelfSignedCertificate -Type Custom -DnsName P2SChildCert -KeySpec Signature `
-Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" `
-Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")


RUN > certmgr.msc


Right-click on "Root" -> Export -> no private key -> base64 der format.
Copy content to P2S gateway.
````

# azure-p2s-bash

````
#!/bin/bash

### Create CA Azure P2S
openssl genrsa -aes256 -out MyAzureVPN.key 2048
openssl req -x509 -sha256 -new -key MyAzureVPN.key -out MyAzureVPN.cer -days 3650 -subj /CN="MyAzureVPN"

### Create cert request - Generate certificate from request and sign it
openssl genrsa -out client1Cert.key 2048
openssl req -new -out client1Cert.req -key client1Cert.key -subj /CN="MyAzureVPN"
openssl x509 -req -sha256 -in client1Cert.req -out client1Cert.cer -CAkey MyAzureVPN.key -CA MyAzureVPN.cer -days 1800 -CAcreateserial -CAserial serial
openssl pkcs12 -export -out client1Cert.pfx -inkey client1Cert.key -in client1Cert.cer -certfile MyAzureVPN.cer


echo "ROOTCA = MyAzureVPN.cer ClientCert = client1Cert.pfx"

exit 0
````
