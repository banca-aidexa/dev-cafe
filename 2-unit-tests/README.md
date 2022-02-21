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

## How you should write your Unit Tests

In order to test a single Unit you should _mock or stub_ the other collaborators of the Unit. A collaborator is another class, method, function used in that particular Unit.

```java
@RestController
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
