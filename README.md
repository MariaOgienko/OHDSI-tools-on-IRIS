# Instructions for Integrating OHDSI/Broadsea with the InterSystems IRIS Data Platform

### Summary
This document provides step‑by‑step instructions for installing InterSystems IRIS, deploying the OHDSI Broadsea tool‑stack, and connecting the two platforms. It is intended for engineers or data scientists who need a reproducible procedure that works on Linux, macOS, or Windows (via Docker). Screenshots referenced in the original draft are preserved; if you are viewing this in a text‑only context, follow the descriptive captions instead.

### Prerequisites
- Hardware  4 CPU, 8 GB RAM minimum (16 GB recommended)
- Operating System — Linux or macOS; Windows users should run InterSystems IRIS in Docker.
- Docker Desktop  Required for container‑based deployment.
- Git -  For cloning Broadsea.
- Java 17+  Needed by several OHDSI components.
- Internet access  To pull images, drivers, and R packages.

#### Minimum versions
Minimum versions – InterSystems IRIS 2024.3 (most features work on 2024.1.3); SqlRender 2.1.1 (next release); WebAPI 2.13.1 (PR #1234); OHDSI R packages – latest versions from the OHDSI GitHub repository. These packages are automatically downloaded and installed as part of the OHDSI Broadsea setup process.

### Installing InterSystems IRIS for Health natively (available for macOS / Linux)

All examples use InterSystems IRIS for Health 2025.1 Community Edition unless stated otherwise. The same commands work on an Enterprise license by substituting the image/tag.
Download the community edition from [InterSystems WRC](https://wrc.intersystems.com/wrc/Login.csp?Local=1) (Actions → Online Distribution → InterSystems IRIS or IRISHealth). Make sure you are using IRISHealth_Community-2024.3 version or later.

Open a terminal and execute:
```
# use the directory with IRISHealth file
cd ~/Downloads/IRISHealth_Community-2024.3.0.217.0-macos

# Make the irisinstall installation file executable:
chmod +x irisinstall

# Run the installation:
sudo ./irisinstall

# After installation is complete, run the command verify the service is running (“Status = running”):
iris list
```

Additional steps to perform in the terminal during installation:
**Instance name** — keep the default IRISHEALTH unless you need multiple instances.
**Destination directory** — absolute path beginning with /.
**Installation type** — choose Development unless you have a server‑only use‑case.
**Owner user** — your current OS username (whoami).

#### Managing the InterSystems IRIS Service
During the installation of the application, you may need to configure other settings:
```
# status check to verify the service is running (“Status = running”)
iris list

#Launching the application
iris start IRISHEALTH

# Stop the application
iris stop IRISHEALTH
```
Access the Management Portal at http://localhost:52773/csp/sys/UtilHome.csp after the service is running.
For more detailed instructions, please refer to the official InterSystems [documentation](https://docs.intersystems.com/iris20251/csp/docbook/Doc.View.cls?KEY=PAGE_deployment_install).

###Running InterSystems IRIS for Health in Docker (recommended)
*Note for Windows users: It is recommended to run IRIS inside a container (e.g., using Docker), as this approach ensures easier setup, better compatibility, and improved isolation from the host system.*

Find the latest tag on [Docker Hub](https://hub.docker.com/r/intersystems/iris-community).  For further details on how to run InterSystems IRIS for Health in Containers visit [Intersystems Documentation](https://docs.intersystems.com/iris20251/csp/docbook/DocBook.UI.Page.cls?KEY=ADOCK)

Execute:
```
docker run --name my-iris -d -v dbvolume:/durable intersystems/iris-community:2025.1

# The next commands can be updated to use advanced flags
docker exec -u root my-iris chown -R irisowner:irisowner /durable

docker container rm -f my-iris

docker run --name my-iris -d --publish 1972:1972 --publish 52773:52773 -v dbvolume:/durable --env ISC_DATA_DIRECTORY=/durable/iris intersystems/iris-community:2025.1
```
Browse to **http://127.0.0.1:52773/csp/sys/UtilHome.csp** and log in with the default account (_SYSTEM / SYS), then set a strong password.
For custom [volumes](https://docs.docker.com/engine/storage/volumes/), entry‑points, or ARM images see the official [Docker guide](https://docs.docker.com/manuals/).

### Deploying OHDSI Broadsea and Connecting to InterSystems IRIS
