# E-Commerce Demo API — Endpoint Reference

**API version:** 1.0.0  
**API title:** E-Commerce Demo API  
**Format:** REST / JSON  
**Authentication:** HTTP Bearer token for protected endpoints  
**Storage:** In-memory demo data  
**Database:** Not required

> This documentation is based on the supplied FastAPI source code. It describes the behavior implemented in that source, including validation, response fields, and error handling. The API is intended for demonstration and documentation, not production authentication.

## Table of contents

- [1. Overview](#1-overview)
- [2. Base URL and common conventions](#2-base-url-and-common-conventions)
- [3. Authentication](#3-authentication)
- [4. Common error format](#4-common-error-format)
- [5. Login API](#5-login-api)
- [6. Product APIs](#6-product-apis)
  - [6.1 Get all products](#61-get-all-products)
  - [6.2 Get product by ID](#62-get-product-by-id)
  - [6.3 Create product](#63-create-product)
  - [6.4 Update product](#64-update-product)
  - [6.5 Delete product](#65-delete-product)
- [7. Cart APIs](#7-cart-apis)
  - [7.1 Get current cart](#71-get-current-cart)
  - [7.2 Add product to cart](#72-add-product-to-cart)
  - [7.3 Update cart item quantity](#73-update-cart-item-quantity)
  - [7.4 Remove item from cart](#74-remove-item-from-cart)
  - [7.5 Clear cart](#75-clear-cart)
- [8. Order APIs](#8-order-apis)
  - [8.1 Create order from cart](#81-create-order-from-cart)
  - [8.2 Get current user's orders](#82-get-current-users-orders)
  - [8.3 Get order by ID](#83-get-order-by-id)
  - [8.4 Cancel order](#84-cancel-order)
- [9. Health check](#9-health-check)
- [10. Important implementation notes](#10-important-implementation-notes)

---

## 1. Overview

The E-Commerce Demo API provides endpoints for:

| Resource | Description |
|---|---|
| Login | Authenticates a demo user and returns an access token. |
| Products | Retrieves, creates, updates, and deletes product records. |
| Cart | Retrieves a user's cart and adds, updates, removes, or clears cart items. |
| Orders | Creates orders from the cart, retrieves orders, and cancels eligible orders. |
| Health | Reports whether the API service is running. |

The API uses JSON for request and response bodies where applicable.

## 2. Base URL and common conventions

The supplied source code does not define a deployment host or port. Replace `{base_url}` in the examples with the actual server URL.

```text
{base_url}
```

The API uses the `/api/v1` path prefix for business endpoints. The health endpoint is `/health`.

### Content type

For requests with a JSON body, send:

```http
Content-Type: application/json
```

### Path parameters

Values enclosed in curly braces are path parameters. Replace them with actual values.

Example:

```text
/api/v1/products/{product_id}
```

For product ID `1`, use:

```text
/api/v1/products/1
```

### HTTP methods used

| Method | General purpose in this API |
|---|---|
| GET | Retrieve resources. |
| POST | Authenticate, create resources, or perform an order cancellation action. |
| PUT | Update a product. |
| PATCH | Update a cart item's quantity. |
| DELETE | Delete a product, remove a cart item, or clear a cart. |

## 3. Authentication

The API uses HTTP Bearer token authentication for protected endpoints.

### Demo credentials

| Field | Demo value |
|---|---|
| Username | `demo` |
| Password | `Demo@123` |

### Login to obtain an access token

Send a `POST` request to `/api/v1/login` with the demo username and password. The successful response includes an `access_token`.

The supplied implementation uses this fixed demo token:

```text
demo-access-token
```

### Authorization header

Include the token in the `Authorization` header when calling a protected endpoint:

```http
Authorization: Bearer demo-access-token
```

### Endpoints that require authentication

- `POST /api/v1/products`
- `PUT /api/v1/products/{product_id}`
- `DELETE /api/v1/products/{product_id}`
- All Cart endpoints
- All Order endpoints

The Login endpoint and the two product retrieval endpoints do not require authentication in the supplied implementation. The Health endpoint also does not require authentication.

### Authentication errors

| HTTP status | Condition | Response detail |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing. | `Missing Bearer token` |
| `401 Unauthorized` | Token does not match the configured demo token. | `Invalid or expired access token` |

The authentication implementation uses a fixed token and is explicitly not secure production authentication.

## 4. Common error format

### Validation error — HTTP 422

When request validation fails, the API returns a response in this format:

```json
{
  "detail": "Request validation failed",
  "errors": [
    {
      "field": "body.quantity",
      "message": "Input should be greater than 0",
      "type": "greater_than"
    }
  ]
}
```

The `field`, `message`, and `type` values depend on the validation failure. The example illustrates the response structure; the exact message and type are generated by validation.

| Field | Type | Description |
|---|---|---|
| `detail` | string | General validation failure message. |
| `errors` | array | List of validation errors. |
| `errors[].field` | string | Location of the invalid field. |
| `errors[].message` | string | Description of the validation failure. |
| `errors[].type` | string | Validation error type. |

### Internal server error — HTTP 500

Unexpected exceptions are returned without exposing internal exception details:

```json
{
  "detail": "Internal server error"
}
```

### Other error responses

Individual endpoints may return `400`, `401`, `404`, or `409` depending on the operation. Their conditions and response examples are documented with each endpoint below.

---

## 5. Login API

### Authenticate user

Authenticates a user using the demo username and password. On successful authentication, the API returns a Bearer access token and basic user information.

| Property | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/api/v1/login` |
| Authentication | Not required |
| Request content type | `application/json` |
| Success status | `200 OK` |

#### Request body

```json
{
  "username": "demo",
  "password": "Demo@123"
}
```

| Field | Type | Required | Constraints | Description |
|---|---|---:|---|---|
| `username` | string | Yes | 1–50 characters | Demo account username. |
| `password` | string | Yes | 1–100 characters | Demo account password. |

#### Successful response — `200 OK`

```json
{
  "access_token": "demo-access-token",
  "token_type": "bearer",
  "expires_in": null,
  "user": {
    "username": "demo",
    "name": "Demo User"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `access_token` | string | Access token to use in the Authorization header. |
| `token_type` | string | Token type. The value is `bearer`. |
| `expires_in` | null | Expiration duration is not configured; the response value is `null`. |
| `user` | object | Basic information about the authenticated user. |
| `user.username` | string | Username of the authenticated user. |
| `user.name` | string | Display name of the authenticated user. |

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Username does not exist or password is incorrect. | `{"detail":"Invalid username or password"}` |
| `422 Unprocessable Entity` | Username or password is missing or violates validation constraints. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X POST "{base_url}/api/v1/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "demo",
    "password": "Demo@123"
  }'
```

---

## 6. Product APIs

Product endpoints allow clients to retrieve product information. Authenticated users can also create, update, and delete products.

### Product object

The product response uses the following fields:

| Field | Type | Description |
|---|---|---|
| `id` | integer | Unique product identifier. |
| `name` | string | Product name. |
| `description` | string | Product description. |
| `price` | number | Product price. |
| `stock` | integer | Available stock quantity. |

The source initializes these demo products:

| ID | Name | Description | Price | Stock |
|---:|---|---|---:|---:|
| 1 | Laptop | Demo laptop | 55000.00 | 10 |
| 2 | Wireless Mouse | Wireless USB mouse | 799.00 | 25 |
| 3 | Keyboard | USB keyboard | 1299.00 | 15 |

### 6.1 Get all products

Retrieves all products currently stored by the API.

| Property | Value |
|---|---|
| Method | `GET` |
| Endpoint | `/api/v1/products` |
| Authentication | Not required |
| Success status | `200 OK` |

#### Request parameters

No path parameters, query parameters, or request body are required.

#### Successful response — `200 OK`

```json
{
  "items": [
    {
      "id": 1,
      "name": "Laptop",
      "description": "Demo laptop",
      "price": 55000.0,
      "stock": 10
    },
    {
      "id": 2,
      "name": "Wireless Mouse",
      "description": "Wireless USB mouse",
      "price": 799.0,
      "stock": 25
    },
    {
      "id": 3,
      "name": "Keyboard",
      "description": "USB keyboard",
      "price": 1299.0,
      "stock": 15
    }
  ],
  "total": 3
}
```

| Field | Type | Description |
|---|---|---|
| `items` | array of objects | List of product records. |
| `items[].id` | integer | Product identifier. |
| `items[].name` | string | Product name. |
| `items[].description` | string | Product description. |
| `items[].price` | number | Product price. |
| `items[].stock` | integer | Available stock quantity. |
| `total` | integer | Number of products returned. |

The example shows the initial in-memory demo data. The actual response can change after product creation, update, or deletion.

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X GET "{base_url}/api/v1/products"
```

### 6.2 Get product by ID

Retrieves the details of a product using its unique product ID.

| Property | Value |
|---|---|
| Method | `GET` |
| Endpoint | `/api/v1/products/{product_id}` |
| Authentication | Not required |
| Success status | `200 OK` |

#### Path parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `product_id` | integer | Yes | Unique identifier of the product to retrieve. |

#### Example request

```http
GET /api/v1/products/1
```

#### Successful response — `200 OK`

```json
{
  "id": 1,
  "name": "Laptop",
  "description": "Demo laptop",
  "price": 55000.0,
  "stock": 10
}
```

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `404 Not Found` | No product exists with the supplied ID. | `{"detail":"Product not found"}` |
| `422 Unprocessable Entity` | The product ID cannot be converted to an integer. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X GET "{base_url}/api/v1/products/1"
```

### 6.3 Create product

Creates a new product. A valid Bearer token is required. The API generates the product ID and stores the new product in memory.

| Property | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/api/v1/products` |
| Authentication | Required |
| Request content type | `application/json` |
| Success status | `201 Created` |

#### Headers

```http
Authorization: Bearer demo-access-token
Content-Type: application/json
```

#### Request body

```json
{
  "name": "Monitor",
  "description": "24-inch LED monitor",
  "price": 12000.0,
  "stock": 8
}
```

| Field | Type | Required | Constraints | Description |
|---|---|---:|---|---|
| `name` | string | Yes | 1–100 characters | Name of the product. |
| `description` | string | No | Maximum 500 characters; defaults to an empty string. | Description of the product. |
| `price` | number | Yes | Greater than 0; infinity and NaN are not allowed. | Product price. |
| `stock` | integer | Yes | 0 or greater. | Available stock quantity. |

#### Successful response — `201 Created`

```json
{
  "id": 4,
  "name": "Monitor",
  "description": "24-inch LED monitor",
  "price": 12000.0,
  "stock": 8
}
```

The `id` is generated by the API. The response includes the generated ID and the submitted product fields.

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `422 Unprocessable Entity` | Required product data is missing or invalid. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

**Implementation note:** The route decorator documents `409 Product conflict`, but the supplied create-product function does not implement a conflict check. Do not describe a specific `409` trigger as implemented behavior without confirmation from the developer.

#### cURL example

```bash
curl -X POST "{base_url}/api/v1/products" \
  -H "Authorization: Bearer demo-access-token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Monitor",
    "description": "24-inch LED monitor",
    "price": 12000.0,
    "stock": 8
  }'
```

### 6.4 Update product

Updates one or more fields of an existing product. A valid Bearer token is required. Only fields included in the request body are updated.

| Property | Value |
|---|---|
| Method | `PUT` |
| Endpoint | `/api/v1/products/{product_id}` |
| Authentication | Required |
| Request content type | `application/json` |
| Success status | `200 OK` |

#### Path parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `product_id` | integer | Yes | ID of the product to update. |

#### Headers

```http
Authorization: Bearer demo-access-token
Content-Type: application/json
```

#### Request body

All fields are optional. Include at least one field to update.

```json
{
  "name": "Updated Laptop",
  "description": "Updated laptop description",
  "price": 58000.0,
  "stock": 12
}
```

| Field | Type | Required | Constraints | Description |
|---|---|---:|---|---|
| `name` | string or null | No | If supplied as a string: 1–100 characters. | Updated product name. |
| `description` | string or null | No | Maximum 500 characters. | Updated product description. |
| `price` | number or null | No | If supplied as a number: greater than 0; infinity and NaN are not allowed. | Updated product price. |
| `stock` | integer or null | No | If supplied as an integer: 0 or greater. | Updated stock quantity. |

#### Successful response — `200 OK`

```json
{
  "id": 1,
  "name": "Updated Laptop",
  "description": "Updated laptop description",
  "price": 58000.0,
  "stock": 12
}
```

The response contains the complete updated product object, including fields that were not changed.

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `400 Bad Request` | The request body contains no fields to update. | `{"detail":"Provide at least one field to update"}` |
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `404 Not Found` | The product ID does not exist. | `{"detail":"Product not found"}` |
| `422 Unprocessable Entity` | Product ID or supplied field values are invalid. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

**Implementation note:** The route decorator lists `422` and does not explicitly list `400`, but the function itself raises `400` when no update fields are provided. This documentation includes the behavior implemented by the function.

#### cURL example

```bash
curl -X PUT "{base_url}/api/v1/products/1" \
  -H "Authorization: Bearer demo-access-token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Laptop",
    "price": 58000.0
  }'
```

### 6.5 Delete product

Deletes a product using its ID. A product cannot be deleted while it is referenced by an existing cart.

| Property | Value |
|---|---|
| Method | `DELETE` |
| Endpoint | `/api/v1/products/{product_id}` |
| Authentication | Required |
| Success status | `204 No Content` |

#### Path parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `product_id` | integer | Yes | ID of the product to delete. |

#### Headers

```http
Authorization: Bearer demo-access-token
```

#### Successful response — `204 No Content`

The product is deleted. The response has no response body.

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `404 Not Found` | The product ID does not exist. | `{"detail":"Product not found"}` |
| `409 Conflict` | The product is referenced by an existing cart. | `{"detail":"Remove the product from carts before deleting it"}` |
| `422 Unprocessable Entity` | The product ID is invalid. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X DELETE "{base_url}/api/v1/products/1" \
  -H "Authorization: Bearer demo-access-token"
```

---

## 7. Cart APIs

Cart endpoints allow an authenticated user to view and manage their shopping cart. The supplied implementation maintains one cart per demo user.

### Cart response object

The cart response has the following structure:

```json
{
  "items": [
    {
      "product_id": 1,
      "name": "Laptop",
      "unit_price": 55000.0,
      "quantity": 2,
      "subtotal": 110000.0
    }
  ],
  "total_items": 2,
  "total_amount": 110000.0
}
```

| Field | Type | Description |
|---|---|---|
| `items` | array of objects | Products currently in the cart. |
| `items[].product_id` | integer | Product identifier. |
| `items[].name` | string | Product name. |
| `items[].unit_price` | number | Current unit price of the product. |
| `items[].quantity` | integer | Quantity of the product in the cart. |
| `items[].subtotal` | number | Unit price multiplied by quantity, rounded to two decimal places. |
| `total_items` | integer | Sum of quantities across all cart items. |
| `total_amount` | number | Sum of item subtotals, rounded to two decimal places. |

### 7.1 Get current cart

Retrieves the authenticated user's current shopping cart. If the cart is empty, the API returns an empty items array and zero totals.

| Property | Value |
|---|---|
| Method | `GET` |
| Endpoint | `/api/v1/cart` |
| Authentication | Required |
| Success status | `200 OK` |

#### Headers

```http
Authorization: Bearer demo-access-token
```

#### Request parameters

No path parameters, query parameters, or request body are required.

#### Successful response — `200 OK`

Example with an empty cart:

```json
{
  "items": [],
  "total_items": 0,
  "total_amount": 0.0
}
```

Example with cart items:

```json
{
  "items": [
    {
      "product_id": 1,
      "name": "Laptop",
      "unit_price": 55000.0,
      "quantity": 2,
      "subtotal": 110000.0
    }
  ],
  "total_items": 2,
  "total_amount": 110000.0
}
```

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X GET "{base_url}/api/v1/cart" \
  -H "Authorization: Bearer demo-access-token"
```

### 7.2 Add product to cart

Adds a product and its requested quantity to the authenticated user's cart. If the product is already in the cart, the requested quantity is added to its existing quantity.

| Property | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/api/v1/cart/items` |
| Authentication | Required |
| Request content type | `application/json` |
| Success status | `201 Created` |

#### Headers

```http
Authorization: Bearer demo-access-token
Content-Type: application/json
```

#### Request body

```json
{
  "product_id": 1,
  "quantity": 2
}
```

| Field | Type | Required | Constraints | Description |
|---|---|---:|---|---|
| `product_id` | integer | Yes | Greater than 0. | ID of the product to add. |
| `quantity` | integer | Yes | 1–100. | Quantity to add to the cart. |

#### Successful response — `201 Created`

The API returns the updated cart using the cart response structure.

```json
{
  "items": [
    {
      "product_id": 1,
      "name": "Laptop",
      "unit_price": 55000.0,
      "quantity": 2,
      "subtotal": 110000.0
    }
  ],
  "total_items": 2,
  "total_amount": 110000.0
}
```

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `404 Not Found` | The requested product does not exist. | `{"detail":"Product not found"}` |
| `409 Conflict` | Adding the requested quantity would exceed available stock. | `{"detail":"Requested quantity exceeds available stock"}` |
| `422 Unprocessable Entity` | Product ID or quantity is invalid. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X POST "{base_url}/api/v1/cart/items" \
  -H "Authorization: Bearer demo-access-token" \
  -H "Content-Type: application/json" \
  -d '{
    "product_id": 1,
    "quantity": 2
  }'
```

### 7.3 Update cart item quantity

Changes the quantity of a product that is already in the authenticated user's cart. The new quantity replaces the existing quantity; it is not added to it.

| Property | Value |
|---|---|
| Method | `PATCH` |
| Endpoint | `/api/v1/cart/items/{product_id}` |
| Authentication | Required |
| Request content type | `application/json` |
| Success status | `200 OK` |

#### Path parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `product_id` | integer | Yes | ID of the product whose cart quantity must be updated. |

#### Headers

```http
Authorization: Bearer demo-access-token
Content-Type: application/json
```

#### Request body

```json
{
  "quantity": 3
}
```

| Field | Type | Required | Constraints | Description |
|---|---|---:|---|---|
| `quantity` | integer | Yes | 1–100. | New quantity for the cart item. |

#### Successful response — `200 OK`

The API returns the updated cart using the cart response structure.

```json
{
  "items": [
    {
      "product_id": 1,
      "name": "Laptop",
      "unit_price": 55000.0,
      "quantity": 3,
      "subtotal": 165000.0
    }
  ],
  "total_items": 3,
  "total_amount": 165000.0
}
```

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `404 Not Found` | Product does not exist. | `{"detail":"Product not found"}` |
| `404 Not Found` | Product exists but is not in the user's cart. | `{"detail":"Item not found in cart"}` |
| `409 Conflict` | Requested quantity exceeds available stock. | `{"detail":"Requested quantity exceeds available stock"}` |
| `422 Unprocessable Entity` | Product ID or quantity is invalid. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X PATCH "{base_url}/api/v1/cart/items/1" \
  -H "Authorization: Bearer demo-access-token" \
  -H "Content-Type: application/json" \
  -d '{
    "quantity": 3
  }'
```

### 7.4 Remove item from cart

Removes a specified product from the authenticated user's cart.

| Property | Value |
|---|---|
| Method | `DELETE` |
| Endpoint | `/api/v1/cart/items/{product_id}` |
| Authentication | Required |
| Success status | `200 OK` |

#### Path parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `product_id` | integer | Yes | ID of the product to remove from the cart. |

#### Headers

```http
Authorization: Bearer demo-access-token
```

#### Successful response — `200 OK`

The API returns the updated cart after removing the item.

```json
{
  "items": [],
  "total_items": 0,
  "total_amount": 0.0
}
```

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `404 Not Found` | The specified item is not in the cart. | `{"detail":"Item not found in cart"}` |
| `422 Unprocessable Entity` | Product ID is invalid. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X DELETE "{base_url}/api/v1/cart/items/1" \
  -H "Authorization: Bearer demo-access-token"
```

### 7.5 Clear cart

Removes all items from the authenticated user's cart.

| Property | Value |
|---|---|
| Method | `DELETE` |
| Endpoint | `/api/v1/cart` |
| Authentication | Required |
| Success status | `200 OK` |

#### Headers

```http
Authorization: Bearer demo-access-token
```

#### Request parameters

No path parameters, query parameters, or request body are required.

#### Successful response — `200 OK`

```json
{
  "items": [],
  "total_items": 0,
  "total_amount": 0.0
}
```

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X DELETE "{base_url}/api/v1/cart" \
  -H "Authorization: Bearer demo-access-token"
```

---

## 8. Order APIs

Order endpoints allow an authenticated user to create an order from their cart, retrieve their orders, and cancel an order that is still in `PLACED` status.

### Order object

```json
{
  "order_id": 1001,
  "username": "demo",
  "items": [
    {
      "product_id": 1,
      "name": "Laptop",
      "unit_price": 55000.0,
      "quantity": 2,
      "subtotal": 110000.0
    }
  ],
  "total_amount": 110000.0,
  "status": "PLACED",
  "created_at": "2026-10-25T06:30:00+00:00"
}
```

| Field | Type | Description |
|---|---|---|
| `order_id` | integer | Unique order identifier. |
| `username` | string | Username associated with the order. |
| `items` | array of objects | Products included in the order. |
| `items[].product_id` | integer | Product identifier. |
| `items[].name` | string | Product name at order creation. |
| `items[].unit_price` | number | Unit price used to calculate the order. |
| `items[].quantity` | integer | Quantity ordered. |
| `items[].subtotal` | number | Unit price multiplied by quantity, rounded to two decimal places. |
| `total_amount` | number | Total order amount, rounded to two decimal places. |
| `status` | string | Order status. New orders use `PLACED`; cancelled orders use `CANCELLED`. |
| `created_at` | string | Order creation timestamp in ISO 8601 format. |
| `cancelled_at` | string | Cancellation timestamp in ISO 8601 format; returned after successful cancellation. |

### 8.1 Create order from cart

Creates an order using all items currently in the authenticated user's cart. The API validates product availability and stock before reducing inventory. After successful order creation, the cart is cleared.

| Property | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/api/v1/orders` |
| Authentication | Required |
| Request body | None |
| Success status | `201 Created` |

#### Headers

```http
Authorization: Bearer demo-access-token
```

#### Request parameters

No path parameters, query parameters, or request body are required. The order is created from the current cart.

#### Successful response — `201 Created`

```json
{
  "order_id": 1001,
  "username": "demo",
  "items": [
    {
      "product_id": 1,
      "name": "Laptop",
      "unit_price": 55000.0,
      "quantity": 2,
      "subtotal": 110000.0
    }
  ],
  "total_amount": 110000.0,
  "status": "PLACED",
  "created_at": "2026-10-25T06:30:00+00:00"
}
```

The order ID is generated by the API. The order total is calculated using current product prices. Stock is reduced after all cart items pass validation, and the cart is cleared after the order is stored.

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `400 Bad Request` | The user's cart is empty. | `{"detail":"Cart is empty"}` |
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `404 Not Found` | A product in the cart is unavailable. | `{"detail":"Product {product_id} is unavailable"}` |
| `409 Conflict` | A cart item's quantity exceeds the product's available stock. | `{"detail":"Insufficient stock for product {product_id}"}` |
| `422 Unprocessable Entity` | Invalid request. No request fields are defined for this operation in the supplied implementation. | Common validation error format if request validation is triggered. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X POST "{base_url}/api/v1/orders" \
  -H "Authorization: Bearer demo-access-token"
```

### 8.2 Get current user's orders

Retrieves all orders associated with the authenticated user.

| Property | Value |
|---|---|
| Method | `GET` |
| Endpoint | `/api/v1/orders` |
| Authentication | Required |
| Success status | `200 OK` |

#### Headers

```http
Authorization: Bearer demo-access-token
```

#### Request parameters

No path parameters, query parameters, or request body are required.

#### Successful response — `200 OK`

```json
{
  "items": [
    {
      "order_id": 1001,
      "username": "demo",
      "items": [
        {
          "product_id": 1,
          "name": "Laptop",
          "unit_price": 55000.0,
          "quantity": 2,
          "subtotal": 110000.0
        }
      ],
      "total_amount": 110000.0,
      "status": "PLACED",
      "created_at": "2026-10-25T06:30:00+00:00"
    }
  ],
  "total": 1
}
```

| Field | Type | Description |
|---|---|---|
| `items` | array of order objects | Orders belonging to the authenticated user. |
| `total` | integer | Number of orders returned. |

If the user has no orders, the API returns an empty `items` array and `total: 0`.

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X GET "{base_url}/api/v1/orders" \
  -H "Authorization: Bearer demo-access-token"
```

### 8.3 Get order by ID

Retrieves a specific order using its order ID. The API returns an order only if it belongs to the authenticated user.

| Property | Value |
|---|---|
| Method | `GET` |
| Endpoint | `/api/v1/orders/{order_id}` |
| Authentication | Required |
| Success status | `200 OK` |

#### Path parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `order_id` | integer | Yes | Unique identifier of the order to retrieve. |

#### Headers

```http
Authorization: Bearer demo-access-token
```

#### Example request

```http
GET /api/v1/orders/1001
```

#### Successful response — `200 OK`

```json
{
  "order_id": 1001,
  "username": "demo",
  "items": [
    {
      "product_id": 1,
      "name": "Laptop",
      "unit_price": 55000.0,
      "quantity": 2,
      "subtotal": 110000.0
    }
  ],
  "total_amount": 110000.0,
  "status": "PLACED",
  "created_at": "2026-10-25T06:30:00+00:00"
}
```

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `404 Not Found` | Order does not exist or does not belong to the authenticated user. | `{"detail":"Order not found"}` |
| `422 Unprocessable Entity` | The order ID is invalid. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X GET "{base_url}/api/v1/orders/1001" \
  -H "Authorization: Bearer demo-access-token"
```

### 8.4 Cancel order

Cancels an order that belongs to the authenticated user and is currently in `PLACED` status. When cancellation succeeds, the API restores the ordered quantities to product stock.

| Property | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/api/v1/orders/{order_id}/cancel` |
| Authentication | Required |
| Success status | `200 OK` |

#### Path parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `order_id` | integer | Yes | ID of the order to cancel. |

#### Headers

```http
Authorization: Bearer demo-access-token
```

#### Request body

No request body is required.

#### Example request

```http
POST /api/v1/orders/1001/cancel
```

#### Successful response — `200 OK`

```json
{
  "order_id": 1001,
  "username": "demo",
  "items": [
    {
      "product_id": 1,
      "name": "Laptop",
      "unit_price": 55000.0,
      "quantity": 2,
      "subtotal": 110000.0
    }
  ],
  "total_amount": 110000.0,
  "status": "CANCELLED",
  "created_at": "2026-10-25T06:30:00+00:00",
  "cancelled_at": "2026-10-25T06:35:00+00:00"
}
```

The timestamp values above are illustrative. The API generates the actual timestamps at runtime.

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `401 Unauthorized` | Bearer token is missing or invalid. | `{"detail":"Missing Bearer token"}` or `{"detail":"Invalid or expired access token"}` |
| `404 Not Found` | Order does not exist or does not belong to the authenticated user. | `{"detail":"Order not found"}` |
| `409 Conflict` | Order status is not `PLACED`. | `{"detail":"Only orders with PLACED status can be cancelled"}` |
| `422 Unprocessable Entity` | The order ID is invalid. | Common validation error format. |
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X POST "{base_url}/api/v1/orders/1001/cancel" \
  -H "Authorization: Bearer demo-access-token"
```

---

## 9. Health check

### Check API health

Checks whether the API service is running. This endpoint does not require authentication.

| Property | Value |
|---|---|
| Method | `GET` |
| Endpoint | `/health` |
| Authentication | Not required |
| Success status | `200 OK` |

#### Request parameters

No path parameters, query parameters, or request body are required.

#### Successful response — `200 OK`

```json
{
  "status": "healthy",
  "service": "E-Commerce Demo API",
  "version": "1.0.0"
}
```

| Field | Type | Description |
|---|---|---|
| `status` | string | Health status of the API. The successful value is `healthy`. |
| `service` | string | Name of the API service. |
| `version` | string | API version. |

#### Error responses

| Status | Condition | Response body |
|---|---|---|
| `500 Internal Server Error` | Unexpected server-side error. | `{"detail":"Internal server error"}` |

#### cURL example

```bash
curl -X GET "{base_url}/health"
```

---

## 10. Important implementation notes

The following points are relevant when using or publishing this API documentation:

1. **In-memory storage:** Users, products, carts, and orders are stored in application memory. Data is not persisted to a database and may be reset when the application restarts.
2. **Demo authentication only:** The implementation uses a fixed token (`demo-access-token`). It is explicitly not secure authentication and must not be presented as production-grade security.
3. **No configured token expiry:** The login response returns `expires_in: null`.
4. **Protected operations:** Product creation, product update, product deletion, all cart operations, and all order operations require a valid Bearer token.
5. **Product deletion restriction:** A product cannot be deleted while it is present in a cart.
6. **Stock validation:** Cart additions, cart quantity updates, and order creation validate requested quantities against available stock.
7. **Order creation:** The API validates all cart products before reducing stock. After successful order creation, the cart is cleared.
8. **Order cancellation:** Only orders in `PLACED` status can be cancelled. Cancellation restores stock for the products in the order.
9. **User-specific orders:** Order retrieval and cancellation are restricted to orders associated with the authenticated user.
10. **No pagination parameters:** The supplied list endpoints do not implement pagination or filtering query parameters.
11. **API documentation interfaces:** The FastAPI application exposes Swagger UI at `/docs`, ReDoc at `/redoc`, and the OpenAPI JSON document at `/openapi.json`, relative to the server root.

### Source-code items to confirm with the developer

These are documentation-review observations, not additional API behavior:

- The create-product route declares a `409 Product conflict` response, but the function does not currently raise that response.
- The update-product function raises `400 Bad Request` for an empty update body, although `400` is not listed in its route response declarations.
- The `ErrorResponse` Pydantic model is defined in the source, but the custom `422` validation handler returns an object containing both `detail` and `errors`; it is not the same shape as `ErrorResponse`.
- The login endpoint returns a fixed demo token. Confirm whether this is intentionally a demo-only implementation before documenting it as a real authentication flow.
- The API source does not specify a production base URL, deployment environment, rate limits, or pagination behavior.

---

**End of E-Commerce Demo API Endpoint Reference**

