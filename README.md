# E-Commerce Microservices Backend

A containerized e-commerce backend built with **Python and Flask**, designed around a **microservices architecture**.

The system separates customer management, inventory, sales, and product reviews into independent services that communicate through REST APIs. It demonstrates backend API development, service-to-service communication, database integration, testing, containerization, logging, and error handling.

## Architecture

The application is composed of four independent services:

| Service | Port | Responsibility |
|---|---:|---|
| **Customer Service** | `5001` | Customer profiles and wallet management |
| **Inventory Service** | `5002` | Product catalog and stock management |
| **Sales Service** | `5003` | Product discovery and purchase processing |
| **Reviews Service** | `5000` | Product reviews and moderation |

Each service owns its API and persistence layer and is containerized independently.

```text
                         ┌──────────────────┐
                         │ Customer Service │
                         │      :5001       │
                         └────────▲─────────┘
                                  │
                                  │ customer / wallet
                                  │
┌──────────────────┐      ┌───────┴──────────┐
│ Inventory Service│◄────►│   Sales Service   │
│      :5002       │      │       :5003       │
└────────▲─────────┘      └───────────────────┘
         │
         │ validates products
         │
┌────────┴─────────┐
│ Reviews Service  │
│      :5000       │
└──────────────────┘
```

All services run on a shared Docker network, allowing them to communicate while remaining logically separated.

## Key Features

- **Microservices architecture** with four independently containerized Flask services
- **RESTful APIs** for customers, inventory, purchases, and reviews
- **Inter-service communication** through HTTP requests
- **Database persistence** using Flask-SQLAlchemy / SQLAlchemy
- **Customer wallet management** for e-commerce transactions
- **Inventory and stock validation** during purchases
- **Purchase workflow** coordinating multiple backend services
- **Review submission and moderation**
- **Request validation and structured HTTP error responses**
- **Service health-check endpoints**
- **Application logging** for operational visibility and debugging
- **Automated API testing** with pytest
- **Docker Compose orchestration** for running the complete system locally

## Purchase Workflow

A purchase demonstrates the interaction between multiple services.

```text
Client
  │
  │ POST /sales
  ▼
Sales Service
  │
  ├──► Customer Service
  │      Verify customer
  │      Check wallet balance
  │
  ├──► Inventory Service
  │      Find product
  │      Check available stock
  │
  ├── Calculate total purchase price
  │
  ├── Validate sufficient funds
  │
  ├── Update inventory
  │
  └── Update customer wallet
```

This workflow allows business responsibilities to remain separated while still supporting transactions that span multiple services.

## Services

### Customer Service

Manages customer accounts and wallet balances.

Core capabilities include:

- Register customers
- Retrieve customer information
- Update customer profiles
- Delete customers
- Add funds to a customer's wallet
- Deduct funds from a customer's wallet
- Validate incoming customer data
- Report service/database health

### Inventory Service

Maintains the product catalog and stock levels.

Core capabilities include:

- Add products
- Retrieve inventory
- Update product information
- Delete products
- Track available stock
- Validate price and quantity inputs
- Report service/database health

### Sales Service

Coordinates the purchasing process and interacts with both the Customer and Inventory services.

Core capabilities include:

- List products currently available for purchase
- Retrieve product details
- Process purchases
- Validate customers
- Validate inventory availability
- Check customer wallet balances
- Calculate transaction totals
- Store purchase records
- Handle failures from dependent services

### Reviews Service

Manages customer feedback for products.

Core capabilities include:

- Submit product reviews
- Validate ratings
- Verify that customers and products exist
- Update and delete reviews
- Retrieve reviews by customer
- Retrieve approved reviews by product
- Moderate submitted reviews
- Check the health of dependent services

## Tech Stack

**Backend**
- Python
- Flask
- Flask Blueprints

**Data**
- SQLAlchemy
- Flask-SQLAlchemy

**Architecture**
- REST APIs
- Microservices
- Service-to-service HTTP communication

**Infrastructure**
- Docker
- Docker Compose
- Docker bridge networking

**Testing & Reliability**
- pytest
- Health-check endpoints
- Application logging
- Input validation
- HTTP error handling

## Project Structure

```text
E-Commerce-Microservices-Backend/
│
├── Customer_services/
│   ├── app.py
│   ├── routes.py
│   ├── models.py
│   ├── db.py
│   ├── tests_customers.py
│   └── Dockerfile
│
├── Inventory_service/
│   ├── app.py
│   ├── routes.py
│   ├── models.py
│   ├── db.py
│   ├── tests_inventory.py
│   └── Dockerfile
│
├── Sales/
│   ├── app.py
│   ├── routes.py
│   ├── models.py
│   ├── db.py
│   ├── test_sales.py
│   └── Dockerfile
│
├── Reviews_service/
│   ├── app.py
│   ├── routes.py
│   ├── models.py
│   ├── db.py
│   └── Dockerfile
│
├── docker-compose.yml
├── requirements.txt
└── README.md
```

## Getting Started

### Prerequisites

Make sure you have installed:

- Docker
- Docker Compose
- Git

### Clone the repository

```bash
git clone https://github.com/leaazzar/E-Commerce-Microservices-Backend.git
cd E-Commerce-Microservices-Backend
```

### Start the application

Build and launch all four services:

```bash
docker compose up --build
```

Docker Compose starts the services on the following ports:

```text
Reviews Service     → localhost:5000
Customer Service    → localhost:5001
Inventory Service   → localhost:5002
Sales Service       → localhost:5003
```

To stop the application:

```bash
docker compose down
```

## Example API Usage

### Create a Customer

```bash
curl -X POST http://localhost:5001/api/v1/customers \
  -H "Content-Type: application/json" \
  -d '{
    "username": "johndoe",
    "full_name": "John Doe",
    "password": "example-password"
  }'
```

### Add an Inventory Item

```bash
curl -X POST http://localhost:5002/api/v1/inventory \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Wireless Headphones",
    "category": "Electronics",
    "price_per_item": 79.99,
    "count_in_stock": 20,
    "description": "Wireless over-ear headphones"
  }'
```

### Purchase an Item

```bash
curl -X POST http://localhost:5003/api/v1/sales \
  -H "Content-Type: application/json" \
  -d '{
    "customer_username": "johndoe",
    "item_name": "Wireless Headphones",
    "quantity": 1
  }'
```

### Submit a Review

```bash
curl -X POST http://localhost:5000/api/v1/ \
  -H "Content-Type: application/json" \
  -d '{
    "customer_username": "johndoe",
    "item_name": "Wireless Headphones",
    "rating": 5,
    "comment": "Great product!"
  }'
```

## Testing

The repository includes automated tests for the individual services.

Run tests with:

```bash
pytest
```

Tests cover service behavior, endpoint responses, input validation, and error scenarios.

## Engineering Concepts Demonstrated

This project was designed to apply several backend software-engineering concepts in a realistic, multi-service setting:

- **Service decomposition** - each business domain (customers, inventory, sales, reviews) is an independent service with its own API and database
- **Database-per-service** - services never share tables; they exchange data only through their REST APIs
- **Distributed workflows** - the purchase flow coordinates customer, wallet, and inventory checks across service boundaries
- **Failure handling** - services validate input and return structured HTTP errors, including when a dependent service is unavailable
- **Observability** - per-service logging and health-check endpoints support debugging and monitoring
- **Automated testing** - pytest suites verify endpoint behavior, validation rules, and error scenarios
- **Containerization** - each service ships with its own Dockerfile and the full system runs with a single Docker Compose command
