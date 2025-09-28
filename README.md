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

req.setTokenNo(tokenNo);
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

### b. Merchant Hosted Transaction (VBV Flow)

Merchant collects the customer's card details and submits from backend. 3D Secure authentication handled by redirecting the customer.

#### Step-by-Step Integration

1. Configure request: alias, keyPath, URLs, amount, currency, track ID
2. Set UDF fields (optional)
3. Set card details (PCI-compliant only): card number, expiry, CVV, etc.
4. Configure tokenization (optional): `tokenNo`, `tokenFlag`
5. Set device/browser headers if using wallet flows (Apple Pay, Samsung Pay, etc.)
6. Set split payment indicator and payload (if used)
7. Initiate transaction:

```java
Reply reply = new OabIpayConnection().initiateTransaction(req);
```

8. Redirect customer to `reply.getRedirectUrl()` for authentication

### c. Inquiry / Reversal / Refund

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

### d. Refund to Customer Account

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

## Callback Handling

```java
String tranData = "...";
ReplyTranData replyTranData = new ReplyTranData();
replyTranData.setAlias("aliasname");
replyTranData.setKeyPath("/opt/filepath/");
replyTranData.setTrandata(tranData);
Reply reply = OabIpayReplyBuilder.prepareReply(replyTranData);
```

## Security Guidelines

* All communication over HTTPS
* Never log/store sensitive card data
* Do not expose private key or alias on the client

## Support

Contact: **[pg-support@oman-arabbank.com](mailto:pg-support@oman-arabbank.com)**
