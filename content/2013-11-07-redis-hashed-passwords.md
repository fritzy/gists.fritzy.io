---
date: 2013-11-07 13:00:00 PST
title: Redis Lua: Storing and Checking Hashed Passwords
tags: redis, lua
type: text/html
---

**See the update at the bottom: this approach isn't secure (but I still think it's interesting).**

Storing passwords in plain text is bad.
Time and time again, databases get hacked and passwords get leaked.
Hashing a password creates a consistent string from a password that you can store, but isn't the password, and isn't easily reversed.
Reversing a hash is difficult, and typically done with brute force if the hashing algorthim is fairly secure.

People end up creating giant lookup tables called "rainbow tables" of every possible string combination up to a certain number of characters in order to quickly reverse hashes.
The best defense against this is to hash something larger than a typical password because it takes exponential time to create a rainbow table for each additional character.
We do this with dynamic and static salts.
Salts are used to add length to the password string before hashing.
A static salt is global across the application, and a dynamic salt is unique for each user.

Awhile back ago, [@antirez](http://twitter.com/antirez) accepted a pull request for `redis.sha1hex()` in Redis's EVAL scripts.
This gives us a great way to store and check hashed passwords.

<script src="https://gist.github.com/fritzy/7361109.js"></script>
    
    > EVAL "setpassword script" 1 testpass bob "hi"
    "Password must be at least 8 characters."

    > EVAL "setpassword script" 1 testpass bob "hi there"
    (nil)

    > EVAL "checkpassword script" 1 testpass bob "hi there"
    1) (nil)
    2) (integer) 1

    > EVAL "checkpassword script" 1 testpass bob "wrong password"
    1) (nil)
    2) (nil)

There are a few things to note here.

* The schema for creating your pre-hashed string isn't important except that you need to be consistent.
* We could use anything as the dynamic salt, like the user's phone number, but you'd have to get the user's password and re-generate the hash everytime this information is changed.
* I'm using an error-first return pattern for these scripts. The first argument is falsey if there is no error, and a string if there is an error. This is a common JavaScript pattern, and useful in Redis Lua scripts.
* The password length check is actually counting bytes.

## Update:

Again, [Pierre Chapuis @pchapuis](http://twitter.com/pchapuis) has some good feedback.
He references [http://codahale.com/how-to-safely-store-a-password/](http://codahale.com/how-to-safely-store-a-password/) which argues against using typical hashing algorithms for hashing.
Since someone who has your redis db also has your Lua scripts, the salts are rather available to the cracker, even the dynamic ones.

We can easily argue that bcrypt/scrypt is the better way to go.
Unfortunately, it's not available in Redis, but that's not a terribly valid reason to discount it.

This approach is much better than nothing, but perhaps outdated.

As I'm writing this, [Alex Gaynor @alex\_gaynor](http://twitter.com/alex_gaynor) just gave the [same advice](https://twitter.com/alex_gaynor/status/398584652811628544).
