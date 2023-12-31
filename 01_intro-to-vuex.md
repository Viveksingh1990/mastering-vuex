Intro to Vuex
=============

### A note about Vuex 4

Vuex 4 was recently released, which is the latest version of Vuex made to be compatible with Vue 3. Almost all 4 APIs have remained the same from Vuex 3, which this course teaches. However, there are a few breaking changes that you can [learn about here](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html#breaking-changes).

⚠️ The Case for State Management
--------------------------------

Managing state in an application full of components can be difficult. Facebook discovered this the hard way and created the Flux pattern, which is what Vuex is based upon. Vuex is Vue’s own state management pattern and library. In this lesson, we’ll look at why an application might need Vuex, and how it can enhance your app.

When we talk about state, we mean the data that your components depend on and render. Things like blog posts, to-do items, and so on. Without Vuex, as your app grows, each Vue component might have its own version of state.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371882428_0.png?alt=media&token=bad49a29-5fff-4dcd-bf42-f5c8beec69b7)

But if one component changes its state, and a distant relative is also using that same state, we need to communicate that change. There’s the default way of communicating events up and passing props down to share data, but that can become overly complicated.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371882429_1.png?alt=media&token=2c411c9f-d5cf-4404-8009-00b73e24a622)

Instead, we can consolidate all of our state into one place. One location that contains the current state of our entire application. One single source of truth.

* * *

**A Single Source of Truth** This is what Vuex provides, and every component has direct access to this global State.

Just like the Vue instance’s data, this State is reactive. When one component updates the State, other components that are using that data get notified, automatically receiving the new value.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371889694_2.png?alt=media&token=f9cc01f0-4644-4867-92ad-17ccd6cc7a6e)

But just consolidating data into a single source of truth doesn’t fully solve the problems of managing state. What happens when many components alter the State in different ways, from different locations?

We need some standardization. Otherwise, changes to our State could be unpredictable and untraceable.

* * *

**A State Management Pattern** This is why Vuex provides a full state management pattern for a simple and standardized way to make state changes. And if you’re familiar with Vue, Vuex should look quite similar.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371892222_3.png?alt=media&token=911a39ae-99b2-4026-9132-71fae035ffc5)

Just as Vue provides a root Vue instance created with `new Vue`, Vuex offers a store created with `new Vuex.Store`.

While the Vue instance has a data property, the Vuex store has state. Both are reactive.

And while the instance has methods, which among other things can update data, the store has Actions, which can update the state.

And while the instance has computed properties, the store has getters, which allow us to access filtered, derived, or computed state.

Additionally, Vuex provides a way to _track_ state changes, with something called Mutations. We can use Actions to commit Mutations, and from the Vue DevTools, we can even trace back in time through a record of each mutation to the state.

* * *

Now let’s take a look at an example Vuex Store.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371895007_4.png?alt=media&token=27387520-216e-47da-99bd-41e72cebd7bc)

In our **State**, we have an isLoading property, along an array for todos.

Below that we have a **Mutation** to switch our isLoading state between true and false. And a Mutation to set our state with the todos that we’ll receive from an API call in our action below.

Our **Action** here has multiple steps. First, it’ll commit the Mutation to set the isLoading status to true. Then it’ll make an API call, and when the response returns, it will commit the Mutation to set the isLoading status to false. Finally it’ll commit the Mutation to set the state of our todos with the response from our API.

If we need the ability to only retrieve the todos that are labeled done, we can use a Getter for that, which will retrieve only the specific state that we want.

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371897321_5.png?alt=media&token=1a6cfcd6-5eb0-4d63-b991-f0dbd0c460ea)

Now let’s take a look at this in motion.

* * *

Vuex in Motion
--------------

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371900954_6.gif?alt=media&token=a92f02df-8800-4f4a-ac63-6690f9453e66)

* * *

Let’s ReVue
-----------

Hopefully you now understand why you might need Vuex and how it can help enhance your application by providing a single source of truth for your State, along with a common library of Actions, Mutations and Getters.

In the next lesson, we’ll start implementing Vuex into our example application.
