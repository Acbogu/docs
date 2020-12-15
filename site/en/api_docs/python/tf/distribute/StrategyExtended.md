description: Additional APIs for algorithms that need to be distribution-aware.

<div itemscope itemtype="http://developers.google.com/ReferenceObject">
<meta itemprop="name" content="tf.distribute.StrategyExtended" />
<meta itemprop="path" content="Stable" />
<meta itemprop="property" content="__init__"/>
<meta itemprop="property" content="batch_reduce_to"/>
<meta itemprop="property" content="colocate_vars_with"/>
<meta itemprop="property" content="reduce_to"/>
<meta itemprop="property" content="update"/>
<meta itemprop="property" content="value_container"/>
<meta itemprop="property" content="variable_created_in_scope"/>
</div>

# tf.distribute.StrategyExtended

<!-- Insert buttons and diff -->

<table class="tfo-notebook-buttons tfo-api nocontent" align="left">
<td>
  <a target="_blank" href="https://github.com/tensorflow/tensorflow/blob/r2.3/tensorflow/python/distribute/distribute_lib.py#L1917-L2435">
    <img src="https://www.tensorflow.org/images/GitHub-Mark-32px.png" />
    View source on GitHub
  </a>
</td>
</table>



Additional APIs for algorithms that need to be distribution-aware.

<pre class="devsite-click-to-copy prettyprint lang-py tfo-signature-link">
<code>tf.distribute.StrategyExtended(
    container_strategy
)
</code></pre>



<!-- Placeholder for "Used in" -->

Note: For most usage of <a href="../../tf/distribute/Strategy.md"><code>tf.distribute.Strategy</code></a>, there should be no need to
call these methods, since TensorFlow libraries (such as optimizers) already
call these methods when needed on your behalf.


Some common use cases of functions on this page:

* _Locality_

<a href="../../tf/distribute/DistributedValues.md"><code>tf.distribute.DistributedValues</code></a> can have the same _locality_ as a
_distributed variable_, which leads to a mirrored value residing on the same
devices as the variable (as opposed to the compute devices). Such values may
be passed to a call to <a href="../../tf/distribute/StrategyExtended.md#update"><code>tf.distribute.StrategyExtended.update</code></a> to update the
value of a variable. You may use
<a href="../../tf/distribute/StrategyExtended.md#colocate_vars_with"><code>tf.distribute.StrategyExtended.colocate_vars_with</code></a> to give a variable the
same locality as another variable. You may convert a "PerReplica" value to a
variable's locality by using <a href="../../tf/distribute/StrategyExtended.md#reduce_to"><code>tf.distribute.StrategyExtended.reduce_to</code></a> or
<a href="../../tf/distribute/StrategyExtended.md#batch_reduce_to"><code>tf.distribute.StrategyExtended.batch_reduce_to</code></a>.

* _How to update a distributed variable_

