# Basic Use-Cases

> Sample Project:

```shell
https://github.com/veterinarium/public-sfs
```

This section describes the basic integration workflows.  

Please download a sample project from GitHub that contains several examples of the SFS API usage.

## Creating Patients

Creating patients from EMR into SFS has two obvious benefits:

1. _Saving the clinic's staff time._ Almost always the information required to create a patient in Smart Flow Sheet has been already entered into EMR user interface upon patient addmission to the hospital. Using `/hospitalization` API method it is possible to transfer that information into SFS and we will automatically create a new patient record for the user.

2. _Automatically collecting charges for the patient._ Smart Flow Sheet will not send medical records for the patients created from SFS user interface. To receive medical records for the patient you will need to create a patient in SFS explicitly using `/hospitalization` API method. 

Usually, what we recommend is that you place 'Smart Flow Sheet' button on your user interface. Clinic staff could then press it when they are ready to create a patient's flowsheet (usually when a doctor decides to leave a patient in the hospital). Before creating a patient in Smart Flow Sheet, you may want to allow a user to select a [treatment template](#retreive-active-treatment-templates) and [department](#retreive-existing-departments) for the patient. After the required treatment template and department has been selected, you should use [this](#create-a-patient) API to create patient record in Smart Flow Sheet.

After the patient has been submitted to Smart Flow Sheet, you may want to query the patient's hospitalization status using [this](#get-hospitalization) API. With its help, you can get some additional information, e.g. patient image file to update your user interface. Or you may track the `status` of the patient, and as soon as the patient has been discharged, you could download and attach to your medical records the set of files related to the patient, e.g. the [flowsheet report](#download-the-flowsheet-report).

## Discharging Patients

At the end of hospitalization, the clinic staff will usually discharge the patient. During this step, Smart Flow Sheet generates a set of documents (in pdf format) that your practice management software might want to attach to the patient record. Below is the list of such reports with references of how to download them using the API:

