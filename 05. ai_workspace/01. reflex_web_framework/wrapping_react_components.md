# **Wrapping React**
One of Reflex's most powerful features is the ability to wrap React components and take advantage of the vast ecosystem of React libraries.
/gitcommi
If you want a specific component for your app but Reflex doesn't provide it, there's a good chance it's available as a React component. Search for it on npm, and if it's there, you can use it in your Reflex app. You can also create your own local React components and wrap them in Reflex.

Once you wrap your component, you publish it to the Reflex library so that others can use it.

**Simple Example**
Simple components that don't have any interaction can be wrapped with just a few lines of code.

Below we show how to wrap the Spline library can be used to create 3D scenes and animations.

```python 
import reflex as rx


class Spline(rx.Component):
    """Spline component."""

    # The name of the npm package.
    library = "@splinetool/react-spline"

    # Any additional libraries needed to use the component.
    lib_dependencies: list[str] = [
        "@splinetool/runtime@1.5.5"
    ]

    # The name of the component to use from the package.
    tag = "Spline"

    # Spline is a default export from the module.
    is_default = True

    # Any props that the component takes.
    scene: rx.Var[str]


# Convenience function to create the Spline component.
spline = Spline.create


# Use the Spline component in your app.
def index():
    return spline(
        scene="https://prod.spline.design/joLpOOYbGL-10EJ4/scene.splinecode"
    )
```

**ColorPicker Example**
Similar to the Spline example we start with defining the library and tag. In this case the library is react-colorful and the tag is HexColorPicker.

We also have a var color which is the current color of the color picker.

Since this component has interaction we must specify any event triggers that the component takes. The color picker has a single trigger on_change to specify when the color changes. This trigger takes in a single argument color which is the new color.

```python 
from reflex.components.component import NoSSRComponent


class ColorPicker(NoSSRComponent):
    library = "react-colorful"
    tag = "HexColorPicker"
    color: rx.Var[str]
    on_change: rx.EventHandler[lambda color: [color]]


color_picker = ColorPicker.create


class ColorPickerState(rx.State):
    color: str = "#db114b"


def index():
    return rx.box(
        rx.vstack(
            rx.heading(
                ColorPickerState.color, color="white"
            ),
            color_picker(
                on_change=ColorPickerState.set_color
            ),
        ),
        background_color=ColorPickerState.color,
        padding="5em",
        border_radius="1em",
    )

```

## What Not To Wrap
There are some libraries on npm that are not do not expose React components and therefore are very hard to wrap with Reflex.

A library like spline below is going to be difficult to wrap with Reflex because it does not expose a React component.

```python
import { Application } from '@splinetool/runtime';

// make sure you have a canvas in the body
const canvas = document.getElementById('canvas3d');

// start the application and load the scene
const spline = new Application(canvas);

spline.load('https://prod.spline.design/6Wq1Q7YGyM-iab9i/scene.splinecode');
```

You should look out for JSX, a syntax extension to JavaScript, which has angle brackets `(<h1>Hello, world!</h1>)`. If you see JSX, it's likely that the library is a React component and can be wrapped with Reflex.

If the library does not expose a react component you need to try and find a JS React wrapper for the library, such as react-spline.


```js
import Spline from '@splinetool/react-spline';

export default function App() {
  return (
    <div>
      <Spline scene="https://prod.spline.design/6Wq1Q7YGyM-iab9i/scene.splinecode" />
    </div>
  );
}
```

# Find The Component
There are two ways to find a component to wrap:

Write the component yourself locally.
Find a well-maintained React library on npm that contains the component you need.
In both cases, the process of wrapping the component is the same except for the library field.

Wrapping the Component
To start wrapping your React component, the first step is to create a new component in your Reflex app. This is done by creating a new class that inherits from rx.Component or rx.NoSSRComponent.

See the API Reference for more details on the rx.Component class.

This is when we will define the most important attributes of the component:

```
library: The name of the npm package that contains the component.

tag: The name of the component to import from the package.

alias: (Optional) The name of the alias to use for the component. This is useful if multiple component from different package have a name in common. If alias is not specified, tag will be used.

lib_dependencies: Any additional libraries needed to use the component.

is_default: (Optional) If the component is a default export from the module, set this to True. Default is False.

***Optionally, you can override the default component creation behavior by implementing the create class method. Most components won't need this when props are straightforward conversions from Python to JavaScript. However, this is useful when you need to add custom initialization logic, transform props, or handle special cases when the component is created.
```

When setting the `library` attribute, it is recommended to included a pinned version of the package. Doing so, the package will only change when you intentionally update the version, avoid unexpected breaking changes.