A distributed variable is variables created on multiple devices. As discussed
in the [glossary](https://www.tensorflow.org/api_docs/python/tf/distribute),
mirrored variable and SyncOnRead variable are two examples. The standard
pattern for updating distributed variables is to:

1. In your function passed to <a href="../../tf/distribute/Strategy.md#run"><code>tf.distribute.Strategy.run</code></a>,
   compute a list of (update, variable) pairs. For example, the update might
   be a gradient of the loss with respect to the variable.
2. Switch to cross-replica mode by calling
   `tf.distribute.get_replica_context().merge_call()` with the updates and
   variables as arguments.
3. Call
   <a href="../../tf/distribute/StrategyExtended.md#reduce_to"><code>tf.distribute.StrategyExtended.reduce_to(VariableAggregation.SUM, t, v)</code></a>
   (for one variable) or <a href="../../tf/distribute/StrategyExtended.md#batch_reduce_to"><code>tf.distribute.StrategyExtended.batch_reduce_to</code></a>
   (for a list of variables) to sum the updates.
4. Call <a href="../../tf/distribute/StrategyExtended.md#update"><code>tf.distribute.StrategyExtended.update(v)</code></a> for each variable to update
   its value.

Steps 2 through 4 are done automatically by class
<a href="../../tf/keras/optimizers/Optimizer.md"><code>tf.keras.optimizers.Optimizer</code></a> if you call its
<a href="../../tf/keras/optimizers/Optimizer.md#apply_gradients"><code>tf.keras.optimizers.Optimizer.apply_gradients</code></a> method in a replica context.

In fact, a higher-level solution to update a distributed variable is by
calling `assign` on the variable as you would do to a regular <a href="../../tf/Variable.md"><code>tf.Variable</code></a>.
You can call the method in both _replica context_ and _cross-replica context_.
For a _mirrored variable_, calling `assign` in _replica context_ requires you
to specify the `aggregation` type in the variable constructor. In that case,
the context switching and sync described in steps 2 through 4 are handled for
you. If you call `assign` on _mirrored variable_ in _cross-replica context_,
you can only assign a single value or assign values from another mirrored
variable or a mirrored <a href="../../tf/distribute/DistributedValues.md"><code>tf.distribute.DistributedValues</code></a>. For a _SyncOnRead
variable_, in _replica context_, you can simply call `assign` on it and no
aggregation happens under the hood. In _cross-replica context_, you can only
assign a single value to a SyncOnRead variable. One example case is restoring
from a checkpoint: if the `aggregation` type of the variable is
<a href="../../tf/VariableAggregation.md#SUM"><code>tf.VariableAggregation.SUM</code></a>, it is assumed that replica values were added
before checkpointing, so at the time of restoring, the value is divided by
the number of replicas and then assigned to each replica; if the `aggregation`
type is <a href="../../tf/VariableAggregation.md#MEAN"><code>tf.VariableAggregation.MEAN</code></a>, the value is assigned to each replica
directly.



<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2"><h2 class="add-link">Attributes</h2></th></tr>

<tr>
<td>
`experimental_require_static_shapes`
</td>
<td>
Returns `True` if static shape is required; `False` otherwise.
</td>
</tr><tr>
<td>
`parameter_devices`
</td>
<td>
Returns the tuple of all devices used to place variables.
</td>
</tr><tr>
<td>
`worker_devices`
</td>
<td>
Returns the tuple of all devices used to for compute replica execution.
</td>
</tr>
</table>



## Methods

<h3 id="batch_reduce_to"><code>batch_reduce_to</code></h3>

<a target="_blank" href="https://github.com/tensorflow/tensorflow/blob/r2.3/tensorflow/python/distribute/distribute_lib.py#L2199-L2223">View source</a>

<pre class="devsite-click-to-copy prettyprint lang-py tfo-signature-link">
<code>batch_reduce_to(
    reduce_op, value_destination_pairs, experimental_hints=None
)
</code></pre>

Combine multiple `reduce_to` calls into one for faster execution.


<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Args</th></tr>

<tr>
<td>
`reduce_op`
</td>
<td>
Reduction type, an instance of <a href="../../tf/distribute/ReduceOp.md"><code>tf.distribute.ReduceOp</code></a> enum.
</td>
</tr><tr>
<td>
`value_destination_pairs`
</td>
<td>
A sequence of (value, destinations) pairs. See
`reduce_to()` for a description.
</td>
</tr><tr>
<td>
`experimental_hints`
</td>
<td>
A `tf.distrbute.experimental.CollectiveHints`. Hints
to perform collective operations.
</td>
</tr>
</table>



<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Returns</th></tr>
<tr class="alt">
<td colspan="2">
A list of mirrored values, one per pair in `value_destination_pairs`.
</td>
</tr>

</table>



<h3 id="colocate_vars_with"><code>colocate_vars_with</code></h3>

<a target="_blank" href="https://github.com/tensorflow/tensorflow/blob/r2.3/tensorflow/python/distribute/distribute_lib.py#L2083-L2128">View source</a>

<pre class="devsite-click-to-copy prettyprint lang-py tfo-signature-link">
<code>colocate_vars_with(
    colocate_with_variable
)
</code></pre>

Scope that controls which devices variables will be created on.

No operations should be added to the graph inside this scope, it
should only be used when creating variables (some implementations
work by changing variable creation, others work by using a
tf.compat.v1.colocate_with() scope).

This may only be used inside `self.scope()`.

#### Example usage:



```
with strategy.scope():
  var1 = tf.Variable(...)
  with strategy.extended.colocate_vars_with(var1):
    # var2 and var3 will be created on the same device(s) as var1
    var2 = tf.Variable(...)
    var3 = tf.Variable(...)

  def fn(v1, v2, v3):
    # operates on v1 from var1, v2 from var2, and v3 from var3

  # `fn` runs on every device `var1` is on, `var2` and `var3` will be there
  # too.
  strategy.extended.update(var1, fn, args=(var2, var3))
```

<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Args</th></tr>

<tr>
<td>
`colocate_with_variable`
</td>
<td>
A variable created in this strategy's `scope()`.
Variables created while in the returned context manager will be on the
same set of devices as `colocate_with_variable`.
</td>
</tr>
</table>



<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Returns</th></tr>
<tr class="alt">
<td colspan="2">
A context manager.
</td>
</tr>

</table>



<h3 id="reduce_to"><code>reduce_to</code></h3>

<a target="_blank" href="https://github.com/tensorflow/tensorflow/blob/r2.3/tensorflow/python/distribute/distribute_lib.py#L2168-L2194">View source</a>

<pre class="devsite-click-to-copy prettyprint lang-py tfo-signature-link">
<code>reduce_to(
    reduce_op, value, destinations, experimental_hints=None
)
</code></pre>

Combine (via e.g. sum or mean) values across replicas.


<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Args</th></tr>

<tr>
<td>
`reduce_op`
</td>
<td>
Reduction type, an instance of <a href="../../tf/distribute/ReduceOp.md"><code>tf.distribute.ReduceOp</code></a> enum.
</td>
</tr><tr>
<td>
`value`
</td>
<td>
A per-replica value with one value per replica.
</td>
</tr><tr>
<td>
`destinations`
</td>
<td>
A mirrored variable, a per-replica tensor, or a device
string. The return value will be copied to all destination devices (or
all the devices where the `destinations` value resides). To perform an
all-reduction, pass `value` to `destinations`.
</td>
</tr><tr>
<td>
`experimental_hints`
</td>
<td>
A `tf.distrbute.experimental.CollectiveHints`. Hints
to perform collective operations.
</td>
</tr>
</table>



<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Returns</th></tr>
<tr class="alt">
<td colspan="2">
A tensor or value mirrored to `destinations`.
</td>
</tr>

</table>



<h3 id="update"><code>update</code></h3>

<a target="_blank" href="https://github.com/tensorflow/tensorflow/blob/r2.3/tensorflow/python/distribute/distribute_lib.py#L2233-L2300">View source</a>

<pre class="devsite-click-to-copy prettyprint lang-py tfo-signature-link">
<code>update(
    var, fn, args=(), kwargs=None, group=(True)
)
</code></pre>

Run `fn` to update `var` using inputs mirrored to the same devices.

<a href="../../tf/distribute/StrategyExtended.md#update"><code>tf.distribute.StrategyExtended.update</code></a> takes a distributed variable `var`
to be updated, an update function `fn`, and `args` and `kwargs` for `fn`. It
applies `fn` to each component variable of `var` and passes corresponding
values from `args` and `kwargs`. Neither `args` nor `kwargs` may contain
per-replica values. If they contain mirrored values, they will be unwrapped
before calling `fn`. For example, `fn` can be `assign_add` and `args` can be
a mirrored DistributedValues where each component contains the value to be
added to this mirrored variable `var`. Calling `update` will call
`assign_add` on each component variable of `var` with the corresponding
tensor value on that device.

#### Example usage:



```python
strategy = tf.distribute.MirroredStrategy(['/gpu:0', '/gpu:1']) # With 2 devices
with strategy.scope():
  v = tf.Variable(5.0, aggregation=tf.VariableAggregation.SUM)
def update_fn(v):
  return v.assign(1.0)
result = strategy.extended.update(v, update_fn)
# result is
# Mirrored:{
#  0: tf.Tensor(1.0, shape=(), dtype=float32),
#  1: tf.Tensor(1.0, shape=(), dtype=float32)
# }
```

If `var` is mirrored across multiple devices, then this method implements
logic as following:

```python
results = {}
for device, v in var:
  with tf.device(device):
    # args and kwargs will be unwrapped if they are mirrored.
    results[device] = fn(v, *args, **kwargs)
return merged(results)
```

Otherwise, this method returns `fn(var, *args, **kwargs)` colocated with
`var`.

<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Args</th></tr>

<tr>
<td>
`var`
</td>
<td>
Variable, possibly mirrored to multiple devices, to operate on.
</td>
</tr><tr>
<td>
`fn`
</td>
<td>
Function to call. Should take the variable as the first argument.
</td>
</tr><tr>
<td>
`args`
</td>
<td>
Tuple or list. Additional positional arguments to pass to `fn()`.
</td>
</tr><tr>
<td>
`kwargs`
</td>
<td>
Dict with keyword arguments to pass to `fn()`.
</td>
</tr><tr>
<td>
`group`
</td>
<td>
Boolean. Defaults to True. If False, the return value will be
unwrapped.
</td>
</tr>
</table>



<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Returns</th></tr>
<tr class="alt">
<td colspan="2">
By default, the merged return value of `fn` across all replicas.  The
merged result has dependencies to make sure that if it is evaluated at
all, the side effects (updates) will happen on every replica. If instead
"group=False" is specified, this function will return a nest of lists
where each list has an element per replica, and the caller is responsible
for ensuring all elements are executed.
</td>
</tr>

</table>



<h3 id="value_container"><code>value_container</code></h3>

<a target="_blank" href="https://github.com/tensorflow/tensorflow/blob/r2.3/tensorflow/python/distribute/distribute_lib.py#L2337-L2350">View source</a>

<pre class="devsite-click-to-copy prettyprint lang-py tfo-signature-link">
<code>value_container(
    value
)
</code></pre>

Returns the container that this per-replica `value` belongs to.


<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Args</th></tr>

<tr>
<td>
`value`
</td>
<td>
A value returned by `run()` or a variable created in `scope()`.
</td>
</tr>
</table>



<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Returns</th></tr>
<tr class="alt">
<td colspan="2">
A container that `value` belongs to.
If value does not belong to any container (including the case of
container having been destroyed), returns the value itself.
`value in experimental_local_results(value_container(value))` will
always be true.
</td>
</tr>

</table>



<h3 id="variable_created_in_scope"><code>variable_created_in_scope</code></h3>

<a target="_blank" href="https://github.com/tensorflow/tensorflow/blob/r2.3/tensorflow/python/distribute/distribute_lib.py#L2057-L2081">View source</a>

<pre class="devsite-click-to-copy prettyprint lang-py tfo-signature-link">
<code>variable_created_in_scope(
    v
)
</code></pre>

Tests whether `v` was created while this strategy scope was active.

Variables created inside the strategy scope are "owned" by it:

```
>>> strategy = tf.distribute.MirroredStrategy()
>>> with strategy.scope():
...   v = tf.Variable(1.)
>>> strategy.extended.variable_created_in_scope(v)
True
```

Variables created outside the strategy are not owned by it:

```
>>> strategy = tf.distribute.MirroredStrategy()
>>> v = tf.Variable(1.)
>>> strategy.extended.variable_created_in_scope(v)
False
```

<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Args</th></tr>

<tr>
<td>
`v`
</td>
<td>
A <a href="../../tf/Variable.md"><code>tf.Variable</code></a> instance.
</td>
</tr>
</table>



<!-- Tabular view -->
 <table class="responsive fixed orange">
<colgroup><col width="214px"><col></colgroup>
<tr><th colspan="2">Returns</th></tr>
<tr class="alt">
<td colspan="2">
True if `v` was created inside the scope, False if not.
</td>
</tr>

</table>




