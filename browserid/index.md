BrowserID
=====

This is the production BrowserID specification, working live at <tt>https://browserid.org</tt>.

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

BrowserID defines messages using the [JOSE specifications](http://www.ietf.org/dyn/wg/charter/jose-charter) for signing JSON-formatted objects.

### Public Key ###

A BrowserID public key is a JSON object that includes fields:

* <tt>alg</tt> the algorithm for which this key was generated, using JOSE taxonomy
* additional fields specified by the algorithm, e.g. <tt>n</tt> and <tt>e</tt> for RSA public keys.

For example:

    {
      "alg": "RS256",
      "n" : "4b3c34...",
      "e" : "93bc32...",
    }

This data structure should move to [JSON Web Keys](http://tools.ietf.org/html/draft-jones-json-web-key-01).

### Identity Certificate ###

An Identity Certificate is a JWT object with the following claims:

* <tt>exp</tt> the expiration as per JWT
* <tt>iss</tt> the domain of the issuer as per JWT
* <tt>public-key</tt> the serialized public key as defined above
* <tt>principal</tt> the principal being certified.

The principal is a JSON object that indicates the type of principal, e.g.

    {"email": "bob@example.com"}

or

    {"host": "intermediate.example.com"}

A complete JWT set of claims then looks like:

    {
      "iss": "example.com",
      "exp": "1313971280961",
      "public-key": {
        "alg": "RS256",
        "n" : "4b3c34...",
        "e" : "93bc32...",
      },
      "principal": {
        "email": "john@example.com"
      }
    }

Which, when signed, becomes a base64url-encoded data structure which looks like (with linebreaks and truncated values for for easier reading):

    eyJhbGciOiJSUzI1NiJ9.
    eyJpc3MiOiJicm93c2VyaWQub3JnIiwiZXhwIjoxM...
    hv5wVN0HPINUZlLi4SJo9RzJhMU5_6XZsltYWODDD...

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

    {"alg": "RS64"}

and a payload of:

    {"exp":1320280579437,"aud":"http://localhost:10001"}

### Backed Identity Assertion ###

A Backed Identity Assertion is a combination of an Identity Assertion
and a single sequence of Identity Certificates that verifiably tie the
assertion to an issuing domain. Most often, a backed identity
assertion is a single certificate tying a public-key to an Identity,
signed by the domain, and an Identity Assertion signed by the
just-certified public key.

A Backed Identity Assertion is:

    <cert-1>~...<cert-n>~<identityAssertion>;

where each cert and the identity assertion are base64url-encoded data structures, as defined above.

Web-Site Signin Flow
--

<em>This section is informative.</em>

Consider a web site, <tt>http://example.com</tt>, receiving a visit
from a user. This web site wishes to obtain the user's verified email
address using BrowserID. The user in question, for the purposes of
this description, is Alice. Alice owns two email addresses,
<tt>alice@homedomain</tt> and <tt>alice@workdomain</tt>.

* <tt>example.com</tt> presents a login button with a JavaScript click handler.
* when Alice clicks the login button, <tt>example.com</tt>'s click handler invokes:
<pre>
  navigator.id.get(gotAssertion);
</pre>
where <tt>gotAssertion</tt> is a callback function.
* Alice is presented with a user-agent dialog that lets her select which email to present to <tt>example.com</tt>.
* If Alice chooses to cancel the transaction, <tt>gotAssertion</tt> is invoked with a null argument.
* If Alice chooses to authenticate using one of her email addresses, <tt>gotAssertion</tt> is invoked with a Backed Identity Assertion.
* <tt>example.com</tt> should take this assertion and pass it back to its server. This can be accomplished with an AJAX request. For example, using jQuery:

<pre>
     function gotAssertion(assertion) {
       $.post("/verifyAssertion", {assertion: assertion}, afterVerifyAssertion);
     }
</pre>

This assertion is a Backed Identity Assertion, as defined above. We call it <tt>assertion</tt> here for simplicity, since the Relying Party typically need only pass this assertion to a verifier service without worrying about the specific semantics of the assertion string.

Identity Provisioning Flow
--

Consider Alice, a user of <tt>EyeDee.me</tt>, with email address <tt>alice@eyedee.me</tt>. Alice wishes to user her <tt>alice@eyedee.me</tt> identity to log into web sites that support the BrowserID protocol:

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
   navigator.id.cancelAuthentication();
 }
</pre>

Once this is successfully completed, the user-agent returns to the BrowserID user-interface, and attempts to load the provisioning URL as in the previous step.
* Once a certificate for <tt>alice@eyedee.me</tt> is installed, the user-agent completes the login to <tt>example.com</tt> by creating an assertion and delivering it to <tt>example.com</tt> as in the Main Protocol Flow above.

By the end of this flow, Alice has obtained, within her user-agent, a certificate for her email address issued directly by her email address's domain.

User-Agent Compliance
--
<em>This section is normative.</em>

The User-Agent plays an important role in BrowserID support. Here, we define, normatively, the API that user agents MUST implement, including specific behaviors in response to these API calls. Relying Parties and Identity Providers can safely skip this section.

A compliant BrowserID User-Agent must implement the <tt>navigator.id</tt> object, which serves both for issuing assertions and exposing a provisioning flow to identity providers.

### Issuing Assertions ###

The User Agent MUST offer the following API call:

<tt>navigator.id.get(object callback, object options);</tt>

The Relying Party MAY call the <tt>navigator.id.get</tt> method when it wishes to request that the User Agent generate an identity assertion as soon as it can. When this happens, the User Agent SHOULD pursue the following actions:

1. Establish the origin of the requesting site (including scheme and non-standard port).
1. Check local BrowserID store for known identities that have been successfully used previously.
1. Present the list of known identities. The User Agent MAY suggest a preferred identity out of that list based on heuristics or other internal state, e.g. the email last used on that site.
1. When the user selects an Identity:
 - check that the associated certificate is still valid. If not, initiate a provisioning workflow for that Identity, then continue once it returns successfully.
 - generate an Identity Assertion using the requesting site's origin as audience and the current time. Bundle with the associated certificate to create a Backed Identity Assertion, and invoke the <tt>callback</tt> with, as first and only parameter, a serialization of the Backed Identity Assertion, then terminate the login workflow.
1. If no Identities are known, or if the user wishes to use a new Identity, the User Agent should prompt the user for this new identity and use it to initiate a Provisioning workflow (see below). Once provisioning has completed, the User Agent SHOULD present the updated list of identities to the user.
1. If, at any point, the user cancels the login process, the User Agent SHOULD invoke the <tt>callback</tt> with a single null argument and terminate the login workflow.

By the end of the process, the User Agent MUST invoke the <tt>callback</tt> with either a Backed Identity Assertion or null as first parameter.

### Provisioning ###

The User Agent should support a provisioning workflow when a user wants to authenticate with a new email address. A provisioning workflow is initiated with some context:

* the email address being provisioned
* information about the security status of the session (user's own computer, shared computer, public computer, ...)
* whether the authentication workflow has been invoked yet (initially <tt>false</tt>).

During a provisioning action, the User Agent MUST support the following API calls:

<tt>navigator.id.beginProvisioning(object callback)</tt>

The User Agent SHOULD expect the callback function to accept parameters <tt>email</tt> and <tt>cert_duration_s</tt>.

In response to this call, the User Agent should invoke the callback with parameters based on the provisioning context. The <tt>email</tt> parameter MUST be the email address which the user-agent is attempting to provision. The <tt>cert_duration_s</tt> parameter should be the requested validity duration for the certificate, which the User Agent SHOULD determine based on the security level of the session. For example, public computers should have very short certificate validity.

<tt>navigator.id.genKeyPair(object callback);</tt>

The User Agent SHOULD expect the callback to accept parameter <tt>pubkey</tt>, a serialized public-key string as per the above public-key spec.

In response to this call, the User Agent MUST generate a fresh keypair associated with the email address for this provisioning context. The secret key should be stored internally, and the <tt>callback</tt> should be invoked with the serialized public-key as sole argument.

<tt>navigator.id.registerCertificate(certificate);</tt>

The User Agent SHOULD expect the certificate to be a valid serialized certificate, as per the above spec. The User Agent SHOULD expect the trust root for this certificate to comply with the characteristics described in the "Acceptable Trust Paths" section.

The User Agent MUST associate this certificate with the email address for this provisioning context and store this association internally for later issuance of Backed Identity Assertions.

<tt>navigator.id.raiseProvisioningFailure(string reason);</tt>

The User Agent MUST interrupt this provisioning workflow.

If the context indicates that the authentication workflow has already been invoked, then the User Agent SHOULD return from this workflow indicating failure to authenticate the user.

If the context indicates that the authentication workflow has NOT already been invoked, then the User Agent SHOULD begin the Authentication Workflow (described below).

#### WebIDL ####

<pre>
 module navigator {
     module id {
         void beginProvisioning(object callback);
         void genKeyPair(string email, object callback);
         void registerCertificate(string certificate);
         void raiseProvisioningFailure(string reason);
     }
 };
</pre>

### Authenticating ###

The User Agent MUST support an authentication workflow when a user wants to certify a new email address but has failed the provisioning workflow. An authentication workflow is initiated with some context:

* the email address which requires authentication

During an authentication workflow, the User Agent MUST support the following API calls:

    navigator.id.beginAuthentication(object callback);

The User Agent SHOULD expect a callback function as parameter to this API call.

When this function is invoked, the User Agent MUST invoke the callback function, passing to it the context's email address as parameter.

    navigator.id.completeAuthentication();

When this function is invoked, the User Agent MUST return to its provisioning workflow, retrieving the appropriate context for that provisioning workflow, with the added flag <tt>authenticationPerformed = true</tt>.

    navigator.id.raiseAuthenticationFailure(string reason);

When this function is invoked, the User Agent MUST return to its provisioning workflow and proceed with the failure case.

#### WebIDL ####

<pre>
 module navigator {
     module id {
         void beginAuthentication(object callback);
         void completeAuthentication();
         void raiseAuthenticationFailure(string reason);
     }
 };
</pre>

Primary Authority Compliance
--

<em>This section is normative.</em>

A primary authority MUST:
* declare support and parameters for BrowserID
* provide a user-authentication web flow
* provide a user-key-certification web flow

### BrowserID Support Document ###

A BrowserID support document MUST be a well-formed JSON document with at least these three fields: <tt>public-key</tt>, <tt>authentication</tt>, and <tt>provisioning</tt>. The document MAY contain additional JSON fields.

The value of the <tt>public-key</tt> field MUST be a Public Key serialized as a JSON object, as defined above.

The value of the <tt>authentication</tt> field MUST be a relative reference to a URI, as defined by [RFC3986](https://tools.ietf.org/html/rfc3986).

The value of the <tt>provisioning</tt> field MUST also be a relative reference to a URI.

#### BrowserID Delegated Support Document ####

A BrowserID delegated-support document MUST be a well-formed JSON document with at least one field: <tt>authority</tt>. This field MUST be a domain name.

### Declaring Support and Parameters for BrowserID ###

To declare support for BrowserID, a domain MUST publish either a BrowserID support document OR a BrowserID delegated-support document at a specific URI relative to the domain's SSL URI. The relative reference URI for this document is <tt>/.well-known/browserid</tt>, as per [RFC5785](https://tools.ietf.org/html/rfc5785). The domain MAY choose to reference this BrowserID support document from a host-meta file (as per RFC5785).

The BrowserID support document (or delegated-support document) MUST be served with Content-Type <tt>application/json</tt>.

The BrowserID support document (or delegated-support document) MAY be served with cache headers to indicate longevity of the BrowserID support parameters.

### Authenticating Users ###

A BrowserID-compliant domain MUST provide a user-authentication web flow starting at the URI referenced by the <tt>authentication</tt> field in its published BrowserID support document. The specifics of the user-authentication flow are up to the domain. The flow MAY use redirects to other pages, even other domains, to complete the user authentication process. The flow SHOULD NOT use <tt>window.open()</tt> or other techniques that target new windows/tabs.

The domain MAY serve this authentication workflow using anti-framing directives (e.g. <tt>X-FRAMES-OPTION</tt>).

The authentication flow MUST complete at a URI relative to the BrowserID-compliant domain. The completion page content MUST include a JavaScript call to either <tt>navigator.id.completeAuthentication()</tt> if authentication was successful or <tt>navigator.id.raiseAuthenticationFailure()</tt> if the use cancelled authentication.

### Certifying Users ###

A BrowserID-compliant domain MUST provide user-key certification at the URI referenced by the <tt>provisioning</tt> field in its published BrowserID support document.

The domain SHOULD deliver, at that URI, an HTML document with either embedded or reference JavaScript, which it can expect to be evaluated in a standard user-agent frame. The domain SHOULD NOT use anti-framing directives (e.g. <tt>X-FRAMES-OPTION</tt>) when that URI is requested.

The domain SHOULD determine, without any user-facing content, the user's state of authentication with the domain. The domain MAY use cookies or localStorage to make this determination.

The domain MUST call, in JavaScript:
<pre>
navigator.id.beginProvisioning(provisionEmailFunction);
</pre>
with <tt>provisionEmailFunction</tt> a function that accepts an email address and a duration (in integral seconds) as parameter.

Once the requested email provided as parameter to the <tt>provisionEmailFunction</tt>, the domain SHOULD check that the user is properly authenticated to use this email address. If she isn't, the domain SHOULD call:
<pre>
 navigator.id.raiseProvisioningFailure(explanation)
</pre>
with <tt>explanation</tt> a string explaining the failure. The domain SHOULD conclude all JavaScript activity after making this call.

You SHOULD use one of the following <tt>explanation</tt> codes:
* <tt>user is not authenticated as target user</tt> - Indicates UA should show sign in screen again, due to an error

If the user is properly authenticated, the domain MUST call:
<pre>
 navigator.id.genKeyPair(gotPublicKey);
</pre>
with <tt>gotPublicKey</tt> a function that accepts a public-key JSON object.

The domain's JavaScript SHOULD send this public-key to the domain's backend server. The domain's backend server SHOULD certify this key along with the email address provided to its <tt>provisionEmailFunction</tt> function, and an expiration date at least 1 minute in the future. The backend server MUST NOT issue a certificate valid longer than 24 hours. The backend server SHOULD NOT issue a certificate valid longer than the duration passed to the <tt>provisionEmailFunction</tt> earlier. The domain's backend server SHOULD deliver the generated Identity Certificate back to its JavaScript context. The domain's JavaScript MUST finally call:
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

A relying party MAY use a verification service that performs these steps and returns a summary of results.  In that case, the verification service MUST perform all the checks described here.  In order to perform audience checking, the verification service SHOULD require that the relying party indicate the expected value of the <tt>audience</tt> parameter.

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
