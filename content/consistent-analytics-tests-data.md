+++
title = "Build a consistent test set for analytics"
date = 2023-04-30
+++

I've few algorithm that I want to test this days. For that I need several entities objects like sales, customer, products data.

The constraint is that, _obviously_ all entities are correctly linked between each other.

This post will demonstrate how I build a small pipeline to:

- generate a consistent set of entities
- start computing small things

Let's start with a small utility file, calls it `utils.py` for example.

```python
import os
import pandas as pd

from typing import Callable
from dotenv import load_dotenv

load_dotenv()

STORAGE = os.environ.get('ANALYTICS_STORAGE', '/tmp/analytics/storage')

def load_dataset(name: str):
    dataset_file = os.path.join(STORAGE, f"{name}.parquet")
    return pd.read_parquet(dataset_file)

def dataset(name: str, depends: list[str] | None = None):
    def decorator(func: Callable[[], pd.DataFrame]):
        def wrapper(*args, **kwargs):
            if depends:
                dependencies = [
                    load_dataset(name)
                    for name in depends
                ]
                df = func(*args, *dependencies ,**kwargs)
            else:
                df = func(*args, **kwargs)

            output_file = os.path.join(STORAGE, f"{name}.parquet")

            if not os.path.exists(STORAGE):
                os.makedirs(STORAGE)

            df.to_parquet(output_file, index=False)

            return df
        return wrapper
    return decorator
```

The `dataset` function is a decorator. You can use it to add behavior to your own custom functions. This one will just take the dataframe that _should_ be returned by your own function and write the content in a parquet file, in a storage folder.

It includes a very nice feature to load dependencies of already registered dataset to inject them as a dependency for the current function.

Create a `customers.py` file and reuse the following content to build your own customers dataset.

```python
from faker import Faker
import pandas

from analytics.utils import dataset

@dataset(name='customers')
def generate_customers(num_customers) -> pandas.DataFrame:
    fake = Faker()

    customers = []
    for _ in range(num_customers):
        name = fake.company()
        email = fake.email()
        phone = fake.phone_number()
        address = fake.address()
        country = fake.country()
        customers.append((name, email, phone, address, country))

    df = pandas.DataFrame(customers, columns=['name', 'email', 'phone', 'address', 'country'])
    return df

if __name__ == "__main__":
    generate_customers(5)
```

Generate the parquet file by just executing your customers scripts.

Add a product reference dataset.

```python
from faker import Faker
import pandas as pd
import random

from analytics.utils import dataset

@dataset(name="products")
def generate_products(num_products):
    fake = Faker()

    categories = ['Electronics', 'Clothing', 'Home', 'Sports', 'Books']

    products = []
    for _ in range(num_products):
        name = fake.text(max_nb_chars=50)
        category = random.choice(categories)
        price = round(random.uniform(10, 1000), 2)
        quantity = random.randint(1, 100)
        products.append((name, category, price, quantity))

    df = pd.DataFrame(products, columns=['name', 'category', 'price', 'quantity'])
    return df


if __name__ == "__main__":
    generate_products(30)
```

The challenge is to combine both of them an generates sales values.

We are going to introduce the dependency description there to inject product and customer in the sales generation process.

```python
import pandas
import numpy as np
from faker import Faker

from analytics.utils import dataset

@dataset(name="sales", depends=["customers", "products"])
def generate_sales(num_sales, customers_df, products_df):
    fake = Faker()
    sales = []

    for _ in range(num_sales):
        # Choisir un client aléatoire
        customer = customers_df.sample(n=1)

        # Choisir un produit aléatoire
        product = products_df.sample(n=1)

        # Générer une date de vente aléatoire
        date = fake.date_between(start_date='-1y', end_date='today')

        # Générer une quantité aléatoire de produit vendu
        quantity = np.random.randint(1, 10)

        # Calculer le montant total de la vente
        total_amount = product['price'].values[0] * quantity

        # Ajouter les informations de vente à la liste de ventes
        sales.append((
            date,
            customer['name'].values[0],
            customer['email'].values[0],
            product['name'].values[0],
            quantity,
            product['price'].values[0],
            total_amount
        ))

    # Créer un DataFrame pandas avec les informations de vente
    df = pandas.DataFrame(sales, columns=[
        'date',
        'customer_name',
        'customer_email',
        'product_name',
        'quantity',
        'price',
        'total_amount'
    ])

    return df

if __name__ == "__main__":
    generate_sales(100)
```

By executing correctly the scripts you'll have a correct sales test files to start modeling.

Example:

```python
import pandas
from analytics.utils import dataset


@dataset(name="max_sales_per_customers", depends=["sales"])
def customer_sales(sales: pandas.DataFrame):
    customer_sales = (
        sales[["customer_name", "total_amount"]]
            .groupby("customer_name")
            .sum()
    )

    return customer_sales


if __name__ == "__main__":
    customer_sales()
```

You have now a quite correct framework to build pipelines that is fed with consistent data.
