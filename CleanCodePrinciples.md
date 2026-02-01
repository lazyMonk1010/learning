# Clean Code Design Principles

---

## DRY (Don't Repeat Yourself)

Encourages developers to avoid duplicating code in a system. Basically what does it mean? If you have to do some task again and again repetitively—like calculating something, fetching some API, updating something in a database—then instead of writing the same code again and again, wouldn't it be better to just write a function and use that? That makes sense; that is the DRY principle.

**Formally it means:**
- Writing reusable pieces of code
- Avoiding redundancy

**What are the advantages?**
- Maintainable code
- Reusability
- Consistency

**Example:**

```javascript
// ❌ Violation - same logic repeated
function getTotalWithTax(order1) {
  return order1.subtotal * 1.1;
}
function getOrderTotal(order2) {
  return order2.subtotal * 1.1;
}

// ✅ DRY - single reusable function
const TAX_RATE = 1.1;
function calculateTotalWithTax(subtotal) {
  return subtotal * TAX_RATE;
}
```

---

## KISS (Keep It Simple, Stupid)

Keep your code as simple as possible. Don't over-engineer or add unnecessary complexity. If a simple loop works, don't replace it with a fancy abstraction just to look clever. Readable, straightforward code is easier to debug, understand, and change.

**Formally it means:**
- Prefer simplicity over complexity
- Avoid unnecessary abstractions and clever tricks
- Solve the problem in the most direct way

**What are the advantages?**
- Easier to understand and onboard
- Fewer bugs
- Faster to debug and modify

**Example:**

```javascript
// ❌ Overcomplicated
function isEven(n) {
  return n % 2 === 0 ? true : false;
}

// ✅ KISS
function isEven(n) {
  return n % 2 === 0;
}
```

---

## YAGNI (You Aren't Gonna Need It)

Don't build features or add code "just in case" you might need them later. Build only what is required *now*. Extra code adds maintenance cost, tests, and confusion—and often the "future" never comes or requirements change.

**Formally it means:**
- Implement only current requirements
- Avoid speculative or future-proofing code
- Add complexity only when there is a real need

**What are the advantages?**
- Less code to maintain
- Faster delivery
- Less risk of unused or wrong abstractions

**Example:**

```javascript
// ❌ YAGNI violation - generic "for the future" design
class UserService {
  createUser() {}
  updateUser() {}
  deleteUser() {}
  archiveUser() {}
  restoreUser() {}
  mergeUsers() {}  // "We might need this someday"
}

// ✅ YAGNI - only what you need now
class UserService {
  createUser() {}
  updateUser() {}
}
```

---

## Separation of Concerns (SoC)

Different parts of your system should handle different responsibilities. UI code shouldn't contain business logic; database code shouldn't format strings for display. Split your application into distinct layers or modules, each with one clear job.

**Formally it means:**
- Each module/layer has a single, well-defined responsibility
- Minimize coupling between different concerns
- Organize code by *what* it does, not by *when* it runs

**What are the advantages?**
- Easier to test (e.g. business logic without UI)
- Easier to change one part without breaking others
- Clearer structure and onboarding

**Example:**

```javascript
// ❌ Mixed concerns - UI + logic + data in one place
function handleSubmit() {
  const name = document.getElementById('name').value;
  const valid = name.length > 0 && name.length < 100;  // validation
  if (valid) {
    fetch('/api/users', { body: JSON.stringify({ name }) });  // API call
    document.getElementById('message').innerText = 'Saved!';  // UI update
  }
}

// ✅ Separated - validation, API, UI in different places
function validateName(name) {
  return name.length > 0 && name.length < 100;
}
async function saveUser(user) {
  await fetch('/api/users', { body: JSON.stringify(user) });
}
function handleSubmit() {
  const name = document.getElementById('name').value;
  if (validateName(name)) {
    await saveUser({ name });
    document.getElementById('message').innerText = 'Saved!';
  }
}
```

---

## Law of Demeter (Principle of Least Knowledge)

A module should only talk to its "immediate friends"—not to friends of friends. In practice: don't chain method calls on objects you got from another object (e.g. `user.getAddress().getCity()`). Ask the direct object for what you need, or give it a method that does the work.

**Formally it means:**
- Only use your own methods/properties and those of objects you create or receive as parameters
- Avoid long chains of dependencies (e.g. `a.getB().getC().doSomething()`)
- Prefer "tell, don't ask" where it makes sense

