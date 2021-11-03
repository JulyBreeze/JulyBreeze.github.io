---
title: Firebase
date: 2019-04-18 14:55:15
tags: firebase
---

### Auth

Firebase Authentication integrates tightly with other Firebase services, and it leverages industry standards like OAuth 2.0 and OpenID Connect, so it can be easily integrated with your custom backend.

### Cloud Storage

Uploads and downloads are robust, they restart where they stopped.  
Cloud Storage stores your files in a Google Cloud Storage bucket, making them accessible through both Firebase and Google Cloud

### Real-time database

#### 1.Docs

Realtime database is NoSQL cloud-hosted database.  
Data is synced across all clients in realtime, and remains available when your app goes offline. All of your clients share one Realtime Database instance and automatically receive updates with the newest data.

Data is stored as JSON.

具有的特点和怎么实现的简要解释：  
1)Realtime  
Instead of typical HTTP requests, the Firebase Realtime Database uses data synchronization. Every time data changes, any connected device receives that update within milliseconds.

2)Offline  
Firebase apps remain responsive even when offline, because the Firebase Realtime Database SDK persists your data to disk locally. Once connectivity is reestablished, the client device receives any changes it missed, synchronizing it with the current server state.

3)Scale across multiple databases  
By splitting your data across multiple database instances in the same Firebase project.  
Streamline authentication with Firebase Authentication on your project and authenticate users across your database instances. Control access to the data in each database with custom Firebase Realtime Database Rules for each database instance.

#### 2
网上查到的有关realtime怎么实现的：  
It is a mix of NoSQL database, Publish/subscribe server, web-sockets server, client-side library:  
User subscribe to some data;  
A server keep track of those subscriptions, as soon a new subscription happen, some data is sent back as message;  
When one client writes some data, then the pub server(like redis) broadcast this update to all the other active subscriptions.

基本就是Push Model:  
Client-side update is transmitted to server without initialzing a new connection;  
Server pushes the update to other connected client side.

### 3.Online/offline status

The server monitors the connection. If at any point, the connection times out or is actively closed by the Realtime Database client, the server checks security a second time (to make sure the operation is still valid) and then invokes the event.

DB provides a special location at

```html
/.info/connected
```

which is updated every time the Firebase Realtime Database client's connection state changes.

这个special location不会在clients之间分享

```java
const connectedRef = firebase.database().ref(".info/connected");

connectedRef.on("value", (snapshot) => {
  if(snapshot.val() === true) {
    alert("connected");
  }
})
```

### 4.Listeners

Data stored in a Firebase Realtime Database is retrieved by attaching an **asynchronous listener** to a database reference.  
An event listener may receive several different events:

```java
//used to read a static snapshot of the contents at a given database path,
"value"

//used when retrieving a list of items from the database
//triggered once for each existing child and again every time a new child is added
"child_added"

//is triggered any time a child node is modified, including any modifications to descendants of the child node
"child_changed"
"child_removed"
```

Writes from a single client will always be written to the server and broadcast out to other users in-order.  
'value' events are always triggered last and are guaranteed to contain updates from any other events which occurred before that snapshot was taken.

### 5.使用

1.Data structure  
Data is stored as JSON objects.

Avoid nesting data: flatten data structure.  
If the data is instead split into separate paths, also called **denormalization**, it can be efficiently downloaded in separate calls, as it is needed.

2.Read/Write Data  
Firebase data is retrieved by attaching an asynchronous listener to a firebase.database.Reference.

1)Write

```java
//can save data to a specified reference
//or replacing any existing data at that path
firebase.database().ref('users').set({
  name: 'John',
  email: 'xxx@gmail.com'
});
```

2)Read & Listen for changes

Use 'value' event to read a static snapshot of the contents at a given path, as they existed at the time of the event.  
This method is triggered once when the listener is attached and again every time the data, including children, changes.

The listener receives a snapshot that contains the data at the specified location in the database at the time of the event. You can retrieve the data in the snapshot with the val() method.

```java
//triggered every time data changed
//snapshot contains all data at that location
xxxRef.on('value', (snapshot) => {
  console.log(snapshot.val());
})

//triggered once and then does not triggered again.
xxxRef.once('value').then((snapshot) => {
  //do something here
});
```

You can traverse into the snapshot by calling child() to return child snapshots.

3)Update  

4)Push  
push() enerates a new child location using a unique key and returns its Reference.

If you provide a value to push(), the value is written to the generated location.  
If you don't pass a value, nothing is written to the database and the child remains empty (but you can use the Reference elsewhere).

```java
.push().key; //get the key for a new data
```

5)Delete

```java
.remove(); //delete
```

6)Detach listener

```java
//with no arguments removes all listeners at that location.
.off();
```

```java
ref(path)
```