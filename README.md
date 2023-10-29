# FINAL-PROJECT-OS-SERVER-SYSTEM-ADMIN---22.83.0886

Repository ini berisi dokumentasi yang menjelaskan cara instalasi dan konfigurasi berbagai layanan server, seperti SSH, DHCP, FTP/Samba, DNS, mail server, dan web server, serta Database Server. Dokumen ini ditujukan untuk memandu pengguna dalam mengatur dan mengelola server-server ini dengan langkah-langkah yang jelas.

## Daftar Isi
1. [Instalasi dan Konfigurasi SSH](#1-instalasi-dan-konfigurasi-ssh)
2. [Instalasi dan Konfigurasi DHCP Server](#2-instalasi-dan-konfigurasi-dhcp-server)
3. [Instalasi dan Konfigurasi FTP dan Samba Server](#3-instalasi-dan-konfigurasi-ftp-dan-samba-server)
4. [Instalasi dan Konfigurasi Database Server](#4-instalasi-dan-konfigurasi-database-server)
5. [Instalasi dan Konfigurasi DNS Server](#5-instalasi-dan-konfigurasi-dns-server)
6. [Instalasi dan Konfigurasi Web Server](#6-instalasi-dan-konfigurasi-web-server)
7. [Instalasi dan Konfigurasi Mail Server](#7-instalasi-dan-konfigurasi-mail-server)

## 1. Instalasi dan Konfigurasi SSH Server

### 1.1 Instalasi SSH
**Langkah 1: Lakukan Instalasi Paket SSH Server**

```
apt-get install openssh-server
```
### 1.2 Konfigurasi SSH
**Langkah 1: Buka Direktori konfigurasi ssh dengan text editor(disini saya menggunakan nano)**
```
nano /etc/ssh/sshd_config
```
**Langkah 2: Edit Konfigurasi seperti dibawah ini**
```
Include /etc/ssh/sshd_config.d/*.conf

Port 9029
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody
```
Saya mengubah Port dari 22 ke 9029 untuk mengamankanya agar tidak menggunakan default port,dan untuk "PermitRootLogin" bisa di biarkan "prohibit-password" jika anda tidak mengizikan root untuk login dengan remote SSH

**Langkah 3: Restart layanan SSH Server**
```
systemctl restart sshd
```
**Langkah 4: Menguji Konfigurasi**
```
ssh root@IPADDR -p 9029
```
Bisa disesuaikan dengan IP dan Port yang anda konfigurasi

## 2. Instalasi dan Konfigurasi DHCP Server

### 2.1 Instalasi DHCP
**Langkah 1: Instalasi Paket DHCP Server**
```
apt-get install isc-dhcp-server
```
Jika anda menggunakan basis Linux Distro Red Hat Enterprise Linux (RHEL) seperti centos biasanya nama paket nya adalah:
```
yum install dhcp
```
### 2.2 Konfigurasi DHCP Server
**Langkah 1: Buka Direktori Konfigurasi DHCPD**
```
nano /etc/dhcp/dhcpd.conf
```
**Langkah 2: Edit Konfigurasi file Seperti dibawah ini**
```
Line 49
# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}
```
Sesuaikan dengan IP Address,Prefix dan Hostname Server Anda
```
subnet 192.168.171.0 netmask 255.255.255.0 {
  range 192.168.171.11 192.168.171.254;
  option domain-name-servers 192.168.171.10,1.1.1.1;
  option domain-name "finalprojectku.com";
  option routers 192.168.171.10;
  option broadcast-address 192.168.171.255;
  default-lease-time 600;
  max-lease-time 7200;
}
```
**Langkah 3: Deklarasikan DHCP Server dengan interface yang akan anda fungsikan sebagai DHCP Server**
```
nano /etc/default/isc-dhcp-server
```
Sesuaikan dengan interface yang anda gunakan
```
INTERFACES=""
```
seperti ini:
```
INTERFACES="enp0s8"
```
**Langkah 4: Restart Layanan DHCPD**
```
systemctl restart isc-dhcp-server
```
**Langkah 5: Menguji Konfigurasi**

1.Dari sisi Client anda bisa melakukan Request IP ke DHCP POOL(saya menggunakan Windows sebagai Client)

Gambar

2.Dari sisi Server anda bisa mengamati bahwa ada activity bahwa DHCP meminjamkan IP ke Client
```
Oct 29 02:37:17 finalprojectku systemd[1]: Starting isc-dhcp-server.service - LSB: DHCP server...
Oct 29 02:37:17 finalprojectku isc-dhcp-server[16797]: Launching IPv4 server only.
Oct 29 02:37:17 finalprojectku isc-dhcp-server[16797]: Starting ISC DHCPv4 server: dhcpdingore stale pid file /var/run/dhcpd.pid
Oct 29 02:37:19 finalprojectku isc-dhcp-server[16797]: .
Oct 29 02:37:19 finalprojectku systemd[1]: Started isc-dhcp-server.service - LSB: DHCP server.
Oct 29 02:45:14 finalprojectku dhcpd[16734]: DHCPDISCOVER from 0a:00:27:00:00:1e via enp0s8
Oct 29 02:45:15 finalprojectku dhcpd[16734]: DHCPOFFER on 192.168.171.11 to 0a:00:27:00:00:1e (MNHAQIQI) via enp0s8
Oct 29 02:45:15 finalprojectku dhcpd[16734]: DHCPREQUEST for 192.168.171.11 (192.168.171.10) from 0a:00:27:00:00:1e (MNHAQIQI) via enp0s8
Oct 29 02:45:15 finalprojectku dhcpd[16734]: DHCPACK on 192.168.171.11 to 0a:00:27:00:00:1e (MNHAQIQI) via enp0s8
```
