Mastering Vuex Orientation
==========================

Welcome back to Mastering Vuex. In this lesson, I’ll be orienting you to the example application we’ll be using throughout this course.

Prerequisites for this course include knowledge of:

*   Vue CLI
*   Vue Router
*   Single File .vue Components
*   API Calls with Axios

If you’ve been following along with our Real World Vue course, you can probably skip ahead to the next lesson, unless you want a refresher on what we built in that course, or you want to get the example app running so you can code along with us during the course.

But if you’re just joining us, join me as we get our example app running and take a tour of how it’s working.

* * *

Downloading the App
-------------------

The starting code for our app is located on [github here](https://github.com/Code-Pop/real-world-vue/releases/tag/lesson11-vuex-start). Please download it to your computer.

* * *

Getting the app up and running
------------------------------

If you navigate to the project from your terminal, you can run `npm install` to install all of the project’s dependencies.

Since our app will be making API calls, we’ll be using [json-server](https://github.com/typicode/json-server). This is a full fake REST API, which allows us to perform API calls that pull from a mock database. You’ll need to install the library if you haven’t already. To do so, run this command in your terminal: `npm install -g json-server`. We then need to run the command `json-server --watch db.json`, which turns on json-server and tells it to _watch_ our db.json file, which is our mock database.

Now, to get our app running live, we’ll run the command: `npm run serve`. Our terminal will let us know which localhost port our app is running on.

* * *

Exploring the app in the browser
--------------------------------

Once we pull up that localhost in our browser, we can see our app.

On the main page, we’re displaying a list of events that we’re pulling in with our API. When I click on an event, we’re taken to the `event-show` page, which displays the full details of that event. We’re using Vue Router for our site navigation, which also allows us to navigate between pages.

Now that we’ve seen the app running live, let’s look the project itself.

* * *

App Tour
--------

We created the app using the Vue CLI, which gave us this directory structure.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578379462404_0.png?alt=media&token=c80a176d-8329-4080-9e10-e2ed278ab47d)

In our **views** folder, we have three components, which are loaded when we navigate to the route they live at.

The **EventCreate** component currently only has a simple template.

📃 **/src/iews/EventCreate.vue**

        <template>
          <h1>Create Event</h1>
        </template>
    

The **EventList** is much more interesting.

📃 **/src/views/EventList.vue**

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
    

When this component is created, we are making an API call to get our events with `EventService.getEvents()`. Then we’re setting our component’s `events` equal to that API call’s response. We’re also catching and logging any errors to the console.

📃 **/src/views/EventList.vue**

        <template>
          <div>
            <h1>Events Listing</h1>
            <EventCard v-for="event in events" :key="event.id" :event="event"/>
          </div>
        </template>
    

In our template, we’re using `v-for` to create an **EventCard** component for each of our `events`, and passing in the event as a prop so **EventCard** can use it.

Let’s take a look at the **EventCard** component in our **components** folder.

📃 **/src/components/EventCard.vue**

        <template>
          <router-link class="event-link" :to="{ name: 'event-show', params: { id: event.id } }">
            <div class="event-card -shadow">
              <span class="eyebrow">@{{ event.time }} on {{ event.date }}</span>
              <h4 class="title">{{ event.title }}</h4>
              <BaseIcon name="users">{{ event.attendees.length }} attending</BaseIcon>
            </div>
          </router-link>
        </template>
        
        <script>
        export default {
          props: {
            event: Object
          }
        }
        </script>
        
        <style scoped>
        ...
        </style>
    

It’s receiving the `event` object as a prop, and displaying some of its details in the template.

Notice how it’s also using the **BaseIcon** component.

📃 **/src/components/BaseIcon.vue**

        <template>
            <div class="icon-wrapper">
              <svg class='icon' :width="width" :height="height">
                <use v-bind="{'xlink:href':'/feather-sprite.svg#' + name}"/>
              </svg>
              <slot></slot>
            </div>
        </template>
            
        <script>
        export default {
          name: 'Icon',
          props: {
            name: String,
            width: {
              type: [Number, String],
              default: 24
            },
            height: {
              type: [Number, String],
              default: 24
            }
          }
        }
        </script>
            
        <style scoped>
        ...
        </style>
    

This is a component that accepts a `name`, `width` and `height` prop, and draws an svg icon accordingly. It uses the **feather-sprite.svg** file (located in our **public** folder) as its library of icons that it draws from. To learn more about this component and how it was globally registered within our **main.js** file, watch our lessons on [Global Components](https://www.vuemastery.com/courses/real-world-vue-js/global-components) and [Slots](https://www.vuemastery.com/courses/real-world-vue-js/slots).

* * *

Understanding our API Calls
---------------------------

Let’s take a closer look at the API call within **EventList**. If you need a refresher on this topic, watch our lesson on [API Calls with Axios](https://www.vuemastery.com/courses/real-world-vue-js/API-calls-with-Axios).

📃 **/src/views/EventList.vue**

        import EventService from '@/services/EventService.js'
        ...
          created() {
            EventService.getEvents()
              .then(response => {
                this.events = response.data
              })
              .catch(error => {
                console.log('There was an error:', error.response)
              })
          }
        ...
    

Notice how it’s using `EventService` to call the `getEvents` method. We imported this file above, from our **services** folder. Let’s explore that file.

📃 **/src/services/EventService.js**

        import axios from 'axios'
        
        const apiClient = axios.create({
          baseURL: `http://localhost:3000`,
          withCredentials: false, // This is the default
          headers: {
            Accept: 'application/json',
            'Content-Type': 'application/json'
          }
        })
        
        export default {
          getEvents() {
            return apiClient.get('/events')
          },
          getEvent(id) {
            return apiClient.get('/events/' + id)
          }
        }
    

Here, we’re importing the Axios library. If you haven’t yet, you can install Axios by running this command in your terminal: `npm install axios`

We’re then creating a single Axios instance with `apiClient`. When we create it, we’re giving it a `baseURL`, and setting some default configurations.

As noted earlier, json-server is the **db.json** file as our mock “database”, which contains all of our events. So when we visit `localhost:3000/events`, we can see all of the events that live in the **db.json** file. And because json-server is a full fake REST API, we can GET events from it and POST events to it, etc.

At the bottom of the file, we’re exporting a couple methods that use our `apiClient` to get all of the events, or just one event (by it’s `id`). We already saw how the **EventList** component calls `getEvents` in its `created` hook, but where are we using `getEvent`?

Let’s take a look in our final **view** component: **EventShow**.

📃 **/src/views/EventShow.vue**

        <script>
        import EventService from '@/services/EventService.js'
        
        export default {
          props: ['id'],
          data() {
            return {
              event: {}
            }
          },
          created() {
            EventService.getEvent(this.id)
              .then(response => {
                this.event = response.data
              })
              .catch(error => {
                console.log('There was an error:', error.response)
              })
          }
        }
        </script>
    

When it’s `created`, it makes the `getEvent` API call, passing in its prop (`this.id`), which is used to get the event by its `id`. The component is then setting its `event` data equal to the event that was retrieved (or catching and logging an error instead).

📃 **/src/views/EventShow.vue**

        <template>
          <div>
            <div class="event-header">
              <span class="eyebrow">@{{ event.time }} on {{ event.date }}</span>
              <h1 class="title">{{ event.title }}</h1>
              <h5>Organized by {{ event.organizer }}</h5>
              <h5>Category: {{ event.category }}</h5>
            </div>
            <BaseIcon name="map"><h2>Location</h2></BaseIcon>
            <address>{{ event.location }}</address>
            <h2>Event details</h2>
            <p>{{ event.description }}</p>
            <h2>Attendees
              <span class="badge -fill-gradient">{{ event.attendees ? event.attendees.length : 0 }}</span>
            </h2>
            <ul class="list-group">
              <li v-for="(attendee, index) in event.attendees" :key="index" class="list-item">
                <b>{{ attendee.name }}</b>
              </li>
            </ul>
          </div>
        </template>
    

In the template, we’re printing out details from the `event` object we got back from the `getEvent` API call.

But where did this component get its `id` prop from? To understand that, let’s head over to our **router.js** file and explore how we’re using Vue Router.

* * *

Understanding Our Routing
-------------------------

If any of this feels confusing, please watch our lessons on [Vue Router Basics](https://www.vuemastery.com/courses/real-world-vue-js/vue-router) and [Dynamic Routing & History Mode](https://www.vuemastery.com/courses/real-world-vue-js/dynamic-routing-history-mode).

📃 **/src/router.js**

        import Vue from 'vue'
        import Router from 'vue-router'
        import EventCreate from './views/EventCreate.vue'
        import EventList from './views/EventList.vue'
        import EventShow from './views/EventShow.vue'
        
        Vue.use(Router)
        
        export default new Router({
          mode: 'history',
          routes: [
            {
              path: '/',
              name: 'event-list',
              component: EventList
            },
            {
              path: '/event/create',
              name: 'event-create',
              component: EventCreate
            },
            {
              path: '/event/:id',
              name: 'event-show',
              component: EventShow,
              props: true
            }
          ]
        })
    

At the top, we’re importing all of our **view** components, so we can use them in our `routes`. But first we’re telling Vue to use the **Router**, which we imported above as well: `Vue.use(Router)`.

We’re then exporting a `new Router` instance, which contains an array of `routes` objects.

        {
          path: '/',
          name: 'event-list',
          component: EventList
        },
    

Our root route (`'``/``'`) loads our **EventList** component.

        {
          path: '/event/create',
          name: 'event-create',
          component: EventCreate
        },
    

When we navigate to `'/event/create``'`, our **EventCreate** component is loaded.

        {
          path: '/event/:id',
          name: 'event-show',
          component: EventShow,
          props: true
        }
    

And when we navigate to `'``/event/``'` + some `'``:id``'`, our app loads the **EventShow** component. Because we have `props: true`, that means the dynamic segment (`:id`) will be passed into the **EventShow** component as a prop. Which brings us back to our original question: From where did **EventShow** get the `id` prop it uses when it makes the `getEvent` API call? It took it from the URL params. So if our URL was `/event/1`, **EventShow**’s `id` prop would be `1`.

* * *

But wait, there’s more…
-----------------------

We have one more component, **NavBar**, which contains router-links to our other routes: 📃 **/src/components/NavBar.vue**

        <template>
          <div id="nav" class="nav">
            <router-link to="/" class="brand">Real World Events</router-link>
            <nav>
              <router-link :to="{ name: 'event-list' }">List</router-link> |
              <router-link :to="{ name: 'event-create' }">Create</router-link>
            </nav>
          </div>
        </template>
        ...
    

And speaking of router-links, notice how there’s one in our **EventCard** component, too.

📃 **/src/components/EventCard.vue**

        <router-link class="event-link" :to="{ name: 'event-show', params: { id: event.id } }">
        ... 
        </router-link>
    

This allows us to click on the **EventCard** and have us routed to: `'/event/:id``'`, where `id` is the `event.id` from the **EventCard** we just clicked on. For example: `'/event/1``'`. This routes us to the **EventShow** component, which pulls the `id` of the event to show from the URL.

* * *

Let’s ReVue
-----------

As you’ve seen, our example app is using Vue Router to create a single page application where we can route between views, which trigger API calls when those components are loaded. If any of what I covered was confusing, you’ll want to watch the lessons in our Real World Vue course before diving into Vuex.
