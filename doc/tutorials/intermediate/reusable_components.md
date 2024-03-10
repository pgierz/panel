# Reusable Components

In this guide, we will explore how to structure our code to make it easily reusable and avoid callback hell:

- Write `Parameterized` and `Viewer` classes that encapsulate multiple components.

## Writing Parameterized Classes

When creating larger Panel projects, we recommend using classes. This approach is useful for several reasons:

1. Organizing complex pieces of code and functionality
2. Creating reusable components composed of multiple Panel objects

A Parameterized class must inherit from `param.Parameterized` and should declare one or more parameters. Here, we will start building a `DataExplorer` by declaring two parameters:

1. `data`: Accepts a DataFrame
2. `page_size`: Controls the page size

```{pyodide}
import pandas as pd
import panel as pn
import param

pn.extension("tabulator")

class DataExplorer(param.Parameterized):
    data = param.DataFrame(doc="Stores a DataFrame to explore")

    page_size = param.Integer(
        default=10, doc="Number of rows per page.", bounds=(1, 20)
    )

data_url = "https://assets.holoviz.org/panel/tutorials/turbines.csv.gz"
df = pd.read_csv(data_url)

explorer = DataExplorer(data=df, page_size=5)
```

This explorer doesn't do anything yet, so let's learn how we can turn the UI-agnostic parameter declarations into a UI. For that purpose, we will learn about `pn.Param`.

`pn.Param` allows mapping parameter declarations to widgets that allow editing the parameter value. There is a default mapping from `Parameter` type to the appropriate type, but as long as the input matches, this can be overridden.

Let's start with the simplest case:

```{pyodide}
pn.Param(explorer.param, widgets={"page_size": pn.widgets.IntInput}).servable()
```

Notice that each parameter was mapped to a widget appropriate for editing its value, i.e., the `data` was mapped to a `Tabulator` widget, and the `page_size` was mapped to an `IntInput` widget.

If you try playing with the `page_size` widget, you will notice that it doesn't actually do anything.

So next, let's explicitly map the parameter to a widget using the `Widget.from_param` method. This will also let us provide additional options, e.g., to provide `start` and `end` values for the slider and layout options for the table.

```{pyodide}
pn.Column(
    pn.widgets.IntSlider.from_param(explorer.param.page_size, start=5, end=20, step=5),
    pn.widgets.Tabulator.from_param(explorer.param.data, page_size=explorer.param.page_size, sizing_mode='stretch_width')
).servable()
```

## Creating Reusable Viewer Components

The whole point of using classes is to encapsulate the logic in them, so let's do that. For that, we can use a slight extension of the `Parameterized` class that makes the object behave as if it were a regular Panel object. The `Viewer` class does exactly that; all you have to do is implement the `__panel__` method:

```{pyodide}
import pandas as pd
import panel as pn
import param

pn.extension("tabulator")


class DataExplorer(pn.viewable.Viewer):

    data = param.DataFrame(doc="Stores a DataFrame to explore")
    page_size = param.Integer(default=10, doc="Number of rows per page.", bounds=(1, None))

    def __panel__(self):
        return pn.Column(
            pn.widgets.IntSlider.from_param(self.param.page_size, start=5, end=25, step=5),
            pn.widgets.Tabulator.from_param(self.param.data, page_size=self.param.page_size, sizing_mode='stretch_width')
        )

data_url = "https://assets.holoviz.org/panel/tutorials/turbines.csv.gz"
df = pd.read_csv(data_url)

DataExplorer(data=df).servable()
```

### Exercise: Extend the DataExplorer

Extend the `DataExplorer` class by adding parameters to control the Tabulator `theme` and toggling the `show_index` option

:::{dropdown} Solution

```{pyodide}
import pandas as pd
import panel as pn
import param
from panel.widgets import IntSlider, Tabulator

pn.extension("tabulator")


class DataExplorer(pn.viewable.Viewer):
    data = param.DataFrame(doc="Stores a DataFrame to explore")
    page_size = param.Integer(
        default=10, doc="Number of rows per page.", bounds=(1, None)
    )
    theme = param.Selector(
        default="simple",
        objects=["simple", "default", "site", "midnight"],
    )
    show_index = param.Boolean(
        default=True, doc="Whether or not to display the index of the data"
    )

    def __panel__(self):
        return pn.Column(
            IntSlider.from_param(self.param.page_size, start=5, end=25, step=5),
            self.param.theme,
            self.param.show_index,
            Tabulator.from_param(
                self.param.data,
                page_size=self.param.page_size,
                sizing_mode="stretch_width",
                theme=self.param.theme,
                show_index=self.param.show_index,
            ),
        )


data_url = "https://assets.holoviz.org/panel/tutorials/turbines.csv.gz"
df = pd.read_csv(data_url)

DataExplorer(data=df).servable()
```

:::

## Recap

We have learned how to structure our code to make it easily reusable and avoid callback hell.

We should now be able to write `Parameterized` and `Viewer` classes that encapsulate multiple components.

## Resources

### How-To

- [Combine Existing Components](../../how_to/custom_components/custom_viewer.ipynb)
  - [Plot Viewer](../../how_to/custom_components/examples/plot_viewer.md)
  - [Table Viewer](../../how_to/custom_components/examples/table_viewer.md)
- [Generate UI from Parameters](../../how_to/param/index.md)