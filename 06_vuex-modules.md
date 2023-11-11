Modules
=======

In the previous two lessons we used Vuex to encapsulate the state of our application and standardize how that state gets modified (through actions and mutations). However, as our application gets bigger, we’re going to end up with a gigantic `store.js` file. This is where Vuex modules come in, allowing us to keep our Vuex code organized and easier to test.

🛑 Problem: We need to organize our code
----------------------------------------

There has to be a better way to organize our Vuex code, as we’ve just put everything in our `store.js` file up until this point.

☑️ Solution
-----------

Vuex has an option called modules which makes it simple to split out different parts of your state into different files. For example, if your app has events and users it doesn’t make sense to pile all the state, mutations, actions, and getters into one big /src/store.js file. Instead, we might break the functionality into two separate Vuex modules.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370636027_0.gif?alt=media&token=81fffc36-4e58-4d4d-ac1d-b21120a7c2a4)

Later, we might have additional functionality with events having their own comments, and users having the events they can mark as “attending”. These features might also be candidates to split out into their own Vuex modules.

We can split out our Vuex code based on data models, or we can split it out based on features. How you implement this is entirely up to you.

* * *

👈 Back to our Example app
--------------------------

In our example app, let’s start by creating a `store` directory, and move our current `store.js` file inside of it. For the time being, let’s make sure our app still works after this move, by modifying our `main.js` to look inside our new directory. We simply need to change:

        import store from './store'
    

to:

        import store from './store/store'
    

Now our **store.js** file is importing with no problem.

* * *

🏗️ Building Our First Module
-----------------------------

Before we actually build out our first module, I’d like to add someplace where we’re printing out our user’s name. I’m going to change our homepage title so it prints our current user’s name:

📃 **/src/views/EventList.vue**

        <template>
          <div>
            <h1>Events for {{ user.name }}</h1>
            ...
        </template>
        <script>
            // omitting code  
            ...mapState(['events', 'eventsTotal', 'user'])
          }
        }
        </script>
    

We’re not adding any new code. I just want to show you that our `store.js` has the following for our user data:

