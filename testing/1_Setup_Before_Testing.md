# Setup Before Testing

Project: **Nerriza Transport Services - Client Billing and Collections**
Automation types: Excel, PDF and Email

Complete every item below **before** the first full run of `Main.xaml`. Tick the boxes as you go.

---

## 1. Computer and Studio

- [ ] UiPath Studio is open and the project `NerrizaBillingAutomation` is loaded (`project.json`).
- [ ] **Show Classic** is enabled in the Activities panel (so the classic Excel Application Scope, Read Range, Write Range and Send SMTP Mail Message are visible).
- [ ] Packages restored with no errors (Manage Packages): `UiPath.System.Activities`, `UiPath.Excel.Activities`, `UiPath.PDF.Activities`, `UiPath.Mail.Activities`, `UiPath.UIAutomation.Activities`.
- [ ] **Microsoft Excel (desktop version) is installed and activated.** The classic Excel Application Scope needs it.
- [ ] No leftover Excel processes (Task Manager > end every "Microsoft Excel").
- [ ] Internet connection works (Gmail SMTP uses `smtp.gmail.com`, port 465).
- [ ] Every `.xaml` file opens without red error marks: `Main`, `GetRunInputs`, `ReadBillingData`, `ReadBillingPolicy`, `ProcessInvoices`, `SendClientEmail`, `SaveProcessedWorkbook`, `SendSummaryEmail`.

---

## 2. Gmail sender account (needed for sending emails)

The robot logs in to Gmail with an **App Password**, not your normal password. Use a throwaway test Gmail if you can.

- [ ] Choose the sender Gmail address: `____________________@gmail.com`
- [ ] Turn on **2-Step Verification** (Google Account > Security).
- [ ] Create an **App Password** (Google Account > Security > App passwords). Copy the 16 characters.
- [ ] Keep the App Password somewhere private. **Never** write it into the project files. The robot asks for it in a masked dialog at the start of every run.
- [ ] Open the sender inbox in a browser so you can check the sent emails.

> If you get a login error during the run, the App Password is wrong or 2-Step Verification is off.

---

## 3. Test email addresses (never email real clients)

- [ ] Open `Data\BillingData.xlsx`, sheet **Client_Contacts**.
- [ ] Replace `YOURNAME` in all 7 email cells with your Gmail name, using plus addresses, for example:

| Client ID | Email to type |
|---|---|
| C001 | `yourname+client1@gmail.com` |
| C002 | `yourname+client2@gmail.com` |
| C003 | `yourname+client3@gmail.com` |
| C004 | `yourname+client4@gmail.com` |
| C005 | `yourname+client5@gmail.com` |
| C006 | `yourname+client6@gmail.com` |
| C007 | `yourname+client7@gmail.com` |

- [ ] Save and **close** the workbook (the robot cannot read it while it is open in Excel).
- [ ] Decide the **Finance Head/CFO test address** you will type at run time, for example `yourname+cfo@gmail.com`: `____________________`

Plus addresses all arrive in the same inbox, so you can read every email in one place.

---

## 4. Test data

- [ ] `Data\BillingData.xlsx` has two sheets: **Client_Billing** (11 invoices, 17 columns) and **Client_Contacts** (7 clients).
- [ ] `Data\BillingPolicy.pdf` exists (one page, "Billing Terms and Payment Policy").
- [ ] Make a **backup copy** of `BillingData.xlsx` (for example `BillingData_ORIGINAL.xlsx`, kept **outside** the `Data` folder). If you change the data while testing, you can restore it.
- [ ] The `Output` folder exists. The robot writes `PolicyText.txt` and `BillingData_Processed.xlsx` there.
- [ ] `Output\BillingData_Processed.xlsx` is **closed** in Excel before every run (the robot overwrites it).

### The "as of" date to use

Use **10/30/2026** (October 30, 2026). The sample invoice dates were chosen so that some invoices are already overdue on this date.

---

## 5. Workflow settings to check

- [ ] `Main.xaml` is the entry point (Project panel shows Main in bold).
- [ ] Optional cleanup: in **ReadBillingData** and **ReadBillingPolicy**, clear any default values you typed earlier in the Arguments panel for `in_WorkbookPath` and `in_PdfPath`. `Main` now passes the paths itself (`Data\BillingData.xlsx` and `Data\BillingPolicy.pdf`).
- [ ] Optional: remove the unused `UiPath.Testing.Activities` package (Manage Packages).

---

## 6. How to run

1. Open `Main.xaml` and press **F5** (Debug).
2. Answer the five dialogs in this order:
   1. **As-of date** - type `10/30/2026`
   2. **Send follow-up reminders to clients already emailed?** - choose **Skip** or **Yes**
   3. **Sender Gmail** - the sender address from step 2
   4. **Gmail App Password** - the 16-character code
   5. **Finance Head / CFO email** - the CFO test address from step 3
3. Watch the **Output panel** (one line per invoice and per email) and the message boxes at the end:
   - "Email Sent!" (summary email)
   - "Run Summary" (counts and totals)

---

## 7. Things to know before you test

- The source file `Data\BillingData.xlsx` is **not changed** by the robot. The results are written to a new workbook, `Output\BillingData_Processed.xlsx` (sheet **Processed**).
- Because the source file is not updated, "already emailed" rows (Email Status) are only the ones that were **already marked in the sample data** (INV-008 and INV-009). Running the robot twice will email the same clients twice.
- Invalid rows (INV-006, INV-007, INV-011) get Status **Invalid** and no email.
- A paid invoice (INV-003) gets Status **Paid** and no email.
- Emails go out with the **sender Gmail** you type at run time. The policy text from the PDF is inside every client email.

---

## 8. Screenshots to take for the documentation

| # | What to capture | What it shows |
|---|---|---|
| 1 | `Main.xaml` flowchart (whole canvas) | The overall process and the decisions that stop the run |
| 2 | Each workflow opened in the designer (8 total) | The structure and the annotations that explain each step |
| 3 | Arguments panel of `ProcessInvoices` | Argument names and directions (`in_`, `out_`, `io_`) |
| 4 | Variables panel of `Main` | UpperCamelCase variable names |
| 5 | The five input dialogs | Usability of the start of the run |
| 6 | Output panel after a good run | One result line per invoice and per email |
| 7 | Each email type in the inbox | Billing statement, overdue reminder, follow-up reminder |
| 8 | The summary email with its attachment | Counts, totals and the attached workbook |
| 9 | `BillingData_Processed.xlsx`, sheet Processed | The computed columns and statuses |
| 10 | Each error message box | Reliability: missing billing file, missing PDF, cancelled dialog |
