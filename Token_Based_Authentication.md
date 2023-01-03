## Authenication

# Basics. Authentication and Authorization

# 🔐 Authentication —

this is the process of checking user credentials (login/password). Authentication of the user by comparing the login / password entered by him with the saved data during registration.

# 🕵🏻‍♂️ Authorization —

is a check of the user’s rights to access certain resources.

For example, after authentication, the user Parth (with email parth@palminfotech.com) gets the right to access and receive some data from the “api/getAppUsers” resource. When user Parth accesses the “dashboard” resource, the authorization system will check whether the user has the right to access this resource.

## Steps

### Steps of Authentication:

1. User Denis log in with email parth@palminfotech.com and password;
2. Backend checks that user with this email registered and checks what role the user has;
3. Backend generate Token for that user with this role;
4. User visit some protected resource, or make request to a server;

### Steps of Authorization:

5. The server checks the rights (role) of the user in the token and passes or rejects the request accordingly;

![User Authentication and Authorization sequence diagram.](/media/Authentication_Authorization_Sequence_diagram.webp "User Authentication and Authorization sequence diagram")

### What is JSON Web Token?

JSON Web Token (JWT) — is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed.

Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes and resources that are permitted with that token.

JSON Web Tokens contains three blocks separated by dots:

- Header
- Payload
- Signature

The first two blocks are in JSON format and additionally encoded in Base64 format.

JWT typically looks like the following:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJpZFVzZXIiOjM5LCJVc2VyVHlwZSI6IkFkbWluIiwiaWREZXZpY2UiOjEyLCJTZXJpYWwiOiJzdHJpbmciLCJpYXQiOjE2NzI2MzUzNTMsImV4cCI6MTY3NTIyNzM1MywiYXVkIjoibG9jYWxob3N0IiwiaXNzIjoibG9jYWxob3N0In0.
C2yGnizZ26JJvgbQ5G5WqSuQNLLcMm5XMq9gIf04Dlg
```

To verify detail you can paster any JWT token to following site.

# https://jwt.io/

# Header

The Header commonly consist of two properties: the type of the token, which is JWT, and the signing algorithm being used.
For example:

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

The header encoded with Base64.

# Payload

The second part of the token is the payload, contains key-value fields, moreover, the JWT standard defines several reserved names (iss, aud, exp, and others)

There are three types of claims: registered, public, and private claim.

- Registered claims— these are a set of predefined claims which are not mandatory but recommended, to provide a set of useful. Some of them are: iss (issuer), exp (expiration time), sub (subject), aud (audience), and others.
- Public claims — these can be defined at will by those using JWTs.
- Private claims— these are the custom data created to share information between parties.

```
{
  "idUser": 39,
  "UserType": "Admin",
  "idDevice": 12,
  "Serial": "string",
  "iat": 1672635353,
  "exp": 1675227353,
  "aud": "localhost",
  "iss": "localhost"
}
```

The payload encoded with Base64.

# Signature

The signature can be generated using both symmetric encryption algorithms and asymmetric ones. In addition, there is a separate standard that unsubscribes the format of an encrypted JWT token.

The signature is used to verify the message wasn’t changed along the way, and, in the case of tokens signed with a private key, it can also verify that the sender of the JWT is who it says it is.

JSON Web Tokens mainly used for authorization on server. JWT sends with every request from client to server.

Tokens (and the signature of the token) are generated on the server based on the secret key, which is stored on the server, and payloads.

The token is eventually stored on the client and used when it is necessary to authorize any request. This solution is great for SPA development.

## Auth JSON Web Token types

# Access token

Access token is used to authorize requests and store additional information about the user (aka user_id, user_roles and etc, this information is also called payload). All fields in payload are a free set of fields needed to implement your private business logic. That is, user_id and user_roles are not mandatory, but are part of the business logic.

To make the application more secure, you need to store the access token in the memory of the client application, not in LocalStorage or any other place where a hacker can easily steal our token.

## Refresh token

Refresh token issued by the server upon successful authentication and used to obtain a new pair of access and/or refresh tokens. We store it exclusively in an httpOnly cookie.

If the server returns a refresh token along with an access token, you can try storing the refresh token in an insecure storage like LocalStorage. This may lead to the application being hacked.
Imagine that a hacker has stolen an access token, but after some time the access token will expire and if the hacker also has a refresh token, he will try to refresh the tokens and get a new pair of access and refresh tokens.
But this doesn’t happen if the refresh token is stored in an HttpOnly Cookie because the information in the HttpOnly Cookie is only available on the server and JS can’t get the information from this type of Cookie.

Since we are trying to protect our application as much as possible from hacking, we must store our refresh token exclusively in an HttpOnly Cookie.

Each token has its own lifetime, for example access: 30 min, refresh: 14 days.

This means that after 30 minutes the access token will expire and the client will try to renew the token, but if the user does not interact with our application within 14 days, the user will be automatically logged out.

Since tokens are not encrypted information, it is highly recommended not to store any sensitive data (passwords, payment details, etc.) as part of the payload.

# Why you must store refresh tokens in database?

The refresh token is stored on the server to account for access and invalidate stolen tokens. This way the server knows exactly which clients it should trust.

If you don’t store the refresh token in the database, then there is a good chance that the tokens will move uncontrollably between hackers.

# How to set Refresh token in from mobile app

In order for the RefreshToken to be successfully set and sent by the mobile app, the addresses of the authentication endpoints (/api/auth/login, /api/auth/refresh, /api/auth/logout) must be located in the site’s domain space. For example,

## Login with JWT (Authentication)

1. The user logs in to the application, passing the login and password;
2. The server will check the authenticity of the login/password;
3. If successful, creates and writes a session to the database;
4. Creates an Access token;
5. Sends to the client an Access and Refresh token;
6. The client saves tokens (access in the application memory, refresh is networked as a cookie automatically).

Note:
When adding a session to a table in the database, it is worth checking how many refresh sessions the user has in total, and if there are too many of them or the user connects simultaneously from several domains, it is worth taking action. You can check that the user has a maximum of 5 simultaneous refresh sessions, and when trying to install the next one, delete the previous ones. All other checks are up to you, depending on the task.

![User Authentication with JWT sequence diagram](/media/User_Authentication_with_JWT_sequence_diagram.webp "User Authentication with JWT sequence diagram")

## Refresh Access token

To use the authentication feature on more than one device, you need to store all the refresh tokens for each user. During each login, a record is created with IP and other meta information, the so-called Refresh Session.

1. The client (frontend) checks before the request whether the Access token TTL (lifetime has expired);
2. If the Access token has expired, the client makes a request for POST auth/refresh-tokens and the Refresh token in the Cookie;
3. The server verify that Refresh token are not expired;
4. Throws a error if it fails;
5. If successful, creates a new refresh session and writes it to the database;
6. Creates an Access token;
7. Sends an Access and Refresh token (Refresh token in the Cookie)to the client (taken from the refresh session created above).
8. Client save Access token in the memory of the application;
9. Make request with new Access token;
10. The server checks the user’s rights (Authorization);
11. If the user has the necessary rights, then he gets the result, otherwise he gets an error, for example, 403 status code;

![Refresh session sequence diagram](/media/Refresh_session_sequence_diagram.webp "Refresh session sequence diagram")

For example, I drew a sequence diagram of the Refresh token flow (the diagram does not cover the case when the refresh-session ends with an error, for example, due to an expired Refresh token):

# Tips:

1. At the time of the refresh both tokens are updated.
2. Refresh token has TTL, so that in case the user is offline for more than 14 days (or your custom time), when you have to re-login via login and password.