```python 
class MyBaseComponent(rx.Component):
    """MyBaseComponent."""

    # The name of the npm package.
    library = "my-library@x.y.z"

    # The name of the component to use from the package.
    tag = "MyComponent"

    # Any additional libraries needed to use the component.
    lib_dependencies: list[str] = ["package-deps@x.y.z"]

    # The name of the alias to use for the component.
    alias = "MyComponentAlias"

    # If the component is a default export from the module, set this to True.
    is_default = True / False

    @classmethod
    def create(cls, *children, **props):
        """Create an instance of MyBaseComponent.

        Args:
            *children: The children of the component.
            **props: The props of the component.

        Returns:
            The component instance.
        """
        # Your custom creation logic here
        return super().create(*children, **props)

```

# Wrapping a Dynamic Component

When wrapping some libraries, you may want to use dynamic imports. This is because they may not be compatible with Server-Side Rendering (SSR).

To handle this in Reflex, subclass `NoSSRComponent` when defining your component. It works the same as `rx.Component`, but it will automatically add the correct custom code for a dynamic import.

Often times when you see an import something like this:

```js
import dynamic from 'next/dynamic';

const MyLibraryComponent = dynamic(() => import('./MyLibraryComponent'), {
  ssr: false
});

```
You can wrap it in Reflex like this:
```python 
from reflex.components.component import NoSSRComponent


class MyLibraryComponent(NoSSRComponent):
    """A component that wraps a lib needing dynamic import."""

    library = "my-library@x.y.z"

    tag = "MyLibraryComponent"

```

It may not always be clear when a library requires dynamic imports. A few things to keep in mind are if the component is very client side heavy i.e. the view and structure depends on things that are fetched at run time, or if it uses `window` or `document` objects directly it will need to be wrapped as a `NoSSRComponent`.

**Some examples are:**

1. Video and Audio Players
2. Maps
3. Drawing Canvas
4. 3D Graphics
5. QR Scanners
6. Reactflow

The reason for this is that it does not make sense for your server to render these components as the server does not have access to your camera, it cannot draw on your canvas or render a video from a file.

In addition, if in the component documentation it mentions nextJS compatibility or server side rendering compatibility, it is a good sign that it requires dynamic imports.

# Advanced - Parsing a state Var with a JS Function
When wrapping a component, you may need to parse a state var by applying a JS function to it.


## Define the parsing function
First you need to define the parsing function by writing it in `add_custom_code`.

```python 
def add_custom_code(self) -> list[str]:
    """Add custom code to the component."""
    # Define the parsing function
    return [
        """
        function myParsingFunction(inputProp) {
            // Your parsing logic here
            return parsedProp;
        }"""
    ]

```

## Apply the parsing function to your props
Then, you can apply the parsing function to your props in the `create` method.


```python
from reflex.vars.base import Var
from reflex.vars.function import FunctionStringVar

    ...
    @classmethod
    def create(cls, *children, **props):
        """Create an instance of MyBaseComponent.
        
        Args:
            *children: The children of the component.
            **props: The props of the component.
            
        Returns:
            The component instance.
        """
        # Apply the parsing function to the props
        if (prop_to_parse := props.get("propsToParse")) is not None:
            if isinstance(prop_to_parse, Var):
                props["propsToParse"] = FunctionStringVar.create("myParsingFunction").call(prop_to_parse)
            else:
                # This is not a state Var, so you can parse the value directly in python
                parsed_prop = python_parsing_function(prop_to_parse)
                props["propsToParse"] = parsed_prop
        return super().create(*children, **props)
    ...

```

# Props 
When wrapping a React component, you want to define the props that will be accepted by the component. This is done by defining the props and annotating them with a rx.Var.

Broadly, there are three kinds of props you can encounter when wrapping a React component:

**Simple Props:** These are props that are passed directly to the component. They can be of any type, including strings, numbers, booleans, and even lists or dictionaries.
**Callback Props:** These are props that expect to receive a function. That function will usually be called by the component as a callback. (This is different from event handlers.)
**Component Props:** These are props that expect to receive a components themselves. They can be used to create more complex components by composing them together.
**Event Handlers:** These are props that expect to receive a function that will be called when an event occurs. They are defined as rx.EventHandler with a signature function to define the spec of the event.


## Simple Props
Simple props are the most common type of props you will encounter when wrapping a React component. They are passed directly to the component and can be of any type (but most commonly strings, numbers, booleans, and structures).

For custom types, you can use TypedDict to define the structure of the custom types. However, if you need the attributes to be automatically converted to camelCase once compiled in JS, you can use rx.PropsBase instead of TypedDict.

