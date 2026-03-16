# AuthenticatorAnywhere
 
This Web based TOTP Authenticator can be used on **any device** that has a **modern** web browser. Chrome, Edge, Safari.

Import, export and share your database between **all** your devices. The database format is compatible with the **Aegis** Authenticator database. https://getaegis.app/.

It was designed for Desktops, but it can be used on mobile.

It runs entirely in your browser using Javascript. 

**The database is encrypted and stored in your browser's memory. If you clear your cache, or reinstall your browser, your database is gone. Please create/export a backup and keep it in a safe place.**

It also shows the next upcoming code. A nice old man feature!

More features are listed here: https://www.emogic.com/tech-notes/authenticator-anywhere/

Modify the code at https://codepen.io/vpelss/pen/EayojzL

# Run It

https://vpelss.github.io/AuthenticatorAnywhere/

# Use

If you already have a saved encrypted Aegis Authenticator database, just click the menu icon, paste the db in the text box and click Import.

If you have no backed up data, click the plus and enter your Authenticator data manually, scan a QR code or import an image of a QR code. The data for manually entering can be found in the URL link found near the QR code. eg: otpauth://totp/Example:alice@google.com?secret=JBSWY3DPEHPK3PXP&issuer=Example

Should you need to set or change your password, click the menu and click 'Change Password' after you have encrypted the database.

# Tips

- Use a strong password
- Back up your database and keep it off site
- Close AutheticatorAnywhere when you are done
- Do not keep the password in a password manager. If password manager is compromised, then your authenticator line of defense might fail too

# What is TOTP Authentication?

https://www.youtube.com/watch?v=7sI2PJaMCAk

# Code explanation : TOTP

For those trying to write their own TOTP code, I tried to follow the RFCs:

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
 
- I am using window.crypto.subtle.digest for my SHA1 code, but there are libraries if you need to support old browsers

- For Uint8Array concatenation I used https://evanhahn.com/the-best-way-to-concatenate-uint8arrays/

- after completing all my code based on rfc6238 I found out that modern browsers have window.crypto.subtle.sign("HMAC", cryptoKey, msg) , so now my many lines of code can be replaced by 3 lines. Ha on me! My code is left in the code's comments.

# Code explanation : Unencrypting and Encrypting the json database

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
- using settings n=32768, r =8, p=1, dkLen=32 set 'hashedPassword' from scrypt.scrypt(password, salt, n, r, p, dkLen) : you provide the password
- note: scrypt is intended to make automated password attempts onerous. 'hashedPassword' is a **heavily** hashed password
- generate a random 12 byte 'nonce2' value 
- set 'key' from window.crypto.subtle.encrypt using 'nonce2', 'hashedPassword', 'masterKey' : We are encrypting the 'masterKey' used to encrypt/decrypt our initial 'plainext'. It is encrypted with a hashed version of our password and a nonce value
- note: the encrypted 'key' is 16 bytes longer than 'masterKey'
- set 'slotKey' to a value equal to 'key' minus the last 16 bytes
- set 'tag2' to the last removed 16 bytes mentioned previously
- store 'nonce2', 'salt', 'tag2', 'slotKey' in the JSON header.slots

Steps to Decrypt:

Short version: Unencrypt the 'masterKey' using your password (hashed) and the nonce used to encrypt it. Then unencrypt the 'plaintext' using the 'masterKey' and the nonce used to encrypt it.

- get public 'nonce2', 'salt', 'tag2', 'slotKey' values in the JSON header.slots
- using settings n=32768, r =8, p=1, dkLen=32 set 'hashedPassword' from scrypt.scrypt(password, salt, n, r, p, dkLen) : you provide the password
- note: 'hashedPassword' is a **heavily** hashed password
- set 'key' by concatenating 'slotKey' + 'tag2'
- set 'masterKey' using decrypt(nonce2, hashedPassword, key);
- get public 'nonce', 'db', and 'tag' in the JSON header.params
- set 'database' by concatenating 'db' + 'tag'
- Note: the tag is used to prove that the encrypted database was not altered
- set 'plaintext' from window.crypto.subtle.decrypt using the nonce, masterKey, and 'database' you want decrypted


