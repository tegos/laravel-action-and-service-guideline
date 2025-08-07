# Action and Service Guidelines with Naming Conventions

## Overview

This guideline defines when to use **Actions** and **Services** in your PHP application, provides naming conventions,
and outlines best practices for organizing and implementing them. The goal is to ensure consistency, maintainability,
and scalability in your codebase.

## When to Use Actions and Services

### **Use Actions When:**

- **Complete business operations** with clear start/end boundaries (e.g., creating an order, canceling an item).
- **Command-like operations** triggered by user actions or system events (e.g., user login, search query logging).
- **Orchestrating multiple services or sub-actions** to achieve a business goal (e.g., processing a search with filters
  and sorting).
- **Transactional workflows** requiring atomicity (e.g., creating an order with database transactions).
- **Single-use operations** specific to one context or controller (e.g., deleting a banner directory).
- **Operations with side effects** such as notifications, logging, or cache clearing (e.g., sending order issue
  notifications).
- **Complex business processes** involving multiple steps (e.g., processing a cart checkout).

**Examples:**

```php
OrderCreateAction                     // Creates an order with multiple steps
SearchAction                          // Orchestrates search with filtering and sorting
CartItemAddAction                     // Adds an item to the cart
CommunicationCloseConversationAction  // Closes a user conversation
PriceExportDownloadAction             // Downloads exported price data
```

### 🔧 **Use Services When:**

- **Reusable business logic** used across multiple actions or contexts (e.g., calculating delivery schedules).
- **Domain-specific calculations** or transformations (e.g., normalizing part names).
- **Stateless utilities** with no side effects (e.g., validating cart items).
- **Data processing or formatting** operations (e.g., transforming search results).
- **Cross-cutting concerns** like validation, logging, or API interactions (e.g., interfacing with external APIs).
- **Pure functions** that produce consistent output for the same input (e.g., price calculations).
- **Infrastructure operations** like API calls or data synchronization (e.g., syncing search indices).

**Examples:**

```php
CartItemValidator                   // Validates cart items
DeliveryScheduleService             // Calculates delivery schedules
SearchResultService                 // Processes search results
TehnomirApi                         // Handles external API interactions
PartNameNormalizerAI                // Normalizes part names
```

## Action Naming Conventions

### 📝 **General Pattern:**

```
[Domain][Object][Verb]Action
```

- **Domain**: The business context (e.g., `Order`, `Cart`, `Search`).
- **Object**: The entity being acted upon, if applicable (e.g., `Item`, `Cart`, `Query`).
- **Verb**: The action being performed (e.g., `Create`, `Cancel`, `Fetch`).
- **Suffix**: Always end with `Action`.

### ✅ **Good Examples:**

#### **CRUD Operations:**

```php
OrderCreateAction                   // Creates an order
OrderIndexAction                    // List orders
CartDestroyAction                   // Deletes a cart
SupplierUpdateAction                // Updates supplier details
UserProfileAction                   // Retrieves user profile
```

#### **Domain-Specific Operations:**

```php
OrderCancelAction                   // Cancels an order
CartItemAddAction                   // Adds an item to the cart
CartCheckoutPossibilityAction       // Checks if cart is ready for checkout
PriceImportUploadAction             // Uploads price import data
VehicleFindByVinAction              // Finds a vehicle by VIN
```

#### **Complex Operations:**

```php
OrderCartItemDTOsFetchAction        // Fetches cart item DTOs for an order
SearchBrandGroupingAction           // Groups search results by brand
PriceImportHandleDuplicateAction    // Handles duplicate price imports
StatMetricDailyCollectAction        // Collects daily metrics
```

### ❌ **Avoid These Patterns:**

```php
// Too generic
ProcessAction                       ❌ // Lacks domain context
HandleAction                        ❌ // Too vague
ExecuteAction                       ❌ // No clear purpose

// Verb-first (inconsistent with domain-first)
CreateOrderAction                   ❌ // Should be OrderCreateAction
UpdateSupplierAction                ❌ // Should be SupplierUpdateAction
SendNotificationAction              ❌ // Should be OrderIssueNotificationAction

// Missing domain context
ItemCancelAction                    ❌ // Should be OrderItemCancelAction
StatisticAction                     ❌ // Should be StatisticSearchVinDailyAction
```

## Service Naming Conventions

### 📝 **General Pattern:**

```
[Domain][Purpose]Service
```

- **Domain**: The business context (e.g., `Order`, `Cart`, `Search`).
- **Purpose**: The specific function (e.g., `Calculation`, `Validation`, `Api`).
- **Suffix**: Typically `Service`, but can use `Validator`, `Api`, or other descriptive terms for clarity.

### ✅ **Good Examples:**

#### **Calculation Services:**

