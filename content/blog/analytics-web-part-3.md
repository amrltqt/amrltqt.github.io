+++
title = "Analytics on the web stack - Part 3"
date = 2023-04-20
+++

Let's continue to have fun with analytics on the web stack. Today I want to discuss the dynamic filtering that we can build to serve dashboard users.

First question is, why do we want dynamic filtering?

Certainly not only for the aesthetic and the sentiment of power it brings. It's more to have focus and comparison capabilities for users and for the dashboard builder to avoid higher development complexity to prepare the data.

Dynamic filtering help to focus on specific dimension of the provided information. Like specific country performance, specific user segments or a subset of the device fleet analyzed.
It will help to display two country at a time, to check which one is the higher etc. As builder, you will not prepare several files or dataset for each combinations possible, you will let the dynamic filtering do the job. Again the story of the last miles of data analytics.

How to implement it with react?

Honnestly, it you keep it simple it's straightforward.

```js
export default function App() {
  const { table, allCountries } = useData();
  const [country, setCountry] = useState();

  const filteredData = table.filter((row) =>
    !country || (
      country && row.country.toLowerCase().includes(country.toLowerCase())
    )
  );

  return (
    <div>
      <ComboboxWidget
        values={allCountries}
        selected={country}
        setSelected={setCountry}
      />
      <Table values={filteredData}>
    </div>
  );
}
```

The main idea is to keep raw data and all the constraints in single place.
Everytime a constraint (like a filter) is updated by the user, the filtering logic is applied on the raw data. A new projection is computed and send to components to be refreshed.

It's honnestly much more simpler than what we have in classical react application.

The process could be summarized like this:

1. Load all the necessary raw data
2. Generate the filter values and their default value
3. Hydrate each component with the filtered data

Secure that every modifier of the filters are well connected with each interaction and let the flow of the rerendering do the job. With modern computer, except if you manipulate half a GB of data you shouldn't suffer a lot from transformation, recomputation and rerendering performance.
