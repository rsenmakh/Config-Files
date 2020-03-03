```nginx
server {
        listen 80;
        listen [::]:80;
        server_name www.domain.com domain.com;
        return 301 https://domain.com$request_uri;
}

server {
        listen 443 ssl;
        listen [::]:443 ssl;
        server_name www.domain.com;

        ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem; # managed by Certbot

        return 301 https://domain.com$request_uri;   
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2 ipv6only=on;
        server_name domain.com;

        ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        root /var/www/domain.com/www/; #Укажите путь к корню сайта
        index index.php index.html;
     
        charset UTF-8;
        autoindex off;
        client_max_body_size 4m;
        error_page 403 =404;

        location ~ /\. {
                access_log off;
                log_not_found off;
                deny all;
        }

        location ~* \.(tpl|twig|ini|log|txt)$ {
                deny all;
        }

        location ~* \/admin/index.php {

                location ~ \.php$ {
                        include snippets/fastcgi-php.conf;
                        fastcgi_pass unix:/var/run/php/php7.2-fpm-domain.com.sock; #Укажите путь к вашему сокету
                        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                        include fastcgi_params;
                }

        }
        
        location ~* \/(admin|catalog|image).+(\.php) {
                deny all;
        }

        location ~* \/system.+\.(js|css) {
                allow all;
        }
        
        location ~* \/system.+(\.*) {
                deny all;
        }

        location = /sitemap.xml {
                rewrite ^(.*)$ /index.php?route=extension/feed/d_google_sitemap last; # Укажите путь к вашему sitemap.xml
        } 

        location = /favicon.ico {
                log_not_found off;
                access_log off;
                }

        location = /robots.txt {
                allow all;
                log_not_found off;
                access_log off;
        }

        location = /apple-touch-icon.png {
                log_not_found off;
                access_log off;
        }

        location = /apple-touch-icon-precomposed.png {
                log_not_found off;
                access_log off;
        }

        location / {
                try_files $uri $uri/ @rewrite;
        }

        location @rewrite {
                rewrite ^/(.+)$ /index.php?_route_=$1 last;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm-domain.com.sock; #Укажите путь к вашему сокету
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
     
        location ~* \.(jpg|jpeg|gif|png|ico|pdf|ppt|bmp|rtf|svg|otf|woff|woff2|ttf|tag)$ {
                expires max;
                add_header Pragma public;
                add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        }

        location ~* \.(js|css|txt)$ {
                expires 3d;
                add_header Pragma public;
                add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        }
}
