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
docker-compose up -d --build
```

Now the BOINC server is ready for you at `http://127.0.0.1/scienterprise` if you deploy it on localhost or `http://$(public ip)/scienterprise` if you deploy it somewhere else. And you can also access the admin page by simply add `_ops` to the end of your url with username `admin` and no password (by default). 

**Note**
- If you are not running the server on localhost (`127.0.0.1`), you need to specify `URL_BASE` when start the server by running
```bash
URL_BASE=http://1.2.3.4 docker-compose up -d
```
where `1.2.3.4` is replaced by the public ip address of your machine.
- If you are running the server on a commercial cloud, remember to add policy that allows inward traffic through port 80.

**Useful Commands**
- In your machine
  - Start and stop the BOINC server, run
  ```bash
  docker-compose up -d
  docker-compose down
  
  # To delete all the volumes
  docker-compose down -v
  ```
  within the same directory as `docker-compose.yml`.
  - Rebuild the images after modifying any dockerfile
  ```bash
  docker-compose build
  
  # Build and launch in one command
  docker-compose up -d --build
  
  # Build without cache
  docker-compose build --no-cache
  ```
  - Enter the apache server, run
  ```bash
  docker-compose exec apache bash
  ```
  within the same directory as `docker-compose.yml`.
  - Show status of all docker images on your machine
  ```bash
  docker ps
  ```
- In the apache server
  - Start all the daemons (see status of daemons on `http://boinc.scienterprise.cn/scienterprise/server_status.php`)
  ```bash
  bin/start
  ```
  - Update `Username` and `Password` for admin page
  ```bash
  cd html/ops
  htpasswd -c .htpasswd ${new_username}
  # Follow the prompt to enter ${new_password}
  ```

## Onboard a Project
As mentioned in [BOINC Technical Documentation](https://boinc.berkeley.edu/trac/wiki/ProjectMain), there are several ways of creating applications on a BOINC server: bundle the executable with a virtual machine image containing the running environment, package up the excutable and environment into a docker image, write a native BOINC application, use a BOINC wrapper together with the executable.

Considering feasibility and convenience, the "wrapper" way is the most appropriate and simplest for Scienterprise server. In this way, an existing application can be run by using a corresponding [wrapper program provided by BOINC](https://boinc.berkeley.edu/trac/wiki/WrapperApp).

Since the building processes of "wrapper" way and "native" way are similar, current repo contains one sample application for each way: [worker](https://github.com/BOINC/boinc/tree/master/samples/worker) and [upper_case](https://github.com/BOINC/boinc/tree/master/samples/example_app/bin/22489/x86_64-pc-linux-gnu).

The following steps are workable for both ways. To know more about each step, read the [BOINC Server Guide](https://wiki.debian.org/BOINC/ServerGuide/) and [BOINC Technical Documentation](https://boinc.berkeley.edu/trac/wiki/ProjectMain).
1. Create a directory under `apps/` in apache server and then add executable files and app configuration to it
  - For "native" app `upper_case`, only need one executable `example_app_22489_x86_64-pc-linux-gnu`.
  - For "wrapper" app `worker`, need two executables `worker` and `wrapper_26014_x86_64-pc-linux-gnu`.
```bash
apps
|-- upper_case
|   `-- 1.0
|       `-- x86_64-pc-linux-gnu
|           |-- example_app_22489_x86_64-pc-linux-gnu
|           `-- version.xml
`-- worker
    `-- 1.0
        `-- x86_64-pc-linux-gnu
            |-- version.xml
            |-- worker
            |-- worker_job_1.0.xml
            |-- wrapper_26014_x86_64-pc-linux-gnu
```
2. Inform local database of the new app by appending the following lines to `project.xml`
```bash
    <app>
        <name>upper_case</name>
        <user_friendly_name>native_upperCASE</user_friendly_name>
    </app>
```
Then run
```bash
bin/xadd
```
3. Sign the executable(s) under `apps/` using `code_sign_private`
  - `code_sign_private` was generated when building the server. You can find it by running `docker-compose run makeproject bash` in local machine and then `cd /run/secrets/keys`. Read [this](https://boinc.berkeley.edu/trac/wiki/KeySetup) to know more. 
  - Run 
  
```bash
bin/sign_executable /path/to/executable /path/to/code_sign_private >> filename.sig
```
to sign the executable.

After this step, `apps/` looks like
```bash
apps
|-- upper_case
|   `-- 1.0
|       `-- x86_64-pc-linux-gnu
|           |-- example_app_22489_x86_64-pc-linux-gnu
|           |-- example_app_22489_x86_64-pc-linux-gnu.sig
|           `-- version.xml
`-- worker
    `-- 1.0
        `-- x86_64-pc-linux-gnu
            |-- version.xml
            |-- worker
            |-- worker.sig
            |-- worker_job_1.0.xml
            |-- wrapper_26014_x86_64-pc-linux-gnu
            `-- wrapper_26014_x86_64-pc-linux-gnu.sig
```
4. Update the BOINC database
```bash
bin/update_versions
```
5. Create workunit template file in `templates/`
6. Create and stage an input file
```bash
bin/stage_file /path/to/input_file
```
7. Generate a workunit
```bash
bin/create_work --appname ${app_name} --wu_name ${work_unit_name} /path/to/staged_input_file
```

Now the new workunit is available for user to download and run. See workunit on `${ip_address}/ops` from your browser. You can see result after one user has finished the task in `upload/` on your apache server.


