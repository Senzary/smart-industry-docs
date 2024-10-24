# Devices

*current version:* **1.0**

in iotlogiq a device represents a actual sensor that will be assigned to one asset at a time.

a device is fabricated by a manufacturer, should belong to a customer, has a model, an unique name, a device type (also refered to as device profile), and belongs to one or more groups.

it sends messages through gateways registered on different LNS providers, with information about telemetry and other relevant data (uplinks).

it is also able to receive messages that represent commands to change configuration values on the device, or to have it perform a specific action, or behave in a specific way. (downlinks).

-----

## 🧬 current types of devices

we currently use device profile/type to group similar sensors based on their purpose and the telemetry the can deliver. 

however, because device profiles cannot have attributes added so that devices that derive from in inherit information that characterize them, we are in a similar situation as we were with the asset types.

----- 

## 🔗 current relationship types

currently we can assign many devices to an asset of type machine.

- `Machine` 1---deviceToMachine--->n `Device`

according to smart industries original requirement, areas should also be able to relate to devices directly; however, this would break customer main dashboard `location` -> `area` -> `device` structure.

- `Area` 1---deviceToArea--->n `Device`

-----

## 🎯 what about specific device characterization?

so, let's say we need to create a submergible level sensor that has a specific range that comes out of its fabrication; this is because they are meant to be used at a certain depth if under water.

currently we would have to do it manually, adding an attribute `range` to every device of type `submergible-level-sensor` we create.

we could also try to automate it on the onboarding process, but it would probably require hard coding on decoder level.

## 🏭 device template proposal

just as it is proposed for assets, the goal here would be for us to be able to create devices of a specific device type (profile) that represents the standard template for such a device.

using the previous example, we would create a `submergible-level-sensor-template` that would contain these `configuration` & `thresholds` fields.

it would also make sense to add `telemetryPayload` & `attributesPayload` attributes to represent expected decoder output regarding telemetry and attributes objects.

```js
// for the submergible level sensor, for instance
const configurationFields = {
    range: {
        type: "number",
        required: true,
        description: "depth under water that this device was built for.",
        units: {
            metric: "m",
            imperial: "ft"
        }
        editableAfterCreation: false
    },
    reportInterval: {
        type: "number",
        required: true,
        description: "device reporting interval.",
        units: {
            metric: "s",
            imperial: "min"
        }
    }
};
SubmergibleSensorTemplate.serverAttributes.configuration = configurationFields;
```

> note: 🚧 Work in progress @ernestomedinam.