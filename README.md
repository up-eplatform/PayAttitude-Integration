# API DOCUMENT FOR PAYATTITUDE SCHEME 

# INTRODUCTION 
This document describes the steps to be taken by merchants that want to receive Scheme (VisaCard, MasterCard, Amex, VerveCard and PayAttitude) details on their respective sites and process the transactions on our Payment Gateway. 

**NOTE**: This document is for merchants that have been setup (by E-Commerce Ops) on the payment gateway.


## INTEGRATION PROCEDURE

* GO TO https://test.payarena.com/prospectivemerchants on your browser
* Fill in details in the form that will be presented to you.
* Submit the form
* Test Configuration details will be forwarded to you via the email you provided. This will include the following: 
   Merchant ID
   Cryptographic Key
* The Merchant ID and Cryptographic key are used to identify a Webshop on UP MPI each time a Webshop sends a payment request, i.e. a create order request to UP MPI.
* Review this documentation and commence implementation
* Review UAT documentation and ensure that all UAT requirements are met on merchant site
* System Integration Test and User Acceptance Test with Unified Payments team
* On successful UAT, Unified Payments team will initiate Go-live process and provide production parameters
* Apply production parameters and confirm successful pilot

**TRANSACTION FLOW**

![TRANSACTION PROCESS FLOW](https://github.com/up-eplatform/PayAttitude-Integration/blob/a25bd43d31320632d823ecc5d40c080632bb0821/img/payattitude%20flow.png)

**PROCESS DESCRIPTION**
1. Customer dials USSD code and enters payment details(phone number and amount),The aggregator captures the data and sends it to UP MPI
2. UP MPI collects phone details and sends authentication requests to the scheme(PayAttitude).  
3. PayAttitude contacts the customer to authenticate the transaction
4. The customer authenticates transaction and sends a response to PayAttitude. 
5. PayAttitude contacts the issuer to authorize transaction 
6. The Issuer authorizes the transaction and sends the authorization response to PayAttitude
7. PayAttitude sends the authorization response to UP MPI
8. UP MPI sends the authorization response and the status of the transaction is displayed. 

## IMPLEMENTATION
For a seamless integration, kindly follow the outlined steps below:
**Step 1:** Create a json representation of data elements as follows: 
```json
{
  "id": "MERCHANTID",
	"description": "payment for goods",
	"amount": 200, 
	"fee": 0,
	 "currency": "566",
	"returnUrl": "http://mywebsite.com/returnurl",
	"secretKey": "AJAHD45S45F4S4S45AS45D54S",
	"scheme": "",
	 "vendorId": "",
	 "parameter": "",
	"count" : 0
	 }
   ```
   
**Step 2:** Send the json data created above as a POST request to https://test.payarena.com/Aggregator, the header should specify Accept and Content-Type as application/json. This will return an id for the transaction posted above.

**Step 3:** Generate a SHA1 string of your secret key.

**Step 4:** Create a json representation of the card details as follows:

```json
{
  "secretKey": "AJAHD45S45F4S4S45AS45D54S",
  "scheme": "",
  "cardNumber": "",
  "expiry": "",
  "cvv": "",
  "cardholder": "",
  "mobile": "07087419908",
  "pin": ""
}
 ```


**Step 5:** Using the AES algorithm, sample code provided below, encrypt the json data in Step 4 above using the first 16 characters of the string obtained in Step 3. See an example below:

 **C#** 
```
public static string Encrypt(byte[] dataToEncrypt, byte[] key, byte[] iv)
{
	using (var aes = new AesCryptoServiceProvider())
	 {
		aes.Mode = CipherMode.CBC;
	 	aes.Padding = PaddingMode.PKCS7;
		aes.Key = key;
	   	aes.IV = iv;
		using (var memoryStream = new MemoryStream())
		 {
		var cryptoStream = new CryptoStream(memoryStream, aes.CreateEncryptor(), CryptoStreamMode.Write);
	 cryptoStream.Write(dataToEncrypt, 0, dataToEncrypt.Length); cryptoStream.FlushFinalBlock();
	 return memoryStream.ToArray().Aggregate("", (current, txt) => current + txt.ToString("X2");
}

}

} 
```

where dataToEncrypt is a byte array of the json card details in step 4 above and key and iv are byte arrays of the first 16 characters in step 3 above.

**Step 6:** 

send a POST to the below URL(endpoint) with the payload to complete the transaction: 

URL
https://test.payarena.com/Home/PayAttitudeTransactionPost
Request Payload
```json
{
  "Id": "TRANSACTIONID",
  "Mid": "MERCHANTID",
  "Payload": "ENCRYPTEDDATA"
}
```
Set Accept to application/json in the request header.

**Response**
```json
{
    "Order Id": "TRANSACTION ID",
    "Amount": "TRANSACTION AMOUNT",
    "Description": "DESCRIPTION OF TRANSACTION",
    "Currency": "TRANSACTION CURRENCY",
    "Status": "TRANSACTION STATUS",
    "PAN": "CUSTOMER PHONE NUMBER",
    "TranTime": "TRANSACTION DATETIME",
    "StatusDescription": "DESCRIPTION OF THE TRANSACTION STATUS"
}
```

**Note:** Upon receipt of the payment request, the PayAttitude transaction takes about 90seconds to be completed. If the customer does not authenticate the transaction within that time, a timeout response will be sent back to the requestor.
 
**TRANSACTION STATUS QUERY** 

To query the status of a transaction, send a GET request in the format below. 
https://test.payarena.com/Status/Transactionid . Where transactionID is the ID received as a response after the order was created.
The status is the status of the transaction, this could be either of the following in the table below:

|Status | Description |
| --- | --- |
| Approved/Approved Successful | A transaction is approved when the customer is successfully debited and value for transaction is received. |
| Cancelled| A transaction is cancelled when the customer decides to not enter payment details and returns back to the merchant site. |
| Declined|A transaction is declined when one of the following happens:Unsucccessful authentication,Unsuccessful authorization,System error |
| Declined| A transaction is declined when one of the following happens:Unsucccessful authentication,Unsuccessful authorization,System error |
| Initiated| A transaction is initiated when the customer abandons a transaction. |

 




## Contact
Created by [@eplatform(eplatform@up-ng.com) - feel free to contact us for clerifications.
