# Ledger — Personal Finance Tracker

Ledger is a modern, responsive **personal finance tracking web application** designed to help users manage their income, expenses, monthly budgets, and spending habits in one simple interface.

The application supports **Supabase authentication and cloud data synchronization**, while also providing a **local-only mode** using browser `localStorage` when cloud synchronization is not required.

---

## ✨ Features

### 🔐 Authentication

* User registration and sign-in
* Supabase Authentication support
* Account-based personal data
* Local-only mode without requiring an account
* Sign-out functionality
* Session handling

### 💰 Transaction Management

* Add income transactions
* Add expense transactions
* Edit existing transactions
* Delete transactions
* Add transaction descriptions
* Record transaction amounts
* Select transaction dates
* Add optional notes
* Categorize transactions

### 📊 Financial Overview

The dashboard provides a monthly overview of:

* Net balance
* Total income
* Total expenses
* Number of income transactions
* Number of expense transactions
* Monthly spending progress

### 📅 Monthly Tracking

* Navigate between months
* View transactions for the selected month
* Automatically calculate monthly income
* Automatically calculate monthly expenses
* Automatically calculate monthly net balance

### 🎯 Monthly Budget

* Set a spending target for each month
* View spending progress against the budget
* Visual budget progress indicator
* Budget warning when spending exceeds the target

### 🏷️ Expense Categories

Ledger supports several categories:

* 🍔 Food & Dining
* 🚗 Transport
* 🏠 Housing
* 💡 Utilities
* 🛍️ Shopping
* 🎬 Entertainment
* ❤️ Health
* 📦 Other

The dashboard displays a category-based spending breakdown to help users understand where their money is going.

### 🔎 Transaction Search

Search transactions using:

* Description
* Notes
* Category

This makes it easier to find specific financial records.

### ☁️ Supabase Cloud Sync

When Supabase is configured, user data can be stored in the cloud.

The application uses separate database records for:

* Transactions
* Monthly budgets

Row Level Security policies are included so users can manage only their own financial records.

### 💾 Local Storage Mode

Ledger can also operate without a cloud database.

In local-only mode:

* Transactions are stored in browser `localStorage`
* Budgets are stored in browser `localStorage`
* No account is required
* Data remains on the current browser/device

### ⚡ Realtime Synchronization

Supabase Realtime can be enabled for the `transactions` and `budgets` tables to allow changes to be synchronized across connected devices.

---

## 🛠️ Technologies Used

| Technology   | Purpose                                      |
| ------------ | -------------------------------------------- |
| HTML5        | Application structure                        |
| CSS3         | Styling and responsive layout                |
| JavaScript   | Application logic and interactions           |
| Supabase     | Authentication, database and synchronization |
| PostgreSQL   | Cloud database                               |
| LocalStorage | Local data persistence                       |
| Google Fonts | Typography                                   |

The application uses Supabase's JavaScript client through the CDN.

---

## 🎨 UI & Design

Ledger uses a dark, modern interface with:

* Responsive layout
* Minimal dashboard design
* Financial summary cards
* Category progress bars
* Transaction ledger
* Slide-out transaction form
* Monthly budget modal
* Search interface
* Mobile responsive styling
* Accessible focus states

The application is designed to work across desktop, tablet and mobile screen sizes.

---

## 📁 Project Structure

```text
Ledger/
│
├── index.html
└── README.md
```

The current application is implemented as a single HTML file containing the HTML structure, CSS styles and JavaScript functionality.

---

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/ledger.git
```

### 2. Open the project

Open the project folder in **Visual Studio Code**.

### 3. Run the application

Because Ledger is a client-side web application, you can run it using a local development server.

For example, with VS Code:

1. Install the **Live Server** extension.
2. Open `index.html`.
3. Right-click the file.
4. Select **Open with Live Server**.

---

# ☁️ Supabase Setup

Ledger can use Supabase for authentication and cloud storage.

## 1. Create a Supabase project

Create a new project in Supabase.

Then open:

```text
Project Settings → API
```

Copy your:

* Project URL
* Publishable/Anon public key

Then configure them in the application.

```javascript
const SUPABASE_URL = "YOUR-SUPABASE-URL";

const SUPABASE_ANON_KEY =
  "YOUR-SUPABASE-PUBLIC-KEY";