```php
DeliveryScheduleService             // Calculates delivery schedules
ExchangeService                     // Handles currency and exchange rates
```

#### **Validation Services:**

```php
CartItemValidator                   // Validates cart items
RequestReturnQuantityValidator      // Validates return quantities
SearchDetailValidator               // Validates search details
```

#### **Data Services:**

```php
CartImportService                   // Imports cart data
SearchResultService                 // Processes search results
InternalPriceExportService          // Exports internal price data
CalculationLoggerService            // Logs calculations
```

#### **Integration Services:**

```php
TehnomirApi                         // Interacts with Tehnomir API
LaximoApi                           // Interacts with Laximo API
OpendatabotApi                      // Interacts with Opendatabot API
Erp1CApi                            // Interacts with ERP 1C system
```

#### **Business Logic Services:**

```php
OrderConditionService               // Manages order conditions
SupplierService                     // Handles supplier-related logic
ProductCrossService                 // Manages product cross-references
```

### ❌ **Avoid These Patterns:**

```php
// Too generic
DataService                         ❌ // Lacks specific purpose
ApiService                          ❌ // Needs specific API context (e.g., TehnomirApi)
UtilityService                      ❌ // Too vague

// Missing domain context
Validator                           ❌ // Should be CartItemValidator
Processor                           ❌ // Should be PartNamePostProcessor
```

## File Organization

### 📁 **Directory Structure:**

Organize actions and services by domain to maintain clarity and scalability. Sub-domains (e.g., `OrderItem`,
`SearchQuery`) are nested under their parent domain.

```
app/
├── Actions/
│   ├── Cart/
│   │   ├── CartCheckoutPossibilityAction.php
│   │   ├── CartItemAddAction.php
│   │   └── CartItemDTOsFetchAction.php
│   ├── Order/
│   │   ├── OrderCreateAction.php
│   │   ├── OrderCancelAction.php
│   │   └── OrderItem/
│   │       ├── OrderItemCancelAction.php
│   │       └── OrderItemUpdateStatusAction.php
│   ├── Search/
│   │   ├── SearchAction.php
│   │   ├── SearchFilterAction.php
│   │   └── SearchQuery/
│   │       ├── SearchQueryTrackLogAction.php
│   │       └── SearchQueryHistoryAction.php
│   ├── Supplier/
│   │   ├── SupplierCreateAction.php
│   │   └── SupplierScheduleBatchAction.php
├── Services/
│   ├── Cart/
│   │   ├── CartService.php
│   │   └── CartItemValidator.php
│   ├── Order/
│   │   ├── OrderConditionService.php
│   │   └── RequestReturnQuantityValidator.php
│   ├── Search/
│   │   ├── SearchResultService.php
│   │   └── SearchDetailValidator.php
│   ├── Supplier/
│   │   └── DeliveryScheduleService.php
│   ├── Tehnomir/
│   │   └── TehnomirApi.php
```

## Best Practices

### **Action Best Practices:**

1. **Single Responsibility**: Each action handles one business operation (e.g., `OrderCreateAction` creates an order,
   not updates it).
2. **Dependency Injection**: Inject dependencies via constructor (e.g., services, repositories).
3. **Handle Method**: Use `handle()` as the main entry point for execution.
4. **Type Safety**: Use strict types and return type declarations for clarity.
5. **Exception Handling**: Let business exceptions bubble up to the caller or handle them explicitly in orchestrators.
6. **Transactions**: Use database transactions for operations requiring atomicity.

```php
final class OrderCreateAction implements Actionable
{
    public function __construct(
        private readonly OrderRepository $orderRepository,
        private readonly OrderConditionService $orderConditionService,
        private readonly OrderCartItemDTOsFetchAction $cartItemFetchAction,
        private readonly OrderIssueNotificationAction $notificationAction
    ) {}

    public function handle(OrderCreateDTO $dto): Order
    {
        $this->orderConditionService->validate($dto);
        
        return DB::transaction(function() use ($dto) {
            $cartItems = $this->cartItemFetchAction->handle($dto->userId);
            $order = $this->orderRepository->create($dto);
            $this->notificationAction->handle($order);
            
            return $order;
        });
    }
}
```

### 🔧 **Service Best Practices:**

1. **Stateless**: Avoid storing state in instance variables.
2. **Pure Functions**: Ensure consistent output for the same input (e.g., `DeliveryScheduleService`).
3. **Focused Interface**: Provide specific, well-defined methods (e.g., `calculateTotal` in `CartService`).
4. **Reusable**: Design for use across multiple actions (e.g., `SearchResultService` used by multiple search actions).
5. **Testable**: Write services to be easily unit-tested in isolation.

