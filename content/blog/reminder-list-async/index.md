---
title: Reminder-List-Async
date: "2020-04-29T22:12:03.284Z"
description: "React, redux with asynchrounous api calls using Thunk"
---

![](./thunk.jpg)

[Demo](https://quirky-wozniak-117e30.netlify.app//)

[Github Code](https://github.com/DanSimonson/reminder-list-async)

I will focus on code related closely to our primary subject- thunk. If you are unsure about some of the code as it relates to redux, you can refer to the prior blog about redux as it follows the same pattern as this tutorial.

```javascript
state = {
  loading: "",
  data: [],
  error: "",
}
```

- loading - used to display message when page is loading
- data - holds the reminders
- error = contains the error message

The state properties allows us to hold the reminders returned from the api, display a loading message until they are returned and finally give an error message if something goes wrong.

Next, we will discuss the actions and reducers. The first action's result will determine which of the final two actions are implemented.

If the **GET REMINDER REQUEST** action is successful we will set loading to true and then call the **GET REMINDER SUCCESS** action/reducer, set loading to false, and load the reminders into the payload. If something goes awry, we will we will set loading to true and call the **GET REMINDER FAILURE** action/reducer, set loading to false, and display an error message from the api.

Inside ReminderActions.js

```javascript
import axios from "axios"
import {
  GET_REMINDER_REQUEST,
  GET_REMINDER_FAILURE,
  GET_REMINDER_SUCCESS,
} from "./ReminderTypes"

export const getReminderRequest = () => {
  return {
    type: GET_REMINDER_REQUEST,
  }
}

const getReminderSuccess = reminders => {
  return {
    type: GET_REMINDER_SUCCESS,
    payload: reminders,
  }
}

const getReminderFailure = error => {
  return {
    type: GET_REMINDER_FAILURE,
    payload: error,
  }
}
```

.

Inside ReminderReducers.js

```javascript
import React from "react"
import {
  GET_REMINDER_REQUEST,
  GET_REMINDER_FAILURE,
  GET_REMINDER_SUCCESS,
} from "./ReminderTypes"

const initialState = {
  loading: true,
  reminders: [],
  error: "",
}

const ReminderReducer = (state = initialState, action) => {
  switch (action.type) {
    case GET_REMINDER_REQUEST:
      return {
        ...state,
        loading: true,
      }
    case GET_REMINDER_SUCCESS:
      return {
        loading: false,
        reminders: action.payload,
        error: "",
      }
    case GET_REMINDER_FAILURE:
      return {
        loading: false,
        reminders: [],
        error: action.payload,
      }
    default:
      return state
  }
}
```

.

Now we will explain the asynchronous part of our application.

To better track our asynchronous actions we have npm installed and added a chrome devtools extension called **composeWithDevTools** using redux-devtools-extension. [For further instructions see redux-devtools-extension documentation](https://github.com/zalmoxisus/redux-devtools-extension)

Then we npm install axios and redux-thunk. We need to apply our redux thunk middleware from redux-thunk and use applyMiddleware from redux to connect asynchronous functionality to our application.

Inside Store.js

```javascript
import { createStore, applyMiddleware } from "redux"
import { composeWithDevTools } from "redux-devtools-extension"
import thunk from "redux-thunk"

import ReminderReducer from "./ReminderReducer"

const Store = createStore(
  ReminderReducer,
  composeWithDevTools(applyMiddleware(thunk))
)

export default Store
```

.

The store is connected but we need to define our asyncronous action creator. The **getReminders** action is special. Instead of returning an action object, it will return another function. It doesn't have to be a pure function like our other actions. It is allowed to have side effects like async api calls. It receives it's dispatch method as an arguement.

Our action creator first calls the **getReminderRequest** action, setting loading to true. If the call is successful, we call **getReminderSuccess** action passing in the reminders and those reminders are then stored in our state. If unsuccessful, we call **getReminderFailure** and pass in an error message from the api which stores that error message in our state.

Inside ReminderAction.js

```javascript
export const getReminders = () => {
  return (dispatch) => {
    dispatch(getReminderRequest());
    axios
      .get("https://jsonplaceholder.typicode.com/todos")
      .then((response) => {
        let reminders = response.data;
        reminders = reminders.splice(0, 10);
        dispatch(getReminderSuccess(reminders));
      })
      .catch((error) => {
        const errorMsg = error.message;
        dispatch(getReminderFailure(errorMsg));
      });
  };
```

.

The final step is to subscribe our reminder container component to the store and display the reminders.

To accomplish our attempt to display the data we use useEffect from redux, connect from react-redux and getReminders from our action creator component.

The mapStateToProps returns the state while the mapDispatchToProps dispatches the getReminders async action that makes the api call and returns the reminders - done asynchronously of course.

We destructure the state with data and the getReminders function, passing these needed elements to our ReminderContainer function. The useEffect hook allows us to call and load data from the api when the component first mounts to the page.

We further destructure _reminders, loading, error_ returned from the state and then use jsx conditional rendering techniques to display the data.

```javascript
import React, { useEffect } from "react"
import { connect } from "react-redux"
import { getReminders } from "../Reminders/ReminderAction"

function ReminderContainer({ getReminders, data }) {
  const { reminders, loading, error } = data
  useEffect(() => {
    getReminders()
  }, [])

  return loading ? (
    <h2>Loading</h2>
  ) : error ? (
    <h2>{error}</h2>
  ) : (
    <div>
      <h2>Reminder List</h2>
      {reminders.map(reminder => (
        <p key={reminder.id}>{reminder.title}</p>
      ))}
    </div>
  )
}

const mapStateToProps = state => {
  return {
    data: state,
  }
}
const mapDispatchToProps = dispatch => {
  return {
    getReminders: () => dispatch(getReminders()),
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(ReminderContainer)
```

.

Interestingly, thunk is a word that has been used for computer programming to describe a delay in a calculation until its result is needed.

Also, the word 'thunk' has been used as the past particible of the word think in the English language.

For example, you might not have 'thunk' it, but you can learn to use advanced react if you apply yourself.

Happy coding.
