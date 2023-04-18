+++
title = "Analytics on the web stack - Part 1"
date = "2023-04-16"
+++

First day of experimenting around loading data, building charts and connected data viz.
Very good sensation, interesting results.

Let's elaborate on how to simplify data exposition on your frontend application.

1. I want a static blob of data that I can expose through any S3 compatible storage.
2. Integrated schema to simplify data exploration on the frontend.
3. Memory / reading efficient

The very good candidate for that is to use an arrow IPC file. This are some risks with this solution. Arrow IPC by nature is not a storage medium, parquet would be a prefered medium. Why? Just because IPC do not enforce compatibility and is meant to be use between two process of a same software.
As I aims to load data in the morning, and expose it only for the current day I do not assume too much risks.

## Build the file

Let's build an IPC file with python, pandas, pyarrow.

I'm working in python 3.11 with the following dependencies

```requirements.txt
pandas
faker
pyarrow
```

Here is the script I use to generate an IPC example file.

```python
import pandas
import numpy
import pyarrow
import datetime

from faker import Faker

fake = Faker()
Faker.seed(5478)

countries = [
    fake.country()
    for _ in range(5)
]

data = [
    {
        "id": i,
        "date": datetime.date(2023, 1, 1) + datetime.timedelta(days=i),
        "a": fake.pyint(max_value=200),
        "b": fake.pyint(max_value=200),
        "categ": countries[fake.pyint(max_value=len(countries) - 1)]
    }
    for i in range(100)
]


df = pandas.DataFrame(data)

df["date"] = pandas.to_datetime(df["date"])
df["id"] = df["id"].astype(numpy.int32)
df["a"] = df["a"].astype(numpy.int32)
df["b"] = df["b"].astype(numpy.int32)


def export(df, to):
    table = pyarrow.Table.from_pandas(df)
    with pyarrow.OSFile(to, 'wb') as sink:
        with pyarrow.ipc.new_stream(sink, table.schema) as writer:
            for batch in table.to_batches():
                writer.write_batch(batch)

export(df, './vite-project/public/raw.arrow')
```

Note:

1. I cast the types after the dataframe creation to secure a correct translation on the Js side afteward.
2. The export tasks is straightforward thanks to the clean integration of pyarrow / pandas

It's easy to imagine that this process is started by airflow after refreshing data in your Snowflake instance for instance.

## Access data in Front end

The previous task assume that you created a vite application with react.
Check https://vitejs.dev/guide/ to do the same.

In the vite project you usually have public folder for static assets like images, fonts. Let's add the arrow files generated there. In production it means that we publish the IPC to the CDN, the Nginx or whatever you're using to expose static files.

The code to load the IPC file is straightforward.

```shell
npm i apache-arrow
```

To access data I created this simple hook

```js
import { tableFromIPC } from "apache-arrow";

function* iterrows(table) {
  let currentIndex = 0;
  for (currentIndex = 0; currentIndex < table.numRows; currentIndex++) {
    let row = {};
    table.schema.fields.forEach((field) => {
      row[field.name] = table.getChild(field.name).get(currentIndex);
    });

    yield row;
  }
}

function useData(target) {
  const [table, setTable] = useState(null);
  const [fields, setFields] = useState(null);

  useEffect(() => {
    tableFromIPC(fetch(target))
      .then((table) => {
        setFields(table.schema.fields);
        setTable(Array.from(iterrows(table)));
      })
      .catch(console.error);
  }, []);

  return {
    table,
    fields: fields?.map((field) => field.name),
  };
}
```

This hook load the data from the `target` IPC file and create a arrow table.
To simplify reuse of the data I choose to iterate over values to create an array of object that contains each "rows" of data.

To use this code you can call useData in your component like this:

```js
export default function App() {
  const { table, fields } = useData("raw.arrow");

  if (!table || !fields) {
    return <div>Loading..</div>;
  }

  console.log(fields);
  console.table(table);

  return null;
}
```

Let's see how to build on a dashboard for another part.
