# Overview
This is a specification for non-existent UI Framework called Glassjs. Just an attempt to define a good API that customer would need to interact with. We (and/or somebody else) may implement this spec.

## Goals

* Zero or near to zero boilerplate code
* Want to build a framework which is dynamic. Dynamic as in, if you have a big app, you should be able to release just a component without having to redeploy whole app.
* Very minimal framework related references in User code base. If there has to be, it will be agnostic to this framework name so if somebody else implements with better one, you should be able to run same code there.
* Huge emphasis on conventions over configuration (Can we achive zero configuration?)
* Should include all features that a modern UI app would need
* Loose coupling between discrete components and if it has to, communicate using events
* First class Server side generation support
* Pure framework without any any custom UI components (like no Tree widget or others) but we may build such resharable based on material design
* Moduralized CSS
* Supports only Evergreen browsers (chrome, firefox, edge and safari)

## Non Goals

* Conforming to web-components. I feel that to achieve (almost) zero configuration and boilerplate, it has to deviate from that.

## Inspiration
UI development technologies and how folks view UI development has evolved a lot since last few years, thanks to many improvements to language and also many UI frameworks. So this framework is no exception. This would not have become except for many frameworks that have come before.

This specification and syntax is inspired by

* Markojs
* Aurelia
* Aura (Salesforce)
* Vuew
* React
* And many others

Credit all the contributors of those libraries for doing an incredible job.

# Basics

## Component

`Component` is ANY reusable bits of an UI Application.

Each component is a file with extn `.cmp` and stored under its own folder with same name as component file name without `.cmp` extension, within the namespace folder.

A Component is identified by a name which also dictates where it is stored in the file system. Each component is of a `type` and `type` defines what kind of functionality that component exhibits.

Here are the types:

* `view` (default)
* `app`
* `event`
* `interface`
* `lib`
* ...

Depending on the type, component support various types of blocks of code. Each block is associated with semantic meaning and syntactic language.

Here are the various blocks of code.

* `markup` (required)
* `script`
* `style`
* `doc`
* ...

Each block of code is stored with pair of start and end tags with their name. For ex.,

```xml
<markup>
    ... markup
</markup>

<script>
    ... script code
</script>

<style>
    ... style code
<style>
```

## Component Names
Component name must follow `localCamelCase` format with following rules.

* Must being with letter
* and followed by alphanumeric chars
* Must be unique in a namespae

## Namespace

Namespace is a string that virtually groups set of components. Components are ALWAYS be part of one (only one) namespace. Components with same namespace may have different privileges.

Component is always referred in the format `namespace:componentName`.

Each Javascript package can consists of multiple namespaces and each namespace consists of multiple components.

# File Structure

File structure of typical project using this framework is as shown below.

```console
$ tree
.
├── LICENSE
├── README.md
├── package.json
└── src
    └── ui
        ├── main
        │   ├── main
        │   │   └── main.cmp
        ├── ns1
        │   ├── comp1
        │   │   └── comp1.cmp
        │   └── comp2
        │       └── comp2.cmp
        └── ns2
            ├── comp1
            │   └── comp1.cmp
            └── comp2
                └── comp2.cmp
```

* `src/ui` is base path (which could be anything)
* Under `src/ui` each folder indicates it is a namespace. In this example `ns1` and `ns2` are namespaces
* Under each namespace there will one folder for each component named after component name itself
* Within the component folder, there will be a file with extension `.cmp` which is main component code.
* There could be multiple other files within each component folder, which are referred within the each component code.

## App Component
App component is main entry point into UI application, similar to main methods in programming languages. App component is the first page that loads from the server and initializes the initial document. All other interactions within this document context.

No other components can include app component as child include another app component.

App component is compiled differently that depending on the compiler, it may be compiled into bundle with all of its non-conditional components are compiled into to reduce the page load times. Where as other components, usually compiled individually.

App components are also gets new additional life cycle methods compared to view components.

Here is most simple UI application one can define.

`src/ui/main/main.cmp`
```html
<markup type="app">
    Hello World
</markup>
```

Since `type` is default attribute, you can specify same code using short form as below.

```html
<markup=app>
    Hello World
</markup>
```

# View Component

Component with `type="view"` are called view components. View Component is used to build any visible part of the application (dom in browser and string render in server).

Here is a most basic view component you can define.

```xml
<markup>
    Hello World
</markup>
```
When rendered this should show a text node in the UI with `Hello World`.

## Component Description
Description of the component can be specified using `desc` attribute in `markup` tag as follows.

```xml
<markup desc="Just displays static 'Hello World'"/>
    Hello World
</markup>
```

If you have larger block of code, you can include `desc` tag within the component
```xml
<markup/>
    <desc>
        This is description block which explains what this component does.
    </desc>
    Hello World
</markup>
```

