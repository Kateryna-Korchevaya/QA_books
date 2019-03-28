# Cheatsheets: Cypress Best practices & Common Code reviews

## General

Ask yourself: Will I understand this after a month?

## Cookies

#### Cookies: Set them before `visit`

```js
cy.setCookie("cookie_consent", "agreed")
cy.visit("…")
```

## Waiting

#### Prefer `.get` with longer timeout over `.wait`

```js
cy.visit("»some slow page«")

// Sub-optimal
cy.wait(10 * 1000)
cy.get("…").click()

// Good
cy.get("…", { timeout: 14 * 1000 }).click()

// Also good
// Change defaultCommandTimeout in configuration to 14000
cy.get("…").click()
```

#### Avoid chaining `.wait` as it could be confusing

```js
// Bad
cy.visit("some slow page")
cy.get("element load").wait(10000).click()
// This is actually not doing what it looks it's doing
// It's trying to find element with default timeout (4s), THEN waiting 10s, and THEN clicking
// So if the element won't be on the page in 4s, test will fail

// Better, if needed. But still prefer timeout
cy.visit("some slow page")
cy.wait(10000)
cy.get("element load").click()
```

## Selecting & targeting

* **TODO**: Styled components, `Foo__StyledFoo_xyz`
* **TODO**: Selecting dynamic classes (invalid, touched, ...)

#### Don't rely on generated attributes
 

```js
// Bad
cy.get("[class='LanguageCurrent__Container-sc-1qu37au-0 ewtsmp']").click()

// Better
cy.get("[class^='LanguageCurrent']").click()
```

#### Strive for descriptive selectors

```js
cy.get("[data-tkey='booking.global.agreement.text_new2']").check() // 😐
cy.get(".ReservationAgreement checkbox").check() // 🙏
```

#### Explain unclear selectors

```js
// Suboptimal
cy.get(".InsuranceOption:last").click()

// Good
cy.get(".InsuranceOption:last").click() // Select "Premium insurance"
```

#### Classes are fine as selectors when you use verbose naming methodology (like BEM)

```js
cy.get(".BookingPassengerEditSummaryInfo .BookingPassengerEditSummaryInfo-wrap-single._original")
// ==>
cy.get(".BookingPassengerEditSummaryInfo-wrap-single._original")
```


###### Classical CSS

```css
.Hotels .Ad .Title .Actions {}
```

###### BEM

```css
.Hotels-Ad-Title-Actions {}
```

#### Avoid unnecessary `find`s

```js
cy
  .get(".SpecialAssistanceFormComponent")
  .find("button")
  .click()
// ==>
cy
  .get(".SpecialAssistanceFormComponent button")
  .click()
```

#### Avoid implementation details in selectors

```js
cy.get("[src='/images/flags/spFlag-cz.png']")
// ===>
cy.get("[src$='cz.png']")
```

#### Keep selectors consistent

###### Given
```html
<div class="PaymentFormCard">
  <input
    type="text"
    name="payment.card.number"
    autocomplete="cc-number"
    data-test="payment-cc-number-input"
  />
  <!-- … more inputs -->
</div>
```

```js
cy.get(".PaymentFormCard [autocomplete='cc-number']").type("...")
cy.get("input[name='payment.card.number']").type("...")
cy.get("[data-test='payment-cc-number-input']").type("...")
```

All are valid and acceptable selectors, but for clarify, choose one and stick to it.

#### Prefer using values over text content for translation independent tests

```html
<select name="gender">
  <option value="mr">Hombre</option>
  <option value="ms">Mujer</option>
</select>

<select name="birthMonth">
  <option value="01">Leden</option>
  <option value="02">Únor</option>
  <!-- ... -->
</select>
```

```js
// Sub-optimal
cy.get("select[name='gender']").select("Hombre")
cy.get("select[name='birthMonth']").select("Leden")

// Good
cy.get("select[name='gender']").select("mr")
cy.get("select[name='birthMonth']").select("01")
```

## Structuring

#### Consider lot of `it`s vs fewer `it`s

Few `it`s

```js
it("title", () => {
  cy.veryClearCommand()
  veryClearFunction()
  cy.log("comment what next command does")
  notSoDescriptiveFunction()
})
```

**Lot of `it`s**

```js
describe("title", () => {
  it("very clear description", () => {
    cy.veryClearCommand()
  })
  
  it("another very clear description", () => {
    cy.anotherVeryClearCommand()
  })
  
  it("description of not descriptive command", () => {
    cy.notSoDescriptive()
  })
})
```

#### Strive for both extensiveness and brevity

