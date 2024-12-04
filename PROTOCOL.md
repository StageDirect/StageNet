# StageNet Protocol Specification

## Overview
This document serves as a technical overview of the StageNet protocol. It includes the different data exchanges that take place to power the protocol.

---

## Protocol Phases

### 1. Discovery Phase
The console broadcasts a discovery packet over UDP to locate available gateways on the network. Gateways respond with a request to establish a link.

#### **Discovery Request (CONSOLE → GATEWAY)**
- **Method**: UDP Broadcast
- **Destination Port**: 44325
- **Payload**:
  - **Type**: `DISCOVERY_REQ`
  - **Payload**: None

#### **Discovery Response (GATEWAY → CONSOLE)**
- **Method**: UDP Response to Sender IP
- **Destination Port**: 5001
- **Payload**:
  - **Type**: `DISCOVERY_RESP`
  - **Fields**:
    - `GatewayIP`: String (The IP address of the gateway)
    - `GatewayID`: String (A unique identifier for the gateway)

#### Example:
`DISCOVERY_REQ` (CONSOLE → BROADCAST ADDRESS):

```json
{
  "Type": "DISCOVERY_REQ"
}
```

`DISCOVERY_RESP` (GATEWAY → CONSOLE):
```json
{
  "Type": "DISCOVERY_RESP",
  "GatewayIP": "192.168.1.100",
  "GatewayID": "GATEWAY_01"
}
```

---

### 2. Connection Phase

Once the console has recognized gateway(s), on start-up it sends a request to the gateway prompting it to initiate a WebSocket connection with the console to begin the data communication.

#### **Connect Request (CONSOLE → GATEWAY)**
- **Method**: UDP Request
- **Payload**:
  - **Type**: `CONNECT_REQ`
  - **Fields**:
    - `GatewayIP`: String (The IP address of the console)
    - `ConnectionID`: String (A unique connection identifier)

#### **Connect Response (GATEWAY → CONSOLE)**
- **Method**: WebSocket Connection
- **Payload**:
  - **Type**: `CONNECT_RESP`
  - **Fields**:
    - `ConnectionID`: String (Unique connection identifier, matches the one from `CONNECT_REQ`)

#### Example:
`CONNECT_REQ` (CONSOLE → GATEWAY):

```json
{
  "Type": "CONNECT_REQ",
  "ConnectionID": "some-connection-identifier"
}
```

`CONNECT_RESP` (GATEWAY → CONSOLE):

```json
{
  "Type": "CONNECT_RESP",
  "ConnectionID": "some-connection-identifier"
}
```

---

### 3. Lease Request Phase
Gateways request StageNet addresses from the console, to assosciate with DMX channels. The console reserves these addresses and sends a response back to the gateway. The gateway matches the received addresses with the DMX addresses, and sends them back to the console where it stores them.

#### **Lease Request (GATEWAY → CONSOLE)**
- **Method**: WebSocket
- **Payload**:
  - **Type**: `LEASE_REQ`
  - **Fields**:
    - `GatewayID`: String (Unique identifier for the gateway)
    - `DMXAddressRange`: Array of integers (Range of DMX addresses requesting StageNet addresses)

#### **Lease Response (CONSOLE → GATEWAY)**
- **Method**: WebSocket
- **Payload**:
  - **Type**: `LEASE_ACK`
  - **Fields**:
    - `ConnectionID`: String (Connection identifier)
    - `LeasedStageNetAddresses`: Array of strings (Assigned StageNet addresses)
    - `LeaseID`: String (Unique identifier for this lease)

#### Example:
`LEASE_REQ` (GATEWAY → CONSOLE):

```json
{
  "Type": "LEASE_ACK",
  "ConnectionID": "12345",
  "LeasedStageNetAddresses": ["SN1", "SN2", "SN3", "SN4", "SN5"],
  "LeaseID": "lease_001"
}
```

---

### 4. Data Transmission Phase
Once StageNet addresses are leased, the gateway can send control data (like DMX values) to the console, which will map these to their respective StageNet addresses.

#### **Data Packet (GATEWAY → CONSOLE)**
- **Method**: WebSocket
- **Payload**:
  - **Type**: `DATA_PKT`
  - **Fields**:
    - `GatewayID`: String (Unique identifier for the gateway)
    - `StageNetAddress`: String (Assigned StageNet address)
    - `Value`: Integer (The current value of the hardware)

#### Example:
`DATA_PKT` (GATEWAY → CONSOLE):

```json
{
  "Type": "DATA_PKT",
  "GatewayID": "GATEWAY_01",
  "StageNetAddress": "SN1",
  "Value": 255
}
```

---

### 5. Disconnection Phase
A gateway can disconnect from the console by sending a disconnect request over the WebSocket connection.

#### **Disconnect Request (GATEWAY → CONSOLE)**
- **Method**: WebSocket
- **Payload**:
  - **Type**: `DISCONNECT_REQ`
  - **Fields**:
    - `ConnectionID`: String (Connection identifier)

#### **Disconnect Response (CONSOLE → GATEWAY)**
- **Method**: WebSocket
- **Payload**:
  - **Type**: `DISCONNECT_ACK`
  - **Fields**:"
    - `ConnectionID`: String (Connection identifier)

#### Example:
`DISCONNECT_REQ` (GATEWAY → CONSOLE):

```json
{
  "Type": "DISCONNECT_REQ",
  "ConnectionID": "12345"
}
```

**DISCONNECT_ACK** (CONOSLE → GATEWAY):
```json
{
  "Type": "DISCONNECT_ACK",
  "ConnectionID": "12345"
}
```

---

## Error Codes
- `ERROR_INVALID_PACKET`: When a packet is not in the expected format or the fields are missing.
- `ERROR_CONNECTION_FAILED`: When a WebSocket connection attempt fails.
- `ERROR_LEASE_FAILED`: When the lease request cannot be processed.