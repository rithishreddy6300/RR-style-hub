# StyleHub – Fashion Mini Store

StyleHub is a full-stack demo shop built with FastAPI, SQLAlchemy, SQLite,
React, Vite, and Stripe test mode. It includes user accounts, a seeded clothing
catalog, per-user carts, Stripe Elements checkout, webhook-driven order status,
and order history.

## Project layout

```text
backend/
  app/
    core/       Environment configuration
    models/     SQLAlchemy ORM models
    routers/    Auth, catalog, cart, order, and payment APIs
    schemas/    Request and response validation
  tests/
frontend/       React + TypeScript + Vite client
```

## Run the API

From the repository root in PowerShell:

```powershell
cd backend
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
```

Edit `backend/.env` before starting the server. Set `JWT_SECRET_KEY` to a
unique random secret of at least 32 characters and use the test keys from your
Stripe Dashboard for `STRIPE_SECRET_KEY` and `STRIPE_PUBLISHABLE_KEY`. Keep
`.env` private and out of source control. `CORS_ORIGINS` is a JSON array of
allowed frontend origins; replace the local development origins for deployment.
The separate admin page is available at `http://localhost:5173/admin`. Its
default demo login is username `REDDY` and password `6300648854`; set
`ADMIN_USERNAME` and `ADMIN_PASSWORD` in `backend/.env` to private credentials
before deploying.

```powershell
uvicorn app.main:app --reload
```

The API runs at `http://localhost:8000`; interactive API documentation is at
`http://localhost:8000/docs`. Demo products are inserted automatically when the
products table is empty.

## Run the frontend

In a second PowerShell terminal:

```powershell
cd frontend
npm install
Copy-Item .env.example .env
npm run dev
```

The Vite app runs at `http://localhost:5173`. `VITE_API_URL` defaults to
`http://localhost:8000`.

## Configure Stripe test webhooks

Install and authenticate the Stripe CLI, then run it in another terminal:

```powershell
stripe listen --forward-to localhost:8000/api/payments/webhook
```

Copy the `whsec_...` value printed by the CLI into `STRIPE_WEBHOOK_SECRET` in
`backend/.env`, then restart the API. The webhook updates an order to paid,
failed, or canceled after checking the Stripe signature, payment-intent ID,
amount, and currency. The browser never marks an order paid.

Use Stripe's test card `4242 4242 4242 4242`, a future expiry date, and any CVC
to exercise a successful test payment. No live payment credentials should be
used.

## API surface

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/api/auth/register` | Public | Register with email and password |
| `POST` | `/api/auth/login` | Public | Obtain a bearer JWT |
| `GET` | `/api/auth/me` | JWT | Read the signed-in profile |
| `GET` | `/api/products` | Public | List the active catalog |
| `GET` | `/api/products/{product_id}` | Public | Read one active product |
| `GET` | `/api/cart` | JWT | Read the signed-in user's cart |
| `POST` | `/api/cart/items` | JWT | Add an item by product ID and quantity |
| `PUT` | `/api/cart/items/{product_id}` | JWT | Set an item quantity |
| `DELETE` | `/api/cart/items/{product_id}` | JWT | Remove an item |
| `POST` | `/api/orders/checkout` | JWT | Place a cash-on-delivery order or create a Stripe payment |
| `GET` | `/api/orders` | JWT | List the signed-in user's orders |
| `GET` | `/api/orders/{order_id}` | JWT | Read one of the signed-in user's orders |
| `POST` | `/api/payments/webhook` | Stripe signature | Process payment-intent events |
| `POST` | `/api/admin/login` | Public | Obtain an admin bearer JWT |
| `GET` | `/api/admin/products` | Admin JWT | List products for management |
| `POST` | `/api/admin/products` | Admin JWT | Add a product to the customer catalog |
| `PUT` | `/api/admin/products/{product_id}` | Admin JWT | Update a catalog product |

Prices are stored as integer INR paise (minor currency units) and never trusted
from the client (for example, 5,900 paise displays as ₹59.00). Stripe payments
are restricted to INR (`STRIPE_CURRENCY=inr`). Checkout supports cash on
delivery, UPI, debit card, and credit card; card and UPI payments use Stripe
test mode. Every checkout collects the recipient's name, phone, street address,
city, state, and postal code and stores those details with the order.
Passwords use bcrypt; input is validated; JWTs expire; order lines preserve
product/price snapshots. The frontend keeps the bearer token in session storage
for the current browser tab.

## Tests and build

From the repository root:

```powershell
Set-Location backend
py -m pytest
Set-Location ..\frontend
npm run lint
npm run build
```

## Production follow-up

This is a complete test-mode project flow, not a production deployment
configuration. Before launch, add schema migrations, deployment-specific CORS
origins, rate limiting and account abuse controls, secure refresh/session
management, inventory reservation and shipping/tax policy, observability, and
an HTTPS deployment. The SQLite database is suitable for local development and
the project uses ORM table creation for this initial demo.
