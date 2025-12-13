---
title: "CGRateS Tutorial Series: Installation - Mongo DB Server Install - Debian 8 - Part 1"
date: 2017-11-02
categories:
  - Howtos
tags:
  - cgrates
  - mongodb
  - debian
draft: false
---
Below are the steps to prepare a stand alone mongodb server for cgrates on Debian 8 jessie

**Install sudo**

```console
# apt-get install sudo
```

**Import key**

```console
# apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6 
```

**Add repository URL**

```console
# echo "deb http://repo.mongodb.org/apt/debian jessie/mongodb-org/3.4 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```

**Update and Install**
```console
# apt-get update 

# apt-get install -y mongodb-org
```

**Enable and Start**

```console
# systemctl enable mongod.service 

# systemctl start mongod
```

**Setting up CGRates User**


Create a file cgrates_user.js with the following content

```js
db = db.getSiblingDB('cgrates')
db.createUser(
 {
 user: "cgrates",
 pwd: "CGRateS.org",
 roles: [ { role: "dbAdmin", db: "cgrates" } ]
 }
)
```
Run the following command to create the user
```console
# mongo create_user.js
```
Succesful output will be as follows:

```console
MongoDB shell version v3.4.10
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.10

Successfully added user: {Successfully added user: { "user" : "cgrates", "roles" : [ { "role" : "dbAdmin", "db" : "cgrates" } ]}
```

By default mongo server will only listen to localhost, modify that in** /etc/mongod.conf**

Quick Restart and you're good to go!
```console
# systemctl restart mongod
```
