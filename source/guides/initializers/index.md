## Using initializers

When an Ember.js application starts up you often want to configure things like 3rd party libraries. Ember.js allows you to register initializers that will be called when the app starts up.


```javascript
Ember.Application.initializer({
  name: "initializerName",
 
  initialize: function(container, application) {
    // your initialization code
  }
});
```

A basic initializer requires two things, a name (which allows initializer ordering, see below) and an initialize function containing the code you want to run at startup time. You're given access to the container (for access to things like the ember-date store) and the application which can be used to control asynchronous initializers(see below).

```javascript
App.initializer({
  after: "store",
  name: "second",

  initialize: function(container, application) {
    var store = container.lookup("store:main");
    // do things with store
  }
});
```

### Initializer ordering
It's often the case that your initializer must be run after another initializer e.g. your initializer needs access to the ember-data store. Ember allows you to specify 'before' and 'after' so that you may dictate the order in which the initializers are run.

```javascript
App.initializer({
  name: "first",
  initialize: function() {
    // called first
  }
});

App.initializer({
  after: "first",
  name: "second",
  before: "last",

  initialize: function() {
    // called after first but before last
  }
});

App.initializer({
  name: "last",

  initialize: function() {
    // called after second
  }
});
```

### Asynchronous initializers

Often your initializers will depend on asynchronous calls (perhaps ajax) and you need to tell ember to wait until you're done before continuing.

```javascript
Ember.Application.initializer({
  name: "currentUser",
 
  initialize: function(container, application) {
      // Pause the initializer chain
      application.deferReadiness();

      var store = container.lookup("store:main");
      store.find('user', 'current').then(function(currentUser){

      	container.register('user:current', user, {instantiate: false, singleton: true});
      	container.injection('route', 'currentUser', 'user:current');
        container.injection('controller', 'currentUser', 'user:current');

      	// Resume the initializer chain
      	application.advanceReadiness();
      });
   }
});
```

### Initializers for muliple ember apps

If you're loading multiple ember apps on the same page you may specify different initializers for each app.

```javascript
AppABase = Ember.Application.extend();
AppBBase = Ember.Application.extend();

AppABase.initializer({
	name: "appAInitializer",
	initialize: function(container, application){

  }
});

AppBBase.initializer({
	name: "appBInitializer",
	initialize: function(container, application){

  }
});

AppA = AppABase.create({...});
AppB = AppBBase.create({...});
```
