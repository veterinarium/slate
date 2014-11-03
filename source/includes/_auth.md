# Authentication

```http
GET /inventoryitems HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/json
emrApiKey: "emr-api-key-received-from-sfs"
clinicApiKey: "clinic-api-key-taken-from-account-web-page"
```

You authenticate to the SFS API by providing both EMR API key and clinic API key in all API requests to the server in a header. 

`emrApiKey: emr-api-key-received-from-sfs`

`clinicApiKey: clinic-api-key-taken-from-account-web-page`

All API requests must be made over HTTPS. Calls made over plain HTTP will fail. You must authenticate for all requests.

<aside class="warning">
You must replace `emr-api-key-received-from-sfs` with your personal **emr API key** and `clinic-api-key-taken-from-account-web-page` with the **clinic API key** taken from `Account Info` web-page.
</aside>
