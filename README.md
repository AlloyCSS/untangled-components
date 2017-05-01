# Untangled Components

Reusable components for the [Untangled](http://untangled-web.github.io/untangled/) web framework. These components are
usable from stock Om, but their mutations assume you at least are willing to use the Untangled `mutate` multimethod for
your application and are using the standard Om database format (e.g. you don't have to use anything else from Untangled,
just the default Om database format and one `defmulti`).

![](https://clojars.org/AlloyCSS/untangled-components/latest-version.svg)

## Project Layout

```
├── dev
│   ├── clj
│   └── cljs
├── resources
│   ├── public
│   │   ├── css  ------------- The output folder for the generated stock CSS
├── script  ------------------ Location of figwheel convenience script
├── src
│   ├── css-guide  ----------- An Om-based UI describing the CSS available
│   │   ├── guideui
│   │   └── styles
│   ├── guide  --------------- A devcards-based doc for the React/Om-based elements and components. 
│   ├── main  ---------------- The Library source (exported as the library untangled-ui. Server and client)
│   ├── test  ---------------- Untangled specs for anything that merits BDD logic tests (server and client)
│   └── visuals  ------------- A set of devcards for visual regression testing.
```

## Running

There are four builds: `test`, `visuals`, `css-guide`, and `guide`. Select them via `JVM_OPTS` with `-D`

The `visuals` build is for the visual regression cards that show each possible visible state of a component, and are
(TODO) run through a browser-based image capture diff to detect visual regressions due to code and CSS changes.

The `guide` build is the development guide cards. These cards demonstrate live examples of the components and have full
markdown documentation. The hope is to evolve this to a better UI, but devcards allows for rapid development at the 
moment.

The `test` build is for untangled-specs. Those adding component that have any algorithms with complexity are encourage to
write specifications around those algorithms to help ensure correctness. Many of the components are quite simple, so
most components probably will not have specs; however, things like the forms support include a number of more interesting
behaviors that need full testing support.

The `css-guide` build is for the raw CSS with examples of the DOM/CSS rules to do raw rendering. Please note you will
need to install [npm](https://www.npmjs.org/package/npm) in order to run this task. [Learn more](README-css.md)

```
JVM_OPTS="-Dguide -Dvisuals" lein run -m clojure.main script/figwheel.clj
```

Open a browser on (port settable `:figwheel` section of `project.clj`):

- [localhost:8001](http://localhost:8001/) *(Index of the pages below, with their JVM OPT for running)*
- [localhost:8001/guide.html](http://localhost:8001/guide.html)
- [localhost:8001/css-guide.html](http://localhost:8001/css-guide.html)
- [localhost:8001/visuals.html](http://localhost:8001/visuals.html)
- [localhost:8001/test.html](http://localhost:8001/test.html)

## Form Support (Nearly Complete)

Support for declarative forms that have full-stack integration, validation
support, pluggable validation, pluggable rendering, pluggable form elements, 
nested to-one and to-many relations, and pluggable database integration for
persistence.

## Image Library and Crop

Image library (upload/selection) and clipping tools. Includes pluggable
image metadata/image storage (e.g. save name of image in postgresql, image
data in S3). The image clipping returns clip bounds for an image for 
embedding arbitrary images CMS-style. Clipping/serving the resulting
image is up to you.

## Ensure Internationalization Will Work (IN PROGRESS)

These are thoughts on how this should work...evolving.

First you should understand how Untangled does i18n:

We use GNU gettext to extract strings from the source by compiling the source to js, and using `xgettext`. So, any
calls to, say, `(tr "Hello")` turn into `tr("Hello")` in Javascript, which the predefined tools for gettext can work with
(extraction, message merging, translator tools like POEdit). If you call tr on a variable, this does not work, since the
js ends up as `tr(v)`. We have a function called `tr-unsafe` that allows you to use a variable (`tr` will intentionally
crash the compile if you use anything but a literal string for the reason), but since `tr-unsafe` will result in a js call that 
cannot be extracted statically from source you must somehow tell gettext about the possible values that variable can have, 
or your i18n will fail.

Approaches to this are:
- Manually add the missing values to your translation files by hand
- Add a function to your code base that never gets called, and just calls `tr` with the missing ones to aid in extraction.
This results in automatic extraction and serves as documentation you can put "near" the uses of `tr-unsafe`. 
- Have labels be something that are not ever stored in state. This is ok (probably preferable) for things like Button, 
but more complex things like menu's are probably declared with labels when defined, not when used.
- Have the user pass you a lambda for doing the label generation. This approach is a problem with Om, because we cannot 
store a function in app state (it breaks serialization of history). So,
we could, for example, allow them to drop in a unique keyword and define some multimethod for the component.

```clj
(defmulti render-button-label (fn [k] k))

(defmethod render-button-label :ok [k] (tr "OK"))
```

This is extensible, but I don't love it.

Comments/ideas welcome.

## Thoughts about Implementation of Elements/Components/Layouts

Some components have no real active nature (e.g. buttons): they just have things like properties and callbacks and no need 
for an Om query. For those it seems best to just do normal React kinds of things (pass parameters in a map, and children 
via varargs).

Other components like checkboxes need to manage state. For simple ones it may make sense to let the user completely manage
the state (e.g. the boolean as to the checked state of a box). 

In more complex components, like menus, there may be some significant advantage
to making them full-on Om components. For example, a `close-all-menus` mutation is impossible to write unless the menus 
are represented in the app state.

It is common for a parent component to want to look at the state of some child component. For example, what is the current
date selected in that calendar? If there is user-usable state within your component, you might choose to make simple helper
methods for asking for that data, or you can simply let them access your component's props by keyword. The former is better
for complex components (e.g. calendar) and the latter is probably fine for simple ones. In both cases these functions
should be `(f component-props)`. E.g., they should be usable from the parent that has that component's properties locally. These
are also easy enough to use from custom mutations since the ident is supposed to be standardized and well-known for all
components (e.g. getting a component's state is just a `(get-in state-map component-ident)`).

In utilities like layout helpers: These are probably plain React components (or even just functions).

We expect standards around this to resolve rapidly, but please participate in the discussion as we progress.


## API Standards

These are thoughts...interested in input at this early stage:

- See exploration_cards in the `guide` build.
- See menu_visuals and menus in the `visuals` build.
- Changes to appearance for stateful components should go through mutations
- Mutations should be written in an IDE-friendly way using the new 0.7.1-SNAPSHOT+ untangled-client `defmutation`.
- Lean towards having all of the UI components interact nicely with the VCR support viewer, so avoid using component
local state for stateful components except in performance hotspots (e.g. animations like panning an image might not be fast if they use 
`transact!`). 
- Many simple components will be simple stateless functions and not even React components. These are easy, in that anything
they do will be determined directly from arguments.
