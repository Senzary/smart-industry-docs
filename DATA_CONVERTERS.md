# Data converters

data converters are functions in charge of handling incoming and outgoing messages for an Thingsboard integration.

just as explained on [integrations](./INTEGRATIONS.md) there are two types of data converters: one to handle incoming messages and decode encoded uplink payloads, and one to handle outgoing messages after encoding a downlink payload.

## 🔄 what is a decoder?

it's a fundamental part of the uplink data converter, as it allows to translate an incoming message payload into a JSON dictionary that turns it into useful data for our rule chains to turn into information.

the shape of the incoming message depends on the IoT platform sending them; wemostly work with ThingPark and Helium.

in general terms, decoding methods depends on manufacturer choices & standards, so a decoder is usually good for devices of a specific brand or set of models.

a decoder function in Thingsboard receives a payload variable with the encoded message as an array of decimal bytes, and a metadata variable with a JSON dictionary that holds information about the incoming message.

before returning an output, a decoder should make sure that telemetry variable names & measuring units are normalized according to senzary standards (check the [asset-device-templates file](https://docs.google.com/spreadsheets/d/1HixYcAHCBtbior9mxfjL5ProJTaHQr0gGJF7lmwbSPg/edit?usp=sharing)).

## 👩🏽‍💻 what is a decoder development and runtime environment like?

- development environment is very restricted, there is no way to import code nor resources in order to allow reusing code.
- code can be written in JS ES5 and runs on an engine named NashornJS embedded on a Java Virtual Machine (JVM)*, that is rather slow for big loops and probably recursiveness; the alternative is TBEL which is a custom Thingsboard Java implementation.
- it is also impossible to implement TDD or even a set of unit tests in order to make sure output messages are correct for different types of input messages.

> (*) because our Thingsboard instance has been deployed in monolithic mode.

> 💃🏽 we have enabled a ES5 environment repo so that we can code on IDE's, share code and use unit testing on our decoders; check it out at [senzary-decoders](https://github.com/Senzary/senzary-decoders)

## 💡 what should happen in decoder scripts then?

because of the previous three circumstances, our decoders should:

1. **be as specific as they can be** because the more things we do in these scripts, the more undetectable errors and rework we are going to have, as testing for the impact of changes in complex code is cumbersome in these circumstances
2. **be as granular as they can be** because if one decoder is used for many differen type of devices, or for different IoT platform origins, then we have a single point of failure for all of these items, and debugging successfully is hard because of (1)
3. **only normalize telemetry related variable names and units** because standards for this do not require duplicating and maintaining variables with these values, as would be the case when trying to normalize LNS property names, which duplicates code unnecessarily and makes it hell to maintain
4. **be named according to specific guidelines** because if (1) and (2) then it has to be easy to find a specific one amongst the rest of them

## 🎯 what is the desired structure of its output? 

- it should be a standard, predictable object with properties that group information from the device (`telemetry`), information about the device and its context (`attributes`), and Thingsboard required `deviceName` and `deviceType` variables.

desired output would look like a JSON dictionary with the following structure:

```json
{
	"deviceName": "",
	"deviceType": "", // device profile
	"attributes": {
		// devEUI
		// deviceName
		// deviceType
		// deviceSource
		// customer
		// groups
		// model
		// tags -> e.g.: [
		//      "cust:CustomerName",
		//      "model:modBXL01",
		//      "group:Group1",
		//		"group"Group2",
		//      "deviceSource:tpe-aes", where values are <lns-code>-<tenant>
		// ]
	},
	"telemetry": {
		// devEUI
		// deviceName
		// deviceType
		// battery_v
		// stateOfCharge_pct
		// pressure_wc, pressure_bar
		// temperature_c, temperature_f
		// humidity_pct
		// latitude
		// longitude
		// ts

		//      ...any other telemetry device sends, only normalization
		//      required is measure units; proposal is to add unit suffix
		//      for metric and imperial units.
		//      e.g.: level_m (metric), level_ft (imperial)
		
		// raw: this would be the full message received from iot platform
		//      so that network message variables are parsed on rule chain
		//      in order to be able to provide adequate support to LNS parsing
		//      dictionaries.
	}
}
```

proposal is to use camelCase throughout the whole application whenever possible, with the exception of unit specific telemetry values, server/client side attribute variables on rule chains, and similar.

-----


## 📐 data converters naming rules

just as it is the case for integrations and rule chains, we have to set up naming rules for data converters.

proposed rules are:

- it uses kebab-case syntax (no uppercase, hyphens or dashes instead of spaces, no underscores... [more](https://developer.mozilla.org/en-US/docs/Glossary/Kebab_case))
- it follows integration name but replaces `integration` with `decoder` or `encoder` as is the case, e.g: `netvox-r718x-http-decoder`, `ellenex-pressure-models-mqtt-encoder`, `milesight-http-decoder`

-----

## ⏳ when should the data converter be created?

just as it happens with the integrations, the goal is to keep the eye on the business case and consider the goal of the data we are going to receive.

in this sense, before we start building our decoder we must have competed the steps that would allow the integration to be created.

it is absolutely necessary that before working on a decoders code, these steps are met:
- we have the devices documentation & its decoded telemetry fields mapped to its templates expected telemetry fields, its `dictionary` or telemetry map
- we have documented all of this aspects on the Senzary `asset-device-templates` google sheet file

at this point we can create the decoder and start working on the tests and then the code, as described on the README.md file on the [senzary-decoders repo](https://github.com/Senzary/senzary-decoders).


> note: 🚧 Work in progress @ernestomedinam.

-----

## 👀 changelog

| timestamp | author | changes |
| :-: | :- | :- |
| 2024-10-28T21:50:22.528Z | @ernestomedinam | creates DATA_CONVERTERS.md file. |
