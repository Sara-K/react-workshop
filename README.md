# Exercise 4 - State management
:book: At the moment our application is only a composition of visual elements. All data so far is hard-coded into our app. Unfortunately, any real-world app needs to handle data, and there are many, many ways of approaching this problem. 

## In this exercise you will learn:
- Context API
- Redux

## Reminder: Todo app spec

Here's the spec for our todo app as discussed in the previous exercise, for reference.

![](../images/todo-app.png)

### Header

- There will be an `h1` header for the name of this glorious app
- There will be a sub-header with slightly emphasized text stating how many total tasks there are and how many of those are completed.

### Adding a task

- There will be a textbox where a user can enter the description of a task
- There will be an "Add" button which will add the task to the list of existing tasks/todos.

### Listing todos

- There will be a list of todo items. Each todo item will consist of:
  - A checkbox with the description of the todo
  - A delete button which will remove the todo item permanently

![](../images/todo-app-components.png)

1. `App`. Will contain the header text and the sub-components.
1. `Summary`. Will contain the total number of tasks and show how many of those are completed.
1. `AddTodo`. Will contain the textbox and Add-button.
1. `TodoList`. Will contain the list for all todo items.
1. `TodoItem`. Will contain a checkbox that marks a task as In Progress or Done, and a Delete button.

## 4.1 - Context API
Remember that React has built-in internal state in class components. Sometimes, all you need is internal state, in which case you should use that mechanism. Learning when to use which mechanism is one of the learning curves with this stack.

When we use props we pass data from a parent component to a child component. It allows us to access state at different levels of the component. In situations where you’re looking to get the state from the top of your react tree to the bottom you might end up passing props through components that do not necessarily need them.

React Context is a really good alternative to solve this problem.  React Context is a way for a child component to access a value in a parent component. With context we can share data that can be considered global for a tree of React components. Lets say we want to add a “theme” prop in order to style the delete button in todoItem.

:pencil2: create a new file `theme.js` and copy & paste the following content:
```js
import React from "react";

export const themes = {
    light: {
      foreground: '#000000',
      background: '#c4d3c9',
    },
    dark: {
      foreground: '#ffffff',
      background: '#222222',
    },
  };
```

:pencil2: Now in `App.js` import the `theme.js` file and send the theme prop to the `TodoList` component:
```js
import {themes} from './theme’;

<TodoList
    theme={themes.light}
    todoItems={this.state.todoItems}
/>
```

:pencil2: In the `TodoList` component pass the prop to `TodoItem`.
```js
const TodoList = ({todoItems, theme}) => (
    <div>
        {todoItems.map(todoItem => (
            <TodoItem key={todoItem.id} id={todoItem.id} description={todoItem.description} theme={theme}/>
        ))}
    </div>
)
```

:pencil2: Now we need to use this `theme` prop to set the background color of the delete button:
```js
const TodoItem = ({ id, description, theme }) => (
    <div>
        <input type="checkbox" id={`todoItemCheckbox-${id}`} />
        <label htmlFor={`todoItemCheckbox-${id}`}>{description}</label>
        <button type="button" style={{backgroundColor: theme.background}}>Delete</button>
    </div>
);
```

If we use context, we can avoid passing props through `Todolist` to set the background color of the button.  

:pencil2: Rename `theme.js` to `themeContext.js`. Create a context for the current theme (with "light" as the default) in `themeContext.js`.
```js
export const ThemeContext = React.createContext(
    themes.light
);
```

:pencil2: Use a Provider to pass the current theme to the tree below. Now any component can read it, no matter how deep it is.
```js
import {ThemeContext} from './themeContext’;

<ThemeContext.Provider value={themes.light}>
    <TodoList
        todoItems={this.state.todoItems}
    />
</ThemeContext.Provider>
```

:pencil2: Assign a `contextType` to read the current theme context. React will find the closest theme Provider above and use its value.

```js
import {ThemeContext} from './themeContext’;

class TodoItem extends React.Component {

static contextType = ThemeContext;

render() {
    let props = this.props;
    let theme = this.context;
    
    return(
    <div>
        <input type="checkbox" id={`todoItemCheckbox-${props.id}`} />
        <label htmlFor={`todoItemCheckbox-${props.id}`}>{props.description}</label>
        <button type="button" style={{backgroundColor: theme.background}} >Delete</button>
    </div>
    )
}
```

The delete button should now render to the sceen with the current theme color. 
Context can make it easier to pass props around in certain scenarios, but it doesn’t give you access to some of the benefits redux offers.

## 4.1 - Redux in a hurry

> :exclamation: Please just read this section. We'll implement the examples into our app in the next sections.

Redux is currently one of the most popular solutions. It has nothing to do with React and can be used with Angular and other SPA frameworks, or alone. However, because of it's event-like, one-directional handling of state mutation, it is a particularly good fit with React.

Redux is a _state container_. All state that needs to be shared between components in our application will live and be maintained in Redux. Remember that React also has built-in internal state in class components. Sometimes, all you need is internal state, in which case you should use that mechanism and not Redux. Learning when to use which mechanism is one of the learning curves with this stack.

### Understanding Redux

![](../images/redux03.png)

Let's say we have an _Add_-button to add a new todo item. Adding this new todo item to the list of todo items would look like this:

1. The function provided to the button's `onClick` handler would call an _action_. An action is just an object that has an identifiable `type` and whatever data you need to mutate the state based on that action. For adding a new todo item, we'll need an action that looks like this:

