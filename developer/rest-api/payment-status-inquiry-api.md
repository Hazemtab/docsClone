# Payment Status Inquiry API

## [Introduction](payment-status-inquiry-api.md#introduction)

The Payment Status Inquiry API endpoint is a part of Ottu's Check Status API designed to check the status of a specific payment transaction. This is especially useful when your system may not have received notifications about changes to a transaction's status. The Payment Status Inquiry API effectively acts as a manual status confirmation mechanism, reflecting the structure of a [payment webhook notification](../webhook/payment-notification.md). The endpoint can be triggered for payment transactions in the following states: `pending`, `attempted`, `failed`, or `expired`. If the transaction state is already `paid` or `authorized`, the status is returned immediately without needing to re-confirm with third-party [Payment Gateways](../../user-guide/payment-gateway.md) (PGs). However, if the transaction state is not up-to-date and is still listed in one of the aforementioned states, Ottu will trigger an API call to the PG to update the transaction state. In cases where multiple payment options were `attempted` using different PGs, all PGs that support payment status checks will be called, ensuring that you receive the most updated status for the payment.

## [Installation](payment-status-inquiry-api.md#installation)

### [Prerequisites](payment-status-inquiry-api.md#prerequisites)

For this API to work efficiently, here are a few things you need to be familiar with:

1. **Payment Gateway:** At least one Payment Gateway that supports status checks should be available. You can find more about Payment Gateways [here](../../user-guide/payment-gateway.md).
2. **Webhook:** The Payment Webhook response, as this is the response format which Payment Status Inquiry API returns. More details can be found [here](../webhook/payment-notification.md).

### [Authentication](payment-status-inquiry-api.md#authentication)

