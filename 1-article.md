Honestly, organizing business logic in Laravel sometimes feels like herding cats. Controllers, jobs, events-everyone's fighting for attention, and before you know it, you've got a mess that's one step away from spaghetti code.
In my projects, I use **Actions** and **Services** to bring structure and clarity. This is an **opinionated guideline**, based on how my team works - not a universal rulebook.
Yeah, Actions aren't exactly revolutionary - they're basically the [Command pattern](https://refactoring.guru/design-patterns/command) rebranded.
And no, I'm not trying to cosplay as Domain-Driven Design purists. It's simply a focused, practical approach to organizing business logic in Laravel.

{% details TL;DR - Quick Summary %}

- **Actions**: Handle full business operations (checkout, registration, import)
- **Services**: Reusable, stateless logic (calculation, validation, formatting, API calls, you get it)
- Naming: Keep it obvious: `OrderCreateAction`, `DeliveryScheduleService` with clear, descriptive names
- Controllers should be on a diet; send that logic to Actions and Services
- DTOs make your data flow squeaky clean and testable

{% enddetails %}

## Why Actions and Services?

Okay, so here's the deal - Laravel is super flexible. Like, to the point where you could toss your business logic just about anywhere and nobody would blink... until your project turns into a spaghetti monster.
Joel Clermont totally nails this in [_Where should you put your business logic?_](https://masteringlaravel.io/daily/2025-05-21-where-should-you-put-your-business-logic). Freedom’s fun, but man, it bites back.
Here's my two cents: split your logic into Actions (for the big-picture stuff) and Services (for the repeatable, "stateless" bits). This isn't just some random idea - Nuno Maduro's all about it in [_Laravel Actions: The Secret Sauce_](https://youtu.be/r1480BoFulQ).
Actions are context-free; doesn't matter if you're hitting HTTP, console, or some background job, the code stays consistent. Super handy and ensuring consistency as teams scale.

## Quick Decision: Action or Service?

### 🤔 **Ask Yourself:**

1. **Is this an entire business thing?** → Action (e.g., `OrderCreateAction`).
2. **Gonna reuse this in a bunch of places?** → Service (e.g., `CartItemValidator`).
3. **Does it mess with the outside world: notifications, logs, whatever?** → Action (e.g., `OrderIssueNotificationAction`).
4. **Is this pure data processing or calculation?** → Service (e.g., `DeliveryScheduleService`).
5. **Is this triggered by user/event?** → Action (e.g., `CartItemAddAction`).
6. **Is this domain-specific logic or infrastructure?** → Service (e.g., `TehnomirApi`).

{% details **Decision Matrix** %}

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


{% enddetails %}

{% details **Examples: actions and services** %}

| **Actions: Complete Business Operations**                                                     | **Services: Reusable Business Logic**                                        |
|-----------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|
| **OrderCreateAction** - Creates an order, handles payment, inventory, fires off notifications | **CartItemValidator** - Checks if cart items play by your business rules     |
| **CartCheckoutPossibilityAction** - Can you check out? Picks delivery dates                   | **DeliveryScheduleService** - Figures out delivery dates from supplier logic |
| **VehicleFindByVinAction** - Digs through APIs, normalizes, caches                            | **PartNameNormalizerAI** - Makes part names less weird with some AI magic    |
| **PriceImportUploadAction** - Handles file uploads, catches duplicates                        | **TehnomirApi** - Talks to external APIs and deals with their messiness      |
| **SearchBrandGroupingAction** - Runs search, filters, groups brands                           | **SearchResultService** - Cleans up and tweaks your search results           |
| **StatMetricDailyCollectAction** - Scoops up metrics, stores them, fires off reports          | **OrderConditionService** - Calculates discounts, shipping, payment terms    |

{% enddetails %}

## Actions: Focused Business Operations

Actions are single-purpose classes that handle complete business operations. Want to create an order? Log a search query? That's an Action's jam. Think Command pattern vibes: it wraps up a job with a clear "start here, end there" boundary.

### Action Guidelines

- **When to Use**: If your logic messes with the world (sends a notification, runs a transaction, or handles a user-triggered thing like `OrderCreateAction` or `CartItemAddAction`), put it in an Action.
- **Naming**: Don't overthink it-stick to `[Domain][Object][Verb]Action` (like `OrderItemCancelAction` or `SearchQueryTrackLogAction`). This ensures clarity, as Nabil Hassen emphasizes in [_Action Pattern in Laravel_](https://nabilhassen.com/action-pattern-in-laravel-concept-benefits-best-practices).
- **Structure**: Dump Actions in `app/Actions/[Domain]` (say, `app/Actions/Order`). The main entry point? `handle()`.
  
Like so:

```php
namespace App\Actions\Order;

use App\DataTransferObjects\OrderCreateDTO;
use App\Models\Order;
use App\Repositories\OrderRepository;
use App\Services\OrderConditionService;
use App\Actions\OrderIssueNotificationAction;
use Illuminate\Support\Facades\DB;

final class OrderCreateAction
{
    public function __construct(
        private readonly OrderRepository $orderRepository,
        private readonly OrderConditionService $orderConditionService,
        private readonly OrderIssueNotificationAction $notificationAction
    ) {}

    public function handle(OrderCreateDTO $dto, int $userId): Order
    {
        $this->orderConditionService->validate($dto);
        
        return DB::transaction(function () use ($dto, $userId) {
            $order = $this->orderRepository->create($dto, $userId);
            $this->notificationAction->handle($order);
            
            return $order;
        });
    }
}
```

**Best Practices**:
  - One job, one Action. Don't mash up ten things.
  - Constructor-inject your dependencies (services, repos, other Actions, etc).
  - Wrap stuff in `DB::transaction()` for atomicity, as Nuno Maduro suggests in [_Actions in Laravel Cloud_](https://youtube.com/shorts/wD1DAeRQ778).
  - Let business exceptions bubble up - let the global handler resolve the details.
  - Return affected resource or nothing.

Actions are killer for choreographing business logic: like having `OrderCreateAction` tap `OrderIssueNotificationAction` on the shoulder when it's done. Keeps your code tidy, modular, and super easy to test. No more spaghetti.

## Services: Reusable Business Logic

Alright, let's talk services. Basically, these are your go-to spots for logic you need all over the place-stuff like calculations, API calls...that sort of thing. They're not meant to run the whole show (leave that to your Actions), but if you find yourself copy-pasting the same logic more than once, that's your cue to extract a Service.

Here's the thing: if you start cramming every random utility into one mega-service, congrats, you've built a monster nobody wants to debug. Keep it lean, keep it mean, and you'll thank yourself later.

### Service Guidelines

- **When to Use**: For reusable logic (e.g., `DeliveryScheduleService`), validations (e.g., `CartItemValidator`), or integrations (e.g., `TehnomirApi`).
- **Naming**: Stick to `[Domain][Purpose]Service` (e.g., `SearchResultService`, `RequestReturnQuantityValidator`). If you need to toss in a `Validator` or `Api` at the end, go for it. Keep names straightforward and relevant.
- **Where It Lives**: Place in `app/Services/[Domain]` (e.g., `app/Services/Cart`). Methods should be laser-focused and stateless.
  
Here's a quick sample:

```php
namespace App\Services\Supplier;

use DateTime;

final class DeliveryScheduleService
{
    public function calculateSchedule(DateTime $orderDate, array $supplierRules): DateTime
    {
        // Just calculates the delivery schedule, nothing magical
        return $this->applySupplierRules($orderDate, $supplierRules);
    }

    private function applySupplierRules(DateTime $orderDate, array $rules): DateTime
    {
        // Don't overthink this, just a placeholder
        return $orderDate;
    }
}
```

**Best Practices**:
  - Stateless. Pure functions. No side effects, please.
  - If it's depending on more than 4-5 other Services, you're probably doing too much.
  - Design for reuse across Actions or contexts.
  - You should be able to write a unit test for it without wanting to throw your laptop out the window.

In short: Services are your go-to for calculation, validating cart items, normalizing data, figuring out delivery dates, formatting API payloads, and messing with third-party systems. They provide reusable building blocks for Actions and keep your codebase modular and testable.

## Organizing and Composing

### Directory Structure

Organize by domain for scalability, as shown in the guideline:

```
app/
├── Actions/
│   ├── Order/
│   │   ├── OrderCreateAction.php
│   │   └── OrderItem/
│   │       ├── OrderItemCancelAction.php
│   ├── Search/
│   │   ├── SearchQueryTrackLogAction.php
├── Services/
│   ├── Order/
│   │   ├── OrderConditionService.php
│   ├── Supplier/
│   │   ├── DeliveryScheduleService.php
```

Not rocket science, but trust me - future you will thank present you for this.

### Action Composition

Actions can depend on Services and sub-Actions (up to 5-8 dependencies) to orchestrate workflows. Avoid deep chains and limit to one transaction per Action. For example, `OrderCreateAction` uses `OrderConditionService` and `OrderIssueNotificationAction`.

### Slicing Up Actions

If one Action starts acting like it's the main character, split it up. Stuff trying to do too much? Break it down. But dont create action for every tiny task - use them for meaningful operations to avoid unnecessary complexity.
Instead of an overloaded `OrderProcessAction`, just make `OrderCreateAction`, `OrderPaymentProcessAction`, and `OrderInventoryUpdateAction`. Each one with a clear, single job. No drama, no confusion.

## Controller Integration

Controllers? Yeah, keep them skinny. None of that bloated logic soup - just pass the heavy lifting to Actions. Got Form Requests, DTOs, API Resources? Use them. 
Check out this snippet:

```php
namespace App\Http\Controllers\Customer\Order;

use App\Actions\Customer\Order\OrderCreateAction;
use App\DataTransferObjects\Customer\Order\OrderCreateDTO;
use App\Http\Controllers\Controller;
use App\Http\Requests\Customer\Order\OrderStoreRequest;
use App\Http\Resources\Customer\Order\OrderResource;
use App\Repositories\User\CurrentAuthUserRepository;

final class OrderController extends Controller
{
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

Bare minimum in the controller - just enough to grab what you need, delegate it to an Action, and wrap up the response. No one wants to scroll through a controller that looks like War and Peace.

Oh, and those exceptions like `UserCheckException`? Actions throw them, and some global handler swoops in to catch them and spit out a tidy JSON error. You don't need to clutter up your controller with try/catch gymnastics - just let the framework do its thing.

## Why This Approach?

This approach has brought consistency to my Laravel projects. Nuno Maduro dropped some gems in [_Actions in Laravel Cloud_](https://youtube.com/shorts/wD1DAeRQ778) about how keeping things uniform makes it way less painful to get new folks up to speed-or just to remember what the heck you wrote three months ago. Actions handle operations with side effects, while Services provide stateless reusable utilities. This isn't DDD-it's just a pragmatic setup, tailored for clarity and scalability.

## Best Practices

- **Actions**: Do one thing, have a `handle()` method, use transactions when you need them, keep it type-safe, exception bubbling.
- **Services**: Stateless, pure functions, focused interfaces, reusable across Actions.
- **Folders & Naming**: Group stuff by what it does (the "domain"), and name it so you instantly know what's up: `[Domain][Object][Verb]Action`, `[Domain][Purpose]Service`.
- **Composition**: Limit Action dependencies to 5-8, Service dependencies to 4-5, avoid deep Action chains.
- **Controllers**: Keep thin, use Form Requests, DTOs, and API Resources.

## When Not to Use Actions and Services
For simple CRUD operations with minimal logic (e.g., fetching a single record) don't do overengineering. Actions are way too much for that, just use a repo or simple controller code. If a Service only has one use case, consider keeping the logic in the Action to avoid unnecessary abstraction or only service.


## Wrapping Up

Actions and Services have streamlined my Laravel development, balancing simplicity and scalability. Actions encapsulate operations, while Services provide reusable logic, creating a maintainable codebase. It's opinionated, sure – but it works for my crew.
Take it, tweak it, or toss it, whatever fits your vibe. Check the full guidelines on [GitHub](https://github.com/tegos/laravel-action-and-service-guideline/blob/main/action-and-service-guidelines.md). Try this approach in your next Laravel project and share your experience in the comments. Have a different way to organize business logic? Let’s hear it!
