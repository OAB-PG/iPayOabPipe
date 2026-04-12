# OAB Payment Gateway Integration

## Overview

This document outlines the steps required for a merchant to integrate with the OAB Payment Gateway. The integration enables merchants to initiate payments, handle responses, and ensure secure transaction processing.

## Prerequisites

Before starting the integration, ensure the following:

* **Key credentials:**

  * Key Path
  * Alias Name
* Secure communication over HTTPS

## Transaction Types

### a. Purchase Transaction

#### Request Setup

```java
String keyPath = "/opt/filepath/";
String alias = "aliasname";
String currency = "512";
String language = "EN";
String receiptURL = "https://merchant.com/responseurl/";
String errorURL = "https://merchant.com/errorurl/";
String trackid = "87234234234";
String amount = "10.000";

Request req = new Request();
req.setKeyPath(keyPath);
req.setAlias(alias);
req.setCurrencycode(currency);
req.setLangid(language);
req.setResponseURL(receiptURL);
req.setErrorURL(errorURL);
req.setAmt(amount);
req.setTrackid(trackid);

req.setUdf1("User Defined value 1");
req.setUdf2("User Defined value 2");
req.setUdf3("User Defined value 3");
req.setUdf4("User Defined value 4");
req.setUdf5("User Defined value 5");

req.setTokenNumber(tokenNumber);
req.setTokenFlag(tokenFlag);

req.setSplitPaymentIndicator("1");
SplitPaymentPayload splitPayLoad = new SplitPaymentPayload();
splitPayLoad.setAliasName("account001");
splitPayLoad.setNotes("Salary");
splitPayLoad.setType("1");
splitPayLoad.setReference("87234234234932334");
splitPayLoad.setSplitAmount("10");
req.addSplitPaymentPayload(splitPayLoad);

RequestTranData reqTranData = OabIpayRequestBuilder.prepareRequestTranData(req);
```
### b. Token Registration Transaction

Important correction:
- Removed `tokenNumber` and `tokenFlag`, as they are not required for token registration
- Updated the amount to a small value such as `0`, in line with the token registration flow
- Updated the builder call to `buildTokenRegistrationData(req)` to match the actual implementation
- Do not include `splitPaymentIndicator` or `SplitPaymentPayload` in the card registration to token request
- 
```java
RequestTranData reqTranData = OabIpayRequestBuilder.buildTokenRegistrationData(req);
```
#### Form Submission

```html
<form action="<%=reqTranData.getWebAddress() %>" method="post">
  <input type="hidden" name="tranportalId" value="<%= reqTranData.getTranportalId() %>" />
  <input type="hidden" name="responseURL" value="<%= reqTranData.getResponseURL() %>" />
  <input type="hidden" name="errorURL" value="<%= reqTranData.getErrorURL() %>" />
  <input type="hidden" name="trandata" value="<%= reqTranData.getTrandata() %>" />
  <button type="submit">Submit</button>
</form>
```

### c. Merchant Hosted Transaction (VBV Flow)

Merchant collects the customer's card details and submits from backend. 3D Secure authentication handled by redirecting the customer.

#### Step-by-Step Integration

1. Configure request: alias, keyPath, URLs, amount, currency, track ID
2. Set UDF fields (optional)
3. Set card details (PCI-compliant only): card number, expiry, CVV, etc.
4. Configure tokenization (optional): `tokenNumber`, `tokenFlag`
5. Set device/browser headers if using wallet flows (Apple Pay, Samsung Pay, etc.)
6. Set split payment indicator and payload (if used)
7. Initiate transaction:

```java
Reply reply = new OabIpayConnection().initiateTransaction(req);
```

8. Redirect customer to `reply.getRedirectUrl()` for authentication

### d. Inquiry / Reversal / Refund

#### Inquiry

```java
Reply reply;
String proxyHost = "proxyhost";
Integer proxyport = 8080;

if(actionBy.equals("TRACKID")) {
  reply = new OabIpayConnection(proxyHost, proxyport).processInquiryByTrackId(req);
} else if (actionBy.equals("TRANID")) {
  reply = new OabIpayConnection(proxyHost, proxyport).processInquiryByTranId(req);
}
```

#### Reversal

```java
Reply reply;
reply = new OabIpayConnection().processReversalByTranId(req);
```

#### Refund

```java
SplitPaymentPayload splitPaymentPayload = new SplitPaymentPayload();
splitPaymentPayload.setSplitTranId(splitTranId);
splitPaymentPayload.setNotes("Refund");
splitPaymentPayload.setReference(refNumber);
splitPaymentPayload.setSplitAmount(splitAmount);
req.addSplitPaymentPayload(splitPaymentPayload);

Reply reply = new OabIpayConnection().processRefundByTranId(req);
```

### f. Refund to Customer Account

