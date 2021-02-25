# Authentication

## How to make a request
Making an API request to the ReCharge API requires these steps:

1. ReCharge requires an API key for requests to the server. We only accept requests over HTTPS. To obtain API access, **[contact our support team](https://support.rechargepayments.com/hc/en-us/requests/new)**.

2. Add the following header to your API client. You can also make a request to our API by setting up your credentials and headers in [Postman](https://www.postman.com/product/api-client/).

> ### Header
>`X-Recharge-Access-Token: *YOUR_API_TOKEN*`

3. Make a `GET` request to `https://api.rechargeapps.com` with your token populated in the header to receive general data about your ReCharge integration.

## Example requests

<!--
type: tab
title: cURL
-->

### Request

```curl
curl -i -H 'X-Recharge-Access-Token: your_api_token' \
-X GET https://api.rechargeapps.com
```

### Response 

```json
{
    "store": {
        "country": "US",
        "currency": "USD",
        "email": "example_mail@gmail.com",
        "iana_timezone": "America/New_York",
        "province": "California",
        "shop_owner": "Mike Flynn",
        "store_id": 112,
        "store_name": "example_store",
        "store_timezone": "(GMT-08:00) America/Los_Angeles",
        "store_url": "example_store.myshopify.com"
    }
}
```
<!--
type: tab
title: Python
-->

### Request

```python
import requests

headers = {"X-Recharge-Access-Token": "your_api_token"}
url = "https://api.rechargeapps.com"

result = requests.get(url, headers=headers)
```

### Response

```json
{
    "store": {
        "country": "US",
        "currency": "USD",
        "email": "example_mail@gmail.com",
        "iana_timezone": "America/New_York",
        "province": "California",
        "shop_owner": "Mike Flynn",
        "store_id": 112,
        "store_name": "example_store",
        "store_timezone": "(GMT-08:00) America/Los_Angeles",
        "store_url": "example_store.myshopify.com"
    }
}
```


<!--
type: tab
title: PHP
-->

### Request

```php
<?php

$request = new HttpRequest();
$request->setUrl('https://api.rechargeapps.com/');
$request->setMethod(HTTP_METH_GET);

$request->setHeaders(array(
  'content-type' => 'application/json',
  'x-recharge-access-token' => 'your_api_token'
));

try {
  $response = $request->send();

  echo $response->getBody();
} catch (HttpException $ex) {
  echo $ex;
}
```

### Response

```json
{
    "store": {
        "country": "US",
        "currency": "USD",
        "email": "example_mail@gmail.com",
        "iana_timezone": "America/New_York",
        "province": "California",
        "shop_owner": "Mike Flynn",
        "store_id": 112,
        "store_name": "example_store",
        "store_timezone": "(GMT-08:00) America/Los_Angeles",
        "store_url": "example_store.myshopify.com"
    }
}
```

<!--
type: tab
title: Ruby
-->

### Request

```ruby
require 'uri'
require 'net/http'
require 'openssl'


url = URI("https://api.rechargeapps.com/")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true

request = Net::HTTP::Get.new(url)
request["x-recharge-access-token"] = 'your_api_token'
request["content-type"] = 'application/json'


response = http.request(request)
puts response.read_body
```

### Response
```json
{
    "store": {
        "country": "US",
        "currency": "USD",
        "email": "example_mail@gmail.com",
        "iana_timezone": "America/New_York",
        "province": "California",
        "shop_owner": "Mike Flynn",
        "store_id": 112,
        "store_name": "example_store",
        "store_timezone": "(GMT-08:00) America/Los_Angeles",
        "store_url": "example_store.myshopify.com"
    }
}
```

<!-- type: tab-end -->