```python 
class CustomReactType(TypedDict):
    """Custom React type."""

    # Define the structure of the custom type to match the Javascript structure.
    attribute1: str
    attribute2: bool
    attribute3: int


class CustomReactType2(rx.PropsBase):
    """Custom React type."""

    # Define the structure of the custom type to match the Javascript structure.
    attr_foo: str  # will be attrFoo in JS
    attr_bar: bool  # will be attrBar in JS
    attr_baz: int  # will be attrBaz in JS


class SimplePropsComponent(MyBaseComponent):
    """MyComponent."""

    # Type the props according the component documentation.

    # props annotated as `string` in javascript
    prop1: rx.Var[str]

    # props annotated as `number` in javascript
    prop2: rx.Var[int]

    # props annotated as `boolean` in javascript
    prop3: rx.Var[bool]

    # props annotated as `string[]` in javascript
    prop4: rx.Var[list[str]]

    # props annotated as `CustomReactType` in javascript
    props5: rx.Var[CustomReactType]

    # props annotated as `CustomReactType2` in javascript
    props6: rx.Var[CustomReactType2]

    # Sometimes a props will accept multiple types. You can use `|` to specify the types.
    # props annotated as `string | boolean` in javascript
    props7: rx.Var[str | bool]

```

## Callback Props
Callback props are used to handle events or to pass data back to the parent component. They are defined as `rx.Var` with a type of `FunctionVar` or `Callable`.

```python 
from typing import Callable
from reflex.vars.function import FunctionVar


class CallbackPropsComponent(MyBaseComponent):
    """MyComponent."""

    # A callback prop that takes a single argument.
    callback_props: rx.Var[Callable]

```

## Component Props

Some components will occasionally accept other components as props, usually annotated as `ReactNode`. In Reflex, these are defined as `rx.Component`.

```python 
class ComponentPropsComponent(MyBaseComponent):
    """MyComponent."""

    # A prop that takes a component as an argument.
    component_props: rx.Var[rx.Component]

```

## Event Handlers
Event handlers are props that expect to receive a function that will be called when an event occurs. They are defined as `rx.EventHandler` with a signature function to define the spec of the event.

```
from reflex.vars.event_handler import EventHandler
from reflex.vars.function import FunctionVar
from reflex.vars.object import ObjectVar


class InputEventType(TypedDict):
    """Input event type."""

    # Define the structure of the input event.
    foo: str
    bar: int


class OutputEventType(TypedDict):
    """Output event type."""

    # Define the structure of the output event.
    baz: str
    qux: int


def custom_spec1(
    event: ObjectVar[InputEventType],
) -> tuple[str, int]:
    """Custom event spec using ObjectVar with custom type as input and tuple as output."""
    return (
        event.foo.to(str),
        event.bar.to(int),
    )


def custom_spec2(
    event: ObjectVar[dict],
) -> tuple[Var[OutputEventType]]:
    """Custom event spec using ObjectVar with dict as input and custom type as output."""
    return Var.create(
        {
            "baz": event["foo"],
            "qux": event["bar"],
        },
    ).to(OutputEventType)


class EventHandlerComponent(MyBaseComponent):
    """MyComponent."""

    # An event handler that take no argument.
    on_event: rx.EventHandler[rx.event.no_args_event_spec]

    # An event handler that takes a single string argument.
    on_event_with_arg: rx.EventHandler[
        rx.event.passthrough_event_spec(str)
    ]

    # An event handler specialized for input events, accessing event.target.value from the event.
    on_input_change: rx.EventHandler[rx.event.input_event]

    # An event handler specialized for key events, accessing event.key from the event and provided modifiers (ctrl, alt, shift, meta).
    on_key_down: rx.EventHandler[rx.event.key_event]

    # An event handler that takes a custom spec. (Event handler must expect a tuple of two values [str and int])
    on_custom_event: rx.EventHandler[custom_spec1]

    # Another event handler that takes a custom spec. (Event handler must expect a tuple of one value, being a OutputEventType)
    on_custom_event2: rx.EventHandler[custom_spec2]

```

Custom event specs have a few use case where they are particularly useful. If the event returns non-serializable data, you can filter them out so the event can be sent to the backend. You can also use them to transform the data before sending it to the backend.

When wrapping a React component, you may need to define custom code or hooks that are specific to the component. This is done by defining the `add_custom_code`or `add_hooks` methods in your component class.

## Custom Code

Custom code is any JS code that need to be included in your page, but not necessarily in the component itself. This can include things like CSS styles, JS libraries, or any other code that needs to be included in the page.

```python 
class CustomCodeComponent(MyBaseComponent):
    """MyComponent."""

    def add_custom_code(self) -> list[str]:
        """Add custom code to the component."""
        code1 = """const customVariable = "Custom code1";"""
        code2 = """console.log(customVariable);"""

        return [code1, code2]

```

