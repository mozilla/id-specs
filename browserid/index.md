BrowserID
=========

This is the _Beta1_ BrowserID specification, going live at `https://login.persona.org` in September 2012.

Overview
--------

This specification, BrowserID, defines a mechanism for websites to request, from the user via her user-agent, a signed assertion of email-address ownership.
Web sites can use this mechanism to register users on their first visit and log them back in on subsequent visits.
The trust path for these assertions of email-address ownership is federated: individual domains can certify their own users.
A fallback identity provider is provided to bootstrap users with email addresses at domains that do not yet support the protocol.

Terms
-----

<dl>
  <dt>Identity</dt>
  <dd>An email address controlled by the user.</dd>

  <dt>Public Key</dt>
  <dd>The public portion of an asymmetric cryptographic keypair used in a digital signature algorithm.</dd>

  <dt>Identity Certificate</dt>
  <dd>A digitally signed statement that binds a given Public Key to a given Identity.</dd>

  <dt>Identity Provider</dt>
  <dd>A signer of Identity Certificates for identities that are directly within this authority's domain, e.g. `example.com` certifies `*@example.com`.</dd>

  <dt>Fallback Identity Provider</dt>
  <dd>A signer of Identity Certificates for identities that are _not_ directly within this authority's domain.</dd>

  <dt>Audience/Relying Party</dt>
  <dd>A system, typically a web site, that needs to verify an Identity.</dd>

  <dt>Identity Assertion</dt>
  <dd>A digitally signed statement indicating a request to login to a particular relying party.</dd>

  <dt>Backed Identity Assertion</dt>
  <dd>An Identity Assertion combined with the requisite Identity Certificates that enable a Relying Party to fully verify the Identity Assertion.</dd>
</dl>

Objects / Messages
------------------

