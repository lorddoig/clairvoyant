# Clairvoyant

Clairvoyant is a library that offers flexible tracing for ClojureScript.

## Usage

Add Clairvoyant to your project `:dependencies`.

```clj
[spellhouse/clairvoyant "0.1.0-SNAPSHOT"]
```

## Example

```clj
(ns foo
  (:require [clairvoyant.core :as trace :include-macros true]))


(trace/trace-forms {:tracer trace/default-tracer}
 (defn add [a b]
   (+ a b)))

(add 1 2)
```

Check your JavaScript console and look for a grouped message titled
`foo/add [a b]`. Open it and you should see

```
x = 1
y = 2
3
```

## Design

Clairvoyant is based on two concepts: tracers and source code 
transformation. Although this is not a dramatic departure from similar 
tools, Clairvoyant maintains an emphasis on extensibility which affords
the programmer finer control over the tracing proccess. This
extensibility is provided by Clojure's protocols and mulitmethods.


### Tracers

Tracers are defined around a small set of protocols which
correspond to key points in a trace's lifecycle: `ITraceEnter`,
`ITraceError`, and `ITraceExit`. Respectively, these align with when
a trace begins, when an error is encountered, and when a trace has
ended. The locations of these points may vary from form to form but in
general they're positioned to strategically.

At each point in the trace the tracer will recieve data in the form of
a map containing at the least the following information.

| Key     | Description                      | Example      |
|-----------------------------------------------------------|
| `:op`   | The form operator                | `fn`         |
| `:form` | The form being traced            | `(fn [x] x)` |
| `:ns`   | The form's originating namespace | `foo`        |

More information about trace data can be found in the 
[Trace Data](#trace-data) section.

### Source code transformation

`clairvoyant.core/trace-forms` is the sole macro used for source code
trasnformation. It walks each form given to it delegating to the public
`trace-form` multimethod that is responsible for returning the transformed
form containing the tracer hook points. To ensure a high level of control
Clairvoyant avoids macroexpansion. This is important because it allows the
programmer to trace any form they wish rather than just the special forms.

## Trace data

### `defn`, `fn`, `fn*`

 Key           | Description                              | Example
--------------------------------------------------------------------
 `:name`       | The function's name (if it provided)     | `add`
 `:arglist`    | The function's signature                 | `[a b]`
 `:args`       | The arguments passed to the function     | `[1 2]`
 `:anonymous?` | Whether or not the function is anonymous | `false`

### `defmethod`

 Key             | Description                        | Example
--------------------------------------------------------------------------
 `:name`         | The multimethod's name             | `+`
 `:arglist`      | The methods's signature            | `[a b]`
 `:dispatch-val` | The dispatch value                 | `[Number Number]`
 `:args`         | The arguments passed to the method | `[1 2]`


### `reify`

 Key         | Description                          | Example
-----------------------------------------------------------------------------
 `:protocol` | The protocol's name (fully resolved) | `clojure.core/ILookup`
 `:name`     | The functions's name                 | `-lookup`
 `:arglist`  | The function's signature             | `[a b]`


## What about Clojure?

This library was born out of frustration with `println` debugging in
ClojureScript. For the moment it will remain a ClojureScript project but.

## License

Copyright © 2014 Christopher Joel Holdbrooks

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.