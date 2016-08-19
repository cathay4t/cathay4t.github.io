---
title: Plan of Zatel project
layout: default
---

**Still planning, no real code/demo yet, just noting down some ideas**

* TOC
{:toc}

## 1. Layout ##

 * `zateld` for daemon listening on UNIX sockets.
 * `zatel-dbus` for dbus.
 * `zatel-rest` for REST and WebSocket.
 * `zatel-cli` for CLI command.
 * `zatel-clib` for C library client API.
 * `python-zatel` for Python library client API.
 * `zatel-plugin-lsm` plugin for LSM.
 * `zatel-plugin-multipath` plugin for multipath.

## 2. Features ##

 * Use cache to eliminate duplicate call.
 * Use udev(or better uevent) for change monitoring.
 * Plugins could add new property to any existing interface.
 * No license restriction on plugins.
   (Plugins are running in separate PID, only communicate with daemon via
    IPC socket)

## 3. Quick user guide ##

### zateld config ###

```
main.enable_plugins = yes|no
lsm.uri.ontap = ontap+ssl://root@filer-a.ontap.example.com
lsm.password.ontap = noidea
lsm.uri.emc = smispy+ssl://admin@smis.provider.example.com?no_ssl_verify=yes
lsm.password.emc = noidea
```

### zatel-cli example ###

### zatel-dbus example ###

### zatel-rest example ###

## 4. Lower-level(socket) API Document ##

UNIX socket address `\0/zatel/v0.1` for data query and change.
UNIX socket address `\0/zatel/v0.1/notify` for notification.

For data query and change, client just send command like 'disk list' to
socket, then will receive data in JSON format.

### 4.1. Notification ###

Notification Socket address: `\0/zatel/v0.1/notify`.
Notification JSON format:

```json
{
    "notify_type":      "ADD|CHANGE|DEL|FAIL|WARN|PREFAIL",
    "notify_msg":       "fooa happend cause foob into fooc state",
    "data_type":        "disk|part|pv|vg|lv|etc",
    "data_id":          "<id>"
}
```

### 4.2. Disk ###

TODO: Should support NVMe also.

#### 4.2.1. Query Disks ####

Command: `disk list [<disk_id>]`

```json
{
    "errno":                0,
    "strerror":             "OK",
    "errmsg":               "",
    "disk_id":              "<vpd83_of_disk>",
    "blk_paths":            ["/dev/sda", "/dev/sdb", "/dev/sdc"],
    "opt_io_size":          4096,
    "min_io_size":          512,
    "logical_sector_size":  512,
    "physical_sector_size": 4096,
    "lsm_raid_type":        "SINGLE_DISK",
    "lsm_is_hw_raid":       false,
    "lsm_is_san_disk":      false,
    "lsm_volume_id":        "",
    "interface_type":       "ATA|SAS|SCSI|FC|iSCSI|NVMe",
    "interface_speed":      "6Gbps",
    "rotation_speed":       10000,
    "part_ids":             ["<id_of_part>", ],
    "mpath_id":             "<id_of_mpath>",
    "fs_id":                "<id_of_fs>",
    "pv_id":                "<id_of_pv>"
}
```

#### 4.2.2.Turn On/Off Disk Identification LED ####

Command: `disk led ident [on|off] <id_of_disk>`

```json
{
    "errno":                1,
    "strerror":             "NO_SUPPORT",
    "errmsg":               "Specified disk does not support this action"
}
```

#### 4.2.3. Turn On/Off Disk Fault LED ####

Command: `disk led fault [on|off] <id_of_disk>`

```json
{
    "errno":                1,
    "strerror":             "NO_SUPPORT",
    "errmsg":               "Specified disk does not support this action"
}
```

#### 4.2.4. Query partitions ####

TODO: Need find a way to allow multipath override partition's `blk_path`.
TODO: Need new class for partition so that filesystem and lvm pv could link
      against it.
TODO: Need a new way to allow search `blk_path`, return `multipath list`

Command: `disk part list <id_of_disk>`

For multipath disk, use command `multipath part list <mpath_id>` in stead.

