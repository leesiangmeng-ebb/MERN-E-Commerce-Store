# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Run from the root directory:

```bash
npm run dev          # Run frontend and backend concurrently
npm run backend      # Run Express backend only (nodemon, port 5000)
npm run frontend     # Run Vite dev server only
```

Run from `frontend/`:

```bash
npm run build        # Production build
npm run lint         # ESLint (max-warnings 0)
npm run preview      # Preview production build
```

No test framework is configured.

## Environment Variables

Backend reads from `.env` in the root:

```
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/huxnStore
NODE_ENV=development
JWT_SECRET=<secret>
PAYPAL_CLIENT_ID=<paypal-client-id>
```

## Architecture

This is a monorepo with a separate Express backend and React frontend:

- **`backend/`** — Express API server (port 5000)
- **`frontend/`** — React SPA via Vite (dev proxy forwards `/api/` and `/uploads/` to `localhost:5000`)

### Backend

MVC-style structure:

- `backend/index.js` — Entry point; mounts middleware and routes
- `backend/config/db.js` — Mongoose connection
- `backend/models/` — Mongoose schemas: `userModel`, `productModel`, `orderModel`, `categoryModel`
- `backend/routes/` — Route definitions for users, products, orders, categories, uploads
- `backend/controllers/` — Business logic per resource
- `backend/middlewares/authMiddleware.js` — `authenticate` (JWT via HTTP-only cookie) and `authorizeAdmin` (checks `isAdmin` flag)
- `backend/utils/createToken.js` — Generates JWT, sets `jwt` HTTP-only cookie (30-day expiry)

Authentication uses JWT stored as an HTTP-only cookie (`jwt`). The `authenticate` middleware reads `req.cookies.jwt`, verifies it with `JWT_SECRET`, and attaches `req.user`. Admin routes additionally call `authorizeAdmin`.

File uploads use Multer; files are stored in `/uploads/` and served statically by Express.

### Frontend

- `frontend/src/main.jsx` — Initializes Redux store, React Router, and renders `<App>`
- `frontend/src/App.jsx` — Top-level routes with `<Outlet>`
- `frontend/src/pages/` — Route-level components grouped by domain (`Auth/`, `Admin/`, `Products/`, `Orders/`, `User/`)
- `frontend/src/components/` — Shared components (`PrivateRoute`, `Header`, `Modal`, `Loader`, etc.)
- `frontend/src/redux/` — Redux Toolkit store and slices

### State Management

Redux Toolkit with RTK Query:

- `redux/store.js` — Combines all reducers; adds RTK Query middleware
- `redux/api/apiSlice.js` — RTK Query base API (tag types: `Product`, `Order`, `User`, `Category`)
- `redux/api/` — Resource-specific RTK Query slices (`usersApiSlice`, `productApiSlice`, `categoryApiSlice`, `orderApiSlice`)
- `redux/features/auth/authSlice.js` — `userInfo` persisted to localStorage (30-day expiry)
- `redux/features/cart/cartSlice.js` — Cart items, shipping address, payment method; persisted to localStorage
- `redux/features/favorites/favoriteSlice.js` — Favorites list; persisted to localStorage
- `redux/constants.js` — API URL constants (`USERS_URL`, `PRODUCT_URL`, `ORDERS_URL`, `PAYPAL_URL`)

### Key API Endpoints

| Prefix | Notes |
|---|---|
| `POST /api/users` | Register |
| `POST /api/users/auth` | Login (sets JWT cookie) |
| `POST /api/users/logout` | Logout (clears cookie) |
| `GET/PUT /api/users/profile` | Auth required |
| `GET /api/products` | Paginated; `GET /api/products/allproducts` for full list |
| `POST /api/products/filtered-products` | Filter by category/price |
| `POST /api/upload` | Image upload (multipart) |
| `GET /api/config/paypal` | Returns PayPal client ID |
| `GET /api/orders/total-orders` | Admin dashboard stats |
| `GET /api/orders/total-sales-by-date` | Used by ApexCharts in admin |