```js
const addTodo = description => ({
  type: "ADD_TODO",
  description
});

// Identical to:

function addTodo(description) {
  return {
    type: "ADD_TODO",
    description: description
  };
}
```

2. The Redux Store object provides a `dispatch` function. This function _dispatches actions to the store_ (who would've thought!). So now the _store_ receives the `addTodo` action.
3. The _store_ will have a set of _reducers_ connected to it. A _reducer_ is simply a function that is passed in the store's _current state and the new action we dispatched to the store_. A reducer will look like this:

```js
const todosReducer = (state = [], action) => {
  switch (action.type) {
    case "ADD_TODO":
      return; /* new state based on the received action */
    default:
      return state;
  }
};
```

A few things to note:

- The `state` parameter defaults to an empty array. We call this the _default state_ for this reducer.
- The switch cases can return whatever we want. It's up to us to define _how the action changes the state_.
- Since this is simply a function that takes the old state, an action, and returns the new state, it's very easy to test, reason about, and debug.
- There should be absolutely no side-effects in a reducer (i.e no network calls, no filesystem calls, no DOM event triggering, etc). All side effects should be in _actions_. Reducers simply act upon the result of actions and sets a new state based on it.
- Reducers should never mutate the existing state object that's passed in as parameter. Instead, it should create and clone new state in order to define and return the new state.
- (We would of course put the action types such as `'ADD_TODO'` in constants so they can be reused and refactored safely across actions and reducers).

An implemented reducer for handling new todo items could look like this:

```js
const todosReducer = (todos = [], action) => {
  switch (action.type) {
    case 'ADD_TODO': {
      const newTodoId = todos.length + 1;
      return [
        ...todos,
        new Todo(newTodoId, action.description);
      ];
    }
    default:
      return todos;
  }
};
```

Again, note the following:

- Instead of mutating the existing state, we're returning _a new array_.
- We make good use of the JavaScript [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) to make sure our new todo list contains everything in the old list, _plus_ our new todo item which we instantiate using the class we made earlier.

Now that we have set a new state based on the action we dispatched when the user clicked on the button, we must _connect our React component to the Redux state_. Thankfully, Redux provides the glue for this, we just need to apply it.

Using our todo app as the example, we would glue the pieces together like this:

> Again, just read along for now, we'll implement this properly later.

```jsx
import React from "react";
import { connect } from "react-redux";
import TodoList from "./TodoList";
import { deleteTodo } from "./todoActions";

const TodoListContainer = props => (
  <TodoList todoItems={props.todoItems} onDeleteTodo={props.onDeleteTodo} />
);

const mapStateToProps = state => ({
  todoItems: state.todos
});

const mapDispatchToProps = dispatch => ({
  onDeleteTodo: todoId => dispatch(deleteTodo(todoId))
});

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoListContainer);
```

So there's a few things going on here.

- We created a new React component called `TodoListContainer`. We'll get back to this in a minute.
- There are two new functions: `mapStateToProps` and `mapDispatchToProps`. Both has to do with connecting Redux to our React app.
- We import the `connect` function from the `react-redux` package. This function takes everything we've defined here and connects everything together.

#### `mapStateToProps`

This function receives `state` as the first parameter. `state` is the entire state-tree in Redux. We'll get back to this in a minute. The takeaway here is that we use the state-parameter to select what in our state-tree we want to send in as _props_ to our component. In this case, we know we defined the prop `todoItems` back in `TodoList.jsx`, so we _map the todo list in the Redux state-tree to the `todoItems`-prop_.

#### `mapDispatchToProps`

This function receives the `dispatch` function as the first parameter. As mentioned earlier when we explained actions, we use the `dispatch` function to dispatch actions (side effects) to the Redux store. In this example, we _map the `onDeleteTodo`-prop to a function that takes a todo-ID and dispatch the `deleteTodo` action to the store with that ID_.

If there are tiny explosions in your head right now, that's ok :) We'll get there!

#### Containers vs. Components

With the introduction of Redux, a naming convention of `FooContainer` and `FooComponent` was suggested by the community in order to separate the glue-logic from the visual DOM elements. Confusingly enough, they opted to use the _Component_ terminology for this as well, even though everything we make in React is already a "React Component"...

The TL;DR version is this:

- _Containers_ are concerned with how things work and connects, and does not render DOM layouts. We typically use this as the example above, to glue React and Redux together, then just pass everything over to a _Component_.
- _Components_ are concerned with how things look. They are typically as simple and stupid as possible, only receiving data through props and returns JSX.

In this workshop, we'll use the naming conventions used in the workshop authors' work projects. This is not necessarily how you'd see components named everywhere else, or an acknowledged community naming practice. Nonetheless, we find it to be declarative and working well:

- We'll name _Containers_ using the _Container_ suffix, and _Components_ without any suffix. For example the `TodoListContainer`_-component_ (tongue straight now!) wires up data for the `TodoList`_-component_, which will only receive props and render the list to the screen. The files on disk will be named accordingly: `TodoListContainer.jsx` and `TodoList.jsx`.

#### Spreading props

Because _Containers_ just passes props on to it's sibling _Component_, we can just pass all props on aswell:

```jsx
/* ... */

const TodoListContainer = props => <TodoList {...props} />;

/* ... */
```

This way we don't have to specify PropTypes validation, and we reduce boilerplate when adding new props in `mapState(..)` or `mapDispatch(..)`. This is, however, considered an anti-pattern by the community because it makes the code less explicit and harder to debug.


