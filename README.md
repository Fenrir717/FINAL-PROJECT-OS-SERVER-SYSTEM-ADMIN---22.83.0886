# FINAL-PROJECT-OS-SERVER-SYSTEM-ADMIN---22.83.0886

Repository ini berisi dokumentasi yang menjelaskan cara instalasi dan konfigurasi berbagai layanan server, seperti SSH, DHCP, FTP/Samba, DNS, mail server, dan web server, serta Database Server. Dokumen ini ditujukan untuk memandu pengguna dalam mengatur dan mengelola server-server ini dengan langkah-langkah yang jelas.

## Daftar Isi
1. [Instalasi dan Konfigurasi SSH](#1-instalasi-dan-konfigurasi-ssh-server)
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
### 1.3 Menguji Konfigurasi
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
### 2.3 Menguji Konfigurasi

1.Dari sisi Client anda bisa melakukan Request IP ke DHCP POOL(saya menggunakan Windows sebagai Client)

![DHCP Client](./Screenshot/1.png)

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
## 3. Instalasi dan Konfigurasi FTP dan Samba Server
Samba dan FTP (vsftpd) adalah dua layanan yang memiliki fungsi serupa, yaitu memungkinkan pertukaran file dan berbagi sumber daya di jaringan. Keduanya digunakan untuk mentransfer file antara komputer dalam jaringan,Karena itu saya akan Mendokumentasikanya dalam 1 bab saja
### 3.1 Instalasi FTP dan Samba
**Langkah 1: Instalasi Paket samba**
```
apt update
apt-get install samba
```
**Langkah 2: Instalasi Paket VSFTPD(FTP)y**
Sepertinya ada banyak pilihan seperti "PROFTPD" tapi disini saya menggunakan VSFTPD
```
apt update
apt-get install vsftpd
```

### 3.2 Konfigurasi Samba

**Langkah 1: Lakukan Backup Konfigurasi Default**
```
cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
```
**Langkah 2: Hapus isi Konfigurasi didalam file samba.conf**
```
echo "" | tee /etc/samba/smb.conf
```
**Langkah 3: Buka file konfigurasi Utama Samba**
```
nano /etc/samba/smb.conf
```
**Langkah 4: isi file Konfigurasi**
```
[anonymous]
path = /home/public
public = yes
writable = yes
guest ok = yes
guest only = yes

#=========================================================================#
[Fenrir Group]
path = /home/private
public = no
writeable = yes
valid user = fenrir_group
```
Dalam Sekenario ini,saya membuat 2 Direktori yaitu:

public = untuk user umum

private = untuk user group tertentu

**Langkah 5: Membuat Direktori dan Konfigurasi User**
```
mkdir /home/public
mkdir /home/private
groupadd fenrir_group
useradd -G fenrir_group fenrir
smbpasswd -a fenrir
New SMB password:
Retype new SMB password:
Added user fenrir.
```
**Langkah 6: Set permission untuk kedua Direktori**
```
chmod 777 /home/public
chmod 770 /home/private
chown :fenrir_group /home/private
```
**Langkah 7: Restart Service SMB**
```
systemctl restart smbd
```
### 3.3 Konfigurasi VSFTPD(FTP)
**Langkah 1: Buka File Konfigurasi Utama Vsftpd**
```
nano /etc/vsftpd.conf
```
**Langkah 2 : Lakukan Perubahan Konfigurasi berikut :**
```
listen=YES
listen_ipv6=NO
anonymous_enable=NO
#HILANGKAN TANDA PAGAR PADA LINE KONFIGURASI DIBAWAH INI#
write_enable=YES
local_umask=022
ascii_upload_enable=YES
ascii_download_enable=YES
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
ls_recurse_enable=YES
```
**Langkah 3: Menambahkan User baru dan Memasukanya ke list chroot**
Menambahkan user baru
```
useradd black_lotus
passwd black_lotus
New password:
Retype new password:
passwd:
password updated successfully
```
menambahkan user baru ke list 
```
nano /etc/vsftpd.chroot_list
#Tambahkan user baru di direktori ini
black_lotus
```
**Langkah 4: Restart Layanan Vsftpd**
```
systemctl restart vsftpd
```
### 3.4 Pengujian Konfigurasi

1. Samba Server
  ini adalah pengujian dari sisi client

Gambar 2

Gambar 3

2. FTP Server
   ini adalah Tampilan dari FTP client(Menggunakan Filezila)

   Gambar 4






