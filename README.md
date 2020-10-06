#  Usage

To create an account and add credits, please send a mail to contact@lapreuvenumerique.com

# Description of the web services

Requests are made on the following url:  https://api.lapreuvenumerique.com/[method_to_call]

Each service is described as following :

Method : GET / POST / PATCH / PUT / DELETE -> THe http method to use to call the service
Url -> The url of the web service
Authorization: Secret key OR API key -> The token to send via the authorization header (more info in the <b>Headers </b> section)

## Headers

Two headers must be sent to access each service, except the `Test` service

The header x-customeruid must be sent for each service with your personal CustomerUID value in it
The authorization header must also be sent with either the api key or the secret key (see details in each service) as a Basic token

```javascript
headers.append("X-CustomerUID", customerUID})
headers.append('Authorization', 'Basic ' + apiKey OR secretKey)
```

## Response

If the service succeded, it sends a response with a status < 400 (usually 200), and returned the JSON object which is shown after the description of the service.
If an error happened, a response with a defined status will be sent. Here are the common errors most of the services can send :

| Status | Name                  | Details                                                      |
| ------ | --------------------- | ------------------------------------------------------------ |
| 401    | UNAUTHORIZED          | The headers you provided were imcomplete or your credentials are incorrect |
| 403    | FORBIDDEN             | You tried to access a service that requires the secret key with an API key (only services with secret authorization header) |
| 422    | UNPROCESSABLE_ENTITY  | Some required parameters are missing, or the provided ones don't match the expected type |
| 500    | INTERNAL_SERVER_ERROR | An error happened server side, this is not a normal behaviour and you should contact the administrator |

The specifics errors of each service are explain after their description

# Web services

## Test - Test if the server is online

Method: GET
Url : /1.0/test
Authorization: None
Description : Used to test if the api is online

Response: 

```json
{
    "message": "La preuve numérique",
    "version": "1.0"
}
```

---

# Proofs related web services



## ProofDeposit - Save file information on the server

Method: PUT
Url: /1.0/proofdeposit
Authorization: API key
Description: Stores a proof on the client DB and stores it if demanded

The file has to be passed in a FormData 

| Parameter    | Acceptable Values              | Default value | Details                                                      |
| ------------ | ------------------------------ | ------------- | ------------------------------------------------------------ |
| file         | `File`<code>Required</code>    |               | The proof to store                                           |
| fileName     | `string`                       | null          | If provided, replace the actual name of the proof uploaded with this one |
| folderName   | `string`                       | null          | The folder the proof is related with                         |
| reference    | `string`                       | null          | The reference of the proof                                   |
| uids         | `string`                       | null          | Uid related to the proof                                     |
| sourceApp    | `string`<code>Required</code>  |               | The app from which the demand comes                          |
| email        | `string`<code>Required</code>  |               | The mail to which send a receipt of the transaction          |
| noMail       | `boolean`                      | false         | Wether or not a confirmation email is sent                   |
| copy         | `string[]`                     | null          | Other email to which send a receipt of the transaction       |
| identity     | `string`<code>Required</code>  |               | The person who sent the proof                                |
| batchNumber  | `string`                       | null          | The batch number the proof is related with                   |
| topic        | `string`                       | null          | The topic related to the file, topics are created by the admin |
| rgpdDuration | `integer`                      | 1             | The number of years the file has to be saved, used if keepFile is `true` |
| visa         | `string`                       | null          |                                                              |
| keywords     | `string[]`                     | null          | The keywords related to the proof, usefull to query it afterwards |
| noDuplicate  | `boolean`<code>Required</code> |               | Whether or not the file is stored if it's already on the server |
| keepFile     | `boolean`<code>Required</code> |               | Wether or not the file is saved on the server                |
| locale       | `string`                       | en            | Locale for the returned message                              |

Response:

```json
{
    "message": "Proof successfully uploaded",
    "uid": 'xxx'
}
```

| Status | Name                     | Details                                                      |
| ------ | ------------------------ | ------------------------------------------------------------ |
| 403    | FORBIDDEN                | You don't have enough credit to store the proof              |
| 409    | CONFLICT                 | The proof is already stored and you sent `noDuplicate` to true |
| 413    | REQUEST_ENTITY_TOO_LARGE | The size of the proof you uploaded was superior to the max size authorized for your account |

Example: 

