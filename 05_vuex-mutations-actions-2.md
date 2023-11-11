Mutations & Actions Pt. 2
=========================

In our [last lesson](https://www.vuemastery.com/courses/real-world-vue-js/vuex-mutations-actions-1) we learned how to start creating actions and mutations using Vuex with an API, which is something that happens a lot in a real world application. In this lesson, we’ll build out Vuex Mutations and Actions for our **EventList** & **EventShow** pages, and even implement some pagination.

* * *

🛑 Problem: Loading our EventList using Vuex
--------------------------------------------

In our lesson on [APIs with Axios](https://www.vuemastery.com/courses/real-world-vue-js/API-calls-with-Axios) we built out an **EventService**, which has a `getEvents` method to populate our events when the **EventList** component is created. That component currently looks like this:

📃 **/views/EventList.vue**

        <template>
          <div>
            <h1>Events Listing</h1>
            <EventCard v-for="event in events" :key="event.id" :event="event"/>
          </div>
        </template>
        
        <script>
        import EventCard from '@/components/EventCard.vue'
        import EventService from '@/services/EventService.js'
        
        export default {
          components: {
            EventCard
          },
          data() {
            return {
              events: []
            }
          },
          created() {
            EventService.getEvents()
              .then(response => {
                this.events = response.data
              })
              .catch(error => {
                console.log('There was an error:', error.response)
              })
          }
        }
        </script>
    

And in the browser it looks like this:

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370807779_0.jpg?alt=media&token=2b7873d3-7d04-4ae7-9492-d15fe15a0660)

We want this component to properly use Vuex to retrieve and display events.

☑️ Solution
-----------

The first step to making this component use Vuex is to create a new Mutation and an Action. Here’s a diagram showing what we want to happen from inside our **EventList** component.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370807780_1.jpg?alt=media&token=bb422eb3-ded9-4f90-93af-c383eeccf8c4)

We already have `events:[]` inside our Vuex State (as shown below), and we’re already importing the **EventService**, so the new code inside our **store.js** is:

📃 /**store.js**

          state: {
            ...
            events: []  // already exists
          }
          mutations: {
            ...
            SET_EVENTS(state, events) {
              state.events = events
            }
          },
          actions: {
        
            ...
            fetchEvents({ commit }) {
              EventService.getEvents()
                .then(response => {
                  commit('SET_EVENTS', response.data)
                })
                .catch(error => {
                  console.log('There was an error:', error.response)
                })
            }
          }
    

Notice our `SET_EVENTS` Mutation sets all the events, and our `fetchEvents` action simply calls our EventService and then calls our Mutation.

Back in our **EventList.vue** we’ll make a bunch of small changes so it looks like this:

📃 **/views/EventList.vue**

        <script>
        import EventCard from '@/components/EventCard.vue'
        import { mapState } from 'vuex'
        
        export default {
          components: {
            EventCard
          },
          created() {
            this.$store.dispatch('fetchEvents')
          },
          computed: mapState(['events'])
        }
        </script>
    

I imported the `mapState` helper, and removed the line that imported EventService. I also removed our data option, and our `created` lifecycle hook simply calls our new Action. If we look in our browser, we can see that it properly lists out our events and looks the same as above.

* * *

🛑 Problem: Pagination
----------------------

Often when showing lists of data in web apps, like events, we might have thousands of events and we probably shouldn’t fetch all of them at once (It could be huge, and cause the browser to slow). Instead, we need to paginate like Google search results. Since we’re building a real-world application, let’s try our hand at implementing pagination.

☑️ Solution
-----------

