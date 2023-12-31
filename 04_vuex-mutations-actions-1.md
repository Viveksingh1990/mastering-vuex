Mutations & Actions Pt. 1
=========================

Now that we have access to our Vuex State, we can start to store our application’s data there. With Vuex, we can achieve this by using a Mutation to place data in our State. In this tutorial, we’ll look at Mutations and then see how we can wrap Mutations in Actions to make them more scalable and future-proof.

* * *

Mutations
=========

As we discovered in the [Intro to Vuex](https://www.vuemastery.com/courses/real-world-vue-js/intro-to-vuex) lesson, we can use Mutations to update, or mutate, our State.

For a simple example, let’s say our State has a `count` property:

**store.js**

        state: {
          count: 0
        }
    

Now, below our state, we can write a mutation that allows us to increment that value.

**store.js**

    
        mutations: {
          INCREMENT_COUNT(state) {
            state.count += 1
          }
        }
    

As you can see, our `INCREMENT_COUNT` mutation is taking in our Vuex state as an argument and using it to increment the `count`.

Now, let’s _commit_ that Mutation from within a component. Inside our **EventCreate** component, we’ll add a method:

        incrementCount() {
          this.$store.commit('INCREMENT_COUNT')
        },
    

Here, our `incrementCount` method simply commits the `INCREMENT_COUNT` Mutation that it has access to with `this.$store`.

If we add a button, we can click on it to trigger this Mutation.

        <button @click="incrementCount">Increment</button>
    

Checking the Vue DevTools, we can see our `count` is being updated in the Vuex tab.

* * *

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578381056952_0.png?alt=media&token=1450d615-4f72-4302-961f-7809feee30a0)

Also, notice how our Mutation was logged in the DevTools as well. If we click on **Base State**, we’re able to see the State of our app prior to the Mutation being committed. In other words, `count` reverts to 0.

This allows us to do “time-travel debugging” so we can see what the State of our application was at given points in time, and we can see how our Mutations affected our State.

**Why All Caps?** If you’re wondering why our Mutation is in all capital letters, that’s because it’s common within Flux-based patterns to put them in all caps. This is entirely optional, and you’ll often see Mutations written in camelCase instead. All caps does make it more immediately visually clear what Mutations are available to you when scanning your files, and more clear when you are committing a Mutation versus an Action, Getter, etc. But again, the choice is up to you (and/or your team).

* * *

Dynamic Mutations
-----------------

Currently, we’re only updating our `count` by 1. What if we wanted to update it by a dynamic value? We can pass in a payload to a Mutation to make it dynamic.

To see this in action, let’s add an input to our template, and use v-model to bind it to a new data property called `incrementBy`.

        <input type="number" v-model.number="incrementBy">
    

Note that we are using the `.number` modifier to typecast the input value as a number.

        data() {
          return {
            incrementBy: 1
          }
        }
    

Now we’ll pass in the `incrementBy` value from our data as a payload when we commit our Mutation.

        incrementCount() {
          this.$store.commit('INCREMENT_COUNT', this.incrementBy)
        },
    

In our Vuex Store, the `INCREMENT_COUNT` Mutation can receive that payload in its second argument and use it to update our `count` dynamically.

        INCREMENT_COUNT(state, value) {
          state.count += value
        }
    

Now whatever number is typed into the input can be used to update our `count` State.

* * *

Actions
-------

While Vuex Mutations are synchronous, meaning they will happen one after the other, Actions can be asynchronous. They can contain multiple steps that actually happen in an order different from the order in which they are written. If you remember from our lesson on APIs, Axios functions asynchronously.

We can use Actions to wrap some business logic around a Mutation, or Mutations.

In our Intro to Vuex lesson, we looked at how an Action could be written to commit a Mutation that sets the `isLoading` State to true, then makes an API call, and when that call’s response returns, it commits a Mutation to set the `isLoading` State to false before committing a Mutation to set the `todos` State with the API’s response.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370696756_1.png?alt=media&token=512b37b9-fc19-4691-a8e5-274abc4a8907)

It’s important to understand that the Mutations within an Action may or may not be committed, depending on how the surrounding logic and circumstances pan out.

