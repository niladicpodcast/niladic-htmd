---
title: How to Setup Let's Encrypt with nginx
author: Cody
published: 2016-05-15T00:00:00+00:00
_hash: 17f74d8e390edccec21b09ec0e4a7b4d5b4c18e7af5c5972c76494a12574b979
updated: 2026-04-07T09:03:47.412804+00:00
...

<a href="https://letsencrypt.org/" target="_blank">Let's Encrypt</a> is a service to get free HTTPS certificates and automate renewing them.
This is a guide to setting up Let's Encrypt when using `nginx` as a reverse proxy.

Let's say you have a service running on localhost on a port other than 80 (ex. 8080).

Nginx will listen on port 80 and transfer traffic to the service or services only available internally.

If you would like to use a domain or a subdomain to access the service then add a DNS record for the server's IP address.

To get a certificate first use this `nginx` config.
Replace `example.com` with your domain or subdomain.

```
server {
  listen 80;
  server_name example.com;
  location /.well-known/acme-challenge {
    root /var/www/letsencrypt;
  }
}
```

Create the web root

```shell
sudo mkdir -p /var/www/letsencrypt
```

Install Let's Encrypt

```shell
git clone https://github.com/certbot/certbot
cd certbot
# replace $email with your email address
./certbot-auto certonly -a webroot --webroot-path /var/www/letsencrypt --agree-tos --expand --text --email $email -d example.com
```

You can add additional subdomains with `-d` options.

```shell
./certbot-auto certonly -a webroot --webroot-path /var/www/letsencrypt --agree-tos --expand --text --email $email -d example.com -d sub1.example.com -d sub2.example.com
```

To renew the certs you can run:

```shell
/path/to/certbot/certbot-auto renew --renew-hook="systemctl reload nginx"
```

This will reload `nginx` if the cert is renewed.

To automatically renew the certificate, add this to the root crontab. This will run once a day.
```
# Renew Let's Encrypt cert if required
4 4 * * * /path/to/certbot/certbot-auto renew --renew-hook="systemctl reload nginx"
```

So now you have certificates for the services. Time to use them!

`nginx` will handle the `HTTPS` traffic and send `HTTP` to internal services.

```
# Redirect HTTP to HTTPS
server {
  listen 80 default;
  server_name example.com;
  return 301 https://$server_name$request_uri;
}

server {
  # Listen on HTTPS
  listen 443 default ssl;
  server_name localhost example.com;

  # Certificate
  ssl_certificate   /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
  ssl_trusted_certificate /etc/letsencrypt/live/example.com/fullchain.pem;

  # For Let's Encrypt renewal
  location /.well-known/acme-challenge {
    root /var/www/letsencrypt;
  }

  # Redirect to service running on localhost
  location / {
    proxy_pass http://localhost:8080;
  }
}
```
