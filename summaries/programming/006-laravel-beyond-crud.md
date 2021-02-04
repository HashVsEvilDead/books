Laravel Beyond CRUD
==========
Building larger-than-average web applications
-------------

Brent Roose • 2020 • 163 pages

[Goodreads](https://www.goodreads.com/book/show/55285463-laravel-beyond-crud)

Author
------------
Brent Roose is a developer working for Spatie, a company contributing a lot in the laravel ecosystem, and the owener of stitcher.io.


Introduction
============

In this book, the author writes about the knowledge he gained over the years in designing larger-than-average web applications.
He takes a close look at the Laravel way and what did and didn't work for them. He talks about theory, patterns, and principles, though everything is in context of a real-life web application.
The goal is to learn a mindset so so that you can apply it to your problems, and come up with the solutions that work best for you.


Summary
=======

Domain oriented Laravel
------------------------------------

Split the business logic in domains but keep the code that integrates them with the framework in the application layer.
```
# One specific domain folder per business concept
src/Domain/Invoices/
    ├── Actions
    ├── QueryBuilders
    ├── Collections
    ├── DataTransferObjects
    ├── Events
    ├── Exceptions
    ├── Listeners
    ├── Models
    ├── Rules
    └── States

# The admin HTTP application
src/App/Admin/
    ├── Controllers
    ├── Middlewares
    ├── Requests
    ├── Resources
    └── ViewModels

# The REST API application
src/App/Api/
    ├── Controllers
    ├── Middlewares
    ├── Requests
    └── Resources

# The console application
src/App/Console/
    └── Commands
```

Working with data
------------------------------------

Make data a first-class citizen by using DTOs (Data Transfer Objects). DTOs are the entry point for data into the codebase. As soon as we're working with data from the outside, we want
to convert it to a DTO. We need to do this mapping somewhere, we could either use a Factory living in the application layer (since it has to know about specific requests and other kinds of user input) or a static method inside the DTO itself (we would mix application-specific code within the domain but, at the same time, we would keep all the logic related to that piece of data in the same class).
```
class CreateInvoiceData extends DataTransferObject
{
    /** @var \Domain\Invoices\DataTransferObjects\InvoiceLineData[] */
    public array $invoiceLines = [];

    public ?string $number = null;

    public ?int $totalPrice = null;

    public function withNumber(string $number): self
    {
        $clone = clone $this;

        $clone->number = $number;

        return $clone;
    }

    public function withTotalPrice(int $totalPrice): self
    {
        $clone = clone $this;

        $clone->totalPrice = $totalPrice;

        return $clone;
    }
}
```

Actions
------------------------------------

Just like we don't want to work with random arrays full of data, we also don't want the most critical part of our project, the business functionality, to be spread throughout random functions and classes.
We can encapsule the business functionalities in actions. They allow the programmer to think in ways that are closer to the real world, instead of the code.
Say you need to make changes to the way invoices are created. A typical Laravel application will probably have this invoice creation logic spread across a controller and a model, maybe a job which generates the PDF, and finally an event listener to send the invoice mail. That's a lot of places you need to know of. Once again our code is spread across the codebase, grouped by its technical properties, rather than its meaning. Actions reduce the cognitive load that's introduced by such a system. If you need to work on how invoices are created, you can simply go to the action class, and start from there.

```
class CreateInvoiceAction
{
    private CalculateInvoicePricesAction $calculateInvoicePricesAction;

    private DetermineInvoiceNumberAction $determineInvoiceNumberAction;

    private SaveInvoiceAction $saveInvoiceAction;

    private CreatePaymentAction $createPaymentAction;

    private CreatePdfAction $createPdfAction;

    private SendInvoiceMailAction $sendInvoiceMailAction;

    public function __construct(
        CalculateInvoicePricesAction $calculateInvoicePricesAction,
        DetermineInvoiceNumberAction $determineInvoiceNumberAction,
        SaveInvoiceAction $saveInvoiceAction,
        CreatePaymentAction $createPaymentAction,
        CreatePdfAction $createPdfAction,
        SendInvoiceMailAction $sendInvoiceMailAction
    ) {
        $this->calculateInvoicePricesAction = $calculateInvoicePricesAction;
        $this->determineInvoiceNumberAction = $determineInvoiceNumberAction;
        $this->saveInvoiceAction = $saveInvoiceAction;
        $this->createPaymentAction = $createPaymentAction;
        $this->createPdfAction = $createPdfAction;
        $this->sendInvoiceMailAction = $sendInvoiceMailAction;
    }

    public function __invoke(
        Client $client,
        CreateInvoiceData $data
    ): Invoice {
        $data = ($this->calculateInvoicePricesAction)($client, $data);

        $data = ($this->determineInvoiceNumberAction)($client, $data);

        $invoice = ($this->saveInvoiceAction)($client, $data);

        $payment = ($this->createPaymentAction)($invoice);

        $pdf = ($this->createPdfAction)($invoice);

        ($this->sendInvoiceMailAction)($invoice, $pdf, $payment);

        return $invoice;
    }
}
```
#### Alternatives to actions
There are two paradigms I need to mention at this point that wouldn't need a concept like actions.
The first one will be known to people who are familiar with DDD� commands and handlers. Actions are a simplified version of them. Where commands and handlers make a distinction between what needs to happen and how it needs to happen, actions combine these two responsibilities into one. It's true, however, that the command bus offers more flexibility than actions. On the other hand it also requires you to write more code. For the scope of our projects, splitting actions into commands and handlers was taking it a step too far. We would almost never need the added flexibility, yet it would take a lot longer to write the code.
The second alternative worth mentioning is event driven systems. If you ever worked in an event driven system, you might think that actions are too directly coupled to the places where they are actually used. Again the same argument applies: event driven systems offer more flexibility, yet for our projects it would have been overkill to use them. Furthermore, event driven systems add a layer of indirectness that makes the code more complex to reason about. While this indirectness does offer benefits, they wouldn't outweigh the cost of maintenance for us.

Models
------------------------------------

Together with DTOs and Actions, the true core of the project.
Laravel provides a lot of functionality in its Eloquent model classes, which means that they not only represent the data in a data store, they also allow you to build queries, load and save data, have a
built-in event system, and more. We shouldn't ditch all the functionalities but we should embrace them in such a way that large projects stay maintainable.
#### Models ≠ business logic
Try to avoid calculating things on the fly and, instead, store it. This way you improve performance and you can query the data. An example could be storing the invoice total price instead of having a method in the model for calculating it.
#### Scaling down models
Ideally, we only want to keep the data read from the database, simple accessors for stuff we can't calculate beforehand, casts, and relations.
You can move the query scopes to a specific class that extends the query builder and bind it to the Eloquent model with the `newEloquentBuilder` method.
```
class InvoiceQueryBuilder extends Builder
{
    public function wherePaid(): self
    {
        return $this->where('status', Invoice::STATUS_PAID);
    }
}
```
You can move the logic used to filter collections of models to a specific class that extends the collection class and bind it to the Eloquent model with the `newCollection` method.
```
class InvoiceLineCollection extends Collection
{
    public function creditLines(): self
    {
        return $this->filter(fn (InvoiceLine $invoiceLine) =>
            $invoiceLine->isCreditLine()
        );
    }
}
```
#### Empty bags of nothingness
Martin Fowler once wrote about how we need to avoid objects becoming nothing more than empty bags of data, something he calls an anti-pattern. You might be thinking we're doing the same with our model classes. Alan Kay (the one who came up with the term OOP) instead said that he regretted calling the paradigm "object oriented" and not "process oriented". Alan argues that he's actually a proponent of splitting process and data.

States
------------------------------------

Resource: https://www.youtube.com/watch?v=N12L5D78MAA
The state pattern is one of the best ways to add state-specific behaviour to models, while still keeping them clean, manageable and preventing them from handling business logic.
Instead of having multiple "if"s spreaded in the codebase for handling the different behavior related to the given state, we treat "a state" as a first-class citizen. Every state is represented by a separate class where we encapsulate the logic that depends on the given state. We laverage polymorphism to get rid of all the conditionals related to the states.
As we want to move business logic away from models, and only allowing them to provide data in a workable way from the database, the same thinking can be applied to states and transitions. States should be used to read or provide data. Transitions will make sure our model state is correctly transitioned from one to another leading to acceptable side effects. Splitting these two concerns in separate classes provide better testability and reduced cognitive load.

Enums
------------------------------------

Enums can be used to represent the different states in single class. We will not get rid of the multiple conditionals but we can use the class as a parameter allowing code completition, static analysises, etc. If we need a collection of related values and there are little places where the application flow is actually determined by those values, then we can use simple enums. But if we start attaching more and more value-related functionality to them, it's time to start looking at the state pattern instead.

Managing domains
------------------------------------

How do you start using domains, how to identify them, and how to manage them in the long run?
#### Teamwork
The architecture proposed in this book isn't about writing the smallest number of characters or about the elegance of code. It's about making large codebases easier to navigate, to
allow as little room as possible for confusion, and to keep the project healthy for a long time.
#### Identifying domains
The only way to identify domains is to understand the business problem. To understand the business problem, we have to discuss it thoroughly with our clients.

Testing domains
------------------------------------

#### Test factories
The actual goal of these factory classes is to help you write integration tests, without having to spend too much time setting up the system for it.
A test factory is nothing more than a simple class. There's no package to require, no interfaces to implement or abstract classes to extend. They can be used to create models, set up DTOs, and sometimes even request classes.
```
<?php

namespace Tests\Factories;

use Domain\Invoices\Models\Invoice;
use Support\TestFactories\Factory;

class InvoiceFactory extends Factory
{
    private string $number = '001';

    private ?ClientFactory $clientFactory = null;

    public static function new(): InvoiceFactory
    {
        // Example: InvoiceFactory::new()->create();
        return new self();
    }

    public function create(array $extra = []): Invoice
    {
        return Invoice::create(
            $extra + [
                'number' => $this->number,
                'total_price' => $this->totalPrice,
                // You can accept a factory for relation or use the default one
                'client_id' => ($this->clientFactory ?? ClientFactory::new())->create()->id
            ]
        );
    }

    public function withNumber(string $number): self
    {
        // The factory is immutable so that we can make several models with small differences easier.
        // Example, 2 invoices with the same client and different numbers:
        //  $clientFactory = ClientFactory::new()->create();
        //  $invoiceFactory = InvoiceFactory::new()->withClientFactory($clientFactory);
        //  $invoice1 = $invoiceFactory->withNumber(1)->create();
        //  $invoice2 = $invoiceFactory->withNumber(2)->create();
        $clone = clone $this;

        $clone->number = $number;

        return $clone;
    }

    public function withClientFactory(ClientFactory $clientFactory): self
    {
        $clone = clone $this;

        $clone->clientFactory = $clientFactory;

        return $clone;
    }
}

```

Entering the application layer
------------------------------------

#### Several applications
The first important thing to understand is that one project can have several applications. In fact, every Laravel project already has two by default: the HTTP and console apps.
In general, an application's goal is to get some kind of input, transform it for use in the domain layer, and represent the output to the user or store it somewhere.
#### Structuring HTTP applications
With larger-than-average applications, we should split the code in application modules that could have a 1-to-1 relation with a domain but it's not required.
```
Admin
└── Invoices
    ├── Controllers
    │   ├── IgnoreMissedInvoicesController.php
    │   ├── InvoiceStatusController.php
    │   ├── InvoicesController.php
    │   ├── MissedInvoicesController.php
    │   └── RefreshMissedInvoicesController.php
    ├── Filters
    │   ├── InvoiceMonthFilter.php
    │   ├── InvoiceOfferFilter.php
    │   ├── InvoiceStatusFilter.php
    │   └── InvoiceYearFilter.php
    ├── Middleware
    │   ├── EnsureValidHabitantInvoiceCollectionSettingsMiddleware.php
    │   ├── EnsureValidInvoiceDraftSettingsMiddleware.php
    │   └── EnsureValidOwnerInvoiceCollectionSettingsMiddleware.php
    ├── Queries
    │   ├── InvoiceCollectionIndexQuery.php
    │   └── InvoiceIndexQuery.php  ├── Requests
    │   └── InvoiceRequest.php
    ├── Resources
    │   ├── InvoiceCollectionDataResource.php
    │   ├── InvoiceCollectionResource.php
    │   ├── InvoiceDataResource.php
    │   ├── InvoiceDraftResource.php
    │   ├── InvoiceIndexResource.php
    │   ├── InvoiceLabelResource.php
    │   ├── InvoiceLineDataResource.php
    │   ├── InvoiceLineResource.php
    │   ├── InvoiceMainOverviewResource.php
    │   ├── InvoiceMainOverviewResource.php
    │   └── InvoiceResource.php
    ├── ViewModels
    │   ├── InvoiceCollectionHabitantContractPreviewViewModel.php
    │   ├── InvoiceCollectionOwnerContractPreviewViewModel.php
    │   ├── InvoiceCollectionPreviewViewModel.php
    │   ├── InvoiceDraftViewModel.php
    │   ├── InvoiceIndexViewModel.php
    │   ├── InvoiceLabelsViewModel.php
    │   └── InvoiceStatusViewModel.php
```
We should use the Support module for general purpose classes like a base request class, middleware that's used everywhere, etc.
#### View models
In essence, view models are simple classes that take data and transform it into something usable for the view.
```
class PostFormViewModel
{
    public function __construct(User $user, Post $post = null)
    {
        $this->user = $user;
        $this->post = $post;
    }
    public function post(): Post
    {
        return $this->post ?? new Post();
    }
    public function categories(): Collection
    {
        return Category::allowedForUser($this->user)->get();
    }
}
```
This is what the controller looks like:
```
class PostsController
{
    public function create()
    {
        $viewModel = new PostFormViewModel(current_user());
        return view('blog.form', compact('viewModel'));
    }
    public function edit(Post $post)
    {
        $viewModel = new PostFormViewModel(current_user(), $post);
        return view('blog.form', compact('viewModel'));
    }
}
```
And finally, it can be used in the view like so:
```
<input value="{{ $viewModel->post()->title }}" />
<input value="{{ $viewModel->post()->body }}" />
<select>
    @foreach($viewModel->categories() as $category)
        <option value="{{ $category->id }}">
            {{ $category->name }}
        </option>
    @endforeach
</select>
```
Laravel's view composers work similarly but they are registered somewhere in the global state so they should be avoided in large applications.

Jobs
------------------------------------

Jobs belong in the application layer because, similarly to controllers, they provide another way of exposing business functionality to the outside world.
They are classes that are responsible for: configuring whether they are queueable, how many of them can be run simultaneously, if their execution should be delayed, if they should be chained, etc.


Conclusion
==========

The main goal of this book it's not about learning patterns or concrete solutions, it's more about learning a mindset. Goal achieved.


Bibliography and Resources
==========================
- Gary Bernhardt's thoughts on type systems: https://www.destroyallsoftware.com/talks/ideology. Not everyone shares my preference for strong type systems these days. Gary Bernhardt has a fantastic talk about the differences between these two groups of people.
- Matthias Noback on the difference between value objects and DTOs: https://github.com/spatie/data-transfer-object/issues/17. Originally, I called DTOs "Value Objects", which have a slightly different meaning. Matthias was so friendly to jump into a GitHub thread to discuss the differences.
- Alan Kay's vision of OOP: https://www.youtube.com/watch?v=oKg1hTOQXoY. Alan Kay, who invented the term OOP, explains his original vision of "object-oriented". This talk highly influenced how I think about programming in an OO language.
- Refactoring to actions by Freek Van der Herten: https://freek.dev/1371-refactoring-to-actions. Over the years, we applied the principles described in this book in the projects of Spatie; Freek explores how to refactor old projects to a domain-oriented design.
- Martin Fowler on transaction scripts: https://martinfowler.com/eaaCatalog/transactionScript.html. Transaction scripts, described by Martin Fowler, are the basis of actions.
- Martin Fowler on anemic domain models: https://martinfowler.com/bliki/AnemicDomainModel.html. Martin Fowler also writes about anemic models, and how it's an anti-pattern. My vision leans closer to Alan Kay's: represent processes and data separately, and have them work together.
- Custom eloquent collections by Tim MacDonald: https://timacdonald.me/giving-collections-a-voice/. Tim MacDonald gave the inspiration about using custom eloquent collections; a pattern we still enjoy to this day.
- Dedicated query builders by Tim MacDonald: https://timacdonald.me/dedicated-eloquent-model-query-builders. Tim also wrote more in-depth about dedicated query builders.
- Christopher Okhravi explains state machines in depth: https://www.youtube.com/watch?v=N12L5D78MAA. If you're still unsure about the state pattern, Christopher Okhravi explains it thoroughly in this video.
- Symfony's workflow package to build complex state machines: https://symfony.com/doc/current/workflow/workflow-and-state-machine.html. Symfony's workflow package is an excellent alternative to the simplified state package I wrote.
- Sandi Metz's vision of truly OO code: https://www.youtube.com/watch?v=29MAL8pJImQ. Another eye-opener when it comes to object-oriented programming: Sandi Metz shows how to get rid of all if statements, and what it has to do with OO.
- Freek gets started with domain-oriented Laravel: https://freek.dev/1486-getting-started-with-domain-oriented-laravel. Another practical introduction to domain-oriented Laravel by Freek.
- Tighten's approach to model factories: https://tighten.co/blog/tidy-up-your-tests-with-class-based-model-factories. Both Spatie and Tighten came up with a similar approach to improved test factories at the same time. It's interesting to read about their approach as well.
