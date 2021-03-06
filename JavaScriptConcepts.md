# JavaScript

## DOM
* DOM manipulation is expensive. If your application is big or very dynamic, the time spent in manipulating DOM elements rapidly adds up and you hit a performance bottleneck.
* Angular solved it by **dirty model checking** by updating the DOM only when model changes. It still has problems. The static text eg., p tag is updated every time the model is changed when they have same ancestor which is unneccessary. So, the solution is to add another layer between DOM and the HTML template called **Virtual DOM** proposed by React. 
* Virtual DOM is an in-memory representation of the actual elements that are being created for your page.
```html
<div>
  <div id="header">
    <h1>Hello, {{state.subject}}!</h1>
    <p>How are you today?</p>
  </div>
</div>
```
* The above HTML is represented in DOM as below
```javascript
{
  tag: 'div',
  children: [
    {
      tag: 'div',
      attributes: {
        id: 'header'
      },
      children: [
        {
          tag: 'h1',
          children: 'Hello, World!'
        },
        {
          tag: 'p',
          children: 'How are you today?'
        }
      ]
    }
  ]
}
```
* If the `subject` changes here from `Hello, World!` to `Hello, mom!` then only `h1` tag is surgically updated without updating the whole DOM.
* The repainting of the UI is the most expensive part, and React efficiently ensures that the real DOM receives only batched updates to repaint the UI.
* Frequent DOM manipulations are expensive and performance heavy.
* Virtual DOM is a virtual representation of the real DOM.
* When state changes occur, the virtual DOM is updated and the previous and current version of virtual DOM is compared. This is called “diffing”.
* The virtual DOM then sends a batch update to the real DOM to update the UI.
* React uses virtual DOM to enhance its performance.
* It uses the observable to detect state and prop changes.
* React uses an efficient diff algorithm to compare the versions of virtual DOM.
* It then makes sure that batched updates are sent to the real DOM for repainting or re-rendering of the UI.
* Good way to append multiple DOM elements as children is by using `createDocumentFragment` method as shown below.
```javascript
let a = document.createElement('a');
let container = document.querySelector('.container');
let documentFragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    let cloneA = a.cloneNode(true);
    cloneA.text = `Row N° ${i}`;
    documentFragment.appendChild(cloneA);
}
container.appendChild(documentFragment);
```

## Web

### Single Page Application(SPA)
* Earlier, when client requests the server for a page, it is sent from server. When user clicks any link on that page, a new page from server is sent back to the client. So, we see a whole page refresh. With a SPA, we only make AJAX requests and send JSON back to the client without page refresh. SPA uses history API for routing. 

## Memory
* Using a SPA implies staying on the same page for a much longer time. If the page that is never fully reloaded starts progressively using more and more memory, that can seriously affect the performance and even cause the browser's tab to crash.
* Causes of memory leaks
  * accidental global variables
    * use strict mode to prevent
  * closures - Closures are an unavoidable and an integral part of JavaScript, so it is important to:
      * understand when the closure has been created and what objects are retained by it,
      * understand closure's expected lifespan and usage (especially if used as a callback).
  * Having a setTimeout or a setInterval referencing some object in the callback is the most common way of preventing the object from being garbage collected.
    * being aware of the objects referenced from the timer's callback,
    * using the handle returned from the timer to cancel it when necessary.
  * event listeners
    * unregister the event listener once no longer needed, by creating a reference pointing to it and passing it to removeEventListener(). 
    * addEventListener() can take a third parameter, which is an object providing additional options. Given that {once: true} is passed as a third parameter to addEventListener(), the listener function will be automatically removed after handling the event once. 
  * Cache - It can grow indefinitely if we don't clear the cache once the associated object is deleted. In the example below, the user object is still in cache after it has been deleted from memory.
```javascript
let user_1 = { name: "Peter", id: 12345 };
const mapCache = new Map();

function cache(obj){
    if (!mapCache.has(obj)){
        const value = `${obj.name} has an id of ${obj.id}`;
        mapCache.set(obj, value);

        return [value, 'computed'];
    }

    return [mapCache.get(obj), 'cached'];
}

cache(user_1); // ['Peter has an id of 12345', 'computed']
user_1 = null; // removing the inactive user

//Garbage Collector
console.log(mapCache); // ((…) => "Peter has an id of 12345")) // user1 entry is still in cache
```
To work around this issue, we can use WeakMap. It is a data structure with weakly held key references which accepts only objects as keys. If we use an object as the key, and it is the only reference to that object – the associated entry will be removed from cache and garbage collected. In the following example, after nulling the user_1 object, the associated entry gets automatically deleted from the WeakMap after the next garbage collection. 
```javascript
let user_1 = { name: "Peter", id: 12345 };
const weakMapCache = new WeakMap();

function cache(obj){
    // ...same as above, but with weakMapCache
    return [weakMapCache.get(obj), 'cached'];
}
cache(user_1); // ['Peter has an id of 12345', 'computed']
console.log(weakMapCache); // ((…) => "Peter has an id of 12345"}
user_1 = null; // removing the inactive user

// Garbage Collector
console.log(weakMapCache); // ((…) => {}) - first entry gets garbage collected
```
* Detached DOM elements - If a DOM node has direct references from JavaScript, it will prevent it from being garbage collected, even after the node is removed from the DOM tree. 

