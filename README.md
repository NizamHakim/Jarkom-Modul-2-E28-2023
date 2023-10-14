# Jarkom-Modul-2-E28-2023
## Kelompok E28
- Shafa Nabilah Hanin / 5025211222
- Nizam Hakim Santoso / 5025211209

# Lapres Praktikum 2
## Daftar Isi
- [Topologi](#Topologi)
- [Network Configuration](#Config)
- [Files](#Files)
  - [Router](#Router)
  - [NakulaClient dan SadewaClient](#NakulaClient-dan-SadewaClient)
  - [YudhistiraDNSMaster](#YudhistiraDNSMaster)
  - [WerkudaraDNSSlave](#WerkudaraDNSSlave)
  - [AbimanyuWebServer](#AbimanyuWebServer)
  - [PrabukusumaWebServer](#PrabukusumaWebServer)
  - [WisanggeniWebServer](#WisanggeniWebServer)
  - [ArjunaLoadBalancer](#ArjunaLoadBalancer)

## Topologi
![](/images/topologi.png)

## Network Configuration
- **Router**
  ```
  auto eth0
  iface eth0 inet dhcp

  auto eth1
  iface eth1 inet static
      address 192.220.1.1
      netmask 255.255.255.0

  auto eth2
  iface eth2 inet static
      address 192.220.2.1
      netmask 255.255.255.0
  ```
- **NakulaClient**
  ```
  auto eth0
  iface eth0 inet static
      address 192.220.1.2
      netmask 255.255.255.0
      gateway 192.220.1.1
      up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **SadewaClient**
  ```
  auto eth0
  iface eth0 inet static
	    address 192.220.1.3
	    netmask 255.255.255.0
	    gateway 192.220.1.1
	    up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **AbimanyuWebServer**
  ```
  auto eth0
  iface eth0 inet static
	    address 192.220.1.4
	    netmask 255.255.255.0
	    gateway 192.220.1.1
	    up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **PrabukusumaWebServer**
  ```
  auto eth0
  iface eth0 inet static
	    address 192.220.1.5
	    netmask 255.255.255.0
	    gateway 192.220.1.1
	    up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **WisanggeniWebServer**
  ```
  auto eth0
  iface eth0 inet static
	    address 192.220.1.6
	    netmask 255.255.255.0
	    gateway 192.220.1.1
	    up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **YudhistiraDNSMaster**
  ```
  auto eth0
  iface eth0 inet static
	    address 192.220.2.2
	    netmask 255.255.255.0
	    gateway 192.220.2.1
	    up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **WerkudaraDNSSlave**
  ```
  auto eth0
  iface eth0 inet static
	    address 192.220.2.3
	    netmask 255.255.255.0
	    gateway 192.220.2.1
	    up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **ArjunaLoadBalancer**
  ```
  auto eth0
  iface eth0 inet static
	    address 192.220.2.4
	    netmask 255.255.255.0
	    gateway 192.220.2.1
	    up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```

## Files
File-file ini harus ada pada /root masing-masing node
### Router
- **iptables.sh**
  ```shell
  iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.220.0.0/16
  ```
### NakulaClient dan SadewaClient
- **nameserver.sh**
  ```shell
  # DNS Master
  echo nameserver 192.220.2.2 > /etc/resolv.conf
  # DNS Slave
  echo nameserver 192.220.2.3 >> /etc/resolv.conf
  # Internet
  echo nameserver 192.168.122.1 >> /etc/resolv.conf
  ```
- **initClient.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install dnsutils -y
  apt-get install lynx -y
  /root/nameserver.sh
  clear
  echo "dnsutils installed"
  echo "lynx installed"
  echo "nameserver changed"
  echo "Client Ready"
  ```
### YudhistiraDNSMaster
- **named.conf.local**
  ```
  zone "arjuna.E28.com" {
        type master;
        file "/etc/bind/jarkom/arjuna.E28.com";
  };
  
  zone "abimanyu.E28.com" {
        type master;
        notify yes;
        also-notify { 192.220.2.3; };
        allow-transfer { 192.220.2.3; };
        file "/etc/bind/jarkom/abimanyu.E28.com";
  };
  
  zone "1.220.192.in-addr.arpa" {
  	type master;
    	file "/etc/bind/jarkom/1.220.192.in-addr.arpa";
  };
  ```
- **named.conf.options**
  ```
  options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      192.168.122.1;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        // dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
  };
  ```
- **arjuna.E28.com**
  ```
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     arjuna.E28.com. root.arjuna.E28.com. (
                     2022101001         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      arjuna.E28.com.
  @       IN      A       192.220.2.4     ; IP Arjuna
  www     IN      CNAME   arjuna.E28.com.
  @       IN      AAAA    ::1
  ```
- **abimanyu.E28.com**
  ```
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     abimanyu.E28.com. root.abimanyu.E28.com. (
                     2022101001         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
  ;
  @               IN      NS      abimanyu.E28.com.
  @               IN      A       192.220.1.4     ; IP Abimanyu
  www             IN      CNAME   abimanyu.E28.com.
  parikesit       IN      A       192.220.1.4     ; IP Abimanyu
  www.parikesit   IN      CNAME   parikesit.abimanyu.E28.com.
  nsl             IN      A       192.220.2.3     ; IP Werkudara
  baratayuda      IN      NS      nsl
  @               IN      AAAA    ::1
  ```
- **abimanyu.E28.ptr**
  ```
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     abimanyu.E28.com. root.abimanyu.E28.com. (
                     2022101001         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
  ;
  1.220.192.in-addr.arpa. IN      NS      abimanyu.E28.com.
  4                       IN      PTR     abimanyu.E28.com.
  ```
- **initDNSMaster.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install bind9 -y
  clear

  cp /root/named.conf.local /etc/bind/named.conf.local

  mkdir /etc/bind/jarkom

  cp /root/arjuna.E28.com /etc/bind/jarkom/arjuna.E28.com
  cp /root/abimanyu.E28.com /etc/bind/jarkom/abimanyu.E28.com
  cp /root/abimanyu.E28.ptr /etc/bind/jarkom/1.220.192.in-addr.arpa
  cp /root/named.conf.options /etc/bind/named.conf.options

  service bind9 start
  service bind9 restart
  echo "DNS Master Ready"
  ```
### WerkudaraDNSSlave
- **named.conf.local**
  ```
  zone "abimanyu.E28.com" {
    type slave;
    masters { 192.220.2.2; }; // Yudhistira
    file "/var/lib/bind/abimanyu.E28.com";
  };

  zone "baratayuda.abimanyu.E28.com" {
    type master;
    file "/etc/bind/baratayuda/baratayuda.abimanyu.E28.com";
  };
  ```
- **named.conf.options**
  ```
  options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        // dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
  };
  ```
- **baratayuda.abimanyu.E28.com**
  ```
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     baratayuda.abimanyu.E28.com. root.baratayuda.abimanyu.E28.com. (
                     2022101001         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      baratayuda.abimanyu.E28.com.
  @       IN      A       192.220.1.4     ; IP Abimanyu
  www     IN      CNAME   baratayuda.abimanyu.E28.com.
  rjp     IN      A       192.220.1.4     ; IP Abimanyu
  www.rjp IN      CNAME   rjp.baratayuda.abimanyu.E28.com.
  @       IN      AAAA    ::1
  ```
- **initDNSSlave.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install bind9 -y
  clear

  cp /root/named.conf.local /etc/bind/named.conf.local
  cp /root/named.conf.options /etc/bind/named.conf.options
  mkdir /etc/bind/baratayuda
  cp /root/baratayuda.abimanyu.E28.com /etc/bind/baratayuda/baratayuda.abimanyu.E28.com

  service bind9 start
  service bind9 restart
  echo "DNS Slave Ready"
  ```
### AbimanyuWebServer
- **arjuna-nginx-block**
  ```
  server {

        listen 8002;

        root /var/www/arjuna.E28;

        index index.php index.html index.htm;
        server_name _;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        }

  location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/default_error.log;
        access_log /var/log/nginx/default_access.log;
  }
  ```
- **abimanyu-apache-block**
  ```
  <VirtualHost *:80>
        ServerName abimanyu.E28.com
        ServerAlias www.abimanyu.E28.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/abimanyu.E28

        <Directory /var/www/abimanyu.E28/index.php/home>
                Options +Indexes
        </Directory>

        Alias "/home" "/var/www/abimanyu.E28/index.php/home"
  
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
- **parikesit.abimanyu-apache-block**
  ```
  <VirtualHost *:80>
        ServerName parikesit.abimanyu.E28.com
        ServerAlias www.parikesit.abimanyu.E28.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.E28

        <Directory /var/www/parikesit.abimanyu.E28/public>
                Options +Indexes
        </Directory>

        <Directory /var/www/parikesit.abimanyu.E28/secret>
                Options -Indexes
        </Directory>

        ErrorDocument 403 /error/403.html
        ErrorDocument 404 /error/404.html

        <Directory /var/www/parikesit.abimanyu.E28/public/js>
                Options +Indexes
        </Directory>

        Alias "/js" "/var/www/parikesit.abimanyu.E28/public/js"

        <Directory /var/www/parikesit.abimanyu.E28>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
  
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
- **parikesit-htaccess**
  ```
  <VirtualHost *:80>
        ServerName parikesit.abimanyu.E28.com
        ServerAlias www.parikesit.abimanyu.E28.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.E28

        <Directory /var/www/parikesit.abimanyu.E28/public>
                Options +Indexes
        </Directory>

        <Directory /var/www/parikesit.abimanyu.E28/secret>
                Options -Indexes
        </Directory>

        ErrorDocument 403 /error/403.html
        ErrorDocument 404 /error/404.html

        <Directory /var/www/parikesit.abimanyu.E28/public/js>
                Options +Indexes
        </Directory>

        Alias "/js" "/var/www/parikesit.abimanyu.E28/public/js"

        <Directory /var/www/parikesit.abimanyu.E28>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
- **rjp.baratayuda.abimanyu-apache-block**
  ```
  <VirtualHost *:14000 *:14400>
        ServerName rjp.baratayuda.abimanyu.E28.com
        ServerAlias www.rjp.baratayuda.abimanyu.E28.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/rjp.baratayuda.abimanyu.E28

        <Directory /var/www/rjp.baratayuda.abimanyu.E28>
                AuthType Basic
                AuthName "Restricted Content"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
- **default-apache-block**
  ```
  <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        Redirect / http://www.abimanyu.E28.com/

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
- **ports-apache.conf**
  ```
  Listen 80
  Listen 14000
  Listen 14400

  <IfModule ssl_module>
        Listen 443
  </IfModule>

  <IfModule mod_gnutls.c>
        Listen 443
  </IfModule>
  ```
- **downloadResources.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install wget -y
  apt-get install unzip -y

  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=1pPSP7yIR05JhSFG67RVzgkb-VcW9vQO6&export=download"
  unzip /root/x.zip
  rm /root/x.zip

  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=1LdbYntiYVF_NVNgJis1GLCLPEGyIOreS&export=download"
  unzip /root/x.zip
  rm /root/x.zip

  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=1a4V23hwK9S7hQEDEcv9FL14UkkrHc-Zc&export=download"
  unzip /root/x.zip
  rm /root/x.zip

  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX&export=download"
  unzip /root/x.zip
  rm /root/x.zip
  ```
- **initWebServer.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install nginx -y
  apt-get install php -y
  apt-get install php-fpm -y
  apt-get install apache2 -y
  apt-get install libapache2-mod-php7.0 -y
  /root/downloadResources.sh
  clear

  service nginx start
  service nginx status

  cp /root/arjuna-nginx-block /etc/nginx/sites-available/default
  mkdir /var/www/arjuna.E28
  cp /root/arjuna.yyy.com/* /var/www/arjuna.E28
  ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled
  service nginx restart
  service php7.0-fpm start
  service php7.0-fpm restart

  cp /root/abimanyu-apache-block /etc/apache2/sites-available/abimanyu.E28.com.conf
  mkdir /var/www/abimanyu.E28
  cp /root/abimanyu.yyy.com/* /var/www/abimanyu.E28
  a2ensite abimanyu.E28.com

  cp /root/parikesit.abimanyu-apache-block /etc/apache2/sites-available/parikesit.abimanyu.E28.com.conf
  mkdir /var/www/parikesit.abimanyu.E28
  cp -r /root/parikesit.abimanyu.yyy.com/* /var/www/parikesit.abimanyu.E28
  mkdir /var/www/parikesit.abimanyu.E28/secret
  touch /var/www/parikesit.abimanyu.E28/secret/rahasia.html
  a2ensite parikesit.abimanyu.E28.com

  cp /root/default-apache-block /etc/apache2/sites-available/000-default.conf
  a2ensite 000-default

  a2enmod rewrite
  touch /var/www/parikesit.abimanyu.E28/.htaccess
  cp /root/parikesit-htaccess /var/www/parikesit.abimanyu.E28/.htaccess

  cp /root/rjp.baratayuda.abimanyu-apache-block /etc/apache2/sites-available/rjp.baratayuda.abimanyu.E28.com.conf
  mkdir /var/www/rjp.baratayuda.abimanyu.E28
  cp /root/rjp.baratayuda.abimanyu.yyy.com/* /var/www/rjp.baratayuda.abimanyu.E28
  cp /root/ports-apache.conf /etc/apache2/ports.conf
  a2ensite rjp.baratayuda.abimanyu.E28.com

  htpasswd -c -b /etc/apache2/.htpasswd Wayang baratayudaE28

  service apache2 restart\
  echo "WebServer Ready"
  ```
### PrabukusumaWebServer
- **nginx-block**
  ```
  server {

        listen 8001;

        root /var/www/arjuna.E28;

        index index.php index.html index.htm;
        server_name _;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        }

  location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/default_error.log;
        access_log /var/log/nginx/default_access.log;
  }
  ```
- **downloadResources.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install wget -y
  apt-get install unzip -y
  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX&export=download"
  unzip /root/x.zip
  rm /root/x.zip
  ```
- **initWebServer.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install nginx -y
  apt-get install php -y
  apt-get install php-fpm -y
  /root/downloadResources.sh
  clear

  service nginx start
  cp /root/nginx-block /etc/nginx/sites-available/default
  mkdir /var/www/arjuna.E28
  cp /root/arjuna.yyy.com/* /var/www/arjuna.E28
  ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled

  service nginx restart
  service php7.0-fpm start
  service php7.0-fpm restart
  echo "WebServer Ready"
  ```
### WisanggeniWebServer
- **nginx-block**
  ```
  server {

        listen 8003;

        root /var/www/arjuna.E28;

        index index.php index.html index.htm;
        server_name _;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        }

  location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/default_error.log;
        access_log /var/log/nginx/default_access.log;
  }
  ```
- **downloadResources.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install wget -y
  apt-get install unzip -y
  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX&export=download"
  unzip /root/x.zip
  rm /root/x.zip
  ```
- **initWebServer.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install nginx -y
  apt-get install php -y
  apt-get install php-fpm -y
  /root/downloadResources.sh
  clear

  service nginx start
  cp /root/nginx-block /etc/nginx/sites-available/default
  mkdir /var/www/arjuna.E28
  cp /root/arjuna.yyy.com/* /var/www/arjuna.E28
  ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled

  service nginx restart
  service php7.0-fpm start
  service php7.0-fpm restart
  echo "WebServer Ready"
  ```
### ArjunaLoadBalancer
- **nginx-lb-block**
  ```
  upstream myweb  {
        server 192.220.1.4:8002; #IP Abimanyu
        server 192.220.1.5:8001; #IP Prabukusuma
        server 192.220.1.6:8003; #IP Wisanggeni
  }

  server {
        listen 80;
        server_name arjuna.E28.com;

        location / {
        proxy_pass http://myweb;
        }
  }
  ```
- **initLoadBalancer.sh**
  ```shell
  apt-get clean
  apt-get update
  apt-get install nginx -y
  clear

  service nginx start
  cp /root/nginx-lb-block /etc/nginx/sites-available/default
  ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled

  service nginx restart
  echo "LoadBalancer Ready"
  ```





