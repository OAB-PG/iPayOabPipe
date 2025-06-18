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

### b. Merchant Hosted Transaction (VBV Flow)

In a **Merchant Hosted** (VBV 3D Secure) flow, the merchant collects the customer's card details (such as card number, expiry, CVV) and initiates the transaction from the backend server using the OAB Payment Gateway SDK.

This method is used when merchants require a **seamless and customized checkout experience** while still redirecting the customer for 3D Secure authentication.

#### Implementation Overview

1. **Configure Request Details**\
   Set the following transaction parameters:

   - `keyPath` – Local path to the encryption key file
   - `alias` – Merchant alias name provided by the bank
   - `currencycode`, `langid`, `amt`, and `trackid`
   - `responseURL` and `errorURL` – Merchant endpoints for post-authentication redirection

2. **Populate UDF Fields (Optional)**\
   Set values for UDF1 to UDF5 for custom tracking or metadata.

3. **Collect and Assign Card Details**\
   Securely collect and assign:

   - Card number (`cardNo`)
   - Cardholder name (`cardName`)
   - Expiry month/year (`expMonth`, `expYear`)
   - CVV2 (`cvv2`)

   > ⚠️ *Card details must be collected over HTTPS and handled only in PCI-DSS-compliant environments.*

4. **Configure Split Payment (Optional)**\
   If using split payment:

   - Set `splitPaymentIndicator` to "1"
   - Create a `SplitPaymentPayload` and set:
     - Alias
     - Notes
     - Type (1 = fixed)
     - Reference (unique identifier)
     - Split amount
   - Add the payload to the request

5. **Handle Tokenization (Optional)**\
   If tokenization is supported and required:

   - Set `tokenNo` (if available)
   - Set `tokenFlag` based on action type (e.g., "2" for saving token)

6. **Build and Send the Request**\
   Below is an example of how the request is built:

```java
req = new Request();
req.setKeyPath(resourcePath);
req.setAlias(aliasName);
req.setCurrencycode(currency);
req.setLangid(language);
req.setResponseURL(receiptURL);
req.setErrorURL(errorURL);
req.setAmt(amount);
req.setTrackid(trackid);
req.setUdf1(udf1);
req.setUdf2(udf2);
req.setUdf3(udf3);
req.setUdf4(udf4);
req.setUdf5(udf5);
req.setTokenNo(tokenNo);
req.setTokenFlag(tokenFlag);

req.setCardNo(cardNo);
req.setCardName(cardName);
req.setExpMonth(expMonth);
req.setExpYear(expYear);
req.setCvv2(cvv2);

// Set transaction type only for wallet-based flows:
// AP = Apple Pay, SP = Samsung Pay, QR = QR Payment, AMP = Mobile Payment
// ❗ Do NOT set type for card-based transactions
req.setType("AP"); // Replace "AP" with "SP", "QR", or "AMP" as needed

// Set sample browser/device context values
req.setBrowserAcceptHeader("sample/browser-accept-header");
req.setBrowserIP("sample.client.ip.address");
req.setBrowserJavaEnabled(false); // or true based on client environment
req.setBrowserJavascriptEnabled(true); // or false if JS is disabled
req.setBrowserLanguage("en-US");
req.setBrowserColorDepth("24");
req.setBrowserScreenHeight("1080");
req.setBrowserScreenWidth("1920");
req.setBrowserTZ("+240"); // Browser timezone offset in minutes
req.setBrowserUserAgent("sample-user-agent-string");

req.setSplitPaymentIndicator("1");
SplitPaymentPayload splitPayLoad = new SplitPaymentPayload();
splitPayLoad.setAliasName("accountalias");
splitPayLoad.setNotes("Salary");
splitPayLoad.setType("1");
splitPayLoad.setReference(String.valueOf(Math.abs(new Random().nextInt())));
splitPayLoad.setSplitAmount("10");
req.addSplitPaymentPayload(splitPayLoad);

if (tokenNo != null && tokenNo.isEmpty()) {
    req.setTokenNo(tokenNo);
    if ("1".equals(action)) {
        req.setTokenFlag("2");
    }
}

Reply reply = new OabIpayConnection().initiateTransaction(req);
```

