# Metafields

|Scope|Description|
|-|-|
|write_\*RESOURCE\*|\*RESOURCE\* should be the API resource for which you're creating the metafield.|

## What are Metafields?
Metafields allow you to add additional attributes to certain ReCharge API resources. Use them to store extra information. A use case could be integrating ReCharge with a connector app to pass store data to an ERP. The following resources support Metafields:

- Stores
- Customers
- Subscriptions
- Orders
- Charges

<!-- theme: info -->
There is a limit of 50 metafields per `owner_id`.

When making a `GET` to retrieve a list of `/metafields/`, you must specify the `owner_resource` as a query string. For example:

`https://api.rechargeapps.com/metafields?owner_resource=customer`

## Use cases

<!-- 
type: tab
title: Retrieve metafields belonging to specific customer 
-->

`GET` to GET /metafields?owner_resource=customer&owner_id=:customer_id

<!--
type: tab
title: Update a metafield
-->

`PUT` to `https://api.rechargeapps.com/metafields/:id`

```json
{
  "metafield": {
    "description": "phone_number_of_customer",
    "owner_id": 18293088,
    "owner_resource": "customer",
    "value": "0333103133",
    "value_type": "integer"
  }
}
```

<!-- type: tab-end -->

## Resources
[Metafield reference](https://developer.rechargepayments.com/#metafields)