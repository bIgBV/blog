---
layout: post
title: Working with arrays in javascript may suprise you.
---

Fixing a bug in our frontend redux app gave me new insight to the way objects
are passed around in javascript. We have been using react with redux extensively at
[Recruiterbox](https://recruiterbox.com) for the past year. All of the new code
being added is written with the react + redux combo.

As I was building a new feature in the product I came across a very weird bug.
When I dispatched an action from my component, the reducer didn't update the
state since it had already been modified, this resulted in my container not
being re-rendered leading to my component not updating with the new state.

A breif overview of the way redux works first -- Data in react can only be
passed from a parent component to it's children. Redux is a functional state
container, what this means is that you have your application state in the redux
store. Your components dispatch actions which are handled by the reducer. The
reducer is a function which takes the current state and the action as
parameters, applies the action to the state and returns a completely new,
updated state object. The state present in the store is in turn mapped to your
component via a container which is re-rendered each time the state is
updated.

Coming back to the bug, after a lot of debugging (and hating myself) with the
help of my colleague I was able to understand what was causing the state to
update without the action dispatch even completing. I used a very helpful
library called [deep-freeze](https://preview.npmjs.com/package/deep-freeze) to
figure out that the state was being modified from outside a reducer. Turns out
the array which I was passing as the props to my component was a reference to
the array stored in the state. This brings me to the first revelation about the
way parameters are passed in javascript.

## Jascript always passes by value except when you pass an object

* When you pass a parameter to a function in Javscript, it passes by value.
  This means the an entirely new copy of the paramerter is passed to the
  function.

* If you change the value of the variable passed, it doesn't change the value of
  the underlying primitive or object, it just points to a completely new
  primitive or object.

* However if you change the property of the obejct passed, the underlying
  object itself is modified.[1]

Therefore, when I was passing my array from the state as props to my component,
due to it modifying the array internally, it was actually updating the array
present in the state. This is because the property of the array was being
updated and this meant the reference was updated.

To fix this, we made sure that any props being passed to the component were
deep copied and at all times a new copy of the array was worked on. We used the
[Update](https://www.npmjs.com/package/react-addons-update) library and
[Underscore.js](http://underscorejs.org) to create a simple helper function to
deepcopy arrays of nested objects.

```javascript
export function deepCopy(data) {
    if (_.isEmpty(data)) {
        return data;
    }

    let merge = {};
    if (_.isArray(data)) merge = [];
    return Update(data, {$merge: merge});
}
```


The whole exercise taught me how important immutability is and my love for
langauages like rust increased even more.

[1]: http://stackoverflow.com/questions/6605640/javascript-by-reference-vs-by-value
