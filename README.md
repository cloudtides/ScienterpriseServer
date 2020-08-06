# Scienterprise Server

Scienterprise (Science@Enterprise) is a BOINC project aims to help scientific research by using idle computing resources donated by enterprises.

The server is built on [boinc-server-docker](https://github.com/marius311/boinc-server-docker). It packs all dependencies of a BOINC project server into a Docker application and makes it much easier to start a BOINC project server. To know more about Docker, please check [it](https://docs.docker.com/) out and the two tutorials about Docker: [Docker tutorial for beginners through examples](https://takacsmark.com/getting-started-with-docker-in-your-project-step-by-step-tutorial/#data-in-docker-containers), [Docker compose tutorial for beginners by example](https://takacsmark.com/docker-compose-tutorial-beginners-by-example-basics/).

This guide will show how to launch the Scienterprise server (based on [BOINC Project Cookbook](https://github.com/marius311/boinc-server-docker/blob/master/docs/cookbook.md)) and onboard a project.

## Launch the Server
### Requirements
The docker images are proved to run normally on Ubuntu 16.04. There might be some issues if you try it on Mac or other versions of Ubuntu. Here are some requirements for Ubuntu 16.04:
- [Docker](https://docs.docker.com/engine/installation/) (>=17.03.0ce)
- [docker-compose](https://docs.docker.com/compose/install/) (>=1.13.0 but !=1.19.0 due to a [bug](https://github.com/docker/docker-py/issues/1841))
- git

### Build and Run
Clone this repo to your machine,
```bash
git clone git@github.com:scienterprise/ScienterpriseServer.git
cd ScienterpriseServer
```
then run it,
```bash
docker-compose up -d
```

Now the BOINC server is ready for you at `http://127.0.0.1/scienterprise` if you deploy it on localhost or `http://$(public ip)/scienterprise` if you deploy it somewhere else.

**Note**
- When you modify any dockerfile, you need to rebuild the image first 'docker-compose build'. Or combine build and run into `docker-compose up -d --build`.
- If you are not running the server on localhost (`127.0.0.1`), you need to specify `URL_BASE` when start the server by running
```bash
URL_BASE=http://1.2.3.4 docker-compose up -d
```
where `1.2.3.4` is replaced by the public ip address of your machine.
- If you are running the server on a commercial cloud, remember to add policy that allows inward traffic through port 80.
## Onboard a Project
