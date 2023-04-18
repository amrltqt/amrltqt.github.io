+++
title = "Analytics on the web stack - Part 2"
date = "2023-04-18"
+++

Ok, I need to admit that building a dashboard on using simply the current web stack request a bit more development than what I though at the beginning.

Nothing insurmountable, but still some difficulties.

Here is a small demonstration of what you can achieve with a bit of js and react. I took me honnestly few hours to build and I'm quite happy with the result.

![screenshot](images/behavior.gif)

To resume in few points what I learn doing this experiment:

- React states and hooks are awesome for drilling a dataset
- Chartjs is a really marvellous library
- Few base widget are essential to be comfortable
- Javascript can produce transformation, but I'm unsure about the acceptable level

## React states are awesome to drill a dataset

The way react manage state is really appreciated when it comes to manipulating fields are related components. By nature, analytics dashboard are build like trees with one or few raw datasets at its root.

I often call a dashboard the last mile of a data pipeline transportation.

So in a way it behaves exactly like a data pipeline, as a DAG of computing units. The main difference is that computing is on the edge, on client side and that as part of the source there are the filter and parameters that the user provide.

What's important is that you load the data that you need once and only once. Then compute what you want.

```js
export default function App() {
  const { table, fields } = useData("raw.arrow");

  if (!table || !fields) {
    return <div>Loading..</div>;
  }

  /** The root of the dashboard */
  return <main>...</main>;
}
```

Based on the previous article example the root of the dashboard is made of the App application. When `useData("raw.arrow")` is resolve, data is fully available and you can apply filtering, transformation or copy of the data.

To have a dynamic refresh, just use react `useState()` to maintain the filters that you want to apply on your data.

We'll build an entire article about that.

The complete example is here in the meantime:

```js
const { table: raw_table, fields } = useData("raw.arrow");

const [metrics, setMetrics] = useState(["a", "b"]);
const [minDate, setMinDate] = useState(null);
const [maxDate, setMaxDate] = useState(null);

if (!raw_table || !fields) {
  return <div>Loading..</div>;
}

let table = raw_table.filter((row) => {
  if (!minDate || !maxDate) return true;
  let current = DateTime.fromMillis(row.date);
  return minDate.ts < current.ts && current.ts < maxDate.ts;
});

const groups = group_by_sum_categories(table, metrics);
```

## Chartjs is a really marvellous library

I have no words. I just discover the lib and I'm amazed by the rendering, the speed and the level of configuration available.

The react-chartjs-2 library is a really simple wrapper that do not affect the raw capabilities of the chart.js lib. There is exactly what I need in term of charts and I'm just starting discovering the capabilities.

100% sure that I'm going to spend much more time on it in the coming weeks.

## Few base widget are essential to be comfortable

Rebuilding widgets, especially with chart.js is really comfortable. I don't think that it's worth rebuilding a wrapper on top of it. Even for less technicall people, it will just affect badly their own experience with the lib.

However, few widgets would be higly appreciated out of the box.

- filtering
- layouts
- transformation utilities

Filtering is the main difficulty honnestly, especially dates. I'm not sure how to deal with that now. I'm inspired by existing dashboard capabilities but I'm missing a bit of experience building it.

Layouts with flex, grid are not necessarily easy and could be cumbersome. Tailwindcss help me a lot to simplify that but to win a bit of speed while developping I would love to have a very directed layout component. Like the one provided by android application for example, but in react.

Transformations are simple in JS, but honnestly, rebuilding a group by everytime you need it is an error prone process. I'm thinking about using lodash to do those kind of thing.
Let's see where it goes.

## Javascript can produce transformation, but I'm unsure about the acceptable level

I'm not sure that I can accept to have group by in JS. Honnestly it's a purist question, but if you have the opportunity to precompute things and just feed your dashboard with the pre-computed data why don't do it?

I think there is a balance to find.

Most of modern web client are absolutely powerful compared to the past 5 / 10 years and we have to consider that. We can offload some computing on the edge, at the last moment, only when needed. But at the same time a simple group by could be reprocessed several thousands of time on the client side just by re-rendering.

On the other side, leaving all the work on your server when the dashboard is not really used is a waste of ressources. Usually you think about autoscaling, distribution, having the good infrastructure to handle the load and it could be totally useless.

I don't have a clear answer for the "what's the right balance" question except that we have to start measuring the usage to find the good compromise.

See you in the next part.
