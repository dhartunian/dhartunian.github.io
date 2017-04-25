
# Give Vue.js a shot if you’re adding interactivity to a traditional web-app

## A few weeks ago I was working on a client project that involves a sizable redesign of their entire site. The site’s a traditional server…

## Give Vue.js a shot if you’re adding interactivity to a traditional web-app

A few weeks ago I was working on a client project that involves a sizable redesign of their entire site. The site’s a traditional server-based web application: An HTTP server serves up HTML generated from static templates filled-in with variables retrieved from a database.

The specific ticket I had picked up involved adding a simple dropdown on a magazine subscription page:

*When a user selects the “International” shipping category, instead of showing a freeform text box in the shipping address form, show a dropdown of countries*

Sounds straightforward, right? Changes to a legacy application are rarely straightforward.

The problem I ran into was that the subscription page already had a few hundred lines of jQuery managing all sorts of little things (remove the shipping address form if the subscription is a gift, toggle between states and provinces if the user selects US vs Canada shipping, etc.). Somewhere in there I needed to add a listener to the shipping category and tell it to swap some divs out when the selection changed.

This was when I got nervous.

I’m not 100% comfortable with jQuery to begin with (I’m a late bloomer with Javascript). There was a decent amount of code that was concatenating strings together to figure out which div ids to target for changes. That made getting a working model of those few hundred lines of code in my head surprisingly hard.

I have previously wanted to introduce React to our legacy projects to help with the growing frontend complexity but have resisted because of some pretty high barriers to entry for a traditional webapp:

* React “owns” the rendering of its components so, if you’re not using server-side React rendering, you need your backend to deliver an empty div, and let React put its own data inside once the page loads.

* To pass data to React from the backend, you need to either render templated JSON (yuck) or use a JSON endpoint (we don’t have any!)

I remembered hearing before that Vue.js was a framework that was easier for newcomers to get started with than React and I got curious.

I spent an afternoon getting Vue hooked up in our application and seeing if it could solve our problems and in around 4 hours I had a working replacement for our javascript code that was many times easier to understand and extend.

How was this possible and why did it work?

## Straightforward integration with server-rendered HTML

Vue is able to integrate with existing HTML really easily (please tell me if React can do this too!). I can have a div like this:

    <div id="my-component">
      <form blahblah>
        ...
      </form>
    </div>

And hook up to it with Vue like this:

    new Vue({
      el: '#my-component'
      ...
    })

This is the magic! The div can already exist in the DOM, drawn by your backend. Vue will just read it in, take over its rendering, and keep everything that’s already in there as-is! It doesn’t matter who serves your HTML, you can keep your templates where they are and layer in interactivity separately.

If I have data to write to the DOM that gets updated after the page loads, I can use Vue’s simple templating like this

    <div id="my-component">
      <div class="container>
        {{ my_variable_or_method_in_vue }}
      </div>
    </div>

And in Vue:

    new Vue({
      el: '#my-component'
      data: { 
        my_variable_or_method_in_vue: 2343 
      }
    })

If anything changes the value of that data in my Vue component, the DOM will automatically get updated! Complex to implement, I’m sure, but simple to reason about for me, the user.

## Convenient helpers for commonly desired functionality

The basic task of flipping divs on and off based on some variable is made really easy with Vue. The following div will only render if our Vue component’s my_flag variable is true.

    <div v-if="my_flag">
     flag specific secret info
    </div>

Want to keep the div but toggle its display CSS property instead?

    <div v-show="my_flag">
     flag specific secret info that MUST stay in the DOM
    </div>

From what I understand, this style comes from Angular, which I’ve never used. It’s quite refreshing to replace code that’s just grabbing divs by their collars all around the page with something tied to a boolean, and can easily be reasoned about.

## Every time I ran into a stumbling block, the functionality I needed existed, was easy to use, and wonderfully documented

I made my first Vue app as a client project, just by reading their [getting started docs](http://vuejs.org/v2/guide/) and trying things out. They have a wonderful [Chrome dev tools plugin](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd) that helps with debugging (just like React!)

Now, some of this is because I’m familiar with frameworks like React and have some intuition for how frameworks like this work, but I found very few surprises with Vue. Most features were very narrowly scoped which made it easy to decide whether they would solve my problems or not.

One problem I ran into, for example, was with how the magazine subscription page communicated information about prices to the front-end. When the user selects different subscription types, we show a “subtotal” on the screen (which also reflects their coupon code!). Since our app never used JSON for anything, this information was coded right into the form using attributes like data-value="3999" for a subscription choice that costs $39.99, for instance. Our legacy javascript would read these values out, apply coupon codes, and then sum them up and dump them into a div for the user to see.

Since with Vue, all our data should flow from the component, we need to somehow load the values up into the component’s data fields. We can’t render templated javascript, so we need to read these values from the DOM as well. Luckily, Vue provides a way to connect to arbitrary DOM nodes in the component using refs (just like React!) and provides a function hook mounted which you can use to do a one-time load of the data from the DOM after the component is loaded.

Once these variables are loaded into the component, the rest of the form and page can depend on them directly. Changing a subscription option changes your selected price point, and the computed price can use these variables instead of having to depend on the DOM directly.

Here’s an example mixing HTML and Javascript together:

**HTML**

The {{ subscription_price }} is a Vue template that renders using data or functions defined in the component. These values will automatically update if they or their dependencies change.

    <div id='my-component'>
      <form>
        <input data-amount="3999" 
               type="radio" 
               ref="print_individual" /> Print
        <input data-amount="1999" 
               type="radio" 
               ref="digital_individual" /> Digital
        <div>Subtotal: {{ subscription_price }}</div>
        <button type="submit">Submit</button>
      </form>
    </div>

**Vue component**

    new Vue({
      el: '#my-component',
      data: {
        prices: {
          digital_individual: 0,
          print_individual: 0
        },
        ...
      },
      computed: {
       subscription_price: function() {
         return this.prices[this.subscription_selection]
        }
      },
      mounted: function() {
        this.prices.print_individual = 
            parseInt(this.$refs.print_individual.dataset.amount);
        this.prices.digital_individual = 
            parseInt(this.$refs.digital_individual.dataset.amount);
      }
    })

* data is where your component state is stored

* computed is data that requires executing a function to determine its value

* mounted as mentioned earlier this is a special function that is called once as soon as the component is loaded into the DOM

Hopefully this gives you some idea of how the Vue programming model works. Methods and data can be used using a simple templating language in the frontend to dump changing data into containers without worrying too much about *when* exactly it’s going to change or *who* is going to change it.

Here’s a [JSFiddle with the full example](https://jsfiddle.net/bpeo20k1/) and all code so you can see how it all works.

## Conclusion

None of the functionality that Vue offers exceeds that of React or other frameworks I’ve seen. But I really appreciated how the **design** and **documentation** of this functionality was so clearly made to help with the exact challenges I was facing as I tried to refactor our legacy javacsript code to be a bit better structured while adding new functionality.

Give it a shot if you’re having some of the same problems! I would love to hear about your experience!

## Resources
[**Introduction - Vue.js**
*Vue.js - Intuitive, Fast and Composable MVVM for building interactive interfaces.*vuejs.org](https://vuejs.org/v2/guide/)
[**Create a Basic Component using Vue.js - vue Video Tutorial #free**
*Vue.js is a "progressive framework for building user interfaces." The core of Vue is focused on the view layer only. It…*egghead.io](https://egghead.io/lessons/vue-create-a-basic-component-using-vue-js)
