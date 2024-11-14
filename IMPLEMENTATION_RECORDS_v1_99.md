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

- `bundle-template` in order to represent a set of devices that are to be sent to a customer's location

- `package-template` in order to represent a box with devices that belong to a bundle and may be sent in shipments along with other packages

- `shipment-template` in order to represent a shipment as a set of packaged sent together to a final address

```js
// what a bundle would look like
const bundleFields = {
    "name": { // '<customerCode>-<bundle-kit>-<int>
        "type": "string", 
        "description": "name for a bundle related to a customer location.",
        "required": true
    }
    "kit": {
        "type": "string", // data-room, rest-room,
        "description": "name for kit this bundle is based on; this allows bundle to know what type of devices it requires."
    },
    "status": {
        "type": "string", // pending, partial, complete
        "description": "status for this bundle regarding number of devices already shipped to client.",
        "required": true,
        "editable": true,
        "default": "pending"
    },
    "deviceTypes": {
        "type": "json",
        "description": "an array of json objects describing deviceType and units of type."
    }
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
        "description": "address of the location related to bundle that contains this package",
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
        "required": true,
    }
};
const shipmentRelations = [{
    "to": "ASSET", // of type package
    "type": "Contains"
}];
```

## 🚧 WIP

- implement templates & create rule chains for provisioning assets
- create test location assets for customer based on irl excel file
- provision test devices & create bundles for partner
- build partner dashboards (start on bundles view, use wireframe on draw.io)
- test partner process to receive hardware, create packages, print labels and ship

-----

## 👀 changelog

| timestamp | author | changes |
| :-: | :- | :- |
| 2024-11-14T01:05:26.116Z | @ernestomedinam | creates IMPLEMENTATION_RECORDS_v1_99.md file. |