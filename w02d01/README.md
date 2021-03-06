# Composing Component

#### In this section:

- Pass Data
- Raise and Handle Events
- Multiple Components in sync
- Lifecycle Hooks


<h1 style="color:dodgerblue">Pass Data</h1>

## Passing data to components

First we create another component `counters.jsx` an d in this comp. we wanna rendere a **list** of counters.

Back to `/index.js` instead of `<CounterComponent />` we wanna import `<CountersComponent />`

```
import React, { Component } from 'react';
import CounterComponent from './counter'

class CountersComponent extends Component {
    state = {  }

    render() { 
        return (
            <div>

                <CounterComponent />
                <CounterComponent />
                <CounterComponent />
                <CounterComponent />

            </div>
        )
    }
}
 
export default CountersComponent;
```

Instead of hardcoding let's make an `array of counters`:


```
  state = {
    counters: [
      { id: 1, value:0 },
      { id: 2, value:0 },
      { id: 3, value:0 },
      { id: 4, value:0 },
    ]
  }
```

Just like this:

```
state = {
    counters: [
      { id: 1, value: 0 },
      { id: 2, value: 0 },
      { id: 3, value: 0 },
      { id: 4, value: 0 }
    ]
  };

  render() {
    return (
      <div>
        {this.state.counters.map((item => <CounterComponent key={item.id}>{item.id}</CounterComponent>))}
      </div>
    );
  }
}
```

> If we do `{ id: 1, value:7 },` it will still render `0`. **Why?**

First let's add the `value` attribute

```
  render() {
    return (
      <div>
        {this.state.counters.map(item => (
          <CounterComponent value={item.value} key={item.id} selected={true}>{item.id}</CounterComponent>
        ))}
      </div>
    );
  }
}
```

Back to `/counter.jsx` we add temporary inside the render method 

```
  render() {
      console.log('props', this.props);     
```

> Every react comp. has a property called `props` _a basic javascript obj which includes all the attributes that we set_ in `CountersComponent

```
{"value":0,"selected":true,"children":1}
{"value":0,"selected":true,"children":2}
{"value":0,"selected":true,"children":3}
See?
{"value":4,"selected":true,"children":4}
```

Back to `/counter/jsx` instead of

```
 state = {
    count: 0,
    message: "Click To Start to count"
  };
```

We are gonna do 

```
  state = {
    count: this.props.value,
    message: "Click To Start to count"
  };
```

Back to  chrome we can see le last item has `value = 4` 

![Imgur](https://www.dropbox.com/s/ug9ha0vkqmepprb/a.png?raw=1)


## Passing Children

We learned that the `attrs` we set in our `CountersComponent`

```
 <CounterComponent value={item.value} key={item.id} selected={true}>
```

can be passed to the `CounterComponent` using the `props` property.

We have anothe property called **`children`** and _we use that when we want to pass something beetween the opening and closing tag of an elment_

If we add a tag:

```
<div>
	{this.state.counters.map(item => (
	<CounterComponent value={item.value} key={item.id} selected={true}>
		<h4>Hey I am children</h4>
	</CounterComponent>
	))}
</div>
```

and we `log` again the `props` property:

```
value: 0, selected: true, children: {…}
```

To access to its children we simply do `{this.props.children}`

and we can do it **dynamically** just like this:

```
<CounterComponent value={item.value} key={item.id} selected={true}>
	<h4>#{item.id}</h4>
</CounterComponent>
```

## Props vs State

So here in the `Counter` compo. we are suing `props` to initialise the `state`

```
class Counter extends Component {
    state = {
        // count: 0
        count: this.props.value
    };
```

> What's the difference between props and state ?

`Props` *includes data that we give to a component, whereas* `state` *includes data that is local or private to that component*. It meanse other components cannot access to that state. It;s completley internal.

`Props` is *readonly*: we cannot change the input to this component inside of this component.

<h1 style="color:dodgerblue">Raise and Handle Events</h1>

_The component that **owns** a piece of the state, should be the one **modifying** it_.


![Imgur](https://www.dropbox.com/s/faxwblt69vgr8b3/raising-event.png?raw=1)


In order to `delete` a single counter, the `Counter` component - which in Angular would have played the role of the *child* component - will raise an event to `Counters`

```
<button
	onClick={this.props.onDelete}
	className="btn btn-danger btn-sm ml-2">Delete
</button>
```

 and the *parent* component will to handle the event.
 
 ```
         return (
            <div>
                {counters.map((item) => (
                    <Counter
                        key={item.id}
                        onDelete={this.handleDelete}
                        value={item.value}
                        id={item.id}>
                    </Counter>
                    ))}
            </div>
        );
    }
    
    handleDelete = () => {
        console.log('whatever');
    };
 ```
 
 
 
 
 
## Single Source of Truth

Let's add a `btn` to reset all counters to `0`

Back to `counters.jsx`


```
render() {
    const { counters } = this.state;
    
    return (
        <div>
            <button onClick={this.handleReset} className="btn btn-primary m-2">Reset</button>
            
            {counters.map((item, index) => (
                <Counter
                    key={item.id}
                    onDelete={() => this.handleDelete(index)}
                    counter={item}>
                </Counter>
                ))}
        </div>
    );
}

