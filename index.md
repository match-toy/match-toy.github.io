---
# You don't need to edit this file, it's empty on purpose.
# Edit theme's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
# layout: home
# title: Home
---


## What is it?
Match-toy is a pattern matching library for JavaScript. [Pattern matching](https://en.wikipedia.org/wiki/Pattern_matching) is a way to check a sequence of a given input against one or more specific patterns. Many languages like Elixir/Erlang, Rust, F#, Elm, Haskell or Scala have this as a built-in feature.

Pattern matching is a very powerful concept and Match-toy is an attempt to bring this to javascript with an elegant and familiar syntax (jQuery-like chains) plus a simple [domain specific language](#dsl).

With Match-toy, you'll be able to do:
- [Literal Pattern](#literal)
- [Bind Pattern](#bind)
- [Typed Pattern](#typed)
- [Wildcard Pattern](#wildcard)
- [Range Pattern](#range)
- [List splitting](#rest)
- [Mapping Pattern](#mapping)
- [RegExp Pattern](#regexp)
- [Logical Pattern](#logical-or)
- [As Pattern](#as)

All this in about 7.5kb gzipped.

[Try it now](https://npm.runkit.com/match-toy), then check out how to [install](#install) and [use](#usage) it.