---
title: Reminder-list
date: "2020-04-26T22:12:03.284Z"
description: "Gentle introduction to Redux state management"
---

This blog is intended to be a gentle introduction into the use of Redux with React for state management. It is my intention for this post to be the first of a two part blog. First, I will introduce Redux basics and then the second blog will delve into Asynchronous API calls using Thunk.

### Three core concepts!

1.  The store holds the state of the application
2.  Actions are object that describe what happened, while action creators are functions that describe what happened.
3.  Reducers tie the (actions/actioncreators) and store together, changing the application state.

our application folder structure look like this:

```javascript
Reminder-list
  Components
    ReminderContainer.css
    ReminderContainer.js
  Reminders
    ReminderActions.js
    ReminderReducer.js
    ReminderTypes.js
    Store.js
  App.css
  App.js
  index.css
  index.js

  ...

```

Although not absolutely neccessary, it is the custom to define our action types as constants with uppercase style in a separate file to avoid bugs.

here is our ActionTypes.js file which we export individually:

```javascript
export const ADD_REMINDER = "ADD_REMINDER"
export const DELETE_REMINDER = "DELETE_REMINDER"
```

Then we make our ReminderActions.js file. The difference between plain actions and actioncreators is that actions are objects and actioncreators are functions that return objects. ActionCreators are useful because they allow for the use of asynchronous code if it is needed.

Actions always return a'type,' and one can think of this as a type of action.Optional payload data can also be passed in to the action to further define what it does.

ReminderActions.js file:

```javascript
import { ADD_REMINDER, DELETE_REMINDER } from "./ReminderTypes"

let TodoId = 1

export const addReminder = text => ({
  type: ADD_REMINDER,
  id: TodoId++,
  text: text,
})

export const deleteReminder = id => ({
  type: DELETE_REMINDER,
  id: id,
})
```

We pass in 'text' and 'id' along with the type of actions. These payloads will be used to alter the state in the reducer.

Now we define our Reducers, which will alter the state of the application and serve as a way point between our actions and soon to be created store, where the state is stored.

ReminderReducer.js

```javascript
import { ADD_REMINDER, DELETE_REMINDER } from "./ReminderTypes"

const INITIAL_DATA = []

const ReminderReducer = (state = INITIAL_DATA, action) => {
  switch (action.type) {
    case ADD_REMINDER:
      return [
        ...state,
        {
          id: action.id,
          text: action.text,
        },
      ]
    case DELETE_REMINDER:
      const numIndex = parseInt(action.id)
      return state.filter(reminder => reminder.id !== numIndex)
    default:
      return state
  }
}

export default ReminderReducer
```

It's important to note that the reducers must be pure functions because the state must be immutable (it can't be changed).

We accomplish this by using the spread operator '...' in one case to create a copy and load the state with a new copy and with the filter method in the second case that creates a whole new array after filtering out the deleted item.
