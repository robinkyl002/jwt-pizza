# Routes, Navigation, and API Map

This document maps the React routes to their views, shows which views navigate to other routes, and lists the user actions or lifecycle events that call backend APIs.

## Route Entry Points

Routes are registered in `src/app/app.tsx` through the `navItems` array. The same array feeds the router, header navigation, and footer navigation.

| Route | View | Header/footer exposure | Notes |
| --- | --- | --- | --- |
| `/` | `Home` | Not shown in nav/footer | Home page. The route exists even though the header logo is not a link. |
| `/diner-dashboard` | `DinerDashboard` | User initials link when logged in | Shows user profile and order history. |
| `/menu` | `Menu` | Header as `Order` | Loads menu items and available stores. |
| `/franchise-dashboard` | `FranchiseDashboard` | Header/footer as `Franchise` for non-admin users | Shows franchise/store management if the current user owns a franchise; otherwise shows a franchise sales view. |
| `/about` | `About` | Footer | Static about page. |
| `/history` | `History` | Footer | Static history page. |
| `/admin-dashboard` | `AdminDashboard` | Header as `Admin` for admin users | The view itself renders `NotFound` for non-admin users. |
| `/:subPath?/create-franchise` | `CreateFranchise` | Hidden | Usually accessed as `/admin-dashboard/create-franchise`. |
| `/:subPath?/close-franchise` | `CloseFranchise` | Hidden | Usually accessed as `/admin-dashboard/close-franchise` with franchise state. |
| `/:subPath?/create-store` | `CreateStore` | Hidden | Usually accessed as `/franchise-dashboard/create-store` with franchise state. |
| `/:subPath?/close-store` | `CloseStore` | Hidden | Accessed from admin or franchise dashboards with franchise/store state. |
| `/payment` | `Payment` | Hidden | Requires a logged-in user; redirects to nested login when needed. |
| `/delivery` | `Delivery` | Hidden | Shows completed order and order JWT. |
| `/:subPath?/login` | `Login` | Header as `Login` when logged out | Also works as nested workflow login, such as `/payment/login` or `/franchise-dashboard/login`. |
| `/:subPath?/register` | `Register` | Header as `Register` when logged out | Also works as nested workflow registration. |
| `/:subPath?/logout` | `Logout` | Header as `Logout` when logged in | Clears session and returns home. |
| `/docs/:docType?` | `Docs` | Hidden | `/docs` shows service API docs; `/docs/factory` shows factory API docs. |
| `*` | `NotFound` | Hidden | Catch-all route. |

## Global Navigation

| Source | Destination | Condition | Implementation |
| --- | --- | --- | --- |
| Header nav | `/menu` | Always shown | `Order` nav item. |
| Header nav | `/franchise-dashboard` | Shown when the user is not an admin | `Franchise` nav item uses `isNotAdmin`. |
| Header nav | `/admin-dashboard` | Shown when the user has the `admin` role | `Admin` nav item uses `isAdmin`. |
| Header nav | `/login` | Shown when logged out | The route pattern is normalized from `/:subPath?/login`. |
| Header nav | `/register` | Shown when logged out | The route pattern is normalized from `/:subPath?/register`. |
| Header nav | `/logout` | Shown when logged in | The route pattern is normalized from `/:subPath?/logout`. |
| Header user initials | `/diner-dashboard` | Shown when logged in | Lets the current user jump to the diner dashboard. |
| Footer nav | `/franchise-dashboard` | Always rendered by footer config | No role constraint is checked in `Footer`. |
| Footer nav | `/about` | Always rendered by footer config | Static page. |
| Footer nav | `/history` | Always rendered by footer config | Static page. |
| Breadcrumb | Parent path or sibling path | Any route with breadcrumb links | `useBreadcrumb()` trims the last path segment and preserves `location.state`; `useBreadcrumb('login')` or `useBreadcrumb('register')` swaps to a sibling auth route. |

## View Navigation

