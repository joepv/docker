## Start to migrate

* old machine cloned to temporary machine
* temporary e-mail address used on new machine
*

## Migrate UniFi controller



Login to the controller:
https://192.168.2.101:8443

* Download backup
* Turn of the controller on Windows 10
* I give the new server the same IP address as the old contoller, if you do not do that get APs again
* zo change ip of controller
* change ip of debian box
*  `interface ip set address name="Ethernet" static 192.168.10.42 255.255.255.0 192.168.1.1`