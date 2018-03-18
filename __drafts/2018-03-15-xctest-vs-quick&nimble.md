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

# Compare methods signatures (XCTestEqual, expect, beforeEach vs setUp)

# Describe scenario Bank

The scenario that I choosed to compare those two frameworks is a simple Bank functionalitly. Let's imagine that we have super `SafeBank` in which we can create account for Customer, transfer moneys or print some basic information about the bank customer.

Here you have API for `SuperSafeBank` class.

```swift
    func createAccount(for client: Person, initialBalance: Int = 0) throws
    func hasAccount(_ owner: Person) -> Bool 
    func accountInfo(owner: Person) throws -> String
    func accountBalance(_ owner: Person) throws -> Int
    func transfer(_ money: Int, from sender: Person, to receiver: Person, then completion: (() -> ())? = nil) throws {
```


# Comparing

* ## Account creation tests

### Quick & Nibmle

```swift
context("Account creation & Account Info") {
    beforeEach {
        john = Person(firstName: "John", lastName: "Smith")
    }

    it("No account created for John Smith.") {
        expect(sut.hasAccount(john)).to(beFalse())
    }

    it("Create account for new client") {
        expect { try! sut.createAccount(for: john, initialBalance: 200) }.toNot(throwError())
    }

    it("Create account and check if exist") {
        try! sut.createAccount(for: john)
        expect(sut.hasAccount(john)).to(beTrue())
    }

    it("Create account for existing client - should throw error") {
        try! sut.createAccount(for: john)
        expect {
            try sut.createAccount(for: john)
        }.to(throwError { (error) in
            expect(error) == BankError.accountAlreadyExist
        })
    }
}
```

### XCTest

```swift
func testNoAccountCreated() {
    let john = Person(firstName: "John", lastName: "Smith")
    XCTAssertFalse(sut.hasAccount(john), "No account created for John Smith.")
}

func testCreateAndCheckAccount() {
    let john = Person(firstName: "John", lastName: "Smith")
    try! sut.createAccount(for: john)
    XCTAssertTrue(sut.hasAccount(john), "Create account for new client")
}

func testCreateAccountAndCheckIfExist() {
    let nonBankClient = Person(firstName: "John", lastName: "Smith")
    XCTAssertFalse(sut.hasAccount(nonBankClient), "Create account and check if exist")
}

func testCreateAccountForExistingClient() {
    let john = Person(firstName: "John", lastName: "Smith")
    try! sut.createAccount(for: john)

    XCTAssertThrowsError(try sut.createAccount(for: john)) { (error) -> Void in
        XCTAssertEqual(error as? BankError, BankError.accountAlreadyExist)
    }
}
```


* ## Money transfering tests asynchronous testing

### Quick & Nimble

```swift
context("Money transfer") {
    var john: Person!
    var paul: Person!

    beforeEach {
        john = Person(firstName: "John", lastName: "Doe")
        paul = Person(firstName: "Paul", lastName: "Smith")

        try! sut.createAccount(for: john, initialBalance: 1000)
        try! sut.createAccount(for: paul, initialBalance: 500)
    }

    it("Transfer 300 from John to Paul") {
        try! sut.transfer(300, from: john, to: paul)

        expect(try! sut.accountBalance(john)).toEventually(equal(700), timeout: 6)// transfer might last some time that's why timeout here
        expect(try! sut.accountBalance(paul)).toEventually(equal(800), timeout: 6)// transfer might last some time that's why timeout here
    }
}
```

### XCTest

```swift
func testMoneyTransfer() {
    let john = Person(firstName: "John", lastName: "Doe")
    let paul = Person(firstName: "Paul", lastName: "Smith")
    try! sut.createAccount(for: john, initialBalance: 1000)
    try! sut.createAccount(for: paul, initialBalance: 500)

    let expectation = XCTestExpectation(description: "Money transfer")

    try! sut.transfer(300, from: john, to: paul, then: {
        XCTAssertEqual(try! self.sut.accountBalance(john), 700, "Sender account balance")
        XCTAssertEqual(try! self.sut.accountBalance(paul), 800, "Receiver account balance")
        expectation.fulfill()
    })
    self.wait(for: [expectation], timeout: 6)
}
```


# Conclusions write test for people