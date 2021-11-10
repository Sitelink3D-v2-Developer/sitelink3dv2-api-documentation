# SmartView Machine List SmartApp

The Machine List SmartApp provides a continuous update of machine data. For each machine, two separate records
are returned:

- {machine_urn} : a frame detailing low frequency data such as:
    - urn : key for this machine.
    - device : the hardware sending data for this machine.
    - machine : details for the plant.
    - lease_start : earliest date that this asset context is valid from (which is indicative of when the machine
started providing data today).
    - state : details of machine state (operator/material/delay/. . . ).
    - event : details of most recent haul event (load/dump/close).
    - haul_state: LOADED or EMPTY.
- {machine_urn}.position : a frame detailing high frequency data.

The updates from the Machine List SmartApp are frames of the shape
​
```javascript
{
    "{URN}": {
        "#": "",            // transport hash value
        "at": 163643267415, // ms since epoch that record was generated
        "device": {
            "{KEY}": "{VALUE}"
        },
        "{KEY}": "{VALUE}"
        "urn": "{URN}"
    },
    "{URN}.position": {
        "at": 163643267415, // ms since epoch that record was generated
        "{KEY}": "{VALUE}"
        "urn": "{URN}"
    }
}
```

## Machine URNs
​
Our data is keyed by machine URNS, which look like:
​
`"urn:X-topcon:{type}:ac:{asset_class}:oem:{oem}:model:{model}:sn:{serial_number}:id:{machine_name}"`
​
The various fields are separated out for the `"device"` and `"machine"` records; they can also be overridden in RDM by editing the asset record. See the RDM list example [here](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-examples/tree/main/components/metadata/metadata_list) to view the assets at a site using the API.
​
## Machine Data
​
The records keyed by `{URN}` provide low-frequency information about a machine as aggregated at a point in time.
The data has:
​
- a machine that it represents.
- a device representing the equipment on the machine that sent data. Examples include:
  - GX55.
  - GX60.
  - FC5000.
  - Mobile Phone.
  - Tablet.
  - ISOCAN2.
- state settings that Sitelink3D v2 tracks over time.

​
Position information is sent in separate packets because it changes much more frequently.
​
### Scalar fields

- `"#"` : a hash value computed for this record. This changes whenever the record changes. Ideally you should not see records with the same hash value.
- `"asset_class"` : the machine asset_class value.
- `"at"`: ms since epoch that record was generated.
- `"haul_state"`: a machine goes through states "UNKNOWN", "LOADED", "EMPTY". Typically relevant for haul trucks.
- `"lease_start"`: a field indicating when the device sending data was registered with Sitelink3D v2.
- `"urn"` : the machine URN (also used to key the record).
​

### Asset fields

Fields `device` and `machine` provide objects that detail the URNs expanded into their interesting constituent pieces:
​
- `"device"`:
    - "ac": asset class.
    - "lease_start": field indicating when the device was registered with Sitelink3D v2.
    - "name": alias given to the device.
    - "type": "device".
    - "urn": device URN.
    - "uuid": internal identifier.
- `"machine"`: 
    - "asset_class".
    - "model".
    - "name".
    - "oem".
    - "type": "machine".
​

### State fields
​
The `state` field provides an object that details state associated with the machine.
Values are either
​
1. Internal ids (called `X_uuid` for ids that are constrained as UUIDs or `X_id` for general ids)
2. Data fields (called `X` or `X_name`)

The possible keys expand over time; we currently have (with sample values):
​
- ID fields
    - "activityType_uuid": "a7c1694a-66ef-485a-9130-bbe6a59e1edd"
    - "alignment_id": "ALIGN-1"
    - "delay_uuid": "de063ede-36bc-11ec-b051-42290e164240"
    - "material_uuid": "c7b3f680-ed4a-4c3b-8a6e-5243d365e194"
    - "operator_uuid": "92a5d225-2b09-472b-aaeb-a6fc50724f83"
    - "sequence_id": "SEQ-1"
    - "surface_id": "SURF-1"
    - "task_id": "TASK-1"


- Data fields
    - "delay": "Lift"
    - "material": "Broken Concrete"
    - "operator": "George Jefferson"
    - "sequence_name": "Sequence 1"
    - "surface_name": "Surface 1"
    - "task_name": "Task 1: Broken Concrete"

## Position Data
​
The records keyed by `{URN}.position` provide high-frequency information about a machine position at a point in time.
There are different keys for `type="wgs"` and `type="local"`
​
### WGS records

- General fields
    - "type": "wgs"
    - "at": 1636434050467
    - "urn": ```"urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:ZXA-990"```
​
- Position data
    "altitude": 242.84283150493314
    "direction": 89.53143310546875
    "latitude": 37.33958410824185
    "longitude": -95.51793316689746


