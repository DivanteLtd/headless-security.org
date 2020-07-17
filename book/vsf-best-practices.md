# VSF1 Best Practices

[Vue Storefront 1](https://github.com/DivanteLtd/vue-storefront) become a pretty popular frontend framework for building modern eCommerce sites. The number of features, configuration and deployment options is only growing. With great flexibility there comes responsibility too. Here you can find some Best Practices and Security Standards for building [custom VSF modules](https://docs.vuestorefront.io/guide/cookbook/module.html) and [deploying the sites to production](https://docs.vuestorefront.io/guide/installation/production-setup.html).

## Config file is public

Vue Storefront uses [node-config](https://github.com/lorenwest/node-config) for managing it's [configuration files](https://docs.vuestorefront.io/guide/basics/configuration.html). As powerful this library is - we must remember that VSF works in both CSR/SPA (Client-Side Rendering/Single Page Application) and SSR (Server Side Rendering) modes using **exactly the same codebase**. This means that the `config/*.json` files are **all bundled within Webpack** and sent to the user browser to be accessible from the JavaScript code executed in SPA mode.

This is how your config file looks in the Client's browser (as a part of the `app.*.js` bundle):

```js
(window.webpackJsonp=window.webpackJsonp||[]).push([[6],{0:function(e){e.exports=JSON.parse('{"server":{"host":"0.0.0.0","port":3000,"protocol":"http","api":"api","devServiceWorker":false,"useHtmlMinifier":true,"htmlMinifierOptions":{"minifyJS":true,"minifyCSS":true},"useOutputCacheTagging":false,"useOutputCache":false,"outputCacheDefaultTtl":86400,"availableCacheTags":["attribute","C","category","checkout","compare","error","home","my-account","P","page-not-found","product","taxrule"],"invalidateCacheForwarding":false,"dynamicConfigReload":true,"dynamicConfigContinueOnError":false,"dynamicConfigExclude":["entities","boost","localForage","query","shipping","ssr","storeViews"],"dynamicConfigInclude":[],"elasticCacheQuota":4096,"ssrDisabledFor":{"extensions":["css","eot","gif","ico","jpg","jpeg","js","json","png","raw","svg","tiff","tif","ttf","woff","woff2"]},"trace":{"enabled":false,"config":{}}},"initialResources":[{"filters":["vsf-newsletter-modal","vsf-languages-modal","vsf-layout-empty","vsf-layout-minimal","vsf-order-confirmation","vsf-search-panel"],"type":"script","onload":true,"rel":"prefetch"},{"filters":["vsf-category","vsf-home","vsf-not-found","vsf-error","vsf-product","vsf-cms","vsf-checkout","vsf-compare","vsf-my-account","vsf-static","vsf-reset-password"],"type":"script","onload":true,"rel":"prefetch"}],"staticPages":
```

Moreover, if the `dynamicConfigReload` option is on (and it's enabled by default - part of the config file is included in the `__INITIAL_STATE__` passed from SSR to CSR mode:

```html
  <script>window.__INITIAL_STATE__={version:"",__DEMO_MODE__:!1,config:{elasticsearch:{httpAuth:"",host:"/api/catalog",index:"vue_storefront_magento_1",min_score:.02,csrTimeout:5e3,ssrTimeout:1e3,queryMethod:"GET",disablePersistentQueriesCache:!0,searchScoring:{attributes:{attribute_code:{scoreValues:{attribute_value:{weight:1}}}},fuzziness:2,cutoff_frequency:.01,max_expansions:3,minimum_should_match:"75%",prefix_length:2,boost_mode:"multiply",score_mode:"multiply",max_boost:100,function_min_score:1},searchableAttributes:{name:{boost:4},sku:{boost:2},"category.name":{boost:1}}},api:{url:"https://demo.storefrontcloud.io",saveBandwidthOverCache:!0},graphql:{host:"https://demo.storefrontcloud.io/graphql",port:8080},cart:{thumbnails:{width:150,height:150},serverMergeByDefault:!0,serverSyncCanRemoveLocalItems:!1,serverSyncCanModifyLocalItems:!1,synchronize:!0,synchronize_totals:!0,setCustomProductOptions:!0,setConfigurableProductOptions:!0,askBeforeRemoveProduct:!0,displayItemDiscounts:!0,productsAreReconfigurable:!0,minicartCountType:"quantities",create_endpoint:"https://demo.storefrontcloud.io/api/cart/create?token={{token}}",updateitem_endpoint:"https://demo.storefrontcloud.io/api/cart/update?token={{token}}&cartId={{cartId}}",deleteitem_endpoint:"https://demo.storefrontcloud.io/api/cart/delete?token={{token}}&cartId={{cartId}}",pull_endpoint:"https://demo.storefrontcloud.io/api/cart/pull?token={{token}}&cartId={{cartId}}",totals_endpoint:"https://demo.storefrontcloud.io/api/cart/totals?token={{token}}&cartId={{cartId}}",paymentmethods_endpoint:"https://demo.storefrontcloud.io/api/cart/payment-methods?token={{token}}&cartId={{cartId}}",shippingmethods_endpoint:"https://demo.storefrontcloud.io/api/cart/shipping-methods?token={{token}}&cartId={{cartId}}",shippinginfo_endpoint:"https://demo.storefrontcloud.io/api/cart/shipping-information?token={{token}}&cartId={{cartId}}",collecttotals_endpoint:"https://demo.storefrontcloud.io/api/cart/collect-totals?token={{token}}&cartId={{cartId}}",deletecoupon_endpoint:"https://demo.storefrontcloud.io/api/cart/delete-coupon?token={{token}}&cartId={{cartId}}",applycoupon_endpoint:"https://demo.storefrontcloud.io/api/cart/apply-coupon?token={{token}}&cartId={{cartId}}&coupon={{coupon}}"},
```

### Keep sensitive information out of the config file

**Never put any sensitive information** in the configuration file. Especially please don't put any **shared authorization tokens** neither **private keys** in the config files. As convenient and tempting it might be - it's far better to use the `ENV` variables (accessible only from the [server side modules](https://github.com/DivanteLtd/vue-storefront/blob/master/src/modules/google-cloud-trace/server.ts)), another configuration file which won't be included in the CSR code so Webpack won't bundle it in.

**Example:**
A classical example of a **bad practice** regarding the config file would be a dedicated feature for saving Contact forms into Zendesk. To send the contact request you'll need to have a Zendesk API token and use it along with `fetch()` call when  the user submits the contact form. This request never should be executed directly from the frontend, because sharing Zendesk API token gives much more permissions than only submitting new forms (eg. you can get the history of the requests etc).

Even more extreme case would be sharing the Payment's provider authorization keys (including secret/private keys). These keys are usually required for signing off/validating the payment status changes and never should be exposed out of the `vue-storefront-api` or `storefront-api`. Please make sure the module you're using uses only the public API keys in the `vue-storefront` frontend layer for integration.

### Use `dynamicConfigExclude` properly

This feature let you to [prevent some config properties](https://docs.vuestorefront.io/guide/basics/configuration.html#server) from being transferred within the `__INITIAL_STATE__` - so they won't be visible anymore in the `View Source` mode of your page. However, these properties **are still there in the `app.js` bundle**

### Use the `purge-config` loader

With the latest VSF 1.12.1 release we've added a feature called `purgeConfig` which lets you configure which config properties will be **excluded and not bundled by Webpack** into resulting JS files. Please check the [PR#4540](https://github.com/DivanteLtd/vue-storefront/pull/4540) for details.

It works using the Webpack plugin to filter out the sensitive information from the config files as they're loaded and bundled. However, if you import the config file server side - you can still access all the properties.

Webpack filters are pretty powerful in regards of composing the resulting JS bundles:
```js
      {
        test: /core\/build\/config\.json$/,
        loader: path.resolve('core/build/purge-config.js')
      }
```

### Use the Yireo config validator 

This validation script can help you to make sure your VSF instance is properly configured with Magento, and there are plans to add some security validation checks as well. Check the details at [Yireo Github repository](https://github.com/yireo/vsf-config-validator)

## Use the `search-query` search adapter

From Vue Storefront 1.12 on there is a [new `search-query` search adapter](https://github.com/DivanteLtd/vue-storefront-api/pull/390/files#diff-1c1dccf8e51ee40b3321b647cee837b8R56). The goal for introducing this new adapter was mostly to a) limit the bundle size and remove the business logic of constructing Elastic Search DSL queries in the frontend app, b) limit and control the way people are querying the ElasticSearch by giving them just a subset of operators defined within our own DSL defined in the [`storefront-query-builder`](https://github.com/DivanteLtd/storefront-query-builder). By setting the `config.api` to `search-query` you're virtually removing the ElasticSearch DSL query building in the frontend. 

**Note:** By switching this option in the frontend you're not **disabling** an option of switching the Elastic DSL engine in the `vue-storefront-api`. Please look at [this feature-request](https://github.com/DivanteLtd/vue-storefront-api/issues/487) in order to disable other query engines at the backend.

**Note:** If by some reasons you can't use `search-query` search adapter (eg. you're on VSF 1.10 or so) please look at this PoC of [limiting the Elastic DSL query ranges](https://github.com/DivanteLtd/vue-storefront-api/issues/486).

## Use the API to query authorized data sources

The config file [embeded within `vue-storefront`](https://github.com/DivanteLtd/vue-storefront/tree/master/config) is publicly available to the users. However, the config files used by [`vue-storefront-api`](https://github.com/DivanteLtd/vue-storefront-api/tree/master/config) or [`storefront-api`](https://github.com/DivanteLtd/storefront-api/tree/master/config) **are secret**. They're guaranteed not to be served to the client devices in any form.

For example, the Magento2 secret API keys (OAauth2) are stored in the `vue-storefront-api` configuration files and **all the Magento2 API requests are done ONLY by vue-storefront-api**. 

You shouldn't query any data source that requires a private-key based authorization from the `vue-storefront` application directly. Please always add your custom extension for querying such data sources as the [`vue-storefront-api` extensions](https://docs.vuestorefront.io/guide/archives/extensions.html#extending-the-api) or [`storefront-api` modules](https://docs.storefrontapi.com/guide/modules/tutorial.html). Then please call only these **public proxy** from within the `vue-storefront` application.

There are two exceptions from this general rule:
- if the API key is public (eg. it let the user to query only the public information - for example product catalog),
- if there is a dynamic API key / JWT Token / any other form of token which is acquired by the user from the authorization service based on the credentials provided by the user (eg. password/login). This token of course won't be stored in the config file. This is how the Vue Storefront [passes the user tokens](https://github.com/DivanteLtd/vue-storefront/blob/dfdb9f5264d2ca08aaad7281843cd4eda643ef2b/core/lib/sync/task.ts#L55) for the requests like `order-history`.

## Don't process any user-related information in the SSR

User related data - by default - is never processed in the Vue Storefront SSR rendering mode. SSR output is a subject of [output caching](https://docs.vuestorefront.io/guide/cookbook/checklist.html#_2-ssr-output-cache) and because the application is **session-less by design**. We're not using cookies neither any other form of server session. The Server Side generated output must be always user-agnostic serving only public information that could be indexed by the crawlers and in general being shared between the users.

It's fairly easy to cache the user-related, sensitive information in the output cache by accident - if you processed the user-related info server side. Please don't do this.

The `user` module by default is not supporting the `isServer` option, the whole `MyAccount` section is even redirecting the requests to home page when entered SSR. The authorization module is based on the `localStorage` where the user token is stored after a successful authorization and then passed out to all subsequent `vue-storefront-api` or `storefront-api` requests.

## Always use the HTTP proxy 

Vue Storefront runs (by default) on the port `3000` and the API is exposed on `8080`. Never route production traffic to these ports. Node.js HTTP Servers (used by `vue-storefront`) are not meant to be used on production. By many different reasons - including scalability, throttling, process management and of course security. You always should use HTTP Proxy like `nginx` or `varnish` in front of both: frontend and API services.

Here you've got a short tutorial on [how to properly setup VSF with nginx](https://docs.vuestorefront.io/guide/installation/production-setup.html#production-setup-bare-vps)

## Don't expose Magento frontend 

This advise is especially important for Magento1 which has just ended it life June 2020. Keeping the non-maintained piece of software is never a good idea. If you've got your frontend on Vue Storefront they you probably don't need to expose any other part of Magento to the public internet.

All the requests Vue Storefront is making to the Magento APIs are **always proxied** via `vue-storefront-api` or the `storefront-api`. Only these apps need to contact Magento directly. 
There could be one distinction of this rule when you're using Magento Checkout Fallback - so the whole frontend is on Vue Storefront, but the checkout is still processed by Magento itself. 
In that case you could modify the `.htaccess` file on your Magento instance to prevent requests others than to the `/checkout` URL schema.

If you're using `nginx` as your HTTP Proxy you can do this using the [access module](http://nginx.org/en/docs/http/ngx_http_access_module.html). 

## Change the `invalidateCacheKey`

Ok, this is minor severity advice. If you're using the SSR cache, then please change the [`invalidateCacheKey`](https://docs.vuestorefront.io/guide/basics/ssr-cache.html#dynamic-tags). Otherwise anyone can clear-out your output cache (stored in Redis) by just calling out the `https://yourdomain.com/invalidate?tag=*&key=aeSu7aip`, including the default `invalidateCacheKey`.

## Secure your sandbox/dev env.

Many times you'd like to have a development instance of Vue Storefront exposed publicly so you could share the link with the clients/developers etc. It's a good practice to secure such an instance with a password, VPN etc. Here's the easiest way to run VSF password protected.

We strongly recommend using a kind of HTTP server as the proxy in front of Vue Storefront. Let it be `nginx` (suggested in our [production setup docs](https://docs.vuestorefront.io/guide/installation/production-setup.html)) or `Varnish` or even `Apache`. Any of those HTTP servers allows you to add some authorization or redirects layer before Vue Storefront.

This is a recommended way.

## Consider adding the token-based authorization for catalog requests

All the write/user-sensitive operation are secured with token-based authorization in Vue Storefront. The catalog requests are open (via `vue-storefront-api` that can be used for limiting the data access, filtering etc). It might be worth considering the token-based, anti-csrf kind of authorization for catalog requests as well. It will let you easier control, throttle and limit the API request. That might be pretty usefull when not using the external WAF (Web Application Firewall) and still needing to implement some kind of anti-scrapping protection. I've desribed how it can work in the [`vue-storefront#4624`](https://github.com/DivanteLtd/vue-storefront/issues/4624)

## Add custom Express.js routes to fetch sensitive data

As described above you might want to fetch the data from 3rd party/authorized data sources using `vue-storefront-api` or `storefront-api` custom extensions/modules. There is another way - adding server-middlewares directly within the `vue-storefront` application.

### Express.js middelware

This other option is to create a `Express.js` middleware. Our `core/scripts/server.ts` is a classical Node.js application so it should be easy. To do so you might want to create a [server module](https://github.com/DivanteLtd/vue-storefront/blob/develop/src/modules/compress/server.ts).

Server modules are located in `src/modules` and always have the `server.ts` entry point which responds to one of the few server entry points:

- `afterProcessStarted` - executed just [after the server started](https://github.com/DivanteLtd/vue-storefront/blob/2c6e0e1c8e73952beabf550fe4530344a6bcce15/core/scripts/server.ts#L13).
- `afterApplicationInitialized` - executed just [after Express app got initialized](https://github.com/DivanteLtd/vue-storefront/blob/2c6e0e1c8e73952beabf550fe4530344a6bcce15/core/scripts/server.ts#L34). It's a good entry point to bind new request handlers (`app.get(...)`, `app.use(...)`). Read more on [Express.js request handlers and routing](https://expressjs.com/en/guide/routing.html).
- `beforeOutputRenderedResponse` - executed [after the SSR rendering has been done](https://github.com/DivanteLtd/vue-storefront/blob/2c6e0e1c8e73952beabf550fe4530344a6bcce15/core/scripts/server.ts#L189) but before sending it out to the browser; It lets you override the rendered SSR content with your own.
- `afterOutputRenderedResponse` - executed [after advanced output processing pipeline](https://github.com/DivanteLtd/vue-storefront/blob/2c6e0e1c8e73952beabf550fe4530344a6bcce15/core/scripts/server.ts#L212) executed.
- `beforeCacheInvalidated`, `afterCacheInvalidated` - executed [before and after cache has been invalidated](https://github.com/DivanteLtd/vue-storefront/blob/2c6e0e1c8e73952beabf550fe4530344a6bcce15/core/scripts/server.ts#L76)

Here is an [example how to bind](https://github.com/DivanteLtd/vue-storefront/blob/develop/src/modules/google-cloud-trace/server.ts) tracing module just after server process started:

```js
import { serverHooks } from '@vue-storefront/core/server/hooks'

serverHooks.afterProcessStarted((config) => {
  let trace = require('@google-cloud/trace-agent')
  if (config.has('trace') && config.get('trace.enabled')) {
    trace.start(config.get('trace.config'))
  }
})
```

[Another example](https://github.com/DivanteLtd/vue-storefront/blob/develop/src/modules/compress/server.ts) - pretty common case - binding new Express middleware to process all user requests BEFORE they're processed by SSR rendering pipeline (including custom URL addresses):

```js
import { serverHooks } from '@vue-storefront/core/server/hooks'

const compression = require('compression')
serverHooks.afterApplicationInitialized(({ app, isProd }) => {
  if (isProd) {
    console.log('Output Compression is enabled')
    app.use(compression({ enabled: isProd }))
  }
})
```

If you'd like to bind custom URL address this example can be modified like this:

```js
import { serverHooks } from '@vue-storefront/core/server/hooks'

serverHooks.afterApplicationInitialized(({ app, isProd }) => {
  app.get('/custom-url-address', (req, res) => {
    res.end('Custom response')
  })
})
```
### Advanced output processing

However, by using [advanced output processing](https://docs.vuestorefront.io/guide/core-themes/layouts.html#how-it-works) you can easily generate any text data output from your Vue Storefront site you want. Including JSON, XML and others. It's a way to generate sitemaps and other data-based documents.



