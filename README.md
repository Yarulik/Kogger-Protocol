# Open Serial Binary Protocol (SBP) specification

## Protocol frame structure

SYNC1 | SYNC2 | ROUTE | MODE | ID | LENGTH | PAYLOAD | CHECK1 | CHECK2|
|----------|----------|----------|----------|----------|----------|----------|----------|----------|
U1 | U1 | U1 | U1 | U1 | U1 | BYTE[LENGTH] | U1 | U1 |
0xBB | 0x55 | BITFIELD | BITFIELD |  1 … 255 |  0 … 128 | BYTEARRAY | 0 … 0xFF | 0 … 0xFF |

### ROUTE
Name | Bits | Description
|----------|----------|----------|
DEV_ADDRESS | 0:3 bit | Device address. Default and broadcast address is 0x0.
RESERVED | 2 bit | Reserved

### MODE
Name | Bits | Description
|----------|----------|----------|
TYPE | 0:1 bit | 0 — Reserved, 1 — CONTENT: DEVICE →  HOST, 2 — SETTING: HOST →  DEVICE, 3 —GETTING: HOST →  DEVICE
RESERVED | 2 bit | Reserved
VERSION | 3:5 bit | Field defines the payload data version
MARK | 6 bit | Once device is switched on, this flag is always in reset state (ZERO). It can be set to active state (ONE) by the host (see the CMD_MARK command) and the slave device keeps the flag in active state in every frame until hardware reset occurs or is reset by the host. Therefore the host monitors the device's actual settings.
RESPONSE | 7 bit | HOST → DEVICE: Set the flag to active state (ONE) in order to get the result of processing the command. The flag doesn't affect the response if one is provided by the TYPE field. DEVICE → HOST: The flag is in reset state (ZERO) by default. Payload goes according to the command specification. If flag is set, the payload contains the result of command processing (see CMD_RESP command).

## Checksum
The checksum algorithm used is the Fletcher-16.
Example source code for calculating the checksum:
```
uint8_t CHECK1 = 0;
uint8_t CHECK2 = 0;
void CheckSumUpdate(uint8_t byte) {
	CHECK1 += byte;
	CHECK2 += CHECK1;
}
```

## Number Formats
- Multi-byte values are ordered in Little Endian format
- Floating point values are transmitted in IEEE754 single or double precision
- bit-field in LSB format

Name | Type | Size (Bytes) | Range
|----------|----------|----------|----------|
S1 | int8_t | 1 | -128 ... 127 
U1 | uint8_t | 1 | 0 … 255
S2 | int16_t | 2 | -32768 … 32767
U2 | uint16_t | 2 | 0 … 65535
S4 | int32_t | 4 | -2'147'483'648  ... 2'147'483'647 
U4 | uint32_t | 4 | 0 … 4'294'967'295
F4 | float | 4 | -1*2^+127 ... 2^+127 
D8 | double | 8 | -1*2^+1023 ... 2^+1023 