**What are the advantages?**
- Loose coupling
- Easier to refactor (internal structure can change)
- Clearer dependencies

**Example:**

```javascript
// ❌ Violation - reaching through multiple objects
const city = customer.getOrder().getAddress().getCity();

// ✅ Better - ask the direct object for what you need
const city = customer.getOrderCity();  // Order encapsulates how to get city
// or: const city = customer.getCity();  // Customer delegates to order/address
```

---

## Composition over Inheritance

Prefer building behavior by *composing* smaller parts (e.g. using interfaces, dependency injection, or "has-a" relationships) instead of deep inheritance trees ("is-a"). Inheritance often leads to rigid, fragile code; composition is more flexible and easier to change.

**Formally it means:**
- Favor "has-a" over "is-a" where possible
- Use small, focused components combined together
- Avoid deep or wide inheritance hierarchies

**What are the advantages?**
- More flexible and easier to change
- Less coupling and fewer surprises from overridden methods
- Easier to test (inject dependencies)

**Example:**

```javascript
// ❌ Inheritance - rigid, fragile
class Bird {
  fly() {}
}
class Penguin extends Bird {
  fly() { throw new Error("Can't fly"); }  // Violates Liskov, awkward
}

// ✅ Composition - flexible
class Bird {
  constructor(flyBehavior) {
    this.flyBehavior = flyBehavior;
  }
  fly() {
    this.flyBehavior.fly();
  }
}
const canFly = { fly() { console.log('Flying'); } };
const cannotFly = { fly() { console.log('Cannot fly'); } };
const sparrow = new Bird(canFly);
const penguin = new Bird(cannotFly);
```

---

## Avoid Premature Optimization

Don't optimize for speed or memory until you have evidence of a real problem (e.g. profiling, user-reported slowness). First make the code correct and clear; then measure; then optimize only the parts that matter.

**Formally it means:**
- Write clear, correct code first
- Measure before optimizing (profiling, benchmarks)
- Optimize only proven bottlenecks

**What are the advantages?**
- Less time wasted on optimizations that don't matter
- Cleaner, more maintainable code
- Focus on impact: optimize what actually slows things down

**Example:**

```javascript
// ❌ Premature - "clever" micro-optimization, hard to read
let s = '';
for (let i = 0; i < arr.length; ++i) s += arr[i] + (i < arr.length - 1 ? ',' : '');

// ✅ Clear first, optimize only if needed
let s = arr.join(',');
```

---

## Meaningful Names

Names of variables, functions, and classes should reveal intent. Avoid abbreviations (unless very common), magic numbers, and generic names like `data`, `temp`, or `x`. Good names act as documentation and reduce the need for comments.

**Formally it means:**
- Use names that describe purpose and usage
- Avoid misleading or vague names
- Keep consistent naming conventions (e.g. camelCase, PascalCase)

**What are the advantages?**
- Code is self-documenting
- Easier to read and refactor
- Fewer misunderstandings and bugs

**Example:**

```javascript
// ❌ Unclear
const d = 86400;  // what is 86400?
function proc(u) {
  return u.fn + ' ' + u.ln;
}

// ✅ Meaningful
const SECONDS_PER_DAY = 86400;
function getFullName(user) {
  return user.firstName + ' ' + user.lastName;
}
```

---

## Single Level of Abstraction (SLA)

Inside a function or block, work at one level of abstraction. Don't mix low-level details (e.g. indexing arrays, parsing strings) with high-level steps (e.g. "validate user", "save order"). Extract sub-steps into well-named functions so the main flow reads like a to-do list.

**Formally it means:**
- Each function does one logical thing at one level of detail
- Lower-level details are hidden in smaller functions
- The main function reads as a sequence of high-level steps

**What are the advantages?**
- Easier to understand the flow
- Easier to change implementation details
- Better readability and maintainability

**Example:**

```javascript
// ❌ Mixed levels - parsing, validation, and high-level logic mixed
function processOrder(orderText) {
  const parts = orderText.split(',');
  const id = parseInt(parts[0], 10);
  const amount = parseFloat(parts[1]);
  if (id > 0 && amount > 0) {
    saveToDb({ id, amount });
    sendConfirmation(id);
  }
}

// ✅ Single level of abstraction
function processOrder(orderText) {
  const order = parseOrder(orderText);
  if (isValidOrder(order)) {
    saveOrder(order);
    sendConfirmation(order.id);
  }
}
```

---

*This document covers Clean Code design principles excluding SOLID (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion), which are typically documented separately.*