The above example will render the following JS code in the page:

```python 
/* import here */

const customVariable = "Custom code1";
console.log(customVariable);

/* rest of the page code */

```

## Custom Hooks

Custom hooks are any hooks that need to be included in your component. This can include things like `useEffect`, `useState`, or any other hooks from the library you are wrapping.

- Simple hooks can be added as strings.
- More complex hooks that need to have special import or be written in a specific order can be added as `rx.Var` with a `VarData` object to specify the position of the hook.
    - The `imports` attribute of the `VarData` object can be used to specify any imports that need to be included in the component.
    - The `position` attribute of the `VarData` object can be set to `Hooks.HookPosition.PRE_TRIGGER` or `Hooks.HookPosition.POST_TRIGGER` to specify the position of the hook in the component.


**The position attribute is only used for hooks that need to be written in a specific order.**

If an event handler need to refer to a variable defined in a hook, the hook should be written before the event handler.
If a hook need to refer to the memoized event handler by name, the hook should be written after the event handler.

```python
from reflex.vars.base import Var, VarData
from reflex.constants import Hooks
from reflex.components.el.elements import Div

class ComponentWithHooks(Div, MyBaseComponent):
    """MyComponent."""

    def add_hooks(self) -> list[str| Var]:
        """Add hooks to the component."""
        hooks = []
        hooks1 = """const customHookVariable = "some value";"""
        hooks.append(hooks1)

        # A hook that need to be written before the memoized event handlers.
        hooks2 = Var(
            """useEffect(() => {
                console.log("PreTrigger: " + customHookVariable);
            }, []);
            """,
            _var_data=VarData(
                imports={"react": ["useEffect"],\},
                position=Hooks.HookPosition.PRE_TRIGGER
            ),
        )
        hooks.append(hooks2)

        hooks3 = Var(
            """useEffect(() => {
                console.log("PostTrigger: " + customHookVariable);
            }, []);
            """,
            _var_data=VarData(
                imports={"react": ["useEffect"],\},
                position=Hooks.HookPosition.POST_TRIGGER
            ),
        )
        hooks.append(hooks3)
        return hooks

```

The `ComponentWithHooks` will be rendered in the component in the following way:

```python
export function Div_7178f430b7b371af8a12d8265d65ab9b() {
  const customHookVariable = "some value";

  useEffect(() => {
    console.log("PreTrigger: " + customHookVariable);
  }, []);

  /* memoized triggers such as on_click, on_change, etc will render here */

  useEffect(() => {
    console.log("PostTrigger: "+ customHookVariable);
  }, []);

  return jsx("div", {\});
}

```

You can mix custom code and hooks in the same component. Hooks can access a variable defined in the custom code, but custom code cannot access a variable defined in a hook.

# Styles and Imports

When wrapping a React component, you may need to define styles and imports that are specific to the component. This is done by defining the `add_styles` and `add_imports` methods in your component class.

### Imports
Sometimes, the component you are wrapping will need to import other components or libraries. This is done by defining the `add_imports` method in your component class. That method should return a dictionary of imports, where the keys are the names of the packages to import and the values are the names of the components or libraries to import.

Values can be either a string or a list of strings. If the import needs to be aliased, you can use the `ImportVar` object to specify the alias and whether the import should be installed as a dependency.

```python 
from reflex.utils.imports import ImportVar


class ComponentWithImports(MyBaseComponent):
    def add_imports(self):
        """Add imports to the component."""
        return {
            # If you only have one import, you can use a string.
            "my-package1": "my-import1",
            # If you have multiple imports, you can pass a list.
            "my-package2": ["my-import2"],
            # If you need to control the import in a more detailed way, you can use an ImportVar object.
            "my-package3": ImportVar(
                tag="my-import3",
                alias="my-alias",
                install=False,
                is_default=False,
            ),
            # To import a CSS file, pass the full path to the file, and use an empty string as the key.
            "": "my-package-with-css/styles.css",
        }

```

The tag and library of the component will be automatically added to the imports. They do not need to be added again in `add_imports`


### Styles
Styles are any CSS styles that need to be included in the component. The style will be added inline to the component, so you can use any CSS styles that are valid in React.

```python 
class StyledComponent(MyBaseComponent):
    """MyComponent."""

    def add_style(self) -> dict[str, Any] | None:
        """Add styles to the component."""

        return rx.Style(
            {
                "backgroundColor": "red",
                "color": "white",
                "padding": "10px",
            }
        )

```
# Assets

If a wrapped component depends on assets such as images, scripts, or stylesheets, these can be kept adjacent to the component code and included in the final build using the `rx.asset` function.

