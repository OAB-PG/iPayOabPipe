# OAB Payment Gateway Integration

## Overview

This document outlines the steps required for a merchant to integrate with the OAB payment gateway. The integration will enable merchants to initiate payments, handle responses, and ensure secure transaction processing.

## Prerequisites

Before starting the integration, ensure the following:

- **Key credentials:**
  - Key Path
  - Alias Name
- Secure communication over HTTPS

## Transaction Types

### a. Purchase Transaction

#### Request Processing

```java
// Define key details
String keyPath = "/opt/filepath/";
String alias = "aliasname";
String currency = "512";
String language = "EN";
String receiptURL = "https://merchant.com/responseurl/";
String errorURL = "https://merchant.com/errorurl/";
String trackid = "87234234234";

// Create a new request instance
Request req = new Request();

// Set required fields
req.setKeyPath(keyPath);
req.setAlias(alias);
req.setCurrencycode(currency);
req.setLangid(language);
req.setResponseURL(receiptURL);
req.setErrorURL(errorURL);
req.setAmt(amount);
req.setTrackid(trackid);

// Set user-defined fields
req.setUdf1("User Defined value 1");
req.setUdf2("User Defined value 2");
req.setUdf3("User Defined value 3");
req.setUdf4("User Defined value 4");
req.setUdf5("User Defined value 5");

// Optional: Token transactions
req.setTokenNo(tokenNo);
req.setTokenFlag(tokenFlag);

// Optional: Enable split payment
req.setSplitPaymentIndicator("1");

// Create and configure split payment payload
SplitPaymentPayload splitPayLoad = new SplitPaymentPayload();
String splittrackid = "87234234234932334";
splitPayLoad.setAliasName("account001");
splitPayLoad.setNotes("Salary");
splitPayLoad.setType("1");
splitPayLoad.setReference(splittrackid);
splitPayLoad.setSplitAmount("10");

// Add split payment details to the request
req.addSplitPaymentPayload(splitPayLoad);



// Prepare the request transaction data
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

### Response Processing

```java
// Define key details
String keyPath = "/opt/filepath/";
String alias = "aliasname";

// Retrieve parameters from request
String replyTranData = request.getParameter("trandata");
String trackId = request.getParameter("trackId");

// Create a new ReplyTranData instance and set values
ReplyTranData tranData = new ReplyTranData();
tranData.setKeyPath(keyPath);
tranData.setAlias(alias);
tranData.setTrandata(replyTranData);
tranData.setTrackId(trackId);

// Process the transaction response
Reply reply = OabIpayReplyBuilder.prepareReply(tranData);

// Retrieve result details
reply.getResult();
reply.getDate();
reply.getRef();
reply.getAuth();
reply.getTrackId();
reply.getTranId();
reply.getAmt();
reply.getUdf1();
reply.getUdf2();
reply.getUdf3();
reply.getUdf4();
reply.getUdf5();
reply.getPaymentId();
reply.getTokenNo();
reply.getTranDate();
reply.getTranRequestDate();
reply.getTranResponseDate();
```

### b. Inquiry Transaction

#### Request Creation

```java
// Create a new request instance
Request req = new Request();
req.setAlias(alias);
req.setKeyPath(keyPath);
req.setCurrencycode(currency);
req.setAmt(amount);
req.setTransid(transid);

// Set proxy settings
req.setProxy(true);
req.setProxyHost("proxy.host");
req.setProxyPort(1002);

```

#### Inquiry Processing

```java
Reply reply;
if (/* transid is the original track ID */) {
    reply = new OabIpayConnection().processInquiryByTrackId(req);
} else if (/* transid is the original transaction ID */) {
    reply = new OabIpayConnection().processInquiryByTranId(req);
} else if (/* transid is the original payment ID */) {
    reply = new OabIpayConnection().processInquiryByPaymentId(req);
} else if (/* transid is the original reference number */) {
    reply = new OabIpayConnection().processInquiryByRefNo(req);
}

reply.getResult();
```

### c. Reversal Transaction

#### Request Creation

```java
// Create a new request instance
Request req = new Request();
req.setAlias(alias);
req.setKeyPath(keyPath);
req.setCurrencycode(currency);
req.setAmt(amount);
req.setTransid(transid);

// Set proxy settings
req.setProxy(true);
req.setProxyHost("proxy.host");
req.setProxyPort(1002);

```

#### Reversal Processing

```java
Reply reply;
if (/* transid is the original track ID */) {
    reply = new OabIpayConnection().processReversalByTrackId(req);
} else if (/* transid is the original transaction ID */) {
    reply = new OabIpayConnection().processReversalByTranId(req);
}

reply.getResult();
```

### d. Refund Transaction

#### Request Creation

```java
// Create a new request instance
Request req = new Request();
req.setAlias(alias);
req.setKeyPath(keyPath);
req.setCurrencycode(currency);
req.setAmt(amount);
req.setTransid(transid);

// Set proxy settings
req.setProxy(true);
req.setProxyHost("proxy.host");
req.setProxyPort(1002);
```

#### Refund Processing

```java
SplitPaymentPayload splitPaymentPayload = new SplitPaymentPayload();
splitPaymentPayload.setSplitTranId(splitTranId);
splitPaymentPayload.setNotes("Refund");
splitPaymentPayload.setReference(refNumber);
splitPaymentPayload.setSplitAmount(splitAmount);

req.addSplitPaymentPayload(splitPaymentPayload);
Reply reply = new OabIpayConnection().processRefundByTranId(req);
reply.getResult();
```

## Security Guidelines

- Use HTTPS for all API calls.
- Store API keys securely and never expose them in client-side code.

## Support

For any issues or questions, contact our support team at **[pg-support@oman-arabbank.com](mailto\:pg-support@oman-arabbank.com)**.

