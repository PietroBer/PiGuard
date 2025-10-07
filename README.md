# PiGuard
PiGuard is a docker-compose that combines WireGuard and PiHole to create a VPN with adblock features, designated to be used on mobile devices where it's more difficult to install adblocking software.

## Why?
The idea behind it is to create a VPN tunnel to your VPS or homeserver so that you can use PiHole adblocking features on your devices, primarily your phone.\
I share this little `compose.yaml` after searching and reading many blogpost and combining them to create a working environment for my needs.

## Ideal scenario
You have a personal linux-based server (either your Homeserver or a cheap VPS) with a static IP address.\
The `compose.yaml` create two containers: wg-easy to manage Wireguard VPN capabilities with an easy to use webUI and PiHole to work as a DNS server with built-in adblocking.\
I assume you already have installed in your server Docker.

## Installation
Just edit the `compose.yaml` to your need and load it directly to your server via Portainer or any other Docker manager.

### Create the Wireguard HASH password
To create your password's HASH open a terminal in your server\
`docker run --rm -it ghcr.io/wg-easy/wg-easy wgpw <PASSWORD>`\
<br>
For example if you want to use "CARTMAN" as your password:\
`docker run --rm -it ghcr.io/wg-easy/wg-easy wgpw CARTMAN`\
<br>
You should get something like\
`PASSWORD_HASH='$2b$12$coPqCsPtcFO.Ab99xylBNOW4.Iu7OOA2/ZIboHN6/oyxca3MWo7fW'`\
<br>
Before using you have to remove the two ' at the start and the end and add $ wherever there is one $. So you get:\
`PASSWORD_HASH=$$2b$$12$$coPqCsPtcFO.Ab99xylBNOW4.Iu7OOA2/ZIboHN6/oyxca3MWo7fW`\
<br>
Now you can add it to your `compose.yaml`

### How to access your services
- Wireguard-WebUI: http://YOUR_SERVER_IP:51821
- PiHole: http://YOUR_SERVER_IP:8880/admin

### Setup PiHole
Before finishing you have to allow PiHole to work in your VPN.\
Access PiHole-WebUI (http://YOUR_SERVER_IP:8880/admin)\
Go to `settings` -> `DNS`, enable `Expert mode` (in the top right there is a basic toggle).\
In `Interface settings` select `Permit all origins` and `Save & apply`. 
<br>
If you want to add more list to PiHole I use Hagezi Pro++ without any problems (for now), chek it at https://github.com/hagezi/dns-blocklists

## How to use it
Go to you Wireguard UI (http://YOUR_SERVER_IP:51821)\
Create a new profile: `+ New`, give it a name for remeber (ie MyPhone).\
You can load the configuration to your phone via the QR code or by downloading the config file. If you chek the config file you can verify that Wireguard use PiHole IP as the DNS server and the endpoint is your server IP\
<br>
Make sure you havn't set some alternative DNS to your phone/tablet or PC, otherwise this may not work.\
<br>
Now connect your phone to the VPN and just use it, you should see way less ad in your browser and apps.\
After surfing the web go to the PiHole dashboard (http://YOUR_SERVER_IP:8880/admin) and you can see the numbers of blocked queries.

## Miscellaneous and common problems
### Nginx-proxy-manager
Nginx-proxy-manager uses port 80 so in the `compose.yaml` PiHole uses port 8880, but you can change it with wathever you like\
<br>
If you want to add a prosxy on Nginx-proxy-manager for your Pihole you have to:\
In `advanced` tab add 
```
location = / {
  return 301 /admin;
}
```

### Port already used (es 53)\
Change in your `compose.yaml` 53:53 with 5353:53 (you can change 5353 withj any port that you like, but it must not be used by any other services on your server)\
<br>
### Error creating the network\
This may happens if you already tried to create a setup like this but you failed and there is still an unused network in your docker with the same paramenters.\
To check your docker networks\
`sudo docker network ls`\
<br>
From the list remove the network you are not using\ 
`sudo docker network rm NETWORK_NAME`

## Thanks
These are some but absolutely not all the source that I used to create my setup (just the one I remember).\
https://pimylifeup.com/wireguard-docker/  
https://docs.pi-hole.net/docker/  
https://github.com/hagezi/dns-blocklists
