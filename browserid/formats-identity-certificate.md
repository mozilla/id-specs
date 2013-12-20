## Identity Certificate Format

An Identity Certificate associates a client-generated public key with an email address. It is a JWT signed by an Identity Provider, it conforms to JWS, and it is serialized using JWS Compact Serialization.

### Header

The Header of an Identity Certificate contains the following fields:

- `alg`: Required. String. Algorithm used for signing. Suggested values: `"RS256"`, `"RS384"`,  or `"RS512"`.
- `kid`: Optional, Recommended. String. Key ID of the signing key used by the Identity Provider.
- `typ`: Optional. String. MIME Type of the payload. Value: `"JWT"`.

The fields `alg` and `kid` are defined by JWS. The field `typ` is defined by JWT.

__TODO__: JWA `alg` parameters for DSA? Which RS values to support?

### Payload

The Payload of an Identity Certificate is a JWT Claims Set with the following Claims, all of which are required:

- `iss`: String. The hostname of the Identity Provider signing this certificate. For informational purposes only.
- `sub`: String. The email address being certified. This must be a string conforming to the HTML5 definition of an email address.
- `iat`: Integer. The time at which the certificate was issued, in seconds since the UTC epoch.
- `exp`: Integer. The time at which the certificate expires, in seconds since the UTC epoch. Certificates must expire within 24 hours of `iat`.
- `pubkey`: Object. The client's public key, serialized as defined elsewhere in this specification.

The fields `iss`, `sub`, `iat`, and `exp` are defined by JWT. The field `pubkey` is a Private Claim Name, as per JWT.

Additional claims may be added by Identity Providers for application-specific use. All claims that are not understood by consumers SHOULD be ignored.