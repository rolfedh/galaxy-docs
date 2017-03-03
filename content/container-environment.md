---
title: Container environment
order: 14
description: Learn in what environment Galaxy runs your app
---

<!-- Note: post logger2 release, we'll also document the environment variables
     that we set, entry point, etc.
-->

Galaxy runs your app in a set of containers using the [Docker](https://www.docker.com/) container platform.

The actual code that is run is determined by the "base image" --- either the [default base image](/base-image-packages.html) or a [custom base image](/custom-base-images.html).

Galaxy runs your app on 64-bit Linux machines in the UTC time zone.

When Galaxy wants to shut down your container (because you have deployed a new version or because you are scaling down), it first sends the `SIGTERM` signal to your container, waits 30 seconds, and then sends the `SIGKILL` signal.  If you'd like to do some cleanup work before your container dies, you can catch the `SIGTERM` signal (eg, with [`process.on('SIGTERM')`](https://nodejs.org/api/process.html#process_signal_events) in Node).  The `SIGKILL` signal cannot be caught --- your container immediately dies.

<h2 id="env-vars">Environment variables</h2>

Galaxy runs your app with the following environment variables set:

| Environment variable | Default value | meaning |
| -------------------- | ------------- | ------- |
| `GALAXY_APP_ID` | App identifier like `Ss8o4Y6KrvFBKKkqM` | Internal identifier for your app. |
| `GALAXY_APP_VERSION_ID` | Integer like `123` | Your app's version. |
| `GALAXY_CONTAINER_ID` | Container ID including app ID, like `Ss8o4Y6KrvFBKKkqM-3n28` | Internal identifier for this container. |
| `GALAXY_LOGGER` | `system` | For historical reasons, indicates to your container that log collection is handled by Galaxy. |
| `HTTP_FORWARDED_COUNT` | `1` | Tells your app that it is running behind a proxy. |
| `KADIRA_OPTIONS_HOSTNAME` | Short container ID, like `3n28` | Used to configure the third-party Kadira service. |
| `METEOR_SETTINGS` | A JSON object, set [when you deploy your app](/deploy-guide.html#settings-create) | Available as [`Meteor.settings`](https://docs.meteor.com/api/core.html#Meteor-settings). Galaxy may add fields to your settings object to enable features such as [prerender](/seo.html), and may reformat the JSON object. |
| `PORT` | Integer like `3000` | [Port on which your app should listen](#network-incoming). |
| `ROOT_URL` | Your app's default hostname, prefixed with `http://` or `https://` depending on whether you use Force HTTPS | Used by [`Meteor.absoluteUrl()`](https://docs.meteor.com/api/core.html#Meteor-absoluteUrl) to generate links. |

You can add your own environment variables and override any of these except for the container-specific ones (`GALAXY_CONTAINER_ID` and `KADIRA_OPTIONS_HOSTNAME`) [via your settings.json file](/environment-variables.html).

<h2 id="network">Network environment</h2>

<h3 id="network-incoming">Incoming connections</h3>

Galaxy runs your app containers in a firewalled network environment.  Only one port is exposed for external connections.  Galaxy tells your app what port to listen on via the `$PORT` environment variable, which contains a number such as `3000`.

Galaxy forwards HTTP connections (port 80) on your app's configured [domains](/custom-domains.html) to the port exposed for external connections. HTTPS connections (port 443) are also forwarded to this port if you've [configured encryption](/encryption.html).  You cannot serve connections on any other ports.

<h3 id="network-outgoing">Outgoing connections and IP whitelisting</h3>

When your app connects to other services like your database, those services' connections will always appear to come from one of a fixed set of IP addresses.  These IP addresses are not the IP addresses of the individual machines your container runs on, so don't be surprised if they don't match.  (These addresses are also distinct from the addresses that our "ingress" DNS address points to --- don't point your DNS at these IP addresses!)

Some services can be configured to only allow access from a list of IP addresses. For an extra layer of security, you can use these IP addresses in a "whitelist" on that service. Whitelisting is especially common for databases, and may be required by your database provider.

Note that whitelisted IP addresses are shared between all Galaxy customers, so you should still control access to your services by other means. Whitelisting is meant to protect your app from non-targeted attacks.

Add these IP addresses to your whitelist:

- For the us-east-1 cluster: 34.197.187.203, 34.197.229.75, 34.197.156.92, 34.197.222.74
- For the eu-west-1 cluster: 34.248.186.245, 34.248.14.239, 34.248.124.59

If your software wants you to specify your whitelist as a list of [CIDRs](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) rather than a list of IP addresses, just add the three characters `/32` to the end of each IP address.
