# Install with docker

## 1. Create required directory
> ```mkdir config data logs tls```

## 2. Create key pair for **Vault**

> ```cd tls```
> 
> ```openssl genrsa -out ca.key 2048```
> 
> ```openssl req -new -key ca.key -subj "/CN=My CA" -out ca.csr```
> 
> ```openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt```
> 
> ```openssl genrsa -out tls.key 2048```
> 
> ```openssl req -new -key tls.key -subj "/CN=myvault" -out tls.csr```
> 
> ```echo subjectAltName = DNS:myvault,IP:127.0.0.1 >> server1.cnf```
> 
> ```echo extendedKeyUsage = serverAuth >> server1.cnf```
> 
> ```openssl x509 -req -in tls.csr -CAkey ca.key -CA ca.crt -out tls.crt -CAcreateserial -extfile server1.cnf```

## 3. Config **Vault** server 

> ```cd ..```
> 
> ```
> cat <<EOF | tee config/vault.hcl 
> storage "raft" { 
>   path    = "/vault/data" 
>   node_id = "node1" 
> } 
> listener "tcp" {
>  address       = "0.0.0.0:8200" 
>  tls_cert_file = "/vault/tls/tls.crt" 
>  tls_key_file  = "/vault/tls/tls.key" 
> } 
> disable_mlock = true
> api_addr = "http://127.0.0.1:8200"
> cluster_addr = "https://127.0.0.1:8201" 
> ui = true 
> EOF
> ```

## 4. Create container

> ```docker container run --cap-add=IPC_LOCK -d -p 127.0.0.1:8200:8200 -v $(pwd)/config:/vault/config -v $(pwd)/data:/vault/data -v $(pwd)/logs:/vault/logs  -v $(pwd)/tls:/vault/tls --rm --name  myvault vault server ``` <br>
> or  <br>
> use docker compose file <br>
>
> **Warnning** <br>
> might get some error with file permission

## 5. Install **Vault** in client to connect to container

> <https://developer.hashicorp.com/terraform/install>

## 6. Unseal **Vault**

> ```export VAULT_ADDR=https://127.0.0.1:8200```
> ```export VAULT_CACERT=./tls/ca.crt```
>
> ```vault operator init```
>
> **after your finish init you need to keep your key and token**
>
> ```vault operator unseal <key>```
> 