| Current view | Trigger | Navigates to | State passed |
| --- | --- | --- | --- |
| `Home` | `Order now` button | `/menu` | None |
| `Menu` | `Checkout` form submit | `/payment` | `{ order }` |
| `Payment` | Unauthenticated mount check | `/payment/login` | Existing `location.state`, normally `{ order }` |
| `Payment` | `Pay now` success | `/delivery` | `{ order: confirmation.order, jwt: confirmation.jwt }` |
| `Payment` | `Cancel` button | `/menu` | `{ order }` |
| `Delivery` | `Order more` button | `/menu` | None |
| `Login` | Successful login | Parent route | Preserves current route state through `useBreadcrumb()` |
| `Login` | `Register` text link | Sibling `register` route | Preserves current route state through `useBreadcrumb('register')` |
| `Register` | Successful registration | Parent route | Preserves current route state through `useBreadcrumb()` |
| `Register` | `Login` text link | Sibling `login` route | Preserves current route state through `useBreadcrumb('login')` |
| `Logout` | Mount effect after logout | `/` | None |
| `DinerDashboard` | `Buy one` link when no orders | `/menu` | None |
| `FranchiseDashboard` | `Create store` button | `/franchise-dashboard/create-store` | `{ franchise }` |
| `FranchiseDashboard` | Store `Close` button | `/franchise-dashboard/close-store` | `{ franchise, store }` |
| `FranchiseDashboard` | `login` link in sales view | `/franchise-dashboard/login` | None |
| `AdminDashboard` | `Add Franchise` button | `/admin-dashboard/create-franchise` | None |
| `AdminDashboard` | Franchise `Close` button | `/admin-dashboard/close-franchise` | `{ franchise }` |
| `AdminDashboard` | Store `Close` button | `/admin-dashboard/close-store` | `{ franchise, store }` |
| `CreateFranchise` | Successful create | Parent route, normally `/admin-dashboard` | Preserves current route state |
| `CreateFranchise` | `Cancel` button | Parent route, normally `/admin-dashboard` | Preserves current route state |
| `CreateStore` | Successful create | Parent route, normally `/franchise-dashboard` | Preserves current route state |
| `CreateStore` | `Cancel` button | Parent route, normally `/franchise-dashboard` | Preserves current route state |
| `CloseFranchise` | Successful close | Parent route, normally `/admin-dashboard` | Preserves current route state |
| `CloseFranchise` | `Cancel` button | Parent route, normally `/admin-dashboard` | Preserves current route state |
| `CloseStore` | Successful close | Parent route, normally `/admin-dashboard` or `/franchise-dashboard` | Preserves current route state |
| `CloseStore` | `Cancel` button | Parent route, normally `/admin-dashboard` or `/franchise-dashboard` | Preserves current route state |

## Backend API Configuration

All backend calls go through `pizzaService`, currently backed by `HttpPizzaService` in `src/service/httpPizzaService.ts`.

Relative API paths are prefixed with `import.meta.env.VITE_PIZZA_SERVICE_URL`. Absolute paths are used as-is, which is how the factory API is called with `import.meta.env.VITE_PIZZA_FACTORY_URL`.

`HttpPizzaService.callEndpoint()` sends JSON requests with `credentials: 'include'`. If `localStorage.token` exists, it also sends `Authorization: Bearer <token>`.

## Actions That Call APIs

