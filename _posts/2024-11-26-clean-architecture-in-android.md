---
title: Clean architecture in Android
date: 2024-11-19 20:30:00
categories: []
tags: [architecture]
---

```mermaid
flowchart LR
  Domain ~~~ Data --> Domain
  Data ~~~ Domain

  UI-Layer --> Domain

```