```javascript
let formData = new FormData();

      formData.append("file", new File('c:/temp/file.txt'))
      formData.append("folderName", "123r")
      formData.append("reference", "ref")
      formData.append("uids", "456...")
      formData.append("sourceApp", "electron")
      formData.append("copy", "test@test.com")
      formData.append("identity", "John Doe")
      formData.append("batchNumber", "123ab")
      formData.append("topic", "Default")
      formData.append("rgpdDuration", "2")
      formData.append("visa", "124")
      formData.append("keywords", ["test", "code"])
      
      try {
        const response = await fetch("https://lapreuvenumerique/1.0/proofdeposit", {
          method: "PUT",
          body: formData
        });
        console.log(response)
      } catch (err) {
        console.log(err)
      }
```



## BeginMassDeposit - Start transaction

Method: POST
Url: /1.0/beginmassdeposit
Authorization: API key
Description: Opens a new transaction to upload multiple proofs with the same information. <b>The transaction stays open for 5 minutes maximum and deletes all the proofs of the transaction if not ended before via the `EndMassDeposit` service</b>

| Parameter    | Acceptable Values              | Default value | Details                                                      |
| ------------ | ------------------------------ | ------------- | ------------------------------------------------------------ |
| folderName   | `string`                       | null          | The folder the proofs are related with                       |
| reference    | `string`                       | null          | The reference of the proofs                                  |
| sourceApp    | `string`<code>Required</code>  |               | The app from which the demand comes                          |
| email        | `string`<code>Required</code>  |               | The mail to which send a receipt of the transaction after it ends |
| copy         | `string[]`                     | null          | Other email to which send a receipt of the transaction       |
| identity     | `string`<code>Required</code>  |               | The person who sent the demands                              |
| batchNumber  | `string` <code>Required</code> |               | The batch number of the transaction to which all the proofs that will be sent belong |
| topic        | `string`                       | null          | The topic of the transaction                                 |
| rgpdDuration | `integer`                      | 2             | The number of years the file has to be saved, used if keepFile is `true` |
| visa         | `string`                       | null          |                                                              |
| keywords     | `string[]`                     | null          | The keywords related to the proofs, usefull to query it afterwards |
| noDuplicate  | `boolean`<code>Required</code> |               | Whether or not the files arestored if they are already on the server |
| keepFile     | `boolean`<code>Required</code> |               | Wether or not the files isaresaved on the server             |
| locale       | `string`                       | en            | Locale for the returned message                              |

Response: 

```json
{
    "message": "Started a new mass deposit transaction",
    "transactionId": "xxx"
}
```

| Status | Name     | Details                                                |
| ------ | -------- | ------------------------------------------------------ |
| 409    | CONFLICT | A mass upload with this batch number is already opened |

<b>If an error occurs during an upload, the transaction will be closed and no proof will be saved</b>

## MassDeposit - Add proofs to the transaction

Method: POST
Url: /1.0/massdeposit
Authorization: API key
Description: Uploads mutliples files with the same information

| Parameters    | Acceptable value              | Details                                              |
| ------------- | ----------------------------- | ---------------------------------------------------- |
| transactionId | `string`<code>Required</code> | The id Returnsed by the beginMassTransaction service |
| file          | `File`<code>Required</code>   | The proof to save                                    |
| locale        | `string`                      | en                                                   |

Response: 

```json
{
    "message": "Proof successfully added to current transaction",
    "total": 2
}
```

| Status | Name                     | Details                                                      |
| ------ | ------------------------ | ------------------------------------------------------------ |
| 403    | FORBIDDEN                | You don't have enough credit to store the proof              |
| 404    | NOT_FOUND                | The transaction id provided does not exist                   |
| 409    | CONFLICT                 | The proof is already stored and you sent `noDuplicate` to true |
| 413    | REQUEST_ENTITY_TOO_LARGE | The size of the proof you uploaded was superior to the max size authorized for your account |



## EndMassDeposit - End transaction

Method: POST
Url: /1.0/endmassdeposit
Authorization: API key
Description: Ends the mass deposit with the provided transaction ID and store all the proofs passed with the previous method

| Parameters    | Acceptable value              | Details                                              |
| ------------- | ----------------------------- | ---------------------------------------------------- |
| transactionId | `string`<code>Required</code> | The id Returnsed by the beginMassTransaction service |
| locale        | `string`                      | en                                                   |

| Status | Name      | Details                                    |
| ------ | --------- | ------------------------------------------ |
| 404    | NOT_FOUND | The transaction id provided does not exist |



## QueryProof - Request proofs

Method: POST
Url: /1.0/queryproof
Authorization: API key
Description: Returns all the proofs corresponding to the requested parameters.

If the proof has an uploaded file attached, in the result, the value *hasFile* has the value *true* 