### Local records

Local records are produced when both:

1. the site has a localization.
2. data is sent from the machine in local format.
​
- General fields

    - "type": "local"
    - "at": 1636434050467
    - "urn": ```"urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:ZXA-990"```
​
- Position data

    - "northing"  : 1234556.789
    - "easting"   : 1234556.789
    - "elevation" : 2.4
​
## Rotation Data

For localized records, you may also receive packets keyed by `"{URN}.rotation"` which contain
​
- General fields
    - "type": "local"
    - "at": 1636434050467
    - "urn": ```"urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:ZXA-990"```
​
- Rotation data
    - "yaw"
    - "pitch"
    - "roll"

## Sample Output
```
{ 
    "urn:X-topcon:machine:ac:excavator:oem:CAT:model:KITTEN:sn::id:Bob%E2%80%99s%20Excavator": { 
        "#": "6b9b7932", 
        "asset_class": "excavator", 
        "at": 1624230470154, 
        "device": { 
            "ac": "HMI", 
            "lease_start": 1624230470145, 
            "name": "4600e334", 
            "type": "device", 
            "urn": "urn:X-topcon:device:ac:HMI:oem:RDS:model:ISOCAN2:sn:4600e334:id:4600e334", 
            "uuid": "5172243c-f4d8-3781-88c1-207aef725999" 
        }, 
        "haul_state": "UNKNOWN", 
        "lease_start": 1624230470145, 
        "machine": { 
            "asset_class": "excavator", 
            "model": "KITTEN", 
            "name": "Bob%E2%80%99s%20Excavator", 
            "oem": "CAT", 
            "type": "machine" 
        }, 
        "state": { 
            "delay": "Load", 
            "delay_uuid": "6aea7329-b389-11eb-b3a3-2a01fae03fa3", 
            "operator": "George Jefferson", 
            "operator_uuid": "92a5d225-2b09-472b-aaeb-a6fc50724f83" 
        }, 
        "urn": "urn:X-topcon:machine:ac:excavator:oem:CAT:model:KITTEN:sn::id:Bob%E2%80%99s%20Excavator" 
    }, 
    "urn:X-topcon:machine:ac:excavator:oem:CAT:model:KITTEN:sn::id:Bob%E2%80%99s%20Excavator.position": { 
        "altitude": 229.08, 
        "at": 1624230470154, 
        "latitude": 37.35245163, 
        "longitude": -95.53001581, 
        "urn": "urn:X-topcon:machine:ac:excavator:oem:CAT:model:KITTEN:sn::id:Bob%E2%80%99s%20Excavator" 
    }, 
    "urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:James%E2%80%99s%20Hole%20Maker": { 
        "#": "854ae954", 
        "asset_class": "truck", 
        "at": 1624230470101, 
        "device": { 
            "ac": "Phone", 
            "lease_start": 1624230470092, 
            "name": "KKR-318", 
            "type": "device", 
            "urn": "urn:X-topcon:device:ac:Phone:oem:Apple:model:x86_64:sn::id:KKR-318", 
            "uuid": "eaed1060-c454-3b79-9454-bf98cd2ae530" 
        }, 
        "event": { 
            "at": 1624230470101, 
            "material": "Broken Concrete", 
            "material_axis": "volume", 
            "material_state": "Default", 
            "material_unit": "cubic_metres", 
            "material_uuid": "c7b3f680-ed4a-4c3b-8a6e-5243d365e194", 
            "quantity": 8, 
            "region": "Airport - Load Broken Concrete", 
            "region_uuid": "032c90ed-4631-41b3-9cf1-eeca895cac8f", 
            "type": "load", 
            "uuid": "ab5f925c-0504-41e0-8aa5-7600ca34cfb1" 
        }, 
        "haul_state": "LOADED", 
        "lease_start": 1624230470092, 
        "machine": { 
            "asset_class": "truck", 
            "name": "James%E2%80%99s%20Hole%20Maker", 
            "oem": "nonesuch", 
            "type": "machine" 
        }, 
        "state": { 
            "material": "Broken Concrete", 
            "material_uuid": "c7b3f680-ed4a-4c3b-8a6e-5243d365e194", 
            "operator": "Archie Bunker", 
            "operator_uuid": "e5ba2644-1f46-4f9b-b52a-5d7529742561" 
        }, 
        "urn": "urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:James%E2%80%99s%20Hole%20Maker" 
    }, 
    "urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:James%E2%80%99s%20Hole%20Maker.position": { 
        "altitude": 241.89, 
        "at": 1624230470101, 
        "latitude": 37.33705219, 
        "longitude": -95.51040453, 
        "urn": "urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:James%E2%80%99s%20Hole%20Maker" 
    } 
} 
```
