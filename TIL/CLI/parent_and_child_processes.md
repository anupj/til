# Parent and child processes

- When one instance of the shell invokes another, the original shell
  is called the _parent_ and the new instance is called the _child_.

> This is true of any Linux program that invokes any other Linux program

- The invoking program is the _parent_, and the invoked program is its _child_.

- A running Linux program is known as a _process_. A _process_ can invoke any
  number of children, but each child has only one parent.

- Every time you run a simple command, **you create a child process**. Even when
  you run a simple command like `ls`, that command secretly runs inside a new child process
  with its own (copied) environment.