This API endpoint uses both [API Key](authentication.md#api-keys) and [Basic Authentication](authentication.md#basic-authentication). No special permissions are required for Basic Authentication.

### [How it Works](payment-status-inquiry-api.md#how-it-works)

1.  Payment Status Inquiry for `pending`, `attempted`, `failed`, or `expired` states\
    See [Payment Transaction State](../../user-guide/payment-tracking.md#payment-transaction-state-and-payment-attempt-state).\


    <figure><img src="../../.gitbook/assets/ideas - Inquiry API (&#x60;pending&#x60;, &#x60;attempted&#x60;, &#x60;failed&#x60;, or &#x60;expired&#x60;) (5).png" alt="" width="375"><figcaption></figcaption></figure>
2. Payment Status Inquiry for `paid` or `authorized` states

<figure><img src="../../.gitbook/assets/ideas - Inquiry API for &#x60;paid&#x60; or &#x60;authorized&#x60; (1).png" alt="" width="375"><figcaption></figcaption></figure>

### [Limitations](payment-status-inquiry-api.md#limitations)

The Payment Status Inquiry API implements a throttling mechanism to prevent potential system abuse. \
**Here are the rules:**

1. **Initial Grace Period (10 minutes):** If the Inquiry API is called within 10 minutes from when the payment transaction is created, the request will be throttled.
2. **First Request:** After the initial grace period, the first request is permitted. Any subsequent requests for the same transaction within the next 30 minutes will be throttled.
3. **Second Request** After the first 30-minute throttle period, a second request is allowed. Further requests for the same transaction within another 30 minutes will be throttled.
4. **Subsequent Requests:** If the number of requests for the same transaction exceeds three within a single day, any further requests will be denied.

{% hint style="info" %}
These rules are per transaction. Additionally, the endpoint allows a maximum of 30 requests per minute across all transactions
{% endhint %}

### [Requesting a Status Inquiry](payment-status-inquiry-api.md#requesting-a-status-inquiry)

To request a status inquiry, you must provide at least one of the following identifiers: \
[session\_id](checkout-api.md#session\_id-string-mandatory) or [order\_no](checkout-api.md#order\_no-string-optional). For a more detailed technical understanding and the implementation specifics of these operations, please refer to the Open API schema in the [API Schema Reference](payment-status-inquiry-api.md#api-schema-reference).

{% swagger src="../../.gitbook/assets/Ottu API (23).yaml" path="/b/pbl/v2/inquiry/" method="post" %}
[Ottu API (23).yaml](<../../.gitbook/assets/Ottu API (23).yaml>)
{% endswagger %}

## [FAQ](payment-status-inquiry-api.md#faq)

#### :digit\_one: [**What are the prerequisites to using the Payment Status Inquiry API?**](payment-status-inquiry-api.md#what-are-the-prerequisites-to-using-the-payment-status-inquiry-api)

You should have at least one [Payment Gateway](../../user-guide/payment-gateway.md) that supports status checks, and you should be familiar with the [Payment Webhook response](../webhook/payment-notification.md).

#### :digit\_two: [**What types of authentication does the Payment Status Inquiry API support?**](payment-status-inquiry-api.md#what-types-of-authentication-does-the-payment-status-inquiry-api-support)

The API endpoint supports both [API Key](authentication.md#api-keys) and [Basic Authentication](authentication.md#basic-authentication).

#### :digit\_three: [**Which payment transaction states can I inquire using the Payment Status Inquiry API?**](payment-status-inquiry-api.md#which-payment-transaction-states-can-i-inquire-using-the-payment-status-inquiry-api)

You can trigger the endpoint for payment transactions in the `pending`, `attempted`, `failed`, or `expired` states. If the transaction state is `paid` or `authorized`, the status is returned immediately. See [Payment Transactions State](../../user-guide/payment-tracking.md#payment-transactions-states).

#### :digit\_four: [**How does Ottu handle inquiries for transactions with outdated states?**](payment-status-inquiry-api.md#how-does-ottu-handle-inquiries-for-transactions-with-outdated-states)

If a transaction state is not up-to-date and is still listed as `pending`, `attempted`, `failed`, or `expired`, Ottu will trigger an API call to the [Payment Gateway](../../user-guide/payment-gateway.md) to update the transaction state.

#### :digit\_five: [**What if multiple payment options were attempted using different Payment Gateways?**](payment-status-inquiry-api.md#what-if-multiple-payment-options-were-attempted-using-different-payment-gateways)

If multiple payment options were `attempted` using different [Payment Gateways](../../user-guide/payment-gateway.md), all Gateways that support payment status checks will be called, ensuring that you receive the most updated status for the payment.

#### :digit\_six: [**Are there any limitations or throttling rules for the Payment Status Inquiry API?**](payment-status-inquiry-api.md#are-there-any-limitations-or-throttling-rules-for-the-payment-status-inquiry-api)

Yes, the Payment Status Inquiry API has a throttling mechanism. It includes an initial grace period, a limit on the number of requests within certain time intervals, and a limit on the total requests in a single day. Please refer to the [Limitations](payment-status-inquiry-api.md#limitations) section for detailed information.

#### :digit\_seven: [**What information do I need to provide to request a status inquiry?**](payment-status-inquiry-api.md#what-information-do-i-need-to-provide-to-request-a-status-inquiry)

To request a status inquiry, you need to provide either the [session\_id](checkout-api.md#session\_id-string-mandatory) or [order\_no](checkout-api.md#order\_no-string-optional) for the transaction.





We hope you found this guide to the Payment Status Inquiry API useful. As you proceed with your implementation, remember the following key points:

* **Stay within the Request Limits:** Be mindful of our API’s built-in throttling mechanisms to ensure smooth operation.
* **Understand the Webhook Response:** Knowing how to interpret the Payment Webhook response is crucial for accurate results. Check [Payment Notification](../webhook/payment-notification.md).
* **Use the Correct Identifier:** Provide either the [session\_id](checkout-api.md#session\_id-string-mandatory) or [order\_no](checkout-api.md#order\_no-string-optional) when requesting a status inquiry.
* **Consider the Transaction State:** The states `paid` and `authorized` will return the status immediately, while others will trigger a status check with the [Payment Gateway](../../user-guide/payment-gateway.md).