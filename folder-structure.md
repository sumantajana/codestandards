Folder Structure
================

Please note
------------

While this gist has been shared and followed for years, I regret not giving more background. It was originally a gist for the engineering org I was in, not a "general suggestion" for any React app.

Typically I avoid folders altogether. Heck, I even avoid new files. If I can build an app with one 2000 line file I will. New files and folders are a pain.

But when you're in a decently large organization where different development teams work within pretty well-defined feature boundaries of an application, I like the following approach but I would keep the feature folders flat, no route nesting.

Also, don't you dare throw this at your eng team and be like "this is the way". Make up your own minds and don't use me as some weird appeal to authority, I'm just an average dev like anybody else.

Motivations
-----------

- Clear feature ownership
- Module usage predictibility (refactoring, maintainence, you know
  what's shared, what's not, prevents accidental regressions,
  avoids huge directories of not-actually-reusable modules, etc)
- CI runs only the tests that matter (future)
- Code splitting (future)

How it works
------------

The file structure maps directly to the route hierarchy, which maps
directly to the UI hierarchy.

It's inverted from the model that we've used in other systems. If we
consider all folders being either a "generic" or a "feature" folder, we
only have one "feature" folder but many "generic" folders.

Examples of "feature" folders:

- Surveys
- Admin
- Users
- Author

Examples of "generic" folders:

- components
- helpers
- stores
- actions

Given this route config:

```js
var routes = (
  <Route name="App">
    <Route name="Admin">
      <Route name="Users"/>
      <Route name="Reports"/>
    </Route>
    <Route name="Course">
      <Route name="Assignments"/>
    </Route>
  </Route>
);
```

We would now set up our directories like this:

```
app
└── screens
    └── App
        └── screens
            ├── Admin
            │   └── screens
            │       ├── Reports
            │       └── Users
            └── Course
                └── screens
                    └── Assignments
```

Next, each of these screens has an `index.js` file, which is the file
that handles the entry into the screen, also known as a "Route Handler"
in react router. Its very much like a `Route` in Ember. We'll also have
some top-level application bootstrapping stuff at the root, like
`config/routes.js`.

```
app
├── config
│   └── routes.js
├── screens
│   └── App
│       ├── screens
│       │   ├── Admin
│       │   │   ├── screens
│       │   │   │   ├── Reports
│       │   │   │   │   └── index.js
│       │   │   │   └── Users
│       │   │   │       └── index.js
│       │   │   └── index.js
│       │   └── Course
│       │       ├── screens
│       │       │   └── Assignments
│       │       │       └── index.js
│       │       └── index.js
│       └── index.js
└── index.js
```

With this structure, each screen has its own directory to hold its
modules. In other words, we've introduced "scope" into our application
file structure.

Each will probably have a `components` directory.

```
app
├── config
│   └── routes.js
├── screens
│   └── App
│       ├── components
│       ├── screens
│       │   ├── Admin
│       │   │   ├── components
│       │   │   ├── screens
│       │   │   │   ├── Reports
│       │   │   │   │   ├── components
│       │   │   │   │   └── index.js
│       │   │   │   └── Users
│       │   │   │       ├── components
│       │   │   │       └── index.js
│       │   │   └── index.js
│       │   └── Course
│       │       ├── components
│       │       ├── screens
│       │       │   └── Assignments
│       │       │       ├── components
│       │       │       └── index.js
│       │       └── index.js
│       └── index.js
└── index.js
```

These components are used *only in the current screen*, not even the
child screens. So what about when you've got some shared components
between screens?

### Shared Modules

Every screen also has a "shared" generic directory. If its children
share any components with each other or the parent, we put the shared
code in "shared". Here is our growing app with some new shared, and not
shared modules.

```
app
├── config
│   └── routes.js
├── screens
│   └── App
│       ├── components
│       ├── screens
│       │   ├── Admin
│       │   │   ├── components
│       │   │   ├── screens
│       │   │   │   ├── Reports
│       │   │   │   │   ├── components
│       │   │   │   │   ├── stores
│       │   │   │   │   │   └── ReportsStore.js
│       │   │   │   │   └── index.js
│       │   │   │   └── Users
│       │   │   │       ├── components
│       │   │   │       └── index.js
│       │   │   ├── shared
│       │   │   │   └── stores
│       │   │   │       ├── AccountStore.js
│       │   │   │       └── UserStore.js
│       │   │   └── index.js
│       │   └── Course
│       │       ├── components
│       │       ├── screens
│       │       │   └── Assignments
│       │       │       ├── components
│       │       │       └── index.js
│       │       └── index.js
│       ├── shared
│       │   └── components
│       │       ├── Avatar.js
│       │       └── Icon.js
│       └── index.js
├── shared
│   └── util
│       └── createStore.js
└── index.js
```

Note `Admin/shared`; `Reports` and `Users` can both access the shared
stores. Additionally, every screen in the app can use `Avatar.js` and `Icon.js`.

We put shared components in the nearest `shared` directory possible and
move it toward the root as needed.

### Shared module resolution

The way modules in CommonJS are resolved is pretty straightforward in
practice: its all relative from the current file.

There is one piece of "magic" in the way modules resolve. When you do a
non-relative require like `require('moment')` the resolver will first
try to find it in `node_modules/moment`. If its not there, it will look
in `../node_modules/moment`, and on up the tree until it finds it.

We've made it so that `shared` resolves the same way with webpack
`modulesDirectories`. This way you don't have to
`require('../../../../../../../../../../shared/Avatar')` you can simply
do `require('components/Avatar')` no matter where you are.

### Tests

Tests live next to the modules they test. Tests for
`shared/util/createStore.js` live in `shared/util/__tests__/createStore.test.js`.

Now our app has a bunch of `__tests__` directories:

```
app
├── __tests__
├── config
│   └── routes.js
├── screens
│   └── App
│       ├── components
│       │   ├── __tests__
│       │   │   └── AppView.test.js
│       │   └── AppView.js

... etc.

├── shared
│   └── util
│       ├── __tests__
│       │   └── createStore.test.js
│       └── createStore.js
└── index.js
```

### Why "Screens"?

The other option is "views", which has become a lot like "controller".
What does it even mean? Screen seems pretty intuitive to me to mean "a
specific screen in the app" and not something that is shared. It has the
added benefit that there's no such thing as an "MSC" yet, so the word
"screen" causes people to ask "what's a screen?" instead of assuming
they know what a "view" is supposed to be.
