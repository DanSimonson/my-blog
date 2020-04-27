---
title: Reminder-list
date: "2020-04-26T22:12:03.284Z"
description: "Gentle introduction to Redux state management"
---

This blog is intended to be a gentle introduction into the use of Redux with React for state management. It is my intention for this post to be the first of a two part blog. First, I will introduce Redux basics and then the second blog will delve into Asynchronous API calls using Thunk.

### Three core concepts!

1.  The store holds the state of the application
2.  Actions are object that describe what happened, while action creator are functions that describe what happened.
3.  Reducers tie the (actions/actioncreators) and store together, changing the application state.

```javascript
let name = "dan"
```