```javascript
  function createElement() {
    const div = document.createElement('div');
    div.id = 'detached';
    return div;
  }

  // this will keep referencing the DOM element even after deleteElement() is called
  const detachedDiv = createElement();

  document.body.appendChild(detachedDiv);

  function deleteElement() {
      document.body.removeChild(document.getElementById('detached'));
  }

  deleteElement(); // Heap snapshot will show detached div#detached
```
* One of the possible solutions is to move DOM references into the local scope.

```javascript
  function createElement() {...} // same as above

  // DOM references are inside the function scope

  function appendElement() {
      const detachedDiv = createElement();
      document.body.appendChild(detachedDiv);
  }

  appendElement();

  function deleteElement() {
       document.body.removeChild(document.getElementById('detached'));
  }

  deleteElement(); // no detached div#detached elements in the Heap Snapshot
```

## Variables
* `window` and `document`, for example, are global variables supplied by the browser. In a Node environment, you can access `process` object as a global variable. 

## Loops
* `String, Array, TypedArray, Map`, and `Set` are all built-in iterables, because each of their prototype objects implements an `@@iterator` method.
* The `for...of` statement creates a loop iterating over iterable objects, including: built-in String, Array, array-like objects (e.g., arguments or NodeList), TypedArray, Map, Set, and user-defined iterables.
* The `for...in` statement iterates over the enumerable properties of an object, in an arbitrary order.

## Arrays
* Three dots `...` accepts not only arrays, but any iterable objects (strings, maps, sets, etc).
* To transform arrayLike parameter into an array and return it, use
```javascript
  Array.from(arrayLike, [mapFunction])
  ```
* Surely `.indexOf(element) !== -1` must be replaced with `arr.includes(element, [fromIndex])`. 
* In javascript arrays are not true arrays like in java and in other languages. They're actually objects so typeof will return "object" for them. To know if something's really an array its constructor can be compared to Array. [refer here for other types](https://webbjocke.com/javascript-check-data-types/)
```javascript
// Returns if a value is an array
function isArray (value) {
return value && typeof value === 'object' && value.constructor === Array;
}

// ES5 actually has a method for this (ie9+)
Array.isArray(value);
```

## Strings
* ECMAScript 2015 provides a correct method to verify a substring existence: 
```javascript
string.includes(substring, [fromIndex])
```
The method returns a boolean that indicates if substring is a substring of string, optionally starting from an index `fromIndex`. 

## Numbers
* Use `Number.isNaN` function to check for a number.

## Functions
* Regular functions can be invoked in 4 ways.
  * simple invocation
  * method invocation - object method
  * indirect invocation - call, apply
  * constructor invocation
* pass optional parameters for `null` or `undefined` values. Without considerable amount of refactoring, but for a considerable code readability arguments object should be migrated to rest parameters. 
* The arrow function allows efficiently to bind `this` lexically. It uses the enclosing context and does not define its own `this`.
* Arrow functions don't have `this`, `arguments`. So, it cannot be used for object methods, constructor functions.

### Recursion
* https://github.com/hackreactor/recursion_in_javascript
* [HackReactor Recursion](https://docs.google.com/presentation/d/e/2PACX-1vR-oZyN4pYr3gbnPcrkUBtDrfnR16d6TnZGxhAI4Q52Fu_hm0_KAmEzf_HjEJNhCaPPln40XAriXTbu/pub?start=false&loop=false&delayms=60000#slide=id.g5f8fe97093_0_0)

# OOPS

## Objects
* The **`Object.assign()`** method copies all [enumerable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable) [own properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) from one or more *source objects* to a *target object*. It returns the target object.
```javascript
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

const returnedTarget = Object.assign(target, source);

console.log(target);
// expected output: Object { a: 1, b: 4, c: 5 }
```
* The **`Object.seal()`** method seals an object, preventing new properties from being added to it and marking all existing properties as non-configurable. Values of present properties can still be changed as long as they are writable.
* The **`Object.freeze()`** method **freezes** an object. A frozen object can no longer be changed; freezing an object prevents new properties from being added to it, existing properties from being removed, prevents changing the enumerability, configurability, or writability of existing properties, and prevents the values of existing properties from being changed. In addition, freezing an object also prevents its prototype from being changed. `freeze()` returns the same object that was passed in.
* Object does not maintain insertion order like a [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
* Object converts other types of `keys` to string implicitly, but `Map` maintains the original type.
* Object only supports one object as a key. If you want to use multiple keys as objects, use `Map`.
* Returns if a value is an object
```javascript
function isObject (value) {
return value && typeof value === 'object' && value.constructor === Object;
}
```
## Class
* You can use getters and setters to get and set values in a class like below: 
```javascript
class User {

  constructor(name) {
    // invokes the setter
    this.name = name;
  }

  get name() {
    return this._name;
  }

  set name(value) {
    if (value.length < 4) {
      alert("Name is too short.");
      return;
    }
    this._name = value;
  }

}

let user = new User("John");
alert(user.name); // John

user = new User(""); // Name is too short.
```
