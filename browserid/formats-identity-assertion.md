## Identity Assertion Format

An Identity Assertion associates a client-generated public key with a request to a specific Relying Party. It is a JWT signed by the client's private key, it conforms to JWS, and it is serialized using JWS Compact Serialization.

### Header

The Header of an Identity Assertion contains the following fields:

- `alg`: Required. String. Algorithm used for signing. Suggested values: `"RS256"`, `"RS384"`,  or `"RS512"`.
- `kid`: Optional. String. Key ID of the signing key used by the client.
- `typ`: Optional. String. MIME Type of the payload. Value: `"JWT"`.

The fields `alg` and `kid` are defined by JWS. The field `typ` is defined by JWT.

__TODO__: JWA `alg` parameters for DSA? Which RS values to support?

### Payload

The Payload of an Identity Assertion is a JWT Claims Set with the following Claims, all of which are required:

- `aud`: String. The Relying Party for whom the Identity Assertion is intended. Formatted as a URL including a scheme, hostname, and port, but omitting paths, query parameters, and fragments. The port may be omitted if it is the default for the scheme (80 for HTTP, 443 for HTTPS).
- `exp`: Integer. The time at which the assertion expires, in seconds since the UTC epoch. Assertions should expire within a few minutes of being created.

The fields `aud` and `exp` are defined by JWT.

Additional claims may be added by clients for application-specific use. All claims that are not understood by consumers SHOULD be ignored.

### Backed Identity Assertions

Identity Assertions are often combined with an associated Identity Certificate to form a Backed Identity Assertion.

Backed Identity Assertions are simply the concatenation of a serialized Identity Certificate and a serialized Identity Assertion, with a "`~`" character delimiting the two:

                 Backed Identity Assertion
     ______________________________________________
    /                                               \
    header.payload.signature~header.payload.signature
    \______________________/ \______________________/
      Identity Certificate      Identity Assertion

Backed Identity Assertions:

1. Vouche for an email address, per the signature on the Identity Certificate.
1. Associate an email address with a public key, per the contents of the Identity Certificate.
2. Associate a public key with a client, per the signature on the Identity Assertion.
3. Associate a client with a Relying Party, per the contents of the Identity Assertion.

Thus, if a Backed Identity Assertion is valid, a Relying Party is able know the client's email address and that the client is currently intending to communicate with that specific Relying Party.