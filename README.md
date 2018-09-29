# Angular Model - @angular-extensions/model

by [@tomastrajan](https://twitter.com/tomastrajan)

[![npm](https://img.shields.io/npm/v/@angular-extensions/model.svg)](https://www.npmjs.com/package/@angular-extensions/model) [![npm](https://img.shields.io/npm/l/@angular-extensions/model.svg)](https://github.com/angular-extensions/model/blob/master/LICENSE) [![npm](https://img.shields.io/npm/dm/@angular-extensions/model.svg)](https://www.npmjs.com/package/@angular-extensions/model) [![Build Status](https://travis-ci.org/angular-extensions/model.svg?branch=master)](https://travis-ci.org/angular-extensions/model) [![Twitter Follow](https://img.shields.io/twitter/follow/tomastrajan.svg?style=social&label=Follow)](https://twitter.com/tomastrajan)[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

Simple state management with minimalistic API, one way data flow,
multiple model support and immutable data exposed as RxJS Observable.

- [Changelog](https://github.com/angular-extensions/model/blob/master/CHANGELOG.md)

## Documentation

![@angular-extensions/model dataflow diagram](https://raw.githubusercontent.com/tomastrajan/angular-model-pattern-example/master/src/assets/model_graph.png 'ngx-model dataflow diagram')

## Getting started in Angular CLI projects

1.  Install `@angular-extensions/model` library

    ```
    ng add @angular-extensions/model
    ```

2.  Generate model service

    ```
    ng g @angular-extensions/model:model examples/todo --items
    ```

3.  Use model service in your component. Let's generate new `todo` component

    ```
    ng g component examples/todo --inline-template
    ```

    And then adjust component implementation as in the example below

    ```ts
    import { Component } from '@angular/core';
    import { TodoService } from './todo.service';

    @Component({
      selector: 'app-todo',
      template: `
        <!-- template subscription to todos using async pipe -->
        <ng-container *ngIf="todoService.todos$ | async as todos">
          <h1>Todos ({{todos.length}})</h1>
          <ul>
            <li *ngFor="let todo of todos">
              {{todo.prop}}
            </li>
          </ul>
          <button (click)="addTodo()">Add todo</button>
        </ng-container>
      `,
      styleUrls: ['./todo.component.css']
    })
    export class TodoComponent {
      constructor(public todoService: TodoService) {}

      addTodo() {
        this.todoService.addTodo({ prop: 'New todo!' });
      }
    }
    ```

4.  Use our new `<app-todo></app-todo>` component in the template of the `app.component.html`

Please mind that you might be using different application prefix than `app-` o adjust accordingly.

## Model API

The model has a small API that as shown in in the illustration above.

- `get(): T` - returns current model value
- `set(data: T): void` - sets new model value
- `data$: Observable<T>` - observable of the model data, every call `set(newData)` will push new model state to to this observable (the data is **immutable by default** but this can be changed using one of the other provided factory functions as described below)

Check out generated `todo.service.ts` to see an example of how the model should be used.
In general, the service will implement methods in which it will retrieve current model state, mutate it and set new state back to the model.
Model will then take care of pushing immutable copies of the new state to all components which are subscribed using `data$`.

#### Available Model Factories

Models are created using model factory as shown in example `todo.service.ts`, check line `this.model = this.modelFactory.create(initialData);`.
Multiple model factories are provided out of the box to support different use cases:

- `create(initialData: T): Model<T>` - create basic model which is immutable by default (`JSON` cloning)
- `createMutable(initialData: T): Model<T>` - create model with no immutability guarantees (you have to make sure that model consumers don't mutate and corrupt model state) but much more performance because whole cloning step is skipped
- `createMutableWithSharedSubscription(initialData: T): Model<T>` - gain even more performance by skipping both immutability and sharing subscription between all consumers (eg situation in which many components are subscribed to single model)
- `createWithCustomClone(initialData: T, clone: (data: T) => T)` - create immutable model by passing your custom clone function (`JSON` cloning doesn't support properties containing function or regex so custom cloning functionality might be needed)

## Model Schematics API

Model services are generated using Angular CLI. It is a 3rd party schematics so we have to
specify it when running `ng g` command like this `ng g @angular-extensions/model:<schematics-name> <schematics parameters>`.
The schematics currently contains only one schematic called `model`.

#### Basic usage

```
ng g @angular-extensions/model:model example/todo
```

#### Supported options

- `--items` - creates service for collection of items (it will expose `todos$: Observable<Todo[]>;` instead of `todo$: Observable<Todo>`)
- `--flat` - generates service file directly in the`examples` folder without creating folder with the name `todos` (default: `false`)
- `--spec` - generate service test file (default: `true`)
- `--module` - will decide how to register service into Angular dependency injection context (service will use `providedIn: 'root'` when no module was provided, module can be provided as a path to module relative to the location of generated service, eg `ng g @angular-extensions/model:model examples/auth --module ../app.module.ts`)
- `--project` - project in which to generate the service (for multi project Angular CLI workspaces, will generate service in the first project by default, when no project was provided)

## Getting started without Angular CLI

It is also possible to use `@angular-extensions/model` in Angular project which do not use Angular CLI.

1.  Install `@angular-extensions/model` library

    ```
    npm i -S @angular-extensions/model
    ```

2.  Create new model service in `src/app/examples/todo/todo.service.ts`

    ```ts
    import { Injectable } from '@angular/core';
    import { Model, ModelFactory } from '@angular-extensions/model';
    import { Observable } from 'rxjs';

    const initialData: Todo[] = [];

    @Injectable({
      providedIn: 'root'
    })
    export class TodoService {
      private model: Model<Todo[]>;

      todos$: Observable<Todo[]>;

      constructor(private modelFactory: ModelFactory<Todo[]>) {
        this.model = this.modelFactory.create(initialData);
        this.todos$ = this.model.data$;
      }

      addTodo(todo: Todo) {
        const todos = this.model.get();

        todos.push(todo);

        this.model.set(todos);
      }
    }

    export interface Todo {
      prop: string;
    }
    ```

3.  Use new model service in some of your components as described in point 3 and above in `Getting started in Angular CLI projects` section

## Relationship to older Angular Model Pattern and `ngx-model` library

This is a new enhanced version of older library called `ngx-model` which was in turn implementation of [Angular Model Pattern](https://tomastrajan.github.io/angular-model-pattern-example).
All the original examples and documentation are still valid. The only difference is that
you can add `@angular-extensions/model` with `ng add` instead of installing `ngx-model` or having to copy model pattern
implementation to your project manually.

Check out the [Blog Post](https://medium.com/@tomastrajan/model-pattern-for-angular-state-management-6cb4f0bfed87) and
[Advanced Usage Patterns](https://tomastrajan.github.io/angular-model-pattern-example#/advanced)
for more how-tos and examples.
