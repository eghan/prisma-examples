# GraphQL Server with Authentication & Permissions

This example shows how to implement a **GraphQL server with an email-password-based authentication workflow and authentication rules**, based on Prisma, [graphql-yoga](https://github.com/prisma/graphql-yoga), [graphql-shield](https://github.com/maticzav/graphql-shield) & [GraphQL Nexus](https://nexus.js.org/).

## How to use

### 1. Download example & install dependencies

Clone this repository:

```
git clone git@github.com:prisma/prisma-examples.git
```

Install npm dependencies:

```
cd prisma-examples/typescript/graphql-auth
npm install
```

Note that this also generates Prisma Client JS into `node_modules/@prisma/client` via a `postinstall` hook of the `@prisma/client` package from your `package.json`.

### 2. Start the GraphQL server

Launch your GraphQL server with this command:

```
npm run dev
```

Navigate to [http://localhost:4000](http://localhost:4000) in your browser to explore the API of your GraphQL server in a [GraphQL Playground](https://github.com/prisma/graphql-playground).

### 5. Using the GraphQL API

The schema that specifies the API operations of your GraphQL server is defined in [`./src/schema.graphql`](./src/schema.graphql). Below are a number of operations that you can send to the API using the GraphQL Playground.

Feel free to adjust any operation by adding or removing fields. The GraphQL Playground helps you with its auto-completion and query validation features.

#### Retrieve all published posts and their authors

```graphql
query {
  feed {
    id
    title
    content
    published
    author {
      id
      name
      email
    }
  }
}
```

<Details><Summary><strong>See more API operations</strong></Summary>

#### Register a new user

You can send the following mutation in the Playground to sign up a new user and retrieve an authentication token for them:

```graphql
mutation {
  signup(name: "Sarah", email: "sarah@prisma.io", password: "graphql") {
    token
  }
}
```

#### Log in an existing user

This mutation will log in an existing user by requesting a new authentication token for them:

```graphql
mutation {
  login(email: "sarah@prisma.io", password: "graphql") {
    token
  }
}
```

#### Check whether a user is currently logged in with the `me` query

For this query, you need to make sure a valid authentication token is sent along with the `Bearer`-prefix in the `Authorization` header of the request:

```json
{
  "Authorization": "Bearer __YOUR_TOKEN__"
}
```

With a real token, this looks similar to this:

```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiJjanAydHJyczFmczE1MGEwM3kxaWl6c285IiwiaWF0IjoxNTQzNTA5NjY1fQ.Vx6ad6DuXA0FSQVyaIngOHYVzjKwbwq45flQslnqX04"
}
```

Inside the Playground, you can set HTTP headers in the bottom-left corner:

![](https://imgur.com/ToRcCTj.png)

Once you've set the header, you can send the following query to check whether the token is valid:

```graphql
{
  me {
    id
    name
    email
  }
}
```

#### Create a new draft

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground.

```graphql
mutation {
  createDraft(
    title: "Join the Prisma Slack"
    content: "https://slack.prisma.io"
  ) {
    id
    published
  }
}
```

#### Publish an existing draft

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground. The authentication token must belong to the user who created the post.

```graphql
mutation {
  publish(id: "__POST_ID__") {
    id
    published
  }
}
```

> **Note**: You need to replace the `__POST_ID__`-placeholder with an actual `id` from a `Post` item. You can find one e.g. using the `filterPosts`-query.

#### Search for posts with a specific title or content

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground. 

```graphql
{
  filterPosts(searchString: "graphql") {
    id
    title
    content
    published 
    author {
      id
      name
      email
    }
  }
}
```

#### Retrieve a single post

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground. 

```graphql
{
  post(id: "__POST_ID__") {
    id
    title
    content
    published
    author {
      id
      name
      email
    }
  }
}
```

> **Note**: You need to replace the `__POST_ID__`-placeholder with an actual `id` from a `Post` item. You can find one e.g. using the `filterPosts`-query.

#### Delete a post

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground. The authentication token must belong to the user who created the post.

```graphql
mutation {
  deletePost(id: "__POST_ID__") {
    id
  }
}
```

> **Note**: You need to replace the `__POST_ID__`-placeholder with an actual `id` from a `Post` item. You can find one e.g. using the `filterPosts`-query.

</Details>

### 6. Changing the GraphQL schema

To make changes to the GraphQL schema, you need to manipulate the [`Query`](./src/resolvers/Query.ts) and [`Mutation`](./src/resolvers/Mutation.ts) types. 

Note that the [`dev`](./package.json#L6) script also starts a development server that automatically updates your schema every time you save a file. This way, the auto-generated [GraphQL schema](./src/generated/schema.graphql) updates whenever you make changes in to the `Query` or `Mutation` types inside your TypeScript code.


## Evolving the app

Evolving the application typically requires four subsequent steps:

1. Migrating the database schema using SQL
1. Update your Prisma schema by introspecting the database with `prisma2 introspect`
1. Generating Prisma Client to match the new database schema with `prisma2 generate`
1. Use the updated Prisma Client in your application code

For the following example scenario, assume you want to add a "profile" feature to the app where users can create a profile and write a short bio about themselves.

### 1. Change your database schema using SQL

The first step would be to add a new table, e.g. called `Profile`, to the database. In SQLite, you can do so by running the following SQL statement:

```sql
CREATE TABLE "Profile" (
  "id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT, 
  "bio" TEXT,
  "user" TEXT NOT NULL UNIQUE REFERENCES "User"(id) ON DELETE SET NULL
);
```

To run the SQL statement against the database, you can use the `sqlite3` CLI in your terminal, e.g.:

```bash
sqlite3 test.db \
'CREATE TABLE "Profile" (
  "id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT, 
  "bio" TEXT,
  "user" TEXT NOT NULL UNIQUE REFERENCES "User"(id) ON DELETE SET NULL
);'
```

Note that we're adding a unique constraint to the foreign key on `user`, this means we're expressing a 1:1 relationship between `User` and `Profile`, i.e.: "one user has one profile".

While your database now is already aware of the new table, you're not yet able to perform any operations against it using Prisma Client. The next two steps will update the Prisma Client API to include operations against the new `Profile` table.

### 2. Introspect your database

The Prisma schema is the foundation for the generated Prisma Client API. Therefore, you first need to make sure the new `Profile` table is represented in it as well. The easiest way to do so is by introspecting your database:

```
npx prisma2 introspect
```

> **Note**: You're using [npx](https://github.com/npm/npx) to run the Prisma Framework CLI that's listed as a development dependency in [`package.json`](./package.json). Alternatively, you can install the CLI globally using `npm install -g prisma2`. When using Yarn, you can run: `yarn prisma2 dev`.

The `introspect` command creates a new `schema.prisma` file and renames the old `schema.prisma` to `schema.backup.prisma`. The updated `schema.prisma` now includes the `Profile` model and its 1:1 relation to `User`:

```prisma
model Post {
  author    User?
  content   String?
  id        Int     @id
  published Boolean @default(false)
  title     String
}

model User {
  email   String   @unique
  id      Int      @id
  name    String?
  posts   Post[]
  profile Profile?
}

model Profile {
  bio  String?
  id   Int     @id
  user User
}
```

### 3. Generate Prisma Client

With the updated Prisma schema, you can now also update the Prisma Client API with the following command:

```
npx prisma2 generate
```

This command updated the Prisma Client API in `node_modules/@prisma/client`.

### 4. Use the updated Prisma Client in your application code

You can now use your `PrismaClient` instance to perform operations against the `Profile` new table. Here are some examples. 

#### Create a new profile for an existing user

```ts
const profile = await prisma.profiles.create({
  data: {
    bio: "Hello World",
    user: {
      connect: { email: "alice@prisma.io" }
    }
  },
})
```

#### Create a new user with a new profile

```ts
const user = await prisma.users.create({
  data: {
    email: 'john@prisma.io',
    name: 'John',
    profile: {
      create: { 
        bio: "Hello World"
      }
    }
  },
})
```

#### Update the profile of an existing user

```ts
const userWithUpdatedProfile = await prisma.users.update({
  where: { email: "alice@prisma.io" },
  data: {
    profile: {
      update: {
        bio: "Hello Friends"
      }
    }
  }
})
```

## Next steps

- Read the holistic, step-by-step [Prisma Framework tutorial](https://github.com/prisma/prisma2/blob/master/docs/tutorial.md)
- Check out the [Prisma Framework docs](https://github.com/prisma/prisma2) (e.g. for [data modeling](https://github.com/prisma/prisma2/blob/master/docs/data-modeling.md), [relations](https://github.com/prisma/prisma2/blob/master/docs/relations.md) or the [Photon.js API](https://github.com/prisma/prisma2/blob/master/docs/photon/api.md))
- Share your feedback in the [`prisma2-preview`](https://prisma.slack.com/messages/CKQTGR6T0/) channel on the Prisma Slack
- Create issues and ask questions on [GitHub](https://github.com/prisma/prisma2/)
- Track the Prisma Framework's progress on [`isprisma2ready.com`](https://isprisma2ready.com)