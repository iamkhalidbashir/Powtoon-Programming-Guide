# PowToon React Guide

*A mostly reasonable approach to React.*

## Table of Contents

  1. [Prefer Composition over Inheritance and Mixins](#prefer_composition_over_inheritance_and_mixins)
  1. [Stateless vs Pure vs Regular Components](#stateless_vs_pure_vs_regular_components)
  1. [Naming](#naming)
  1. [Declaration](#declaration)
  1. [Alignment](#alignment)
  1. [Quotes](#quotes)
  1. [Spacing](#spacing)
  1. [Props](#props)
  1. [Refs](#refs)
  1. [Parentheses](#parentheses)
  1. [Tags](#tags)
  1. [Methods](#methods)
  1. [Ordering](#ordering)
  1. [isMounted](#ismounted)
  1. [Component Functions](#component_functions)

## Prefer Composition over Inheritance and Mixins
In general, mixins and inheritance are worse then composition in most of the cases
in theory ([even in OOP](https://www.joezimjs.com/javascript/composition-is-king/)).
When following, like we do, the Functional Programming approach, inheritance should
be used even rarely and with the addition of language limitations, like bad handling
if static members, composition should always replace inheritance and mixins.

"Higher order components" are the implementation of composition with React components.

To read more about this:
* [Dan Abramov about why mixins should be replaced with higher order components](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)
* [Facebook's documentation about composition vs. inheritance](https://facebook.github.io/react/docs/composition-vs-inheritance.html)

**Bad:**
```jsx
class BaseButton extends React.Component {
  static PropTypes = {
    onClick: React.PropTypes.func.isRequired,
    text: React.PropTypes.string.isRequired
  }
  
  getClassName(){
    return 'base-button'
  }
  
  render(){
    const {text, onClick} = this.props
    return <Button className={this.getClassName()} onClick={onClick}>{text}</Button>
  }
}

class RedButton extends BaseButton {
  getClassName(){
    return 'red-button'
  }
}
```

**Good:**
```jsx
class BaseButton extends React.Component {
  static PropTypes = {
    onClick: React.PropTypes.func.isRequired,
    text: React.PropTypes.string.isRequired,
    className: React.PropTypes.string
  }
  
  static defaultProps = {
    className: 'base-button'
  }
  
  render(){
    const {onClick, text, className} = this.props
    return <Button className={className} onClick={onClick}>{text}</Button>
  }
}

class RedButton extends React.Component {
  static PropTypes = {
    onClick: React.PropTypes.func.isRequired,
    text: React.PropTypes.string.isRequired
  }
  
  render(){
    return <BaseButton className={'red-button'} {...this.props} />
  }
}
```

If you want to add functionality to a component, create a wrapper component
that uses the component inside it's render function.

```jsx
class Hidable extends React.Component {
  state = {
    hidden: false
  }
  
  onClick = () => {
    const {hidden} = this.state
    this.setState({hidden: !hidden})
  }
  
  render(){
    const {hidden} = this.state
    const {children} = this.props
    
    return (
      <div onClick={this.onClick} hidden={hidden} >
        {children}
      </div>
    )
  }
}

function SomeComponent(){
 return (
   <Hidable>
     <span>Some Text</span>
   </Hidable>
 )
}

```

Sometimes, composition looks better using a wrapping function. This way of wrapping
is also good because it can be used as a decorator in future when the language
fully support it (and this should happen soon).

```jsx
function makeHidable(WrappedComponent){
  return class Hidable extends React.Component {
    state = {
      hidden: false
    }
    
    onClick = () => {
      const {hidden} = this.state
      this.setState({hidden: !hidden})
    }
    
    render(){
      const {hidden} = this.state

      <div onClick={this.onClick} hidden={hidden} >
        <WrappedComponent {...this.props} />
      </div>
    }
  }  
}

class SomeClass extends React.Component {
  ...
}
SomeClass = makeHidable(<SomeClass />)

function SomeComponent(){
  return (
    <HidableSomeClass />
  )
}

// makeHidable can also be used as a decorator

@makeHidable
class SomeClass extends React.Component {
  ...
}

```

**[⬆ back to top](#table-of-contents)**

## Stateless vs Pure vs Regular Components
* A Stateless Component is a component without a state defined by a function. For example:
  ```jsx
  function Hello({someone}){
    return <div>{`Hello ${someone}`}</div>
  }
  ```
  It is the shortest in terms of lines of code.
  
* A Pure Component is a component that launches a shallow compare inside it's
[`ShouldComponentUpdate()`](https://facebook.github.io/react/docs/react-component.html#shouldcomponentupdate).
It is the fastest of the three. Yes- [it is faster then stateless components](https://medium.com/modus-create-front-end-development/component-rendering-performance-in-react-df859b474adc#.i6l3cqqzp).

* A Regular React Component. It is the most flexible one.

>//TODO: More guidelines will follow. For now, choose the component type you need based on the explanations above.

**[⬆ back to top](#table-of-contents)**

## Naming

  - **Reference Naming**: Use PascalCase for React components and camelCase for their instances.
  eslint: [`react/jsx-pascal-case`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-pascal-case.md)

    ```jsx
    // bad
    import reservationCard from './ReservationCard'

    // good
    import ReservationCard from './ReservationCard'

    // bad
    const ReservationItem = <ReservationCard />

    // good
    const reservationItem = <ReservationCard />
    ```

  - **Component Naming**: Use the filename as the component name. For example,
  `ReservationCard.jsx` should have a reference name of `ReservationCard`.
  However, for root components of a directory, use `index.jsx` as the filename
  and use the directory name as the component name:

    ```jsx
    // bad
    import Footer from './Footer/Footer'

    // bad
    import Footer from './Footer/index'

    // good
    import Footer from './Footer'
    ```
    
  - **Higher-order Component Naming**: Use a composite of the higher-order component's
  name and the passed-in component's name as the `displayName` on the generated component.
  For example, the higher-order component `withFoo()`, when passed a component `Bar`
  should produce a component with a `displayName` of `withFoo(Bar)`.

  > Why? A component's `displayName` may be used by developer tools or in error messages,
  and having a value that clearly expresses this relationship helps people understand what is happening.

    ```jsx
    // bad
    export default function withFoo(WrappedComponent) {
      return function WithFoo(props) {
        return <WrappedComponent {...props} foo />
      }
    }

    // good
    export default function withFoo(WrappedComponent) {
      function WithFoo(props) {
        return <WrappedComponent {...props} foo />
      }

      const wrappedComponentName = WrappedComponent.displayName
        || WrappedComponent.name
        || 'Component'

      WithFoo.displayName = `withFoo(${wrappedComponentName})`
      return WithFoo
    }
    ```

  - **Props Naming**: Avoid using DOM component prop names for different purposes.

  > Why? People expect props like `style` and `className` to mean one specific thing.
  Varying this API for a subset of your app makes the code less readable and less maintainable, and may cause bugs.

    ```jsx
    // bad
    <MyComponent style="fancy" />

    // good
    <MyComponent variant="fancy" />
    ```

**[⬆ back to top](#table-of-contents)**

## Declaration

  - Do not use `displayName` for naming components. Instead, name the component by reference.

    ```jsx
    // bad
    export default React.createClass({
      displayName: 'ReservationCard',
      // stuff goes here
    })

    // good
    export default class ReservationCard extends React.Component {
    }
    ```

**[⬆ back to top](#table-of-contents)**

## Alignment

  - Follow these alignment styles for JSX syntax.
  eslint: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md)

  > Why? This should help to better keep track of a component's start, end and children.

  ```jsx
  // bad
  <Foo superLongParam="bar"
       anotherSuperLongParam="baz" />

  // good
  <Foo
    superLongParam="bar"
    anotherSuperLongParam="baz"
  />

  // if props fit in one line then keep it on the same line
  <Foo bar="bar" />

  // children get indented normally
  <Foo
    superLongParam="bar"
    anotherSuperLongParam="baz"
  >
    <Quux />
  </Foo>
  ```

**[⬆ back to top](#table-of-contents)**

## Quotes

  - Always use double quotes (`"`) for JSX attributes, but single quotes (`'`) for all other JS.
  eslint: [`jsx-quotes`](http://eslint.org/docs/rules/jsx-quotes)

  > Why? Regular HTML attributes also typically use double quotes instead of single, so JSX attributes mirror this convention.

    ```jsx
    // bad
    <Foo bar='bar' />

    // good
    <Foo bar="bar" />

    // bad
    <Foo style={{ left: "20px" }} />

    // good
    <Foo style={{ left: '20px' }} />
    ```

**[⬆ back to top](#table-of-contents)**

## Spacing

  - Always include a single space in your self-closing tag.
  eslint: [`no-multi-spaces`](http://eslint.org/docs/rules/no-multi-spaces),
  [`react/jsx-space-before-closing`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-space-before-closing.md)

    ```jsx
    // bad
    <Foo/>

    // very bad
    <Foo                 />

    // bad
    <Foo
     />

    // good
    <Foo />
    ```

  - Do not pad JSX curly braces with spaces.
  eslint: [`react/jsx-curly-spacing`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-curly-spacing.md)

    ```jsx
    // bad
    <Foo bar={ baz } />

    // good
    <Foo bar={baz} />
    ```

**[⬆ back to top](#table-of-contents)**

## Props

  - Always use camelCase for prop names.

    ```jsx
    // bad
    <Foo
      UserName="hello"
      phone_number={12345678}
    />

    // good
    <Foo
      userName="hello"
      phoneNumber={12345678}
    />
    ```

  - Omit the value of the prop when it is explicitly `true`.
  eslint: [`react/jsx-boolean-value`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-boolean-value.md)

    ```jsx
    // bad
    <Foo
      hidden={true}
    />

    // good
    <Foo
      hidden
    />
    ```

  - Always include an `alt` prop on `<img>` tags. If the image is presentational,
  `alt` can be an empty string or the `<img>` must have `role="presentation"`.
  eslint: [`jsx-a11y/img-has-alt`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/img-has-alt.md)

    > //TODO: is this really necessary?
  
    ```jsx
    // bad
    <img src="hello.jpg" />
    
    // good
    <img src="hello.jpg" alt="Me waving hello" />
    
    // good
    <img src="hello.jpg" alt="" />
    
    // good
    <img src="hello.jpg" role="presentation" />
    ```

  - Do not use words like "image", "photo", or "picture" in `<img>` `alt` props.
  eslint: [`jsx-a11y/img-redundant-alt`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/img-redundant-alt.md)

    > //TODO: is this really necessary?
    
    > Why? Screenreaders already announce `img` elements as images, so there is no need to include this information in the alt text.

    ```jsx
    // bad
    <img src="hello.jpg" alt="Picture of me waving hello" />

    // good
    <img src="hello.jpg" alt="Me waving hello" />
    ```

  - Use only valid, non-abstract [ARIA roles](https://www.w3.org/TR/wai-aria/roles#role_definitions). eslint: [`jsx-a11y/aria-role`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/aria-role.md)

    ```jsx
    // bad - not an ARIA role
    <div role="datepicker" />

    // bad - abstract ARIA role
    <div role="range" />

    // good
    <div role="button" />
    ```

  - Do not use `accessKey` on elements.
  eslint: [`jsx-a11y/no-access-key`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/no-access-key.md)

  > Why? Inconsistencies between keyboard shortcuts and keyboard commands used by people using screenreaders and keyboards complicate accessibility.

  ```jsx
  // bad
  <div accessKey="h" />

  // good
  <div />
  ```

  - Avoid using an array index as `key` prop, prefer a unique ID.
  
  > Why? In short, a key is the only thing React uses to identify DOM elements.
  What happens if you push an item to the list or remove something in the middle?
  If the key is same as before React assumes that the DOM element represents the same component as before.
  But that is no longer true. more info can be found in [this blog post](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318).
  Please understand how it works. 

  ```jsx
  // bad
  {todos.map((todo, index) =>
    <Todo
      {...todo}
      key={index}
    />
  )}

  // good
  {todos.map(todo => (
    <Todo
      {...todo}
      key={todo.id}
    />
  ))}
  ```

  - Always define explicit defaultProps for all non-required props.

  > Why? propTypes are a form of documentation, and providing defaultProps means the reader of your code doesn’t
  have to assume as much. In addition, it can mean that your code can omit certain type checks.

  ```jsx
  // bad
  function SFC({ foo, bar, children }) {
    return <div>{foo}{bar}{children}</div>
  }
  SFC.propTypes = {
    foo: PropTypes.number.isRequired,
    bar: PropTypes.string,
    children: PropTypes.node,
  }

  // good
  function SFC({ foo, bar }) {
    return <div>{foo}{bar}</div>
  }
  SFC.propTypes = {
    foo: PropTypes.number.isRequired,
    bar: PropTypes.string,
    children: PropTypes.node,
  }
  SFC.defaultProps = {
    bar: '',
    children: null,
  }
  ```

**[⬆ back to top](#table-of-contents)**

## Refs

  - Always use ref callbacks.
  eslint: [`react/no-string-refs`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-string-refs.md)
    
  > Remember! If an element unmounts, ref function is called with undefined. 

  ```jsx
  // bad
  <Foo
    ref="myRef"
  />
  
  // good
  fooRef = undefined
  setRef = ref => this.fooRef = ref
  
  <Foo
    ref={this.setRef}
  />
  ```

**[⬆ back to top](#table-of-contents)**

## Parentheses

  - Wrap JSX tags in parentheses when they span more than one line.
  eslint: [`react/wrap-multilines`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/wrap-multilines.md)

    ```jsx
    // bad
    render() {
      return <MyComponent className="long body" foo="bar">
               <MyChild />
             </MyComponent>
    }

    // good
    render() {
      return (
        <MyComponent className="long body" foo="bar">
          <MyChild />
        </MyComponent>
      )
    }

    // good, when single line
    render() {
      const body = <div>hello</div>
      return <MyComponent>{body}</MyComponent>
    }
    ```

**[⬆ back to top](#table-of-contents)**

## Tags

  - Always self-close tags that have no children.
    eslint: [`react/self-closing-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/self-closing-comp.md)

    ```jsx
    // bad
    <Foo className="stuff"></Foo>

    // good
    <Foo className="stuff" />
    ```

  - If your component has multi-line properties, close its tag on a new line.
    eslint: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md)

    ```jsx
    // bad
    <Foo
      bar="bar"
      baz="baz" />

    // good
    <Foo
      bar="bar"
      baz="baz"
    />
    ```

**[⬆ back to top](#table-of-contents)**

## Methods

  - Bind event handlers for the render method using the arrow function on the class.
    eslint: [`react/jsx-no-bind`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)

    > Why? A render should be treated as a very heavy function because it is potentially called many times.
    Creating a new function or binding a function is a heavy JS operation.
    A bind call in the render path creates a brand new function on every single render
    and it could cause severe performance issues.
  
    ```jsx
    // bad
    class extends React.Component {
      onClickDiv() {
        // do stuff
      }
  
      render() {
        return <div onClick={this.onClickDiv.bind(this)} />
      }
    }
    
    // bad
    class extends React.Component {
      onClickDiv() {
        // do stuff
      }
    
      render() {
        return <div onClick={() => this.onClickDiv} />
      }
    }
  
    // good
    class extends React.Component {
      onClickDiv = () => {
        // do stuff
      }
  
      render() {
        return <div onClick={this.onClickDiv} />
      }
    }
    ```
    
    - In case of iterations, a *inline component* with recompose's [withHandlers()](https://github.com/acdlite/recompose/blob/master/docs/API.md#withhandlers)
     should be used
    
    ```jsx
    import {withHandlers} from 'recompose'
  
    const ItemList({items, onItemClick}) => {
      return (
        <ul>
          {items.map((item, index) => (
            <Item
              key={item.key}
              onClick={onItemClick}
            />
          ))}
        </ul>
      )
    }
    
    const enhance = withHandlers({
      handleClick: {onClick, name} => event => onClick(name)
    })
    const Item = enhance(({handleClick, name}){
      return <div onClick={handleClick}>Hey!</div>
    })
    
    //OR:
    
    class Item extends Component {
      static propTypes = {
        onClick: PropTypes.func.isRequired,
        name: PropTypes.string.isRequired
      }
      handleClick = e => {
        const {onClick, name} = this.props
        onClick(name)
      }
      render() {
        return <div onClick={this.handleClick}>Hey!</div>
      }
    }
  
    ```

  - Do not use underscore prefix for internal methods of a React component.
    > Why? Underscore prefixes are sometimes used as a convention in other languages to denote privacy.
    But, unlike those languages, there is no native support for privacy in JavaScript, everything is public.
    Regardless of your intentions, adding underscore prefixes to your properties does not actually make them private,
    and any property (underscore-prefixed or not) should be treated as being public.
    See issues [#1024](https://github.com/airbnb/javascript/issues/1024),
    and [#490](https://github.com/airbnb/javascript/issues/490) for a more in-depth discussion.

    ```jsx
    // bad
    React.createClass({
      _onClickSubmit() {
        // do stuff
      },

      // other stuff
    })

    // good
    class extends React.Component {
      onClickSubmit() {
        // do stuff
      }

      // other stuff
    }
    ```

  - Be sure to return a value in your `render` methods.
  eslint: [`react/require-render-return`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/require-render-return.md)

    ```jsx
    // bad
    render() {
      (<div />)
    }

    // good
    render() {
      return (<div />)
    }
    ```

**[⬆ back to top](#table-of-contents)**

## Ordering

  - Ordering for `class extends React.Component`:

  1. optional `static` methods
  1. `constructor`
  1. `getChildContext`
  1. `componentWillMount`
  1. `componentDidMount`
  1. `componentWillReceiveProps`
  1. `shouldComponentUpdate`
  1. `componentWillUpdate`
  1. `componentDidUpdate`
  1. `componentWillUnmount`
  1. *clickHandlers or eventHandlers* like `onClickSubmit()` or `onChangeDescription()`
  1. *getter methods for `render`* like `getSelectReason()` or `getFooterContent()`
  1. `render`

**[⬆ back to top](#table-of-contents)**

## isMounted

  - Do not use isMounted.
  eslint: [`react/no-is-mounted`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-is-mounted.md)

    > Why? [`isMounted` is an anti-pattern][anti-pattern], is not available when using ES6 classes, and is on its way to being officially deprecated.
  
    [anti-pattern]: https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html

**[⬆ back to top](#table-of-contents)**

## Component Functions

  - Extract as many functions as possible out of the component.

    > Why?
    > - First of all, we would like a class to expose as less as possible functions because
      they are public. There are no private functions in JS.
    > - Secondary, functions that are out of the component's scope will have no reference to the
      component's instance ("this"), and this will help to ensure they are pure.
    > - The third reason is that it saves us memory. Arrow functions are created for each
      instance of the component. We would prefer create as less as we can, instead.
    
    ```jsx
    // bad
    class Item extends Component {
      
      //...
      
      getStyle = () => {
        const {width, height} = this.props
        return {
          width: width * 2,
          height: height * 2
        }
      }
      
      render() {
        return (
          <div onClick={this.onClick} style={this.getStyle()}>
            hey!
          </div>
        )
      }
    }
    
    // good
    const getStyle = ({width, height}) => ({
      width: width * 2,
      height: height * 2
    })
        
    class Item extends Component {
            
      //...

      render() {
        const {width, height} = this.props
        const style = getStyle({width, height})
        
        return (
          <div onClick={this.onClick} style={this.getStyle()}>
            hey!
          </div>
        )
      }
    }
    
    // TODO: BETTER
    ```
        
  - Do not use render functions.
  
    > Why? For the same reasons specified in the previous section, but instead of using
      a plain function, use an inline component (with recompose if needed).
        
    ```jsx
    // bad
      // ...
      onClick = e => {
        const {onClick, itemId} = this.props
        onClick(itemId)
      }
      
      // this function will be created as part of construction the component,
      // for every instance of the component.
      renderItem = () => {
        const {itemId} = this.props
        <div onClick={this.onClick}>{`item_${itemId}`}</div>
      }
      
      render() {
        return (
          <div onClick={this.onClick}>
            {this.renderItem()}
          </div>
        )
      }
    }
    
    // good
      //..
      onClick = e => {
        const {onClick, itemId} = this.props
        onClick(itemId)
      }

      render() {
        const {itemId} = this.props
        return (
          <div onClick={this.onClick}>
            <Item itemId={itemId} onClick={onClick}/>
          </div>
        )
      }
    }
    
    //This inline component is created only once.
    const Item = ({itemId, onClick}) => <div onClick={onClick}>{`item_${itemId}`}</div>
    ```
  
**[⬆ back to top](#table-of-contents)**
