# JsTOTPAuthenticator

I tried to follow the RFCs:

TOTP: Time-Based One-Time Password Algorithm: https://datatracker.ietf.org/doc/html/rfc6238

HOTP: An HMAC-Based One-Time Password Algorithm: https://datatracker.ietf.org/doc/html/rfc4226 more notably https://datatracker.ietf.org/doc/html/rfc4226#section-5.3

HMAC: Keyed-Hashing for Message Authentication: https://datatracker.ietf.org/doc/html/rfc2104

Wikipedia was much more helpful: 

Time-based one-time password: https://en.wikipedia.org/wiki/Time-based_one-time_password

HMAC: https://en.wikipedia.org/wiki/HMAC

Notes:

- Your authentication secrets are in Base32 RFC 4648 and will need to be converted to a Hexidecimal string. I modified https://mojoauth.com/binary-encoding-decoding/base32-with-javascript-in-browser#encoding-data-to-base32 to do this

  - https://en.wikipedia.org/wiki/Base32

  - https://datatracker.ietf.org/doc/html/rfc4648#section-6
 
- I am using window.crypto.subtle.digest for my SHA1 code as most older browsers will likely be dead or dying

- For Uint8Array concatination I used https://evanhahn.com/the-best-way-to-concatenate-uint8arrays/
