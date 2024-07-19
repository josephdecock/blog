+++
title = 'PAR events in ASP.NET Core'
# date = 2024-07-19
draft = true
series = ['par']
+++

## Event For Extensibility

Add an event that will be invoked before authorization parameters are pushed to facilitate extensibility. Event implementors can

- Tweak parameters before they are pushed
- Control the client authentication process at the PAR endpoint (for example, set a client assertion)
- Take over the entire push to the PAR endpoint, and pass the resulting parameter back to the handler so that it can use it at the authorize endpoint
- Indicate to the handler that PAR should be skipped entirely


The PushedAuthorizationContext passed to the event is similar to the existing context classes used in the other OIDC events. The OpenIdConnectMessage that it contains represents the request that the handler will send to the PAR endpoint. The event can manipulate this message to change the pushed parameters.

If the event wants to control the behavior of the handler more, there are methods on the context to indicate this intent, similar to many of the other contexts (e.g., AuthorizationCodeReceivedContext.HandleCodeRedemption).

To control client authentication, there is HandleClientAuthentication. This causes the handler to not set the client secret.

To skip PAR entirely, there is SkipPush. This indicates that no backchannel call was made in the event, and that the handler should not push either. Instead, the handler will send authorization parameters in the front channel (as if PAR was disabled).

To control pushing to the PAR endpoint, there is HandlePush(string requestUri). This indicates that the event made the backchannel call and received the request_uri as the result. The handler will then use that request_uri when it makes the authorize request.