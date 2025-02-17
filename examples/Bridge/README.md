# Bridge functions

## Basic

Basic 'Bridge'. Indeed this sample pulling data from Modbus Server and stores it to local registers. Local registers can be accessed via Modbus Client instance that running aside.

## True

Fullfunctional ModbusRTU to ModbusTCP bridge.

## MultipeServerID

Respond for multiple ModbusRTU IDs from single device

```c
uint16_t rawRequest(id_ip, uint8_t* data, uint16_t len, cbTransaction cb = nullptr, uint8_t unit = MODBUSIP_UNIT);
uint16_t rawResponce(id_ip, uint8_t* data, uint16_t len, uint8_t unit = MODBUSIP_UNIT);
uint16_t errorResponce(id_ip, Modbus::FunctionCode fn, Modbus::ResultCode excode, uint8_t unit = MODBUSIP_UNIT);
```
- `id_ip` SlaveId (`uint8_t`) or server IP address (`IPAddress`)
- `data` Pointer to data buffer to send
- `len` Byte count to send
- `unit` UnitId (ModbusTCP/TLS only)
- `fn` function code in responce
- `excode` Exceprion code in responce

```c
union frame_arg_t {
struct frame_arg_t {
    bool to_server; // true if frame is responce for local Modbus server/slave
    union {
        // For ModbusRTU
		uint8_t slaveId;
        // For ModbusTCP/TLS
		struct { 
			uint8_t unitId; // UnitId as passed in Modbus header
			uint32_t ipaddr; // IP address from which frame is received
			uint16_t transactionId; // TransactionId as passed im Modbus header
		};
    };
};
typedef std::function<ResultCode(uint8_t*, uint8_t, void*)> cbRaw; // Callback function Type for STL
typedef ResultCode (*cbRaw)(uint8_t* frame, uint8 len, void* data); // Callback function Type
bool onRaw(cbRaw cb = nullptr);
```
- `frame` Modbus payload frame with stripped MBAP, slaveid and crc
- `len` frame size in bytes
- `data` Pointer to frame_arg_t filled with frame header information
*Returns:*
- If a special error code `Modbus::EX_PASSTHROU` returned frame will be processed normally
- If a special error code `Modbus::EX_FORCE_PROCESS` returned frame will be processed even if addressed to another Modbus unit
- Any other return code disables normal frame processing. Only transactional callback will be executed (if any and transaction data is correct)
The callback is executed only on Modbus frame with valid header and CRC.