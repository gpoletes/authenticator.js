# The Kloudless Authenticator JS library

The Kloudless Authenticator is a JavaScript library that authenticates users
to cloud services and connects their accounts to your
[Kloudless](https://developers.kloudless.com) app.

The library lets you open a pop-up that allows the user to choose a cloud
service to connect. The pop-up closes once the account has been
successfully connected.

[View a JSFiddle example of the Authenticator in action here.](https://jsfiddle.net/vinodchandru/4f8nsy68/embedded/)

## How it works

The library uses the
[Kloudless OAuth 2.0 Out-of-band flow](https://gist.github.com/vinodc/e73868b42e36bf0166d7)
to obtain an OAuth access token. The token is verified and used to obtain
information on the account that was connected. This data is then returned to
your application via a callback.

See the [Kloudless Docs](https://developers.kloudless.com/docs#authentication)
for other ways to authenticate users.

## Usage

Embedding the Kloudless JavaScript library will expose a global
`Kloudless` object. The JS file is currently hosted on S3 and can be embedded
in your page using this tag:

```html
<script type="text/javascript"
 src="https://static-cdn.kloudless.com/p/platform/sdk/kloudless.authenticator.js"></script>
```

You **must** add the domain of the site you are including the JS file on in your
app's list of [Trusted Domains](https://developers.kloudless.com/applications/*/details#trusted-domains).
e.g. "google.com" or "localhost:8000". Otherwise, your web page cannot receive the
OAuth access token as it is not trusted.

For developers using the older Kloudless Authentication mechanism (Authenticator v0.1),
please see the [Migration Guide](#migration-guide) below on how to migrate to this version.

### authenticator

```javascript
Kloudless.authenticator(element, params, callback);
```

The **authenticator** method sets a click listener on an element to trigger the
Kloudless authentication pop-up. It accepts the following required arguments:

* `element`  
  `element` specifies the element you want to set the listener on. This can be
  a DOM element or a jQuery object that references a single DOM element.

* `params`  
  `params` specifies a map of query parameters to include with the OAuth request.
  At a minimum, this must include your Kloudless application ID as the client ID.
  An error is thrown if this is not provided.

  You may also find it valuable to provide a `scope` that determines which services
  the user can choose from to connect. If only a single service is available, the
  service selection screen is skipped and the user directly proceeds to connecting
  that service. `scope` can either be an Array of different scopes, or a
  space-delimited string. It will be converted into a space-delimited string if
  it is an Array. Refer to the docs for more information on Scopes.

  A full list of parameters supported is available on the
  [OAuth docs](https://developers.kloudless.com/docs/latest/authentication#header-first-leg-1).
  `state`, `response_type` and `redirect_uri` are not required as they will be set
  automatically.

  For example:

      {
        'client_id': 'APP_ID_ABC_123',
        'scope': 'gdrive box dropbox salesforce.crm all:admin'
      }

  Your application's App ID is available on the App Details page in the
  [Developer Portal](https://developers.kloudless.com/applications/*/details).

* `callback`  
  `callback` specifies a function which is passed a `result` object with the
  response to the OAuth 2.0 Out-of-band flow. `result` contains the access token
  obtained via the OAuth flow, as well as the metadata of the
  [account](https://developers.kloudless.com/docs#accounts)
  that was connected:

      {
          'access_token': 'TOKEN123ABC',
          'token_type': 'Bearer',
          'scope': 'box:normal.storage', // Currently, the requested scope is returned
          'state': 'randomstate123',
          'account': {
              'id': 123,
              'service': 'box',
              ...
          }
      }

  **Security Requirement:**
  If you are transferring this information to your backend, be sure to
  [verify](https://gist.github.com/vinodc/e73868b42e36bf0166d7#verify-the-token)
  the token on your backend, especially if you use the Kloudless API Key for
  requests. Otherwise, a malicious user could spoof the account data without
  your application's knowledge.

##### Example Usage

[View a JSFiddle example of the Authenticator in action here.](https://jsfiddle.net/vinodchandru/4f8nsy68/embedded/)

Here is a slightly different example:

```javascript
var e = document.getElementById("auth-button");

// You can also use jQuery:
e = $('#auth-button');

Kloudless.authenticator(e, {
    'client_id': 'oeD8Kzi8oN2uHvBALivVA_3zlo2OhL5Ja6YtfBrtKLA',
}, function (result) {
    if (result.error) {
        console.error('An error occurred:', result.error);
        return;
    }
    console.log('Yay! I now have a newly authenticated', result.account.service,
        'account with ID', result.account.id);
});
```

### stop

```javascript
Kloudless.stop(element);
```
**stop** stops further click events on an element from triggering the
Kloudless authentication pop-up.

##### Example Usage

```javascript
Kloudless.stop($("#auth-button"))
```

## Example apps

Here are some example apps using the authenticator:

* JSFiddle example: https://jsfiddle.net/vinodchandru/4f8nsy68/embedded/
* Kloudless Interactive Docs: https://developers.kloudless.com/interactive-docs
* https://github.com/vinodc/cloud-text-editor

## Migration Guide from older Authentication protocol

Developers can easily migrate to this version from the previous v0.1 Authenticator
library. This library uses the Kloudless OAuth 2.0 authentication flow rather
than the previous authentication mechanism.

Here are the changes needed:

* Replace the previous script tag with the new one. The version has been removed
  from the file name.  
  Previous:  
  `<script src="https://static-cdn.kloudless.com/p/platform/sdk/kloudless.authenticator.v0.1.js"></script>`  
  New:  
  `<script src="https://static-cdn.kloudless.com/p/platform/sdk/kloudless.authenticator.js"></script>`
* The `authenticator()` method now accepts different parameters for `params`.
  See the documentation above for the current format. Here are changes needed:
  * Use `client_id` instead of `app_id`.
  * Use `scopes` instead of `services`. Visit the documentation on Scopes
    to learn more.
  * The `admin` flag has now been incorporated as a part of scopes. 
* The `authenticator()` method now invokes the `callback` provided with a single
  argument that contains the response to the OAuth flow. It would previously
  send two arguments. 
* The `callback` method's argument has a new format. Previously, only the
  Account ID, `id`, and the Service name, `service`, were provided.
  Now, the full response to the OAuth 2.0 flow is provided. Account data is
  retrieved and included in the response, if successful. Otherwise, error
  data is provided. See above documentation for more information on the
  format.
* Add the domain of the web page including the Authenticator library
  to your Kloudless application's list of
  [Trusted Domains](https://developers.kloudless.com/applications/*/details#trusted-domains).
  This allows the `callback` to receive the access token.

## Contributing

### Implementation

The authenticator opens a pop-up with several query parameters indicating what
to display. The following required parameters are present:

* client_id: The ID of the application to connect the account to.
* origin: The host that opened the pop-up. Used to send a message back to the
  caller only.
* redirect_uri: Is `urn:ietf:wg:oauth:2.0:oob`.
* response_type: Is `token`.
* request_id: A random ID for this authenticator instance.

The pop-up is opened from within an iframe located on the API server to get
around cross-domain postMessage restrictions (especially IE).

### Testing
Automated testing is not present. For now, you can manually confirm that it
works by running the test server included in this package.

    $ KLOUDLESS_APP_ID=app_id npm test

where 'app_id' above is a Kloudless App ID specifying which app to connect the
accounts to. You can create an application in the Developer Portal for testing
purposes.

Then navigate to `localhost:3000` and click the button to test if it works.
`window.Kloudless.baseUrl` should be set to a URI that directs to the API server.
An easy way to do this is by just building the file with the correct base URL.

### Building

Build a debug version to enable sourcemaps:

    npm install
    gulp --debug

To build pointing to a custom API server, use the `--url` option:

    gulp --url=http://custom-api-server:8080

The result will be at `build/kloudless.authenticator.min.js`.

### Security Vulnerabilities

If you have discovered a security vulnerability with this library or any other
part of Kloudless, we appreciate your help in disclosing it to us privately by
emailing security@kloudless.com.

## To Do
### Features
* Make it object-oriented.
* Add method to allow user to manually make auth pop-up appear.
* Allow parameters to be configurable without rebinding.
* Allow authenticator to skip account data retrieval for efficiency purposes.

### Testing
* Karma + Jasmine test suite.

### Bugfixes
* Error checking for invalid parameters.
 * A null/undefined element breaks the library.

## Support

Feel free to contact us at support@kloudless.com with any feedback or
questions. Other methods to contact us are listed
[here](https://developers.kloudless.com/docs#getting-help).
