# Docker

More: <https://github.com/hexops/dockerfile>

## Info

|What|Where|
|-|-|
|Official Page|<https://www.docker.com/>|
|Docs|<https://docs.docker.com/get-started/>|
|Download|<https://desktop.docker.com/win/stable/Docker%20Desktop%20Installer.exe>|
|Install|```choco install docker-desktop```|

## Cool things to run with docker

A list of docker containers

### Powershell in Docker

Run the container

```shell
docker run -it mcr.microsoft.com/azure-powershell
```

Trigger script in Powershell in Docker

```shell
docker run -it -v C:\Users\username\src:/src mcr.microsoft.com/azure-powershell:3.6.1-ubuntu-18.04 pwsh -file /src/script.ps1
```

### Tutorials

- <https://www.ezzeddinabdullah.com/posts/how-to-clean-text-data-at-the-command-line>

### I2P

Sources:

- <https://geti2p.net/de/download>
- <https://hub.docker.com/r/meeh/i2p.i2p/>

```shell
docker pull meeh/i2p.i2p
```

### Drawio

Sources:

- <https://github.com/fjudith/docker-draw.io>

```shell
docker run -it --rm --name="draw" -p 8080:8080 -p 8443:8443 fjudith/draw.io
```

### NoteCalc

Sources:

- <https://github.com/bbodi/notecalc3>

```shell
git clone https://github.com/bbodi/notecalc3.git
cd notecalc3
docker build . --tag notecalc3
docker run --rm -d -p 5000:5000 notecalc3
```

### Archive Box

Sources:

- <https://github.com/ArchiveBox/ArchiveBox>
- <https://github.com/ArchiveBox/ArchiveBox/wiki/Docker>
- <https://nixintel.info/osint-tools/make-your-own-internet-archive-with-archive-box/>

```shell
docker run -v $PWD:/data archivebox/archivebox init
docker run -v $PWD:/data archivebox/archivebox add 'https://0xfab1.net'
docker run -v $PWD:/data -it archivebox/archivebox manage createsuperuser
docker run -v $PWD:/data -p 8000:8000 archivebox/archivebox server 0.0.0.0:8000
```

### Wireguard

Sources:

- <https://github.com/linuxserver/docker-wireguard>

```shell
docker pull ghcr.io/linuxserver/wireguard
```

### IPFS

Sources:

- <https://registry.hub.docker.com/r/ipfs/go-ipfs>
- <https://github.com/ipfs/go-ipfs>

```shell
docker pull ipfs/go-ipfs
docker run -d --name ipfs_host -v $ipfs_staging:/export -v $ipfs_data:/data/ipfs -p 4001:4001 -p 127.0.0.1:8080:8080 -p 127.0.0.1:5001:5001 ipfs/go-ipfs:latest
docker exec ipfs_host ipfs swarm peers
docker logs -f ipfs_host
```

### Matrix Synapse

Sources:

- <https://registry.hub.docker.com/r/matrixdotorg/synapse/>

```shell
docker pull matrixdotorg/synapse
docker run -it --rm --mount type=volume,src=synapse-data,dst=/data -e SYNAPSE_SERVER_NAME=my.matrix.host -e SYNAPSE_REPORT_STATS=yes matrixdotorg/synapse:latest generate
```

### Jellyfin

Sources:

- <https://github.com/jellyfin/jellyfin>
- <https://jellyfin.org/>

```bash
docker pull jellyfin/jellyfin:latest
mkdir -p /srv/jellyfin/{config,cache}
docker run -d -v /srv/jellyfin/config:/config -v /srv/jellyfin/cache:/cache -v /media:/media --net=host jellyfin/jellyfin:latest
```

### Nextcloud

Sources:

- <https://hub.docker.com/_/nextcloud/>
- <https://nextcloud.com>

```bash
docker run -d -p 8080:80 nextcloud
```

### Collabora Online

Sources:

- <https://nextcloud.com/collaboraonline/>
- <https://www.collaboraoffice.com/code/docker/>
- <https://github.com/CollaboraOnline/online>
- <https://collaboraonline.github.io/>

```bash
docker run -d -p 8080:80 nextcloud
```

### Burpsuite

```bash
docker run -d --name burpsuite -e DISPLAY -v ${HOME}:/home/burpsuite -v /tmp/.X11-unix/:/tmp/.X11-unix/ --p 8080:8080 alexandreoda/burpsuite
```

### mitmproxy

```bash
docker run --rm -it -p 8080:8080 mitmproxy/mitmproxy
```

### pytorch

```bash
docker run --gpus all --rm -ti --ipc=host pytorch/pytorch:latest
```
