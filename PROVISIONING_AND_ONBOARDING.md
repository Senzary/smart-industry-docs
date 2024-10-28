# Provisioning & onboarding

usually these terms are used interchangeably but we will focus on a subtle fact to differentiate their scope:

provisioning
-----
refers to the initial setup & configuration of an entity, in our context usually a device; it's basically about getting it ready to go.

onboarding
-----
refers to integrating an entity into a system, which means shaping and fitting this entity into a specific set of rules, hierarchies & processes; it's basically about getting it to make sense in a larger scope with a given context (e.g.: a business process) 

## 🔖 content

1. [🧬 types of entities to provision & onboard](#-types-of-entities-to-provision--onboard)
2. [🚀 device provisioning](#-device-provisioning)
3. [🎁 device onboarding before shipping](#-device-onboarding-before-shipping)
4. [🚚 onboarding on shipping](#-onboarding-on-shipping)
5. [📩 onboarding on hardware reception on client site](#-onboarding-on-installation-on-client-site)
6. [🔧 onboarding on installation on client site](#-onboarding-on-installation-on-client-site)
7. [🏭 asset onboarding before shipping](#-asset-onboarding-before-shipping)
8. [🔗 resources](#-resources)

-----

## 🧬 types of entities to provision & onboard

### for a device

| process | where | who | when | how | in status | out status |
| :-- | :--: | :-- | :-- | :-- | :--: | :--: |
| provisioning | Thingpark | Sz staff | hardware reception | batch process on junction dashboard based on templates | NA | `provisioned` |
| onboarding | Thingpark & Thingsboard | Sz staff | before shipping | batch process on junction dashboard based on templates | `provisioned` | `not-shipped` |
| onboarding | Zoho & Thingsboard | Sz staff | on shipping | Zoho to Thingsboard api request | `not-shipped` | `shipped` |
| onboarding | Thingsboard | End user | hardware reception | wizard with QR claim process | `shipped` | `connected` |
| onboarding | Thingsboard | End user | installation on site | wizard to assign devices to assets | `connected` | `operative` |

### for other type of entities

| entity | process | where | who | when | how |
| :-- | :-- | :--: | :-- | :-- | :-- |
| asset | onboarding | Thingsboard | Sz staff | before shipping | batch process on junction dashboard based on templates |
| asset | onboarding | Thingsboard | End user | any time | add/edit asset forms & batch process on main dashboard based on templates |
| user | onboarding | Thingsboard | Sz staff | before shipping | batch process on junction dashboard w/user groups |
| user | onboarding | Thingsboard | End user | any time | add/edit user/user groups forms on main dashboard |

-----

## 🚀 device provisioning

when hardware is received an initial setup protocol is performed and a `csv` formatted text file is created with:

- devEUI
- joinEUI/appEUI
- appKey
- tags `manufacturer:<Manufacturer>` `model:<model-x>` `templateId:<device-template-id>`*
- deviceName as `<manufacturer>-<model.toLowerCase()>-<shortDevEUI.toUpperCase()>`
- list of configuration fields based on device-template with default values for this device; e.g.: for all devices, `reportInterval` should be defined; e.g.: for a submergible level sensor, `range` is needed as a manufacturing value*

once the file is ready, a batch process for provisioning is executed; this feature is to be available on the junction dashboard on our Thingsboard main tenant.

the goal for this process is to provision the devices on a Thingpark tenant and perform conectivity tests; directly when its a gateway, or through any available local LoRaWAN gateway when an end node.

> (*): if template exists when device is being provisioned.

> note: templateId refers to the device-template that will serve to caractherize this device on Thingsboard, which is necessary so that when the device connects and starts sending messages, the rule chain is able to create necessary attributes for the device entity; go here for more on [device templates](./DEVICES.md).

-----

## 🎁 device onboarding before shipping

before sending a device to a customer or end user it must be contextualized, or better put, we must fit it into its customer's context; hence, we shall come up with:

- tag for templateId if template did not exist when device was provisioned `templateId:<device-template-id>`
- tag for device groups, whether through several `group:this-group` or one `groups:this-group,that-group`, ideally using kebab-case syntax
- tag for customer code `customer:<customer-code>` which should be an attribute of the customer entity on Thingsboard, ideally 3-5 numbers/letters with kebab-case syntax; e.g.: `customer:shell` `customer:aes` `customer:jax`
- list of configuration fields based on device-template with default values for this device; e.g.: for all devices, `reportInterval` should be defined; e.g.: for a submergible level sensor, `range` is needed as a manufacturing value 

this process should be similar to the previous one as it should also be possible to run it for a batch of devices.

pending detail from implementation in order to understand if process happens by updating all attributes (confirming or overriding those added on previous process), or only be adding up tags and updating specific properties on thingpark.

also pending detail from implementation as to whether update & set up rulechains to cover any required action on Thingsboard, or if it's necessary to do some Thingsboard fetching on this process. 

-----

## 🚚 onboarding on shipping

this will happen whether through a ZoHo integration that sends a request to Thingsboard api in order to update `administrativeStatus` attribute for devices as they are being shipped.

this could be as an action triggered by satus update on ZoHo, for instance when shipping a device updating its status to shipped and performing this action; the action would send a post request to `/api/device` with required body and querystring data in order to have the device update its server attribute `administrativeStatus` from `not-shipped` to `shipped`.

this process should trigger an Attributes Updated message for the asset, if using asset template rulechains through inheritable `ruleChainId` attributes.

----- 

## 📩 onboarding on hardware reception on client site

end user will receive shipped devices and want to check them out as soon as possible; in general terms, the bigger the client the higher the probability there will be a time between device arrival to site and device arrival to end user.

regarding this, there could be two approaches to this device claiming process:

| scenario | pros | cons |
| :-- | :-- | :-- |
| QR code is used to return a string representing a device EUI, so that end user on site is able to log in, start claim process with QR code, and scan the QR code as part of the process. | <ul><li>no custom QR tag required</li><li>easier to implement</li></ul> | <ul><li>no info about device since shipped</li></ul>  |
| QR code is used as unauth link that takes user to a page where the device gets scanning device coordinates and has a `connect device` button that takes user to log in and pipes them to the claim process with QR code. | <ul><li>info on device whenever tag is scanned</li></ul> | <ul><li>requires custom QR & buying/applying new tags to devices</li><li>requires creating an additional view</li></ul> |

input administrative status for device is: `shipped`; output status is `connected`.

> note: 👀 pending choice on approach! 

## 🔧 onboarding on installation on client site

the goal of this process is to help the user physically install the device on its corresponding asset after having it connected.

ideally the asset that this device will be assigned to (installed on) is already an existing entity in IoTlogIQ.

this wizard should allow the user to start the process for a specific device starting with a set of tips and resources with installationn related information (text, images & video curated by Sz staff, maybe hosted as public confluence pages).

installation should include assigning the device to its corresponding asset -has to be under the scope of the end user-.

after installing the device & updating its administrative status to `operative` from `connected`, then the user may be redirected from the confirmation view that completes this process to the assigned asset default widget/dashboard, or the device default widget/dashboard if the device was not assigned to an asset on this process.

## 🏭 asset onboarding before shipping

similar to device onboarding before shipping, this process consists on creating and characterizing every asset out customer's end users will require in order to provide administrative structure and business logic to their smart industry solutions.

> note: 🚧 Work in progress @ernestomedinam.

## 🔗 resources

1. [diagrams on draw.io](https://drive.google.com/file/d/1ebPrsc_xBOc4PTnb4uB9ddq7Oij5tWX5/view?usp=sharing)

-----

## 👀 changelog

| timestamp | author | changes |
| :-: | :- | :- |
| 2024-10-28T21:11:27.673Z | @ernestomedinam | adds changelog to PROVISIONING_AND_ONBOARDING.md file. |