The first thing to notice is that our trusty `json-server` actually has built-in [API pagination](https://github.com/typicode/json-server#paginate). If we send it `_limit` as a parameter we can limit the number of items we show on a page, and `_page` will only give us the data on our particular page. Perfect!

So if we construct a URL like so: `/events?_limit=3&_page=2` our API will return 3 events per page, and will give us the events to list on page 2. Let’s first modify our `getEvents` method in our **EventService** to use these two parameters.

📃 /services/EventService.js

          getEvents(perPage, page) {
            return apiClient.get('/events?_limit=' + perPage + '&_page=' + page)
          },
    

Then inside Vuex, we need to modify our Action:

📃 **/store.js**

          actions: {
            ...
            fetchEvents({ commit }, { perPage, page }) {
              EventService.getEvents(perPage, page)
                .then(response => {
                  commit('SET_EVENTS', response.data)
                })
                .catch(error => {
                  console.log('There was an error:', error.response)
                })
            },
            ...
    

Notice in the second argument we’re using ES2015 argument destructuring to pull out `{ perPage, page }` . This is because the second argument with both mutations and actions is effectively a `payload`. The `payload` in both Actions and Mutations can be a single variable or a single object.

So, to call this from our **EventList.vue**, we’ll do the following:

📃 **/views/EventList.vue**

        ...
        <script>
        import EventCard from '@/components/EventCard.vue'
        import { mapState } from 'vuex'
        
        export default {
          components: {
            EventCard
          },
          created() {
            this.$store.dispatch('fetchEvents', {
              perPage: 3,  // <-- How many items to display per page
              page: this.page // <-- What page we're on    
            })
          },
          computed: {
            page() {  // What page we're currently on
              return parseInt(this.$route.query.page) || 1
            },
            ...mapState(['events'])
          }
        }
        </script>
    

Notice our new computed property `page()`. It looks into the URL to see if we have a page query parameter, otherwise it assumes we’re on the first page. So if our URL is `http://localhost:8080/?page=2` then `this.$route.query.page` is `2`. We call `parseInt` on the `page` query parameter to ensure it’s a number.

Notice our payload for the action is `{ perPage: 3, page: this.page }`.

Lastly, let’s add some new router links in this same file’s template.

📃 **/views/EventList.vue**

        <template>
          <div>
            <EventCard v-for="event in events" :key="event.id" :event="event"/>
            <template v-if="page != 1">
              <router-link :to="{ name: 'event-list', query: { page: page - 1 } }" rel="prev">Prev Page</router-link> | 
            </template>
            <router-link :to="{ name: 'event-list', query: { page: page + 1 } }">Next Page</router-link>
          </div>
        </template>
    

Notice we have the beginnings of some pagination going on. We added a bunch of events to our **db.json** \*\*\*\*file so pagination will be more fun. You can [download mine here](https://raw.githubusercontent.com/Code-Pop/real-world-vue/lesson13-mutations%26actions2-finish/db.json) if you want to follow along.

Let’s see if this works:

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370814995_2.gif?alt=media&token=18975b62-c441-42ef-bcae-58c6bfdeb255)

Looks like on the first page we see a limited list of events. Great, it’s kind of working. However, when we use the link to go to page two, nothing changes. But if we refresh, it does work.

* * *

🛑 Problem: The Component isn’t reloading
-----------------------------------------

What’s going on here, is that our router sees that we’re loading the same ‘event-list’ named route, so it doesn’t need to reload the component. This is like clicking a navigation link twice. When someone clicks on a navigation link twice and they’re already on that page, do we want it to reload the component? No. That’s what’s going on. `created()` is not getting called again when we go to the second page, because it’s not reloading the component.

Inevitably, you’ll run into this as a Vue developer: where you want to reload a component with a change of query parameters.

☑️ Solution: Updating our router view
-------------------------------------

There are two ways to fix this:

1.  Watch the `page` computed property to see if it changes (which it does when we change the page), and when it does, dispatch the `fetchEvent` action.
    
2.  Tell our router to reload components when the full URL changes, including the query parameters. We’ll do this. It’s super simple.
    

📃 /App.vue

        <template>
          <div id="app">
            <NavBar/>
            <router-view :key="$route.fullPath"/>
          </div>
        </template>
        ...
    

Now when we switch pages, everything works as expected.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370824356_3.gif?alt=media&token=1a3b7aa0-a813-4a10-85a7-56bb22b1ef9e)

* * *

🎓 Extra Credit
---------------

You’ll notice on our current app the “Next Page” link doesn’t disappear when we reach the last page, so we can continue to paginate onto blank pages, which obviously we don’t want to happen. For extra credit, you might want to try to implement a solution for this.

There are many ways we could cause this link to disappear. It would be easy if we knew how many events we have total. **json-server** is actually giving us this data on each event request listing as a header. We can see this by looking at the Chrome DevTools → Network tab, and inspecting one of our API calls. We’ll see:

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370824357_4.png?alt=media&token=a62f3f66-c8a2-470f-9b15-5d888b0d2b5e)

There it is! Now inside our Vuex `fetchEvents` action we could print this with:

