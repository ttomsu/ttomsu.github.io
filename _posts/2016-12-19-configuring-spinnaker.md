---
title: "Configuring Spinnaker"
date: 2016-12-09
redirect_from: /2016/12/19/configuring-spinnaker/
tags: spinnaker
---

We handle numerous questions every day in the [Spinnaker Slack channel](
http://join.spinnaker.io/) related to configuring Spinnaker.

Each of the core components runs as a Spring Boot application, and it has a
powerful system for composing and overwriting values via YAML files.

# Config Hierarchy
It's important to first understand that there are shared settings that apply to
every service, and some that are specific to an individual service. Shared
settings can be overridden by the individual services.

The common settings are defined in the spinnaker.yml  file (found in
`/opt/spinnaker/config`), and component-specific settings are found in files like
`gate.yml`, `clouddriver.yml`, etc.

Now, it's important that you do not edit these files. They are [source controlled]
(https://github.com/spinnaker/spinnaker/tree/master/config), so the next time
you update Spinnaker, any changes you've made will be clobbered.

Instead, you should create a separate `-local.yml`  file, and replicate the YAML
settings you want to overwrite.

When each of the services come up, they will load the config files in this order
(I'll use the Gate service as the example):

 1. `spinnaker.yml`
 2. `spinnaker-local.yml`
 3. `gate.yml`
 4. `gate-local.yml`

 So lets say there's a snippet of YAML in spinnaker.yml  that you want to change:

```
the:
  special:
    setting: 42
```

You'll first create (or edit) the `spinnaker-local.yml`  file that overwrites this
setting

```
the:
  special:
    setting: 1337
```

Now, all of the component services will have the.special.setting  property with
a value of 1337.

> "But what if I want it changed just for a specific component?"

No problem - just overwrite it in that component's `-local.yml`  file.

You may recognize this as a "last write wins" kind of pattern - `spinnaker.yml`
sets it to a default value, it's overwritten in `spinnaker-local.yml`, and then
overwritten again in a component's `-local.yml`  file.

# Profiles
Spring Profiles are a powerful way to group logical sets of configurations that
can be turned on and off with command line arguments or environmental
properties.

For example, we bake in common settings for Google, Azure, and GitHub OAuth
settings directly into  Gate. Here is a snippet:

```
spring:
  profiles: googleOAuth
  oauth2:
    client:
      # Set these values in your own config file (e.g. gate-googleOAuth.yml).
      # clientId:
      # clientSecret:
      accessTokenUri: https://www.googleapis.com/oauth2/v4/token
      userAuthorizationUri: https://accounts.google.com/o/oauth2/v2/auth
      scope: "profile email"
    resource:
      userInfoUri: https://www.googleapis.com/oauth2/v3/userinfo
    userInfoMapping:
      email: email
      firstName: given_name
      lastName: family_name
```

Notice the name of the profile is `googleOAuth`. The comment suggests writing
`clientId`  and `clientSecret` into a file named `gate-googleOAuth.yml`, which would look
like:

```
spring:
  oauth2:
    client:
      clientId: abcdefgh
      clientSecret: shhhDontTellAnyone
```

That looks strangely familiar to `gate-local`  setup, no? That's because local is
in fact a Spring profile! It's applied automatically by each component at
startup time.

The easiest way to set your active profiles is using the `SPRING_PROFILES_ACTIVE`
environmental variable and restart. Be sure to include the local profile too!

```
export SPRING_PROFILES_ACTIVE=local,googleOAuth
```