| Parameter    | Acceptable Values              | Default value | Details                                                      |
| ------------ | ------------------------------ | ------------- | ------------------------------------------------------------ |
| uid          | `string`                       | null          | The uid of the file to query                                 |
| folderName   | `string`                       | null          |                                                              |
| reference    | `string`                       | null          |                                                              |
| uids         | `string`                       | null          |                                                              |
| sourceApp    | `string`                       | null          |                                                              |
| identity     | `string`                       | null          |                                                              |
| batchNumber  | `string`                       | null          |                                                              |
| topic        | `string`                       | null          |                                                              |
| rgpdDuration | `integer`                      | null          |                                                              |
| keywords     | `string[]`                     | null          |                                                              |
| uploadDate   | `Date`                         | null          | The minimum date <b>from</b> which you want to retrieve the proofs |
| deadline     | `Date`                         | null          | The minimum date from which the RGPD of the proofs expired   |
| fileName     | `string`                       | null          |                                                              |
| page         | `integer`<code>Required</code> |               | The page of result to retrieve, one page has 50 proofs       |
| locale       | `string`                       | en            | Locale for the returned message                              |

Response:

```json
{
	"number": 1,
	"files": [
        {
            "uid": 'xxx',
            "fileName": "",
            "path": "",
            "folderName": "",
            "reference": "",
            "uids": "",
            "sourceApp": "",
            "identity": "",
            "batchNumber": "",
            "topic": "",
            "hasFile":true,
            "rgpdDuration": "2",
            "visa": "",
            "keywords": [
                ""
            ],
            "uploadDate": "",
            "fingerprint": ""
        }
}
```



## QueryDeadline

Method: GET
Url: /1.0/deadlines
Authorization: API key
Description: Returns all the deadlines of the proofs which are saved on the server only(The date the RGPD expired)

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
    "number": 1,
    "files": [
        "uid": 'xxx',
        "deadline": ""
    ]
}
```



## ProofExists - Check existence

Method: POST
Url: /1.0/proofexists
Authorization: API key
Description: Returns the proofs id with the provided fingerprint. It can be the fingerprint of the file, or the fingerprint of the receipt. In the ids array, you'll find a list of ids starting with *FI-* if it's a file or *RE-* if it's a receipt

You can use the preformated message, or build your own based on the json response datas.

You'll also get some informatations of the client, like it's name, address, and logo (png image string in base64)

| Parameters  | Acceptable value               | Details                                   |
| ----------- | ------------------------------ | ----------------------------------------- |
| fingerprint | `string` <code>Required</code> | The fingerprint of the file to search for |
| locale      | `string`                       | en                                        |

Response: 

```json
{  
    "message": "xxx"
    "ids": ['xxx']
    "proofs": [
        {
            "uid": "xxx",
            "infile": false,
            "inar": true,
            "folderName": "",
            "identity": "John Doe",
            "reference": "",
            "fileName": "bill.pdf",
            "uploadDate": "2020-09-11T13:11:34.567Z"
        }
    ],
    "client": {
        "name": "",
        "address": "",
        "logo": ""
    }
}
```

| Status | Name      | Details                               |
| ------ | --------- | ------------------------------------- |
| 404    | NOT_FOUND | The fingerprint provided is not saved |



## FingerprintCompare - Test fingerprint

Method: POST
Url: /1.0/fingerprintcompare
Authorization: API key
Description: Checks if the fingerprint provided is equal to the one of the proof with the provided ID

| Parameters  | Acceptable value               | Details                              |
| ----------- | ------------------------------ | ------------------------------------ |
| fingerprint | `string` <code>Required</code> | The fingerprint to test              |
| fileID      | `number` <code>Required</code> | The id of the file to test in the DB |
| locale      | `string`                       | en                                   |

```json
{
    "message": "The provided fingerprint of the file 10 is correct"
}
```

| Status | Name      | Details                                                      |
| ------ | --------- | ------------------------------------------------------------ |
| 404    | NOT_FOUND | No proof found with the provided id                          |
| 409    | CONFLICT  | The fingerprint does not corresponds to the one saved for the file |



## ProofsCount - Count proofs

Method: GET
Url: /1.0/proofscount
Authorization: API key
Description: Returns the number of proof in the DB

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
    "number": 120
}
```



## ProofFileDownload - Download proof

Method: GET
Url: /1.0/prooffiledownload/:uid
Authorization: API key
Description: Downloads the file with the requested ID

