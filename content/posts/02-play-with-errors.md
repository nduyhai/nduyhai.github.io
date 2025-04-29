+++
date = '2025-04-29T17:49:04+07:00'
draft = false
title = '02 Play With Errors'
+++

## Introduction

During the development of payment systems, handling HTTP requests may seem straightforward at first glance, but it actually hides many unpredictable risks. Behind the familiar try-catch blocks, there are "hidden errors" that are often undetected or improperly handled — leading to serious consequences such as transaction failures, data loss, or even financial loss.  
In this article, we will dive deep into how hidden errors commonly occur during HTTP request handling in payment workflows, why they are easily overlooked, and how we can design more resilient systems to minimize these risks.

## Repository
This article is accompanied by a Proof of Concept (PoC) repository, which provides example code for the issues and solutions discussed.  
You can find it here: [payment-errors PoC](https://github.com/nduyhai/payment-errors/tree/main)

## Problem Statement

Let's take a look at the following sample code — a typical payment processing workflow:

```java
@Transactional
public void initiatePayment(PaymentRequest request) {
  // Step 1: Create local Payment record
  Payment payment = new Payment();
  payment.setUserId(request.getUserId());
  payment.setAmount(request.getAmount());
  payment.setStatus("INITIATED");
  paymentRepository.save(payment);

  try {
    log.info("Initiating payment for user: {}", request.getUserId());
    // Step 2: Call external service
    String url = "https://api.external-payment.com/resources/update";
    HttpEntity<ExternalPaymentRequest> externalRequest =
        new HttpEntity<>(buildExternalRequest(payment));

    ResponseEntity<ExternalPaymentResponse> response =
        restTemplate.postForEntity(url, externalRequest, ExternalPaymentResponse.class);

    if (!response.getStatusCode().is2xxSuccessful()) {
      throw new IllegalAccessException("Failed external call: " + response.getStatusCode());
    }

    ExternalPaymentResponse externalResponse = response.getBody();
    if (externalResponse == null || !externalResponse.isSuccess()) {
      throw new IllegalAccessException("External response failed");
    }

    // Step 3: Update payment status to CONFIRMED
    payment.setStatus("CONFIRMED");
    payment.setExternalTransactionId(externalResponse.getTransactionId());
    paymentRepository.save(payment);

  } catch (Exception ex) {
    // Step 4: Update payment status to FAILED
    payment.setStatus("FAILED");
    paymentRepository.save(payment);

    log.error("Payment failed due to external error", ex);
  }
}
```

At first glance, this flow seems complete: creating a local payment record, calling the external API, confirming success or failure, and updating the database accordingly. However, beneath the surface, there are serious hidden risks. 

Let's analyze the code carefully: it only treats a payment as successful if the HTTP response status is 200 **and** the body has `success=true`. In all other cases, it immediately marks the payment as **FAILED**. This approach introduces several critical issues that need to be carefully considered.

### Network Errors

Network errors — such as timeouts, connection refusals, or DNS failures — can occur at any time during the external API call. In such cases, the line:

```java
restTemplate.postForEntity(...)
```
typically throws an exception. The try-catch block then treats this as a payment failure.  
But is this always correct?

- In the case of a **timeout**, the request may have already reached the external partner, and they might have **processed** it successfully (or it might still fail).
- In the case of **connection refused** or **DNS failure**, it's more likely that the partner system **never received** the request.

In both cases, however, the system blindly marks the payment as **FAILED**.  
If the partner did in fact process the transaction successfully, this inconsistency could lead to severe issues:  
When the system retries or reprocesses failed transactions, it may unintentionally **double-charge** the user!

Clearly, the system is missing an important intermediate transaction state — something like **UNKNOWN** — indicating uncertainty about whether the transaction succeeded or failed.

For example:

```java
public enum Status {
    INITIATED,
    CONFIRMED,
    FAILED,
    UNKNOWN
}
```

For transactions in the **UNKNOWN** state, the system should have a dedicated mechanism to **reconcile** or **query** the final status with the external partner later.

### Abnormal Responses

Other problematic scenarios involve abnormal or unexpected API responses:

- The server returns HTTP 200 OK but the response body is invalid or missing.
- The response body contains an error code, but the system does not properly interpret it.
- The external API's documentation is incomplete, leaving certain cases undefined.

In these cases, it's crucial to have:

- A clear and complete documentation agreement with the partner regarding which HTTP statuses indicate success, failure, or require special handling.
- Any unknown or undefined statuses should default to **UNKNOWN**, not blindly assumed as success or failure.


<img src="/assets/res_1.png" alt="Unknown status">


The same principle applies to parsing error codes inside the response body — avoid making assumptions without strict definitions.


<img src="/assets/res_2.png" alt="Unknown status">

### Uncontrolled Retry

Another hidden risk is **uncontrolled retries**.  
If the system does not carefully distinguish between retryable and non-retryable failures, it may unnecessarily or incorrectly retry transactions that were already processed, compounding the risk of double charges or inconsistent states.

For errors that are eligible for retry, ensure that the external partner accepts retry attempts and that their API is idempotent.
## Conclusion

In payment systems, handling external API interactions must be done with extreme caution.  
We must recognize that not every error is final and that not every success is guaranteed.  
By introducing proper transaction states like **UNKNOWN**, building reconciliation mechanisms, and working closely with external partners to define clear API behaviors, we can make our systems much more robust, reliable, and safe for users.  
Ignoring hidden errors can silently turn small technical oversights into serious financial and reputational disasters.
