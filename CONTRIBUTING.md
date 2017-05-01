# How to contribute

Adding something to this library should be quick and easy. The DOM/CSS 
for a ton of these is already worked out. What we
need here are active defui versions of those that can be used easily 
without copying/pasting DOM.

## Getting Started

1. Make sure you run the gulp command in the README.css to generate 
the CSS.
2. Pick a component that has CSS/layout, but is not yet here. 
some of the stuff has gotten out of sync. So, only implement things that are currently
rendering correctly. 
3. Ask "is anyone working on/can I work on X?" and create an Issue on GitHub
4. Write the defui/model/mutations, probably all together in a single new namespace.
Components that interact full-stack (like image library) need more thought/discussion.
5. Make devcards for the rendering of that component in various states (for
visual regression testing) in the `visuals` source folder.
6. Make a devcard that wraps the component in an untangled-app (see examples)
to show the callback/interactions working. Include devcard documentation that
describes the use of the component. These go in the `guide` source folder.

Please contact us on the `#untangled` channel of [clojurians.slack.com](http://clojurians.slack.com) if you 
want to help.

## Making Changes

### Write in CLJC

To support server-side rendering all code should be written in CLJC files. 

- Rendering must be pure! Don't use `js/setTimout` or other js-only things in your UI. That stuff belongs in mutations.
If your lifecycle (e.g. componentWillMount) or something needs js things, make a clj/cljs function where the clj version 
is a no-op, and call *that* from the UI.
- Untangled mutations (via `defmutation`) can be put in cljs-only blocks. They'll never be attempted on server-side code anyhow.

### Create a Card for Visual Regression Tests

All components must have one or more devcards in the `visuals` source directory that show their various states 
statically (for visual regression testing). Use the same package structure, and name that namespace with a -cards suffix.
E.g. src/untangled.ui.calendar.cljs -> visuals/untangled.ui.calendar-cards.cljs

### Create a Card for Live Documentation/Usage

All components must have a devcard in the `guide` source directory that documents/demonstrates
your live working component with callbacks, etc. Use MockNetwork or real server-side code for simulating full stack 
interactions (examples coming soon).

### Naming Conventions

- Things that generate DOM should be prefixed with `ui-`, e.g. `ui-button`. This will help with naming conflicts and
helps the user understand what generates UI and what does not.
- Mutations should be written with `defmutation` so that their symbols end up in the namespace of the components.
- Mutations should be written via helper functions of `(f state-map args)` that can be composed into other people's
mutations, if necessary. The name of these helpers should be suffixed with `impl` (e.g. `close-all-impl`)
- Idents for components should use Om table names prefixed to their namespace (e.g. `:untangled.ui.menu/by-id`)
    - Make this DRY. Define a table-name symbol as ::id. One stateful component per namespace.

SEE COMMENTS IN `menu.cljc`
