## Public Key Format

BrowserID Public Keys follow [the JWK specification](http://tools.ietf.org/html/draft-ietf-jose-json-web-key-18#section-3) and come in two flavors, as defined by their required `kty` parameter:

1. `"RSA"`: [As defined by JWA](http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms-18.html#section-6.3).
2. `"DSA"`: Custom extension to JWA, defined below.

Only the `kty`, `kid`, and RSA/DSA-specific parameters are used.

Identity Providers are strongly urged to define `kid` values for their keys.

[As per JWS](http://tools.ietf.org/html/draft-ietf-jose-json-web-signature-18#appendix-C), all trailing "`=`" characters are omitted from base64url encoded values.

### Parameters for DSA Keys

__TODO__: Fully define p, q, g, and y.

JWKs can represent DSA keys. In this case, the `kty` member value MUST be `DSA`.

#### Parameters for DSA Public Keys

The following members MUST be present for DSA public keys.

##### "p" () Parameter

TODO. It is represented as the base64url encoding of the value's unsigned big endian representation as an octet sequence. The octet sequence MUST utilize the minimum number of octets to represent the value.

##### "q" () Parameter

TODO. It is represented as the base64url encoding of the value's unsigned big endian representation as an octet sequence. The octet sequence MUST utilize the minimum number of octets to represent the value.

##### "g" () Parameter

TODO. It is represented as the base64url encoding of the value's unsigned big endian representation as an octet sequence. The octet sequence MUST utilize the minimum number of octets to represent the value.

##### "y" () Parameter

TODO. It is represented as the base64url encoding of the value's unsigned big endian representation as an octet sequence. The octet sequence MUST utilize the minimum number of octets to represent the value.

#### Parameters for DSA Private Keys

##### "x" (Private Exponent) Parameter

The `x` (private exponent) member contains the private exponent value for the DSA private key. It is represented as the base64url encoding of the value's unsigned big endian representation as an octet sequence. The octet sequence MUST utilize the minimum number of octets to represent the value.