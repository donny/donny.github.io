---
layout: post
title: My Elm adventure
---

On Sunday, 8th of May 2016, I started reading about [Elm](http://elm-lang.org/). I was interested because of its functional reactive programming nature and it's one of the [inspirations](https://twitter.com/jspahrsummers/status/309772877052391424) (though not primary) for [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) and [Redux](https://www.infoq.com/news/2015/11/redux). I started reading about its syntax, its architecture, looking at a few examples, and watching several [awesome](https://www.youtube.com/watch?v=oYk8CKH7OhE) [videos](https://www.youtube.com/watch?v=3_M2G9U51GA) about Elm. I especially like the concept of [signals](http://elm-lang.org:1234/guide/reactivity) and mailboxes. It's not a foreign concept if you're coming from ReactiveCocoa or other FRP frameworks. In Elm, signals felt natural and it fit nicely with the language (I believe it's due to the fact that Elm is functional rather than imperative). Elm signals did not feel *bolted-on* like in ReactiveCocoa.

Well, on Tuesday (10th of May 2016), signals were removed from Elm :scream: with the post titled [A Farewell to FRP](http://elm-lang.org/blog/farewell-to-frp). I was completely surprised and dumbstruck. But not for long, the more I read and internalise the post and the discussion on Elm [slack channel](http://elmlang.herokuapp.com), it became clear that subscriptions in Elm 0.17 was actually a really big deal and it simplified a lot of things. Rather than setting up the signals wiring, subscriptions let our components sit around and wait for messages, whether it's from WebSocket, the clock, etc. And it becomes the underlying library code's responsibility to handle a bunch of tricky resource management stuff behind the scenes (e.g. WebSocket connection). I think it's a win for users.

Our team used [IdeaBoardz](http://www.ideaboardz.com) to conduct our retros and I thought that its UI could be improved :wink:. Thus, I decided to spend the following week after my learning to build [Elmütt](https://github.com/donny/elmutt) that is a fresh clone of IdeaBoardz written in Elm. It uses Elm 0.17 and Bootstrap 4 for the front end; and Python 2.7, Flask, Redis, and WebSocket for the back end. And it is deployable to [Heroku](https://www.heroku.com).

I am really happy with Elm :thumbsup: :100: and it is such a breath of fresh air. Below is my notes that I took whilst learning about Elm. I think you should try Elm.

### Elm

This text file describes the [Elm](http://elm-lang.org) programming language, specifically version [0.17](http://elm-lang.org/blog/farewell-to-frp). It was designed by [Evan Czaplicki](https://twitter.com/czaplic) to provide “the best of functional programming in your browser”. Elm is about:

- No runtime errors in practice. No `null`. No `undefined is not a function`.
- [Friendly](http://elm-lang.org/blog/compiler-errors-for-humans) error messages that help you add features [more quickly](http://elm-lang.org/blog/compilers-as-assistants).
- Well-architected code that stays well-architected as your app grows.
- Automatically enforced [semantic versioning](https://twitter.com/czaplic/status/601826927838650369) for all Elm packages.

Elm is functional, no state and no side effects. Elm is a strongly typed language with static typing, similar to Haskell and other ML inspired languages.

#### Getting Started

Install the necessary software:

{% highlight shell %}
  brew install elm
  npm install -g elm-server
  npm install -g watchman
{% endhighlight %}

And create a simple project:

{% highlight shell %}
  mkdir elm
  cd elm
  elm package install
  elm package install elm-lang/html
  touch Main.elm
{% endhighlight %}

Edit the file `Main.elm` to contain the following code:

{% highlight elm %}
  module Main exposing (main)
  import Html
  main = Html.text "Hello World"
{% endhighlight %}

Compile the file: `elm make Main.elm` and open the resultant `index.html`. Alternatively, run `elm reactor` and go to `http://localhost:8000`. Unfortunately, it doesn’t support live reload. To get the live reload functionality, install `elm-server` and run `elm-server Main.elm`.

#### The Elm Architecture

The [Elm Architecture](http://guide.elm-lang.org/architecture/) is the recommended pattern for building Elm applications. It’s a unidirectional architecture that consists of three components:
- Model is a type defining the data structure of the state.
- View is a function that transforms state into rendering (e.g. HTML).
- Update is a function from previous state and an action to new state. An action is a type that encapsulate a user action or an external event.

In code, it looks like the following:

{% highlight elm %}
  module Main exposing (main)
  import Html
  import Html.App
  main : Program Never
  main =
    Html.App.beginnerProgram
      { model = "Hello, world!" , view = Html.text , update = identity }
{% endhighlight %}

Or, in the expanded form:

{% highlight elm %}
  module Main exposing (main)

  import Html exposing (Html, text, div)
  import Html.App

  type alias Model = String

  model : Model
  model = "Hello, world!"

  view : Model -> Html Msg
  view model =
    div [] [ text (toString model) ]

  type Msg = NoOp

  update : Msg -> Model -> Model
  update msg model =
    model

    main : Program Never
    main =
      Html.App.beginnerProgram
        { model = model
          , view = Html.text
          , update = update
        }
{% endhighlight %}

#### References

- [Unidirectional User Interface Architectures](http://staltz.com/unidirectional-user-interface-architectures.html)
- [Elmification of Swift](https://medium.com/design-x-code/elmification-of-swift-af14b7f92b30#.r7nkch8td)
- [Controlling Time and Space: understanding the many formulations of FRP](https://www.youtube.com/watch?v=Agu6jipKfYw)
- [Elm Update Loop](http://fredcy.github.io/updateloop.html)
- [Elm - A Front End Language with Style](https://thoughtbot.com/upcase/videos/elm-a-front-end-language-with-style)
- [Elm 0.17 Subscriptions vs old Elm Signals](https://www.reddit.com/r/elm/comments/4jsbcm/eli5_elm_017_subscriptions_vs_old_elm_signals/)
- [Hacker News in Elm](https://github.com/massung/elm-hn)
