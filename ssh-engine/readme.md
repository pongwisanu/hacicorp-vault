# Create cert for SSH

```<something>``` is your input

## 1. Create secrets engine
> ```vault login <token>```
>
> ```vault secrets enable -path=<secret_engine_name> ssh```
> exp. ```vault secrets enable -path=ssh-sign ssh```

## 2. Create Key pair for CA
> ```mkdir pki```
>
> ```vault write <secret_engine_name>/config/ca generate_signing_key=true```
>
> ```vault read -field=public_key <secret_engine_name>/config/ca > pki/trusted-user-ca-keys.pem```
>

## 3. Copy key to server that want to ssh
> scp or what ever
>
> **in server** <br>
> ```echo "TrustedUserCAKeys /etc/ssh/trusted-user-ca-keys.pem" >> /etc/ssh/sshd_config```
>
> ```systemctl restart sshd``` <br>
> **exit**

## 4. Create role ()
> ```
> vault write <secret_engine_name>/roles/<role_name> - <<EOF
> {
>    "algorithm_signer": "rsa-sha2-256",
>    "allow_user_certificates": true,
>    "allowed_users": "<user>" or "*",
>    "allowed_extensions": "permit-pty,permit-port-forwarding",
>    "default_extensions": {
>        "permit-pty": ""
>    },
>    "key_type": "ca",
>    "default_user": "<user>" or delete if don't want it,
>    "ttl": "30m0s" # delete if don't want it
>    }
> EOF 
> ```

## 5. Create user key pair to sign with role
>
> ```ssh-keygen -t ed25519 -N "" -C "<user>" -f pki/<user>```
>
> ```vault write -field=signed_key <secret_engine_name>/sign/<role_name> \ public_key=@pki/<user>.pub  > pki/<user>-cert.pub```


## 6. try login
> ```ssh -i pki/<user>-cert.pub -i pki/<user> <user>@<ip/domain> ```
