# bt.gd
A (simple) script and text based behaviour tree for Godot.

Either create the tree directly:

```GDScript
_bt = BT.create() \
	.parallel() \
		.repeat() \
			.sequence() \
				.do(set_next_pos) \
				.do(walk_to_next_pos.bind(walk_speed)) \
				.wait(func() -> float: return randf_range(0.0, 1.0)) \
			.end() \
		.repeat() \
			.sequence() \
				.do(change_color) \
				.wait(0.75) \
			.end()
```

Or feed a text

```GDScript
 var text := "
 parallel
 	repeat
 		sequence
 			set_next_pos
 			walk_to_next_pos {0.5 * walk_speed}
			wait $wait_time
	repeat
		sequence
			change_color
			wait 0.75"
_bt = BT.create(self, text)
```

To actually run the behaviour tree, call `_bt.tick()` whenever you want it to update - for realtime
games this is usually every frame, but for turn-based games this could be on each turn.
Behaviour trees created via text also support variables from the target as arguments. If you want
these to be updated even after the tree's initialisation, give them a $ prefix (this is probably
expensive performance-wise, so only use this for testing purposes).

You can also use expressions by using curly braces, for example like this: `{0.5 * _speed}`. The
expression will be evaluated at initialisation if you don't prefix it with $.

Text-based behaviour trees also support the node `tree`. If it's at the root, it creates a tree.
If you use `tree '<name>'` inside a tree, it will insert this tree. You can only insert trees defined
beforehand, preventing circular insertion. For non-text-based trees, use the insert_tree() method.

Composite nodes (more than 0 children):
* `sequence`
* `selector`
* `parallel`
* `race`
* `randomizer`
* `if_else`

Decorator nodes (1 child):
* `ignore`
* `invert`
* `override`
* `repeat`
* `retry`

Other nodes (0 children):
* `nothing`
* `fail`
* `success`
* `log`
* `wait`
* `do`/`prep_do`/`check`

`do` is the tree node that allows using your own custom behaviours. Your behaviour function should
return `BT.Status.Success`, `BT.Status.Fail` or `BT.Status.Running`, or a `bool`. If the function
is `void` or returns `null`, the status is set to Success; in all other cases the internal status
is not changed. There's `BT.Status.Nothing`, which means the result does not change the execution of
the parent decorator or composite node.

You can also create your own custom nodes by inheriting from the `BT.N` class. The `do` and `check`
nodes both use the `NAction` class.

* Originally a port of https://github.com/ratkingsminion/simple-behaviour-tree
* Inspired by fluid BT: https://github.com/ashblue/fluid-behavior-tree
* Also by PandaBT: http://www.pandabt.com/documentation/2.0.0
* About behaviour trees: https://www.gamedeveloper.com/programming/behavior-trees-for-ai-how-they-work
