BrowserID
=====

This is the <em>development</em> BrowserID specification, working live at https://dev.diresworb.org.

Overview
-

This specification, BrowserID, defines a mechanism for websites to request, from the user via her user-agent, a signed assertion of email-address ownership. Web sites can use this mechanism to register users on their first visit and log them back in on subsequent visits. The trust path for these assertions of email-address ownership is federated: individual domains can certify their own users. A fallback identity provider is provided to bootstrap users with email addresses at domains that do not yet support the protocol.

Terms
-

- Identity: an email address controlled by the user.

- Public Key: the public portion of an asymmetric cryptographic keypair used in a digital signature algorithm.

- Identity Certificate: a digitally signed statement that binds a given Public Key to a given Identity.

- Identity Provider: a signer of Identity Certificates for identities that are directly within this authority's domain, e.g. <tt>example.com</tt> certifies <tt>*@example.com</tt>.

- Fallback Identity Provider: a signer of Identity Certificates for identities that are ''not'' directly within this authority's domain.

- Audience/Relying Party: a system, typically a web site, that needs to verify an Identity.

- Identity Assertion: a digitally signed statement indicating a request to login to a particular relying party.

- Backed Identity Assertion: an Identity Assertion combined with the requisite Identity Certificates that enable a Relying Party to fully verify the Identity Assertion.

Objects / Messages
-

