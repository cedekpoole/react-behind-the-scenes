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
    - [What is Fiber?](#what-is-fiber)
    - [How Does Fiber Work?](#how-does-fiber-work)

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
_Figure 3: The Render Phase_

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