| Area/view | Trigger | Service method | HTTP API | Backend |
| --- | --- | --- | --- | --- |
| `App` | Initial app mount with a local token | `getUser()` | `GET /api/user/me` | Pizza Service |
| `Login` | Login form submit | `login(email, password)` | `PUT /api/auth` | Pizza Service |
| `Register` | Register form submit | `register(name, email, password)` | `POST /api/auth` | Pizza Service |
| `Logout` | Route mount | `logout()` | `DELETE /api/auth` | Pizza Service |
| `Payment` | Mount auth check | `getUser()` | `GET /api/user/me` | Pizza Service |
| `Menu` | Route mount | `getMenu()` | `GET /api/order/menu` | Pizza Service |
| `Menu` | Route mount | `getFranchises(0, 20, '*')` | `GET /api/franchise?page=0&limit=20&name=*` | Pizza Service |
| `Payment` | `Pay now` button | `order(order)` | `POST /api/order` | Pizza Service |
| `Delivery` | `Verify` button | `verifyOrder(jwt)` | `POST {VITE_PIZZA_FACTORY_URL}/api/order/verify` | Pizza Factory |
| `DinerDashboard` | Mount or user change | `getOrders(user)` | `GET /api/order` | Pizza Service |
| `FranchiseDashboard` | Mount or user change | `getFranchise(user)` | `GET /api/franchise/:userId` | Pizza Service |
| `AdminDashboard` | Mount, user change, or page change | `getFranchises(franchisePage, 3, '*')` | `GET /api/franchise?page=:page&limit=3&name=*` | Pizza Service |
| `AdminDashboard` | Filter submit | `getFranchises(franchisePage, 10, '*filter*')` | `GET /api/franchise?page=:page&limit=10&name=*filter*` | Pizza Service |
| `CreateFranchise` | Create form submit | `createFranchise(franchise)` | `POST /api/franchise` | Pizza Service |
| `CloseFranchise` | `Close` button | `closeFranchise(franchise)` | `DELETE /api/franchise/:franchiseId` | Pizza Service |
| `CreateStore` | Create form submit | `createStore(franchise, store)` | `POST /api/franchise/:franchiseId/store` | Pizza Service |
| `CloseStore` | `Close` button | `closeStore(franchise, store)` | `DELETE /api/franchise/:franchiseId/store/:storeId` | Pizza Service |
| `Docs` | Route mount for `/docs` | `docs(docType)` | `GET /api/docs` | Pizza Service |
| `Docs` | Route mount for `/docs/factory` | `docs('factory')` | `GET {VITE_PIZZA_FACTORY_URL}/api/docs` | Pizza Factory |

## Non-backend Fetch

`Footer` fetches `/version.json` on mount to display the app version. That file is served from `public/version.json` during local development and from the built static assets after deployment.

## API Method Reference

| Service method | HTTP API | Notes |
| --- | --- | --- |
| `login(email, password)` | `PUT /api/auth` | Stores returned JWT in `localStorage.token`. |
| `register(name, email, password)` | `POST /api/auth` | Stores returned JWT in `localStorage.token`. |
| `logout()` | `DELETE /api/auth` | Removes `localStorage.token` immediately after starting the request. |
| `getUser()` | `GET /api/user/me` | Only calls the API when a local token exists; removes the token if the request fails. |
| `getMenu()` | `GET /api/order/menu` | Returns available pizzas. |
| `getOrders(user)` | `GET /api/order` | The `user` parameter is not used by the HTTP implementation. |
| `order(order)` | `POST /api/order` | Returns an order confirmation and JWT. |
| `verifyOrder(jwt)` | `POST {VITE_PIZZA_FACTORY_URL}/api/order/verify` | Sends `{ jwt }` to the factory API. |
| `getFranchise(user)` | `GET /api/franchise/:userId` | Uses `user.id`. |
| `createFranchise(franchise)` | `POST /api/franchise` | Sends the franchise object. |
| `getFranchises(page, limit, nameFilter)` | `GET /api/franchise?page=:page&limit=:limit&name=:nameFilter` | Used by menu store loading and admin franchise management. |
| `closeFranchise(franchise)` | `DELETE /api/franchise/:franchiseId` | Uses `franchise.id`. |
| `createStore(franchise, store)` | `POST /api/franchise/:franchiseId/store` | Uses `franchise.id`; sends the store object. |
| `closeStore(franchise, store)` | `DELETE /api/franchise/:franchiseId/store/:storeId` | Uses `franchise.id` and `store.id`. |
| `docs(docType)` | `GET /api/docs` or `GET {VITE_PIZZA_FACTORY_URL}/api/docs` | `docType === 'factory'` selects the factory API. |
