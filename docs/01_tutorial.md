# Tutorial

This tutorial is divided into three levels. This will help you reason about **state management complexity** and how different abstractions fits with dealing with different levels of complexity.

How exactly to evaluate **state management complexity** of an application is difficult, especially before hand. You want to get started quickly and efficiently, but as your application grows and you learn more about its nature, the tool needs to provide lower level and flexible APIs.

A good way to evaluate **state management complexity** is to use the component tree size and depth in relation to consumption of across these component trees. So called "shared state". In other words, the more complex UI and the more state shared across that complex UI, the more **state management complexity** you have.

Enough theory, let's look at it from a practical perspective.

## Level 1: Just give me some global state

Declarative global state stores is a popular abstraction managing shared state across components. They are straight forward and solve the immediate problem of sharing state. In **Impact** you would use `globalStore` to create a reactive store:

```tsx
import { globalStore } from 'impact-app'

const useAppStore = globalStore({
    // Each key is a signal
    count: 0,
    // A getter is a lazily evaluated derived signal
    get double() {
        return this.count * 2
    },
    // Methods runs in the context of the store and
    // can change signal values, also asynchronously
    increment() {
        this.count++
    }
})

function App() {
    // Use the store in any component and it will only reconcile
    // when any accessed signals change
    const appStore = useAppStore()

    return (
        <div>
            <h1>{appStore.count}</h1>
            <h4>{appStore.double}</h4>
            <button onClick={appStore.increase}>
                Increase
            </button>
        </div>
    )
}
```

The `globalStore` is just an accessible abstraction over lower level APIs like `signal`, `store` and `globalContext`. Each key you define in the global state store is a signal, where *getters* are lazily evaluated derived signals. Components will only reconcile when the consumed signal changes. The state defined in the store is protected so changes to it can only be made from within the store.

This level of managing state is close to libraries like [jotai](https://jotai.org/) and [zustand](https://github.com/pmndrs/zustand).

The `globalStore` might be all you need for your application, but as the store itself grows you will benefit from moving to the next level.

## Level 2: I need to compose multiple stores

As the complexity of your state store increases you will want to split it up into different domains or namespaces. With **Impact** you can continue using the declarative store definition, but rather compose them in a `globalContext` using the `store` API.

```ts
import { globalContext, store, effect } from 'impact-app'

// Compose the stores together using the hooks pattern
const useNotifications = () => store({
    stack: [],
    add(newNotification) {
        this.stack = [newNotification, ...this.stack]
        setTimeout(() => {
            this.stack = this.stack.filter((notification) => notification !== newNotification)
        }, 2000)
    }
})

// Pass any dependencies the store might need. This being stores
// or other references
const useTodos = (notifications, api) => {
    const todos = store({
        all: [],
        get completed() {
            return this.all.filter((todo) => todo.completed)
        }
        get remaining() {
            return this.all.filter((todo) => !todo.completed)
        }
        async add(newTodo) {
            this.all = [newTodo, ...this.all]
            try {
                await api.post('/todos', newTodo)
                notifications.add({ type: 'success', text: "Todo saved" })
            } catch {
                this.all = this.all.filter((todo) => todo !== newTodo)
                notifications.add({ type: 'error', text: "Failed adding todo" })
            }
            
            
        }
    })

    // The effect runs immediately and tracks any signals you access. When
    // the signal changes, the effect runs again
    effect(() => {
        if (todos.remaining === 10) {
            notifications.add({ type: 'warning', text: "Oh oh, make sure you keep sane!" })
        }
    })

    return todos
}

// Use "globalContext" to compose together stores and related logic into a single
// global context to be consumed by any components
const useAppStore = globalContext(() => {
    const api = new Api()
    const notifications = useNotifications()
    const todos = useTodos(notifications, api)
        
    return {
        todos,
        notifications
    }
})
```

With `globalContext` you still consume a single global store, but it is composed together using the `store` API, which is the same accessible declarative API over signals as `globalStore`. The function scope of `globalContext` is used to compose together the stores however you want and you can start taking advantage of `effect` and whatever else you want to initialise and manage within your global context.

This level of managing state is close to what you know from [redux](https://redux.js.org/), [mobx](https://mobx.js.org/README.html), [overmind](https://overmindjs.org/) and the likes.

You get very far with this level of abstraction, but there is another level giving you even more control.

## Level 3: Scoped state management

In extremely complex web applications you might have multiple pages and features that has some shared state across those pages/features and some state scoped to the page/feature. This is where **Impact** gives you `context`. Unlike Reacts context this is, like `globalContext`, a reactive context. The only difference from `globalContext` is that it needs to be provided to a component tree.

At this point you can still use the `store` API, but you will more likely embrace raw signals.

```tsx
import { context, signal, cleanup } from 'impact-app'

// The page context receives props from its Provider, maybe data from a router
// used with React
const usePage = context(({ initialCount }) => {
    const initialCount = signal(initialCount)

    return {
        // We use getters to expose the signal value. This makes it readonly and
        // more accessible
        get initialCount() {
            return initialCount.value
        }
    }
})

const useFeature = context(() => {
    const { initialCount } = usePage()

    const count = signal(initialCount)

    // The context scope is reactive, which means it will only run once. You are free to define variables,
    // and side effects
    const interval = setInterval(() => {
        count.value++
    }, 1000)

    // When the provider of this context unmounts you can clean up
    cleanup(() => clearInterval(interval))

    return {
        get count() {
            return count.value
        }
    }
})

function Feature() {
    const feature = useFeature()

    return <h1>Count: {feature.count}</h1>
}

function Page() {
    return (
        <useFeature.Provider>
            <Feature />
        </useFeature.Provider>
    )
}

function App() {
    return (
        <usePage.Provider initialCount={10}>
            <Page />
        </usePage.Provider>
    )
}
```

Reading this example you see that you gain even more control of your state managent. First of all you can mount your state management with the related component tree. This simplifies lazy loading. You can also pass props to the reactive contexts, initializing state with information coming from your React application. You will also be able to consume contexts from other contexts, in this case the feature context consumes the page context.

At this level of managing complexity new possibilities open up. Things like disposing of state management as the related component tree unmounts. You can optimise consumption of data in components by giving each nested piece of data its own signal. Now that your state management is fused with React itself you can also start taking advantage of new patterns to manage data fetching.



