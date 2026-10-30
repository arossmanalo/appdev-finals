# Test Cases

**Project:** Nerriza Transport Services - Client Billing and Collections robot
**What the robot does:** reads the client billing sheet and the payment policy PDF, works out each client's bill, decides whether it is Paid, Partially Paid, Unpaid or Overdue, emails the clients, saves a processed Excel file, and emails a summary to the Finance Head/CFO.

---

## How to use this document

1. Finish everything in `1_Setup_Before_Testing.md` first.
2. Do the tests in order, from **TC-01** to **TC-17**.
3. For every test, follow the **Steps**, then compare what happens with **You should see**.
4. Write **Pass** if everything matches, or **Fail** if anything is different. Add a short note about what was different.
5. Take the screenshot mentioned under **Screenshot**.

### Words used in this document

| Word | Meaning |
|---|---|
| **As-of date** | The "today" date the robot uses to decide if an invoice is late. All tests use **10/30/2026** unless the test says otherwise. |
| **Skip / Yes** | The question "Send follow-up reminders to clients already emailed?" Skip = do not email them again. Yes = send them a reminder. |
| **Overdue** | The due date is before the as-of date and the client still owes money. |
| **Paid** | The client owes nothing (balance is 0). No email is sent. |
| **Partially Paid** | The client paid some money but not all, and the invoice is not late yet. |
| **Unpaid** | The client paid nothing and the invoice is not late yet. |
| **Invalid** | The row has bad data (a blank or wrong Parcels, Rate or Invoice Date). The robot skips it. |
| **Output panel** | The panel at the bottom of UiPath Studio where the robot prints one line for each invoice and each email. |
| **Processed sheet** | The sheet named **Processed** in `Output\BillingData_Processed.xlsx`. This is where the robot saves its results. |

---

## Quick reference: the right answers

Use this table to check the robot's work. It shows what should happen to each invoice on **10/30/2026**.

| Invoice | Client | Total Billed | Amount Paid | Balance | Status | What the robot does |
|---|---|---:|---:|---:|---|---|
| INV-001 | C001 | 9,520.00 | 0.00 | 9,520.00 | **Overdue** | Sends an overdue reminder |
| INV-002 | C002 | 6,720.00 | blank (counts as 0) | 6,720.00 | **Unpaid** | Sends a billing statement |
| INV-003 | C003 | 8,960.00 | 8,960.00 | 0.00 | **Paid** | No email |
| INV-004 | C004 | 12,096.00 | 6,000.00 | 6,096.00 | **Overdue** | Sends an overdue reminder |
| INV-005 | C005 | 7,392.00 | 3,000.00 | 4,392.00 | **Partially Paid** | Sends a billing statement |
| INV-006 | C006 | - | - | - | **Invalid** (Parcels is blank) | Skips it, no email |
| INV-007 | C007 | - | - | - | **Invalid** (Invoice Date is blank) | Skips it, no email |
| INV-008 | C001 | 4,256.00 | 0.00 | 4,256.00 | Overdue | Already emailed before. **Skip** = ignore it. **Yes** = send a follow-up |
| INV-009 | C002 | 3,360.00 | 0.00 | 3,360.00 | Unpaid | Already emailed before. **Skip** = ignore it. **Yes** = send a follow-up |
| INV-010 | C003 | 8,232.00 | 0.00 | 8,232.00 | **Overdue** | Sends an overdue reminder |
| INV-011 | C004 | - | - | - | **Invalid** (Rate is "N/A") | Skips it, no email |

**How the amounts are worked out** (example INV-001: 100 parcels at 85 each):
- Gross = parcels x rate = 100 x 85 = **8,500.00**
- VAT = 12% of gross = **1,020.00**
- Total billed = gross + VAT = **9,520.00**
- Due date = invoice date + payment terms (30 days) = **10/01/2026**
- Balance = total billed - amount paid = **9,520.00**

---

# Part 1: Normal runs

## TC-01 Full run, skipping follow-ups

**Goal:** Check that the robot works from start to finish when follow-ups are skipped.

**Steps**
1. Open `Main.xaml` and press **F5**.
2. As-of date: type `10/30/2026`.
3. Follow-up question: choose **Skip**.
4. Type the sender Gmail, the App Password and the CFO email.
5. Wait for the robot to finish. Click **OK** on each message box.

**You should see**
- A message box **"Email Sent!"** (the summary email).
- A message box **"Run Summary"** with these numbers:

| Item | Number |
|---|---:|
| Processed | 6 |
| Paid | 1 |
| Partially Paid | 1 |
| Unpaid | 1 |
| Overdue | 3 |
| Invalid | 3 |
| Skipped (already emailed) | 2 |
| Emails sent | 5 |
| Emails failed | 0 |
| Total billed | 52,920.00 |
| Total paid | 17,960.00 |
| Total balance | 34,960.00 |

- In the inbox: **5 client emails** (3 overdue reminders for INV-001, INV-004, INV-010 and 2 billing statements for INV-002, INV-005) and **1 summary email**.
- The file `Output\BillingData_Processed.xlsx` exists.

