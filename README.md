# Expense Tracking System

A full-stack expense tracking application built with **Python**, **FastAPI**, **Streamlit**, and **MySQL**. Record expenses by date, update existing entries, and view category-wise spending analytics.

## Features

- **Add expenses:** Enter an amount, category, and notes for a selected date.
- **View expenses:** Fetch expenses already saved for the selected date.
- **Update daily expenses:** Save a new set of entries for a date (replaces all existing entries for that date).
- **Analyze spending:** See totals and percentage shares by category within a date range.
- **Visualizations:** View a Streamlit bar chart and breakdown table.
- **Logging:** Record database activity in `server.log`.
- **Tests:** Run database helper tests with `pytest`.

Available categories: **Rent, Food, Shopping, Entertainment, Other**.

> **Current implementation:** The entry form displays five rows per selected date. Saving removes existing entries for that date and inserts the nonzero amounts currently entered. This is not an individual-record edit operation.

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | Streamlit |
| Backend / REST API | FastAPI, Pydantic, Uvicorn |
| Database | MySQL, MySQL Connector/Python |
| Analytics | Pandas, Streamlit charts |
| HTTP client | Requests |
| Testing | Pytest |
| Logging | Python `logging` module |

## Project Structure

```text
Project_expense_management/
├── backend/
│   ├── db_helper.py         # MySQL database operations
│   ├── logging_setup.py     # Logging configuration
│   ├── server.py            # FastAPI routes
│   └── server.log           # Generated log output
├── frontend/
│   ├── add_update_ui.py     # Daily expense form
│   ├── analytics_ui.py      # Analytics chart and table
│   └── app.py               # Streamlit entry point
├── tests/
│   ├── backend/
│   │   └── test_db_helper.py  # Database helper tests
│   ├── frontend/
│   └── conftest.py
├── requirements.txt
└── README.md
```

## Prerequisites

- Python 3.10+ (use a version supported by your installed packages)
- MySQL Server running locally
- pip and an optional Python virtual environment

## Installation

### 1. Clone or open the project

```bash
git clone <your-repository-url>
cd Project_expense_management
```

If you already have the project folder, just open a terminal in its root directory.

### 2. Create and activate a virtual environment

**Windows (PowerShell):**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install -r requirements.txt
```

**Dependency note:** The uploaded `requirements.txt` lists `streamlit`, `pandas`, `pytest`, `pydantic`, `uvicorn`, `my-sql-connector`, and `requests`, but the code also imports **FastAPI** and `mysql.connector`. Make sure your requirements include the correct packages for those imports:

```text
fastapi
mysql-connector-python
```

Replace `my-sql-connector` with `mysql-connector-python` if you are using Oracle's standard MySQL Connector/Python. Check that the pinned versions of your other packages are available in your Python environment.

## Database Setup

1. Start MySQL Server.
2. Open MySQL Workbench or the MySQL command-line client.
3. Run the following SQL (the table structure is consistent with the columns used by `db_helper.py`):

```sql
CREATE DATABASE IF NOT EXISTS expense_manager;
USE expense_manager;

CREATE TABLE IF NOT EXISTS expenses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    expense_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    category VARCHAR(100) NOT NULL,
    notes TEXT
);
```

4. Configure your MySQL connection in `backend/db_helper.py` (`host`, `user`, `password`, `database`). The current uploaded implementation uses a hardcoded local connection. **For a safer project, move credentials into environment variables** and do not commit real passwords or `.env` files to Git. If a real password was previously shared or committed, rotate it.

The database name expected by the code is `expense_manager`; the table is `expenses`.

## Run the Application

Open **two terminals** in the project root and activate the virtual environment in each.

### Terminal 1 — Start the FastAPI backend

```bash
cd backend
python -m uvicorn server:app --reload --host 127.0.0.1 --port 8000
```

Backend URL: <http://localhost:8000>

Interactive API documentation: <http://localhost:8000/docs>

### Terminal 2 — Start the Streamlit frontend

From the project root:

```bash
python -m streamlit run frontend/app.py
```

Streamlit normally opens at <http://localhost:8501>.

**Important:** Keep the backend running while using the frontend. Both frontend modules send HTTP requests to `http://localhost:8000`.

