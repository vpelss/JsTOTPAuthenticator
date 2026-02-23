# JsTOTPAuthenticator

This web based TOTP Authenticators can be used on **any** device that has a modern web browser.

You can easily import, export and share your database between **all** your devices. The database format is compatable with the **Aegis** Authenticator database. https://getaegis.app/. So you can still use that.

It was designed for Desktops, but it can be used on mobile.

It also shows the next upcomming code. An nice old man feature!

Run at: 

- https://vpelss.github.io/JsTOTPAuthenticator/ 

- https://codepen.io/vpelss/pen/EayojzL

# Use

If you already have a saved encrytpted Aegis Authenticator database, just click the menu icon, paste the db in the text box and click Import.

If you have no backed up data, click the plus and enter your Authenticator data manually. The data can be found in the URL usually found near the QR code. eg: otpauth://totp/Example:alice@google.com?secret=JBSWY3DPEHPK3PXP&issuer=Example

I do plan to implimemt a paste the url feature in the near future, and possibly copy and paste a qr code option.

Should you need to set or change your password, click the menu and click 'Change Password' after you have unecryped the databse.

# TOTP

For those trying to write thier own TOTP code, I tried to follow the RFCs:

TOTP: Time-Based One-Time Password Algorithm: https://datatracker.ietf.org/doc/html/rfc6238

HOTP: An HMAC-Based One-Time Password Algorithm: https://datatracker.ietf.org/doc/html/rfc4226 more notably https://datatracker.ietf.org/doc/html/rfc4226#section-5.3

HMAC: Keyed-Hashing for Message Authentication: https://datatracker.ietf.org/doc/html/rfc2104

But wikipedia was much more helpful: 

Time-based one-time password: https://en.wikipedia.org/wiki/Time-based_one-time_password

HMAC: https://en.wikipedia.org/wiki/HMAC

Notes:

- Authentication secrets are in Base32 RFC 4648 and will need to be converted to a Hexidecimal string. I modified https://medium.com/@at.kishor.k/demystifying-base32-an-in-depth-guide-to-this-encoding-standard-697b9426fc25 to do this

- https://en.wikipedia.org/wiki/Base32

- https://datatracker.ietf.org/doc/html/rfc4648#section-6
 
- I am using window.crypto.subtle.digest for my SHA1 code as most older browsers will likely be dead or dying

- For Uint8Array concatination I used https://evanhahn.com/the-best-way-to-concatenate-uint8arrays/

- after completing all my code based on rfc6238 I found out that modern browsers have window.crypto.subtle.sign("HMAC", cryptoKey, msg) , so now my many lines of code can be replaced by 3 lines. Ha on me! Well at least I learned a few things.

# Unencrypting and Encrypting the Aegis json database

Steps to Encrypt:

Short version: Using a random 'nonce' and 'masterKey value encypt your 'plaintext'. Then encrypt your 'masterKey' using your password (hashed) and a second nonce. Store all non 'plaintext' values in the JSON.

- generate a random 32 byte 'masterKey', and a random 12 byte 'nonce' value
- encrypt to 'database' using window.crypto.subtle.encrypt using the 'nonce', 'masterKey', and 'plaintext' you want encrypted
- note: the encrypted text is 16 bytes longer than the 'plaintext'
- set 'db' to a value of 'database' minus the last 16 bytes
- set 'tag' to the last removed 16 bytes mentioned previously
- Note: that tags are only used to authenticate that the encrypted data was not tampered with 
- store 'nonce', 'db', and 'tag' in the JSON header.params
- generate a random 32 byte 'salt' value
- using settings n=32768, r =8, p=1, dkLen=32 set 'hashedPassword' from scrypt.scrypt(password, salt, n, r, p, dkLen) : you proivide the password
- note: scrypt is intended to make automated password attempts onerous. 'hashedPassword' is a **heavily** hashed password
- generate a random 12 byte 'nonce2' value 
- set 'key' from window.crypto.subtle.encrypt using 'nonce2', 'hashedPassword', 'masterKey' : We are encrypting the 'masterKey' used to encrypt/decrypt our initial 'plainext'. It is encypted with a hashed version of our password and a nonce value
- note: the encrypted 'key' is 16 bytes longer than 'masterKey'
- set 'slotKey' to a value equal to 'key' minus the last 16 bytes
- set 'tag2' to the last removed 16 bytes mentioned previously
- store 'nonce2', 'salt', 'tag2', 'slotKey' in the JSON header.slots

Steps to Decrypt:

Short version: Unencrypt the 'masterKey' using your password (hashed) and the nonce used to encrypt it. Then unencrypt the 'plaintext' using the 'masterKey' and the nonce used to encrypt it.

- get public 'nonce2', 'salt', 'tag2', 'slotKey' values in the JSON header.slots
- using settings n=32768, r =8, p=1, dkLen=32 set 'hashedPassword' from scrypt.scrypt(password, salt, n, r, p, dkLen) : you proivide the password
- note: 'hashedPassword' is a **heavily** hashed password
- set 'key' by concatinating 'slotKey' + 'tag2'
- set 'masterKey' using decrypt(nonce2, hashedPassword, key);
- get public 'nonce', 'db', and 'tag' in the JSON header.params
- set 'database' by concatinating 'db' + 'tag'
- Note: the tag is used to prove that the encrypted database was not altered
- set 'plaintext' from window.crypto.subtle.decrypt using the nonce, masterKey, and 'database' you want decrypted
