---
title: "Key React Hooks Explained: useState, useEffect, and useContext"
seoTitle: "React Hooks: useState, useEffect, useContext"
seoDescription: "Understand React hooks like `useState`, `useEffect`, and `useContext` for managing state, handling side effects, and avoiding prop drilling"
datePublished: Sat Jun 29 2024 17:15:56 GMT+0000 (Coordinated Universal Time)
cuid: cly0dvmwj000409l5gveg7n0j
slug: key-react-hooks-explained
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719681839583/1a3df9b2-0f88-4d7c-bb69-ddae349cdd4e.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1719681882579/ef191a12-1475-450e-9e80-7b48662901c9.jpeg

---

# `useState()`

The purpose of `useState` is to handle reactive data. Any <mark>data that changes in the application is called state</mark>. When the state changes you want react to update the UI, so the latest changes are reflected to the end user.

### How to use it?

```jsx
const [count, setCount] = useState(0)
```

> `useState()` returns an array with 2 values:
> 
> 1. Current state
>     
>     In our example, count is the state variable which is set to 0. This initial state can be a number, boolean, string, or object
>     
> 2. Set function to update the state.
>     
>     You can update the state in two ways:
>     
>     i. Directly passing the next state `setCount(2)` // Now, the updated state of
>     
>     count will be 2.
>     
>     ii. Using function `setCount( prevCount => prevCount + 1)` In this function, we check the current state which is 2. The value of `prevCount` will be 2, then
>     
>     we increment it by 1.
>     

**Docs:**[https://react.dev/reference/react/useState#usage](https://react.dev/reference/react/useState#usage)

# `useEffect()`

`useEffect` is a Hook in React that allows you to <mark>synchronise a component with an external system</mark>. It is a way to handle side effects, such as fetching data or subscribing to a event in a functional component.

### How to use it?

```jsx
useEffect(() => {
    console.log('count changed')
},[count])
```

Here, the \`count changed \` will be logged every time the count changes.

> `useEffect` takes two arguments:
> 
> 1. A callback function
>     
>     The callback function is called after the component has rendered.
>     
> 2. A list of dependencies
>     
>     The list of dependencies is used to determine when to re-run the effect, by comparing the current values of the dependencies to the previous values.
>     

In the callback function, you can perform any side effects, such as fetching data or subscribing to an event. You can also return a cleanup function, which is called before the component is unmounted or the effect is re-run.

> Note:
> 
> 1. If you don’t pass dependencies the effect will re-run on every render of component.
>     
> 2. When state variables are used in useEffect , make sure to include in dependency array.
>     
> 3. Make sure to handle side effects, with a cleanup function.
>     

**Docs:**[https://react.dev/reference/react/useEffect#reference](https://react.dev/reference/react/useEffect#reference)

# `useContext()`

`useContext` is a way to <mark>manage state globally without passing props</mark> down through multiple levels of the component tree.

### The Problem

> <mark>Prop Drilling:</mark> Prop drilling refers to the process of passing down props through multiple layers of components, even when some of those components do not directly use the props. We can solve this problem using `useContext()`

### How to use it?

> Three things to understand:

1. Creating context
    
    ```jsx
    const EmailContext = React.createContext(null);
    ```
    
2. Provide a value for the context using ContextName.Provider
    
    ```jsx
    function App() {
        const value = { email: "in.mohsin@outlook.com" };
    
        return (
          <EmailContext.Provider value={value}>
            <Child />
          </EmailContext.Provider>
        );
    }
    ```
    
3. Use `useContext()` in the component that needs to consume the context
    
    ```jsx
    function Child() {
        const context = useContext(EmailContext);
        return <div>The email is: {context.email}</div>;
    }
    ```
    

> When the context value is updated by the provider, the component consuming the context re-renders automatically. This allows the component to always have the latest data.

**Docs:**[https://react.dev/reference/react/useContext#usage](https://react.dev/reference/react/useContext#usage)