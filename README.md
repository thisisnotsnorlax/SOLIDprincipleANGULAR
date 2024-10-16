# Activity 25: SOLID PRINCIPLES in Angular

 ## Applying SOLID Principles in Angular
We are going to start with the most vague concept that is the least obvious in terms of how we should actually apply it to our Angular development: SOLID principles. The reason it is vague and not immediately obvious is because these principles have been established generically for Object Oriented Programming, not specifically for Angular applications. These principles pre-date Angular by ten years or so.

**Single-responsibility principle**

**Open-closed principle**

**Liskov substitution principle**

**Interface segregation principle**

**Dependency inversion principle**


## Single-responsibility principle

We will use this concept quite heavily, and we will save most of the discussion of this principle for our next lesson on smart and dumb components which embodies this concept well.

The general idea with this principle is that each class should have one responsibility. In an Angular context, whether that class is a component, directive, pipe, service, or anything else - it should have one responsibility.

Determining what a “responsibility” is, is part of the fun vagueness of these principles that can make them hard to apply. What is a “responsibility”? For example, imagine a service that manages the data/state for todos in an application. That service might:

Load todos

Create todos

Delete todos

Edit todos

Is this four different responsibilities? Should we really break this up into four different services each with just one responsibility? No, we absolutely should not do that. We are probably looking at how to define the “responsibility” here in too strict of a sense. Instead, we could say the responsibility of this service is to: manage the data related to todos. This would then fit the definition of having a single responsibility.

## Open-closed principle

This is another principle that applies quite well to Angular. This principle states that an entity should be open for extension, but closed for modification. In other words, when creating new features we should incorporate/extend/build on top of existing entities, not modify those existing entities.

Again, this is one of those things that needs to be taken in context and applied where appropriate. For example, let’s consider a simple ButtonComponent:

```typescript
@Component({
    selector: 'app-button',
    template: `
        <button>Hi</button>
    `
})
export class ButtonComponent {}
```

It’s a button that say’s “Hi”, fantastic. Now let’s say our requirements have changed - not only do we want a button that says “Hi”, we want to be able to configure its colour!

We could modify our component to this by accepting an input:

```typescript
@Component({
  selector: 'app-button',
  template: `
      <button [style.backgroundColor]="color">Hi</button>
  `,
  standalone: true
})
export class ButtonComponent {
  @Input() color? = '#cecece';
}
```
Now we can supply it with a colour:
```typescript
<app-button color="blue"></app-button>
```
But, technically, this is a violation of the open-closed principle. Alternatively, we could create a directive to apply the button colour instead:
```typescript
import { Directive, HostBinding, Input } from "@angular/core";

@Directive({
  selector: 'app-button[color]',
  standalone: true
})
export class ButtonColorDirective {
  @HostBinding('style.backgroundColor') @Input() color? = '#cecece';
}
```
NOTE: By using the selector app-button[color] this directive will only apply to app-button components that have the color attribute. If we had have just done color as the selector then it would apply to any component with the color attribute which we might not want. On top of that, we are also using the selector name as an input - this allows us to both attach the directive to a component and receive input on it with just one attribute.

Liskov Substitution Principle

We will incorporate this concept somewhat when we look at the dependency inversion principle in a moment, but this is not a principle that is generally particularly prevalent in Angular development.

The basic idea is that a sub-class should be able to replace a base class and the program should still function. With OOP, we can extend classes, e.g:
```typescript
export class ButtonComponent {}
```
```typescript
export class SpecialButtonComponent extends ButtonComponent {}
```

The idea with the Liskov Substitution principle is that we should be able to use our SpecialButtonComponent in place of the ButtonComponent and everything will still function. Our SpecialButtonComponent does everything the ButtonComponent does because it extends it, so it should be able to be used in place of the ButtonComponent.

You can make use of inheritance in Angular, even for components as I have given in the example above, but it is not all that common and can be awkward. An important thing to keep in mind with Angular is that if we have a component:

```typescript
@Component({
    selector: 'app-button',
    template: `
        <button>Hi</button>
    `
})
export class ButtonComponent {

    someClassMember: boolean;

    someMethod(){

    }
}
```

and we extend it:
```typescript
@Component({
    selector: 'app-special-button',
    template: ``
})
export class SpecialButtonComponent extends ButtonComponent {

    anotherMethod(){
    }
}
```
Our extended component only extends the class. With Angular, we use a @Component decorator to supply the template. Our extended class will not have the template of our base class, it will need to define its own template. What will be available to the extended class is the stuff in the class, e.g:

`someClassMember (from base class)`

`someMethod (from base class)`

`anotherMethod (from sub-class)`

Another awkward thing about extending classes in Angular is dealing with the dependency injection system. For example, let’s say our base button component injects the ChangeDetectorRef:
```typescript
@Component({
    selector: 'app-button',
    template: `
        <button>Hi</button>
    `
})
export class ButtonComponent {

    someClassMember: boolean;

    constructor(private cdr: ChangeDetectorRef){}

    someMethod(){

    }
}
```
This will mean our sub-class will need to pass the ChangeDetectorRef up to the super class, even if it doesn’t need to use it itself:
```typescript
@Component({
    selector: 'app-special-button',
    template: ``
})
export class SpecialButtonComponent extends ButtonComponent {

    constructor(cdr: ChangeDetectorRef){
        super(cdr)
    }

    anotherMethod(){

    }
}
```

## Interface Segregation principle

The interface segregation principle states that: Clients should not be forced to depend upon interfaces that they do not use.

This is something that is relevant to Angular, let’s consider an example. We might have an application that handles displaying Articles. The specific entity here doesn’t really matter, the main point is that we are displaying something using the master/detail pattern. We did this with the todo application, but I think articles work better for this example since they will generally have more properties.

