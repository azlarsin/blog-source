title: JavaScript - Object 与 Map 比较
author: MDN
date: '2017-04-11 19:15:18'
tags: [javascript, es6, Object, Map]

---

摘选自 MDN.
<!-- desc -->

> from **[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_Collections#Object_and_Map_compared](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_Collections#Object_and_Map_compared)**

### Object and Map compared
Traditionally, [objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) have been used to map strings to values. Objects allow you to set keys to values, retrieve those values, delete keys, and detect whether something is stored at a key. Map objects, however, have a few more advantages that make them better maps.

* The keys of an Object are Strings, where they can be of any value for a Map.
* You can get the size of a Map easily while you have to manually keep track of size for an Object.
The iteration of maps is in insertion order of the elements.
* An Object has a prototype, so there are default keys in the map. (this can be bypassed using map = Object.create(null)).

These three tips can help you to decide whether to use a Map or an Object:
* Use maps over objects when keys are unknown until run time, and when all keys are the same type and all values are the same type.
* Use maps in case if there is a need to store primitive values as keys because object treats each key as a string whether it's a number value, boolean value or any other primitive value.
* Use objects when there is logic that operates on individual elements.


More: 

- Map vs Object: [http://stackoverflow.com/questions/18541940/map-vs-object-in-javascript](http://stackoverflow.com/questions/18541940/map-vs-object-in-javascript)
- Map vs Set: [http://stackoverflow.com/questions/24085708/javascript-map-object-vs-set-object](http://stackoverflow.com/questions/24085708/javascript-map-object-vs-set-object)
