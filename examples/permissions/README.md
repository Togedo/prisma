# Permissions

This example demonstrates how to implement **permission rules** in combination with an email-password-based authentication workflow. Feel free to use it as a template for your own project!

## Overview

This directory contains a GraphQL server (based on [`graphql-yoga`](https://github.com/graphcool/graphql-yoga/)) which connects to a Prisma database service.

```
.
├── README.md
├── database
│   ├── datamodel.graphql
│   └── prisma.yml
├── package.json
├── src
│   ├── generated
│   │   └── prisma.graphql
│   ├── auth.js
│   ├── index.js
│   ├── schema.graphql
│   └── utils.js
├── yarn.lock
├── .env
└── .graphqlconfig.yml
```

## Get started

### 0. Prerequisites

You need to have the following things installed:

* Node.js 8 (or higher)
* Yarn (or npm)
* Docker (only for deploying locally)

### 1. Download the example

Clone the Prisma monorepo and navigate to this directory or download _only_ this example with the following command:

```sh
curl https://codeload.github.com/graphcool/prisma/tar.gz/master | tar -xz --strip=2 prisma-master/examples/1.0/permissions
cd 1.0/permissions
```

### 2. Deploy the Prisma database service

You can now [deploy](https://www.prismagraphql.com/docs/reference/cli-command-reference/database-service/prisma-deploy-kee1iedaov) the Prisma service:

```sh
yarn install
yarn prisma deploy
```

> Note: Whenever you make changes to files in the `database` directory, you need to invoke `prisma deploy` again to make sure your changes get applied to the running service.

### 3. Deploy the GraphQL server

Your GraphQL web server that's powered by [`graphql-yoga`](https://github.com/graphcool/graphql-yoga/) is now ready to be deployed. This is because the Prisma database service it connects to is now available.

```sh
yarn start
```

The server is now running on [http://localhost:4000](http://localhost:4000).

## Testing the service

The easiest way to test the deployed service is by using a [GraphQL Playground](https://github.com/graphcool/graphql-playground).

### Open a Playground

You can either start the [desktop app](https://github.com/graphcool/graphql-playground) via

```sh
yarn playground
```

Or you can open a Playground by navigating to [http://localhost:4000](http://localhost:4000) in your browser.

### Register users with the `signup` mutation

You can send the following mutation in the Playground to create a new `User` node and at the same time retrieve an authentication `token` for it:

```graphql
mutation {
  signup(
    email: "alice@graph.cool"
    password: "graphql"
  ) {
    token
    user {
      id
    }
  }
}
```

If no `admin` field is set, the role defaults to `CUSTOMER`. Create users with the `ADMIN` role by setting `admin` to `true`:

```graphql
mutation {
  signup(
    email: "super_admin@graph.cool"
    password: "12345"
    admin: true
  ) {
    token
    user {
      id
    }
  }
}
```

### Logging in an existing user with the `login` mutation

This mutation will log in an _existing_ user by requesting a new authentication `token` for her:

```graphql
mutation {
  login(
    email: "alice@graph.cool"
    password: "graphql"
  ) {
    token
  }
}
```

### Checking whether a user is currently logged in with the `me` query

For this query, you need to make sure a valid authentication token is sent along with the `Bearer `-prefix in the `Authorization` header of the request. Inside the Playground, you can set HTTP headers in the bottom-left corner:

![](https://i.imgur.com/BLNI8z1.png)

Once you've set the header, you can send the following query to check whether the token is valid:

```graphql
{
  me {
    id
    email
  }
}
```

If the token is valid, the server will return the `id` and `email` of the `User` node that it belongs to.

### Change the password with the `updatePassword` mutation

This mutation changes the password of the authenticated user. Make sure the Authorization header is set:

```graphql
mutation {
  updatePassword(
    oldPassword: "graphql"
    newPassword:"dKt9kAn6gkq"
  ) {
    id
  }
}
```

You can verify the password change by trying the login mutation with the new password.

Admin users can also change the password of other users. Make sure the provided Authorization token is obtained from a `login` mutation of a user with the `ADMIN` role:

```graphql
mutation {
  updatePassword(
    userId: "cjcaldr891d1d0180hl8lb1lp"
    newPassword:"test"
  ) {
    id
    email
  }
}
```

### Create Posts via `createPost` mutation

With this mutation authorized users can create a new post. Make sure the `Authorization` header is set:

```graphql
mutation {
  createPost(
    title: "Observation of Gravitational Waves from a Binary Black Hole Merger"
  ) {
    id
  }
}
```

### `updatePosts` mutation

With this mutation users with the default `CUSTOMER` role can change their own posts and users with the `ADMIN` role can also change postst of other users:

```graphql
mutation {
  updatePost(
    id: "cjcanspge1e4z01802of5gqp1"
    title: "Hyperloop Alpha - SpaceX"
  ) {
    title
  }
}
```
