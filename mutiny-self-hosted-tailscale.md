# Tutorial to Self-host Mutiny Wallet - Tailscale version

## Goal

To self host an instance of [Mutiny wallet](https://www.mutinywallet.com) and access it securely from your phone over VPN. 

This version of the tutorial assumes you will use tailscale to connect from your phone/computer to your server.  A publicly accesible IP is not required so this version is suitable for self hosting on a raspberry pi or similiar computer.


## WARNING 
Mutiny stores your channel state in your browser cache as well as in a postgres database on your server.  If you lose the state in one of the two locations it will resync, for example if you lose your browser
cache the next time you access mutiny it will sync from the postgres server.  The opposite is also true.  One issue with the self hosted mutiny is there isn't an easy way to handle the postgres backup, this is because
every time you do a lightning transaction the channel state is updated, and there isn't a way that I know of to trigger a postgres backup on each update to the database.  If you accidently restore a previous version
of the channel state you will be in a bad place and possibly (almost certinaly, lose funds).

### Update the machine 
```
sudo apt update
sudo apt upgrade
```

### Install docker following the two steps from this tutorial
[Installing Docker on Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

Add your user to the docker group.
```
sudo usermod -aG docker $USER
```

logout and log back in in order for usermod to take effect

### Install git and clone mutiny-deploy repo  
```
sudo apt install git
git clone https://github.com/MutinyWallet/mutiny-deploy.git
```

### Start mutiny containers
```
cd mutiny-deploy
docker compose up -d
```

# Install tailscale
```
curl -fsSL https://tailscale.com/install.sh | sh
```

# Login to tailscale
```
sudo tailscale login
```

# In a web browser configure your tailnet to support https
```
https://login.tailscale.com/admin/dns
-> enable HTTPs certs
```

# Start tailscale 
```
tailscale serve --bg http://localhost:14499
```


### Test it out

Install the tailscale app on your phone and login
Browse to https://domainname
