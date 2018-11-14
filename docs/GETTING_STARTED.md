# Flash Write Monitor

This application monitors per-process write to disk on top of the filesystem and
provides eMMC lifespan information for devices equipped with eMMC compatible to
the JEDEC 5.0 eMMC standard, as well as granular information for eMMCs that have
vendor-specific information.

## Getting Started

Power-on the module and:

* Make sure you have Torizon installed. You can install it following the Torizon documentation.
* Make sure you have access to the Linux console, either from the debug serial or SSH.
    * For ssh you can use the default tordy credentials - user: `tordy` / password: `tordy`
    * For debug serial you can use the default root credentials - user: `root` / password: empty
* Make sure that the board is connected to the internet.
    * You can e.g. ping a known website `ping google.com`

Once you are logged in, create a Docker volume to persist data:

```
docker volume create flash-data
```

Start a system-wide analysis:

```
docker run -dt --privileged=true \
    --name flash \
    -v /var/run/dbus:/var/run/dbus \
    -v /dev:/dev \
    -v /etc/issue:/etc/issue \
    --net=host \
    --pid=host \
    --mount type=tmpfs,destination=/var/volatile/log \
    --mount type=volume,source=flash-data,target=/app/.storage \
    toradexlabs/flash-analytics-tool
```

Get the board IP address:

```
ip a show dev eth0
```

Access the remote UI from your PC using the URL corresponding to the board IP:

```
http://<board-ip>
# For instance, if the board IP is 192.168.10.34
http://192.168.10.34
```

# Abbreviation list

This section lists acronyms used in the project context and their meaning.

