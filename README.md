# fanuc_ros_cgio

## Overview

This is a simple Fanuc Karel CGI-like program that allows access to controller
IOs through the web server running on a Fanuc controller.

**NOTE**: this is only meant as a convenience tool for quick and incidental
external access to a controller's IO without needing to use any additional
hardware. Do not use this on production systems or in contexts where any kind
of determinism is required. The author recommends using any of the supported
fieldbuses in those cases.


## Requirements

As this is written in Karel, option `R632` is a base requirement. In addition,
this needs the base *Web Server* (`HTTP`) and the *Web Server Enhancements*
(`R626`) options.

Other requirements include a functioning networking setup (make sure you can
ping the controller and the controller's website shows up when opening
`http://robot_ip/`), and correctly configured *HTTP Authentication* settings.
Either unlock the *KAREL* resource completely, set global credentials or add an
entry allowing access to the `ros_cgio` Karel program only. Refer to section
6.5 *HTTP AUTHENTICATION* of the *FANUC Robot series - Ethernet Function -
Operator's Manual* (document `B-82974EN/01` for the R-30iA) for more
information.


## Installation

Translate the `ros_cgio.kl` either with Roboguide or with the supplied
`Makefile`. The latter will require a compatible version of GNU Make for
Windows to be available.

Finally copy the resultant p-code file to the controller. Make sure to check
the web server security settings (see [Requirements](#Requirements)).

No further setup is required.


## Example usage

Use any HTTP capable client to interact with the program. See the
[Parameters](#Parameters) section for more information on parameters and their
values.

### Read

The following shows a *read* of the second *Digital output* port on the
controller:

```
$ curl -s "http://robot_ip/KAREL/ros_cgio?io_op=read&io_type=2&io_idx=2"
{"result" : "success", "op" : "read", "type" : "2", "idx" : "2", "value" : "1"}
```

The response shows the port is currently in the `ON` (1) state.

### Write

The following shows a *write* to the first *Robot digital output* port on
the controller:

```
$ curl -s "http://robot_ip/KAREL/ros_cgio?io_op=write&io_type=9&io_idx=1&io_val=1"
{"result" : "success", "op" : "write", "type" : "9", "idx" : "1", "value" : "1"}
```

The response shows the port was set to the `ON` (1) state.


## Parameters

The following parameters are used by the script. Note that for *reads*, the
`io_val` parameter is not required (it will be ignored if set).

### io_op

The type of operation to perform. Use `read` for reading, `write` for
writing.

### io_type

The 'type' of the IO port. This corresponds directly to the `port_type`
parameter of the `GET_PORT_VAL(..)` and `SET_PORT_VAL(..)` Karel routines,
and expects the same values. See `FR:\kliotyps.kl` for defined values.

Note that the current implementation supports reading and writing to *digital*
type IO ports only.

### io_idx

The numerical index into the port array that is to be read from or written to.
One-based. Corresponds directly to the indices shown on the various IO screens
on the teach pendant.

Invalid values will result in an error document returned.

### io_val

In case of a `write` operation, the value to be written. For *Digital output*
ports, `0` is `OFF`, anything else is `ON`. For *Grouped outputs*, the value
is copied directly into the output. Maximum value is limited by the width of
the group.

Note that this parameter is only required for *writes*, and is ignored in case
`io_op` is set to `read`.


## Return documents

Return documents are JSON encoded, and will include the result of the requested
operation (either `success` or `error`), as well as a copy of the request
parameters that were provided.

In case of an error, only a `reason` field is included in the document. The
following shows a *write* to a non-existing *Digital output* port and the
returned document:

```
$ curl -s "http://robot_ip/KAREL/ros_cgio?io_op=write&io_type=2&io_idx=1000&io_val=0"
{"result" : "error", "reason" : "Port write error: 13002"}
```

The reason includes the specific error code returned by the `SET_PORT_VAL`
Karel routine (in this case: `PRIO-002: Illegal port number`).


## Limitations / Known issues

The following limitations and known issues exist:

 - reads/writes only a single port at a time
 - only supports digital IO ports (`dout`, `rout`, etc)
 - no checks on values in case of writing (anything `> 0` will be interpreted
   as `ON`)
 - no checks on whether a WRITE was successful or not
 - HTTP headers returned will not reflect outcome of the request (ie:
   `HTTP/1.0 200 OK` is always returned, even if the requested operation
   failed)


## Bugs, feature requests, etc

Please use the [GitHub issue tracker][].



[GitHub issue tracker]: https://github.com/gavanderhoorn/fanuc_ros_cgio/issues
