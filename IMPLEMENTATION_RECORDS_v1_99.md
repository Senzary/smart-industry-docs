# Choices & log for v1.99 implementation 

## 🏢 provisioning customers & partners

customer groups where created for `Clients` and `Partners`, where the first are our clients with their subsidiaries and clients as sub-customers; and the latter represent handlers and other business partners that intervene in some part of the process.

```ts
// this is what customer schema looks like
enum CustomerGroups {
    CLIENTS = "Clients",
    PARTNERS = "Partners"
}
interface Customer {
    title: string;
    // customerCode is a 5-10 char long unique value
    // to represent a customer 
    // e.g.: aesglobal, aespanama, mgadist... 
    customerCode: string;
    groups: CustomerGroups[];
    isotypeUrl?:: string,
    allowWhiteLabel?: boolean;
    parentCustomerId?: string;
    description?: string;
    homeDashboard?: string;
    country?: string;
    city?: string;
    stateProvince?: string;
    zipCode?: string;
    address?: string;
    address2?: string;
    phone?: string;
    email?: string;
};
```

customers will have user groups `Subsidiaries` and `Clients`, to be created automatically on customer creation. 

more groups can be added in the future if the need arises, a script on api manager should be used then to create those groups for current active customers.

other entity groups where also created for customers and should happen automatically on creation:

| name | entity type |
| :-- | :--: |
| devices | DEVICE |
| assets | ASSET |
| areas | ASSET |
| locations | ASSET |
| regions | ASSET |
| user-management | DASHBOARD |
| device-management | DASHBOARD |
| asset-management | DASBOARD |
| customer-admin | USER |
| customer-operator | USER |
| customer-viewer | USER | 

user groups where created based on these roles, created as generic roles on `TENANT`:

| name | type | devices | assets | alarms | users | dashboards |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: |
| manager | GENERIC | read, update | create, read, update, delete | read, update, delete | create, read, update, delete | read |
| operator | GENERIC | read | read, update | read, update | read | read |
| viewer | GENERIC | read | read | read | read | read |

### 🤖 process automation

this process will be automated using an api manager https endpoint, where the client should be a zoho integration script or a senzary staff member using an http client to send the request.

-----

## 🏭 provisioning asset templates

we've added a set of provisioning assets as required by the desire to incorporate hardware preparation & shipment to client into smart industry, whether done by us or one of our partners.

for this we have added an asset type `provisioning` that has two templates thus far:

- `package-template` in order to represent a box with devices that belong to a bundle and may be sent in shipments along with other packages

- `shipment-template` in order to represent a shipment as a set of packaged sent together to a final address

we've also implemented an asset type `bundle` and a `data-room-bundle-template` of such type in order to represent a set of devices that are to be sent to a customer's location.

```js
// what a bundle would look like
// <customerCode>-bundle-<token_hex(16)>  
dataRoomBundleEntity.name = "mgadist-bundle-f9bf78b9a18ce6d46a0cd2b0b86df9da"; 
const dataRoomBundleFields = {
    "displayName": {
        "type": "string",
        "description": "human readable name for this bundle",
        "required": true
    },
    "template": {
        "type": "string", // data-room-bundle-template
        "description": "complete name for this bundle template; this allows bundle to know what type of devices it requires.",
        "required": true
    },
    "status": {
        "type": "enum",
        "enumOptions": ["pending", "partial", "complete"],
        "description": "status for this bundle regarding number of devices already shipped to client.",
        "editable": true,
        "default": {
            "value": "pending"
        },
    },
    "devices": {
        "type": "json",
        "description": "an array of json objects describing deviceType and units of type.",
        "default": {
            "value": [{
                "deviceType": "air-conditions-sensor",
                "qty": 1
            }]
        }
    },
};
const bundleRelations = [{
    "to": "ASSET", // of type package
    "type": "Contains"
}, {
    "from": "ASSET", // of type location
    "type": "Contains"
}];
```

