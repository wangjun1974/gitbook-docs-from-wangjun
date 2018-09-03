
## Overcloud Certificate Creation




### What tls-prepare-certs does

* SET SOME VARIABLES WE ARE GOING TO NEED
* CREATING A CERTIFICATE AUTHORITY
* ADDING THE CERTIFICATE AUTHORITY TO CLIENTS
* CREATING AN SSL/TLS KEY
* GENERATE CERTIFICATE SIGNING REQUEST
* CREATING THE SSL/TLS CERTIFICATE

### What tls-prepare-barbican-certs.sh does

* CREATING A CERTIFICATE AUTHORITY
* ADDING THE CERTIFICATE AUTHORITY TO CLIENTS
* CREATING AN Security admin KEY
* CREATING THE SSL/TLS CERTIFICATE
* CREATING AN cinder-server KEY
* CREATING THE SSL/TLS CERTIFICATE
* CREATING AN nova-server KEY
* CREATING THE SSL/TLS CERTIFICATE

### file tls-prepare-certs.sh
```
#!/bin/bash
cd ~
source ~/stackrc
unset OS_CACERT
FILE="/home/stack/index.txt"
if [ -f $FILE ]; then
  rm -f "$FILE"
  touch $FILE
else
  touch $FILE
fi
echo 1000 > serial
echo 1000 > crlnumber
#set some variables we are going to need
#OCVIP=`/usr/share/cbis/undercloud/tools/yaml_helper.py -s user_config.yaml -f CBIS:subnets:external:ip_range_start`
OCVIP="2001:db8:0:0:0:0:0:84"
#sed -i "s/192.168.136.84/'$OCVIP'/" openssl.cnf
#CREATING A CERTIFICATE AUTHORITY
sudo rm -f *.pem
openssl genrsa -out ca.key.pem 4096
openssl req -key ca.key.pem -new -x509 -days 7300 -extensions v3_ca -out ca.crt.pem -subj "/C=FI/ST=Finland/L=Espoo/O=Nokia/CN=$OCVIP"
#ADDING THE CERTIFICATE AUTHORITY TO CLIENTS
sudo cp ca.crt.pem /etc/pki/ca-trust/source/anchors/
# Removing the below line as we're no longer using certifi. just in case, leaving it marked out.
#sudo cp /home/stack/ca.crt.pem /usr/lib/python2.7/site-packages/certifi/cacert.pem
sudo update-ca-trust extract
#CREATING AN SSL/TLS KEY
openssl genrsa -out server.key.pem 4096
# generate certificate signing request (server.csr.pem):
openssl req -config openssl.cnf -key server.key.pem -new -out server.csr.pem -subj "/C=FI/ST=Finland/L=Espoo/O=Nokia/CN=$OCVIP"
# CREATING THE SSL/TLS CERTIFICATE
openssl ca -config openssl.cnf -extensions v3_req -days 3650 -in server.csr.pem -out server.crt.pem -cert ca.crt.pem -keyfile ca.key.pem -batch
# Done
```


### file tls-prepare-barbican-certs.sh
```
#!/usr/bin/env bash


cd ~
source ~/stackrc
unset OS_CACERT
FILE="/home/stack/index.txt"
#OCVIP=`/usr/share/cbis/undercloud/tools/yaml_helper.py -s user_config.yaml -f CBIS:subnets:external:ip_range_start`
OCVIP="2001:db8:0:0:0:0:0:84"
if [ ! -f $FILE ]; then
    touch $FILE
    echo 1000 > serial
    echo 1000 > crlnumber
    #set some variables we are going to need
    sed -i "s/replacethis/'$OCVIP'/" openssl.cnf
else
    touch $FILE
fi

#CREATING A CERTIFICATE AUTHORITY
sudo rm -f ca-internal.key.pem
openssl genrsa -out ca-internal.key.pem 4096
openssl req -key ca-internal.key.pem -new -x509 -days 7300 -extensions v3_ca -out ca-internal.crt.pem -subj "/C=FI/ST=Finland/L=Espoo/O=Nokia/OU=Internal
/CN=$OCVIP"
#ADDING THE CERTIFICATE AUTHORITY TO CLIENTS
sudo cp ca-internal.crt.pem /etc/pki/ca-trust/source/anchors/

sudo update-ca-trust extract



#CREATING AN Security admin KEY
openssl genrsa -out secadmin.key 4096
# generate certificate signing request (server.csr.pem):
openssl req -config openssl.cnf -key secadmin.key -new -out secadmin.csr -subj "/C=FI/ST=Finland/L=Espoo/O=Nokia/CN=Security admin"
# CREATING THE SSL/TLS CERTIFICATE
openssl ca -config openssl.cnf -extensions v3_req -days 3650 -in secadmin.csr -out secadmin.pem -cert ca-internal.crt.pem -keyfile ca-internal.key.pem -b
atch

#CREATING AN cinder-server KEY
openssl genrsa -out cinder-server.key 4096
# generate certificate signing request (server.csr.pem):
openssl req -config openssl.cnf -key cinder-server.key -new -out cinder-server.csr -subj "/C=FI/ST=Finland/L=Espoo/O=Nokia/CN=Cinder server"
# CREATING THE SSL/TLS CERTIFICATE
openssl ca -config openssl.cnf -extensions v3_req -days 3650 -in cinder-server.csr -out cinder-server.pem -cert ca-internal.crt.pem -keyfile ca-internal.
key.pem -batch

#CREATING AN nova-server KEY
openssl genrsa -out nova-server.key 4096
# generate certificate signing request (server.csr.pem):
openssl req -config openssl.cnf -key nova-server.key -new -out nova-server.csr -subj "/C=FI/ST=Finland/L=Espoo/O=Nokia/CN=Nova server"
# CREATING THE SSL/TLS CERTIFICATE
openssl ca -config openssl.cnf -extensions v3_req -days 3650 -in nova-server.csr -out nova-server.pem -cert ca-internal.crt.pem -keyfile ca-internal.key.
pem -batch
```
