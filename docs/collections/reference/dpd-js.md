<!--{
  title: 'Using dpd.js',
  tags: ['reference', 'collection', 'http', 'websockets', 'cors']
}-->

## Accessing Collections - Using dpd.js

`dpd.js` is an auto-generated library that updates as you update the resources in your Deployd API. If you are writing your front-end in the `public` directory, all you have to do is include a `<script src="/dpd.js"></script>` tag in your HTML and your browser has access to all your Deployd Collections.

### Using dpd.js on a different origin

You can use the dpd.js library outside of the `public` folder by using an absolute URL to the file. If your app is hosted at `my-app.deploydapp.com`, it would look something like this:

    <script src="http://my-app.deploydapp.com/dpd.js"></script>

This will not work on browsers that do not support Cross-Origin Resource Sharing (namely Internet Explorer 7 and below).

### Accessing the Collection

The API for your Collection is automatically generated as `dpd.[collectionname]`.

Examples:

    dpd.todos
    dpd.users
    dpd.todolists

*Note: If your Collection name has a dash in it (e.g. `/todo-lists`), the dash is removed when accessing it in this way (e.g. `dpd.todolists`).*

You can also access your collection by using `dpd(collectionName)` as a function.

Examples:

    dpd('todos')
    dpd('users')
    dpd('todo-lists')

*Note: Collections accessed in this way will not have helper functions besides `get`, `post`, `put`, `del`, and `exec` (see [Accessing Custom Resources](../../extensions/accessing-custom-resources.md) for details on these generic functions)*


### Collection API

The examples below use a Collection called `/todos` with the following schema:

- `id`
- string `title`
- string `category`

#### Callbacks

Every function in the Collection API takes a callback function (represented by `fn` in the docs) with the signature `function(result, error)`.

The callback will be executed asynchronously when the API has received a response from the server. You cannot call a remote API synchronously:

    // Does not work
    var result = dpd.todos.get();

<!--...-->

    // Works as expected
    dpd.todos.get(function(result, error) {
      // Work with result
    });

The `result` argument differs depending on the function. If the result failed, it will be `null` and the `error` argument will contain the error message.

The `error` argument, if there was an error, is an object:

 - `status` (number): The HTTP status code of the request. Common codes include:
  - 400 - Bad Request: The request contained invalid data and could not be completed
  - 401 - Unauthorized: The current session is not authorized to perform that action
  - 500 - Internal Server Error: Something went wrong on the server
 - `message` (string): A message describing the error. Not always present.
 - `errors` (object): A hash of error messages corresponding to the properties of the object that was sent - usually indicates validation errors. Not always present.

Examples:
  
    {
      "status": 401,
      "message": "You are not allowed to access that collection!"
    }

<!--...-->

    {
      "status": 400,
      "errors": {
          "title": "Title must be less than 100 characters",
          "category": "Not a valid category"
      }
    }

#### .get([id], [query], fn) <!-- api -->

##### Listing Data

The `.get(fn)` function returns an array of objects in the collection.

    // Get all todos
    dpd.todos.get(function(results, error) {
      //Do something
    });

`results` is an array of objects: 

    [
      {
        "id": "320d6151a9aad8ce",
        "title": "Wash the dog",
        "category": "pets"
      }, {
        "id": "320d6151a9aad8ce"
        "title": "Write autobiography",
        "category": "writing"
      }
    ]

If the collection has no objects, it will be an empty array:

    []    

##### Querying Data

The `.get(query, fn)` function filters results by the specified query object. See [Querying Collections](../querying-collections.md) for information on constructing a query.

    // Get all todos that are in the pets category
    dpd.todos.get({category: 'pets'}, function(results, error) {
      // Do something
    });

`results` is an array of objects: 

    [
      {
        "id": "320d6151a9aad8ce",
        "title": "Wash the dog",
        "category": "pets"
      }
    ]

##### Getting a Specific Object

The `.get(id, fn)` function returns a single object by its `id` property.

    // Get a specific todo
    dpd.todos.get("320d6151a9aad8ce", function(result, error) {
      // Do something
    });

`result` is the object that you requested:

    {
      "id": "320d6151a9aad8ce",
      "title": "Wash the dog",
      "category": "pets"
    }

#### .post([id], object, fn) <!-- api -->

##### Creating an Object

The `.post(object, fn)` function creates an object in the collection with the specified properties. 

    // Create a todo
    dpd.todos.post({title: "Walk the dog"}, function(result, error)) {
      // Do something
    });

