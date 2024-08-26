### What is difference between npm and npx
<table><tbody><tr><td colspan="2"><p style="text-align:center"><strong>&nbsp;npm&nbsp;</strong></p></td><td colspan="2"><p style="text-align:center"><strong>npx</strong></p></td></tr><tr><td colspan="2">If you wish to run package through npm then you have to specify that package in your&nbsp;package.json and install it locally.</td><td colspan="2">A package can be executable without installing the package. It is an npm package runner so if any packages aren’t already installed it will install them automatically.</td></tr><tr><td colspan="2">To use `create-react-app` in npm the commands are `npm install create-react-app` then `create-react-app myApp` (Installation required).</td><td colspan="2">In npx you can create a react app without installing the package:<br>`npx create-react-app myApp`<br>This command is required in every app’s life cycle only once.</td></tr><tr><td colspan="2">Npm is a tool that use to install packages.</td><td colspan="2">&nbsp;Npx is a tool that use to execute packages.</td></tr><tr><td colspan="2">Packages used by npm are installed globally. You have to care about pollution in the long term.</td><td colspan="2">&nbsp;Packages used by npx are not installed globally. You don’t have to worry about for pollution in the long term.&nbsp;</td></tr></tbody></table>

### React App Walkthrough

### Create React App

1. **Create a new React app:** 
Open your terminal and run the following command to create a new React app using Create React App without installing it globally:
> npx create-react-app your-app-name
> Replace "your-app-name" with the desired name for your React app.
2. **Navigate to the project directory:** 
Change into the newly created app directory using the following command:
> cd your-app-name

3. **Run the development server:**
 Start the development server to see your React app in action. Run the following command:
 > npm start
 > This will start the development server and open your app in a new browser window.


<ins> **createRoot Function** </ins>

The createRoot function is part of React's Concurrent Mode API, introduced to enable smoother user experiences by allowing React to work on multiple tasks concurrently. It is used to create a root in the React tree where concurrent rendering can take place. Concurrent Mode is an experimental set of features in React that may undergo changes in future releases.


```
import React, { createRoot } from 'react';

import ReactDOM from 'react-dom';



// Create a root with createRoot

const root = createRoot(document.getElementById('root'));



// Concurrently render a component

const App = () => (

<div>

<h1>Hello, React Concurrent Mode!</h1>

</div>

);



// Render the component within the root

root.render(<App />);
```
<ins> **JSX** </ins>

JSX (JavaScript XML) is a syntax extension for JavaScript, commonly used with React to describe what the UI should look like. JSX provides a more concise and readable syntax for defining the structure of React components. It looks similar to XML or HTML but is actually a syntactic sugar for React.createElement calls.



<ins> **Functional Component** </ins>

A functional component in React is a simple JavaScript function that returns JSX (JavaScript XML) to describe the structure of the UI. Functional components are often referred to as "stateless components" because they don't have internal state or lifecycle methods



<ins> **export default** </ins>

In JavaScript (and also in React), the export default syntax is used to export a single "default" value or function from a module. This allows you to import that value or function without using curly braces {} when importing it in another module.



<ins> **Creating Components in React** </ins>

Create a new file for your component: Start by creating a new JavaScript or JSX file for your component. Let's say you want to create a basic functional component named MyComponent.


```
// MyComponent.js

import React from 'react';

const MyComponent = () => {

return (

<div>

<p>This is my component!</p>

</div>

);

};

export default MyComponent;

```
**Use the component in another file:** You can then use this component in another file by importing it:
```
// App.js

import React from 'react';

import MyComponent from './MyComponent';

const App = () => {

return (

<div>

<h1>Hello, React App!</h1>

<MyComponent />

</div>

);

};

export default App;
```

### Document Object Model ( DOM )

DOM (Document Object Model) is a programming interface for web documents. It represents the structure of HTML or XML documents as a tree-like model where each node represents a part of the document. The DOM provides a way for programs to dynamically access and update the content, structure, and style of web pages.

### Virtual DOM

The Virtual DOM is a concept used in various frontend libraries and frameworks, notably React.js, to optimize the performance of web applications by minimizing the number of direct manipulations to the actual DOM. It works by maintaining a lightweight, in-memory representation of the actual DOM known as the virtual DOM.

There are 2 important concepts used by which React handles creation of DOM,

1. **Reconciliation**

>Reconciliation in React is the process by which the framework efficiently updates the user interface in response to changes in application state. This mechanism revolves around React's Virtual DOM, a lightweight representation of the actual DOM. When a component's state changes, React re-renders the component and its children, comparing the new Virtual DOM with the previous one through a diffing algorithm. This comparison identifies the minimal set of changes needed to update the actual DOM, ensuring optimal performance by only applying necessary updates. Reconciliation plays a crucial role in enhancing the responsiveness and efficiency of React applications, abstracting away the complexities of DOM manipulation and providing developers with a streamlined approach to building dynamic user interfaces.



2. **Diffing Algorithm**

>The diffing algorithm in React is a core mechanism responsible for efficiently updating the user interface by identifying the minimal set of changes needed to reconcile the Virtual DOM with the actual DOM. When a component's state changes, React re-renders the component and its children, generating a new Virtual DOM tree. The diffing algorithm then compares this new Virtual DOM tree with the previous one, recursively traversing both trees to identify any differences in their structure or content. By strategically analyzing the changes, such as additions, removals, or updates of elements, React determines the most efficient way to update the actual DOM. This process helps minimize unnecessary updates, optimizing performance and ensuring a responsive user experience in React applications.