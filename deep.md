
---

# 📦 Understanding the Code – React Without JSX

```js
const App = () => {
  return React.createElement(
    "div",
    {},
    React.createElement("h1", {}, "Pixel Perfect Pizzas")
  );
};

const container = document.getElementById("root");
const root = ReactDOM.createRoot(container);
root.render(React.createElement(App));
```

---

## 🧠 What's Going On Here?

This is a minimal React app written **without JSX** — the fancy HTML-like syntax we usually use in React. Let's break it down in simple terms:

---

### 🔧 `const App = () => { ... }`

We're defining a **functional React component** named `App`.

Instead of returning this:

```jsx
return <h1>Hello</h1>;
```

We return this:

```js
React.createElement("h1", {}, "Pixel Perfect Pizzas");
```

Why? Because React can run without JSX. JSX is just a cleaner way of writing UI components.

---

### 🧱 `React.createElement(...)`

This function builds elements in the **virtual DOM**.

The syntax:

```js
React.createElement(type, props, ...children);
```

So:

```js
React.createElement("h1", {}, "Pixel Perfect Pizzas");
```

Means:

> “Hey React, create an `<h1>` element, no props, and put this text inside: *Pixel Perfect Pizzas*.”

Then we wrap it all inside a `<div>`:

```js
React.createElement("div", {}, React.createElement(...));
```

So the final output looks like this in the browser:

```html
<div>
  <h1>Pixel Perfect Pizzas</h1>
</div>
```

---

### 🪄 Why Not Just Use JSX?

Because **JSX is syntactic sugar**. It looks nice and reads better, but under the hood, React turns this:

```jsx
<div><h1>Pixel Perfect Pizzas</h1></div>
```

Into exactly this:

```js
React.createElement("div", {}, React.createElement("h1", {}, "Pixel Perfect Pizzas"));
```

Knowing this helps you understand how React actually works behind the scenes.

---

### 🔌 The Final Part: Rendering

```js
const container = document.getElementById("root");
const root = ReactDOM.createRoot(container);
root.render(React.createElement(App));
```

Here's what’s happening:

1. Grab the `<div id="root">` from your HTML.
2. Create a React root.
3. Tell React to render the `App` component inside that root.

---

## 🤓 TL;DR

* You're manually doing what JSX does automatically.
* `React.createElement` is the raw way React builds elements.
* Understanding this gives you deeper insight into how React works.
* JSX just makes your life easier once you know the basics.

---

### 💡 Want to See the JSX Version?

Here’s what the same component would look like using JSX:

```jsx
const App = () => {
  return (
    <div>
      <h1>Pixel Perfect Pizzas</h1>
    </div>
  );
```



```jsx
// UserProfileCard component using React.createElement
const UserProfileCard = (props) => {
  // The 'return' statement describes the UI structure for this component.
  // React.createElement is used to create virtual DOM elements.
  // Its arguments are: type (e.g., "div", "img"), props (object), children (more elements or text).
  return React.createElement(
    "div", // 1. The main wrapper element: an HTML <div>
    { className: "user-profile-card" }, // Props for the <div>: assigns a CSS class
    
    // Children of the main <div>:
    React.createElement(
      "img", // 2. An image element: <img>
      {
        src: props.imageUrl, // Image source, passed via component props
        alt: props.name, // Alternative text for the image, also from props
        className: "profile-image", // CSS class for the image
      }
    ),
    
    React.createElement(
      "h2", // 3. A heading element for the user's name: <h2>
      { className: "user-name" }, // CSS class for the name heading
      props.name // The actual name text, passed via component props
    ),
    
    React.createElement(
      "p", // 4. A paragraph element for the user's title: <p>
      { className: "user-title" }, // CSS class for the title paragraph
      props.title // The user's title text, passed via component props
    ),
    
    React.createElement(
      "button", // 5. A button element: <button>
      {
        className: "contact-button", // CSS class for the button
        onClick: () => alert(`Contacting ${props.name}!`), // Event handler for button click
      },
      "Contact" // Text displayed inside the button
    )
  );
};

};
```

---
```js
// Example data for the user profile
const userData = {
  name: "Ali Ahmadi",
  title: "Software Engineer",
  imageUrl: "https://via.placeholder.com/150", // Placeholder image URL
};

// 1. Get the DOM element where the React app will be mounted.
//    This assumes you have an HTML element like <div id="root"></div> in your index.html.
const container = document.getElementById("root");

// 2. Create a React root. This is the entry point for your React application.
//    ReactDOM.createRoot is the modern way to initialize a React app.
const root = ReactDOM.createRoot(container);

// 3. Render the UserProfileCard component into the root.
//    Here, we pass the 'userData' object as props to our UserProfileCard component.
//    React.createElement is used again to create an instance of our component.
root.render(React.createElement(UserProfileCard, userData));
```