* [Flowsheet report](#download-the-flowsheet-report)
* [Medical records report](#download-the-medical-records-report)
* [Billing report](#download-the-billing-report)
* [Notes report](#download-the-notes-report)
* If the anesthetic sheet was created, completed, and finalized, for the patient, then the associated [Anesthetic Sheet and Anesthetic Records reports](#retreive-anesthetic-sheet-and-anesthetic-records-reports) can be downloaded
* [Form report](/#download-the-form-report) for each form created for the patient. To download all forms upon patient discharge you will need to [get all patients forms](#get-patient-s-forms) first, and then call the [get form values](#get-form-values) API and download the document from the `Form.reportFilePath`

There are two options for discharging the patient in Smart Flow Sheet:

1. _The user initiates discharge of the patient from Smart Flow Sheet user interface._ In this case, Smart Flow Sheet will prepare all pdf reports and send the `hospitalization.discharged` event that your EMR may consume, download the appropriate pdf files from Smart Flow Sheet and attach them to the patient records.

2. _The user initiates discharge operation from the EMR side._ In this case, the EMR should call [discharge](#discharge-hospitalization) API to initiate discharging of the patient in Smart Flow Sheet. After the successful call to this API, the response will include the [hospitalization](#the-hospitalization-object) object with the `status` field equal to `discharged`, but the `reportPath` field will be empty. At this point, all patient reports are not generated yet - Smart Flow Sheet only initiated the document generation process. To download the files, you have a couple of implementation strategies: 
 * wait for the `hospitalization.discharged` event and then start downloading the documents
 * wait at least 15 seconds and then call the [get hospitalization]()  API. If the `reportPath` is not empty, then you may start downloading the reports
 * add some sort of "Download patient documents" button that will call the [get hospitalization](#get-hospitalization)  API, and if the `reportPath` is not empty, then the download may start. Otherwise, propose to repeat the operation later because the documents are not ready yet

<aside class="warning">
Generation of the patient reports may take several minutes for the hospitalizations with a significant number of days of treatment (flowsheets). Subscribing to the `hospitalization.discharged` event is the best way to be notified about operation completion. 
</aside>

## Working with the Forms

Smart Flow provides a feature called Customised Hospitalization Forms. These give the user the ability to digitalize the majority of your paper forms, such as: 

* Surgical consent
* Euthanasia consent
* Admission
* and many more...

To make use of these forms and understand how this feature works, please read [this article](https://forum.smartflowsheet.com/solution/articles/13000031853-smart-flow-customizable-hospital-forms-). 
There are two types of forms in Smart Flow that can be created as follows:

1. those that could be used to create a patient in Smart Flow (e.g. Admission form). These forms have `Show on Main Screen` setting set to YES on `Settings/Forms` web page
2. those that do not provide such an option, but rather can be added to the existing patient from the Smart Flow user interface

**Creating Patients from Forms**

By default Smart Flow Sheet provides a couple of forms that allow you to create patients from the “Smart Flow” iPad app. This means clients have the ability from the iPad to fill out their own information, take a photo of their pet, enter a reason for their visit and update and/or enter the name of their regular clinic from the waiting room. Upon completion of the form, the patient is created in Smart Flow and appears on the whiteboard.

As soon as the patient is created from the form, Smart Flow Sheet will notify the EMR by sending `hospitalizations.created` and `forms.created` events. You might want to consume and handle these events to:

* Automatically register both the client and patient into the EMR.

* Attach to Smart Flow Sheet hospitalization to receive treatment events as well as the patient`s medical records and pdf reports. 

There are three steps that should be followed to get client’s data entered on the form:

1. The EMR should [consume](#get-notified-about-new-hospitalizations) the `hospitalizations.created` event that is sent from Smart Flow Sheet as soon as a patient is added from the form. The `hospitalization` object will be provided with the event. When the event is received both the client and patient records should be created in the EMR.

2. In a second step, you should [attach](#attach-to-existing-hospitalization) your internal records to SFS hospitalization. This is required to receive all other types of events related to this patient, as well as to be able to use any [hospitalization API](#hospitalizations).

3. Finally, Smart Flow Sheet will send the `forms.created` [event](#retreive-forms-with-events) that you should consume to get the rest of the data entered by the client during the check-in process.

**Editing Forms Notifications**

When the user edits (changes the content of the form or its status to `finalized`) or deletes a form then Smart Flow Sheet fires the `forms.updated` event. 
When the user finalizes the form then Smart Flow Sheet generates a pdf report for this form.

**Parsing Custom Forms and Properties**

Smart Flow Sheet allows users to create a custom set of forms and properties from the `Settings/Forms` web page. If you want to parse the data from these custom forms then you should utilize `Internal Name` attributes when creating forms and properties.

<img src="images/formsetting.png"> 

The value of `Internal Name` attribute ('checkinform' on a picture above) of the form will be transferred with the `name` property of the [form object](#the-form-object).

<img src="images/propertysetting.png"> 

The value of `Internal Name` attribute of the property ('pet_name' on a picture above) will be transferred with the `name` property of the [formfield object](#the-formfield-object).

## Inventory Import

The most common reason to integrate with the SFS services is to be able to receive medical records from SFS upon treatment execution. This information can be used to automatically collect charges for the inventory items stored in EMR database. 

In order to accomplish this goal the SFS API provides a possibility to import inventory items from the EMR system into SFS. We will store external identifiers for the inventory items during inventory import, and will provide these identifiers back to EMR with the medical records information entered during treatment execution. In this way EMR will be able to associate entered medical record with the correct inventory item, and create a charge.

To import inventory you may use next API methods: 

* `/invetoryitem` - to import single inventory item;

* `/invetoryitems` - to import multiple inventory items; 

The later API method works asynchronously and sends `inventoryitems.imported` event to the url specified with the webhook. 

After the import is finished, the user can start using the inventory immediately by adding imported items to the patients` flowsheets. For the inventory items being added for the first time, Smart Flow Sheet will display a popup window. In this window, the user can adjust the display name of the item, and which SFS section (monitoring, activity, fluid, medication, or procedure) it should be attached to.

<img src="images/mappopup.png"> 

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

## Billing for a Surgery

When the patient is admitted for surgery, Smart Flow Sheet provides access to the electronic anesthetic sheet feature.  The anesthetic sheet allows medical staff to map the patient’s vitals and calculate pre-op and emergency drugs. After the surgery is over, the clinic staff finalizes the anesthetic sheet to:

* Generate anesthetic sheet protocols.
* Calculate an invoice for the surgery.

To make it easier for your EMR to calculate billing for the surgery, Smart Flow Sheet sends the `anesthetics.finalized` [event](#finalize-anesthetic-event) with the included [anesthetic](#the-anesthetic-object) object as soon as the user presses the 'Finalize Anesthetiс Sheet' button on the iPad. 