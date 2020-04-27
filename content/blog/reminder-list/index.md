---
title: Reminder-list
date: "2020-04-26T22:12:03.284Z"
description: "Gentle introduction to Redux state management"
---

[Demo](https://nostalgic-perlman-df8a29.netlify.app/)

[Github Code](https://github.com/DanSimonson/reminder-list)

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

Actions always return a 'type,' and one can think of this as a type of action.Optional payload data can also be passed into the action to further define what it does.

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

We accomplish this by using the spread operator '...' in one case to create a copy and load the state with the new copy and with the filter method in the second case that creates a whole new array after filtering out the deleted item.

Now that that we have Reducers that change the state we store that state in - you guessed it - the store.

Store.js

```javascript
import { createStore } from "redux"
import ReminderReducer from "./ReminderReducer"

const Store = createStore(ReminderReducer)

export default Store
```

The store is provided but one more step is needed for react to use it. We must pass it into one of our root files as a prop so React knows about it. In this case, I pass the react-redux Provider functionality into the App.js file.

```javascript
import React from "react"
import "./App.css"
import ReminderContainer from "./Components/ReminderContainer"
import { Provider } from "react-redux"
import Store from "./Reminders/Store"

function App() {
  return (
    <Provider store={Store}>
      <div className="App">
        <ReminderContainer />
      </div>
    </Provider>
  )
}

export default App
```

The store is created and React sees it now. but we are not finished. We need to get the state so we can use it and we need to be able to dispatch an actions to set the state when we need it changed.

All that work happens in our ReminderContainer.js file:

```javascript
import React, { useState, useEffect } from "react"
import { connect } from "react-redux"
import { addReminder, deleteReminder } from "../Reminders/ReminderActions"
import "./ReminderContainer.css"

function ReminderContainer(props) {
  const [text, setText] = useState("")
  const buttonOne = (
    <button
      className="Btn"
      type="button"
      onClick={() => {
        props.addReminder(text)
        setText("")
      }}
      style={{ marginTop: "25px" }}
    >
      Add
    </button>
  )
  const buttonTwo = (
    <button className="Btn" type="button" style={{ marginTop: "25px" }}>
      Add
    </button>
  )

  const onChangeReminderText = e => {
    setText(e.target.value)
  }
  //style={{ marginLeft: "25px" }} // style={{ width: "100%" }}
  const reminderList = props.reminders.length ? (
    props.reminders.map(todo => {
      return (
        <div key={todo.id} className="output">
          <p>{todo.text}</p>
          <button
            className="Btn positionAbsolute"
            onClick={() => props.deleteReminder(todo.id)}
          >
            Delete
          </button>
        </div>
      )
    })
  ) : (
    <p className="center">No Reminders</p>
  )
  return (
    <div className="divContainer">
      <div className="picDiv">
        <img
          src="https://res.cloudinary.com/dmglopmul/image/upload/v1587687687/projectPhotos/blogs/Honey-Do-List2.jpg"
          alt="Honey Do"
        />
      </div>
      <div className="reminderInputDiv">
        <input
          onChange={onChangeReminderText}
          value={text}
          type="text"
          className="honeyInput"
        ></input>
        {text !== "" ? buttonOne : buttonTwo}
      </div>
      <div className="wrapper">
        {/*<h2 className="title">Honey Do List</h2>*/}
        <div className="reminderListDiv">{reminderList}</div>
      </div>
    </div>
  )
}

const mapStateToProps = state => {
  return {
    reminders: state,
  }
}

const mapDispatchToProps = dispatch => {
  return {
    addReminder: text => dispatch(addReminder(text)),
    deleteReminder: id => dispatch(deleteReminder(id)),
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(ReminderContainer)
```

for React and Redux to 'connect,' we need to import that functionality. This is accomplished by importing - you guessed it - 'connect' from react-redux.

we use it at the bottom of the page and it does just what its name implies: it connects redux to react.

Two more built in react-redux functions allow us to do the crucial work of getting everything to work.

Getting the state is accomplished with mapStateToProps that permits us to use the state as a prop in this file.

The mapDispatchToProps allows us to dispatch the actions to then alter the state.

Whew, it's kind of complicated, but you will get it. Practice this tutorial a few times and try modifying it on your own.

‘Tell me and I forget. Teach me and I remember. Involve me and I learn'
–Benjamin Franklin