```

The application is designed to fall back to local-only mode when Supabase is not configured.

---

## 2. Create the Transactions Table

Run the following SQL in the Supabase SQL Editor:

```sql
create table transactions (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  type text not null check (type in ('income','expense')),
  description text not null,
  amount numeric(12,2) not null check (amount > 0),
  category text not null default 'other',
  date date not null,
  note text,
  created_at timestamptz default now()
);

alter table transactions enable row level security;

create policy "Users manage their own transactions"
on transactions
for all
using (auth.uid() = user_id)
with check (auth.uid() = user_id);

create index transactions_user_date_idx
on transactions (user_id, date);
```

---

## 3. Create the Budgets Table

```sql
create table budgets (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  month text not null,
  amount numeric(12,2) not null check (amount > 0),
  created_at timestamptz default now(),
  unique (user_id, month)
);

alter table budgets enable row level security;

create policy "Users manage their own budgets"
on budgets
for all
using (auth.uid() = user_id)
with check (auth.uid() = user_id);
```

The project already includes the corresponding database structure and Row Level Security configuration.

---

## 4. Enable Realtime

For multi-device synchronization, enable Realtime for:

```text
transactions
budgets
```

Ledger listens for database changes and refreshes the displayed data when relevant records change.

---

# 🔒 Security

Ledger uses **Supabase Row Level Security (RLS)** to restrict database access to the authenticated user who owns the records.

The database policies use:

```sql
auth.uid() = user_id
```

This means users can manage their own transactions and budgets rather than accessing another user's records.

> **Important:** Never expose a Supabase service-role key in frontend code. Only use the public client key intended for browser applications.

---

# 📱 Responsive Design

Ledger is designed to adapt to different screen sizes.

### Desktop

* Two-column dashboard
* Category spending panel
* Transaction ledger
* Search
* Monthly navigation
* Budget overview

### Mobile

* Single-column layout
* Responsive summary cards
* Compact navigation
* Mobile-friendly transaction interface

---

# 🧮 Financial Calculations

Ledger automatically calculates:

### Net Balance

```text
Net = Total Income - Total Expenses
```

### Budget Progress

```text
Budget Usage = Expenses / Monthly Budget × 100
```

The application also identifies when expenses exceed the configured monthly budget.

---

# 🔄 Data Storage

Ledger supports two storage modes.

## Cloud Mode

When Supabase authentication is active:

```text
User
 ↓
Supabase Authentication
 ↓
Transactions / Budgets
 ↓
PostgreSQL Database
```

## Local Mode

When using the application without an account:

```text
User
 ↓
Ledger
 ↓
Browser LocalStorage
```

Transaction and budget data are stored using separate local-storage keys.

---

# 📌 Main Use Cases

Ledger can be used for:

* Personal expense tracking
* Monthly budget management
* Income tracking
* Spending analysis
* Student financial management
* Household expense tracking
* Simple personal accounting
* Financial record keeping

---

# 🔮 Future Improvements

Possible future enhancements include:

* 📈 Interactive financial charts
* 📊 Advanced spending analytics
* 📥 CSV transaction import
* 📤 CSV/PDF financial reports
* 🔔 Budget notifications
* 💱 Multiple currencies
* 🏦 Bank account integrations
* 📱 Progressive Web App support
* 🌙 Additional themes
* 📅 Recurring transactions
* 💳 Payment tracking
* 📆 Yearly financial reports
* 🤖 AI-powered spending insights

---

# 🤝 Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a new branch.

```bash
git checkout -b feature/new-feature
```

3. Make your changes.
4. Commit your changes.

```bash
git commit -m "Add new feature"
```

5. Push the branch.

```bash
git push origin feature/new-feature
```

6. Open a Pull Request.

---

# 📄 License

This project is available for educational and personal use.

Add an appropriate open-source license if you plan to distribute the project publicly.

---

# 👨‍💻 Author

**Shaheem Nizar**

BEng (Hons) Software Engineering

GitHub: `https://github.com/ShaheemNizar`

---

## ⭐ Project Summary

**Ledger** is a clean and responsive personal finance tracker that combines simple transaction management with monthly budgeting, category-based spending analysis, authentication and cloud synchronization.

Built using:

```text
HTML5
CSS3
JavaScript
Supabase
PostgreSQL
LocalStorage
```

If you find the project useful, consider giving the repository a ⭐.
