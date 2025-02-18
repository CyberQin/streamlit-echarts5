# Streamlit - ECharts5

This is a fork repo of [streamlit_echarts](https://share.streamlit.io/andfanilo/streamlit-echarts). Major changes to upstream:  
1. rename package to `streamlit-echarts5`, you need to rename all import namespaces`
1. bump echarts to v5.5.1
1. bump react to v18
1. bump `react-scripts` to v5.0.1
1. remove `React.useCallback` since it is out of rules of hooks,it not recommend in loop or conditional. use normal callback to events ,but with `React.memo(EchartsChart)` to avoid event failling. It's seems good for all demo now,but not tested yet when event callback and props really change  at the same time
1. `React.memo` component for performance, since `streamlit.setComponentValue` always rerender parent,and give a set props to a new object,but props not changed actually,React shallow compare treat a new object not equal to old one even their values are equal.eg`props = {...props}` will call rerender. No rerender in the two examples about events。
1. add notMerge param to func, mannually handle this,but with default value True
1. update python build system, use pyproject.toml to manage info, use pdm as venv manager.


**Since I Changed too much deps , versions, mechanism, it's not tested egnough and not easy to be accepted by origin Repo. I won't create pull request to origin repo recently,but publish to pypi as `streamlit-echarts5`.  
I won't frequently update this repository. If you have new feature request,try fork this repo when no response from issues**  


[![Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://share.streamlit.io/andfanilo/streamlit-echarts-demo/master/app.py)

A Streamlit component to display [ECharts](https://echarts.apache.org/en/index.html).

![](./img/demo.gif)

## Install

```shell script
pip install streamlit-echarts5
```

## Usage

This library provides 2 functions to display echarts :

- `st_echarts` to display charts from ECharts json options as Python dicts
- `st_pyecharts` to display charts from Pyecharts instances

Check out the [demo](https://share.streamlit.io/andfanilo/streamlit-echarts-demo/master/app.py) and [source code](https://github.com/andfanilo/streamlit-echarts-demo) for more examples.

**st_echarts example**

```python
from streamlit_echarts5 import st_echarts

options = {
    "xAxis": {
        "type": "category",
        "data": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
    },
    "yAxis": {"type": "value"},
    "series": [
        {"data": [820, 932, 901, 934, 1290, 1330, 1320], "type": "line"}
    ],
}
st_echarts(options=options)
```

**st_pyecharts example**

```python
from pyecharts import options as opts
from pyecharts.charts import Bar
from streamlit_echarts5 import st_pyecharts

b = (
    Bar()
    .add_xaxis(["Microsoft", "Amazon", "IBM", "Oracle", "Google", "Alibaba"])
    .add_yaxis(
        "2017-2018 Revenue in (billion $)", [21.2, 20.4, 10.3, 6.08, 4, 2.2]
    )
    .set_global_opts(
        title_opts=opts.TitleOpts(
            title="Top cloud providers 2018", subtitle="2017-2018 Revenue"
        ),
        toolbox_opts=opts.ToolboxOpts(),
    )
)
st_pyecharts(b)
```

## API

### st_echarts API

```
st_echarts(
    options: Dict
    theme: Union[str, Dict]
    events: Dict[str, str]
    notMerge: bool = True
    height: str
    width: str
    renderer: str
    map: Map
    key: str
)
```

- **options** : Python dictionary that resembles the JSON counterpart of
  [echarts options](https://echarts.apache.org/en/tutorial.html#ECharts%20Basic%20Concepts%20Overview).
  For example the basic line chart in JS :

```javascript
// JS code
option = {
  xAxis: {
    type: "category",
    data: ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
  },
  yAxis: { type: "value" },
  series: [{ data: [820, 932, 901, 934, 1290, 1330, 1320], type: "line" }],
};
```

is represented in Python :

```python
# Python code
option = {
    "xAxis": {
        "type": "category",
        "data": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
    },
    "yAxis": { "type": "value" },
    "series": [
        {"data": [820, 932, 901, 934, 1290, 1330, 1320], "type": "line" }
    ],
}
```

- **theme** : [echarts theme](https://echarts.apache.org/en/tutorial.html#Overview%20of%20Style%20Customization).
  You can specify built-int themes or pass over style configuration as a Python dict.
- **events** : Python dictionary which maps an [event](https://echarts.apache.org/en/tutorial.html#Events%20and%20Actions%20in%20ECharts) to a Js function as string.
  For example :

```python
{
    "click": "function(params) { console.log(params.name) }"
}
```

will get mapped to :

```javascript
myChart.on("click", function (params) {
  console.log(params.name);
});
```

Return values from events are sent back to Python, for example:

```python
option = {
    "xAxis": {
        "type": "category",
        "data": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
    },
    "yAxis": { "type": "value" },
    "series": [
        {"data": [820, 932, 901, 934, 1290, 1330, 1320], "type": "line" }
    ],
}
events = {
    "click": "function(params) { console.log(params.name); return params.name }",
    "dblclick":"function(params) { return [params.type, params.name, params.value] }"
}
value = st_echarts(option, events=events)
st.write(value)  # shows name on bar click and type+name+value on bar double click
```

The JS code needs to be a one-liner. You can use Javascript minifiers like https://javascript-minifier.com/ or https://www.minifier.org/ to transform your Javascript code to a one-liner.

- **height** / **width** : size of the div wrapper
- **map** : register a map using the dedicated `Map` class

```python
from streamlit_echarts5 import Map
with open("USA.json", "r") as f:
    map = Map(
        "USA",
        json.loads(f.read()),
        {
            "Alaska": {"left": -131, "top": 25, "width": 15},
            "Hawaii": {"left": -110, "top": 28, "width": 5},
            "Puerto Rico": {"left": -76, "top": 26, "width": 2},
        },
    )
options = {...}
st_echarts(options, map=map)
```

You'll find a lot of GeoJSON data inside the [source code of echarts-countries-js](https://github.com/echarts-maps/echarts-countries-js/tree/master/echarts-countries-js).

- **renderer** : SVG or canvas
- **key** : assign a fixed identity if you want to change its arguments over time and not have it be re-created.

### st_pyecharts API

```python
def st_pyecharts(
    chart: Base
    theme: Union[str, Dict]
    events: Dict[str, str]
    height: str
    width: str
    renderer: str
    map: Map
    key: str
)
```

- **chart** : Pyecharts instance

The docs for the remaining inputs are the same as its `st_echarts` counterpart.

## Development

### Install

- JS side

```shell script
cd frontend
npm install
```

- Python side

```shell script
conda create -n streamlit-echarts5 python=3.12
conda activate streamlit-echarts5
pip install -e .
```

### Run

Both webpack dev server and Streamlit need to run for development mode.

- JS side

```shell script
cd frontend
npm run dev
```

- Python side

Demo example is on https://github.com/andfanilo/streamlit-echarts-demo. But you need to change all import module from `streamlit_echarts` to `streamlit_echarts5`


```shell script
git clone https://github.com/andfanilo/streamlit-echarts-demo
cd streamlit-echarts-demo/
# After you replace the import module,then run next shell
streamlit run app.py
```

## build package
- Build frontend
```shell script
cd frontend
npm run build
```
- Build wheel
```shell script
# cd to project root
# ensure python env have 'build' installed
python -m build
```

## Caveats

### Theme definition

- Defining the theme in Pyecharts when instantiating chart like `Bar(init_opts=opts.InitOpts(theme=ThemeType.LIGHT))`
  does not work, you need to call theme in `st_pyecharts(c, theme=ThemeType.LIGHT)`.

### On Javascript functions

This library also provides the `JsCode` util class directly from `pyecharts`.

This class is used to indicate javascript code by wrapping it with a specific placeholder.
On the custom component side, we parse every value in options looking for this specific placeholder
to determine whether a value is a JS function.

As such, if you want to pass JS functions as strings in your options,
you should use the corresponding `JsCode` module to wrap code with this placeholder :

- In Python dicts representing the JSON option counterpart,
  wrap any JS string function with `streamlit_echarts.JsCode` by calling `JsCode(function).js_code`.
  It's a smaller version of `pyecharts.commons.utils.JsCode` so you don't need to install `pyecharts` to use it.

```
series: [
    {
        type: 'scatter', // this is scatter chart
        itemStyle: {
            opacity: 0.8
        },
        symbolSize: JsCode("function (val) { return val[2] * 40;}").js_code,
        data: [["14.616","7.241","0.896"],["3.958","5.701","0.955"],["2.768","8.971","0.669"],["9.051","9.710","0.171"],["14.046","4.182","0.536"],["12.295","1.429","0.962"],["4.417","8.167","0.113"],["0.492","4.771","0.785"],["7.632","2.605","0.645"],["14.242","5.042","0.368"]]
    }
]
```

- In Pyecharts, use `pyecharts.commons.utils.JsCode` directly, JsCode automatically calls `.js_code` when dumping options.

```
.set_series_opts(
        label_opts=opts.LabelOpts(
            position="right",
            formatter=JsCode(
                "function(x){return Number(x.data.percent * 100).toFixed() + '%';}"
            ),
        )
    )
```

**Note**: you need the JS string to be on one-line. You can use Javascript minifiers like https://javascript-minifier.com/ or https://www.minifier.org/ to transform your Javascript code to a one-liner.

### st_pyecharts VS using pyecharts with components.html

While this package provides a `st_pyecharts` method, if you're using `pyecharts` you can directly embed your pyecharts visualization inside `st.html`
by passing the output of the chart's `.render_embed()`.

```python
from pyecharts.charts import Bar
from pyecharts import options as opts
import streamlit.components.v1 as components

c = (Bar()
    .add_xaxis(["Microsoft", "Amazon", "IBM", "Oracle", "Google", "Alibaba"])
    .add_yaxis('2017-2018 Revenue in (billion $)', [21.2, 20.4, 10.3, 6.08, 4, 2.2])
    .set_global_opts(title_opts=opts.TitleOpts(title="Top cloud providers 2018", subtitle="2017-2018 Revenue"),
                     toolbox_opts=opts.ToolboxOpts())
    .render_embed() # generate a local HTML file
)
components.html(c, width=1000, height=1000)
```

Using `st_pyecharts` is still something you would want if you need to change data regularly
without remounting the component, check for examples `examples/app_pyecharts.py` for `Chart with randomization` example.

![](./img/randomize.gif)

## Credits

- It's really a wrapper around [echarts-for-react](https://github.com/hustcc/echarts-for-react).
