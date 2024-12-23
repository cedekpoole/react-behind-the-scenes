# How React Works Behind the Scenes

I am currently following **Jonas Schmedtmann's** course on React.js and have decided to create detailed notes for future reference. These notes will be stored in this repository. When needed, I will create diagrams in Figma to explain particular sections visually.

## Table of Contents

- [**Section 1:** React, Props, and State](#section-1-what-is-react)
- [**Section 2:** Components, Instances, and Elements](#section-2-components-instances-and-elements)
  - [Summary](#section-2-summary)
- [**Section 3:** How Rendering Works](#section-3-how-rendering-works)
  - [Overview](#overview)
  - [Render Phase Steps](#render-phase-steps)
  - [The Reconciler and Fiber Architecture](#the-reconciler-and-fiber-architecture)
  - [Diffing Algorithm](#diffing-algorithm)
  - [The Commit Phase](#the-commit-phase)
  - [Summary of Section 3](#summary-of-section-3)
- [**Section 4:** Rules for Render Logic](#section-4-rules-for-render-logic)
  - [What is Render Logic?](#what-is-render-logic)
  - [Pure Functions and React Components](#pure-functions-and-react-components)
  - [Event Handlers](#event-handlers)
    - [The Difference: Render Logic vs. Event Handlers](#the-difference-render-logic-vs-event-handlers)
  - [Side Effects and Render Logic](#side-effects-and-render-logic)
  - [Pure Components in React](#pure-components-in-react)
    - [Event Handlers and Side Effects](#event-handlers-and-side-effects)
  - [Managing Side Effects: useEffect Hook](#managing-side-effects-useeffect-hook)
  - [Key Takeaways](#key-takeaways)
- [**Section 5:** React Hooks](#section-5-react-hooks)
  - [What are Hooks?](#what-are-hooks)
  - [Basic Hooks](#basic-hooks)
    - [Rules of Hooks](#rules-of-hooks)
    - [Why Call Hooks in the Same Order? (Order Matters)](#why-call-hooks-in-the-same-order-order-matters)
  - [Custom Hooks](#custom-hooks)
  - [More Advanced Hooks](#more-advanced-hooks)
  - [Key Takeaways](#key-takeaways)
- [**Section 6:** Advanced State Management](#section-6-advanced-state-management)
  - [Understanding State Accessibility: Local vs. Global State](#understanding-state-accessibility-local-vs-global-state)
  - [Understanding State Domain: UI State vs. Remote State](#understanding-state-domain-ui-state-vs-remote-state)
  - [Placing State: Where Should State Live?](#placing-state-where-should-state-live)

---

## Section 1: What is React?

React is a JavaScript library used for building user interfaces. Here's a brief overview of its core principles:

- **Declarative**: We tell React what we want, and it figures out how to do it.
- **Library, not a framework**: React provides tools for UI building but doesn't impose structure like a framework.
- **Component-based**: Reusable components are the building blocks of React apps.
- **Unidirectional Data Flow**: Data flows from parent to child components.
- **Virtual DOM**: React uses a virtual DOM (JavaScript representation of the actual DOM) to optimize rendering.
- **Popularity**: React is widely used and has a large community of developers.

### What are props?

Props (short for properties) are used to pass data from one component to another, usually from parent to child components.

- They are read-only (immutable).
- Useful for configuring and customizing components, making them dynamic and reusable.

### What is state?

State represents local data specific to a component. Unlike props, state can change over time.

- **Mutable**: State can be updated using React's `setState` method (or using hooks like `useState`).
- **Triggers re-render**: When the state changes, the component re-renders.
- **Maintains UI updates**: State allows the UI to respond to user events and preserve component-specific data between renders.

---

## Section 2: Components, Instances, and Elements

React’s core building blocks revolve around components, instances, and elements.

### Components

A component is a function that returns a piece of the user interface (UI) described using JSX.

```jsx
function Tab({ item }) {
  return (
    <div>
      <h4>All Contacts</h4>
      <p>Content: {item}</p>
    </div>
  );
}
```

- They describe a piece of UI, like the Tab component above.
- Components are functions that can return JSX, which is later transformed into React elements.
- A reusable 'Blueprint' or 'Template' for UIs.

### Instances

When we "use" a component, we create an instance of that component.

```JSX
    function App() {
        return (
            <div className="tabs">
                <Tab item={content[0]}/>
                <Tab item={content[1]}/>
                <Tab item={content[2]}/>
            </div>
        )
    }
```

- Instances are created whenever we use a component multiple times.
- React internally calls the component function (e.g., Tab()).
- Each instance has its own state and props, making it a "physical" representation of the component.
- We sometimes use components and component instances interchangeably (e.g. a UI is made up of components when in fact they are made of component instances) but they are different

### React Elements

When a component instance is rendered, it creates a React element.

- JSX is converted to React.createElement() function calls.
- A React element is an object containing all the information needed to render the component's DOM elements.

### DOM Elements

Once React elements are created, they are used to generate actual DOM elements (HTML) that are displayed in the browser.

- React elements are virtual representations, while DOM elements are their physical counterpart in the browser.

### Section 2 Summary

![Section 2 Summary](./images/01.png)
_Figure 1: Components, Instances, Elements and DOM Elements_

- **Components**: act as the blueprints or templates for the UI
- **Instances**: Physical manifestations of components with their own state and props.
- **React Elements**: Created when JSX is converted into React.createElement() calls. These elements are used to build the virtual DOM.
- **DOM elements**: The actual HTML elements rendered in the browser.

## Section 3: How Rendering Works

### Overview

At the highest level, rendering in React means taking your components and turning them into actual HTML elements that the browser can display. But before that happens, React goes through several steps to ensure the UI is efficiently updated based on the current state of the app.

#### What is a React Component?

A React component is a function or class that returns JSX (which is JavaScript XML). This JSX represents what the user interface (UI) should look like, including what HTML elements are shown and how they interact with data.

For example:

```jsx
function Greeting() {
  return <h1>Hello, World!</h1>;
}
```

In this case, the component is `Greeting`, and when called, it returns the JSX `<h1>Hello, World!</h1>`.

#### The Render Function

When React renders a component, it follows these steps:

1. **React calls the component function**:

   - The component function (like `Greeting()`) runs and returns JSX.

2. **JSX is turned into a React element**:

   - React elements are lightweight JavaScript objects that describe the UI. These elements are not HTML but tell React what HTML needs to be created.

   Example:

   ```javascript
   const reactElement = { type: "h1", props: { children: "Hello, World!" } };
   ```

#### What is the Virtual DOM?

The **Virtual DOM** is an internal copy of the actual DOM. It's not displayed in the browser; instead, it’s a lightweight version of the DOM that lives in memory. Think of it as a blueprint of what your UI should look like at any given time.

- Why does React use a Virtual DOM?
  - Efficiency: Updating the actual DOM can be slow because the browser has to re-draw parts of the page. React minimizes the number of updates by first making changes to the Virtual DOM. It compares the Virtual DOM to a previous version (a process called diffing) and then applies the smallest number of updates to the real DOM.

### Render Phase Steps

Now that we understand components, React elements, and the Virtual DOM, here’s what happens during the render phase:

1. **React calls the render function**: This means React invokes the component’s function (like `Greeting()`) to generate a React element.

2. **Virtual DOM update**:
   - The React element created in step 1 is used to update the Virtual DOM. If the Virtual DOM has changed since the last render, React figures out the minimum changes that need to be applied to the real DOM.
   - For example, if your component only changed one small part of the UI, React will update only that part in the real DOM.

> **Note**: When React renders a component, it also renders all of its child components to ensure the entire tree is up to date. This happens even if some child components haven't changed.

![The Render Phase](./images/03.png)  
_Figure 2: The Render Phase_

### The Reconciler and Fiber Architecture

The reconciler is the part of React that compares the Virtual DOM with the real DOM and decides what needs to be updated.

**What is Fiber?**
Fiber is a key part of React's architecture designed to improve performance by breaking the rendering process into smaller, manageable chunks

- Traditional DOM updates happen all at once. If your app has a large component tree, updating the entire tree could slow down your app and make it unresponsive.
- Fiber architecture allows React to break the work into small pieces and spread it out over multiple frames. This way, React can:
  - Pause work.
  - Resume work.
  - Abort unnecessary work.
  - This prevents the main JavaScript thread from being blocked, ensuring that the app remains responsive.

#### How Does Fiber Work?

**Fiber Tree Creation**

On the initial render, React creates a Fiber tree. This Fiber tree represents the entire component tree, but in a more efficient, linked-list format.

- Each component instance (like Greeting()) and each DOM element has a corresponding fiber node.
- Fiber nodes keep track of:
  - Component state (i.e., what data is changing).
  - Props (i.e., data passed down from parent components).
  - Side effects (i.e., updates that need to happen, like DOM changes or useEffect logic).

**Persistent and Mutable Structure**:
One of the key features of Fiber is that the Fiber tree is persistent. This means that it’s not re-created every time a component re-renders.

- Instead, the tree is mutated when a component changes. React doesn’t need to create new fibers from scratch—it can update only what needs to change.

**Fiber: Unit of Work**:
A fiber is also known as a unit of work. Each fiber (each node in the Fiber tree) represents a chunk of work that React needs to do, such as updating state or making DOM updates. Because the Fiber architecture breaks down rendering into these chunks, React can work on them in small parts, pausing or aborting when needed.

**Linked List and Sibling Relationships**:
Unlike the typical parent-child structure of the React element tree, the Fiber tree is structured as a linked list, where fibers are arranged in sibling relationships. This structure allows React to traverse the tree more efficiently and manage work in smaller portions.

**Asynchronous Rendering**:
The Fiber architecture enables React to handle rendering asynchronously, meaning it doesn’t have to complete everything in one go.

- Asynchronous work means React can pause and resume rendering whenever needed.
- If React is rendering a large component tree, it can break that work into smaller tasks, preventing long renders from blocking the main JavaScript thread (which is responsible for user interactions, animations, etc.).
  This ensures the app stays responsive, even during complex or long renders.

### Diffing Algorithm

When React updates the Virtual DOM, it uses a diffing algorithm to compare the old Virtual DOM with the new one. This algorithm helps React determine what has changed and what needs to be updated in the real DOM.

Now, let’s use a fun analogy to explain diffing. Imagine you’re playing a “spot the difference” game where you have two seemingly identical images, but there are small differences between them.

- **Old Virtual DOM**: This is the first image.
- **New Virtual DOM**: This is the second image after changes in the app (like a state update).

React needs to figure out the differences between these two images (Virtual DOMs). The process of finding the differences is called diffing. But React doesn’t just compare pixel-by-pixel. It uses some smart strategies to speed up the process:

1. **Same Type of Element**: React assumes that if two elements are the same type (like two `<div>` elements), it doesn’t need to look too deeply inside them unless their attributes or children have changed.

   Example:

   - You have a `<div>` containing text. After a state update, the text inside the `<div>` changes.
   - React will only compare the content of the `<div>`, not the entire structure of the element.

2. **Keys for Lists**: Imagine you’re comparing two lists of items. React uses keys to identify which items are new, which have been removed, and which have been moved around.

   Example:

   - Let’s say you have a list of three items: `["Apple", "Banana", "Cherry"]`.
     You change the list to `["Banana", "Apple", "Cherry"]`.
   - By using keys (unique identifiers for each item), React can see that only the order of the first two items has changed, so it only updates those items, not the entire list.

### The Commit Phase

**React’s Rendering Phases: The Big Picture**

When you interact with a React application (like clicking a button), React needs to recalculate and update the UI to reflect any changes. This is done through two main phases:

- **Render Phase**: React calculates what needs to be changed by comparing the new state of the app to the old one.
- **Commit Phase**: React applies those changes to the real DOM and makes them visible to the user.

![React Rendering Phases](./images/render.png)

_Figure 3: The Render Phase and the Commit Phase_

Now, let’s dive into the commit phase.

The commit phase is the second phase of React’s update process. Once React has figured out what needs to be changed during the render phase, it moves on to actually updating the DOM in the commit phase.

#### What Happens in the Commit Phase?

1. **Inserting, Deleting, and Updating DOM Elements:**

   - React takes the list of changes (calculated during the render phase) and applies them to the real DOM. It might:
     - Insert new elements (like adding a new list item),
     - Delete elements (removing an old item),
     - Update existing elements (changing a button’s text).
   - These DOM changes are synchronous, meaning they happen all at once without interruption. This ensures that the browser always has a fully consistent UI to display.

2. **Flushing Updates:**

   - React processes the effects list (side effects) created during the render phase, flushing (applying) those changes to the actual DOM.
   - Side effects could include anything from updating state, setting refs, to triggering useEffect hooks.
   - The effects list keeps track of all the changes React needs to apply (see figure 2)

#### Why Must the Commit Phase Be Synchronous?

- The commit phase is synchronous, meaning that React updates the entire DOM in one go. This is necessary because:
  - If React paused halfway through updating the DOM, the user might see a partial update (a broken UI where only some changes are visible).
  - By updating everything in one step, React ensures the user interface is always in a consistent, correct state.

#### Render Phase vs. Commit Phase

- **Render Phase**:

  - Can be paused, resumed, and even discarded. This is why it can handle complex operations without blocking the main thread, keeping the app responsive.
  - React builds a list of updates (the work it needs to do) but doesn’t yet apply them to the DOM.
  - The Fiber tree (React’s internal data structure) is updated, but no visible changes happen yet (it creates a work-in-progress tree).

- **Commit Phase**:
  - Cannot be interrupted. Once React starts updating the DOM, it finishes in one go.
  - This is when the calculated updates (from the render phase) are flushed to the DOM.
  - After the commit phase, the user can see the new, updated UI.

#### Fiber and the Commit Phase

**The Fiber Tree**

- In React, each element/component has a corresponding fiber (an object that holds information about the component’s state, props, and DOM updates).
- React builds and updates a Fiber tree during the render phase, where each fiber corresponds to a React element or DOM element.
- Once React completes the commit phase, the work-in-progress Fiber tree becomes the current Fiber tree. This means React doesn’t throw away the Fiber tree after each update but keeps reusing it, which saves time.

#### What Happens After the Commit Phase?

1. **Browser Paint:**
   - After React updates the DOM in the commit phase, the browser notices the DOM changes.
   - The browser schedules a paint operation, where it re-renders the user interface on the screen. This happens asynchronously, whenever the browser has idle time.
2. **Effect Hooks (useEffect)**:

   - Once the DOM is updated, React runs any side effects like useEffect hooks. These side effects can involve tasks like fetching data from APIs, adding event listeners, etc.
   - For example:

   ```javascript
   useEffect(() => {
     console.log("This runs after the commit phase");
   }, []);
   ```

![React Rendering Process](./images/04.png)

_Figure 4: Recap - The React Rendering Process_

#### React vs. React DOM: Who Does What?

React has two main pieces:

1.  **React (the Library)**:

    - React is the core library that handles the render phase. It figures out what changes need to be made and builds the Virtual DOM (or Fiber tree), but React itself doesn’t actually touch the real DOM.

2.  **React DOM (the Renderer)**:

    - React DOM is the part responsible for the commit phase. It takes the changes calculated by React and applies them to the DOM. This is why you need both `React` and `ReactDOM `in your `index.js` file:

      ```javascript
      import React from "react";
      import ReactDOM from "react-dom";
      ```

    - React DOM takes care of the commit phase, making sure the browser sees the correct UI. It is technically a renderer that deals specifically with the DOM.

#### React and Different Hosts

React was designed to be platform-independent, meaning it can work with many different environments (called hosts). The commit phase could be directed toward different output targets, depending on the platform. React can be used to create:

- **Web apps**: Using React DOM, which updates the real DOM in web browsers.
- **Native mobile app**s: Using React Native, which applies updates to native mobile components.
- **Other platforms**: You can even create documents (PDFs, Figma designs, etc.) using other renderers.

Each of these hosts (like React DOM or React Native) is responsible for applying the changes in the commit phase. The concept of a Virtual DOM really only makes sense in the context of web apps because on other platforms (like mobile), React is managing updates to a different kind of element.

#### Why Does React Split the Render and Commit Phases?

React divides its work into two phases (render and commit) to improve efficiency and performance:

- Render Phase: Can be paused and resumed, allowing React to handle heavy computations or complex updates without freezing the app. It allows concurrent rendering.
- Commit Phase: Must happen synchronously to ensure the UI remains consistent and is updated in one go. It flushes all the changes to the real DOM.

This split allows React to make the app responsive even when handling lots of updates.

### Summary of Section 3

Let’s bring it all together with a more complex example:

- Imagine you have a React app with a list of users displayed as cards, a search bar, and a "load more" button at the bottom.
- You type into the search bar, which triggers a state update that filters the list of users.
- Meanwhile, you also click the "load more" button to add more users to the list.

Here’s how React handles this efficiently:

1. **Virtual DOM Updates**: When you type into the search bar, React updates the Virtual DOM to reflect the new, filtered list of users.

2. **Diffing**: React compares the old Virtual DOM (with all users) to the new Virtual DOM (with filtered users). Since only some users need to be removed from the list, React figures out the minimal changes required and only updates the affected user cards in the real DOM.

3. **Fiber Prioritization**: Since typing in the search bar is a higher-priority action (it affects what the user sees immediately), React uses Fiber to prioritize updating the user list, while pausing the "load more" updates in the background. Once the user list is updated, React can resume work on loading more users.

4. **Commit Phase**: After reconciliation and diffing, React enters the commit phase and applies the necessary changes to the real DOM—updating only the relevant user cards and leaving the rest of the DOM untouched.

## Section 4: Rules for Render Logic

### What is Render Logic?

Render logic refers to all the code in a React component that determines how the UI should look. This is the code that runs when a component renders, meaning it determines the JSX (JavaScript XML) that React converts into actual DOM elements visible on the screen.

Example of Render Logic:

```jsx
function MyComponent(props) {
  const title = "Hello World!";

  return (
    <div>
      <h1>{title}</h1>
    </div>
  );
}
```

- The const title = "Hello World!"; is part of the render logic because it defines data that the component will display.
- The return statement containing JSX is also part of render logic. This is where you tell React how the component should look.

### Pure Functions and React Components

In functional programming (which React heavily relies on), pure functions are those that:

- Always return the same output for the same input.
- Do not cause side effects, meaning they don't alter external variables, make HTTP requests, manipulate the DOM directly, or set timers.

In React, the render logic must behave like a pure function. If given the same props, a component must render the same result (output) consistently. This helps React optimize rendering and makes your components predictable.

Example of Pure Function:

```jsx
function calculateArea(radius) {
  return Math.PI * radius * radius;
}
```

This function will always return the same output if you give it the same radius. It does not depend on external factors and has no side effects.

React expects component render logic to behave similarly.

### Event Handlers

Event handler functions are pieces of logic that respond to user actions. These functions are usually passed to elements to react to events like clicks, form submissions, etc.

Event handlers are different from render logic because they do not participate in rendering the UI but trigger actions when the user interacts with the component.

Example:

```jsx
function MyComponent() {
  const handleClick = () => {
    alert("Button clicked!");
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

Here, the handleClick function is an event handler. It doesn’t affect how the component is displayed. Instead, it executes when the user clicks the button.

#### The Difference: Render Logic vs. Event Handlers

- **Render logic** is responsible for describing the UI of the component.
- **Event handlers** are responsible for changing the state of the application or interacting with the outside world after user input.

### Side Effects and Render Logic

Side effects are actions that a function takes that affect the outside world, such as:

- Mutating a global variable.
- Making an HTTP request.
- Changing the DOM directly.

React enforces that render logic should not cause side effects. For example, you cannot directly fetch data or modify props within the render logic because that would make the component impure and unpredictable.

**Forbidden Side Effects in Render Logic:**

- Making HTTP requests
- Modifying DOM elements directly (e.g., document.querySelector)
- Setting timers (e.g., setTimeout)
- Mutating props

If you perform a side effect during render, React might behave unexpectedly or even crash, such as causing an infinite render loop.

### Pure Components in React

React components should behave like pure functions. If the same props are passed in, the same JSX should always be returned. This allows React to optimize rendering performance through techniques like memoization or re-render batching.

For example, updating state inside the render logic is a common mistake that can lead to infinite loops because changing the state triggers a re-render, which would again try to change the state.

```jsx
function MyComponent() {
  const [count, setCount] = useState(0);

  // ⚠️ This will cause an infinite loop
  setCount(count + 1);

  return <div>{count}</div>;
}
```

#### Event Handlers and Side Effects

Event handlers, on the other hand, are allowed to perform side effects because they are triggered in response to user actions. They can update state, make HTTP requests, or even manipulate the DOM (though it's better to let React handle that).

Example of Event Handler with Side Effects:

```jsx
function MyComponent() {
  const [data, setData] = useState(null);

  const handleClick = async () => {
    const response = await fetch("https://api.example.com/data");
    const result = await response.json();
    setData(result);
  };

  return (
    <div>
      <button onClick={handleClick}>Fetch Data</button>
      {data && <div>{data.title}</div>}
    </div>
  );
}
```

In this case, handleClick performs a side effect (fetching data from an API), but it is safe because it's an event handler and not part of the render logic.

### Managing Side Effects: useEffect Hook

When you need to perform side effects at specific times (such as after a component first renders or when certain data changes), you can use the useEffect hook. This hook allows you to manage side effects outside of the render logic, keeping the component pure.

Example with useEffect:

```jsx
function MyComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      const response = await fetch("https://api.example.com/data");
      const result = await response.json();
      setData(result);
    };
    fetchData();
  }, []); // Empty array means this runs only once after initial render.

  return <div>{data ? data.title : "Loading..."}</div>;
}
```

The useEffect hook is used here to fetch data after the component renders for the first time, without causing side effects during the render phase itself.

### Key Takeaways

- Render logic in React components must be pure and should not cause side effects.
- Event handlers handle user interactions and are allowed to perform side effects.
- Use useEffect for side effects that need to occur when a component renders or updates.
- Always separate render logic from code that changes application state or interacts with the outside world to avoid issues like infinite loops and unpredictable behavior.

## Section 5: React Hooks

### What are Hooks?

React hooks are special functions provided by React that allow us to "hook into" React's core functionalities, like managing state, side effects, and context, without writing class components. Essentially, hooks expose internal React mechanisms, giving developers access to powerful features that were previously only available in class-based components.

Hooks enable function components to:

- Manage their own state (`useState`)
- Perform side effects such as data fetching or logging (`useEffect`)
- Access context (`useContext`)
- And much more, like memoization and referencing DOM elements.

React keeps track of the component structure via the Fiber Tree, which is an internal data structure built from the **Virtual DOM**. Each element in the Fiber Tree is a fiber, which stores a linked list of the hooks used in a component.

For example, hooks like `useState` and `useEffect` are added to this linked list in the order they are called. This order must always be maintained, or React will lose track of the state associated with each hook.

### Basic Hooks

1. `useState`: Manages state in function components.

Every component in React can have its own state, allowing it to manage dynamic data (e.g., a counter value).

Example:

```jsx
const [count, setCount] = useState(0); // 'count' is the state, 'setCount' is the function to update it
```

- Every time `setCount` is called, the component re-renders, and the new value of count is displayed.

2. `useEffect`: Handles side effects in function components.

React updates the UI in response to state changes, but certain tasks—like fetching data, or interacting with external APIs—should happen outside this pure rendering process. useEffect allows us to perform such tasks after the component renders.

Example:

```jsx
useEffect(() => {
  console.log("Component mounted or updated");
}, [count]); // Runs whenever 'count' changes
```

- `useEffect` runs after the first render and whenever the count variable changes.

#### Rules of Hooks

There are two crucial rules to remember when working with hooks:

1. **Call hooks only at the top level of your component:**

- Hooks should not be called inside loops, conditionals, or nested functions. This is because React relies on the order in which hooks are called to keep track of the associated state. If the order changes between renders, React will get confused about which piece of state belongs to which hook.

2. **Call hooks only inside React functions:**

- Hooks must only be used inside React functional components or custom hooks, not in regular JavaScript functions.

#### Why Call Hooks in the Same Order? (Order Matters)

Hooks are registered in a linked list in the Fiber Tree. This linked list is built during the component's first render, and React uses the order of this list to keep track of hook state between renders.

For example:

- First, `useState()` is called, storing the initial state.
- Then, `useEffect()` is called, which registers a side effect.

If a hook is conditionally rendered (e.g., within an if statement), the order of the list breaks, causing React to lose track of which hook manages which state or effect. This can lead to errors or unexpected behavior. Therefore, hooks must always be called in the same order.

### Custom Hooks

React allows developers to create custom hooks, which start with the prefix use. Custom hooks allow for reusing logic between components that isn't tied to UI behavior. They can encapsulate complex logic, making it easier to share and reuse across different components.

For example, if two components need to fetch data, you can create a custom hook:

```jsx
function useFetchData(url) {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch(url)
      .then((response) => response.json())
      .then((result) => setData(result));
  }, [url]);

  return data;
}

// Usage:
const data = useFetchData("https://api.example.com");
```

This custom hook encapsulates the logic for fetching data, which can be reused across different components.

### More Advanced Hooks

- `useReducer`: A more complex version of `useState`, often used for managing state in large components or when the state logic is complex. It’s similar to Redux in that it uses a reducer function.

Example:

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    default:
      return state;
  }
}
```

- `useContext`: Allows a component to access global context without passing props through multiple levels.

Example:

```jsx
const ThemeContext = React.createContext("light");
const theme = useContext(ThemeContext); // Access the current value of ThemeContext
```

- `useRef`: Allows you to reference DOM elements or store a mutable value that does not trigger a re-render when updated.

Example:

```jsx
const inputRef = useRef(null);
useEffect(() => {
  inputRef.current.focus(); // Focuses the input element on mount
}, []);
```

### Key Takeaways

- React hooks allow you to use state and other React features in function components.
- They simplify React's API by eliminating the need for class components.
- Hooks must follow the rules (called at the top level, in the same order).
- Start with `useState` and `useEffect` to manage local state and side effects.
- Custom hooks provide a way to reuse logic across components

## Section 6: Advanced State Management

In this section, we will dive into the world of **state management** in React applications, specifically looking at how we can categorize, locate, and manage different types of state.

### Understanding State Accessibility: Local vs. Global State

Think of "state" like personal information you store for each part of an app. In a social setting, local state is like personal notes you take that only you need to see. It’s kept within a single React component or a small group of related components, and no one else in the app needs access to it.

In contrast, global state is like a shared announcement board everyone reads. Global state can be accessed by multiple parts of the app and is stored in a way that everyone who needs it can see it. Deciding whether state should be local or global depends on whether it should be accessible by other parts of the app.

> **Quick Tip:** If you wonder whether a piece of state should be local or global, imagine if you rendered the component twice: should a change in one copy affect the other? If yes, it’s likely global state. If no, it’s probably local.

### Understanding State Domain: UI State vs. Remote State

Now, let’s separate state by its domain—essentially, where the data comes from.

- **UI State**: This is state related to the user interface (UI). Think of UI state as settings and preferences within the app that make it look or feel a certain way, like dark mode, open panels, form inputs, etc. This state is often straightforward and typically stored directly in the app.

- **Remote State**: This is data fetched from a remote server, often through an API. It’s like information retrieved from a library database—often complex, and may require revalidation or frequent updates. Managing remote state well often involves handling it asynchronously and caching it for efficiency.

Because remote state can be complex, tools like React Query and SWR offer specialized features like caching and automatic re-fetching to handle this efficiently.

### Placing State: Where Should State Live?

When deciding where to place state in the app code, you have several options:

1. Within a Local Component: For simple, isolated states that don’t need to be accessed by other parts of the app, keep them in the component they belong to, using hooks like useState or useReducer.

2. Lifted to a Parent Component: If several components need to share a piece of state, you can move (or “lift”) the state up to a parent component that all those components have access to.

3. Using the Context API for Global State: When state needs to be shared globally across various parts of the app, the Context API provides a way to make state accessible throughout the component tree. However, Context API isn’t really managing the state; it’s simply making it accessible. The actual management still happens via useState or useReducer.

4. Using Third-Party Libraries: For larger applications or complex state needs, third-party libraries like Redux, Zustand, React Query, or SWR can manage global state more efficiently. Each has a unique focus: Redux, for instance, is versatile for managing UI and remote state, while React Query and SWR specialize in remote state.

5. Using the URL for Global State: Sometimes, it’s convenient to store global state in the URL, making it easily shareable and bookmarkable across pages.

6. Using Browser Storage: For persistent data that doesn’t trigger UI changes, like user preferences, local storage or session storage in the browser is an option. This is like storing information on your computer’s hard drive—it stays even if you refresh the page.

### Choosing the Right Tool for Each Type of State

We can now combine state classifications (local/global, UI/remote) to decide the best management tool:

- Local UI State: For small or temporary states that don’t need to be shared, use useState or useReducer.

- Local Remote State: Use useEffect to fetch data for smaller apps directly within the component. For example, a simple fetch request can pull data from an API, and useState or useReducer can store it.

- Global Remote State: In larger applications, remote state often benefits from tools like React Query or SWR. These tools efficiently handle tasks like caching, refreshing, and synchronizing data with the server.

- Global UI State: When UI settings or preferences need to be shared globally, the Context API with useState or useReducer is a good choice. For more performance-heavy needs, libraries like Redux can add extra features.
