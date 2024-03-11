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

Our endgoal is to access /api/v1/flag endpoint with administrator JWT token. Let us now look at the source code of challenge.

```python
from flask import jsonify, current_app
import python_jwt as jwt, datetime
import json
import os

from app.middleware.middleware import *
from . import api_blueprint

JSON_DIR = os.path.join(os.path.dirname(__file__), 'json')

@api_blueprint.route('/get_ticket', methods=['GET'])
def get_ticket():

    claims = {
        "role": "guest", 
        "user": "guest_user"
    }
    
    token = jwt.generate_jwt(claims, current_app.config.get('JWT_SECRET_KEY'), 'PS256', datetime.timedelta(minutes=60))
    return jsonify({'ticket: ': token})


@api_blueprint.route('/chat/<int:chat_id>', methods=['GET'])
@authorize_roles(['guest', 'administrator'])
def chat(chat_id):

    json_file_path = os.path.join(JSON_DIR, f"{chat_id}.json")

    if os.path.exists(json_file_path):
        with open(json_file_path, 'r') as f:
            chat_data = json.load(f)
        
        chat_id = chat_data.get('chat_id', None)
        
        return jsonify({'chat_id': chat_id, 'messages': chat_data['messages']})
    else:
        return jsonify({'error': 'Chat not found'}), 404


@api_blueprint.route('/flag', methods=['GET'])
@authorize_roles(['administrator'])
def flag():
    return jsonify({'message': current_app.config.get('FLAG')}), 200
```
Our goal is to exploit the /view endpoint 

We also find "countries.yaml" file which have some data related to countries. First country named "Flagistan" is interesting and our end goal. Note the entries in the deny list of Flagistan country.

Sent GET request here:
/view?country=Flagistan

Seems like, we have to modify a cookie. Set "iso" to anything except the values in the deny list of Flagistan, refresh it and we have the flag. 

![img](https://i.imgur.com/kOIp2ZK.jpeg)


Thankyou for Reading

