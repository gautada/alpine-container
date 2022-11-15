# Alpine

[Alpine Linux](https://alpinelinux.org) is an independent, non-commercial, general purpose Linux distribution designed for power users who appreciate security, simplicity and resource efficiency.

This container is the base image that uses Alpine Linux as the intial image for other containers as such it's primary purpose is to provide functionalityservices for other downstream containers.

## Services

- Entrypoint: A stacked collection of startup scripts that launch supporting services for the running container but still provide a mechanism for cli command execution.
- *DROP THIS* Exitpoint: Similar to entry pointbut on container application exit.
- Healthcheck: A drop-in based system to stack all the needed checks for reporting container health including liveness and readiness.
- Sudo: Tightly controlled access to privilege escalation through a stacked collection
- Backup: A customizable container backup solution that defines a daily full backup and manages an hourly delta mechanism. 
- Timezones: This and all downstream containers are set to the same timezone
- Schedular: A time based esecution schedular for container self management
- Environment Profile: A stackable mechanism for CLI environment and porfile definitions
- Tools: A defined set of tools that help with container debugging and manipulation.

## Versions

- April 30, 2022 [alpine](https://alpinelinux.org/releases/) - Active version is [3.15 .4](https://git.alpinelinux.org/aports/log/?h=v3.15.4)
- May 31, 2022 [alpine](https://alpinelinux.org/releases/) - Active version is [3.16.0](https://git.alpinelinux.org/aports/log/?h=v3.16.0)
- September 8, 2022 [alpine](https://alpinelinux.org/releases/) - Active version is [3.16.2](https://git.alpinelinux.org/aports/log/?h=v3.16.0)

## Notes

### Entrypoint
This container provides the `/entrypoint` script and sets the `ENTRYPOINT` value in the **Containerfile**. The `/entrypoint` script call the subsequent scripts in the `/etc/entrypoint.d` drop-in folder.  These scripts start the container services and optionally executes the container processes.  If not overload by CLI parameter or entrypoint script, the default process is `crond`.

### Healthcheck
By default this image provides a method for checking the running health of a container. The script `/usr/sbin/container-healthcheck` runs all subscripts in the healthcheck drop-in folder
`/etc/container/healthcheck.d`. Healthchecks are generally considered to be cummulative. Downstream images should add more healthchecks using the `Containerfile` via `COPY hc-crond.sh /etc/container/healthcheck.d/hc-crond.sh`

### Profile 
The default profile for Alpine is the `ash` shell.  This is configured as default using `ENV ENV="/etc/profile"` in the `Containerfile`. Usually to customize just add scripts `/etc/profile.d` folder

### Sudo
To allow for super user access the container uses a wheel mechanism.  This requires the 
default user to be part of the `wheel` group via `/usr/sbin/usermod -aG wheel $USER`. To enable super user access to members of the `wheel` group, a wheel defintion file (wheel-[name]) must be provided that defines a sudo access line (i.e. `%wheel         ALL = (ALL) NOPASSWD: /usr/sbin/container-backup`). The wheel definition file should be loaded into the wheel drop-in folder in the `Containerfile` via `COPY wheel-backup /etc/container/wheel.d/wheel-backup`.

### Timezone
By default the timezone is set to US/New York a.k.a US/Eastern.  This provides consistency across containers.
```
RUN apk add --no-cache tzdata
RUN cp -v /usr/share/zoneinfo/America/New_York /etc/localtime
RUN echo "America/New_York" > /etc/timezone
```

### User
Each downstream image should co figure their own downstream default user in the `Containerfile`
```
ARG USER=postgres
VOLUME /opt/$USER
RUN /bin/mkdir -p /opt/$USER \
 && /usr/sbin/addgroup $USER \
 && /usr/sbin/adduser -D -s /bin/ash -G $USER $USER \
 && /usr/sbin/usermod -aG wheel $USER \
 && /bin/echo "$USER:$USER" | chpasswd \
 && /bin/chown $USER:$USER -R /opt/$USER /etc/backup /var/backup /tmp/backup /opt/backup
USER $USER
WORKDIR /home/$USER
```

### Version
This the operating system container defines two version [aliases](https://linuxhandbook.com/linux-alias-command/) (`osversion` and `cversion`)
- **Operating System(OS) Version** - `osversion` prints the release version of Alpine linux
- **Container Version** - `cversion` prints the container's version, this is mainly for downstream containers where the primary application's version is represented. For instance if the contain is to provide `podman` this would return `podman --version`. This allows for a standard mechanism to determine the running version of the container. **This should be overloaded in downstream systems**. For better compatability the script `/bin/version` is provided infront of the `cversion` alias.  This script can be called in an `exec` mode and should be called in lieu of a direct call to `cversion`.


## Development

All container services should move to docker-compose for their build environments but for systems that need to **bootstrap** to get up and running the following is a template for manually getting things running.

### Build

docker build --build-arg ALPINE_VERSION=3.16.2 --file Containerfile --label revision="$(git rev-parse HEAD)" --label version="$(date +%Y.%m.%d)" --no-cache --tag alpine:build .

### Run



