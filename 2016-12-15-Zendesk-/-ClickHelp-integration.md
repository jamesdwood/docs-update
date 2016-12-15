---
title: Zendesk / ClickHelp integration
layout: post
author: james.wood
permalink: /zendesk-/-clickhelp-integration/
source-id: 1TglA-C4ms7brsjqK5XZCMb2ycA8t4vKhqAXko5ek9QQ
published: true
---
[[TOC]]

# **Enabling QProtocol data collection on site**

To collect QProtocol data, the following tasks must be completed:

1. [Include the ](#heading=h.7fxuxdo4uzo1)**[UV AP**I](#heading=h.7fxuxdo4uzo1)[ script](#heading=h.7fxuxdo4uzo1)

2. [Include the ](#heading=h.5w897uljmlw)**[smartserve.j**s](#heading=h.5w897uljmlw)[ script](#heading=h.5w897uljmlw)

3. [Emitting events](#heading=h.wqk23q1wuymg)

## These represent our minimum implementation requirements.

## **Including the UV API script**

Implement on every page:

<script>!**function**(){**function** **n**(){**function** **n**(n){p.level=n}**function** **e**(n,e){p.info(n,"event emitted"),e=c(e||{}),e.meta=e.meta||{},e.meta.type=n,u.push(e),r(),v.listeners=f(v.listeners,**function**(n){**return**!n.disposed})}**function** **o**(n,e,o){**function** **r**(){**return** p.info("Replaying events"),t(**function**(){s(v.events,**function**(t){c.disposed||l(n,t.meta.type)&&e.call(o,t)})}),f}**function** **i**(){**return** p.info("Disposing event handler"),c.disposed=!0,f}p.info("Attaching event handler for",n);**var** c={type:n,callback:e,disposed:!1,context:o||window};v.listeners.push(c);**var** f={replay:r,dispose:i};**return** f}**function** **t**(n){p.info("Calling event handlers"),a++;**try**{n()}**catch**(n){p.error("UV API Error",n.stack)}a--,r()}**function** **r**(){**if**(0===u.length&&p.info("No more events to process"),u.length>0&&a>0&&p.info("Event will be processed later"),u.length>0&&0===a){p.info("Processing event");**var** n=u.shift();v.events.push(n),t(**function**(){s(v.listeners,**function**(e){**if**(!e.disposed&&l(e.type,n.meta.type))**try**{e.callback.call(e.context,n)}**catch**(n){p.error("Error emitting UV event",n.stack)}})})}}**function** **i**(n,e,o){**var** t=v.on(n,**function**(){e.apply(o||window,arguments),t.dispose()});**return** t}**function** **s**(n,e){**for**(**var** o=n.length,t=0;tn.INFO||console&&console.info&&console.info.apply(console,arguments)},error:**function**(){p.level>n.ERROR||console&&console.error&&console.error.apply(console,arguments)}};n.ALL=0,n.INFO=1,n.ERROR=2,n.OFF=3,n(n.ERROR);**var** v={on:o,emit:e,once:i,events:[],listeners:[],logLevel:n};**return** v}"object"==**typeof** module&&module.exports?module.exports=n:window&&(window.uv=n())}();</script>

The UV API should be implemented as an inline script on every page of your website.  We recommend adding the script as early in the head as possible.  It must be loaded before the **_smartserve.js_** script and any script that emits an event.

This code executes in a fraction of a millisecond (1 microsecond) and sets up an API that acts as a conduit for events.  Any script can emit events using the API, even before **_smartserve.js_** loads and those events can be read and subscribed to by any script using the API.

For more information on the UV API,  see [https://github.com/QubitProducts/uv-api](https://github.com/QubitProducts/uv-api).  Using uv.emit to emit events is explained below in the section [Emitting ](#heading=h.wqk23q1wuymg)[events](#heading=h.wqk23q1wuymg).

# **Including the smartserve.js script**

**Include on every page:**

<script src="https://dd6zx4ibq538k.cloudfront.net/smartserve-{your-property-id}.js" async></script>

The smartserve script is an asynchronous javascript bundle hosted on a cdn.  This script contains all the code that Qubit uses to serve experiences, collect data, and compute segments.  We regularly republish this script to include new experiences and segments or make modifications to existing experiences and segments.

It is best practice and recommended to put this in the page header.  It is also recommended to use an async script tag so that the script is non-blocking.

Placing the tag early on each page will lead to better data accuracy and will allow experiences to run faster.  Anything that causes the script to run late in the page load, such as being placed in a tag manager or at the end of the document, can cause low data capturing rates. Unfortunately, there is little that can be done to capture data if visitors leave the page before the script has been loaded.

# **Emitting events**

By emitting structured events using the UV API, our scripts are able to consume information about the page and the visitor in a reliable way.  This information can be used to decide whether to show an experience, to write content within an experience, or to move a visitor into or out of a segment.  Every event emitted will be sent to our backend where it is slotted into a table depending on its type.

**Note**: Before any event is sent to the backend,** ****_smartserve.js_** will add context information stored in cookies and meta information such as the URL of the page. This data will be available to query.

## **Implementation Overview**

Events are emitted using the UV API and must conform to a certain schema.  Events are emitted using javascript, usually on page load or when a new view is rendered on a single page app.  An example is shown below.

uv.emit('theEventType', {  aFieldFromTheSchema: 'theValueWhichMustBeTheCorrectType',  aNumberField: 3,  aBooleanField: false})       

Below is a breakdown of a website by page type, alongside the events that could be emitted on each page.  This guide represents a typical set up and there may be exceptions.

<table>
  <tr>
    <td>Page</td>
    <td> Available events</td>
  </tr>
  <tr>
    <td>All</td>
    <td>ecView, ecUser, ecBasketItem, ecBasketSummary</td>
  </tr>
  <tr>
    <td>Home</td>
    <td>ecView, ecUser, ecBasketItem, ecBasketSummary</td>
  </tr>
  <tr>
    <td>Product</td>
    <td>ecView, ecUser, ecProduct, ecBasketItem, ecBasketSummary</td>
  </tr>
  <tr>
    <td>Listing</td>
    <td>ecView, ecUser, ecProduct, ecBasketItem, ecBasketSummary, ecFilterCriteria, </td>
  </tr>
  <tr>
    <td>Basket</td>
    <td>ecView, ecUser, ecBasketItem, ecBasketSummary</td>
  </tr>
  <tr>
    <td>Checkout</td>
    <td>ecView, ecUser, ecBasketItem, ecBasketSummary</td>
  </tr>
  <tr>
    <td>Transaction</td>
    <td>ecView, ecUser, ecBasketItemTransaction, ecBasketTransactionSummary
</td>
  </tr>
</table>


There are also some events that are emitted later than the page load or view render, usually as a result of a visitor's action.  These events are usually implemented as part of an advanced implementation.  See [Emitting events in an advanced implementation](#heading=h.mptuo84gxcpb)*.*

### **Field types**

The following table identifies the valid field types for Qubit's schema. If the JS type for a field is wrong then the entire event will be rejected from the pipeline.

<table>
  <tr>
    <td>Field Type</td>
    <td>JS Type</td>
    <td>Notes</td>
  </tr>
  <tr>
    <td>String</td>
    <td>String</td>
    <td>Up to 256 characters long</td>
  </tr>
  <tr>
    <td>String array</td>
    <td>Array</td>
    <td></td>
  </tr>
  <tr>
    <td>Boolean</td>
    <td>Boolean</td>
    <td></td>
  </tr>
  <tr>
    <td>Integer</td>
    <td>Number</td>
    <td>Fractions will be floored e.g. 2.78 -> 2</td>
  </tr>
  <tr>
    <td>Float</td>
    <td>Number</td>
    <td></td>
  </tr>
  <tr>
    <td>Money</td>
    <td>Number</td>
    <td>Must be rounded to 2 decimal places otherwise the event will be invalid</td>
  </tr>
  <tr>
    <td>Currency</td>
    <td>String</td>
    <td>Must be a 3 letter ISO 4217 currency code</td>
  </tr>
  <tr>
    <td>Email</td>
    <td>String</td>
    <td>Must be a valid email and up to 256 characters long</td>
  </tr>
  <tr>
    <td>Date</td>
    <td>String</td>
    <td>Dash separated date YYYY-MM-DD e.g. 1988-12-15</td>
  </tr>
  <tr>
    <td>Price</td>
    <td>Object</td>
    <td>An object containing 'value' and a ‘currency’</td>
  </tr>
  <tr>
    <td>EpochTimeMS</td>
    <td>Number</td>
    <td>A UNIX timestamp in milliseconds</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>


### **View events**

The ecView event is a special event because it is required by **_smartserve.js_** for data collection and processing.  It is therefore essential that every page emits an ecView event and that it is emitted before any other event.  

#### Recommended minimum implementation

Emit ecView on every page, with the type field populated.

uv.emit('ecView', {  type: 'home'})

#### Schema for ecView       

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>type</td>
    <td>Type: String
Description: Can be either home, category, product, checkout, transaction, help, contact or other
Example: 'category'</td>
  </tr>
  <tr>
    <td>subtypes</td>
    <td>Type: String Array
Description: An unordered list of subtypes to describe the view
Example: [ 'Womens', ‘Dresses’, ‘Cocktail Dresses’]</td>
  </tr>
  <tr>
    <td>environment</td>
    <td>Type: String
Description: Reports the environment i.e. development, staging or production
Example: ‘production’</td>
  </tr>
</table>


### **User events**

ecUser events are emitted once per view and report the visitor details.  The event should be emitted on every page as long as there is data available for the user.  For many websites, the event will be present and well populated when the visitor logs on. Websites with a newsletter subscription form that the visitor has previously submitted, might be able to populate the name and email address for the visitor.

#### Recommended minimum implementation

Emit ecUser events on every page with the username, id, firstName , lastName , email and language once the user is logged in.

uv.emit('ecUser', {  user: {    email: ['bob.builder@gmail.com](mailto:%27bob.builder@gmail.com)',    id: 'bb123xyz',    username: 'bobbuilder',    firstName: 'bob',    lastName: 'builder',    language: 'en-gb'   }})

On multi-language sites, always emit an ecUser event on every page with at least the language populated.

uv.emit('ecUser', {  user: {    language: 'en-us'   }})

#### Schema for ecUser

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>user.firstName</td>
    <td>Type: String
Description: The visitor's first name
Example: 'John'</td>
  </tr>
  <tr>
    <td>user.lastname</td>
    <td>Type: String
Description: The visitor’s last name
Example: ’Smith’</td>
  </tr>
  <tr>
    <td>user.username</td>
    <td>Type: String
Description: The visitor’s username
Example: 'JohnSmith’</td>
  </tr>
  <tr>
    <td>user.email</td>
    <td>Type: Email
Description: The visitor’s primary email address
Example: ‘john@johnsmith.com’</td>
  </tr>
  <tr>
    <td>user.language</td>
    <td>Type: String
Description: An IETF compatible string.
Example: ‘en-us’</td>
  </tr>
  <tr>
    <td>user.id</td>
    <td>Type: String
Description:  The visitor’s Id 
Example: ‘bb123xyz’</td>
  </tr>
</table>


## **Product events**

Product events report a product loaded on a page. This could be, for example, a main product, a linked product on a product detail page, a product in a listing page, or search page.  The eventType field differentiates between the different scenarios and can be listing, detail, linked_product or other.

#### Recommended minimum implementation

Emit ecProduct events on product detail pages with eventType set as detail.  Include the product sku, productId, name and price.

uv.emit('ecProduct', {  eventType: 'detail',  product: {    sku: 'sso-099',    productId: '1209012233',    name: 'Nike shoes',    stock: 20,        price: {      currency: 'GBP',      value: 34    },    url: '[http://www.fashionunion.com/dresses/red-cocktail-dress.html](http://www.fashionunion.com/dresses/red-cocktail-dress.html)',    description: 'This red cocktail dress is perfect for any occasion.',    category: [      'Dresses',      'Cocktail Dresses',      'Red'    ],    images: [      'http://www.fashionunion.com/dresses/red-cocktail-dress-1.jpg',      'http://www.fashionunion.com/dresses/red-cocktail-dress-2.jpg'    ]  }})

Emit ecProduct events on product listing pages for each product shown on the page.  If the page is paginated, only emit ecProduct events for the products currently shown to the visitor. Include eventTypeset as listing and the product sku, productId, name and price.

uv.emit('ecProduct', {  eventType: 'listing',  product: {    sku: 'sso-099',    productId: '1209012233',    name: 'Nike shoes',    price: {      currency: 'GBP',      value: 34    }  }})uv.emit('ecProduct', {  eventType: 'listing',  product: {    sku: 'sso-154',    productId: '120039504',    name: 'Blue sweater',    price: {      currency: 'GBP',      value: 66    }  }}) 

#### Schema for ecProduct      

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>product.sku</td>
    <td>Type: String
Description: SKU (stock keeping unit), uniquely identifying
the product
Example: 'DRESS101'</td>
  </tr>
  <tr>
    <td>
product.productId</td>
    <td>
Type: String
Description: The product ID that is the same for different
styles and sizes
Example: ‘1234’</td>
  </tr>
  <tr>
    <td>product.name</td>
    <td>Type: String
Description: The product name
Example: ‘Red Cocktail Dress’</td>
  </tr>
  <tr>
    <td>product.price.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>product.price.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
  <tr>
    <td>product.stock</td>
    <td>Type: Integer
Description: The number of available units 
Example: 20</td>
  </tr>
  <tr>
    <td>product.url</td>
    <td>Type: String
Description: The URL of the product’s page
Example: ‘http://www.fashionunion.com/dresses/red-cocktail-dress.html’</td>
  </tr>
  <tr>
    <td>
product.description</td>
    <td>

Type: String
Description: The product’s description
Example: ‘This red cocktail dress is perfect for any occasion’</td>
  </tr>
  <tr>
    <td>product.category</td>
    <td>Type: String Array
Description:  The product’s categories
Example: [‘dresses’, ‘cocktail dresses’, ‘red’]</td>
  </tr>
  <tr>
    <td>product.images</td>
    <td>Type: String Array
Description: The product’s images
Example: ['http://www.fashionunion.com/dresses/red-cocktail-dress-1.jpg','http://www.fashionunion.com/dresses/red-cocktail-dress-2.jpg’]</td>
  </tr>
  <tr>
    <td>eventType</td>
    <td>Type: String
Description: The type of product event, e.g. 'listing', 'detail', 
or 'linked_product'
Example: ‘detail’</td>
  </tr>
</table>


## **Basket events**

Basket events should be emitted on every page as long as there are one or more items in the visitor's basket.  They should not be emitted on an order summary page.  There are two types of basket events that are emitted, ecBasketItem and ecBasketSummary.

Individual items are emitted as ecBasketItem events, each of which contains the summary for the full basket.  This denormalisation is essential for query performance.

After emitting one or more ecBasketItem events, a ecBasketSummary event should be emitted.  If item level detail is not known, it is acceptable to emit just an ecBasketSummary event without any ecBasketItemevents.

### **Recommended minimum implementation**

Emit ecBasketItem events for each item in the basket on all pages when the visitor has one or more basket items.  Events should include the basket subtotal and total, the product sku, productId, name and price, a quantity which should be 1, if there are not more than one of the products in the basket, and a subtotal, which should be the same as the product price unless there is more than one.

**var** basketSummary = {  subtotal: {    currency: 'GBP',    value: 86  },  total: {    currency: 'GBP',    value: 86  }}uv.emit('ecBasketItem', {  basket: basketSummary,  product: {    sku: 'sso-099',    productId: '1209012233',    name: 'Nike shoes',    stock: '20',        price: {      currency: 'GBP',      value: 34    }  },    url:'http://www.fashionunion.com/dresses/red-cocktail-dress.html',    description: 'This red cocktail dress is perfect for any occasion.',    category: [      'Dresses',      'Cocktail Dresses',      'Red'    ],    images: [      'http://www.fashionunion.com/dresses/red-cocktail-dress-1.jpg',      'http://www.fashionunion.com/dresses/red-cocktail-dress-2.jpg'    ],  quantity: 1,  subtotal: {    currency: 'GBP',    value: 34  }})     

Emit an ecBasketSummary event on all pages when the visitor has one or more basket items.  The event should include the basket subtotal and total.

uv.emit('ecBasketSummary', {  basket: basketSummary})

#### Schema for ecBasketItem

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>basket.subtotal.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>basket.subtotal.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: 'USD'</td>
  </tr>
  <tr>
    <td>
basket.total.value</td>
    <td>

Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>basket.total.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
  <tr>
    <td>product.sku</td>
    <td>Type: String
Description: SKU (stock keeping unit), uniquely identifying
the product
Example: ‘DRESS101’</td>
  </tr>
  <tr>
    <td>product.productId</td>
    <td>Type: String
Description: The product ID that is the same for
different styles and sizes
Example: ‘1234’</td>
  </tr>
  <tr>
    <td>product.name</td>
    <td>Type: String
Description: The product name
Example: ‘Red Cocktail Dress’</td>
  </tr>
  <tr>
    <td>product.stock</td>
    <td>Type: Integer
Description: The number of available units 
Example: 20</td>
  </tr>
  <tr>
    <td>product.price.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>product.price.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
  <tr>
    <td>product.url</td>
    <td>Type: String
Description: The URL of the product’s page
Example: ‘http://www.fashionunion.com/dresses/red-cocktail-dress.html’</td>
  </tr>
  <tr>
    <td>product.description</td>
    <td>Type: String
Description: The product’s description
Example: ‘This red cocktail dress is perfect for any occasion’</td>
  </tr>
  <tr>
    <td>product.category</td>
    <td>Type: String Array
Description:  The product’s categories
Example: [‘dresses’, ‘cocktail dresses’, ‘red’]</td>
  </tr>
  <tr>
    <td>


product.images</td>
    <td>




Type: String Array
Description: The product’s images
Example: ['http://www.fashionunion.com/dresses/red-cocktail-dress-1.jpg','http://www.fashionunion.com/dresses/red-cocktail-dress-2.jpg’]</td>
  </tr>
  <tr>
    <td>quantity</td>
    <td>Type: Integer
Description: The number of products described by the 
line item
Example: 2</td>
  </tr>
  <tr>
    <td>subtotal.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>subtotal.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
</table>


#### Schema for ecBasketSummary

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>basket.subtotal.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>basket.subtotal.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: 'USD'</td>
  </tr>
  <tr>
    <td>basket.total.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>basket.total.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
</table>


## **Transaction events**

Transaction events are very important because they report revenue for your website. They are implemented in a similar way to basket events except they should only be implemented on an order summary page and they should include the transaction.id.

### **Recommended minimum implementation**

Emit ecBasketItemTransaction events for each item in the basket on the order summary page. Events should include the transaction id, the basket subtotal and total, the product sku, productId, name and price, a quantity, which should be 1 if there are not more than one of the products in the basket, and a subtotal which should be the same as the product price unless there is more than one.

**var** transactionSummary = {  id: 'aa-44345-hsdsd'}**var** basketSummary = {  subtotal: {    currency: 'GBP',    value: 86  },  total: {    currency: 'GBP',    value: 86  }}uv.emit('ecBasketItemTransaction', {  transaction: transactionSummary,  basket: basketSummary,  product: {    sku: 'sso-099',    productId: '1209012233',    name: 'Nike shoes',    stock: '20',    price: {      currency: 'GBP',      value: 34    }  },    url: 'http://www.fashionunion.com/dresses/red-cocktail-dress.html',    description: 'This red cocktail dress is perfect for any occasion.',    category: [      'Dresses',      'Cocktail Dresses',      'Red'    ],    images: [      'http://www.fashionunion.com/dresses/red-cocktail-dress-1.jpg',      'http://www.fashionunion.com/dresses/red-cocktail-dress-2.jpg'    ],  quantity: 1,  subtotal: {    currency: 'GBP',    value: 34  }})Emit an ecBasketTransactionSummary event on the order summary page after the ecBasketItemTransaction events. The event should include the transaction id and the basket subtotal and total.

uv.emit('ecBasketTransactionSummary', {  transaction: transactionSummary,  basket: basketSummary})

#### Schema for ecBasketItemTransaction

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>transaction.id</td>
    <td>Type: String
Description: An ID unique to the transaction
Example: '83748372'</td>
  </tr>
  <tr>
    <td>basket.subtotal.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>basket.subtotal.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
  <tr>
    <td>basket.total.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>basket.total.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
  <tr>
    <td>product.sku</td>
    <td>Type: String
Description: SKU (stock keeping unit), uniquely identifying
the product
Example: ‘DRESS101’</td>
  </tr>
  <tr>
    <td>product.productId</td>
    <td>Type: String
Description: The product ID that is the same for
different styles and sizes
Example: ‘1234’</td>
  </tr>
  <tr>
    <td>product.name</td>
    <td>Type: String
Description: The product name
Example: ‘Red Cocktail Dress’</td>
  </tr>
  <tr>
    <td>product.stock</td>
    <td>Type: Integer
Description: The number of available units 
Example: 20</td>
  </tr>
  <tr>
    <td>product.price.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>product.price.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
  <tr>
    <td>product.url</td>
    <td>Type: String
Description: The URL of the product’s page
Example: ‘http://www.fashionunion.com/dresses/red-cocktail-dress.html’</td>
  </tr>
  <tr>
    <td>product.description</td>
    <td>Type: String
Description: The product’s description
Example: ‘This red cocktail dress is perfect for any occasion’</td>
  </tr>
  <tr>
    <td>product.category</td>
    <td>Type: String Array
Description:  The product’s categories
Example: [‘dresses’, ‘cocktail dresses’, ‘red’]</td>
  </tr>
  <tr>
    <td>product.images</td>
    <td>Type: String Array
Description: The product’s images
Example: ['http://www.fashionunion.com/dresses/red-cocktail-dress-1.jpg','http://www.fashionunion.com/dresses/red-cocktail-dress-2.jpg’]</td>
  </tr>
  <tr>
    <td>quantity</td>
    <td>Type: Integer
Description: The number of products described by the 
line item
Example: 2</td>
  </tr>
  <tr>
    <td>subtotal.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>subtotal.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
</table>


#### Schema for ecBasketTransactionSummary

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>transactionId</td>
    <td>Type: String
Description: An ID unique to the transaction
Example: '83748372'</td>
  </tr>
  <tr>
    <td>basket.subtotal.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>basket.subtotal.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
  <tr>
    <td>basket.total.value</td>
    <td>Type: Money
Description: The price value
Example: 9.99</td>
  </tr>
  <tr>
    <td>basket.total.currency</td>
    <td>Type: Currency
Description: The ISO 4217 currency e.g. GBP, USD
Example: ‘USD’</td>
  </tr>
</table>


# **Emitting events in an advanced implementation**

There are additional events that could be emitted in an advanced Qubit implementation.

* [ecInteraction](#heading=h.ubnfpsrzs3kv)

* [ecUserLogin](#heading=h.6sgxhsyvwdy5)

* [ecUserSignup](#heading=h.7t1nsghxjv9o)

* ecVoucher

* [ecStoreLocator](#heading=h.wendwtipb7ph)

* [ecRecommendation](#heading=h.1opa2rgyou)

* [ecProductRecommendation](#heading=h.b4zg77opvknw)

* [ecFormSubmission](#heading=h.uov1p849s5j4)

#### Schema for ecInteraction

Emit ecInteraction events to track visitor interaction with page elements, such as a home button.

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>type</td>
    <td>Type: String
Description: The interaction type 
Example: 'click', ’view’, ’hover’, ‘download’, ‘share’</td>
  </tr>
  <tr>
    <td>name</td>
    <td>Type: String
Description: A unique name to identify the page element
Example: ‘TopNavHomeButton’’</td>
  </tr>
</table>


#### Schema for ecUserLogin

Emit to track visitor logins.

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>user</td>
    <td>Type: String
Description: Data for the currently logged in visitor
Example: 'id': ‘2861102bace6e6620948564f0ce0a7cd’,’title’: ’Mr’, ‘firstName’: ‘John’, ‘lastName’: ‘Smith’</td>
  </tr>
  <tr>
    <td>firstLogin</td>
    <td>Type: Boolean
Description: Set to true if it is the first time a visitor has logged in
Example: true</td>
  </tr>
</table>


#### Schema for ecUserSignup

Emit to track when a visitors sign up for a newsletter or event, for example.

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>user</td>
    <td>Type: String
Description: Data for the visitor that signs up fora newsletter or event
Example:  'id': ‘2861102bace6e6620948564f0ce0a7cd’,’title’: ’Mr’, ‘firstName’: ‘John’, ‘lastName’: ‘Smith’</td>
  </tr>
  <tr>
    <td>type</td>
    <td>Type: String
Description: What the visitor has signed up for
Example: ‘newsletter’</td>
  </tr>
  <tr>
    <td>name</td>
    <td>Type: String
Description: A unique name to identify what the visitor has signed up for
Example: ‘loyaltyaccount’</td>
  </tr>
</table>


#### Schema for ecVoucher

Emit to track when a visitor enters a voucher code.

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>entry</td>
    <td>Type: String
Description: The voucher code entered by the visitor
Example: 12321</td>
  </tr>
  <tr>
    <td>entrySuccess</td>
    <td>Type: Boolean
Description: Whether or not the voucher was successfully applied to the transaction
Example: true</td>
  </tr>
  <tr>
    <td>voucher.id</td>
    <td>Type: String
Description: A unique voucher Id. The ID should be the same for all visitors that are using the voucher/promotion
Example: '873748372'</td>
  </tr>
  <tr>
    <td>voucher.label</td>
    <td>Type: String
Description: The name of the voucher/promotion
Example: ‘SummerPromo’</td>
  </tr>
  <tr>
    <td>
transaction</td>
    <td>
Type: String
Description: The transaction that the voucher was applied to
Example: ‘id’: 83748372, ‘firstTransaction’: true, ‘paymentType’: ‘paypal’, ‘parentId’: 83748371</td>
  </tr>
  <tr>
    <td>basket</td>
    <td>Type: String
Description: The basket the voucher applies to, including,unique basket Id, total basket cost, and number of items inThe basket
Example: ‘id’: ‘BASK123’, ‘total’: 29.99, ‘quantity’: 3 </td>
  </tr>
</table>


#### Schema for ecStoreLocator

Emit to track when and how a visitor queries a store locator.

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>type</td>
    <td>Type: String
Description: The query method used to locate a store
Example: 'zipcode'</td>
  </tr>
  <tr>
    <td>query</td>
    <td>Type: String
Description: The query value entered against the type
Example: ‘10010’</td>
  </tr>
  <tr>
    <td>store</td>
    <td>Type: String
Description: Data for the store returned by the query
Example: ‘id’: ‘12424’, ‘locality’: ‘London’, ‘country’: ‘UnitedKingdom’</td>
  </tr>
</table>


#### Schema for ecRecommendation

Emit to track when a product recommendation is made to a visitor.

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>id</td>
    <td>Type: String
Description: The segment id that the visitor is a member of that prompted the recommendation
Example: 'lookalike'</td>
  </tr>
  <tr>
    <td>skus</td>
    <td>Type: String Array
Description: The skus of the recommended products
Example: [‘DRESS101’,  ‘DRESS102’, ‘DRESS501’]</td>
  </tr>
</table>


#### Schema for ecProductRecommendation

Emit to track when a related item product recommendation is made to a visitor.

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>rank</td>
    <td>Type: Float
Description: A rank between 0 and 1 (0 being the first shown / highest ranking recommendation)
Example: 0.8</td>
  </tr>
</table>


#### Schema for ecFormSubmission

Emit to track when a visitor submits a form.

<table>
  <tr>
    <td>Field name</td>
    <td>Reference</td>
  </tr>
  <tr>
    <td>submissionId</td>
    <td>Type: String
Description: A unique Id to identify the form
Example: 'Form001'</td>
  </tr>
  <tr>
    <td>name</td>
    <td>Type: String
Description: A unique name to identify the form
Example: ‘ContactUsEnquiry’</td>
  </tr>
</table>


