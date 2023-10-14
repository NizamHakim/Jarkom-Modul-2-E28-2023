# Jarkom-Modul-2-E28-2023
## Kelompok E28
- Shafa Nabilah Hanin / 5025211222
- Nizam Hakim Santoso / 5025211209

# Lapres Praktikum 2
## Topologi
![](/images/topologi.png)

## Config
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
