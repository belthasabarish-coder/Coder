# SAP Banking and Treasury Management - End-to-End Test Case Scenarios

This document provides comprehensive end-to-end test case scenarios for SAP Banking and Treasury Management in SAP S/4HANA and ECC environments.

---

## Table of Contents
1. [Module Overview](#module-overview)
2. [Test Scenarios Directory](#test-scenarios-directory)
   - [Section 1: Bank Account Management (BAM)](#section-1-bank-account-management-bam)
   - [Section 2: Cash & Liquidity Management](#section-2-cash--liquidity-management)
   - [Section 3: Bank Communication Management (BCM) & Payment Processing](#section-3-bank-communication-management-bcm--payment-processing)
   - [Section 4: Electronic Bank Statement (EBS) & Reconciliation](#section-4-electronic-bank-statement-ebs--reconciliation)
   - [Section 5: Treasury and Risk Management (TRM) - Money Market](#section-5-treasury-and-risk-management-trm---money-market)
   - [Section 6: Treasury and Risk Management (TRM) - Foreign Exchange (FX)](#section-6-treasury-and-risk-management-trm---foreign-exchange-fx)
   - [Section 7: In-House Cash (IHC)](#section-7-in-house-cash-ihc)
   - [Section 8: Accounting, Valuation, and Month-End Closing](#section-8-accounting-valuation-and-month-end-closing)

---

## Module Overview

| Module Abbreviation | Full Name / Description |
| :--- | :--- |
| **BAM** | Bank Account Management (SAP S/4HANA Cash Management) |
| **CLM** | Cash & Liquidity Management |
| **BCM** | Bank Communication Management |
| **EBS** | Electronic Bank Statement (MT940, CAMT.053, BAI2) |
| **TRM-MM** | Treasury & Risk Management - Money Market |
| **TRM-FX** | Treasury & Risk Management - Foreign Exchange |
| **IHC** | In-House Cash |
| **TRM-ACC** | Treasury Accounting & Valuation |

---

## Test Scenarios Directory

### Section 1: Bank Account Management (BAM)

#### Test Case ID: TC-BAM-001
* **Module:** Bank Account Management (BAM)
* **Scenario:** Create a New House Bank Account with Workflow Approval
* **Description:** Verify the end-to-end process of requesting, approving, and creating a new House Bank Account in SAP Fiori / BAM.
* **Prerequisites:**
  - FI House Bank master record exists (e.g., House Bank `BNK01`).
  - Workflow setup for BAM approvals configured.
* **Test Steps:**
  1. Log into SAP Fiori Launchpad as Cash Manager.
  2. Open App **Manage Bank Accounts**.
  3. Choose **Create** -> Enter Bank, House Bank, Account Number, IBAN, Currency (USD), and Company Code (`1000`).
  4. Submit the account creation request.
  5. Log in as Treasury Approver / CFO.
  6. Open App **My Inbox** and approve the creation request.
  7. Return to **Manage Bank Accounts** as Cash Manager and verify the GL Account assignment (Connectivity Path / House Bank Account Key).
* **Expected Result:**
  - Account status changes from `In Approval` to `Active`.
  - House Bank Account Key is linked to the assigned GL Account in Universal Journal (ACDOCA).

---

### Section 2: Cash & Liquidity Management

#### Test Case ID: TC-CLM-001
* **Module:** Cash Management (CLM)
* **Scenario:** Cash Position Analysis & Intra-day Cash Pooling / Sweep
* **Description:** Test daily cash position visibility and execute an automatic cash concentration / sweep between subsidiary and header bank accounts.
* **Prerequisites:**
  - Bank Accounts created with assigned memo records / opening balances.
  - Cash concentration hierarchy set up in SAP.
* **Test Steps:**
  1. Open Fiori App **Cash Position Overview** / **Short-Term Cash Position**.
  2. Filter by Company Code `1000` and `2000`, Currency `EUR`.
  3. Observe deficit in Subsidiary Account (`2000`) and surplus in Header Account (`1000`).
  4. Navigate to App **Make Bank Transfers** or **Concentrate Cash**.
  5. Select the proposed sweep transfer proposal from Subsidiary to Header Account.
  6. Execute transfer proposal and generate payment request.
* **Expected Result:**
  - Payment proposal creates internal bank transfer posting.
  - Updated **Cash Position Overview** reflects adjusted liquidity across both accounts.

---

### Section 3: Bank Communication Management (BCM) & Payment Processing

#### Test Case ID: TC-BCM-001
* **Module:** Bank Communication Management (BCM) / FI-AP
* **Scenario:** Automatic Payment Run (F110) with BCM Approval & Payment Medium Workbench (PMW) XML Generation
* **Description:** Execute payment run, route payment batch through BCM approval workflow, and generate ISO 20022 `pain.001` XML file.
* **Prerequisites:**
  - Open Vendor Invoices due for payment in Company Code `1000`.
  - BCM multi-step approval rules configured based on threshold amounts.
* **Test Steps:**
  1. Run Transaction `F110` (or Fiori App **Manage Payments**) for Parameter Date & Run ID.
  2. Execute Proposal and Verify Payment Proposal items.
  3. Execute Payment Run.
  4. Log into App **Approve Payments** (BCM Workflow).
  5. Review Payment Batch (Batch ID generated). Approve payment batch level 1 and level 2.
  6. Run Transaction `BNK_CJ3` / PMW output to create `pain.001.001.03` XML file.
* **Expected Result:**
  - Payment document created (`Clearing Document`).
  - Batch status updated to `Approved` in BCM Monitor (`BNK_MONI`).
  - Valid ISO 20022 XML `pain.001` file generated for bank transmission.

---

### Section 4: Electronic Bank Statement (EBS) & Reconciliation

#### Test Case ID: TC-EBS-001
* **Module:** Electronic Bank Statement (EBS)
* **Scenario:** Processing MT940 / CAMT.053 Statement and Automatic Clearing
* **Description:** Upload electronic bank statement file, process search strings, and verify posting to main bank account and clearing accounts.
* **Prerequisites:**
  - Posting rules and search strings configured for EBS transaction codes (e.g., NTRF, NCHK).
  - Open items exist in Subledger / Bank Clearing GL Account.
* **Test Steps:**
  1. Access Transaction `FF_5` or Fiori App **Process Bank Statement Files**.
  2. Select Statement Format: `CAMT.053` or `MT940`.
  3. Upload sample statement file containing incoming customer payment and outgoing vendor wire clearing.
  4. Execute statement processing in foreground/background.
  5. Open Transaction `FEBAN` / App **Reprocess Bank Statement Items**.
* **Expected Result:**
  - **Posting Area 1:** Dr Main Bank GL / Cr Bank Clearing GL.
  - **Posting Area 2:** Dr Bank Clearing GL / Cr Customer Account (with document clearing).
  - Statement status shows `Completed` / Green for matched items.

---

### Section 5: Treasury and Risk Management (TRM) - Money Market

#### Test Case ID: TC-TRM-001
* **Module:** TRM - Money Market
* **Scenario:** Fixed-Term Deposit Deal Creation, Settlement, and Interest Accrual
* **Description:** End-to-end deal lifecycle for a Fixed-Term Deposit investment including deal capture, settlement, interest calculation, and accounting release.
* **Prerequisites:**
  - Business Partner (Bank) created with Treasury roles (`TR0600`).
  - Product Type `51A` (Fixed-Term Deposit) configured.
* **Test Steps:**
  1. Execute Transaction `FTR_CREATE` (or App **Create Financial Transaction**).
  2. Select Product Type `51A`, Transaction Type `100` (Investment), Business Partner `BANK_US_01`.
  3. Enter Amount `$1,000,000`, Rate `5.00%`, Start Date & End Date (3 Months maturity). Save Deal (ID generated, e.g., `10024`).
  4. Execute Transaction `FTR_EDIT` -> Choose **Settle** -> Save deal settlement.
  5. Run Transaction `TBB1` (Post Flows) to execute principal outflow posting.
  6. At Month-End, run Transaction `TPM44` (Accrual/Deferral) for interest accrual.
  7. On maturity date, run Transaction `TBB1` for principal repayment + interest inflow.
* **Expected Result:**
  - `TBB1` posts Dr Investment Account / Cr Main Bank Account.
  - `TPM44` posts Dr Interest Receivable / Cr Interest Income.
  - Maturity `TBB1` clears investment account and posts total cash received.

---

### Section 6: Treasury and Risk Management (TRM) - Foreign Exchange (FX)

#### Test Case ID: TC-TRM-002
* **Module:** TRM - Foreign Exchange
* **Scenario:** FX Forward Contract Creation, Confirmation, Settlement, and Mark-to-Market Valuation
* **Description:** Create an FX Spot/Forward deal to hedge currency exposure, perform bank confirmation matching, and run Mark-to-Market (MTM) valuation.
* **Prerequisites:**
  - Product Type `60A` (FX Forward) configured.
  - Daily Exchange Rates & Yield Curves updated in SAP Market Data (`OB08` / `TEX1`).
* **Test Steps:**
  1. Open `FTR_CREATE` -> Select Product Type `60A`, Transaction Type `102` (FX Forward).
  2. Input Buy Currency `EUR 500,000`, Sell Currency `USD 550,000`, Forward Rate `1.1000`, Value Date = Spot + 30 Days. Save Deal.
  3. Settle deal via `FTR_EDIT`.
  4. Perform MT300 confirmation matching via App **Manage Correspondence**.
  5. Perform Period-End Mark-to-Market (MTM) Valuation using Transaction `TPM1`.
* **Expected Result:**
  - Financial deal created with correct cash flows.
  - `TPM1` calculates gain/loss based on forward curve rates and posts unrealized FX gain/loss to Financial Accounting (FI).

---

### Section 7: In-House Cash (IHC)

#### Test Case ID: TC-IHC-001
* **Module:** In-House Cash (IHC)
* **Scenario:** Intercompany Internal Payment via In-House Cash Bank
* **Description:** Process zero-cost internal payment between two subsidiaries within the group using In-House Cash internal bank accounts.
* **Prerequisites:**
  - IHC Center setup with internal bank accounts for Subsidiary A (`Comp 1000`) and Subsidiary B (`Comp 2000`).
* **Test Steps:**
  1. Subsidiary A posts vendor invoice payable to Subsidiary B.
  2. Run payment program `F110` in Subsidiary A using IHC payment method.
  3. IDoc generated from `F110` sent to IHC Center.
  4. Run Transaction `IHC_APPL` in IHC Center to process incoming IDoc and generate internal IHC payment order.
  5. Post internal payment order.
* **Expected Result:**
  - Subsidiary A IHC account debited; Subsidiary B IHC account credited.
  - Intercompany vendor & customer open items cleared without external bank fees.

---

### Section 8: Accounting, Valuation, and Month-End Closing

#### Test Case ID: TC-TRM-ACC-001
* **Module:** Treasury Accounting & Valuation
* **Scenario:** Period-End Foreign Currency Revaluation and Position Valuation
* **Description:** Run period-end valuation for all open Treasury deals across Money Market and FX, posting valuation results to General Ledger under IFRS / US GAAP rules.
* **Prerequisites:**
  - Accounting Structure & Valuation Area setup (e.g., `001` US GAAP, `002` IFRS).
  - Exchange rates & interest rate curves maintained for period-end date.
* **Test Steps:**
  1. Run Transaction `TPM1` (Execute Position Valuation).
  2. Enter Company Code `1000`, Valuation Area `001`, Key Date (e.g., `2025-03-31`).
  3. Run in Test Mode first to review calculated fair values and exchange rate gains/losses.
  4. Remove Test Mode flag and Execute in Production Mode.
  5. Run Transaction `TPM20` (Posting Journal) to review posted FI accounting documents in Universal Journal (`ACDOCA`).
* **Expected Result:**
  - Valuation entries posted correctly: Dr/Cr Financial Asset Fair Value Adjustment vs. Dr/Cr P&L / OCI.
  - FI document linked to Treasury Position ID.

---
