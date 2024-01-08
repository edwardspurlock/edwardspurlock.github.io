---
layout: posts
title: Build a Google Firebase Web Application (Coursera)
date: 2022-04-30 12:32
author: edward
categories: blog Development, Google
slug: build-a-google-firebase-web-application-coursera
status: published
---



I want to improve my Firebase skills, so I've started th[e Build a Google Firebase Web Application](https://www.coursera.org/learn/build-a-google-firebase-webapp) course on Coursera.





I've had to fix a problem halfway through - the instructor's video shows one link in Firebase to "Database", but the current version of Firebase splits the options into two - "Firestore Database" and "Realtime Database".





Worse yet, the firebaseConfig object automatically created when setting up the project does not include a databaseURL property needed by firebase.initializeApp(firebaseConfig), causing the script to throw an error when run:





``` wp-block-code
[2022-04-30T17:23:35.218Z]  @firebase/database: FIREBASE FATAL ERROR: Can't determine Firebase Database URL.  Be sure to include databaseURL option when calling firebase.initializeApp().
```





I was able to figure out the problem - copy the URL from the Realtime Database page in the Firebase console:  
`https://<PROJECT_NAME>-default-rtdb.firebaseio.com/`





I then added it to the firebaseConfig object:





``` wp-block-code
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "<PROJECT_NAME>.firebaseapp.com",
  projectId: "<PROJECT_NAME>",
  storageBucket: "<PROJECT_NAME>.appspot.com",
  messagingSenderId: "12########06",
  appId: "1:12########06:web:84******************db",
  measurementId: "G-*********H",
  databaseURL: "https://<PROJECT_NAME>-default-rtdb.firebaseio.com/"
};
```





I then re-ran the script, and the database populated with the first test records. Woo-hoo!


