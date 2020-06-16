# General Best Practices

Here is a list of the general security best practices for designing the SPA'a (Single Page Applications)

## Always use HTTPS

Progressive Web Apps by default must be served from the SSL-certified servers, using HTTPS protocol. It's the only way for making the Service Workers to work. Only via HTTPS we're sure the data passed back and forth between Browser and the APIs are securely encrypted. This is especially important because of passing the customer's / users tokens via HTTP Headers or request parameters.

## Use tokens for API authorization

With SPA's you always need to query some kind of API for a user's data. Authorization is a key feature of any enterprise grade application. If you remember the beginnings of web 2.0 and Web API’s back then, a typical authorization scenario was based on an API key or HTTP authorization. With ease of use came some strings attached. Basically these "static" (API key) and not strongly encrypted (basic auth.) methods were not secure enough.

Here, delegated authorization methods come into action. By delegated, we mean that authorization can be given by an external system / identity provider. One of the first methods of providing such authentication was the OpenID standard (NOTE: http://openid.net/) developed around 2005. It could provide a "One Login" and Single "Sign On" for any user. Unfortunately, it wasn’t widely accepted by identification providers like Google, Facebook or e-mail providers.

The OAuth standard works pretty similarly to OpenID. The authorization provider allows Application Developers to register their own applications with the required data-scope to be obtained in the name of the user. The user authorizes specific applications to use with their account. 

[Read more on JWT](https://microservicesbook.org/ch5-related-techniques.html#json-web-tokens-jwt)

## Set proper CORS headers

When your API's are deployed on the same domain as the frontend - there is no problem fetching the data. Usually though you're in a case when you're making the request to 3rd party service or just a different microservice within your K8s/cloud stack. In that case make sure you've got the CORS headers properly set. CORS headers control [who can request the data (which domains)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) and can be used to limit the scope of these requests.

**Note:** The CORS headers are being controlled by the browser. It's not a valid solution for preventing the data access/requests server-side.
