# Treatments

The are two objects used by Smart Flow Sheet to notify EMR when medical records entered into the flowsheet:

* `treatment` - this object is sent with the synchronous `treatment.record_entered` event when one medical record has been entered/removed;

* `treatments` - this object is sent with the asynchronous `treatments.records_entered` event when one or several medical records has been entered/removed. Smart Flow Sheet expects EMR to send the `treatments` object back with the `/treatments` API method after the `treatments.records_entered` event has been processed

## The treatments object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events (e.g. `treatment.record_entered`). Should be assigned `treatments` value
**id** | String | **Required if sent to `/treatments` API method**. Identificator of the object. Will be transferred to EMR with the `treatments.records_entered` event. EMR should return this field with the `/treatments` API method
**treatments** | Array | The array of `treatment` objects. See description of the `treatments` object [below](#the-treatment-object)


## The treatment object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | Describes the type of the object transferred with the SFS events (e.g. `treatment.record_entered`). Should be assigned `treatment` value
**inventoryId** | String | Inventory item external id (which was provided with inventory upload). If “Null” then there is no inventory item for this treatment found in SFS
**name** | String | Name of the treatment parameter as it is shown on the flowsheet 
**hospitalizationId** | String | Hospitalization external id (which was provided with hospitalization creation)
**treatmentGuid** | String | **Required**. A unique internal identifier of the treatment item (which corresponds to the treatment at particular hour on a flowsheet) 
**time** | Date | Treatment time (UTC time that corresponds to an hour on a flowsheet). Time format: YYYY-MM-DDThh:mm:ss.sssTZD (e.g. 1997-07-16T19:20:30.000+00:00)
**status** | String | This field describes what have happened to the medical record. Can be one of the following: 1. `changed`, 2. `added`, 3. `removed`, 4. `not_changed`
**qty** | Double | Quantity. In case if the medication has been given to a patient this value will be equal to the `volume` field (described below). Otherwise it will be `1`
**volume** | Double | The volume of the medication that has been given to a patient. In case of non-medication treatment this value will not be provided.
**units** | String | The units of the medication volume (ml, tab, etc). For non-medication items this value will not be provided
**value** | String | The string value that was entered during treatment execution
**mediaPath** | String | The path to the media file that has been attached to the treatment
**mediaContentType** | String | The content type of media file (e.g. `image/jpg`, `video/mp4`, etc...)
**doctorName** | String | The name of the doctor on duty. This value will be provided only in case if the name of the doctor is specified on a correspondent flowsheet
**doctor** | Medic | The [`medic`](#the-medic-object) object that corresponds to the doctor on duty. This value will be provided only in case if associated medic has been imported from EMR
**type** | Integer | The type of the treatment, should be one of following types: `0` - Flowsheet treatment, `1` - Anesthetic treatment, `2` - Treatment task.
**billed** | Boolean | If `true` then treatment should be included in the billing
**asyncOperationStatus** | Integer | **Required if sent to `/treatments` API method**. This field should be filled in by EMR when sending the `treatments` object with the `/treatments` API method. This is usually happens in response to the `treatments.records_entered` async event. This field describes the status of the asynchronous operation for each treatment.  Should be: 1. `less than 0` - error occured; 2. `greater or equal 0` - operation succeed.
**asyncOperationMessage** | String | *Optional*. May contain the error message in case the `asyncOperationStatus` field represents the error (less than 0).

## Retreive single medical record

> Example of `treatment.record_entered` event JSON:

```json
{
    "clinicApiKey": "clinic-api-key",
    "eventType": "treatment.record_entered",
    "object": {
	    "objectType": "treatmet",
		"name": "Cefazolin 100",
		"inventoryId": "external-inventory-id",
		"hospitalizationId": "external-hospitalization-id1",
		"treatmentGuid": "sfs-treatment-guid1",
		"time":"2013-03-28T14:23:56.000+00:00",
		"status": "changed",
		"qty": 3.5,
		"volume": 3.5,
		"value": "IZ",
		"mediaPath": "https://attached-image-webfile-path",
		"mediaContentType": "image/jpg",
		"units": "ml",
		"doctorName": "name-of-the-doctor",
		"doctor": {
		    "objectType": "medic",
	        "medicId": "some-emr-internal-id-2",
	        "name": "name-of-the-doctor",
	        "medicType": "doctor"
			}
	}
}
```

The `treatment.record_entered` event is sent from SFS when one medical record has been entered/removed. SFS will send `treatment` object with all information about medical record.

* Url: webhook provided by EMR
* Method: POST
* Synchronous (can be changed to asynchronous at some point)
* Transfers [`treatment`](#the-treatment-object) object included in the `event` object
* Expected response with 200 Http code in case of success.
* In case of the error, EMR should return 400 Http code and optionally the [`Error`](#the-error-object) object

## Retreive multiple medical records

> Example of `treatments.records_entered` event JSON:

```json
{
    "clinicApiKey": "clinic-api-key",
    "eventType": "treatments.records_entered",
    "object": {
	    "objectType": "treatments",
		"id": "sfs-operation-id",
		"treatments":[
	        {
	            "objectType": "treatment",
	            "name": "Cefazolin 100",
	            "inventoryId": "external-inventory-id",
	            "hospitalizationId": "external-hosp-id1",
	            "treatmentGuid": "sfs-treatment-guid1",
	            "time":"2013-03-28T14:23:56.000+00:00",
	            "status": "changed",
	            "qty": 3.5,
	            "volume": 3.5,
	            "value": "IZ",
	            "units": "ml",
	            "doctorName": "name-of-the-doctor",
				"doctor": {
				    "objectType": "medic",
			        "medicId": "some-emr-internal-id",
			        "name": "name-of-the-doctor",
			        "medicType": "doctor"
					}	            
	        },
	        {
	            "objectType": "treatment",
	            "name": "Temperature",
	            "inventoryId": null,
	            "hospitalizationId": "external-hosp-id2",
	            "treatmentGuid": "sfs-treatment-guid2",
	            "time":"2013-03-28T14:23:56.000+00:00",
	            "status": "added",
	            "qty": 1,
	            "volume": null,
	            "value": "36.6",
	            "units": null,
	            "doctorName": "name-of-the-doctor-2",
	            "doctor": null
	        }		
		]
	}
}
```

The `treatments.records_entered` event is sent from SFS when one or several medical record has been entered/removed. SFS will send `treatments` object with all information about medical records. It is required that this event handled asynchronously by EMR. SFS expects EMR to make a call to the `treatments` API method and transfer back `treatments` objects with the all original information, plus including the results of processing.

* Url: webhook provided by EMR
* Method: POST
* Asynchronous 
* Transfers [`treatments`](#the-treatments-object) object included in the `event` object
* Expected response with 200 Http code in case of success.
* In case of the error, EMR should return 400 Http code and optionally the [`Error`](#the-error-object) object

<aside class="warning">
It is very important that you call the [`/treatments`](#send-medical-records-processing-results) API method, after you finish processing  the medical records that SFS sends to EMR with this event. Make sure to preserve and send back the `id` field received with the `treatments` object.
</aside>

## Send medical records processing results

> Example Request:

```http
POST /treatments HTTP/1.1
User-Agent: MyClient/1.0.0
Content-Type: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
```
```json
{
	"id": "sfs_operation_id",
	"treatments": [
		{
			"objectType": "treatment",
			"name": "Cefazolin 100",
			"inventoryId": "external-inventory-id",
			"hospitalizationId": "external-hosp-id1",
			"treatmentGuid": "sfs-treatment-guid1",
			"time":"2013-03-28T14:23:56.000+00:00",
			"status": "changed",
			"qty": 3.5,
			"volume": 3.5,
			"value": "IZ",
			"units": "ml",
			"doctorName": "name-of-the-doctor",
			"asyncOperationStatus": -1,
			"asyncOperationMessage": "Some error message"
		},
		{
			"objectType": "treatment",
			"name": "Temperature",
			"inventoryId": null,
			"hospitalizationId": "external-hosp-id2",
			"treatmentGuid": "sfs-treatment-guid2",
			"time":"2013-03-28T14:23:56.000+00:00",
			"status": "added",
			"qty": 1,
			"volume": null,
			"value": "36.6",
			"units": null,
			"doctorName": "name-of-the-doctor",
			"asyncOperationStatus": 0
		}
	]
}
```

With this request Smart Flow Sheet will receive the results of medical records processing by EMR. SFS then will update medical records with the appropriate status transferred with `asyncOperationStatus` field. Call this method after asynchronous processing of the `treatments.records_entered` event finished.

* Url: /treatments
* Method: POST
* Synchronous
* Accepts [`treatments`](#the-treatments-object) object
* Returns HTTP status 200 on success
* In case of error returns the [`Error`](#the-error-object) object

<aside class="warning">
It is very important that you call this API method, after you finish processing  the medical records that SFS sends to EMR with the `treatments.records_entered` event. Make sure to preserve and send back the `id` field received with the `treatments` object.
</aside>