handleReset = ()  => {
    const { counters } = this.state;
    counters.map((item) => {
        item.value = 0;
        return item;
    });
    this.setState(counters);
}
 
```

**The _handleReset()_ won't work!**

```
class Counter extends Component {
    state = {
        // count: 0
        count: this.props.counter.value
    };
```

This piece of code will is executed only once when an instance of a Counter component is created. So that's why when the page loads we get the inital value

```
state = {
        counters: [
            { id: 1, value: 4 },
            { id: 2, value: 0 },
            { id: 3, value: 0 },
            { id: 4, value: 0 }
        ]
    };
```

but when we click the *Reset Btn* the local state is not updated!

> How can we fix this problem?

We need to remove the local state in our counter component and have a single source of truth.
 
 
![Imgur](https://www.dropbox.com/s/lfks31et1of0aev/Single%20Source%20of%20Truth.png?raw=1)



## Removing the Local State

So we want to remove the state:

```
class Counter extends Component {
    state = {
        // count: 0
        count: this.props.counter.value
    };
```

and relay only on the `props` to get all the data the component needs.

Now `counters.jsx` looks like:


```
import React, { Component } from 'react';

class Counter extends Component {

    styles = {
        fontSize: 25,
        fontWeight: "bold"
    };

    render() {
        let classes = this.getBadgeClass();
        console.log(this.props.counter);

        return (
            <div style={{ display: "block" }}>
                <span style={this.styles} className={classes}>
                    {this.formatCount()}
                </span>

                <button
                    onClick={() => this.props.onIncrement(this.props.counter)}
                    className="btn btn-secondary btn-sm"
                >
                    Increment
                </button>

                <button
                    onClick={this.props.onDelete}
                    className="btn btn-danger btn-sm ml-2"
                >
                    Delete
                </button>
            </div>
        );
    }

    getBadgeClass() {
        let classes = "badge m-2 badge ";
        classes += this.props.counter.value === 0 ? "badge-warning" : "badge-primary";
        return classes;
    }

    formatCount() {
        const { value } = this.props.counter;
        return value === 0 ? "Zero" : value;
    }
}
 
export default Counter;

```

<h1 style="color:dodgerblue">Multiple Components in Sync</h1>


Let's take out app to the next level and make it more complex:

![Imgur](https://www.dropbox.com/s/dff5yq5ann2tfuj/Multiple%20Components%20in%20Sync.png?raw=1)

**1** In our `index.js` we need to:

<s>`ReactDOM.render(<Counters />, document.getElementById('root'))`</s>

`ReactDOM.render(<App />, document.getElementById('root'));`

**2** Back to `App.js`

```
import React, { Fragment } from 'react';
import logo from './logo.svg';
import './App.css';
import Navbar from './components/navbar/navbar';
import Counters from './components/counter/parent/counters';

function App() {
    return (
    <React.Fragment>
        <Navbar />
        <main className="container">
            <Counters />
        </main>
   </React.Fragment>
  );
}

export default App;
```

We want to display the tolat amount of counters inside the `nav`.

> How we can share the data bertween 2 components in line?

![Imgur](https://www.dropbox.com/s/dp1qk31b19911cl/Multiple%20Components%20in%20Sync2.png?raw=1)

**We need to lift the state up**: so in this case we want to lift the state of the `Counters` component to it's parent aka the `App` component. Now both counters and navbar have the same parent.


## Lifting the State Up

We should move the the `state` and all the methods related to the state from `Counters` to `App` component which will look like:

```
render() { 
        return (
            <React.Fragment>
                <Navbar />
                <main className="container">
                    <Counters
                        oneReset={this.handleeReset}
                        onDelete={this.handleDelete}
                        onIncrement={this.handleIncrement}>
                    </Counters>
                </main>
            </React.Fragment>
        );
    }
```


Because we have not `state` anymore `Counters` we need to relay on the `props`, just like this:


```
class Counters extends Component {

