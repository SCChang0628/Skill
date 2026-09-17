# ROS 2 C++ Refactor Guide

Read the sections relevant to the current node. These are decision criteria, not a mandate to rewrite already-clear code.

## Functions and callbacks

- Prefer a named member callback for substantial processing. A short lambda is fine when it is already the local style.
- Keep callbacks recognizable: validate input, update state or call a clearly named operation, then return.
- Split functions that mix ROS I/O with substantial independent control logic or contain several named stages.
- Do not split a short linear sequence into trivial one-line helpers.
- Prefer readable early returns over deep nesting when each guard expresses a clear rejection condition.

## Node layout and construction

A conventional order is constructor, setup/parameter functions, callbacks, domain logic, helpers, then members. Follow the package when it differs.

Constructors may declare parameters, create ROS interfaces and timers, initialize simple state, and emit a short startup log. Extract setup only when the constructor is genuinely difficult to scan; avoid catch-all functions such as `initializeEverything()`.

Headers should reveal the node's structure without large implementation bodies. Use source files for non-trivial implementation unless local conventions or templates require otherwise.

## Publishers, subscriptions, services, and actions

- Name interfaces by purpose, not sequence: `odom_sub_`, not `sub1_`.
- Name callbacks by event or input: `odomCallback()`, `controlTimerCallback()`.
- Keep service input, validation, operation, and response obvious.
- Keep action goal, cancel, accepted/execution, feedback, and result responsibilities distinct enough to inspect.
- Avoid generic dispatchers for a few ordinary interfaces.

## QoS, time, and parameters

- Treat QoS as behavior. Preserve depth, reliability, durability, and history during style work.
- Use `rclcpp::SensorDataQoS()` for sensor streams only when appropriate to the existing behavior or requested design.
- Make units visible in timing names and expressions, such as `control_rate_hz` or `std::chrono::milliseconds(20)`.
- Preserve timer type, frequency, clock source, and timeout semantics.
- Group related parameter declarations. Use explicit parameter types and validate values whose invalidity could cause unsafe or undefined behavior.
- Avoid a configuration class for a handful of simple parameters.

## State, concurrency, and ownership

- Prefer an explicit `enum class` and readable transitions when several booleans obscure mutually exclusive robot states.
- Do not introduce a state-machine framework without a real project need.
- Preserve executors, callback groups, locks, and threading assumptions during refactors.
- Keep mutex scope narrow and synchronization visible near shared mutable state.
- Prefer RAII. Avoid manual allocation unless an external C API requires it.
- Do not replace ownership types merely for visual consistency.

## Robotics math and TF2

- Use descriptive quantities and unit suffixes where they prevent ambiguity: `wheel_radius_m`, `linear_velocity_mps`, `angular_velocity_radps`.
- Keep formulas in readable stages rather than one compressed expression.
- Use clear transform direction names such as `map_to_base_transform`.
- Preserve frame direction, coordinate convention, units, and lookup timing.
- Handle expected transform failures intentionally and avoid high-rate log flooding.

## Naming, includes, and comments

- Prefer domain names such as `WaypointNavigator`, `publishCommand()`, and `has_goal_` over vague `Manager`, `Handler`, `Data`, or `obj` unless the generic term is truly the domain concept.
- Avoid `m_`, `p_`, and `ptr_` prefixes unless the project uses them consistently.
- Keep message types visible when they communicate meaning. Use aliases only for genuinely long repeated types or established local style.
- Follow existing include grouping. Remove unused includes only when compilation can confirm transitive dependencies were not relied on.
- Comments should explain why: physical assumptions, surprising ROS behavior, frame/unit conventions, safety constraints, or workarounds.

## Logging and failures

- Use `RCLCPP_*` macros for normal node diagnostics.
- Prefer actionable messages that help diagnose robot state.
- Use throttled logging when a recurring high-rate condition must be visible.
- For expected runtime failures, return or enter a safe state and log when useful.
- For unrecoverable initialization failures, fail clearly with the actual cause.
- Do not swallow exceptions or use broad `catch (...)` blocks without a meaningful recovery action.

## Common AI-style complexity to remove

Look for unnecessary nested or immediately invoked lambdas, excessive `auto`, template machinery, `std::function` indirection, generic wrappers, tuples in place of named data, speculative helper classes, giant constructors/callbacks, magic numbers, nested ternaries, long method chains, and abstractions with only one call site.

Remove them only when the result is clearer and behavior is preserved. A straightforward implementation is desirable for infrastructure and robot-control code, but should remain professional and technically correct.

## Final readability check

The code should let an engineer quickly answer:

- Which topics, services, and actions does the node use?
- Which parameters, frames, rates, and units matter?
- What does each callback do?
- Where is the control or navigation logic?
- What state is maintained, and how does it transition?
- What happens when data is invalid or stops arriving?
- How is concurrency controlled?
- Which logs help diagnose a failure?
