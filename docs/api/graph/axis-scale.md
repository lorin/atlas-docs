Scales determine how the data value for a line will get mapped to the Y-Axis.
There are currently four scales that can be used for an axis:

* [Linear](#linear)
* [Logarithmic](#logarithmic)
* [Power of 2](#power-of-2)
* [Square Root](#square-root)

## Linear

A linear scale uniformly maps the input values (domain) to the Y-axis location (range).
If _v_ is datapoint in a time series, then _y=m*v+b_ where _m_ and _b_ are automatically
chosen based on the domain and range.

This is the default scale for an axis and will get used if no explicit scale is set. Since
1.6, it can also be used explicitly:

@@@ atlas-uri { hilite=scale%3dlinear }
/api/v1/graph?e=2012-01-01T00:00&q=minuteOfHour,:time,1e3,:add,minuteOfHour,:time&scale=linear
@@@

@@@ atlas-graph { show-expr=false }
/api/v1/graph?e=2012-01-01T00:00&q=minuteOfHour,:time,1e3,:add,minuteOfHour,:time&scale=linear
@@@

## Logarithmic

A logarithmic scale emphasizes smaller values when mapping the input values (domain) to the
Y-axis location (range). This is often used if two lines with significantly different magnitudes
are on the same axis. If _v_ is datapoint in a time series, then _y=m*log(v)+b_ where _m_
and _b_ are automatically chosen based on the domain and range. In many cases, using a separate
[Y-axis](multi-y.md) can be a better option that doesn't distort the line as much.

To use this mode, add `scale=log` (prior to 1.6 use `o=1`).

@@@ atlas-uri { hilite=scale%3dlog }
/api/v1/graph?e=2012-01-01T00:00&q=minuteOfHour,:time,1e3,:add,minuteOfHour,:time&scale=log
@@@

@@@ atlas-graph { show-expr=false }
/api/v1/graph?e=2012-01-01T00:00&q=minuteOfHour,:time,1e3,:add,minuteOfHour,:time&scale=log
@@@

## Power of 2

Since 1.6.

A power scale that emphasizes larger values when mapping the input values (domain) to the
Y-axis location (range). If _v_ is datapoint in a time series, then _y=m*v<sup>2</sup>+b_
where _m_ and _b_ are automatically chosen based on the domain and range. To emphasize smaller
values see the [square root scale](#square-root).

To use this mode, add `scale=pow2`.

@@@ atlas-uri { hilite=scale%3dpow2 }
/api/v1/graph?e=2012-01-01T00:00&q=minuteOfHour,:time,1e3,:add,minuteOfHour,:time&scale=pow2
@@@

@@@ atlas-graph { show-expr=false }
/api/v1/graph?e=2012-01-01T00:00&q=minuteOfHour,:time,1e3,:add,minuteOfHour,:time&scale=pow2
@@@

## Square Root

Since 1.6.

A power scale that emphasizes smaller values when mapping the input values (domain) to the
Y-axis location (range). If _v_ is datapoint in a time series, then _y=m*v<sup>0.5</sup>+b_
where _m_ and _b_ are automatically chosen based on the domain and range. To emphasize larger
values see the [power of 2 scale](#power-of-2).

To use this mode, add `scale=sqrt`.

@@@ atlas-uri { hilite=scale%3dsqrt }
/api/v1/graph?e=2012-01-01T00:00&q=minuteOfHour,:time,1e3,:add,minuteOfHour,:time&scale=sqrt
@@@

@@@ atlas-graph { show-expr=false }
/api/v1/graph?e=2012-01-01T00:00&q=minuteOfHour,:time,1e3,:add,minuteOfHour,:time&scale=sqrt
@@@