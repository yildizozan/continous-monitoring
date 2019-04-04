
# Group 1 - Continuous Monitoring

1.  Using Prometheus in Grafana
2.  Prometheus
3.  Kiali
4.  Jeager  with  OpenTrace


# Grafana

Grafana includes built-in support for Prometheus.

## Adding the data source to Grafana

1. Open the side menu by clicking the Grafana icon in the top header.
2. In the side menu under the `Dashboards` link you should find a link named `Data Sources`.
3. Click the `+ Add data source` button in the top header.
4. Select `Prometheus` from the _Type_ dropdown.

> NOTE: If you're not seeing the `Data Sources` link in your side menu it means that your current user does not have the `Admin` role for the current organization.

## Data source options

| Name              | Description                                                                                                                           |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| _Name_            | The data source name. This is how you refer to the data source in panels & queries.                                                   |
| _Default_         | Default data source means that it will be pre-selected for new panels.                                                                |
| _Url_             | The http protocol, ip and port of you Prometheus server (default port is usually 9090)                                                |

## Query editor

Open a graph in edit mode by click the title > Edit (or by pressing `e` key while hovering over panel).

![](http://docs.grafana.com/img/docs/v45/prometheus_query_editor_still.png)

| Name               | Description                                                                                                                                                                                                                                                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| _Query expression_ | Prometheus query expression, check out the [Prometheus documentation](http://prometheus.io/docs/querying/basics/).                                                                                                                                                                                                                  |
| _Legend format_    | Controls the name of the time series, using name or pattern. For example `{{hostname}}` will be replaced with label value for the label `hostname`.                                                                                                                                                                                 |
| _Min step_         | Set a lower limit for the Prometheus step option. Step controls how big the jumps are when the Prometheus query engine performs range queries. Sadly there is no official prometheus documentation to link to for this very important option.                                                                                       |
| _Resolution_       | Controls the step option. Small steps create high-resolution graphs but can be slow over larger time ranges, lowering the resolution can speed things up. `1/2` will try to set step option to generate 1 data point for every other pixel. A value of `1/10` will try to set step option so there is a data point every 10 pixels. |
| _Metric lookup_    | Search for metric names in this input field.                                                                                                                                                                                                                                                                                        |
| _Format as_        | Switch between Table, Time series or Heatmap. Table format will only work in the Table panel. Heatmap format is suitable for displaying metrics having histogram type on Heatmap panel. Under the hood, it converts cumulative histogram to regular and sorts series by the bucket bound.                                           |


> NOTE: Grafana slightly modifies the request dates for queries to align them with the dynamically calculated step.
> This ensures consistent display of metrics data but can result in a small gap of data at the right edge of a graph.

### Query variable

Variable of the type _Query_ allows you to query Prometheus for a list of metrics, labels or label values. 
The Prometheus data source plugin
provides the following functions you can use in the `Query` input field.

| Name                          | Description                                                             |
| ----------------------------- | ----------------------------------------------------------------------- |
| _label_names()_               | Returns a list of label names.                                          |
| _label_values(label)_         | Returns a list of label values for the `label` in every metric.         |
| _label_values(metric, label)_ | Returns a list of label values for the `label` in the specified metric. |
| _metrics(metric)_             | Returns a list of metrics matching the specified `metric` regex.        |
| _query_result(query)_         | Returns a list of Prometheus query result for the `query`.              |

#### Using interval and range variables

It's possible to use some global built-in variables in query variables; `$__interval`, `$__interval_ms`, `$__range`, `$__range_s` and `$__range_ms`, see [Global built-in variables](/reference/templating/#global-built-in-variables) for more information. These can be convenient to use in conjunction with the `query_result` function when you need to filter variable queries since
`label_values` function doesn't support queries.

Make sure to set the variable's `refresh` trigger to be `On Time Range Change` to get the correct instances when changing the time range on the dashboard.

**Example usage:**

Populate a variable with the the busiest 5 request instances based on average QPS over the time range shown in the dashboard:

```
Query: query_result(topk(5, sum(rate(http_requests_total[$__range])) by (instance)))
Regex: /"([^"]+)"/
```

Populate a variable with the instances having a certain state over the time range shown in the dashboard, using the more precise `$__range_s`:

```
Query: query_result(max_over_time(<metric>[${__range_s}s]) != <state>)
Regex:
```

### Using variables in queries

There are two syntaxes:

- `$<varname>` Example: `rate(http_requests_total{job=~"\$job"}[5m])`
- `[[varname]]` Example: `rate(http_requests_total{job=~"[[job]]"}[5m])`

Why two ways? The first syntax is easier to read and write but does not allow you to use a variable in the middle of a word. When the _Multi-value_ or _Include all value_
options are enabled, Grafana converts the labels from plain text to a regex compatible string. Which means you have to use `=~` instead of `=`.

## Getting Grafana metrics into Prometheus

Grafana exposes metrics for Prometheus on the `/metrics` endpoint.
## Configure the Datasource with Provisioning

Here are some provisioning examples for this datasource.

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://localhost:9090
```

# Creating Graph Panel

![](http://docs.grafana.com/img/docs/v45/graph_overview.png)

The main panel in Grafana is simply named Graph. It provides a very rich set of graphing options.

1. Clicking the title for a panel exposes a menu.  The `edit` option opens additional configuration
options for the panel.
2. Click to open color & axis selection.
3. Click to only show this series. Shift/Ctrl + click to hide series.

## General

![](http://docs.grafana.com/img/docs/v51/graph_general.png)
The general tab allows customization of a panel's appearance and menu options.

### Info

- **Title** - The panel title of the dashboard, displayed at the top.
- **Description** - The panel description, displayed on hover of info icon in the upper left corner of the panel.
- **Transparent** - If checked, removes the solid background of the panel (default not checked).

## Axes

![](http://docs.grafana.com/img/docs/v51/graph_axes_grid_options.png)
The Axes tab controls the display of axes.

### Left Y/Right Y

The **Left Y** and **Right Y** can be customized using:

- **Unit** - The display unit for the Y value
- **Scale** - The scale to use for the Y value, linear or logarithmic. (default linear)
- **Y-Min** - The minimum Y value. (default auto)
- **Y-Max** - The maximum Y value. (default auto)
- **Decimals** - Controls how many decimals are displayed for Y value (default auto)
- **Label** - The Y axis label (default "")

Axes can also be hidden by unchecking the appropriate box from **Show**.

### X-Axis

Axis can be hidden by unchecking **Show**.

For **Mode** there are three options:

- The default option is **Time** and means the x-axis represents time and that the data is grouped by time (for example, by hour or by minute).

- The **Series** option means that the data is grouped by series and not by time. The y-axis still represents the value.

![](http://docs.grafana.com/img/docs/v51/graph-x-axis-mode-series.png)

- The **Histogram** option converts the graph into a histogram. A Histogram is a kind of bar chart that groups numbers into ranges, often called buckets or bins. Taller bars show that more data falls in that range. Histograms and buckets are described in more detail [here](http://docs.grafana.org/features/panels/heatmap/#histograms-and-buckets).

    <img src="http://docs.grafana.com/img/docs/v43/heatmap_histogram.png" class="no-shadow">


### Y-Axes

- **Align** - Check to align left and right Y-axes by value (default unchecked/false)
- **Level** - Available when *Align* is checked. Value to use for alignment of left and right Y-axes, starting from Y=0 (default 0)

## Legend


![](http://docs.grafana.com/img/docs/v51/graph-legend.png)

### Options

- **Show** - Uncheck to hide the legend (default checked/true)
- **Table** - Check to display legend in table (default unchecked/false)
- **To the right** - Check to display legend to the right (default unchecked/false)
- **Width** - Available when *To the right* is checked. Value to control the minimum width for the legend (default 0)

### Values

Additional values can be shown along-side the legend names:

- **Min** - Minimum of all values returned from metric query
- **Max** - Maximum of all values returned from the metric query
- **Avg** - Average of all values returned from metric query
- **Current** - Last value returned from the metric query
- **Total** - Sum of all values returned from metric query
- **Decimals** - Controls how many decimals are displayed for legend values (and graph hover tooltips)

The legend values are calculated client side by Grafana and depend on what type of
aggregation or point consolidation your metric query is using. All the above legend values cannot
be correct at the same time. For example if you plot a rate like requests/second, this is probably
using average as aggregator, then the Total in the legend will not represent the total number of requests.
It is just the sum of all data points received by Grafana.

### Hide series

Hide series when all values of a series from a metric query are of a specific value:

- **With only nulls** - Value=*null* (default unchecked)
- **With only zeros** - Value=*zero* (default unchecked)

## Display styles

![](http://docs.grafana.com/img/docs/v51/graph_display_styles.png)
Display styles control visual properties of the graph.

### Draw Options

#### Draw Modes

- **Bar** - Display values as a bar chart
- **Lines** - Display values as a line graph
- **Points** - Display points for values

#### Mode Options

- **Fill** - Amount of color fill for a series (default 1). 0 is none.
- **Line Width** - The width of the line for a series (default 1).
- **Staircase** - Draws adjacent points as staircase
- **Points Radius** - Adjust the size of points when *Points* are selected as *Draw Mode*.

#### Hover tooltip

- **Mode** - Controls how many series to display in the tooltip when hover over a point in time, All series or single (default All series).
- **Sort order** - Controls how series displayed in tooltip are sorted, None, Ascending or Descending (default None).
- **Stacked value** - Available when *Stack* are checked and controls how stacked values are displayed in tooltip (default Individual).
   - Individual: the value for the series you hover over
   - Cumulative - sum of series below plus the series you hover over

#### Stacking & Null value

If there are multiple series, they can be displayed as a group.

- **Stack** - Each series is stacked on top of another
- **Percent** - Available when *Stack* are checked. Each series is drawn as a percentage of the total of all series
- **Null value** - How null values are displayed

### Series overrides

![](http://docs.grafana.com/img/docs/v51/graph_display_overrides.png)
The section allows a series to be rendered differently from the others. For example, one series can be given
a thicker line width to make it stand out and/or be moved to the right Y-axis.

#### Dashes Drawing Style

There is an option under Series overrides to draw lines as dashes. Set Dashes to the value True to override the line draw setting for a specific series.

### Thresholds

![](http://docs.grafana.com/img/docs/v51/graph_display_thresholds.png)
Thresholds allow you to add arbitrary lines or sections to the graph to make it easier to see when
the graph crosses a particular threshold.

### Time Regions

![](http://docs.grafana.com/img/docs/v54/graph_time_regions.png)
Time regions allow you to highlight certain time regions of the graph to make it easier to see for example weekends, business hours and/or off work hours.

## Time Range

![](http://docs.grafana.org/img/docs/v51/graph-time-range.png)
The time range tab allows you to override the dashboard time range and specify a panel specific time.
Either through a relative from now time option or through a timeshift.

# Prometheus

## Architecture

This diagram illustrates the architecture of Prometheus and some of its ecosystem components

![](https://prometheus.io/assets/architecture.png)

# QUERY EXAMPLES

Return all time series with the metric `http_requests_total`:

    http_requests_total

Return all time series with the metric `http_requests_total` and the given
`job` and `handler` labels:

    http_requests_total{job="apiserver", handler="/api/comments"}

Return a whole range of time (in this case 5 minutes) for the same vector,
making it a range vector:

    http_requests_total{job="apiserver", handler="/api/comments"}[5m]

Note that an expression resulting in a range vector cannot be graphed directly,
but viewed in the tabular ("Console") view of the expression browser.

Using regular expressions, you could select time series only for jobs whose
name match a certain pattern, in this case, all jobs that end with `server`.
Note that this does a substring match, not a full string match:

    http_requests_total{job=~".*server"}

All regular expressions in Prometheus use [RE2
syntax](https://github.com/google/re2/wiki/Syntax).

To select all HTTP status codes except 4xx ones, you could run:

    http_requests_total{status!~"4.."}

## Using functions and operators

Return the per-second rate for all time series with the `http_requests_total` metric name, as measured over the last 5 minutes:

    rate(http_requests_total[5m])

Assuming that the `http_requests_total` time series all have the labels `job` (fanout by job name) and `instance` (fanout by instance of the job), we might want to sum over the rate of all instances, so we get fewer output time series, but still preserve the `job` dimension:

    sum(rate(http_requests_total[5m])) by (job)
