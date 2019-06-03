# Sabnzbd Transmission with Web UI and OpenVPN
[![Docker Automated build](https://img.shields.io/docker/automated/haugene/transmission-openvpn.svg)](https://hub.docker.com/r/haugene/transmission-openvpn/)
[![Docker Pulls](https://img.shields.io/docker/pulls/mhutton81/sabmissionvpn.svg)](https://hub.docker.com/r/mhutton81/sabmissionvpn/)


This container contains OpenVPN and Transmission with a configuration
where Transmission is running only when OpenVPN has an active tunnel.
It bundles configuration files for many popular VPN providers to make the setup easier.

You need to specify your provider and credentials with environment variables,
as well as mounting volumes where the data should be stored.
An example run command to get you going is provided below.

It also bundles an installation of Tinyproxy to also be able to proxy web traffic over your VPN,
as well as scripts for opening a port for Transmission if you are using PIA or Perfect Privacy providers.

GL HF! And if you run into problems, please check the README twice.

## Sabnzbdplus
I have added SABnzbdplus to the container and it will allow you to use the tunneled vpn connection to download your usenets. Sabnzbd is run from the start.sh script located in the openvpn folder of this repository. I have is set to point to the /sabnzbd directory which is located outside of the container. 
this allows the sabnzbd.ini file to be written too and saved when you stop the container. for more information please go to https://sabnzbd.org/


## Run container from Docker registry
The container is available from the Docker registry and this is the simplest way to get it.
To run the container use this command:

```
$ docker run --cap-add=NET_ADMIN -d --name sabmissionvpn \
              -v /your/storage/path/:/data \
              -v /your/storage/path/:/sabnzbd \
              -v /your/storage/path/:/downloads \
              -v /etc/localtime:/etc/localtime:ro \
              -e CREATE_TUN_DEVICE=true \
              -e OPENVPN_PROVIDER=PIA \
              -e OPENVPN_CONFIG=US\ Seattle \
              -e OPENVPN_USERNAME=USER \
              -e OPENVPN_PASSWORD=PASS \
              -e WEBPROXY_ENABLED=true \
              -e WEBPROXY_PORT=9999 \
              -e LOCAL_NETWORK=192.168.0.0/24 \
              --log-driver json-file \
              --log-opt max-size=10m \
              -p 9091:9091 \
              -p 9090:9090 \
              mhutton81/sabmissionvpn 
```

You must set the environment variables `OPENVPN_PROVIDER`, `OPENVPN_USERNAME` and `OPENVPN_PASSWORD` to provide basic connection details.

The `OPENVPN_CONFIG` is an optional variable. If no config is given, a default config will be selected for the provider you have chosen.
Find available OpenVPN configurations by looking in the openvpn folder of the GitHub repository. The value that you should use here is the filename of your chosen openvpn configuration *without* the .ovpn file extension. For example:

```
-e "OPENVPN_CONFIG=ipvanish-AT-Vienna-vie-c02"
```

You can also provide a comma separated list of openvpn configuration filenames.
If you provide a list, a file will be randomly chosen in the list, this is useful for redundancy setups. For example:
```
-e "OPENVPN_CONFIG=ipvanish-AT-Vienna-vie-c02,ipvanish-FR-Paris-par-a01,ipvanish-DE-Frankfurt-fra-a01"
```
If you provide a list and the selected server goes down, after the value of ping-timeout the container will be restarted and a server will be randomly chosen, note that the faulty server can be chosen again, if this should occur, the container will be restarted again until a working server is selected.

To make sure this work in all cases, you should add ```--pull-filter ignore ping``` to your OPENVPN_OPTS variable.

As you can see, the container also expects a data volume to be mounted.
This is where Transmission will store your downloads, incomplete downloads and look for a watch directory for new .torrent files.
By default a folder named transmission-home will also be created under /data, this is where Transmission stores its state.

### Supported providers

This is a list of providers that are bundled within the image. Feel free to create an issue if your provider is not on the list, but keep in mind that some providers generate config files per user. This means that your login credentials are part of the config an can therefore not be bundled. In this case you can use the custom provider setup described later in this readme. The custom provider setting can be used with any provider.

| Provider Name                | Config Value (`OPENVPN_PROVIDER`) |
|:-----------------------------|:-------------|
| Anonine | `ANONINE` |
| AnonVPN | `ANONVPN` |
| BlackVPN | `BLACKVPN` |
| BTGuard | `BTGUARD` |
| Cryptostorm | `CRYPTOSTORM` |
| Cypherpunk | `CYPHERPUNK` |
| FreeVPN | `FREEVPN` |
| FrootVPN | `FROOT` |
| FrostVPN | `FROSTVPN` |
| GhostPath | `GHOSTPATH` |
| Giganews | `GIGANEWS` |
| HideMe | `HIDEME` |
| HideMyAss | `HIDEMYASS` |
| IntegrityVPN | `INTEGRITYVPN` |
| IPredator | `IPREDATOR` |
| IPVanish | `IPVANISH` |
| IronSocket | `IRONSOCKET` |
| Ivacy | `IVACY` |
| IVPN | `IVPN` |
| Mullvad | `MULLVAD` |
| Newshosting | `NEWSHOSTING` |
| NordVPN | `NORDVPN` |
| OVPN | `OVPN` |
| Perfect Privacy | `PERFECTPRIVACY` |
| Private Internet Access | `PIA` |
| PrivateVPN | `PRIVATEVPN` |
| ProtonVPN | `PROTONVPN` |
| proXPN | `PROXPN` |
| proxy.sh | `PROXYSH ` |
| PureVPN | `PUREVPN` |
| RA4W VPN | `RA4W` |
| SaferVPN | `SAFERVPN` |
| SlickVPN | `SLICKVPN` |
| Smart DNS Proxy | `SMARTDNSPROXY` |
| SmartVPN | `SMARTVPN` |
| Surfshark | `SURFSHARK` |
| TigerVPN | `TIGER` |
| TorGuard | `TORGUARD` |
| Trust.Zone | `TRUSTZONE` |
| TunnelBear | `TUNNELBEAR`|
| UsenetServerVPN | `USENETSERVER` |
| Windscribe | `WINDSCRIBE` |
| VPNArea.com | `VPNAREA` |
| VPN.AC | `VPNAC` |
| VPN.ht | `VPNHT` |
| VPNBook.com | `VPNBOOK` |
| VPNFacile | `VPNFACILE` |
| VPNTunnel | `VPNTUNNEL` |
| VyprVpn | `VYPRVPN` |

### Required environment options
| Variable | Function | Example |
|----------|----------|-------|
|`OPENVPN_PROVIDER` | Sets the OpenVPN provider to use. | `OPENVPN_PROVIDER=provider`. Supported providers and their config values are listed in the table above. |
|`OPENVPN_USERNAME`|Your OpenVPN username |`OPENVPN_USERNAME=asdf`|
|`OPENVPN_PASSWORD`|Your OpenVPN password |`OPENVPN_PASSWORD=asdf`|

### Network configuration options
| Variable | Function | Example |
|----------|----------|-------|
|`OPENVPN_CONFIG` | Sets the OpenVPN endpoint to connect to. | `OPENVPN_CONFIG=UK Southampton`|
|`OPENVPN_OPTS` | Will be passed to OpenVPN on startup | See [OpenVPN doc](https://openvpn.net/index.php/open-source/documentation/manuals/65-openvpn-20x-manpage.html) |
|`LOCAL_NETWORK` | Sets the local network that should have access. Accepts comma separated list. | `LOCAL_NETWORK=192.168.0.0/24`|
|`CREATE_TUN_DEVICE` | Creates /dev/net/tun device inside the container, mitigates the need mount the device from the host | `CREATE_TUN_DEVICE=true`|

### Firewall configuration options
When enabled, the firewall blocks everything except traffic to the peer port and traffic to the rpc port from the LOCAL_NETWORK and the internal docker gateway.

If TRANSMISSION_PEER_PORT_RANDOM_ON_START is enabled then it allows traffic to the range of peer ports defined by TRANSMISSION_PEER_PORT_RANDOM_HIGH and TRANSMISSION_PEER_PORT_RANDOM_LOW.

| Variable | Function | Example |
|----------|----------|-------|
|`ENABLE_UFW` | Enables the firewall | `ENABLE_UFW=true`|
|`UFW_ALLOW_GW_NET` | Allows the gateway network through the firewall. Off defaults to only allowing the gateway. | `UFW_ALLOW_GW_NET=true`|
|`UFW_EXTRA_PORTS` | Allows the comma separated list of ports through the firewall. Respects UFW_ALLOW_GW_NET. | `UFW_EXTRA_PORTS=9910,23561,443`|
|`UFW_DISABLE_IPTABLES_REJECT` | Prevents the use of `REJECT` in the `iptables` rules, for hosts without the `ipt_REJECT` module (such as the Synology NAS). | `UFW_DISABLE_IPTABLES_REJECT=true`|

### Health check option

Because your VPN connection can sometimes fail, Docker will run a health check on this container every 5 minutes to see if the container is still connected to the internet. By default, this check is done by pinging google.com once. You change the host that is pinged.

| Variable | Function | Example |
|----------|----------|-------|
| `HEALTH_CHECK_HOST` | this host is pinged to check if the network connection still works | `google.com` |

### Permission configuration options
By default the startup script applies a default set of permissions and ownership on the transmission download, watch and incomplete directories. The GLOBAL_APPLY_PERMISSIONS directive can be used to disable this functionality.

| Variable | Function | Example |
|----------|----------|-------|
|`GLOBAL_APPLY_PERMISSIONS` | Disable setting of default permissions | `GLOBAL_APPLY_PERMISSIONS=false`|

### Alternative web UIs
You can override the default web UI by setting the ```TRANSMISSION_WEB_HOME``` environment variable. If set, Transmission will look there for the Web Interface files, such as the javascript, html, and graphics files.

[Combustion UI](https://github.com/Secretmapper/combustion), [Kettu](https://github.com/endor/kettu) and [Transmission-Web-Control](https://github.com/ronggang/transmission-web-control/) come bundled with the container. You can enable either of them by setting```TRANSMISSION_WEB_UI=combustion```, ```TRANSMISSION_WEB_UI=kettu``` or ```TRANSMISSION_WEB_UI=transmission-web-control```, respectively. Note that this will override the ```TRANSMISSION_WEB_HOME``` variable if set.

| Variable | Function | Example |
|----------|----------|-------|
|`TRANSMISSION_WEB_HOME` | Set Transmission web home | `TRANSMISSION_WEB_HOME=/path/to/web/ui`|
|`TRANSMISSION_WEB_UI` | Use the specified bundled web UI | `TRANSMISSION_WEB_UI=combustion`, `TRANSMISSION_WEB_UI=kettu` or `TRANSMISSION_WEB_UI=transmission-web-control`|

### Transmission configuration options

You may override Transmission options by setting the appropriate environment variable.

The environment variables are the same name as used in the transmission settings.json file
and follow the format given in these examples:

| Transmission variable name | Environment variable name |
|----------------------------|---------------------------|
| `speed-limit-up` | `TRANSMISSION_SPEED_LIMIT_UP` |
| `speed-limit-up-enabled` | `TRANSMISSION_SPEED_LIMIT_UP_ENABLED` |
| `ratio-limit` | `TRANSMISSION_RATIO_LIMIT` |
| `ratio-limit-enabled` | `TRANSMISSION_RATIO_LIMIT_ENABLED` |

As you can see the variables are prefixed with `TRANSMISSION_`, the variable is capitalized, and `-` is converted to `_`.

Transmission options changed in the WebUI or in settings.json will be overridden at startup and will not survive after a reboot of the container. You may want to use these variables in order to keep your preferences.

PS: `TRANSMISSION_BIND_ADDRESS_IPV4` will be overridden to the IP assigned to your OpenVPN tunnel interface.
This is to prevent leaking the host IP.

### Web proxy configuration options

This container also contains a web-proxy server to allow you to tunnel your web-browser traffic through the same OpenVPN tunnel.
This is useful if you are using a private tracker that needs to see you login from the same IP address you are torrenting from.
The default listening port is 8888. Note that only ports above 1024 can be specified as all ports below 1024 are privileged
and would otherwise require root permissions to run.
Remember to add a port binding for your selected (or default) port when starting the container.

| Variable | Function | Example |
|----------|----------|-------|
|`WEBPROXY_ENABLED` | Enables the web proxy | `WEBPROXY_ENABLED=true`|
|`WEBPROXY_PORT` | Sets the listening port | `WEBPROXY_PORT=8888` |

### User configuration options

By default everything will run as the root user. However, it is possible to change who runs the transmission process.
You may set the following parameters to customize the user id that runs transmission.

| Variable | Function | Example |
|----------|----------|-------|
|`PUID` | Sets the user id who will run transmission | `PUID=1003`|
|`PGID` | Sets the group id for the transmission user | `PGID=1003` |

### Dropping default route from iptables (advanced)

Some VPNs do not override the default route, but rather set other routes with a lower metric.
This might lead to the default route (your untunneled connection) to be used.

To drop the default route set the environment variable `DROP_DEFAULT_ROUTE` to `true`.

*Note*: This is not compatible with all VPNs. You can check your iptables routing with the `ip r` command in a running container.

### Custom pre/post scripts

If you ever need to run custom code before or after transmission is executed or stopped, you can use the custom scripts feature.
Custom scripts are located in the /scripts directory which is empty by default.
To enable this feature, you'll need to mount the /scripts directory.

Once /scripts is mounted you'll need to write your custom code in the following bash shell scripts:

| Script | Function |
|----------|----------|
|/scripts/openvpn-pre-start.sh | This shell script will be executed before openvpn start |
|/scripts/transmission-pre-start.sh | This shell script will be executed before transmission start |
|/scripts/transmission-post-start.sh | This shell script will be executed after transmission start |
|/scripts/transmission-pre-stop.sh | This shell script will be executed before transmission stop |
|/scripts/transmission-post-stop.sh | This shell script will be executed after transmission stop |

Don't forget to include the #!/bin/bash shebang and to make the scripts executable using chmod a+x


## Access the WebUI
But what's going on? My http://my-host:9091 isn't responding?
This is because the VPN is active, and since docker is running in a different ip range than your client the response
to your request will be treated as "non-local" traffic and therefore be routed out through the VPN interface.

### How to fix this
The container supports the `LOCAL_NETWORK` environment variable. For instance if your local network uses the IP range 192.168.0.0/24 you would pass `-e LOCAL_NETWORK=192.168.0.0/24`.

## Known issues, tips and tricks

#### Use Google DNS servers
Some have encountered problems with DNS resolving inside the docker container.
This causes trouble because OpenVPN will not be able to resolve the host to connect to.
If you have this problem use dockers --dns flag to override the resolv.conf of the container.
For example use googles dns servers by adding --dns 8.8.8.8 --dns 8.8.4.4 as parameters to the usual run command.

#### Restart container if connection is lost
If the VPN connection fails or the container for any other reason loses connectivity, you want it to recover from it. One way of doing this is to set environment variable `OPENVPN_OPTS=--inactive 3600 --ping 10 --ping-exit 60` and use the --restart=always flag when starting the container. This way OpenVPN will exit if ping fails over a period of time which will stop the container and then the Docker deamon will restart it.

#### Reach sleep or hybernation on your host if no torrents are active
By befault Transmission will always [scrape](https://en.wikipedia.org/wiki/Tracker_scrape) trackers, even if all torrents have completed their activities, or they have been paused manually. This will cause Transmission to be always active, therefore never allow your host server to be inactive and go to sleep/hybernation/whatever. If this is something you want, you can add the following variable when creating the container. It will turn off a hidden setting in Tranmsission which will stop the application to scrape trackers for paused torrents. Transmission will become inactive, and your host will reach the desidered state.
```
-e "TRANSMISSION_SCRAPE_PAUSED_TORRENTS_ENABLED=false"
```
## Controlling Transmission remotely
The container exposes /config as a volume. This is the directory where the supplied transmission and OpenVPN credentials will be stored.
If you have transmission authentication enabled and want scripts in another container to access and
control the transmission-daemon, this can be a handy way to access the credentials.
For example, another container may pause or restrict transmission speeds while the server is streaming video.

## Running on ARM (Raspberry PI)
Since the Raspberry PI runs on an ARM architecture instead of x64, the existing x64 images will not
work properly. There are 2 additional Dockerfiles created. The Dockerfiles supported by the Raspberry PI are Dockerfile.armhf -- there is
also an example docker-compose-armhf file that shows how you might use Transmission/OpenVPN and the
corresponding nginx reverse proxy on an RPI machine.
You can use the `latest-armhf` tag for each images (see docker-compose-armhf.yml) or build your own images using Dockerfile.armhf.



