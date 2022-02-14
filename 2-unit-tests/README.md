# Unit Testing

Unit Testing is the art of writing maintenable source code. And you do so by writing tests that guarantee you are doing the right thing. 

> Unit Testable code is Clean Code.

## Why you should write Unit Tests

We are in continuous search of faster bugs-free software delivery.
The pressure on deadlines may induce you to think you do not have time to write more code to test your actual code.

The more your tests resemble the way your software is used, the more confidence they can give you.

> Code without unit tests is Legacy Code.

## How you should write your Unit Tests

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

https://martinfowler.com/articles/practical-test-pyramid.html

## How to test ETL / Data-related stuffs


## Agile Development Learning Path

XPeppers Learning Path: https://github.com/xpeppers/starway-to-orione
