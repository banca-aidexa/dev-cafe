# Clean Code

> Clean Code is code written for other Humans.

It means, you should write code which is **readable**: it will be easier to understand, maintain and extend.

Clean Code is (more) secure code.

## Why should you write Clean Code

Pretend the next one who should maintain your code is:

1. a serial killer
1. who knows your home address

## Naming things

> There are only two hard things in Computer Science: cache invalidation and naming things.
-- Phil Karlton

### Do not use abbreviations

Characters are free to use.
Do not use abbreviations or minified code when writing a variable, function, class or method name.

```javascript
// bad
function getFn(user) {
    const n = getUserName();
    const s = getUserSurname();
    return n + ' ' + s;
}

// good
function getFullName(user) {
    const name = getUserName();
    const surname = getUserSurname();
    return name + ' ' + surname;
}
```

### Pick a convenction and stick to it

Do not mix `camelCase`, `PascalCase` and `snake_case`.
Pick one and stick to it.

### Do not use Magic Numbers

If you are using a number which has a *meaning*, save it in a named constant.

```javascript
// bad
if (amount > 100000) {
    console.log('you are requesting too much')
}

// good
const MAXIMUM_AMOUNT_IN_EURO = 100000;

if (amount > MAXIMUM_AMOUNT_IN_EURO) {
    console.log('you are requesting too much')
}

// bad
if (process.runningTime > 3600000) {
    console.log('one hour is passed')
}

// good
const MILLISENCONDS_IN_ONE_HOUR = 1000 * 60 * 60;

if (process.runningTime > MILLISECONDS_IN_ONE_HOUR) {
    console.log('one hour is passed')
}
```

### Express what your function is doing

```javascript
// bad - it's hard to tell from the function name what is added
function addToDate(date, month) {
  // ...
}
const date = new Date();
addToDate(date, 1);

// good
function addMonthToDate(month, date) {
  // ...
}
const date = new Date();
addMonthToDate(1, date);
```

### Clearly express conditions

```javascript
// bad - higher cognitive load
if (user.age >= 18 && user.drivingLicenseType === 'B') {
    console.log('drive your Tesla!')
}

// good - well written prose
if (canDrive(user)) {
    console.log('drive your Tesla!')
}

function canDrive(user) {
    return user.age >= 18 && user.drivingLicenseType === 'B';
}
```

As a rule of thumb your boolean variables/functions should always start with "is" or "has" or "can".

#### Avoid double negation

```javascript
// bad
if (!account.isDisabled) {
  // ...
}

// good
if (account.isEnabled) {
  // ...
}
```

## Do not comment your code

Your code should read as a well written prose, it's like a joke: if you have to explain, is not that good.
Comments can confuse and stay out of sync.
If you want to give an "instruction manual" use **types** and **unit tests**

```javascript
// bad - tautological
// this function multiply two numbers
function multiply(a, b) {
    return a * b;
}

// --------------------------------------------

// bad - unsync comment
// this function should return the maximum amount considering stampExpenses and substitutive tax
function getMaxAmount(product) {
    return MAX_AMOUNT * product.stampExpensesPercentage;
}

// --------------------------------------------

// bad - commented code
doStuff();
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();

// --------------------------------------------

// good - code to overcome a specific issue
function getDocument() {
    // Internet Explorer specific work-around
    return document.getElementsById;

    return document.querySelectorAll;
}

// --------------------------------------------

// good - optimized code which must stay this way for performance reasons

/* This is the fastest implementation of the hasing algorithm
* we have optimized it because the readable version was 200x slower and was impacting UX
*/
function hashIt(data) {
  let hash = 0;
  const length = data.length;

  for (let i = 0; i < length; i++) {
    const char = data.charCodeAt(i);
    hash = (hash << 5) - hash + char;

    // Convert to 32-bit integer
    hash &= hash;
  }
}
```
```java
// good - copied code from external source
// Converts a Drawable to Bitmap. via https://stackoverflow.com/a/46018816/2219998.
@NonNull
private Bitmap getBitmapFromDrawable(@NonNull Drawable drawable) {
    final Bitmap bmp = Bitmap.createBitmap(drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight(), Bitmap.Config.ARGB_8888);
    final Canvas canvas = new Canvas(bmp);
    drawable.setBounds(0, 0, canvas.getWidth(), canvas.getHeight());
    drawable.draw(canvas);
    return bmp;
}

// --------------------------------------------
```
```typescript
// bad - tautological comment
// this function sums two numbers. It returns NaN if one of the argument is not a number. It throws an error if both parameters are not number
function sum(a, b) {
    if (Number.isNaN(a) || Number.isNaN(b)) {
        return Number.NaN;
    }

    if (Number.isNaN(a) && Number.isNaN(b)) {
        throws new Error('both the arguments are not number');
    }

    return a + b;
}

// good - use types and test
function sum(a: number, b: number): number {
    if (Number.isNaN(a) || Number.isNaN(b)) {
        return Number.NaN;
    }

    if (Number.isNaN(a) && Number.isNaN(b)) {
        throws new Error('both the arguments are not number');
    }

    return a + b;
} 

// ... in test file

expect(sum(1,2)).toBe(3);
expect(sum('a', 2)).toBe(Number.NaN);
expect(sum('a', 'b')).toThrowError('both the arguments are not number')
```

## Avoid premature optimization

Start writing correct code. Then write it readable. Do not optimize before you measure there is a problem.

```javascript
// bad - just showing-off you know binary operations
return amount << 1;

// good
return amount * 2;
```

```javascript
// bad - more performant, less readable
const isParameterPresent = !!parameter;

// good
const isParameterPresent = Boolean(parameter);
// or
const isParameterPresent = parameter !== null && parameter !== undefined;
```

## Avoid deep nesting

You may be a Street Fighter fan, but please don't do this.

![Nested Hadouken](https://miro.medium.com/max/720/1*g4NuK2wpgB5hn_46bvzPmQ.png)

### Prefer early return to long if-else chain

```javascript
// bad
function evaluateStep() {
  let response;
  
  if (condition1) {
    response = 'first';
      if (condition1_1) {
        response = 'first_first';
      } else if (condition1_2) {
        response = 'first_second';
      }
  } else {
    response = 'second';
  }

  return response;
}

// better - no-else technique
function evaluateStep() { 
  if (condition1_1) {
    return 'first_first';
  }
  
  if (condition1_2) {
    return 'first_second';
  }
  
  if (condition1) {
    return 'first';
  }
  
  return 'second';
}
```
