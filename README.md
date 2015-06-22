# Streaming Message Protocol, Specification

## Introduction

The Streaming Message Protocol (SMP) is a lightweight message framing protocol for exchanging 
messages between peers across transports such as: TCP, TLS, WebSockets. SMP allows whole and 
multipart messages in binary codec.

__DRAFT VERSION__

## Comments and Suggestions

Please submit a [Issue](https://github.com/smprotocol/smp/issues).

## Implementations

- [Node.JS](https://github.com/smprotocol/smp-node) (official).

## Specification 1.0-DRAFT

```
Streaming Message Protocol

VERSION: 1.0-DRAFT
DATE: 22 JUNE 2015
EDITOR: MARK W. B. ASHCROFT <mark@fluidecho.com>


Abstract

The Streaming Message Protocol (SMP) is a message framing protocol layer for transports such as: 
TCP, TLS, WebSockets. It sends and receives binary messages from 0 to 281.471 TB in size. The 
protocol is designed as a streaming data systems, provides mechanisms for efficiently sized 
sequential and identified frames or whole messages. The receiver or receivers collect whole 
messages or frames (multipart messages). The peers can send and receive continuous "streaming" 
frames.


Copyright Notice

Copyright (c) 2015 Mark W. B. Ashcroft.

This Specification is free software; you can redistribute it and/or modify it under the terms of 
the GNU General Public License as published by the Free Software Foundation; either version 3 of 
the License, or (at your option) any later version.

This Specification is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; 
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the 
GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program; if not, 
see <http://www.gnu.org/licenses>.


Definitions

- MESSAGE
    A whole MESSAGE when ARGUMENTS byte size is less than the MAX-MESSAGE-SIZE.
    
- FRAME
    Part of a "message" which is identified and sequential.
    
- FRAMES
    Collective of FRAME, up-to 281.471 TB in size.

- FRAME#
    The sequential number of the FRAME, to a maximum of 4,294,967,295.
    
- MAX-MESSAGE-SIZE
    The maximum size of bytes which a whole MESSAGE will be formed, if greater will become 
    sequential FRAMES of this size or less.
    
- ARGUMENT
    Combined LENGTH in bytes of the data PAYLOAD plus the data PAYLOAD, maximum of 65,535.
    
- ARGC
    ARGUMENT count, the number of ARGUMENTS within the MESSAGE or FRAME, second half of the first 
    OCTET in the MESSAGE or FRAME, from 0 up-to 16 ARGUMENTS.
    
- ID
    "Message" identifier sequential number, from 0 to 65,535.
    
- LENGTH
    Size in bytes of a ARGUMENT.
    
- PAYLOAD
    The data "message" in binary to send.
    
- CODE
    Meta CODE sent as the first half OCTET in the MESSAGE or FRAME.
     
- OCTET
    Eight bits.
    
- BYTE
    An OCTET of bits.
    
- MTU
    Maximum Transmission Unit is the optimal number of bytes to send over the network wire.


Framing

All integer bytes are big endian.

Octets for a MESSAGE:
+-----------------------------------------------------------------------------------------+
| MESSAGE                                                                                 |
+--------------------------+-------------------+------------------------------------------+
|                          | ARGUMENT          | ADDITIONAL ARGUMENTS                     |
+--------+-----------------+--------+----------+------------------------------------------+
|        | META: CODE/ARGC | LENGTH | PAYLOAD  | ...                                      |
+--------+-----------------+--------+----------+------------------------------------------+
| OCTETS | 0               | 1, 2   | <LENGTH> | ...                                      |
+--------+-----------------+--------+----------+------------------------------------------+

Octets for a FRAME:
+-----------------------------------------------------------------------------------------+
| FRAME                                                                                   |
+----------------------------------------------+-------------------+----------------------+
|                                              | ARGUMENT          | ADDITIONAL ARGUMENTS |
+--------+-----------------+------+------------+--------+----------+----------------------+
|        | META: CODE/ARGC | ID   | FRAME#     | LENGTH | PAYLOAD  | ...                  |
+--------+-----------------+------+------------+--------+----------+----------------------+
| OCTETS | 0               | 1, 2 | 3, 4, 5, 6 | 7, 8   | <LENGTH> | ...                  |
+--------+-----------------+------+------------+--------+----------+----------------------+


Meta Codes

+------+-------------------+--------------------------------------------------------------+
| CODE | DESCRIPTION       | NOTES                                                        |
+------+-------------------+--------------------------------------------------------------+
|    0 | WHOLE MESSAGE     | Payload data, a whole complete MESSAGE, when ARGUMENTS       | 
|      |                   | PAYLOAD size < MAX-MESSAGE-SIZE.                             |
+------+-------------------+--------------------------------------------------------------+
|    1 | NEW FRAME         | Payload data, first FRAME sent once to every receiver.       |
+------+-------------------+--------------------------------------------------------------+
|    2 | CONTINUING FRAME  | Payload data, a middle FRAME.                                |
+------+-------------------+--------------------------------------------------------------+
|    3 | LAST FRAME        | Payload data, the last FRAME sent.                           |
+------+-------------------+--------------------------------------------------------------+
|    4 | INFOMATION        | Information, contained within the PAYLOAD.                   |
+------+-------------------+--------------------------------------------------------------+
|    5 | ERROR             | Error, first ARGUMENT is a error code, second ARGUMENT is    |
|      |                   | the error message.                                           |
+------+-------------------+--------------------------------------------------------------+
|    6 | HEARTBEAT         | Heartbeat, with optional PAYLOAD data.                       |
+------+-------------------+--------------------------------------------------------------+
|    7 | STOP              | Action, stop sending or receiving MESSAGES or FRAMES, will   |
|      |                   | immediately close socket connection.                         |
+------+-------------------+--------------------------------------------------------------+
|    8 | PAUSE             | Action, pause sending or receiving MESSAGES or FRAMES.       |
+------+-------------------+--------------------------------------------------------------+
|    9 | RESUME            | Action, resume sending or receiving MESSAGES or FRAMES, with |
|      |                   | optional PAYLOAD data.                                       |
+------+-------------------+--------------------------------------------------------------+
|   10 | RESERVED          | Reserved for future versions.                                |
+------+-------------------+--------------------------------------------------------------+
|   11 | RESERVED          | Reserved for future versions.                                |
+------+-------------------+--------------------------------------------------------------+
|   12 | RESERVED          | Reserved for future versions.                                |
+------+-------------------+--------------------------------------------------------------+
|   13 | USER DEFINED      | User can set their own custom code.                          |
+------+-------------------+--------------------------------------------------------------+
|   14 | USER DEFINED      | User can set their own custom code.                          |
+------+-------------------+--------------------------------------------------------------+
|   15 | USER DEFINED      | User can set their own custom code.                          |
+------+-------------------+--------------------------------------------------------------+


Examples

Send PAYLOAD data "hello world" as a MESSAGE (CODE 0) shown in spaced hexadecimal:

01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64


Send PAYLOAD data "hello world" as a FRAME (CODE 1) with ID 1 and FRAME# 1, shown in spaced 
hexadecimal:

11 00 00 01 00 00 00 01 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64


Conventions

- If a MESSAGE ARGUMENTS size is greater than the MAX-MESSAGE-SIZE, the MESSAGE must be FRAMES of 
  the MAX-MESSAGE-SIZE or less.

- ID and FRAME# must be a sequential number.

- A new FRAME must send CODE 1 to every receiver once or CODE 3 if the last FRAME.

- The receiver will treat as a new "message" if receives a FRAME with CODE 1.

- The receiver will treat as a ended "message" if receives a FRAME with CODE 3.

- If receiving CODE 7 will not send any more MESSAGES or FRAMES to the 
  sender and the sender will immediately close the socket connection.


Message and Frame Payload Data

The PAYLOAD data can be any binary codec. PAYLOAD data is made up of 0 or more ARGUMENTS. The 
reason for ARGUMENTS is to be able to mix text based and binary data codecs within the one 
MESSAGE or FRAME. For example you may have a MESSAGE/FRAME which has binary PAYLOAD data from a 
photograph and JSON "text based" data about that photograph; you could have ARGUMENT[0] as the JSON 
(serialized to binary) and ARGUMENT[1] as the photograph binary data. Having ARGUMENTS means you do 
not have to serialize both text and binary data together using something like MsgPack, however you 
can if you want.

```


