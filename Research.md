# 📦 RakNet Packet ID Handling 

| Packet ID (Hex) | Constant Name *(guess)*                  | Description / Notes                                    | Handling Function                                |
|------------------|-------------------------------------------|----------------------------------------------------------|--------------------------------------------------|
| `0x10`           | ID_REMOTE_DISCONNECTION_NOTIFICATION      | Remote peer disconnected cleanly                         | `FUN_140c66458(this, packet)`                    |
| `0x11`           | ID_CONNECTION_ATTEMPT_FAILED              | Connection attempt failed                                | Logs warning + `RakNetClient::HandleConnectionFailed` |
| `0x12`           | ID_NO_FREE_INCOMING_CONNECTIONS           | Server full                                              | Ignored                                          |
| `0x14`           | ID_NO_FREE_INCOMING_CONNECTIONS (alt)     | Server full again?                                       | Logs warning                                     |
| `0x15`           | ID_DISCONNECTION_NOTIFICATION *(likely)* | Server actively disconnected this client                 | `RakNetClient::HandleDisconnectionByServer`     |
| `0x16`           | ID_CONNECTION_LOST                        | Lost connection to server                                | Logs warning + `RakNetClient::HandleConnectionLost` |
| `0x19`           | ID_INCOMPATIBLE_PROTOCOL_VERSION                                   | Custom packet?                                           | `FUN_140c670c0(this, packet)`                    |
| `0x1B`           | ID_TIMESTAMP              | Reads byte 10 to determine actual type                   | Special-case handling                            |
| `0x82`           | PACKET_SERVER_IDENTIFY                                   | May be DrasaOnlineLoginServer or DrasaCharacterService           | `RakNetClient::HandleError`                      |
| `0x83`           | HandleTimeSync                                   | Possibly time sync                                       | `RakNetClient::HandleTimeSync`                   |
| `0x84`           | Frame Set Packet                | Custom payload                                           | `FUN_140c66dd0(this, packet, 0)`                 |
| `0x85`           | User-defined packet type 1                | Custom payload                                           | `FUN_140c66dd0(this, packet, 1)`                 |
| `0x86`           | Changing Map                                  | Changing map                    |                     |
| `0x87`           | Unknown                                   | Resets internal variable (`this->field232_0x120 = 0`)    | State reset                                      |
| `0x88`           | Unknown                                   | Sets state + optionally triggers callback                | Sets field + optional call                       |
| `0x89`           | Unknown                                   | Custom logic                                             | `FUN_140c66938(this, packet)`                    |
| `0x8a`           | PACKET_CLIENT_IDENTIFY                  | Identify the client with a parameter DrasaOnlineClient     | `FUN_140c66b18(this, packet)`                    |
| `0x8b`           | Send local config ?                     | Send the computer specs and also the spellbar              | `FUN_140c66b18(this, packet)`                    |
| `0x8E`           | Unknown                                   | Custom logic                                             | `FUN_140c66b18(this, packet)`                    |
| Other            | Unknown or unrecognized packet            | Treated as error or fallback                             | `RakNetClient::HandleError(this, packet)`        |
