# poolr.js

  Small but usefull resource pooling/limiting/throttling library for node.js.

  Poolr runs asyncronous tasks in parallel until a configured limit is
  reached and then switches to serial execution.
  This helps maximizing the use of limited resources like filedescriptors
  or remote (database-) connections.

  poolr tries to be small and intuitive to use. For a much more advanced
  implementation of resource pools take a look at
  [node-pool](https://github.com/coopernurse/node-pool)

  You can add any function you like to a pool:

  `staticFunc(foo, bar, callback)` becomes:
  `thisPool.addTask(staticFunc, foo, bar, callback);`

  For object methods you can either pass the object as a default context as a
  second argument to the constructor of poolr, or use function binding:

    var someObj = new someClass();
    var thisPool = new poolr(10, someObj);
    thisPool.addTask(someObj.someMethod, arg1, argN, callback);

  or:

    var someObj = new someClass();
    var thisPool = new poolr(10);
    thisPool.addTask(someObj.someMethod.bind(someObj), arg1, argN, callback);



## Synopsis

A basic example limiting cradle connections to two at a time:

    var cradle = require('cradle'),
        poolr  = require('poolr');

    cradle.setup({host: 'localhost', port: 5984});
    var conn      = new (cradle.Connection)(),
        db        = conn.database('test'),
        couchPool = new poolr(2, db);

    // all cradle functions are called via the pool:
    couchPool.addTask(db.exists, function(err, result) {
        if (err) throw err;
            console.log(result + ' exists');
        }
    });
    couchPool.addTask(db.save, {foo:'bar'}, function(err, res) {
        if (err) throw err;
        console.log(res);
    });
    // ...and so on.
    // Add more as fast as you like. Only two functions will be run at a time


## poolr

`new poolr()` ceates a new reource pool. The constructor takes one integer
argument representing the maximum number of tasks allowed for using this
resource in parallel.
If you want to support a default context for binding functions to, you can
supply it as a second argument.

  `new poolr(5);` creates a resource pool for 5 parallel exections
  `new poolr(5, someObj);` creates a resource pool for 5 parallel exections
  with `someObj` as `this` context for tasks.


## addTask()

add a task to the resource pool. Takes the function to be called as the
first argument. Later arguments are passed to the function.
The last argument is expected to be a callback to pass back the results.
If the function is an object method and needs its object context, you
should either supply the object as a second argument to the poolr constructor
or bind the method to its object:

    var obj = new someClass();
    myPool = new poolr(10, obj);

    addTask(staticFunction, arg1, argn, callback);

    addTask(otherObj.someFunction.bind(otherObj), arg1, argn, callback);

    myPool.addTask(obj.someFunction, arg1, argn, callback);

addTask() returns `true` if the task is run immediately and `false` it
it will be run within the next free slot.

## License

(The MIT License)

Copyright (c) 2011 Bernhard Weisshuhn <bkw@codingforce.com>

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
