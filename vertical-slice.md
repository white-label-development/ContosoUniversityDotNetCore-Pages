# vertical-slice-architecture

Exploring this pattern

## Vertical Slice Architecture: CodeMash - notes

Traditional n-tier: UI - BLL - DAL - DB (sprocs)

Then DDD: UI - Services - Domain - Repository

Also Hexagonal Architecture

Also Clean Architecture: Enterprise Business Rules (in center) - Application Business Rules - Interface Adapters - Frameworks and Drivers (on outer edge)

These are all variations on the same theme ^^

Person.cs - PersonController.cs - PersonService.cs - PersonRepository.cs

### The vertical slices

Model and organize application architecture around the axes of change, rather than down through n layers.

All code related to a feature is in the same place.

Given an `InvoiceService` with `Approve(), Reject(), Flag()` in reality for a given request only one of these methods will ever be called - they can be broken up!

`Approve()` becomes an `ApproveInvoice {}` (method to class)

`Reject()` becomes an `RejectInvoice {}` (method to class)

^^ JB noticed a pattern here and abstracted to

Input -> Request Handler -> Output

Which with MediatR became

`IRequest<T>` -> `IRequestHandler<T>` -> `T` ( a single request with a single reponse)

`Task<TResponse> Handle(TRequest request) { ... }`

GETS = query. POST = Command.

Nested classes for queries help show that a query is only for it's associated page. Throwing out "re-use" cos it hardly happens and adds complexity.

```c#
public class Index{
    public class Query : IRequest<Result>{
        public string SortOrder {get;set;}
        public string SearchString {get;set;}
    }
}
```

Single Response object (DTO) scoped to original request. Can have inner types

```c#
public class Model {
    // some properties
    public class Enrollment {
        ...
    }
}
```

### Commands

Task based UIs 'Transfer', 'Corect' rather than 'edit'. Task maps to Handler.

Simple commands can be coded right into the handler.

For more complex commands can try in handler then refactor out.

Large class and long method code smells ->

+ Extract Class
+ Extract Class
+ Extract Class
+ Replace Method with Method Object
+ **Compose Method** (used a lot in refactoring Handlers)
+ **Extract Method**
+ **Move Method**

Use good bits of DDD: Entities etc..

Push behaviour down from procedural handler into domain model. `MyDomainObject.Handle(CreateEdit.Command message)` (which is called by Handler).

### Validation

Request validation (form fields etc) can be managed in the handler. (Checkout FluentValidation)

Deeper validation (Domain level) eg: I can only approve invoices that have not been rejected/

This is done by the domain object Approve()ing itself.

### other stuff

that does not fit into single req/response model.

Razor pages work well with this.

If using JS frontends the pattern is ofc lots of Web APIs:

/order/approve -> Handler -> Order Json

/order/reject  -> Handler -> Order Json

### Cross-Cutting Concerns

logging, authentication etc. Solved here using Pipeline Behaviour (see decorator pattern). Stackable/Chainable. Like ActionAttributes or ActionFilters.

Also good for Transactions, Concurrency and Retries

### Testing

Arrange (Request -> Act (Handler) -> Assert(Response) - so kind of end to end tests.

See `ExecuteScopeAsync()` for how to mimic production in tests, idea being every click in production has a test attached,

localTestDb used.

Unit Test Rich Domain Model. Integration tests for outer stuff. JB does not unit test the Handlers (which have dependencies), only things that are isolated.