```php
final class DeliveryScheduleService
{
    public function calculateSchedule(DateTime $orderDate, array $supplierRules): DateTime
    {
        // Pure function to calculate delivery schedule based on rules
        $schedule = $this->applySupplierRules($orderDate, $supplierRules);
        
        return $schedule;
    }
    
    private function applySupplierRules(DateTime $orderDate, array $rules): DateTime
    {
        // Implementation details
    }
}
```

## Quick Decision: Action or Service?

### 🤔 **Ask Yourself:**

1. **Is this a complete business operation?** → Action (e.g., `OrderCreateAction`).
2. **Will this be reused across multiple actions?** → Service (e.g., `CartItemValidator`).
3. **Does this have side effects (e.g., notifications, logging)?** → Action (e.g., `OrderIssueNotificationAction`).
4. **Is this pure data processing or calculation?** → Service (e.g., `DeliveryScheduleService`).
5. **Is this triggered by a user or event?** → Action (e.g., `CartItemAddAction`).
6. **Is this domain-specific logic or infrastructure?** → Service (e.g., `TehnomirApi`).

### 📊 **Decision Matrix:**

| Characteristic         | Action                                                          | Service                                                              |
|------------------------|-----------------------------------------------------------------|----------------------------------------------------------------------|
| **Primary Purpose**    | Execute complete business operations                            | Provide reusable business logic and utilities                        |
| **Scope**              | End-to-end business process (create order, process checkout)    | Single responsibility within domain (validate, calculate, transform) |
| **Reusability**        | Usually one-off, context-specific                               | Designed for reuse across multiple actions/contexts                  |
| **Side Effects**       | Often has side effects (notifications, logging, cache clearing) | Pure/stateless operations with no side effects                       |
| **Business Flow**      | Complete start-to-finish thing                                  | Partial step, not the whole journey                                  |
| **What Triggers It?**  | User, events, HTTP calls                                        | Other code calls it (like Actions or other Services)                 |
| **Complexity Level**   | High - orchestrates multiple steps/components                   | Low to Medium - focused on specific task                             |
| **How Do You Run It?** | `handle()` is your go-to                                        | Multiple public methods, pick your poison                            |
| **Laravel Style**      | Call it straight from your controller                           | Inject it into Actions                                               |
| **Where's the Logic?** | Coordinates the business process                                | Holds the domain-specific know-how                                   |

### Examples: actions and services

| **Actions: Complete Business Operations**                                                     | **Services: Reusable Business Logic**                                        |
|-----------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|
| **OrderCreateAction** - Creates an order, handles payment, inventory, fires off notifications | **CartItemValidator** - Checks if cart items play by your business rules     |
| **CartCheckoutPossibilityAction** - Can you check out? Picks delivery dates                   | **DeliveryScheduleService** - Figures out delivery dates from supplier logic |
| **VehicleFindByVinAction** - Digs through APIs, normalizes, caches                            | **PartNameNormalizerAI** - Makes part names less weird with some AI magic    |
| **PriceImportUploadAction** - Handles file uploads, catches duplicates                        | **TehnomirApi** - Talks to external APIs and deals with their messiness      |
| **SearchBrandGroupingAction** - Runs search, filters, groups brands                           | **SearchResultService** - Cleans up and tweaks your search results           |
| **StatMetricDailyCollectAction** - Scoops up metrics, stores them, fires off reports          | **OrderConditionService** - Calculates discounts, shipping, payment terms    |

## Action Composition & Splitting

### 🔗 **Composing Actions**

Actions can use both **Services** and other **Actions** as dependencies:

```php
final class OrderCreateAction implements Actionable
{
    public function __construct(
        // Services for reusable logic
        private readonly UserService $userService,
        private readonly OrderConditionService $orderConditionService,
        
        // Sub-actions for discrete operations
        private readonly OrderCartItemDTOsFetchAction $cartItemFetchAction,
        private readonly OrderItemNotificationAction $notificationAction
    ) {}
    
    public function handle(OrderCreateDTO $dto, int $userId): Order
    {
        // Use services for calculations
        $conditions = $this->orderConditionService->calculateConditions($user, $items);
        
        // Use sub-actions for discrete steps
        $cartItems = $this->cartItemFetchAction->handle($dto->items, $userId);
        
        return DB::transaction(function() use ($dto, $cartItems) {
            $order = Order::create([...]);
            
            // Side effects via sub-actions
            $this->notificationAction->handle($order);
            
            return $order;
        });
    }
}
```

### ✂️ **When to Split Actions**

Split large actions when they have:

- **Multiple unrelated responsibilities** (user validation + payment + inventory)
- **Reusable components** that other actions could use
- **Different transaction boundaries** (order creation vs payment processing)
- **Complex conditional logic** that makes testing difficult

