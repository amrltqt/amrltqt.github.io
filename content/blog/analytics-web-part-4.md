+++
title = "Analytics on the web stack - Part 4"
date = 2023-04-29
+++

After a difficult come back at work this week, I didn't find the time to speak about layouting.

When you're dealing with a classical website, you have to deal with a large amount of custom layouts. In an analytical dashboard, it's more or less always simple cards with the minimum of customization.

For us it's cool, it's a new opportunity to simplify the development flow. We can focus on iterating on vizualisation and on the information shared instead of spending time building the app.

To deal well with layouting you need to configure two kind of components:

- layout managers
- widget containers

Layout managers leverage the children reserved word to distribute the children components inside the layout.

Let's have a look of the `FlexRowLayout` component that help to align correctly components in a row whatever your screen size.

```jsx
export function FlexRowLayout({ children }) {
  return (
    <div className="flex space-x-2">
      {children.map((child) => (
        <div className="flex-1">{child}</div>
      ))}
    </div>
  );
}
```

The layout is made of a container, the higher level `div` and a collection of lower level container that wrap the children components.

The high level component introduce the flex CSS property and each subcomponent secure that there is an equal repartition of the managed widgets.

The space-x-2 is really cool, it secure that all spaces will be 2. By reusing this component you will bring consistency in your dashboard design.

The same strategy apply well for a column oriented layout.

```jsx
export function FlexColLayout({ children }) {
  return (
    <div className="flex flex-col space-y-2">
      {children.map((child) => (
        <div>{child}</div>
      ))}
    </div>
  );
}
```

Using it simplify the flow of your application. All the children the FlexRowLayout will be in the row, well spaced and you don't care anymore about flex stuff.

```jsx
export default function App() {
  return (
    <main>
      <FlexRowLayout>
        <div className="bg-white p-2">
          <p>Number of orders</p>
          <p className="text-xl font-medium">2.6k</p>
        </div>
        <div className="bg-white p-2">
          <p>Customer satisfaction</p>
          <p className="text-xl font-medium">4.3</p>
        </div>
      </FlexRowLayout>
    </main>
  );
}
```

If you create a container to provide a standard visual experience on all your component, the combination of both will start to give you a lot of productivity.

Let's build a Card as a container.

```jsx
export function Card({ children }) {
  return (
    <div className="bg-white p-2 rounded shadow ring-gray-300">{children}</div>
  );
}
```

Again the use of the children props give us power.

```jsx
export default function App() {
  return (
    <main>
      <FlexRowLayout>
        <Card>
          <p>Number of orders</p>
          <p className="text-xl font-medium">2.6k</p>
        </Card>
        <Card>
          <p>Customer satisfaction</p>
          <p className="text-xl font-medium">4.3</p>
        </Card>
      </FlexRowLayout>
    </main>
  );
}
```

The visual experience is standardized by reusing both layout and container and provide you productivity. You avoid cognitive load of replicating classes and dealing with css properties everywhere.

And that's _exactly_ what we want when we build a dashboard that focus on sharing information through charts and values.

In the end you application root will combine layout, containers and the configured widgets in a simple tree of components.

```jsx
export default function App() {
  return (
    <FlexColLayout>
      <FlexRowLayout>
        <Card>
          <p>Number of orders</p>
          <p className="text-xl font-medium">2.6k</p>
        </Card>
        <Card>
          <p>Customer satisfaction</p>
          <p className="text-xl font-medium">4.3</p>
        </Card>
      </FlexRowLayout>
      <Card>
        <h2>Result table</h2>
        <table> ... </table>
      </Card>
    </FlexColLayout>
  );
}
```

See you in a next part, hope it brings you inspiration and will help you to build dashboard with react.
