# Migrate UniFi controller from Windows 10 to Docker container

![unifi-backup](img/unifi-backup.png)

* Make back-up.
* Shut down controller on Windows 10.

* Change IP windows 10.
`netsh interface ip set address name="Ethernet" static 192.168.2.110 255.255.255.0 192.168.2.254`

* Change IP Debian 10.

Edit the file `/etc/network/interfaces` to change it to a static address:

```
auto eno1
iface eno1 inet static
    address 192.168.1.101
    subnet 255.255.255.0
    gateway 192.168.1.254
```
Create compose file.

```
version: '3.7'

services:
  controller:
    image: "jacobalberty/unifi:${TAG:-latest}"
    container_name: unifi-controller
    hostname: unifi
    ports:
      - "3478:3478/udp"   # STUN
      - "6789:6789/tcp"   # Speed test
      - "8080:8080/tcp"   # Device/ controller comm.
      - "8443:8443/tcp"   # Controller GUI/API as seen in a web browser
      - "8880:8880/tcp"   # HTTP portal redirection
      - "8843:8843/tcp"   # HTTPS portal redirection
      - "10001:10001/udp" # AP discovery
    environment:
      TZ: 'Europe/Amsterdam'
    volumes:
      - ./data:/unifi/data
      - ./log:/unifi/log
      - ./cert:/unifi/cert
    init: true
    restart: always
```

Turn on, import back-up

![unifi-restore](img/unifi-restore.png)

Adopting Access Points/Switches/Security Gateway
Layer 3 Adoption

The default example requires some l3 adoption method. You have a couple options to adopt.
Force adoption IP

Run UniFi Docker and open UniFi in browser. Go under Settings -> Controller and then enter the IP address of the Docker host machine in "Controller Hostname/IP", and check the "Override inform host with controller hostname/IP". Save settings and restart UniFi Docker container. 

![unifi-adoption](img/unifi-adoption.png)

restart container

# Migrate DSMR reader

postgres pg_dump -d dsmrreader -v | gzip --fast > /home/joep/dsmrdb-backup.sql.gz

copy over with scp to new server

create compose file

run docker

docker-compose stop dsmr
docker exec -t dsmr-database dropdb dsmrreader -U dsmrreader
docker exec -t dsmr-database createdb -O dsmrreader dsmrreader -U dsmrreader
cat dsmrreader.sql | docker exec -i dsmr-database psql -U dsmrreader
docker-compose start dsmr