---
section: getting-started
layout: getting-started
title: Keyword lists and maps
redirect_from: /getting-started/maps-and-dicts.html
---

Now let's talk about associative data structures. Associative data structures are able to associate a key to a certain value. Different languages call these different names like dictionaries, hashes, associative arrays, etc.

In Elixir, we have two main associative data structures: keyword lists and maps. It's time to learn more about them!

## Keyword lists

Keyword lists are a data-structure used to pass options to functions. Imagine you want to split a string of numbers. We can use `String.split/2`:

```elixir
iex> String.split("1 2 3", " ")
["1", "2", "3"]
```

However, what happens if there is an additional space between the numbers:

```elixir
iex> String.split("1  2  3", " ")
["1", "", "2", "", "3"]
```

As you can see, there are now empty strings in our results. Luckily, the `String.split/3` function allows the `trim` option to be set to true:

```elixir
iex> String.split("1  2  3", " ", [trim: true])
["1", "2", "3"]
```

`[trim: true]` is a keyword list. Furthermore, when a keyword list is the last argument of a function, we can skip the brackets and write:

```elixir
iex> String.split("1  2  3", " ", trim: true)
["1", "2", "3"]
```

As the name implies, keyword lists are simply lists. In particular, keyword lists are 2-item tuples where the first element (the key) is an atom and the second element can be any value. Both representations are the same:

```elixir
iex> [{:trim, true}] == [trim: true]
true
```

Since keyword lists are lists, we can use all operations available to lists. For example, we can use `++` to add new values to a keyword list:

```elixir
iex> list = [a: 1, b: 2]
[a: 1, b: 2]
iex> list ++ [c: 3]
[a: 1, b: 2, c: 3]
iex> [a: 0] ++ list
[a: 0, a: 1, b: 2]
```

You can read the value of a keyword list using the brackets syntax:

```elixir
iex> list[:a]
1
iex> list[:b]
2
```

In case of duplicate keys, values added to the front are the ones fetched:

```elixir
iex> new_list = [a: 0] ++ list
[a: 0, a: 1, b: 2]
iex> new_list[:a]
0
```

Keyword lists are important because they have three special characteristics:

  * Keys must be atoms.
  * Keys are ordered, as specified by the developer.
  * Keys can be given more than once.

