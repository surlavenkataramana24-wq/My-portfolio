# E-Commerce Demo API Documentation

A REST API documentation portfolio project demonstrating API documentation practices, including endpoint descriptions, authentication, request parameters, JSON request and response examples, HTTP status codes, error handling, and cURL examples.

> **Portfolio project:** This documentation was prepared as a technical-writing sample based on a demonstration FastAPI application. The API uses in-memory demo data and is not intended for production use.

## Project Overview

The E-Commerce Demo API provides endpoints for managing the core operations of a simple e-commerce application:

- **Login:** Authenticate a demo user and obtain a Bearer access token.
- **Products:** Retrieve, create, update, and delete products.
- **Cart:** View the cart, add products, update quantities, remove items, and clear the cart.
- **Orders:** Create orders, retrieve orders, and cancel eligible orders.
- **Health:** Check whether the API service is running.

## Documentation

The detailed endpoint reference is available here:

**[E-Commerce API Endpoint Reference](ecommerce_api_endpoint_documentation.md)**

The endpoint reference includes:

- API overview, base URL conventions, and HTTP methods
- Authentication requirements and authorization headers
- Endpoint descriptions and HTTP methods
- Path parameters and request body fields
- JSON request and response examples
- Success and error HTTP status codes
- Validation rules and error response formats
- cURL request examples
- Implementation notes where behavior in the source code needs clarification

## API Endpoint Summary

| Resource | Method | Endpoint | Description |
|---|---|---|---|
| Login | POST | `/api/v1/login` | Authenticate a demo user |
| Products | GET | `/api/v1/products` | Retrieve all products |
| Products | GET | `/api/v1/products/{product_id}` | Retrieve a product by ID |
| Products | POST | `/api/v1/products` | Create a product |
| Products | PUT | `/api/v1/products/{product_id}` | Update a product |
| Products | DELETE | `/api/v1/products/{product_id}` | Delete a product |
| Cart | GET | `/api/v1/cart` | Retrieve the current cart |
| Cart | POST | `/api/v1/cart/items` | Add a product to the cart |
| Cart | PATCH | `/api/v1/cart/items/{product_id}` | Update cart item quantity |
| Cart | DELETE | `/api/v1/cart/items/{product_id}` | Remove an item from the cart |
| Cart | DELETE | `/api/v1/cart` | Clear the cart |
| Orders | POST | `/api/v1/orders` | Create an order from the cart |
| Orders | GET | `/api/v1/orders` | Retrieve the current user's orders |
| Orders | GET | `/api/v1/orders/{order_id}` | Retrieve an order by ID |
| Orders | POST | `/api/v1/orders/{order_id}/cancel` | Cancel an eligible order |
| Health | GET | `/health` | Check API health |

## API Details

| Property | Details |
|---|---|
| API name | E-Commerce Demo API |
| API version | 1.0.0 |
| API style | REST |
| Data format | JSON |
| Authentication | HTTP Bearer token for protected endpoints |
| API framework | FastAPI |
| Data storage | In-memory demo data |

The supplied application exposes interactive API documentation through Swagger UI at `/docs`, ReDoc at `/redoc`, and the OpenAPI JSON document at `/openapi.json`, when the application is running.

## Tools and Technologies

- Markdown
- REST API concepts
- HTTP methods and status codes
- JSON
- OpenAPI
- Swagger UI
- FastAPI (used by the demonstration application)
- Git and GitHub

## Documentation Approach

The documentation was organized to help API consumers understand and use each endpoint. The work includes:

1. Reviewing the API source and identifying available endpoints.
2. Grouping endpoints by resource.
3. Identifying authentication requirements.
4. Documenting path parameters and request fields.
5. Preparing representative JSON request and response examples.
6. Explaining success, validation, and error responses.
7. Providing cURL examples for request execution.
8. Recording implementation notes to distinguish implemented behavior from route-level declarations.

## Important Notes

- This is a learning and portfolio demonstration, not a production-ready e-commerce service.
- Data is stored in memory and may reset when the application restarts.
- The authentication mechanism uses a fixed demo token and is not secure for production.
- Replace `{base_url}` in request examples with the URL of the running API instance.
- Do not use real credentials, payment details, or sensitive customer information with this demo.

## Author

**Venkata Ramana Surla**  
Technical Writer | API Documentation Portfolio

GitHub: [surlavenkataramana24-wq](https://github.com/surlavenkataramana24-wq)

---

*This repository is a technical-writing portfolio sample demonstrating structured API reference documentation.*
