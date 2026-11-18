# Personal Finance Analyser

A free, self-hosted monthly expense analyser. You add your bank statements, and it shows **Income vs Expenditure**, **where the money goes** and **how much you save**. It **learns your categories** as you go, and its **Dream Planner** tells you how many months each goal will take.

- One file: `index.html` (plain HTML, CSS and JavaScript — no server, no database)
- Your data is saved as JSON files in **your own OneDrive**, using Microsoft login and the Microsoft Graph API
- Costs ₹0: free hosting on GitHub Pages and the free OneDrive storage you already have
- No paid AI. Categories are learned with local rules and fuzzy name matching

---

## 1. Try it right away (no setup)

Double-click `index.html`. It opens in your browser and works fully. Data is **saved in this browser only** until you connect OneDrive (step 4).

## 2. Put it online for free (GitHub Pages)

1. Create a free account at https://github.com and click **New repository**. Name it e.g. `finance` and set it to **Private** or Public. The page link works either way, but anyone with the link can open the empty app. Your data stays private in your OneDrive.
2. Click **Add file → Upload files** and upload `index.html` (and this README). Click **Commit**.
3. Go to **Settings → Pages**. Under *Build and deployment*, pick **Deploy from a branch**, branch **main**, folder **/ (root)**, then **Save**.
4. After about a minute your app is at:
   `https://<your-github-username>.github.io/finance/`
   Copy this address. You need it in the next step.

> GitHub Pages on a private repository needs a paid GitHub plan. On the free plan, keep the repository public. Only the app code is public. Your statements and data never go to GitHub.

## 3. Register the app with Microsoft (free, one time, about 5 minutes)

This gives the app a **Client ID**, so it can ask you to sign in and save to your OneDrive.

1. Open https://entra.microsoft.com and sign in with the Microsoft account whose OneDrive you want to use (for example your Outlook/Hotmail account).
   If it says you have no directory or tenant, create a free Azure account at https://azure.microsoft.com/free. The app registration itself is free.
2. Go to **Identity → Applications → App registrations → New registration**.
   - **Name:** `Personal Finance Analyser`
   - **Supported account types:** *Accounts in any organizational directory and personal Microsoft accounts*
   - **Redirect URI:** choose **Single-page application (SPA)** and paste your GitHub Pages address exactly, including the last `/`, e.g. `https://devashish.github.io/finance/`
   - Click **Register**.
3. On the Overview page, copy the **Application (client) ID**. It looks like `1a2b3c4d-....`
4. (Optional) Go to **API permissions → Add a permission → Microsoft Graph → Delegated** and add `Files.ReadWrite` (`User.Read` is already there). If you skip this, Microsoft asks for it the first time you sign in.

## 4. Connect the app to OneDrive

Choose **one** of these:

- **Best (works on every device):** open `index.html` in a text editor and paste the ID near the top:
  ```js
  clientId: '1a2b3c4d-....',
  ```
  Then upload the file to GitHub again.
- **Quick:** open your GitHub Pages link, go to **Settings**, paste the ID and click **Save ID**. This is remembered only in that browser.

Then click **Sign in with Microsoft**. The app creates this folder structure automatically:

```
OneDrive/
└── Users/
    └── Devashish/
        ├── Statements/
        │   ├── statements.json            ← list of imported statements
        │   ├── 2025-08/
        │   │   ├── Acct_Statement_XX2589.pdf   ← your original file
        │   │   └── transactions.json      ← { "month": "2025-08", "transactions": [...] }
        │   └── 2025-09/ ...
        ├── Categories/categorization.json ← categories + what it has learned
        ├── Dreams/dreams.json             ← your goals
        ├── Analysis/
        │   ├── 2025-08.json               ← income, expenditure, savings, by category
        │   └── monthly_summary.json
        └── Settings/user.json             ← your name, accounts, preferences
```

If you already used the app in the browser before signing in, it offers to copy that data to OneDrive.

> **Work account (e.g. company OneDrive)?** Many companies block apps that are not approved by their IT team. Use your personal Microsoft account to avoid that.

---

## How it works

### Adding statements
- **PDF, Excel (.xlsx/.xls) or CSV.** Tested with ICICI (Date / Description / Amount / DR-CR) and HDFC (Withdrawal / Deposit / Balance) PDFs. For other banks, PDFs with a normal table layout usually work. If one doesn't, download the Excel/CSV version from net banking.
- **Password-protected PDFs:** you are asked for the password. It is used only to open the file in your browser and is never saved. If you add several statements from the same bank, it tries the last password first.
- **Check before adding:** you see the account, holder name, period and totals before anything is saved. For HDFC, totals are checked against the bank's own statement summary.
- **Overlapping statements** are fine. A transaction that appears in two files is counted once.

