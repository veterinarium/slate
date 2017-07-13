# Medics

There are the `medics` and `medic` objects used to import the list of clinic stuff from EMR to SFS.

## The medics object

> Example `medics` object with 2 nested `medic` objects:

```json
{
	"objectType": "medics",
    "id": "some-operation-id",
    "medics": [
		{
			"objectType": "medic",
	        "medicId": "some-emr-internal-id-1",
	        "name": "Dr. Jackson",
	        "medicType": "doctor"
		},
		{
			"objectType": "medic",
	        "medicId": "some-emr-internal-id-2",
	        "name": "Dr. Smith",
	        "medicType": "doctor"
		}
	]
}
```

This object should be sent with the `/medics` API method (POST or PUT method).

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events (e.g. `medics.imported`). Should be assigned `medics` value
**id** | String | *Optional*. Identificator of the object. Will be transferred to EMR with the SFS events (e.g. `medics.imported`)
**medics** | Array | The array of `medic` objects. See description of the `medic` object [below](#the-medic-object)


## The medic object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events (e.g. `medics.imported`). Should be assigned `medic` value.
**medicId** | String | **Required**. The EMR internal ID of the medic.
**name** | String | **Required**. The unique name of the medic for a particular `medicType`.
**medicType** | String | *Required*. The type of the medic imported in SFS. Can be one of the following: 1. `doctor`, 2. `technician`, 3. `anesthetist`. 
**asyncOperationStatus** | Integer | *Optional*. The status of the asynchronous operation for the medic object. This will be filled in by SFS when sending the `medics.imported` event. Should be: 1. `less than 0` - error occured; 2. `greater or equal 0` - operation succeed.
**asyncOperationMessage** | String | *Optional*. May contain the error message in case the `asyncOperationStatus` field represents the error (less than 0).

## Create or update single medic

> Example Request:

```http
POST /medic HTTP/1.1
User-Agent: MyClient/1.0.0
Content-Type: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
```
```json
{
    "medicId": "some-emr-internal-id-1",
    "name": "Dr. Jackson",
    "medicType": "doctor"
}
```

Creates or updates single medic object sent in the request. SFS will attempt to link the transferred medic object with the internal SFS items by name.

* Url: /medic
* Method: POST
* Synchronous
* Accepts [`medic`](#the-medic-object) object
* Returns HTTP status 201 in case the new item has been created or an existing item has been updated
* In case of error returns the [`Error`](#the-error-object) object

## Create or update multiple medics

> Example Request:

```http
POST /medics HTTP/1.1
User-Agent: MyClient/1.0.0
Content-Type: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
```
```json
{
    "id": "some-operation-id",
    "medics": [
		{
	        "medicId": "some-emr-internal-id-1",
	        "name": "Dr. Jackson",
	        "medicType": "doctor"
		},
		{
	        "medicId": "some-emr-internal-id-2",
	        "name": "Dr. Smith",
	        "medicType": "doctor"
		}
	]
}
```

Asynchronously creates or updates the medics objects sent with the request. Before creating a medic object system tries to match it with any existing records by name. If exact match is found then records are linked, otherwise a new medic is created.

* Url: /medics
* Method: POST 
* Asynchronous
* Accepts [`medics`](#the-medics-object) object.
* Returns HTTP status 200 in case the input data has been accepted for import
* In case of error returns the [`Error`](#the-error-object) object

After an import operation finishes SFS will send the `medics.imported` [event](#receiving-the-status-of-the-import-operation) to the registered webhook. We will pass back the same `medics` object to EMR, and fill in the status of the import operation for every object provided.

<aside class="warning">
This is the asynchronous method. Please expect to receive and handle the `medics.imported` event that SFS will send to you after the import is complete.
</aside>

<aside class="notice">
Because of asynchronous nature of this API method, we recommend to pass the unique operation identifier with the `id` field of the `medics` object. We will send this object back to you with the `medics.imported` event, and you will be able to distinguish the operation. 
</aside>

## Retreive existing medics

* Url: /medics
* Method: GET
* Synchronous 
* Returns HTTP status 200 and a collection of all [`medic`](#the-medic-object) objects linked with EMR
* In case of error returns the [`Error`](#the-error-object) object

## Retreive single medic 

Returns the medic object. Specify the `id` of the medic in the EMR. The same `id` that was supplied when the object was created

* Url: /medic/{id}
* Method: GET
* Synchronous 
* Returns HTTP status 200 and an [`medic`](#the-medic-object) object linked with EMR
* In case of error returns the [`Error`](#the-error-object) object

## Delete single medic

This method deletes a medic by id.

* Url: /medic/{id}
* Method: DELETE
* Synchronous
* If item cannot be found in SFS, the [`Error`](#the-error-object) object will be returned with HTTP 404 status code

## Receiving the status of the import operation

> Example of `medics.imported` event JSON:

```json
{
    "clinicApiKey": "clinic-api-key",
    "eventType": "medics.imported",
    "object": {
	    "objectType": "medics",
		"id": "some-emr-internal-id",
		"medics": [
			{
			    "objectType": "medic",
		        "medicId": "some-emr-internal-id-1",
		        "name": "Dr. Jackson",
		        "medicType": "doctor",
		        "asyncOperationStatus": 0
			},
			{
			    "objectType": "medic",
		        "medicId": "some-emr-internal-id-2",
		        "name": "Dr. Smith",
		        "medicType": "doctor",
		        "asyncOperationStatus": -1,
			    "asyncOperationMessage":  "Some error message"
			}
		]
	}
}
```

SFS will send the `medics.imported` event to the url provided by EMR at the end of [asynchronous import](#create-or-update-multiple-medics) operation. SFS sends the `medics` object with this event. Every `medic` object will contain `asyncOperationStatus` and optionally `asyncOperationMessage` in case of errors.

* Url: webhook provided by EMR
* Method: POST
* Synchronous
* Transfers [`medics`](#the-medics-object) object included in the `event` object
* Expected response with 200 Http code in case of success.
* In case of the error, EMR should return 400 Http code and optionally the [`Error`](#the-error-object) object