7. **Redirect Customer to Redirect URL**\
   The customer should be redirected to the `redirectUrl` returned in the `Reply` to complete the 3D Secure authentication and finalize the transaction.

#### Example Use Case

This method is ideal for platforms with:

- A fully branded checkout page
- Secure server infrastructure
- The need to manage card entry directly while using bank-hosted 3DS authentication

#### Important Notes

- The `redirectUrl` returned from the API must be used as-is, without modification.
- Never log or store cardholder data.
- Do not expose private keys, aliases, or card details on the client side.
- Always enforce HTTPS for all communications.

### c. Inquiry / Reversal / Refund Transactions

#### Request Creation

```java
// Create a new request instance
Request req = new Request();
req.setAlias(alias);
req.setKeyPath(keyPath);
req.setCurrencycode(currency);
req.setAmt(amount);
req.setTransid(transid);

```


#### Inquiry Processing

```java
// Set proxy settings
String proxyHost = "proxyhost";
Integer proxyport = 8080;
if(actionBy.equals("TRACKID")){
    reply = new OabIpayConnection(proxyHost, proxyport).processInquiryByTrackId(req);
} else if (actionBy.equals("PAYMENTID")){
    reply = new OabIpayConnection(proxyHost, proxyport).processInquiryByPaymentId(req);
} else if (actionBy.equals("TRANID")){
    reply = new OabIpayConnection(proxyHost, proxyport).processInquiryByTranId(req);
} else if (actionBy.equals("REFNO")){
    reply = new OabIpayConnection(proxyHost, proxyport).processInquiryByRefNo(req);
}
```


#### Reversal Processing

```java
Reply reply;
// Set proxy settings
String proxyHost = "proxyhost";
Integer proxyport = 8080;

if (/* transid is the original track ID */) {
    reply = new OabIpayConnection().processReversalByTrackId(req);
} else if (/* transid is the original transaction ID */) {
    reply = new OabIpayConnection().processReversalByTranId(req);
}

reply.getResult();
```

#### Refund Processing

#### Split Request Creation

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

## Response Parsing (Callback Handling)

When the payment gateway redirects back to the merchant's `responseURL` or `errorURL`, the `trandata` is received as an encrypted string. This string must be parsed securely using the provided alias and key path.

### Sample Response Parsing

```java
    String tranData = "6593A06B027BD3D4177C4D1055317118D2CB3D0523746881E8029E5185E230D0C727434597A49722A0453533AD05A2B9E28134E015010AEBFF2A17F12991C0CDABE706A4F6A1D5FC0B99ADD8AAA0004D867B920B0244BD5201F2F1D9D2725E9E13ECB9A9C236D458E0D88B2AFADD39C93302FAB3C6B2BEDDBDD6D95734E6E4872F6944DBC695D89B0C06BA44AB4BB0F406FEF46210DDF959338C5D3580CDB46887A44BF6BF7991A373394C1C12334A13FFF3C0D1CF24D1AC400BF5A4171A1BE88C3C762E6FF86976384940DC42EE35FA1078196607F4335D98A0C1398DB918976280A54125006473B3B5964B92F6D8779E13DD632767FE15BCB7E0D05E288DC5";
  String keyPath = "/opt/filepath/";
  String alias = "aliasname";

    ReplyTranData replyTranData = new ReplyTranData();
    replyTranData.setAlias(alias);
    replyTranData.setKeyPath(keyPath);
    replyTranData.setTrandata(tranData);

    Reply reply = OabIpayReplyBuilder.prepareReply(replyTranData);

    System.out.println("Reply Payment Id : " + reply.getPaymentId());
    System.out.println("Reply Tran Id    : " + reply.getTranId());
    System.out.println("Reply Result     : " + reply.getResult());
    System.out.println("Reply Auth       : " + reply.getAuth());
    System.out.println("Reply Track Id   : " + reply.getTrackId());
    System.out.println("Reply Ref No     : " + reply.getRef());

```

### Notes
- Ensure the `trandata` is read securely from the HTTP request.
- The reply object provides all the transaction response fields.
- Always validate the `Result` field to determine if the transaction was approved, declined, or failed.


## Support

For any issues or questions, contact our support team at **[pg-support@oman-arabbank.com](mailto\:pg-support@oman-arabbank.com)**.
