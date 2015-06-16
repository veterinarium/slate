# Hospitalizations

Hospitalization is an abstraction used in Smart Flow Sheet to represent the patient record. As soon as patient is admitted in the hospital the Hospitalization object is created. In turn, the hospitalization object nests patient and client information. This information can be transferred and retrieved from SFS using the patient object and the client object.   

The are the `hospitalizations` and `hospitalization` objects used to create hosptitalizations in SFS and receiving events from the Smart Flow Sheet services.

## The hospitalizations object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events. Should be assigned `hospitalizations` value
**id** | String | *Optional*. Identificator of the object. Will be transferred to EMR with the SFS events 
**hospitalizations** | Array | The array of [`hospitalization`](#the-hospitalization-object) objects. 


This object will be sent with the `hospitalizations.discharged` event in case when multiple patients were [discharged](#discharge-hospitalization-event) from the Smart Flow Sheet whiteboard.

## The hospitalization object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events. Should be assigned `hospitalization` value
**hospitalizationId** | String | **Required**. EMR internal ID of the hospitalization
**departmentId** | Integer | *Optional*. Id of the department patient should be created at. If not specified hospitalization will be created in SFS default department
**hospitalizationGuid** | String | *Optional*. A unique internal identifier of the hospitalization. This field will be transferred with the SFS events.
**dateCreated** | Date | **Required**. Specifies the date and time of the patient arrival in the hospital. Time format: YYYY-MM-DDThh:mm:ss.sssTZD (e.g. 1997-07-16T19:20:30.000+00:00)
**treatmentTemplateName** | String | *Optional* The name of the treatment template to be used for created hospitalization. If not specified then the "Default" template will be used.
**temperatureUnits** | String | *Optional*. Units for the temperature. Can be `F` or `C`. If not specified then clinic’s default temperature units will be used
**weightUnits** | String | *Optional*. Units for the weight. Can be `kg` or `lbs`. If not specified then clinic’s default weight units will be used
**weight** | Double | **Required**. Weight of the patient. Smart Flow Sheet requires weight to be specified for every patient from the moment hospitalization is created
**estimatedDaysOfStay** | Integer | *Optional*. This value will be used to create requested number of days of hospitalization. If this value is not specified then by default 2 days will be created
**fileNumber** | String | *Optional*. The value of emr file number, that will be shown on a flowsheet
**caution** | Boolean | *Optional*. Whether to show `caution stripe` on a flowsheet or not
**dnr** | Boolean | *Optional*. Whether to show `dnr stripe` on a flowsheet or not
**doctorName** | String | *Optional*. The name of the doctor on duty
**medicId** | String | *Optional*. Alternatively to specifying the `doctorName` field, you can provide the id of the [`medic`](#the-medic-object) object that corresponds to the doctor on duty, and has been registered with the [`appropriate API`](#create-or-update-single-medic) call
**diseases** | Array | *Optional*. Array of strings. A collection of diseases
**cageNumber** | String | *Optional*. The cage number. This value will be shown on the whiteboard near patient name.
**color** | String | *Optional*. The RGB hex color code (eg. #439FE0). This value is used to color patient`s info panel on the whiteboard and flowsheets.
**reportPath** | String | *Optional*. The path to the flowsheet report file that has been generated during patient discharge
**patient** | Patient | *Required when creating new hospitalization. Optional if used to update existing hospitalization*. The [`patient`](#the-patient-object) object

## The patient object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events. Should be assigned `patient` value
**patientId** | String | **Optional**. EMR internal ID of the patient
**name** | String | **Required**. The name of the patient
**birthday** | Date | *Optional*. Patient`s birthday. Time format: YYYY-MM-DDThh:mm:ss.sssTZD (e.g. 1997-07-16T19:20:30.000+00:00)
**sex** | String | **Required**. Patient's sex type. There is a set of standard predefined strings that specify the sex type of a patient: `M`, `F`, `MN`, `FS`
**species** | String | *Optional*. Patient's species
**color** | String | *Optional*. Patient's color
**breed** | String | *Optional*. Patient's breed
**criticalNotes** | String | *Optional*. The value of the critical notes that will be shown on a flowsheet.
**customField** | String | *Optional*. The value of the custom field that will be shown on a flowsheet.
**owner** | Client | *Required when creating new hospitalization. Optional if used to update existing hospitalization*. The [`client`](#the-client-object) object

## The client object


### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events. Should be assigned `client` value
**ownerId** | String | **Optional**. EMR internal ID of the client
**nameLast** | String | **Optional**. Pet owner's last name
**nameFirst** | String | *Optional*. Pet owner's first name
**homePhone** | String | *Optional*. Pet owner's home phone
**workPhone** | String | *Optional*. Pet owner's work phone

## Create a patient

> Example Request:

```http
POST /hospitalization HTTP/1.1
User-Agent: MyClient/1.0.0
Content-Type: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
```
```json
{
	"objectType": "hospitalization",
	"hospitalizationId": "emr-hospitalization-id",
	"dateCreated": "2014-11-03T17:54:55.221+00:00",
	"temperatureUnits": "C",
	"weightUnits": "kg",
	"weight": 5.8,
	"estimatedDaysOfStay": 1,
	"fileNumber": "# 123",
	"dnr": true,
	"caution": false,
	"color": "#439FE0",
	"doctorName": "Dr. Ivan",
	"medicId": "dr-ivan-emr-id",
	"diseases": [
		"high temperature",
		"vomiting"
	],
	"patient": {
		"objectType": "patient",
		"patientId": "emr-patient-id",
		"name": "Jordi Alba",
		"species": "Canin",
		"owner": {
			"objectType": "client",
			"ownerId": "emr-client-id",
			"nameLast": "Dow",
			"nameFirst": "Jack",
			"workPhone": "555-55-55"
		},
		"color": "Brown",
		"sex": "M",
		"criticalNotes": "Cefazolin allergy",
		"customField": "some notes"
	}
}
```

* Url: /hospitalization
* Method: POST
* Synchronous
* Accepts [`hospitalization`](#the-hospitalization-object) object
* Returns HTTP status 201 in case the new hospitalization has been created
* In case of error returns the [`Error`](#the-error-object) object

<aside class="warning">
The Smart Slow Sheet API does not allow to create several hospitalizations with the same `hospitalizationId`. However, it may happen that the user needs to re-submit the hospitalization that has been previously created (and then deleted or discharged from Smart Flow Sheet user interface). In this case, the second call of this API will return the error with the message `Hospitalization already exists`.  If this happens, we advise to show the user the error message and prompt them if they would like to re-submit the patient. If user selects the option to re-submit the patient, then EMR should:

1. Make a call to [`delete hospitalization`](#delete-hospitalization) API first;
2. Call [create a patient](#create-a-patient) API again to re-submit the patient information.
</aside>

## Delete hospitalization

This method deletes hospitalization by id. Specify the `hospitalizationId` of the hospitalization object in the EMR. The same `hospitalizationId` that was supplied when hospitalization had been created. 
This method also "unmaps" the `hospitalizationId` from the internal hospitalization record - this allows to submit the patient to Smart Flow Sheet again afterward.

* Url: /hospitalization/{hospitalizationId}
* Method: DELETE
* Synchronous
* If hospitalization cannot be found in SFS, the [`Error`](#the-error-object) object will be returned with HTTP 404 status code

## Discharge hospitalization event

> Example of `hospitalizations.discharged` event JSON:

```json
{
    "clinicApiKey": "clinic-api-key",
    "eventType": "hospitalizations.discharged",
    "object": {
	    "objectType": "hospitalizations",
		"id": "sfs-operation-id",
		"hospitalizations": [
			{
				"objectType": "hospitalization",
				"hospitalizationId": "emr-hospitalization-id",
				"hospitalizationGuid": "sfs-hospitalization-guid",
				"dateCreated": "2014-11-05T10:20:19.000+00:00",
				"fileNumber": "#File number",
				"reportPath": "https://pdf-report-webfile-path"
			}
		]
	}
}
```

As soon as one or several patients have been discharged from the Smart Flow Sheet whiteboard, SFS will notify EMR by sending one of the following events:

1. The `hospitalization.discharged` event in case if single patient was discharged. In this case the [hospitalization](#the-hospitalization-object) object will be transferred with the event.

2. The `hospitalizations.discharged` event in case if multiple patients were discharged. In this case the [hospitalizations](#the-hospitalizations-object) object will be transferred with the event.

Every `hospitalization` object transferred with these events will contain the path to the flowsheet report pdf file in the `reportPath` field. If for some reason the value of this field is empty then you may explicitly [download the flowsheet report](#download-the-flowsheet-report).

<aside class="notice">
There is no API that allow to discharge a patient. Any patient can be discharged only using Smart Flow Sheet user interface.
</aside>

## Download the Flowsheet report

> Example Request:

```http
POST /hospitalization/emr-hospitalization-id/flowsheetreport HTTP/1.1
User-Agent: MyClient/1.0.0
Content-Type: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
timezoneName: Europe/Helsinki
```

This method allows to download the flowsheet report from Smart Flow Sheet.

All the dates in the downloaded report will be represented in the time zone that you **must** explicitly specify in the `timezoneName` header of the HTTP request (e.g. `timezoneName: Europe/Helsinki`). Please visit this [web-page](http://en.wikipedia.org/wiki/List_of_tz_database_time_zones) for the complete list of the time zone names. 

Specify the `hospitalizationId` of the hospitalization object in the EMR. The same `hospitalizationId` that was supplied when hospitalization had been created.

* Url: /hospitalization/{hospitalizationId}/flowsheetreport
* Method: GET
* Synchronous
* Returns the pdf report with the output stream. The Content-Type header will contain the `application/pdf` value
* In case of error returns the [`Error`](#the-error-object) object

## Download the Medical Records report

> Example Request:

```http
POST /hospitalization/emr-hospitalization-id/medicalrecordsreport HTTP/1.1
User-Agent: MyClient/1.0.0
Content-Type: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
timezoneName: Europe/Helsinki
```

This method allows to download the medical record report from Smart Flow Sheet. Use this method when receiving `hospitalizations.discharged` event, as at this point the report should contain all medical records of the specified hospitalization. 

All the dates in the downloaded report will be represented in the time zone that you **must** explicitly specify in the `timezoneName` header of the HTTP request (e.g. `timezoneName: Europe/Helsinki`). Please visit this [web-page](http://en.wikipedia.org/wiki/List_of_tz_database_time_zones) for the complete list of the time zone names. 

Specify the `hospitalizationId` of the hospitalization object in the EMR. The same `hospitalizationId` that was supplied when hospitalization had been created.

* Url: /hospitalization/{hospitalizationId}/medicalrecordsreport
* Method: GET
* Synchronous
* Returns the pdf report with the output stream. The Content-Type header will contain the `application/pdf` value
* In case of error returns the [`Error`](#the-error-object) object

## Download the Inventory report

> Example Request:

```http
POST /hospitalization/emr-hospitalization-id/inventoryreport HTTP/1.1
User-Agent: MyClient/1.0.0
Content-Type: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
timezoneName: America/Chicago
```

This method allows to download the inventory report from Smart Flow Sheet. Use this method when receiving `hospitalizations.discharged` event, as at this point the report should contain the complete inventory usage information of the specified hospitalization. 

All the dates in the downloaded report will be represented in the time zone that you **must** explicitly specify in the `timezoneName` header of the HTTP request (e.g. `timezoneName: America/Chicago`). Please visit this [web-page](http://en.wikipedia.org/wiki/List_of_tz_database_time_zones) for the complete list of the time zone names. 

Specify the `hospitalizationId` of the hospitalization object in the EMR. The same `hospitalizationId` that was supplied when hospitalization had been created.

* Url: /hospitalization/{hospitalizationId}/inventoryreport
* Method: GET
* Synchronous
* Returns the pdf report with the output stream. The Content-Type header will contain the `application/pdf` value
* In case of error returns the [`Error`](#the-error-object) object

