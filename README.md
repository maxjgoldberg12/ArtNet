# ArtNet

A pure Swift implementation of the Art-Net protocol for DMX512 lighting control over Ethernet networks.

## Overview

Art-Net is an industry-standard Ethernet protocol for transferring DMX512 lighting data over TCP/IP networks. This library provides a complete Swift implementation of the Art-Net protocol, allowing you to:

- Send and receive Art-Net packets
- Control DMX512 lighting fixtures over networks
- Implement lighting controllers and nodes
- Handle RDM (Remote Device Management) communications
- Manage device discovery and configuration

## Features

- ✅ **Complete Art-Net Protocol Support** - All major packet types including ArtPoll, ArtDmx, ArtSync, ArtAddress, and more
- ✅ **Type-Safe Addressing** - Hierarchical addressing with Net, Subnet, and Universe components
- ✅ **RDM Support** - Remote Device Management for fixture configuration and monitoring
- ✅ **Cross-Platform** - Works on iOS, macOS, Linux, and other Swift-supported platforms
- ✅ **Zero Dependencies** - Pure Swift implementation with no external dependencies
- ✅ **Codable Support** - Built-in encoding/decoding with proper endianness handling
- ✅ **Comprehensive Testing** - Extensive test suite with real packet examples

## Installation

### Swift Package Manager

Add this package to your `Package.swift` file:

```swift
dependencies: [
    .package(url: "https://github.com/maxjgoldberg12/ArtNet.git", from: "1.0.0")
]
```

Or add it through Xcode:
1. File → Add Package Dependencies
2. Enter the repository URL
3. Select version requirements

## Quick Start

### Creating and Encoding Packets

```swift
import ArtNet

// Create a DMX data packet
let dmxPacket = ArtDmx(
    sequence: 1,
    physical: 0,
    portAddress: PortAddress(universe: 1, subnet: 0, net: 0),
    lightingData: Data([255, 128, 0, 255]) // RGBW values
)

// Encode the packet to binary data
let encoder = ArtNetEncoder()
let packetData = try encoder.encode(dmxPacket)

// Send over UDP socket...
```

### Device Discovery

```swift
// Create a poll packet to discover devices
let pollPacket = ArtPoll(
    behavior: [.diagnostics],
    priority: .low
)

let pollData = try encoder.encode(pollPacket)
// Broadcast this data to discover Art-Net devices on the network
```

### Decoding Received Packets

```swift
// Decode received Art-Net data
let decoder = ArtNetDecoder()

// Determine packet type from opcode
let opCode = OpCode(rawValue: /* extract from packet header */)

switch opCode {
case .dmx:
    let dmxPacket = try decoder.decode(ArtDmx.self, from: receivedData)
    print("Received DMX data for universe \(dmxPacket.portAddress.universe)")
    
case .pollReply:
    let replyPacket = try decoder.decode(ArtPollReply.self, from: receivedData)
    print("Device: \(replyPacket.shortName) at \(replyPacket.address)")
    
default:
    print("Unsupported packet type: \(opCode)")
}
```

## Core Concepts

### Addressing

Art-Net uses a hierarchical addressing system:

```swift
// Create a port address
let portAddress = PortAddress(
    universe: 1,    // 4-bit (0-15) - Individual universe within subnet
    subnet: 2,      // 4-bit (0-15) - Subnet within net
    net: 3          // 7-bit (0-127) - Network identifier
)

print(portAddress) // PortAddress(universe: 1, subnet: 2, net: 3)
```

### Key Packet Types

#### ArtDmx - DMX Data Transfer
```swift
let dmxPacket = ArtDmx(
    sequence: 1,                    // Packet sequence number
    physical: 0,                    // Physical input port
    portAddress: portAddress,       // Destination address
    lightingData: dmxData          // Up to 512 bytes of DMX data
)
```

#### ArtPoll - Device Discovery
```swift
let pollPacket = ArtPoll(
    behavior: [.replyWithoutPolling, .diagnostics],
    priority: .low
)
```

#### ArtSync - Synchronization
```swift
let syncPacket = ArtSync()
// Send after multiple ArtDmx packets to synchronize output
```

#### ArtAddress - Remote Configuration
```swift
let addressPacket = ArtAddress(
    netSwitch: 1,
    bindingIndex: 0,
    shortName: "My Controller",
    longName: "Professional Lighting Controller",
    inputAddresses: [portAddress],
    outputAddresses: [portAddress],
    subSwitch: 0,
    command: .ledNormal
)
```