`rx.asset` returns a relative path that references the asset in the compiled output. The target files are copied into a subdirectory of `assets/external` based on the module where they are initially used. This allows third-party components to have external assets with the same name without conflicting with each other.

For example, if there is an SVG file named `wave.svg` in the same directory as this component, it can be rendered using `rx.image` and `rx.asset`.

```
class Hello(rx.Component):
    @classmethod
    def create(cls, *children, **props) -> rx.Component:
        props.setdefault("align", "center")
        return rx.hstack(
            rx.image(
                src=rx.asset("wave.svg", shared=True),
                width="50px",
                height="50px",
            ),
            rx.heading("Hello ", *children),
            **props
        )

```

# Local Components
You can also wrap components that you have written yourself. For local components (when the code source is directly in the project), we recommend putting it beside the files that is wrapping it.

You can then use the path obtained via `rx.asset` to reference the component path.

So if you create a file called `hello.js` in the same directory as the component with this content:

```python 
// /path/to/components/hello.js
import React from 'react';

export function Hello() {
  return (
    <div>
      <h1>Hello!</h1>
    </div>
  )
  
```

You can specify the library as follow (note: we use the `public` directory here instead of `assets` as this is the directory that is served by the web server):

```python 
import reflex as rx

hello_path = rx.asset("hello.js", shared=True)
public_hello_path = "$/public/" + hello_path


class Hello(rx.Component):
    # Use an absolute path starting with $/public
    library = public_hello_path

    # Define everything else as normal.
    tag = "Hello"

```


# Local Packages
If the component is part of a local package, available on Github, or downloadable via a web URL, it can also be wrapped in Reflex. Specify the path or URL after an `@` following the package name.

Any local paths are relative to the `.web` folder, so you can use `../` prefix to reference the Reflex project root.

