<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/cabernet-code/blog/assets/357114/75a6bad1-5dcc-404d-a9b7-a47ce04faa77" >
  <source media="(prefers-color-scheme: light)" srcset="https://github.com/cabernet-code/blog/assets/357114/e8fbe5b1-61d8-4ef6-9268-8d587da82818">
  <p align="center">
    <img alt="Shows an illustrated sun in light mode and a moon with stars in dark mode." width=20 src="https://github.com/cabernet-code/blog/assets/357114/e8fbe5b1-61d8-4ef6-9268-8d587da82818">
  </p>
</picture>

# From Concept to Coding: Transforming Business Rules into Reliable Code

**YAN JUSTINO**  
[MSc. Software Engineering](https://repositorio.ufrn.br/handle/123456789/26370) · [PhD. Student](https://www.cesar.school/doutorado-profissional-em-engenharia-de-software/)  
[AWS](https://www.youracclaim.com/users/yan-justino/badges) · [OCA](https://www.youracclaim.com/users/yan-justino/badges) · [ORCID](https://orcid.org/0000-0001-7248-716X) · [Tech Lead at ITAÚ Unibanco]()

![Static Badge](https://img.shields.io/badge/post-design-50A6C5?style=for-the-badge)

> The limits of my language mean the limits of my world  
> **-Ludwig Wittgenstein**.

## CONTEXT

Software engineering can put a lot of effort into achieving a clear definition of a system's boundaries. Part of this work involves designing modules, components, and their connectors, as well as the allocation structures that meet a set of requirements which, under certain quality expectations, respond to business demands.

In collaboration with business experts, the role of development is to "fill in" the gaps in these software structures with (rich) domain behaviors. For this purpose, development teams adopt methods like _Business Process Management_ (BPM), _Domain-Driven Design_ (DDD), _Business Capability Analysis_, etc., to represent domain contexts and express business rules.

Business rules are the guidelines, criteria, or conditions defined within an organization to govern its processes, operations, and transactions. They cover a wide range of areas and can specify: eligibility criteria, policies, processes, standards, etc. Business rules are often implemented in information systems. This type of automation allows efficient monitoring of these rules in the operational daily life of organizations.

However, automating business rules in information systems also demands a level of quality that ensures reliability in operations. In this sense, it's important for developers to implement business behaviors appropriately to the point of being free of side effects and easy to maintain. Otherwise, the system will present failures that may pose risks to the organization and especially to people.

Comparing the quality requirements on information systems with software development practices, we must ask ourselves:

1. _How do developers implement business rules in information systems?_
2. _What practices can be adopted to encode rules in a way that ensures their correctness?_
3. _What strategies does the development team adopt to manage the lifecycle of rules in the system?_

These questions will guide this post, which aims to present concepts and software design practices that can generate valuable insights to assist developers in modeling, building, and maintaining business rules in information systems.

## BACKGROUND

### Business Rules in Domain-Driven Design

Aiming to bring business language closer to software design, Eric Evans [^1] proposes the creation of rich domain models that represent a "distilled knowledge" of the business. In this sense, Evans establishes the models as "the backbone of a language used by all team members."

[^1]: https://en.wikipedia.org/wiki/Domain-driven_design.

However, the understanding of what a rich model is can be very abstract and full of misconceptions. One of them is thinking that a rich model is simply a contrast to anemic models (classes devoid of behavior). One consequence of this narrow view is the construction of entities excessively rich in behaviors.

> [!CAUTION]
>> _People new to DDD tend to model many behaviors. They innocently believe in the fallacy that DDD is about modeling the real world. Later, they attempt to model too many real-world behaviors of an entity._ **-Scott Millett and Nick Tune** [^2]

[^2]: Book: Patterns, Principles, and Practices of Domain-Driven Design.

Evans also states that "the rules of a business usually do not fit within the responsibility of an Entity or Value Object" because they "can overload the basic meaning of the domain object". However, he reiterates that "taking rules out of the domain layer is even worse." To resolve this issue, Evans borrows the concept of predicates and creates specialized objects that evaluate and provide a boolean result as an answer.

> [!NOTE]
>> _The concepts of predicate and boolean results go back to a philosophical-mathematical movement in the early 20th century called the Linguistic Turn. During this period, philosophers adopted language as the central focus of their investigations. Thinkers like Frege, Russell, and Wittgenstein, using logic and mathematics, sought a formal approach to explain and abstract linguistic phenomena. These studies have a great influence on how we conceive high-level programming languages today._

These objects are called **Specifications** and represent predicates that determine whether or not an object satisfies some criteria. **Code 1** illustrates a specification of the

logic if a customer is of legal age. By extracting this logic into its own class, it broadens expressiveness, since the scope of the class points to a single concept; it also enhances testability, as this structure has few dependencies to be validated.

```csharp
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T entity);
}

public class LegalAgeSpecification: ISpecification<Customer>
{
    public bool IsSatisfiedBy(Customer entity) => 
       entity.CalculateAge() >= 18;
}
```
<sup>**Code 1.** Specification model in C#<sup>

The Specification technique is a powerful concept in DDD. However, it's somewhat frustrating to see how it's buried in the high-level conceptual design. In the building blocks view of DDD (see Figure 1), it's not highlighted like other domain model concepts. In his book, Evans chose to associate the technique with the concept of Value Objects. Other authors like Vaughn Vernon, Scott Millett, and Nick Tune followed in the same line.

![image](https://github.com/cabernet-code/blog/assets/357114/b1b18834-d7c3-47df-b9d7-224d98df40f8)
<sup>**Figure 1.** DDD building blocks</sup>

### Business Rules via Design By Contract

Specifications are not an exclusive concept of Evans/Fowler. Two decades before the publication of DDD, **Bertrand Meyer** [^3], a master in engineering from _École Polytechnique_ in Paris, described in the chapter **Design By Contract: Building Reliable Software** in his book **Object-Oriented Software Construction**, how to enhance software reliability (Reliability) using an approach called _Design by Contract_.

In summary, _Design by Contract_ applies a set of relationship rules between a class and its clients (class consumers), like a formal contract composed of pre-conditions and post-conditions, which express for each part their rights and obligations. These relationships are done through **Specifications** aiming to meet the aspect of correctness (_correctness_) of reliability. For Meyer, "A Specification is given by **Assertions**, which are expressions involving and declaring predicates about an entity, satisfying some stage of software execution". The assertions can be represented by the following **Correctness Formula**:

```math
{P} A {Q}
```

[^3]: https://en.wikipedia.org/wiki/Bertrand_Meyer

This formula expresses that any execution of **(${A}$)**, started in a state maintained by **(${P}$)**, will be concluded in a state maintained by **(${Q}$)**. Consider the following example,

```math
{x >= 9} x := x + 5 {x >= 14}
```

In this example, if **(${P}$)** represents a number **(${x}$)** greater than or equal to 9; and **(${A}$)** is a function whose result is a sum of **(${x}$)** plus the number 5; then **(${Q}$)** will represent a number **(${x}$)** equal or superior to 14. In this perspective, **(${P}$)** represents the **pre-conditions** expressing the conditions under which a routine must function appropriately; while **(${Q}$)** represents the **post-conditions**, which express the resulting state of the executed routine.

Regarding pre-conditions, Meyer states that they are an **obligation** for the client and a **benefit** for the provider. Post-conditions, on the other hand, are understood as a benefit for the client and an obligation for the provider.

## EVALUATION

This section discusses the implementation of business rules in information systems, highlighting challenges associated with the imperative paradigm and the importance of practices like using design patterns and assertions to ensure correctness and ease of maintenance.

### How do developers implement business rules in information systems?

The answer to this question is closely linked to the languages, techniques, and tools adopted by programmers. From a general perspective, it's understood that most of the main languages in the industry like C, Java, Python, C# have a strong trait in the Imperative paradigm.

> [!NOTE]
>> _Imperative languages are built around the concept of state mutation and flow control. This means that programs written in imperative languages usually modify their states through variable assignments and use control structures like loops (for, while) and conditionals (if-else) to control the execution order of instructions._

Despite some of these languages offering support for other paradigms, the imperative trait still has much influence on how developers write their programs. **Code 2** illustrates an example of this type of programming.

```csharp
// Vineyard is a premium wine retailer that caters to both online 
// shoppers and visitors to its physical stores, offering an exquisite 
// selection of wines from around the globe. 
namespace Vineyard.Orders;

public class OrderRegistrationUse

Case
{
    private const string Available = "AVAILABLE";
    private const string InTransit = "IN_TRANSIT";

    public Order RegisterOrder(OrderRegistration? command)
    {
        ArgumentNullException.ThrowIfNull(command);

        var customer = CustomerRepository.GetById(command.CustomerId);
        if (customer is null) throw new ArgumentNullException(nameof(customer));
        var isOfLegalAge = DateTime.Now.Year - customer.Birthday.Year >= 18;

        var product = ProductRepository.GetBySku(command.Sku);
        var isValidStatus = product?.Status is Available or InTransit;
        var isNotExpired = product is not null && product.DueDate < DateTime.Now;

        var discount = 0m;

        if (customer.DateOfFirstPurchase.HasValue)
        {
            if (customer.DateOfFirstPurchase < DateTime.Now.AddYears(-1))
            {
                // after 1 year, loyal customers get 10%
                discount = Math.Max(discount, .10m);
                if (customer.DateOfFirstPurchase < DateTime.Now.AddYears(-5))
                {
                    // after 5 years, 12%
                    discount = Math.Max(discount, .12m);
                }
            }
        }
        else
        {
            // first time purchase discount of 15%
            discount = Math.Max(discount, .15m);
        }

        if (customer.IsTheBirthday()) discount += 0.05m;

        var order = new Order(customer, product)
        {
            IsEligible = isOfLegalAge && isValidStatus && isNotExpired,
            Discount = discount
        };

        //TODO : Register in Database

        return order;
    }
}
```
<sup>**Code 2.** Imperative code example<sup>

The C# code presented earlier illustrates an application for _Vineyard_, a fictional store specialized in selling imported wines. In this code, there's a class called `OrderRegistrationUseCase` that implements the `RegisterOrder(OrderRegistration command)` method, whose responsibility is to register the `Order` entity. For this, the service uses the following rules:

- The `Order` must contain a `Customer`
- The `Customer` must be at least 18 years old
- The `Order` must contain a `Product`
- The `Product` must be in the `AVAILABLE` or `IN_TRANSIT` status
- The `Product` cannot be expired
- When the date of the first purchase of the `Customer` is more than 1 year, they receive a 10% discount.
- When the date of the first purchase of the `Customer` is more than 5 years, they receive a 12% discount.
- When it's the first purchase of the `Customer`, they receive a 15% discount.
- If it's the `Customer`'s birthday, they receive an additional 5% discount.

Although illustrative, this code is based on a real implementation, where the business rules are written imperatively in the code. This is not uncommon and can be identified in various systems, from different domains. In this type of implementation, the origins and motivations of the business rules that ended up in the source code are hidden, possibly lost in some static artifact or in the mind of some expert.

> [!IMPORTANT]
>> _Personally, I see source code beyond a set of machine instructions. In my view, it should be understood as a communication artifact between developers at the coding stage. I like to think that among all the artifacts of the software development lifecycle, source code is one of the most dynamic. Perhaps these ideas are influenced by my appreciation for a movement in which many software engineering artifacts are "as code"._

### What practices can be adopted to encode rules in a way that ensures their correctness?

In this section, we'll illustrate the use of some practices to encode business rules in software. For this, we'll make use of some rules defined by Meyer for the use of Specifications. They form a set of guidelines that direct the creation of Assertions and start from the principle that "under no circumstances should the body of a routine test its preconditions". They are:

- Assertions are not mechanisms for validating inputs;
- Assertions are not control structures;
- Use Assertions to write correct software;
- Use Assertions for documentation;

#### Assertions Are Not Mechanisms for Validating Inputs

Assertions should only validate the scope of the business rule they propose. In this sense, create specialized modules to handle input validations. One way to do this could be using patterns like decorator, to isolate the code that will validate the input objects from the routine. **Code 3** illustrates an example of implementation.

```csharp
public class OrderRegistrationValidation : IOrderRegistrationUseCase
{
    private IOrderRegistrationUseCase service;

    public OrderRegistrationValidation(IOrderRegistrationUseCase service)
    {
        this.service = service;
    }

    public Order RegisterOrder(OrderRegistration? input)
    {
        ArgumentNullException.ThrowIfNull(input);
        
        var customer =

 CustomerRepository.GetById(input.CustomerId);
        ArgumentNullException.ThrowIfNull(customer);
        
        var product = ProductRepository.GetBySku(input.Sku);
        ArgumentNullException.ThrowIfNull(product);

        var command = input with { Customer = customer, Product = product };

        return service.RegisterOrder(command);
    }
}
```
<sup>**Code 3.** Example of using the decorator pattern for input validation<sup>

By removing the input validations from the `OrderRegistrationUseCase` class, its complexity is reduced, isolating the business scope. **Code 4** illustrates the `OrderRegistrationUseCase` method without input validations.

```csharp
public class OrderRegistrationUseCase : IOrderRegistrationUseCase
{
    private const string Available = "AVAILABLE";
    private const string InTransit = "IN_TRANSIT";

    public Order RegisterOrder(OrderRegistration? command)
    {
        var customer = command?.Customer!;
        var product = command?.Product!;

        var isOfLegalAge = DateTime.Now.Year - customer.Birthday.Year >= 18;
        var isValidStatus = product.Status is Available or InTransit;
        var isNotExpired = product.DueDate < DateTime.Now;

        var discount = 0m;

        if (customer.DateOfFirstPurchase.HasValue)
        {
            if (customer.DateOfFirstPurchase < DateTime.Now.AddYears(-1))
            {
                // after 1 year, loyal customers get 10%
                discount = Math.Max(discount, .10m);
                if (customer.DateOfFirstPurchase < DateTime.Now.AddYears(-5))
                {
                    // after 5 years, 12%
                    discount = Math.Max(discount, .12m);
                }
            }
        }
        else
        {
            // first time purchase discount of 15%
            discount = Math.Max(discount, .15m);
        }

        if (customer.IsTheBirthday()) discount += .05m;

        var order = new Order(customer, product)
        {
            IsEligible = isOfLegalAge && isValidStatus && isNotExpired,
            Discount = discount
        };

        //TODO : Register in Database

        return order;
    }
}
```
<sup>**Code 4.** Code outcome without input validations<sup>

#### Assertions Are Not Control Structures

Following the principle that "the body of a routine should not test its preconditions", we need to make our code more declarative. For this, we will remove the validation flows so they do not form control structures. One of the possible ways to perform this is to extract the business rules from the `RegisterOrder` method into new methods in the `OrderRegistrationUseCase` class. **Code 5** illustrates the outcome of this extraction.

```csharp
public class OrderRegistrationUseCase : IOrderRegistrationUseCase
{
    private const string Available = "AVAILABLE";
    private const string InTransit = "IN_TRANSIT";

    public Order RegisterOrder(OrderRegistration? command)
    {
        var customer = command?.Customer!;
        var product = command?.Product!;

        var order = new Order(customer, product)
        {
            IsEligible = IsOrderEligible(product, customer),
            Discount = CalculateDiscount(customer)
        };

        //TODO : Register in Database

        return order;
    }

    private bool IsOrderEligible(Product product, Customer customer)
    {
        // Implementation of eligibility logic
    }
    
    private decimal CalculateDiscount(Customer customer)
    {
        // Implementation of discount calculation logic
    }
}
```
<sup>**Code 5.** Code outcome without rules in the method body<sup>

After these small changes, **Code 5** is significantly different from **Code 2**. The `RegisterOrder` method of the `OrderRegistrationUseCase` class now presents a more declarative scope. However, how can we ensure the correctness of our rules?

#### Use Assertions to Write Correct Software

With the extractions performed in the `OrderRegistrationUseCase` class, it was possible to remove the control structures that were in the method body, promoting better readability to the code. The next step is to isolate the Assertions from the `OrderRegistrationUseCase` class, making them more atomic, reusable, and testable. For this, the **Business Rule Pattern** [^4], proposed by **Steve Smith** [^5], will be adopted. These are some important points to note about the pattern:

[^4]: Business Rule Pattern - https://www.pluralsight.com/courses/c-sharp-design-patterns-rules-pattern
[^5]: Steve Smith - https://ardalis.com/blog

- Each rule should follow the **Single Responsibility Principle**.
- To ensure that the rule follows the Single Responsibility Principle, keep in mind the **KISS Principle**!
- Rules can be ordered, filtered, or aggregated as needed.
- The rules engine determines how to process the rules.
- The logic for determining the priorities

 of the rules resides in the rules engine.

The first step to implement this pattern will be the removal of the validation methods created in **Code 5** to their own structure. **Code 6** demonstrates the transformation of these methods into Assertions of the `Customer` entity in a class called `CustomerRules`.

```csharp
public class CustomerRules
{
    public class IsOfLegalAge : ISpecification<Customer>
    {
        public bool IsSatisfiedBy(Customer customer) => customer.CalculateAge() >= 18;
    }
    
    public class IsFirstPurchase : ISpecification<Customer>
    {
        public bool IsSatisfiedBy(Customer customer) => customer.DateOfFirstPurchase is null;
    }
    
    public class IsLoyalCustomer : ISpecification<Customer>
    {
        private int years;

        public IsLoyalCustomer(int years)
        {
            this.years = years;
        }

        public bool IsSatisfiedBy(Customer customer)
        {
            return customer.DateOfFirstPurchase.HasValue && 
                   DateTime.Now.Year - customer.DateOfFirstPurchase.Value.Year >= years;
        }
    }
}
```
<sup>**Code 6.** Creation of Assertions for the Customer entity<sup>

Now, the `CustomerRules` class has greater autonomy in relation to the `OrderRegistrationUseCase`. Each of its sub-classes implements the `ISpecification<T>` interface and has behavior free of side effects and is testable.

Next, we need to create the class that will compose the Assertions to deal with the business rule. In the _Business Rule Pattern_, this type of class is called a **Rules Engine**. **Code 7** demonstrates the implementation of the `IsOrderEligible` Rules Engine, which determines whether an order is eligible or not.

```csharp
public class IsOrderEligible
{
    private static ISpecification<Customer> LegalAgeSpecification => new CustomerRules.IsOfLegalAge();
    private static List<ISpecification<Product>> ProductSpecifications => new List<ISpecification<Product>>
    {
        new ProductRules.IsAvailableOrInTransit(),
        new ProductRules.IsNotExpired()
    };

    public static bool Execute(Customer customer, Product product)
    {
        if (!LegalAgeSpecification.IsSatisfiedBy(customer)) return false;

        foreach (var spec in ProductSpecifications)
        {
            if (!spec.IsSatisfiedBy(product)) return false;
        }

        return true;
    }
}
```
<sup>**Code 7.** Creation of the Rules Engine<sup>

With all the rules that were coupled to the use case extracted, **Code 2** has been refactored to a more declarative version focused on specifications rather than instructions. **Code 8** presents the final version of the `OrderRegistrationUseCase` class after refactoring.

```csharp
public class OrderRegistrationUseCase : IOrderRegistrationUseCase
{
    public Order RegisterOrder(OrderRegistrationCommand? command)
    {
        var customer = command?.Customer!;
        var product = command?.Product!;

        var order = new Order(customer, product)
        {
            IsEligible = IsOrderEligible.Execute(customer, product),
            Discount = CalculateDiscount.Execute(customer)
        };

        //TODO: Register in Database

        return order;
    }
}
```
<sup>**Code 8.** Final outcome of the OrderRegistrationUseCase class<sup>

### What strategies does the development team adopt to manage the lifecycle of rules in the system?

This section is reserved to present the last of the rules listed in the previous section. With it, we will discuss how to associate the benefits of the practices adopted previously with the practice of documentation to achieve better maintenance, understanding, and collaboration.

#### Use Assertions for Documentation

Taking advantage of the isolation of the Assertions that were implemented in the previous section, it's possible to use all the features of the C# compiler like comments and attributes to document the business rules. This is a way to maintain traceability and manage the business rules that were included in the project over time clearer for the development team. **Code 9** and **Code 10** present an example of documentation of the rules.

```csharp
public class CustomerRules
{
    /// <summary>
    /// In Brazil, the consumption of alcoholic beverages by minors under 18 years old
    /// is prohibited. Federal Law 13.106/16 amended the Child and Adolescent Statute,
    /// making it a crime to sell, supply, serve, administer, or deliver alcoholic beverages
    /// to a child or adolescent.
    /// <see href="https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2015/lei/l13106.htm"/>
    /// </summary>
    public class IsOfLegalAge : ISpecification<Customer>
    {
        public bool IsSatisfiedBy(Customer customer) => customer.CalculateAge() >= 18;
    }
    
   

 /// <summary>
    /// First purchase rule defined by the Marketing Department Manager
    /// and embraced by the sales manager during the Black Friday
    /// of 2022.
    /// <see href="https://internal-system/features/98098999"/>
    /// </summary>
    [Obsolete("Rule valid until July 2023")]
    public class IsFirstPurchase : ISpecification<Customer>
    {
        public bool IsSatisfiedBy(Customer customer) => customer.DateOfFirstPurchase is null;
    }
    
    /// <summary>
    /// Implementation of the loyalty program after the results of
    /// Black Friday 2022
    /// <see cref="IsFirstPurchase"/>
    /// <see href="https://internal-system/features/99090000"/>
    /// </summary>
    public class IsLoyalCustomer : ISpecification<Customer>
    {
        private int years;

        public IsLoyalCustomer(int years)
        {
            this.years = years;
        }

        public bool IsSatisfiedBy(Customer customer) => customer.DateOfFirstPurchase < DateTime.Now.AddYears(years * -1);
    }
}
```
<sup>**Code 9.** Using comments and attributes in Assertions<sup>

```csharp
/// <summary>
/// Discount rule during order registration
/// Discounts are granted in the following cases:
/// 1 - First purchase = 15%
/// 2 - Loyalty customer for more than 1 year = 10%
/// 3 - Loyalty customer for more than 5 years = 12%
/// 4 - Birthday customers receive an additional 5% discount
/// <see cref="CustomerRules.IsFirstPurchase"/>
/// <see cref="CustomerRules.IsLoyalCustomer"/>
/// <see href="https://internal-system/features/99090000"/>
/// </summary>
public class CalculateDiscount
{
    private static List<(ISpecification<Customer> Spec, decimal Discount)> CustomerSpecs => new List<(ISpecification<Customer>, decimal)>
    {
        (new CustomerRules.IsFirstPurchase(), .15m),
        (new CustomerRules.IsLoyalCustomer(1), .10m),
        (new CustomerRules.IsLoyalCustomer(5), .12m)
    };

    public static decimal Execute(Customer customer)
    {
        var discount = 0m;

        foreach (var (spec, discountValue) in CustomerSpecs)
        {
            if (spec.IsSatisfiedBy(customer))
                discount = Math.Max(discount, discountValue);
        }
        
        if (customer.IsTheBirthday()) discount += .05m;
        
        return discount;
    }
}
```
<sup>**Code 10.** Documentation of the Rules Engine<sup>

Some developers are not in favor of this type of documentation in code. However, looking at a context where a team constantly needs to change rules in the application and those rules do not have adequate or practical registration, keeping this information in the code will provide a bit of traceability for this type of change, which is rarely explicit in a code versioning system.

#### Keep Everything Organized

A last aspect that needs to be discussed is the organization of the code structure to accommodate the business rules. The solution presented here was organized as follows:
- The Assertion classes were placed alongside the Domain Models;
- The rules engines were placed in the use case scope within a folder called `BusinessRules`.
- Each engine was separated by category.

In our example, we implemented two types of engine patterns: **Eligibility Pattern** and **Discount Calculation Pattern**. Referring to the concept of _Business Rule Management System (BRMS)_ presented in the post by [Amit Telkar](https://blogs.perficient.com/2017/09/17/10-business-rule-patterns-in-the-digital-transformation-and-cognitive-era/), he introduces other patterns of rules engines like _Validation Pattern, Authority and Approval Pattern, Legacy Rules Extraction, Scoring & Selection Pattern, Unstructured Big Data Pattern, Events Pattern_, etc. **Figure 2** illustrates the final organization of the solution.

![image](https://github.com/cabernet-code/blog/assets/357114/44846f15-9754-453c-bc10-cebcd64907ab)

<sup>**Figure 2.** Solution structure<sup>

## Conclusion

The text discusses the complexity of defining and implementing business rules in information systems within the field of software engineering. It emphasizes the importance of collaboration between developers and business experts to create software structures that meet quality requirements and adequately respond to business demands. Methods like Business Process Management (BPM), Domain-Driven Design (DDD), and Business Capability Analysis are highlighted as essential for mapping and effectively expressing business rules.

The text delves into concepts of Domain-Driven Design (DDD) and Design by Contract, proposed by Eric Evans and Bertrand Meyer, respectively, to illustrate approaches in modeling rich domains and implementing business rules with reliability. Specifications are presented as a technique for defining clear

 and verifiable criteria, while assertions and formal contracts between classes and their consumers are discussed as methods to ensure software correctness and reliability.

Finally, the text addresses recommended practices for encoding, documenting, and maintaining business rules in information systems, highlighting the importance of validation isolation, the use of assertions for correctness, and proper documentation to facilitate maintenance and understanding. These practices are pointed out as fundamental for the development of robust, flexible systems aligned with business objectives, underscoring the ongoing need for the adaptation and evolution of business rules in response to market changes.

## Disclaimer

This text serves as a focused review on the concepts of "Design by Contract," with special attention to its implementation in the C# language. While we aim to cover the main points and offer insights on how these concepts apply to software development, it's important to recognize that the discussion presented here does not exhaust the depth and breadth of "Design by Contract."

## Dear readers,

I hope you found valuable insights in the discussion about code design and organization.

If you have already tried these strategies in your projects, how was it? What advantages and obstacles did you encounter? If you haven't tried them yet, would you consider implementing them after this reading?

Your feedback is crucial for enriching this discussion.

## References

You can access the source code of this example by clicking [here](https://github.com/yanjustino/Vineyard)

- https://softwarehut.com/blog/tech/design-patterns-rules-engine
- https://www.michael-whelan.net/rules-design-pattern/
- https://app.pluralsight.com/library/courses/c-sharp-design-patterns-rules-pattern
- https://deviq.com/design-patterns/rules-engine-pattern
- https://blogs.perficient.com/2017/09/17/10-business-rule-patterns-in-the-digital-transformation-and-cognitive-era/
- https://martinfowler.com/bliki/BusinessReadableDSL.html
