# Overview

# Architecture Overview

Chaplin is an architecture for JavaScript web applications using the [Backbone.js](http://documentcloud.github.com/backbone/) library. The code is derived from [moviepilot.com](http://moviepilot.com/), a large single-page application.

While Backbone is an easy starting point, it provides only basic, low-level patterns. Especially, Backbone provides little structure above simple routing, individual models, views and their binding. Chaplin addresses this limitations by providing a light-weight but flexible structure which leverages well-profen design patterns and best practises.

## Chaplin’s Structure

From top to bottom, a Chaplin application consists of these modules:

* `Application` – The bootstrapper of the whole application
* `Router` – Mapping URLs to controller actions based on a configuration file
* `Dispatcher` – Starting and stopping controllers when a route matches
* `Layout` – Showing and hiding of main views, handling of internal links
* `mediator` – Cross-module communication using Publish/Subscribe
* Several `Controllers` – individual application modules
* `Models` and `Collections` hold the data, `Views` provide the user interface

## Application Flow

Every Chaplin application starts with a class that inherits from `Application`. This is merely a bootstrapper which instantiates and configures the four core moules: `Router`, `Dispatcher`, `Layout` and `mediator`.

After creating the `Router`, the routes are registered. Usually they are read from a configuration file called  `routes.coffee`/`routes.js`. A route maps a URL pattern to a controller action. For example, the path `/` can be mapped to the `index` action of the `HomeController`.

Eventually, the `Application` starts the routing. The `Router` starts to observe the current URL. If a route matches, it notifies the other modules.

This is where the `Dispatcher` takes over. It loads the target controller and its dependencies (e.g. `HomeController`). Then, the controller is instantiated and the controller action is called (e.g. `index`). An *action* is a method of the controller. The `Dispatcher` also keeps track of the currently active controller, and disposes the previously active controller.

Typically, a controller creates a `Model` or `Collection` and a corresponding `View`. The model or collection may fetch some data from the server which is then rendered by the view. By convention, the models, collection and views are saved as properties on the controller instance.

In a Chaplin application, the `Layout` module serves as the top-level view manager. After a controller action was called, the `Layout` is responsible for switching the main view. It hides the view of the previous controller and shows the view of the new controller.

The `Layout` also observes clicks on `a` links in the document. If the user clicks on an internal link, it notifies the `Dispatcher` so the target controller is started.

Last but not least, there’s the `mediator`. It allows other application modules to communicate with each other in a decoupled and robust way.

---

## The Chaplin Modules in Detail

### Router

The `Router` is responsible for observing URL changes. It maps URLs to *separate controllers*, in particular *controller actions*.

By convention, routes should be declared in a separate file, the `routes` module. For example:

```coffeescript
match 'likes/:id', 'likes#show'
```

This works much like the [Ruby on Rails counterpart](http://guides.rubyonrails.org/routing.html). If a route matches, a `matchRoute` event is published passing a `params` hash which contains pattern matches (like `id` in the example above) and additional GET parameters.

[Learn more about the Router](router.html)

### Dispatcher

Between the router and the controllers, there is the `Dispatcher` listening for routing events. On such events, it loads the target controller, creates an instance of it and calls the target action. The action is actually a method of the controller. The previously active controller is automatically disposed.

A specific controller may also be started programatically. To start a specific controller, an app-wide `!startupController` event can be published:

```
mediator.publish '!startupController', 'controller', 'action', params
```

The `Dispatcher` handles the `!startupController` event.

[Learn more about the Dispatcher](dispatcher.html)

### Layout

The `Layout` is the top-level application view. When a new controller was activated, the `Layout` is responsible for changing the main view to the view of the new controller.

In addition, the `Layout` handles the activation of internal links. That is, you can use a normal `<a href="/foo">` element to link to another application module.

[Learn more about the Layout](layout.html)

### Controllers

A controller is the place where a model and associated views are instantiated. Typically, a controller represents one screen of the application. There can be one current controller which provides the main view and represents the current URL.

By convention, there is a controller for each application module. A controller may provide several action methods like `index`, `show`, `edit` and so on. These actions are called by the `Dispatcher` when a route matches.

[Learn more about controllers](controller.html)

### Mediator

The mediator is an event broker that implements the [Publish/Subscribe](http://en.wikipedia.org/wiki/Publish/Subscribe) design pattern. It should be used for most of the inter-module communication in Chaplin applications. Modules can emit events using `mediator.publish` in order to notify other modules, and listen for such events using `mediator.subscribe`. The mediator can also be used for sharing data between several modules easily, like a user model or other persistent and globally accessible data.

[Learn more about the mediator](mediator.html)

## Memory Management and Object Disposal

One of the core concerns of the Chaplin architecture is a proper memory management. While there isn’t a broad discussion about garbage collection in JavaScript applications, it’s an important topic. In event-driven systems, registering events creates references between objects. If these references aren’t removed when one of the modules isn’t used any longer, the garbage collector can’t free the memory.

Backbone provides little out of the box so Chaplin ensures that every controller, model, collection and view cleans up after itself. Chaplin extends the Model, Collection and View classes of Chaplin to implement a powerful disposal process.

[Learn more about disposal](disposal.html)

![Ending](http://s3.amazonaws.com/imgly_production/3362023/original.jpg)
