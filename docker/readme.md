# Install with docker

## 1. Create required directory
> mkdir config data logs tls

## 2. Create key pair for #Vault#
> cd tls

> openssl genrsa -out ca.key 2048
> 
> openssl req -new -key ca.key -subj "/CN=My CA" -out ca.csr
> 
> openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
>
> openssl genrsa -out tls.key 2048
>
> openssl req -new -key tls.key -subj "/CN=myvault" -out tls.csr
>
> echo subjectAltName = DNS:myvault,IP:127.0.0.1 >> server1.cnf;
>
> echo extendedKeyUsage = serverAuth >> server1.cnf
>
> openssl x509 -req -in tls.csr -CAkey ca.key -CA ca.crt -out tls.crt -CAcreateserial -extfile server1.cnf

## 3. Config Vault server 
