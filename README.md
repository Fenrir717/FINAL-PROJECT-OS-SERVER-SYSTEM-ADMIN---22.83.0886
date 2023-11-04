# FINAL-PROJECT-OS-SERVER-SYSTEM-ADMIN---22.83.0886

Repository ini berisi dokumentasi yang menjelaskan cara instalasi dan konfigurasi berbagai layanan server, seperti SSH, DHCP, FTP/Samba, DNS, mail server, dan web server, serta Database Server. Dokumen ini ditujukan untuk memandu pengguna dalam mengatur dan mengelola server-server ini dengan langkah-langkah yang jelas.

## Daftar Isi
1. [Instalasi dan Konfigurasi SSH](#1-instalasi-dan-konfigurasi-ssh-server)
2. [Instalasi dan Konfigurasi DHCP Server](#2-instalasi-dan-konfigurasi-dhcp-server)
3. [Instalasi dan Konfigurasi FTP dan Samba Server](#3-instalasi-dan-konfigurasi-ftp-dan-samba-server)
4. [Instalasi dan Konfigurasi Web Server](#4-instalasi-dan-konfigurasi-web-server)
5. [Instalasi dan Konfigurasi DNS Server](#5-instalasi-dan-konfigurasi-dns-server)
6. [Instalasi dan Konfigurasi Database Server](#6-instalasi-dan-konfigurasi-database-server)
7. [Instalasi dan Konfigurasi Mail Server](#7-instalasi-dan-konfigurasi-mail-server)
8. [Instalasi dan Konfigurasi Monitoring Server](#8-instalasi-dan-konfigurasi-monitoring-server)
   

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

![Login User](./Screenshot/2.png)

ini tampilan dari isi direkotori Samba yang diKonfigurasi sebelumnya :

![Direktori Samba](./Screenshot/3.png)

2. FTP Server
   ini adalah Tampilan dari FTP client(Menggunakan Filezila)

![Filezila Client FTP](./Screenshot/5.png)

## 4. Instalasi dan Konfigurasi Web Server
Saya memilih untuk menggunakan Apache2 sebagai server web utama untuk proyek saya daripada Nginx. tidak ada alasan khusus saya memilih apache2,anda bisa memilih Nginx. saya juga akan menginstall CMS wordpress untuk Webserver ini.Apache2 memiliki dukungan yang baik untuk PHP, yang merupakan salah satu persyaratan utama untuk menjalankan WordPress dengan lancar.

### 4.1 Instalasi Apache2

**Langkah 1: Instalasi Paket Apache2**
```
apt update
apt-get install apache2
```
### 4.2 Konfigurasi Apache2
**Langkah 1: Buka File Konfigurasi Apache2**
```
nano /etc/apache2/sites-available/000-default.conf
```
**Langkah 2: Sesuaikan Konfigurasi ini dengan domain yang anda gunakan**
```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@finalprojectku.com
        ServerName finalprojectku.com
        DocumentRoot /var/www/html
```
**Lengkah 3: Restart Layanan Apache2**
```
systemctl restart apache2
```
**Langkah 4: Cek Apache2**

Jika Konfigurasi Berhasil seharusnya muncul layanan web default seperti gambar dibawah ini:

![Tampilan Default Apache2](./Screenshot/6.png)

### 4.3 Konfigurasi CMS Wordpress pada Apache2

**Langkah 1: Melakukan Instalasi PHP**
```
apt-get update
apt-get install php php-mysql
```
**Langkah 2: Melakukan Instalasi Database Server**

Untuk ini sudah dibuat bab lain jadi anda bisa cek ke [Instalasi Database Server](#6-instalasi-dan-konfigurasi-database-server)

**Langkah 3: Buat Database untuk wordpress**

login ke Database Terlebih dahulu:
```
mysql -u root -p
```
buat database untuk Wordpress dan berikan password sesuai keinginan anda
```
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
**Langkah 4: Download dan Extract Paket Wordpress**
```
curl -O https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```
**Langkah 5: Pindahkan isi direktori Wordpress**

saya menyimpan file unduhan di /tmp,jadi diseusikan lokasi nya
```
mv /tmp/wordpress/* /var/www/html/
```
**Langkah 6: Copy file Konfigurasi utama wordpress**
```
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```
Buka Kofigurasi file nya
```
nano /var/www/html/wp-config.php
```
**Langkah 7: Sesuikan isi Database nya dengan Database Wordpress yang telah dibuat**
```
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'wordpressuser' );

/** Database password */
define( 'DB_PASSWORD', '1' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );
```
**Langkah 8: Set hak akses**
```
chown -R www-data:www-data /var/www/html
```
**Langkah 9: Instalasi Admin Wordpress**

1. Pilih Bahasa yang diinginkan 
![Pilih Bahasa](./Screenshot/9.png)

2. Isi Nama situs,Password,dan Email
![Setting Akun](./Screenshot/10.png)

3. Login dengan Akun yang sudah dibuat
![Login](./Screenshot/11.png)

4. CMS sudah Berhasil diinstall
![CMS](./Screenshot/12.png)


### 4.4 Pengujian Konfigurasi Apache2

Tampilan dari Postingan yang ada di Webserver

![Contoh Postingan](./Screenshot/14.png)

## 5. Instalasi dan Konfigurasi DNS Server

### 5.1 Instalasi BIND9

**Langkah 1: Instalasi Paket bind9**
```
apt update
apt-get install bind9
```
### 5.2 Konfigurasi BIND9

**Langkah 1: copy file untuk Konfigurasi "Forward" dan "Reverse"**
```
cd /etc/bind
root@finalprojectku:/etc/bind# root@finalprojectku:/etc/bind# ls
bind.keys  db.127  db.empty  named.conf                named.conf.local    rndc.key
db.0       db.255  db.local  named.conf.default-zones  named.conf.options  zones.rfc1918
cp db.local db.forward
cp db.127 db.reverse
```
**Langkah 2: Konfigurasi file db.forward**
```
nano db.forward
```
Ubahlah Konfigurasi seperti dibawah ini:
```
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     finalprojectku.com. root.finalprojectku.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      finalprojectku.com.
@       IN      A       192.168.171.10
ns      IN      A       192.168.171.10
www     IN      A       192.168.171.10
mail    IN      A       192.168.171.10
@       IN      MX      10 finalprojectku.com.
```
disesuaikan dengan domain anda

**Langkah 3: Konfigurasi file db.reverse**
```
nano db.reverse
```
ubahlah Konfigurasi seperti dibawah ini:
```
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     finalprojectku.com. root.finalprojectku.com. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      finalprojectku.com.
10      IN      PTR     finalprojectku.com.
```

**Langkah 4: Buka Konfigurasi named.conf.local untuk konfigurasi DNS Zones**
```
nano named.conf.local
```
Ubahlah isi file konfigurasi seperti dibawah ini:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "finalprojectku.com" {
        type master;
        file "/etc/bind/db.forward";
};
zone "171.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.reverse";
};
```
**Langkah 5: Konfigurasi Forwarders**
```
nano named.conf.options
```
tambahkan DNS forwarders
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
        //      192.168.171.10;
        //      8.8.8.8;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        listen-on-v6 { any; };
};
```
**Langkah 6: Konfigurasi DNS diperangkat Server**
```
nano /etc/resolv.conf
```
ubahlah jadi seperti ini:
```
domain finalprojectku.com
search finalprojectku.com
nameserver 192.168.171.10
nameserver 8.8.8.8
```
**Langkah 7: Restart Layanan Bind9**
```
systemctl restart bind9
```

### 5.3 Pengujian Konfigurasi DNS

**Langkah 1: Instalasi paket dns resolver**
```
apt-get install dnsutils
```
1. Lakukan Nslookup untuk menguji Domain kita:
```
root@finalprojectku:/etc/bind# nslookup finalprojectku.com
Server:         192.168.171.10
Address:        192.168.171.10#53

Name:   finalprojectku.com
Address: 192.168.171.10

root@finalprojectku:/etc/bind# nslookup 192.168.171.10
10.171.168.192.in-addr.arpa     name = finalprojectku.com.
```
ini menunjukkan bahwa Konfigurasi DNS untuk mengubah alamat domain ke IP dan sebaliknya(foward&reverse) sudah berjalan dengan baik

2. Kita uji pada sisi Client:
![Ini Pengujian DNS di Client](./Screenshot/7.png)

## 6. Instalasi dan Konfigurasi Database Server
Dalam proyek ini, saya melakukan instalasi database server menggunakan MariaDB, sebuah sistem manajemen basis data open-source yang kuat. MariaDB digunakan untuk menyimpan dan mengelola data yang diperlukan oleh aplikasi dan situs web. Saya juga menginstal phpMyAdmin, antarmuka web yang memudahkan administrasi dan manajemen database MariaDB, memungkinkan pengguna untuk dengan mudah membuat, mengedit, dan mengelola basis data melalui antarmuka berbasis web yang intuitif.

### 6.1 Instalasi MariaDB

**Langkah 1: Instalasi Paket MariaDB**
```
sudo apt-get update
sudo apt-get install mariadb-server
```
### 6.2 Konfigurasi MariaDB
**Langkah 1: Jalankan Perintah ini**
```
mysql_secure_installation
```
**Langkah 2: Ikuti Konfigurasi dibawah ini**

Anda Bisa Mengikuti Langkah-Langkah ini:
```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
Bagian "Disallow root login remotely" bisa diatur seusia keperluan,karena disitu saya perlu login dengan root account.

### 6.3 Instalasi dan Konfigurasi Phpmyadmin

**Langkah 1: Lakukan instalasi paket**
```
apt-get install phpmyadmin
```
**Langkah 2: Konfigurasi Phpmyadmin**

1. Bisa langsung Konfigurasi otomatis agar bisa ditampilkan ke Web Server anda
   
![phpmyadmin](./Screenshot/8.png)

2. atau bisa Menggunakan Command Line
```
ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
a2enconf phpmyadmin
systemctl reload apache2
```

 ### 6.4 Menguji Konfigurasi 

berhasil dibuka melalui Webserver Apache2
![Phpmyadmin](./Screenshot/13.png)

## 7. Instalasi dan Konfigurasi Mail Server
Saya akan mendokumentasikan tentang instalasi dan mengonfigurasi mail server pada sistem Linux Debian menggunakan postfix sebagai MTA (Mail Transfer Agent) dan dovecot sebagai server IMAP/POP3. Selanjutnya, saya akan mengintegrasikan mail server ini dengan webmail Roundcube. Roundcube adalah antarmuka web yang memudahkan pengguna mengakses dan mengelola email melalui browser, memberikan kenyamanan dan akses cepat tanpa perlu klien email desktop.

 ### 7.1 Instalasi Postfix dan Dovecot

 **Langkah 1: Instalasi paket Postfix**
 ```
 apt-get install -y postfix sasl2-bin
```
**Langkah 2: Pilih tergantung kebutuhan(saya memilih no configuration karena akan dikonfigurasi secara manual**
![postfix](./Screenshot/15.png)

**Langkah 3: Instalasi paket Dovecot**

```
 apt-get install -y dovecot-core dovecot-pop3d dovecot-imapd
```

### 7.2 Konfigurasi Postfix

**Langkah 1: Rename file dan copy file Konfigurasi utama Postfix**
```
cp /usr/share/postfix/main.cf.dist /etc/postfix/main.cf
```
**Langkah 2: Membuka Direktori Konfigurasi utama Postfix**
```
nano /etc/postfix/main.cf
```

**Langkah 3: Lakukan Perubahan Konfigurasi sebagai berikut**
```
#Hilangkan tanda "#" dan lakukan Perubahan Konfigurasi seperti dibawah ini

#line 82 : Hilangkan tanda pagar
mail_owner = postfix

#line 98 : Hilangkan tanda pagar dan ganti domain nya sesuai domain name anda
myhostname = finalprojectku.com

#line 106 :Hilangkan tanda pagar dan ganti domain nya sesuai domain name anda
mydomain = finalprojectku.com

#line 127 : Hilangkan tanda pagar
myorigin = $mydomain

#line 141 : Hilangkan tanda pagar
inet_interfaces = all

#line 189 : Hilangkan tanda pagar
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

#line 232 : Hilangkan tanda pagar
local_recipient_maps = unix:passwd.byname $alias_maps

#line 277 : Hilangkan tanda pagar
mynetworks_style = subnet

#line 294 : Tambahkan subnet Jaringan anda
mynetworks = 127.0.0.0/8, 192.168.171.0/24

#line 416 : Hilangkan tanda pagar
alias_maps = hash:/etc/aliases

#line 427 : Hilangkan tanda pagar
alias_database = hash:/etc/aliases

#line 449 : Hilangkan tanda pagar
home_mailbox = Maildir/

#line 585: Tambahkan tanda pagar 
#smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)

#line 659 : Tambahkan command ini
sendmail_path = /usr/sbin/postfix

#line 664 : Tambahkan command ini
newaliases_path = /usr/bin/newaliases

#line 669 : Tambahkan command ini
mailq_path = /usr/bin/mailq

#line 675 : Tambahkan command ini
setgid_group = postdrop

#line 679 : tambahkan tanda pagar
#html_directory =

#line 683 : tambahkan tanda pagar
#manpage_directory =

#line 688 : tambahkan tanda pagar
#sample_directory =

#line 692 : tambahkan tanda pagar
#readme_directory =

#line 692 : agar listen Ipv4 jika ingin ipv4 maka ubah ke(all)
inet_protocols = ipv4

# Tambahkan dibaris akhir
# menonaktifkan SMTP VRFY untuk mengamankan mail server
disable_vrfy_command = yes
```
**Langkah 4: Restart Layanan Postfix**
```
systemctl restart postfix
```

### 7.3 Konfigurasi Dovecot

**Langkah 1: Buka file Konfigurasi utama dovecot**
```
nano /etc/dovecot/dovecot.conf
```
**Langkah 2: Lakukan Perubahan seperti dibawah ini**
```
#line 30 : Hilangkan tanda pagar
listen = *, ::
```

**Langkah 3: Buka file Konfigurasi dibawah ini**
```
nano /etc/dovecot/conf.d/10-auth.conf
```
**Langkah 4: Lakukan Perubahan Konfigurasi**
```
#line 10 : Hilangkan tanda pagar dan Tambahkan
disable_plaintext_auth = no

#line 100 : Tambahkan command
auth_mechanisms = plain login
```
**Langkah 5: Buka File Konfigurasi**
```
 nano /etc/dovecot/conf.d/10-mail.conf
```
**Langkah 6: Lakukan Perubahan Konfigurasi**
```
#line 30 : Ubah lokasi nya ke Maildir
mail_location = maildir:~/Maildir
```

**Langkah 7: Buka File Konfigurasi**
```
nano /etc/dovecot/conf.d/10-master.conf
```
**Langkah 8: Lakukan Perubahan Konfigurasi**
```
#Hilangkan tanda pagar
service imap-login {
  inet_listener imap {
    port = 143
  }

#line 107-109 : hilangkan tanda pagar dan tambahkan
  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }
```

**Langkah 9: Restart Layanan Dovecot**
```
systemctl restart dovecot
```

### 7.4 Konfigurasi Webmail Roundcube

**Langkah 1: Download dan extract File Roundcube**
```
wget https://github.com/roundcube/roundcubemail/releases/download/1.6.4/roundcubemail-1.6.4-complete.tar.gz
tar xzvf roundcubemail-1.6.4-complete.tar.gz
```

**Langkah 2: Buat Folder di var/ww/html**
```
mkdir -p /var/www/html/roundcube
```
**Langkah 3: Pindahkan isi direktori roundcube**
saya menyimpan file unduhan di /tmp,jadi diseusikan lokasi nya
```
mv /tmp/roundcubemail-1.6.4/* /var/www/html/roundcube/
```

**Langkah 4: install php-fpm**
```
apt-get install -y php-fpm
```

**Langkah 5: Buat Database User untuk Roundcube**
(Bisa disesuaikan ini hanya contoh)
```
mysql -u root
create database roundcubemail;
CREATE USER roundcube@localhost IDENTIFIED BY '1';
grant all privileges on roundcubemail.* to roundcube@'localhost' identified by '1';
flush privileges;
exit;
```
atau jika user roundcube belum mendapat akses ke database bisa menggunakan
```
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE ON roundcubemail.* TO 'roundcube'@'localhost';
```
**Langkah 6: Eksekusi Command ini**
```
 mysql roundcubemail < /var/www/html/roundcube/SQL/mysql.initial.sql
```
**Langkah 7: Buka direktori utama Roundcubemail**
```
cd /var/www/html/roundcube/config/
```
**Langkah 8: Copy file default konfigurasi**
```
cp config.inc.php.sample config.inc.php
```
**Langkah 9: Buka File Konfigurasi**
```
nano config.inc.php
```

**Langkah 9: Ubah isi Konfigurasi**
```
dari ini
<?php

/*
 +-----------------------------------------------------------------------+
 | Local configuration for the Roundcube Webmail installation.           |
 |                                                                       |
 | This is a sample configuration file only containing the minimum       |
 | setup required for a functional installation. Copy more options       |
 | from defaults.inc.php to this file to override the defaults.          |
 |                                                                       |
 | This file is part of the Roundcube Webmail client                     |
 | Copyright (C) The Roundcube Dev Team                                  |
 |                                                                       |
 | Licensed under the GNU General Public License version 3 or            |
 | any later version with exceptions for skins & plugins.                |
 | See the README file for a full license statement.                     |
 +-----------------------------------------------------------------------+
*/

$config = [];

// Database connection string (DSN) for read+write operations
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
// Currently supported db_providers: mysql, pgsql, sqlite, mssql, sqlsrv, oracle
// For examples see http://pear.php.net/manual/en/package.database.mdb2.intro-dsn.php
// NOTE: for SQLite use absolute path (Linux): 'sqlite:////full/path/to/sqlite.db?mode=0646'
//       or (Windows): 'sqlite:///C:/full/path/to/sqlite.db'
$config['db_dsnw'] = 'mysql://roundcube:pass@localhost/roundcubemail';

// IMAP host chosen to perform the log-in.
// See defaults.inc.php for the option description.
$config['imap_host'] = 'localhost:143';

// SMTP server host (for sending mails).
// See defaults.inc.php for the option description.
$config['smtp_host'] = 'localhost:587';

// SMTP username (if required) if you use %u as the username Roundcube
// will use the current username for login
$config['smtp_user'] = '%u';

// SMTP password (if required) if you use %p as the password Roundcube
// will use the current user's password for login
$config['smtp_pass'] = '%p';

// provide an URL where a user can get support for this Roundcube
$config['support_url'] = '';

// Name your service. This is displayed on the login screen and in the window title
$config['product_name'] = 'Roundcube Webmail';

// This key is used to encrypt the users imap password which is stored
// in the session record. For the default cipher method it must be
// exactly 24 characters long.
// YOUR KEY MUST BE DIFFERENT THAN THE SAMPLE VALUE FOR SECURITY REASONS
$config['des_key'] = 'rcmail-!24ByteDESkey*Str';

// List of active plugins (in plugins/ directory)
$config['plugins'] = [
    'archive',
    'zipdownload',
];

// skin name: folder from skins/
$config['skin'] = 'elastic';
```
Ubah menjadi seperti ini dan sesuaikan dengan database anda

```
<?php

/*
 +-----------------------------------------------------------------------+
 | Local configuration for the Roundcube Webmail installation.           |
 |                                                                       |
 | This is a sample configuration file only containing the minimum       |
 | setup required for a functional installation. Copy more options       |
 | from defaults.inc.php to this file to override the defaults.          |
 |                                                                       |
 | This file is part of the Roundcube Webmail client                     |
 | Copyright (C) The Roundcube Dev Team                                  |
 |                                                                       |
 | Licensed under the GNU General Public License version 3 or            |
 | any later version with exceptions for skins & plugins.                |
 | See the README file for a full license statement.                     |
 +-----------------------------------------------------------------------+
*/

$config['enable_installer'] = false;

// Database connection string (DSN) for read+write operations
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
// Currently supported db_providers: mysql, pgsql, sqlite, mssql, sqlsrv, oracle
// For examples see http://pear.php.net/manual/en/package.database.mdb2.intro-dsn.php
// NOTE: for SQLite use absolute path (Linux): 'sqlite:////full/path/to/sqlite.db?mode=0646'
//       or (Windows): 'sqlite:///C:/full/path/to/sqlite.db'
$config['db_dsnw'] = 'mysql://roundcube:1@localhost/roundcubemail';

// IMAP host chosen to perform the log-in.
// See defaults.inc.php for the option description.
$config['imap_host'] = 'finalprojectku.com:143';

// SMTP server host (for sending mails).
// See defaults.inc.php for the option description.
$config['smtp_host'] = 'finalprojectku:25';

// SMTP username (if required) if you use %u as the username Roundcube
// will use the current username for login
$config['smtp_user'] = '%u';

// SMTP password (if required) if you use %p as the password Roundcube
// will use the current user's password for login
$config['smtp_pass'] = '%p';

// provide an URL where a user can get support for this Roundcube installation
$config['support_url'] = '';

// Name your service. This is displayed on the login screen and in the window title
$config['product_name'] = 'Roundcube Webmail';

// This key is used to encrypt the users imap password which is stored
// in the session record. For the default cipher method it must be
// exactly 24 characters long.
// YOUR KEY MUST BE DIFFERENT THAN THE SAMPLE VALUE FOR SECURITY REASONS
$config['des_key'] = 'rcmail-!24ByteDESkey*Str';

// List of active plugins (in plugins/ directory)
$config['plugins'] = [
    'archive',
    'zipdownload',
];

// skin name: folder from skins/
$config['skin'] = 'elastic';
$config['smtp_server'] = 'localhost';
$config['smtp_user'] = '';
$config['smtp_pass'] = '';
```
**Langkah 10: Restart semua layanan**
```
systemctl apache2 postfix dovecot
```
**Langkah 11: Membuat contoh user untuk test Email**
saya akan membuat 2 user untuk saling mengirim email
```
root@finalprojectku:~# adduser fenrir717
Adding user `fenrir717' ...
Adding new group `fenrir717' (1004) ...
Adding new user `fenrir717' (1004) with group `fenrir717 (1004)' ...
Creating home directory `/home/fenrir717' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for fenrir717
Enter the new value, or press ENTER for the default
        Full Name []: fenrir
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
Adding new user `fenrir717' to supplemental / extra groups `users' ...
Adding user `fenrir717' to group `users' ...
root@finalprojectku:~# adduser byul
Adding user `byul' ...
Adding new group `byul' (1005) ...
Adding new user `byul' (1005) with group `byul (1005)' ...
Creating home directory `/home/byul' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for byul
Enter the new value, or press ENTER for the default
        Full Name []: yi byul
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
Adding new user `byul' to supplemental / extra groups `users' ...
Adding user `byul' to group `users' ...
```
**Langkah 12: Menyelesaikan salah satu Problem yang sering terjadi**

Terkadang terdapat masalah yaitu Konfigurasi Postfix karena menggunakan 
```
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
```
ini jika belum diKonfigurasi lebih lanjut bisa menyebabkan masalah error seperti
"SMTP Error: Failed to add recipient : Temporary lookup failure (Code: 451)"

Peyebabnya
"
Penyebab utama masalah ini adalah pengaturan konfigurasi Postfix yang menggunakan tipe database hashing (hash) untuk berkas alias. Tipe ini memerlukan berkas database yang sesuai dengan konfigurasi alias, dan berkas tersebut harus dibangun atau diperbarui saat konfigurasi alias berubah."

Solusi?
```
newaliases
```
atau
```
postalias
```
setelah itu
```
postfix reload
```
Perintah ini membangun atau memperbarui berkas database alias, seperti /etc/aliases.db, sesuai dengan konfigurasi alias yang ada.

### 7.5 Menguji Konfigurasi

Kita akan menguji Konfigurasi,untuk melakukan test mengirim email di 2 user yang dibuat sebelumnya:

1. Login Ke Account yang dibuat
![Login Roundcube](./Screenshot/16.png)

2. Mengirim Email dari user fenrir717@finalprojectku.com 
![sent mail 1](./Screenshot/17.png)

3. Pesan sudah berhasil masuk ke user byul@finalprojectku.com
![indbox email byul](./Screenshot/18.png)

4. Sekarang kita kirim Email dari user byul@finalprojectku.com ke Fenrir717
![sent mail 2](./Screenshot/19.png)

5. Pesan sudah berhasil masuk ke user Fenrir717
![indbox email fenrir](./Screenshot/20.png)

**Dengan begini Konfigurasi Webmail Roundcube dengan MTA Postfix dan Dovecot sudah Berhasil Berjalan**

## 8. instalasi-dan-konfigurasi-monitoring-server

Saya akan melakukan instalasi dan konfigurasi monitoring server yang terdiri dari dua tahap utama. Tahap pertama mencakup pemantauan sumber daya server seperti CPU, memory, dan lainnya menggunakan Prometheus dan Grafana. Tahap kedua adalah log monitoring dengan menggunakan Promtail dan Loki. Dengan alat-alat ini, saya akan memastikan server berjalan dengan baik, serta melacak dan menganalisis log untuk deteksi masalah dan pemecahan masalah yang lebih baik.

### 8.1 Instalasi Prometheus,node-exporter,Loki,promtail dan Grafana

**Langkah 1: Instalasi paket Prometheus dan Node-exporter**
```
apt-get install prometheus prometheus-node-exporter
```
**Langkah 2: Instalasi Paket Grafana**
```
wget -q -O /usr/share/keyrings/grafana.key https://packages.grafana.com/gpg.key
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/oss/deb stable main" | tee -a /etc/apt/sources.list.d/grafana.list
apt-get update
apt-get install grafana
```
**Langkah 3: Instalasi paket Loki dan Promtail** 
```
apt-get update
apt-get install loki
apt-get install promtail
```
optinal(jika ingin logging nya lebih mudah)
```
apt-get install rsyslog
```

### 8.2 Konfigurasi Prometheus

**Langkah 1: Direktori utama prometheus**
```
nano /etc/prometheus/prometheus.yml
```
**Langkah 2: Tambahkan Monitoring target untuk membuat metrics dari node-exporter**
```
#menggunakan port default yaitu 9100,anda bisa menganti ini
  - job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100']
```

### 8.3 Konfigurasi Grafana untuk Visualisasi data Resource dari Node Exporter

**Langkah 1: Buka Konfigurasi utama Grafana**
```
nano /etc/grafana/grafana.ini
```

**Langkah 2: Edit Konfigurasi ini**
```
[server]

#hilangkan tanda ";"
;protocol = http

# This is the minimum TLS version allowed. By default, this value is empty. Accepted values are: TLS1.2, TLS1.3. If not>;min_tls_version = ""

# The ip address to bind to, empty will bind to all interfaces
;http_addr =

#hilangkan tanda ";" jika ingin ganti port
;http_port = 3000

#hilangkan tanda ";" jika ingin diakses dengan domain/sub domain tertentu
;domain = localhost

# Redirect to correct domain if host header does not match domain
# Prevents DNS rebinding attacks
;enforce_domain = false
```