For example, [the Ecto library](https://github.com/elixir-lang/ecto) makes use of these features to provide an elegant DSL for writing database queries:

```elixir
query =
  from w in Weather,
    where: w.prcp > 0,
    where: w.temp < 20,
    select: w
```

Although we can pattern match on keyword lists, it is rarely done in practice since pattern matching on lists requires the number of items and their order to match:

```elixir
iex> [a: a] = [a: 1]
[a: 1]
iex> a
1
iex> [a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
iex> [b: b, a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
```

In order to manipulate keyword lists, Elixir provides [the `Keyword` module](https://hexdocs.pm/elixir/Keyword.html). Remember, though, keyword lists are simply lists, and as such they provide the same linear performance characteristics as them: the longer the list, the longer it will take to find a key, to count the number of items, and so on. For this reason, keyword lists are used in Elixir mainly for passing optional values. If you need to store many items or guarantee one-key associates with at maximum one-value, you should use maps instead.

## Maps

Whenever you need a key-value store, maps are the "go to" data structure in Elixir. A map is created using the `%{}` syntax:

```elixir
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> map[:a]
1
iex> map[2]
:b
iex> map[:c]
nil
```

Compared to keyword lists, we can already see two differences:

  * Maps allow any value as a key.
  * Maps' keys do not follow any ordering.

In contrast to keyword lists, maps are very useful with pattern matching. When a map is used in a pattern, it will always match on a subset of the given value:

```elixir
iex> %{} = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
** (MatchError) no match of right hand side value: %{2 => :b, :a => 1}
```

As shown above, a map matches as long as the keys in the pattern exist in the given map. Therefore, an empty map matches all maps.

Variables can be used when accessing, matching and adding map keys:

```elixir
iex> n = 1
1
iex> map = %{n => :one}
%{1 => :one}
iex> map[n]
:one
iex> %{^n => :one} = %{1 => :one, 2 => :two, 3 => :three}
%{1 => :one, 2 => :two, 3 => :three}
```

[The `Map` module](https://hexdocs.pm/elixir/Map.html) provides a very similar API to the `Keyword` module with convenience functions to manipulate maps:

```elixir
iex> Map.get(%{:a => 1, 2 => :b}, :a)
1
iex> Map.put(%{:a => 1, 2 => :b}, :c, 3)
%{2 => :b, :a => 1, :c => 3}
iex> Map.to_list(%{:a => 1, 2 => :b})
[{2, :b}, {:a, 1}]
```

Maps have the following syntax for updating a key's value:

```elixir
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}

iex> %{map | 2 => "two"}
%{2 => "two", :a => 1}
iex> %{map | :c => 3}
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
```

The syntax above requires the given key to exist. It cannot be used to add new keys. For example, using it with the `:c` key failed because there is no `:c` in the map.

When all the keys in a map are atoms, you can use the keyword syntax for convenience:

```elixir
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
```

Another interesting property of maps is that they provide their own syntax for accessing atom keys:

```elixir
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}

iex> map.a
1
iex> map.c
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
```

Elixir developers typically prefer to use the `map.field` syntax and pattern matching instead of the functions in the `Map` module when working with maps because they lead to an assertive style of programming. [This blog post by José Valim](https://dashbit.co/blog/writing-assertive-code-with-elixir) provides insight and examples on how you get more concise and faster software by writing assertive code in Elixir.

## `do`-blocks and keywords

As we have seen, keywords are mostly used in the language to pass optional values. In fact, we have used keywords before in this guide. For example, we have seen:

```elixir
iex> if true do
...>   "This will be seen"
...> else
...>   "This won't"
...> end
"This will be seen"
```

It happens that `do` blocks are nothing more than a syntax convenienice on top of keywords. We can rewrite the above to:

```elixir
iex> if true, do: "This will be seen", else: "This won't"
"This will be seen"
```

Pay close attention to both syntaxes. In the keyword list format, we separate each key-value pair with commas, and each key is followed by `:`. In the `do`-blocks, we get rid of the commas and separate each keyword by a newline. They are useful exactly because they remove the verbosity when writing blocks of code. Most of the time, you will use the block syntax, but it is good to know they are equivalent.

Note that only a handful of keyword lists can be converted to blocks: `do`, `else`, `catch`, `rescue`, and `after`. Those are all the keywords used by Elixir control-flow constructs. We have already learned some of them and we will learn others in the future.

With this out of the way, let's see how we can work with nested data structures.

## Nested data structures

Often we will have maps inside maps, or even keywords lists inside maps, and so forth. Elixir provides conveniences for manipulating nested data structures via the `put_in/2`, `update_in/2` and other macros giving the same conveniences you would find in imperative languages while keeping the immutable properties of the language.

Imagine you have the following structure:

```elixir
iex> users = [
  john: %{name: "John", age: 27, languages: ["Erlang", "Ruby", "Elixir"]},
  mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}
]
[
  john: %{age: 27, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
  mary: %{age: 29, languages: ["Elixir", "F#", "Clojure"], name: "Mary"}
]
```

We have a keyword list of users where each value is a map containing the name, age and a list of programming languages each user likes. If we wanted to access the age for john, we could write:

```elixir
iex> users[:john].age
27
```

It happens we can also use this same syntax for updating the value:

```elixir
iex> users = put_in users[:john].age, 31
[
  john: %{age: 31, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
  mary: %{age: 29, languages: ["Elixir", "F#", "Clojure"], name: "Mary"}
]
```

The `update_in/2` macro is similar but allows us to pass a function that controls how the value changes. For example, let's remove "Clojure" from Mary's list of languages:

```elixir
iex> users = update_in users[:mary].languages, fn languages -> List.delete(languages, "Clojure") end
[
  john: %{age: 31, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
  mary: %{age: 29, languages: ["Elixir", "F#"], name: "Mary"}
]
```

There is more to learn about `put_in/2` and `update_in/2`, including the `get_and_update_in/2` that allows us to extract a value and update the data structure at once. There are also `put_in/3`, `update_in/3` and `get_and_update_in/3` which allow dynamic access into the data structure. [Check their respective documentation in the `Kernel` module for more information](https://hexdocs.pm/elixir/Kernel.html).

This concludes our introduction to associative data structures in Elixir. You will find out that, given keyword lists and maps, you will always have the right tool to tackle problems that require associative data structures in Elixir.