### RDM Support

```swift
// RDM unique identifier
let rdmUID = RdmUID(rawValue: "001A7DDA7113")!

// RDM packet
let rdmPacket = ArtRdm(
    rdmVersion: .standard,
    net: 0,
    command: .process,
    address: 1,
    rdmPacket: rdmData
)
```

## Network Integration

### UDP Socket Example

```swift
import Network
import ArtNet

class ArtNetController {
    private let connection: NWConnection
    private let encoder = ArtNetEncoder()
    private let decoder = ArtNetDecoder()
    
    init() {
        // Art-Net uses UDP port 6454
        connection = NWConnection(
            to: .hostPort(host: "2.255.255.255", port: 6454),
            using: .udp
        )
    }
    
    func sendDMX(universe: PortAddress.Universe, data: Data) {
        let packet = ArtDmx(
            portAddress: PortAddress(universe: universe, subnet: 0, net: 0),
            lightingData: data
        )
        
        do {
            let encodedData = try encoder.encode(packet)
            connection.send(content: encodedData, completion: .idempotent)
        } catch {
            print("Failed to encode packet: \(error)")
        }
    }
}
```

## Protocol Constants

```swift
// Standard Art-Net port
let artNetPort: UInt16 = 6454

// Broadcast address for device discovery  
// Art-Net spec suggests 2.255.255.255 for directed broadcast (recommended), or 255.255.255.255 for universal broadcast
let broadcastAddress = "2.255.255.255"

// Maximum DMX channels per universe
let maxDMXChannels = 512

// Protocol version
let protocolVersion = ProtocolVersion.current // Version 14
```

## Advanced Features

### Custom OEM Definitions
```swift
// Define custom OEM codes
let customOEM = OEMCode(rawValue: 0x1234)
```

### Network Address Handling
```swift
// IPv4 network addresses
let nodeIP = NetworkAddress.IPv4(rawValue: "192.168.1.100")!
let subnetMask = SubnetMask(rawValue: "255.255.255.0")!

// MAC addresses
let macAddress = MacAddress(rawValue: "00:1A:7D:DA:71:13")!
```

### Diagnostic Data
```swift
let diagnosticPacket = DiagnosticData(
    priority: .critical,
    data: "System Error: Temperature too high".data(using: .utf8)!
)
```

## Development

### Building
```bash
swift build
```

### Testing
```bash
swift test
```

### Running Specific Tests
```bash
swift test --filter PacketTests.testArtDmx
```

## Protocol Reference

This library implements Art-Net protocol version 4 as specified in the [Art-Net User Guide](https://artisticlicence.com/WebSiteMaster/User%20Guides/art-net.pdf).

### Supported Packet Types

| OpCode | Packet Type | Description |
|--------|-------------|-------------|
| 0x2000 | ArtPoll | Device discovery |
| 0x2100 | ArtPollReply | Device status response |
| 0x2300 | DiagnosticData | Diagnostic information |
| 0x2400 | ArtCommand | Text-based commands |
| 0x5000 | ArtDmx | DMX512 data transfer |
| 0x5100 | ArtNzs | Non-zero start code data |
| 0x5200 | ArtSync | Synchronization signal |
| 0x6000 | ArtAddress | Remote configuration |
| 0x7000 | ArtInput | Input port configuration |
| 0x8000 | ArtTodRequest | RDM device discovery |
| 0x8100 | ArtTodData | RDM device list |
| 0x8200 | ArtTodControl | RDM discovery control |
| 0x8300 | ArtRdm | RDM messages |
| 0x8400 | ArtRdmSub | Compressed RDM data |
| 0x9700 | ArtTimeCode | SMPTE time code |
| 0x9900 | ArtTrigger | Macro triggers |
| 0xF200 | FirmwareMaster | Firmware upload |
| 0xF300 | FirmwareReply | Firmware acknowledgment |
| 0xF800 | ArtIpProg | IP configuration |
| 0xF900 | ArtIpProgReply | IP configuration response |

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes with tests
4. Ensure all tests pass
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Related Projects

- [Art-Net Protocol Specification](https://artisticlicence.com/WebSiteMaster/User%20Guides/art-net.pdf)
- [ESTA Standards](https://tsp.esta.org/tsp/documents/published_docs.php)
