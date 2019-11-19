## Recurring payments

{% include jumbotron.html body="A recurring payment enables you to charge a
credit card without any consumer interaction. When an initial payment token is
generated subsequent payments is made through server-to-server requests. " %}

### Prerequisites

Prior to making any server-to-server requests, you need to supply the payment
instrument details and a payment token to Swedbank Pay by initial purchase or
[card verification][payment-verify].

There are two ways to initiate recurring payments procedures,
depending on if you want to make an initial charge or not:

* Initiate a recurring payment flow and **charge the credit card**.
  This is done by creating a "Purchase Payment" and generating a
  recurrence token.

* Initiate a recurring payment flow **without charging the credit card**.
  This is done by creating  a "Verify Payment" and generating a recurrence token.

#### Generate RecurrenceToken

* When posting a `Purchase` payment,
  you need to make sure that the attribute
  `generateRecurrenceToken` is set to `true`

{:.code-header}
**Attribute**

```JS
"generateRecurrenceToken": "true"
```

* When posting a `Verify` payment, a payment token will be generated
  automatically.

#### Creating a Payment

* You need to `POST` a [Purchase payment][card-payment-purchase] / and
  generate a recurrence token (safekeep for later recurring use).

* You need to `POST` a [Verify payment][payment-verify], that will
  automatically generate a recurrence token (for later recurring use).

#### Retreive Recurrence Token

The recurrence token can then be retrieved by doing a `GET` request against
the `payment`.
You need to store this `recurrenceToken` in your system and keep
track of the corresponding consumer-ID.

#### Delete Recurrence Token

You can delete a created recurrence token with a `PATCH`-request.
Please see technical reference for details
[here][card-payments-remove-payment-token].

### Recurring purchases

When you have a Recurrence token stored away. You can use the same token in a
subsequent [`recurring payment`][card-payment-recur] `POST`.
This will be a server-to-server affair, as we have both payment instrument
details and recurrence token from the initial payment.
Please note that this `POST`request is made directly on the payment level,
and will not create a payment order.

#### Options after a payment

You have the following options after a server-to-server Recur payment `POST`.

##### Autorization (intent)

* **Authorization (two-phase):** If you want the credit card to reserve the
  amount, you will have to specify that the intent of the purchase is
  Authorization.
  The amount will be reserved but not charged.
  You will later (i.e. when you are ready to ship the purchased products)
  have to make a [Capture][card-payment-capture] or
  [Cancel][card-payment-cancel] request.

##### Capture (intent)

* **AutoCapture (one-phase)**: If you want the credit card to be charged
  right away, you will have to specify that the intent of the purchase is
  AutoCapture.
  The credit card will be charged and you don't need to do any more
  financial operations to this purchase.​​​​​

##### General

* **Defining CallbackURL**: When implementing a scenario,
  it is optional to set a [`CallbackURL`][technical-reference-callback]
  in the `POST` request.
  If callbackURL is set Swedbank Pay will send a postback request to this URL
  when the consumer has fulfilled the payment.

### Verify

A `Verify` payment lets you post verifications to confirm the validity of
card information, without reserving or charging any amount.
This option is often used to initiate a recurring payment
flow where you do not want to charge the consumer right away.
Please note that all boolean credit card attributes involving rejection of
certain card types are optional and set on contract level.

{:.code-header}
**Request**

```http
POST /psp/creditcard/payments HTTP/1.1
Host: api.payex.com
Authorization: Bearer <MerchantToken>
Content-Type: application/json

{
  "payment": {
    "operation": "Verify",
    "currency": "NOK",
    "description": "Test Verification",
    "payerReference": "AB1234",
    "userAgent": "Mozilla/5.0...",
    "language": "nb-NO",
    "generatePaymentToken": true,
    "generateRecurrenceToken": false,
    "urls": {
      "hostUrls": ["http://test-dummy.net"],
      "completeUrl": "http://test-dummy.net/payment-completed",
      "cancelUrl": "http://test-dummy.net/payment-canceled",
      "paymentUrl": "http://example.com/perform-payment",
      "logoUrl": "https://test-dummy.net/payment-logo.png",
      "termsOfServiceUrl": "https://test-dummy.net/payment-terms.html"
    },
    "payeeInfo": {
      "payeeId": "12345678-1234-1234-1234-123456789012",
      "payeeReference": "CD1234",
      "payeeName": "Merchant1",
      "productCategory": "A123",
      "orderReference": "or-12456",
      "subsite": "MySubsite"
    }
  },
  "creditCard": {
    "rejectCreditCards": false,
    "rejectDebitCards": false,
    "rejectConsumerCards": false,
    "rejectCorporateCards": false
  }
}
```

{:.code-header}
**Response**

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "payment": {
        "id": "/psp/creditcard/payments/5adc265f-f87f-4313-577e-08d3dca1a26c",
        "number": 1234567890,
        "created": "2016-09-14T13:21:29.3182115Z",
        "updated": "2016-09-14T13:21:57.6627579Z",
        "operation": "Verify",
        "state": "Ready",
        "currency": "NOK",
        "amount": 0,
        "description": "Test Verification",
        "payerReference": "AB1234",
        "initiatingSystemUserAgent": "PostmanRuntime/3.0.1",
        "userAgent": "Mozilla/5.0",
        "language": "nb-NO",
        "transactions": { "id": "/psp/creditcard/payments/5adc265f-f87f-4313-577e-08d3dca1a26c/transactions" },
        "verifications": { "id": "/psp/creditcard/payments/5adc265f-f87f-4313-577e-08d3dca1a26c/verifications" },
        "urls" : { "id": "/psp/creditcard/payments/5adc265f-f87f-4313-577e-08d3dca1a26c/urls" },
        "payeeInfo" : { "id": "/psp/creditcard/payments/5adc265f-f87f-4313-577e-08d3dca1a26c/payeeInfo" },
        "settings": { "id": "/psp/creditcard/payments/5adc265f-f87f-4313-577e-08d3dca1a26c/settings" }
    },
    "operations": [
        {
            "href": "https://api.payex.com/psp/creditcard/payments/5adc265f-f87f-4313-577e-08d3dca1a26c",
            "rel": "update-payment-abort",
            "method": "PATCH",
            "contentType": "application/json"
        },
        {
            "href": "https://ecom.payex.com/creditcard/payments/verification/123456123412341234123456789012",
            "rel": "redirect-verification",
            "method": "GET",
            "contentType": "application/json"
        },
        {
            "method": "GET",
            "href": "https://ecom.dev.payex.com/creditcard/core/scripts/client/px.creditcard.client.js?token=123456123412341234123456789012",
            "rel": "view-verification",
            "contentType": "application/javascript"
        },

        {
            "method": "POST",
            "href": "https://ecom.dev.payex.com/psp/creditcard/confined/payments/{paymentId:guid}/verifications",
            "rel": "direct-verification",
            "contentType": "application/json"
        }

    ]
}
```

[payment-verify]: #verify
[card-payment-purchase]: #create-payment
[card-payment-recur]: #recurring-payments
[card-payment-capture]: #capture-sequence
[card-payment-cancel]: #cancel-sequence
[card-payments-remove-payment-token]: #delete-payment-token
[technical-reference-callback]: /other-features#callback