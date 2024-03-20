# Vault Warden
Alternative implementation of the Bitwarden server API written in Rust and compatible with [upstream Bitwarden clients](https://bitwarden.com/download/)*, perfect for self-hosted deployment where running the official resource-heavy service might not be ideal.

## Run Vault Warden with TLS
```
docker run -d --name vaultwarden \
    -e ROCKET_TLS='{certs="/ssl/bitwarden.crt",key="/ssl/bitwarden.key"}' \
    -v $VW_SSL_DIR:/ssl/ \
    -v $VW_DB_PATH:/data/ \
    -p 443:80 \
    vaultwarden/server:latest
```

# Create self sign cert
## create root ca cert
```
openssl genpkey -algorithm RSA -aes256 -out private-ca.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```
```
openssl req -x509 -new -nodes -sha256 -days 10950 -key private-ca.key -out self-signed-ca-cert.crt
```

## Create vault warden cert:
```
openssl genpkey -algorithm RSA -out bitwarden.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```
```
openssl req -new -key bitwarden.key -out bitwarden.csr
```
```
openssl x509 -req -in bitwarden.csr -CA self-signed-ca-cert.crt -CAkey private-ca.key -CAcreateserial -out bitwarden.crt -days 365 -sha256 -extfile bitwarden.ext
```

## ref
https://github.com/dani-garcia/vaultwarden