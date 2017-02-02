# Forms

The are three objects used by Smart Flow Sheet to notify EMR when forms are filled in:

* `forms`

* `form`

* `formfield`

All objects are sent with the asynchronous `forms.created` event when one or several forms has been entered.

## The forms object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | *Optional*. Describes the type of the object transferred with the SFS events (e.g. `forms.created`). Should be assigned `forms` value
**id** | String | Identificator of the object. Will be transferred to EMR with the `forms.created` event.
**forms** | Array | The array of `form` objects. See description of the `form` object [below](#the-form-object)


## The form object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | Describes the type of the object transferred with the SFS events (e.g. `forms.created`). Should be assigned `form` value
**formGuid** | String | **Required**. A unique internal identifier of the form item
**hospitalizationId** | String | Hospitalization external id (which was provided with hospitalization creation)
**name** | String | **Required**. A unique identifier of the form type. Can be one of the following: 1. `admission`, 2. `custom`
**title** | String | The title of the form (e.g. "Client Self Check-in")
**fields** | Array | The array of `formfield` objects. See description of the `formfield` object [below](#the-formfield-object)

## The formfield object

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**objectType** | String | Describes the type of the object transferred with the SFS events (e.g. `forms.created`). Should be assigned `formfield` value
**name** | String | **Required**. A unique identifier of the form field
**title** | String | Field label (the one that users sees while filling in the form field)
**contentType** | String | This field describes the type of content of the `value` field (e.g. `text/plain`, `image/jpg`)
**value** | String | The string value of the form field

## Retreive forms

> Example of `forms.created` event JSON:

```json
{
    "clinicApiKey": "clinic-api-key",
    "eventType": "forms.created",
    "object": {
	    "objectType": "forms",
		"id": "sfs-operation-id",
		"forms":[
	        {
	            "objectType": "form",
	            "formGuid": "sfs-form-guid1",
	            "hospitalizationId": "external-hosp-id1",
	        }	
		]
	}
}
```
The `forms.created` event is sent from SFS when one or several forms were created. SFS will send [forms](#the-forms-object) object with all information entered into the forms.

* Url: webhook provided by EMR
* Method: POST
* Asynchronous 
* Transfers [forms](#the-forms-object) object included in the `event` object
* Expected response with 200 Http code in case of success.
* In case of the error, EMR should return 400 Http code and optionally the [`Error`](#the-error-object) object