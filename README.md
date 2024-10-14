# How React Works Behind the Scenes

I am currently following **Jonas Schmedtmann's** course on React.js and have decided to create detailed notes for future reference. These notes will be stored in this repository. When needed, I will create diagrams in Figma to explain particular sections visually.

## Table of Contents

- [**Section 1:** React, Props, and State](#section-1-what-is-react)
- [**Section 2:** Components, Instances, and Elements](#section-2-components-instances-and-elements)
  - [Summary](#section-2-summary)
- [**Section 3:** How Rendering Works](#section-3-how-rendering-works)
  - [Overview](#overview)

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

How are components displayed on the screen? How does React know when to update the UI?

### Overview

In practice, it might look like React only re-renders the component that has changed. However, React re-renders the entire app and then compares the virtual DOM with the actual DOM to determine what has changed and updates only the changed parts. In React, rendering is NOT updating the DOM, it only updates the virtual DOM. Renders are batched together and are not immediate because React is optimized to minimize the number of updates to the actual DOM.

![How Components are Displayed on Screen](./images/render.png)

_Figure 2: How Components are Displayed on Screen_

In practice, React follows these steps:

1. **Initial Render**: React renders the UI for the first time.
2. **Re-render**: React updates the UI when the state or props of a component change.
3. **Virtual DOM**: React creates a virtual DOM to optimize rendering.
4. **Diffing Algorithm**: React compares the virtual DOM with the actual DOM to determine what has changed.
5. **Reconciliation**: React updates only the changed parts of the actual DOM.

## The Render Phase

The render phase is the first phase of React's rendering process. When a component is rendered, React goes through a series of steps to ensure that the UI reflects the current state of the application.

### Steps in the Render Phase:

1. **Render Function**:
   - React calls the component function, which returns a React element.
   - React elements are plain JavaScript objects that describe what the UI should look like.
2. **Update Virtual DOM**:
   - React updates the Virtual DOM with the new elements.
   - The Virtual DOM is a lightweight, JavaScript representation of the actual DOM.
   - It consists of a tree of all React elements created from the component tree. This tree allows React to efficiently update only parts of the actual DOM that need to be changed.

> **Note:** Rendering a component will re-render all its children, even if they haven't changed. React ensures that the entire component tree stays up-to-date.

![The Render Phase](./images/03.png)  
_Figure 3: The Render Phase_

---

### The Reconciler: Fiber

React's reconciliation process is powered by the **Fiber architecture**, which improves efficiency and allows React to handle rendering in smaller chunks.

#### Key Concepts of Fiber:

- **Fiber Tree Creation**:

  - On the initial render, React constructs a **Fiber tree**. This tree is a lightweight representation of the React element tree (or virtual DOM).
  - Each component instance and DOM element gets a corresponding **fiber node**.

- **Persistent Structure**:
  - Fibers are **NOT** re-created on every render. Instead, the Fiber tree is persistent and mutable.
  - During the reconciliation process, React mutates the existing fibers rather than creating new ones. This helps React track the component's state, props, and side effects efficiently.

#### What Fibers Track:

- Fibers keep track of important information for each component instance, including:

  - Current **state** and **props**.
  - **Side effects** (e.g., updating DOM nodes, refs, or running certain hooks).
  - Pending **work** (e.g., state updates, DOM changes, etc.).

- Each fiber also maintains a **work queue**, containing tasks such as:
  - Updating state.
  - Performing DOM updates.
  - Running side effects (e.g., `useEffect`).
  - Processing refs.

Because of this, fibers are also referred to as **units of work**.

#### Fiber Tree Structure:

- Unlike the React element tree, which is structured as a typical parent-child hierarchy, the Fiber tree is structured as a **linked list**, with nodes arranged in sibling relationships.
- The Fiber tree is a complete representation of the DOM structure, enabling React to manage work more efficiently.

#### Asynchronous Work:

- One of the key advantages of Fiber is its ability to manage rendering asynchronously. This allows React to:
  - **Pause**, **abort**, or **resume** rendering work at any time.
  - Break long renders into smaller, manageable chunks without blocking the main JavaScript thread.

By working in small chunks, React prevents long renders from freezing the UI, allowing for a more responsive user experience.

---

By understanding how the render phase and Fiber architecture work, you can better grasp how React efficiently manages updates and ensures that the UI reflects the current state of the application without unnecessary re-renders or performance issues.
