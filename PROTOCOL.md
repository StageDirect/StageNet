# StageNet Protocol Specification

## Overview
This document serves as a technical overview of the StageNet protocol. It includes the different data exchanges that are used to transfer data.

---

## Protocol Phases

### 1. Discovery Phase
The console broadcasts a discovery packet over UDP on a unique port to locate available gateways on the network. Gateways respond with their connection details.

#### **Discovery Request (CONSOLE → GATEWAY)**
- **Method**: UDP Broadcast
- **Destination Port**: 44325
- **Payload**:
  - **Type**: `DISCOVERY_REQ`
  - **ConsoleIP**: String (The IP address of the console)

#### **Discovery Response (GATEWAY → CONSOLE)**
- **Method**: UDP Response to `ConsoleIP`
- **Destination Port**: 5001
- **Payload**:
  - **Type**: `DISCOVERY_RESP`
  - **Fields**:
    - `GatewayIP`: String (The IP address of the gateway)
    - `GatewayID`: String (A unique identifier for the gateway)
    - `WSPort`: Integer (The port of the gateway's WebSocket server)

#### Example:
`DISCOVERY_REQ` (CONSOLE → BROADCAST ADDRESS):
```json
{
  "Type": "DISCOVERY_REQ",
  "ConsoleIP": "192.168.1.100"
}
```

`DISCOVERY_RESP` (GATEWAY → CONSOLE):
```json
{
  "Type": "DISCOVERY_RESP",
  "GatewayIP": "192.168.1.2000",
  "GatewayID": "GATEWAY_01",
  "WSPort": 4000
}
```

---

### 2. Connection Phase
After discovery, the console establishes WebSocket connections directly with each gateway.

#### **WebSocket Connection (CONSOLE → GATEWAY)**
- **Method**: WebSocket Connection
- **URL**: `ws://<gateway-ip>:<ws-port>/stagenet`

---

### 3. Lease Request Phase
After connection is established, gateways inform the console of their data channels (such as DMX), and the console assigns StageNet addresses.

#### **Lease Request (GATEWAY → CONSOLE)**
- **Method**: WebSocket Message
- **Payload**:
  - **Type**: `LEASE_REQ`
  - **Fields**:
    - `GatewayID`: String (Unique identifier for the gateway)
    - `DMXAddressRange`: Array of integers (Range of DMX addresses requesting StageNet addresses)

#### **Lease Response (CONSOLE → GATEWAY)**
- **Method**: WebSocket Message
- **Payload**:
  - **Type**: `LEASE_RESP`
  - **Fields**:
    - `LeaseID`: String (Unique identifier for this lease)
    - `LeasedStageNetAddresses`: Array of strings (Assigned StageNet addresses)

#### Example:
`LEASE_REQ` (GATEWAY → CONSOLE):
```json
{
  "Type": "LEASE_REQ",
  "GatewayID": "GATEWAY_01",
  "DMXAddressRange": [1, 2, 3, 4, 5]
}
```

`LEASE_RESP` (CONSOLE → GATEWAY):
```json
{
  "Type": "LEASE_RESP",
  "LeaseID": "lease_001",
  "LeasedStageNetAddresses": ["SN1", "SN2", "SN3", "SN4", "SN5"]
}
```

---

### 4. Data Transmission Phase
Once StageNet addresses are leased, the console sends control data using the assigned addresses.

#### **Data Packet (CONSOLE → GATEWAY)**
- **Method**: WebSocket Message
- **Payload**:
  - **Type**: `DATA_PKT`
  - **Fields**:
    - `StageNetAddress`: String (Assigned StageNet address)
    - `Value`: Integer (The value to set)

#### Example:
`DATA_PKT` (CONSOLE → GATEWAY):
```json
{
  "Type": "DATA_PKT",
  "StageNetAddress": "SN1",
  "Value": 255
}
```

---

### 5. Disconnection Phase
Either party can initiate a clean disconnection by closing the WebSocket connection.

#### **Disconnection**
- **Method**: WebSocket Disconnection
- Standard WebSocket closure with code 1000 (Normal Closure)

---

## Error Codes
- `ERROR_INVALID_PACKET`: When a packet is not in the expected format or fields are missing
- `ERROR_LEASE_FAILED`: When the lease request cannot be processed
- Standard WebSocket close codes for connection issues