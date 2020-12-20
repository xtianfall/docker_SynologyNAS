# docker_SynologyNAS
## Install
(How to install Docker on non-intel Synology NAS)
1. Create a new shared folder in your NAS volume. 
For example: **/volume1/NASDocker**
2. Get the server name (to use in step 10) from:
	Control Panel > Info Center > Net > Server Name

3. Open a SSH conection to NAS server and get root user:
```bash
ssh <USER>@<IP>
sudo -i
```

4. Download docker instalation, decompress and remove the file.
```bash
/bin/wget https://raw.githubusercontent.com/xtianfall/docker_SynologyNAS/master/catdsm-docker.tgz
tar -xvpzf catdsm-docker.tgz -C /
rm catdsm-docker.tgz
```

5. Change the current PATH to use the docker files.
```bash
PATH=/opt/sbin:/opt/bin:/opt/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

6. Edit the docker daemon file to change the default directory of files.
```bash
vi /etc/docker/daemon.json
```
> Enter in insert mode typing: i
> Edit the *graph* line
>> For example (got it in step 1):
>> "graph":**"/volume1/NASDocker"**,
> Exit from insert mode: esc
> Save and quit the editor typing: :wq (and enter)

7. Run the docker service:
```bash
/opt/etc/init.d/S60dockerd
```

8. Run Portainer docker:
```bash
docker run -d 
  --network=host 
  --name="Portainer" 
  --restart always 
  -v /var/run/docker.sock:/var/run/docker.sock 
  -v portainer_data:/data 
  portainer/portainer-ce
```

9. Enter in `<NAS_IP>:9000` using the browser and create an admin user. 
Then select Docker and click in connect. 
This is Portainer.

10. Run HomeBridge docker:
(replacing NAS (line 8) with your own server name (step 2))
```bash
docker run \
  --net=host \
  --name=homebridge \
  --restart always \
  -e PUID=0 -e PGID=0 \
  -e HOMEBRIDGE_CONFIG_UI=1 \
  -e HOMEBRIDGE_CONFIG_UI_PORT=8080 \
  -e DSM_HOSTNAME=<NAS> \
  -v  /var/run/docker.sock:/var/run/docker.sock \
  -v homebridge_data:/data \
  oznu/homebridge
```
It will take a little to set all up. When the pairing code appears end the command with `control+c`.

11. In the Portainer page, enter in local docker, then go to containers, click on homebridge and click on start.

12. To finish the home bridge installation, enter in `<NAS_IP>:8080` enter with `admin`as user and passwrod and edit the configuration file to add this line before “bridge”:
```bash
"mdns": {"interface": "<NAS_IP>"},
```

13. Set autostart of Docker service when restart the NAS:
Go to: Control Panel > Task Scheduler > Create > Triggered Task > User-defined Script
>User: root
>Event: startup
>Activity Settings -> Execute Command:
>>bash /opt/etc/init.d/S60dockerd

14. Reboot the Nas:
```bash
reboot
```

## Plugins
To add plugins follow this steps:
[Docker Homebridge Plugins guide](https://github.com/oznu/docker-homebridge#homebridge-plugins)

## Uninstall
<b>Login to DSM with your/any admin account</b>, at this point go to:

  <b>Control Panel</b> -> <b>Task Scheduler</b> -> <b>Create</b> -> <b>Triggered Task</b> -> <b>User-defined Script</b>.

  <b>FIRST TASK</b>
  User: <b>root</b>
  Event: <b>Shutdown</b>
  Activity Settings -> Execute Command:
  ```
  rm -rfv /opt/etc/init.d/S60dockerd
  ```

  <b>SECOND TASK</b>
  User: <b>root</b>
  Event: <b>Bootup</b>
  Activity Settings -> Execute Command:
  ```
rm -rfv /var/run/docker.pid & rm -rfv /var/run/docker.sock & rm -rfv /var/run/docker & rm -rfv /opt/var/lib & rm -rfv /opt/usr/bin & rm -rfv /opt/etc/default & rm -rfv /etc/docker & rm -rfv /volume1/NASDocker
```

  <b>Reboot your NAS</b>.

  When the reboot is completed you can <b>login to DSM</b> and <b>delete both the tasks</b>.

