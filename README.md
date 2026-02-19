# JsTOTPAuthenticator

This web based TOTP Authenticators can import your encrytpted **Aegis** Authenticator database. https://getaegis.app/. There is a desktop app AVDA, but I wanted an OS agnostic version. This also fills the iOS gap.

It was mainly designed for Desktops, but it can be used on mobile.

Run at: 

- https://vpelss.github.io/JsTOTPAuthenticator/ 

- https://codepen.io/vpelss/pen/EayojzL

# Use

If you already have a saved encrytpted Aegis Authenticator database, just click the menu icon, paste the db in the text box and click Import.

If you have no backed up data, click the plus and enter your Authenticator data manually. The data can be found in the URL usuially found near the QR code. eg: otpauth://totp/Example:alice@google.com?secret=JBSWY3DPEHPK3PXP&issuer=Example

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

- generate a random 32 byte masterKey, a random 12 byte nonce value
-  encrypt to 'database' using window.crypto.subtle.encrypt and the masterKey, nonce, and plaintext you want encrypted
- set 'db' to 'database' , removing the last 16 bytes
- set 'tag' to the last removed 16 bytes mentioned previously

- 
-  //create a 12 byte nonce
    let nonce = new Uint8Array(12);
    self.crypto.getRandomValues(nonce);
    //create a 32 bit masterKey
    let masterKey = new Uint8Array(32);
    self.crypto.getRandomValues(masterKey);
    //create a 16 bit tag
    //let tag = new Uint8Array(16);
    //self.crypto.getRandomValues(tag);
    //convert localStorageObject to content Uint8Array
    let tempdb = JSON.stringify(dbGlobal, null, "\t"); //we don't need the formatting
    let content = TE.encode(tempdb);
    let database = await encrypt(nonce, masterKey, content);
   
    // database is an ArrayBuffer, not Uint8Array, so fix it
    database = new Uint8Array(database);
    //db is 16 bytes larger, the last 16 byte tag is removed and stored as the tag
    //  let database = concatenateUint8arrays([dbData, tag]);
    let length = database.length;
    let dbData = database.slice(0, length - 16);
    //let dbData = Uint8Array.fromBase64(db);
    let db = dbData.toBase64();
    //let db = localStorageObject.db;
    localStorageObject.db = db;
    //tag = Uint8Array.fromHex(tagString);
    let tag = database.slice(length - 16, length);
    let tagString = tag.toHex();
    //tagString = localStorageObject.header.params.tag;
    localStorageObject.header.params.tag = tagString;
    //nonce = Uint8Array.fromHex(nonceString);
    let nonceString = nonce.toHex();
    //nonceString = localStorageObject.header.params.nonce;
    localStorageObject.header.params.nonce = nonceString;

    //let saltString = localStorageObject.header.slots[0].salt;
    //let salt = Uint8Array.fromHex(saltString);
    let salt = new Uint8Array(32); //32 byte salt , 64 hex characters
    self.crypto.getRandomValues(salt);
    let saltString = salt.toHex();
    localStorageObject.header.slots[0].salt = saltString;

    let n = localStorageObject.header.slots[0].n;
    let r = localStorageObject.header.slots[0].r;
    let p = localStorageObject.header.slots[0].p;
    let dkLen = 32;
    let keyScrypt = await scrypt.scrypt(pw, salt, n, r, p, dkLen); //asyncronous

    //let nonceString = localStorageObject.header.slots[0].key_params.nonce;
    //nonceString = nonceString.normalize("NFKC");
    //let nonce = Uint8Array.fromHex(nonceString);
    self.crypto.getRandomValues(nonce); //new masterKey nonce
    nonceString = nonce.toHex();
    localStorageObject.header.slots[0].key_params.nonce = nonceString;
    //let masterKey = await decrypt(nonce, keyScrypt, keyData);
    let keyData = await encrypt(nonce, keyScrypt, masterKey);
    //keyDaya is 16 bytes bigger than masterKey so tag is that last 16 bytes and so masterKey the remaining first part
    //let keyData = concatenateUint8arrays([slotKey, tag]);
    keyData = new Uint8Array(keyData);
    length = keyData.length;
    let slotKey = keyData.slice(0, length - 16);
    //let slotKey = Uint8Array.fromHex(slotKeyString);
    let slotKeyString = slotKey.toHex();
    //let slotKeyString = localStorageObject.header.slots[0].key;
    localStorageObject.header.slots[0].key = slotKeyString;
    tag = keyData.slice(length - 16, length);
    //let tag = Uint8Array.fromHex(tagString);
    tagString = tag.toHex();
    //let tagString = localStorageObject.header.slots[0].key_params.tag;
    localStorageObject.header.slots[0].key_params.tag = tagString;

    //let encrypted = TD.decode(db);
    let encrypted = db;
    return encrypted;

Steps to Decrypt:

