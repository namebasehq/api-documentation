# Auth for the Namebase API

To obtain an API key, visit the settings page on https://namebase.io/pro and follow the instructions in the user interface.

**Be careful with your API keys: they grant unrestricted access to your exchange account.**

## Authenticating to the API

The Namebase API expects HTTP Basic Auth with a valid pair of Namebase API keys. Use the `ACCESS_KEY` as the username and the `SECRET_KEY` as the password. To authorize your requests, send an HTTP request to one of the API endpoints with the following `Authorization` header:

```javascript
Authorization: Basic /* base64-encoded string of ACCESS_KEY:SECRET_KEY */
```

Here is a code sample of API key authentication using Node.js and the `node-fetch` package:

```javascript
const credentials = Buffer.from(`${ACCESS_KEY}:${SECRET_KEY}`);
const encodedCredentials = credentials.toString('base64');
const authorization = `Basic ${encodedCredentials}`;
fetch(/*ENDPOINT*/, {
  method: 'GET',
  headers: {
    Authorization: authorization,
    Accept: 'application/json',
    'Content-Type': 'application/json',
  },
});
```

Note: for Node.js, you may want to use the officially supported Node.js client library linked to in the documentation homepage.
