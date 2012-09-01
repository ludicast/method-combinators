method-combinators
==================

tl;dr
---

This library gives you some handy function combinators you can use to make [Method Decorators] in CoffeeScript or [JavaScript][js]:

[Method Decorators]: https://github.com/raganwald/homoiconic/blob/master/2012/08/method-decorators-and-combinators-in-coffeescript.md#method-combinators-in-coffeescript "Method Decorators in CoffeeScript"

```coffeescript

before (...) -> something
  
# => (methodBody) ->
#      (argv...) ->
#        ((...) -> something).apply(this, argv)
#        methodBody.apply(this, argv)
    
after (...) -> something

# => (methodBody) ->
#      (argv...) ->
#        __ret__ = methodBody.apply(this, argv)
#        ((...) -> something).apply(this, argv)
#        __ret__
    
around (...) -> something

# => (methodBody) ->
#      (argv...) ->
#        (...) -> something).call(
#          this,
#          (__ret__ = => methodBody.apply(this, argv)),
#          argv...
#        )
#       __ret__

provided (...) -> something

# => (methodBody) ->
#      (argv...) ->
#        if ((...) -> something).apply(this, argv)
#          methodBody.apply(this, argv)

```

The library is called "Method Combinators" because these functions are isomorphic to the combinators from Combinatorial Logic.

Back up the truck, Chuck. What's a Method Decorator?
---

A method decorator is a function that takes a function as its argument and returns a new function that is to be used as a method body. For example, this is a method decorator:

```coffeescript
mustBeLoggedIn = (methodBody) ->
                   ->
                     if currentUser?.isValid()
                       methodBody.apply(this, arguments)
```

```javascript
mustBeLoggedIn = function (methodBody) {
  return function () {
    if (currentUser?.isValid()) {
      return methodBody.apply(this, arguments)
    }
  }
}
```

You use it like this:

```coffeescript
class SomeControllerLikeThing

  showUserPreferences:
    mustBeLoggedIn ->
      #
      # ... show user preferences
      #
```

```javascript
function SomeControllerLikeThing() {}

SomeControllerLikeThing.prototype.showUserPreferences =
  mustBeLoggedIn(
    function() {
      //
      // ... show user preferences
      //
    }
  );
```

And now, whenever `showUserPreferences` is called, nothing happens unless `currentUser?.isValid()` is truthy. And you can reuse `mustBeLoggedIn` wherever you like. Since method decorators are based on function combinators, they compose very nicely, you can write:

```coffeescript
triggersMenuRedraw = (methodBody) ->
                       ->
                         __rval__ = methodBody.apply(this, arguments)
                        @trigger('menu:redraww')
                        __rval__

class AnotherControllerLikeThing

  updateUserPreferences:
    mustBeLoggedIn \
    triggersMenuRedraw \
    ->
      #
      # ... save updated user preferences
      #
```

```javascript
triggersMenuRedraw = function(methodBody) {
  return function () {
    var __rval__ = methodBody.apply(this, arguments);
    this.trigger('menu:redraww');
    return __rval__;
  }
};

function AnotherControllerLikeThing() {};

AnotherControllerLikeThing.prototype.updateUserPreferences =
  mustBeLoggedIn(
    triggersMenuRedraw(
      function() {
        //
        // ... save updated user preferences
        //
      }));
```

Fine. Method Decorators look cool. So what's a Method Combinator?
---

Method combinators are convenient function combinators for making method decorators. When writing decorators, the same few patterns tend to crop up regularly:

1. You want to do something *before* the method's base logic is executed.
2. You want to do something *after* the method's base logic is executed.
3. You want to do wrap some logic *around* the method's base logic.
4. You only want to execute the method's base logic *provided* some condition is truthy.

Method *combinators* make these four kinds of method decorators extremely easy to write. Instead of:

```coffeescript
mustBeLoggedIn = (methodBody) ->
                   ->
                     if currentUser?.isValid()
                       methodBody.apply(this, arguments)

triggersMenuRedraw = (methodBody) ->
                       ->
                         __rval__ = methodBody.apply(this, arguments)
                        @trigger('menu:redraww')
                        __rval__
```

We write:

```coffeescript
mustBeLoggedIn = provided -> currentUser?.isValid()

triggersMenuRedraw = after -> @trigger('menu:redraww')
```

```javascript
mustBeLoggedIn =
  provided(
    function() {
      return typeof currentUser !== "undefined" && currentUser !== null ? currentUser.isValid() : void 0;
    }
  );

triggersMenuRedraw = 
  after(
    function() {
      return this.trigger('menu:redraww');
    }
  );
```

And they work exactly as we expect:

```coffeescript
class AnotherControllerLikeThing

  updateUserPreferences:
    mustBeLoggedIn \
    triggersMenuRedraw \
    ->
      #
      # ... save updated user preferences
      #
```

```javascript
function AnotherControllerLikeThing() {};

AnotherControllerLikeThing.prototype.updateUserPreferences =
  mustBeLoggedIn(
    triggersMenuRedraw(
      function() {
        //
        // ... save updated user preferences
        //
      }));
```

The combinators do the rest!

So these are like RubyOnRails controller filters?
---

There are some differences. These are much simpler, which is in keeping with JavaScript's elegant style. For example, in Rails all of the filters can abort the filter chain by returning something falsy. The `before` and `after` decorators don't act as filters. Use `provided` is that's what you want.

Is it any good?
---

[Yes][y].

[y]: http://news.ycombinator.com/item?id=3067434

[js]: https://github.com/raganwald/method-combinators/blob/master/lib/method-combinators.js

Can I install it with npm?
---

Yes: `npm install method-combinators`

How to get started
---

Eat a hearty breakfast. Breakfast is the most important meal of the day! [:-)](https://github.com/facebook/javelin/)

Et cetera
---

Method Combinators was created by [Reg "raganwald" Braithwaite][raganwald]. It is available under the terms of the [MIT License][lic].

[raganwald]: http://braythwayt.com
[lic]: https://github.com/raganwald/YouAreDaChef/blob/master/license.md