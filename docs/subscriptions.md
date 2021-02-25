# Subscriptions

|Scope|Description|
|-|-|
|`read_subscriptions`| Required to read subscriptions record.|
|`write_subscription`| Required to write to the subscriptions record.|

## What is a Subscription?
Subscriptions represent individual items a customer receives on a recurring basis. Subscriptions are the core resources of the ReCharge API. A subscription record is comprised of a product added to an address. 

You can update the start date when a customer will first be charged and the frequency of each charge, the date of the month when an order is created, and the product's shipping frequency using the Subscriptions resource.

A customer can only have one subscription of the same product on one address. Customers can have multiple subscriptions of the same product but only if those subscriptions are attached to a different [Address](https://developer.rechargepayments.com/#update-an-address).

<!-- theme: info -->
> In the past products needed to be attached to a ruleset to be added to subscriptions, but we are in the process of deprecating [rulesets](https://support.rechargepayments.com/hc/en-us/articles/360008830873-Creating-subscription-rulesets).

## Updating a Subscription
If you want to change any of the following attributes, you must update all of them because they are tied to one another:

- `order_interval_unit`
- `order_interval_frequency`
- `charge_interval_frequency`

<!--theme: info-->
> When updating subscription `status` attribute from “CANCELLED” to “ACTIVE”, the following attributes will be set to `null`: `cancelled_at`, `cancellation_reason` and `cancellation_reason_comments`

### Request limit

When updating Subscriptions in bulk, there is a limit of 20 subscriptions per request.

## Deleting a Subscription
If a user deletes a subscription via the ReCharge Merchant Portal, you will still be able to retrieve it using the Subscriptions API. Deleting a Subscription via API is the only way to completely remove a subscription record from the database. 

## Delay a Charge regeneration
You can now delay subscription charge regenerations.

Each subscription update triggers a charge regeneration. This can be time consuming if your application is performing multiple updates in sequence. Use `commit_update: false` in the body of your Subscriptions `PUT` request to perform this action more quickly. Doing so will delay charge regeneration by 5 seconds. This allows you to run multiple calls to perform changes much faster because your application does not need to wait for every charge regeneration to complete between requests and responses. 

<!-- theme: info-->
> - For extra safety, we make sure that if a charge is processed before the regeneration resolves, we will auto-trigger the regeneration before processing.
> - If you want to trigger the regeneration yourself, set the `commit_update` parameter to `false`.

## Resources
- [Subscriptions reference](https://developer.rechargepayments.com/?php#subscriptions)