```java
Request req = new Request();
req.setKeyPath("/path/to/merchant/key/");
req.setAlias("merchantAlias");
req.setCurrencycode("512");
req.setTransid("originalTransactionId");
req.setReveiverAccount("customerAccountNumber");
req.setReceiverName("customerName");
req.setAmt("refundAmount");
req.setSwiftBankId("bankSwiftCode");
req.setBranch("branchCode");
req.setPurpose("refundPurposeCode");
req.setCountry("countryCode");
req.setLocation("cityOrRegionCode");
req.setTrackid(String.valueOf(Math.abs(new Random().nextInt())));

Reply reply = new OabIpayConnection().refundToCustomerAccount(req);
```

### g. Tokenized Purchase Transaction

Use this flow to perform a purchase using a previously registered card token.

```java
Request req = new Request();

req.setKeyPath(keyPath);
req.setAlias(aliasName);
req.setCurrencycode(currency);
req.setLangid(language);
req.setAmt(amount);
req.setTrackid(trackid);
req.setTokenNumber(tokenNo);

req.setUdf1(udf1);
req.setUdf2(udf2);
req.setUdf3(udf3);
req.setUdf4(udf4);
req.setUdf5(udf5);
req.setUdf6(udf6);
req.setUdf7(udf7);
req.setUdf8(udf8);
req.setUdf9(udf9);
req.setUdf10(udf10);
req.setUdf11(udf11);
req.setUdf12(udf12);
req.setUdf13(udf13);
req.setUdf14(udf14);
req.setUdf15(udf15);
req.setUdf16(udf16);
req.setUdf17(udf17);
req.setUdf18(udf18);
req.setUdf19(udf19);
req.setUdf20(udf20);

Reply reply = new OabIpayConnection().tokenizedCardPurchase(req);

```

## Callback Handling

```java
String tranData = "...";
ReplyTranData replyTranData = new ReplyTranData();
replyTranData.setAlias("aliasname");
replyTranData.setKeyPath("/opt/filepath/");
replyTranData.setTrandata(tranData);
Reply reply = OabIpayReplyBuilder.prepareReply(replyTranData);
```
### Response Handling and Result Description

The merchant must validate the transaction outcome using `reply.getResult()`.  
A transaction must be considered successful only when the returned result code matches the expected success code for that transaction type. Any other result code must be treated as a failure.

#### Response Fields

| Field | Description |
|---|---|
| `reply.getPaymentId()` | Unique payment identifier returned by the gateway |
| `reply.getErrorText()` | Error message returned by the gateway when the transaction fails or encounters an issue |
| `reply.getResult()` | Final transaction result code used to determine success or failure |
| `reply.getTranId()` | Gateway transaction identifier |
| `reply.getTrackId()` | Merchant track ID sent in the request |
| `reply.getRef()` | Retrieval reference number for the transaction |
| `reply.getTranDate()` | Transaction processing date |
| `reply.getTranRequestDate()` | Date and time when the request was received by the gateway |
| `reply.getTranResponseDate()` | Date and time when the response was generated by the gateway |
| `reply.getTokenCustId()` | Customer token ID returned by the gateway, used for future tokenized transactions |
| `reply.getBrandType()` | Card brand returned by the gateway |
| `reply.getMaskedCard()` | Masked card number used in the transaction |
| `reply.getCardName()` | Cardholder name associated with the card |
| `reply.getUdf1()` to `reply.getUdf20()` | User-defined fields returned in the response, if provided in the request |

#### Success Result Codes by Transaction Type

| Transaction Type | Success Result Code | Description |
|---|---|---|
| Purchase Transaction | `CAPTURED` | The purchase transaction is successful only when the result is `CAPTURED` |
| Token Registration | `REGISTERED` | The card token registration is successful only when the result is `REGISTERED` |
| Inquiry Transaction | `SUCCESS` | The inquiry is successful only when the result is `SUCCESS` |
| Reversal Transaction | `VOIDED` | The reversal is successful only when the result is `VOIDED` |
| Card Refund | `CAPTURED` | The card refund is successful only when the result is `CAPTURED` |
| Account Refund | `CAPTURED` | The account refund is successful only when the result is `CAPTURED` |

#### Failure Handling

Any result code other than the expected success result for the specific transaction type must be treated as a failure.

Examples:

- For purchase transactions, values such as `NOT CAPTURED` and `AUTH ERROR` must be treated as failure
- For token registration, any value other than `REGISTERED` must be treated as failure
- For inquiry transactions, values such as `FAILURE(SUSPECT)`, `FAILURE(NOT CAPTURED)`, and `AUTH ERROR` must be treated as failure
- For reversal transactions, any value other than `VOIDED` must be treated as failure
- For card refund and account refund transactions, any value other than `CAPTURED` must be treated as failure

## Security Guidelines

* All communication over HTTPS
* Never log/store sensitive card data
* Do not expose private key or alias on the client

## Support

Contact: **[pg-support@oman-arabbank.com](mailto:pg-support@oman-arabbank.com)**
