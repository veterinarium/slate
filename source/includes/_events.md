# Events

Smart Flow Sheet will send events to EMR on different occasions:

1. [Inventory import finished](#recieving-the-status-of-import-operation)
2. [When single medical record has been entered or removed](#retreive-single-medical-record)
3. [When mutliple medical records have been entered or removed](#retreive-multiple-medical-records)

In the future there can be other events added to this, e.g. sending flowsheet report or medical records report on patient discharge, etc…

Events will be sent only for those hospitalizations that have been created from EMR. Events on hospitalizations created directly from SFS won`t be sent to EMR webhook. 

Sending events require webhook to be registered in SFS. Requirements to webhook:

* HTTP POST method
* Accepts json object of the [`Event`](the-event-object) type
* Returns HTTP code 200 in case of success
* Returns HTTP code 400 in case of error and optionally sends the [`Error`](#the-event-object) object
* Some events (that post 1 item) require synchronous processing
* Some events require asynchronous processing, with further notifications of the results

## The event object

> Example of the event object:

```json
{
    "clinicApiKey": "clinic-api-key",
    "eventType": "treatments.records_entered",
    "object": {
	    
	}
}
```

### Attributes

Parameter | Type | Description
---------- | ------- | -------
**clinicApiKey** | String | Clinic api key to associate clinic on EMR`s side
**eventType** | String | The [`type`](#types-of-events) of the event
**object** | Object | Object of one of the following type: `treatment`, `treatments`, `inventoryitems`, `hospitalization`. This list can expand as soon as new objects will be sent with the API.


## Types of events

Smart Flow Sheet will send several events on different occasions. Below is short description of the events.

Event | Description
---------- | -------
**hospitalization.discharged** | Sent after a patient has been discharged from the Smart Flow Sheet whiteboard
**hospitalizations.discharged** | Sent after multiple patients have been discharged from the Smart Flow Sheet whiteboard
**inventoryitems.imported** | Sent after importing of emr inventory items finished
**treatment.record_entered** | Sent from SFS when one medical record has been entered/removed
**treatments.records_entered** | Sent from SFS when multiple medical records have been entered/removed
