# Unit Testing

Unit Testing is the art of writing maintenable source code. And you do so by writing tests that guarantee you are doing the right thing. 

> Unit Testable code is Clean Code.

The foundation of your test suite will be made up of unit tests. The number of unit tests in your test suite will largely outnumber any other type of test.

## What is a Unit?

If you ask three different people what _"unit"_ means in the context of unit tests, you'll probably receive four different, slightly nuanced answers. To a certain extent it's a matter of your own definition and it's okay to have no canonical answer.

If you're working in a functional language a unit will most likely be a single function. Your unit tests will call a function with different parameters and ensure that it returns the expected values. In an object-oriented language a unit can range from a single method to an entire class.

## Why you should write Unit Tests

We are in continuous search of faster bugs-free software delivery.
The pressure on deadlines may induce you to think you do not have time to write more code to test your actual code.

The more your tests resemble the way your software is used, the more confidence they can give you.

> Code without unit tests is Legacy Code.

## What should i test?

The good thing about unit tests is that you can write them for all your production code classes, regardless of their functionality or which layer in your internal structure they belong to. You can unit tests controllers just like you can unit test repositories, domain classes or file readers. 

A unit test  should at least test the _public interface_ of the unit under test. Private methods/functions which are used in the public interface should be already covered by testing the public methods.

I often hear opponents of unit testing (or TDD ) arguing that writing unit tests becomes pointless work where you have to test all your methods in order to come up with a high test coverage. They often cite scenarios where an overly eager team lead forced them to write unit tests for getters and setters and all other sorts of trivial code in order to come up with 100% test coverage.

There's so much wrong with that.

