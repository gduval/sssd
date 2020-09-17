Run Ansible playbook --or-- follow manual steps below


The following is based on https://sssd.io/ and https://github.com/SSSD/sssd





### Step 1: Install all necessary packages with the following commands:

 

sudo apt update

sudo apt upgrade

sudo apt install sssd heimdal-clients msktutil


### Step 2: Move the default Kerberos configuration file, and create a new file:



sudo mv /etc/krb5.conf /etc/krb5.conf.default

sudo vi /etc/krb5.conf

The new file should contain the following contents:

libdefaults]
default_realm = {DOMAIN_NAME}
rdns = no
dns_lookup_kdc = true
dns_lookup_realm = true

[realms]
NOTS.LOCAL = {
kdc = {DOMAIN_SERVER}
admin_server = {DOMAIN_SERVER}
}

### Step 3: Initialize Kerberos and generate a keytab file:


kinit administrator

klist

msktutil -N -c -b 'CN=COMPUTERS' -s $HOSTNAME/$HOSTNAME.prod.mco -k my-keytab.keytab --computer-name $HOSTNAME --upn $HOSTNAME$ --server {DOMAIN_SERVER} --user-creds-only

msktutil -N -c -b 'CN=COMPUTERS' -s $HOSTNAME/$HOSTNAME -k my-keytab.keytab --computer-name $HOSTNAME --upn $HOSTNAME$ --server {DOMAIN_SERVER} --user-creds-only

kdestroy

### Step 4: Configure SSSD:


sudo mv my-keytab.keytab /etc/sssd/my-keytab.keytab

sudo nano /etc/sssd/sssd.conf



After saving, set the appropriate permissions on that file:


sudo chmod 0600 /etc/sssd/sssd.conf

### Step 5: Configure PAM:



sudo nano /etc/pam.d/common-session

Find the line that contains "session required pam_unix.so" near the bottom of the file. (It may not show up with a search due to weird spacing.) Add the following line immediately below it:



session required pam_mkhomedir.so skel=/etc/skel umask=0077

After saving & exiting, restart SSSD:



sudo systemctl restart sssd