When possible, BrowserID defines messages using the [JavaScript Object Signing and Encryption (JOSE) specifications](http://www.ietf.org/dyn/wg/charter/jose-charter) for signing JSON-formatted objects.
However, because JOSE is still in active development, and due to historical accidents, the BrowserID protocol makes decisions that may diverge from the current JOSE spec.
Later versions of the protocol are expected to match JOSE more closely.

Future messages will contain a "version" property to distinguish their format from this initial specification.


### Public Key

A BrowserID public key is based [JSON Web Key (JWK)](http://tools.ietf.org/html/draft-ietf-jose-json-web-key-01), but that standard is in flux.
The current format looks like:

    { "algorithm": "RS",
      "n": "15498874758090276039465094105837231567265546373975960480941122651107772824121527483107402353899846252489837024870191707394743196399582959425513904762996756672089693541009892030848825079649783086005554442490232900875792851786203948088457942416978976455297428077460890650409549242124655536986141363719589882160081480785048965686285142002320767066674879737238012064156675899512503143225481933864507793118457805792064445502834162315532113963746801770187685650408560424682654937744713813773896962263709692724630650952159596951348264005004375017610441835956073275708740239518011400991972811669493356682993446554779893834303",
      "e": "65537"
    }

or:

    { algorithm": "DS",
      "g": "c52a4a0ff3b7e61fdf1867ce84138369a6154f4afa92966e3c827e25cfa6cf508b90e5de419e1337e07a2e9e2a3cd5dea704d175f8ebf6af397d69e110b96afb17c7a03259329e4829b0d03bbc7896b15b4ade53e130858cc34d96269aa89041f409136c7242a38895c9d5bccad4f389af1d7a4bd1398bd072dffa896233397a",
      "p": "ff600483db6abfc5b45eab78594b3533d550d9f1bf2a992a7a8daa6dc34f8045ad4e6e0c429d334eeeaaefd7e23d4810be00e4cc1492cba325ba81ff2d5a5b305a8d17eb3bf4a06a349d392e00d329744a5179380344e82a18c47933438f891e22aeef812d69c8f75e326cb70ea000c3f776dfdbd604638c2ef717fc26d02e17",
      "q": "e21e04f911d1ed7991008ecaab3bf775984309c3",
      "y": "2b0b6f44f58ec4fd5043be6c68433bc839bb867276f90a9c7a68071097167d2cab2df53aa5ae928843d15a42412123ee24c4067d7b8587850d1f09fa39cc5bb52f8b8844c3132440f2e455aea8235535b28a8f01588209f145ee1f265257fe9999bc90547ba985052ad4fb320fb9153878164bf3572dc5c4fe493e66506f2b04"
    }

This structure includes:

* `algorithm` the algorithm for which this key was generated, using JOSE taxonomy (`RS` or `DS`)
* additional fields specified by the algorithm, e.g. RSA keys use decimal `n` for the modulus and `e` for the exponent, DSA keys use hex `g`, `p`, `q`, and `y`.

### Identity Certificate

An Identity Certificate is a signed JSON Web Token (JWT)-like object.
This starts with a JWT Header.
BrowserID currently omits the `typ` field in the JWT Header, so a typical header looks like:

    {"alg": "RS256"}

This indicates that the Identity Certificate is signed by an RSA key with a 256-bit security level (i.e. 2048-bit modulus).

The JWT Claims Set contains the following claims:

* `iat`: the "issued-at" time, when the certificate becomes valid (in milliseconds-since-epoch)
* `exp`: the expiration time, as per JWT, when the certificate ceases being valid (milliseconds-since-epoch)
* `iss` the domain of the issuer as per JWT
* `public-key` the serialized public key as defined above
* `principal` the principal being certified.

The principal is a JSON object that indicates the type of principal.
The only value currently accepted is an email address, e.g.

    {"email": "bob@example.com"}

A complete JWT set of claims then looks like:

    {
      "iss": "mockmyid.com",
      "iat": 1347994401401,
      "exp": 1347998001401,
      "principal": {"email": "warner@mockmyid.com"},
      "public-key": {
        "algorithm": "DS",
        "g": "c52a4a0ff3b7e61fdf1867ce84138369a6154f4afa92966e3c827e25cfa6cf508b90e5de419e1337e07a2e9e2a3cd5dea704d175f8ebf6af397d69e110b96afb17c7a03259329e4829b0d03bbc7896b15b4ade53e130858cc34d96269aa89041f409136c7242a38895c9d5bccad4f389af1d7a4bd1398bd072dffa896233397a",
        "p": "ff600483db6abfc5b45eab78594b3533d550d9f1bf2a992a7a8daa6dc34f8045ad4e6e0c429d334eeeaaefd7e23d4810be00e4cc1492cba325ba81ff2d5a5b305a8d17eb3bf4a06a349d392e00d329744a5179380344e82a18c47933438f891e22aeef812d69c8f75e326cb70ea000c3f776dfdbd604638c2ef717fc26d02e17",
        "q": "e21e04f911d1ed7991008ecaab3bf775984309c3",
        "y": "2b0b6f44f58ec4fd5043be6c68433bc839bb867276f90a9c7a68071097167d2cab2df53aa5ae928843d15a42412123ee24c4067d7b8587850d1f09fa39cc5bb52f8b8844c3132440f2e455aea8235535b28a8f01588209f145ee1f265257fe9999bc90547ba985052ad4fb320fb9153878164bf3572dc5c4fe493e66506f2b04"
      }
    }

Which, when signed, becomes a base64url-encoded data structure (using `_-` instead of `+/`) which looks like the following (with linebreaks added for easier reading; the full string has exactly two periods and no whitespace):

    eyJhbGciOiJSUzI1NiJ9
    .eyJwdWJsaWMta2V5Ijp7ImFsZ29yaXRobSI6IkRTIiwieSI6IjJiMGI2ZjQ0ZjU4ZWM0ZmQ1MDQzYmU2YzY4NDMzYmM4MzliYjg2NzI3NmY5MGE5YzdhNjgwNzEwOTcxNjdkMmNhYjJkZjUzYWE1YWU5Mjg4NDNkMTVhNDI0MTIxMjNlZTI0YzQwNjdkN2I4NTg3ODUwZDFmMDlmYTM5Y2M1YmI1MmY4Yjg4NDRjMzEzMjQ0MGYyZTQ1NWFlYTgyMzU1MzViMjhhOGYwMTU4ODIwOWYxNDVlZTFmMjY1MjU3ZmU5OTk5YmM5MDU0N2JhOTg1MDUyYWQ0ZmIzMjBmYjkxNTM4NzgxNjRiZjM1NzJkYzVjNGZlNDkzZTY2NTA2ZjJiMDQiLCJwIjoiZmY2MDA0ODNkYjZhYmZjNWI0NWVhYjc4NTk0YjM1MzNkNTUwZDlmMWJmMmE5OTJhN2E4ZGFhNmRjMzRmODA0NWFkNGU2ZTBjNDI5ZDMzNGVlZWFhZWZkN2UyM2Q0ODEwYmUwMGU0Y2MxNDkyY2JhMzI1YmE4MWZmMmQ1YTViMzA1YThkMTdlYjNiZjRhMDZhMzQ5ZDM5MmUwMGQzMjk3NDRhNTE3OTM4MDM0NGU4MmExOGM0NzkzMzQzOGY4OTFlMjJhZWVmODEyZDY5YzhmNzVlMzI2Y2I3MGVhMDAwYzNmNzc2ZGZkYmQ2MDQ2MzhjMmVmNzE3ZmMyNmQwMmUxNyIsInEiOiJlMjFlMDRmOTExZDFlZDc5OTEwMDhlY2FhYjNiZjc3NTk4NDMwOWMzIiwiZyI6ImM1MmE0YTBmZjNiN2U2MWZkZjE4NjdjZTg0MTM4MzY5YTYxNTRmNGFmYTkyOTY2ZTNjODI3ZTI1Y2ZhNmNmNTA4YjkwZTVkZTQxOWUxMzM3ZTA3YTJlOWUyYTNjZDVkZWE3MDRkMTc1ZjhlYmY2YWYzOTdkNjllMTEwYjk2YWZiMTdjN2EwMzI1OTMyOWU0ODI5YjBkMDNiYmM3ODk2YjE1YjRhZGU1M2UxMzA4NThjYzM0ZDk2MjY5YWE4OTA0MWY0MDkxMzZjNzI0MmEzODg5NWM5ZDViY2NhZDRmMzg5YWYxZDdhNGJkMTM5OGJkMDcyZGZmYTg5NjIzMzM5N2EifSwicHJpbmNpcGFsIjp7ImVtYWlsIjoid2FybmVyQG1vY2tteWlkLmNvbSJ9LCJpYXQiOjEzNDc5OTQ0MDE0MDEsImV4cCI6MTM0Nzk5ODAwMTQwMSwiaXNzIjoibW9ja215aWQuY29tIn0
    .ddZEPpT3E1BdZfOLABoRZhvnKidzpU8jj0XLUTrbq7khtOqSwVUCk4nYCgIiy73cHLemInurE8Fm_4uwUw27fJ8nP4IM7BBe_5deBEEIAx5I_ckru5JfsSITbmMx-dw5-A8aTsjYnr_amPj1QfELhXKd0F59DQvFm_kiWsZzygmaLh5DGzJqPcqIJDUh6R35Brg6stlwJSXJHtMkXQ7khjStKiN822RjmSOAYUMfMZwe8r-Y3TxmLDeVcNpbjXEy7p2TGdYLuBc9072JHN0y3riuu7DiG4TLZ83SMtzwnyBzBuMRN0gX_JRxEfBkeeEfDpOJ4JAzHeiRTExAoEkpSQ

#### JOSE Spec

The JOSE spec currently does not specify a certificate format beyond JWS signatures.
If it eventually does, we will consider moving to it.

### Identity Assertion

An Identity Assertion is a JWT-like object, signed by an Identity Certificate, with the following claims:

* `exp` for expiration (milliseconds-since-epoch)
* `aud` for the relying party (audience), a URL that includes the scheme and any non-standard port number, like `https://123done.org` or `https://example.com:8080`

An assertion might look like (with whitespace added for readability):

    eyJhbGciOiJEUzEyOCJ9
    .eyJleHAiOjEzNDc5OTQ2OTgxNDAsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MTAwMDEifQ
    .F32SIQWeKNDFSRVdpOWkWrEdKdG-itlsFkPhY_P4eXdtG9YiG24kvw

which is a JWT-like object with header:

    {"alg":"DS128"}

and a payload of:

    {'aud': 'http://localhost:10001', 'exp': 1347994698140}

### Backed Identity Assertion

A Backed Identity Assertion is a combination of an Identity Assertion and an Identity Certificate that verifiably tie the assertion to an issuing domain.
The certificate ties a public-key to an Identity (signed by the domain which "owns" that Identity), and the assertion is signed by the public key.
The combination is expressed as a single string like this:

    <cert-1>~<identityAssertion>

where the cert and the identity assertion are base64url-encoded data structures, as defined above, and the strings are joined by tilde characters (U+007E).
For example, cert-1 could have an "iss" of the issuing domain (e.g. "example.com"), a "public-key" of the user's certified key, a "principal" of `{"email": "user@example.com"}`, and would be signed by the example.com private key.
The identityAssertion would have an "exp" and "aud" field, and is signed by the user's private key.

A Backed Identity Assertion might look like this (with whitespace added for readability):

    eyJhbGciOiJSUzI1NiJ9
    .eyJwdWJsaWMta2V5Ijp7ImFsZ29yaXRobSI6IkRTIiwieSI6IjJiMGI2ZjQ0ZjU4ZWM0ZmQ1MDQzYmU2YzY4NDMzYmM4MzliYjg2NzI3NmY5MGE5YzdhNjgwNzEwOTcxNjdkMmNhYjJkZjUzYWE1YWU5Mjg4NDNkMTVhNDI0MTIxMjNlZTI0YzQwNjdkN2I4NTg3ODUwZDFmMDlmYTM5Y2M1YmI1MmY4Yjg4NDRjMzEzMjQ0MGYyZTQ1NWFlYTgyMzU1MzViMjhhOGYwMTU4ODIwOWYxNDVlZTFmMjY1MjU3ZmU5OTk5YmM5MDU0N2JhOTg1MDUyYWQ0ZmIzMjBmYjkxNTM4NzgxNjRiZjM1NzJkYzVjNGZlNDkzZTY2NTA2ZjJiMDQiLCJwIjoiZmY2MDA0ODNkYjZhYmZjNWI0NWVhYjc4NTk0YjM1MzNkNTUwZDlmMWJmMmE5OTJhN2E4ZGFhNmRjMzRmODA0NWFkNGU2ZTBjNDI5ZDMzNGVlZWFhZWZkN2UyM2Q0ODEwYmUwMGU0Y2MxNDkyY2JhMzI1YmE4MWZmMmQ1YTViMzA1YThkMTdlYjNiZjRhMDZhMzQ5ZDM5MmUwMGQzMjk3NDRhNTE3OTM4MDM0NGU4MmExOGM0NzkzMzQzOGY4OTFlMjJhZWVmODEyZDY5YzhmNzVlMzI2Y2I3MGVhMDAwYzNmNzc2ZGZkYmQ2MDQ2MzhjMmVmNzE3ZmMyNmQwMmUxNyIsInEiOiJlMjFlMDRmOTExZDFlZDc5OTEwMDhlY2FhYjNiZjc3NTk4NDMwOWMzIiwiZyI6ImM1MmE0YTBmZjNiN2U2MWZkZjE4NjdjZTg0MTM4MzY5YTYxNTRmNGFmYTkyOTY2ZTNjODI3ZTI1Y2ZhNmNmNTA4YjkwZTVkZTQxOWUxMzM3ZTA3YTJlOWUyYTNjZDVkZWE3MDRkMTc1ZjhlYmY2YWYzOTdkNjllMTEwYjk2YWZiMTdjN2EwMzI1OTMyOWU0ODI5YjBkMDNiYmM3ODk2YjE1YjRhZGU1M2UxMzA4NThjYzM0ZDk2MjY5YWE4OTA0MWY0MDkxMzZjNzI0MmEzODg5NWM5ZDViY2NhZDRmMzg5YWYxZDdhNGJkMTM5OGJkMDcyZGZmYTg5NjIzMzM5N2EifSwicHJpbmNpcGFsIjp7ImVtYWlsIjoid2FybmVyQG1vY2tteWlkLmNvbSJ9LCJpYXQiOjEzNDc5OTQ0MDE0MDEsImV4cCI6MTM0Nzk5ODAwMTQwMSwiaXNzIjoibW9ja215aWQuY29tIn0
    .ddZEPpT3E1BdZfOLABoRZhvnKidzpU8jj0XLUTrbq7khtOqSwVUCk4nYCgIiy73cHLemInurE8Fm_4uwUw27fJ8nP4IM7BBe_5deBEEIAx5I_ckru5JfsSITbmMx-dw5-A8aTsjYnr_amPj1QfELhXKd0F59DQvFm_kiWsZzygmaLh5DGzJqPcqIJDUh6R35Brg6stlwJSXJHtMkXQ7khjStKiN822RjmSOAYUMfMZwe8r-Y3TxmLDeVcNpbjXEy7p2TGdYLuBc9072JHN0y3riuu7DiG4TLZ83SMtzwnyBzBuMRN0gX_JRxEfBkeeEfDpOJ4JAzHeiRTExAoEkpSQ
    ~eyJhbGciOiJEUzEyOCJ9
    .eyJleHAiOjEzNDc5OTQ2OTgxNDAsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MTAwMDEifQ
    .F32SIQWeKNDFSRVdpOWkWrEdKdG-itlsFkPhY_P4eXdtG9YiG24kvw


Multiple certificates in a single backed assertion will be used by certificate-chaining, which is not defined or implemented yet.
The elements of these chains will also be joined by tilde characters, e.g.:

    <cert-1>~<cert-2>~<cert-3>~<identityAssertion>.

The first element will be certified by the issuing domain's private key, and each subsequent element will be certified by the previous one.

### BrowserID Support Document

A BrowserID support document MUST be a well-formed JSON document with at least these three fields: `public-key`, `authentication`, and `provisioning`.
The document MAY contain additional JSON fields.

The value of the `public-key` field MUST be a Public Key object as described above.

The value of the `authentication` field MUST be a relative reference to a URI, as defined by [RFC3986](https://tools.ietf.org/html/rfc3986).

The value of the `provisioning` field MUST also be a relative reference to a URI.

For example:

     {
       "public-key": {
           "algorithm": "RS",
           ...,
        },
        "authentication": "/browserid/sign_in.html",
        "provisioning": "/browserid/provision.html"
     }

#### BrowserID Delegated Support Document

A BrowserID delegated-support document MUST be a well-formed JSON document with at least one field: `authority`.
This field MUST be a domain name.

For example:

     {
        "authority": "otherexample.com"
     }


Web-Site Signin Flow
--------------------

_This section is informative._

The BrowserID JavaScript API can be used by web sites to implement authentication.
Implementing BrowserID support consists of:

  1. registering callback functions that will be invoked when the user logs in or out via `navigator.id.watch()`
  2. invoking `navigator.id.request()` when the user clicks a login button on your site.
  3. invoking `navigator.id.logout()` when the user clicks a logout button on your site.

### Key Ideas

Typically on the web, login sessions are implemented with cookies and are completely managed by a website.
With BrowserID you can continue to implement sessions as you do today, but must take a couple extra steps to synchronize your session with BrowserID.
Specifically, you should handle the case where the user logs in or out via BrowserID, while your page is not loaded.

**Watching Login State**: The `navigator.id.watch()` function allows you to register javascript functions that will be invoked when users log in or out, which may occur while they're visiting your website, or while it's not even loaded.
Each page of your site should call `navigator.id.watch()` and be prepared to handle asynchronous changes to login state.

**Telling BrowserID who is logged in**: An important parameter to `navigator.id.watch()` is `loggedInEmail`.
When supplied, this parameter tells BrowserID whether the current page load is associated with a logged in user.
By supplying this parameter, you can speed up your page load by suppressing unnecessary callbacks (and using less CPU and network resources):

  * If you supply `null`, and no user is logged in according to BrowserID, your `onlogout` callback will not be invoked.
  * If you supply an email address and that user is still logged in via BrowserID, your `onlogin` callback will not be invoked.
  * If you supply an email address and a different user is logged in via BrowserID, your `onlogin` callback will be invoked with a new assertion.
    `onlogout` will not be invoked [XXX is this right?]

### Example Code

    /* get the email address of currently logged in user from your server */
    var emailAddressOfCurrentlyLoggedInUser = ...;

    navigator.id.watch({
      loggedInEmail: emailAddressOfCurrentlyLoggedInUser,
      onlogin: function(assertion) {
        // a user has logged in!  now verify the assertion on your server, create a session,
        // and update your UI
      },
      onlogout: function() {
        // a user has logged out!  make a call to your server or redirect the user
        // to tear down the session
      },
      onready: function() {
        // this is called *after* onlogin or onlogout, so you can display the login state
        // and user-specific information.
      }
    });

    // now let's set up code that will run when the user clicks on the login button
    var loginButton = document.querySelector("#loginbutton");
    loginButton.onclick = function() {
      navigator.id.request()
    };

    // and set up code that will run when the user clicks on the logout button
    var logoutButton = document.querySelector("#logoutbutton");
    logoutButton.onclick = function() {
      navigator.id.logout(); // this will cause `onlogout` to be invoked.
    };

### API

#### navigator.id.watch(&lt;options&gt;);

Register callbacks to be notified when the user logs in or out.
The option block has the following properties:

  * `loggedInEmail` *(optional)* - The email address of the currently logged in user.
    May be a string (email address), or `null` (indicating no user is logged in).
    If provided, the `onlogin` or `onlogout` callbacks will not be invoked if the users' login state is consistent with the value provided.
    If omitted or `undefined`, one of the two callbacks will be invoked on every page load.
  * `onlogin` *(required)* - A callback that will be invoked and passed a single argument, an assertion, when the user logs in.
  * `onlogout` *(required)* - A callback that will be invoked when the user logs out.
  * `onready` *(optional)* - A callback that will always be called once the navigator.id service is initialized (after `onlogin` or `onlogout` have been called).
    By waiting to display UI until this point, you can avoid UI flicker in the case where your session is out of sync with BrowserID.

#### navigator.id.request(&lt;options&gt;);

Request an identity from the user.
This will cause a dialog to be opened to prompt the user for an email address to use to log into the site.
This function must be invoked from within a click handler.
The argument is an options block which may contain the following properties:

  * `requiredEmail` *(optional)* - An email address that the user must use to log in.
    When provided, the user may not select a different address, but may cancel the sign-in.
  * `privacyPolicy` - URL to site's privacy policy.
    When provided, a link will be displayed in the sign-in dialog.
  * `termsOfService` - URL to site's terms of service.
    When provided, a link will be displayed in the sign-in dialog.
  * `oncancel` - a callback that will be invoked if the user refuses to share an identity with the site.

#### navigator.id.logout();

A function that should be invoked when a user wishes to logout of the current site (for instance, when clicking on an in-content logout button).
Will cause the `onlogout` callback passed to `navigator.id.watch()` to be invoked.


Identity Provisioning Flow
--------------------------

_This section is informative._

Consider Alice, a user of `EyeDee.me`, with email address `alice@eyedee.me`.
Alice wishes to use her `alice@eyedee.me` identity to log into web sites that support the BrowserID protocol:

* Alice visits `example.com` and clicks "login."
* In the BrowserID interface, Alice types her email address `alice@eyedee.me`.
* The user-agent checks `https://eyedee.me/.well-known/browserid` and determines that `eyedee.me` supports BrowserID.
  From this configuration file it determines the provisioning and authentication URLs.
* The user-agent loads, in an invisible IFRAME, the provisioning URL `https://eyedee.me/browserid/provision.html`, delivering to that URL any cookies that have previously been set and making available to that page's JavaScript any `localStorage` that corresponds to the `eyedee.me` origin.
* The provisioning URL's script determines if Alice is properly authenticated and, if so, triggers key generation within the user agent, obtains the public key, signs it, and registers the resulting certificate with the user agent:

        // get parameters of provisioning
        navigator.id.beginProvisioning(function(email, cert_duration) {

           // ... check if the current user is authenticated as 'email' ...
           if (notAuthenticated()) {
               navigator.id.raiseProvisioningFailure("user isn't authenticated");
               return;
           }

           // request a keypair be generated by browserid and get the public key
           navigator.id.genKeyPair(function(pubkey) {

               // ... interact with the server to sign the public key and get
               // a certificate ...
               someServerInteraction(function(cert){
                   // pass the certificate back to BrowserID and complete the
                   // provisioining process
                   navigator.id.registerCertificate(cert);
               });
           });
        });

* If Alice is not properly authenticated, the user agent loads the authentication URL `https://eyedee.me/browserid/authenticate.html` in a dialog interface, where Alice can then proceed to log into `EyeDee.me` using whatever flow/method EyeDee.me wishes.

        // set up UI
        navigator.id.beginAuthentication(function(email) {
          // update UI to display the email address
        });

        function onAuthentication() {
          // check if the user authenticated successfully, if not, tell them
          // it's a bad password. otherwise..
          navigator.id.completeAuthentication();
        }

        function onCancel() {
          navigator.id.raiseAuthenticationFailure("user canceled");
        }

Once this is successfully completed, the user-agent returns to the BrowserID user-interface, and attempts to load the provisioning URL as in the previous step.

* Once a certificate for `alice@eyedee.me` is installed, the user-agent completes the login to `example.com` by creating an assertion and delivering it to `example.com` as in the Main Protocol Flow above.

By the end of this flow, Alice has obtained, within her user-agent, a certificate for her email address issued directly by her email address's domain.

User-Agent Compliance
---------------------

_This section is normative._

The User-Agent plays an important role in BrowserID support.
We define, normatively, the API and behaviors of a conforming BrowserID-enabled user agent.
Relying Parties and Identity Providers can safely skip this section.

A compliant BrowserID User-Agent MUST implement:

* Keypair and Certificate Storage
* IdP Determination
* IdP Provisioning
* IdP Authentication
* Login-state Tracking
* Generation of Identity-Backed Assertions
* RP Authentication

### Keypair and Certificate Storage

The User Agent MUST implement an internal store of keypairs and associated certificates.
We denote `CERTS(identity)` for ease of specification.
Its value is either `undefined` or an object with properties:

* `cert`, a JSON Certificate as defined above,
* `publicKey`, a JSON Public Key as defined above, matching the public key in `cert`,
* `secretKeyIdentifier`, an identifier for the corresponding secret key.

The User Agent MAY implement this keypair and certificate storage in a different manner, with a different interface.
The User Agent SHOULD be careful to disclose the actual secret key in very few settings.

### IdP Determination

The User Agent MUST be able to determine an IdP, designated by a domain name, for a given Identity.
The User Agent MUST follow these steps:

* for the given Identity, e.g. `alice@example.com`, determine the domain portion of the identifier, e.g. `example.com`
* look up the BrowserID configuration at `https://<DOMAIN>/.well-known/browserid`, e.g. in this case `https://example.com/.well-known/browserid`.
  If this lookup fails, use the Fallback IdP.
* parse the BrowserID configuration file as a JSON object.
* if the configuration is a proper BrowserID Support document, then store the JSON object as `IDP(identity)` for the requested Identity, with `provisioning` and `authentication` resolved as URLs relative to `https://<domain>`
* if the configuration is a proper BrowserID Delegated Support document, then proceed recursively with the configuration lookup process for the delegated domain, i.e. the value of the `authority` parameter.
* otherwise, fail and use the Fallback IdP.

#### Fallback IdP

The User Agent SHOULD use `browserid.org` as the IdP when IdP Determination proceeds in Fallback Mode.

### IdP Provisioning

The User Agent MUST support a provisioning workflow when a user wants to authenticate with a new email address.
A provisioning workflow is initiated with some context:

* the identity address being provisioned, e.g. `alice@example.com`
* security level of the session (user's own computer, shared computer, public computer, ...)
* whether the authentication workflow has been invoked yet for this provisioning workflow (initially `false`).

We denote these `CONTEXT.identity`, `CONTEXT.securityLevel`, and `CONTEXT.authenticationDone`.

The User Agent MUST determine a new context value, `CONTEXT.certValidityDuration`, an integer indicating the recommended validity of a certificate, measured in seconds and based on `CONTEXT.securityLevel`.
This determination MAY be made in any way the User Agent deems useful.
This value MUST NOT be longer than 24 hours.

The User Agent MUST proceed as follows:

* determine `IDP(identity)`, e.g. `example.com` for `alice@example.com` if `example.com` supports BrowserID.

* open an invisible web content frame, e.g. IFRAME, settings its document location to `IDP(identity).provisioning`

* respond to the IdP API calls.

#### `navigator.id.beginProvisioning(object callback)`

The User Agent SHOULD expect the callback parameter to be a function.

The User Agent MUST invoke the callback asynchronously (i.e. after the call to `beginProvisioning` has returned) with parameters `CONTEXT.identity` and `CONTEXT.certValidityDuration`.

`navigator.id.genKeyPair(object callback);`

The User Agent MUST proceed to Provisioning Hard-Fail if this API call is made before `navigator.id.beginProvisioning`.

The User Agent SHOULD expect the callback parameter to be a function.

The User Agent MUST generate a fresh keypair associated with the email address for this provisioning context.
The secret key should be stored as `CERTS(identity)` where `CERTS(identity.cert` remains `undefined` for now.
The User Agent MUST invoke `callback` with, as its single argument, `CERTS(identity).publicKey` serialized as a string.

`navigator.id.registerCertificate(certificate);`

The User Agent MUST proceed to Provisioning Hard-Fail if this API call is made before `navigator.id.genKeyPair`.

If `certificate` is not a valid serialized JSON Certificate, or if the trust-root for `certificate` is not `IDP(identity)`, the User Agent SHOULD proceed to Provisioning Hard-Fail.

The User Agent MUST store the value of `certificate` in `CERTS(identity).cert`

`navigator.id.raiseProvisioningFailure(string reason);`

The User Agent MUST proceed with Provisioning Soft-Fail.

#### Provisioning Soft-Fail

The User Agent SHOULD close the provisioning content frame.

If `CONTEXT.authenticationDone` is `true`, proceed with Provisioning Hard-Fail.

Otherwise, set `CONTEXT.authenticationDone` to `true` and proceed with Authentication.

#### Provisioning Hard-Fail

The User Agent SHOULD close the provisioning content frame if this hasn't yet been done.

The User Agent MUST declare failure on identity provisioning.
The User Agent SHOULD surface an error message to the user.
The User Agent MAY allow the user to retry.

### IdP Authentication

The User Agent MUST support an authentication workflow using `CONTEXT` unchanged from the preceding Provisioning flow.

The User Agent MUST open a user-visible content frame and set its document location to `IDP(CONTEXT.identity).authentication`.
The User Agent SHOULD provide a trusted indicator of the document's origin to the user, e.g. by displaying a URL bar.

The User Agent SHOULD allow the content frame to be navigated to any URL it desires, as per normal web flows.
The User Agent SHOULD treat this content frame like a normal content frame opened to that origin as if it were a top-level frame, carrying appropriate cookies and `localStorage`.

The User Agent MAY choose to limit the capabilities of this content frame in certain ways, e.g. allowing only SSL origins, restricting plug-ins, etc.

The User Agent MUST support the following API calls in this content frame when the content frame document location matches the origin of `IDP(CONTEXT.identity).authentication`.
The User Agent SHOULD proceed to Authentication Hard-Fail if these calls are invoked from an origin that doesn't match.

#### `navigator.id.beginAuthentication(object callback)`

The User Agent SHOULD expect `callback` to be a function.

The User Agent MUST invoke the callback asynchronously (i.e. after the call to `beginAuthentication` has returned) with `CONTEXT.identity` as single parameter.

#### navigator.id.completeAuthentication();

The User Agent MUST invoke the Provisioning Flow, continuing with the existing `CONTEXT`.

#### navigator.id.raiseAuthenticationFailure(string reason);

The User Agent MUST proceed to Provisioning Hard-Fail.

#### WebIDL

    module navigator {
        module id {
            void beginProvisioning(object callback);
            void genKeyPair(object callback);
            void registerCertificate(string certificate);
            void raiseProvisioningFailure(string reason);

            void beginAuthentication(object callback);
            void completeAuthentication();
            void raiseAuthenticationFailure(string reason);
        }
    };


### Tracking User State

The User Agent MUST track the user's login state at various Web Origins, based on decisions the user made in the context of BrowserID-mediated logins, so that the following information is available to the User Agent when making decisions:

* `HISTORY(origin).loggedIn` is `true` if the user should currently be logged into `origin`.
* `HISTORY(origin).id` is the last ID used to log into `origin`, or `undefined`.
* `HISTORY(origin).lastLogin` is a `Date` of the last time the user logged into the origin, or `undefined` if never.

Note that the above data is denoted programmatically to make this specification easier to read.
The data _MUST NOT_ be available to web content.

### Generating Identity-Backed Assertions

When

### RP Authentication

The User Agent MUST offer the following APIs and behave as described in response to these calls.

#### navigator.id.watch(object params);

The User Agent SHOULD throw an exception immediately UNLESS `params` includes the following parameters:

1. `onlogin`: a function
2. `onlogout`: a function

If `params` includes the above required parameters, the User Agent SHOULD store the entire `params` object scoped to the `document`, including the above callbacks and any additional fields.
If the `document` DOM object disappears for any reason (unloading, closing, etc.), these callbacks should be garbage-collected.

If `params.loggedInEmail` is `undefined`, the User Agent SHOULD:

* if `HISTORY(origin).loggedIn`, invoke `onlogin` callback for the identity `HISTORY(origin).id` (see othe fire either the `onlogin` or `onlogout`
The User Agent SHOULD determine whether `params.loggedInEmail` matches the User Agent's login state for the `document.location.origin`.

If

#### navigator.id.request(object options);

The Relying Party MAY call the navigator.id.request method when it wishes to request that the User Agent generate an identity assertion as soon as it can.
When this happens, the User Agent SHOULD pursue the following actions:

1. Establish the origin of the requesting site (including scheme and non-standard port).
2. Check local BrowserID store for known identities that have been successfully used previously.
3. Present the list of known identities.
The User Agent MAY suggest a preferred identity out of that list based on heuristics or other internal state, e.g.
the email last used on that site.
4. When the user selects an Identity:
 * check that the associated certificate is still valid.
   If not, initiate a provisioning workflow for that Identity, then continue once it returns successfully.
 * generate an Identity Assertion using the requesting site's origin as audience and the current time.
   Bundle with the associated certificate to create a Backed Identity Assertion, and fire a `login` event on the `navigator.id` object with a serialization of the Backed Identity Assertion in the `assertion` field of the event, then terminate the login workflow.
5. If no Identities are known, or if the user wishes to use a new Identity, the User Agent should prompt the user for this new identity and use it to initiate a Provisioning workflow (see below).
Once provisioning has completed, the User Agent SHOULD present the updated list of identities to the user.
6. If, at any point, the user cancels the login process, fire a `logincanceled` event on the `navigator.id` object and terminate the login workflow.

By the end of the process, the User Agent MUST fire one of two events on the `navigator.id` object:

* A `loginCancelled` event if the user chose not to log in.

* A `login` event if the user chose to log in.
  This event MUST include the Backed Identity Assertion in the `assertion` property.

XXX: should we provide error information if it's not just a user cancel?


Primary Authority Compliance
----------------------------

_This section is normative._

A primary authority MUST:
* declare support and parameters for BrowserID
* provide a user-authentication web flow
* provide a user-key-certification web flow

### Declaring Support and Parameters for BrowserID

To declare support for BrowserID, a domain MUST publish either a BrowserID support document OR a BrowserID delegated-support document at a specific URI relative to the domain's SSL URI.
The relative reference URI for this document is `/.well-known/browserid`, as per [RFC5785](https://tools.ietf.org/html/rfc5785).
The domain MAY choose to reference this BrowserID support document from a host-meta file (as per RFC5785).

The BrowserID support document (or delegated-support document) MUST be served with Content-Type `application/json`.

The BrowserID support document (or delegated-support document) MAY be served with cache headers to indicate longevity of the BrowserID support parameters.

### Authenticating Users

A BrowserID-compliant domain MUST provide a user-authentication web flow starting at the URI referenced by the `authentication` field in its published BrowserID support document.
The specifics of the user-authentication flow are up to the domain.
The flow MAY use redirects to other pages, even other domains, to complete the user authentication process.
The flow SHOULD NOT use `window.open()` or other techniques that target new windows/tabs.

The authentication flow MUST complete at a URI relative to the BrowserID-compliant domain.
The completion page content MUST include a JavaScript call to either `navigator.id.completeAuthentication()` if authentication was successful or `navigator.id.raiseAuthenticationFailure()` if the use cancelled authentication.

### Certifying Users

A BrowserID-compliant domain MUST provider user-key-certification at the URI referenced by the `provisioning` field in its published BrowserID support document.

The domain SHOULD deliver HTML and JavaScript at that URI, which it can expect to be evaluated in a standard user-agent IFRAME.

The domain SHOULD determine, without any user-facing content, the user's state of authentication with the domain.
The domain MAY use cookies or localStorage to make this determination.

The domain MUST call, in JavaScript:

    navigator.id.beginProvisioning(provisionEmailFunction);

with `provisionEmailFunction` a function that accepts an email address and a certificate validity duration as parameters.

Once the email address determined, the domain SHOULD check that the user is properly authenticated to use this email address.
If she isn't, the domain SHOULD call

    navigator.id.raiseProvisioningFailure(explanation)

with `explanation` a string explaining the failure.
The domain SHOULD concludes all JavaScript activity after making this call.

You SHOULD use one of the following `explanation` codes:
* `user is not authenticated as target user` - Indicates UA should show sign in screen again, due to an error

If the user is properly authenticated, the domain MUST call:

    navigator.id.genKeyPair(gotPublicKey);

with `gotPublicKey` a function that accepts a JWK-string-formatted public-key.

The domain's JavaScript SHOULD then send this JWK string to the domain's backend server.
The domain's backend server SHOULD certify this key along with the email address provided to its `provisionEmailFunction` function, and an expiration date at least 1 minutes in the future.
The backend server SHOULD NOT issue a certificate valid longer than 24 hours.
The domain's backend server SHOULD then deliver an Identity Certificate back to its JavaScript context.
The domain's JavaScript MUST finally call:

    navigator.id.registerCertificate(certificate);

with the Identity Certificate string.

Assertion Verification
----------------------

Backed Identity Assertions SHOULD NOT be verified in the client, in JavaScript or otherwise, since client runtimes may be altered to circumvent such verification.
Instead, Backed Identity Assertions SHOULD be sent to a trusted server for verification.

To verify a Backed Identity Assertion, a Relying Party SHOULD perform the following checks:

1. If the `exp` date of the assertion is earlier than the current time by more than a certain interval, the assertion has expired and must be rejected.
A Relying Party MAY choose the length of that interval, though it is recommended that it be less than 5 minutes.
2. If the `audience` field of the assertion does not match the Relying Party's origin (including scheme and optional non-standard port), reject the assertion.
A domain that includes the standard port, of 80 for HTTP and 443 for HTTPS, SHOULD be treated as equivalent to a domain that matches the protocol but does not include the port.
(XXX: Can we find an RFC that defines this equality test?)
3. If the Identity Assertion's signature does not verify against the public-key within the last Identity Certificate, reject the assertion.
4. If there is more than one Identity Certificate, then reject the assertion unless each certificate after the first one is properly signed by the prior certificate's public key.
5. If the first certificate (or only certificate when there is only one) is not properly signed by the expected issuer's public key, reject the assertion.
The expected issuer is either the domain of the certified email address in the last certificate, or the issuer listed in the first certificate if the email-address domain does not support BrowserID.
6. If the expected issuer was designated by the certificate rather than discovered given the user's email address, then the issuer SHOULD be `browserid.org`, otherwise reject the assertion.

Note that a relying party may, at its discretion, use a verification service that performs these steps and returns a summary of results.
In that case, the verification service MUST perform all the checks described here.
In order to perform audience checking, the verification service must be told what audience to expect by the relying party.

Security Considerations
-----------------------

things to write about:

* certificate validity period
* UAs reusing a key to get a new certificate
* timing attacks
* javascript implementations, good RNGs

References
----------

* JWT: http://self-issued.info/docs/draft-jones-json-web-token-04.html
* JWK: http://tools.ietf.org/html/draft-ietf-jose-json-web-key-01
