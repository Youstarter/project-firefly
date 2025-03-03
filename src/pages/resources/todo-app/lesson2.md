---
keywords:
  - Adobe I/O
  - Extensibility
  - API Documentation
  - Developer Tooling
---

# Lesson 2: Setup Runtime actions

In this lesson, we'll setup the Runtime actions to handle the CRUD operations.
The app we're building can handle multiple todo lists and each todo list can have several todo items.

## Global configuration file 

To avoid long todo lists, we'll define a `MAX_TODO_ITEMS` value within a global configuration file that we will import from the `actions` folder but also from the `web-src` folder.
We'll create the file at the root of the App Builder App and name it `defaults.json`.
  
```json
{
  "MAX_TODO_ITEMS": 10
}
```  

Then we're going to import the value in our action `todolist/index.js`:

```javascript
const { MAX_TODO_ITEMS } = require('../../defaults.json');
```

In the next lesson, we'll also show how to import the value from the `web-src` folder.

## Install aio-lib-state

We'll be using [aio-lib-state](https://github.com/adobe/aio-lib-state) to store the todo items. So first we're going to install the dependency with:

````bash
npm i --save @adobe/aio-lib-state
````  

Then we're going to import it as well:

``javascript
const stateLib = require('@adobe/aio-lib-state');
``

## Main function

We'll setup the [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations inside the main function. 

### Parameters 

First, we'll define an `operation` parameter and make it required: 

```javascript
const requiredParams = ['operation']; 
```  

The `operation` parameter can take 4 possible values: 

* `create` to create an empty todo list
* `read` to read a todo list
* `update` to update a todo list with a todo item 
* `delete` to delete a todo list

We'll also use additional optional parameters: 

* `name` to identify a list
* `todo` to identify a todo inside a list  

```javascript
const { operation, name, todo } = params;
```

### CRUD operations

Next, we'll initialize the state library and retrieve a `todolist` value with `state.get()`.

```javascript
const state = await stateLib.init();

let todoList = await state.get(`todolist`);
if (todoList?.value) {
  todoList = todoList.value;
}
else {
  todoList = [];
}
```  

The `todolist` will hold all todo lists objects and is an empty array by default.

Finally, we'll define which operation to perform based on the value of `operation` and return the response.

```javascript
let body = {};
switch (operation) {
  case 'create':
    // Find the todo list by name
    if (!todoList.find(({ name: todoListName }) => todoListName === name)) {
      // If none found, create an empty list with the given name
      todoList.unshift({
        name,
        todos: []
      });
      
      // Store the new list in the state storage with no expiry time
      await state.put(`todolist`, todoList, { ttl: -1 });

      body.message = `"${name}" added.`;
    } else {
      return errorResponse(400, `"${name}" already exists.`, logger);
    }
    break;

  case 'read':
    // Simply return the todo lists
    body.todoList = todoList;
    break;

  case 'update':
    if (todo) {
      // Find the todo list by name
      const foundTodoList = todoList.find(({ name: todoListName }) => todoListName === name);
      if (foundTodoList) {
        // Find the todo item by id
        const todoIndex = foundTodoList.todos.findIndex(({ id }) => id === todo.id);
        if (todoIndex !== -1) {
          // Update the todo item
          foundTodoList.todos[todoIndex] = todo;
          body.message = `Todo "${todo.id}" updated in "${name}".`;
          
          await state.put(`todolist`, todoList, { ttl: -1 });
        } else {
          // Create a new todo item
          if (foundTodoList.todos.length < MAX_TODO_ITEMS) {
            foundTodoList.todos.unshift(todo);
            body.message = `Todo "${todo.id}" added to "${name}".`;
            
            await state.put(`todolist`, todoList, { ttl: -1 });
          } else {
            return errorResponse(400, `Max ${MAX_TODO_ITEMS} todos reached for "${name}".`, logger);
          }
        }
      } else {
        return errorResponse(400, `${name} not found.`, logger);
      }
    } else {
      return errorResponse(400, `Todo is missing.`, logger);
    }
    break;

  case 'delete':
    // Filter out the todo list to delete by name
    const updatedTodoList = todoList.filter(({ name: todoListName }) => todoListName !== name);
    
    await state.put(`todolist`, updatedTodoList, { ttl: -1 });
    
    body.message = `"${name}" todo list deleted.`;
    break;

  default:
    return errorResponse(400, 'CRUD operation not found', logger);
}

return {
  statusCode: 200,
  body
};
``` 

For every operation except `read`, we are using the `state.put()` function to update the `todolist` value.
Additionally, we set the time to live option to `-1` so that the value of `todolist` won't expire.

See the full action code [here](https://github.com/AdobeDocs/adobeio-samples-todoapp/blob/master/actions/todolist/index.js).


