# Inventory Items

The are the `inventoryitems` and `inventoryitem` objects used to import inventory from EMR to SFS.

## The inventoryitems object

> Example `inventoryitems` object with 2 nested `inventoryitem` objects:

```json
{
	"objectType": "inventoryitems",
    "id": "some-operation-id",
    "inventoryitems": [
		{
			"objectType": "inventoryitem",
	        "id": "some-emr-internal-id",
	        "name": "Cefazolin 100",
	        "concentration": 100,
	        "concentrationUnits": "mg",
	        "concentrationVolume": "ml"
		},
		{
			"objectType": "inventoryitem",
	        "id": "some-emr-internal-id2",
	        "name": "Ampicillin"
		}
	]
}
```

This object should be sent with the `/inventoryitems` API method (POST or PUT method).

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events (e.g. `inventory.imported`). Should be assigned `iventoryitems` value
**id** | String | *Optional*. Identificator of the object. Will be transferred to EMR with the SFS events (e.g. `inventory.imported`)
**inventoryitems** | Array | The array of `inventoryitem` objects. See description of the `inventoryitem` object [below](#the-inventoryitem-object)


## The inventoryitem object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events (e.g. `inventory.imported`). Should be assigned `iventoryitem` value
**id** | String | **Required**. The EMR internal ID of the inventory item
**name** | String | **Required**. The unique name of the inventory item
**concentration** | Double | *Optional*. The concentration value for the given medication. This value should not be specified if the inventory item is not a medication.
**concentrationUnits** | String | *Required if the concentration value is specified. Otherwise optional.* Units that define the amount of drug. There is no limitation on what data will be transferred. This value should not be specified if inventory item is not a medication.
**concentrationVolume** | String | *Required if the concentration value is specified. Otherwise optional.* Units for the volume. There is no limitation on what data will be transferred. This value should not be specified if inventory item is not a medication.
**asyncOperationStatus** | Integer | *Optional*. The status of the asynchronous operation for the inventory item. This will be filled in by SFS when sending the `inventoryitems.imported` event. Should be: 1. `less than 0` - error occured; 2. `greater or equal 0` - operation succeed.
**asyncOperationMessage** | String | *Optional*. May contain the error message in case the `asyncOperationStatus` field represents the error (less than 0).

## Create or update single inventory item

> Example Request:

```http
POST /inventoryitem HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
```
```json
{
    "id": "external-inventory-id",
    "name": "Cefazolin 100",
    "concentration": 100,
    "concentrationUnits": "mg",
    "concentrationVolume": "ml"
}
```

Creates or updates single inventory item sent in the request. SFS will attempt to link the transferred inventory item with the internal SFS items by name.

* Url: /inventoryitem
* Method: POST
* Synchronous
* Accepts [`inventoryitem`](#the-inventoryitem-object) object
* Returns HTTP status 201 in case the new item has been created or an existing item has been updated
* In case of error returns the [`Error`](#the-error-object) object

## Create or update multiple inventory items

> Example Request:

```http
POST /inventoryitems HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
```
```json
{
    "id": "some-emr-operation-id",
    "inventoryitems": [
		{
	        "id": "some-emr-internal-id",
	        "name": "Cefazolin 100",
	        "concentration": 100,
	        "concentrationUnits": "mg",
	        "concentrationVolume": "ml"
		},
		{
	        "id": "some-emr-internal-id2",
	        "name": "Ampicillin"
		}
	]
}
```

Asynchronously creates or updates the inventory items sent with the request. Before creating an inventory item system tries to match it with any existing records by name. If exact match is found then records are linked, otherwise a new inventory item is created.

* Url: /inventoryitems
* Method: POST 
* Asynchronous
* Accepts [`inventoryitems`](#the-inventoryitems-object) object.
* Returns HTTP status 200 in case the input data has been accepted for import
* In case of error returns the [`Error`](#the-error-object) object

After an import operation finishes SFS will send the `inventoryitems.imported` [event](#recieving-the-status-of-import-operation) to the registered webhook. We will pass back the same `inventoryitems` object to EMR, and fill in the status of the import operation for every inventory item provided.

<aside class="warning">
This is the asynchronous method. Please expect to recieve and handle the `inventoryitems.imported` event that SFS will send to you after the import is complete.
</aside>

<aside class="notice">
Because of asynchronous nature of this API method, we recommend to pass the unique operation identifier with the `id` field of the `inventoryitems` object. We will send this object back to you with the `inventoryitems.imported` event, and you will be able to distinguish the operation. 
</aside>

## Retreive existing inventory items

* Url: /inventoryitems
* Method: GET
* Synchronous 
* Returns HTTP status 200 and a collection of all [`inventoryitem`](#the-inventoryitem-object) objects linked with EMR
* In case of error returns the [`Error`](#the-error-object) object

## Retreive single inventory item 

Returns the inventory item. Specify the `id` of the inventory item in the EMR. The same `id` that was supplied when object was created

* Url: /inventoryitem/{id}
* Method: GET
* Synchronous 
* Returns HTTP status 200 and an [`inventoryitem`](#the-inventoryitem-object) object linked with EMR
* In case of error returns the [`Error`](#the-error-object) object

## Delete single inventory item

This method deletes an inventory item by id.

* Url: /inventoryitem/{id}
* Method: DELETE
* Synchronous
* If item cannot be found in SFS, the [`Error`](#the-error-object) object will be returned with HTTP 404 status code

## Recieving the status of import operation

> Example of `inventoryitems.imported` event JSON:

```json
{
    "clinicApiKey": "clinic-api-key",
    "eventType": "inventoryitems.imported",
    "object": {
	    "objectType": "inventoryitems",
		"id": "some-emr-internal-id",
		"inventoryitems": [
			{
			    "objectType": "inventoryitem",
		        "id": "some-emr-internal-id",
		        "name": "Cefazolin 100",
		        "concentration": 100,
		        "concentrationUnits": "mg",
		        "concentrationVolume": "ml",
		        "asyncOperationStatus": 0
			},
			{
			    "objectType": "inventoryitem",
		        "id": "some-emr-internal-id2",
		        "name": "Ampicillin",
		        "asyncOperationStatus": -1,
			    "asyncOperationMessage":  "Some error message"
			}
		]
	}
}
```

SFS will send the `inventoryitems.imported` event to the url provided by EMR at the end of [asynchronous import](#create-or-update-multiple-inventory-items) operation. SFS sends the `inventoryitems` object with this event. Every `inventoryitem` object will contain `asyncOperationStatus` and optionally `asyncOperationMessage` in case of errors.

* Url: webhook provided by EMR
* Method: POST
* Synchronous
* Transferres [`inventoryitems`](#the-inventoryitems-object) object included in the `event` object
* Expected response with 200 Http code in case of success.
* In case of the error, EMR should return 400 Http code and optionally the [`Error`](#the-error-object) object
