# How React Works Behind the Scenes

I am currently following **Jonas Schmedtmann's** course on React.js and have decided to create detailed notes that I can refer back to. These notes will be stored in this repository. I will also create diagrams, when needed, in figma as means to explain particular sections.

## Table of Contents

- [**Section 1:** React, Props and State](#section-1-what-is-react)
- [**Section 2:** Components, Instances and Elements](#components-instances-and-elements)
  - [Summary](#section-1-summary)

## Section 1: What is React?

- Declarative (we tell React what we want and it figures out how to do it)
- React is a library, not a framework
- Create reusable components (the building blocks of React apps)
- React is all about building UIs (User Interfaces)
- One-way data flow (downwards) called unidirectional data flow (from parent to child)
- Uses a virtual DOM (a JavaScript representation of the actual DOM)
- Is very popular and has a huge community

### What are props?

- They are data passed down from a parent component to a child component
- Props are read-only (immutable)
- A tool to configure and customise components, making them dynamic and reusable

### What is state?

- State is data that is local or private to a component
- State is mutable (can be changed)
- When state changes, React re-renders the component
- This allows devs to update the UI in response to events and persist local variables between renders

## Section 2: Components, Instances and Elements

### Components

```JSX
    function Tab({ item }) {
        return (
            <div>
                <h4>All Contacts</h4>
                <p>Content: {item}</p>
            </div>
        )
    }
```

- They describe a piece of UI (e.g. Tab)
- A function that returns React Elements (element tree), normally written as JSX
- A 'Blueprint' or 'Template'

### Instances

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

- Instances are created when we 'use' a component - we can use the same component multiple times
- React internally calls Tab()
- 'Physical' manifestation of component
- Has own state and props; has its own lifecycle
- We sometimes use components and component instances interchangeably (e.g. a UI is made up of components when in fact they are made of component instances) but they are different

### React Elements

- As react executes the code in each of these instances, they will each return one or more react elements.
- JSX is converted to React.createElement() function calls and a React element is the result
- A React element contains all the info necessary to create DOM elements for the current component instance

### DOM Elements

- A React element is converted into actual DOM elements (HTML) - the actual visual representation of the component instance in the browser
- React elements are not rendered to the DOM, they live inside the app

### Section 2 Summary

![Section 2 Summary](./images/01.png)
Figure 1: Components, Instances, Elements and DOM Elements

- Components are the blueprints
- Instances are the physical manifestations of the components that each have their own state and props
- As React calls each component instance, each JSX will create a react.createElement() function call which will create a react element for each component instance; elements are the objects that React uses to create and update the DOM
- DOM elements are the actual HTML elements that are rendered to the browser
