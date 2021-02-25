# Products

|Scope|Description|
|-|-|
|`read_products`| Required to read products record.|
|`write_products`| Required to write to the products record.|

## What is a Product?
Products are the items tied to a subcription in ReCharge. If using an external catalog such as Shopify or BigCommerce, the Product resource contains information about the item in the remote catalog.

Within the product resource, you can set the charge frequency for that item, the day in a month to charge a customer, and more.

## "Product catalog" versus Products endpoint
The ReCharge product catalog is where data about each item in your external catalog is stored within our system. With our direct integrations, ReCharge automatically pulls information such as price, variant, SKU etc. into our product catalog at the time a user creates a Product in the Merchant Portal. If you're using our API integration, you need to manually add your inventory to the product catalog.

The products endpoint contains the subscription settings associated to a catalog product.

## Subscriptions and products

In the ReCharge Merchant Portal you can create products with subscription rules. Although you can create products by passing the `collection_id`, it is not recommended. Set the following fields when creating products via API, as rulesets will one day be deprecated. The following fields are required: `charge_interval_frequency`, `order_interval_frequency_options`, `order_interval_unit` and `storefront_purchase_options`.

|Property|Required|Value|Note|
|-|-|-|-|
|`charge_interval_frequency`|Required if item is prepaid| Min value: 1 <br>  Max value: 999||
|`cutoff_day_of_month`|Optional|• Min value: 1<br>• Max value: 31|`shipping_interval_unit` must be equal to "month"|
|`cutoff_day_of_week`|Optional|• Min value: 0 (Monday)<br>• Max value: 6 (Sunday)|`order_interval_unit` must be equal to "week".|
|`discount_type`|Optional||Valid option is only "percentage"|
|`order_day_of_month`|Optional||No default. Only applicable when `order_interval_unit` = “month”.|
|`order_day_of_week`| Optional|Value of 0 equals Monday, 1 equals Tuesday, etc.|Only applicable when `order_interval_unit` = “week”.|
|`order_interval_frequency_options`|Required|Must be array of integers or integers that are individual strings|Each integer/string must be greater than 0 and less than 999|
|`order_interval_unit`|Required|Valid values are “day”, “week”, and “month||
|`shopify_product_id`|Required|||
|`storefront_purchase_options`|Required||Valid options are “subscription_only” or “subscription_and_onetime”.|

## Use cases
<!--
type: tab
title: Updating a Product
-->
`PUT` to `products:id`

```json
'{
  "discount_amount": 15.0,
  "subscription_defaults": {
    "charge_interval_frequency": 4,
    "order_interval_frequency_options": [
      "1",
      "6"
    ]
  }
}'
```
<!-- type: tab-end -->


## Resources
[Products reference](https://developer.rechargepayments.com/#products)

