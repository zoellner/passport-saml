Node SAML
=============
# Deprecation Notice
Please use `@node-saml/node-saml` going forward. The npm package `node-saml` is now marked as deprecated and won't receive any updates.

NPM versions up to 3.x are based on this repo, the 4.x beta releases are already based on https://github.com/node-saml/node-saml. See that repo for a change log. Please note that there are breaking changes from v3 to v4 (i.e. from the code in this repo to the new repo in the @node-saml organiation).

## Old Readme
This is a [SAML 2.0](http://en.wikipedia.org/wiki/SAML_2.0) library based on the SAML implementation of [passport-saml](https://github.com/bergie/passport-saml).

v1 is a backward compatible drop-in replacement for the saml.js library from passport-saml.

v2 modernizes (async instead of callbacks) and restructures the library to be independend of some of the assumptions made for passport.js (e.g. callback format, express.js request objects)

## Installation

    $ npm install node-saml

## Usage

  ```javascript
    const {SAML} = require('node-saml');

    const options = {};
    const saml = new SAML(options);

    // await saml.getLogoutResponseUrl({user, samlLogoutRequest}, {additionalParams});
    // const {success} = await saml.validateRedirect(query, originalQuery);
    // await saml.validatePostResponse(body);
    // await saml.validatePostRequest(body);
    // await saml.getAuthorizeForm();
    // await saml.getAuthorizeUrl(options);
    // await saml.getLogoutUrl(user, options);
    // saml.generateServiceProviderMetadata(decryptionCert, signingCert);
  ```

## Upgrade from v1
### Breaking Changes
- All functions that previously required a callback do now return a Promise, e.g.

```javascript
  // old
  saml.validatePostResponse(body, (err, result) => {});

  // new
  const result = await saml.validatePostResponse(body);
```

- Instead of passing in a full express.js request object, functions now only require an object with the parameters that are actually relevant to that function

e.g.
```javascript
  // old
  saml.getLogoutResponseUrl(req, {additionalParams}, callback)

  // new
  await saml.getLogoutResponseUrl({user, samlLogoutRequest}, {additionalParams});
```

- Errors thrown are now proper Javascript Errors. It was a mix of js Errors and strings before.


#### Config parameter details:

 * **Core**
  * `callbackUrl`: full callbackUrl (overrides path/protocol if supplied)
  * `path`: path to callback; will be combined with protocol and server host information to construct callback url if `callbackUrl` is not specified (default: `/saml/consume`)
  * `protocol`: protocol for callback; will be combined with path and server host information to construct callback url if `callbackUrl` is not specified (default: `http://`)
  * `host`: host for callback; will be combined with path and protocol to construct callback url if `callbackUrl` is not specified (default: `localhost`)
  * `entryPoint`: identity provider entrypoint (is required to be spec-compliant when the request is signed)
  * `issuer`: issuer string to supply to identity provider
  * `audience`: expected saml response Audience (if not provided, Audience won't be verified)
  * `cert`: the IDP's public signing certificate used to validate the signatures of the incoming SAML Responses, see [Security and signatures](#security-and-signatures)
  * `privateCert`: see [Security and signatures](#security-and-signatures)
  * `decryptionPvk`: optional private key that will be used to attempt to decrypt any encrypted assertions that are received
  * `signatureAlgorithm`: optionally set the signature algorithm for signing requests, valid values are 'sha1', 'sha256' (default), or 'sha512'
  * `digestAlgorithm`: optionally set the digest algorithm used to provide a digest for the signed data object, valid values are 'sha1', 'sha256' (default), or 'sha512'
  * `xmlSignatureTransforms`: optionally set an array of signature transforms to be used in HTTP-POST signatures. By default this is `[ 'http://www.w3.org/2000/09/xmldsig#enveloped-signature', 'http://www.w3.org/2001/10/xml-exc-c14n#' ]`
 * **Additional SAML behaviors**
  * `additionalParams`: dictionary of additional query params to add to all requests; if an object with this key is passed to `authenticate`, the dictionary of additional query params will be appended to those present on the returned URL, overriding any specified by initialization options' additional parameters (`additionalParams`, `additionalAuthorizeParams`, and `additionalLogoutParams`)
  * `additionalAuthorizeParams`: dictionary of additional query params to add to 'authorize' requests
  * `identifierFormat`: if truthy, name identifier format to request from identity provider (default: `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`)
  * `acceptedClockSkewMs`: Time in milliseconds of skew that is acceptable between client and server when checking `OnBefore` and `NotOnOrAfter` assertion condition validity timestamps. Setting to `-1` will disable checking these conditions entirely. Default is `0`.
  * `attributeConsumingServiceIndex`: optional `AttributeConsumingServiceIndex` attribute to add to AuthnRequest to instruct the IDP which attribute set to attach to the response ([link](http://blog.aniljohn.com/2014/01/data-minimization-front-channel-saml-attribute-requests.html))
  * `disableRequestedAuthnContext`: if truthy, do not request a specific authentication context. This is [known to help when authenticating against Active Directory](https://github.com/bergie/passport-saml/issues/226) (AD FS) servers.
  * `authnContext`: if truthy, name identifier format to request auth context (default: `urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport`); array of values is also supported
  * `RACComparison`: Requested Authentication Context comparison type. Possible values are 'exact','minimum','maximum','better'. Default is 'exact'.
  * `forceAuthn`: if set to true, the initial SAML request from the service provider specifies that the IdP should force re-authentication of the user, even if they possess a valid session.
  * `passive`: if set to true, the initial SAML request from the service provider specifies that the IdP should prevent visible user interaction if possible.
  * `providerName`: optional human-readable name of the requester for use by the presenter's user agent or the identity provider
  * `skipRequestCompression`: if set to true, the SAML request from the service provider won't be compressed.
  * `disableRequestACSUrl`: if truthy, SAML AuthnRequest from the service provider will not include the optional AssertionConsumerServiceURL. Default is falsy so it is automatically included.
 * **InResponseTo Validation**
  * `validateInResponseTo`: if truthy, then InResponseTo will be validated from incoming SAML responses
  * `requestIdExpirationPeriodMs`: Defines the expiration time when a Request ID generated for a SAML request will not be valid if seen in a SAML response in the `InResponseTo` field. Default is 8 hours.
  * `cacheProvider`: Defines the implementation for a cache provider used to store request Ids generated in SAML requests as part of `InResponseTo` validation. Default is a built-in in-memory cache provider. For details see the 'Cache Provider' section.
 * **Issuer Validation**
  * `idpIssuer`: if provided, then the IdP issuer will be validated for incoming Logout Requests/Responses. For ADFS this looks like `https://acme_tools.windows.net/deadbeef`

 * **Logout**
  * `logoutUrl`: base address to call with logout requests (default: `entryPoint`)
  * `additionalLogoutParams`: dictionary of additional query params to add to 'logout' requests
  * `logoutCallbackUrl`: The value with which to populate the `Location` attribute in the `SingleLogoutService` elements in the generated service provider metadata.


### generateServiceProviderMetadata( decryptionCert, signingCert )


As a convenience, the class exposes a `generateServiceProviderMetadata` method which will generate a service provider metadata document suitable for supplying to an identity provider. This method will only work on when a `callbackUrl` (or `protocol`, `host`, `path`) were specified when instantiating the class (since the relative path for the callback is not sufficient information to generate a complete metadata document).

The `decryptionCert` argument should be a public certificate matching the `decryptionPvk` and is required if the strategy is configured with a `decryptionPvk`.

The `signingCert` argument should be a public certificate matching the `privateCert` and is required if the strategy is configured with a `privateCert`.

## Security and signatures

Authentication requests can be signed using RSA signature with SHA1, SHA256 or SHA512 hashing algorithms.

To select hashing algorithm, use:

```js
...
  signatureAlgorithm: 'sha1' // (deprecated)
  signatureAlgorithm: 'sha256', // (default - your IDP should support it, otherwise think about upgrading it)
  signatureAlgorithm: 'sha512' // (most secure - check if your IDP supports it)
...
```

To sign them you need to provide a private key in the PEM format via the `privateCert` configuration key.

Formats supported for `privateCert` field are,

1. Well formatted PEM:

```
-----BEGIN PRIVATE KEY-----
<private key contents here delimited at 64 characters per row>
-----END PRIVATE KEY-----

```
```
-----BEGIN RSA PRIVATE KEY-----
<private key contents here delimited at 64 characters per row>
-----END RSA PRIVATE KEY-----

```
(both versions work)
See example from tests of the first version of [well formatted private key](test/static/acme_tools_com.key).

2. Alternativelly a single line private key without start/end lines where all rows are joined into single line:

See example from tests of [singleline private key](test/static/singleline_acme_tools_com.key).

Add it to strategy options like this:

```javascript
    privateCert: fs.readFileSync('./privateCert.pem', 'utf-8')
```


It is a good idea to validate the signatures of the incoming SAML Responses. For this, you can provide the Identity Provider's public PEM-encoded X.509 signing certificate using the `cert` confguration key. The "BEGIN CERTIFICATE" and "END CERTIFICATE" lines should be stripped out and the certificate should be provided on a single line.

```javascript
    cert: 'MIICizCCAfQCCQCY8tKaMc0BMjANBgkqh ... W=='
```

If you have a certificate in the binary DER encoding, you can convert it to the necessary PEM encoding like this:

```bash
     openssl x509 -inform der -in my_certificate.cer -out my_certificate.pem
````

If the Identity Provider has multiple signing certificates that are valid (such as during the rolling from an old key to a new key and responses signed with either key are valid) then the `cert` configuration key can be an array:

```javascript
    cert: [ 'MIICizCCAfQCCQCY8tKaMc0BMjANBgkqh ... W==', 'MIIEOTCCAyGgAwIBAgIJAKZgJdKdCdL6M ... g=' ]
```

The `cert` configuration key can also be a function that receives a callback as argument calls back a possible error and a  certificate or array of certificates. This allows the Identity Provider to be polled for valid certificates and the new certificate can be used if it is changed:

```javascript
    cert: function(callback) { callback(null, polledCertificates); }
```

## Usage with Active Directory Federation Services

Here is a configuration that has been proven to work with ADFS:

```javascript
  {
    entryPoint: 'https://ad.example.net/adfs/ls/',
    issuer: 'https://your-app.example.net/login/callback',
    callbackUrl: 'https://your-app.example.net/login/callback',
    cert: 'MIICizCCAfQCCQCY8tKaMc0BMjANBgkqh ... W==',
    authnContext: 'http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/windows',
    identifierFormat: null
  }
```

Please note that ADFS needs to have a trust established to your service in order for this to work.

For more detailed instructions, see [ADFS documentation](docs/adfs/README.md).

## SAML Response Validation - NotBefore and NotOnOrAfter

If the `NotBefore` or the `NotOnOrAfter` attributes are returned in the SAML response, Node SAML will validate them
against the current time +/- a configurable clock skew value. The default for the skew is 0s. This is to account for
differences between the clock time on the client (Node server with Node SAML) and the server (Identity provider).

`NotBefore` and `NotOnOrAfter` can be part of either the `SubjectConfirmation` element, or within in the `Assertion/Conditions` element in the SAML response.

## Subject confirmation validation

When configured (turn `validateInResponseTo` to `true` in the Node SAML config), the `InResponseTo` attribute will be validated.
Validation will succeed if Node SAML previously generated a SAML request with an id that matches the value of `InResponseTo`.

Also note that `InResponseTo` is validated as an attribute of the top level `Response` element in the SAML response, as well
as part of the `SubjectConfirmation` element.

Previous request id's generated for SAML requests will eventually expire. This is controlled with the `requestIdExpirationPeriodMs` option passed into the Node SAML config. The default is 28,800,000 ms (8 hours).
Once expired, a subsequent SAML response received with an `InResponseTo` equal to the expired id will not validate and an error will be returned.

## Cache Provider

When `InResponseTo` validation is turned on, Node SAML will store generated request ids used in SAML requests to the IdP. The implementation of how things are stored, checked to see if they exist, and eventually removed is from the Cache Provider used by Node SAML.

The default implementation is a simple in-memory cache provider. For multiple server/process scenarios, this will not be sufficient as the server/process that generated the request id and stored in memory could be different than the server/process handling the SAML response. The `InResponseTo` could fail in this case erroneously.

To support this scenario you can provide an implementation for a cache provider by providing an object with following functions:

```javascript
{
  async function save(key, value) {
    // save the key with the optional value, returns a Promise that resolves to the the value
  },
  async function get(key) {
    // returns a Promise that resolves to the value if found, null otherwise
  },
  async function remove(key) {
    // removes the key from the cache, returns a Promise that resolves to the
    // key removed, null if no key is removed
  }
}
```

Provide an instance of an object which has these functions passed to the `cacheProvider` config option when using Node SAML.

## SLO (single logout)

Node SAML has built in support for SLO including
* Signature validation
* IdP initiated and SP initiated logouts
* Decryption of encrypted name identifiers in IdP initiated logout
* `Redirect` and `POST` SAML Protocol Bindings

## ChangeLog

See [Releases](https://github.com/zoellner/node-saml/releases) to find the changes that go into each release.

## FAQ

## Node Support Policy

We only support [Long-Term Support](https://github.com/nodejs/Release) versions of Node.

We specifically limit our support to LTS versions of Node, not because this package won't work on other versions, but because we have a limited amount of time, and supporting LTS offers the greatest return on that investment.

It's possible this package will work correctly on newer versions of Node. It may even be possible to use this package on older versions of Node, though that's more unlikely as we'll make every effort to take advantage of features available in the oldest LTS version we support.

As each Node LTS version reaches its end-of-life we will remove that version from the `node` `engines` property of our package's `package.json` file. Removing a Node version is considered a breaking change and will entail the publishing of a new major version of this package. We will not accept any requests to support an end-of-life version of Node. Any merge requests or issues supporting an end-of-life version of Node will be closed.

We will accept code that allows this package to run on newer, non-LTS, versions of Node.
