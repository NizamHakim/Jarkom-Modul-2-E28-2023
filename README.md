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
- [Soal Praktikum](#Soal-Praktikum)
  - [No 1](#No-1)
  - [No 2](#No-2)
  - [No 3](#No-3)
  - [No 4](#No-4)
  - [No 5](#No-5)
  - [No 6](#No-6)
  - [No 7](#No-7)
  - [No 8](#No-8)
  - [No 9](#No-9)
  - [No 10](#No-10)
  - [No 11](#No-11)
  - [No 12](#No-12)
  - [No 13](#No-13)
  - [No 14](#No-14)
  - [No 15](#No-15)
  - [No 16](#No-16)
  - [No 17](#No-17)
  - [No 18](#No-18)
  - [No 19](#No-19)
  - [No 20](#No-20) 
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
File-file ini harus ada pada `/root` masing-masing node. File-file yang ada di `/root` tidak akan terhapus meskipun project ditutup.
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
  RewriteEngine On
  RewriteCond %{REQUEST_URI} ^/public/images/(.*)(abimanyu)(.*\.(png|jpg))
  RewriteCond %{REQUEST_URI} !/public/images/abimanyu.png
  RewriteRule abimanyu http://parikesit.abimanyu.E28.com/public/images/abimanyu.png$1 [L,R=301]
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
  unzip -o /root/x.zip
  rm /root/x.zip

  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=1LdbYntiYVF_NVNgJis1GLCLPEGyIOreS&export=download"
  unzip -o /root/x.zip
  rm /root/x.zip

  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=1a4V23hwK9S7hQEDEcv9FL14UkkrHc-Zc&export=download"
  unzip -o /root/x.zip
  rm /root/x.zip

  wget -O "x.zip" "https://drive.google.com/u/0/uc?id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX&export=download"
  unzip -o /root/x.zip
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
  unzip -o /root/x.zip
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
  unzip -o /root/x.zip
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
### .bashrc
Masukkan shell script berikut ke `/root/.bashrc` agar script tersebut otomatis dieksekusi ketika node di start. Gunakan command `nano /root/.bashrc` dan tambahkan `/root/script-name.sh` ke dalam `.bashrc`
| Node | Script |
| --- | --- |
| **Router** | `/root/iptables.sh` |
| **NakulaClient** | `/root/initClient.sh` |
| **SadewaClient** | `/root/initClient.sh` |
| **YudhistiraDNSMaster** | `/root/initDNSMaster.sh` |
| **WerkudaraDNSSlave** | `/root/initDNSSlave.sh` |
| **AbimanyuWebServer** | `/root/initWebServer.sh` |
| **PrabukusumaWebServer** | `/root/initWebServer.sh` |
| **WisanggeniWebServer** | `/root/initWebServer.sh` |
| **ArjunaLoadBalancer** | `/root/initLoadBalancer.sh` |

## Soal Praktikum
### No 1
> Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian sebagai berikut. Folder topologi dapat diakses pada drive berikut.

#### Answer:  
(none) 
#### Testing:  
(none)  

### No 2
> Buatlah website utama pada node arjuna dengan akses ke arjuna.yyy.com dengan alias www.arjuna.yyy.com dengan yyy merupakan kode kelompok.

#### Answer:
Buka _Yudhistira_ untuk update package list dan install aplikasi bind9 dengan command berikut:
```
 apt-get update
 apt-get install bind9 -y
```

Lakukan perintah berikut pada _Yudhistira_:
```
cp /root/named.conf.local /etc/bind/named.conf.local
```

Isikan konfigurasai domain arjuna.E28.com (yyy: kelompok E28) dengan sintaks:
```
zone "arjuna.E28.com" {
      type master;
      file "/etc/bind/jarkom/arjuna.E28.com";
};
```

Buat folder jarkom dalam `/etc/bind`:
```
mkdir /etc/bind/jarkom
```

Copykan file db.local pada path /etc/bind ke dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi arjuna.E28.com
```
cp /root/arjuna.E28.com /etc/bind/jarkom/arjuna.E28.com
```

Kemudian buka arjuna.E28.com dengan `nano /etc/bind/jarkom/arjuna.E28.com` dan edit isinya serta tambahkan alias menjadi `www.arjuna.E28.com` seperti di bawah ini: 
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

Restart bind9 dengan perintah:
```
service bind9 restart
```

Untuk setting nameserver pada NakulaClient dan SadewaClient, arahkan nameserver menuju IP Yudhistira dengan mengedit file `resolv.conf` 
```
echo nameserver 192.220.2.2 > /etc/resolv.conf
```

#### Testing:  
Untuk memeriksa, dapat dilakukan dengan `host -t CNAME www.arjuna.E28.com` atau `ping www.arjuna.E28.com -c 5` pada NakulaClient dan SadewaClient
(gambar)
### No 3
> Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke abimanyu.yyy.com dan alias www.abimanyu.yyy.com.

#### Answer:  
Cara pengerjaannya sama dengan no. 2, yaitu:
Lakukan perintah berikut pada _Yudhistira_:
```
nano /etc/bind/named.conf.local/
```

Isikan konfigurasai domain abimanyu.E28.com (yyy: kelompok E28) dengan sintaks:
```
zone "abimanyu.E28.com" {
      type master;
      file "/etc/bind/jarkom/abimanyu.E28.com";
};
```

Buat folder jarkom dalam `/etc/bind`:
```
mkdir /etc/bind/jarkom
```

Copykan file db.local pada path /etc/bind ke dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi abimanyu.E28.com
```
cp /root/abimanyu.E28.com /etc/bind/jarkom/abimanyu.E28.com
```

Kemudian buka arjuna.E28.com dengan `nano /etc/bind/jarkom/abimanyu.E28.com` dan edit isinya serta tambahkan alias menjadi `www.abimanyu.E28.com` seperti di bawah ini: 
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
@               IN      AAAA    ::1
```

Restart bind9 dengan perintah:
```
service bind9 restart
```

Untuk setting nameserver pada NakulaClient dan SadewaClient, arahkan nameserver menuju IP Yudhistira dengan mengedit file `resolv.conf` 
```
echo nameserver 192.220.2.2 > /etc/resolv.conf
```
#### Testing:  
Untuk memeriksa, dapat dilakukan dengan `host -t CNAME www.abimanyu.E28.com` atau `ping www.abimanyu.E28.com -c 5` pada NakulaClient dan SadewaClient
(gambar)
### No 4
> Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain parikesit.abimanyu.yyy.com yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.

#### Answer:  
Pada Yudhistira, edit file `/etc/bind/jarkom/abimanyu.E28.com` lalu tambahkan subdomain parikesit.abimanyu.E28.com yang mengarah ke IP abimanyu
```
nano `/etc/bind/jarkom/abimanyu.E28.com`
```
Tambahkan konfigurasi seperti di bawah ini untuk abimanyu.E28.com
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
@               IN      AAAA    ::1
```
#### Testing:  
Restart service bind
```
service bind9 restart
```

Untuk memeriksa, dapat dilakukan dengan `host -t CNAME www.parikesit.abimanyu.E28.com` atau `ping www.parikesit.abimanyu.E28.com -c 5` pada NakulaClient dan SadewaClient
(gambar)

### No 5
> Buat juga reverse domain untuk domain utama. (Abimanyu saja yang direverse).

#### Answer:  
Edit `/etc/bind/named.conf.local` pada Yudhistira
```
nano /etc/bind/named.conf.local
```

Tambahkan konfigurasi dalam file tersebut dengan reverse 3 byte awal dari IP Abimanyu
```
zone "1.220.192.in-addr.arpa" {
	type master;
  	file "/etc/bind/jarkom/1.220.192.in-addr.arpa";
};
```
Copykan file ptr ke dalam folder jarkom dan ubah namanya menjadi `1.220.192.in-addr.arpa`
```
cp /root/abimanyu.E28.ptr /etc/bind/jarkom/1.220.192.in-addr.arpa
```
Edit file sehingga menjadi seperti di bawah ini:
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

#### Testing:  
Restart service bind
```
service bind9 restart
```

Untuk memeriksa, dapat dilakukan dengan command berikut pada client
```
// Install package dnsutils
// Pastikan nameserver di /etc/resolv.conf telah dikembalikan sama dengan nameserver dari Router
apt-get update
apt-get install dnsutils

//Kembalikan nameserver agar tersambung dengan EniesLobby
host -t PTR "IP Yudhistira"
```
(gambar)
### No 6
> Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.

#### Answer:  
Lakukan konfigurasi pada server Yudhistira dalam file `/etc/bind/named.conf.local` hingga menjadi seperti di bawah ini:
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

Restart service bind
```
service bind9 restart
```

Lakukan konfigurasi pada Werkudara dengan update package list dan install aplikasi bind9 dengan command berikut:
```
 apt-get update
 apt-get install bind9 -y
```
Kemudian, buka file `/etc/bind/named.conf.local` pada werkudara dan tambahkan perintah seperti di bawah ini:
```
zone "abimanyu.E28.com" {
  type slave;
  masters { 192.220.2.2; }; // Yudhistira
  file "/var/lib/bind/abimanyu.E28.com";
};
```
Restart service bind
```
service bind9 restart
```

#### Testing:  
Pada Yudhistira, silakan matikan service bind9
```
service bind9 stop
```

Pada Nakulaclient dan SadewaClient pastikan nameserver mengarah ke IP Yudhistira dan Werkudara
```
# DNS Master
echo nameserver 192.220.2.2 > /etc/resolv.conf
# DNS Slave
echo nameserver 192.220.2.3 >> /etc/resolv.conf
```
### No 7
> Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu baratayuda.abimanyu.yyy.com dengan alias www.baratayuda.abimanyu.yyy.com yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.

#### Answer:  
Edit file `abimanyu.E28.com ` pada Yudhistira seperti di bawah ini:
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

Pada Werkudara, lakukan konfigurasi dengan menambahkan perintah berikut pada `named.conf.local`:
```
zone "baratayuda.abimanyu.E28.com" {
  type master;
  file "/etc/bind/baratayuda/baratayuda.abimanyu.E28.com";
};
```

Buat folder `baratayuda` pada _Werkudara_:
```
mkdir /etc/bind/baratayuda
```

Edit file `/etc/bind/baratayuda/baratayuda.abimanyu.E28.com` seperti di bawah ini:
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
@       IN      AAAA    ::1
``` 
#### Testing:  
Restart service bind9
```
service bind9 restart
```

### No 8
> Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses rjp.baratayuda.abimanyu.yyy.com dengan alias www.rjp.baratayuda.abimanyu.yyy.com yang mengarah ke Abimanyu.

#### Answer:  
#### Testing:  

### No 9
> Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.

#### Answer:  
#### Testing:  

### No 10
> Kemudian gunakan algoritma Round Robin untuk Load Balancer pada Arjuna. Gunakan server_name pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003. Contoh: (Prabakusuma:8001, Abimanyu:8002, Wisanggeni:8003)

#### Answer:  
#### Testing:  

### No 11
> Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server www.abimanyu.yyy.com. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy.

#### Answer:  
Konfigurasi apache www.abimanyu.E28.com diatur pada file `abimanyu-apache-block` yaitu pada lines berikut  
```
<VirtualHost *:80>
      ServerName abimanyu.E28.com
      ServerAlias www.abimanyu.E28.com
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/abimanyu.E28

      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- VirtualHost: port yang digunakan untuk mengakses abimanyu.E28.com  
- ServerName: nama yang digunakan untuk mengakses abimanyu.E28.com  
- ServerAlias: alias dari abimanyu.E28.com  
- DocumentRoot: tempat file-file milik abimanyu.E28.com  

Copy `abimanyu-apache-block` ke `/etc/apache2/sites-available`  
```shell
cp /root/abimanyu-apache-block /etc/apache2/sites-available/abimanyu.E28.com.conf
```

File-file dari abimanyu.E28.com diatur pada directory `/var/www/abimanyu.E28`. Buat directory `/var/www/abimanyu.E28` kemudian copy files dari `/root/abimanyu.yyy.com` dan paste ke dalamnya  
```shell
mkdir /var/www/abimanyu.E28
cp /root/abimanyu.yyy.com/* /var/www/abimanyu.E28
```

Aktifkan website  
```shell
a2ensite abimanyu.E28.com
```

#### Testing:
Gunakan `lynx` pada NakulaClient untuk mengakses www.abimanyu.E28.com  
```shell
lynx www.abimanyu.E28.com
```
Menampilkan `index.html` yang ada di `/var/www/abimanyu.E28`  

![](/images/lynx-abimanyu.png)

### No 12
> Setelah itu ubahlah agar url www.abimanyu.yyy.com/index.php/home menjadi www.abimanyu.yyy.com/home.

#### Answer:  
Alias dari directory pada www.abimanyu.E28.com diatur dalam `abimanyu-apache-block` yaitu pada lines berikut  
```
<Directory /var/www/abimanyu.E28/index.php/home>
	Options +Indexes
</Directory>

Alias "/home" "/var/www/abimanyu.E28/index.php/home"
```
Ketika user mengakses www.abimanyu.yyy.com/home maka akan diteruskan menuju www.abimanyu.yyy.com/index.php/home  

#### Testing:  
Gunakan `lynx` pada NakulaClient untuk mengakses www.abimanyu.E28.com/home  
```shell
lynx www.abimanyu.E28.com/home
```
Menampilkan `index.html` yang ada di `/var/www/abimanyu.E28`  

![](/images/lynx-abimanyu.png)

### No 13
> Selain itu, pada subdomain www.parikesit.abimanyu.yyy.com, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy.

#### Answer:  
Konfigurasi apache www.parikesit.abimanyu.E28.com diatur pada file `parikesit.abimanyu-apache-block` yaitu pada lines berikut  
```
<VirtualHost *:80>
        ServerName parikesit.abimanyu.E28.com
        ServerAlias www.parikesit.abimanyu.E28.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.E28
  
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- VirtualHost: port yang digunakan untuk mengakses parikesit.abimanyu.E28.com  
- ServerName: nama yang digunakan untuk mengakses parikesit.abimanyu.E28.com  
- ServerAlias: alias dari parikesit.abimanyu.E28.com  
- DocumentRoot: tempat file-file milik parikesit.abimanyu.E28.com  

Copy `parikesit.abimanyu-apache-block` ke `/etc/apache2/sites-available`.  
```shell
cp /root/parikesit.abimanyu-apache-block /etc/apache2/sites-available/parikesit.abimanyu.E28.com.conf
```

File-file dari parikesit.abimanyu.E28.com diatur pada directory `/var/www/parikesit.abimanyu.E28`. Buat directory `/var/www/parikesit.abimanyu.E28` kemudian copy files dari `/root/parikesit.abimanyu.yyy.com` dan paste ke dalamnya  
```shell
mkdir /var/www/parikesit.abimanyu.E28
cp -r /root/parikesit.abimanyu.yyy.com/* /var/www/parikesit.abimanyu.E28
```

Tambahkan directory secret dan isi dengan sebarang file html untuk digunakan di nomor selanjutnya  
```shell
mkdir /var/www/parikesit.abimanyu.E28/secret
touch /var/www/parikesit.abimanyu.E28/secret/rahasia.html
```

Aktifkan website  
```shell
a2ensite parikesit.abimanyu.E28.com
```

#### Testing:    
Gunakan `lynx` pada NakulaClient untuk mengakses www.parikesit.abimanyu.E28.com  
```shell
lynx www.parikesit.abimanyu.E28.com
```
Menampilkan folder-folder yang yang ada di `/var/www/parikesit.abimanyu.E28`  

![](/images/lynx-parikesit.abimanyu.png)

### No 14
> Pada subdomain tersebut folder /public hanya dapat melakukan directory listing sedangkan pada folder /secret tidak dapat diakses (403 Forbidden).

#### Answer:  
Untuk mengaktifkan directory listing pada www.parikesit.abimanyu.E28.com/public maka pada `parikesit.abimanyu-apache-block` ditambahkan lines berikut  
```
<Directory /var/www/parikesit.abimanyu.E28/public>
	Options +Indexes
</Directory>
```
Sedangkan untuk menonaktifkan directory listing pada www.parikesit.abimanyu.E28.com/secret maka pada `parikesit.abimanyu-apache-block` ditambahkan lines berikut  
```
<Directory /var/www/parikesit.abimanyu.E28/secret>
	Options -Indexes
</Directory>
```

#### Testing:  
Gunakan `lynx` pada NakulaClient untuk mengakses www.parikesit.abimanyu.E28.com/public  
```shell
lynx www.parikesit.abimanyu.E28.com/public
```
Menampilkan folder-folder yang yang ada di `/var/www/parikesit.abimanyu.E28/public`  

![](/images/public.png)

Gunakan `lynx` pada NakulaClient untuk mengakses www.parikesit.abimanyu.E28.com/secret  
```shell
lynx www.parikesit.abimanyu.E28.com/secret
```
Menampilkan halaman forbidden 403  

![](/images/secret.png)

### No 15
> Buatlah kustomisasi halaman error pada folder /error untuk mengganti error kode pada Apache. Error kode yang perlu diganti adalah 404 Not Found dan 403 Forbidden.

#### Answer:  
Untuk kustomisasi halaman error 403 dan 404 pada www.parikesit.abimanyu.E28.com maka pada `parikesit.abimanyu-apache-block` ditambahkan lines berikut  
```
ErrorDocument 403 /error/403.html
ErrorDocument 404 /error/404.html
```
Dimana 403.html dan 404.html tersebut disimpan pada directory `/var/www/parikesit.abimanyu.E28/error`  

#### Testing:  
Gunakan `lynx` pada NakulaClient untuk mengakses www.parikesit.abimanyu.E28.com/secret  
```shell
lynx www.parikesit.abimanyu.E28.com/secret
```
Menampilkan halaman forbidden 403 yang telah dikustomisasi  

![](/images/secret.png)

Gunakan `lynx` pada NakulaClient untuk mengakses www.parikesit.abimanyu.E28.com/asdf  
```shell
lynx www.parikesit.abimanyu.E28.com/asdf
```
Menampilkan halaman not found 404 yang telah dikustomisasi  

![](/images/asdf.png)

### No 16
> Buatlah suatu konfigurasi virtual host agar file asset www.parikesit.abimanyu.yyy.com/public/js menjadi www.parikesit.abimanyu.yyy.com/js.

#### Answer:  
Alias dari directory pada www.parikesit.abimanyu.E28.com diatur dalam `parikesit.abimanyu-apache-block` yaitu pada lines berikut  
```
<Directory /var/www/parikesit.abimanyu.E28/public/js>
	Options +Indexes
</Directory>

Alias "/js" "/var/www/parikesit.abimanyu.E28/public/js"
```
Ketika user mengakses www.parikesit.abimanyu.E28.com/js maka akan diteruskan menuju www.parikesit.abimanyu.yyy.com/public/js  

#### Testing:  
Gunakan `lynx` pada NakulaClient untuk mengakses www.parikesit.abimanyu.E28.com/js  
```shell
lynx www.parikesit.abimanyu.E28.com/js
```
Menampilkan isi dari `/var/www/parikesit.abimanyu.E28/public/js`  

![](/images/js.png)

### No 17
> Agar aman, buatlah konfigurasi agar www.rjp.baratayuda.abimanyu.yyy.com hanya dapat diakses melalui port 14000 dan 14400.

#### Answer:  
Konfigurasi apache www.rjp.baratayuda.abimanyu.E28.com diatur dalam `rjp.baratayuda.abimanyu-apache-block` yaitu pada lines berikut  
```
<VirtualHost *:14000 *:14400>
      ServerName rjp.baratayuda.abimanyu.E28.com
      ServerAlias www.rjp.baratayuda.abimanyu.E28.com
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/rjp.baratayuda.abimanyu.E28

      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- VirtualHost: port yang digunakan untuk mengakses rjp.baratayuda.abimanyu.E28.com  
- ServerName: nama yang digunakan untuk mengakses rjp.baratayuda.abimanyu.E28.com  
- ServerAlias: alias dari rjp.baratayuda.abimanyu.E28.com  
- DocumentRoot: tempat file-file milik rjp.baratayuda.abimanyu.E28.com  

Bisa dilihat bahwa rjp.baratayuda.abimanyu.E28.com bisa diakses melalui 2 port yaitu 14000 dan 14400  

Kemudian tambahkan 14000 dan 14400 ke `ports-apache.conf` sehingga menjadi seperti berikut  
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

Copy `rjp.baratayuda.abimanyu-apache-block` ke `/etc/apache2/sites-available` dan `ports-apache.conf` ke `/etc/apache2/ports.conf`  
```shell
cp /root/rjp.baratayuda.abimanyu-apache-block /etc/apache2/sites-available/rjp.baratayuda.abimanyu.E28.com.conf
cp /root/ports-apache.conf /etc/apache2/ports.conf
```

File-file dari rjp.baratayuda.abimanyu.E28.com diatur pada directory `/var/www/rjp.baratayuda.abimanyu.E28`. Buat directory `/var/www/rjp.baratayuda.abimanyu.E28` kemudian copy files dari `/root/rjp.baratayuda.abimanyu.yyy.com` dan paste ke dalamnya  
```shell
mkdir /var/www/rjp.baratayuda.abimanyu.E28
cp /root/rjp.baratayuda.abimanyu.yyy.com/* /var/www/rjp.baratayuda.abimanyu.E28
```

Aktifkan website  
```shell
a2ensite rjp.baratayuda.abimanyu.E28.com
```

#### Testing:  
Gunakan `lynx` pada NakulaClient untuk mengakses www.rjp.baratayuda.abimanyu.E28.com melalui port 14000 atau 14400  
```shell
lynx www.rjp.baratayuda.abimanyu.E28.com:14000
lynx www.rjp.baratayuda.abimanyu.E28.com:14400
```
Menampilkan isi dari `/var/www/rjp.baratayuda.abimanyu.E28`  

![](/images/rjp.png)

Gunakan `lynx` pada NakulaClient untuk mengakses www.rjp.baratayuda.abimanyu.E28.com melalui port selain 14000 dan 14400  
```shell
lynx www.rjp.baratayuda.abimanyu.E28.com:8080
```
Menampilkan halaman error karena port tidak sesuai  

![](/images/rjp-error.png)

### No 18
> Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy.

#### Answer:  
Untuk menambahkan autentikasi ketika mengakses www.rjp.baratayuda.abimanyu.E28.com maka pada `rjp.baratayuda.abimanyu-apache-block` ditambahkan lines berikut  
```
<Directory /var/www/rjp.baratayuda.abimanyu.E28>
	AuthType Basic
	AuthName "Restricted Content"
	AuthUserFile /etc/apache2/.htpasswd
	Require valid-user
</Directory>
```
- AuthType: pilihan tipe autentikasi  
- AuthName: message yang ditampilkan pada halaman autentikasi  
- AuthUserFile: file tempat username dan password  
- Require: persyaratan yang harus dipenuhi untuk bisa mengakses directory  

Kemudian untuk membuat username dan password jalankan command berikut  
```shell
htpasswd -c -b /etc/apache2/.htpasswd Wayang baratayudaE28
```
Command ini akan membuat file `.htpassword` yang berisi Username: Wayang dengan Password: baratayudaE28 yang telah di hash  

#### Testing:  
Gunakan `lynx` pada NakulaClient untuk mengakses www.rjp.baratayuda.abimanyu.E28.com melalui port 14000 atau 14400  
```shell
lynx www.rjp.baratayuda.abimanyu.E28.com:14000
```
Menampilkan halaman autentikasi  

![](/images/username.png)

![](/images/password.png)

Autentikasi berhasil   

![](/images/rjp.png)

### No 19
> Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke www.abimanyu.yyy.com (alias).

#### Answer:  
Untuk mengalihkan akses melalui IP kita bisa melakukannya dengan mengedit default block dari apache, default block dari apache diatur dalam `default-apache-block` sebagai berikut  
```
<VirtualHost *:80>
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/html

      Redirect / http://www.abimanyu.E28.com/

      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Ketika user mengakses node abimanyu melalui IP maka akan secara otomatis menggunakan default block dari apache, oleh karena itu kita meredirect akses menuju www.abimanyu.E28.com  

Copy `default-apache-block` ke `/etc/apache2/sites-available`  
```shell
cp /root/default-apache-block /etc/apache2/sites-available/000-default.conf
```

Aktifkan website  
```shell
a2ensite 000-default
```

#### Testing:  
Gunakan `lynx` pada NakulaClient untuk mengakses IP Abimanyu 192.220.1.4  
```shell
lynx 192.220.1.4
```
Menampilkan `index.html` yang ada di `/var/www/abimanyu.E28`  

![](/images/lynx-abimanyu.png)

### No 20
> Karena website www.parikesit.abimanyu.yyy.com semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png.

#### Answer:  
Enable module rewrite menggunakan command  
```shell
a2enmod rewrite
```

Module rewrite ini diatur dalam `parikesit-htaccess`  
```
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/public/images/(.*)(abimanyu)(.*\.(png|jpg))
RewriteCond %{REQUEST_URI} !/public/images/abimanyu.png
RewriteRule abimanyu http://parikesit.abimanyu.E28.com/public/images/abimanyu.png$1 [L,R=301]
```

Buat file `.htaccess` pada directory `/var/www/parikesit.abimanyu.E28` kemudian copy `parikesit-htaccess` ke dalamnya  
```
touch /var/www/parikesit.abimanyu.E28/.htaccess
cp /root/parikesit-htaccess /var/www/parikesit.abimanyu.E28/.htaccess
```

Konfigurasi `.htaccess` pada `parikesit.abimanyu-apache-block` diatur menggunakan lines berikut  
```
<Directory /var/www/parikesit.abimanyu.E28>
	Options +FollowSymLinks -Multiviews
	AllowOverride All
</Directory>
```

#### Testing:  
Gunakan `lynx` pada NakulaClient untuk mengakses file pada parikesit.abimanyu.E28.com/public/images yang mengandung substring abimanyu  
```
lynx parikesit.abimanyu.E28.com/public/images/not-abimanyu.png
lynx parikesit.abimanyu.E28.com/public/images/abimanyu-student.jpg
lynx parikesit.abimanyu.E28.com/public/images/abimanyu.png
```
Menampilkan halaman download abimanyu.png karena telah dialihkan menuju `parikesit.abimanyu.E28.com/public/images/abimanyu.png`  

![](/images/abimanyu-img.png)


