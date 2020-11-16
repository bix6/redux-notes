# Redux Notes

- [Redux Tutorial](https://redux.js.org/tutorials/fundamentals/part-1-overview)
- [Better Redux Tutorial](https://redux.js.org/tutorials/essentials/part-1-overview-concepts)

## Redux Funadementals Tutorial

### What is Redux

- Pattern and library for managing and updating application state using events called "actions"
- Helps manage global state (state that is needed across many parts of the app)

### Redux Libraries and Tools

- Redux - standalone JS library often used with other packages
- React-Redux - Let React components interact with a Redux store by reading pieces of state and dispatching actions to update the store
- Redux Toolkit - Recommended approach for writing Redux logic. Builds in suggested best practices, simplifies redux tasks, prevents mistakes and makes it easier to write Redux apps
- Redux DevTools Extension - Shows a history of changes in state; helps debugging; enables time-travel debugging

### Redux Basics

- Center of all Redux apps is the `store`
- Store is a container that holds apps global state
- Store is a JavaScript object with special functions/abilities
  - Must never directly modify or change the state in the store
  - Instead, create a plain `action` object that describes "something that happened in the app" and then `dispatch` the action to the store to tell it what happened
  - When action is dispatched, store runs the root `reducer` function and lets it calculate the new state based on the old state and action
  - Finally the store notifies `subscribers` that the state has been updated so the UI can be updated with fresh data

### Redux Core Example App

- This example just uses a script tag to load the Redux library with basic JS/HTML for UI - usually we'd install with npm and use React for UI
- First up we define intial state

```
// Definie initial state for the app
const initialState = {
    value: 0;
}
```

- Typically we store a JS object as the root piece of the state with other values inside that obj
- Then we define a reducer function that takes two args: the current `state` and an `action` object describing what happened
- There's no state intially so we provide `initialState` as the default value for this reducer
- The default case just returns the unchanged state if there's an action we don't care about

```
// Create a reducer function that determines what the new state should be when something happens
function counterReducer(state = initialState, action) {
    // Reducers usually look at the type of action to decide how to update state
    switch(action.type) {
        case 'counter/incremented':
            return {...state, value: state.value + 1}
        case 'counter/decremented':
            return {...state, value: state.value - 1}
        // If the reducer doesn't care about the action type we return the state unchanged
        default:
            return state
    }
}
```

- Action objects always have a `type` field which is a string that acts as a unique name for the action
- This should be understandable by anyone reading the code
- Note that the state is updated immutably by copying the existing state and updating the copy, instead of modifying the original object directly
- Now that we have a reducer function, we can create a `store` by calling `createStore`

```
// Create a new Redux store and use the `counterReducer` for update logic
const store = Redux.createStore(counterReducer);
```

- This uses the reducer function to generate the intial state and to calculate any future updates
- Then we setup the UI to use the store and update on render

```
// Our UI is some text in a single HTML elem
const valueEl = document.getElementById('value');

// Whenever the store state changes, update the UI by reading the latest store state
function render() {
    const state = store.getState();
    valueEl.innerHTML = state.value.toString();
}

// Update the UI with initial data
render()
// And subscribe to update whenever the data changes
store.subscribe(render)
```

- So we write a function (render) that gets the latest state from the Redux store and then updated the UI
- `store.subscribe()` let's us pass a subscriber function that will be called every time the store is updated
- So we pass in render; everytime the store updates, render is called with the new values
- Finally we need to respond to the user input by creating `action` objects that describe what happened and `dispatch` them to the store
- Calling `store.dispatch(action)` runs the reducer, calculates the updated state and runs the subscribes to update the UI

```
// Handle user inputs by "dispatching"
document.getElementById('increment').addEventListener('click', function() {
    store.dispatch({ type: 'counter/incremented' })
})

document.getElementById('decrement').addEventListener('click', function() {
    store.dispatch({ type: 'counter/decremented' })
})

document.getElementById('incrementIfOdd').addEventListener('click', function() {
    // Write logic to decide what to do based on state
    if (store.getState().value % 2 !== 0) {
        store.dispatch({ type: 'counter/incremented' })
    }
})

document.getElementById('incrementAsync').addEventListener('click', function() {
    // Write async logic to interact with store
    setTimeout(function() {
        store.dispatch({ type: 'counter/incremented '})
    }, 1000)
})
```

- This shows a variety of use cases: general increment/decrement, increment based on state, async increment

### Data Flow

- Actions are dispatched in response to a user interaction
- The store runs the reducer function to calculate a new state
- UI reads the new state teo display new values

## Redux Essentials

- This is the essential tutorial which is recommended over the fundamentals tutorial because it uses latest tools and best practices
- Skipping some sections that are repeated above
- We use Redux to make it easier to understand when, where, why and how the state in our app is being changed, and how the application logic will behave when changes occur

### Redux terms and concepts

#### State Management

- We'll start by looking at a small React counter compoment

```
function Counter() {
    // State: a counter value
    const [counter, setCounter] = useState(0);

    //Action: code that causes an update to the state when something happens
    const increment = () => {
        setCounter(prevCounter => prevCounter + 1)
    }

    // View: the UI definition
    return (
        <div>
            Value: {counter} <button onClick={increment}>Increment</button>
        </div>
    )
}
```

- This is a self contained app with the following parts:
  - state: source of truth
  - view: declarative description of UI based on current state
  - actions: events that occur in the app based on user input and trigger updates in state
- Small example of one way data flow
  - State describes app condition at specific point in time
  - UI rendered based on that state
  - When something happens, state is updated
  - UI re-renders with new state
- Simplicity breaks down when multiple components need to share/use the same state; this is why we extract into a store

#### Immutability

- Mutable means changeable
- Immutable means not changeable
- When we change an array or object, we mutate it
- To update values immutably, code must make copies and then modify the copies
- We can use the spread operator (...) or array methods that return new copies of the array
- Redux expects all state updated to be done immutably

#### Terminology

##### Actions

- `action` is a plain JS obj with a `type` field
  - event that describes something that happened in the app
- `type` should be a string with a descriptive name like `todos/todoAdded`
  - We usually do `domain/eventName` where domain is the feature/category, eventName is the specific action
- Action object can have other fields, by convention we put that info in the `payload` field
- Typical action:

```
const addTodoAction = {
    type: 'todos/todoAdded',
    payload: 'Buy milk'
}
```

##### Action Creators

- `action creator` is a function that creates and return an action object
  - Use these so we don't have to write the action object by hand each time

```
const addTodo = text => {
    return {
        type: 'todos/todoAdded',
        payload: text
    }
}
```

##### Reducers

- `reducer` is a function that receives the current state and an action object, decides how to update the state and return the new state
  - `(state, action) => newState`
  - Like an event listener which handles events based on the received action/event type
  - Similar to `array.reduce` which is where the name comes from
    - Takes an array input and a reducer function, returns a single output
- Reducers must always follow rules:
  - Only calculate new state based on state and action arguments
  - Not allowed to modify existing state; must make immutable updates (copy existing state and make changes to copied values)
  - No asynchronous logic, random values or other side effects
- Logic inside reducer function:
  - Check to see if reducer cares about this action
    - IF so, copy state, update copy with new values, return it
  - Otherwise, return existing state unchanged

```
const initialState = { value: 0 }

function counterReducer(state = intitialState, action) {
    // If we care about the action, copy state and update the value
    if (action.type === 'counter/increment') {
        return {...state, value: state.value + 1}
    }
    // Return state unchanged
    return state;
}
```

##### Store

- Current redux app state lives in an object called `store`
- Store is created by passing in a reducer and has a method called getState

```
import { configureStore } from '@reduxjs/toolkit';
const store = configureStore({ reducer: counterReducer })
console.log(store.getState());
```

##### Dispatch

- Redux store has a method called `dispatch`
- Only way to update the state is to call `store.dispatch()` and pass in an action object
- The store runs its reducer function and saves the new state value inside

```
store.dispatch({ type: 'counter/increment' })
console.log(store.getState())
```

- Dispatching actions is similar to triggering an event
- Typically we call action creators to dispatch the right action

```
const increment = () => {
    return {
        type: 'counter/increment'
    }
}

store.dispatch(increment());
```

##### Selectors

- Selectors are functions that extract specific pieces of information from a store state value
- This helps in large apps, don't need to repeat logic if different parts of app need the same data

```
const selectCounterValue = state => state.value

const currentValue = selectCounterValue(store.getState());
```

#### Redux Application Data Flow

- One way data flow
- Initial setup:
  - Redux store createdf with root reducer function
  - Store calls root reducer once, saves the return value as initial state
  - UI is first rendered by accessing current state of Redux store, also subscribes to any future updates
- Updates:
  - Something happens in the app, like a user clicking a button
  - App dispatches an action to the Redux store like `dispatch({ type: 'counter/increment' })
  - STore runs reducer function again with previous state and current action, saves return value as the new state
  - Store notifies all parts of UI that are subscribed that the store has been updated
  - Each UI component that needs data from the store checks if the parts they need have changed
  - Each compoent that sees it data has changed forces a re-render to update with the new state

## Redux Essentials, Part 2: Redux App Structure

- Look at a real world example to see how everything fits together

### Counter Example App

- Created using the [Redux template for Create-React-App](https://github.com/reduxjs/cra-template-redux)
- Configured with standard Redux app structure, using Redux Toolkit to create redux store and logic and React-Redux to connect Redux Store with React Components
- Start the app (I also created one on dev4 so I can browse the code) and look at the Redux section of devtools (only worked on Chrome for me)
- Redux dev tools are super powerful, we can see state as it changes, stack trace, jump around, etc.
- The app is laid out with `/app` holding `store.js` and `/features` holding `/counter` which holds `counter.js` and `counterSlice.js`
- First up is `store.js`

```
import { configureStore } from '@reduxjs/toolkit`
import counterReducer from '../features/counter/counterSlice'

export default configureStore({
    reducer: {
        counter: counterReducer
    }
})
```

- Redux store is created using `configureStore` and passing in a reducer
- Our app may have many features each with its own reducer, we can pass all those into the configureStore
- The key names in the object will define the keys in the final state value
- Passing in `{counter: counterReducer}` says we want a `state.counter` in our Redux state and we want the `counterReducer` function to decide if/how to update the `state.counter` value
- Redux store can add plugins (middleware and enhancers). `configureStore` adds several middleware automatically for better dev experience; enables DevTools extension

#### Redux Slices

- A slice is a collection of Redux reducer logic and actions for a single feature in our app, typically defined together in a single file
- Name comes from splitting the root Redux state object into multiple slices of state
- For example we might do something like this:

```
import { configureStore } from '@reduxjs/toolkit'
import usersReducer from '../features/users/usersSlice'
import postsReducer from '../features/posts/postsSlice'
import commentsReducer from '../features/comments/commentsSlice'

export default configureStore({
  reducer: {
    users: usersReducer,
    posts: postsReducer,
    comments: commentsReducer
  }
})
```

- Since `usersReducer` is resposible for updating `state.users` slice we refer to it as a `slice reducer` function
- Behind the scenes we're running the `combineReducers()` function to combine all our reducers into the single root state

#### Creating Slice Reducers and Actions

- Now we're in `counterSlice.js`

```
import { createSlice } from '@reduxjs/toolkit';

export const counterSlice = createSlice({
    name: 'counter',
    initialState: {
        value: 0
    },
    reducers: {
        increment: state => {
            // Redux Toolkit allows us to write "mutating" logic in reducer
            // It doesn't actually mutate the state because it uses the immer library
            // which detects changes to a "draft store" and produces a brand new
            // immutable state based off those changes
            state.value += 1
        },
        decrement: state => {
            state.value -= 1
        },
        incrementByAmount: (state, action) => {
            state.value += action.payload
        }
    }
})

export const { increment, decrement, incrementByAmount } = counterSlice.actions

export default counterSlice.reducer
```

- We know that we're dispatching Redux actions like `{type: "counter/increment"}` but we're defining them differently because we're using `createSlice`
- We could have written them by hand but that's tedious
- In this case we take the `name` and combine it with each of our reducers to create somehting like `counter/increment`; `createSlice` does this work for us
- So this creates our actions and then we can just call it like `counterSlice.actions.increment()`
- It also generates the slice reducer function that knows how to respond to all these action types

```
const newState = counterSlice.reducer(
    { value: 10 },
    counterSlice.actions.increment()
)
```

#### Rules of Reducers; Reducers and Immutable Updates

- Certain rules we have to follow, already covered this. Follow the rules or things break.
- Never do this `state.value = 123`
- Instead do this `return {... state, value: 123 }`
- Writing immutable logic by hand is hard which is why Redux Toolkits `createSlice` lets us write immutable updates an easier way
- This is done with a tool called `Immer` that uses a special JS tool `Proxy` to wrap the data provided and write code that "mutates" the wrapped data
- Immer tracks all the changes we've tried to make and uses that to return a safely immutably updated value
- This makes our code way simpler
- **We can only write mutating logic inside Redux Toolkits `createSlice` and `createReducer` because they use immer**
- So if we look at our code now we see that we can just `state.value +=1` because we're inside `createSlice`
- We don't need to look at the `action` object since it's passed in anyways
- However, for `incrementByAmount` we need a payload so we pass in both `state` and `action` arguments

#### Writing Async Logic with Thunks

- A thunk is a specific kind of Redux function that can contain async logic
- Thunks are written with two functions:
  - Inside thunk function which gets `dispatch` and `getState` as arguments
  - Outside creator function, which creates and returns the thunk function
- This is a thunk action creator:

```
// This is a thunk and allows async logic
// It can be dispatched like a regular action: `dispatch(incrementAsync(10))`
// This will call the thunk with the `dispatch` function as the first argument
// Async code can then be executed and other actions can be dispatched
export const incrementAsync = amount => dispatch => {
    setTimeout(() => {
        dispatch(incrementByAmount(amount))
    }, 1000)
}
```

- Then we can use them with `store.dispatch(incrementAsync(5))`
- Using thunks requires `redux-thunk` middleware
  - This is automatically added when we use Toolkit's `configureStore`
- To make AJAX calls we use a thunk

```
// The outside "thunk creator" function
const fetchUserById = userId => {
    // the inside "thunk function"
    return async (dispatch, getState) => {
        try {
            //make an async call
            const user = await userAPI.fetchById(userId)
            // dispatch an action when we get the response
            dispatch(userLoaded(user))
        }
        catch (err) {
            // Handle error
        }
    }
}
```

#### React Counter Component

- First up is `Counter.js` file:

```
import { useSelector, useDispatch } from 'react-redux';
import { decrement, increment, incrementByAmount, incrementAsync, selectCount } from './counterSlice';
import styles from './Counter.module.css';

export function Counter() {
    const count = useSelector(selectCount);
    const dispatch = useDispatch();
    const [ incrementAmount, setIncrementAmount ] = useState('2');

    return (
        <div>
            <div className={styles.row}>
                <button
                    className={styles.button}
                    aria-label="Increment value"
                    onClick={() => dispatch(increment())}
                >
                    +
                </button>
                <span className={styles.value}>{count}</span>
                <button
                    className={styles.button}
                    aria-label="Decrement value"
                    onClick={() => dispatch(decrement())}
                >
                    -
                </button>
            </div>
            {/* rendering stuff here */}
        </div>
    )
}
```

- We have a function component called `Counter` that stores some data in a `useState` hook
- `count` isn't coming from a `useState` hook
- React includes several built in hooks like `useState` and `useEffect`
  - [info about hooks](https://reactjs.org/docs/hooks-state.html)
- We can create our own [custom hooks](https://reactjs.org/docs/hooks-custom.html)
- React Redux incluedes a [number of hooks](https://react-redux.js.org/api/hooks)
- We use `useSelector` to extract values from the store since we can't import it directly
  - `counterSlice.js` had the selector at the bottom `export const selectCount = state => state.counter.value`
  - If we could acces the store we'd retrieve it with `const count = selectCount(store.getState())`
  - But we don't so we use `const count = useSelector(selectCount)`
- We could write an inline selector `const countPlusTwo = useSelector(state => state.counter.value + 2)`
- useSelector will re run the selector function whenever the store updates

- Similarly we could dispatch actions wwith `store.dispatch(increment())` but that requires having access to the store
- So instead we use `const dispatch = useDispatch()`
- And then we dispatch with `dispatch(increment())`

- Only put global state that is needed across the app in the Redux store
- In this case we use `const [ incrementAmount, setIncrementAmount ] = useState('2');`
- Since we're using a function component we have to use useState since there is no this
- So in this case we set up `incrementAmount` with the default value of 2
- To update incrementAmount, we call setIncrementAmount
- In a class we would be doing this.state.incrementAmount but useState is the function compoment version

```
const [ incrementAmount, setIncrementAmount ] = useState('2');

// later
return (
    <div className={styles.row}>
        <input
            className={styles.textbox}
            aria-label="Set increment amount"
            value={incrementAmount}
            onChange={e => setIncrementAmount(e.target.value)}
        />
        <button
            className={styles.button}
            onClick={() => dispatch(incrementByAmount(Number(incrementAmount) || 0))}
        >
            Add Amount
        </button>
        <button
            className={styles.asyncButton}
            onClick={() => dispatch(incrementAsync(Number(incrementAmount) || 0))}
        >
            Add Async
        </button>
    </div>
)
```

- Most form states should probably not be kept in redux
  - Keep the data in form components and then dispatch Redux actions to update store

#### Providing the store

- How do the useSelector and useDispatch hooks know which store to talk to?
- In `index.js` we wrap our app in a Provider and pass it the store

```
ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
)
```

#### What You've Learned

- Create a Redux Store using Redux Toolkit's `configureStore`
  - `configureStore` accepts a `reducer` function as a named argument
  - `configureStore` automatically sets up the store with good defaults
- Redux logic is typically organized into files called slices
  - A slice contains the reducer logic and actions related to a specific feature / section of the Redux state
  - Redux Toolkit's `createSlice` generates action creators and action types for each individual reducer function we provide
- Redux reducers must follow specific rules
  - Should only calculate a new state value based on `state` and `action` arguments
  - Must make immutable updates by copying the existing state
  - Cannot contain any async logic or other "side effects"
  - Redux Toolkits `createSlice` uses Immer to allow mutating immutable updates
- Async logic is typically written in special functions called "thunks"
  - Thunks receive `dispatch` and `getState` as arguments
  - Redux Toolkit enables the `redux-thunk` middleware by default
- React-Redux allows React components to interact with a Redux store
  - Wrapping the app with `<Provider store={store}>` enables all components to use the store
  - Global state should go in the Redux store, local state should stay in React components
