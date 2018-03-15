# About XCTest

XCTest is a framework to write unit tests in your iOS projects. It allows you to test your code using many test assertions like: boolean assertions, nil and non-nil assertions, error assertions etc. You can also test you asynchronous code using expectations.

More about it you can find at the official [Apple documentation](https://developer.apple.com/documentation/xctest)

# Short review about  BDD, Qucik & Nimble

* ## BDD

The first conception, that can be useful to understand how the Quick & Nimble works, is a Behavior-Driven-Development(BDD). You might wonder what excatly is BDD?

Basically BDD is about to create a software by describing it's behavior. It's a combination of two: Test-Driven-Development (TDD) and User-Acceptance-Test (UAT).

I [found here](https://medium.com/@TechMagic/get-started-with-behavior-driven-development-ecdca40e827b) a very helpful definition for what the BDD is:
>The primary purpose of BDD methodology is to improve communication amongst the stakeholders of the project so that each feature is correctly understood by all members of the team before development process starts. This helps to identify key scenarios for each story and also to eradicate ambiguities from requirements.()

and there is one another sentence that I really liked from [another blog post](https://www.toptal.com/freelance/your-boss-won-t-appreciate-tdd-try-bdd):

>Behavior-driven development should be focused on the business behaviors your code is implementing: the “why” behind the code. It supports a team-centric (especially cross-functional) workflow.

In my blog post I don't want to describe whole concept of BDD but I want focus on comparing two frameworks for testing Swift/Objective-C code. So if you want to know more about BDD concept please check blog posts linked above.

* ## Quick

Before we start, you need to know that Quick & Nimble are separate frameworks that can(and probably should) be used togheter - to get a better and efficent way to test your code.

Quick is a behavior-driven-development framework. It shares a pack of useful methods that allows you to structure your test code expressively, using `beforeEach`, `it`, `context` methods.

More about `Quick` you can find [here](https://github.com/Quick/Quick).

* ## Nimble

Nimble is a framework that should be used for expressing the expected outcomes from code that you're testing. All expectations are handled by `expect` methods. And as you can see below, the code with `expect` usage is very understandable and readable. Even for not programmers.

```swift
expect(1 + 1).to(equal(2))
expect(1.2).to(beCloseTo(1.1, within: 0.1))
expect(3) > 2
expect("seahorse").to(contain("sea"))
expect(["Atlantic", "Pacific"]).toNot(contain("Mississippi"))
expect(ocean.isClean).toEventually(beTruthy())
```

More about `Nimble` you can find [here](https://github.com/Quick/Nimble).

## Describe scenario Bank

## Compare methods signatures (XCTestEqual, expect, beforeEach vs setUp)

## Compare example MoneyTransfert

## Conclusions write test for people