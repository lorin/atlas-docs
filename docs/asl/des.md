[Double exponential smoothing](http://www.itl.nist.gov/div898/handbook/pmc/section4/pmc433.htm)
(DES) is a simple technique for generating a smooth trend line from another time series. This
technique is often used to generate a dynamic threshold for alerting.

!!! warning 
    Alerts on dynamic thresholds should be expected to be noisy. They are looking
    for strange behavior rather than an actual problem causing impact. Make sure you will
    actually spend the time to tune and investigate the alarms before using this approach.
    See the [alerting philosophy](alerting-philosophy.md) guide for more information on best
    practices.

## Tuning

The [:des](ref/des.md) operator takes 4 parameters:

* An input time series
* `training` - the number of intervals to use for warming up before generating an output
* `alpha` - is a data smoothing factor
* `beta` - is a trend smoothing factor

### Training

The training parameter defines how many intervals to allow the DES to warmup. In the graph
below the gaps from the start of the chart to the smoothed lines reflects the training window
used:

@@@ atlas-graph { show-expr=false }
/api/v1/graph?tz=UTC&e=2012-01-01T09:00&s=e-2d&q=:true,800,:fadd,input,:legend,:true,400,:fadd,30,0.1,1,:des,training+%3D+30,:legend,:true,90,0.1,1,:des,training+%3D+90,:legend,:list,(,nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:cq,),:each&l=0
@@@

Typically a training window of 10 has been sufficient as DES will adjust to the input fairly
quick. However, in some cases if there is a massive change in the input it can cause DES to
oscillate, for example:

@@@ atlas-graph { show-expr=false }
/api/v1/graph?tz=UTC&e=2012-01-01T09:00&s=e-6h&w=750&h=150&q=nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:sum,:dup,10,0.1,0.5,:des,0.9,:mul,:2over,:lt,:rot,$$name,:legend,:rot,prediction,:legend,:rot,:vspan,60,:alpha,alert+triggered,:legend
@@@

### Alpha

Alpha is the data smoothing factor. A value of 1 means no smoothing. The closer the value
gets to 0 the smoother the line should get. Example:

@@@ atlas-graph { show-expr=false }
/api/v1/graph?tz=UTC&e=2012-01-01T09:00&s=e-2d&q=:true,1200,:fadd,input,:legend,:true,10,1,1,:des,800,:fadd,alpha+%3D+1,:legend,:true,10,0.1,1,:des,400,:fadd,alpha+%3D+0.1,:legend,:true,10,0.01,1,:des,alpha+%3D+0.01,:legend,:list,(,nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:cq,),:each&l=0
@@@

### Beta

Beta is a trend smoothing factor. Visually it is most apparent when alpha is small. Example
with `alpha = 0.01`:

@@@ atlas-graph { show-expr=false }
/api/v1/graph?tz=UTC&e=2012-01-01T09:00&s=e-2d&q=:true,1200,:fadd,input,:legend,:true,10,0.01,1,:des,800,:fadd,beta+%3D+1,:legend,:true,10,0.01,0.1,:des,400,:fadd,beta+%3D+0.1,:legend,:true,10,0.01,0.01,:des,beta+%3D+0.01,:legend,:list,(,nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:cq,),:each&l=0
@@@

## Recommended Values

Experimentally we have converged on 3 sets of values based on how quickly it should adjust
to changing levels in the input signal.

| Helper                             | Alpha  | Beta  |
|------------------------------------|--------|-------|
| [:des-fast](ref/des-fast.md)       | 0.1    | 0.02  |
| [:des-slower](ref/des-slower.md) | 0.05   | 0.03  |
| [:des-slow](ref/des-slow.md)     | 0.03   | 0.04  |

Here is an example of how they behave for a sharp drop and recovery:

@@@ atlas-graph { show-expr=false }
/api/v1/graph?tz=UTC&e=2012-01-01T09:00&s=e-6h&w=750&h=150&q=:true,input,:legend,:true,:des-fast,des-fast,:legend,:true,:des-slower,des-slower,:legend,:true,:des-slow,des-slow,:legend,:list,(,nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:cq,),:each
@@@

For a more gradual drop:

@@@ atlas-graph { show-expr=false }
/api/v1/graph?tz=UTC&e=2012-01-02T09:00&s=e-6h&w=750&h=150&q=:true,input,:legend,:true,:des-fast,des-fast,:legend,:true,:des-slower,des-slower,:legend,:true,:des-slow,des-slow,:legend,:list,(,nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:cq,),:each&l=0
@@@

If the drop is smooth enough then DES can adjust without ever triggering.

## Alerting

For alerting purposes the DES line will typically get multiplied by a fraction and then
checked to see whether the input line drops below the DES value for a given interval.

```
# Query to generate the input line
nf.cluster,alerttest,:eq,
name,requestsPerSecond,:eq,:and,
:sum,

# Create a copy on the stack
:dup,

# Apply a DES function to generate a prediction
:des-fast,

# Used to set a threshold. The prediction should
# be roughly equal to the line, in this case the
# threshold would be 85% of the prediction.
0.85,:mul,

# Create a boolean signal line that is 1
# for datapoints where the actual value is
# less than the prediction and 0 where it
# is greater than or equal the prediction.
# The 1 values are where the alert should
# trigger.
:lt,

# Apply presentation details.
:rot,$name,:legend,
```

The vertical spans show when the expression would have triggered with due to the input
dropping below the DES line at 85%:

@@@ atlas-graph { show-expr=false }
/api/v1/graph?tz=UTC&e=2012-01-01T09:00&s=e-6h&w=750&h=150&q=:true,:true,0.85,:mul,:des-fast,:2over,:lt,:vspan,40,:alpha,:rot,input,:legend,:rot,des-fast,:legend,:rot,triggered,:legend,:list,(,nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:cq,),:each&l=0
@@@

@@@ atlas-graph { show-expr=false }
/api/v1/graph?tz=UTC&e=2012-01-02T09:00&s=e-6h&w=750&h=150&q=:true,:true,0.85,:mul,:des-fast,:2over,:lt,:vspan,40,:alpha,:rot,input,:legend,:rot,des-fast,:legend,:rot,triggered,:legend,:list,(,nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:cq,),:each&l=0
@@@

## Epic Macros

There are two helper macros, [des-epic-signal](ref/des-epic-signal.md) and
[des-epic-viz](ref/des-epic-viz.md), that match the behavior of the previous epic DES alarms.
The first generates a signal line for the alarm. The second creates a visualization to make
it easier to see what is happening. Both take the following arguments:

* `line` - input line
* `trainingSize` - training size parameter for DES
* `alpha` - alpha parameter for DES
* `beta` - beta parameter for DES
* `maxPercent` - percentage offset to use for the upper bound. Can be set to NaN to disable the upper bound check.
* `minPercent` - percentage offset to use for the lower bound. Can be set to NaN to disable the lower bound check.
* `noise` - a fixed offset that is the minimum difference between the signal and prediction that is required before the signal should trigger. This is primarily used to avoid false alarms where the percentage bound can become ineffective for routine noise during the troughs.

Examples:

@@@ atlas-graph { show-expr=true }
/api/v1/graph?e=2012-01-01T09:00&h=150&l=0&q=nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:sum,10,0.1,0.02,0.15,0.15,10,:des-epic-viz&s=e-6h&tz=UTC&w=750
@@@

Example with no lower bound:

@@@ atlas-graph { show-expr=true }
/api/v1/graph?e=2012-01-02T09:00&h=150&l=0&q=nf.cluster,alerttest,:eq,name,requestsPerSecond,:eq,:and,:sum,10,0.1,0.02,0.15,NaN,10,:des-epic-viz&s=e-6h&tz=UTC&w=750
@@@
