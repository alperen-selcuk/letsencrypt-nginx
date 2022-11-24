# NGINX LB with LETSENCRYPT

## app

öncelikle çalışacak appleri yapalım klasik bir python flask kullanacağız.

app1.py ve app2.py Dockerfile ları ile birlikte repoda mevcut.

## nginx-lb

öncelikle nginx kuracağız.

apt-get install -y nginx



## nginx load balancer conf.

```
upstream app {
    server <app1 ip>:80;
    server <app2 ip>:80;
}

server {
    listen 80 default_server;
    server_name <domain>;

    # All other requests get load-balanced
    location / {
        return 301 https://$server_name$request_uri;
    }
}

server {
    listen 443 ssl default_server;

    server_name <domain>;

    ssl_certificate      /etc/letsencrypt/live/lb.serversforhackers.com/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/lb.serversforhackers.com/privkey.pem;

    charset utf-8;

    location / {
        include proxy_params;
        proxy_pass http://app;
        proxy_redirect off;

        # Handle Web Socket connections
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```