| URL parameters | Acceptable Value              | Details                         |
| -------------- | ----------------------------- | ------------------------------- |
| uid            | `Number`<code>Required</code> | The uid of the file to download |
| locale         | `string`                      | en                              |

Response: 

```
The file in blob format
```

| Status | Name      | Details                                                      |
| ------ | --------- | ------------------------------------------------------------ |
| 404    | NOT_FOUND | No file found with the provide id, or the file is not saved on the server |
| 409    | CONFLICT  | The apikey provided in the header is already downloading a file |



## ProofReceiptDownload - Download past receipt

Method: GET
Url: /1.0/proofreceiptdownload/:uid
Authorization: API key
Description: Downloads the receipt of the requested proof

| URL parameters | Acceptable value               | Details              |
| -------------- | ------------------------------ | -------------------- |
| uid            | `integer`<code>Required</code> | The uid of the proof |
| locale         | `string`                       | en                   |

Response:

```
A pdf file in blob format
```

| Status | Name      | Details                                                      |
| ------ | --------- | ------------------------------------------------------------ |
| 404    | NOT_FOUND | No file found with the provide id                            |
| 409    | CONFLICT  | The apikey provided in the header is already downloading a file |



## GetFingerprint - Get fingerprint

Method: POST
Url: /1.0/getfingerprint
Authorization: API key
Description: Returns the fingerprint of the provided file

| Parameter | Acceptable value            | Details                              |
| --------- | --------------------------- | ------------------------------------ |
| file      | `File`<code>Required</code> | The file to get the fingerprint from |
| locale    | `string`                    | en                                   |

```json
{
	"fingerprint": "xxxx"
}
```



## VerifyIntegrity - Verify blockchain integrity

Method: GET
Url: /1.0/verifyintegrity
Authorization: API key
Description: Checks if the client blockchain is correct for each record

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{ 
    message: "The integrity of the blockchain with X record(s) is correct"
}
```

| Status | Name                | Details                                                      |
| ------ | ------------------- | ------------------------------------------------------------ |
| 456    | UNRECOVERABLE_ERROR | Your bnlockchain is compromised, please contact an administrator |



---

# Client related web services

## Login - Test credentials

Method: GET
Url: /1.0/login
Authorization: API key
Description: Checks if the headers are correct with the APIkey as authorization

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
	"status": "SUCCESS"
}
```



## LoginSecret - Test secret credentials

Method: GET
Url: /1.0/loginsecret
Authorization: Secret key
Description: Checks if the headers are correct with the secret key as authorization and send back information

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{ 
            "uid": "xxx", 
            "name": "Name", 
            "creditsNumber": 1000,
            "maxSize": 150000, 
            "email": "email@test.com",
            "creditsEnabled": true | false,
            "creditFingerpint": 1,
            "creditSizeKo": 150,
            "creditFile": 1
}
```



## GetAllInfos - Clients infos

Method: GET
Url: /1.0/client
Authorization: API key
Description: Returns all the public informations about the client

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{ 
            "uid": "xxx", 
            "name": "Name", 
            "creditsNumber": 1000,
            "maxSize": 150000, 
            "email": "email@test.com",
            "creditsEnabled": true | false,
            "creditFingerpint": 1,
            "creditSizeKo": 150,
            "creditFile": 1
}
```



## TransactionLogs - Get transaction logs

Method: GET
Url: /1.0/transactionlogs/:page
Authorization: Secret key
Description: Returns a page with 30 transaction logs of the client

| Url parameters | Acceptable value | Details |
| -------------- | ---------------- | ------- |
| page           | `integer`        |         |
| locale         | `string`         | en      |

Response: 

```json
{
    "number": 2,
    "logs": [
        {
        	"fileId": "xxx",
            "transactionDate": "2020-07-23T07:29:54.806Z",
            "creditCost": 1
        }
        {
        	"fileId": 'xxx',
            "transactionDate": "2020-07-24T06:04:21.188Z",
            "creditCost": 1
        }
    ]
}
```



## AccessLogs - Get access logs

Method: GET
Url: /1.0/accesslogs/:page
Authorization: Secret key
Description: Returns a page with 30 access logs of the client

| URL parameters | Acceptable value | Details                          |
| -------------- | ---------------- | -------------------------------- |
| page           | `integer`        | The page of the logs to retrieve |
| locale         | `string`         | en                               |

```json
{
    "number": 30,
    "logs": [
        {
        	"id": 1,
            "operationDate": "2020-07-8T10:01:15.188Z",
            "apiKey": "xxxx",
            "fileId": null,
            "operation": "QUERY"
        },
        {
            "id": 2,
            "operationDate": "2020-07-23T07:26:12.832Z",
            "apiKey": "xxxx",
            "fileId": 'xxx',
            "operation": "MASS_UPLOAD"
        },
		...
    ]
}
```

