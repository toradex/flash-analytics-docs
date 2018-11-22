# Flash Analytics Tool Core API

This readme provides the Flash Analytics Tool Core HTTP API documentation.

# Back-end API reference

The back-end API currently implements the following routes - system information,
write statistics, settings and lifespan information. All endpoints return JSON
encoded data.

Following the API calls are described:

The JSON are described in the format:

`'key': (type) [range if applicable] short description.`

Example:

`'mlc': (int) [0-100] percentage of lifespan for MLC cells`

Real data example would be:

`'mlc': 30`

## Base URL

The request should be made to the same IP of the board:

`var hostURL = window.location.hostname`
`"http://" + hostURL`

## System Information

`/sysinfo/os_version/`

Response:

```
{
    "os_version": (string) - string with OS version
}
```

`/sysinfo/kernel_version/`

Response:

```
{
    "kernel_version": (string) - usually "x.y.z", e.g. "4.18.7"
}
```

`/sysinfo/tool_version/`

Response:

```
{
    "flash_analytics_version": (string) - release number not fully defined
                but something as "0.4" or "0.4.1" can be expected
}
```

`/sysinfo/module_info/`

Response:

```
{
    "module_info": {
        "product": (string), - product name which already is a brief description
                    e.g. "Colibri iMX6S 256 MB IT"
        "revision": (string)", - product version, e.g "V1.1A"
        "serial": (string), - unique ID for each unit, e.g ""
    }
}
```

`/sysinfo/is_supp/`

Response:

```
{
    "supported": (integer) returns -1 if undefined
                                    0 if not supported
                                    1 if supported
}
```

`/sysinfo/ini_timestamp/`

Response:

```
{
    "initial_timestamp": (integer) returns -1 if undefined
                        otherwise returns number of seconds since epoch
}
```

## Write Statistics

`/writestats/writes_snapshot`

Returns data for the last timestamp recorded by the app

Response:

```
{
    'processes': (array) [n slots] [
        # each "slot" (position in the array) corresponds to a
        # running process on the system
        {
            'pid': (integer) [positive] process ID (PID) which is unique in the
                                        snapshot. Zero is not a valid PID.
            'pname': (string) process name. This is a non unique value since
                                more than one instance of a process can run at
                                the same time,
            'total_write': (decimal) the process total write to the filesystem
                                        in KiB
            'write_rate': (decimal) the process write-rate to the filesystem in
                                        KiB/s

         } ...,
    ],
    'timestamp': (integer) returned besides the array of processes, since it
                                the same value for all processes in the snapshot
}
```

`/writestats/writes_dump`

Returns all data recorded by the app

Response:

```
{
    'processes': (array) [n slots] [
        # each "slot" (position in the master array) corresponds to a
        # running process on the system at a specific point in time.
        # A process may have several entries, but for a specific process,
        # the 'timestamp' and 'total_write' fields are unique.
        {
            'pid': (integer) [positive] process ID (PID). There may be n entries
                                        with the same PID.
            'pname': (string) process name. There may be n entries with the same
                                process name.
            'timestamp': (integer) the timestamp for the current entry
            'total_write': (decimal) the process total write to the filesystem
                                        in KiB
            'write_rate': (decimal) the process write-rate to the filesystem in
                                        KiB/s

         } ...,
    ]
}
```

`/writestats/writes_window/<time_ini>/<time_end>`

Returns all data recorded by the app within a time window defined by two points
in time.

time_ini: (integer) initial time of the time window defined as seconds since epoch
time_end: (integer) final time of the time window defined as seconds since epoch,
                        must be equal or greater than time_ini

E.g. for time_ini = 1539045335; time_end = 1539045348:

/writestats/writes_window/1539045335/1539045348

Response:

Same as in _/writestats/writes_dump_.

`/writestats/writes_lastsecs/<num_secs>`

Returns all data recorded by the app between current time and n seconds before.

num_secs: (integer) the amount of seconds for the time window.

E.g.for num_secs = 30:

/writestats/writes_lastsecs/30

Response:

Same as in _/writestats/writes_dump_.

## Settings

`/settings/bkp_status`

Return status of backup from RAM to flash.

Response:

```
{
    "enabled": (boolean) "true" if backup to flash is enabled (ON), "false"
                                if the backup is disabled (OFF).
    "status": (integer) 0 (zero) if no active backup or 1 (one) if a
                                backup is ongoing. It is almost always zero.
    "progress": (decimal) percentage of backup done, in case of active backup.
}
```

`/settings/bkp_on_off/<on_off>`

on_off: (integer) 0 to disable backup to flash, 1 to enable.

Response:

```
{
    "ret": (integer) 0 on success, -1 on failure
}
```

## Lifespan Information

