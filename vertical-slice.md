# vertical-slice-architecture
Exploring this pattern


### Vertical Slice Architecture: CodeMash - notes

Traditional n-tier: UI - BLL - DAL - DB (sprocs)

Then DDD: UI - Services - Domain - Repository

Also Hexagonal Architecture

Also Clean Architecture: Enterprise Business Rules (in center) - Application Business Rules - Interface Adapters - Frameworks and Drivers (on outer edge)

These are all variations on the same theme ^^

Person.cs - PersonController.cs - PersonService.cs - PersonRepository.cs

#### The vertical slices

Model and organize application architecture around the axes of change, rather than down through n layers.

All code related to a feature is in the same place.

Given an `InvoiceService` with `Approve(), Reject(), Flag()` in reality for a given request only one of these methods will ever be called - they can be broken up!

`Approve()` becomes an `ApproveInvoice {}` (method to class)

`Reject()` becomes an `RejectInvoice {}` (method to class)

^^ JB noticed a pattern here and abstracted to

Input -> Request Handler -> Output

Which with MediatR became

IRequest<T> -> IRequestHandler<T> -> T ( a single request with a single reponse)

`Task<TResponse> Handle(TRequest request) { ... }`

GETS = query. POST = Command.

Nested classes for queries help show that a query is only for it's associated page. Throwing out "re-use" cos it hardly happens and adds complexity.

```
public class Index{
    public class Query : IRequest<Result>{
        public string SortOrder {get;set;}
        public string SearchString {get;set;}
    }
}
```



