State & Getters
===============

In the last lesson, we saw an overview of how Vuex works. In this tutorial, we’ll look at how we can access the State from our Vuex store from within our components, both directly and with the help of Getters.

* * *

Accessing State
---------------

If we take a look at our **main.js** file, we see we’re importing our Vuex **store** file, and providing it to our root Vue instance. This was set up for us because we selected Vuex when we created our project with the Vue CLI.

        import store from './store' 
        
        new Vue({
          router,
          store, // <-- injecting the store for global access
          render: h => h(App)
        }).$mount('#app')
    

This makes the **store** globally accessible throughout our app by injecting it into every component. This way, any component can access the **store** and the properties on it (such as State, Actions, Mutations and Getters) by using `$store`.

Now let’s add some State so we can look at how to access it from a component. We can create a `user` object.

        state: {
          user: { id: 'abc123', name: 'Adam Jahr' }
        }
    

We can access this `user` State from anywhere in our app, but since we’ll soon be creating events that will need to know what user created them, let’s access this State from the **EventCreate.vue** file.

        <template>
          <h1>Create Event, {{ $store.state.user }}</h1>
        </template>
    

This works, but notice in the browser how we’re displaying the entire `user` object We can use dot notation to pinpoint the exact property from our `user` State that we want to display. In this case, we want to display just the `name`.

        <template>
          <h1>Create Event, {{ $store.state.user.name }}</h1>
        </template>
    

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370857331_0.png?alt=media&token=faeae2dc-2933-471e-8fa6-ad2722466dde)

Great, now we see our user’s name. But what if we needed to use the user’s name in multiple places within our component? Sure, we could write `this.$store.state.user.name` all over the place… Or we could write it once, in a computed property, called `userName`.

        computed: {
          userName() {
            return this.$store.state.user.name
          }
        }
    

This way, our template becomes more readable and less redundant.

        <h1>Create an Event, {{ userName }}</h1>
        <p>This event is created by {{ userName }}</p>
    

And if we needed to use it in a method of our component, we could simply say `this.userName`.

* * *

The `mapState` Helper
---------------------

If we need to access different parts of our State from the same component, it can get verbose and repetitive to have multiple computed properties each returning `this.$store.state.something`. To simplify things, we can use the `mapState` helper, which generates computed properties for us.

Let’s first add some more State to our Vuex Store so we can see this in action. We’ll add an array of event categories:

        state: {
          user: { id: 'abc123', name: 'Adam Jahr' },
          categories: ['sustainability', 'nature', 'animal welfare', 'housing', 'education', 'food', 'community']
        }
    

Now, in **EventCreate.vue**, we can import `mapState`

        import { mapState } from 'vuex'
    

Then use it to map our State to a computed property that can retrieve our user’s name, and our categories.

        computed: mapState({
          userName: state => state.user.name,
          categories: state => state.categories
        })
    

Notice how we’re using an arrow function that takes in `state` and returns the property of the state we want, `state.user.name` and `state.categories`.

If we’re wanting to access the top-level State (not using dot notation), there’s an even simpler way to write this, like so:

        computed: mapState({
          userName: state => state.user.name,
          categories: 'categories' // <-- simplified syntax for top-level State
        })
    

Notice all we need to do is use the State’s string value `'categories'`. This is equivalent to `state => state.categories`.

We could simplify the `mapState` syntax even more by passing an array of strings of the State values we want to map to computed properties, like so:

        computed: mapState(['categories', 'user'])
    

Of course, now in our template we’d just need to use dot notation to access our user’s name.

        <h1>Create an Event, {{ user.name }}</h1>
    

* * *

Object Spread Operator
----------------------

As you probably noticed, `mapState` returns an object of computed properties. But it’s currently preventing us from adding additional, local computed properties that aren’t mapped to our Store’s State.

