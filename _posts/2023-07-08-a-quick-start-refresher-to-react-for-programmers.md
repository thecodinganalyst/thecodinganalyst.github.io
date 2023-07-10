---
title: "A Quick Start Refresher to React for Programmers"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - React
  - Frontend
---

Like [Angular](https://www.thecodinganalyst.com/knowledgebase/Getting-started-with-angular/), React is another popular library to build frontend single-page-applications. We can create a react app by using [Create React App](https://create-react-app.dev/), run `npx create-react-app --template typescript <app-name>` and the scaffolding of the application will be generated for us in typescript.

## React Component returns HTML rendering in JSX 

Each function is supposed to return a rendering of a html component, in the form of JSX. JSX is essentially a javascript extension that includes a html syntax, so that we can return html in our react functions. It is almost completely the same as html, except that 

1. because there are some html keywords that clashes with some javascript keywords (e.g. class), there are some slight changes to the html DOM, which can be found at [https://react.dev/reference/react-dom/components](https://react.dev/reference/react-dom/components). 

2. there should be a single root element, you can use `<></>` as the root. 

3. all the tags must be closed.

4. html syntax are inside () brackets, while js/ts syntax are inside {} brackets.

A simple way is to use a jsx convertor - [https://transform.tools/html-to-jsx](https://transform.tools/html-to-jsx) to easily transform your html to jsx.

```
const TaskList = () => {
  const title: string = "Tasks";
  return (
    <>
      <b>{title}</b>
      <ul>
        <li>Task 1</li>
        <li>Task 2</li>
        <li>Task 3</li>
      </ul>
    </>
  );
}
export default TaskList;
```
[CodeSandbox](https://codesandbox.io/s/react-basic-spwtdj)

## React is a state machine, rerenders upon state changes

To have states in react, we need to use the [`useState` hook](https://react.dev/reference/react/useState) to define the state. 

1. The `const [tasks, setTasks] = useState(initialTask)` sets the state variable to be named `tasks`, and we can use the function `setTasks` to update the state. The initial value of `tasks` is set to `initialTask`.

> `useState` is a hook, and hooks are only allowed to be called at the top level of your component.

2. In the below example, only changes to the state - `tasks`, i.e. using of `setTasks`, will trigger a rerender of the component. Meaning, if we simply reassign the task with e.g. tasks = updatedTask, or tasks.push(newTask), etc, will not trigger the rerendering. 

> Instead of assigning a new value to setTasks, the setter function also accepts a function as the parameter - `setTasks(currentTasks => return newTasks)`, so that it can return a new state based on the current state. 

> If `count = 5`, calling `setCount(count + 1)` 3 times is calling `setCount(5 + 1)` 3 times, which results in `count = 6`. Use `setCount(count => count + 1)` instead if you want to increment it 3 times. 

3. States are immutable, so to reassign the states with the existing values, we can use something like {...task, done: checked} to create a new copy of task, and set the done property to checked. Or for the array, we can use the array functions like [`map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) or [`filter`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) that returns a new copy of the array. 

4. Because the UI is assigned with values from the state, we need to provide functions to update the state, so that the UI will be updated upon human inputs. In the below example, without the `onChange` method, to call the `checkBoxHandler` to update the value of the `task`, clicking on the checkbox will not check or uncheck it. 

```
<input
  type="checkbox"
  checked={task.done}
  onChange={(e) => checkBoxHandler(task.id, e.target.checked)}
/>
```

5. Every iterating/looping component/tag needs to have a `key` with a unique value, to uniquely identify the looping component, e.g. `<li key={task.id}>`.

```
import { useState } from "react";

interface Task {
  id: number;
  title: string;
  done: boolean;
}

const initialTasks: Task[] = [
  { id: 0, title: "Buy Milk", done: false },
  { id: 1, title: "Water Plants", done: true },
  { id: 2, title: "Send Letter", done: false }
];

const TaskList = () => {
  const title: string = "Tasks";
  const [tasks, setTasks] = useState(initialTasks);
  const checkBoxHandler = (id: number, checked: boolean) => {
    setTasks(
      tasks.map((task) => {
        if (task.id === id) {
          return { ...task, done: checked };
        } else {
          return task;
        }
      })
    );
  };

  return (
    <>
      <b>{title}</b>
      <ul>
        {tasks.map((task) => (
          <li key={task.id}>
            <input
              type="checkbox"
              checked={task.done}
              onChange={(e) => checkBoxHandler(task.id, e.target.checked)}
            />
            {task.title}
          </li>
        ))}
      </ul>
    </>
  );
};
export default TaskList;
```
[CodeSandbox](https://codesandbox.io/s/react-state-k7crss)

## Passing properties in React

We can provide properties to our component in the parameter of our component function. In the below example, we provide 2 properties - `task` and `checkBoxHandler`, inside the curly brackets `{}`, which denotes the contents as part of an object. 

```
const TaskItem = ({ task, checkBoxHandler }: TaskItemProps) => {
  return (
    <li>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => checkBoxHandler(task.id, e.target.checked)}
      />
      {task.title}
    </li>
  );
};
```

Because we are using typescript, the types of everything needs to be provided, so we need to declare the type of the parameter object passed in to the component, so we declare this object as `TaskItemProps`, and we declared it before the function as such.

```
type TaskItemProps = {
  task: Task;
  checkBoxHandler: (id: number, checked: boolean) => void;
};
```

So the full result is as below. 

```
import { useState } from "react";

interface Task {
  id: number;
  title: string;
  done: boolean;
}

const initialTasks: Task[] = [
  { id: 0, title: "Buy Milk", done: false },
  { id: 1, title: "Water Plants", done: true },
  { id: 2, title: "Send Letter", done: false }
];

type TaskItemProps = {
  task: Task;
  checkBoxHandler: (id: number, checked: boolean) => void;
};

const TaskItem = ({ task, checkBoxHandler }: TaskItemProps) => {
  return (
    <li>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => checkBoxHandler(task.id, e.target.checked)}
      />
      {task.title}
    </li>
  );
};

const TaskList = () => {
  const title: string = "Tasks";
  const [tasks, setTasks] = useState(initialTasks);
  const checkBoxHandler = (id: number, checked: boolean) => {
    setTasks(
      tasks.map((task) => {
        if (task.id === id) {
          return { ...task, done: checked };
        } else {
          return task;
        }
      })
    );
  };

  return (
    <>
      <b>{title}</b>
      <ul>
        {tasks.map((task) => (
          <TaskItem
            key={task.id}
            task={task}
            checkBoxHandler={checkBoxHandler}
          ></TaskItem>
        ))}
      </ul>
    </>
  );
};
export default TaskList;
```
[CodeSandbox](https://codesandbox.io/s/react-properties-g2g8dv?file=/src/App.tsx)

## Use useRef to keep variables updated but not triggering re-renders

In React, when we declare any variables normally like `let title = 'hello world'`, and try to mutate it by `title = 'bye world'`, the `title` will be reset back to `hello world` every time the component re-renders. If we want to keep the variable updated, but we don't need to trigger any re-rendering when the value changes, we can use `useRef` to declare it, and use `<ref>.current` to retrieve it or mutate it. 

The below example shows a number text box and an `add` button. Typing a number in the text box and click the add button will add the number to the previous total (which is 0 for the first time), and show the total below. There is another `undo` button, which undoes the last action. In order to do that, each time the `add` button is clicked, the number will be added to a history list. So we need the data in the history list to persist throughout the re-renders, but we don't need any changes to the history to trigger a re-render, so a `useRef` will be the solution.

![accumulator ui](/assets/images/2023/07/accumulator.png)

```
import { useState, useRef } from "react";

const Accumulator = () => {
  const [value, setValue] = useState("");
  const [total, setTotal] = useState(0);
  let history = useRef<number[]>([]);
  const updateTotal = (num: number) => {
    setTotal((total) => total + num);
    history.current.push(num);
    setValue("");
  };
  const undoAdd = () => {
    if (history.current.length > 0) {
      let lastValue = history.current.pop() || 0;
      setValue(String(lastValue));
      setTotal((total) => total - lastValue);
    }
  };
  return (
    <div>
      <input
        type="number"
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
      <button onClick={(e) => updateTotal(+value)}>Add</button>
      <br />
      Total: {total}
      <br />
      <button disabled={history.current.length === 0} onClick={() => undoAdd()}>
        Undo
      </button>
    </div>
  );
};

export default Accumulator;
```
[CodeSandbox](https://codesandbox.io/s/react-ref-demo-wnfljc)

## Use useEffect to create side effects after rendering

1. Call `useEffect(() => { ...execute something })` to execute something everytime after each render

2. Call `useEffect(() => { ...execute something }, [])` to execute something once after the component mounts the first time.

3. Call `useEffect(() => { ...execute something }, [someVar])` to execute something each time `someVar` changes value.

4. Call `useEffect(() => { ...execute something; return () => { ...execute cleanup } }, [])` to execute cleanup actions each time before the effect runs again, and once before the component unmounts.

Below is an example to use effects to fetch the list of users when the component mounts the first time, and fetches the todos for the selected user when the selected user changes.

```
import { useEffect, useState } from "react";

interface Todo {
  userid: number;
  id: number;
  title: string;
  completed: boolean;
}

interface User {
  id: number;
  name: string;
}

export default function TodoList() {
  const [users, setUsers] = useState<User[]>([]);
  const [selectedUserId, setSelectedUserId] = useState<number | undefined>(
    undefined
  );
  const [todos, setTodos] = useState<Todo[]>([]);

  async function fetchUsers() {
    await fetch("https://jsonplaceholder.typicode.com/users")
      .then((response) => response.json())
      .then((json) => {
        setUsers(json);
        setSelectedUserId(json && json.length > 0 ? json[0].id : 0);
      });
  }

  async function fetchTodos(userId: number) {
    await fetch(`https://jsonplaceholder.typicode.com/users/${userId}/todos`)
      .then((response) => response.json())
      .then((json) => setTodos(json));
  }

  useEffect(() => {
    fetchUsers();
  }, []);

  useEffect(() => {
    if (selectedUserId) {
      fetchTodos(selectedUserId!);
    }
  }, [selectedUserId]);

  return (
    <div>
      User:
      <select
        value={selectedUserId}
        onChange={(e) => setSelectedUserId(Number(e.target.value))}
      >
        {users.map((user) => (
          <option key={user.id} value={user.id}>
            {user.name}
          </option>
        ))}
      </select>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            {todo.title} {todo.completed && "✔"}
          </li>
        ))}
      </ul>
    </div>
  );
}
```
[CodeSandbox](https://codesandbox.io/s/react-effects-demo-gfjxg4)
