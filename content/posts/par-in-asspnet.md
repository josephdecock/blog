+++
title = 'Support for PAR in ASP.NET Core 9.0'
date = 2024-07-19
draft = false
series = ['par']
+++

I'm really excited to share that ASP.NET Core 9.0 will include support for
Pushed Authorization Requests (PAR). We've been discussing the new API in [this
issue](https://github.com/dotnet/aspnetcore/issues/51686), and [my
PR](https://github.com/dotnet/aspnetcore/pull/55069) merged just a few hours ago
as of this writing. You'll be able to try it out in preview 7 of ASP.NET Core
9.0.

## Thank You
This is my first API that I've contributed to ASP.NET Core, and I couldn't have
done it without the help and support of some great folks.

- Thanks to Brock Allen and Dominick Baier for supporting open source,
  sponsoring my time to do this development through [Duende
  Software](https://www.duendesoftware.com), connecting me with the folks at
  Microsoft, and being my mentors in all things OAuth and OIDC.
- Special thanks to Stephen Halter from the ASP.NET team for being the champion
  of this new API and providing code reviews and guidance throughout the
  process.
- Also thanks to everyone who contributed to the design discussions, including
  Barry Dorrans, Chris Ross, and Andrew Casey.

## What is PAR?
PAR is a relatively new [OAuth
standard](https://datatracker.ietf.org/doc/html/rfc9126) that improves the
security of OAuth and OIDC flows by moving authorization parameters from the
front channel to the back channel (that is, from redirect URLs in the browser to
direct machine to machine http calls on the back end).

This prevents an attacker in the browser from 
- seeing authorization parameters (which could leak PII) and from 
- tampering with those parameters (e.g., the attacker could change the scope of
  access being requested). 
 
Pushing the authorization parameters also keeps request URLs short. Authorize
parameters might get very long when using more complex OAuth and OIDC features
such as Rich Authorization Requests, and URLs that are long cause issues in many
browsers and networking infrastructure. 

The use of PAR is encouraged by the [FAPI working
group](https://openid.net/wg/fapi/) within the OpenID Foundation. For example,
[the FAPI2.0 Security
Profile](https://openid.bitbucket.io/fapi/fapi-2_0-security-profile.html)
requires the use of PAR. This security profile is used by many of the groups
working on open banking (primarily in Europe), in health care, and in other
industries with high security requirements.

## How to use PAR in ASP.NET Core
If you're using the OpenId Connect handler and your OpenId Connect Provider (OP)
supports it, the handler will use PAR by default. You can also explicitly
disable or require PAR with the new option `PushedAuthorizationBehavior`:

```cs
builder.Services.AddAuthentication()
  .AddOpenIdConnect(opt =>
  {
    // Other allowed values are UseIfAvailable (the default) and Require
    opt.PushedAuthorizationBehavior = PushedAuthorizationBehavior.Disable; 
  });
```

If you set the option to Require, that's you telling the handler that you must
have PAR and if the OP doesn't support it you want the attempt to fail. And if
you set the option to Disable, it does exactly what it says on the tin.

Again, a huge thank you to everyone. Tune in next week for a deep dive on the
customization hooks that allow you to write custom code to
- Tweak parameters before they are pushed
- Control the client authentication process at the PAR endpoint (for example, set a client assertion)
- Take over the entire push to the PAR endpoint, and pass the resulting parameter back to the handler so that it can use it at the authorize endpoint
- Indicate to the handler that PAR should be skipped entirely
