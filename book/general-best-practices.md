# General Best Practices

Here is a list of the general security best practices for designing the SPA'a (Single Page Applications)

## Always use HTTPS

Progressive Web Apps by default must be served from the SSL-certified servers, using HTTPS protocol. It's the only way for making the Service Workers to work. Only via HTTPS we're sure the data passed back and forth between Browser and the APIs are securely encrypted. This is especially important because of passing the customer's / users tokens via HTTP Headers or request parameters.

## Use tokens for authorization

With SPA's you always need to query some kind of API for a user's data. To prevent the