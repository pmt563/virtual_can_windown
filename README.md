# Feeder Class Diagram & Run Instructions

## Class Diagram

```mermaid
classDiagram
    class Feeder {
        -_running: bool
        -_kuksa_client: ClientWrapper
        -_canclient: CANClient
        +start()
        +stop()
        -_process_can_messages()
        -_process_vss_updates()
    }

    class CANClient {
        -_bus
        -_kuksa_client
        +recv() CANMessage
        +send(arbitration_id, data)
        +stop()
    }

    class CANMessage {
        -msg: can.Message
        +get_arbitration_id() int
        +get_data() bytes
        +is_extended_id() bool
        +get_timestamp() float
    }

    class ClientWrapper {
        <<abstract>>
        +start()
        +stop()
        +update_datapoint(path, value) bool
        +subscribe(paths, callback)
        +is_connected() bool
        +supports_subscription() bool
    }

    class DatabrokerClientWrapper {
        +update_datapoint(path, value) bool
        +subscribe(paths, callback)
    }

    class ServerClientWrapper {
        +update_datapoint(path, value) bool
        +subscribe(paths, callback)
    }

    class LoggingClientWrapper {
        +update_datapoint(path, value) bool
        +subscribe(paths, callback)
    }

    Feeder --> CANClient : uses
    Feeder --> ClientWrapper : uses

    CANClient --> CANMessage : creates

    ClientWrapper <|-- DatabrokerClientWrapper
    ClientWrapper <|-- ServerClientWrapper
    ClientWrapper <|-- LoggingClientWrapper

    CANClient --> can.interface.Bus : uses
    CANClient --> kuksa_can_bridge.CanClient : optional
```

------------------------------------------------------------------------

## Run the application

``` bash
docker run -it --rm \
  --name hr_zonal \
  --network kuksa-net \
  -e LOG_LEVEL=DEBUG \
  -v "${PWD}\net_conf.ini:/dist/config/dbc_feeder.ini" \
  -v "${PWD}\vss_dbc.json:/config/vss_dbc.json" \
  -v "${PWD}\vss_dbc.json:/dist/vss_dbc.json" \
  -v "${PWD}\dbc_default_values.json:/dist/dbc_default_values.json" \
  -v "${PWD}\handsomeno1.dbc:/dist/HnR.dbc" \
  ghcr.io/pmt563/virtual_can_windown/hr-zonal:sha-f0ad370@sha256:0f025c53ca36dab4e49e0f6e8b5777f57e7b80f4d6ed4a3b45dc07c6dd445544 \
  --server-type kuksa_databroker \
  --dbc2val \
  --val2dbc \
  --lax-dbc-parsing
```