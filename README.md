![Alt text](app/assets/images/plus_ultra.png?raw=true "Plus Ultra")

# README

Aim:

Starter app with everything needed to launch and run a business. Just add an idea and designer.

Modern Rails 7 Hotwire app with no Javascript build complexity.

## 1. Marketing

- Blog 
- Email lists
- Campaign management

## 2. Sales

- Products 
- Once off & subscription payments via Stripe
- Shopping Cart

## 3. Operations

- User/Customer management with 2FA
- Receipts

## 4. Reporting

- Customer signup 
- Sales performance 
- Campaign tracking

## 5. Testing 

- Watir & Chrome
- install Chrome and compatible version of chromedriver

## Development Environment

### Mailcatcher

* Mailcatcher url: http://127.0.0.1:1080/
 - run mailcatcher in console 
 - mailcatcher -f -b -v

### Stripe CLI
- https://stripe.com/docs/stripe-cli
- stripe login  - to authenticate - will need to do this every 90 days
- stripe listen --forward-to localhost:3000/webhooks/stripe
- Trigger test api calls 
-- stripe trigger checkout.session.completed to test responses will be received

## Application Notes

### Rails 
- Active Storage for file uploads
- ActionText Trix WYSIWYG editor
- dotenv for environment
- Redis for key value store
- Boostrap SCSS
- Sanitize for cleaning up text input by user

### Third parties 
- Google Tag Manager 
- Facebook Open Graph Pixel tracking
- Twilio for SMS - outbound and inbound
- Stripe for purchased and subscriptions

#### Sample .env 

- APP_NAME           = "Plus Ultra"
- MOTTO             = "Go Beyond"
- WHO_AM_I          = "http://localhost:3000"
- STRIPE_PUBLIC_KEY = "will start with pk_..."
- STRIPE_SECRET_KEY = "will start with sk_..."
- DATABASE_NAME_DEV  = "development db name"
- DATABASE_NAME_TEST = "test db name"
- DATABASE_NAME_PROD = "production db name"
- DATABASE_MAX_POOL  = 5
- DATABASE_TIMEOUT   = 5000
- FACEBOOK_DOMAIN_VERIFICATION = "YOUR FACEBOOK VERIFICATION CODE"
- FACEBOOK_OPENGRAPH_IMAGE = "https://www.my-domain.com/images/my-facebook-opengraph.jpg"
- GOOGLE_TAG_MANAGER_ID = ""
- TWILIO_ACCOUNT_SID = ""
- TWILIO_AUTH_TOKEN  = ""
- TWILIO_CALLBACK_URL = "https://www.my-domain.com/webhooks/twilio"
- TWILIO_MOBILE_NUMBER_SID = ""
- TWILIO_MOBILE_NUMBER = "+61400000000"


### Operations
- Admin Dashboard 
- Customer Dashboard
- Responsive HTML Email templates (Outlook compatible)
- Auditing for recording model changes and user who made them - https://github.com/collectiveidea/audited

### Useful Web stuff
- Phony & Phonelib for international telephone numbers
- Invisible Captcha for anti-spam form catching
- QRCode to generate QR codes
- SEO optimisation - Title, Meta Description/Keywords and image tagging
- Charting Chartkick https://chartkick.com/ & ApexCharts https://github.com/styd/apexcharts.rb
- Font awesome free

### Make money with Stripe
- Once off Purchase
- Subscriptions (weekly, fortnightly, monthly, annual) with idempotence.
- Webhooks

# 
* TODO 

- elastic search
- Outbound campaigns and inbound tracking
- subscriptions history
- upgrade to Ruby 3.3 & YJIT
- campaign management

* Ruby version

* System dependencies
- See Gem file 
- Mailcatcher for development and test environments 
- chromedriver 
- postgres

* Configuration

* Database creation
- rails db:create 

* Database initialization

* How to run the test suite
- cd watir 
- ./go.sh TestFile.rb

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions
gem install watir
gem install mailcatcher 

## Setup server 

- setup rails server in /var/apps/site_name 
- setup puma  server in /var/apps/shared/site_name

## Let's Encrypt Free SSL / Certbot

- First setup NGINX (see below) and enure http is visible on web.
- Verify NGINX setup is good: sudo nginx -t
- enable HTTPS through firewall 
- sudo ufw status
- sudo ufw allow 'Nginx Full'
- sudo ufw delete allow 'Nginx HTTP'
- sudo ufw status
- sudo apt install certbot python3-certbot-apache
- sudo certbot --nginx -d my_site.domain -d www.my_site.domain
- Choose to redirect HTTP to HTTPS
- Verify certbot has automated 90 renewal process: sudo systemctl status certbot.timer
- Test certbot renewal: sudo certbot renew --dry-run
- Resolve any errors

## NGINX 

- /etc/nginx/nginx.conf 

user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	gzip_vary on;
	gzip_proxied any;
	gzip_comp_level 6;
	gzip_buffers 16 8k;
	gzip_http_version 1.1;
	gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
        gzip_disable "MSIE [1-6]\.";

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}




- /etc/nginx/sites_enabled - one file for each domain

upstream my_site {
    # Path to Puma SOCK file, as defined previously
    server unix:/var/apps/shared/my_site/sockets/puma.sock fail_timeout=0;
}

server {
    server_name my_site.domain;

    return 301 https://www.mysite.domain$request_uri;


    root /var/apps/my_site/public;

    try_files $uri/index.html $uri @my_site;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    location @my_site {
	      proxy_http_version 1.1;

	      proxy_set_header Upgrade $http_upgrade;
	      proxy_set_header Connection "upgrade";

        proxy_pass http://my_site;
        # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # proxy_set_header Host $http_host;
        #proxy_redirect off;

        proxy_set_header  Host $host;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header  X-Forwarded-Proto $scheme;
        proxy_set_header  X-Forwarded-Ssl on; # Optional
        proxy_set_header  X-Forwarded-Port $server_port;
        proxy_set_header  X-Forwarded-Host $host;
    }

    error_page 500 502 503 504 /500.html;
    client_max_body_size 1G;
    keepalive_timeout 10;

    # Certbot will add SSL/port 443 config here
}

server {
    # Certbot will insert redirection to SSL/Port 443 here

    server_name mysite_domain;
    listen 80;
}


## License

Plus Ultra is released under the [MIT License](https://opensource.org/licenses/MIT).

