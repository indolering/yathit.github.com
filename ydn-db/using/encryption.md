---
layout: article
title:  "Encryption"
date:   2011-3-3
categories: ydn-db using
tag: article
---


## Tansparent Data Encryption ##

Browser database are mainly secured by [single origin policy](http://www.w3.org/Security/wiki/Same_Origin_Policy), such that only your web application can access the database. This is good enough for most web application. However, if user OS is compromised or physically accessible by intruders, browser database has no security provision. Transparent data encryption is a well-known technology provided by YDN-DB that offers encryption of data at rest. This may provide the ability to comply with many laws, regulations, and guidelines established in various industries, when caching sensitive data on client side.

### Enabling store wise encryption ###

Application must provide cryptographically strong pseudo-random encryption key on opening the database for encryption. Use standard symmetric key generation library available on your server. Do not persist encryption key on client, but always transfer over SSL after user login. The key should be generated per user basic.

Encryption key(s) are provided as `Encryption` field on database options. Encryption is enabled on store-wise using same encryption key for whole database. To enable encryption a store, set `encrypted` to `true` on its store schema.

Encrypted store must use out-of-line key and must not use key generator as illustrated in following schema. The following code snippets can be run on browser dev console.

    var schema = {
       version: 1,
       stores: [{
         name: 'encrypt-store',
         encrypted: true
       }]
     };
     var options = {
       Encryption: {
         expiration: 1000*15, //  optional data expiration in ms.
         secrets: [{
           name: 'aaaa',
           key: 'aYHF6vfuGHpfWS*eRLrPQxZjSó~É5c6HjCscqDqRtZasp¡JWSMGaW'
         }]
       }
     };

     var db = new ydn.db.Storage('encrypted-db-name', schema, options);

Record key is hashed with SHA256 function using given encryption key. The field, `secrets`, has array of encryption `key` and `name`. A short key name is keep along with the unencrypted record for later used. Record value is encrypted with RC4 streaming cipher using per-record 64-bit random salt and encryption key. AES and CBC cipher is also available. See [goog.crypt.Sha256](http://docs.closure-library.googlecode.com/git/class_goog_crypt_Sha256.html) and [goog.crypt.Arc4](http://docs.closure-library.googlecode.com/git/class_goog_crypt_Arc4.html) for detail implementation.

You may also set `expiration` interval in milli-seconds to let record to be expired. However noted that the data itself does not expired. The expiration time is a meta data to check on retrieval. If it expires, the record is removed from the database. These meta data can be manipulated if encryption is successful.

Inside database, encrypted record value has four meta data 1) key name 2) salt 3) creation timestamp 4) expiration time and encrypted value. Since record key is hashed, both encryption key and record key is required to retrieve a correct record.

## Using encryption ##

Once enabled encryption, record is encrypted on write and decrypted back on read. Encryption and decryption is performed only on CRUD database operations. Database query, such as `keys`, `values`, returns unencrypted values.

    var value = {msg: 'Hello world'};
    var id = 'id1';
    db.put('encrypt-store', value, id); // value and record_id encrypted on database
    db.values('encrypt-store').done(function(values) {
        console.log(values); // unencrypted values
    });
    db.get('encrypt-store', id).done(function(value) {
        console.log(value); // decrypted record value
    });

Since data is encrypted, database indexing is not available.

In this example, you will also find that record can no longer be retrieved after expiration of 15 seconds.

Note that enabling encryption itself, does not encrypt data.

## Encryption key rotation ##

With symmetric encryption, the level of confidentiality depends on the protection of the keys. To reduce exposure of key, encryption keys are updated periodically.

Multiple encryption keys can be provided. Only the first key is used for encryption. All keys are used for decryption until success. Decryption is deem considered success if require key is available and encrypted value can be parsed by JSON parser.

See [demo application](http://yathit.github.io/sample-encryption/index.html) to try out.

Additional source code is available upon request.