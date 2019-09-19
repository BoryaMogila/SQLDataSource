# SQLDataSource

This package combines the power of [Knex] with the ease of use of [Apollo DataSources].

## BREAKING CHANGES IN v1.0.0

In v1.0.0 this lib has a new fluid interface that plays nicely with Knex and stays more true to the spirit of Apollo DataSources.

```js
const query = this.db.select("*").from("fruit").where({ id: 1 }).cache();

query.then(data => /* ... */ );
```

To use ( or not use ) the caching feature in v1, simply add `.cache()` to your Knex query.

Read more below about getting set up and customizing the cache controls.

## BREAKING CHANGES IN v2.0.0

In v2.0.0 this lib has a new fluid interface that plays nicely with Knex and stays more true to
the spirit of Apollo DataSources with dynamically batching and caching.

```js
// batched query
const query = this.db.select("*").from("fruit").where({ id: 1 }).load();

query.then(data => /* ... */ );
```

```js
// cached query for 5ms
const query = this.db.select("*").from("fruit").where({ id: 1 }).load(5);

query.then(data => /* ... */ );
```

To use ( or not use ) the batching feature in v2, simply add `.load()` to your Knex query.
To use ( or not use ) the caching feature in v2, simply add `.load(ttl)` to your Knex query.

Read more below about getting set up and customizing the cache controls.

## Getting Started

### Installation

To install SQLDataSource: `npm i datasource-sql`

### Usage

```js
// MyDatabase.js

const { SQLDataSource } = require("datasource-sql");

const MINUTE = 60;

class MyDatabase extends SQLDataSource {
  getFruits() {
    return this.db
      .select("*")
      .from("fruit")
      .where({ id: 1 })
      .load(MINUTE);
  }
}

module.exports = MyDatabase;
```

And use it in your Apollo server configuration:

```js
// index.js

const MyDatabase = require("./MyDatabase");

const knexConfig = {
  client: "pg",
  connection: {
    /* CONNECTION INFO */
  }
};

const db = new MyDatabase(knexConfig);

const server = new ApolloServer({
  typeDefs,
  resolvers,
  cache,
  context,
  dataSources: () => ({ db })
});
```

### Caching ( .load( ttl ) )

If you were to make the same query over the course of multiple requests to your server you could also be making needless requests to your server - especially for expensive queries.

SQLDataSource leverages Apollo's caching strategy to save results between requests and makes that
available via `.load(ttl)`.

This method accepts one OPTIONAL parameter, `ttl` that is the number of seconds to retain the data in the cache.
If you don't setup `ttl` work only batching.

## SQLDataSource Properties

SQLDataSource is an ES6 Class that can be extended to make a new SQLDataSource and extends Apollo's base DataSource class under the hood.

( See the Usage section above for an example )

Like all DataSources, SQLDataSource has an initialize method that Apollo will call when a new request is routed.

If no cache is provided in your Apollo server configuration, SQLDataSource falls back to an InMemoryLRUCache like the [RESTDataSource].

### context

The context from your Apollo server is available as `this.context`.

### db

The instance of knex you reference in the constructor is made available as `this.db`.

## Debug mode

To enable more enhanced logging via [knex-tiny-logger], set `DEBUG` to a truthy value in your environment variables.

## Contributing

All contributions are welcome!

Please open an issue or pull request.

[knex]: https://knexjs.org/
[apollo datasources]: https://www.apollographql.com/docs/apollo-server/features/data-sources.html
[dataloader]: https://github.com/facebook/dataloader
[inmemorylrucache]: https://github.com/apollographql/apollo-server/tree/master/packages/apollo-server-caching
[restdatasource]: https://www.apollographql.com/docs/apollo-server/features/data-sources.html#REST-Data-Source
[knex-tiny-logger]: https://github.com/khmm12/knex-tiny-logger
