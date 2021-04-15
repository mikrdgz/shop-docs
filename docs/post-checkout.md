# Create Checkout with Shopify Storefront Cart 

This article demonstrates the most reliable way to generate a ReCharge checkout with Shopify's storefront cart data. This guide will explain how `POST`ing a checkout to ReCharge works, how the client-side JavaScript should work and some debugging tips.

## Who is this guide for?
Existing Shopify stores using the Shopify storefront and ReCharge checkout to handle subscription purchases. This is not applicable for new stores.

**Requirements**

- Existing Shopify customer

- Uses ReCharge for subscription checkouts

## Sending the cart information
To create a cart, you will send a `POST` request to `https://checkout.rechargeapps.com/r/checkout` and receive a `200` OK response. This URL must include a URL parameter containing your Shopify’s store’s [permanent domain](https://shopify.dev/docs/themes/liquid/reference/objects/shop#shop-permanent_domain). Optionally, you can include a cart token parameter, which we’ll use for analytics.

**Full URL Example**
> `https://checkout.rechargeapps.com/r/checkout?domain=MYSHOPIFYDOMAIN&cart_token=SHOPIFYCARTTOKEN`

|**URL parameter**|**Example**|**Required?**|
|-|-|-|
|domain|bare-supplements.myshopify.com|Your “myshopify.com” domain is required|
|`cart_token`|`dabaa155d2680acd088966819ad48b18`|Optional, but useful for analytics|

## What data is required to create a cart?
Below is an example of the cart data we’re required to have to generate a ReCharge checkout. The cart object contains `items`, which is an array of data objects containing `properties`, `quantity`, `variant_id`, and `product_id`.

Each product will store additional details in `properties`, including the required subscription properties to pass unit type and interval frequency.

> **Note**
>
> You can use `order_interval_unit` and `order_interval_frequency` in lieu of the properties `shipping_interval_unit_type` and `shipping_interval_frequency`. These properties are backwards compatible and match our API.


**Cart example**

```json
{
    "cart": {
        "items": [
            {
                "properties": {
                    "shipping_interval_unit_type": "Days",
                    "shipping_interval_frequency": "30"
                },
                "quantity": 1,
                "variant_id": 3844892483,
                "product_id": 1255175363,
            }
        ],
    }
}
```

**Alternate example**

```json
{
    "cart": {
        "items": [
            {
                "properties": {
                    "order_interval_unit": "Days",
                    "order_interval_frequency": "30"
                },
                "quantity": 1,
                "variant_id": 3844892483,
                "product_id": 1255175363,
            }
        ],
    }
}
```

## What other data can I send?

You can also send optional data, including cart `note` and cart `attributes`, which is an object with key-value pairs.

We support additional top-level properties, such as a `discount` code (it must match a code you’ve created in ReCharge).  If you pass a discount code in your request, it will appear pre-filled in the discount field at checkout. 

Customer data can also be included with cart data: `email`, `first_name`, `last_name`, `address1`, `address2`, `city`, `province`, `country`, and `zip`.

**Example cart with additional properties**

```json
{
    "cart": {
        "items": [
            {
                "properties": {
                    "shipping_interval_unit_type": "Days",
                    "shipping_interval_frequency": "30"
                },
                "quantity": 1,
                "variant_id": 3844892483,
                "product_id": 1255175363,
            }
        ],
        "note": "Please gift-wrap my order", # OPTIONAL
        "attributes": {  # OPTIONAL
            "key": "val",
            "key2": "val2"
        }
    },
    "discount": "DISCOUNTCODE10", # OPTIONAL
    "email": "some@email.com", # OPTIONAL
    "first_name": "afirstname", # OPTIONAL
    "last_name": "alastname", # OPTIONAL
    "address1": "123 Street", # OPTIONAL
    "address2": "Apt 301", # OPTIONAL
    "city": "Montreal", # OPTIONAL
    "province": "Quebec", # OPTIONAL
    "country": "Canada", # OPTIONAL
    "zip": "99999-1111" # OPTIONAL
}
```
We accept `note` and `attributes` as standalone keys you can pass outside of the `cart` items array. Passing `note` or `attributes` outside of `cart` will take precedence over `note` or `attribute` inside of the `cart` key.

**Example**

```json
{
    "cart": {
        "items": [
            {
                "properties": {
                    "shipping_interval_unit_type": "Days",
                    "shipping_interval_frequency": "30"
                },
                "quantity": 1,
                "variant_id": 3844892483,
                "product_id": 1255175363,
            }
        ],
    },
    "note": "Please gift-wrap my order", # OPTIONAL
    "attributes": {  # OPTIONAL
        "key": "val",
        "key2": "val2"
    }
}
```

## Updating cart functionality by modifying Shopify theme
The easiest—and recommended—path to updating how your cart functions with ReCharge is to update your existing subscription snippet files on your Shopify Theme. These files are traditionally integrated into your existing store Theme files when you install ReCharge.

### Steps to updating the subscription-cart-footer.liquid file
If your store is already using the `subscription-cart-footer` snippet (your customers land on the `/cart` page), you might be able to simply update your existing file.

1. Download the latest version from the link below
- **Download zip:** [subscription-cart-footer.zip](https://gist.github.com/VizualAbstract/f0a97ce63cc5a83d6f42dfd2a84d1121/archive/d7cea48d671dafe7a54fcd8ab3902889718de444.zip)

- **Available on gist:** [subscription-cart-footer.liquid](https://gist.github.com/VizualAbstract/f0a97ce63cc5a83d6f42dfd2a84d1121#file-subscription-cart-footer-liquid)

2. Sign into your Shopify account and duplicate your published theme

3. Edit the copy of your published theme and search for the snippet file `subscription-cart-footer`

4. Copy the code from the downloaded snippet into the one on your theme

5. Save your changes

### How to use your own JavaScript
You can use your own script code to perform the request client-side. If you prefer, you can leverage some of the code we’ve written and copy and paste these snippets below.

**POST checkout dependency code**
Copy and paste this code somewhere in your theme where it can later be called.

```js
<script>
    function buildCheckoutUrl() {
        // Build the Checkout URL
        var checkout_url = 'https://checkout.rechargeapps.com/r/checkout?',
            url_params = [
                'myshopify_domain=' + Shopify.shop,
            ];
        url_params = url_params
            .concat(get_cart_token())
            .concat(get_ga_linker());
        return checkout_url + url_params.join('&');
    }
    function filterVisible(elems) {
        // Return visible elements
        return elems.filter(function(elem) {
            return !!(elem.offsetWidth || elem.offsetHeight || elem.getClientRects().length);
        });
    }
    function filterValues(elems) {
        // Return inputs with valid values
        return elems.filter(function(elem) {
            var is_active_checkbox = elem.getAttribute('type') === 'radio' && elem.checked,
                is_active_radio = elem.getAttribute('type') === 'checkbox' && elem.checked,
                is_input = elem.getAttribute('type') !== 'checkbox' && elem.getAttribute('type') !== 'radio';
            return !!elem.value && (is_input || is_active_checkbox || is_active_radio);
        });
    }
    function getCartJSON() {
        // Fetch the latest cart data from the /cart.js endpoint
        return fetch('/cart.js')
            .then(function(response) {
                return response.json();
            })
            .then(function(json) {
                return ({ cart: JSON.stringify(json) });
            })
            .catch(function(error) {
                console.error('Error retreiving cart: ', error);
                return ({ cart: {}});
            });
    }
    function getTermsAndConditions() {
        // Find and return concacted terms and conditions values
        var condition_selectors = [
                '#terms',
                '#agree',
            ],
            conditions = document.querySelectorAll(condition_selectors.join(',')),
            conditions_to_update = Array.prototype.slice.apply(conditions),
            conditions_filtered = filterValues(conditions_to_update),
            condition_values = conditions_filtered.map(function(elem) {
                return elem.value;
            });
        if (!condition_values.length) { return {}; }
        return {
            terms_and_conditions: condition_values.join(', ')
        };
    }
    function getNotes() {
        // Find and return concatenated note values if visible
        var note_selectors = [
                '[name="note"]',
            ],
            notes = document.querySelectorAll(note_selectors.join(',')),
            notes_to_update = Array.prototype.slice.apply(notes),
            notes_filtered = filterVisible(notes_to_update),
            note_values = notes_filtered.map(function(elem) { return elem.value; });
        var urlParams = new URLSearchParams(decodeURIComponent(window.location.search)),
            noteFromUrl = urlParams.get('note');
        if (noteFromUrl) {
            note_values.push(noteFromUrl);
        }
        if (!note_values.length) return '';
        var uniqueNoteValues = note_values.filter(function(element, index, array) {
            return array.indexOf(element) === index;
        });
        return { note: uniqueNoteValues.join(', ') };
    }
    function getUTMAttributes() {
        // Retrieve UTMAttributes from Shopify cookie
        var shopifyCookieRegEx = /^_shopify_sa_p/;
        var utmRegEx = /^utm_/;
        var timestampRegEx = /^_shopify_sa_t/;
        var utmParams = {};
        var shopifyCookie = "";
        var timestamp = "";
        document.cookie.split(";")
            .map(function(s){
                return s.trim();
            })
            .forEach(function(s){
                if (shopifyCookieRegEx.test(s)) {
                    shopifyCookie = s;
                }
                if (timestampRegEx.test(s)) {
                    timestamp = decodeURIComponent(s.split("=")[1]);
                }
                return;
            });
        var utmValuesFromCookie = shopifyCookie.split("=")[1];
        decodeURIComponent(utmValuesFromCookie)
            .split("&")
            .forEach(function(s){
                var key = s.split("=")[0];
                var val = s.split("=")[1];
                if (utmRegEx.test(key)) {
                    return utmParams[key] = val;
                }
                return;
            });
        if (!Object.keys(utmParams).length) {
            return undefined;
        }
        utmParams.utm_timestamp = timestamp;
        utmParams.utm_data_source = "shopify_cookie";
        return utmParams;
    }
    function getAttributes() {
        // Find and return cart attributes
        var attributeRegEx = /attributes\[(.*?)\]/,
            attributeNameRegex = /\[(.*?)\]/,
            attribute_selectors = [
                '[name*="attributes"]',
            ],
            attributes = document.querySelectorAll(attribute_selectors.join(',')),
            attributes_to_update = Array.prototype.slice.apply(attributes),
            attributes_filtered = filterValues(attributes_to_update);
        var attribute_data = {},
            utm_attributes = getUTMAttributes(),
            attributes_from_url = new URLSearchParams(decodeURIComponent(window.location.search));
        for (let pair of attributes_from_url) {
            if (attributeRegEx.test(pair[0])) {
                var attributeName = pair[0]
                    .match(attributeNameRegex)[0];
                attributeName = attributeName.substring(1, attributeName.length -1);
                attribute_data[attributeName] = attributes_from_url.get(pair[0]);
            }
        }
        attributes_filtered.forEach(function(attribute) {
            var name = attribute.getAttribute('name'),
                value = attribute.value;
            if (attributeRegEx.test(name)) {
                var extractedName = name.match(attributeNameRegex)[0];
                extractedName = extractedName.substring(1, extractedName.length -1);
                return attribute_data[extractedName] = value;
            }
            return attribute_data[name] = value;
        });
        if (utm_attributes) {
            Object.keys(utm_attributes).forEach(function(key) {
                attribute_data[key] = utm_attributes[key];
            });
        }
        if (!Object.keys(attribute_data).length) return {};
        return { attributes: Object.assign({}, attribute_data) };
    }
    function getCartData() {
        // Return cart attributes and data
        return getCartJSON().then(function(cartDataJSON) {
            var cart_data = {},
                    termsAndConditions = getTermsAndConditions(),
                    notes = getNotes(),
                    attributes = getAttributes(),
                cart_data = cartDataJSON;
            [termsAndConditions, notes, attributes, cart_data].forEach(function(obj) {
                    Object.assign(cart_data, obj);
            });
            return cart_data;
        });
    }
    function get_cart_token() {
        // Build the cart_token param for the URL generator
        try {
            return ['cart_token=' + (document.cookie.match('(^|; )cart=([^;]*)')||0)[2]];
        } catch (e) {
            return [];
        }
    }
    function get_ga_linker() {
        // Build the ga_linker param for the URL generator
        try {
            return [ga.getAll()[0].get('linkerParam')];
        } catch (e) {
            return [];
        }
    }
    function buildInputs(form, obj) {
        // Build input fields for key, value pairs
        Object.keys(obj).forEach(function(key) {
            var input = document.createElement('input');
            input.setAttribute('type', 'hidden');
            input.setAttribute('name', key);
            input.setAttribute('value', typeof(obj[key]) === 'object' ? JSON.stringify(obj[key]) : obj[key]);
            form.appendChild(input);
        });
    }
    function buildForm(data, url) {
        // Build a virtual form
        var form = document.createElement('form');
        form.setAttribute('method', 'post');
        form.setAttribute('action', url);
        form.setAttribute('id', 'rc_form');
        form.style.display = 'none';
        buildInputs(form, data);
        return form;
    }
    function hasSubscription(data) {
        // Check if subscription items are found in cart data
        const cart = JSON.parse(data.cart);
        return [...cart.items].some(item => item.properties && item.properties.shipping_interval_unit_type);
    }
    function ReChargeCartSubmit() {
        // Build and submit cart
        return getCartData().then(function(cart_data) {
            if (!hasSubscription(cart_data)) {
                // Send user to Shopify's checkout if no subscription products are found
                window.location.href = '/checkout';
                return;
            };
            
            var checkout_url = buildCheckoutUrl();
            if (!cart_data) {
                // Cart data not found, fallback to token method
                window.location.href = checkout_url;
                return;
            }
            var xhr = new XMLHttpRequest();
            xhr.open('POST', '/cart/update.js');
            xhr.setRequestHeader('Content-Type', 'application/json');
            xhr.onload = function() {
                if (xhr.status === 200) {
                    window.console.log('done', JSON.parse(xhr.responseText));
                } else if (xhr.status !== 200) {
                    window.console.log('fail', JSON.parse(xhr.responseText));
                }
                var form = buildForm(cart_data, checkout_url);
                document.body.appendChild(form);
                form.submit();
            };
            xhr.send(JSON.stringify(cart_data));
        });
    }
</script>
```

**Available on gist:** [post-checkout-script.html](https://gist.github.com/VizualAbstract/243bf1ca458b52b2ba7000752368b84e#file-post-checkout-script-html)

**Triggering a Checkout POST**

Run this command when you want to perform a `POST` request. The code will check for subscription products and otherwise redirect your customer to the Shopify checkout.

```js
// ReCharge POST Checkout function
ReChargeCartSubmit()
```

### How to verify code works

If you’re worried about the Checkout POST function firing on your store without additional customization, you can quickly verify if it works by running some code in your browser’s JavaScript debugger.

1. Go to your store and add a subscription product to your cart

2. Open up your JavaScript Debugging console and run the following block of code inside:

```js
var checkout_url = "https://checkout.rechargeapps.com"
function buildCheckoutUrl(){var t=["myshopify_domain="+Shopify.shop];return checkout_url+"/r/checkout?"+(t=t.concat(get_cart_token()).concat(get_ga_linker())).join("&")}function filterVisible(t){return t.filter(function(t){return!!(t.offsetWidth||t.offsetHeight||t.getClientRects().length)})}function filterValues(t){return t.filter(function(t){var e="radio"===t.getAttribute("type")&&t.checked,n="checkbox"===t.getAttribute("type")&&t.checked,r="checkbox"!==t.getAttribute("type")&&"radio"!==t.getAttribute("type");return!!t.value&&(r||e||n)})}function getCartJSON(){return fetch("/cart.js").then(function(t){return t.json()}).then(function(t){return{cart:JSON.stringify(t)}}).catch(function(t){return console.error("Error retreiving cart: ",t),{cart:{}}})}function getTermsAndConditions(){var t=document.querySelectorAll(["#terms","#agree"].join(",")),e=filterValues(Array.prototype.slice.apply(t)).map(function(t){return t.value});return e.length?{terms_and_conditions:e.join(", ")}:{}}function getNotes(){var t=document.querySelectorAll(['[name="note"]'].join(",")),e=filterVisible(Array.prototype.slice.apply(t)).map(function(t){return t.value}),n=new URLSearchParams(decodeURIComponent(window.location.search)).get("note");return n&&e.push(n),e.length?{note:e.filter(function(t,e,n){return n.indexOf(t)===e}).join(", ")}:""}function getUTMAttributes(){var t=/^_shopify_sa_p/,e=/^utm_/,n=/^_shopify_sa_t/,r={},o="",i="";document.cookie.split(";").map(function(t){return t.trim()}).forEach(function(e){t.test(e)&&(o=e),n.test(e)&&(i=decodeURIComponent(e.split("=")[1]))});var c=o.split("=")[1];if(decodeURIComponent(c).split("&").forEach(function(t){var n=t.split("=")[0],o=t.split("=")[1];if(e.test(n))return r[n]=o}),Object.keys(r).length)return r.utm_timestamp=i,r.utm_data_source="shopify_cookie",r}function getAttributes(){var t=/attributes\[(.*?)\]/,e=/\[(.*?)\]/,n=document.querySelectorAll(['[name*="attributes"]'].join(",")),r=filterValues(Array.prototype.slice.apply(n)),o={},i=getUTMAttributes(),c=new URLSearchParams(decodeURIComponent(window.location.search));for(let n of c)if(t.test(n[0])){var u=n[0].match(e)[0];u=u.substring(1,u.length-1),o[u]=c.get(n[0])}return r.forEach(function(n){var r=n.getAttribute("name"),i=n.value;if(t.test(r)){var c=r.match(e)[0];return c=c.substring(1,c.length-1),o[c]=i}return o[r]=i}),i&&Object.keys(i).forEach(function(t){o[t]=i[t]}),Object.keys(o).length?{attributes:Object.assign({},o)}:{}}function getCartData(){return getCartJSON().then(function(t){var e={};return[getTermsAndConditions(),getNotes(),getAttributes(),e=t].forEach(function(t){Object.assign(e,t)}),e})}function get_cart_token(){try{return["cart_token="+(document.cookie.match("(^|; )cart=([^;]*)")||0)[2]]}catch(t){return[]}}function get_ga_linker(){try{return[ga.getAll()[0].get("linkerParam")]}catch(t){return[]}}function buildInputs(t,e){Object.keys(e).forEach(function(n){var r=document.createElement("input");r.setAttribute("type","hidden"),r.setAttribute("name",n),r.setAttribute("value","object"==typeof e[n]?JSON.stringify(e[n]):e[n]),t.appendChild(r)})}function buildForm(t,e){var n=document.createElement("form");return n.setAttribute("method","post"),n.setAttribute("action",e),n.setAttribute("id","rc_form"),n.style.display="none",buildInputs(n,t),n}function hasSubscription(t){return[...JSON.parse(t.cart).items].some(t=>t.properties&&t.properties.shipping_interval_unit_type)}function ReChargeCartSubmit(){return getCartData().then(function(t){if(hasSubscription(t)){var e=buildCheckoutUrl();if(t){var n=new XMLHttpRequest;n.open("POST","/cart/update.js"),n.setRequestHeader("Content-Type","application/json"),n.onload=function(){200===n.status?window.console.log("done",JSON.parse(n.responseText)):200!==n.status&&window.console.log("fail",JSON.parse(n.responseText));var r=buildForm(t,e);document.body.appendChild(r),r.submit()},n.send(JSON.stringify(t))}else window.location.href=e}else window.location.href="/checkout"})}
ReChargeCartSubmit();
```

If you have a subscription product in your cart, you’ll be redirected to ReCharge’s checkout, but otherwise be taken to Shopify’s checkout. Use your network tab to confirm the results, ensuring your request to ReCharge is making a `POST` request, and sending your cart data.

> **Need help finding your network tab?**
> - For **Chrome:** [Network Activity Monitor](https://developers.google.com/web/tools/chrome-devtools#network)
> - For **Firefox**: [Network Inspector details](https://developer.mozilla.org/en-US/docs/Tools/Network_Monitor)
> - For **Safari**: [Safari's Web Inspector](https://jun711.github.io/web/how-to-inspect-network-request-and-response-headers-on-safari/)

## How does this script work?

Below is an overview of the steps our script performs before redirecting the user to the correct checkout.

1. Fetch cart data from `/cart.js`
2. Search page for terms and conditions and create a JSON `terms_and_conditions` object with the checked value
- Selector: `#terms`, `#agree`
- Filter out unchecked inputs
3. Search page for notes and create a JSON `note` object
- Selector: `[name="note"]`
4. Search page and URL for attributes and create a JSON `attribute` object with key-value pairs for each attribute.
- Page selector: `[name*="attributes"]`
- URL attributes: Look for URL attributes (e.g. `?attributes[my_attribute]=test`)
- Filter out duplicate attributes

5. Retrieve UTM attributes from Shopify cookies and merge into existing JSON `attribute` object as key-value pairs.
- Only uses cookies prefixed with `utm_` from cookie named `shopify_sa_p`
6. Merge all JSON objects into a single object:

```json
{
  "cart": {
    "token": 'dabaa155d2680acd088966819ad48b18'
    "line_items": [
      {
        "id": 34420101447739,
        "quantity": 1,
        "properties": {
          "shipping_interval_unit_type": "Weeks",
          "shipping_interval_frequency": "1"
        },
      }
    ]
  },
  "terms_and_conditions": "on",
  "note": "Test note",
  "attributes": {
    "my_attribute": "test",
    "utm_source": "emailcampaign",
    "utm_data_source": "shopify_cookie",
    "utm_timestamp": "2021-01-25T15:53:44.767Z"
  }
}
```

7. Search the cart data for subscription properties, looking for `shipping_interval_unit_type`
a. If no subscriptions are found, we’ll redirect the customer to the Shopify checkout page
8. Build checkout URL with base URL, Shopify domain, Cart token (from cookie), and GA UA linker
9. Create a hidden form element
- Use checkout URL for the form
10. Create and a hidden input element for each key-value pair in the merged data object append to form
11. Append the form to the body and submit
- We use a hidden form because we rely on the form’s redirect behavior when submitted
- Using AJAX would mean having to rely on a `200 GET` request to redirect.
 
> **Missing Liquid functionality**
> The script code leaves out a component that was previously done using Shopify Liquid: passing in the customer email and default address of a signed-in customer.
>
> **Tip**: If this is something you want to support, we recommend reviewing the script function  buildForm(data, url), found in the sample code we’ve provided, and using a spread operator to include the information you want in the data object.
>
> **Example**: `data = { ...data, email: 'email@address.com' }`

## Troubleshooting
**ISSUE: No redirect happens**
- You likely had a JavaScript error logged to your script console appear
- The `POST` checkout script has likely encountered a bug or a special edge case that prevents the browser from retrieving cart data
- Navigate to `https://YOURSTORE.com/cart.js` to verify it’s there’s cart data
- Reach out to ReCharge support for help if you have cart data but the redirection still fails

**ISSUE: You were redirected to the wrong checkout**

- If you believe you were taken to the wrong checkout, add a subscription product to your cart and navigate to `https://YOURSTORE.com/cart.js`

- Search for “shipping_interval_unit_type”
    - If found, you should be taken to the ReCharge checkout.
    - If not, then your product is recognized as a one-time purchase and you may have an issue with your product widget.

**ISSUE: Checkout missing customer information or discount**

- You were expecting customer or discount information to be pre-filled into the checkout page that was previously there using the old checkout code.
- Your checkout flow likely had integrations or relies on custom code to modify the request URL to send customer address or discount details to ReCharge.
- You’ll want to review these customizations and integrate them into the POST script. If you’re using our sample code, we recommend making changes to the buildCheckoutUrl() function.

**ISSUE: Missing pre-checkout interaction (e.g., a modal, popup, or upsell)**

- Perhaps you have an upsell addon installed, or you require additional user information, such as a a shipping date or gift wrapping selection.
- If you’re no longer seeing this, you are executing the POST checkout script prematurely. 
- If you’re using our sample code, you’ll need to relocate ReChargeCartSubmit() so it comes later in your code, perhaps after you’ve completed all pre-checkout interactions., or upselling more information

