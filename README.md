> **Source code not published** - Harvard CS50's academic honesty policy prohibits sharing problem set solutions. This README covers architecture and implementation decisions I can walk through in an interview.

# CS50 Finance

Flask web app where users simulate buying and selling stocks with virtual cash. Holdings live in a normalized SQLite database; live prices come from CS50's stock quote API at request time and are merged with database state on the portfolio page.

This is a CS50 problem set implementation. The starter scaffold provided the login route, session authentication, and `helpers.py` (including the `lookup()` API wrapper). I wrote everything else.

## Tech Stack

- **Python 3 / Flask** - routing, sessions, Jinja2 templates
- **SQLite** via CS50's `SQL` class
- **Flask-Session** - server-side filesystem sessions
- **Werkzeug** - password hashing on register, hash verification on login
- **Bootstrap 5** - UI
- **CS50 stock quote API** - real-time price lookup via the provided `lookup()` helper (I did not write this function)

## Features

- **Register** - Validates all fields, checks passwords match, hashes password before storing, rejects duplicate usernames
  - JavaScript show/hide password toggle on the register form
- **Log in / log out** - Session-based authentication (starter-provided route); logout clears the session
- **Quote** - Accepts a stock symbol, hits the API, and returns the current price and company name
- **Buy** - Validates symbol and share count, enforces affordability server-side, inserts or updates the holding in `shares`, debits cash, and logs the transaction to `history`
- **Sell** - Validates share count server-side against current holdings, credits cash at the current market price, and logs the transaction to `history`
- **Portfolio** - Fetches live prices at request time and merges with database holdings to show each position's current value, remaining cash, total portfolio value, and the total cash spent
- **History** - Shows all buy and sell transactions with stock name, price, share count, and timestamp, ordered newest first

## Demo


https://github.com/user-attachments/assets/b0f05574-7b3e-40b0-87a4-e8fc3dd61fef


<img width="1710" height="953" alt="Image" src="https://github.com/user-attachments/assets/ac0eaf9e-2e94-4b94-bc4d-c5501b5d5f73"/>


## Design Decisions

### Relational Database - Three normalized tables instead of one

`users` stores login credentials and cash balance. `shares` stores current positions (one row per symbol per user). `history` is an append-only log of every transaction.

That split keeps the portfolio query simple (`SELECT * FROM shares WHERE user_id = ?`) while `history` independently records every trade with price, share count, and type. Foreign keys: `shares.user_id → users.id`, `history.user_id → users.id`, `history.stock_id → shares.stock_id`.

#### Update vs. insert on repeated buy

If the user already owns a symbol, the buy route runs `UPDATE shares SET shares = shares + ?` rather than inserting a duplicate row. A new symbol gets `INSERT INTO shares`. Either path logs a row to `history` with the correct `stock_id`.

#### Prices not stored in the database

Holdings store symbol and quantity only - no cached prices. The portfolio page calls `lookup(symbol)` per holding at request time, so displayed values always reflect current market price. Buy and sell routes do the same so affordability and proceeds use the same data source.

#### Parameterized queries throughout

All SQL uses `?` placeholders rather than string concatenation, including the joined history query, to prevent SQL injection.

## What Was Provided vs. What I Wrote

Three helpers from `helpers.py` appear throughout the routes below - `lookup(symbol)` calls the CS50 stock API and returns the stock's name, symbol, and current price; `usd(value)` formats a float as a USD currency string; `@login_required` is a decorator that redirects unauthenticated users to login before they can access buy, sell, quote, or portfolio. I did not write any of those.


| Provided by CS50                                     | Written by me                                                   |
| ---------------------------------------------------- | --------------------------------------------------------------- |
| Login route, session handling, `check_password_hash` | Register, buy, sell, quote, portfolio index, history            |
| `@login_required` decorator                          | SQLite schema, all CRUD operations                              |
| `helpers.py` (`lookup`, `apology`, `usd`)            | Input validation, join query for history, portfolio aggregation |
| Base templates and layout                            | Password visibility toggle (JS)                                 |


