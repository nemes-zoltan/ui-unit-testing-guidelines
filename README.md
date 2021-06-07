# UI Unit Testing Guidelines

For any unit to be testable, and to make sure the test cases don't over grow and that they make sense, there are two key principles that we need to touch on before diving in, these are the following: Single Responsibility Principle and Law of Demeter. If the units that we write don't make sense our tests won't either.

## Single Responsibility Principle (SRP)

SRP states that every module, class, or function should have responsibility over a single part of the functionality provided by the software, and that responsibility should be entirely encapsulated by the unit. Responsibility is a reason to change, a class or module should have one, and only one, reason to be changed.

Make components as specific to them as possible. Don't mix API, store communication etc... with UI logic and markup into the same component. Having components with multiple responsibilities are going to lead to a huge test case that will be hard to follow and maintain. Instead try splitting your logic into dedicated  components, this will result in smaller components that are easier to test.

## Law of Demeter (LoD)

In its general form, LoD is a specific case of loose coupling, and can be succinctly summarised in each of the following ways:
 - Each unit should have only limited knowledge about other units: only units "closely" related to the current unit.
 - Each unit should only talk to its friends; don't talk to strangers.
 - Only talk to your immediate friends.

The fundamental notion is that a given object should assume as little as possible about the structure or properties of anything else (including its subcomponents), in accordance with the principle of "information hiding". 

Example: An object A can request a service (call a method) of an object instance B, but object A should not "reach through" object B to access yet another object, C, to request its services. Doing so would mean that object A implicitly requires greater knowledge of object B's internal structure. Instead, B's interface should be modified if necessary so it can directly serve object A's request, propagating it to any relevant subcomponents. 

Let’s take an example of what this all means: Let’s say your working with some data, you have an API that sends you a list of users, each user object has generic user properties `name`, `birthday` etc... and it also has two properties called `address` and `paymentInfo`. The `address` and `pamentInfo` are separate objects with their own set of properties. Following LoD would mean that `User`, `Address` and `PaymentInfo` should have their own dedicated components/classes, the `User` component should not have the necessary markup/logic needed to display  the `address` model. It should instead delegate the `address` model to its own `Address` component which will take care of it, the same applies to the `paymentInfo`.

## What Should You Test?

Easiest to define is what you shouldn't test, which is the internal mechanics of a unit. If you find your self testing how a unit works inside and asking for information about what the units internal state is doing you are testing implementation details, and it will create a very high level of coupling between the unit and the test cases. You shouldn't need to change a test case because you changed a variable name or function name.  So what should you test? Behaviour, the way we need to think about tests is in a blackbox style where you don't see how the internal mechanics work you have inputs going inside and outputs coming outside the box, you should always be testing for a given input what is the produced output. 
Example: Lets take a component for example, you don't check what the components props look like or internal states, you give data for the props which is your input component internals do some magic, produces the markup which is the output that is what you want to test. To obtain different behaviours to write tests for different cases, you really have only 2 options you either interact with the markup which is the output or you introduce new input which should result in a different output, thats really the only 2 options that should concern you.
Remember your tests should break if a functionality was changed, removed but your test should not break if you change a variable or function name. A good test does not mean that it should be a complicated test it should focus on behaviour and not tie the markup or implementation detail to test cases. Read more here and a nice video here.

## Grouping Your Tests with Describe

Whenever you find yourself, having to test a attribute or part of a unit which might have different cases to test, you should always group all the related assertions (it  blocks) under a describe  block. Put describe  blocks for the cases that you are testing. Lets take an example, you have an attribute that changes an elements visibility it can take up the following values: `show`, `hide` or `disabled`, your test case should look something like:

```
describe('attribute_name', () => {
  describe('when shown', () => {
    beforeEach(() => {
      // Setup code for making the element shown
    })
    it('displays the element', () => {
      // Assertions for displaying of the element
    });
    ... // Other assertions if needed
  });
 
  describe('when hidden', () => {
    beforeEach(() => {
      // Setup code for making the element hidden
    })
    it('hides the element', () => {
      // Assertions for hidden element
    });
    ... // Other assertions if needed
  });
 
  describe('when disabled', () => {
    beforeEach(() => {
      // Setup code for making the element disabled
    });
    it('disables the element', () => {
      // Assertions for disabling the element
    });
    ... // Other assertions if needed
  });
})
```

A good test scenario can always be read in a sentence, from the first `describe` block to the last `it` block.

# Subject

All units are the subject of their test cases, its basically a naming convention so that our test cases are more consistent, few examples:
 - Your testing a function `add` which takes up two parameters `a`  and `b`  the subject of your test in this case is `add`, so you would be asserting that `subject(2,3)`  is `5`.
 - Your testing a class  called `Human`, your subject this case is the instantiated object of the class `Human` , this would result in something like: `subject = new Human({ name: "Bill" })`
 - Your testing a component, in this case your subject is the mounted  instance of the component.

# To Mock or not to Mock

General rule of thumb should be to mock as little as possible. Although there are a few exceptions, whenever you find yourself doing one of the following just because a subcomponent is relying on them just mock it out, examples:
 - If you find yourself needing to pass additional parameters to a component because a sub component is relying on them, then mock the subcomponent.
 - If you need to wrap a component in special wrappers because a sub component is using the router, etc.. mock the subcomponent.
 - If you need to mock back-end calls because a sub component is calling them, mock the subcomponent

**Remember your unit tests scope is to test the unit not the sub system below it.**

## Factory

Factories in testing are a useful tool that can help us to create mock data. It alleviates us from having to begin each test file with chunks of mock data. Provides us with means of defining relationships between our models. Experiencing change at any moment in our models, only requires us to update the data in one place. Factories sit under test/factories  each model should have its own dedicated factory. Relationships by default should be optionally created, meaning whenever you define a relationship don't forget to include an option `withEntityName`  and set it by default to false  so the extra relationships are only created if explicitly specified to do so. The following examples are in a library called `rosie`.

```
// Defining factories
 
import { Factory } from 'rosie';
import uuid from 'uuid';
 
const ModelA = Factory.define('ModelA')
.attrs({
  id: () => uuid(),
  name: 'Model name',
  created_at: () => Date.now().toLocaleString(),
  description: 'Model Description',
  // Any other attributes
})
.option('withModelB', false)
.attr('modelBs', ['modelBs', 'withModelB'], (modelBs, withModelB) => {
  if (!withModelB) {
    return [];
  } 
  return modelBs || Factory.buildList('ModelB', 5);
});
 
const ModelB = Factory.define('ModelA')
.attrs({
  id: () => uuid(),
  name: 'Model B name',
  // Any other attributes
});
 
// Using Factories inside test cases
 
...
 
beforeEach(() => {
  const items = Factory.build('ModelA'); // No relations
  const items = Factory.build('ModelA', {}, { withModelB: true }); // With Relations
  ...
})
...
```

## Shared Examples

Shared examples are shared behaviours between units that have similar behaviours. These examples live under `test/shared_examples` , they should abstract out certain generic behaviours. The basic usage of them is by including them in your test cases, example: 

```
// Definition
export default function something(arguments) {
  describe("something", () => {
    it("does something", () => {
      expect(
        subject.someFunction()
      ).toBe("result");
    });
  });
};
 
// Usage
itBehavesLike('something', arguments);
```

## Generic Component Types

Making an effort to extract shared logic between components or writing them in a similar fashion can also grant us with the benefits of easily sharing test examples between them. Here are some ideas for generic component types:
 - Form
 - API
 - Table
 - List