📃 **/src/store/store.js**

        ...
        export default new Vuex.Store({
          state: {
            user: { id: 'abc123', name: 'Adam Jahr' },
            ...
    

So when we call up our homepage we see:

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370636028_1.jpg?alt=media&token=9bb230b8-151b-42a3-ad56-f8228cc0b5bc)

Now let’s build out our first **user** module, as in the future as we build out authentication in our example app we’ll be placing a lot more code in here. To do this, we’ll create a new **modules** folder with a new **user.js** file which just contains our user state.

📃 **/src/store/modules/user.js**

        export const state = {
          user: {
            id: 'abc123',
            name: 'Adam'  // I removed the last name Jahr here our title is on one line
          }
        }
    

Notice I removed Adam’s last name above so our title fits on one line. If you’re following along, feel free to change it to your first name.

Now in order to use this module, we need to include it inside our **store.js** like so:

📃 **/src/store/store.js**

        ...
        import * as user from '@/store/modules/user.js'
        // This pulls in all the constants in user.js 
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
          modules: {
            user  // Include this module
          },
          state: {
            categories: [
              'sustainability',
              // ...
    

In order to get this working in our component we need to add another `.user`:

📃 **/src/views/EventList.vue**

        <template>
          <div>
            <h1>Events for {{ user.user.name }}</h1>
            ...
        </template>
    

We need to do this because our module’s state is now scoped under it’s name. There are certainly ways to get around having to type `user.user` and we’ll show you this in a minute.store

And now in the browser, we can see that things still work, except now we’re a little more organized.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370652903_2.jpg?alt=media&token=8b07ca34-77f0-46be-a077-862457a6cabb)

I noticed there’s one more place in our code we need to update with our new **User module.**

📃 **/src/views/EventCreate.vue**

        <script>
        ...
            createFreshEventObject() {
              const user = this.$store.state.user // <----
              const id = Math.floor(Math.random() * 10000000)
              ...
    

When referencing the state we need to set `user` to `user.user`, so that line needs to get updated to:

        const user = this.$store.state.user.user // <----
    

* * *

🏗️ Creating an Event Module
----------------------------

Next, I’m going to move all of our event State, Mutations, Actions, and Getters into its own **event.js** module. It’s mostly one big copy paste job, which ends up looking like this:

📃 **/src/store/modules/event.js**

        import EventService from '@/services/EventService.js'
        
        export const state = {
          events: [],
          eventsTotal: 0,
          event: {}
        }
        export const mutations = {
          ADD_EVENT(state, event) {
            state.events.push(event)
          },
          SET_EVENTS(state, events) {
            state.events = events
          },
          SET_EVENTS_TOTAL(state, eventsTotal) {
            state.eventsTotal = eventsTotal
          },
          SET_EVENT(state, event) {
            state.event = event
          }
        }
        export const actions = {
          createEvent({ commit }, event) {
            return EventService.postEvent(event).then(() => {
              commit('ADD_EVENT', event)
            })
          },
          fetchEvents({ commit }, { perPage, page }) {
            EventService.getEvents(perPage, page)
              .then(response => {
                commit('SET_EVENTS_TOTAL', parseInt(response.headers['x-total-count']))
                commit('SET_EVENTS', response.data)
              })
              .catch(error => {
                console.log('There was an error:', error.response)
              })
          },
          fetchEvent({ commit, getters }, id) {
            var event = getters.getEventById(id)
            if (event) {
              commit('SET_EVENT', event)
            } else {
              EventService.getEvent(id)
                .then(response => {
                  commit('SET_EVENT', response.data)
                })
                .catch(error => {
                  console.log('There was an error:', error.response)
                })
            }
          }
        }
        export const getters = {
          getEventById: state => id => {
            return state.events.find(event => event.id === id)
          }
        }
    

The only new thing to notice here is that I brought over `import EventService from '@/services/EventService.js'` into this file, and I left the state objects the same, unlike how I changed things earlier in our `user.js` by removing the `user` property name, since we have more than one state property in this module. Now I need to use this module inside our **store.js**:

📃 **/src/store/store.js**

        import Vue from 'vue'
        import Vuex from 'vuex'
        import * as user from '@/store/modules/user.js'
        import * as event from '@/store/modules/event.js'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
          modules: {
            user,
            event
          },
          state: {
            categories: [ ... ]
          }
        })
    

Now if I look in the browser, nothing would be working yet. Our `events`, `eventTotal`, and `event` State must now all be accessed by `event.events`, `event.eventTotal`, and `event.event`. So we need to make two more file changes.

First, in **EventList:**

📃 **/src/views/EventList.vue**

        <template>
          <div>
            <h1>Events for {{ user.name }}</h1>
            <EventCard v-for="event in event.events" :key="event.id" :event="event"/>
            ...
        </template>
        <script>
            ...
            hasNextPage() {
              return this.event.eventsTotal > this.page * this.perPage
            },
            ...mapState(['event', 'user'])
          }
        }
        </script>
    

As you can see above, we made three changes, and the first was the one at the bottom of the file where we changed our `mapState` to just access `event` (this is mapping to our module name which is `event`). Then we just had to make sure to use `event.` when we accessed parts of the state.

In our **EventShow.vue** we’re using our `event` object all over the place, so we’ll solve this problem another way, instead of writing `event.event.time` and so on. There’s another way to use the `mapState` helper, which we covered in our [S](https://www.vuemastery.com/courses/real-world-vue-js/vuex-state-getters)[tate and](https://www.vuemastery.com/courses/real-world-vue-js/vuex-state-getters) [G](https://www.vuemastery.com/courses/real-world-vue-js/vuex-state-getters)[etter’s lesson](https://www.vuemastery.com/courses/real-world-vue-js/vuex-state-getters).

We’ll change our `computed` from `computed: mapState(['event'])` to:

📃 **/src/views/EventShow.vue**

          ...
          computed: mapState({
            event: state => state.event.event
          })
    

Here we’re mapping Component computed property called `event` to the `event` state in our `event` module.

Now everything works again as expected, and when we write `event.time` in our **EventShow.vue** file, it’s mapped to `event.event`.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370657406_3.gif?alt=media&token=2bad4c33-8747-4f30-8e45-76027dbd4585)

* * *

Alternate Syntax for Modules
----------------------------

Lastly, I want to mention that there’s another common module syntax you’ll likely experience in the wild. Instead of importing a module like this:

📃 **/src/store/store.js**

        import * as event from '@/store/modules/event.js'
        ...
    

