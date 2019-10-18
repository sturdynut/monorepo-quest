# Monorepo Quest 🏔

This repo is a step-by-step tutorial to setup a monorepo using Lerna and Yarn Workspaces.  Each step builds on the previous and at the end you will have a monorepo setup with the following:

* Lerna
* Yarn Workspaces
* GraphQL Server
* React Web Application
* Shared Common Library

## Prerequisites

* [Node](https://nodejs.org/en/)
* [Yarn](https://yarnpkg.com/lang/en/)
* [Lerna](https://github.com/lerna/lerna) - `npm install -g lerna`
  *  If you don't want to install lerna on your machine, just run the lerna commands below prefixed with `npx`, like so `npx lerna init`.

## Getting Started

* Fork this repo
* `git checkout steps/0`;

## Helpful Information

* Each step below starts as if you are at the root of your project.

**Branches**

This step-by-step tutorial starts from scratch, but captures each step in a branch in case you get lost or want to skip forward.

Branches are structured like so: `step/[step]`, for example `step/1` would bring you to the final state of step 1 and would be the starting point for step 2.

**Emojis**

Emojis are used throughout this tutorial to indicate specific things, as outlined below.

🌈 - Tips and experiments that are optional and not required to move on to the next step.

📟 - A command that should be ran as part of the step.

✏️ - An edit to a file

👀 - Something you should look at for clarity or verification.

---

## Step 1: Initial Lerna Setup

Lets get Lerna setup and use some Lerna commands to scaffold out our monorepo.

1. 📟 `lerna init`
2. 📟 `lerna create common --yes`
3. 📟 `lerna create server --yes`
4. 📟 `lerna create client --yes`
5. 📟 `lerna exec -- touch index.js` to generate a placeholder index file.
6. 📟 `lerna add common --scope={client,server}`
7. 📟 `lerna bootstrap` from the root
8. Verify symlinks have been created
    * If things worked correctly the common package should now be symlinked to server and client.  Now we can share code between packages within the same repo.  Neat!
      * One easy way to see this is `tree`, install on OSX via `brew install tree`.

### 🌈 Lerna Commands & Flags

Some helpful commands:

* `lerna bootstrap` - Create symlinks between package dependencies and installs 3rd party dependencies for each package.
* `lerna create` - Adds a package to your monorepo.
* `lerna add` - Adds a dependency to all your packages.
* `lerna list` - Shows your packages
* `lerna run` - Runs a command in each of your packages; this is really nice for running tests.
  * For example: `lerna run test`
* `lerna exec` - Runs a command for each package.  Note that you need to provide `--` in front of the command.
  * For example: `lerna exec -- touch index.js`
* We'll get to publishing, diffs, etc later.

Some helpful flags:

* `--scope` - Allows you so specify what packages to run the command for
  * Single package example: `lerna add react --scope=client`
  * Multiple packages example: `lerna add ramda --scope={client,server}`

_For a full list of commands and flags: [click here](https://github.com/lerna/lerna)_

### 🌈 Better Dependency Management with Lerna

Now that we have lerna setup, let's take advantage of a feature known as hoisting.

* 📟 `lerna add ramda` from the root
* 📟 `lerna bootstrap` to install ramda
  * 👀 at each of your package's `package.json` and `node_modules`.  Notice that `ramda` is in each of these.
* 📟 `lerna bootstrap --hoist`
  * 👀 again and notice that `ramda` is still in your packages `package.json` but is no longer in each `node_modules`, rather it is in one place; at the root `node_modules`.

Hoisting is really nice, because it allows you to install shared dependencies in one `node_modules` verse having multiple copies of it throughout your packages.

---

## Step 2: Setup Yarn Workspaces

> If you did not complete the previous step or want to start fresh: 📟 `git checkout steps/1`.

**Yarn Workspaces allows us:**
* To install all over dependencies across all packages in one top level `node_modules` folder, reducing the overall size of our `node_modules` folders that otherwise would live inside each of our packages.
* Have a single `yarn.lock` which will reduces the chances for conflicts and makes reviews easier.
* Eliminate the need to hoist dependencies as we did in the previous step.

We are going to tell Lerna to use Yarn and also that we want to enable Yarn's workspaces feature.

1. ✏️ Add the following to the root `package.json`
    ```
    "private": true,
    "workspaces": ["packages/*"]
    ```
2. ✏️ Add the following to `lerna.json`
    ```
    "useWorkspaces": true,
    "npmClient": "yarn"
    ```
3. 📟 `lerna clean`
    * This removes the existing `node_modules` folders.
3. 📟 `lerna exec -- rm -f ./package-lock.json`
    * This removes the existing `package-lock.json` files.
4. 📟 `lerna bootstrap` from the root
    * 👀 at the root `node_modules` and `yarn.lock`.  More importantly, notice that there are no `node_modules` in our packages nor are there any lock files.

### 🌈 Even Better Dependency Management with Yarn Workspaces

Now that we have Yarn Workspaces setup, let's experiment with adding a new dependency.

* 📟 `lerna add ramda` from the root
* 📟 `lerna bootstrap` to install ramda
  * 👀 at your `node_modules` and each package's `package.json`.  Notice that `ramda` has been added for each project and there is only one `node_modules` folder at the root.

By adding Yarn Workspaces we've eliminated the need to hoist up dependencies manually and just let Yarn Workspaces manage that for us.  How nice!

At this point, you have a monorepo ready to go.  The next steps present a fairly opinionated way to setup a full stack application with GraphQL (server) and React (client) who both use a shared library (common).

---

## Step 3: Setup React App (client)

> If you did not complete the previous step or want to start fresh: 📟 `git checkout steps/2`.

We are going to scaffold out a React app using a tool called [create-react-app](https://github.com/facebook/create-react-app).  This was built by facebook to provide a standardized way to quickly setup a React app.  One of the main advantages is that it shields you from all the configuration that goes along with setting up a React app from scratch.

1. 📟 `rm -rf ./packages/client`
    * This kills off the existing client folder
2. 📟 `npx create-react-app --scripts-version @react-workspaces/react-scripts ./packages/client`
    * This will scaffold out a react app using [create-react-app](https://github.com/facebook/create-react-app) and a custom version of [react-scripts](https://github.com/react-workspaces/react-workspaces-playground) that adds support for Yarn Workspaces to Create React App.
3. 📟 `rm -f yarn.lock && rm -f ./packages/client/yarn.lock`
    * This kills off any existing lock files that could potentially conflict with our top level lock file
4. 📟 `rm -rf ./packages/client/node_modules`
    * This kills off the `node_modules` folder in the client package because we want to load them from the root after we run the subsequent command to bootstrap our monorepo.
5. 📟 `lerna bootstrap`
    * Installs react app dependencies in top level `node_modules`
6. 📟 `cd ./packages/client && yarn start`

You should now have a React app running at this URL:
[http://localhost:3000](http://localhost:3000)

---

## Step 4: Setup GraphQL Server (server)

> If you did not complete the previous step or want to start fresh: 📟 `git checkout steps/3`.

We will be setting up our server using [GraphQL](https://graphql.org/), which in short is a query language for APIs and a runtime for fulfilling those queries with your existing data.

Similar to how we scaffolded out a React app, there are tools out there that make it easy to setup a GraphQL server vs building one from scratch.  We will be using a tool called [GraphQL Yoga](graphql-yoga), which is one of the most popular options with over 5k stars on GitHub.

1. 📟 `lerna add graphql-yoga --scope=server`
   * You should now see `graphql-yoga` in your `server` packages `package.json` file
2  ✏️ Add the following to `packages/server/index.js`:

    ```
    import { GraphQLServer } from 'graphql-yoga';

    const typeDefs = `
      type Query {
        hello(name: String): String!
      }
    `;

    const resolvers = {
      Query: {
        hello: (_, { name }) => `Hello ${name || 'World'}`,
      },
    };

    const server = new GraphQLServer({ typeDefs, resolvers });

    server.start(() => console.log('Server is running on localhost:4000'));

    ```

3. 📟 `lerna add nodemon --scope=server --dev`
4. 📟 `lerna add esm --scope=server`
5. 📟 `lerna bootstrap`
6. 📟 `cd ./packages/server`
7. 📟 `npx nodemon -r esm ./`

You should now have a GraphQL server running at this URL:
[http://localhost:4000](http://localhost:4000)

### 🌈 Our First Query

Let's open up our GraphQL server by going to [http://localhost:4000](http://localhost:4000) in web browser.

You will notice that there is some cool looking tool there.  That is called GraphiQL, a tool that allows you to query your GraphQL server.

In the left pane, you'll want to enter the following:

```
query hello($name: String) {
  hello(name: $name)
}
```

Next, we will need to provide our query with values for `name`.  Click the "QUERY VARIABLES" link at the bottom left and enter the following:

```
{
  "name": "Human"
}
```

Click the play button and you should see something like this:

```
{
  "data": {
    "hello": "Hello Human"
  }
}
```
