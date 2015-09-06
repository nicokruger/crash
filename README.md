Crash
=====
v.1.1.0

Crash performs optimized 2D collisions, powered by [RBush] and [SAT.js], written in javascript.  
It's most obvious use-case is in game engines, but it's flexible enough to be used anywhere.  
Crash is perfectly happy in the browser and on Node.js.


## Contents
* [Contents](#contents)
* [Installation](#installation)
  * [Node.js or Browserify](#nodejs-or-browserify)
  * [Require.js](#requirejs)
  * [Browser](#browser-global)
* [Getting Started](#getting-started)
  * [Adding Colliders](#adding-colliders)
  * [Testing for collisions](#testing-for-collisions)
  * [Unleashing the power of Crash](#unleashing-the-power-of-crash)
  * [What kind of sorcery is this !?](#what-kind-of-sorcery-is-this-)
  * [But what's up with that `moved()`?](#but-whats-up-with-that-moved)
* [Contributing](#contributing)
* [Overview](#overview)
* [API](#api)
  * [Crash]
  * [AABB updates][Crash.updateAABB()]
  * [Collision testing][Crash.test()]
  * [Colliders][Crash.Collider]
  * [Other][Listener]
* [License](#License)


## Installation
Just download one of the following files from this repo.

1. [crash.js]: full source, with comments and all (13.6kB).
2. [crash.min.js]: minified, ready to be used in production (5.6kB or 1.8kB gzipped).

Or get the code from [NPM] by typing the following command in a terminal (without the `$`):
```shell
$ npm install crash-colliders
```

You can find instruction on how to load Crash in your project below.  
When you have installed Crash, head over to the [Getting Started section][getting-started].

### Node.js or Browserify
Add the following snippet to your code:
```javascript
var Crash = require("crash-colliders");
```
Now, you can use the [API] on the `Crash` variable.

### Require.js
If you're using [require.js] in your project, use the following snippet to load Crash:
```javascript
define(["path/to/crash"], function(Crash) {
    // Your code...
});
```
Now, you can use the [API] on the `Crash` variable in your module.

### Browser Global
Add the following snippet to your HTML file:
```html
<script type="text/javascript" src="path/to/crash.js"></script>
```
Now, you can use the [API] on the global `Crash` variable (`window.Crash`).




## Getting Started
Before you can do anything useful with Crash, you have to initialize it. You can do this easily by calling:
```javascript
Crash.init();
```
This method will make sure that RBush is being initialized correctly, so Colliders can be added.  
`Crash.init()` accepts one argument, a number called `maxEntries`. This is specific to RBush, so I refer to their [documentation][rbush-docs-maxentries]. This argument is not required, though, so just leave it out for now and use the default value.  

> __Fun Fact:__ Actually, when `Crash.init()` has not been called yet, a lot of things just work, and nothing should break. You can use the full API safely, although some things may not work correctly (specifically those that need rbush). Although it should be safe (i.e. no errors should be thrown), it's a good habit to call `Crash.init()` before you do anything.


### Adding Colliders
Now that everything is ready to roll, let's add some colliders. All colliders in Crash inherit from the `Crash.Collider` class, which provides some basic methods that perform household tasks, like moving, updating its AABB and testing collisions. That would lead us too far, though, so I refer to the full [API docs][API] for more info.  
All that stuff is awesome, but a `Collider` on its own isn't very useful: it doesn't have a shape. Before you can use a Collider, you have to give it a shape, but, luckily, Crash has some built-in ones for us. Let's try them out!

```javascript
var point =   new Crash.Point  (new Crash.Vector(0,0));
var circle =  new Crash.Circle (new Crash.Vector(5,2),  10);
var box =     new Crash.Box    (new Crash.Vector(-40,0), 10, 15);
var polygon = new Crash.Polygon(new Crash.Vector(3,7),  [new Crash.Vector(0,0), new Crash.Vector(5,0), new Crash.Vector(2,3)]);
```

Wow, what's all that!? Let's clarify this step by step.

1. Each shape has its own constructor, e.g. a point is initialized with `Crash.Point` etc.
2. The first argument to each constructor is a Vector, setting the base position for the Collider. So, for a Point, this would be its position, for a Circle it would be the center and for a Box it would be the bottom-left corner.
3. Some constructors take a few extra arguments: 
 * Circle: the radius
 * Box: the width and height
 * Polygon: an array of Vectors, relative to the base position
4. All the above arguments are required.
5. These constructors also take two more (optional) arguments:
 * insert: a boolean indicating whether the collider should be inserted into RBush. More info on this is following in the next few steps.
 * data: some data to add to the collider. You can store anything you want here, but it doesn't do anything for Crash; it's just there for your convenience.

> __Fun Fact:__ the unit you use for the numbers is completely up to you. Crash only stores the numbers, you can interpret them as you wish, so you can use pixels, millimeters or even some game-specific unit you invented!

> __Another Fun Fact:__ it doesn't matter how you define `position` (e.g. top or bottom left corner for a Box etc.), as long as you use it consistently.
 
> __Important Fun Fact:__ there is a very important difference between Point and Vector: a Point is a Collider, so it can be used for collision checks. A Vector, on the other hand, is just an Object with `x` and `y` properties, used to define positions in Colliders, like the center of a Circle or the corners of a Polygon.


### Testing for collisions
Now that we have some colliders, we would probably like to know if they are colliding. To do this, we use the `Crash.test()` method:

```javascript
if(Crash.test(circle, box)) {
    alert("Oh my, we crashed!");
}
```

This is already quite nice, but not very useful: now we know that our colliders are touching, but we don't know how to undo this crash! Enter `Response`, the all-knowing crash guru.  
First, let's create one:

```javascript
var res = new Crash.Response();
```

The Response class is copied from SAT, so for further information about the kind of info it provides, I refer to the [SAT.js docs][sat-docs].

Then, let's do a subtle change to our testing code:

```javascript
if(Crash.test(circle, box, res)) {
    alert("Oh my, we crashed!");
}
```

Now, we can query the Response for some useful information and undo this embarassing crash:

```javascript
if(Crash.test(circle, box, res)) {
    alert("Oh my, we crashed!");
    var overlap = res.overlapV;
    circle.moveBy(-overlap.x, -overlap.y);
}
```

And just like that, our colliders aren't touching anymore!

> __Fun Fact:__ you could also use circle.test(box, res). This uses `Crash.test()` under the hood, but it's a little more concise.



### Unleashing the power of Crash

Wasn't that exciting!? Just wait for what's to come!  
I assume you don't want to plow through heaps of loops and complex code to do everything we did in the previous steps for *every* frame and *every* collider.  
That's where Crash's real power comes in. Let's `insert()` our colliders and get to some serious collision checking!

```javascript
Crash.insert(point);
Crash.insert(circle);
Crash.insert(box);
Crash.insert(polygon);
```

You could have achieved the same by passing `true` for the `insert` argument of the constructors, like I mentioned at the beginning.

> __Fun Fact:__ just like you can use `Collider.test()` instead of `Crash.test()`, you can use `Collider.insert()`, as a convenience method.

Now that our colliders have been inserted, we can let Crash do all the hard work: it will do all the collision checks for us! And because Crash leverages the power of RBush, only the checks that make sense will actually be performed, which saves a lot of resources.

Before we can let Crash do anything, we would like to make sure it can report to us what it's doing, so we can react to collisions. To do that, we add a listener with `Crash.onCollision()`. This listener will be called every time a collision occurs.

```javascript
var listener = function(a, b, res, cancel) {
    alert("Oh my, we crashed!");
}
Crash.onCollision(listener);
```

Here, `a` and `b` are the colliders that are colliding, `res` is the all-knowing Response, and `cancel` is a function that cancels all further collision checks for this collider. This may be useful if you move the collider (rendering all the subsequent checks useless, as we will have to run them again for the new position) and/or when you use `Crash.check()`, which we will cover in the next step.

> __Fun Fact:__ you can easily remove a listener with `Crash.offCollision(listener)`. Just make sure you saved it somewhere when you added it, so you can pass it to `offCollision()` as an argument!

And now, we're ready for some serious stuff:

```javascript
Crash.testAll(circle);
```

This runs collision checks for all inserted colliders that may be colliding with our circle.
As our circle is colliding with all the other colliders, except our box, the listener we added previously will be called once for `point` and once for `polygon`, but not for `box`. For every call, `a` equals `circle`, `b` the respective collider and `res` gives us some info about the collision.

> __Fun Fact:__ Note that, like with `Crash.test()`, we can pass in a Response, but we don't have to. If we don't, Crash will make one for us and pass that around to the listeners. How convenient!



### What kind of sorcery is this !?
We can even go one step further: let's make Crash do __everything__!

For this to work, we need to make sure that we call `Crash.moved()` on all the colliders that have moved:

> __Fun Fact:__ you probably already guessed: you can use `Collider.moved()` instead!

```
Crash.moved(point);
Crash.moved(circle);
```

This way, Crash will know it only has to call `testAll` for these colliders, the ones that have moved.

And, now, ladies and gentleman, the holy grail of collision checking:

```javascript
Crash.check();
```

And it's done. All checks have run, our colliders are where they should be. With just one function call. Such wow.


#### But what's up with that `moved()`?
All the built-in methods (like `moveTo`, `setOffset` and `rotate`) already call this for you, so you don't have to worry about this. You should only worry when you insert a collider for the first time. Normally, when designing your game (or anything else), you would make sure your colliders aren't already colliding when you load them. When this is the case, though, you can call `Crash.checkAll()` just after you loaded your colliders. This will do the same as `check()`, but for all colliders, not just the ones that moved. Neat, isn't it?



## Contributing

All contributions are very welcome!  
Typos, bug fixes, code cleanup, documentation, tests, a website, you name it!

Questions and feature requests belong in the [issue tracker], __with the right tags__.













## Overview
### Or: about the check() loop

> In this section, I'll explain the basics of Crash, how it works, how the methods fit together and what the basic workflow is.

In this section, I'll assume you're using Crash to power a game (engine), because that's probably what the majority will be using it for, and not having to cover all the edge cases makes it easier to explain.

The very heart of Crash is its [Crash.check()] loop: this is where all the magic is happening. During the update cycle of your game (engine), you can move your objects around freely (probably using some physics) and not care about collisions; just move them to where you would like them to be. You can move the Colliders either by using the move functions (`moveBy`, `rotate`,...), which is the easiest way, or by setting their `x` and `y` coordinates. If you're using the latter, you have to be cautious.  
After every move, a few things have to happen:

1. The AABB of the Collider must be updated. Use [Crash.updateAABB()] to achieve this.
2. The Collider's position in RBush must be updated. Use [Crash.update()] for this, but be aware that this calls also [Crash.updateAABB()]! You could also manually `remove()` and re-`insert()` the collider in `rbush`, just like `update()` is doing.
3. The Collider must be added to [Crash.__moved] in order for it to be collision checked during the next [Crash.check()]. Use [Crash.addToMoved()] to achieve this. You could also add it manually, but make sure you're no adding it twice (that would be a waste of resources).

1\. and 2. can be bundled in one [Crash.update()] call, and all the above can be bundled in one [Crash.moved()] call, which is the method the move functions are calling for you. This is why using the move functions is the easiest: all this is done for you.

When all your Colliders are in place, just call [Crash.check()]. This will iterate over [Crash.__moved] and do the following for every Collider (this happens in [Crash.testAll()]). Let's call this loop A.

First, all the possible collisions are retrieved using RBush, with a [Crash.search()]. Then, for every possibly colliding collider (loop B), Crash will test for a collision using SAT, and if there is, it will call [Crash.\_\_onCollision()], which in turn calls the [Listener]s, passing in the two Colliders and the Response of the collision. If, during this [Crash.\_\_onCollision()], [Crash.BREAK] is set to true, loop B is stopped. This can happen when, for example, the first collider (that will be the one passed from A to B) is moved inside a [Listener]; the Collider will then be added to [Crash.\_\_moved] and thus be checked in the next iteration of loop A, making everything that happens in B useless (it will be done again in the next iteration of A, and the possible collisions may not be accurate anymore). This doesn't happen when the second Collider passed to [Listener]s (the one 'generated' by B) is moved, because it will just be added to [Crash.\_\_moved] to be handled in the next iteration of A, but doesn't render everything happening inside B useless (it's the first Collider that is our focus).  
At the end of loop B (so, for every iteration of A), the Collider passed from A to B has its `lastPos` set to a copy of its position at that moment.  
Then, [Crash.testAll()] \(loop B\) returns, and the next iteration of A begins.

At the end of [Crash.check()] \(loop A\), all the Colliders that have been processed have their `lastCheckedPos` set to a copy of their position at that time.


I hope this has helped to clarify some of Crash's misty bits. If it hasn't, let me know and/or take a look at the source: it's not that complex and it's not that much.






















## API



### Crash
This is the main object, returned by `require()`, injected by `defined()` or set as `window.Crash`. Anything related to Crash sits in this namespace.

### Crash.RBush : *function*
The RBush constructor, as returned by the rbush module.

### Crash.SAT : *object*
The SAT object, as returned by the SAT.js module.

### Crash.Vector : *constructor*
*Alias:* [Crash.V]  
Represents a vector, used by the Colliders to define their positions and corners, and by SAT to perform its calculations.  
I refer to the [SAT docs][sat-docs] for the API definition.

### Crash.V : *constructor*
*Alias:* [Crash.Vector]  
Alias for [Crash.Vector].

### Crash.Response : *constructor*
Provides information about a collision, like overlap distance and direction.  
I refer to the [SAT docs][sat-docs] for the API definition.

### Crash.rbush : *RBush*
The RBush instance that holds the colliders. This is (mostly) used internally to optimize collision checks.  
For further documentation, please see the [RBush docs][rbush-docs].

### Crash.RESPONSE : *Response*
Used by the testing functions. When no Response has been passed to them, they use this instead.

### Crash.BREAK : *boolean*
Whether to stop the currently running check loop. This is set to `true` by [Crash.cancel()]. See [Crash.testAll()] for more info.

### Crash.MAX_CHECKS : *number*
The maximum amount of times to run [Crash.testAll()] during [Crash.check()]. See [Crash.check()] for more info.

### Crash.OVERLAP_LIMIT : *number*
The minimum amount two [Crash.Collider]s should overlap to call the [Listener]s. If falsy, `OVERLAP_LIMIT` will not be taken into account. See [Crash.testAll()] for more info.

### Crash.__listeners : *Array.\<function\>*
*Private*  
An array of functions to call when a collision occurs. You can add to this with [Crash.onCollision()].

### Crash.__notYetInserted : *Array.\<Collider\>*
*Private*  
When [Crash.init()] has not yet been called, and thus when [Crash.rbush] isn't defined yet, Colliders that are [Crash.insert()]ed are pushed to this array, to be inserted into [Crash.rbush] when [Crash.init()] is called.

### Crash.__moved : *Array.\<Collider\>*
*Private*  
An array of colliders that have moved since the last [Crash.check()]. This is used internally by [Crash.check()] to optimize collision checks. For more info, see [Crash.check()].






### Crash.init ([number maxEntries:9]) - .
__maxEntries:__ *number|optional. Default: 9*. The maximum amount of [Crash.Collider]s in a Node. See the [RBush docs][rbush-docs] for more info.  
__*return:*__ *Crash*. For chaining.

Initializes Crash, and more specifically RBush. It creates [Crash.rbush], with `maxEntries` and aabb coordinate names `.aabb.x1`, `.aabb.y1`, etc. Then, it inserts all the [Crash.Collider]s from [Crash.__notYetInserted] in [Crash.rbush].  
You should be able to use all the APIs safely without calling [Crash.init()], but some may not work correctly (specifically those that need RBush). It is a good habit, though, to call [Crash.init()] before you do anything else.

```javascript
Crash.init(16);

Crash.insert(...);
Crash.search(...);
```


### Crash.insert (Collider collider) - .
__collider:__ *Collider*. The [Crash.Collider] to insert.  
__*return:*__ *Crash*. For chaining.

Inserts [Crash.Collider]s in [Crash.rbush], or in [Crash.__notYetInserted] if the former is not defined.  
Before [Crash.Collider]s turn up in [Crash.search()]s or in collision checks (performed by [Crash.testAll()]), you must [Crash.insert()] them.  
You can use [Crash.remove()] to get them out.

```javascript
var circle = new Crash.Circle(new Crash.V(0,0), 5);
// circle won't turn up in searches and collision checks
Crash.insert(circle);
// Now it will!
```


### Crash.remove (Collider collider) - .
__collider:__ *Collider*. The [Crash.Collider] to remove.  
__*return:*__ *Crash*. For chaining.

Removes [Crash.Collider]s from [Crash.rbush], or from [Crash.__notYetInserted] if the former is not defined.

```javascript
Crash.insert(collider);
// collider will turn up in searches and collision checks
Crash.remove(collider);
// now, it won't, anymore
```


### Crash.all () - *Collider[]*
__*return:*__ *Array.\<Collider\>*. An array containing all the [Crash.Collider]s that have been [Crash.insert()]ed.

Returns all the [Crash.Collider]s that have been [Crash.insert()]ed.

```javascript
var allColliders = Crash.all();
```


### Crash.search (Collider collider) - *Collider[]*
__collider:__ *Collider*. The [Crash.Collider] to base the search on.  
__*return:*__ *Array.\<Collider\>*. An array of colliders that may be colliding with `collider`.

Runs `rbush.search()` based on a [Crash.Collider], which will look for all [Crash.Collider]s that have (nearly) colliding axis-aligned bounding boxes (AABBs). This search is optimized by an RTree (that's a special algorithm, designed for this), implemented by RBush.

RBush usually requires an array of AABB coordinates to perform a search, so [Crash.search()] translates the [Crash.collider]'s `aabb` coordinates to the correct array.

```javascript
var possibleCollisions = Crash.search(collider);
// returns an array containing closeByCollider, but not veryFarAwayCollider
```


### Crash.clear() - .
__*return:*__ *Crash*. For chaining.

Clears Crash from all [Crash.Collider]s. This calls `rbush.clear()`, and clears [Crash.__notYetInserted] and [Crash.__moved].


### Crash.addToMoved (Collider collider) - .
__collider:__ *Collider*. The [Crash.Collider] that should be added to [Crash.__moved].  
__*return:*__ *Crash*. For chaining.

Adds a [Crash.Collider] to [Crash.__moved], so it gets collision checked during the next [Crash.check()] round. This does not update the Collider's AABB! If you want to do both, use [Crash.moved()] instead.  
Note that the built-in move methods of Colliders (`moveBy`, `rotate`,...) already call [Crash.moved()] \(which calls `addToMoved()`\) for you.

```javascript
collider.pos.x += 5;
// Is not taken into account in the next check() round

Crash.moved(collider);
// Now it will
```

```javascript
collider.moveBy(5,0);
// Crash.moved(collider), and thus Crash.addToMoved(collider), has already been called for you
```


### Crash.update (Collider collider) - .
__collider:__ *Collider*. The [Crash.Collider] that should be updated.  
__*return:*__ *Crash*. For chaining.

Updates the `aabb` of the [Crash.Collider] \(using [Crash.updateAABB()]\) and updates its position in RBush.


### Crash.moved (Collider collider) - .
__collider:__ *Collider*. The [Crash.Collider] that has moved.  
__*return:*__ *Crash*. For chaining.

Notifies Crash that a [Crash.Collider] has moved. This calls [Crash.update()] and [Crash.addToMoved()], so all the housekeeping is done in one function call.
Note that this is already taken care of for you when using the built-in move methods of the Colliders (`moveBy`, `rotate`,...).

```javascript
collider.pos.x += 5;
// Is not taken into account in the next check() round
// and the aabb is not updated

Crash.moved(collider);
// Now it will
// and its aabb is updated
```

```javascript
collider.moveBy(5,0);
// Crash.move(collider) has already been called for you
```


### Crash.reset ([number maxEntries]) - .
__maxEntries:__ *number|optional*. See [Crash.init()].  
__*return:*__ *Crash*. For chaining.

Resets Crash to a default, empty state. This basically calls [Crash.clear()] and [Crash.init()], and resets some variables.  
This method is primarily used by the test suite.


### Crash.cancel () - *false*
__*return:*__ *false*. Makes it easy to stop event propagation in some EventEmitters.

This method cancels the current check loop. When you call [Crash.check()], a loop will start, which calls [Crash.testAll()] for every collider that has moved since the last [Crash.check()]. Then, [Crash.testAll()] will do a [Crash.search()] and perform all the necessary checks, calling [Crash.__onCollision()] for every collision. Now, if you move a collider during the [Crash.testAll()] loop (i.e. in a [Listener]), it will be added to [Crash.__moved] and all the following collision checks become unnecessary, because they will run again in the next iteration of the [Crash.check()] loop.  
That's where [Crash.cancel()] comes in. When you move a collider inside a [Listener], call [Crash.cancel()] (or the fourth argument of the listener, which is the exact same function) to cancel all further (unnecessary) collision checks.

```javascript
Crash.onCollision(function(a, b, res, cancel) {
    a.moveBy(-res.overlapV.x, -res.overlapV.y);
    cancel();
});
```

> __Important Note:__ all the above is only valid for the *first* collider (named `a`), because that's the one that collision checks are run for. If yo move `b`, nothing has to be done.

```javascript
Crash.onCollision(function(a, b, res, cancel) {
    b.moveBy(res.overlapV.x, res.overlapV.y);
});
```


### Crash.getTestString (string type1, string type2) - *string*
__type1:__ *string*. The type of the first [Crash.Collider].  
__type2:__ *string*. The type of the second [Crash.Collider].  
__*return:*__ *string*. The appropriate SAT testing string.

Gives you the right SAT method name to test for a collision between two [Crash.Collider]s. Be sure to pass [Collider.type], ans not a [Crash.Collider]!

```javascript
var circle = new Crash.Circle(new Crash.Vector(0,0), 10);
var box = new Crash.Box(new Crash.Vector(5,5), 7, 15);

var string = Crash.getTestString(circle.type, box.type); // 'testCirclePolygon'
```


### Crash.onCollision (function listener) - .
__listener:__ *function*. The [Listener] to add to the `collision` event. See [Listener] for more info.  
__*return:*__ *Crash*. For chaining.

Adds a [Listener] to [Crash.__listeners] and will be called every time a collision occurs. See [Listener] for more info.

```javascript
var listener = function(a, b, res, cancel) {
    alert("Oh my, there is a collision!");
    a.moveBy(-res.overlapV.x, -res.overlapV.y);
}
Crash.onCollision(listener);
```


### Crash.offCollision (function listener) - .
__listener:__ *function*. The [Listener] to remove.
__*return:*__ *Crash*. For chaining.

Removes a [Listener] from the `collision` event. So, this is the opposite of [Crash.onCollision()].

```javascript
Crash.offCollision(listener);
```


### Crash.__onCollision (Collider a, Collider b, Response res) - .
*Private*  
__a:__ *Collider*. The [Crash.Collider] that collides with `b`.  
__b:__ *Collider*. The [Crash.Collider] that collides with `a`.  
__res:__ *Response*. The [Crash.Response] for thsi collision.  
__*return:*__ *Crash*. For chaining.

Calls all the [Listener]s when a `collision` event occurs. It takes three arguments: the two [Crash.Collider]s that are colliding and the [Crash.Response] for this collision. It will take care of injecting [Crash.cancel()] by itself.  
Intended for private use.


### Crash.extend (function child, function base) - *undefined*
__child:__ *function*. This constructor that inherits from `parent`.  
__base:__ *function*. The constructor that is the parent of `child`.  
__*return:*__ *undefined*.

Extends the prototype chain of `base` to `child`, so child inherits from base. This is used to make the collider classes inherit from `Collider`.

```javascript
var Child = function(){}
var Parent = function(){}
Crash.extend(Child, Parent);
```








### Crash.updateAABB (Collider collider) - .
__collider:__ *Collider*. The [Crash.Collider] whose AABB should be updated.  
__*return:*__ *Crash*. For chaining.

Calls [Crash.updateAABBPolygon()], [Crash.updateAABBBox()], [Crash.updateAABBCircle()] or [Crash.updateAABBPoint()] based on `collider`'s `type`.


### Crash.updateAABBPolygon (Polygon collider) - .
__collider:__ *Polygon*. The [Crash.Polygon] whose AABB should be updated.  
__*return:*__ *Crash*. For chaining.

Updates a [Crash.Polygon]'s `aabb` attribute ([Collider.aabb]), based on its position and size.


### Crash.updateAABBBox (Box collider) - .
__collider:__ *Box*. The [Crash.Box] whose AABB should be updated.  
__*return:*__ *Crash*. For chaining.

Updates a [Crash.Box]'s `aabb` attribute ([Collider.aabb]), based on its position and size.


### Crash.updateAABBCircle (Circle collider) - .
__collider:__ *Circle*. The [Crash.Circle] whose AABB should be updated.  
__*return:*__ *Crash*. For chaining.

Updates a [Crash.Circle]'s `aabb` attribute ([Collider.aabb]), based on its position and size.


### Crash.updateAABBPoint (Point collider) - .
__collider:__ *Point*. The [Crash.Point] whose AABB should be updated.  
__*return:*__ *Crash*. For chaining.

Updates a [Crash.Point]'s `aabb` attribute ([Collider.aabb]), based on its position.










### Crash.test (Collider a, Collider b, [Response res]) - *boolean*
__a:__ *Collider*. The first [Crash.Collider] to test for.  
__b:__ *Collider*. The second [Crash.Collider] to test for.  
__res:__ *Response|optional*. The optional [Crash.Response] to use.  
__*return:*__ *boolean*. Indicates whether there is a collision between `a`and `b`.

Tests for a collision between `a` and `b`, using SAT. The boolean return value indicates whether `a` and `b` are colliding: `true` if there is a collision, `false` otherwise.  
You can optionally pass in a [Crash.Response] to get some information about the collision (if there is one). If you don't, [Crash.RESPONSE] will be used instead.

```javascript
var c1 = new Crash.Circle(new Crash.V(0,0), 5);
var c2 = new Crash.Point(new Crash.V(3,0));
var c3 = new Crash.Box(new Crash.V(15,20), 10, 10);
var res = new Crash.Response();

Crash.test(c1, c2, res);
// true, info in 'res'

Crash.test(c1, c3);
//false, info in Crash.RESPONSE
```


### Crash.testAll (Collider collider, [Response res]) - *boolean*
__collider:__ *Collider*. The [Crash.Collider] to test collisions for.  
__res:__ *Response|optional*. The optional [Crash.Response] to use.  
__*return:*__ *boolean*. Whether the loop was stopped.

Tests for collisions between `collider` and any [Crash.Collider] that has been [Crash.insert()]ed in [Crash.rbush]. This will [Crash.search()] for `collider`, do a collision check for every [Crash.Collider] returned by that search and finally call [Crash.__onCollision()] for every collision.  
You can stop this loop (the one that checks for collisions) simply by calling [Crash.cancel()]. This sets [Crash.BREAK] to `true` (which is what `testAll` actually looks for). In [Listener]s, the recommended way is to call their `cancel` argument, which is the exact same function as [Crash.cancel()].  
Stopping the loop comes in handy when you move `collider` in any of the [Listener]s, because all consequent collision checks become unnecessary: you will have to run `testAll()` again for the new position.  
The value returned by `testAll()` indicates whether the loop was cancelled: if it was, `false` is returned, otherwise, it returns `true`.

You probably don't want to use this method, because it isn't really intended to be used publicly (but rather by [Crash.check()] internally), and [Crash.check()] is more convenient in most situations anyway. 

Finally, I would like to note a few things:

1. `res` is optional: if you don't pass it, [Crash.RESPONSE] will be used instead.
2. this method doesn't really provide direct feedback, like [Crash.test()] does: it rather calls the attached [Listener]s. This means `res` (or [Crash.RESPONSE]) will be passed to the [Listener]s, and will only hold info about the last collision when the call is finished.
3. if this method returns `false`, the loop was cancelled, so you probably want to run it again.
4. `testAll` will not run [Crash.\_\_onCollision()] when the overlap is smaller than [Crash.OVERLAP\_LIMIT]. If [Crash.OVERLAP\_LIMIT] is falsy, [Crash.\_\_onCollision] will always be called.
5. `testAll` won't call [Crash.update()] on `collider`, so make sure it's updated.
6. `testAll` is not really intended to be used publicly, but rather by [Crash.check()] internally.



```javascript
var c1 = new Crash.Circle(new Crash.V(0,0), 5, true);
var c2 = new Crash.Point(new Crash.V(3,0), true);
var c3 = new Crash.Box(new Crash.V(15,20), 10, 10, true);
var res = new Crash.Response();
Crash.onCollision(function(a, b, res, cancel){
    alert("Oh my, there is a collision!");
});

Crash.testAll(c1, res);
// calls the listener for (c1, c2), but not for (c1, c3).
// the response passed to the listener is 'res'.

Crash.test(c2);
// calls the listener for (c2, c1), but not for (c2, c3).
// the response passed to the listener is Crash.RESPONSE.

Crash.test(c3);
// the listener will not be called.
```


### Crash.check () - .
__*return:*__ *Crash*. For chaining.

Calls [Crash.testAll()] for every [Crash.Collider] in [Crash.__moved]. This means that it performs collision checks for all the [Crash.Collider]s that have moved since the last [Crash.check()], which makes it the perfect function to handle collisions after everything was moved to new positions by physics. This integrates neatly in game loops: just do your updates, and then call [Crash.check()] to handle collisions.

Important to note is that Colliders may be moved inside [Listener]s: they will be added to [Crash.__moved] and checked in further iterations of the (current) [Crash.check()] loop.  
To prevent infinite loops (a collider is moved, checked in the following iteration, moved again, etc.), the loop will be forced to stop after [Crash.MAX_CHECKS] loops, which is `100` by default.

For a [Crash.Collider] to be checked, Crash must be notified it has moved (i.e. it must be pushed to [Crash.__moved]. You can use [Crash.moved()] or [Crash.addToMoved()] to achieve this.

```javascript
// apply updates to colliders ...
// things may be colliding!
Crash.check();
// nothing is colliding anymore!
// if the right listeners have been added, that is.
```


### Crash.checkAll () - .
__*return:*__ *Crash*. For chaining.

This does the same as [Crash.check()], with one difference: it doesn't check [Crash.Collider]s in [Crash.__moved], but rather all [Crash.Collider]s (as returned by [Crash.all()]). This is especially handy when you just loaded you [Crash.Collider]s and you don't know which ones are colliding and/or have moved.

```javascript
// I have no idea what's up with my Colliders:
// I haven't moved any yet, but I don't know if any are colliding!
Crash.checkAll();
// Ahh, that's better! Everything is sorted out!
```


























### Crash.Collider (string type, SAT.Polygon|SAT.Circle satCollider, [boolean insert:false], [any data]) : *constructor*
__type:__ *string*. The type of collider this is. Valid values: `polygon`, `box`, `point`, `circle`.  
__satCollider:__ *SAT.Polygon or SAT.Circle*. The SAT Collider to use for collision checks.  
__insert:__ *boolean|optional*. Whether to [Crash.insert()] this Collider.  
__data:__ *any|optional*. Some data to store in [Collider.data].  
__*return:*__ *Collider*. The new Collider.

This is the main Collider constructor, which provides some housekeeping methods, like `moveBy`, `update`, etc. All the other constructors ([Crash.Polygon], [Crash.Box], [Crash.Point] and [Crash.Circle]) inherit from this class. They pass their type and a custom SAT Collider to [Crash.Collider], and pass through `insert` and `data`. This way, they inherit the housekeeping functions from Collider's prototype and they can add their own on their own prototype.


### Collider.type : *string*
The type of collider this is. Valid values: `polygon`, `box`, `point`, `circle`.  
This attribute is used by [Crash.updateAABB()] to determine how to update the [Collider.aabb] attribute.


### Collider.sat : *SAT.Polygon|SAT.Circle*
The actual SAT collider used to do the collision checking. For [Crash.Polygon]s, [Crash.Box]es and [Crash.Point]s, this is a SAT.Polygon, for [Crash.Circle]s, this is a SAT.Circle.  
For further documentation, I refer to the [SAT.js docs][sat-docs].


### Collider.data : *any*
Some data that has to be carried around with the [Crash.Collider]. It can be of any type, and doesn't mean anything to Crash; it's just for your convenience.


### Collider.pos : *Vector*
This is the same [Crash.Vector] as [Collider.sat]`.pos` (so `collider.pos === collider.sat.pos` is `true`).  
This position is not garanteed to be 'safe', i.e. the [Crash.Collider] may still be colliding with something. [Collider.pos] is the [Crash.Vector] that is moved by [Collider.moveTo()] and [Collider.moveBy()], and is therefore always the most up-to-date, but not always collision checked.


### Collider.lastPos : *Vector*
This is a [Crash.Vector] keeping track of the last position as it was at the end of the [Crash.testAll()] loop. So, during a [Crash.check()] loop, the [Collider.lastPos] attribute as it is inside the [Listener]s, is a copy of [Collider.pos] at the end of the previous iteration.


### Collider.lastCheckedPos : *Vector*
This is a [Crash.Vector] keeping track of the last fully collision checked position of the [Crash.Collider], i.e. a copy of [Collider.pos] at the end of the [Crash.check()] loop.


### Collider.aabb : *object*
An Object with `x1`, `y1`, `x2` and `y2` attributes, that keeps track of the [Crash.Collider]'s axis-aligned bounding box (AABB). It is used to perform [Crash.search()]es.  
When you move a [Crash.Collider], don't forget to update this attribute, with [Crash.updateAABB()]. Note that the built-in move methods (`moveBy`, `rotate`, etc.) already do this for you.


### Collider.insert () - .
__*return:*__ *Collider*. For chaining.

Shortcut for [Crash.insert()].


### Collider.remove () - .
__*return:*__ *Collider*. For chaining.

Shortcut for [Crash.remove()].


### Collider.update () - .
__*return:*__ *Collider*. For chaining.

Shortcut for [Crash.update()].


### Collider.updateAABB () - .
__*return:*__ *Collider*. For chaining.

Shortcut for [Crash.updateAABB()].


### Collider.moved () - .
__*return:*__ *Collider*. For chaining.

Shortcut for [Crash.moved()].


### Collider.search () - *Collider[]*
__*return:*__ *Array.\<Collider\>*. An array of [Crash.Collider]s that may be colliding with `this`.

Shortcut for [Crash.search()].


### Collider.setData (any data) - .
__data:__ *any*. The data to set as the new [Collider.data].  
__*return:*__ *Collider*. For chaining.

Sets the `data` attribute ([Collider.data]) to any value that's passed as the first argument. See [Collider.data] for more info.


### Collider.getData () - *any*
__*return:*__ *any*. The value of the `data` attribute ([Collider.data]).

Returns the value of the `data` attribute ([Collider.data]). See [Collider.data] for more info.


### Collider.moveTo (number x, number y) - .
__x:__ *number*. The x-coordinate to move to.  
__y:__ *number*. The y-coordinate to move to.  
__*return:*__ *Collider*. For chaining.

This moves the [Crash.Collider]'s `pos` ([Collider.pos]) and `sat.pos` attributes __to__ (`x`, `y`). This calls [Collider.moved()] for you.


### Collider.moveBy (number x, number y) - .
*Alias:* [Collider.move()]  
__x:__ *number*. The distance in x direction to move by.  
__y:__ *number*. The distance in y direction to move by.  
__*return:*__ *Collider*. For chaining.

This moves the [Crash.Collider]'s `pos` ([Collider.pos]) and `sat.pos` attributes __by__ (`x`, `y`). This calls [Collider.moved()] for you.


### Collider.move (number x, number y) - .
*Alias:* [Collider.moveBy()]  
__x:__ *number*. The distance in x direction to move by.  
__y:__ *number*. The distance in y direction to move by.  
__*return:*__ *Collider*. For chaining.

Alias of [Collider.moveBy()].


### Crash.Polygon (Vector pos, Vector[] points, [boolean insert:false], [any data]) ~ [Crash.Collider]
*Inherits from:* [Crash.Collider]  
__pos:__ *Vector*. The base position of the Polygon.  
__points:__ *Array.\<Vector\>*. The points/corners of the Polygon.  
__insert:__ *boolean|optional*. Whether to insert the Polygon.  
__data:__ *any|optional*. Any data to set as [Collider.data].  
__*return:*__ *Polygon*.

A Polygon is a [Crash.Collider] with a base position ([Collider.pos]) and a few points/corners. The points are defined by an array of [Crash.Vector]s, relative to the base position ([Collider.pos]), in counter-clockwise order.  
For the `insert` and `data` arguments, see [Crash.Collider].


### Polygon.setPoints (Vector[] points) - .
__points:__ *Array.\<Vector\>*. The points/corners of the Polygon.  
__*return:*__ *Polygon*. For chaining.

A shortcut for [Collider.sat]`.setPoints()`, which calls [Collider.moved()] for you.
 
 
### Polygon.setAngle (number angle) - .
__angle:__ *number*. The angle by which to rotate the Polygon (in radians).  
__*return:*__ *Polygon*. For chaining.

A shortcut for [Collider.sat]`.setAngle()`, which calls [Collider.moved()] for you.
 
 
### Polygon.setOffset (number offset) - .
__offset:__ *number*. The offset by which to translate the points of the Polygon.  
__*return:*__ *Polygon*. For chaining.

A shortcut for [Collider.sat]`.setOffset()`, which calls [Collider.moved()] for you.
 
 
### Polygon.rotate (number angle) - .
__angle:__ *number*. The angle by which to rotate the points of the Polygon.  
__*return:*__ *Polygon*. For chaining.

A shortcut for [Collider.sat]`.rotate()`, which calls [Collider.moved()] for you.
 
 
### Crash.Circle (Vector center, number radius, [boolean insert:false], [any data]) ~ [Crash.Collider]
__center:__ *Vector*. The position of the center of the Circle.  
__radius:__ *number*. The radius of the Circle.  
__insert:__ *boolean|optional*. Whether to insert the Circle.  
__data:__ *any|optional*. Any data to set as [Collider.data].
__*return:*__ *Circle*

A Circle is a [Crash.Collider], with a center and radius.  
For the `insert` and `data` arguments, see [Crash.Collider].


### Crash.Point (Vector position, [boolean insert:false], [any data]) ~ [Crash.Collider]
__position:__ *Vector*. The position of the Point.  
__insert:__ *boolean|optional*. Whether to insert the Point.  
__data:__ *any|optional*. Any data to set as [Collider.data].
__*return:*__ *Point*

A Point is a [Crash.Collider] with just a position. It doesn't have a size.  
For the `insert` and `data` arguments, see [Crash.Collider].


### Crash.Box (Vector position, number width, number height, [boolean insert:false], [any data]) ~ [Crash.Collider]
__position:__ *Vector*. The position of the Box.  
__width:__ *number*. The width of the Box.  
__height:__ *number*. The height of the Box.  
__insert:__ *boolean|optional*. Whether to insert the Box.  
__data:__ *any|optional*. Any data to set as [Collider.data].
__*return:*__ *Box*

A Box is a [Crash.Collider] in a rectangular shape with a position (the bottom-left corner), a width and a height.  
For the `insert` and `data` arguments, see [Crash.Collider].












### Listener (Collider a, Collider b, Response res, function cancel) : *function*
__a:__ *Collider*. The [Crash.Collider] that collides with `b`.  
__b:__ *Collider*. The [Crash.Collider] that collides with `a`.  
__res:__ *Response*. The [Crash.Response] for this collision.  
__cancel:__ *function*. Cancels the current check loop. See [Crash.testAll()] for more info.

A listener is a function that is called every time a collision is detected by [Crash.testAll()], [Crash.check()] or [Crash.checkAll()] and will be passed four arguments: the two [Crash.Collider]s that are colliding, the [Crash.Response] for this collision and [Crash.cancel()], which cancels the current check loop. See [Crash.testAll()] for more info about this.  
The context of the listener (`this`) will be set to `Crash`.  
You can add a listener with [Crash.onCollision()] and you can remove them with [Crash.offCollision()]. All listeners are stored in [Crash.__listeners], which is intended for private use.





## License

The MIT License (MIT)

Copyright (c) 2014-2015 Tuur Dutoit

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.













[crash.js]: https://raw.githubusercontent.com/TuurDutoit/crash/master/crash.js
[crash.min.js]: https://raw.githubusercontent.com/TuurDutoit/crash/master/crash.min.js
[RBush]: https://github.com/mourner/rbush
[rbush-docs]: https://github.com/mourner/rbush/blob/master/README.md
[rbush-docs-maxentries]: https://github.com/mourner/rbush/blob/master/README.md#creating-a-tree
[SAT.js]: https://github.com/jriecken/sat-js
[sat-docs]: https://github.com/jriecken/sat-js/blob/master/README.md
[require.js]: http://requirejs.org
[source code]: https://github.com/TuurDutoit/crash/blob/master/crash.js
[issue tracker]: https://github.com/TuurDutoit/crash/issues
[getting-started]: #getting-started
[API]: #api
[NPM]: https://www.npmjs.com/package/crash-colliders


[Crash]: #crash-1
[Crash.RBush]: #crashrbush--function
[Crash.SAT]: #crashsat--object
[Crash.Vector]: #crashvector--constructor
[Crash.V]: #crashv--constructor
[Crash.Response]: #crashresponse--constructor
[Crash.rbush]: #crashrbush--rbush
[Crash.RESPONSE]: #crashresponse--response
[Crash.BREAK]: #crashbreak--boolean
[Crash.MAX_CHECKS]: #crashmax_checks--number
[Crash.OVERLAP_LIMIT]: #crashoverlap_limit--number
[Crash.__listeners]: #crash__listeners--array
[Crash.__notYetInserted]: #crash__notyetinserted--array
[Crash.__moved]: #crash__moved--array
[Crash.init()]: #crashinit-number-maxentries9---
[Crash.insert()]: #crashinsert-collider-collider---
[Crash.remove()]: #crashremove-collider-collider---
[Crash.all()]: #crashall----collider
[Crash.search()]: #crashsearch-collider-collider---collider
[Crash.clear()]: #crashclear---
[Crash.addToMoved()]: #crashaddtomoved-collider-collider---
[Crash.update()]: #crashupdate-collider-collider---
[Crash.moved()]: #crashmoved-collider-collider---
[Crash.reset()]: #crashreset-number-maxentries---
[Crash.cancel()]: #crashcancel----false
[Crash.getTestString()]: #crashgetteststring-string-type1-string-type2---string
[Crash.onCollision()]: #crashoncollision-function-listener---
[Crash.offCollision()]: #crashoffcollision-function-listener---
[Crash.__onCollision()]: #crash__oncollision-collider-a-collider-b-response-res---
[Crash.extend()]: #crashextend-function-child-function-base---undefined
[Crash.updateAABB()]: #crashupdateaabb-collider-collider---
[Crash.updateAABBPolygon()]: #crashupdateaabbpolygon-polygon-collider---
[Crash.updateAABBBox()]: #crashupdateaabbbox-box-collider---
[Crash.updateAABBCircle()]: #crashupdateaabbcircle-circle-collider---
[Crash.updateAABBPoint()]: #crashupdateaabbpoint-point-collider---
[Crash.test()]: #crashtest-collider-a-collider-b-response-res---boolean
[Crash.testAll()]: #crashtestall-collider-collider-response-res---boolean
[Crash.check()]: #crashcheck----
[Crash.checkAll()]: #crashcheckall----
[Crash.Collider]: #crashcollider-string-type-satpolygonsatcircle-satcollider-boolean-insertfalse-any-data--constructor
[Collider.type]: #collidertype--string
[Collider.sat]: #collidersat--satpolygonsatcircle
[Collider.data]: #colliderdata--any
[Collider.pos]: #colliderpos--vector
[Collider.lastPos]: #colliderlastpos--vector
[Collider.lastCheckedPos]: #colliderlastcheckedpos--vector
[Collider.aabb]: #collideraabb--object
[Collider.insert()]: #colliderinsert----
[Collider.remove()]: #colliderremove----
[Collider.update()]: #colliderupdate----
[Collider.updateAABB()]: #colliderupdateaabb----
[Collider.moved()]: #collidermoved----
[Collider.search()]: #collidersearch----collider
[Collider.setData()]: #collidersetdata-any-data---
[Collider.getData()]: #collidergetdata----any
[Collider.moveTo()]: #collidermoveto-number-x-number-y---
[Collider.moveBy()]: #collidermoveby-number-x-number-y---
[Collider.move()]: #collidermove-number-x-number-y---
[Crash.Polygon]: #crashpolygon-vector-pos-vector-points-boolean-insertfalse-any-data--crashcollider
[Polygon.setPoints()]: #polygonsetpoints-vector-points---
[Polygon.setAngle()]: #polygonsetangle-number-angle---
[Polygon.setOffset()]: #polygonsetoffset-number-offset---
[Polygon.rotate()]: #polygonrotate-number-angle---
[Crash.Circle]: #crashcircle-vector-center-number-radius-boolean-insertfalse-any-data--crashcollider
[Crash.Point]: #crashpoint-vector-position-boolean-insertfalse-any-data--crashcollider
[Crash.Box]: #crashbox-vector-position-number-width-number-height-boolean-insertfalse-any-data--crashcollider
[Listener]: #listener-collider-a-collider-b-response-res-function-cancel--function