Yes, you should test the public interface. More importantly, however, you don't test trivial code. Don't worry, [Kent Beck said it's ok.](https://stackoverflow.com/questions/153234/how-deep-are-your-unit-tests/) You won't gain anything from testing simple getters or setters or other trivial implementations (e.g. without any conditional logic). Save the time, that's one more meeting you can attend, hooray!

## How you should write your Unit Tests

In order to test a single Unit you should _mock or stub_ the other collaborators of the Unit. A collaborator is another class, method, function used in that particular Unit.

```java
@RestController
// this is tghe Unit under test
public class CreditEnginesController {

    // this is a collaborator
    private final MaxAmountRepository maxAmountRepository;
    // this is a collaborator
    private final BureauClient bureauClient;

    @Autowired
    public CreditEnginesController(final MaxAmountRepository maxAmountRepository, final BureauClient bureauClient) {
        this.maxAmountRepository = maxAmountRepository;
        this.bureauClient = bureauClient;
    }

    @GetMapping("/credit-policies/max-amount/{piva}")
    public String maxAmount(@PathVariable final String piva) {
        var maxAmount = maxAmountRepository.findBy(piva);

        return maxAmount
                .orElse(DEFAULT_MAX_AMOUNT);
    }

    @GetMapping("/credit-policies/unpaids/{piva}")
    public Unpaid[] unpaids(@PathVariable final String piva) {
        return bureauClient.fetchUnpaids()
                .map(BureauResponse::getUnpaids)
                .orElse(SingletonList.emptyList());
    }
}
```

Once you got a hang of writing unit tests you will become more and more fluent in writing them. Stub out external collaborators, set up some input data, call your subject under test and check that the returned value is what you expected.

### Mocks and Stubs

Using Mocks or Stubs means that you replace a real thing (e.g. a class, module or function) with a fake version of that thing. The fake version looks and acts like the real thing (answers to the same method calls) but answers with canned responses that you define yourself at the beginning of your unit test.

Regardless of your technology choice, there's a good chance that either your language's standard library or some popular third-party library will provide you with elegant ways to set up mocks. And even writing your own mocks from scratch is only a matter of writing a fake class/module/function with the same signature as the real one and setting up the fake in your test.

They let you focus on testing the real thing under test.

### Structure of the test

You should use the same rule of Clean Code when writing tests. 
Use the following convention:

1. **A**rrange: it is the part where you setup your mock and prepare the context to run the test.
1. **A**ct: it is when you "stimulate" the test making the component under test do its job.
2. **A**ssert: it is where you check your expectations.

Example:
```javascript
test('createUser should return a good User', () => {
    // Arrange
    const name = 'aName';
    const surname = 'aSurname';
    
    // Act
    const user = createUser(name, surname);
    
    // Assert
    expect(user.fullName).toBe('aName aSurname');
})
```

Can you see it? They are very similar to **Acceptance Criteria's** GIVEN/WHEN/THEN.

### Do not test implementation details

There's a fine line when it comes to writing unit tests: they should ensure that all your non-trivial code paths are tested (including happy path and edge cases). At the same time they shouldn't be tied to your implementation too closely.
Tests that are too close to the production code quickly become annoying. As soon as you refactor your production code (quick recap: refactoring means changing the internal structure of your code without changing the externally visible behaviour) your unit tests will break.

This way you lose one big benefit of unit tests: acting as a safety net for code changes. You rather become fed up with those stupid tests failing every time you refactor, causing more work than being helpful; and whose idea was this stupid testing stuff anyways?

What do you do instead? Don't reflect your internal code structure within your unit tests. Test for __observable behaviour__ instead. Think about

if _I enter values X and Y, will the sum function return Z?_

instead of

if _I enter X and Y, will the method SUM of the intenral class MATH be called and the result is give as output of the sum function?_

## How to test Front End projects

You should clearly separe:

- **Presentational** components
    1. Are concerned with how things look.
    4. Have no dependencies on the rest of the app, such as actions or stores.
    5. Don’t specify how the data is loaded or mutated.
    6. Receive data and callbacks exclusively via props.
    7. Rarely have their own state (when they do, it’s UI state rather than data).
    8. Are written as functional components unless they need state, lifecycle hooks, or performance optimizations.

You can **avoid** unit test on them, better test layout with [Snapshots](https://jestjs.io/docs/snapshot-testing).

Examples:

```jsx
export const UltimateBeneficialOwnerAge = ({ onChange, limit }) => 
  (
    <input
      type="number"
      onChange={onChange}
      maxLength={limit} 
    />
  );

export const LegalRepresentativeInfo = ({ infos }) => 
  (
    <ul>
      {infos.map(info => <li>{info.label}: {info.value}</li>)}
    </ul>
  );
```

- **Container** components 
    1. Are concerned with how things work.
    2. May contain both presentational and container components inside but usually don’t have any DOM markup of their own except for some wrapping divs, and never have any styles.
    3. Provide the data and behavior to presentational or other container components.
    5. Are often stateful, as they tend to serve as data sources.

They should be **properly** unit tested, mocking the props where there is fetching involved.

Examples:

```jsx
import { useState, useEffect } from 'react';
import { getInfos } from '../services/legalRepresentative';

export const LegalRepresentative = () => {
  const [infos, setInfos] = useState([]);

  useEffect(() => {
    getInfos(setInfos);
  }, []);
 
  return (
   <LegalRepresentativeInfo infos={infos} />
  );
}
```

[A more complete example involving fetch requests](https://testing-library.com/docs/react-testing-library/example-intro)

### Test clicks

```javascript
import {render, screen, fireEvent} from '@testing-library/react'

const Button = ({onClick, children}) => (
  <button onClick={onClick}>{children}</button>
)

test('calls onClick prop when clicked', () => {
  const handleClick = jest.fn()
  render(<Button onClick={handleClick}>Click Me</Button>)
  
  fireEvent.click(screen.getByText(/click me/i))
  
  expect(handleClick).toHaveBeenCalledTimes(1)
})
```

### Use

- [Jest](https://jestjs.io/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Mock Service Worker](https://mswjs.io/)
- [Cypress](https://www.cypress.io/)

## How to test Microservices

You should clearly separe:

- `Controller` classes provide REST endpoints and deal with HTTP requests and responses
- `Repository` classes interface with the database and take care of writing and reading data to/from persistent storage
- `Client` classes talk to other APIs,like fetching JSON responses via HTTP from 3rd party providers
- `Domain` classes contain our specific Business logic and should not depend on the Framework

https://martinfowler.com/articles/practical-test-pyramid.html

### Use

- [JUnit 5](https://junit.org/junit5/)
- [Mockito](https://site.mockito.org/)
- [MockMVC](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html)
- [WireMock](https://wiremock.org/)

## How to test ETL / Data-related stuffs

TODO

## Agile Development Learning Path

XPeppers Learning Path: https://github.com/xpeppers/starway-to-orione