```php
// ❌ Too complex - handles too many concerns
OrderProcessAction

// ✅ Split into focused actions
OrderCreateAction           // Creates order
OrderPaymentProcessAction   // Handles payment
OrderInventoryUpdateAction  // Updates inventory
```

### 📏 **Composition Guidelines**

- **Actions can depend on Services + Sub-Actions** (max 5-8 total dependencies)
  Actions orchestrate business operations, so they need both reusable logic (Services) and discrete steps (Sub-Actions).
  The 5-8 limit prevents actions from becoming too complex and ensures they remain focused on their primary
  responsibility.
- **Services should only depend on other Services** (max 4-5 dependencies)
  Services provide pure business logic and shouldn't trigger other business operations (Actions). Limiting to 4-5
  dependencies keeps services focused and prevents them from becoming mini-orchestrators themselves.
- **Avoid deep action chains** (A → B → C → D)
- **One main transaction per action** - let orchestrator manage transaction scope

## Typical Laravel Controller Integration

### 📝 **Overview**

Laravel controllers handle HTTP requests, delegating logic to **Actions** and formatting responses with **API Resources
**. They use **Form Requests**, **DTOs**, and **Repositories** for clean integration.

### ✅ **Example: OrderController**

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Customer\Order;

use App\Actions\Customer\Order\OrderCreateAction;
use App\DataTransferObjects\Customer\Order\OrderCreateDTO;
use App\Exceptions\BusinessExceptions\Cart\CartCheckException;
use App\Exceptions\BusinessExceptions\User\UserCheckException;
use App\Exceptions\Contracts\BusinessException;
use App\Http\Controllers\Controller;
use App\Http\Requests\Customer\Order\OrderStoreRequest;
use App\Http\Resources\Customer\Order\OrderResource;
use App\Repositories\User\CurrentAuthUserRepository;

final class OrderController extends Controller
{
    /** @throws BusinessException|UserCheckException|CartCheckException */
    public function store(
        OrderStoreRequest $request,
        CurrentAuthUserRepository $currentAuthUserRepository,
        OrderCreateAction $orderCreateAction
    ): OrderResource
    {
        $user = $currentAuthUserRepository->getApiCustomerUser();
        $validatedInput = $request->safe();
        $orderCreateDTO = new OrderCreateDTO(
            ip: $request->ip(),
            userAgent: $request->userAgent(),
            items: $validatedInput->input('items'),
            directionTypeId: $validatedInput->input('direction_type_id')
        );
        $order = $orderCreateAction->handle($orderCreateDTO, $user->id);
        
        return OrderResource::make($order);
    }
}
```

### 🔧 **Key Components**

- **Form Requests**: Validates requests (e.g., `OrderStoreRequest`).
- **DTOs**: Structures data (e.g., `OrderCreateDTO`).
- **API Resources**: Formats responses (e.g., `OrderResource`).
- **Repositories**: Abstracts data access (e.g., `CurrentAuthUserRepository`).
- **Actions**: Handles logic (e.g., `OrderCreateAction`).

### 🚨 **Exception Handling**

Actions throw custom exceptions (e.g., `UserCheckException`) for errors, formatted as JSON responses.

**Example: Custom Exception**

```php
<?php

declare(strict_types=1);

namespace App\Exceptions\BusinessExceptions\User;

use App\Exceptions\Contracts\BusinessNotReportableException;
use App\Exceptions\Contracts\BusinessRenderException;

final class UserCheckException extends BusinessRenderException implements BusinessNotReportableException
{
    // Thrown with: throw new UserCheckException(messageKey: 'user.error.empty_price_type');
}
```

**Base Exception**

```php
<?php

declare(strict_types=1);

namespace App\Exceptions\Contracts;

use App\Http\Responses\ApiResponse;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class BusinessRenderException extends BusinessException implements HttpExceptionInterface
{
    private int $statusCode;

    public function __construct(
        ?string $userMessage = null,
        ?string $messageKey = null,
        int $statusCode = Response::HTTP_UNPROCESSABLE_ENTITY
    ) {
        $this->statusCode = $statusCode;
        $userMessage = $messageKey ? __($messageKey) : $userMessage;
        
        parent::__construct($userMessage);
    }

    public function render(Request $request): JsonResponse
    {
        return ApiResponse::errorResponse(message: $this->getUserMessage(), status: $this->getStatusCode());
    }
}
```

### 📏 **Controller Guidelines**

- Keep controllers thin, delegating to Actions.
- Use dependency injection.
- Leverage Form Requests, DTOs, API Resources.
- Catch exceptions for HTTP responses.

**Final Note:** These guidelines are flexible to accommodate your needs. Use them to maintain consistency and clarity,
but adapt as necessary based on team conventions and evolving requirements. For any specific naming or organization
decisions, discuss with your team to ensure alignment.

