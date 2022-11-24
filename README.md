# NGINX LB with LETSENCRYPT

## app

öncelikle çalışacak appleri yapalım klasik bir python flask kullanacağız.

app1 ve app2 Dockerfile ları ile birlikte repoda mevcut. 

build alıp containerları çalıştıracağız. 

## nginx-install

öncelikle nginx kuracağız.

apt-get update -y

apt-get install -y nginx

nginx kurduktan sonra cert-bot kurulumu yapacağız. nginx kullandığımız için aşağıdaki şekilde yükleyeceğiz.

```
apt install python-certbot-nginx 
````

spesific domain için certbot çalıştıracağız.

```
sudo certbot --nginx -d dev-ops.expert -d www.dev-ops.expert
```

daha sonra /etc/nginx/sites-available/default dosyasını aşağıdaki komutla değiştireceğiz. ve düzenlemeler yapacağız.

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

    ssl_certificate      <full.pem>;
    ssl_certificate_key  <key.pem>;

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