BrowserID defines messages using the [JavaScript Object Signing and Encryption (JOSE) specifications](http://www.ietf.org/dyn/wg/charter/jose-charter) for signing JSON-formatted objects.

### Public Key ###

A BrowserID public key is a [JSON Web Key (JWK)](http://tools.ietf.org/html/draft-jones-json-web-key-03) object. As specified by JWK, the key looks like:

       {"alg":"RSA",
        "mod": "0vx7agoebGcQSuuPiLJXZptN9nndrQmbXEps2aiAFbWhM78LhWx
    4cbbfAAtVT86zwu1RK7aPFFxuhDR1L6tSoc_BJECPebWKRXjBZCiFV4n3oknjhMs
    tn64tZ_2W-5JsGY4Hc5n9yBXArwl93lqt7_RN5w6Cf0h4QyQ5v-65YGjQR0_FDW2
    QvzqY368QQMicAtaSqzs8KJZgnYb9c7d0zgdAZHzu6qMQvRL5hajrn1n91CbOpbI
    SD08qNLyrdkt-bFTWhAI4vMQFh6WeZu0fM4lFd2NcRwr3XPksINHaQ-G_xBniIqb
    w0Ls1jF44-csFCur-kEgU8awapJzKnqDKgw",
        "exp":"AQAB",
        "kid":"key-2011-04-29"}

This structure includes:

* <tt>alg</tt> the algorithm for which this key was generated, using JOSE taxonomy
* additional fields specified by the algorithm, e.g. <tt>mod</tt> and <tt>exp</tt> for RSA public keys.
* <tt>kid</tt> an optional key identifier.

When more than one key might represent the same entity, a full JWK object is used:

     {"jwk":
      [
       {"alg":"RSA",
        "mod": "0vx7agoebGcQSuuPiLJXZptN9nndrQmbXEps2aiAFbWhM78LhWx
    4cbbfAAtVT86zwu1RK7aPFFxuhDR1L6tSoc_BJECPebWKRXjBZCiFV4n3oknjhMs
    tn64tZ_2W-5JsGY4Hc5n9yBXArwl93lqt7_RN5w6Cf0h4QyQ5v-65YGjQR0_FDW2
    QvzqY368QQMicAtaSqzs8KJZgnYb9c7d0zgdAZHzu6qMQvRL5hajrn1n91CbOpbI
    SD08qNLyrdkt-bFTWhAI4vMQFh6WeZu0fM4lFd2NcRwr3XPksINHaQ-G_xBniIqb
    w0Ls1jF44-csFCur-kEgU8awapJzKnqDKgw",
        "exp":"AQAB",
        "kid":"key-2011-04-29"}

       ...
      ]
     }


### Identity Certificate ###

An Identity Certificate is a JSON Web Token (JWT) object with the following claims:

* <tt>exp</tt> the expiration as per JWT
* <tt>iss</tt> the domain of the issuer as per JWT
* <tt>publicKey</tt> the serialized public key as defined above
* <tt>principal</tt> the principal being certified.

The principal is a JSON object that indicates the type of principal, e.g.

    {"email": "bob@example.com"}

or

    {"host": "intermediate.example.com"}

A complete JWT set of claims then looks like:

    {
      "iss": "example.com",
      "exp": "1313971280961",
      "publicKey": {
        "alg":"RSA",
        "mod": "0vx7agoebGcQSu...",
        "exp":"AQAB"},
      "principal": {
        "email": "john@example.com"
      }
    }

Which, when signed, becomes a base64url-encoded data structure which looks like (with linebreaks and truncated values for for easier reading):

    eyJhbGciOiJSUzI1NiJ9.
    eyJpc3MiOiJicm93c2VyaWQub3JnIiwiZXhwIjoxM...
    hv5wVN0HPINUZlLi4SJo9RzJhMU5_6XZsltYWODDD...

#### Chained Certificates ####

When a certificate certifies a key, it is meant ONLY as a binding of the key to an identity. This binding MUST NOT be interpreted as a grant of certification authority to that key, UNLESS the certificate explicitly indicates such delegation of authority.

To perform this delegation, the certificate MUST include the field:

    ...
      "delegate": {
        ...
      }
    ...

The specifics of the delegation field indicate how much delegation of authority is indicated. At this time, only two types of delegation are supported:

    ...
      "delegate": {
        "all": true
      }
    ...

which explicitly delegates all authority.

Also:

    ...
      "delegate": {
        "domains": ["foo.com", "bar.com"]
      }
    ...

which delegates only certification of users at those domains.

#### JOSE Spec ####

The JOSE spec currently does not specify a certificate format beyond JWS signatures. If it eventually does, we will consider moving to it.

### Identity Assertion ###

An Identity Assertion is a JWT with the following claims:

* <tt>exp</tt> for expiration
* <tt>aud</tt> for the relying party (audience.)

An assertion might look like (with line breaks for readability):

    eyJhbGciOiJSUzY0In0.
    eyJleHAiOjEzMjAyODA1Nzk0MzcsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MTAwMDEifQ.
    JmEBqwOH_qzw6_EHsCRB-CeShGyQ2y0bpapARZ308_8uT6TCWrKBpB8L2bFnMb664lz1nGytkBXF-tTIzGCOjg

which is a JWT with header:

    {"alg": "RS256"}

and a payload of:

    {"exp":1320280579437,"aud":"http://localhost:10001"}

### Backed Identity Assertion ###

A Backed Identity Assertion is a combination of an Identity Assertion and a sequence of Identity Certificates that verifiably tie the assertion to an issuing domain. This combination is expressed as a single string like this:

    <cert-1>~...<cert-n>~<identityAssertion>;

where each cert and the identity assertion are base64url-encoded data structures, as defined above, the strings are joined by tilde characters (U+007E), and the final combination is terminated by a semicolon (U+003B).

The first element is certified by the issuing domain's private key, and each subsequent element is certified by the previous one.

Most often, a backed identity assertion is a single certificate tying a public-key to an Identity (signed by the domain), and an Identity Assertion signed by the just-certified public key, e.g:

    <cert-1>~<identityAssertion>;

in which cert-1 has an "iss" of the issuing domain (e.g. "example.com"), a "publicKey" of the user's certified key, a "principal" of <tt>{"email": "user@example.com"}</tt>, and is signed by the example.com private key. The identityAssertion would have an "exp" and "aud" field, and is signed by the user's private key.

### BrowserID Support Document ###

A BrowserID support document MUST be a well-formed JSON document with at least these three fields: <tt>jwk</tt>, <tt>authentication</tt>, and <tt>provisioning</tt>. The document MAY contain additional JSON fields.

The value of the <tt>jwk</tt> field MUST be a JWK object as described above, with <tt>kid</tt> specified for all keys.

The value of the <tt>authentication</tt> field MUST be a relative reference to a URI, as defined by [RFC3986](https://tools.ietf.org/html/rfc3986).

The value of the <tt>provisioning</tt> field MUST also be a relative reference to a URI.

For example:

     {
        "jwk": [
           {
            "algorithm": "RSA",
            ...,
            "kid": "generated-on-2012-06-20"
           },
           ...
        ],
        "authentication": "/browserid/sign_in.html",
        "provisioning": "/browserid/provision.html"
     }

#### BrowserID Delegated Support Document ####

A BrowserID delegated-support document MUST be a well-formed JSON document with at least one field: <tt>authority</tt>. This field MUST be a domain name.

For example:

     {
        "authority": "otherexample.com"
     }


Web-Site Signin Flow
--

<em>This section is informative.</em>

The BrowserID JavaScript API can be used by web sites to implement
authentication. Implementing BrowserID support consists of:

  1. registering callback functions that will be invoked when the user logs in or out
     via `navigator.id.watch()`
  2. invoking `navigator.id.request()` when the user clicks a login button on your site.
  3. invoking `navigator.id.logout()` when the user clicks a logout button on your site.

### Key Ideas

Typically on the web, login sessions are implemented with cookies and are completely managed by a website.  With BrowserID you can continue to implement sessions as you do today, but must take a couple extra steps to synchronize your session with BrowserID.  Specifically, you should handle the case where the user logs in or out via BrowserID, while your page is not loaded.

**Watching Login State**: The `navigator.id.watch()` function allows you to register javascript functions that will be invoked when users log in or out, which may occur while they're visiting your website, or while it's not even loaded. Each page of your site should call `navigator.id.watch()` and be prepared to handle asynchronous changes to login state.

**Telling BrowserID who is logged in**: An important parameter to `navigator.id.watch()` is `loggedInEmail`. When supplied, this parameter tells BrowserID whether the current page load is associated with a logged in user.  By supplying this parameter, you can speed up your page load by suppressing unnecessary callbacks (and using less CPU and network resources):

  * If you supply `null`, and no user is logged in according to BrowserID, your `onlogout`
    callback will not be invoked.
  * If you supply an email address and that user is still logged in
    via BrowserID, your `onlogin` callback will not be invoked.
  * If you supply an email address and a different user is logged in
    via BrowserID, your `onlogin` callback will be invoked with a new assertion.
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

Register callbacks to be notified when the user logs in or out.  The option block has the following properties:

  * `loggedInEmail` *(optional)* - The email address of the currently logged
    in user.  May be a string (email address), or `null` (indicating
    no user is logged in).  If provided, the `onlogin` or `onlogout`
    callbacks will not be invoked if the users' login state is
    consistent with the value provided.  If omitted or `undefined`,
    one of the two callbacks will be invoked on every page load.
  * `onlogin` *(required)* - A callback that will be invoked and
    passed a single argument, an assertion, when the user logs in.
  * `onlogout` *(required)* - A callback that will be invoked when
    the user logs out.
  * `onready` *(optional)* - A callback that will always be called
    once the navigator.id service is initialized (after `onlogin` or
    `onlogout` have been called).  By waiting to display UI until this
    point, you can avoid UI flicker in the case where your session is out
    of sync with BrowserID.

#### navigator.id.request(&lt;options&gt;);

Request an identity from the user.  This will cause a dialog to be opened to prompt the user for an email address to use to log into the site.  This function must be invoked from within a click handler.  The argument is an options block which may contain the following properties:

  * `requiredEmail` *(optional)* - An email address that the user must
    use to log in.  When provided, the user may not select a different
    address, but may cancel the sign-in.
  * `privacyPolicy` - URL to site's privacy policy.  When provided, a link
    will be displayed in the sign-in dialog.
  * `termsOfService` - URL to site's terms of service.  When provided, a link will
    be displayed in the sign-in dialog.
  * `oncancel` - a callback that will be invoked if the user refuses to
    share an identity with the site.

#### navigator.id.logout();

A function that should be invoked when a user wishes to logout of the current site (for instance, when clicking on an in-content logout button).  Will cause the `onlogout` callback passed to `navigator.id.watch()` to be invoked.


Identity Provisioning Flow
--

<em>This section is informative</em>

Consider Alice, a user of <tt>EyeDee.me</tt>, with email address <tt>alice@eyedee.me</tt>. Alice wishes to use her <tt>alice@eyedee.me</tt> identity to log into web sites that support the BrowserID protocol:

* Alice visits <tt>example.com</tt> and clicks "login."
* In the BrowserID interface, Alice types her email address <tt>alice@eyedee.me</tt>.
* The user-agent checks <tt>https://eyedee.me/.well-known/browserid</tt> and determines that <tt>eyedee.me</tt> supports BrowserID. From this configuration file it determines the provisioning and authentication URLs.
* The user-agent loads, in an invisible IFRAME, the provisioning URL <tt>https://eyedee.me/browserid/provision.html</tt>, delivering to that URL any cookies that have previously been set and making available to that page's JavaScript any <tt>localStorage</tt> that corresponds to the <tt>eyedee.me</tt> origin.
* The provisioning URL's script determines if Alice is properly authenticated and, if so, triggers key generation within the user agent, obtains the public key, signs it, and registers the resulting certificate with the user agent:

<pre>
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
</pre>

* If Alice is not properly authenticated, the user agent loads the authentication URL <tt>https://eyedee.me/browserid/authenticate.html</tt> in a dialog interface, where Alice can then proceed to log into <tt>EyeDee.me</tt> using whatever flow/method EyeDee.me wishes.

<pre>
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
</pre>

Once this is successfully completed, the user-agent returns to the BrowserID user-interface, and attempts to load the provisioning URL as in the previous step.

* Once a certificate for <tt>alice@eyedee.me</tt> is installed, the user-agent completes the login to <tt>example.com</tt> by creating an assertion and delivering it to <tt>example.com</tt> as in the Main Protocol Flow above.

By the end of this flow, Alice has obtained, within her user-agent, a certificate for her email address issued directly by her email address's domain.

User-Agent Compliance
--
<em>This section is normative.</em>

The User-Agent plays an important role in BrowserID support. We define, normatively, the API and behaviors of a conforming BrowserID-enabled user agent. Relying Parties and Identity Providers can safely skip this section.

A compliant BrowserID User-Agent MUST implement:

* Keypair and Certificate Storage
* IdP Determination
* IdP Provisioning
* IdP Authentication
* Login-state Tracking
* Generation of Identity-Backed Assertions
* RP Authentication

### Keypair and Certificate Storage ###

The User Agent MUST implement an internal store of keypairs and associated certificates. We denote `CERTS(identity)` for ease of specification. Its value is either `undefined` or an object with properties:

* `cert`, a JSON Certificate as defined above,
* `publicKey`, a JSON Public Key as defined above, matching the public key in `cert`,
* `secretKeyIdentifier`, an identifier for the corresponding secret key.

The User Agent MAY implement this keypair and certificate storage in a different manner, with a different interface. The User Agent SHOULD be careful to disclose the actual secret key in very few settings.

### IdP Determination ###

The User Agent MUST be able to determine an IdP, designated by a domain name, for a given Identity. The User Agent MUST follow these steps:

* for the given Identity, e.g. `alice@example.com`, determine the domain portion of the identifier, e.g. `example.com`
* look up the BrowserID configuration at `https://<DOMAIN>/.well-known/browserid`, e.g. in this case `https://example.com/.well-known/browserid`. If this lookup fails, use the Fallback IdP.
* parse the BrowserID configuration file as a JSON object.
* if the configuration is a proper BrowserID Support document, then store the JSON object as `IDP(identity)` for the requested Identity, with `provisioning` and `authentication` resolved as URLs relative to `https://<domain>`
* if the configuration is a proper BrowserID Delegated Support document, then proceed recursively with the configuration lookup process for the delegated domain, i.e. the value of the `authority` parameter.
* otherwise, fail and use the Fallback IdP.

#### Fallback IdP ####

The User Agent SHOULD use `browserid.org` as the IdP when IdP Determination proceeds in Fallback Mode.

### IdP Provisioning ###

The User Agent MUST support a provisioning workflow when a user wants to authenticate with a new email address. A provisioning workflow is initiated with some context:

* the identity address being provisioned, e.g. `alice@example.com`
* security level of the session (user's own computer, shared computer, public computer, ...)
* whether the authentication workflow has been invoked yet for this provisioning workflow (initially <tt>false</tt>).

We denote these `CONTEXT.identity`, `CONTEXT.securityLevel`, and `CONTEXT.authenticationDone`.

The User Agent MUST determine a new context value, `CONTEXT.certValidityDuration`, an integer indicating the recommended validity of a certificate, measured in seconds and based on `CONTEXT.securityLevel`. This determination MAY be made in any way the User Agent deems useful. This value MUST NOT be longer than 24 hours.

The User Agent MUST proceed as follows:

* determine `IDP(identity)`, e.g. `example.com` for `alice@example.com` if `example.com` supports BrowserID.

* open an invisible web content frame, e.g. IFRAME, settings its document location to `IDP(identity).provisioning`

* respond to the IdP API calls.

#### `navigator.id.beginProvisioning(object callback)` ####

The User Agent SHOULD expect the callback parameter to be a function.

The User Agent MUST invoke the callback asynchronously (i.e. after the call to `beginProvisioning` has returned) with parameters `CONTEXT.identity` and `CONTEXT.certValidityDuration`.

<tt>navigator.id.genKeyPair(object callback);</tt>

The User Agent MUST proceed to Provisioning Hard-Fail if this API call is made before `navigator.id.beginProvisioning`.

The User Agent SHOULD expect the callback parameter to be a function.

The User Agent MUST generate a fresh keypair associated with the email address for this provisioning context. The secret key should be stored as `CERTS(identity)` where `CERTS(identity.cert` remains `undefined` for now. The User Agent MUST invoke <tt>callback</tt> with, as its single argument, `CERTS(identity).publicKey` serialized as a string.

<tt>navigator.id.registerCertificate(certificate);</tt>

The User Agent MUST proceed to Provisioning Hard-Fail if this API call is made before `navigator.id.genKeyPair`.

If `certificate` is not a valid serialized JSON Certificate, or if the trust-root for `certificate` is not `IDP(identity)`, the User Agent SHOULD proceed to Provisioning Hard-Fail.

The User Agent MUST store the value of `certificate` in `CERTS(identity).cert`

<tt>navigator.id.raiseProvisioningFailure(string reason);</tt>

The User Agent MUST proceed with Provisioning Soft-Fail.

#### Provisioning Soft-Fail ####

The User Agent SHOULD close the provisioning content frame.

If `CONTEXT.authenticationDone` is `true`, proceed with Provisioning Hard-Fail.

Otherwise, set `CONTEXT.authenticationDone` to `true` and proceed with Authentication.

#### Provisioning Hard-Fail ####

The User Agent SHOULD close the provisioning content frame if this hasn't yet been done.

The User Agent MUST declare failure on identity provisioning. The User Agent SHOULD surface an error message to the user. The User Agent MAY allow the user to retry.

### IdP Authentication ###

The User Agent MUST support an authentication workflow using `CONTEXT` unchanged from the preceding Provisioning flow.

The User Agent MUST open a user-visible content frame and set its document location to `IDP(CONTEXT.identity).authentication`. The User Agent SHOULD provide a trusted indicator of the document's origin to the user, e.g. by displaying a URL bar.

The User Agent SHOULD allow the content frame to be navigated to any URL it desires, as per normal web flows. The User Agent SHOULD treat this content frame like a normal content frame opened to that origin as if it were a top-level frame, carrying appropriate cookies and `localStorage`.

The User Agent MAY choose to limit the capabilities of this content frame in certain ways, e.g. allowing only SSL origins, restricting plug-ins, etc.

The User Agent MUST support the following API calls in this content frame when the content frame document location matches the origin of `IDP(CONTEXT.identity).authentication`. The User Agent SHOULD proceed to Authentication Hard-Fail if these calls are invoked from an origin that doesn't match.

#### `navigator.id.beginAuthentication(object callback)` ####

The User Agent SHOULD expect `callback` to be a function.

The User Agent MUST invoke the callback asynchronously (i.e. after the call to `beginAuthentication` has returned) with `CONTEXT.identity` as single parameter.

#### navigator.id.completeAuthentication(); ####

The User Agent MUST invoke the Provisioning Flow, continuing with the existing `CONTEXT`.

#### navigator.id.raiseAuthenticationFailure(string reason); ####

The User Agent MUST proceed to Provisioning Hard-Fail.

#### WebIDL ####

<pre>
 module navigator {
     module id {
         void beginProvisioning(object callback);
         void genKeyPair(string email, object callback);
         void registerCertificate(string certificate);
         void raiseProvisioningFailure(string reason);

         void beginAuthentication(object callback);
         void completeAuthentication();
         void raiseAuthenticationFailure(string reason);
     }
 };
</pre>


### Tracking User State ###

The User Agent MUST track the user's login state at various Web Origins, based on decisions the user made in the context of BrowserID-mediated logins, so that the following information is available to the User Agent when making decisions:

* `HISTORY(origin).loggedIn` is `true` if the user should currently be logged into `origin`.
* `HISTORY(origin).id` is the last ID used to log into `origin`, or `undefined`.
* `HISTORY(origin).lastLogin` is a `Date` of the last time the user logged into the origin, or `undefined` if never.

Note that the above data is denoted programmatically to make this specification easier to read. The data <em>MUST NOT</em> be available to web content.

### Generating Identity-Backed Assertions ###

When

### RP Authentication ###

The User Agent MUST offer the following APIs and behave as described in response to these calls.

#### navigator.id.watch(object params);

The User Agent SHOULD throw an exception immediately UNLESS `params` includes the following parameters:

1. `onlogin`: a function
1. `onlogout`: a function

If `params` includes the above required parameters, the User Agent SHOULD store the entire `params` object scoped to the `document`, including the above callbacks and any additional fields. If the `document` DOM object disappears for any reason (unloading, closing, etc.), these callbacks should be garbage-collected.

If `params.loggedInEmail` is `undefined`, the User Agent SHOULD:

* if `HISTORY(origin).loggedIn`, invoke `onlogin` callback for the identity `HISTORY(origin).id` (see
othe fire either the `onlogin` or `onlogout`
The User Agent SHOULD determine whether `params.loggedInEmail` matches the User Agent's login state for the `document.location.origin`.

If

#### navigator.id.request(object options);

The Relying Party MAY call the navigator.id.request method when it wishes to request that the User Agent generate an identity assertion as soon as it can. When this happens, the User Agent SHOULD pursue the following actions:

1. Establish the origin of the requesting site (including scheme and non-standard port).
1. Check local BrowserID store for known identities that have been successfully used previously.
1. Present the list of known identities. The User Agent MAY suggest a preferred identity out of that list based on heuristics or other internal state, e.g. the email last used on that site.
1. When the user selects an Identity:
 - check that the associated certificate is still valid. If not, initiate a provisioning workflow for that Identity, then continue once it returns successfully.
 - generate an Identity Assertion using the requesting site's origin as audience and the current time. Bundle with the associated certificate to create a Backed Identity Assertion, and fire a <tt>login</tt> event on the <tt>navigator.id</tt> object with a serialization of the Backed Identity Assertion in the <tt>assertion</tt> field of the event, then terminate the login workflow.
1. If no Identities are known, or if the user wishes to use a new Identity, the User Agent should prompt the user for this new identity and use it to initiate a Provisioning workflow (see below). Once provisioning has completed, the User Agent SHOULD present the updated list of identities to the user.
1. If, at any point, the user cancels the login process, fire a <tt>logincanceled</tt> event on the <tt>navigator.id</tt> object and terminate the login workflow.

By the end of the process, the User Agent MUST fire one of two events on the <tt>navigator.id</tt> object:

* A <tt>loginCancelled</tt> event if the user chose not to log in.

* A <tt>login</tt> event if the user chose to log in. This event MUST include the Backed Identity Assertion in the <tt>assertion</tt> property.

XXX: should we provide error information if it's not just a user cancel?


Primary Authority Compliance
--

<em>This section is normative.</em>

A primary authority MUST:
* declare support and parameters for BrowserID
* provide a user-authentication web flow
* provide a user-key-certification web flow

### Declaring Support and Parameters for BrowserID ###

To declare support for BrowserID, a domain MUST publish either a BrowserID support document OR a BrowserID delegated-support document at a specific URI relative to the domain's SSL URI. The relative reference URI for this document is <tt>/.well-known/browserid</tt>, as per [RFC5785](https://tools.ietf.org/html/rfc5785). The domain MAY choose to reference this BrowserID support document from a host-meta file (as per RFC5785).

The BrowserID support document (or delegated-support document) MUST be served with Content-Type <tt>application/json</tt>.

The BrowserID support document (or delegated-support document) MAY be served with cache headers to indicate longevity of the BrowserID support parameters.

### Authenticating Users ###

A BrowserID-compliant domain MUST provide a user-authentication web flow starting at the URI referenced by the <tt>authentication</tt> field in its published BrowserID support document. The specifics of the user-authentication flow are up to the domain. The flow MAY use redirects to other pages, even other domains, to complete the user authentication process. The flow SHOULD NOT use <tt>window.open()</tt> or other techniques that target new windows/tabs.

The authentication flow MUST complete at a URI relative to the BrowserID-compliant domain. The completion page content MUST include a JavaScript call to either <tt>navigator.id.completeAuthentication()</tt> if authentication was successful or <tt>navigator.id.raiseAuthenticationFailure()</tt> if the use cancelled authentication.

### Certifying Users ###

A BrowserID-compliant domain MUST provider user-key-certification at the URI referenced by the <tt>provisioning</tt> field in its published BrowserID support document.

The domain SHOULD deliver HTML and JavaScript at that URI, which it can expect to be evaluated in a standard user-agent IFRAME.

The domain SHOULD determine, without any user-facing content, the user's state of authentication with the domain. The domain MAY use cookies or localStorage to make this determination.

The domain MUST call, in JavaScript:
<pre>
navigator.id.beginProvisioning(provisionEmailFunction);
</pre>
with <tt>provisionEmailFunction</tt> a function that accepts an email address and a certificate validity duration as parameters.

Once the email address determined, the domain SHOULD check that the user is properly authenticated to use this email address. If she isn't, the domain SHOULD call
<pre>
 navigator.id.raiseProvisioningFailure(explanation)
</pre>
with <tt>explanation</tt> a string explaining the failure. The domain SHOULD concludes all JavaScript activity after making this call.

You SHOULD use one of the following <tt>explanation</tt> codes:
* <tt>user is not authenticated as target user</tt> - Indicates UA should show sign in screen again, due to an error

If the user is properly authenticated, the domain MUST call:
<pre>
 navigator.id.genKeyPair(gotPublicKey);
</pre>
with <tt>gotPublicKey</tt> a function that accepts a JWK-string-formatted public-key.

The domain's JavaScript SHOULD then send this JWK string to the domain's backend server. The domain's backend server SHOULD certify this key along with the email address provided to its <tt>provisionEmailFunction</tt> function, and an expiration date at least 1 minutes in the future. The backend server SHOULD NOT issue a certificate valid longer than 24 hours. The domain's backend server SHOULD then deliver an Identity Certificate back to its JavaScript context. The domain's JavaScript MUST finally call:
<pre>
 navigator.id.registerCertificate(certificate);
</pre>
with the Identity Certificate string.

Assertion Verification
-

Backed Identity Assertions SHOULD NOT be verified in the client, in JavaScript or otherwise, since client runtimes may be altered to circumvent such verification. Instead, Backed Identity Assertions SHOULD be sent to a trusted server for verification.

To verify a Backed Identity Assertion, a Relying Party SHOULD perform the following checks:

1. If the <tt>exp</tt> date of the assertion is earlier than the current time by more than a certain interval, the assertion has expired and must be rejected. A Relying Party MAY choose the length of that interval, though it is recommended that it be less than 5 minutes.
1. If the <tt>audience</tt> field of the assertion does not match the Relying Party's origin (including scheme and optional non-standard port), reject the assertion. A domain that includes the standard port, of 80 for HTTP and 443 for HTTPS, SHOULD be treated as equivalent to a domain that matches the protocol but does not include the port.  (XXX: Can we find an RFC that defines this equality test?)
1. If the Identity Assertion's signature does not verify against the public-key within the last Identity Certificate, reject the assertion.
1. If there is more than one Identity Certificate, then reject the assertion unless each certificate after the first one is properly signed by the prior certificate's public key.
1. If the first certificate (or only certificate when there is only one) is not properly signed by the expected issuer's public key, reject the assertion. The expected issuer is either the domain of the certified email address in the last certificate, or the issuer listed in the first certificate if the email-address domain does not support BrowserID.
1. If the expected issuer was designated by the certificate rather than discovered given the user's email address, then the issuer SHOULD be <tt>browserid.org</tt>, otherwise reject the assertion.

Note that a relying party may, at its discretion, use a verification service that performs these steps and returns a summary of results.  In that case, the verification service MUST perform all the checks described here.  In order to perform audience checking, the verification service must be told what audience to expect by the relying party.

Security Considerations
-

things to write about:

* certificate validity period
* UAs reusing a key to get a new certificate
* timing attacks
* javascript implementations, good RNGs

References
--

* JWT: http://self-issued.info/docs/draft-jones-json-web-token-04.html
* JWK: http://self-issued.info/docs/draft-jones-json-web-key.html