For a real-life example, if I asked my friend to pick up some bread from the store, the Mutation here would be **PICK\_UP\_BREAD** whereas the Action is more like **pleasePickUpBread.** There’s a big difference between asking for someone to do something and them actually doing it.

There could be plenty of reasons why she wouldn’t be able to commit that Mutation, so to speak. Her car may break down on the way to the store, or the store might be out of bread. So Actions are more like expressing an intent or desire for something to happen, for some change to be made to the state, depending upon some surrounding circumstances.

Now let’s see Actions in action.

* * *

Seeing them in Action
---------------------

Going back to our counter example, if we only wanted to update the `count` if our app has a `user`, we could write:

        actions: {
          updateCount({ state, commit }, incrementBy) {
            if (state.user) {
              commit('INCREMENT_COUNT', incrementBy)
            } 
        }
    

So what’s happening here?

We’ve created an Action called `updateCount`. It is using [object destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Object_destructuring) to get `state` and `commit` from the Vuex context object: `{ state, commit }`.

The context object is the first argument of any Action, and it exposes the same set of properties that are on the store instance (state, mutations, actions, getters). So you can call `context.commit` to commit a mutation, for example. Or say `context.state.count` to get the value of the `count` State.

Additionally, `updateCount` is taking in the payload `value`.

        ({ state, commit }, value)
    

The payload is the second argument of all Actions.

Our Action is checking to see if we have a `user` stored in our State. If we do, we’ll _commit_ the `INCREMENT_COUNT` Mutation with the `incrementBy` value we’ve passed in as the payload. If we do not have a `user`, the Mutation will not be committed.

Now, within our component, we’d _dispatch_ the Action, which is what _commits_ the Mutation.

        incrementCount() {
          this.$store.dispatch('updateCount', this.incrementBy)
        },
    

It’s important to note that it is recommended to always commit a Mutation from within an Action. While this might initially seem like unnecessary code if your Mutation doesn’t currently need any business logic, doing so future-proofs your app and allows it to scale better. It’s much easier to add the Action now than to refactor a bunch of code throughout your app when you need to later.

Now that we understand how to commit Mutations, and wrap them in an Action, let’s add to our example app.

* * *

Adding to Our Example App
-------------------------

Currently in our app, we are just pulling events from our mock API. But we want a user to be able to create a new event, too, which is added, or _stored,_ within the Vuex Store. We’ll add a Mutation and commit it from an Action.

But first, we need to install a new dependency.

* * *

Installing our Date Picker
--------------------------

We’re about to build out a form that is used to create a new event. But we need a date picker for our form, so let’s download a popular external library for that: [vuejs-datepicker](https://github.com/charliekassel/vuejs-datepicker).

From our command line, we’ll write: `npm install vuejs-datepicker --save`

This will install the library into our project so we can start using it.

* * *

Creating Events
---------------

Let’s head over to our **EventCreate** component because, like it sounds, we’ll be using it to create new events. Just like how we had an input element that used `v-model` to bind values to our data, and a button to commit a Mutation, we’ll use this same process but in an expanded version, with a form that can collect data from our users so they can create new events.

Below is the template for our form. Notice we’re using our newly added `datepicker`.

            <form>
              <label>Select a category</label>
              <select v-model="event.category">
                <option v-for="cat in categories" :key="cat">{{ cat }}</option>
              </select>
              <h3>Name & describe your event</h3>
              <div class="field">
                <label>Title</label>
                <input v-model="event.title" type="text" placeholder="Add an event title"/>
              </div>
              <div class="field">
                <label>Description</label>
                <input v-model="event.description" type="text" placeholder="Add a description"/>
              </div>
              <h3>Where is your event?</h3>
              <div class="field">
                <label>Location</label>
                <input v-model="event.location" type="text" placeholder="Add a location"/>
              </div>
              <h3>When is your event?</h3>
              <div class="field">
                <label>Date</label>
                <datepicker v-model="event.date" placeholder="Select a date"/>
              </div>
              <div class="field">
                <label>Select a time</label>
                <select v-model="event.time">
                  <option v-for="time in times" :key="time">{{ time }}</option>
                </select>
              </div>
              <input type="submit" class="button -fill-gradient" value="Submit"/>
            </form>
    

As you can see, we’re asking a series of questions and using `v-model` on input elements to bind the user’s responses to our data.

Let’s take a look at the `script` section of this component altogether, then explain it piece by piece.

        <script>
        import Datepicker from 'vuejs-datepicker'
        export default {
          components: {
            Datepicker
          },
          data() {
            const times = []
            for (let i = 1; i <= 24; i++) {
              times.push(i + ':00')
            }
            return {
              event: this.createFreshEvent(),
              times,
              categories: this.$store.state.categories,
            }
          },
          methods: {
            createFreshEvent() {
              const user = this.$store.state.user
              const id = Math.floor(Math.random() * 10000000)
              return {
                id: id,
                category: '',
                organizer: user,
                title: '',
                description: '',
                location: '',
                date: '',
                time: '',
                attendees: []
              }
            }
          }
        }
        </script>
    

Let’s break this down.

        import Datepicker from 'vuejs-datepicker'
        export default {
          components: {
            Datepicker
          }
    

At the top, we’re importing our new `datepicker` and registering it as a child component so we can use it in our template.

        data() {
          const times = []
          for (let i = 1; i <= 24; i++) {
            times.push(i + ':00')
          }
          return {
            ...
            times
          }
    

Then in our `data`, we’re using a small algorithm to generate an array of numbers to use for our `times`. Note that above in our data, `{ times }` is the same as `{ times: times }`. If it looks odd to put this logic here, remember that `data()` is a function, so you’re perfectly capable of performing some initial data-based logic within it.

        <select v-model="event.time">
          <option v-for="time in times" :key="time">{{ time }}</option>
        </select>
    

We’re then `v-for`ing through `times` in the template.

Now, let’s look at the rest of our `data`.

        return {
          event: this.createFreshEventObject(),
          categories: this.$store.state.categories,
          times
        }
    

We’re retrieving our `categories` directly from the Vuex Store, and using `v-for` on an `option` element just like with our `times`. But we’re doing something unique with our `event` data.

        event: this.createFreshEventObject(),
    

Instead of putting an event object in our data directly, we’re calling a method that generates a fresh event object whenever this component is created.

That method looks like this:

        createFreshEventObject() {
          const user = this.$store.state.user
          const id = Math.floor(Math.random() * 10000000)
          return {
            id: id,
            category: '',
            organizer: user,
            title: '',
            description: '',
            location: '',
            date: '',
            time: '',
            attendees: []
          }
        }
    

We’re retrieving our `user` from our Vuex Store then returning an object with all of the properties for which we want to collect data, including a property that is populated with our `user` state. We’re also creating a random number for our `id`, and using that to set the `id` of our event.

You might be wondering why we have this method. Why not just have all these properties on our `data` itself? Well, when we submit an event, we want to reset this component’s event data, and this method is a handy way for us to do that. You’ll see us using it later.

If we did not reset our local event object, we could be retaining unnecessary connections between this object and the one we push into our State.

Finally, we just need to add a simple scoped style:

        .field {
          margin-bottom: 24px;
        }
    

* * *

The ADD\_EVENT Mutation
-----------------------

Now in our Vuex Store, we’ll write an `ADD_EVENT` Mutation.

        ADD_EVENT(state, event) {
          state.events.push(event)
        },
    

It receives an `event` argument, then pushes it onto our `events` state.

* * *

The createEvent Action
----------------------

Now, we want to wrap this in an Action, which we’ll call `createEvent`.

But first, we need to import our **EventService.js** file at the top of our **store.js**.

        import EventService from '@/services/EventService.js'
    

Because we’re using it in our Action:

        createEvent({ commit }, event) {
          EventService.postEvent(event)
          commit('ADD_EVENT', event)
        })
    

As you can see, our Action is using the `EventService` we created in our [API Calls with Axios](https://www.vuemastery.com/courses/real-world-vue-js/API-calls-with-Axios) lesson in order to do a POST request with `postEvent(event)`, which adds the event to our local `json.db` file.

See? We’ve added a new POST request to our **EventService** file:

**EventService.js**

        postEvent(event) {
          return apiClient.post('/events', event)
        }
    

It receives an event and can POST it to this endpoint, where our mock events database lives.

Our `createEvent` Action then _commits_ the `ADD_EVENT` Mutation that we just created, which adds the event to our local `events` state in case our app’s UI immediately needs access to that new `event` state.

Now, let’s _dispatch_ this Action from our component.

* * *

Dispatching the eventCreate Action
----------------------------------

Returning to our **EventCreate** component, we can add a method that will _dispatch_ our new Action.

        methods: {
          createEvent() {
            this.$store.dispatch('createEvent', this.event)
          },
        ...
    

We’ll trigger this method when our form is submitted, with:

        <form  @submit.prevent="createEvent">
    

* * *

Resetting our Event Data
------------------------

* * *

Earlier I mentioned that we want to reset our component’s event data object every time a new event is submitted.

We’ll do that like so:

        createEvent() {
          this.$store.dispatch('createEvent', this.event)
          this.event = this.createFreshEventObject()
        }
    

**Problem:** But we don’t want to clear out our event until we know it’s been added to our backend. What if our user was creating an event, clicked submit, stepped onto an elevator and the event never submitted. They’d have to start creating the event all over again.

**Solution:** In our Action, here we can `return` the response from our API. And `.then` commit our Mutation.

        createEvent({ commit }, event) {
          return EventService.postEvent(event).then( () => {
              commit('ADD_EVENT', event.data)
            })
        }
    

Now, when the event is successfully POSTed, we’ll _commit_ `ADD_EVENT`. And we can even wait for the response to return from with our **EventCreate** component, like so:

        createEvent() {
          this.$store.dispatch('createEvent', this.event)
            .then(() => {
              this.event = this.createFreshEventObject()
            })
            .catch(() => {
              console.log('There was a problem creating your event.')
            })
        }
    

Now above, we will only reset our event data (`this.event`) if the POST request was a success.

If the POST request was not a success, we’ll log an error to the console. In a future lesson, we’ll cover how to effectively display this error to the user.

* * *

Routing to our New Event
------------------------

Once we successfully create an event, we’ll want to view that event. In other words, we want to route our user to the **event-show** page for the event they just created.

We can use the [router.push](https://router.vuejs.org/guide/essentials/navigation.html#router-push-location-oncomplete-onabort) method to achieve this, and set the `id` params to be the id of `this.event` that we just created.

        createEvent() {
          this.$store
            .dispatch('createEvent', this.event)
            .then(() => {
              this.$router.push({
                name: 'event-show',
                params: { id: this.event.id }
              })
              this.event = this.createFreshEventObject()
            })
            .catch(() => {
              console.log('There was a problem creating your event.')
            })
        }
    

We just need to make sure we clear the event _after_ we strip the `id` from it for our router params, otherwise `this.event` would be undefined.

* * *

Adjusting EventShow
-------------------

Now we just need to adjust the **EventShow** component’s template so it doesn’t display our entire user object for the event organizer.

We need to now use dot notation to display `event.organizer.name` and do so with a ternary operator so we don’t get an error that `name is undefined` before the component has the data it needs as it renders.

Remember? We did this with the attendees in an earlier lesson:

        <span class="badge -fill-gradient">{{ event.attendees ? event.attendees.length : 0 }}</span>
    

So let’s do that for our organizer name now:

        <h5>Organized by {{ event.organizer ? event.organizer.name : '' }}</h5>
    

* * *

Let’s ReVue
-----------

In this lesson, we looked at Vuex Mutations and how to use them with Actions that can perform potentially asynchronous business logic. We then took that knowledge and added to our example app so our users can create new events, which are added to our mock database as well as our Vuex Store.

* * *

What’s Next?
------------

In the next lesson, we’ll continue learning about Mutations and Actions and how to use them to effectively fetch our event data from an API.

### Lesson Resources

##### Source Code:

*   [Official Mutations Docs](https://vuex.vuejs.org/guide/mutations.html)
    
*   [Official Actions Docs](https://vuex.vuejs.org/guide/actions.html)
    
*   [Object Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Object_destructuring)
    
*   [Vue Router Push](https://router.vuejs.org/guide/essentials/navigation.html#router-push-location-oncomplete-onabort)
    
*   [Starting code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson12-mutations%26actions1-start)
    
*   [Ending code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson12-mutations%26actions1-finish)
