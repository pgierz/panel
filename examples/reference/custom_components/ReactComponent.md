# `ReactComponent`

`ReactComponent` simplifies the creation of custom Panel components using [React](https://react.dev/).

```python
import panel as pn
import param

from panel.custom import ReactComponent

pn.extension()

class CounterButton(ReactComponent):

    value = param.Integer()

    _esm = """
    function App(props) {
        const [value, setValue] = props.state.value;
        return (
            <button onClick={e => setValue(value+1)}>
            count is {value}
            </button>
        );
    }

    export function render({state}) {
        return <App state={state}/>;
    }
    """

CounterButton().servable()
```

:::{note}

`ReactComponent` extends the [`JSComponent`](JSComponent.md) class, which allows you to create custom Panel components using JavaScript.

`ReactComponent` bears similarities to [`AnyWidget`](https://anywidget.dev/) and [`IpyReact`](https://github.com/widgetti/ipyreact), but `ReactComponent` is specifically optimized for use with Panel and React.

:::

## API

### ReactComponent Attributes

- **`_esm`** (str | PurePath): This attribute accepts either a string or a path that points to an [ECMAScript module](https://nodejs.org/api/esm.html#modules-ecmascript-modules). The ECMAScript module should export a `render` function which returns the HTML element to display. In a development environment such as a notebook or when using `--autoreload`, the module will automatically reload upon saving changes. You can use [`JSX`](https://react.dev/learn/writing-markup-with-jsx) and [`TypeScript`](https://www.typescriptlang.org/). The `_esm` script is transpiled on the fly using [Sucrase](https://sucrase.io/).
- **`_import_map`** (dict): This dictionary defines an [import map](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/importmap), allowing you to customize how module specifiers are resolved.
- **`_stylesheets`** (List[str | PurePath] | None): This optional attribute accepts a list of CSS strings or paths to CSS files. It supports automatic reloading in development environments.

#### `render` Function

The `_esm` attribute must export the `render` function. It accepts the following parameters:

- **`state`**: Manages non-Viewable Parameter state similar to React's [`useState`](https://www.w3schools.com/react/react_usestate.asp) hook. The `render` function is rerun if a Parameter changes.
- **`data`**: Represents the Viewable Parameters of the component and provides methods to `.watch` for changes and `.send_event` back to Python.
- **`children`**: Represents the Viewable Parameters of the component. The `render` function is rerun if a child changes.
- **`model`**: The Bokeh model.
- **`view`**: The Bokeh view.
- **`el`**: The HTML element that the component will be rendered into.

Any HTML element returned from the `render` function will be appended to the HTML element (`el`) of the component.

#### Other Lifecycle Methods

DUMMY CONTENT. PLEASE HELP ME DESCRIBE THIS.

- `initialize`: Runs once per widget instance at model initialization, facilitating setup for event handlers or state.
- `teardown`: Cleans up resources or processes when a widget instance is being removed.

## Usage

### Styling with CSS

Include CSS within the `_stylesheets` attribute to style the component. The CSS is injected directly into the component's HTML.

```python
import panel as pn
import param

from panel.custom import ReactComponent

pn.extension()

class CounterButton(ReactComponent):

    value = param.Integer()

    _stylesheets = [
        """
        button {
            background: #0072B5;
            color: white;
            border: none;
            padding: 10px;
            border-radius: 4px;
        }
        button:hover {
            background: #4099da;
        }
        """
    ]

    _esm = """
    function App(props) {
        const [value, setValue] = props.state.value;
        return (
            <button onClick={e => setValue(value+1)}>
            count is {value}
            </button>
        );
    }

    export function render({state}) {
        return <App state={state}/>;
    }
    """

CounterButton().servable()
```

## Send Events from JavaScript to Python

Events from JavaScript can be sent to Python using the `data.send_event` method. Define a handler in Python to manage these events.

```python
import panel as pn
import param

from panel.custom import ReactComponent

pn.extension()

class ButtonEventExample(ReactComponent):

    value = param.Parameter()

    _esm = """
    function App(props) {
        return (
            <button onClick={e => props.data.send_event('click', e) }>
            Click me
            </button>
        );
    }

    export function render({data}) {
        return <App data={data}/>;
    }
    """

    def _handle_click(self, event):
       self.value = str(event.__dict__)

button = ButtonEventExample()
pn.Column(
    button, pn.widgets.TextAreaInput(value=button.param.value, height=200),
).servable()
```

You can also define and send your own custom events:

```python
import panel as pn
import param

from panel.custom import ReactComponent

pn.extension()

class CustomEventExample(ReactComponent):

    value = param.Parameter()

    _esm = """
    function send_event(data) {
        const currentDate = new Date();
        const custom_event = new CustomEvent("click", { detail: currentDate.getTime() });
        data.send_event('click', custom_event)
    }

    function App(props) {
        return (
            <button onClick={e => send_event(props.data)}>
            Click me
            </button>
        );
    }

    export function render({data}) {
        return <App data={data}/>;
    }
    """

def _handle_click(self, event):
    self.value = str(event.__dict__)

button = CustomEventExample()
pn.Column(
    button, pn.widgets.TextAreaInput(value=button.param.value, height=200),
).servable()
```

## Dependency Imports

JavaScript dependencies can be directly imported via URLs, such as those from [`esm.sh`](https://esm.sh/).

```python
import panel as pn

from panel.custom import ReactComponent

pn.extension()

class ConfettiButton(ReactComponent):

    _esm = """
    import confetti from "https://esm.sh/canvas-confetti@1.6.0";

    export function render() {
        return (
            <button onClick={e => confetti()}>
            Click Me
            </button>
        );
    }
    """

ConfettiButton().servable()
```

Use the `_import_map` attribute for more concise module references.

```python
import panel as pn

from panel.custom import ReactComponent

pn.extension()

class ConfettiButton(ReactComponent):
    _importmap = {
        "imports": {
            "canvas-confetti": "https://esm.sh/canvas-confetti@1.6.0",
        }
    }

    _esm = """
    import confetti from "canvas-confetti";

    export function render() {
        return (
            <button onClick={e => confetti()}>
            Click Me
            </button>
        );
    }
    """

ConfettiButton().servable()
```

See [import map](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/importmap) for more info.

## External Files

You can load JSX and CSS from files by providing the paths to these files.

Create the file **counter_button.py**.

```python
from pathlib import Path

import param
import panel as pn

from panel.custom import ReactComponent

pn.extension()

class CounterButton(ReactComponent):

    value = param.Integer()

    _esm = Path("counter_button.jsx")
    _stylesheets = [Path("counter_button.css")]

CounterButton().servable()
```

Now create the file **counter_button.jsx**.

```javascript
function App(props) {
    const [value, setValue] = props.state.value;
    return (
        <button onClick={e => setValue(value+1)}>
        count is {value}
        </button>
    );
}

export function render({state}) {
    return <App state={state}/>;
}
```

Now create the file **counter_button.css**.

```css
button {
    background: #0072B5;
    color: white;
    border: none;
    padding: 10px;
    border-radius: 4px;
}
button:hover {
    background: #4099da;
}
```

Serve the app with `panel serve counter_button.py --autoreload`.

You can now edit the JSX or CSS file, and the changes will be automatically reloaded.

- Try changing `count is {value}` to `COUNT IS {value}` and observe the update.
- Try changing the background color from `#0072B5` to `#008080`.

## Displaying A Single Panel Component

You can display Panel components by defining `ClassSelector` parameters with the `class_` set to subtype of `_Viewable` or tuple of subtypes of `_Viewable`s.

Lets start with the simplest example

```python
import panel as pn
import param

from panel.custom import ReactComponent

class Example(ReactComponent):

    child = param.ClassSelector(class_=pn.pane.Viewable)

    _esm = """
    export function render({ children }) {
      return (
        <button>
            {children.child}
        </button>
    )}
    """

Example(child=pn.panel("A **Markdown** pane!")).servable()
```

If you want to allow a certain type of Panel components only you can specify the specific type in the `class_` argument.

```python
import panel as pn
import param

from panel.custom import ReactComponent

class Example(ReactComponent):

    child = param.ClassSelector(class_=pn.pane.Markdown)

    _esm = """
    export function render({ children }) {
      return (
        <button>
            {children.child}
        </button>
    )}
    """

Example(child=pn.panel("A **Markdown** pane!")).servable()
```

The `class_` argument also supports a tuple of types:

```python
import param
import panel as pn

from panel.custom import ReactComponent

class Example(ReactComponent):

    child = param.ClassSelector(class_=(pn.pane.Markdown, pn.pane.HTML))

    _esm = """
    export function render({ children }) {
      return (
        <button>
            {children.child}
        </button>
    )}
    """

Example(child=pn.panel("A **Markdown** pane!")).servable()
```

## Displaying a List of Panel Components

You can also display a `List` of `Viewable` `objects`.

```python
import panel as pn
import param

from panel.custom import ReactComponent

class Example(ReactComponent):

    objects = param.List(item_type=pn.viewable.Viewable)

    _esm = """
    export function render({ children }) {
      return (
        <div>
        {children.objects}
        </div>
    );
    }"""


Example(
    objects=[pn.panel("A **Markdown** pane!"), pn.widgets.Button(name="Click me!")]
).servable()
```

:::note

You can change the `item_type` to a specific subtype of `Viewable` or a tuple of
`Viewable` subtypes.

:::