The field 'operation' can be one of the following : 

- DOWNLOAD
- QUERY
- UPLOAD
- MASS_UPLOAD
- VERIFY_INTEGRITY

## Credits - Check credits

Method: GET
Url: /1.0/credits
Authorization: API key
Description: Returns the number of credit available for the client

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
    "uid": "",
    "credits": ""
}
```



## CompanyName - Get client name

Method: GET
Url: /1.0/companyname
Authorization: API key
Description: Returns the name of the client

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
    "uid": "",
    "name": ""
}
```



## CompanyDomainName - Get client name and api domain name

Method: GET
Url: /1.0/companydomainname
Authorization: API key
Description: Returns the name of the client COMPANY (DOMAIN)

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
    "uid": "",
    "name": ""
}
```



## GetTopics - Get topics name

Method: GET
Url: /1.0/topics
Authorization: API key
Description: Returns the name of the topics

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
    "number": 1,
    "topics": [
        {
            "name": "Default",
            "index": 1
        }
    ]
}
```



## GetTopicsFull - Get topics name (admin)

Method: GET
Url: /1.0/topicsfull
Authorization: Secret key
Description: Returns the name of the topics

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
    "number": 1,
    "topics": [
        {
            "name": "Default",
            "id": 1
        }
    ]
}
```



## CreateTopic - Add a new topic

Method: PUT
Url: /1.0/topic
Authorization: Secret key
Description: Adds a new topic to the client DB

| Parameter | Acceptable value              | Details                         |
| --------- | ----------------------------- | ------------------------------- |
| name      | `string`<code>Required</code> | The name of the topic to create |
| locale    | `string`                      | en                              |

Response:

```json
{
    "message": "Topic created successfully",
    "id": 2
}
```

| Status | Name     | Details                               |
| ------ | -------- | ------------------------------------- |
| 409    | CONFLICT | A topic with this name already exists |



## DeleteTopic - Delete a topic

Method: DELETE
Url: /1.0/topic/:id
Authorization: Secret key
Description: Delete the topics

| URL parameter | Acceptable value               | Details                       |
| ------------- | ------------------------------ | ----------------------------- |
| id            | `integer`<code>Required</code> | The id of the topic to delete |
| locale        | `string`                       | en                            |

Response : 

```json
{
    "message": "Topic successfully deleted"
}
```

| Status | Name      | Details                             |
| ------ | --------- | ----------------------------------- |
| 404    | NOT_FOUND | No topic found with the provided id |



---

# Apikey web services

## GetKeys - Check keys

Method: GET
Url: /1.0/apikeys
Authorization: Secret key
Description: Returns all the keys created by the client

| Parameter | Acceptable Values | Default value | Details                         |
| --------- | ----------------- | ------------- | ------------------------------- |
| locale    | `string`          | en            | Locale for the returned message |

Response:

```json
{
    "number": X,
    "keys": [
        {
            "id": 1
            "key": "",
            "description: "",
            "active": true
        }
    ]
}
```



## CreateKey - Add key

Method: POST
Url: /1.0/apikey
Authorization: Secret key
Description: Creates a new Apikey with the provided information

| Parameters  | Acceptable Value               | Details                                   |
| ----------- | ------------------------------ | ----------------------------------------- |
| description | `string` <code>Required</code> | By who and for what this key will be used |
| locale      | `string`                       | en                                        |

Response:

```json
{
    "key": "xxxx",
    "id": 1
}
```



## UpdateKey - Change key

Method: PATCH
Url: /1.0/apikey
Authorization: Secret key
Description: Update the information of a key, and can also revoke a key to never use it again

| Parameters    | Acceptable Value                | Details                                                      |
| ------------- | ------------------------------- | ------------------------------------------------------------ |
| active        | `boolean` <code>Required</code> | Whether the key can be used at the time                      |
| description   | `string`                        | The new description of the key                               |
| revoked       | `boolean`                       | Wether or not the key can no longer be used ever             |
| revokedReason | `string`                        | Required if revoked is set to true, explains why the key is revoked |
| locale        | `string`                        | en                                                           |

Response: 

```json
{
    "message": "Your key with id 1 has been updated"
}
```

| Status | Name      | Details                           |
| ------ | --------- | --------------------------------- |
| 404    | NOT_FOUND | No key found with the provided id |

