# node-sales-tax

[![Build Status](https://img.shields.io/travis/valeriansaliou/node-sales-tax/master.svg)](https://travis-ci.org/valeriansaliou/node-sales-tax) [![Test Coverage](https://img.shields.io/coveralls/valeriansaliou/node-sales-tax/master.svg)](https://coveralls.io/github/valeriansaliou/node-sales-tax?branch=master) [![NPM](https://img.shields.io/npm/v/sales-tax.svg)](https://www.npmjs.com/package/sales-tax) [![Downloads](https://img.shields.io/npm/dt/sales-tax.svg)](https://www.npmjs.com/package/sales-tax)

International sales tax calculator for Node (offline, except for VAT number validation). Kept up-to-date.

## Who uses it?

<table>
<tr>
<td align="center"><a href="https://crisp.im/"><img src="https://valeriansaliou.github.io/node-sales-tax/images/crisp.png" height="64" /></a></td>
</tr>
<tr>
<td align="center">Crisp</td>
</tr>
</table>

_👋 You use sales-tax and you want to be listed there? Contact me!_

## How to install?

Include `sales-tax` in your `package.json` dependencies.

Alternatively, you can run `npm install sales-tax --save`.

## How to use?

This module may be used to acquire the billable VAT percentage for a given customer. You may also use it directly to process the total amount including VAT you should bill; and even to validate a customer's VAT number.

In any case, import the module to your code:

`var SalesTax = require("sales-tax");`

### Check if a country / state has sales tax

Check if France has any sales tax (returns `true`):

```javascript
var franceHasSalesTax = SalesTax.hasSalesTax("FR")  // franceHasSalesTax === true
```

### Get the sales tax for a customer

**Given a French customer VAT number** (eg. here `SAS CLEVER CLOUD` with VAT number `FR 87524172699`):

```javascript
SalesTax.getSalesTax("FR", "87524172699")
  .then((tax) => {
    // This customer is VAT-exempt (as it is a business)
    /* tax ===
      {
        type   : "vat",
        rate   : 0.00,
        exempt : true
      }
     */
  });
```

Note: Clever-Cloud is a real living business from France, check [their website there](https://www.clever-cloud.com).

**Given a Latvian customer without any VAT number** (eg. a physical person):

```javascript
SalesTax.getSalesTax("LV")
  .then((tax) => {
    // This customer has to pay 21% VAT (as it is a physical person)
    /* tax ===
      {
        type   : "vat",
        rate   : 0.21,
        exempt : false
      }
     */
  });
```

**Given a Spanish customer who provided an invalid VAT number** (eg. a rogue individual):

```javascript
SalesTax.getSalesTax("ES", "12345523")
  .then((tax) => {
    // This customer has to pay 21% VAT (VAT number could not be authenticated against the European Commission API)
    /* tax ===
      {
        type   : "vat",
        rate   : 0.21,
        exempt : false
      }
     */
  });
```

### Process the price including sales tax for a customer

**Given an Estonian customer without any VAT number, buying for 100.00€ of goods** (eg. a physical person):

```javascript
SalesTax.getAmountWithSalesTax("EE", 100.00)
  .then((amountWithTax) => {
    // This customer has to pay 20% VAT
    /* amountWithTax ===
      {
        type   : "vat",
        rate   : 0.20,
        exempt : false,
        price  : 100.00,
        total  : 120.00
      }
     */
  });
```

### Validate tax number for a customer

**Given a French customer VAT number** (eg. here `SAS CLEVER CLOUD` with VAT number `FR 87524172699`):

```javascript
SalesTax.validateTaxNumber("FR", "87524172699")
  .then((isValid) => {
    // isValid === true
  });
```

**Given a Latvian customer without any VAT number** (eg. a physical person):

```javascript
SalesTax.validateTaxNumber("LV")
  .then((isValid) => {
    // isValid === false
  });
```

**Given a Spanish customer who provided an invalid VAT number** (eg. a rogue individual):

```javascript
SalesTax.validateTaxNumber("ES", "12345523")
  .then((isValid) => {
    // isValid === false
  });
```

### Check if a customer is tax-exempt

**Given a French customer VAT number** (eg. here `SAS CLEVER CLOUD` with VAT number `FR 87524172699`):

```javascript
SalesTax.isTaxExempt("FR", "87524172699")
  .then((isTaxExempt) => {
    // isTaxExempt === true
  });
```

## Where the tax data is pulled from?

The tax data is pulled from [VAT, GST and sales tax rates — ey.com](http://www.ey.com/gl/en/services/tax/worldwide-vat--gst-and-sales-tax-guide---rates).

**It is kept up-to-date with the year-by-year tax changes worldwide.**

Some countries have multiple sales tax, eg. Brazil. In those cases, the returned sales tax is the one on services. Indeed, I consider most users of this module use it for their SaaS business — in other words, service businesses.

## How are tax numbers validated?

For now, this library only supports tax number (VAT number) validation for European countries. European VAT numbers are validated against the official `ec.europa.eu` API, which return whether a given VAT number exists or not. This helps you ensure a customer-provided VAT number is valid (ie. you don't have to bill VAT for this customer).