📃 **/store.js**

        ...
            fetchEvents({ commit }, { perPage, page }) {
              EventService.getEvents(perPage, page)
                .then(response => {
                  console.log('Total events are ' + response.headers['x-total-count'])
                  commit('SET_EVENTS', response.data)
                })
                ...
    

See that console.log line? From here you’d probably want to create `eventsTotal` in your Vuex State, create a mutation & action for it, call the mutation from `getEvents`, and access that State from the `EventList.vue` to see if `eventsTotal > (this.page * 3)`. If this is true, we have a next page. Get it?

Here’s the [starting code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson-13-extra-credit-start) if you want to give it a try, and [here’s a solution](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson-13-extra-credit-finish).

* * *

➡️ A Side Note on Caching
-------------------------

The way we’ve been coding this triggers an API call on every page. In some apps, you may always want to fetch the latest data, so this is exactly what you’d do. However, if you’re dealing with millions of users, you’ll likely implement some sort of caching strategy so you can keep your page snappy. There are many ways to do this, both outside and inside Vue.

* * *

🛑 Problem: Implementing the Show Event Page
--------------------------------------------

Now that we have pagination in place, when a user clicks on an event to get to the **ShowEvent** page, what do we do inside Vuex? Here’s what our page looks like at the moment:

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578370832726_5.gif?alt=media&token=64a58e24-5f94-40d3-8edd-8209e04a26f4)

☑️ Solution
-----------

Like before, we start with Vuex, this time adding a new object to our State called `event` to store the event that is currently being viewed. Then we’ll add a Mutation to set it, and an Action to call our API.

📃 **/store.js**

        export default new Vuex.Store({
          state: {
            ...
            event: {}
          },
          mutations: {
            SET_EVENT(state, event) {
              state.event = event
            }
          },
          actions: {
            ...
            fetchEvent({ commit }, id) {
              EventService.getEvent(id)
                .then(response => {
                  commit('SET_EVENT', response.data)
                })
                .catch(error => {
                  console.log('There was an error:', error.response)
                })
            }
          },
          ...
    

Then in our **EventShow** component, we’ll dispatch the `fetchEvent` Action, and send in the `id` as the payload.

📃 **/views/EventShow.vue**

        ...
        <script>
        import { mapState } from 'vuex'
        
        export default {
          props: ['id'],
          created() {
            this.$store.dispatch('fetchEvent', this.id)
          },
          computed: mapState(['event'])
        }
        </script>
    

And that’s all there is to it! Our **EventShow** page looks the same except now it’s using Vuex.

* * *

🛑 Problem: We’re Loading Data Twice
------------------------------------

We happen to be loading all the data we need to show an event on the **EventList** page. If we view the **EventList** page first, then the **EventShow** page (which many users will do), it seems wasteful to do another call to the API, when we already have the data needed in hand. How might we save our code an extra call?

☑️ Solution
-----------

When we get to the **ShowEvent** page, we need to check if we already have this particular event in our `events` State array. We can use a Getter we already have in our **store.js** for this. We can call it from inside our fetchEvent action in store.js, like so:

📃 **/store.js**

          ...
          actions: {
            ....
            fetchEvent({ commit, getters }, id) {  // Send in the getters
        
              var event = getters.getEventById(id) // See if we already have this event
        
              if (event) { // If we do, set the event
                commit('SET_EVENT', event)
              } else {  // If not, get it with the API.
                EventService.getEvent(id)
                  .then(response => {
                    commit('SET_EVENT', response.data)
                  })
                  .catch(error => {
                    console.log('There was an error:', error.response)
                  })
              }
            }
          },
          getters: { 
            getEventById: state => id => {
              return state.events.find(event => event.id === id)
            }
          }
    

As you can see, at the start of this action we call our `getters.getEventById(id)` with the `id` of the event we want to display. If it is found, we commit a Mutation, otherwise we go ahead and get that single event from the API.

Obviously, if we wanted to make sure our page always had the latest data, we might want to allow our event page to always trigger a new API request, even if we fetched this event in the **EventList** component.

* * *

⏪ Let’s ReVue
-------------

In this lesson we learned a bunch of ways to use Vuex in our code, specifically:

*   How to fetch a list of events and a single event with Vuex.
*   How to paginate with Vuex.
*   How to use query parameters on our router, and ensure our components are reloaded when they change.
*   How to optimize our Vuex State so we’re not reloading data twice.

In the next lesson we’ll learn about modules, to keep our Vuex code more organized.

### Lesson Resources

##### Source Code:

*   [Starting code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson13-mutations%26actions2-start)
    
*   [Ending code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson13-mutations%26actions2-finish)
    
*   [Bigger db.json file](https://raw.githubusercontent.com/Code-Pop/real-world-vue/lesson13-mutations%26actions2-finish/db.json)
    
*   [Extra credit starting code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson-13-extra-credit-start)
    
*   [Extra credit ending code](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson-13-extra-credit-finish)

