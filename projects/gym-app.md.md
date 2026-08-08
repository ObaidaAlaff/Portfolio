# Sala Hall Manager
> A complete offline management system for wedding & event halls — built for Android and Windows.

---

## Overview

Sala Hall Manager is a cross-platform desktop and mobile application designed to help wedding hall owners run their business without relying on the internet or external servers. It covers the full operational loop: booking management, income and expense tracking, account balances, task lists, monthly reports, and data export — all stored locally in an SQLite database on the device.

Built for small-to-medium hall businesses that need a fast, reliable, Arabic-first tool they can use on a Windows PC at the office or an Android phone on the go.

---

## My Role

I designed and built this application end-to-end as a solo project:

- **Architecture** — layered structure separating models, database, screens, dialogs, and shared widgets; singleton `DatabaseHelper` managing all SQLite operations
- **Database layer** — schema design (8 tables with FK relationships), versioned migrations, JOIN-based queries for relational data, and cross-platform initialization (FFI for Windows, native sqflite for Android)
- **UI/UX** — fully RTL Arabic interface using Material Design 3, responsive layout switching between a desktop sidebar and a mobile drawer at the 700 px breakpoint
- **All 8 feature screens** — Dashboard, Income, Expenses, Accounts, Tasks, Reports, Export, Admin
- **Export system** — PDF reports via the `pdf` + `printing` packages; CSV export; SQLite database backup with Android-compatible file saving using `getExternalStorageDirectory`

---

## Key Features

- **Booking calendar** — visual monthly calendar showing booked dates; add/edit/delete bookings with price, deposit, photography flag, and notes
- **Income tracking** — categorized income entries linked to bookings and accounts; filter by category, account, or date range
- **Expense tracking** — categorized expenses with receipt image support and account linking
- **Multi-account ledger** — track balances across multiple accounts with automatic income/expense calculation
- **Task manager** — to-do list with pending/completed toggle and due dates
- **Monthly & yearly reports** — bar charts showing income vs expenses per month; 5-card summary with total income, expenses, profit, and booking count
- **Data export** — export filtered data as PDF or CSV; full database backup to device storage
- **Admin panel** — change hall name, update login credentials, backup/restore the SQLite database
- **Fully offline** — zero internet dependency; all data stays on the device
- **Responsive design** — adapts between desktop sidebar layout and mobile drawer layout automatically

---

## Tech Stack

![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?style=flat&logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-3.x-0175C2?style=flat&logo=dart&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-Local%20DB-003B57?style=flat&logo=sqlite&logoColor=white)
![Material Design 3](https://img.shields.io/badge/Material%20Design-3-757575?style=flat&logo=materialdesign&logoColor=white)
![Android](https://img.shields.io/badge/Android-Target-3DDC84?style=flat&logo=android&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-Desktop-0078D4?style=flat&logo=windows&logoColor=white)

| Package | Purpose |
|---|---|
| `sqflite` + `sqflite_common_ffi` | SQLite on Android and Windows Desktop |
| `sqlite3_flutter_libs` | Native SQLite binaries |
| `path_provider` | Platform-correct storage paths |
| `intl` | Arabic date/number formatting |
| `table_calendar` | Interactive booking calendar |
| `pdf` + `printing` | PDF report generation and sharing |
| `file_picker` | Save-file dialog on Desktop |
| `csv` | CSV export |
| `open_filex` | Open exported files from within the app |

---

## Screenshots

### Dashboard
![Dashboard](screenshots/dashboard.png)
*Stats overview cards (income, expenses, profit, bookings) alongside an interactive booking calendar and upcoming bookings list.*

### Booking Calendar
![Booking Calendar](screenshots/booking_calendar.png)
*Monthly calendar highlighting booked dates; tap any date to view or create bookings.*

### Income
![Income](screenshots/income.png)
*Full income list with category/account/date filters; desktop shows a DataTable, mobile shows a card list.*

### Add / Edit Income
![Add Income Dialog](screenshots/income_dialog.png)
*Dialog for adding or editing an income entry — amount, category, date, account, linked booking, and notes.*

### Expenses
![Expenses](screenshots/expenses.png)
*Categorized expense list with receipt image support; same responsive layout as the income screen.*

### Accounts
![Accounts](screenshots/accounts.png)
*Account balances calculated automatically from initial balance plus all linked income and expense entries.*

### Account Transactions
![Account Transactions](screenshots/account_transactions.png)
*Full transaction history per account with colored income/expense indicators.*

### Tasks
![Tasks](screenshots/tasks.png)
*Task list with segmented filter (All / Pending / Completed) and due dates.*

### Reports
![Reports](screenshots/reports.png)
*Yearly bar chart comparing income and expenses month by month, with a 5-card summary at the top.*

### Export
![Export](screenshots/export.png)
*Filter and export any data range as PDF or CSV; both formats supported on Desktop and Android.*

### Admin Panel
![Admin Panel](screenshots/admin.png)
*Settings for hall name, login credentials, and full database backup/restore.*

---

## Technical Highlights

### Cross-platform SQLite initialization
SQLite requires different setup on Windows (FFI) and Android (native). The app detects the platform at startup and initializes the correct database factory before any DB call is made.

```dart
if (!kIsWeb && (Platform.isWindows || Platform.isLinux || Platform.isMacOS)) {
  sqfliteFfiInit();
  databaseFactory = databaseFactoryFfi;
}
```

### Responsive layout with a single breakpoint
Rather than maintaining two separate widget trees, every screen reads `MediaQuery.sizeOf(context).width < 700` and swaps between a DataTable (desktop) and a `ListView` of cards (mobile). Filters switch from a horizontal `Row` to a vertical `Column + Wrap`. One codebase, two experiences.

### JOIN-based queries for relational display
Income and expense records reference categories, accounts, and bookings by foreign key. All list queries use raw SQL `LEFT JOIN` to pull display names in a single database round-trip rather than loading related records separately.

```sql
SELECT i.*, ic.name AS category_name, a.name AS account_name, b.name AS booking_name
FROM income i
LEFT JOIN income_categories ic ON i.category_id = ic.id
LEFT JOIN accounts a ON i.account_id = a.id
LEFT JOIN bookings b ON i.booking_id = b.id
```

### Android-safe database export
`FilePicker.platform.saveFile()` is Desktop-only and fails silently on Android. The export path is resolved at runtime: Desktop gets a native save dialog, Android writes to `getExternalStorageDirectory()` and shows the path in a dialog so the user can find the file.

### Versioned database migrations
The database uses a version number with an `onUpgrade` callback so new tables or columns can be added in future releases without wiping existing user data.

### Offline-first with no state management library
The app intentionally uses no external state management package. Each screen is a `StatefulWidget` that owns its data list and reloads from SQLite after every write. Simple, predictable, and easy to maintain.

---

## Status

🟡 **In Active Development** — core features complete and in daily use on Android; ongoing UI refinements and additional report types planned.

---

## License

Private project — not open source.