## How to Use

### Add / Update

1. Open the Streamlit page and select **Add/Update**.
2. Choose an expense date.
3. Enter the expense **Amount**, **Category**, and **Notes** in the form.
4. Submit the form to save entries with an amount greater than zero.
5. Existing expenses for that date are replaced with the submitted entries.

### Analytics

1. Select the **Analytics** tab.
2. Choose a start and end date.
3. Click **Get Analytics**.
4. Review spending totals and percentage breakdowns by category, displayed in a bar chart and a table.

## REST API Endpoints

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/expenses/{expense_date}` | Fetch all expenses for a date |
| `POST` | `/expenses/{expense_date}` | Replace expenses for a date with the supplied list |
| `POST` | `/analytics/` | Get category-wise totals and percentages over a date range |

Dates use `YYYY-MM-DD`.

### Fetch expenses

```http
GET /expenses/2024-08-01
```

Example response (illustrative):

```json
[
  {
    "id": 1,
    "expense_date": "2024-08-01",
    "amount": 250.0,
    "category": "Food",
    "notes": "Lunch"
  }
]
```

### Add or replace expenses for a date

```http
POST /expenses/2024-08-01
Content-Type: application/json

[
  {"amount": 250.0, "category": "Food", "notes": "Lunch"},
  {"amount": 100.0, "category": "Shopping", "notes": "Groceries"}
]
```

Example response:

```json
{"message": "Expense updated successfully"}
```

**Warning:** This endpoint first deletes all expenses for the selected date, then inserts the supplied list. An empty list therefore clears that date's expenses. In the uploaded code, deletion and each insertion are separate transactions; consider making the entire replacement atomic before using the application with important data.

### Get expense analytics

```http
POST /analytics/
Content-Type: application/json

{"start_date": "2024-08-01", "end_date": "2024-08-31"}
```

Example response (illustrative):

```json
{
  "Food": {"total": 250.0, "percentage": 71.42857142857143},
  "Shopping": {"total": 100.0, "percentage": 28.57142857142857}
}
```

The analytics query groups expenses by category and sums their amounts for the inclusive date range. With no matching expenses, the API returns an empty object.

## Tests

The uploaded `test_db_helper.py` contains database-dependent tests for fetching an existing date and a date without data. These tests expect particular sample records to exist in the database, so they may fail on a fresh database.

From the **project root**, run:

```bash
python -m pytest tests/ -v
```

Ensure the backend module is importable in your environment; adjust your Python path or package layout if needed. For stable automated testing, use a dedicated test database and fixtures rather than production data.

## Architecture

```text
User (Browser)
      |
      v
Streamlit Frontend (localhost:8501)
      |   HTTP requests via requests
      v
FastAPI Backend (localhost:8000)
      |
      v
Database Helpers (db_helper.py)
      |
      v
MySQL Database (expense_manager)
```

## Known Limitations and Potential Improvements

- Replace hardcoded database credentials with environment variables and rotate any credentials that have been exposed.
- Use a single database transaction for replacing a day's expenses; ensure connections close even when errors occur.
- The Streamlit editor currently shows a fixed five rows, even if a date has more entries.
- Add request timeout and error handling for frontend HTTP calls.
- Validate expense amounts, date ranges, and categories in the API.
- Improve logging to avoid duplicate handlers and respect the logger configuration arguments.
- Add tests that use isolated fixtures or a test database.
- Add authentication, individual-expense editing, budgeting, and data export as future features.
- Exclude `.venv/`, `.env`, `__pycache__/`, and `*.log` from version control using `.gitignore`.

## License

No license was provided with the source files. Add a `LICENSE` file if you intend to distribute the project under an open-source license.
