---
title: Jetpack Datastore in Android
date: 2025-02-19 21:09:01
categories: []
tags: [android, local storage]
---
Jetpack DataStore is a data storage solution that allows you to store key-value pairs or typed objects with protocol buffers. It's based on **Kotlin coroutines and Flow** to store asynchronousy, consistenly (thread-safe and non-blocking).

DataStore provides two different implementations: **Preferences DataStore** and **Proto DataStore**.
- **Preferences DataStore** stores and accesses data using keys. This implementation does not require a predefined schema, and it does not provide type safety.
- **Proto DataStore** stores data as instances of a custom data type. This implementation requires you to define a schema using protocol buffers, but it provides type safety.

- type safety
- preferencesdatastore vs protodatastore
- compare datstore vs sharepreferences
