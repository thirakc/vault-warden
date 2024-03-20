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

# Sync remote drive and crontab
Step to install and config rclone as below!
https://linovox.com/how-to-sync-google-drive-on-linux/

## Configuring using SSH Tunnel (when give the authorize to rclone)
Linux and MacOS users can utilize SSH Tunnel to redirect the headless box port 53682 to local machine by using the following command:
```
ssh -L localhost:53682:localhost:53682 username@remote_server
```
Then on the headless box run rclone config and answer Y to the Use web browser to automatically authenticate? question.
```
...
Remote config
Use web browser to automatically authenticate rclone with remote?
 * Say Y if the machine running rclone has a web browser you can use
 * Say N if running rclone on a (remote) machine without web browser access
If not sure try Y. If Y failed, try N.
y) Yes (default)
n) No
y/n> y
```
Then copy and paste the auth url http://127.0.0.1:53682/auth?state=xxxxxxxxxxxx to the browser on your local machine, complete the auth and it is done.

## after configure rclone, next create script
```
vi ~/.config/rclonescipt.sh
```

## add command in rclonescript.sh
```
#!/bin/bash
rclone sync /home/rakberrypi/apps/vault-warden/vw-data ggdrive:Personal/db/vault-warden/vw-data --create-empty-src-dirs
```

## change permission
```
chmod +x ~/.config/rclonescipt.sh
```

## open crontab
```
crontab -e
```

## add cron expression
```
0 2 * * * ~/.config/rclonescript.sh
```

## after cron work, check log is that crontab doing well
```
cat /var/log/syslog | grep CRON
```

## ref
https://github.com/dani-garcia/vaultwarden
https://rclone.org/remote_setup/