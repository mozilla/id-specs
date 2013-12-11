## Authority Discovery

To discover the authoritative Identity Provider for a given email address:

1. Extract the domain name from the email address and remember this as the Original Domain.

2. Using HTTPS, attempt to GET `/.well-known/browserid` from the Original Domain.

3. Depending on the response:

    a. If the response __redirects to a TLS/SSL-secured location__, then the redirect should be honored and clients should return to Step 3.

    b. If the document is __absent__, __invalid__, __explicitly disables BrowserID support__, or __redirects to a non-TLS/SSL location__, then the domain must not be considered authoritative. Clients may return to Step 2, substituting a Fallback Identity Provider for the Original Domain and adding a query parameter, `domain`, whose value is the domain derived in Step 1.

    c. If the document __explicitly delegates to another domain__, return to Step 2, substituting the delegated domain for the Original Domain and adding a query parameter, `domain`, whose value is the domain derived in Step 1.

    d. If the document is __complete and valid__, the discovered domain should be considered authoritative.

If the `domain` parameter is omitted, its value is considered to be identical to the domain being queried in Step 2.

> __Examples:__
>
> - Direct support by the user's domain.
>
>     1. The user is `alice@example.com`, thus the Original Domain is `example.com`
>     2. GET `https://example.com/.well-known/browserid
>     3. The response is valid: `example.com` is authoritative for `alice@example.com`
>
> - Two levels of delegation by the user's domain.
>
>     1. The user is `alice@example.com`, thus the Original Domain is `example.com`
>     2. GET `https://example.com/.well-known/browserid
>     3. The response delegates to `example.org`
>     4. GET `https://example.org/.well-known/browserid?domain=example.com`
>     5. The response delegates to `accounts.example.org`
>     6. GET `https://accounts.example.org/.well-known/browserid?domain=example.com`
>     7. The response is valid: `accounts.example.org` is authoritative for `alice@example.com`
>
> - No direct support by the user's domain.
>
>     1. The user is `alice@example.com`, thus the Original Domain is `example.com`
>     2. GET `https://example.com/.well-known/browserid
>     3. The response is not a valid Support Document. Attempt discovery at against the `fallback.example` Fallback Identity Provider.
>     4. GET `https://fallback.example/.well-known/browserid?domain=example.com` 
>     5. The response is valid: `fallback.example` is authoritative for `alice@example.com`
