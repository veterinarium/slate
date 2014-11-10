# Basic Use-Cases

> Sample Project:

```shell
https://github.com/veterinarium/public-sfs
```

This section describes the basic integration workflows.  

Please download a sample project from GitHub that contains several examples of the SFS API usage.

## Inventory Import

The most common reason to integrate with the SFS services is to be able to receive medical records from SFS upon treatment execution. This information can be used to automatically collect charges for the inventory items stored in EMR database. 

In order to accomplish this goal the SFS API provides a possibility to import inventory items from the EMR system into SFS. We will store external identifiers for the inventory items during inventory import, and will provide these identifiers back to EMR with the medical records information entered during treatment execution. In this way EMR will be able to associate entered medical record with the correct inventory item, and create a charge.

To import inventory you may use next API methods: 

* `/invetoryitem` - to import single inventory item;

* `/invetoryitems` - to import multiple inventory items; 

The later API method works asynchronously and sends `inventoryitems.imported` event to the url specified with the webhook. 

After import is finished, user should visit the `Settings` page on our web-site and assign SFS-specific category (monitoring, activity, fluid, medication or procedure) to every inventory item received. Then doctors can add these items to the flowsheets.

## Creating Patients

Creating patients from EMR into SFS has two obvious benefits:

1. Saving clinic's staff time. (Almost always the information required to create a patient in Smart Flow Sheet has been already entered into EMR user interface upon patient addmission to the hospital. Using `/hospitalization` API method it is possible to transfer that information into SFS and we will automatically create a new patient record for the user.)

2. Automatically collecting charges for the patient. Smart Flow Sheet will not send medical records for the patients created from SFS user interface. To receive medical records for the patient you will need to create a patient in SFS explicitly using `/hospitalization` API method. 

## Receiving Medical Records

Receiving medical records information from Smart Flow Sheet provides an opportunity to automatically collect charges for the inventory items stored in your database. Please make sure to [import inventory](#inventory-import) into SFS before that.

Medical record information is sent to EMR with the `treatment.record_entered` event (or `treatments.records_entered` event when multiple records has been entered).

Here is an example:

> Example `inventoryitem` object imported with `/inventoryitem` API:

```json
{
    "id": "external-inventory-id",
    "name": "Cefazolin 100",
    "concentration": 100,
    "concentrationUnits": "mg",
    "concentrationVolume": "ml"
}
```

> Example `treatment.record_entered` event object sent to webhook:

```json
{
	"clinicApiKey": "clinic-api-key",
    "eventType": "treatment.record_entered",
    "object": {
      	"objectType": "treatment",
		"name": "Cefazolin 100",
		"inventoryId": "external-inventory-id",
		"hospitalizationId": "external-hospitalization-id",
		"treatmentGuid": "internal-sfs-treatment-guid",
		"time":"2013-03-28T14:23:56.000+00:00",
		"status": "added",
		"qty": 2.25,
		"volume": 2.25,
		"value": "IZ",
		"units": "ml",
		"doctorName": "Dr. Ivan Zak"
	}
}
```

1. EMR imports `Cefazolin 100 mg/ml` inventory item with the `/invetoryitem` API method.

2. Then doctor creates a patient in the EMR, the patient record is transferred into SFS with the `/hospitalization` API method and new patient automatically appears on a Smart Flow Sheet whiteboard.

3. After the patient is created in Smart Flow Sheet, doctor adds `Cefazolin 100 mg/ml` to the flowsheet, and sets the dosage to `25 mg/kg` (see image below).
<img src="images/medcalc.png"> 

4. When `Cefazolin 100 mg/ml` is given to the patient, Smart Flow Sheet sends medical record information with the `treatment.record_entered` event to the EMR webhook, and transfer the volume of the medication dispensed. In this case it will be `2.25 ml`. 

You can find the example JSONs in `Details` section in the right part of this page. As you can notice the object transferred with the `treatment.record_entered` event contains the `inventoryId` field that is equal to the `id` field of the inventory item imported into SFS with the `/inventoryitem` API method. In this way EMR is able to associate entered medical record with the correct inventory item, and create a charge.