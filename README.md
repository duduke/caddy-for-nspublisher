Caddy Reverse Proxy on Publisher for HOST rewrite quick guide:
--------------------------------------------------------------

1. Get the latest caddy amd64 static binary from: https://github.com/caddyserver/caddy/releases (example as of writing: https://github.com/caddyserver/caddy/releases/download/v2.9.1/caddy_2.9.1_linux_amd64.tar.gz)
2. Transfer the caddy_x.x.x_linux_amd64.tar.gz to the publisher (/home/ubuntu/)
3. Login to the Netskope Publisher, and exit the npa_publisher_wizard menu
4. Extract the caddy binary: 
```
tar -xzvf ./caddy_x.x.x_linux_amd64.tar.gz
```
5. Copy the caddy binary to /usr/bin: 
```
sudo cp ./caddy /usr/bin
```
6. Create a foler for the Caddy config file:
```
sudo mkdir /etc/caddy
```
7. Create a new caddy configuration file: `/etc/caddy/CaddyFile` and paste the following content:
```
https://caddy-owa.office.lan:9443 {
    reverse_proxy real-owa.office.lan {
        transport http {
            tls
            tls_insecure_skip_verify
        }
        header_up Host {host}
    }

    tls internal

    # Optional: Log requests
    log {
        output file /var/log/caddy/caddy.log
    }

    # Optional: Define response headers
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    }
}
```
8. Create a caddy systemd service unit file `/etc/systemd/system/caddy.service` and paste the following:
```
# caddy.service
#
# For using Caddy with a config file.
#
# Make sure the ExecStart and ExecReload commands are correct
# for your installation.
#
# See https://caddyserver.com/docs/install for instructions.
#
# WARNING: This service does not use the --resume flag, so if you
# use the API to make changes, they will be overwritten by the
# Caddyfile next time the service is restarted. If you intend to
# use Caddy's API to configure it, add the --resume flag to the
# `caddy run` command or use the caddy-api.service file instead.

[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
Type=notify
User=root
Group=root
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile --force
TimeoutStopSec=5s
LimitNOFILE=1048576
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```
9. Enable the caddy systemd service:
```
sudo systemctl daemon-reload
sudo systemctl enable caddy.service
```
10. Start the caddy service:
`sudo systemctl start caddy.service`



Adding URLs to Caddy
--------------------

1. Add an entry to the Publisher hosts file that points to itself, give the entry a discriptive name for easy recognition.
Example hosts file:
```
127.0.0.1       localhost
127.0.1.1       publisher.lan      publisher
192.168.0.229	publisher.lan publisher
# The lines below are for apps that should go to Caddy
192.168.0.229	caddy-owa.office.lan
192.168.0.229	caddy-webapp1.office.lan	

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
2. Add an entry to /etc/caddy/CaddyFile for each webapp added to the hosts file. 
NOTE: The "reverse_proxy" directive should point to the actual webapp
```
https://caddy-owa.office.lan:9443 {
    reverse_proxy real-owa.office.lan {
        transport http {
            tls
            tls_insecure_skip_verify
        }
        header_up Host {host}
    }

    tls internal

    # Optional: Log requests
    log {
        output file /var/log/caddy/caddy.log
    }

    # Optional: Define response headers
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    }
}
```
3. Add a new Private Application with the following parameters:
- Application Name: Outlook Web Access via Caddy
- Browser Access: **Enabled**
- Host: **caddy-owa.office.lan**
- Browser Access Protocol: **HTTPS**, TCP Port: **9443**
- Trust self-signed certificates: **Enabled/True**
