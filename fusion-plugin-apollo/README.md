# fusion-plugin-apollo

[![Build status](https://badge.buildkite.com/4c8b6bc04b61175d66d26b54b1d88d52e24fecb1b537c54551.svg?branch=master)](https://buildkite.com/uberopensource/fusionjs)

Fusion.js plugin for universal rendering with React and Apollo

This package provides universal rendering for Fusion.js applications leveraging GraphQL. 

The plugin will perform graphql queries on the server, thereby rendering your applications initial HTML view on the server before sending it to the client. Additionally this plugin will also provide initial state hydration on the client side.

---

# Table of contents

- [Installation](#installation)
- [Usage](#usage)
- [API](#api)
  - [Registration API](#registration-api)
    - [`ApolloClientToken`](#apolloclienttoken)
    - [`GraphQLSchemaToken`](#graphqlschematoken)
    - [`GraphQLEndpointToken`](#graphqlendpointtoken)
    - [`GetApolloClientCacheToken`](#GetApolloClientCacheToken)
    - [`ApolloClientCredentialsToken`](#apolloclientcredentialstoken)
    - [`GetApolloClientLinksToken`](#getapolloclientlinkstoken)
    - [`GetDataFromTreeToken`](#getdatafromtreetoken)
    - [`ApolloClientResolversToken`](#apolloclientresolverstoken)
    - [`ApolloBodyParserConfigToken`](#apollobodyparserconfigtoken)
    - [`ApolloContextToken`](#apollocontexttoken)
  = [GQL Macro]($gql)

---

### Installation

```sh
yarn add fusion-plugin-apollo
```

---

### Usage

```js
// ./src/main.js
import React from 'react';
import App from 'fusion-react';
import {RenderToken} from 'fusion-core';

import {
  ApolloRenderEnhancer,
  ApolloClientPlugin,
  ApolloClientToken,
  GraphQLSchemaToken, 
} from 'fusion-plugin-apollo';

export default function() {
  const app = new App(<Hello />);
  app.enhance(RenderToken, ApolloRenderEnhancer);
  app.register(ApolloClientToken, ApolloClientPlugin);
  if (__NODE__) {
    app.register(GraphQLSchemaToken, YourGraphQLSchema);
  }
  return app;
}
```

### Usage with external server

When schema file is not provided, the plugin will not run graphQL server locally. The endpoint of the external graphQL server can be then provided using `GraphQLEndpointToken`.

```js
// ./src/main.js
import React from 'react';
import App from 'fusion-react';
import {RenderToken} from 'fusion-core';

import {
  ApolloRenderEnhancer,
  ApolloClientPlugin,
  ApolloClientToken,
  GraphQLSchemaToken, 
} from 'fusion-plugin-apollo';

export default function() {
  const app = new App(<Hello />);
  app.enhance(RenderToken, ApolloRenderEnhancer);
  app.register(ApolloClientToken, ApolloClientPlugin);
  app.register(GraphQLEndpointToken, 'http://website.com/graphql');
  return app;
}
```

### Loading GraphQL Queries/Schemas

fusion-plugin-apollo ships with a compiler plugin that lets you load graphql queries and schemas with the `gql` macro. 
This macro takes a relative path argument and returns the query/schema as a string. 

NOTE: Because this is a build time feature, the path argument must be a string literal. Dynamic paths are not supported.

```js
import {gql} from 'fusion-plugin-apollo';
const query = gql('./some-query.graphql');
const schema = gql('./some-schema.graphql');
```

---

### API

#### Registration API

##### ApolloClientToken

```js
import {ApolloClientToken} from 'fusion-plugin-apollo';
```

A plugin, which provides an instance of [Apollo Client](https://www.apollographql.com/docs/react/api/apollo-client.html), to be registered and used as within the Apollo Provider. You can use [fusion-apollo-universal-client](https://github.com/fusionjs/fusion-apollo-universal-client) as a barebones Apollo Client token.

```flow
type ApolloClient<TInitialState> = (ctx: Context, initialState: TInitialState) => ApolloClientType;
```


##### GraphQLSchemaToken

```js
import {GraphQLSchemaToken} from 'fusion-plugin-apollo';
```

Your graphql schema is registered on the `GraphQLSchemaToken` token. This is the result of `makeExecutableSchema` or `makeRemoteExecutableSchema` from the `graphql-tools` library.

##### GraphQLEndpointToken

```js
import {GraphQLEndpointToken} from 'fusion-plugin-apollo'; 
```

Optional - the endpoint for serving the graphql API. Defaults to `'/graphql'`. This can also be an endpoint of an external graphQL server (hosted outside fusion app).

```js
type GraphQLEndpoint = string;
```

##### `GetApolloClientCacheToken`

```js
import {GetApolloClientCacheToken} from 'fusion-apollo-universal-client';
```

Optional - A function that returns an Apollo [cache implementation](https://www.apollographql.com/docs/react/advanced/caching.html).

```js
type GetApolloClientCache = (ctx: Context) => ApolloCache
```

###### Default value

The default cache implementation uses [InMemoryCache](https://github.com/apollographql/apollo-client/tree/master/packages/apollo-cache-inmemory).

##### `ApolloClientCredentialsToken`

```js
import {ApolloClientCredentialsToken} from 'fusion-apollo-universal-client';
```

Optional - A configuration value that provides the value of credentials value passed directly into the [fetch implementation](https://github.com/github/fetch). 
The default value is `same-origin`.

```js
type ApolloClientCredentials = 'omit' | 'include' | 'same-origin'
```

##### `GetApolloClientLinksToken`

```js
import {GetApolloClientLinksToken} from 'fusion-apollo-universal-client';
```

Optional - A configuration value that provides a array of [ApolloLinks](https://www.apollographql.com/docs/link/composition.html). The default links are provided as an argument to the provided function.

```js
type GetApolloClientLinks = (Array<ApolloLinkType>) => Array<ApolloLinkType>
```

##### `GetDataFromTreeToken`

```js
import {GetDataFromTreeToken} from 'fusion-plugin-apollo';
```

Optional - A configuration value that provides an option to provide custom `getDataFromTree` function for server side rendering. Default value is [react-apollo](https://www.apollographql.com/docs/react/features/server-side-rendering#getDataFromTree) implementation but with this token you can provide proper server side rendering with [react-apollo-hooks](https://github.com/trojanowski/react-apollo-hooks#server-side-rendering).


##### `ApolloClientResolversToken`

```js
import { ApolloClientResolversToken } from "fusion-apollo-universal-client";
```

Optional - Provides the resolvers for [local state management](https://www.apollographql.com/docs/react/essentials/local-state.html).

##### `ApolloBodyParserConfigToken`

```js
import { ApolloBodyParserConfigToken } from "fusion-apollo-universal-client";
// Example for increasing the json limit
app.register(ApolloBodyParserConfigToken, {
  jsonLimit: '5mb',
});
```

Optional - Provides body parser config to koa-bodyparser for apollo-server. See https://github.com/koajs/bodyparser

##### ApolloContextToken (DEPRECATED)

```js
import {ApolloContextToken} from 'fusion-plugin-apollo';
```

DEPRECATED - A function which returns the apollo context. Defaults to the fusion context. It is not recommended to add custom overrides here. Instead, you can
provide dependencies to your resolvers using the fusion dependency injection system.

See the [Apollo Client context documentation](https://www.apollographql.com/docs/apollo-server/v2/essentials/data.html#context) for more details.

```js
type ApolloContext<T> = (ctx: Context => T) | T;
```


#### gql

```js
import {gql} from 'fusion-plugin-apollo';
```

A macro for loading graphql queries and schemas. Takes a relative path string and returns the contents of the graphql schema/query as a string.

```js
type gql = (path: string): DocumentNode 
```

- `path: string` - Relative path to the graphql schema/query file. NOTE: This must be a string literal, dynamic paths are not supported.

---