# Flash Analytics Tool

The Flash Anasytics Tool is a means to provide the estimated flash memory
lifespan of Toradex modules, based in customer usage of the flash. It is 
suitable for flash usage profiling during development and for predictive
hardware replacement during the lifespan of the CoM.

## Getting Started

Please see Getting Started readme:

* [docs/GETTING_STARTED.md](docs/GETTING_STARTED.md)

## Tools

The Flash Analytics Tool project comprises a set of modules and tools:

### Flash Analytics Tool Core

Application that monitors the writes to the flash and the eMMC health status.
It can monitor overall system writes as well as single application writes.

### Flash Analytics Tool Remote UI

A remote user interface that provides both concise and comprehensive information
about eMMC health and write statistics.

### Flash Analytics Tool HTTP API

API consumed by the remote UI, can be used standalone. Refer to the
[docs/API.md](docs/API.md) documentation for more information.

### Flash Write Emulator

Application that writes to the flash, RAM and a TTY device for debugging
purposes.