`result` is the object that you posted, with any additional calculated properties and the `id`:

    {
      "id": "91c621a3026ca8ef",
      "title": "Walk the dog"
    }

##### Updating an Object

The `.post(id, object, fn)` function, or `.post(object, fn)` where `object` has an `id` property, will update an object. Using the `.post()` function in this way behaves the same as the `put()` function.

This is useful if you want to insert an object if it does not exist and update it if it does.

#### .put([id or query], object, fn) <!-- api -->

##### Updating an Object

The `.put(id, object, fn)` function will update an object that is already in the collection. It will only change the properties that are provided. It is also possible to incrementally update certain properties; see [Updating Objects in Collections](../updating-objects.md) for details.

    // Update a todo
    dpd.todos.put("91c621a3026ca8ef", {title: "Walk the cat"}, function(result, error)) {
      // Do something
    });

You can also use the syntax `put(object, fn)` if `object` has an `id` property:

    // Update a todo
    dpd.todos.put({id: "91c621a3026ca8ef", title: "Walk the cat"}, function(result, error)) {
      // Do something
    });

Finally, you can provide a `query` object to ensure that the object you are updating has the correct properties. You must still provide an `id` property. This can be useful as a failsafe.

    // Update a todo only if it is in the "pets" category
    dpd.todos.put(
      {id: "91c621a3026ca8ef", category: "pets"},
      {title: "Walk the cat"},
      function(result, error) {
        // Do something
      });

`result` is the entire object after the update:

    {
      "id": "91c621a3026ca8ef",
      "title": "Walk the cat",
      "category": "pets"
    }

The `.put()` function will return an error if the `id` and/or `query` does not match any object in the collection:

    {
      "status":400,
      "message":"No object exists that matches that query"
    }

#### .del(id or query, fn) <!-- api -->

##### Deleting an Object

The `.del(id, fn)` function will delete an object from the collection.

    // Delete an object
    dpd.todos.del("91c621a3026ca8ef", function(result, error) {
      // Do something
    });

You can also use the syntax `.del(query, fn)` if `object` has an `id` property. You can add additional properties to the `query` object to ensure that you are removing the correct object:

    // Delete an object
    dpd.todos.del({id: "91c621a3026ca8ef", title: "Walk the dog"}, function(result, error) {
      // Do something
    });

`result` will always be null.

### Realtime API

#### dpd.on(event, fn) <!-- api -->

The `dpd.on(event, fn)` function listens for realtime events emitted from the server. See [Scripting Collections with Events](../scripting-collection-with-events.md) for information on sending realtime events with the `emit()` function.

* `event` - The name of the event to listen for
* `fn` - Callback `function(eventData)`. Called every time the event is received. There is no `error` argument.

<!--seperate-->

    // Listen for a new todo
    dpd.on('todos:create', function(post) {
      // Do something
    });

In your Collection Event:

    // On Post
    emit('todos:create', this); 

Calling `.on()` on the collection itself will namespace the event by the collection name:
  
    // Same as dpd.on('todos:create', fn)
    dpd.todos.on('create', function(post) {
      // Do something
    });

#### dpd.off(event, [fn]) <!-- api -->

The `dpd.off(event)` function stops listening for the specified event.

    dpd.off('todos:create');

You can also provide a function that was originally set as a listener to remove only that function.

    function onTodoCreated(post) {
      // Do something
    }

    dpd.on('todos:create', onTodoCreated);

    dpd.off('todos:create', onTodoCreated);

Calling `.off()` on the collection itself will namespace the event by the collection name:

    // Same as dpd.off('todos:create');
    dpd.todos.off('create');

#### dpd.once(event, fn) <!-- api -->

The `dpd.once(event, fn)` function listens for a realtime event emitted by the server and runs the `fn` callback exactly once.

    dpd.once('todos:create', function(post) {
      // Do something
    });

Calling `.once()` on the collection itself will namespace the event by the collection name:

    // Same as dpd.once('todos:create');
    dpd.todos.once('create', function(post) {
      // Do something
    });

#### dpd.socketReady(fn) <!-- api -->

The `dpd.socketReady(fn)` function waits for a connection to be established to the server and executes the `fn` callback with no arguments. If a connection has already been established, it will execute the `fn` callback immediately. 

It can sometimes take a second or more to establish a connection, and events sent during this time will not be received by your front end. This function is useful for ensuring that you will receive an event when it is broadcast.

    dpd.socketReady(function() {
      // Do something
    });

#### dpd.socket <!-- api -->

The `dpd.socket` object is a [socket.io](http://socket.io/#how-to-use) object. This is useful if you want to finely control how events are received.