```json
{
    "errno":                0,
    "strerror":             "OK",
    "errmsg":               "",
    "part_id":              "todo, find a good id for partition",
    "disk_id":              "<disk_id>",
    "blk_path":             "/dev/sda1",
    "part_type":            "GPT|DOS",
    "start_sector":         1024,
    "end_sector":           1024000
}
```
### 4.2.5. Create Partitions ####
Command:
`disk part create <disk_id> <start_sector> <end_sector> [<auto_align_yes>]`

```json
{
    "errno":                0,
    "strerror":             "OK",
    "errmsg":               "",
    "part_id":              "<part_id>",
    "blk_path":             "<blk_path>"
}
```

### 4.3. LVM ###

#### 4.3.1. Query PVs ####

Command: `lvm list pv [<pv_id>]`

```json
{
    "errno":                0,
    "strerror":             "OK",
    "errmsg":               "",
    "id":                   "<uuid of PV>",
    "backstore_blk_path":   "/dev/mapper/mpatha",
    "vg_id":                "<id_of_vg>"
}
```

### 4.4. File System ###

#### 4.4.1. Query File Systems ####

#### 4.4.2. Format File System ####

#### 4.4.3. Mount File System ####

#### 4.4.4. Umount File System ####

#### 4.4.5. Check File System ####

### 4.5. Multipath ###
#### 4.5.1. Query Multipath Maps ###

Handle by plugin -- `zatel-plugin-multipath`.

Command: `multipath list [id]`

```json
{
    "errno":                0,
    "strerror":             "OK",
    "errmsg":               "",
    "id":                   "<uuid of map>",
    "disk_id":              "<id of disk>",
    "blk_path":             "/dev/mapper/mpatha",
    "name":                 "mpatha",
    "paths":                ["/dev/sda", "/dev/sdb", "/dev/sdc", "/dev/sdd"],
    "path_status":          ["faulty", "ready", "ready", "ready"],
    "_add_disk_multipath_id":   "id",
    "_overide_disk_blk_path": "blk_path"
}
```

The `_add_disk_multipath_id` property means append `mutlipath_id` property
using the value of `id` property to disk object when `disk_id` matches.

The `_overide_disk_blk_path` proeprty will overide `blk_path` property of
disk object when `disk_id` matches.

## 5. Dbus API Document ##

## 6. REST API Document ##

## 7. Python API Document ##

### 7.1. Python Client API ###

### 7.2. Python Plugin API ###

## 8. C API Document ##

### 8.1. C Client API ###

### 8.2. C Plugin API ###

## 9. Dev notes ##

### 9.1. How to broadcast notifications received from plugin to clients ###
Daemon listens on two socket address:

* Plugins will connect to `\0/zatel/v0.1/plugin_notify` and send
  notification to this address.
* Clients will connect to `\0/zatel/v0.1/notification`, daemon will
  forward plugins notifications to client.

### 9.2. How plugin add new properties to other interfaces ###

After got register request from plugin, daemon will invoke command
`<plugin_name> list`, from its reply. Daemon will:

 * Plugin should provide `<target_class_name>_id` property to indicate
   which class it is targeting.
 * Add property to requested object for
   `_add_<class_name>_<plugin_name>_<prop_name>`.
 * Overide property of requested object for
   `_overide_<class_name>_<prop_name>`.

Check Multipath class/plugin for example.

### 9.3. How could plugin inform daemon on data changes ###

Plugins will connect to `\0/zatel/v0.1/plugin_notify` and send notification to
this address:

```json
{
    "notify_type":      "ADD|CHANGE|DEL|FAIL|WARN|PREFAIL",
    "notify_msg":       "fooa happend cause foob into fooc state",
    "data_type":        "disk|part|pv|vg|lv|etc",
    "data_id":          "<id>"
}
```

Then daemon will use command `<plugin_name> list <id>` to get latest status
of certain entry if the change is `ADD|CHANGE|FAIL|WARN|PREFAIL`.

### 9.4. How could daemon inform plugin on data changes ###

Plugin could listen on `\0/zatel/v0.1/notify` like normal client and handle
interested changes.
