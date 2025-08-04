# Laravel Action and Service Guidelines

A clean, scalable architecture for organizing business logic in PHP applications using Action and Service patterns.

## Overview

This repository demonstrates a structured approach to building maintainable PHP applications by separating business
logic into **Actions** (complete operations) and **Services** (reusable logic).

## Why Use This Pattern?

It’s all about maintainability. Actions keep your code focused on single, complete tasks. Services handle the
repetitive, lower-level work.
This separation makes your application easier to test, debug, and extend. You want less clutter, more clarity.

## Documentation

**[Read the full guidelines](action-and-service-guidelines.md)** — includes examples, naming conventions, and best
practices. Dive in for the specifics.

## Quick Start

### Actions - Complete Business Operations

```php
// Creates a complete order with validation, payment, notifications
OrderCreateAction

// Adds item to cart with all side effects
CartItemAddAction

// Processes search with filtering and sorting
SearchAction
```

### Services - Reusable Business Logic

```php
// Validates cart items (used by multiple actions)
CartItemValidator

// Figures out when stuff gets delivered
DeliveryScheduleService

// Handles external API calls
TehnomirApi
```

## Project Structure

```
app/
├── Actions/    # Complete business operations
│   ├── Cart/
│   ├── Order/
│   └── Search/
├── Services/   # Reusable business logic
│   ├── Cart/
│   ├── Order/
│   └── External/
└── ...
```

## 🤔 Quick Decision Guide

| Requirement                         | Where it goes | Example                   |
|-------------------------------------|---------------|---------------------------|
| Initiate a user-driven workflow     | **Action**    | `OrderCreateAction`       |
| Logic or math you use everywhere    | **Service**   | `DeliveryScheduleService` |
| Execute a complete business process | **Action**    | `CartCheckoutAction`      |
| Data validation or transformation   | **Service**   | `CartItemValidator`       |

Use Actions when orchestrating workflows or business processes initiated by user requests. Services are best for logic,
validation, or calculations that need to be reused across multiple features. This separation improves maintainability
and clarity in project organization.

## Naming Conventions (Because Names Actually Matter)

### Actions: `[Domain][Object][Verb]Action`

- ✅ `OrderCreateAction`
- ✅ `CartItemAddAction`
- ❌ `CreateOrderAction` (Nope, not like this. Don’t flip it.)

### Services: `[Domain][Purpose]Service`

- ✅ `DeliveryScheduleService`
- ✅ `CartItemValidator`
- ❌ `DataService` (What even is this? Be specific.)

## Guiding Principles

- **Actions**: Responsible for orchestrating workflows and managing side effects.
- **Services**: Encapsulate pure, reusable business logic—no side effects.
- **Single Responsibility**: Each class is designed for one clear purpose.
- **Dependency Injection**: Employ constructor injection for maintainability.
- **Type Safety**: Explicitly declare types and return values.

## How to Test Stuff

- **Actions:** Hit them with integration tests. Make sure the full workflow doesn’t explode.
- **Services:** Hit with unit tests. Laser focus on the pure logic, nothing else.

## Laravel Controller—Here’s How It Looks

```php
public function store(OrderStoreRequest $request, OrderCreateAction $action): OrderResource
{
    $dto = new OrderCreateDTO(...$request->validated());
    $order = $action->handle($dto, auth()->id());
    return OrderResource::make($order);
}
```

## Learn More

- [Full Guidelines](action-and-service-guidelines.md) - Complete guide