If both are specified, then expanded text takes precedence. There must be only one `desc` tag per component (there could be one for attr but that is at attribute level. See below). In these cases, compiler will warn the ambuguity.

## Html Tags
Component can include all of allowed html tags. Some tags are not allowed due to security or deprecated reasons. List of tags allowed is at the end of this document.

```html
<markup>
    <div>
        <header>Today's Headlines</header>
        <div class="body">
            <ul>
                <li>Company acquired</li>
                <li>500m accounts breached</li>
            </ul>
        </div>
    </div>
</markup>
```

## Html Comments
Html comments can be specified as usual and they will be rendered as specified.

```html
<markup>
    <!-- This is Header -->
    <div>Header</div>

    <!-- This is Body -->
    <div>Body</div>
</markup>
```

## Html Attributes
Sometimes framework may use an attribute which is also a html attribute. In that case, you can prefix with `html-` which will be passed down to html markup as is.

```html
<markup>
    <div html-for="firstName"/>
</markup>
```

## Html Data Attributes
Data attributes can be specified as usual using `data-` prefix.

```html
<markup>
    <div data-firstName="John" data-lastName="Doe"/>
</markup>
```

## Multiple root elements
Markup can contain multiple root elements

```html
<markup>
    <span class="first">John</span>
    <span class="last">Doe</span>
</markup>
```

## Attributes
Each component can define zero or more attributes which becomes the API for external parties to consume this component.

Each attribute is defined within the `markup` block as below.

```xml
<markup>
    <attr name="name" type="string" desc="The name to be displayed" default="World"/>
</markup>
```

where
* `name` is name of the component. Name should follow javascript variable convention at least 2 chars. Some names are reserved (`state`, `s`, `c`)
* `type` is type of the attributes. Valid values are `string`, `number`, `boolean`, `object`
* `desc` is description of the attribute
* `default` default value of the attribute. It is set to this, if one is not specified by the consumer of this component.

Since only `name` is required for an attribute and type defaults to `string`, you can write above component as

```xml
<markup>
    <attr name="name"/>
</markup>
```

If you have loger description for this attribute, it can be specified using `attr` tag in the attribute.

```xml
<markup>
    <attr name="name">
        <desc>
            This is longer version of the description
        </desc>
    </attr>
</markup>
```

## Default Attributes
Each view component inherits a default set of attributes which are as follows. Since they are inherited by default, those cannot be redefined again. If defined, warning will be logged and ignored.

```html
<attr name="class" type="string" desc="String or object representing list of css classes" default=""/>
<attr name="version" type="string" desc="Indicates the version of the component that indicates what behavior this component should exhibit, if it supports multiple" default="1.0"/>
<attr name="body" type="Component[]" desc="The body of this component, that is everything between start and end tags"/>
```

## Referring to attributes in the markup
Each defined attributes can be used in the markup using `${attribute-name}` pattern as below.

```xml
<markup>
    <attr name="name"/>
    Hello ${name}
</markup>
```

## Expressions using attributes
Within `${` and `}` you can use javascript variables and expressions. Not all javascript is supported but simple expressions to simplify rendering the values.

```xml
<markup>
    <attr name="firstName"/>
    <attr name="lastName"/>
    Hello ${firstName + ' ' + lastName}
</markup>
```

## Without Html Escaping
If you have a text that you are sure safe, you can set text as is to `innerHTMl` using `$!{` and `}`

```xml
<markup>
    <attr name="productDocs"/>
    Product: $!{productDocs}
</markup>
```