* SLC - [Single-Level Cell](https://www.micron.com/products/nand-flash/slc-nand),
which is a technology/configuration of the raw NAND.
* MLC - [Multi-Level Cell](https://www.micron.com/products/nand-flash/mlc-nand),
which is a technology/configuration of the raw NAND.
* EOL - End of Life
* eMMC - [Embedded MultiMediaCard](https://www.micron.com/products/managed-nand/emmc),
a type of flash memory.
* IEC - [International Electrotechnical Commission](https://www.iec.ch/)
* JEDEC - [Joint Electron Device Engineering Council](https://www.jedec.org/)
* kB - [kilobyte](https://en.wikipedia.org/wiki/Kilobyte), usually mistaken
with KB.
* KB - [kibibyte](https://en.wikipedia.org/wiki/Kibibyte) in JEDEC format
* KiB - [kibibyte](https://en.wikipedia.org/wiki/Kibibyte) in IEC format. The IEC
format is adopted by this project for displaying units, e.g. KiB, MiB, GiB, etc.
* OS - Operating System

## Supported Modules

Make sure that your module is supported by Torizon and at least JEDEC 5.0:

| Computer on Module | Configuration | Version | JEDEC 5.0 Info | Vendor Info | Torizon Support |
|--------------------|---------------|---------|----------------|-------------|-----------------|
| Apalis iMX6Q       | 2GB IT        | V1.1C   | Yes            | Yes         | Yes             |
| Apalis iMX6Q       | 1GB           | V1.1B   | Yes            | Yes         | Yes             |
| Apalis iMX6D       | 1GB IT        | V1.1B   | Yes            | Yes         | Yes             |
| Apalis iMX6D       | 512MB         | V1.1B   | Yes            | Yes         | Yes             |
| Apalis T30         | 1GB           | V1.1B   | Yes            | Yes         | No              |
| Apalis T30         | 1GB IT        | V1.1B   | Yes            | Yes         | No              |
| Colibri iMX7D      | 1GB           | V1.1A   | Yes            | Yes         | Yes             |
| Colibri iMX6S      | 256MB         | V1.1A   | Yes            | Yes         | Yes             |
| Colibri iMX6S      | 256 MB IT     | V1.1A   | Yes            | Yes         | Yes             |
| Colibri iMX6D      | 512MB         | V1.1A   | Yes            | Yes         | Yes             |
| Colibri iMX6D      | 512MB IT      | V1.1A   | Yes            | Yes         | Yes             |

## Detailed Usage Information

Three docker tags are provided for the project:

- **latest** - latest release. Uses source-code from `master` branch of the
related projects.

- **next** - latest development version, has development tools and therefore is
larger. Uses source-code from `master-next` branch of the related projects.

- **nightly** - experimental version, not exactly built once a day.
Uses source-code from development branches. Expect things to break.

If you plan to keep data history over time, first create a volume.
It is highly recommended for lifetime estimation during development or pilot
stages:

```
docker volume create flash-data
```

Start the container in privileged mode, with a pseudo TTY, bind mount `/dev`
and `/var/run/dbus`, use the same network as the host and either share the PID
namespace with the host or get from another container. An example is provided:

For system-wide analysis (using volume to persist data):

```
docker run -dt --privileged=true \
    --name flash \
    -v /var/run/dbus:/var/run/dbus \
    -v /dev:/dev \
    -v /etc/issue:/etc/issue \
    --net=host \
    --pid=host \
    --mount type=tmpfs,destination=/var/volatile/log \
    --mount type=volume,source=flash-data,target=/app/.storage \
    toradexlabs/flash-analytics-tool
```

For system-wide analysis (without persistent data):

```
docker run -dt --rm --privileged=true \
    --name flash \
    -v /var/run/dbus:/var/run/dbus \
    -v /dev:/dev \
    -v /etc/issue:/etc/issue \
    --net=host \
    --pid=host \
    --mount type=tmpfs,destination=/app/.storage \
    --mount type=tmpfs,destination=/var/volatile/log \
    toradexlabs/flash-analytics-tool
```

For monitoring other container modify the --pid option:

```
From --pid=host to --pid=container:<container-name>
```

For instance, to monitor the write emulator container (without persistent data):

```
# Start the write emulator container - it wears out the flash over a long period running
docker run -d --rm --name=write_emulator -v /dev:/dev --privileged toradexlabs/flash-analytics-tool:emulator
# Start the Flash Analytics Tool
docker run -dt --rm --privileged=true \
    --name flash \
    -v /var/run/dbus:/var/run/dbus \
    -v /dev:/dev \
    -v /etc/issue:/etc/issue \
    --net=host \
    --pid=container:write_emulator \
    --mount type=tmpfs,destination=/app/.storage \
    --mount type=tmpfs,destination=/var/volatile/log \
    toradexlabs/flash-analytics-tool
```

Any chosen configuration provides a web UI at `http://<board_ip>`

## Core HTTP API

The Flash Analytics Tool data can be retrieved without the remote UI. See the
HTTP API documentation file [API.md](API.md).

## Directory structure

- ./src: core of the flash analytics tool and HTTP API implementation.
- ./resources: miscellaneous of tools, scripts and patches to be deployed in the
flash analytics tool container.
- ./sh: deploy script via SCP used during development.
- ./.vscode: VS Code configuration to run SCP deploy script on `build`.

## Detailed Tool Information

This directory contains the flash analytics tool core application, which consists
of a Python application that monitors file system I/O - specifically writes to
the storage device -and reads eMMC status related to lifespan.

### Application Structure

* main.py - this is the main entry point for the project.
* libs/ - directory where the application modules are.
  * api.py - HTTP API implementation.
  * clargs.py - module that parses the command-line options.
  * dbhandle.py - module that implements the database-agnostic part of raw data
  storage in-memory and backup to persistent storage.
  * lifespan.py - module that implements the vendor-agnostic part of reading and
  parsing eMMC lifespan information.
  * regmodels.py - library with functions for calculating data regression.
  * sysinfo.py - module that provides system information.
  * wrstats.py - module that monitors and analyze writes to the filesystem.
  * micron/ - diretory that contains Micron vendor-specific implementations.
    * lifespan.py - module that extends the generic implementation by adding
    vendor-spacific data.
  * database/ - directory that contains database specific DB implementation.
    * sqlite.py - SQLite implementation.
* db/ - directory that keeps DB schemas and lookup tables.
  * flash.sql - SQLite schema for the Flash Analytics Tool data.
  * tdxlut.sql - SQLite schema for SoM lookup table.
  * tdxlut.sqlite - SQLite DB file for SoM lookup table.