and then having a module file which looks like this:

📃 **/src/store/modules/event.js**

        import EventService from '@/services/EventService.js'
        
        export const state = { ... }
        export const mutations = { ... }
        export const actions = { ... }
        export const getters = { ... }
    

You also might see this same module coded up as a single object rather than 5 constants.

📃 **/src/store/modules/event.js**

        import EventService from '@/services/EventService.js'
        
        export default {
          state: { ... },
          mutations: { ... },
          actions: { ... },
          getters: { ... }
        }
    

Which then is imported by doing:

📃 **/src/store/store.js**

        import event from '@/store/modules/event.js'
        ...
    

Both syntaxes are correct, and the reason the first might be preferable is that it’s easier to create private variables and methods. However, both are correct to use.

* * *

Accessing State in Other Modules
--------------------------------

Back in our first lesson on **Actions & Mutations**, we created an event Action, which looked like this and is now in our **/store/modules/event.js** file.

        ...
        export const actions = {
          createEvent({ commit }, event) {
            return EventService.postEvent(event).then(() => {
              commit('ADD_EVENT', event)
            })
          },
    

In the future we may want to access the current user from inside this Action. How could we do this? Well, if we weren’t in a module we could use the context object to access the State like this:

            createEvent({ commit, state }, event) {
                
              console.log('User creating Event is ' + state.user.user.name)
              
              return EventService.postEvent(event).then(() => {
                commit('ADD_EVENT', event)
              })
            },
    

However, since we’re using a module, **this won’t work**. The `state` object here is only our local module’s state. So, to get access to our user’s name, we’d do the following using `rootState` which, as it sounds, gives me access to the root of my Vuex state.

            createEvent({ commit, rootState }, event) {
            
              console.log('User creating Event is ' + rootState.user.user.name)
              
              return EventService.postEvent(event).then(() => {
                commit('ADD_EVENT', event)
              })
            },
    

Notice how with `rootState.user.user.name` I am accessing the root state, which gives me access to the **user** module, and then I can ask for the `name` State from the **user** module.

I can access `rootGetters` in the same way if there are Getters I want to call that are located in a different module.

Accessing another Module’s Actions
----------------------------------

It’s also common to call another module’s actions from inside this action. To do this we simply send in `dispatch` from the context object and call that action’s name.

            createEvent({ commit, dispatch, rootState }, event) {
            
              console.log('User creating Event is ' + rootState.user.user.name)
              
              dispatch('actionToCall')
              
              return EventService.postEvent(event).then(() => {
                commit('ADD_EVENT', event)
              })
            },
    

Yup, we don’t need to mention what module `actionToCall` is in. This is because by default all our actions, mutations, and getters are located in the Global NameSpace.

* * *

Understanding the Global NameSpace
----------------------------------

I’ll say that again: **Actions, Mutations, and Getters (even inside modules) are all registered under the global namespace.** This means that no matter where they’re declared, they’re called without their module name. So if you look at the following code, you’ll notice there’s no mention of any modules:

        this.$store.dispatch('someAction')
        this.$store.getters.filteredList
    

This is on purpose so that multiple modules can react to the same Mutation/Action type. Yup, you might find situations where you have two different modules, one for purchasing and one for logging, and they both want to listen for the “completePurchase” action. The purchasing module would actually do the purchase, and the logging module would log that the purchase took place.

⚠️ The downfall of this implementation is that we could end up with naming collisions. You might want to ensure your action, mutation, and getters never conflict. This is why you might want to turn namespacing on.

* * *

☑️ NameSpacing our Modules
--------------------------

If you want your modules to be more self-contained, reusable, and perhaps avoid accidentally having two modules that are the same name, you could _namespace_ your modules.

Let’s namespace our **event.js** module.

📃 **/src/store/modules/event.js**

        import EventService from '@/services/EventService.js'
        
        export const namespaced = true
        
        export const state = {
          events: [],
            ...
    

With this one line of configuration, our Getters, Mutations, and Actions now must be addressed using this namespace. So in our **EventList.vue:**

        this.$store.dispatch('fetchEvents', { ... })
    

becomes

        this.$store.dispatch('event/fetchEvents', { ... })
    

and in our **EventCreate.vue**

        this.$store.dispatch('createEvent', this.event)
    

becomes:

        this.$store.dispatch('event/createEvent', this.event)
    

In our **EventShow.vue:**

        this.$store.dispatch('fetchEvent', this.id)
    

becomes:

        this.$store.dispatch('event/fetchEvent', this.id)
    

And that’s all there is to it.

* * *

Small Aside about mapActions
----------------------------

We haven’t mentioned the `mapActions` helper yet in this tutorial, but it allows you to map component methods to `store.dispatch` calls. So instead of:

📃 **/src/views/EventShow.vue**

        import { mapState } from 'vuex'
        
        export default {
          props: ['id'],
          created() {
            this.$store.dispatch('event/fetchEvent', this.id)
          },
          computed: mapState({
            event: state => state.event.event
          })
        }
    

We can write:

        import { mapState, mapActions } from 'vuex'
        
        export default {
          props: ['id'],
          created() {
            this.fetchEvent(this.id)
          },
          computed: mapState({
            event: state => state.event.event
          }),
          methods: mapActions('event', ['fetchEvent'])
        }
    

Notice how we imported `mapActions`, how we use the helper with the `methods:` property, and how it allows us to simplify our component code. The first argument to `mapActions` here is the namespace, and the second is an array of methods we want our component to have an alias to.

* * *

Accessing NameSpaced Getters
----------------------------

Although we don’t access any of our getters from our current code, in our state & getter lesson we had the following code:

        computed: {
          getEventById() {
             return this.$store.getters.getEventById
          }
        }  
    

Which we simplified using `mapGetters`:

        computed: mapGetters(['getEventById'])
    

If we wanted to access our namespaced getter method inside our event module, we could do:

        computed: {
          getEventById() {
            return this.$store.getters['event/getEventById']
          }
        }
    

Which could be shortened to:

        computed: mapGetters('event', ['getEventById'])
    

* * *

Does any of this code change with NameSpaced Modules?
-----------------------------------------------------

Remember all this code from earlier?

            createEvent({ commit, dispatch, rootState }, event) {
            
              console.log('User creating Event is ' + rootState.user.user.name)
              
              dispatch('actionToCall')
              
              return EventService.postEvent(event).then(() => {
                commit('ADD_EVENT', event)
              })
            },
    

See the state, action, and mutation getting called? The question is, sssuming all of our modules are NameSpaced, does any of this code need to change?

`rootState.user.user.name` is correct, nothing that needs to change there.

`dispatch('actionToCall')` Only needs to change if the action it’s trying to call is not inside this module.

`commit('ADD_EVENT', event)` Only needs to change if the mutation it’s trying to is not inside this module, and frankly this would not be a best practice. It’s a best practice to never call mutations in other modules, and to only allow mutations to be called by actions in the same module. So I’m not going to show you how to call a mutation in another module, you should never do it.

* * *

How do I call an Action inside of an Action?
--------------------------------------------

There may be times where you need to call an action inside of another action. Nothing wrong with this, it’s pretty simple. If your action is inside of your current module (or you’re not using namespaced modules) you can just do:

📃 /src/store/modules/event.js

        ...
         actions: {
            createEvent({ commit, dispatch }, event) {
              ...
              dispatch('actionToCall')
            }
        ...
    

Notice how I’m including `dispatch` from my object context, and simply calling the action sending in a payload (which is always optional).

* * *

What if the action I want to call is in another module which is namespaced?
---------------------------------------------------------------------------

If you want to call an action inside another namespaced module, you’ll need to use the action’s module name, provide a `payload` as the second argument (null if there is none), and pass `{ root: true }` as the third argument, like so:

📃 **/src/store/modules/event.js**

        ...
         actions: {
            createEvent({ commit, dispatch }, event) {
              ...
              dispatch('moduleName/actionToCall', null, { root: true })
            }
        ...
    

* * *

Let’s ReVue
-----------

In this lesson we learned all about organizing our Vuex code to keep it more scalable. First, we reorganized our Vuex code into separate modules and refactored our code to use those modules. Then we talked about Vue NameSpacing, which allows us to further encapsulate our actions, mutations, and getters. Thanks for reading!

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson14-modules-start)
    
*   [Ending Code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson14-modules-finish)
    
*   [Official Module Docs](https://vuex.vuejs.org/guide/modules.html)