To do that, we can use the [object spread operator](https://github.com/tc39/proposal-object-rest-spread), which allows us to mix in additional computed properties here.

        computed: {
          localComputed() {
            return something
          },
          ...mapState(['categories', 'user']) // <-- using object spread operator
        }
    

* * *

Getters
-------

While we can access the Store’s State directly, sometimes we want to access derived state. In other words, we might want to _process_ the state in some way when we access it.

For example, instead of accessing our State’s `categories`, we might want to know _how many_ categories there are. In other words, we might want to know the `categories` array’s length.

From within our component, we could say:

        this.$store.state.categories.length
    

But what if multiple components need to use this same value? By creating a Vuex Getter, we can avoid unnecessary code duplication. Also, since Getters are cached, this is a bit more performant of an option, too.

Let’s add a Getter to our Store.

**store.js**

        catLength: state => {
          return state.categories.length
        }
    

As you can see, Getters are a function that takes in the `state` as an argument, and allows us to return processed or filtered state.

* * *

**Using our Getter** Now let’s use our `catLength` Getter in our **EventCreate** component. Just like accessing State, we’ll put it in a computed property.

        computed: {
          catLength() {
            return this.$store.getters.catLength
          }
        }
    

If at any point the length of our `categories` State changes, our `catLength` Getter will recalculate and our computed property will update accordingly.

* * *

**Passing getters to Getters** If we needed to _get_ state that we want to process along with another Getter, we can pass in `getters` as the second argument to a Getter. This allows us to access another Getter from within the Getter we’re creating. I know, that sounds a bit confusing.

But for a simple example, let’s say we have an array of todos in our State.

        todos: [
              { id: 1, text: '...', done: true },
              { id: 2, text: '...', done: false },
              { id: 3, text: '...', done: true },
              { id: 4, text: '...', done: false }
            ]
    

We could have a Getter that gets an array of the todos that are labeled `done`.

        doneTodos: (state) => {
          return state.todos.filter(todo => todo.done)
        }
    

And we can use this Getter inside another Getter if we want to find out how many remaining todos there are to complete.

        activeTodosCount: (state, getters) => {
          return state.todos.length - getters.doneTodos.length
        }
    

Now we are able to `return` the difference between the number of todos that are `done` from the total number of todos.

You may be wondering why we wouldn’t just write `activeTodos` like this instead.

        activeTodosCount: (state) => {
          return state.todos.filter(todo => !todo.done).length
        }
    

And we could. This example was just to demonstrate the power of passing in `getters` to a Getter.

* * *

Dynamic Getters
---------------

You might be wondering if we can use dynamic Getters. In other words, can we retrieve some state based upon a parameter. And the answer is yes, we can achieve that by returning a function.

For example, if we had an array of events, we could retrieve an event by id like so:

        getEventById: (state) => (id) => {
          return state.events.find(event => event.id === id)
        }
    

Then in our component, we’d write:

        computed: {
          getEvent() {
            return this.$store.getters.getEventById
          }
        }
    

And in our template, we could pass in an argument.

        <p>{{ getEvent(1) }}</p>
    

Note that dynamic Getters like this will run each time you call them, and the result is not cached.

* * *

The `mapGetters` Helper
-----------------------

Just like we saw with accessing State, we can map Getters to computed properties on our component with the `mapgetters` helper.

First we’d just need to import it:

        import { mapGetters } from 'vuex'
    

Then we can use it like so:

        computed: mapGetters([
          'categoriesLength',
          'getEventById'
        ])
    

Now we have an array of computed properties in our component that are mapped to our Getters.

If we want to rename these Getters, we can do so in an object:

        computed: mapGetters({
          catCount: 'categoriesLength',
          getEvent: 'getEventById'
        })
    

Here, `this.catCount` is mapped to `this.$store.getters.categoriesLength` and `getEvent` is mapped to `this.$store.getters.getEventById`.

**Object Spread Operator** And as you might imagine, if you want to mix these Getters in with local computed properties, you can use the object spread operator here, too.

        computed: {
          localComputed() { return something }
          ...mapGetters({
            catCount: 'categoriesLength',
            getEvent: 'getEventById'
          })
        }
    

* * *

Let’s ReVue
-----------

We looked at accessing State from our components, directly from the template, then with the help of computed properties that we can `mapState` to. We then looked at how Getters allow us to process State when we access it in order to get derived state. And finally how the `mapGetters` helper can create computed properties for our Getters.

In our next lesson, we’ll learn how to add data to our State with Actions and Mutations.


### Lesson Resources

##### Source Code:

*   [Vuex Getters Docs](https://vuex.vuejs.org/guide/getters.html)
    
*   [Object Spread Operator](https://github.com/tc39/proposal-object-rest-spread)
    
*   [Starting code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson11-vuex-start)
    
*   [Ending code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson11-vuex-finish)
