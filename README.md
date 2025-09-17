## RakNet Notes

-   All strings are prefixed with an unsigned short depicting their length.
-   The offline message id will always be: 00ffff00fefefefefdfdfdfd12345678 (hex) - this series of bytes will be referred to as _Magic_
-   The offline message id is sent with unconnected messages such as unconnected pings and pongs.
-   The GUIDS used by RakNet are 8 bytes long.
-   The first byte is used to identify the type of message.

## Data Types

| Type           | Size | Range           | Notes                                                          |
| -------------- | ---- | --------------- | -------------------------------------------------------------- |
| Byte           | 1    | 0-255           | An unsigned integer                                            |
| Long           | 8    | -2^63 to 2^63-1 | Signed 64 bit integer                                          |
| Magic          | 16   |                 | 00ffff00fefefefefdfdfdfd12345678 - Will always be those bytes  |
| Short          | 2    | -32768 to 32767 |                                                                |
| Unsigned Short | 2    | 0 to 65535      |                                                                |
| String         | N/A  | N/A             | A string prefixed by a short which depicts the length.         |
| Boolean        | 1    | 0-1             | 0x00 is False while 0x01 is True                               |
| Address        | 7    |                 | 1 byte for the ip version 4/6, 4 for the IP and 2 for the port |
| uint24le       | 3    |                 | 3-byte little-endian unsigned integer                          |

## Contents

