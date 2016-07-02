# mobx-connect

Super lightweight (2 kb uncompressed) [MobX](https://github.com/mobxjs/mobx) `@connect` decorator for react components.
Similar to `@connect` from react-redux.

## Installation

    npm install mobx-connect --save


## How it works

By decorating your react component with `@connect` 2 things happen:

+ Your components becomes observable (so no need to define @observable, since @connect does it for you).
+ Your state and the store actions are inject into `this.context`.


## Usage example

```js
const React = require('react')
const { connect } = require('mobx-connect')

@connect
class App extends React.Component {

    toggleSetting(key) {
        const { settings} = this.context.state
        settings[key] = !settings[key]
    }

    render() {
        const { settings } = this.context.state

        return <div>
            <SettingsView/>
            <button onClick={() => this.toggleSetting('fullscreen')}>
                Turn {settings.fullscreen ? 'OFF' : 'ON'}
            </button>
            <button onClick={() => this.toggleSetting('logger')}>
                Turn {settings.logger ? 'OFF' : 'ON'}
            </button>
        </div>
    }
}

const SettingsView = connect(function(props, context) {
    const { settings } = context.state

    return <div>
        <h1>Settings</h1>
        <p>Fullscreen: {settings.fullscreen ? 'OFF' : 'ON'}</p>
        <p>Logger: {settings.fullscreen ? 'OFF' : 'ON'}</p>
    </div>
})
```




## Configuration

First create a file called `ContextProvider.js`.
We need to wrap our root component (`App` in this case) around this `ContextProvider` before rendering.

```js
const React = require('react')
const { contextTypes } = require('mobx-connect')

class ContextProvider extends React.Component {
    getChildContext() {
        return this.props.context;
    }
    render() {
        return this.props.children;
    }
}

ContextProvider.childContextTypes = contextTypes;
```

Then we define our default state and our store methods which affect the state.

```js
const { observable } = require('mobx')

const context = {
    // An observable mobx object
    state: observable({
        username: ''
    }),
    store: {
        // Your methods that affect the state here
        // You can make this object deeper for more complicated structures
        // or import from another file
        setUsername(username) {
            context.state.username = username;
        }
    }
}
```

Finally we inject context into our app and render HTML on the browser

```js
ReactDOM.render(<ContextProvider context={context}>
    <App/>
</ContextProvider>, document.getElementById('content'));
```



## Using with React-router

```js
const React = require('react')
const ReactDOM = require('react-dom')
const { observable } = require('mobx')
const { Router, RouterContext, browserHistory } = require('react-router')
const ContextProvider = require('./ContextProvider') // or where ever you put it
const routes = require('./routes') // or where ever you put it

const context = {
    state: observable({}),
    store: {}
}

function createElement(props) {
    return <ContextProvider context={context}>
        <RouterContext {...props} />
    </ContextProvider>
}

// Render HTML on the browser
ReactDOM.render(<Router history={browserHistory}
               render={createElement}
               routes={routes}/>,
document.getElementById('container'))
```


# Author

    https://github.com/nightwolfz
