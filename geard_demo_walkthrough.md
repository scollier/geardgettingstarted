# ToDo:


Explain linux containers; name spaces.
Open up by talking about the environment.  We have two VM's running on this laptop.
The VM's are running RHEL7 beta.

Cover 4 things today:

1. selinux
2. logging
3. systemd integration
4. geard linking
5. geard ssh


Section 1 - Simple

On host 10.16.138.234

Show selinux enabled on both nodes

    getenforce   # once per node
    getenforce

enter fedora container and show processes running

    ps -eZ

bind mount /etc/shadow into container, show lack of access
Do this on both hosts, but on the .233 host, disable selinux.
On the host that has selinux disabled, should be able to access /etc/shadow, not on the other one.

```
docker run -i -t -v /etc/shadow:/etc/shadow fedora:20 bash    # once on each host
docker run -i -t -v /etc/shadow:/etc/shadow fedora:20 bash
cat /etc/shadow
cat /etc/shadow
id
id
exit
```

Show how to do logging


```
docker run -i -t -v /dev/log:/dev/log fedora:20 bash
logger "Log from Dreamworks inside a container"
exit
journalctl -b | grep -i "Log from Dream"
```

Show systemd unit file, show how to start a docker container with systemd, show results in log file

```
cat /etc/systemd/system/demo_container.service
systemctl start demo_container.service
docker ps
ausearch -m SERVICE_START,SERVICE_STOP -ts recent
```

Show how to inspect a docker container

```
docker inspect --format '{{ .State.Pid }}' <Container ID>
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <Container ID>
```

Section 2 - Gear Linking
#### Need clean transistion here.  Perhaps describing basic docker containers, single service / container.  Need automation around multicontainer orchestration, geard helps provide this.
#### What is geard?  It's a mid-tier orchestrator and an integrator - from the deployment perspective, as well as some basic management activities.
#### Explain the geard daemon, ports, geard and client, geard and docker?
#### Explain simple concepts


```
gear install
gear start
gear status
gear list-units
gear stop
```

show the geard code, on host 10.16.138.234, Show the count, the name of the containers, and name of image
Talk about algorithm of deployment


```
cd ~
cat /root/geard/deployment/fixtures/complex_http_scollier-new.json
```

on each host, show the units.  Each web gear should be dead

    gear list-units

Deploy a web gear to each node, explain command, localhost is .234
Show the Dockerfile for the web-server

```
gear deploy complex_http_scollier-new.json localhost 10.16.138.233
gear list-units # on each host
docker ps # on each host, show ports
```

Enter the namespace on .233 and show iptables

```
docker inspect --format '{{ .State.Pid }}' websvr-new-2
nsenter -m -u -n -i -p -t 2725 bash
iptables -nvL -t nat      # show destination for local webserver
```

Use gear link to set up the cross host networking, explain the command
Do this on .234 host

    gear link -n "127.0.0.1:8081:10.16.138.234:4007" 10.16.138.233:43273/websvr-new-2

restart the gear, curl the remote web server

```
tcpdump -qnni eth0 host 10.16.138.234 # do ths on the .233 host
gear restart websvr-new-2   # do this on the .233 host
docker inspect --format '{{ .State.Pid }}' websvr-new-2
nsenter -m -u -n -i -p -t 2725 bash

gear install pmorie/sti-html-app demoapp --isolate
gear list-units
gear start demoapp
gear status demoapp
ps -ef | grep mock_server
gear add-keys --key-file="/root/.ssh/id_rsa.pub" demoapp
```

Section 3 - Uber
On the .233 host, set up the demo

```
cd ~
geard/contrib/demos/teardown.sh
geard/contrib/demos/setup.sh
```


drive some load after running setup

    drive_load.sh &

If need to troubleshoot, look at log files for each gear.

    journalctl -n 100 -u ctr-demo-backend-1