-   [x] Unconnected Pings
-   [x] Unconnected Pongs
-   [x] Open Connection Request 1
-   [x] Open Connection Reply 1
-   [x] Open Connection Request 2
-   [x] Open Connection Reply 2
-   **From here on, the RakNet connection is established and all RakNet messages are contained in a [Frame Set Packet](https://wiki.vg/Raknet_Protocol#Frame_Set_Packet).**
-   [x] Connection Request
-   [x] Connection Request Accepted
-   [x] New Incoming Connection

### Unconnected Pings

Minecraft Bedrock will send out a message to all listed servers (and the local network) to check if any games are available and retrieve the MOTD from the game. These messages are known as unconnected pings and are structured in this format:

`0x01 | client alive time in ms (unsigned long long) | magic | client GUID`

### Unconnected Pongs

After this message, the server will respond with something called an unconnected pong. The reason these messages are unconnected is because the client has not established a connection to the server. This is the format of an unconnected pong:

`0x1c | client alive time in ms (recorded from previous ping) | server GUID | Magic | string length | Edition (MCPE or MCEE for Education Edition);MOTD line 1;Protocol Version;Version Name;Player Count;Max Player Count;Server Unique ID;MOTD line 2;Game mode;Game mode (numeric);Port (IPv4);Port (IPv6);`

Example:

`MCPE;Dedicated Server;527;1.19.1;0;10;13253860892328930865;Bedrock level;Survival;1;19132;19133;`

The client doesn't seem to use the gamemode or the numeric value for the gamemode.

### Open Connection Request 1 |→ Client→Server

The client sends this when attempting to join the server

`0x05 | Magic | Protocol version	(currently 11 or 0x0b) | RakNet Null Padding`

The null padding seems to be used to discover the maximum packet size the network can handle.

The client will send this to the server with decreasing null padding until the server responds with a

### Open Connection Reply 1 |→ Server→Client

The server responds with this once the client attempts to join

`0x06 | magic | server GUID | ServerHasSecurity (boolean) | Cookie (uint32, if server has security) | MTU Size (Unsigned short)`

This is the first half of the handshake between the client and the server.

### Open Connection Request 2 |→ Client→Server

The client responds with this after they receive the open connection reply 1 packet.

`0x07 | magic | Cookie (uint32, if server has security) | Client supports security (Boolean(false), always false for the vanilla client, if server has security) | server Address | MTU Size (Unsigned short) | client GUID (Long)`

### Open Connection Reply 2 |→ Server→Client

This is the last part of the handshake between the client and the server.

`0x08 | magic | server GUID (Long) | client Address | MTU Size | security(Boolean)`

**From here on, all RakNet messages are contained in a [Frame Set Packet](https://wiki.vg/Raknet_Protocol#Frame_Set_Packet).**

### Connection Request |→ Client→Server

This is the part where the client sends the connection request.

`0x09 | client GUID (Long) | Request timestamp (Long) | Secure (Boolean)`

### Connection Request Accepted |→ Server→Client

The server sends this packet in response to the incoming connection request.

`0x10 | client Address | System index (Short, unknown what this does. 0 works as a value (Minecraft client sends 47)) | System adresses ([]Address) | Ping time (Long) | Pong Time (Long)`

### New Incoming Connection |→ Client→Server

Our RakNet connection is now fully successful.

`0x13 | server Address | internal Adress ([20(maybe 10)]Address) (i use 255.255.255.255:0) | Ping time (Long) | Pong Time (Long)`

### Note:

The next 2 packets ( and the first Minecraft Protocoll packet) are all send together as one by the vanilla client.
The RakNetProtocoll allows for them to be sent seperately too,
however servers with a custom raknet implementation might not always handle this case
because this never occurs within the vanilly client.

### New Incomming Connection

The client sends this packet in response to Connection Request Accepted.

`0x13 | serverAddress (uint8) | clientMachineAddresses (address[10], Minecraft sends only one ipv6 together with a placeholder (see below) instead of the other 9) | clientSendTime (uint64) | serverSendTime (uint64)`

placehodler for the other clientMachineAddresses (a.k.a. Internal Adresses):

> 0xd4 0x0b 0xa7 0x86 0xdd 0x98 0x33 0x00 0x00
> each byte replaces one of the 9 missing clientMachineAdresses

After having sent this packet,
you must periodically sent a Connected Ping to keep the connection alive.
The server also sometimes sends a Connected Ping, respond with a Connected Pong.

### Connected Ping

The client sends this packet immeadiatly after/with New Incomming Connection.
This packet should be sent as unreliable.
The client/server will respond to this with a Connected Pong.

`0x00 | Time since start (uint64)`

### Connected Pong

The client or server sends this packet after having received a Connected Ping.
This packet should be sent as unreliable.

`0x00 | Time since start client (uint64) | Time since start server (uint64)`

## Sources

::: tip
If you are interested and want to read more about it here is the documentation for the Bedrock Protocol and RakNet:

[Mojang's Official Protocol Documentation](https://github.com/Mojang/bedrock-protocol-docs)

[RakNet Protocol Documentation](https://wiki.vg/Raknet_Protocol)

[another RakNet Protocol Documentation](https://github.com/vp817/RakNetProtocolDoc)
:::

Thanks to these guys : 
     ZestiiSpaghett
     MedicalJewel105
     SmokeyStack
     ThomasOrs
     Adrian8115
     ismaileke
     Tom-Teclador


# Drakensang RakNet Server Protocol Analysis

## Overview

This document analyzes the RakNet protocol implementation for a Drakensang Online server, focusing on the packet structures and message flow observed through Wireshark captures.

## Key Protocol Messages

### Message 0x88 OR 0x84 - Map Change Notification

**Wireshark Capture:**
```
13821	95.212760247	47.245.158.101	192.168.1.47	RakNet	93	#4: Unknown message ID: 0x88, Unknown message ID: 0x84
```

**Raw UDP Payload:**
```
0000   18 c0 4d 9c 45 f6 b4 9d fd 5e 08 1f 08 00 45 00   ..M.E....^....E.
0010   00 4f 7f fe 40 00 35 11 35 6e 2f f5 9e 65 c0 a8   .O..@.5.5n/..e..
0020   01 2f 08 8e 87 7e 00 3b e8 78 84 04 00 00 60 00   ./...~.;.x....`.
0030   08 02 00 00 03 00 00 00 88 60 00 c9 03 00 00 02   .........`......
0040   00 00 00 84 70 00 14 00 34 37 2e 32 34 35 2e 31   ....p...47.245.1
0050   35 36 2e 32 32 35 3a 33 30 33 32 36 00            56.225:30326.
```

### Packet Structure Analysis

#### Frame Set Header
- **Byte 0**: `0x84` - Frame Set packet identifier
- **Bytes 1-3**: `0x04 0x00 0x00` - Frame Set number (4)
- **Byte 4**: `0x60` - Message flags (reliable ordered)
- **Bytes 5-6**: `0x00 0x08` - Payload length (8 bytes)

#### Message 0x88 (Map Change Notification)
- **Byte 7**: `0x02` - Reliable message number
- **Bytes 8-10**: `0x00 0x00 0x03` - Ordering index
- **Bytes 11-13**: `0x00 0x00 0x00` - Ordering channel + padding
- **Byte 14**: `0x88` - Message ID (Map Change)
- **Bytes 15-16**: `0x60 0x00` - Length field (96 bits = 12 bytes)
- **Bytes 17-18**: `0xc9 0x03` - Unknown data (possibly map ID or flags)
- **Bytes 19-20**: `0x00 0x00` - Padding
- **Bytes 21-22**: `0x02 0x00` - Additional flags or sequence number
- **Bytes 23-24**: `0x00 0x00` - Padding

#### Message 0x84 (Server Address)
- **Byte 25**: `0x84` - Message ID (Server Address)
- **Bytes 26-27**: `0x70 0x00` - Length field (112 bits = 14 bytes)
- **Bytes 28-29**: `0x14 0x00` - String length (20 bytes)
- **Bytes 30-49**: `47.245.156.225:30326` - Server IP address and port (ASCII)
- **Byte 50**: `0x00` - Null terminator

## Map Loading Sequence

### Message 0x86 + 0x88 (Map Load)

**Wireshark Capture:**
```
4669	47.555197566	47.245.158.101	192.168.1.47	RakNet	98	#4: Unknown message ID: 0x86, Unknown message ID: 0x88
```

**Raw UDP Payload:**
```
0000   18 c0 4d 9c 45 f6 b4 9d fd 5e 08 1f 08 00 45 00   ..M.E....^....E.
0010   00 54 5f f2 40 00 35 11 55 75 2f f5 9e 65 c0 a8   .T_.@.5.Uu/..e..
0020   01 2f 08 90 a6 ef 00 40 29 a0 84 04 00 00 60 00   ./.....@).....`.
0030   f1 02 00 00 02 00 00 00 86 0a 00 61 30 30 30 30   ...........a0000
0040   5f 63 68 61 72 0a 00 61 30 30 30 30 5f 63 68 61   _char..a0000_cha
0050   72 ff ff ff ff 00 00 60 00 08 03 00 00 03 00 00   r......`........
0060   00 88                                             ..
```

### Combined Message Structure

#### Frame Set Header
- **Byte 0**: `0x84` - Frame Set packet identifier
- **Bytes 1-3**: `0x04 0x00 0x00` - Frame Set number (4)
- **Byte 4**: `0x60` - Message flags (reliable ordered)
- **Bytes 5-6**: `0x00 0xf1` - Payload length (241 bits = 30.125 bytes)

#### Message 0x86 
- **Byte 7**: `0x02` - Reliable message number
- **Bytes 8-10**: `0x00 0x00 0x02` - Ordering index
- **Bytes 11-13**: `0x00 0x00 0x00` - Ordering channel + padding
- **Byte 14**: `0x86` - Message ID 
- **Bytes 15-16**: `0x0a 0x00` - Length field (10 bytes)
- **Bytes 17-26**: `a0000_char` - Map name (ASCII)
- **Bytes 27-28**: `0x0a 0x00` - Length field (10 bytes)
- **Bytes 29-38**: `a0000_char` - Map name (ASCII, duplicate)
- **Bytes 39-42**: `0xff 0xff 0xff 0xff` - Terminator flags
- **Bytes 43-44**: `0x00 0x00` - Padding

#### Message 0x88 (Map Load Trigger)
- **Byte 45**: `0x60` - Message flags (reliable ordered)
- **Bytes 46-47**: `0x00 0x08` - Payload length (8 bytes)
- **Byte 48**: `0x03` - Reliable message number
- **Bytes 49-51**: `0x00 0x00 0x03` - Ordering index
- **Bytes 52-54**: `0x00 0x00 0x00` - Ordering channel + padding
- **Byte 55**: `0x88` - Message ID (Map Load)

## Character Creation Sequence

### Message 0x8b (Character Creation Request)

**Wireshark Capture:**
```
3229	89.103670811	192.168.1.47	47.245.159.153	RakNet	174	#85: Unknown message ID: 0x8b
```

**Raw UDP Payload:**
```
0000   b4 9d fd 5e 08 1f 18 c0 4d 9c 45 f6 08 00 45 00   ...^....M.E...E.
0010   00 a0 5b 9e 00 00 40 11 8d 49 c0 a8 01 2f 2f f5   ..[...@..I...//.
0020   9f 99 b5 4a 08 90 00 8c 92 03 84 55 00 00 60 03   ...J.......U..`.
0030   aa 21 00 00 18 00 00 00 8b 86 00 03 00 03 00 00   .!..............
0040   00 00 00 00 00 00 00 23 23 01 00 0c 00 44 61 6e   .......##....Dan
0050   44 61 6e 32 31 30 39 31 32 00 00 00 00 00 00 00   Dan210912.......
0060   00 00 00 00 00 00 00 00 00 00 00 80 3f 00 00 00   ............?...
0070   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80   ................
0080   3f 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ?...............
0090   00 00 00 80 3f 00 00 00 00 00 00 00 00 00 00 00   ....?...........
00a0   00 00 00 00 00 00 00 80 3f 00 00 00 00 00         ........?.....
```

#### Character Creation Structure
- **Bytes 0-3**: `0x84 0x55 0x00 0x00` - Frame Set header
- **Byte 4**: `0x60` - Message flags (reliable ordered)
- **Bytes 5-6**: `0x03 0xaa` - Payload length (938 bits = 117.25 bytes)
- **Byte 7**: `0x21` - Reliable message number
- **Bytes 8-10**: `0x00 0x00 0x18` - Ordering index
- **Bytes 11-13**: `0x00 0x00 0x00` - Ordering channel + padding
- **Byte 14**: `0x8b` - Message ID (Character Creation)
- **Bytes 15-16**: `0x86 0x00` - Length field (134 bytes)
- **Bytes 17-18**: `0x03 0x00` - Character data flags
- **Bytes 19-20**: `0x03 0x00` - Additional flags
- **Bytes 21-24**: `0x00 0x00 0x00 0x00` - Padding
- **Bytes 25-28**: `0x00 0x00 0x00 0x00` - Padding
- **Bytes 29-30**: `0x23 0x23` - Character type/class identifier
- **Bytes 31-32**: `0x01 0x00` - Character level (1)
- **Bytes 33-34**: `0x0c 0x00` - Name length (12 bytes)
- **Bytes 35-46**: `DanDan210912` - Character name (ASCII)
- **Bytes 47-160**: Character stats and position data (floating point values)

### Message 0x84 (Character Creation Response)

**Wireshark Capture:**
```
3231	89.185670989	47.245.159.153	192.168.1.47	RakNet	178	#570: Unknown message ID: 0x84
```

**Raw UDP Payload:**
```
0000   18 c0 4d 9c 45 f6 b4 9d fd 5e 08 1f 08 00 45 00   ..M.E....^....E.
0010   00 a4 8f e0 40 00 35 11 24 03 2f f5 9f 99 c0 a8   ....@.5.$./.....
0020   01 2f 08 90 b5 4a 00 90 cc 85 84 3a 02 00 60 03   ./...J.....:..`.
0030   ca e0 01 00 0d 00 00 00 84 86 00 05 00 03 00 cd   ................
0040   c6 b4 06 00 00 00 00 23 23 01 00 0c 00 44 61 6e   .......##....Dan
0050   44 61 6e 32 31 30 39 31 32 00 00 00 00 00 00 00   Dan210912.......
0060   00 00 00 00 00 00 00 00 00 00 00 80 3f 00 00 00   ............?...
0070   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80   ................
0080   3f 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ?...............
0090   00 00 00 80 3f 00 00 00 00 00 00 00 00 00 00 00   ....?...........
00a0   00 00 00 00 00 00 00 80 3f 00 00 00 00 00 00 00   ........?.......
00b0   00 00                                             ..
```

#### Character Creation Response Structure
- **Bytes 0-3**: `0x84 0x3a 0x02 0x00` - Frame Set header
- **Byte 4**: `0x60` - Message flags (reliable ordered)
- **Bytes 5-6**: `0x03 0xca` - Payload length (970 bits = 121.25 bytes)
- **Byte 7**: `0xe0` - Reliable message number
- **Bytes 8-10**: `0x01 0x00 0x0d` - Ordering index
- **Bytes 11-13**: `0x00 0x00 0x00` - Ordering channel + padding
- **Byte 14**: `0x84` - Message ID (Character Creation Response)
- **Bytes 15-16**: `0x86 0x00` - Length field (134 bytes)
- **Bytes 17-18**: `0x05 0x00` - Response flags
- **Bytes 19-20**: `0x03 0x00` - Character ID or status
- **Bytes 21-24**: `0xcd 0xc6 0xb4 0x06` - Character ID (big endian)
- **Bytes 25-28**: `0x00 0x00 0x00 0x00` - Padding
- **Bytes 29-30**: `0x23 0x23` - Character type/class identifier
- **Bytes 31-32**: `0x01 0x00` - Character level (1)
- **Bytes 33-34**: `0x0c 0x00` - Name length (12 bytes)
- **Bytes 35-46**: `DanDan210912` - Character name (ASCII)
- **Bytes 47-160**: Character stats and position data (floating point values)



| Message ID (Hex)  | Name                     | Description                                      | Packet Size       | Flags               |
|-------------------|--------------------------|--------------------------------------------------|-------------------|---------------------|
| 0x82              | DrasaCharacterService    | Contains character service data                  | Large (1258+ bytes)| `60` (reliable ordered) |
| 0x8A              | Character Request        | Requests character data                          | Medium (836–1208 bytes) | `60` (reliable ordered) |
| 0x86              | Character Data           | Character data                                   | Variable          | `60` (reliable ordered) |
| 0x88              | Map Load/Change          | Map load/change notifications                    | Variable          | `60` (reliable ordered) |
| 0x8B              | Character Creation       | Character creation data                          | Large             | `60` (reliable ordered) |
| 0x84              | Server Response          | Server responses                                 | Variable          | `60` (reliable ordered) |


