# vertical-slice-architecture

Exploring this pattern as _easy to add change delete code_

The goal is to minimize coupling between slices and maximize coupling within a slice. In this way, each of our vertical slices can decide for itself how to best fulfill the request:

Get Orders -> ORm to AutoMapper to DTO > Orders

Get Order Details -> Raw SQL to DTO -> Order Details

Approve Invoice -> Event Sourcing on Aggregate Root -> Approved

Cancel Order -> Stored Procedure because why not -> Cancelled

+ [https://www.youtube.com/watch?v=5kOzZz2vj2o]
+ [https://headspring.com/2019/11/05/why-vertical-slice-architecture-is-better/]
+ [https://www.ghyston.com/insights/architecting-for-maintainability-through-vertical-slices/]
+ [https://lurumad.github.io/cross-cutting-concerns-in-asp-net-core-with-meaditr]

(Also see the Fowler Refactoring book and 'Refactoring to Patterns' (use code smells to drive design))

## Vertical Slice Architecture: HISTORY

Traditional n-tier: UI - BLL - DAL - DB (sprocs)

Then DDD: UI - Services - Domain - Repository

Also Hexagonal Architecture

Also Clean Architecture: Enterprise Business Rules (in center) - Application Business Rules - Interface Adapters - Frameworks and Drivers (on outer edge)

These are all variations on the same theme ^^

Person.cs - PersonController.cs - PersonService.cs - PersonRepository.cs = organizing by layer. Gets big. end up with `GetByPersonIdForBirthdayReward(int id)` <- LOL looks familiar!

Had to jump around the solution to lots of different places to get anything done. Change can has unforseen impacts.

### The vertical slices

Model and organize application architecture around the _axes of change_, rather than down through n layers.

All code related to a feature is in the same place = no jumping. What changes together lives together. NOT let's put all `People` together and all `Invoices` together.

Given a (usually too big) `InvoiceService` with `Approve(), Reject(), Flag()` in reality for a given request only one of these methods will ever be called - they can be broken up!

`Approve()` becomes an `ApproveInvoice {}` (method to class)

`Reject()` becomes an `RejectInvoice {}` (method to class)

^^ JB noticed a pattern here and abstracted to

Input -> Request Handler -> Output

Which with MediatR became

`IRequest<T>` -> `IRequestHandler<T>` -> `T` ( a single request with a single reponse)

`Task<TResponse> Handle(TRequest request) { ... }`

### Queries

GETS = query. POST = Command.

Nested classes for queries help show that a query is only for it's associated page. Throwing out "re-use" cos it hardly happens and adds complexity.

```c#
public class Index{
    public class Query : IRequest<Result>{
        public string SortOrder {get;set;}
        public string SearchString {get;set;}
    }
}
// request action parameters become properties of the request object
```

Single Response object (DTO) scoped to original request (no unused fields). Can have inner types

```c#
public class Model {
    // some properties
    public class Enrollment {
        ...
    }
}
```

### Handlers

```c#
public Task<List<Model>> Handle(Query message, CancellationToken token) =>
    _context.Departments.ProjectTo<Model>(_configuration).ToListAsync(token); //automapper projections (works with EF)
```

Don't pre-optimise: start with dumb procedural code right in the handler (a transaction script). Then take a step back, red-green-refactor.

Large class and long method code smells ->

+ Extract Class
+ Extract Class
+ Extract Class
+ Replace Method with Method Object
+ **Compose Method** (used a lot in refactoring Handlers)
+ **Extract Method**
+ **Move Method**

If the handler is stil too big - Push behaviour down from procedural handler into domain model. `MyDomainObject.Handle(CreateEdit.Command message)` (which is called by Handler).

Duplicated logic does occur. Extension methods can help. Introduces coupling.

Use good bits of DDD: Entities etc..

### Commands

Task based UIs 'Transfer', 'Correct' rather than 'edit'. Task maps to Handler. Works well.

Simple commands can be coded right into the handler.

For more complex commands can try in handler then refactor out.

### Validation

Request validation (form fields etc) can be managed in the handler. (Checkout FluentValidation). Also DataAnnotations on request object

Deeper validation (Domain level) eg: I can only approve invoices that have not been rejected.

This is done by the domain object Approve()ing itself.

### other stuff

that does not fit into single req/response model.

Razor pages work well with this.

If using JS frontends the pattern is ofc lots of Web APIs:

/order/approve -> Handler -> Order Json

/order/reject  -> Handler -> Order Json

The point JB makes here is that in these cases ^^ the handlers might return the same object, because the spa client is working on that object and can handle different properties being added or changed.

### Cross-Cutting Concerns

logging, authentication etc. Solved here using Pipeline Behaviour (see decorator pattern). Stackable/Chainable. Like ActionAttributes or ActionFilters.

Also good for Transactions, Concurrency and Retries. @57 min in video?v=5kOzZz2vj2o

register the pipeline behaviours in the order they will be executed

### Testing

Arrange (Request -> Act (Handler) -> Assert(Response) - so kind of end to end tests.

See `ExecuteScopeAsync()` for how to mimic production in tests, idea being every click in production has a test attached,

localTestDb used.

Unit Test Rich Domain Model. Integration tests for outer stuff (where the requests hit the handlers). JB does not unit test the Handlers (which have dependencies), only things that are isolated.
