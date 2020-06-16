# VSF API Best Practices

[Vue Storefront API](https://github.com/DivanteLtd/vue-storefront-api) and its successor the [Storefront API](https://github.com/DivanteLtd/storefront-api) are a data layers designed to limit the contact of your Magento instance (and any other data sources) with outside world. All the API requests Vue Storefront is making in order to get necessary data are proxied by these layers. Please use them wisely to secure your application.


## Always check the data ownership

If you need additional data operations as [`vue-storefront-api` extensions](https://docs.vuestorefront.io/guide/archives/extensions.html#extending-the-api) or [`storefront-api` modules](https://docs.storefrontapi.com/guide/modules/tutorial.html). Please be aware of checking the customer's tokens each time you're returning any data related or including personal information.

For example you might want to support the Product Alerts feature in your VSF site. The feature isn't included out of the box. So naturally, you could add a endpoint to fetch the product alerts (checking if user already subscribed to it):

```
https://example.com/api/ext/product-alert/getList?customerId=100
```

If you use the [`magento2-rest-client`](https://github.com/DivanteLtd/magento2-rest-client) to simply proxy this request to Magento2 API then you'll get the response:

```json
{"code":200,"result":[{"alert_stock_id":"50598","customer_id":"100","customer_email":"test@example.com","product_id":"50533","website_id":"1","add_date":"2020-06-10 13:44:04","send_date":null,"send_count":"0","status":"0","parent_id":null,"store_id":"2","sku":"SOWAC42","parentSku":"SOWACU1416F"}]}
```

By this - innocent as it might look at first glance - feature you're just exposing all the e-mail used for product alerts to anyone who'll just experiment with different `customerId` parameter values.

The right way to handle such requests would be to always pas the `token` parameter along and verify the ownership of the data. Here's [an example how `order-history`  endpoint verifies the ownership of the data](https://github.com/DivanteLtd/magento2-rest-client/blob/29cfac4c4f9982f99b7832621820f93dcf6f9dd4/lib/customers.js#L21)

## Set only the minimal required permissions

The Magento2 API credentials ([oauth2 tokens](https://docs.vuestorefront.io/guide/installation/magento.html#using-native-magento-2-module)) are required for Vue Storefront to work. However in the Magento2 Admin Panel you can set whole range of the  permissions. Please make sure you enable the only the minimal set of features to be accessed via these oauth2 keys which are:

```
Catalog
Sales
My Account
Carts
Stores > Settings > Configuration > Inventory Section
Stores > Taxes
Stores > Attributes > Product
```
![Magento2 API permisssions](https://docs.vuestorefront.io/assets/img/magento_2.08bdbe3f.png)

Please read more about the VSF Data imports in the ["Data Imports Cookbook"](https://docs.vuestorefront.io/guide/cookbook/data-import.html#_2-2-recipe-b-using-on-premise)

## Make sure ElasticSearch and Redis are secured

The services shouldn't be accessible from over the Internet. The network traffic should be limited via system firewall to the access just from the `vue-storefront-api` or `storefront-api` hosts.

## Limit the ElasticSearch indexes accessible via API

In both: `storefront-api` and `vue-storefront-api` config files you can limit the access to only the Elastic indexes that are used for VSF product catalog. Please make sure the [`elasticsearch.indices`](https://github.com/DivanteLtd/vue-storefront-api/blob/29d4ce5998724610f6023275e08dbcc37f802caf/config/default.json#L28) property is set properly.