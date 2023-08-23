# Webhooks

## [Webhooks Overview](./#webhooks-overview)

Ottu offers an optional end-to-end integration with the [Payment Gateway](../../user-guide/payment-gateway.md) (PG) through webhooks. By default, these webhooks are **disabled**, ensuring merchants only receive notifications if they choose to. These webhooks act as a reactive mechanism, notifying merchants either in real-time or on-demand, depending on the event. Whether it’s a payment being processed or a subsequent payment gateway [operation](../operations.md#external-operations) taking place, Ottu ensures that the merchant is promptly informed once the webhooks are enabled. Merchants can enable these webhooks in various ways, either directly from the Ottu dashboard or via the [Checkout API](../checkout-api.md).

Ottu simplifies the integration process by providing a unified response across all payment gateways. Whether you’re working with MPGS, KNET, PayPal, or any other gateway, you only need to handle Ottu’s standardized response. This ensures a consistent experience, eliminating the need to interpret varied responses from different payment gateways. Moreover, for those who might need it, the original response from the payment gateway, always converted to JSON format, is included in Ottu’s webhooks. This means that while Ottu takes on the heavy lifting of interpreting and standardizing payment gateway responses, you still have access to the raw data if required. The beauty of this approach is its extensibility. As you expand and decide to incorporate more payment gateways into your system, there’s no need for additional integration work on your end. Ottu ensures that everything will work seamlessly out of the box, saving you time and effort in the long run.

## [**Types of Webhooks**](./#types-of-webhooks)

Ottu supports two primary types of webhooks:

1.  #### Payment Event Webhook

    This webhook is triggered when a transaction is processed. It’s a real-time notification mechanism that informs the merchant immediately after a payment event.
2.  #### Operations Webhook

    This webhook is asynchronous and is triggered for subsequent payment gateway operations on a transaction, such as refunds, captures, or voids.

## [**Webhook Structure**](./#webhook-structure)

Webhooks from Ottu can be in two formats:

* **JSON:** This is the default format.
* **XML:** Available only upon request. If you require XML formatted webhooks, please contact the support team for activation.

Additionally, the user-agent header in the webhook mimics a Chrome browser by default. For custom user-agent headers, merchants can reach out to our support team for adjustments.

## [**Delivery Guarantees**](./#delivery-guarantees)

Ottu prides itself on a **99.99%** delivery guarantee for webhooks. However, merchants should be aware of the following:

* There might be multiple hits for the same payment event.
* By default, the retry mechanism is enabled. If the webhook fails to deliver on the first attempt, Ottu will try to resend it. The default number of retries is set to 3.

More details on the delivery mechanism and other related configurations will be discussed in subsequent sections.

## [Endpoint Requirements](./#endpoint-requirements)

For a seamless experience, merchants should ensure that their webhook endpoint:

* Is secure, preferably using HTTPS.
* Is always online and accessible to the public.
* Has a response time of less than **25 seconds** to ensure timely processing.
* Returns an HTTP status of 200 or 201. Any other status will lead Ottu to mark the notification as failed.
* Is idempotent. This is crucial as Ottu might notify the URL multiple times for the same event, and this ensures that the repeated calls do not result in unintended side effects.

While Ottu supports endpoints with disabled SSL verification, it’s not recommended due to potential security vulnerabilities.

## [Security](./#security)

Ensuring the integrity and authenticity of webhook data is paramount. To guarantee that the data you receive originates from Ottu and hasn’t been tampered with during transit, we employ a robust [signing mechanism](signing-mechanism.md). Ottu uses the **SHA-256 algorithm** to sign all webhook payloads. This cryptographic hash function provides a consistent and secure way to verify the data’s integrity. By checking the signature, you can be confident that the payload is genuine and has not been altered since it was sent by Ottu. For a detailed guide on how to verify the signature and implement this security measure in your system, click [here](signing-mechanism.md).

## [Webhook Notification Mechanism](./#webhook-notification-mechanism)

All the fields mentioned below come with default values to ensure a smooth integration out of the box. However, Ottu understands that every merchant might have unique requirements. Hence, each of these settings can be tailored to fit your specific needs. Whether it’s adjusting the timeout duration or specifying the number of retries, you have the flexibility to configure them as per your system’s behavior and preferences.

#### [**SSL Verification**](./#ssl-verification)

* **Field:** `ignore_ssl`
* **Default:** `True`
* **Description:** Ottu, by default, bypasses SSL certificate verification when calling the webhook URL. This is handy during development or testing with self-signed certificates. However, for production, it’s recommended to have SSL verification enabled to ensure security.

#### [**Error Notification**](./#error-notification)

* **Field:** `notify_on_error`
* **Default:** `True`
* **Description:** If Ottu encounters an error while calling the webhook URL, it can send an email notification. This ensures you’re promptly informed of any issues.

#### [**Email Recipients for Error Notification**](./#email-recipients-for-error-notification)

* **Field:** `emails`
* **Default:** Empty list
* **Description:** You can specify a list of email recipients who will be alerted in case of an error during the webhook call. This ensures that the right stakeholders are informed and can act if needed.

#### [**Timeout and Retry Mechanism**](./#timeout-and-retry-mechanism)

* **Fields:** `timeout`, `retries`, and `backoff_factor`
* **Defaults:** timeout: `15` seconds, retries: `3`, backoff\_factor: `5` seconds
* **Description:** When Ottu sends a webhook, it waits for 15 seconds (`timeout`) for a response. If there’s no response within this timeframe, it’s considered a timeout. In the event of a timeout, Ottu will retry sending the webhook. It will make 3 attempts (`retries`) in total, waiting 5 seconds (`backoff_factor`) between each retry. This means, in a scenario where all retries are exhausted due to timeouts, Ottu would have tried for a total of 55 seconds to deliver the webhook. It’s crucial to note that retries are only triggered in case of timeouts. For other errors or non-200 HTTP responses, Ottu won’t retry. This means your endpoint should always return an HTTP status of 200 to indicate successful receipt of the webhook. If not, Ottu will consider the notification as `failed`. Given the retry mechanism, there’s a possibility that your endpoint might receive the same webhook multiple times in case of timeouts. It’s essential to design your endpoint to handle such scenarios to avoid duplicate processing.
