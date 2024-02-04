---
title: Malbuster - TryHackme Writeup
date: 2023-02-09 10:03
categories: [ctf, web]
tags: [dicectf, web, ctf]    # TAG names should be lowercase
---

# Intro

So, I got time to solve one web challenge of DiceCTF2024. The name of the challenge is funnylogin. I loved the challenge as it required a little creativity. It took me 1 hour to solve this.

# Challenge

So, we are given a login page. Looking at the source code, we see that this challenge is about SQLI.
```js
const express = require('express');
const crypto = require('crypto');

const app = express();

const db = require('better-sqlite3')('db.sqlite3');
db.exec(`DROP TABLE IF EXISTS users;`);
db.exec(`CREATE TABLE users(
    id INTEGER PRIMARY KEY,
    username TEXT,
    password TEXT
);`);

const FLAG = process.env.FLAG || "dice{test_flag}";
const PORT = process.env.PORT || 3000;

const users = [...Array(100_000)].map(() => ({ user: `user-${crypto.randomUUID()}`, pass: crypto.randomBytes(8).toString("hex") }));
db.exec(`INSERT INTO users (id, username, password) VALUES ${users.map((u,i) => `(${i}, '${u.user}', '${u.pass}')`).join(", ")}`);

const isAdmin = {};
const newAdmin = users[Math.floor(Math.random() * users.length)];
isAdmin[newAdmin.user] = true;

app.use(express.urlencoded({ extended: false }));
app.use(express.static("public"));

app.post("/api/login", (req, res) => {
    const { user, pass } = req.body;

    const query = `SELECT id FROM users WHERE username = '${user}' AND password = '${pass}';`;
    try {
        const id = db.prepare(query).get()?.id;
        if (!id) {
            return res.redirect("/?message=Incorrect username or password");
        }

        if (users[id] && isAdmin[user]) {
            return res.redirect("/?flag=" + encodeURIComponent(FLAG));
        }
        return res.redirect("/?message=This system is currently only available to admins...");
    }
    catch {
        return res.redirect("/?message=Nice try...");
    }
});

app.listen(PORT, () => console.log(`web/funnylogin listening on port ${PORT}`));
```

First, application is generating 100,000 records of random-generated UUID and random password. Then these values are being inserted into database. There is another column named "id" being inserted in the database. This "id" column is not random and is just an incrementation value of all records like "1,2,3,4....n".

![img](https://i.imgur.com/vTzqDMH.png)

## **Question#7** ##

7) *Using the hash of malbuster_3, what is its malware signature based on abuse.ch?*

abuse.ch is also a online platform containing malware database where you can search for malware samples using hashes or malware family.

Generated md5 hash of *malbuster_3* and uploaded it here:
[https://bazaar.abuse.ch/browse/](https://bazaar.abuse.ch/browse/)

Upon searching on abuse.ch, I found malware signature for *malbuster_3*.

![img](https://i.imgur.com/8Av161p.png)






Thankyou for Reading
