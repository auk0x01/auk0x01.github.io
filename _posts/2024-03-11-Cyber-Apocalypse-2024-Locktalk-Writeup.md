---
title: HTB Cyber Apocalypse 2024 (LockTalk) Writeup
date: 2024-03-11
categories: [ctf, web]
tags: [cyberapocalypse, web, ctf]    # TAG names should be lowercase
---

I solved LockTalk web challenge from HTB CyberApocalypse 2024 and here is the writeup for it.

# Challenge:

We are given a page showing different endpoints.

![img](https://i.imgur.com/ccjYIMa.png)

We find a JS file "analytics.js" which seems to have been Obfuscated. De-obfuscated JS code from here: https://obf-io.deobfuscate.io/ and then simply pasted JS code on my browser and made alert popup with flag.

# Challenge #2 (flaglang):

We have a NodeJS server with following code:
```js
const crypto = require('crypto');
const fs = require('fs');
const path = require('path');
const express = require('express');
const cookieParser = require('cookie-parser');
const yaml = require('yaml');

const yamlPath = path.join(__dirname, 'countries.yaml');
const countryData = yaml.parse(fs.readFileSync(yamlPath).toString());
const countries = new Set(Object.keys(countryData));
const countryList = JSON.stringify(btoa(JSON.stringify(Object.keys(countryData))));

const isoLookup = Object.fromEntries([...countries].map(name => [
  countryData[name].iso,
  {...countryData[name], name }
]));


const app = express();

const secret = crypto.randomBytes(32).toString('hex');
app.use(cookieParser(secret));

app.use('/assets', express.static(path.join(__dirname, 'assets')));

app.get('/switch', (req, res) => {
  if (!req.query.to) {
    res.status(400).send('please give something to switch to');
    return;
  }
  if (!countries.has(req.query.to)) {
    res.status(400).send('please give a valid country');
    return;
  }
  const country = countryData[req.query.to];
  if (country.password) {
    if (req.cookies.password === country.password) {
      res.cookie('iso', country.iso, { signed: true });
    }
    else {
      res.status(400).send(`error: not authenticated for ${req.query.to}`);
      return;
    }
  }
  else {
    res.cookie('iso', country.iso, { signed: true });
  }
  res.status(302).redirect('/');
});

app.get('/view', (req, res) => {
  if (!req.query.country) {
    res.status(400).json({ err: 'please give a country' });
    return;
  }
  if (!countries.has(req.query.country)) {
    res.status(400).json({ err: 'please give a valid country' });
    return;
  }
  const country = countryData[req.query.country];
  const userISO = req.signedCookies.iso;
  if (country.deny.includes(userISO)) {
    res.status(400).json({ err: `${req.query.country} has an embargo on your country` });
    return;
  }
  res.status(200).json({ msg: country.msg, iso: country.iso });
});

app.get('/', (req, res) => {
  const template = fs.readFileSync(path.join(__dirname, 'index.html')).toString();
  const iso = req.signedCookies.iso || 'US';
  const country = isoLookup[iso];
  res
    .status(200)
    .type('html')
    .send(template
      .replaceAll('$msg$', country.msg)
      .replaceAll('$name$', country.name)
      .replaceAll('$iso$', country.iso)
      .replaceAll('$countries$', countryList)
    );
});

app.listen(3000);
```
Our goal is to exploit the /view endpoint 

We also find "countries.yaml" file which have some data related to countries. First country named "Flagistan" is interesting and our end goal. Note the entries in the deny list of Flagistan country.

Sent GET request here:
/view?country=Flagistan

Seems like, we have to modify a cookie. Set "iso" to anything except the values in the deny list of Flagistan, refresh it and we have the flag. 

![img](https://i.imgur.com/kOIp2ZK.jpeg)


Thankyou for Reading