    render() {

        return (
            <div>
                <button
                    onClick={this.props.onReset}
                    className="btn btn-primary m-2">Reset
                </button>

                {this.props.counters.map((item) => (
                    <Counter
                        key={item.id}
                        onDelete={this.props.onDelete}
                        onIncrement={this.props.onIncrement}
                        counter={item}>
                    </Counter>
                ))}
            </div>
        );
    }
}
```


...hence the we need to update `App` one more time and pass `counters={this.state.counters}`:

```
class App extends Component {

    state = {
        counters: [
            { id: 1, value: 4 },
            { id: 2, value: 0 },
            { id: 3, value: 0 },
            { id: 4, value: 0 }
        ]
    };
    
    render() { 
        return (
            <React.Fragment>
                <Navbar />
                <main className="container">
                    <Counters
                        counters={this.state.counters}
                        oneReset={this.handleeReset}
                        onDelete={this.handleDelete}
                        onIncrement={this.handleIncrement}>
                    </Counters>
                </main>
            </React.Fragment>
        );
    }
```

Same story for `Navbar`:

```
return (
    <React.Fragment>
    
        <Navbar totalCount={this.state.counters.length} />
        
        <main className="container">
            <Counters
                counters={this.state.counters}
                onReset={this.handleReset}
                onDelete={this.handleDelete}
                onIncrement={this.handleIncrement}>
            </Counters>
        </main>
    </React.Fragment>
);
```

and then...

```
class Navbar extends Component {
    render() { 
        return (
            <nav className="navbar navbar-light bg-light">
                <a className="navbar-brand" href="#">
                    Total Count: {this.props.totalCount}
                </a>
            </nav>
        );
    }
}
```

<h1 style="color:dodgerblue">Lifecycle Hooks</h1>

Our components go through few phases during their lifecycle:

1. *Mounting phase* and this is when an instance of a component is created and inserted into the DOM.
2. *Update phase* and this happens when the `state`, or the `props` of a component get changed.
3. *Unmounting phase*, this happens when a component is removed from the DOM such as when we delete the counter.


> **What are the lifecycle hooks?** 
> Methods that allow us to hook into certain moments during the lifecycle of a component and do something. React will call this methods in the following order:

![Imgur](https://www.dropbox.com/s/p1emztqnkz0bfaq/Lifecycle%20Hooks.png?raw=1)



## Mounting phase

**1. `constructor`**

```
class App extends Component {

    state = {};
    
    constructor(props) {
    	super(props);
    	this.state = this.props.something;
    }
}
```

This constructor is called only once when an instance of a class is created. This is a great opportunity to initialise the propertie - i.e. `this.state`


**2. `render`**


```
class App extends Component {

    state = {};
      
    render() {}
}
```




**3. `componentDidMount`**

This methid is called **after** our component is rendered into the DOM and it's the perfect place to make *AJAX* calls to get data from the server.

```
class App extends Component {

    state = {};
  
    constructor() {
    	super();
    }
    
    componentDidMount() {
    	Ajax Call
    	this.setState({ response });
    }
}
```

> **You cannot use lifecylcle hooks in stateless components!**


## Updating phase

**1. `render`**

In this method we are updating the state `this.setState({ counters });` and this will schedule a call to the `render()`.


```
class App extends Component {

   handleIncrement = (counter) => {
        const counters = [...this.state.counters];
        const index = counters.indexOf(counter);
        counters[index] = { ...counter };
        counters[index].value++;
        this.setState({ counters });
    };    
    render() {}
}
```



**2. `componentDidUpdate`**

This method is called after a component is updated. Which means we have new state or new props, so we can compare this new state/props with the old one and if there's a change we can make a new *AJAX* req. to get data from the server.


```
class App extends Component {
    
        componentDidUpdate(prevProps, prevState) {
        console.log('prevProps', prevProps);
        console.log('prevState', prevState);

        if (prevProps.counter.value !== this.props.counter.value) {
            // Ajax Call
        }
    }
 }
```

## Unmounting phase

This method is called just before a component is removed from the DOM and this give us the opportunity to do any kind of clen up.

```
class App extends Component {
    
    componentWillMount() {
    
    }   
}
```












