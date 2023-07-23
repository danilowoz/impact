# impact-app

> Rich Single Page Application Framework for React

## Background

At [CodeSandbox](https://codesandbox.io) we write most of our code in an object oriented paradigm. This includes projects like **pitcher**, **VSCode extension**, **@codesandbox/pitcher-client** and **@codesandbox/api**. This in combination with creating rich single page applications with a high degree of client side managed state, we wanted to explore a tool where we can more easily move between object oriented codebases and solve some of the challenges of building rich single page applications.

## Who is this for?

Even though you love React for its ability to compose UIs, you feel frustrated when it gets down to business of exposing and consuming application wide state, organising code into domains and making it all discoverable for new developers. Maybe you are using Mobx already, but want to avoid passing class instances all over the place or maybe you are using Redux and miss the ability to encapsulate asynchronous flows and private state.

This framework is not built for solutions like [Next JS](https://nextjs.org/) or [Remix](https://remix.run/), which are more "e-commerce" types of experiences which are primarily data and route driven applications. This framework is for rich single page applications where you are building "editor" type of experiences on the web using React.

## Documentation

- [Getting Started](./docs/01_Getting_Started.md)
- [Hello World](./docs/02_Hello_World.md)
- [Injection In Depth](./docs/03_Injection_In_Depth.md)

