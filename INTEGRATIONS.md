# Integrations

an integration is basically an endpoint that receives messages from LNS or other clients in a specific manner through a known protocol.

in our case these messages usually refer to a device and carry data about it encoded in a known manner.

this is why integrations have tow data converters; one to decode uplink (incoming) messages, and the other to encode downlink (outgoing) messages.

-----

## 🧬 types of integrations

| type | description |
| :- | :- |
| mqtt | machine to machine network protocol |
| http | text or multimedia files protocol | 
| tcp | connection oriented protocol | 
| CoAP | machine to machine network protocol for constrained devices |
| udp | message oriented transport layer protocol | 
| pub/sub | service for asynchrounous messageing between apps |
| thingpark enterprise | messaging platform for the LPWA network | 
| thingpark | message oriented transport layer protocol | 
| the things stack industries | software to operate private LoRaWAN networks |
| chirpstack | LoRaWAN network configuration server |
| loriot | distributed LoRaWAN infraestructure |
| opc-ua | cross-platform for data exchange from sensors to cloud apps |
| kafka | streaming analytics, data integration, streaming platform |
| azure iot hub | cloud-hosted messaging service |
| aws iot | management, storage, and analytics service |
| others & custom | sigfox, particle, kpn things, azure service bus, apache pulgar, aws kinesis, ibm watson iot, ocean-connect, rabbitMQ... and custom ones!

-----

## 👨🏽‍🎤 who deserves an integration?

should it be a client? should it be an LNS? 

should it be a very complex device, or a family of devices? 

| integration based on | impact on data converters |
| :- | :- |
| a client, i.e.: `Jacksonville Airport Integration` |  <li>decoder is very complex as it has to deal with messages from multi-brand multi-purpose devices</li><li>decoder code is non reusable</li> |
| an LNS, i.e.: `Thingpark integration` | <li>multi-brand multi-purpose devices complexity on decoder</li><li>because the goal is to parse network variables first thing on rulechains, decoder complexity is unaffected in this regard</li> |
| a family of devices with same purpose, i.e.: `Pressure sensors integration` | <li>decoder complexity reduced but still high due to the fact that different manufacturers encode using different methods</li><li>decoder code is reused for every client, but operative complexity is added as a check for device profile in thingsboard is required to figure out a device connection in thingpark</li> |
| ✅ a manufacturer, or a set of its models, i.e.: `Ellenex integration` | <li>decoder complexity is low as one decoding method is required and units are known and consistent for each manufacturer</li><li>decoder code is reused for every client & use case, which is ok because business logic belongs to the assets</li> |

with this in mind, what other variables could make it necessary for a new integration to be created?

### because of the type of the integration

let's suppose we are working with `Aurtron` devices and they are sending uplink messages via an mqtt integration named something like `aurtron-dl53-mqtt-integration`, and then we are prompted to start working with other Aurtron devices but this time through an `http` integration.

so, because Thingsboard works with typed integrations, in this case it is necessary to create this new integration of type `http` and named something like `aurtron-dl53-http-integration`.

### because of a version update on the firmware

let's suppose we are working the `Ellenex` devices and they are using an encoding method `A` for their firmware version `1.0`; eventually they start manufacturing devices with version `2.0` that uses a method `B` to encode payloads.

because we would not be able to stop using the current integration, named something like `ellenex-pressure-models-integration` until all Ellenex devices are updated to the latest firmware, which may also never happen, it makes sense to create a new integration in this scenario, with the same type, but then a different name, something like `ellenex-pressure-models-v2-integration`.

> note: 👀 in any case, the description field for a new integration must be filled with useful information about the reason why the integration is being created.

## 📐 integration naming rules

up to this point, you might have noticed some integration names have spaces, some have capitalized words, some don't. 

well, that should only be the case up until this point in time in which this is being written (read, in your case); we have to set up naming rules for integrations.

proposed rules are:

- it uses kebab-case syntax (no uppercase, hyphens or dashes instead of spaces, no underscores... [more](https://developer.mozilla.org/en-US/docs/Glossary/Kebab_case))
- it specifies manufacturer and model set if necessary (`<manufacturer>-<model-set>`), e.g: `netvox-r718x`, `ellenex-pressure-models`, `milesight`
- it adds integration type and closes with the word `integration`: e.g.: `netvox-r718-http-integration`, `ellenex-pressure-models-mqtt-integration`

> note: 👀 when adding integration type, turn the type's name into kebab case; for the things stack industries, yes, it should look like `<manufacturer>-<model-set>-the-things-stack-industries-integration`

-----

## ⏳ when should the integration be created?

to create an integration like the ones we've described, one might need:

- `a manufacturer's set of devices` that are going to be used to send telemetry in order to us to know things about the assets they are going to be assigned to
- a plan (content + diagrams) that describes `the business case` in which these devices are to be immerse
- a chosen & provisioned `deviceProfile` and `deviceTemplate` for the devices that are to be supported by this integration
- this set of devices documentation & its decoded telemetry fields mapped to its templates expected telemetry fields, its `dictionary` or telemetry map
- all of this aspects documented on the Senzary `asset-device-templates` google sheet file

it is at this point that it really makes sense to create the integration, just as it does to start working on the data converters (test & code), and the entities onboarding processes for customers, assets & users.

-----

## 👀 changelog

| timestamp | author | changes |
| :-: | :- | :- |
| 2024-10-28T21:11:27.673Z | @ernestomedinam | creates INTEGRATIONS.md file |
