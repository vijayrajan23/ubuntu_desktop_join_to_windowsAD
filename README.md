# ubuntu_desktop_join_to_windowsAD
## Step1
(setup hostname when install, like: ubuntu)
## Step2
Setup DNS WIND DOMAIn IP .
## Step3
ubuntu:~$  sudo apt update
ubuntu:~$  sudo apt-get install fping
## Step4
Make sure your Ubuntu Desktop machine has access to the Active Directory domain and the Domain Controllers:

ubuntu:~$ dig -t SRV _ldap._tcp.example.com | grep -A2 "ANSWER SECTION"

;; ANSWER SECTION: 

_ldap._tcp.bmbdlocal.lan. 600 IN SRV 0 100 389 server.example.com
## Step5
ping Active Directory domain and the Domain Controllers:

ubuntu:~$ ping example.com
ubuntu:~$ ping server.example.com

## Step6

ubuntu:~$ fping server.example.com

server.example.com is alive

## Step7
Install all necessary packages:

ubuntu:~$ sudo apt-get -y install realmd sssd sssd-tools samba-common krb5-user packagekit samba-common-bin samba-libs adcli ntp

after ask type:EXAMPLE.COM
then OK

## Step8
Setup your ntp service to point to our domain timeservers:

ubuntu:~$ sudo vim /etc/ntp.conf

...
add AD server
server.example.com
...

restart your ntp service:
ubuntu:~$ sudo systemctl restart ntp 

## Step9
Setting up realmd:
ubuntu:~$  sudo vim /etc/realmd.conf
...
[users]
default-home = /home/%D/%U
default-shell = /bin/bash
[active-directory]
default-client = sssd
os-name = Ubuntu Desktop Linux
os-version = 18.04
[service]
automatic-install = no
[example.com]
fully-qualified-names = no
automatic-id-mapping = yes
user-principal = yes
manage-system = no
...

## Step10
Join the Ubuntu machine on the AD domain:

ubuntu:~$ sudo kinit administrator@EXAMPLE.COM

Password for administrator@EXAMPLE.COM:

## Step11
Add the Ubuntu machine in the domain: 

ubuntu:~$ sudo realm --verbose join example.com \
--user-principal=ubuntuhostname/administrator@EXAMPLE.COM --unattended

## Step12
Setting up sssd:

ubuntu:~$ sudo vim /etc/sssd/sssd.conf

access_provider = ad

Restart the sssd service:

ubuntu:~$ sudo systemctl restart sssd 

## Step13

Setup homedir auto-creation for new users:

ubuntu:~$  sudo vim /etc/pam.d/common-session
...
session required pam_unix.so
session optional pam_winbind.so
session optional pam_sss.so
session optional pam_systemd.so
session required pam_mkhomedir.so skel=/etc/skel/ umask=0077
# end of pam-auth-update config
...

## Step14
Check Active Directory users name resolution:

ubuntu:~$ id domainuser@example.com

ubuntu:~$ id rock@example.com

uid=178201238(rock@EXAMPLE.COM) gid=178200513(domain users@EXAMPLE.COM) groups=178200513(domain users@EXAMPLE.COM)

## Step15

Setting up LightDM for CLI mode ubuntu or linux:[no need for ubuntu GUI]

ubuntu:~$ sudo vi /etc/lightdm/lightdm.conf
...
[SeatDefaults]
allow-guest=false
greeter-show-manual-login=true
...

Final Check:
Restart the machine and try to login using the Ubuntu graphical login by domain user and password.

note: In case it does not work as expected, setup previous steps properly.