### Only your own statements
- The first statement that shows a holder name sets **your name**.
- After that, a statement in **someone else's name is not accepted**. Middle names and initials may differ. For example, *Devashish S Salvekar* is accepted for *Devashish Shantanu Salvekar*.
- Some statements (like the ICICI one) don't print a name. For those, the app asks you to confirm the account is yours, once per account.
- Other spellings can be added in **Settings → About you**.

### Income, Expenditure, Savings
- **Income** = all credits in the month (salary, interest, refunds, money from family …)
- **Expenditure** = all debits in the month (rent, food, fuel, EMIs, credit-card bills …)
- **Self Transfer** (money between your own accounts) is ignored on both sides, so it isn't counted twice.
- **Investment / SIP** and **FD / RD** count as savings, not spending.
- **Savings = Income − Expenditure**, and **Savings % = Savings ÷ Income**.
- Months only partly covered by your statements are marked with `*` and left out of averages.

### Self-learning categories
1. For an unknown payee the app asks: *"How should we categorise payments to Ashok?"* You click a category.
2. It remembers the payee (UPI ID and name) in `categorization.json`, including a simple map like `{ "ASHOK": "Family Support" }`.
3. Next time, similar names (**ASHOK**, **ASHOK KUMAR**, **ASHOK UPI**) and the same UPI ID are matched with **fuzzy matching**, and a **confidence %** is shown.
4. After **5 confirmations** (you can change this in Settings) that payee is **auto-approved**, with no more questions.
- Common merchants (Swiggy, Zomato, Blinkit, petrol pumps, ParkPlus, Airtel/Jio, Zerodha/Groww, electricity and gas boards, credit-card bills, salary, interest …) are suggested automatically. You confirm them once.
- Payments to your own UPI ID become **Self Transfer**. Payments with your surname are suggested as **Family**.
- The same payee can mean different things depending on the UPI note. For example, *"taigun emi"* is learned as EMI, separately from plain transfers.
- **Keyword rules** (Settings) always win, e.g. `ZOMATO → Food & Dining`.
- Change any category in **Transactions**. That also teaches the app.

### Dashboard
Cards for Income, Expenditure, Savings and Savings %, each compared with last month. It also has an Income vs Expenditure bar chart, a category pie chart, a savings trend line, recurring payments, a month-on-month category table and automatic insights (savings change, investment share, top categories, biggest increase, how much sooner a dream is possible).

### Dream Planner
- Add a goal, e.g. **Buy KTM 390 Adventure – ₹6,50,000**.
- It uses your **actual average monthly saving** from your statements (last 6 full months). You can also plan with your own number.
- **Without a deadline:** ₹6,50,000 ÷ ₹1,00,000 = 6.5 months → *approx 7 months (by month/year)*.
- **With a deadline**, e.g. 5 months: needs ₹1,30,000/month, you save ₹1,00,000 → *additional ₹30,000/month needed*, with suggestions like "Reduce Food & Dining by ₹4,500" taken from your lifestyle categories.
- **Several dreams:** use ↑ ↓ to set priority and see when each one is reached if you fund them one after another.
- **Add savings** to a dream as you put money aside, and mark it **Achieved** when done.

### Backup
**Settings → Download backup** saves everything as one JSON file. **Restore backup** brings it back, into OneDrive or the browser.

---

## Privacy
- Statements are read **inside your browser**. They are only sent to **your own OneDrive**, nowhere else.
- PDF passwords are never stored.
- The app only asks for `User.Read` and `Files.ReadWrite`. Sign out any time from the top-right button.

## Troubleshooting
| Problem | Fix |
|---|---|
| "Sign-in only works from a web address" | Open the GitHub Pages link, not the file on your computer. |
| Microsoft error *redirect URI mismatch* (AADSTS50011) | In Entra → your app → **Authentication**, the SPA redirect URI must equal the address in your browser bar exactly (with the final `/`). |
| Pop-up blocked | Allow pop-ups for your GitHub Pages site, or just try again. The app falls back to a full-page sign-in. |
| A PDF shows "No transactions found" | Download that statement as Excel/CSV from net banking and add that instead. |
| Totals don't match the bank | The preview warns you. Remove the statement in **Statements** and use the Excel/CSV version. |

## Updating the app
Replace `index.html` in your GitHub repository with a newer version. Your data in OneDrive is not touched.
