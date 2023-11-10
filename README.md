# Quick reference

Source code: [GitHub](https://github.com/jforge/modbus-server)

Container image: [DockerHub](https://hub.docker.com/r/jforge/modbus-server)

# Supported tags and respective `Dockerfile` links

* [`latest`, `1.1.5`](https://github.com/jfirge/modbus-server/blob/v1.1.5/Dockerfile)


# What is Modbus TCP Server?

The Modbus TCP Server is a simple, in python written, Modbus TCP server.
The Modbus registers can be also predefined with values.

The Modbus server was initially created to act as a Modbus slave mock system
for enhanced tests with modbus masters and to test collecting values from different registers.

The Modbus specification can be found here: [PDF](https://modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf)


# QuickStart with Modbus TCP Server and Docker

Step - 1 : Pull the Modbus TCP Server

```bash
docker pull jforge/modbus-server
```

Step - 2 : Run the Modbus TCP Server

```bash
docker run --rm -p 5020:5020 jforge/modbus-server:latest
```

Step - 3 : Predefine registers

The default configuration file is configured to initialize every register with a `0x0000`.
To set register values, you need to create your own configuration file.

```bash
docker run --rm -p 5020:5020 -v ./server_config.json:/server_config.json jforge/modbus-server:latest -f /server_config.json
```

or you mount the config file over the default file, then you can skip the file parameter:

```bash
docker run --rm -p 5020:5020 -v ./server_config.json:/app/modbus_server.json jforge/modbus-server:latest
```

# Configuration

## Configuration file

The path and filename to the general configuration file can be set via command option `-f <file>` after mounting it inside of the container. By default, the script will use `/app/modbus_server.json`.

### Default configuration file of the container

The `/app/modbus_server.json` file comes with following content:

```json
{
"server": {
  "listenerAddress": "0.0.0.0",
  "listenerPort": 5020,
  "tlsParams": {
    "description": "path to certificate and private key to enable tls",
    "privateKey": null,
    "certificate": null
    },
  "logging": {
    "format": "%(asctime)-15s %(threadName)-15s  %(levelname)-8s %(module)-15s:%(lineno)-8s %(message)s",
    "logLevel": "INFO"
    }
  },
"registers": {
  "description": "initial values for the register types",
  "zeroMode": false,
  "initializeUndefinedRegisters": true,
  "discreteInput": {},
  "coils": {},
  "holdingRegister": {},
  "inputRegister": {}
  }
}
```

### Field description

| Field                                    | Type    | Description                                                                                                           |
|------------------------------------------|---------|-----------------------------------------------------------------------------------------------------------------------|
| `server`                                 | Object  | Modbus slave specific runtime parameters.                                                                             |
| `server.listenerAddress`                 | String  | The IPv4 Address to bound to when starting the server. `"0.0.0.0"` let the server listens on all interface addresses. |
| `server.listenerPort`                    | Integer | The TCP port number of the modbus slave to listen to.                                                                 |
| `server.tlsParams`                       | Object  | Configuration parameters to use TLS encrypted modbus tcp slave. (untested)                                            |
| `server.tlsParams.description`           | String  | No configuration option, just a description of the parameters.                                                        |
| `server.tlsParams.privateKey`            | String  | Filesystem path of the private key to use for a TLS encrypted communication.                                          |
| `server.tlsParams.certificate`           | String  | Filesystem path of the TLS certificate to use for a TLS encrypted communication.                                      |
| `server.logging`                         | Object  | Log specific configuration.                                                                                           |
| `server.logging.format`                  | String  | The format of the log messages as described here: https://docs.python.org/3/library/logging.html#logrecord-attributes |
| `server.logging.logLevel`                | String  | Defines the maximum level of severity to log to std out. Possible values are `DEBUG`, `INFO`, `WARN` and `ERROR`.     |
| `registers`                              | Object  | Configuration parameters to predefine registers.                                                                      |
| `registers.description`                  | String  | No configuration option, just a description of the parameters.                                                        |
| `registers.zeroMode`                     | Boolean | By default the modbus registers starts at 1 (`false`) but some implementation requires to start at 0 (`true`).        |
| `registers.initializeUndefinedRegisters` | Boolean | If `true` the server will initialize all not defined registers with a default value of `0`.                           |
| `registers.discreteInput`                | Object  | The pre-defined registers of the register type "Discrete Input".                                                      |
| `registers.coils`                        | Object  | The pre-defined registers of the register type "Coils".                                                               |
| `registers.holdingRegister`              | Object  | The pre-defined registers of the register type "Holding Registers".                                                   |
| `registers.inputRegister`                | Object  | The pre-defined registers of the register type "Input Registers".                                                     |

### Pre-define Registers within the configuration file

Pre-define registers always starts with the register number. We use a json format as configuration file, so the "key" needs to be a string. So, the register number needs also to be a string. During server initialization, the json key that represents the register number will be converted to an integer.

As by the modbus spec, the "Discrete Input" and "Coils" registers contains a single bit. In the json configuration file, we use `true` or `false` as register values.

Example configuration of pre-defined registers from type "Discrete Input" or "Coils":
```
[..]
  "<register_type>": {
    "0": true,
    "1": true,
    "42": true,
    "166": false
  }
[..]
```

As by the modbus spec, the "Holding Registers" and "Input Registers" tables contains a 16-bit word. In the json configuration file, we use a hexadecimal notation, starting with `0x`, as register values.

Example configuration of pre-defined registers from type "Holding Registers" or "Input Registers":
```
[..]
  "<register_type>": {
    "9": "0xAA00",
    "23": "0xBB11",
    "142": "0x1FC3",
    "2346": "0x00FF"
  }
[..]
```


## Configuration file examples

- [src/app/modbus_server.json](https://github.com/jforge/modbus-server/blob/main/src/app/modbus_server.json)
- [examples/abb_coretec_example.json](https://github.com/jforge/modbus-server/blob/main/examples/abb_coretec_example.json)
- [examples/test.json](https://github.com/jforge/modbus-server/blob/main/examples/test.json)



# Docker compose configuration

```yaml
version: '3.8'

services:
  modbus-server:
    container_name: modbus-server
    image: jforge/modbus-server:latest
    restart: always
    command: -f /server_config.json
    ports:
      - 5020:5020
    volumes:
      - ./server.json:/server_config.json:ro
```