We might have an interface representing articles that looks like this:

```typescript
interface Article {
    id: number;
    title: string;
    summary: string;
    author: string;
    datePublished: string;
    dateUpdated: string;
    content: string;    
}
```
We might also have two different components that deal with displaying articles. We might have one component that handles displaying a summary of the article in a list, and we might have one component that handles displaying the full details and content of an article.

Now, let’s say our ArticleSummary component displays the following details:

`title`

`summary`

`author`

`dateUpdated`

If we have our ArticleSummary component use the Article interface we are violating the interface segregation principle. It is depending on an interface that contains properties it does not need. Our component that displays the full details of an article would use all these fields.

To fix this, and adhere to the interface segregation principle, we might do something like this instead:

```typescript
interface Article {
    id: number;
    title: string;
    summary: string;
    author: string;
    dateUpdated: string;
}

interface ArticleDetails extends Article {
    datePublished: string;
    content: string;   
}
```
Now our summary component can use the Article interface, and the full details component can use the ArticleDetails interface, and neither depend on an interface with properties they don’t use. Exactly how we break these interfaces up might depend on how we intend to structure our application.

Once again, this is something to always be taken in context. This is a good ideal, but not always completely necessary. Given the example above, I would probably just shamefully violate the principle and have the one interface. Generally, the bigger and more complex an application becomes, the more important it is to follow rules like these.

## Dependency Inversion

The last principle for us to discuss is dependency inversion and it ties in to our discussion on inheritance and the Liskov substitution principle. This principle states: depend upon abstractions, not concretions.

It is easier to understand this if you know what abstract classes and methods are. I’ll let you know right away that the use cases for this in Angular are reasonably advanced, and not something most people will need to use (we do use these concepts quite a lot second hand though, because Angular exposes some of its functionality this way - we will talk about that soon).

An abstract class or method defines the idea or “shape” of a class or method. For example, I might have an abstract math class that looks like this:
```typescript
abstract class Math {
    abstract add(x: number, y: number): number;
    abstract multiply(x: number, y: number): number;
}
```
This defines the “shape” of a class called Math. It should define a function called add that takes two numbers as parameters, and returns a number. The same goes for multiply.

However, it doesn’t specify how these functions work. This is an abstraction. We could then provide a concrete implementation for this (a concretion) by defining those methods:

```typescript
class MyMath implements Math {
    add(x: number, y: number) {
        return x - y;
    }

    multiply(x: number, y: number) {
        return x / y;
    }
}
```
We are forced to adhere to the “contract” specified by the Math abstract class by using the implements keyword… but we are being a little silly. I am doing what is required, but I am providing nonsensical methods in this case. I could also create another concretion based on the same abstract class:

```typescript
class SensibleMath implements Math {
    add(x: number, y: number) {
        return x + y;
    }

    multiply(x: number, y: number) {
        return x * y;
    }
}
```
This one is more sensible. The idea with this principle is that our application would depend on Math which is just an abstraction. We could then provide whatever implementation of that we want - whether that is SensibleMath or MyMath. As long as we adhere to the structure set out by Math our application will not care either way.

This is used quite a lot by Angular itself. For example, when we create a pipe we do this:

```typescript
@Pipe({
    name: 'myPipe'
})
export class MyPipe implements PipeTransform {
    transform(value){
        return value;
    }
}
```
Angular defines an abstract class called PipeTransform that we implement. By implementing this class, we are forced to supply the transform method which Angular is expecting so that it knows how it should transform the values in the template. Another place we see this in Angular is when we use lifecycle hooks like OnInit and OnDestroy.

But what might this look like in our own code? Again, this is more advanced and not something you will typically run into. However, we can make quite effective use of this principle by using providers.

We have seen this already:
```typescript
@Component({
    selector: 'app-list',
    template: `template goes here`,
    providers: [ListService]
})
export class ListComponent {
    constructor(private listService: ListService)
}
```
We have the ability to specify providers (we are doing it directly on a component here, but you can also specify providers at a module or root level). What we haven’t seen is that we can provide the ListService “token”, but actually use a different implementation for it. For example:
```typescript
@Component({
    selector: 'app-list',
    template: `template goes here`,
    providers: [
        {
            provide: ListService,
            useClass: EmployeeListService
        }
    ]
})
export class EmployeeListComponent {
    constructor(private listService: ListService)
}
```
Our component can still depend upon the abstraction of ListService but the actual implementation (concretion) will be provided by EmployeeListService. This can be swapped out at will as long as we are adhering to the “contract” set out by the abstract ListService. For example, we could do this:

```typescript
@Component({
    selector: 'app-list',
    template: `template goes here`,
    providers: [
        {
            provide: ListService,
            useClass: ProductsListService
        }
    ]
})
export class ProductsListComponent {
    constructor(private listService: ListService)
}
```
Now we are using Products instead. To give further context, maybe our abstract ListService specifies that each service that implements it requires a getListItems property:

```typescript
export abstract class ListService {
    abstract getListItems$: Observable<string[]>
}
```
Then our specific implementations/concretions just need to adhere to this:

```typescript
@Injectable()
export class EmployeeListService implements ListService {
    getListItems$ = of(['Josh', 'Kathy'])
}
```
```typescript
@Injectable()
export class ProductsListService implements ListService {
    getListItems$ = of(['Josh', 'Kathy'])
}
```
We can create as many different implementations of this as we like, and our application only ever depends specifically on the API defined by the abstract ListService.

# HASHNODE LINK : 
### https://siahashnode.hashnode.dev/activity-25-solid-principles-in-angular