![](https://d2mxuefqeaa7sj.cloudfront.net/s_7F57A7A8D35FC5303BAC42EE7925F708AE045380DF4F28A8656DA8AF91080EA4_1539180263216_image.png)

#### Be consistent when possible

```js
// Suboptimal
cy.get("[type=email]").type("test@example.com")
cy.get("[name='contact.phone']").type("123456789")

// Good
cy.get("[name='contact.email']").type("test@example.com")
cy.get("[name='contact.phone']").type("123456789")
```

#### Use hierarchical structure for tests

![](https://d2mxuefqeaa7sj.cloudfront.net/s_7F57A7A8D35FC5303BAC42EE7925F708AE045380DF4F28A8656DA8AF91080EA4_1539243462974_image.png)
![](https://d2mxuefqeaa7sj.cloudfront.net/s_7F57A7A8D35FC5303BAC42EE7925F708AE045380DF4F28A8656DA8AF91080EA4_1539244176381_image.png)

#### Open/load/visit the page in `before` hook, so it’s possible to add `.only` to `it`s

```js
// BAD
describe("Payment", () => {
  it("loads", () => {
    cy.visit()
  })
  it("checks something", () => { // .only not possible to add
    cy.get().click()
  })
})

// GOOD
describe("Payment", () => {
  before(() => {
    cy.visit()
  })
  it("checks something", () => { // .only possible to add
    cy.get().click()
  })
})
```

#### Prefer skipping to commenting

Commented code is not treated as code in editors and it will not be considered in linting / refactorings / searching for usages.

```js
function goToPayment() {
  cy.get(".Booking-dialog .Button").click()
}

it.skip("Something", () => {
  goToPayment() // Editor handles this
})

/*
it("Something", () => {
  goToPayment() // Editor does not handle this
})
*/
```

## Clarity

#### Explain non-obvious

```js
// Suboptimal
cy.get("button").contains("Continue").click()
cy.get("button").contains("Continue").click()
cy.get("button").contains("Continue").click()

// Good
cy.get("button").contains("Continue").click() // Continue to "Shipping"
cy.get("button").contains("Continue").click() // Continue to "Overview"
cy.get("button").contains("Continue").click() // Continue to "Payment"
```

```js
// Suboptimal
cy.get("button").click().click()

// Good
cy.get("button").click().click() // Add two items
```

#### Explain non-standard

```js
// TODO: Explain
Cypress.on(
  "uncaught:exception",
  err => err.message.indexOf("Cannot set property 'aborted' of undefined") === -1,
)
```

#### Explain force usage

```js
cy.get("[name='cardExpirationYear']").type("20", { force: true }) // it's weirdly covered by ...
```

#### Explain skipping tests

```js
describe.skip("…", () => { // Disabled due to flakiness // TODO: Solve it and un-skip

it.skip("…", () => { // Feature is temporarily disabled, un-skip when enabled
```

#### Prefer plain functions over Cypress commands

Prefer simple Javascript functions over Cypress commands.

**Use cypress commands only if:**

* chaining, and using subject returned from previous command
* chaining, and providing returned subject for next commands in chain
* 100% sure you want to extend `cy`

***otherwise, use simple javascript functions***

Bad

```js
// mmbCommands.js
Cypress.Commands.add("mmbLoad", mock => {
  cy
    .visit(`/en/manage/123456789/${mock}?override_api=true}`)
    .get(".BookingHeader")
    .wait(500) // wait for ancilliaries api calls
})

// some-tests-spec.js
describe("MMB: Check-in", () => {
  before(() => {
    cy.mmbLoad("default:default")
  })
```

Good

```js
// mmbCommands.js
export function load(mock) {
  cy
    .visit(`/en/manage/123456789/${mock}?override_api=true}`)
    .get(".BookingHeader")
    .wait(500) // wait for ancilliaries api calls
}

// some-tests-spec.js
import * as mmbCommands from "./../mmbCommands"
describe("MMB: Check-in", () => {
  before(() => {
    mmbCommands.load("default:default")
  })
```

**Why?** Awesome support in IDE and static analysis!

* [Find usages](http://take.ms/e95GS)
* [Go to definition](http://take.ms/5SFIz)
* [Quick definition](http://take.ms/OGQMW)
* [Highlight errors](http://take.ms/XQ5YN)
* ...many others

#### Don’t overcomplicate things with unnecessary levels of abstractions

```js
// Bad
Cypress.Commands.add("mmbIsAccountBookingListVisible", () => {
  cy.get(".AccountBookingList")
})
```

#### Readability > Prettier

![](https://d2mxuefqeaa7sj.cloudfront.net/s_7F57A7A8D35FC5303BAC42EE7925F708AE045380DF4F28A8656DA8AF91080EA4_1539245406229_image.png)

#### Use `within` when appropriate

![](https://d2mxuefqeaa7sj.cloudfront.net/s_7F57A7A8D35FC5303BAC42EE7925F708AE045380DF4F28A8656DA8AF91080EA4_1539256387889_image.png)

## Unsorted (yet)

#### Simplify via afterEach

![](https://d2mxuefqeaa7sj.cloudfront.net/s_77ECE64E7E3CF5A437BDA620719F63E668BE8780F72118332B3499A67B8F19BB_1540207373888_image.png)

#### `should("exist")` is implicit after `get` or `find`

```js
cy.get(".navbar .logo").should("exist")
cy.get(".navbar").find(".logo").should("exist")
// is same as
cy.get(".navbar .logo")
cy.get(".navbar").find(".logo")
```

#### Use `{ }` when using arrow functions for tests

([Slack announcement](https://skypicker.slack.com/archives/C050XFGMN/p1542196373169200))
**Reasoning**
Without `{ }`, expression is implicitly returned.
When returned value is promise-like (and almost all cypress commands are),
Mocha switches to async mode and requires explicit completion
(usually via `done()` callback)
Cypress is doing some behind-the-scenes magic to work even without `{ }`,
but it's not standard and
it's breaking other standardised, well behaved, tools (like test coverage)

Example

```js
// Bad, bad boy
it("some check", () =>
  cy.verySophisticatedCommand("Foo", "Whoo"))

// Good citizen
it("some check", () => {
  cy.verySophisticatedCommand("Foo", "Whoo"))
}
```