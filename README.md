# xero-node
![npm](https://img.shields.io/npm/v/xero-node?label=xero-node)

## Requirements
node version 10.13.0 or higher

## Release of SDK with oAuth 2 support
Version 4.x of Xero NodeJS SDK only supports oAuth2 authentication and the following API sets.
* accounting

### Bank feeds support in OAuth 2
An early release in a separate package is availalbe [bank feeds API](https://github.com/XeroAPI/xero-node-bankfeeds).

## Looking for OAuth 1.0a support?
[![npm package](https://img.shields.io/badge/npm%20package-3.1.2-blue.svg)](https://www.npmjs.com/package/xero-node/v/3.1.2)

We've moved this code into the [oauth1 branch](https://github.com/XeroAPI/xero-node/tree/oauth1).

## Usage
Installation
This SDK is published as an npm package called xero-node.

```sh
npm install --save xero-node
```

## Getting Started

### Create a Xero App
Follow these steps to create your Xero app

* Create a [free Xero user account](https://www.xero.com/us/signup/api/) (if you don't have one)
* Login to [Xero developer center](https://developer.xero.com/myapps)
* Click "Try oAuth2" link
* Enter your App name, company url, privacy policy url.
* Enter the redirect URI (this is your callback url - localhost, etc)
* Agree to terms and condition and click "Create App".
* Click "Generate a secret" button.
* Copy your client id and client secret and save for use later.
* Click the "Save" button. You secret is now hidden.

## Typescript example
A "kitchen sync" app is available that demonstrates interacting with each endpoint.
Just [download the code](https://github.com/XeroAPI/xero-node-oauth2-app) and configure.

## Javascript Example
This is a barebones example showing how to authenticate and display the
name of the Xero organisation you've connected to.

```js
'use strict';

const express = require('express');
const session = require('express-session');
const  xero_node = require('xero-node')

const client_id = 'YOUR-CLIENT_ID'
const client_secret = 'YOUR-CLIENT_SECRET'
const redirectUri = 'http://localhost:5000/callback'
const scopes = 'openid profile email accounting.transactions accounting.settings offline_access'

const xero = new xero_node.XeroClient({
  clientId: client_id,
  clientSecret: client_secret,
  redirectUris: [redirectUri],
  scopes: scopes.split(" ")
});

let app = express()

app.set('port', (process.env.PORT || 3000))
app.use(express.static(__dirname + '/public'))
app.use(session({
    secret: 'something crazy',
    resave: false,
    saveUninitialized: true,
    cookie: { secure: false }
}));

app.get('/', function(req, res) {
  res.send('<a href="/connect">Connect to Xero</a>');
})

app.get('/connect', async function(req, res) {
  try {
    let consentUrl = await xero.buildConsentUrl();
    res.redirect(consentUrl);
  } catch (err) {
    res.send("Sorry, something went wrong");
  }
})

app.get('/callback', async function(req, res) {
    const url = "http://localhost:5000/" + req.originalUrl;
    await xero.setAccessTokenFromRedirectUri(url);

    // Optional: read user info from the id token
    let tokenClaims = await xero.readIdTokenClaims();
    const accessToken = await xero.readTokenSet();

    req.session.tokenClaims = tokenClaims;
    req.session.accessToken = accessToken;
    req.session.xeroTenantId = xero.tenantIds[0];
    res.redirect('/organisation');
})

app.get('/organisation', async function(req, res) {
  try {
    const response = await xero.accountingApi.getOrganisations(xero.tenantIds[0])
    res.send("Hello, " + response.body.organisations[0].name);
  } catch (err) {
    res.send("Sorry, something went wrong");
  }
})

const PORT = process.env.PORT || 5000;
app.listen(PORT, function() {
  console.log("Your Xero basic public app is running at localhost:" + PORT)
})
```

#### Project Structure
```
src/
  |-  gen/        autogenerated TypeScript
  `-  *.ts        handwritten TypeScript
dist/             compiled JavaScript
package.json
```
