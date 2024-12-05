# StageNet Protocol Specification

## Overview
This document serves as a technical overview of the StageNet protocol. It includes the different data exchanges that are used to transfer data. The protocol is designed to abstract various hardware and data interfaces into a unified addressing and communication system.

---

## Protocol Phases

### 1. Discovery Phase
Gateways advertise themselves on the network using mDNS (Multicast DNS). The console discovers available gateways by querying for devices advertising the StageNet service.

#### **Service Advertisement (GATEWAY)**
- **Method**: mDNS Service Advertisement
- **Service Type**: `_stagenet._tcp.local`
- **Port**: 4000 (Default WebSocket port)
- **TXT Records**:
  - `id`: String (Unique identifier for the gateway)
  - `type`: String (Type of gateway, e.g., "DMX", "MIDI")
  - `version`: String (Protocol version)

#### **Service Discovery (CONSOLE)**
- **Method**: mDNS Query
- **Service Type**: `_stagenet._tcp.local`
- **Response**: List of available StageNet gateways with their connection details

#### Example Service:
```
Gateway Service:
Name: gateway-01._stagenet._tcp.local
IP: 192.168.1.200
Port: 4000
TXT Records:
  id=GATEWAY_01
  type=DMX
  version=1.0
```

---

### 2. Connection Phase
After discovery, the console establishes WebSocket connections directly with each gateway.

#### **WebSocket Connection (CONSOLE → GATEWAY)**
- **Method**: WebSocket Connection
- **URL**: `ws://<gateway-ip>:<ws-port>/stagenet`

---

### 3. Lease Request Phase
After connection is established, gateways request a number of StageNet addresses. Once received, the gateway maps these to its hardware channels and informs the console of the mapping.

#### **Lease Request (GATEWAY → CONSOLE)**
- **Method**: WebSocket Message
- **Payload**:
  - **Type**: `LEASE_REQ`
  - **Fields**:
    - `GatewayID`: String (Unique identifier for the gateway)
    - `AddressCount`: Integer (Number of StageNet addresses needed)

#### **Lease Response (CONSOLE → GATEWAY)**
- **Method**: WebSocket Message
- **Payload**:
  - **Type**: `LEASE_RESP`
  - **Fields**:
    - `LeaseID`: String (Unique identifier for this lease)
    - `Addresses`: Array of strings (Available StageNet addresses)

#### **Mapping Update (GATEWAY → CONSOLE)**
- **Method**: WebSocket Message
- **Payload**:
  - **Type**: `MAPPING_UPDATE`
  - **Fields**:
    - `LeaseID`: String (The lease these mappings belong to)
    - `Mappings`: Array of objects
      - `StageNetAddress`: String (The assigned StageNet address)
      - `HardwareType`: String (e.g., "DMX", "MIDI", "ANALOG")
      - `Channel`: Integer (The hardware channel number)

#### Example:
`LEASE_REQ` (GATEWAY → CONSOLE):
```json
{
  "Type": "LEASE_REQ",
  "GatewayID": "GATEWAY_01",
  "AddressCount": 4
}
```

`LEASE_RESP` (CONSOLE → GATEWAY):
```json
{
  "Type": "LEASE_RESP",
  "LeaseID": "lease_001",
  "Addresses": ["SN1", "SN2", "SN3", "SN4"]
}
```

`MAPPING_UPDATE` (GATEWAY → CONSOLE):
```json
{
  "Type": "MAPPING_UPDATE",
  "LeaseID": "lease_001",
  "Mappings": [
    {"StageNetAddress": "SN1", "HardwareType": "DMX", "Channel": 200},
    {"StageNetAddress": "SN2", "HardwareType": "DMX", "Channel": 201},
    {"StageNetAddress": "SN3", "HardwareType": "DMX", "Channel": 220},
    {"StageNetAddress": "SN4", "HardwareType": "DMX", "Channel": 430}
  ]
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
    - `Value`: Integer (The value to set, currently 0-255)

#### Example:
`DATA_PKT` (CONSOLE → GATEWAY):
```json
{
  "Type": "DATA_PKT",
  "StageNetAddress": "SN1",
  "Value": 255
}