``` js
// what a package would look like
const packageFields = {
    "name": { // <partner.customerCode>-package-<int>
        "type": "string",
        "required": true,
        "description": "unique name for this package, created by system."
    },
    "status": {
        "type": "string", // pending-shipment, shipped, delivered
        "description": "status for this package regarding being shipped to client.",
        "required": true,
        "default": "pending-shipment"
    },
    "trackingId": {
        "type": "string",
        "description": "track ID for shipment that has this package.",
        "editable": true
    },
    "trackingUrl": {
        "type": "string",
        "description": "url to click & track shipment.",
        "editable": true
    },
    "writtenAddres": {
        "type": "string",
        "description": "address of the location related to bundle that contains this package"
    }
};
const packageRelations = [{
    "from": "ASSET", // of type bundle
    "type": "Contains"
}, {
    "from": "ASSET", // of type shipment,
    "type": "Contains"
}, {
    "to": "DEVICE",
    "type": "Contains"
}];
```

```js
// what a shipment would look like
const shipmentFields = {
    "name": {
        "required": true,
        "type": "string",
        "description": "unique name assigned to shipment, created by system."
    },
    "status": { // shipped, delivered
        "type": "string",
        "description": "status for this shipment.",
        "required": true,
        "default": "shipped",
        "editable": true
    },
    "trackingId": {
        "type": "string",
        "description": "track ID for shipment that has this package."
    },
    "trackingUrl": {
        "type": "string",
        "description": "url to click & track shipment."
    },
    "writtenAddress": {
        "type": "string",
        "description": "written address for this shipment.",
        "required": true
    }
};
const shipmentRelations = [{
    "to": "ASSET", // of type package
    "type": "Contains"
}];
```

```js
// this is facility-template fields, a location
const facilityFields = {
    "displayName": {
        "type": "string",
        "required": true,
        "description": "human readable name for this asset."
    }
    "latitude": {
        "type": "number",
        "description": "for geographical coordinates"
    },
    "longitude": {
        "type": "number",
        "description": "for geographical coordinates"
    },
    "perimeter": {
        "type": "json",
        "description": "required in order to show perimeter on map views for this location."
    },
    "numberOfFloors": {
        "type": "number",
        "description": "how many floors do this facility have."
    },
    "writtenAddress": {
        "type": "string",
        "description": "written address for this location."
    },
    "commercialImageUrl": {
        "type": "string",
        "description": "an url to an image that shows this facility."
    },
    "floorPlanUrl": {
        "type": "string",
        "description": "an url to a floor plan image of this facility."
    }
};

```

the creation of a bundle is normalized through the API Manager using an endpoint for bundle resources.

```py
base_url = "https://127.0.0.1:5000/api/smart-industry/bundles"
method = "POST"
body = dict(
    api="indumetrix",
    tenant="wesco",
    template="data-room",
    # this is mgadist
    partner_id="8f041160-a1d1-11ef-8f10-1b4ca6fae109", 
    # example location name from customer aespanama
    location_name="aespanama-facility-1749942c0918dcdd34ada62acb2a0ff6" 
)
```

what happens is:
- templates are fetched to validate `template`
- groups for `partner` bundle and entity views are fetched
- `entiyView` for partner to access customer location is created and added to groups
- `bundle` is created and its required attributes are updated, in this case `locationEntityViewName` and `template` 

authentication methods are to be implemented to take requests on these endpoints from zoho or thingsboard instances.

### bundle rules

the API Manager request will create the bundle and update its required attributes; once this attributes are updated, then bundle rule chain will take over and do the following:

- relate the bundle to its template (asset bundle `inherits` to asset bundle-template)
- provision bundle according to default values for non required attributes according to template `fields` attributes (bundle templates seem to just need fields for now, no thresholds)
- relate the bundle to its entity view of type `customer-location-for-partner` based on `locationEntityViewName` attribute

the name of the created rule chain on indumetrix is `bundle-asset-rules`.

there were a couple of generic rule chains built as well to be reused for other assets:

- `common_relate-asset-to-template`
- `common_provision-asset-from-template`


## 🚧 WIP

- ✅ implement templates & create rule chains for provisioning assets
- create test location assets for customer based on irl excel file
- provision test devices & create bundles for partner
- build partner dashboards (start on bundles view, use wireframe on draw.io)
- test partner process to receive hardware, create packages, print labels and ship

-----

## 👀 changelog

| timestamp | author | changes |
| :-: | :- | :- |
| 2024-11-14T01:05:26.116Z | @ernestomedinam | creates IMPLEMENTATION_RECORDS_v1_99.md file. |
| 2024-11-20T17:06:53.215Z | @ernestomedinam | updates implementation records after creating bundles. |