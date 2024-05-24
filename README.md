# ser2net-setup

## Install ser2net
* Install
```sh
sudo apt-get -y install ser2net
sudo apt-get -y libyaml-dev libgensio-dev
```
* configure file `test.yaml`
```
#%YAML 1.2  # Use YAML version 2

define: &banner \r\nser2net port \p device \d [\B] (Debian GNU/Linux)\r\n\r\n

connection: &conn0
  accepter: telnet(rfc2217),tcp,4001
  enable: on
  options:
    max-connections: 4
    banner: *banner
    kickolduser: true
    telnet-brk-on-sync: true
  connector: serialdev,/dev/ttyUSB0,115200n81,local
```
Replace ttyUSB0 by your Serial port

* On the Beaglebone side
```sh
ser2net -l -n -c ./test.yaml
```

* On client side
```sh
nc 10.66.77.28 4001
```
where 10.66.77.28 is the IP or domain of BB




## Configure file `/etc/ser2net.yaml` to run as the service
* cp `test.yaml` above to `/etc/ser2net.yaml`
* Note that the default service(/lib/systemd/system/ser2net.service) one don't have the constraint to start after network service. So need modify it before run
```
[Unit]
Description=Serial port to network proxy
Documentation=man:ser2net(8)
After=network-online.target
Wants=network-online.target
[Service]
EnvironmentFile=-/etc/default/ser2net
ExecStart=/usr/sbin/ser2net -n -c $CONFFILE -P /run/ser2net.pid
Type=exec
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

* run
```sh
sudo systemctl enable ser2net
sudo systemctl restart ser2net
sudo systemctl status ser2net
```



## Reference
* https://groups.io/g/nodered-hamradio/topic/95857282#10545
* https://www.youtube.com/watch?v=9jCQMr3tFnI
