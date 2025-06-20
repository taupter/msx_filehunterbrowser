Source: https://github.com/Konamiman/MSX-UNAPI-specification/blob/master/docs/Ethernet%20UNAPI%20specification%201.1.md

# The Ethernet UNAPI specification

This document describes an UNAPI compliant API specification for Ethernet hardware for MSX computers.

## Index

[1. Introduction](#1-introduction)

[2. API identifier and version](#2-api-identifier-and-version)

[3. API routines](#3-api-routines)

[3.1. ETH_GETINFO: Obtain the implementation name and version](#31-eth_getinfo-obtain-the-implementation-name-and-version)

[3.2. ETH_RESET: Reset hardware](#32-eth_reset-reset-hardware)

[3.3. ETH_GET_HWADD: Obtain the Ethernet address](#33-eth_get_hwadd-obtain-the-ethernet-address)

[3.5. ETH_NET_ONOFF: Enable or disable networking](#35-eth_net_onoff-enable-or-disable-networking)

[3.6. ETH_DUPLEX: Configure duplex mode](#36-eth_duplex-configure-duplex-mode)

[3.7. ETH_FILTERS: Configure frame reception filters](#37-eth_filters-configure-frame-reception-filters)

[3.8. ETH_IN_STATUS: Check for received frames availability](#38-eth_in_status-check-for-received-frames-availability)

[3.9. ETH_GET_FRAME: Retrieve the oldest received frame](#39-eth_get_frame-retrieve-the-oldest-received-frame)

[3.10. ETH_SEND_FRAME: Send a frame](#310-eth_send_frame-send-a-frame)

[3.11. ETH_OUT_STATUS: Check frame transmission status](#311-eth_out_status-check-frame-transmission-status)

[3.12. ETH_SET_HWADD: Set the Ethernet address](#312-eth_set_hwadd-set-the-ethernet-address)

[Appendix A. Ethernet frame formats](#appendix-a-ethernet-frame-formats)

[Appendix B. Acknowledgements](#appendix-b-acknowledgements)

[Appendix C. Version history](#appendix-c-version-history)

## 1. Introduction

MSX-UNAPI is a standard procedure for defining, discovering and using new APIs (Application Program Interfaces) for MSX computers. The MSX-UNAPI specification is described [in a separate document](MSX%20UNAPI%20specification%201.1.md).

This document describes an UNAPI compliant API intended for hardware that implements Ethernet networking capabilities. The functionality provided by this API covers the link layer of a network communications stack; as such, it mainly provides routines to send and receiving raw Ethernet frames to and from an Ethernet network. There are also auxiliary routines to check for network availability, to obtain the local Ethernet address, or to configure reception filters.

The intended client software applications for this API are implementations of the higher level layers of the communications stack, typically, TCP/IP stacks. Anyway it can be useful for other types of software as well, for example network traffic monitors.

Note that it is not mandatory to have actual underlying Ethernet hardware in order to implement this API. As long as the routine signatures and behaviors are preserved, the implementation will ve valid, even if it acts on a completely different type of hardware, or on no harwdare at all (for example, an Ethernet emulation API over a serial or JoyNet cable could be developed).

## 2. API identifier and version

The API identifier for the specification described in this document is: "ETHERNET" (without the quotes). Remember that per the UNAPI specification, API identifiers are case-insensitive.

The Ethernet API version described in this document is 1.1. This is the API specification version that the mandatory implementation information routine must return in DE (see [Section 3.1](#31-eth_getinfo-obtain-the-implementation-name-and-version)).

## 3. API routines

This version of the Ethernet API consists of 11 mandatory routines, which are described below. API implementations may define their own additional implementation-specific routines, as described in the MSX- UNAPI specification.

### 3.1. ETH_GETINFO: Obtain the implementation name and version

* Input:
  * A = 0

* Output:
  * HL = Address of the implementation name string 
  * DE = API specification version supported. D=primary, E=secondary.
  * BC = API implementation version. B=primary, C=secondary.

This routine is mandatory for all implementations of all UNAPI compliant APIs. It returns basic information about the implementation itself: the implementation version, the supported API version, and a pointer to the implementation description string.

The implementation name string must be placed in the same slot or segment of the implementation code (or in page 3), must be zero terminated, must consist of printable characters, and must be at most 63 characters long (not including the terminating zero). Refer to the MSX-UNAPI specification for more details.

### 3.2. ETH_RESET: Reset hardware

* Input:
  * A = 1 

This routine resets the underlying hardware and/or the API implementation state variables to its initial state. After executing this routine, the hardware (if present) and the implementation must remain in the same state as after a computer reset or after the implementation is installed (whatever applies). More precisely, the state after the reset must be as follows:

* The Ethernet address is appropriately set to its default value (it is optional to allow changing this address, see [Section 3.3](#33-eth_get_hwadd-obtain-the-ethernet-address)).
* Networking is enabled. (See [Section 3.5](#35-eth_net_onoff-enable-or-disable-networking)).
* Received frames, if any, are discarded. Any frame transmission in process when the routine was called is aborted.
* Duplex mode is set to half-duplex, when it is possible (see [Section 3.6](#36-eth_duplex-configure-duplex-mode)).
* Reception filters are set to accept broadcast and small frames. Promiscuous mode is disabled. (See [Section 3.7](#37-eth_filters-configure-frame-reception-filters)).

### 3.3. ETH_GET_HWADD: Obtain the Ethernet address

* Input:
  * A = 2

* Output:
  * L-H-E-D-C-B = Current Ethernet address

This routine obtains the current Ethernet address of the implementation. The address is mapped to registers HL, DE and BC in a way that makes it easy to store and retrieve it for Z80, which stores 16 bit values int little-endian format. For example, the address returned by this routine can be stored at address X with these instructions: LD (X),HL - LD (X+2),DE - LD (X+4),BC.

Ethernet addresses (also called MAC addresses) are unique for each physical card and are intended to never be changed after the card is manufactured. See [Appendix A](#appendix-a-ethernet-frame-formats).

If the implementation supports it, the hardware address can be changed by using the ETH_SET_HWADD routine (see [Section 3.12](#312-eth_set_hwadd-set-the-ethernet-address)).

### 3.4. ETH_GET_NETSTAT: Obtain network connection status

* Input:
  * A = 3

* Output:
  * A = current network status
    * 0 NOT connected to an active network
    * 1 connected to an active network

This routine checks whether the underlying hardware is connected to an active network or not (in other words, if the Ethernet cable is appropriately plugged and there is carrier).

It is allowed to use loopback methods (that is, to send a frame and check if it is received back) to check the network connection status. Therefore, it may take a while to execute, so it is not advisable to invoke it too often.

### 3.5. ETH_NET_ONOFF: Enable or disable networking

* Input: 
  * A = 4 
  * B = Action to perform:
    * 0: Obtain current state only
    * 1: Enable networking 
    * 2: Disable networking 
    
* Output: 
  * A = State after routine execution:
    * 1: Networking is enabled
    * 2: Networking is disabled

This routine logically enables or disables networking activity. When disabled, no frames will be recevied (or received frames will be automatically discarded); the behavior when trying to send a frame with networking disabled is undefined.

When the underlying hardware offers ways to phisically disable networking (such as entering in low power mode or similar), these must be used. Otherwise, disabling must be done by software.

### 3.6. ETH_DUPLEX: Configure duplex mode

* Input:
  * A = 5
  * B = action to perform:
    * 0: Obtain current mode only 
    * 1: Set half-duplex mode 
    * 2: Set full-duplex mode 

* Output:
  * A = Mode after routine execution: 
    * 1: Currently half-duplex mode set 
    * 2: Currently full-duplex mode set 
    * 3: Current mode unknown or duplex mode does not apply

Offering the ability to change the duplex mode is optional. When not possible (because only one mode is supported, or because it is not possible to change the current mode by software), the routine must simply return the current mode, as if it were called with B=0.

When it is possible to change the duplex mode by software, the default mode (after a reset, see [Section 3.2](#32-eth_reset-reset-hardware)) should be the half- duplex mode.

The "Unknown or does not apply" mode is primarily intended for implementations that do not act on real Ethernet hardware. For these implementations, the concept of "duplex mode" may not make sense.

### 3.7. ETH_FILTERS: Configure frame reception filters

* Input:
  * A = 6 
  * B = Filter bitmask: 
    * Bit 7: Set to return current configuration only
    * Bit 6: Reserved 
    * Bit 5: Reserved 
    * Bit 4: Set to enable promiscuous mode, reset do disable it 
    * Bit 3: Reserved 
    * Bit 2: Set to accept broadcast frames, reset to reject them 
    * Bit 1: Set to accept small frames (smaller than 64 bytes), reset to reject them
    * Bit 0: Reserved

* Output: 
    * A = Filter configuration after execution (bitmask with same format as B at input)

This routine allows to set or to check the current frame reception filters. Frames whose destination Ethernet address matches the current address of the underlying hardware must always be accepted; these filters decide what to do with special frames.

When bit 7 of B is set, all other bits will be ignored, and the routine will simply return a bitmask with the current configuration, without modifying anything.

When bit 4 is set, promiscuous mode is enabled. In this mode, all received frames are enabled, regardless of their destination address. This mode is usually never enabled, except for special purposes such as network analysis.

When bit 2 is set, broadcast frames are accepted. These frames have a destination address of FF-FF-FF-FF-FF-FF, and are intended to be received by all hosts in the network. Broadcats frames are usually always accepted.

When bit 1 is set, small frames (frames smaller than 64 bytes) are accepted. These frames violate the Ethernet specification but may anyway contain useful information.

Reserved bits must be set to zero at input, and are always returned as zero at output. These bits may be used in future versions of the Ethernet API specification.

### 3.8. ETH_IN_STATUS: Check for received frames availability

* Input: 
  * A = 7 

* Output: 
  * A = availability of incoming frames:
    * 0: No received frames available 
    * 1: At least one received frame is available
  * BC = Size of the oldest available frame (only if A=1)
  * HL = Bytes 12 and 13 of the oldest available frame (only if A=1)

This routine checks whether there are received frames available to be retrieved or not. When A=1 is returned, it means that there is at least one received frame available to be retrieved; there is no way to know how many frames are available.

When at least one frame is available, BC returns the total size of the oldest frame. This size includes the Ethernet header and data; it corresponds to the frame format outlined in Appendix A, which does not include the frame preamble nor the checksum.

When at least one frame is available, BC returns bytes 12 and 13 of the oldest frame. These bytes contain ether the frame size or the ether-type, depending of the frame type (Ethernet II or IEEE802.3). Since the frame data starts at a different boundary on each frame type, client software may use this information to decide where to retrieve the frame, so that the data boundary is always at the same memory address. This may ease frame data manipulation.

The oldest received frame can be retrieved using ETH_GET_FRAME, see [Section 3.9](#39-eth_get_frame-retrieve-the-oldest-received-frame).

### 3.9. ETH_GET_FRAME: Retrieve the oldest received frame

* Input:
  * A  = 8
  * HL = Destination address for the frame, or 0 to discard the frame

* Output:
  * A = operation result:
    * 0: frame has been retrieved or discarded
    * 1: no received frames are available
  * BC = Size of the retrieved frame (if A = 0)

This routine retrieves the oldest received frame (copies it to the specified memory address), and removes it from the implementation internal buffer (usually belonging to the underlying harwdare), so that when the routine is called again, the next received frame is obtained. Successive calls to this routine must return the received frames in the same order as they are received from the network.

The received frame will include the Ethernet header and data, it will not include the frame preamble nor checksum. The frame format will be one of the two formats outlined in Appendix A.

Implementations may not allow the destination address to be a page 1 address (in the range 4000h-7FFFh). Client software should not use this range as destination address when invoking this routine, in order to correctly interoperate with such implementations.

When the specified destination address is zero, the frame is not retrieved but just discarded. That is, it is removed from the implementation internal buffer but not copied to memory.

This specification does not impose a minimum size for the internal reception buffer, other than being big enough to hold one complete frame. But of course, the bigger the better, so that it can hold enough frames, thus minimizing the probability of losing frames. When this buffer is full, new frames must be discarded so that the oldest ones are preserved.

### 3.10. ETH_SEND_FRAME: Send a frame

* Input: 
  * A = 9
  * HL = Frame address in memory
  * BC = Frame length
  * D = Routine execution mode:
    * 0: Synchronous
    * 1: Asynchronous
   
* Output:
  * A = operation result:
    * 0: Frame sent, or transmission started
    * 1: Invalid frame length
    * 3: Carrier lost
    * 4: Excessive collisions
    * 5: Asyncrhonous mode not supported

This routine sends to the network the frame that the client software has composed in memory, at the address specified in HL. The frame format must be one of the two outlined in Appendix A; that is, the frame must be composed of Ethernet header (including the local Ethernet address, see [Section 3.3](#33-eth_get_hwadd-obtain-the-ethernet-address)) and data only, with no preamble nor checksum. (When there is real underlying Ethernet hardware, it is expected that the hardware itself will generate the checksum; otherwise, the implementation must generate it prior to sending the frame.)

Implementations may not allow the frame source address to be a page 1 address (in the range 4000h-7FFFh). Client software should not use this range as source address when invoking this routine, in order to correctly interoperate with such implementations.

Allowed frame lengths are 16 to 1514. If the frame is smaller than 64 bytes, the implementation must pad it with zeros up to 64 bytes (usually the underlying hardware will do it automatically).

All implementations must support the syncrhonous execution mode. In this mode, the routine will start the frame transmission; then will wait until transmission has finished (successfully or not) and will return the appropriate success or error code (in this mode, error code zero means "frame successfully sent").

Implementations may optionally support the asyncrhonous execution mode, and should indeed support it when there is real Ethernet hardware that handles the transmission independently of the Z80 code execution. In this mode, the routine initiates the transmission (unless frame length is invalid, in which case A=1 must be returned) and returns immediately with A=0 (meaning "frame transmission started"). Frame transmission status can then be ckeched by using ETH_OUT_STATUS (see [Section 3.11](#311-eth_out_status-check-frame-transmission-status)).

To check whether asynchronous mode is supported or not, try to send a dummy 16 byte frame (12 zero bytes for source and detination address and then FFFFh for the ether-type) in asyncrhonous mode, and check the return code: it will be 0 if asyncrhonous execution is supported, 5 otherwise.

### 3.11. ETH_OUT_STATUS: Check frame transmission status

* Input:  A = 10

* Output: 
  * A = operation result:
    * 0: No frames were sent since last reset
    * 1: Now transmitting
    * 2: Transmission finished successfully
    * 3: Carrier lost
    * 4: Excessive collisions

This routine returns the state of the last frame send operation initiated by ETH_SEND_FRAME (see [Section 3.10](#310-eth_send_frame-send-a-frame)). It is intended for being used after an asyncronous execution of that routine, but will return valid information for syncrhonous executions as well.

In case of syncrhonous execution, this routine will return the same error code that ETH_SEND_FRAME returned, except that code 0 is converted to 2, and code 1 (invalid length) is not considered a failed transmission (since the routine returns without attempting the transmission).

The obtained information is persistent: successive executions of this routine will always return the same value until another frame is sent (or attempted to be sent) or the reset routine (see [Section 3.2](#32-eth_reset-reset-hardware)) is invoked.

### 3.12. ETH_SET_HWADD: Set the Ethernet address

* Input:
  * A = 11
  * L-H-E-D-C-B = Ethernet address to set
   
* Output:
  * L-H-E-D-C-B = Ethernet address after the routine execution

This routine sets the hardware address if the implementation supports it. When changing the address is not supported, the routine must do nothing and return the current address as if the ETH_GET_ADDRESS routine were called (see [Section 3.3](#33-eth_get_hwadd-obtain-the-ethernet-address)). Setting the address should not be allowed unless there is a good reason for it (for example, when it is not possible to store the address in non-volatile memory).

Ethernet addresses (also called MAC addresses) are unique for each physical card and are intended to never be changed after the card is manufactured. See [Appendix A](#appendix-a-ethernet-frame-formats).

## Appendix A. Ethernet frame formats

In this appendix the Ethernet frame formats are described. This information is provided for reference only, and it is not exhaustive. More detailed information can be found on Internet.

There are two types of Ethernet frame: Ethernet II, and IEEE802.3 with SNAP extension (IEEE in short).

The Ethernet II frame format is as follows (one line is 32 bits):

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                 Ethernet address of receiver                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  E. Add. of receiver (cont.)  |       E. Add. of sender       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Ethernet address of sender (continues)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Ether-Type           |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
.                                                               .
.                    Ethernet frame data                        .
.                                                               .
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The IEEE frame format is as follows:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                 Ethernet address of receiver                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  E. Add. of receiver (cont.)  |       E. Add. of sender       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Ethernet address of sender (continues)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Frame length          |      170      |      170      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|       3       |       0       |       0       |       0       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Ether-Type           |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
.                                                               .
.                    Ethernet frame data                        .
.                                                               .
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The Ethernet addresses (also called MAC address), six bytes long, are unique for each card and are assigned in the factory. The address composed by all ones (six FFh bytes) is special: it is the broadcast address, and when it appears on the destination address field, it indicates that the frame is addressed to all the machines on the network, rather than to a given machine only.

Normal Ethernet addresses have the lowest bit of the first byte set to zero. When it is one, it is a multicast address (the frame is addressed to a group of computers).

The minimum size of an Ethernet frame (including all the headers) is 64 bytes. The maximum size is 1514 bytes.

The "Frame length" of IEEE frames counts from the first byte with value 170. That is, it equals the frame data part plus eight.

The "Ether-Type" field indicates the type of the frame being transported on the frame data part. The two most commonly used types are:

* 0800h: IP datagram
* 0806h: ARP frame

Both frame types, Ethernet II and IEEE, can exist mixed in the same network. To know whether an incoming frame is Ehternet II type or IEEE type, the 16 byte number placed at positions 12 and 13 of the frame must be checked:

* If the number is less than or equal to 1500, it is an IEEE frame (the number is the frame length).

* If the number is greater than 1500, it is an IEEE frame (the number is the frame Ether-Type). All the Ether-Type codes are greater than 1500.

When sending frames, they must be sent in Ethernet II type frames unless we know for sure that the machines on the network work exclusively with the IEEE format.

Note that the frame length and Ether-Type fields are stored in Big- Endian format, that is with the high byte first; contrarywise to MSX, that stores the 16 bit numbers in Little-Endian format, with the low byte first.

## Appendix B. Acknowledgements

The original text version of this document was produced using xml2rfc v1.34 (of http://xml.resource.org/) from a source in RFC-2629 XML format.

## Appendix C. Version history

* Version 1.1:

  * The ETH_GET_HWADD routine now only gets the hardware address (see [Section 3.3](#33-eth_get_hwadd-obtain-the-ethernet-address)) 
  * Added the ETH_SET_HWADD routine (see [Section 3.12](#312-eth_set_hwadd-set-the-ethernet-address))
