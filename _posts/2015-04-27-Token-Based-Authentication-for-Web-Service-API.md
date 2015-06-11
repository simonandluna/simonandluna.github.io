---
layout: post
title: Token based Authentication for Web Service API
comments: true
published: true
categories:
author: Gang Wu (Simon)
---

Token provides the best way of handling web service authentication
for multiple users. Any major applications in the web now use tokens
to authorize their API usages, e.g., Facebook, Twitter, Github, and
so on. Token-based authentication have the following benefits:

 - It's stateless and scalable, since all tokens are stored at the client side.
 - It's very secure, since the token is sent on every request, and no cookie is sent.
 - It works for both web and mobile applications.
 - It's independent with platforms and domains, as long as the user has a valid token

## Server-Based Authentication ##

Traditionally, we use a server-based method to authenticate a user for
API requests using a combination of username and password. Since HTTP
protocol is stateless, this means the application will not know whom the
request is on next request. We have to authentize the user again. Thus, user
login information have to be remembered by the server on the request session,
either in the memory or stored in the disk. This causes two major issues:

### Session ###
Whenever, the user is authenticated, the server has to create a record (state) somewhere,
either in memory or in database. This causes overhead for the server when number of
users increase on using application.

### Scalability ###
The server also faces the scalability issue when number of users increase. When servers
start to replicate in the cloud, we have to handle the session information in the memory,
which limit our scale capability.

## Token-Based Authentication ##
Token-based method overcomes major issues of server-based method. Token, a.k.a, access token,
is an object which contains the identity and privileges of the user account associated with the process or thread. When a user logs on, the system verifies the user's password by comparing it with information stored in a security database. If the password is authenticated, the system produces an access token. Every process executed on behalf of this user has a copy of this access token.

The token has to be sent by the client for every request in its HTTP header. Nothing has to
be stored in the server (stateless) since the client has to save it. The token is generated randomly by some hash mechanisms, so it's not a password and hence you won't worry about being
seen or stolen in the client side, even if they are stored in the cookie file. Moreover, the token also expires after some time, e.g., 3600 seconds by default, which makes it more secure.