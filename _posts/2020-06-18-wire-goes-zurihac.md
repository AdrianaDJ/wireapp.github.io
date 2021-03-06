---
layout: post
title:  "Wire goes ZuriHac"
author: wire
categories: [haskell, events]
image: assets/images/posts/wire-zurihac-article-featured.jpg
feature: false
---

[Wire](wire.com) went to [ZuriHac](https://zfoh.ch/zurihac2020/) last week-end!  With 500 attendants last year, ZuriHac is probably the biggest event for Haskell enthusiasts, academics, and programmers world-wide, and not so small compared to software engineering events in general.  

It is a hybrid between hackathon and conference: people form groups to work on many popular software projects here, but if you prefer, you can spend most of your day in talks and tutorials and making friends with people in the hallway.

This year people didn't meet on a [University Campus](https://www.hsr.ch/) at the Southern shore of Lake Zurich in Switzerland, but on Discord and on YouTube.

That was a new experience with many possibilities to discover, but
also some challenges.

Live-chatting during the talk was super useful, allowing to clarify questions and discuss the talk between participants in real-time. At the same time, it could also distract from the actual presentation, but luckily one could just rewind the talk a bit when necessary and then increase the playback speed a bit to catch up again.

Similarly, while in the previous years it had only been possible to follow one or two conversations at a time (because everything happened in different rooms), this time it was much easier to have a look at many of them. Due to the incredible amount of interesting projects and discussions, it was really tempting to try following too many things at once.

In the end, it was brilliantly organised and a big success. Sad that there was no dip in the lake in the afternoon, and no barbecue at night, but talks and hacking and networking worked almost as well. Better in weird ways: easier to focus, easier to hunt down people you know must be around here somewhere.

The Wire team worked on a couple of things.

**Arian** worked on [haskell-fido2](http://github.com/arianvp/haskell-fido2). A haskell library for Webauthn, a new standard for browser authentication that replaces passwords and/or 2FA codes with unphishable hardware keys. People can use the TPM in their laptops and phones, or a dedicated hardware
key like [Nitrokey](https://nitrokey.com) and [Yubikey](https://www.yubico.com) to authenticate against websites. This library makes it possible to easily add this modern authentication scheme to haskell web services.

We hope to use this library in the future to bring two-factor and/or passwordless authentication to Wire, which has been a highly requested feature.

The library is still very much work in progress. But basic registration and authentication is now working. Special thanks to Laurens, Ruud and Berdario for joining this project during the hackathon! We found the Webauthn standard extremely complex and convoluted, and at times also underspecified. This is a bit worrying for a security-senstitive standard. Arian is planning on writing a separate blog post on the project relatively soon, detailing what what he thinks of the Webauthn specification in its current form and how he thinks it can be potentially be improved.

**@fisx** worked on [Servant](https://github.com/haskell-servant/servant), a library for generating rest API servers (mock or with hand-written, type-checked handlers), application-specific client libraries in many languages, Swagger docs, and some more that are correct by construction. One of the more widely discussed issues is that Servant's type-level machinery could only model one type of response (eg., `200 found`, but not any one of `200 found`, `400 bad request`, `404 not found`).  

The user always had to pick one of the possible responses as the default that is returned, and return all others by throwing exceptions, without the type-level correctness that makes Servant useful.

Now, you can define multiple status codes and return types right in the API specification. For example, we can have an API that describes redirects, missing users, and the existence of a user right in th type-level.

```haskell
type Redirect = WithStatus 301 (ResponseHeaders '[Header "Location" URL] NoContent)
type API =
    "users" :> Capture "uid" Int :> Get '[JSON]
      '[WithStatus 200 User,
        WithStatus 401 UnauthenticatedError,
        WithStatus 301 Redirect
       ]

redirect :: IsMember Redirect xs => URL -> f (Union xs)
redirect x = respond (addHeader NoContent x)

getUser :: Server API -- ~ Int -> Handler (Union '[WithStatus 200 User, WithStatus 404 NotFoundMsg, Redirect])
getUser uid = do
  session <- getSession
  if uid == 0 then redirect "http://other-website.com/users/0"
  else
  case session of
    Unauthenticated -> respond @404 (UnauthenticatedError "please log in")
    _ -> do
      user <- DB.getUserById uid
      case user of
        Nothing -> respond @404 (NotFoundMsg uid "not found")
        Just user -> respond @200 user

main = Warp.run 8080 (serve (Proxy API) getUser)
```

We have thought about this [for a while](https://github.com/wireapp/servant-uverb) now, and during ZuriHac have turned these ideas into a [pull request](https://github.com/haskell-servant/servant/pull/1314) that we want to release soon.  

There will be a longer blog post about this once it's released.  Thanks **@Taneb** for spending most of your weekend on this with @fisx! :)