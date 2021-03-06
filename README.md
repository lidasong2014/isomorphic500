# isomorphic500

[Isomorphic500](https://isomorphic500.herokuapp.com) is a small isomorphic web application featuring photos from [500px](http://500px.com).

It is built on [express](http://expressjs.com) using [React](https://facebook.github.io/react) and [Flux](https://facebook.github.io/flux) with [yahoo/fluxible](http://fluxible.io). It is developed with [webpack](http://webpack.github.io) and [react-hot-loader](http://gaearon.github.io/react-hot-loader/) and written with [babeljs](http://babeljs.io) with the help of [eslint](http://eslint.org).

<a href="https://isomorphic500.herokuapp.com"><img src="https://cloud.githubusercontent.com/assets/120693/6992728/d93c61c8-dadb-11e4-82b3-f08f8bee24c3.png" width="700"></a>

[![Join the chat at https://gitter.im/gpbl/isomorphic500](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/gpbl/isomorphic500?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

The intent of this project is to solidify my experience with these technologies and (maybe) to inspire other developers in their journey with React and Flux. It works also as example of a javascript development environment with all the cool recent stuff :-)

- see the demo on [isomorphic500.herokuapp.com](https://isomorphic500.herokuapp.com) (with source maps!)
- clone this repo and run the server to confirm it is actually working
- edit a react component or a css style, and see the updated app as you save your changes!
- read on for some technical details
- [write issues](https://github.com/gpbl/isomorphic500/issues) and [join the gitter chat](https://gitter.im/gpbl/isomorphic500?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) to discuss :-)

> PS. The previous, non-flux version of this repo is in the [isomorphic-react-template] (https://github.com/gpbl/isomorphic500/tree/isomorphic-react-template) branch.

**Clone this repo**

```
git clone https://github.com/gpbl/isomorphic500.git
cd isomorphic500
npm install
```

**Start the app**

```bash
npm run dev
```

and open [localhost:3000](http://localhost:3000).

You can also try the built app:

```bash
npm run build   # First, build for production
npm run prod    # then, run the production version
```

then open [localhost:8080](http://localhost:8080).

## Application structure

```bash
$ tree src

├── Application.js       # The root Application component
├── actions              # Actions creators
├── app.js               # The Fluxible app
├── assets               # Dir with static files
├── client.js            # Entry point for the client
├── components           # React components
├── config.js            # Load the config on dev or prd
├── constants            # Constants values (e.g. action types)
├── pages                # Contains route handlers components
│   ...
│   └── RouteActions.js  # Actions executed when rendering a route
├── public               # Only in prod: contains static assets loaded with webpack
├── routes.js            # Routes config
├── server               # Server-side-only code
│   ├── ga.js            # Contains Google Analytics code to inject into HtmlDocument
│   ├── HtmlDocument.js  # Components containing <html>...</html> page
│   └── render.js        # Middleware to render HtmlDocument server-side
├── server.js            # Run the express server, setup fetchr service
├── services             # Fetchr services (e.g. load data from 500px API)
├── stores               # Flux stores
├── style                # Contains the Sass styles
└── utils                # Some useful utils
```

### The fluxible app

The [src/app.js](src/app.js) file is the core of the Fluxible application:

- it configures Fluxible with [Application.js](src/Application.js) as the root component.
- it registers the stores so they can work on the same React context
- it adds the [routr plugin](https://github.com/yahoo/fluxible-plugin-routr) (the routing interface) and the [fetchr plugin]((https://github.com/yahoo/fluxible-plugin-fetchr)) (to share the same API requests both client and server-side)
- it makes possible to dehydrate the stores [on the server](src/server/render.js) and rehydrate them [on the client](src/client.js)
- it provides a `componentActionHandler` to make the app react to errors sent by flux actions

### Async data

I used [Fetchr](https://github.com/yahoo/fetchr) and the relative [fluxible-plugin-fetchr](https://github.com/yahoo/fluxible-plugin-fetchr).
[Fetchr services](src/services) run only on server and send [superagent](http://visionmedia.github.com/superagent) requests to 500px.


### Router

Using [fluxible-plugin-routr](https://github.com/yahoo/fluxible-plugin-routr), I could keep the router in a "flux flow": the current route is stored in the [RouteStore](src/stores/RouteStore.js), and the [Application component](src/Application.js) listens to it to know which [page component](src/pages) should render.

Before setting the route, this plugin can execute an action to prefill the stores with the required data. (see the `action` attributes in the routes’s [config](src/routes.js)).

> Note that these actions can send an error to the `done()` callback, so that we can render an error page, as explained below in the "RouteStore" section.

### Stores

Components do not listen to stores directly: they are wrapped in an high-order component using the fluxible `connectToStores` add-on. See for example the [PhotoPage](src/pages/PhotoPage.js).

(Thanks [@gaearon](https://github.com/gaearon) for exploring this technique [in his article](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)).

#### Resource stores

While REST APIs usually return collections as arrays, a resource store keeps the items organized in an object – like the [PhotoStore](src/stores/PhotoStore.js). This simplifies the progressive updates that may happen during the app’s life.

#### The RouteStore

The [RouteStore](src/stores/RouteStore.js) keeps track of the current route.

##### Loading state

When a route is loading (e.g. waiting for the API response), the store set the `isLoading` property to the route object. The Application component will then render a Loader until the route is finished to load.

##### Route errors

A route error happens when a route is not found or when the service fetching critical data has returned an error.

In these cases, the RouteStore set its `currentPageName` to `404` or `500`, so that the Application component can render a [`NotFoundPage`](src/pages/NotFoundPage.js) or an [`ErrorPage`](src/pages/ErrorPage.js).

> Note that a not-found route may come from the router itself (i.e. the route is missing in the [config](src/routes.js)) but also when a route action sends to the callback an error with `{status: 404}`.

## Development

### Webpack

Webpack is used as commonjs module bundler, css builder (using sass-loader) and assets loader (images and svg files).

The [development config](./webpack/dev.config.js) enables source maps, the [Hot Module Replacement](http://webpack.github.io/docs/hot-module-replacement.html) and [react-hot-loader](http://gaearon.github.io/react-hot-loader/). It loads CSS styles with `<style>`, to enable styles live reload). This config is used by the [webpack-dev-server](webpack/server.js), serving the files bundled by Webpack.

The [production config](./webpack/prod.config.js) is used to build the production version with `npm run build`: similar to the dev config, it minifies the JS files, removes the `debug` statements and produces an external `.css` file. Files are served from a express static directory (i.e. `/public/assets`).

Both configs set a `process.env.BROWSER` global variable, useful to require CSS from the components, e.g:

```js
// MyComponent
if (process.env.BROWSER) {
  require('../style/MyComponent.scss');
}
```

Files loaded by webpack are hashed. Javascript and CSS file names are [saved](webpack/plugins/write-stats.js) in a JSON file and passed to the [HtmlDocument](src/server/HtmlDocument.js) component from the [server/render](src/server/render.js) middleware.

### Babeljs

This app is written in Javascript-[Babel](https://babeljs.io/). Babel config is in [.babelrc](.babelrc) (it only enables class properties). On Sublime Text, I installed [babel-sublime](https://github.com/babel/babel-sublime) to have full support of the Babel syntax!

### Linting

I use [eslint](http://eslint.org) with [babel-eslint](https://github.com/babel/babel-eslint) and the [react plugin](https://github.com/yannickcr/eslint-plugin-react) – config in [.eslintrc](.eslintrc). I also configured Sublime Text with [SublimeLinter-eslint](https://github.com/roadhump/SublimeLinter-eslint).

Code style with [jscs](http://jscs.info) using [a config](.jscsrc) inspired by Airbnb's one. On Sublime Text, I installed [SublimeLinter-jscs](https://packagecontrol.io/packages/SublimeLinter-jscs).

You can use this command to run both linters from the command line:

```bash
npm run linter
```

I use [SublimeLinter-scss-lint](https://github.com/attenzione/SublimeLinter-scss-lint) for linting the Sass files ([.scss-lint.yml](.scss-lint.yml)).

### Testing

I'm still a beginner with Flux unit testing – so tests are missing :-) I use [mocha](http://mochajs.org), using [chai](http://chaijs.com) as assertion library.

To run the tests, use this command:

```
npm test
```

There's also the test coverage with [isparta](https://github.com/douglasduteil/isparta) (based on [istanbul](https://github.com/gotwarlost/istanbul)):

```bash
npm run coverage
```

### Debugging

The app uses [debug](https://www.npmjs.com/package/debug) to log debug messages. You can enable/disable the logging from Node by setting the `DEBUG` environment variable before running the server:

```bash
# enable logging for isomorphic500 and Fluxible
DEBUG=isomorphic500,Fluxible node index

# disable logging
DEBUG= node index
```

From the **browser**, you can enable/disable them by sending this command in the JavaScript console:

```js
debug.enable('isomorphic500')
debug.disable()
```
