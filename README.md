#  Credit Score Computation using Aave Subgraph Data

This project calculates credit scores for blockchain wallet addresses by analyzing on-chain financial behavior using data from an Aave subgraph. The credit score is computed based on deposit, borrow, and repay activity.

---

##  Data Collection Method

We use **The Graph Protocol** to access on-chain financial data stored in an **Aave Lending Subgraph**. Specifically:

- The subgraph is queried using the [Subgrounds](https://docs.streamingfast.io/subgrounds/) Python library.
- We collect data on three key financial actions:
  - **Deposits**
  - **Borrows**
  - **Repays**
- Each query fetches up to 5000 records per entity (with support for pagination if needed).
- Wallet addresses are identified using `account.id` in the subgraph schema.

---

##  Feature Selection Rationale

We selected features based on their relevance to risk and creditworthiness:

| Feature           | Description                                               | Reason for Selection                                 |
|------------------|-----------------------------------------------------------|------------------------------------------------------|
| `total_deposited`| Total amount deposited by the wallet                      | Indicates capital inflow and participation           |
| `total_borrowed` | Total amount borrowed by the wallet                       | Core component for credit scoring                    |
| `total_repaid`   | Total amount repaid by the wallet                         | Directly reflects repayment behavior                 |

These three features represent the **financial discipline and risk exposure** of a wallet in decentralized finance (DeFi).

---

##  Scoring Method

We compute the credit score on a scale of **0 to 1000**, using the following formula:

```python
if total_borrowed == 0:
    credit_score = 0.0
else:
    repay_ratio = total_repaid / total_borrowed
    credit_score = round(min(1.0, repay_ratio) * 1000, 2)
```

- **Repay ratio** is used to assess how responsibly a wallet repays its borrowed amount.
- A score of 1000 means the user has repaid their full borrowed amount (or more).
- A score of 0 means no repayment activity despite borrowing (or no borrowing at all).

---

##  Justification of Risk Indicators

These indicators were chosen based on common credit risk principles in both traditional and DeFi lending:

1. **Borrow Amount (`total_borrowed`)**:
   - Higher borrowed amounts reflect greater exposure.
   - Helps gauge if a user is actively engaging with the protocol.

2. **Repayment Amount (`total_repaid`)**:
   - Measures the wallet’s ability and intention to repay debt.
   - A key signal of creditworthiness and low default risk.

3. **Deposit Amount (`total_deposited`)**:
   - Indicates liquidity backing and potential to collateralize.
   - While not directly part of the scoring formula, it can be used as a **supportive metric** for deeper credit analysis.

---

##  Output

A CSV file named `credit_scores.csv` is generated with the following structure:

| wallet | total_deposited | total_borrowed | total_repaid | credit_score |
|--------|------------------|----------------|--------------|---------------|
| 0x...  | 123.45           | 67.89          | 67.89        | 1000.0        |
| 0x...  | 100.00           | 80.00          | 40.00        | 500.0         |

---

##  Future Improvements

- Implement full **pagination** to fetch data beyond the 5000-record limit.
- Introduce additional features such as **collateral type**, **loan duration**, or **liquidation events**.
- Normalize scores across market conditions and asset volatility.