`/lifespan` (this endpoint will be kept until migration to the new API
                finished. Please see '/lspan/<endpoint>' for new API info)

Response:

```
{
    'jedec': {
        'mlc': (integer) [0 - 100] percentage of lifespan for MLC cells,
        'slc': (integer) [0 - 100] percentage of lifespan for SLC cells,
        'pre_eol': (string) self-explaining message about lifespan status with
                respect to reserved blocks. Values are:
                "Not defined" - temporary value mostly seen during
                                    initialization
                "Unexpected value" - error value
                "Normal - consumed less than 80% of the reserved blocks"
                "Warning - consumed 80% of the reserved blocks"
                "Urgent - consumed 90% of the reserved blocks"
    },
    'vendor': {
        'bad_block_count': (array) [3 slots] [
            (integer) - initial bad blocks - the number of bad blocks from
                            factory
            (integer) - runtime bad blocks - the number of bad blocks since
                            started using the eMMC
            (integer) - spare blocks - the number of spare blocks available
        ],
        'block_erase_count': {
            'total': (array) [3 slots] [
                (integer) - minimum block erase count among all blocks
                (integer) - maximum block erase count among all blocks
                (integer) - average block erase count among all blocks
            ],
            'slc': (array) [3 slots] [
                (integer) - minimum block erase count among SLC blocks
                (integer) - maximum block erase count among SLC blocks
                (integer) - average block erase count among SLC blocks
            ],
            'mlc': (array) [3 slots] [
                (integer) - minimum block erase count among MLC blocks
                (integer) - maximum block erase count among MLC blocks
                (integer) - average block erase count among MLC blocks
            ],
        },
        'block_erase_info_and_type': (array of arrays) [n slots] [
            # the inner array will have a predefined length for each eMMC type
            # that means this number is fixed for every different Toradex
            # hardware.
            # for Colibri iMX6, Apalis iMX6 and Apalis iMX7 it is 1024 blokcs
            [
                (integer) - the physical block address for this block
                (integer) - erase count for this block
                (integer) - this block type - 0 means "SLC", 1 means "MLC"
            ] ...
        ],
        'bad_block_info': (array of arrays or arrays ) [n slots - n slots] [
        # I have to improve this part of the API in the back-end code
            # parent array usually have something such as 4 arrays
            # they can be concatenated, since they are originally split due to
            # hardware request limitation
            [
                # each array here correspondo to a single block information
                [
                    (integer) - physical bad block address record
                    (integer) - failed page address record
                    (integer) - 0 means "erase failed", 1 means "program failed"
                    (integer) - failed bus number
                ] ...
            ] ...
        ]
    },
    'analysis': {
        'jedec': { --> exactly the same as 'jedec'
            'mlc': (integer) [0 - 100] percentage of lifespan for MLC cells,
            'slc': (integer) [0 - 100] percentage of lifespan for SLC cells,
            'pre_eol': (string) self-explaining message about lifespan status
                    with respect to reserved blocks. Values are:
                    "Not defined" - temporary value mostly seen during
                                        initialization
                    "Unexpected value" - error value
                    "Normal - consumed less than 80% of the reserved blocks"
                    "Warning - consumed 80% of the reserved blocks"
                    "Urgent - consumed 90% of the reserved blocks"
        },
        'vendor': {
            'percent_bad_blocks': (decimal) percentage of bad blocks - higher is
                                                worse,
            'count_slc_blocks': (integer) number of blocks configured as SLC,
            'count_mlc_blocks': (integer) number of blocks configured as MLC
        }
    }
}
```

`/lspan/health_summary_snapshot`

Return the latest health summary data logged by the system.

Response:

```
{
    '<timestamp>': (object of objects) the 'timestamp' is a string representing
            the snapshot timestamp in seconds since epoch. Each timestamp has
            the health summary at a given point in time.
    {
        'bad_block_count': {
            'initial': (integer) - initial bad blocks - the number of bad blocks
                                from factory
            'runtime': (integer) - runtime bad blocks - the number of bad blocks
                                since started using the eMMC,
            'spare': (integer) - spare blocks - the number of spare blocks available
        },
        'block_erase_count': {
            'mlc_avg': (integer) - average block erase count among MLC blocks,
            'mlc_max': (integer) - maximum block erase count among MLC blocks,
            'mlc_min': (integer) - minimum block erase count among MLC blocks,
            'mlc_sum': (integer) - sum of block erase count among MLC blocks,
            'slc_avg': (integer) - average block erase count among SLC blocks,
            'slc_max': (integer) - maximum block erase count among SLC blocks,
            'slc_min': (integer) - minimum block erase count among SLC blocks,
            'slc_sum': (integer) - sum of block erase count among SLC blocks,
            'total_avg': (integer) - average block erase count among all blocks,
            'total_max': (integer) - maximum block erase count among all blocks,
            'total_min': (integer) - minimum block erase count among all blocks,
            'total_sum': (integer) - sum of block erase count among all blocks
        },
        'life_a_slc': (integer) [0 - 100] percentage of lifespan for SLC cells,
        'life_b_mlc': (integer) [0 - 100] percentage of lifespan for MLC cells,
        'pre_eol_info': (string) self-explaining message about lifespan status
                        with respect to reserved blocks. Values are:
                        "Not defined" - temporary value mostly seen during
                                            initialization
                        "Unexpected value" - error value
                        "Normal - consumed less than 80% of the reserved blocks"
                        "Warning - consumed 80% of the reserved blocks"
                        "Urgent - consumed 90% of the reserved blocks"
    }, ... (for the snapshot API, a single timestamp is returned)
}
```

