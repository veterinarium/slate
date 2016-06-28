# Anesthetics

Anesthetic is an abstraction used in Smart Flow Sheet to represent the anesthetic sheet data. The main purpose of this abstraction is to provide the possibility to download anesthetic sheet report after the patient discharge event. 

## The anesthetics object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object returned with the SFS API. Equals to `anesthetics`
**id** | String | **Required**. Identificator of the object. 
**anesthetics** | Array | The array of `anesthetic` objects. See description of the `anesthetic` object [below](#the-anesthetic-object)


## The anesthetic object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | Describes the type of the object transferred with the SFS API. Equals to `anesthetic`
**dateStarted** | Date | *Optional*. Anesthesia start time. Time format: YYYY-MM-DDThh:mm:ss.sssTZD (e.g. 1997-07-16T19:20:30.000+00:00)
**dateEnded** | Date | *Optional*. Anesthesia end time. Time format: YYYY-MM-DDThh:mm:ss.sssTZD (e.g. 1997-07-16T19:20:30.000+00:00)
**reportPath** | String | *Optional*. The path to the anesthetic sheet report file that has been generated during finalization of the anesthetic sheet.
**recordsReportPath** | String | *Optional*. The path to the anesthetic records report file that has been generated during finalization of the anesthetic sheet.
**surgeon** | Medic | *Optional*. The [`medic`](#the-medic-object) object that corresponds to the doctor assigned to the anesthetic sheet.
**anesthetist** | Medic | *Optional*. The [`medic`](#the-medic-object) object that corresponds to the anesthetist assigned to the anesthetic sheet.
**assistant** | Medic | *Optional*. The [`medic`](#the-medic-object) object that corresponds to the assistant assigned to the anesthetic sheet.


## Retreive anesthetic sheet and anesthetic records reports

> Example Request:

```http
GET /hospitalization/{hospitalizationId}/anesthetics HTTP/1.1
User-Agent: MyClient/1.0.0
Content-Type: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
```
```json
{
	"objectType": "anesthetics",
	"id": "sfs_operation_id",
	"anesthetics": [
		{
	        "objectType": "anesthetic",
	        "dateStarted": "2015-11-04T17:23:07.707+00:00",
	        "dateEnded": "2015-11-04T19:17:03.463+00:00",
	        "reportPath": "https://pdf-anesthetic-sheet-report-webfile-path",
            "recordsReportPath": "https://pdf-anesthetic-records-report-webfile-path",
	        "surgeon": {
	            "objectType": "medic",
	            "medicId": "emrIdm4",
	            "name": "Dr. George",
	            "medicType": "doctor"
	        },
	        "anesthetist": {
	            "objectType": "medic",
	            "medicId": "emrIdm3",
	            "name": "Ivan",
	            "medicType": "doctor"
	        },
	        "assistant": {
	            "objectType": "medic",
	            "medicId": "emrIdm4",
	            "name": "Dr. George",
	            "medicType": "doctor"
	        }
		},
		{
	        "objectType": "anesthetic",
	        "dateStarted": "2015-11-04T17:23:07.707+00:00",
	        "dateEnded": null,
	        "reportPath": null,
            "recordsReportPath": null,
	        "surgeon": {
	            "objectType": "medic",
	            "medicId": "emrIdm4",
	            "name": "Dr. George",
	            "medicType": "doctor"
	        },
	        "anesthetist": {
	            "objectType": "medic",
	            "medicId": "emrIdm3",
	            "name": "Ivan",
	            "medicType": "doctor"
	        },
	        "assistant": null
		}
	]
}
```

This method allows to download all anesthetic sheet reports and anesthetic records reports related to some hospitalization from Smart Flow Sheet.

Specify the `hospitalizationId` of the hospitalization object in the EMR. The same `hospitalizationId` that was supplied when hospitalization had been created.

* Url: /hospitalization/{hospitalizationId}/anesthetics
* Method: GET
* Synchronous 
* Returns the [`Anesthetics`](#the-anesthetics-object) object.
* Expected response with 200 Http code in case of success.
* In case of the error, the [`Error`](#the-error-object) object will be returned


