# docker setup pre-requisites
## RHEL/CentOS/RockyLinux 8
### install rpm
```
yum update
yum install curl git vim make docker
```

### download docker-compose
```
curl -s https://api.github.com/repos/docker/compose/releases/latest | grep browser_download_url  | grep docker-compose-linux-x86_64 | cut -d '"' -f 4 | wget -qi -
chmod +x docker-compose-linux-x86_64
sudo mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
```

### add user to group **docker**
```
newgrp docker
sudo usermod -aG docker $USER
```

### deploy asterisk docker image
```
git clone https://github.com/mlan/docker-asterisk.git
cd demo
```
add volum for asterisk config
```
sudo chmod 775 -R /data/asterisk-conf/
sudo chown -R $USER:docker /data/asterisk-conf/
docker volume create --driver local \
     --opt type=none \
     --opt device=/data/asterisk-conf \
     --opt o=bind asterisk-conf
```

```
$vim docker-compose.yml
volumes:
    asterisk-conf:
        external: true
```

# run the Asterisk container
## start asterisk
run docker compose
`docker-compose up -d`

interactive with ***Asterisk CLI***
`docker-compose exec tele asterisk -rvvvddd`

## setup pjsip.conf
replace `pjsip.conf` to `/data/asterisk-conf/etc/asterisk/pjsip.conf`

### NAT setup
`local_net=[DOCKER_INET/MASK_LEN]`
`external_media_address=[EXTERNAL_ADDRESS]`
`external_signaling_address=[EXTERNAL_ADDRESS]`

### endpoint setup
#### add/modify endpoint
```
[ENDPOINT NAME](endpoint_template)
auth=[AUTH NAME]
aors=[AOR NAME]
```
#### add/modify aor
```
[AOR NAME]
type=aor
max_contacts=1
```
#### add/modify auth
```
[AUTH NAME]
type=auth
auth_type=userpass
password=[PASSWORD]
username=[USER NAME]
```
or using md5 store password
`echo -n "username:realm:my_secret_password" | md5sum`
e.g.
`echo -n "username:asterisk:my_secret_password" | md5sum`
```
[AUTH NAME]
type=auth
auth_type=md5
md5_cred=[MD5_SECRET]
username=[USER NAME]
```
![image](https://github.com/rolny/Asterisk_simple/blob/main/pic/1.png)

in ***Asterisk CLI***
reload pjsip module
`pjsip reload`

### check pjsip peers
in ***Asterisk CLI***
`pjsip show aors`
![image](https://github.com/rolny/Asterisk_simple/blob/main/pic/2.png)

`pjsip show auths`
![image](https://github.com/rolny/Asterisk_simple/blob/main/pic/3.png)

`pjsip show endpoints`
![image](https://github.com/rolny/Asterisk_simple/blob/main/pic/4.png)

`pjsip show transports`
![image](https://github.com/rolny/Asterisk_simple/blob/main/pic/5.png)

## setup extensions.conf
replace `extensions.conf` to `/data/asterisk-conf/etc/asterisk/extions.conf`
```
[DIALPLAN NAME]
exten => [Ext],[PRIORITY],APPLICATION([PJSIP/SIP]/[ENDPOINTSNAME],30,rtT)
```
e.g.
`exten => 1011,1,Dial(PJSIP/1011,30,rtT)`
![image](https://github.com/rolny/Asterisk_simple/blob/main/pic/6.png)