`/lspan/health_summary_dump`

Return all latest health summary entries. It is basically a parent object
containing one child object for each snapshot stored over time.

Response:

Same as in _/lspan/health_summary_snapshot_.

`/lspan/health_summary_window/<time_ini>/<time_end>`

Returns all data recorded by the app within a time window defined by two points
in time.

time_ini: (integer) initial time of the time window defined as seconds since epoch
time_end: (integer) final time of the time window defined as seconds since epoch,
                        must be equal or greater than time_ini

E.g. for time_ini = 1539045335; time_end = 1539045348:

/lspan/health_summary_window/1539045335/1539045348

Response:

Same as in _/lspan/health_summary_snapshot_.

`/lspan/health_summary_lastsecs/<num_secs>`

Returns all data recorded by the app between current time and n seconds before.

num_secs: (integer) the amount of seconds for the time window.

E.g.for num_secs = 30:

/lspan/health_summary_lastsecs/30

Response:

Same as in _/lspan/health_summary_snapshot_.

`/lspan/emmc_blk_snapshot`

Return the latest per-block eMMC data logged by the system.

Response:

```
{
    '<timestamp>': (object of objects) the 'timestamp' is a string representing
            the snapshot timestamp in seconds since epoch. Each timestamp has at
            least one block info, and the latest timestamp has all blocks info.
    {
        '<block_physical_address>': (object) the 'block_physical_address' is a
            string containing the physical address of the current eMMC block.
        {
            'bad_block_info': (string) can be 'healthy', 'erase error' or
                                'program error',
            'erase_count': (integer) erase count for the current block,
            'failed_bus_number': (null or integer) null if block is 'healthy'
                        or integer if the block is not healthy,
            'failed_page_address': (null or integer) null if block is 'healthy'
                        or integer if the block is not healthy,
            'type': (string) either 'MLC' or 'SLC'
        }, ...
    }, ... (for the snapshot API, a single timestamp is returned)
}
```

`/lspan/emmc_blk_dump`

Return all per-block eMMC data entries. Only the latest timestamp has all blocks
information, for other timestamps only the blocks that had a change from previous
timestamp are provided. Nevertheless, all blocks do exist for any given timestamp.

Response:

Same as in _/lspan/emmc_blk_snapshot_.

`/lspan/emmc_blk_window/<time_ini>/<time_end>`

Return all per-block eMMC data entries recorded by the app within a time window
defined by two points in time. Only the latest timestamp has all blocks
information, for other timestamps only the blocks that had a change from previous
timestamp are provided. Nevertheless, all blocks do exist for any given timestamp.

time_ini: (integer) initial time of the time window defined as seconds since epoch
time_end: (integer) final time of the time window defined as seconds since epoch,
                        must be equal or greater than time_ini

E.g. for time_ini = 1539045335; time_end = 1539045348:

/lspan/emmc_blk_window/1539045335/1539045348

Response:

Same as in _/lspan/emmc_blk_snapshot_.

`/lspan/emmc_blk_lastsecs/<num_secs>`

Return all per-block eMMC data entries recorded by the app between current time
and n seconds before. Only the latest timestamp has all blocks
information, for other timestamps only the blocks that had a change from previous
timestamp are provided. Nevertheless, all blocks do exist for any given timestamp.

num_secs: (integer) the amount of seconds for the time window.

E.g.for num_secs = 30:

/lspan/emmc_blk_lastsecs/30

Response:

Same as in _/lspan/emmc_blk_snapshot_.

`/lspan/life_est`

Return the lifetime estimation based in the linear regression of the per-block
eMMC total erase count for MLC blocks and the expected average lifetime of an 
eMMC block configured as MLC.

Response:

```
{
    "lifetime_estimation": (integer) expected eMMC end-of-life in seconds since
                                        epoch
}
```