## CSS Styles
Styles can be specified inside `style` tag. Supported language is [scss](https://sass-lang.com/guide).

```html
<style>
    .header {
        font-size: 1.2em;
    }

    .body {
        font-size: 1em;
    }
</style>
<markup>
    <div class="header">This is the header</div>
    <div class="body">This is body</div>
</markup>
```

## Unscoped CSS Classes

By default all css classes are scoped and applicable only with that component and containment. If you would like a component styles are to be unscoped, add `unscoped` to style tag.

```html
<style unscoped>
    .header {
        font-size: 1.2em;
    }

    .body {
        font-size: 1em;
    }
</style>
<markup>
    <div class="header">This is the header</div>
    <div class="body">This is body</div>
</markup>
```

## Condetional Rendering
If you would like to conditionally render a component use `if` attribute. Note when expression is falsy, the component is removed from the DOM.

```html
<markup>
    <attr name="showBody" type="boolean"/>

    <div if="showBody">This is body</div>
</markup>
```

You can add zero or more `elseif` conditioned sublings and zero or one `else` tag as below.

```html
<markup>
    <attr name="showLevel" type="number"/>

    <div if="showLevel === 1">This is body Level1</div>
    <div elseif="showLevel === 2">This is body Level2</div>
    <div else>This is body Level3-n</div>
</markup>
```

If you have multiple tags to be rendered for each of conditional, you can use expanded tags version.

```html
<markup>
    <attr name="showLevel" type="number"/>

    <if="showLevel === 1">
        <div>This is body Level1</div>
        <div>This is body Level1</div>
    </if>
    <elseif="showLevel === 2">
        <div >This is body Level2</div>
        <div >This is body Level2</div>
    </elseif>
    <else>
        <div>This is body Level3-n</div>
        <div>This is body Level3-n</div>
    </else>
</markup>
```

## Component Visibility Control
If you want to keep the component in the dom but just want to hide it, you can use display attribute. This basically add or removes `display: none` style property.

```html
<markup>
    <attr name="showLevel" type="number"/>

    <div display="showLevel == 1">This is body Level1</div>
    <div display="showLevel == 2">This is body Level2</div>
    <div display="showLevel > 2">This is body Level3-n</div>
</markup>
```

## Iterating Values
If you have an `Iterable` value, you can iterate over values and use it to render values.

```html
<markup>
    <attr name="colors" type="string[]" default="['red', 'blue']">

    <ul>
        <li for="color of colors">${color}</li>
    </ul>
</markup>
```

If you have multiple tags to process for each iteration, then you can use expanded form.

```html
<markup>
    <attr name="colors" type="string[]" default="['red', 'blue']">

    <ul>
        <for="color of colors">
            <li>${color}</li>
        </for>
    </ul>
</markup>
```

If value specified for iteration is not an `Iterable` then array will be created with specified single value.

```html
<markup>
    <attr name="color" type="string" default="red">

    <ul>
        <for="color of colors">
            <li>${color}</li>
        </for>
    </ul>
</markup>
```


### Iteration variable `$value`
In addition to creating requested iteration variable (in this case `color`), it also creates special variable called `$value` so you can omit your variable and use it.

```html
<markup>
    <attr name="colors" type="string[]" default="['red', 'blue']">

    <ul>
        <for="colors">
            <li>${$value}</li>
        </for>
    </ul>
</markup>
```

## Iteration metadata variable `$loop`
Sometimes you may want to access index of iteration or check if you are at last, first or odd or evan. To access those state, you can use special variable `$loop`.

```html
<markup>
    <attr name="colors" type="string[]" default="['red', 'blue']">

    <ul>
        <for="colors">
            <li>Index: ${$loop.length}</li>
            <li>Index: ${$loop.index}</li>
            <li>First: ${$loop.first}</li>
            <li>Last: ${$loop.last}</li>
            <li>Odd: ${$loop.odd}</li>
            <li>Even: ${$loop.even}</li>

            <!-- $loop.value will be same as $value -->
            <li>Loop Value: ${$loop.value}</li>
            <li>Value: ${$value}</li>

        </for>
    </ul>
</markup>
```

## Component Composition
Component can be referred in another component by specifying the namespace and component name.

`ui:displayName`
```html
<markup>
    <attr = name/>

    Hello ${name}
</markup>
```

`ui:names`
```html
<markup>
    <ui:displayName name="John"/>
    <ui:displayName name="Emily"/>
    <ui:displayName name="Kashyap"/>
</markup>
```

## Slots
Slots help you project content from child component into parent component. Parent component defines the slots and their names. Child compoent can project content into those slots.

`display.cmp`
```html
<markup>
    <slot name="header"/>
    this is body
    <slot name="footer"/>
</markup>
```

`main.cmp`
```html
<markup>
    <display>
        <div slot="header">
            this is header
        </div>
        <div slot="footer">
            this is footer
        </div>
    </display>
</markup>
```

Results in following dom.
```html
<div>
    this is header
</div>
this is body
<div>
    this is footer
</div>
```

## Slots default content
When a component defines a slot, it can provide a default content. That default content will be used if there is no content is projected.

`display.cmp`
```html
<markup>
    <slot name="header">
        this is default header
    </slot>
    this is body
    <slot name="footer"/>
</markup>
```

`main.cmp`
```html
<markup>
    <display>
        <div slot="footer">
            this is footer
        </div>
    </display>
</markup>
```

Results in following dom.
```html
<div>
    this is default header
</div>
this is body
<div>
    this is footer
</div>
```

## Routing
Routing can be configured using `route` tag.

```html
<markup type="apo">

    <route path="/about">
        <ui:about/>
    </route>

    <route path="/history">
        <ui:history/>
    </route>
</markup>
```

Since `path` is default attribute, it can be written using short form.

```html
<markup=app>

    <route="/about">
        <ui:about/>
    </route>

    <route="/history">
        <ui:history/>
    </route>
</markup>
```

## Routing path can have path variable
Path can define set of path variables which are available via `$route.params` object, which can be passed to downstream components.

```html
<markup type="app">

    <route path="/users/:userId">
        <ui:userDisplay userId="$route.params.userId"/>
    </route>
</markup>
```

## Nested Routes
Rotues can be nested within other routes. Child routes are relatively place within the parent components's route context.

```html
<markup type="app">
    <route path="/users/:userId">
        <ui:userDisplay userId="$route.params.userId"/>
    </route>
</markup>
```

`ui:userDisplay`
```html
<markup>
    <attr=userId/>

    User Id is ${userId} 

    <route path="/profile">
        <ui:userDetails userId="${userId}"/>
    </route>
    <route path="/edit">
        <ui:userDetails userId="${userId}"/>
    </route>
</markup>
```

## Route Link tag

Continueing above example, let's say other component wants to link to navigate to those routes, it can use `link` tag. Note that link tag always link based on relative paths, which is relative to parent route's context. This is to ensure that component can route even when it is used in different parent contexts.

```html
<markup>
    <attr=userId/>

    User Id is ${userId}
    <link to="/profile">Goto Profile</link>
    <link to="/edit">Goto Edit</link>

    <route path="/profile">
        <ui:userDetails userId="${userId}"/>
    </route>
    <route path="/edit">
        <ui:userDetails userId="${userId}"/>
    </route>

</markup>
```
Since `to` is default attribute link declaration can be shortned to

```html
<link="/profile">Goto Profile</link>
```


## Linking using absolute paths

Lets say that you are developing a user display component which can be used by two different application. However each application is displaying this component under different paths. So when user component wants to link between profile and edit, it cannot hardcode path from beginning as that might change. Instead component can make use of `rpath` attribute which automatically prefixes the parent's route path which would be set to different values depending where `userDisplay` component is used.

`ui:userDisplay`
```html
<markup>
    <attr=userId/>

    User Id is ${userId}

    <route path="/profile">
        <ui:userDetails userId="${userId}"/>
    </route>
    <route path="/edit">
        <ui:userDetails userId="${userId}"/>
    </route>
</markup>
```

`ui:app1`
```html
<markup>
    <route path="/app1/users/:userId">
        <ui:userDisplay userId="$route.params.userId"/>
    </route>
</markup>
```

`ui:app2`
```html
<markup>
    <route path="/app2/users/:userId">
        <ui:userDisplay userId="$route.params.userId"/>
    </route>
</markup>
```

## Controller
Each component can have one script with one or more ES6 classes. One of such classes can be named `Controller`, which acts as controller for this component. Controller can receive life cycle callbacks, interact with server, update view, fire events etc.,

Controller can define instance variables (called state) and methods. Variables can be accessed in component using `c.` (c stands for controller) reference.

```xml
<script>
    class Controller {
        constructor(cmp, ctx) {
            this.name = 'John Doe';
        }
    }
</script>
<markup>
    Hello ${c.name}
</markup>
```

The reason all controller variable access needs to be prefixed with `c` is to avoid any name collissions if component defines an attribute with same name.

```xml
<script>
    class Controller {
        constructor(cmp, ctx) {
            this.name = 'John Doe';
        }
    }
</script>
<markup>
    <attr name="name"/>
    Hello ${name} and ${c.name}
</markup>
```

## Controller Constructor

Each `Controller` can have a contructor which receives two parameters `cmp` and `ctx`.

First parameter `cmp` is an object that represents the component instance. This object would give access information about the current instance of the component. For ex., you can call `cmp.metadata` which gives access to component Metadata, which allows you to query component name, its defined attributes, access to component instance attribute values etc., as we will explore more about it later.

Second parameter `ctx` is an context object from the framework. Context object would provide all needed properties and methods needed to interact with framework. This would be the only connectivity between component and framework runtime.

## Bind Events to Controller methods
You can define methods which can be bound to one or more events. Events follow `React` event convention of `onEventName` format. For ex., `onClick` or `onMouseOver` etc.,

In this example, we are incrementing a state variable on clicking a button which is rendered in the view.

```html
<script>
    class Controller {
        constructor(cmp, ctx) {
            this.count = 0;
        }

        increment() {
            this.count++;
        }
    }
</script>
<markup>
    Current Count is ${c.count}
    <button onClick={increment()}>Click Me!</button>
</markup>
```

Credit: Example is from [Markojs documentation](https://markojs.com/docs/getting-started/#adding-state)

## Passing Parameters to Controller functions

You can pass one or more parameters to controller function.

```html
<script>
    class Controller {
        constructor(cmp, ctx) {
            this.count = 0;
        }

        increment(incrBy) {
            this.count += incrBy;
        }
    }
</script>
<markup>
    Current Count is ${c.count}
    <button onClick="increment(1)">Click Me!</button>
    <button onClick="increment(10)">Click Me to Speed up!</button>
</markup>
```
