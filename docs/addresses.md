# Addresses 

|Scope|Description|
|-|-|
|`read_addresses`| Required to write to the address record.|
|`write_addresses`| Required to write to the address record.|

## What are Addresses?
Addresses represent one of the many shipping locations a customer can have. An address is created when a customer purchases a subscription. [Subscriptions](https://developer.rechargepayments.com/#subscriptions) are tied to a given address. Each customer can have multiple address objects in a many-to-one relationship. You can override shipping rates using this endpoint.

![addresses](assets/images/addresses.png "addresses")

<!-- theme: info -->
> Customers can subscribe to the same product multiple times as long as they use a different address.

## Use cases

<!-- 
type: tab
title: Validate an Address
-->

`POST` to `/addresses/validate/`

### Example request body

```json
{
  "address1": "1776 Washington Street",
  "city": "santa monica",
  "state": "California",
  "zipcode": "90404"
}
```

<!-- 
type: tab
title: Override Shopify shipping lines
-->

`PUT` to /addresses/:id

### Example request body

```json
{
  "shipping_lines_override": [
    {
      "code": "Standard Shipping",
      "price": "10.00",
      "title": "Standard Shipping"
    }
  ]
}
```

<!-- type: tab-end -->

## Resources
[Addresses reference](https://developer.rechargepayments.com/#addresses)