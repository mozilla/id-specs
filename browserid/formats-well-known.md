## Identity Provider (IdP) Support Document Format

To become a federated Identity Provider (IdP), a domain must publish metadata in a well-formed JSON document. This document must be located at `/.well-known/browserid`, it must be available over SSL/TLS, and it must be served with `Content-Type: application/json`.

By publishing a support document, a domain may opt to:

1. Disable first-party support for BrowserID
2. Delegate authority to another domain
3. Directly act as an Identity Provider

Each option takes precedence over those below it in the list.

### Disabling Support

By setting the key `disabled` to the boolean value `true`, a domain may explicitly opt-out of being a native Identity Provider. When encountering this, clients may fail over to a Fallback Identity Provider.

### Delegating Authority

By setting the key `authority` to a hostname, a domain may explicitly delegate authority to a designated host.

### Supporting BrowserID

To natively support the BrowserID protocol, three values must be present:

1. `authentication`, absolute path to the user authentication page.
2. `provisioning`, absolute path to the headless user provisioning page.
3. `keys`, array of public keys used for signing Identity Certificates.

> __Notes:__ 
>
> - Absolute paths must start with `/`, for example, `/foo/bar`.
> - Public keys must be serialized as JWKs.
> - When acting as an identity provider, the support document fulfills the requirements of a JWK Set and should be treated as such.