Some examples of valid specifiers for a package called [`@masenf/hello-react`](https://github.com/masenf/hello-react) are:

- GitHub: `@masenf/hello-react@github:masenf/hello-react`
- URL: `@masenf/hello-react@https://github.com/masenf/hello-react/archive/refs/heads/main.tar.gz`
- Local Archive: `@masenf/hello-react@../hello-react.tgz`
- Local Directory: `@masenf/hello-react@../hello-react`

It is important that the package name matches the name in `package.json` so Reflex can generate the correct import statement in the generated javascript code.

These package specifiers can be used for `library` or `lib_dependencies`.


CODE
```python 
class GithubComponent(rx.Component):
    library = (
        "@masenf/hello-react@github:masenf/hello-react"
    )
    tag = "Counter"

    def add_imports(self):
        return {"": ["@masenf/hello-react/dist/style.css"]}


def github_component_example():
    return GithubComponent.create()
```

UI OUTPUT 
![[Pasted image 20250710205737.png]]

Although more complicated, this approach is useful when the local components have additional dependencies or build steps required to prepare the component for use.

Some important notes regarding this approach:

- The repo or archive must contain a `package.json` file.
- `prepare` or `build` scripts will NOT be executed. The distribution archive, directory, or repo must already contain the built javascript files (this is common).

### Ensure CSS files are exported in `package.json`

In addition to exporting the module containing the component, any CSS files intended to be imported by the wrapped component must also be listed in the `exports` key of `package.json`.

```
{
  // ...,
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "require": "./dist/index.umd.cjs"
    },
    "./dist/style.css": {
      "import": "./dist/style.css",
      "require": "./dist/style.css"
    }
  },
  // ...
}
```
# Serializers
Vars can be any type that can be serialized to JSON. This includes primitive types like strings, numbers, and booleans, as well as more complex types like lists, dictionaries, and dataframes.

In case you need to serialize a more complex type, you can use the `serializer` decorator to convert the type to a primitive type that can be stored in the state. Just define a method that takes the complex type as an argument and returns a primitive type. We use type annotations to determine the type that you want to serialize.

For example, the Plotly component serializes a plotly figure into a JSON string that can be stored in the state.

```python 
import json
import reflex as rx
from plotly.graph_objects import Figure
from plotly.io import to_json


# Use the serializer decorator to convert the figure to a JSON string.
# Specify the type of the argument as an annotation.
@rx.serializer
def serialize_figure(figure: Figure) -> list:
    # Use Plotly's to_json method to convert the figure to a JSON string.
    return json.loads(to_json(figure))["data"]

```

We can then define a var of this type as a prop in our component.

```python 
import reflex as rx
from plotly.graph_objects import Figure


class Plotly(rx.Component):
    """Display a plotly graph."""

    library = "react-plotly.js@2.6.0"
    lib_dependencies: List[str] = ["plotly.js@2.22.0"]

    tag = "Plot"

    is_default = True

    # Since a serialize is defined now, we can use the Figure type directly.
    data: rx.Var[Figure]
```

# Complex Example

In this more complex example we will be wrapping `reactflow` a library for building node based applications like flow charts, diagrams, graphs, etc.

## Import

Lets start by importing the library [reactflow](https://www.npmjs.com/package/reactflow). Lets make a separate file called `reactflow.py` and add the following code:

```
import refex as rx
from typing import Any, Dict, List, Union


class ReactFlowLib(rx.Component):
    """A component that wraps a react flow lib."""

    library = "reactflow"

    def _get_custom_code(self) -> str:
        return """import 'reactflow/dist/style.css';
        """
```

Notice we also use the `_get_custom_code` method to import the css file that is needed for the styling of the library.

## Components
For this tutorial we will wrap three components from Reactflow: `ReactFlow`, `Background`, and `Controls`. Lets start with the `ReactFlow` component.

Here we will define the `tag` and the `vars` that we will need to use the component.

For this tutorial we will define `EventHandler` props `on_nodes_change` and `on_connect`, but you can find all the events that the component triggers in the [reactflow docs](https://reactflow.dev/docs/api/react-flow-props/#onnodeschange).

```python 
import reflex as rx
from typing import Any, Dict, List, Union


class ReactFlowLib(rx.Component): ...


class ReactFlow(ReactFlowLib):

    tag = "ReactFlow"

    nodes: rx.Var[List[Dict[str, Any]]]

    edges: rx.Var[List[Dict[str, Any]]]

    fit_view: rx.Var[bool]

    nodes_draggable: rx.Var[bool]

    nodes_connectable: rx.Var[bool]

    nodes_focusable: rx.Var[bool]

    on_nodes_change: rx.EventHandler[lambda e0: [e0]]

    on_connect: rx.EventHandler[lambda e0: [e0]]
```

Now lets add the `Background` and `Controls` components. We will also create the components using the `create` method so that we can use them in our app.

```python 
import reflex as rx
from typing import Any, Dict, List, Union


class ReactFlowLib(rx.Component): ...


class ReactFlow(ReactFlowLib): ...


class Background(ReactFlowLib):

    tag = "Background"

    color: rx.Var[str]

    gap: rx.Var[int]

    size: rx.Var[int]

    variant: rx.Var[str]


class Controls(ReactFlowLib):

    tag = "Controls"


react_flow = ReactFlow.create
background = Background.create
controls = Controls.create

```

## Building the App
Now that we have our components lets build the app.

Lets start by defining the initial nodes and edges that we will use in our app.

```python 
import reflex as rx
from .react_flow import react_flow, background, controls
import random
from collections import defaultdict
from typing import Any, Dict, List


initial_nodes = [
    \{
        'id': '1',
        'type': 'input',
        'data': {'label': '150'},
        'position': {'x': 250, 'y': 25},
    },
    \{
        'id': '2',
        'data': {'label': '25'},
        'position': {'x': 100, 'y': 125},
    },
    \{
        'id': '3',
        'type': 'output',
        'data': {'label': '5'},
        'position': {'x': 250, 'y': 250},
    },
]

initial_edges = [
    {'id': 'e1-2', 'source': '1', 'target': '2', 'label': '*', 'animated': True},
    {'id': 'e2-3', 'source': '2', 'target': '3', 'label': '+', 'animated': True},
]

```

Next we will define the state of our app. We have four event handlers: `add_random_node`, `clear_graph`, `on_connect` and `on_nodes_change`.

The `on_nodes_change` event handler is triggered when a node is selected and dragged. This function is used to update the position of a node during dragging. It takes a single argument `node_changes`, which is a list of dictionaries containing various types of metadata. For updating positions, the function specifically processes changes of type `position`.

```python 
class State(rx.State):
    """The app state."""
    nodes: List[Dict[str, Any]] = initial_nodes
    edges: List[Dict[str, Any]] = initial_edges

    @rx.event
    def add_random_node(self):
        new_node_id = f'{len(self.nodes) + 1\}'
        node_type = random.choice(['default'])
        # Label is random number
        label = new_node_id
        x = random.randint(0, 500)
        y = random.randint(0, 500)

        new_node = {
            'id': new_node_id,
            'type': node_type,
            'data': {'label': label},
            'position': {'x': x, 'y': y},
            'draggable': True,
        }
        self.nodes.append(new_node)

    @rx.event
    def clear_graph(self):
        self.nodes = []  # Clear the nodes list
        self.edges = []  # Clear the edges list

    @rx.event
    def on_connect(self, new_edge):
        # Iterate over the existing edges
        for i, edge in enumerate(self.edges):
            # If we find an edge with the same ID as the new edge
            if edge["id"] == f"e{new_edge['source']}-{new_edge['target']}":
                # Delete the existing edge
                del self.edges[i]
                break

        # Add the new edge
        self.edges.append({
            "id": f"e{new_edge['source']}-{new_edge['target']}",
            "source": new_edge["source"],
            "target": new_edge["target"],
            "label": random.choice(["+", "-", "*", "/"]),
            "animated": True,
        })

    @rx.event
    def on_nodes_change(self, node_changes: List[Dict[str, Any]]):
        # Receives a list of Nodes in case of events like dragging
        map_id_to_new_position = defaultdict(dict)

        # Loop over the changes and store the new position
        for change in node_changes:
            if change["type"] == "position" and change.get("dragging") == True:
                map_id_to_new_position[change["id"]] = change["position"]

        # Loop over the nodes and update the position
        for i, node in enumerate(self.nodes):
            if node["id"] in map_id_to_new_position:
                new_position = map_id_to_new_position[node["id"]]
                self.nodes[i]["position"] = new_position

```

Now lets define the UI of our app. We will use the `react_flow` component and pass in the `nodes` and `edges` from our state. We will also add the `on_connect` event handler to the `react_flow` component to handle when an edge is connected.


```python 
def index() -> rx.Component:
    return rx.vstack(
        react_flow(
            background(),
            controls(),
            nodes_draggable=True,
            nodes_connectable=True,
            on_connect=lambda e0: State.on_connect(e0),
            on_nodes_change=lambda e0: State.on_nodes_change(
                e0
            ),
            nodes=State.nodes,
            edges=State.edges,
            fit_view=True,
        ),
        rx.hstack(
            rx.button(
                "Clear graph",
                on_click=State.clear_graph,
                width="100%",
            ),
            rx.button(
                "Add node",
                on_click=State.add_random_node,
                width="100%",
            ),
            width="100%",
        ),
        height="30em",
        width="100%",
    )


# Add state and page to the app.
app = rx.App()
app.add_page(index)

```

![[Pasted image 20250710210257.png]]

[

# More React Libraries
## AG Charts

Here we wrap the AG Charts library from the NPM package [ag-charts-react](https://www.npmjs.com/package/ag-charts-react).

In the react code below we can see the first `2` lines are importing React and ReactDOM, and this can be ignored when wrapping your component.

We import the `AgCharts` component from the `ag-charts-react` library on line 5. In Reflex this is wrapped by `library = "ag-charts-react"` and `tag = "AgCharts"`.

Line `7` defines a functional React component, which on line `26` returns `AgCharts` which is similar in the Reflex code to using the `chart` component.

Line `9` uses the `useState` hook to create a state variable `chartOptions` and its setter function `setChartOptions` (equivalent to the event handler `set_chart_options` in reflex). The initial state variable is of type dict and has two key value pairs `data` and `series`.

When we see `useState` in React code, it correlates to state variables in your State. As you can see in our Reflex code we have a state variable `chart_options` which is a dictionary, like in our React code.

Moving to line `26` we see that the `AgCharts` has a prop `options`. In order to use this in Reflex we must wrap this prop. We do this with `options: rx.Var[dict]` in the `AgCharts` component.

Lines `31` and `32` are rendering the component inside the root element. This can be ignored when we are wrapping a component as it is done in Reflex by creating an `index` function and adding it to the app.

```
1 | import React, { useState } from 'react';
2 | import ReactDOM from 'react-dom/client';
3 | 
4 | // React Chart Component
5 | import { AgCharts } from 'ag-charts-react';
6 | 
7 | const ChartExample = () => {
8 |     // Chart Options: Control & configure the chart
9 |     const [chartOptions, setChartOptions] = useState({
10|         // Data: Data to be displayed in the chart
11|         data: [
12|             { month: 'Jan', avgTemp: 2.3, iceCreamSales: 162000 },
13|             { month: 'Mar', avgTemp: 6.3, iceCreamSales: 302000 },
14|             { month: 'May', avgTemp: 16.2, iceCreamSales: 800000 },
15|             { month: 'Jul', avgTemp: 22.8, iceCreamSales: 1254000 },
16|             { month: 'Sep', avgTemp: 14.5, iceCreamSales: 950000 },
17|             { month: 'Nov', avgTemp: 8.9, iceCreamSales: 200000 },
18|         ],
19|         // Series: Defines which chart type and data to use
20|         series: [{ type: 'bar', xKey: 'month', yKey: 'iceCreamSales' }],
21|     });
22| 
23|     // React Chart Component
24|     return (
25|         // AgCharts component with options passed as prop
26|         <AgCharts options={chartOptions} />
27|     );
28| }
29| 
30| // Render component inside root element
31| const root = ReactDOM.createRoot(document.getElementById('root'));
32| root.render(<ChartExample />);
```

## React Leaflet

In this example we are wrapping the React Leaflet library from the NPM package [react-leaflet](https://www.npmjs.com/package/react-leaflet).

On line `1` we import the `dynamic` function from Next.js and on line `21` we set `ssr: false`. Lines `4` and `6` use the `dynamic` function to import the `MapContainer` and `TileLayer` components from the `react-leaflet` library. This is used to dynamically import the `MapContainer` and `TileLayer` components from the `react-leaflet` library. This is done in Reflex by using the `NoSSRComponent` class when defining the component. There is more information of when this is needed on the `Dynamic Imports` section of this [page](https://reflex.dev/docs/wrapping-react/library-and-tags/).

It mentions in the documentation that it is necessary to include the Leaflet CSS file, which is added on line `2` in the React code below. This can be done in Reflex by using the `add_imports` method in the `MapContainer` component. We can add a relative path from within the React library or a full URL to the CSS file.

Line `4` defines a functional React component, which on line `8` returns the `MapContainer` which is done in the Reflex code using the `map_container` component.

The `MapContainer` component has props `center`, `zoom`, `scrollWheelZoom`, which we wrap in the `MapContainer` component in the Reflex code. We ignore the `style` prop as it is a reserved name in Reflex. We can use the `rename_props` method to change the name of the prop, as we will see in the React PDF Renderer example, but in this case we just ignore it and add the `width` and `height` props as css in Reflex.

The `TileLayer` component has a prop `url` which we wrap in the `TileLayer` component in the Reflex code.

Lines `24` and `25` defines and exports a React functional component named `Home` which returns the `MapComponent` component. This can be ignored in the Reflex code when wrapping the component as we return the `map_container` component in the `index` function.


```python 
1 | import dynamic from "next/dynamic";
2 | import "leaflet/dist/leaflet.css";
3 | 
4 | const MapComponent = dynamic(
5 |   () => {
6 |     return import("react-leaflet").then(({ MapContainer, TileLayer }) => {
7 |       return () => (
8 |         <MapContainer
9 |           center={[51.505, -0.09]}
10|           zoom={13}
11|           scrollWheelZoom={true}
12|           style=\{{ height: "50vh", width: "100%" }}
13|        >
14|          <TileLayer
15|            url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
16|          />
17|        </MapContainer>
18|      );
19|    });
20|  },
21|  { ssr: false }
22| );
23|
24| export default function Home() {
25|   return <MapComponent />;
26| }
```
## React PDF Renderer
---
In this example we are wrapping the React renderer for creating PDF files on the browser and server from the NPM package [@react-pdf/renderer](https://www.npmjs.com/package/@react-pdf/renderer).

This example is similar to the previous examples, and again Dynamic Imports are required for this library. This is done in Reflex by using the `NoSSRComponent` class when defining the component. There is more information on why this is needed on the `Dynamic Imports` section of this [page](https://reflex.dev/docs/wrapping-react/library-and-tags/).

The main difference with this example is that the `style` prop, used on lines `20`, `21` and `24` in React code, is a reserved name in Reflex so can not be wrapped. A different name must be used when wrapping this prop and then this name must be changed back to the original with the `rename_props` method. In this example we name the prop `theme` in our Reflex code and then change it back to `style` with the `rename_props` method in both the `Page` and `View` components.

```python 
1 | import ReactDOM from 'react-dom';
2 | import { Document, Page, Text, View, StyleSheet, PDFViewer } from '@react-pdf/renderer';
3 |
4 | // Create styles
5 | const styles = StyleSheet.create({
6 |   page: {
7 |     flexDirection: 'row',
8 |     backgroundColor: '#E4E4E4',
9 |   },
10|   section: {
11|     margin: 10,
12|    padding: 10,
13|     flexGrow: 1,
14|   },
15| });
16|
17| // Create Document Component
18| const MyDocument = () => (
19|   <Document>
20|     <Page size="A4" style={styles.page}>
21|       <View style={styles.section}>
22|         <Text>Section #1</Text>
23|       </View>
24|       <View style={styles.section}>
25|         <Text>Section #2</Text>
26|       </View>
27|     </Page>
28|   </Document>
29| );
30| 
31| const App = () => (
32|   <PDFViewer>
33|     <MyDocument />
34|   </PDFViewer>
35| );
36| 
37| ReactDOM.render(<App />, document.getElementById('root'));
```