## Tutorial to Self-host Mutiny Wallet

The guide assumes you have a public IP and a valid FQDN (domain name). The reason for the public IP is that it uses wireguard as the VPN to connect from your phone to your wallet.  The reason for the FQDN is iOS won't let you grant camera permissions to a website unless it has a valid TLS cert.  I am not sure about android.  Camera permissions are required so you can use QR codes.

If you don't want to use a public IP you could use tailscale instead of wireguard but that is not covered here.

This guide also assumes you want to use letencrypt (a free service) for the TLS cert.

## WARNING 
Mutiny stores your channel state in your browser cache as well as in a postgres database on your server.  If you lose the state in one of the two locations it will resync, for example if you lose your browser
cache the next time you access mutiny it will sync from the postgres server.  The opposite is also true.  One issue with the self hosted mutiny is there isn't an easy way to handle the postgres backup, this is because
every time you do a lightning transaction the channel state is updated, and there isn't a way that I know of to trigger a postgres backup on each update to the database.  If you accidently restore a previous version
of the channel state you will be in a bad place and possibly (almost certinaly, lose funds).

### update the machine (if you get an error about a lock just wait a few minutes and try again)
```
sudo apt update
sudo apt upgrade
```

### install docker following the two setps from this link
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

```
sudo usermod -aG docker $USER
```

logout and log back in in order for usermod to take effect

# install git and clone mutiny-deploy repo  
```
sudo apt install git
git clone https://github.com/MutinyWallet/mutiny-deploy.git
```

### start mutiny containers
```
cd mutiny-deploy
docker compose up -d
```

### install and configure nginx
```
sudo apt install nginx
sudo nano /etc/nginx/sites-available/mutin
```

Use the following nginx configuration:

```
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}   

server {
    listen 80;
    server_name <domain name>;

    location / {
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://localhost:14499;
    }
}
```

```
sudo ln -s /etc/nginx/sites-available/mutiny /etc/nginx/sites-enabled/mutiny
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl enable --now nginx
```

### Install and configure wireguard

```
sudo apt install wireguard

umask 077
wg genkey > privatekey-server ;
wg pubkey < privatekey-server > publickey-server ; 
wg genkey | tee privatekey-server | wg pubkey > publickey-server

wg genkey > privatekey-client ;
wg pubkey < privatekey-client > publickey-client ; 
wg genkey | tee privatekey-client | wg pubkey > publickey-client
```

Find your public and private IPs and save values for later

```
curl ifconfig.co (find your own ip for the client config)
ip addr show ens3 ( find out your local private ip, ens3 should be valid for lunacode, eth0 for other VPSs, if not sure just run ip addr to list all)
```

```
sudo nano /etc/wireguard/mutiny.conf
```

Enter the following config:

```
[Interface]
PrivateKey = privatekey-server
Address = 10.0.0.5/32
ListenPort = 33333

[Peer]
PublicKey = publickey-client
AllowedIPs = 10.0.0.10/32
```

```
sudo nano ~/wg-client.conf
```

Enter the following config:

```
[Interface]
PrivateKey = privatekey-client
Address = 10.0.0.10/32

[Peer]
PublicKey = publickey-server
Endpoint = <public ip of server>:33333
AllowedIPs = <public ip of server>/32
```

Bring up the wireguard interface:

```
sudo wg-quick up mutiny
```

### create QR code with client settings
sudo apt install qrencode
sudo nano wg-client.conf
qrencode -t ansiutf8 < wg-client.conf



### Test it out

Install wireguard app on your phone and scan the qr code.
Browse to https://domainname


### troubleshooting
- see if you can curl https://domainname from your VM

