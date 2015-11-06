---
layout: post
---
<h2 class="post-title">Batch Updating in React</h2>
<p class="post-meta">Friday, November 6th 2015</p>
<p>
This post talks about how <a href='https://facebook.github.io/react/index.html'>React</a> components perform updates for different types of events and how
we can get these updates under our control to behave in a way that is ideal for us.
</p>

<h3>SyntheticEvents</h3>
<p>
React components will batch their updates inside of <a href='https://facebook.github.io/react/docs/events.html'>Events</a>.
This means that when a component's <code>setState</code> function is called
multiple times inside of an <code>OnClick</code> event, the component will only call <code>render</code> once (if <code>shouldComponentUpdate</code> returns <code>true</code>).
This is exactly the behavior I want to have because I want to make sure when I interact with a store's data, it and all the other stores are in a complete, stable, and ready state.
</p>
<h3>AJAX and timeout events</h3>
<p>
Unfortunately, AJAX and window.timeout events are not batched like SyntheticEvents.
It is common for my code to update multiple stores from a single AJAX response.
Bugs crop up by using the default behavior of re-rendering on each call to <code>setState</code>.
Luckily, there is a pattern to batch these updates and it is pretty simple to do to.
I batch the updates by using a single store that sets a <code>isLoading</code> boolean
at the beginning of the AJAX event call and doesn't clear it until all the stores have finished their updates.
The main react component will use this
<code>isLoading</code> boolean inside of its <code>shouldComponentUpdate</code> function. The logic will say to not update if the current state and the next
state is loading.
This <code>isLoading</code> acts as a lock and batches all the stores updates into one single render call.
This holds the invariant in our components that stores will be in a complete and stable state during render calls.
</p>
<h4>UI Lock Example</h4>
<p>
Here is a partially implemented version of a loading lock that performs batch updates for AJAX events.
</p>
{% highlight JavaScript %}
AppDispatcher.dispatch({
  type: 'AJAX_ACTION',
});

_isLoading = false;
var AppStore = {
  isLoading() {
    return _isLoading;
  },
};

AppStore.dispatchToken = AppDispatcher.register(action => {
  switch (action.type) {
    case 'AJAX_ACTION':
      _isLoading = true;
      AppStore.emitChange();
      break;

    case 'AJAX_ACTION_SUCCESS':
      AppDispatcher.waitFor([OtherStore1.dispatchToken, OtherStore2.dispatchToken]);
      _isLoading = false;
      AppStore.emitChange();
      break;
  }
});

var OtherStore1 = {
  // Some stuff...
};

OtherStore1.dispatchToken = AppDispatcher.register(action => {
  switch (action.type) {
    case 'AJAX_ACTION_SUCCESS':
      // Update state
      OtherStore1.emitChange();
      break;
  }
});

var OtherStore2 = {
  // Some stuff...
};

OtherStore2.dispatchToken = AppDispatcher.register(action => {
  switch (action.type) {
    case 'AJAX_ACTION_SUCCESS':
      // Update state
      OtherStore2.emitChange();
      break;
  }
});

function getState() {
  return {
    storeState1: OtherStore1.getAll(),
    storeState2: OtherStore2.getAll(),
    isLoading: AppStore.isLoading(),
  };
}

var App = Reat.createClass({
  shouldComponentUpdate(nextProps, nextState) {
    var update = true;
    if (this.state.isLoading && nextState.isLoading) {
      update = false;
    }

    return update;
  },

  render() {
    // Store are in a stable complete state invariant holds! \o/
  },
});
{% endhighlight %}
