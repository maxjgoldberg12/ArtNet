# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Swift library implementing the Art-Net protocol for DMX512 lighting control over Ethernet networks. Art-Net is an industry-standard protocol for transferring lighting control data over TCP/IP networks.

## Development Commands

### Building
```bash
swift build
```

### Testing
```bash
swift test
```

### Running specific tests
```bash
swift test --filter ArtNetTests.testPortAddress
```

## Architecture

### Core Protocol Structure
- **ArtNetPacket Protocol**: Base protocol that all Art-Net packet types conform to, defining `opCode` and `formatting` properties
- **OpCode Enum**: Defines all Art-Net operation codes (0x2000-0xf900) for different packet types like poll, dmx, sync, etc.
- **PacketHeader**: Internal header structure containing the "Art-Net" identifier and opcode

### Key Packet Types
- **ArtPoll**: Discovery packets sent by controllers to find nodes and media servers
- **ArtPollReply**: Response packets containing device status information  
- **ArtDmx**: Main data packets for transferring DMX512 lighting data to universes
- **ArtSync**: Synchronization packets for coordinating multiple universes
- **ArtAddress**: Remote programming packets for configuring nodes

### Data Types and Utilities
- **PortAddress**: 15-bit addressing combining net, subnet, and universe (Sources/ArtNet/PortAddress.swift)
- **Address**: Universe and subnet addressing (Sources/ArtNet/ArtAddress.swift)
- **Codable Extensions**: Custom encoding/decoding with endianness handling in Encoder.swift/Decoder.swift
- **ArtNetFormatting**: Configuration for little-endian fields and data formatting
- **BitMaskOption**: Protocol for bitmask-based configuration options

### Networking Components
- **MacAddress**: MAC address handling for network identification
- **NetworkAddress**: IP address and network configuration utilities
- **RdmUID**: RDM (Remote Device Management) unique identifiers

The library follows a modular design where each Art-Net packet type is implemented as a separate struct conforming to ArtNetPacket, with shared utilities for encoding, addressing, and network operations.