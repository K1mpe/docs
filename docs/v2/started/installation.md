---
id: installation
title: Install NetDaemon runtime
---

There are several ways to deploy your apps. You could just run the development project and set it up yourself. NetDaemon provides a convenient ways to run your apps:
- Using a Home Assistant add-on
- In a docker container
- Or using the template as base deploy your own.

So let's set it up before we can deploy the apps to it.

## NetDaemon versions
This doc describes setting up the V2 version. For NetDaemon v3 please checkout [installing the NetDaemon V3](v2/started/installation_v3.md).

## Install NetDaemon as a Home Assistant add-on
Two versions are provided, release and dev for both NetDaemon version 2 and version 3. If you feel you always want the latest and greatest changes you can choose the dev build for the version that you selected but be prepare that things break!


1. Make a folder structure under your Home Assistant configuration directory to use for NetDaemon apps. `/config/netdaemon`  
2. Add the `https://github.com/net-daemon/homeassistant-addon` in `Add new repository URL` to the add-on store.

    ![](/img/docs/started/newrepo.png)

3. Add the NetDaemon add-on.

    ![](/img/docs/started/daemon.png)

4. Deploy your apps in the folder `/config/netdaemon/apps` for V2 and `/config/netdaemon3/apps` for V3 .

### Advanced add-on configurations
The standard way running your apps in add-on is just to copy the `.cs` and `.yaml` files to `/config/netdaemon/apps` or (`/config/netdaemon3/apps` is you are using V3) and it will dynamically compile and run them. If you want to use your own project with custom dependencies you will have to use the advanced deployment options. 

Specify the `AppSource`setting in add-on config. If you set a `app_source: folder`, then it will dynamically compile and run apps in `/config/netdaemon/folder`. 

If setting `app_source: daemonapp.csproj` it will compile and run the project in `/config/netdaemon/daemonapp.csproj` with all it's dependencies. To deploy apps Set the `app_source: daemonapp.dll` it will run the published project in `/config/netdaemon/daemonapp.dll`.

**There is no way to include other dependencies in the standard dynamically compiled option** No, for this we recommend the `Advanced add-on configurations` decribed above, to include for example a external nuget package.

## Install as a docker container
If you are using Home Assistant Core and do not have the possibility to run add-ons, using the docker container is a convenient way to run NetDaemon apps. 

**Always use specific versioning tags of docker containers (not latest or dev) cause these are constantly getting new versions and things could break** [You can always find the latest stable version here](https://github.com/net-daemon/netdaemon/releases)

### Example docker run configuration

```bash
docker run -d \
  --name netdaemon \
  --restart=always \
  -e HOMEASSISTANT__HOST=192.168.1.4 \
  -e HOMEASSISTANT__TOKEN=XXXXX \
  -e NETDAEMON__GENERATEENTITIES=False \
  -e LOGGING__MINIMUMLEVEL=info \
  -e TZ=Europe/Stockholm \
  -p 1337:1337 \
  -v ~/netdaemon_config:/data \
  netdaemon/netdaemon
```
_`-p 1337:1337` is only needed if you want the [admin panel](https://github.com/net-daemon/admin)_

### Example using docker-compose.yaml for
```yaml
version: '3.7'
services:
  netdaemon:
    image: netdaemon/netdaemon                  # use netdaemon/netdaemon:ver 
                                                # for specific version
    container_name: netdaemon
    restart: always
    environment:
      - HOMEASSISTANT__HOST=your_ip_or_hostname
      - HOMEASSISTANT__TOKEN=your_token
      - LOGGING__MINIMUMLEVEL=info              # use trace/debug/info
      - TZ='Etc/UTC'                            # Set your current timezone
    ports:
      - 1337:1337                               # This is required only if 
                                                # you are running the GUI
    volumes:
      - /config/netdaemon:/data                 # replace /config/netdaemon 
                                                # to your local folder
```

### Evironment variables
The docker container needs 3 environment variables to run properly.

ENV | Description
-- | --
`HOMEASSISTANT__HOST` | The host that is running Home Assistant (defaults to `localhost`)
`HOMEASSISTANT__PORT` | The port Home Assistant is running on (default to `8123`)
`HOMEASSISTANT__TOKEN` | A Long Lived Acces Token(LLAT) that NetDaemon can use for the comminication with Home Assistant.
`NETDAEMON__GENERATEENTITIES` | Defaults to False, set True if you want the autogenerated entities
`LOGGING__MINIMUMLEVEL` | Defaults to info, values are (trace, debug, info, warning, error)
`TZ` | You will need to set container time zone to make the scheduler work properly
`NETDAEMON__APPSOURCE` | NETDAEMON__APPSOURCE is used to setup more advanced runtime options.  If you set a `folder`, the deafult dynamically compiled apps should be set in this folder. If `daemonapp.csproj` the runtime will run that project with all it's dependencies.Set to `daemonapp.dll` for published project. For standard dynamically compiled projects you do not have to set this setting.
`NETDAEMON__ADMIN` | Set this to `false` to disabled the Admin UI.

### Volumes

Vol | Description
-- | --
/data | The volume of the netdaemon folder should be mapped to `/data` [See how to setup the correct folder here](installation.md#folder-structure-and-where-to-map-the-docker-volume)



### Folder structure and where to map the docker volume
The `~/netdaemon_config` need to point to the `netdaemon` folder. See image below. The **red arrow** in the example configuration below points to the folder that should be mapped to the `/data`!

![](/img/docs/installation/folder_structure_netdaemon.png)