**Screenshot:** the Run Summary box, the Output panel, and the inbox.

**Result:** Pass / Fail  **Notes:** ______________________

---

## TC-02 Full run, sending follow-ups

**Goal:** Check that clients who were already emailed get a follow-up reminder when you choose Yes.

**Steps**
1. Close `BillingData_Processed.xlsx` if it is open.
2. Run `Main` again with as-of date `10/30/2026`.
3. This time choose **Yes** for the follow-up question.

**You should see**
- Run Summary: Processed **8**, Paid 1, Partially Paid 1, Unpaid **2**, Overdue **4**, Invalid 3, Skipped **0**.
- Emails sent **7**. Total billed **60,536.00**, total paid 17,960.00, total balance **42,576.00**.
- In the inbox: the same 5 emails as TC-01, plus **2 follow-up reminders** (for INV-008 and INV-009), plus the summary.
- In the Processed sheet:
  - INV-008: Email Status **Sent**, Follow-up Count **2** (it was 1), Last Reminder Date **10/30/2026**.
  - INV-009: Email Status **Sent**, Follow-up Count **1** (it was 0), Last Reminder Date **10/30/2026**.

**Screenshot:** the Run Summary box and the INV-008 and INV-009 rows of the Processed sheet.

**Result:** Pass / Fail  **Notes:** ______________________

---

## TC-03 A different as-of date

**Goal:** Check that the "late" decision depends on the as-of date.

**Steps**
1. Run `Main` with as-of date **11/15/2026** and choose **Skip**.

**You should see**
- INV-005 (due 11/14/2026) is now **Overdue**, because the due date is before 11/15/2026.
- INV-002 (due 11/19/2026) is still **Unpaid**.
- Run Summary: Processed 6, Paid 1, Partially Paid **0**, Unpaid 1, Overdue **4**, Invalid 3, Skipped 2.

**Result:** Pass / Fail  **Notes:** ______________________

---

# Part 2: Checking the results of one run

Do these checks on the files and emails from **TC-01**. No new run is needed.

## TC-04 The Processed sheet is correct

**Goal:** Check the numbers and tracking columns saved in Excel.

**Steps**
1. Open `Output\BillingData_Processed.xlsx` and go to the **Processed** sheet.
2. Compare each row with the table in "Quick reference".

**You should see**
- The columns **Gross, VAT, Total Billed, Due Date, Balance and Status** are filled in for every valid row.
- For the three overdue reminders (INV-001, INV-004, INV-010): Email Status **Sent**, Follow-up Count **0**, Last Reminder Date **10/30/2026**.
- For the two billing statements (INV-002, INV-005): Email Status **Sent**, Follow-up Count **0**, Last Reminder Date **blank**.
- For INV-003: Status **Paid**, Email Status blank.

**Screenshot:** the whole Processed sheet.

**Result:** Pass / Fail  **Notes:** ______________________

---

## TC-05 A blank "Amount Paid" counts as zero

**Goal:** Check that a blank Amount Paid does not cause an error.

**Steps**
1. Look at INV-002 in the Processed sheet (its Amount Paid cell is blank).
2. Open the email the robot sent for INV-002.

**You should see**
- Status **Unpaid** and Balance **6,720.00** (the same as Total Billed).
- The email has **no** "Amount paid" line.

**Result:** Pass / Fail  **Notes:** ______________________

---

## TC-06 Bad rows are skipped

**Goal:** Check that rows with bad data are marked Invalid and do not stop the robot.

**Steps**
1. Look at INV-006, INV-007 and INV-011 in the Processed sheet.
2. Check the inbox for emails about these invoices.

**You should see**
- Status **Invalid** on all three rows.
- No amounts filled in, and **no emails** for them.
- The robot still processed the rows after them (for example INV-008 and INV-010).

**Result:** Pass / Fail  **Notes:** ______________________

---

## TC-07 A paid invoice gets no email

**Goal:** Check that clients who already paid are not emailed.

**Steps**
1. Look at INV-003 in the Processed sheet.
2. Search the inbox for "INV-003".

**You should see**
- Status **Paid** and Balance **0.00**.
- **No** email about INV-003.

**Result:** Pass / Fail  **Notes:** ______________________

---

# Part 3: Reading the emails

Open the emails in the inbox from TC-01 (and TC-02 for the follow-up).

## TC-08 Billing statement (INV-002 and INV-005)

**You should see**
- Subject: **Billing Statement - INV-xxx - Nerriza Transport Services**
- The email starts with "Dear" and the contact person's name.
- It lists: Invoice ID, Total billed (including 12% VAT), Remaining balance, Due date.
- **INV-005 only:** also "Amount paid: PHP 3,000.00" and "Remaining balance: PHP 4,392.00".
- The full **Billing Terms and Payment Policy** text from the PDF is at the bottom.

**Screenshot:** the INV-005 email.

**Result:** Pass / Fail  **Notes:** ______________________

## TC-09 Overdue reminder (INV-001, INV-004, INV-010)

**You should see**
- Subject starts with **OVERDUE - Payment Reminder for INV-xxx**
- The email says how many days the invoice is late:

