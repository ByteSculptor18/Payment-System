README.md

### Overview

This project simulates a real-world payment gateway architecture. It uses **five key design patterns** to organize the code, separating concerns and making the system easier to extend and maintain.

### Design Patterns Used

| Pattern | Class / Component | Purpose |
| :--- | :--- | :--- |
| **Template Method** | `PaymentGateway` | Defines the standard, invariant steps for processing any payment: **Validate -\> Initiate -\> Confirm**. Concrete gateways (`PaytmGateway`, `RazorpayGateway`) implement the specific logic for each step. |
| **Strategy** | `BankingSystem` Interface | Defines an interface for the core banking logic. Concrete classes (`PaytmBankingSystem`, `RazorpayBankingSystem`) implement different strategies (e.g., different success rates) that are used during the `initiatePayment` step. |
| **Proxy** | `PaymentGatewayProxy` | Wraps a `PaymentGateway` object to add the **retry** functionality. It provides the same interface but enhances the behavior (structural pattern). |
| **Factory** | `GatewayFactory` | Centralizes the logic for creating the appropriate `PaymentGateway` and wrapping it in a `PaymentGatewayProxy` with the correct number of retries (creational pattern). |
| **Singleton** | `GatewayFactory`, `PaymentService`, `PaymentController` | Ensures that only a single instance of these central management classes exists throughout the application, providing a global point of access (creational pattern). |

### 🏗️ Architecture Breakdown

The system is structured into clear layers:

1.  **Client/Controller Layer (`PaymentController`)**: The entry point for all client requests. It uses the `GatewayFactory` to create the appropriate gateway and passes the request to the `PaymentService`.
2.  **Service Layer (`PaymentService`)**: A central service that manages which `PaymentGateway` is currently active (set via the Controller) and handles the payment processing flow.
3.  **Gateway Layer (`PaymentGateway`, `PaytmGateway`, `RazorpayGateway`, `PaymentGatewayProxy`)**:
      * The `PaymentGateway` defines the core payment sequence (Template Method).
      * Concrete Gateways implement the unique validation and confirmation rules.
      * The `PaymentGatewayProxy` adds crucial retry logic for robustness.
4.  **Banking System Layer (`BankingSystem`, `PaytmBankingSystem`, `RazorpayBankingSystem`)**: Abstracted external systems that execute the actual transaction and simulate different success/failure rates (Strategy Pattern).

### Example Output

The `main` function demonstrates two payment requests. The first, via **Paytm**, is configured with up to **3 retries** in the `GatewayFactory` and a lower success rate (20% failure simulation in `PaytmBankingSystem`), often resulting in a retry attempt. The second, via **Razorpay**, is configured with only **1 attempt** (0 retries) and a higher success rate (10% failure simulation in `RazorpayBankingSystem`).

```
Processing via Paytm
------------------------------
[Paytm] Validating payment for Aditya.
[Paytm] Initiating payment of 1000 INR for Aditya.
[BankingSystem-Razorpay] Processing payment of 1000...
[PaymentGateway] Initiation failed for Aditya.
[Proxy] Retrying payment (attempt 2) for Aditya.
[Paytm] Validating payment for Aditya.
[Paytm] Initiating payment of 1000 INR for Aditya.
[BankingSystem-Razorpay] Processing payment of 1000...
[Paytm] Confirming payment for Aditya.
Result: SUCCESS
------------------------------

Processing via Razorpay
------------------------------
[Razorpay] Validating payment for Shubham.
[Razorpay] Initiating payment of 500 USD for Shubham.
[BankingSystem-Razorpay] Processing payment of 500...
[Razorpay] Confirming payment for Shubham.
Result: SUCCESS
------------------------------
```
