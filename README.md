# Two-Factor Authentication (2FA) Chainlink External Adapter (Javascript)

This adapter is an extra layer of security where in addition to a password, the user must confirm a code from/click a button on a trusted device only they have access to.

## Input Params

The structure for the JSON input is as follows. In this example jobSpec is 278c97ffadb54a5bbb93cfec5f7b5503. Actions may be any of the following:

* customerid 
* hashedpin

#### to calculate hashedpin execute the follow

```bash
python3
Python 3.8.5 (default, Jul 28 2020, 12:59:40) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import pyotp
>>> import datetime
>>> import sha3
>>> totp = pyotp.TOTP('VPPRAX5ZS3EAT3ID')
>>> int(sha3.keccak_256(totp.at(datetime.datetime.now(),0).encode('utf-8')).hexdigest()[:12],16)
217506908309239
```
hashedpin is: 217506908309239
```bash
{
  "id": "278c97ffadb54a5bbb93cfec5f7b5503", 
  "data": {
      "customerid": "alice", 
      "hashedpin": "217506908309239"}
}
```
## Output

```bash
{
    "jobRunID":"278c97ffadb54a5bbb93cfec5f7b5503",
    "data":{"result":true},
    "result":true,
    "statusCode":200
}
```
## Live Demo

We have a version of the adapter currently running on Cloud at the following location.

To test it follow this doc  https://github.com/digitalbridgekit/2fa-for-smartcontracts-matic#how-to-test-this-demo-online

To view a Video Demo  https://www.youtube.com/watch?v=GjcK_L5J0DQ

## Install Locally

### Install dependencies:

npm install

### Test 

npm test

### Run
node -e 'require("./index.js").server()'

## Call the external adapter/API server

```bash
curl -X POST -H 'Content-Type: application/json' -d '{"id": "278c97ffadb54a5bbb93cfec5f7b5503", "data": {"customerid": "alice", "hashedpin": "217506908309239"}}' "http://localhost:8080"
``` 

## Docker

To build a Docker container for a specific `2fa`, use the following example:

```bash
make docker adapter=2fa
```

The naming convention for Docker containers will be `2fa-adapter`.

Then run it with:

```bash
docker run -p 8080:8080 -e API_URL='YOUR_API_URL' -it 2fa-adapter:latest
```
_If on a Mac, this requires `gnu-sed` to be installed and set as the default for the command `sed`._

## Serverless

Create the zip:

```bash
make zip adapter=2fa
```

The zip will be created as `./2fa/dist/2fa-adapter.zip`.

### Install to AWS Lambda

- In Lambda Functions, create function
- On the Create function page:
  - Give the function a name
  - Use Node.js 12.x for the runtime
  - Choose an existing role or create a new one
  - Click Create Function
- Under Function code, select "Upload a .zip file" from the Code entry type drop-down
- Click Upload and select the `2fa-adapter.zip` file
- Handler:
    - index.handler for REST API Gateways
    - index.handlerv2 for HTTP API Gateways
- Add the environment variable (repeat for all environment variables):
  - Key: API_KEY
  - Value: Your_API_key
  - NAME: API_URL
  - VALUE: Your_API_url

- Save

By default, Lambda functions time out after 3 seconds. You may want to change that to 60s in case an API takes longer than expected to respond.

#### To Set Up an API Gateway (HTTP API)

If using a HTTP API Gateway, Lambda's built-in Test will fail, but you will be able to externally call the function successfully.

- Click Add Trigger
- Select API Gateway in Trigger configuration
- Under API, click Create an API
- Choose HTTP API
- Select the security for the API
- Click Add

#### To Set Up an API Gateway (REST API)

If using a REST API Gateway, you will need to disable the Lambda proxy integration for Lambda-based adapter to function.

- Click Add Trigger
- Select API Gateway in Trigger configuration
- Under API, click Create an API
- Choose REST API
- Select the security for the API
- Click Add
- Click the API Gateway trigger
- Click the name of the trigger (this is a link, a new window opens)
- Click Integration Request
- Uncheck Use Lamba Proxy integration
- Click OK on the two dialogs
- Return to your function
- Remove the API Gateway and Save
- Click Add Trigger and use the same API Gateway
- Select the deployment stage and security
- Click Add

### Install to GCP

- In Functions, create a new function, choose to ZIP upload
- Select Node.js 12 for the Runtime
- Click Browse and select the `2fa-adapter.zip` file
- Select a Storage Bucket to keep the zip in
- Function to execute: gcpservice
- Click More, Add variable (repeat for all environment variables)
  - NAME: API_KEY
  - VALUE: Your_API_key
  - NAME: API_URL
  - VALUE: Your_API_url