| Invoice | Days late on 10/30/2026 |
|---|---:|
| INV-001 | 29 |
| INV-004 | 25 |
| INV-010 | 25 |

- **INV-004 only:** also "Amount paid: PHP 6,000.00" and "Remaining balance: PHP 6,096.00".
- The policy text is at the bottom.

**Screenshot:** the INV-004 email.

**Result:** Pass / Fail  **Notes:** ______________________

## TC-10 Follow-up reminder (INV-008 and INV-009, from TC-02 only)

**You should see**
- Subject: **Follow-up Reminder - INV-xxx - Nerriza Transport Services**
- The email says it is a follow-up about an invoice sent earlier that still has an unpaid balance.
- The policy text is at the bottom.

**Result:** Pass / Fail  **Notes:** ______________________

## TC-11 Summary email to the Finance Head/CFO

**You should see**
- Subject: **Client Billing and Collections Summary - as of October 30, 2026**
- The body lists the invoice counts, the amounts in PHP, and how many client emails were sent and failed. The numbers match the Run Summary box.
- The file **BillingData_Processed.xlsx** is attached and opens correctly.

**Screenshot:** the summary email with its attachment.

**Result:** Pass / Fail  **Notes:** ______________________

---

# Part 4: When something goes wrong

These tests check that the robot handles problems politely instead of crashing. **Put everything back to normal after each test.**

## TC-12 A wrong date is typed

**Steps**
1. Run `Main`.
2. At the first dialog, type `abc` and click OK.
3. Then type `10/30/2026`.

**You should see**
- A message: **"That is not a valid date. Please try again."**
- The date dialog comes back. After the correct date, the run continues normally.

**Result:** Pass / Fail  **Notes:** ______________________

## TC-13 The user clicks Cancel

**Steps**
1. Run `Main`.
2. On any input dialog, click **Cancel**.

**You should see**
- A message: **"The run inputs were cancelled, so the automation will stop."**
- The robot ends. No emails are sent.

**Result:** Pass / Fail  **Notes:** ______________________

## TC-14 The billing file is missing

**Steps**
1. In the `Data` folder, rename `BillingData.xlsx` to `BillingData_x.xlsx`.
2. Run `Main` and answer the dialogs.
3. **Rename the file back** when done.

**You should see**
- A message: **"Billing file not found"**.
- The robot stops. No emails are sent.

**Screenshot:** the message box.

**Result:** Pass / Fail  **Notes:** ______________________

## TC-15 The policy PDF is missing

**Steps**
1. In the `Data` folder, rename `BillingPolicy.pdf` to `BillingPolicy_x.pdf`.
2. Run `Main` and answer the dialogs.
3. **Rename the file back** when done.

**You should see**
- A message: **"Policy PDF not found"**.
- The robot stops. No emails are sent.

**Screenshot:** the message box.

**Result:** Pass / Fail  **Notes:** ______________________

## TC-16 The App Password is wrong

**Steps**
1. Run `Main`. Type a wrong App Password (for example `wrongpassword`).

**You should see**
- The robot does **not** crash.
- The Output panel shows lines such as **"Email FAILED for INV-001: ..."** for each invoice that needed an email.
- The Run Summary shows **Emails sent 0** and **Emails failed 5** (with Skip).
- A message says the summary email could not be sent.
- In the Processed sheet, the **Email Status** of those rows stays **blank**, because nothing was sent.

**Result:** Pass / Fail  **Notes:** ______________________

## TC-17 The processed file is open in Excel

**Steps**
1. Open `Output\BillingData_Processed.xlsx` in Excel and leave it open.
2. Run `Main` with correct details.

**You should see**
- A message: **"The processed workbook could not be saved. Close it if it is open in Excel and run again."**
- The robot stops before the summary email.

**Result:** Pass / Fail  **Notes:** ______________________

---

# Things that are not bugs

- The robot **never changes** `Data\BillingData.xlsx`. It saves its results in a new file. If you run it twice, the same clients get the same emails twice.
- Only INV-008 and INV-009 count as "already emailed", because the sample data says so.
- A message box stops the robot until you click **OK**.

---

# Results summary

Fill this in when you finish. A project is ready to present when every test says **Pass**.

| Test | What it checks | Result | Date | Tester |
|---|---|---|---|---|
| TC-01 | Full run, Skip | | | |
| TC-02 | Full run, Yes | | | |
| TC-03 | Different as-of date | | | |
| TC-04 | Processed sheet is correct | | | |
| TC-05 | Blank Amount Paid | | | |
| TC-06 | Bad rows skipped | | | |
| TC-07 | Paid invoice gets no email | | | |
| TC-08 | Billing statement email | | | |
| TC-09 | Overdue reminder email | | | |
| TC-10 | Follow-up reminder email | | | |
| TC-11 | Summary email | | | |
| TC-12 | Wrong date typed | | | |
| TC-13 | Cancel clicked | | | |
| TC-14 | Billing file missing | | | |
| TC-15 | Policy PDF missing | | | |
| TC-16 | Wrong App Password | | | |
| TC-17 | Processed file open | | | |
