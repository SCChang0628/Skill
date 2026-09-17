---
name: ros2-cpp-human-refactor
description: Write or refactor ROS 2 C++ so it is readable, conventional, maintainable, and consistent with the surrounding package. Use for new rclcpp code, cleanup of AI-generated or over-engineered nodes, and readability-focused refactors. Preserve behavior and public ROS interfaces unless the user explicitly requests changes.
---

# ROS 2 C++ Human Refactor

Produce code that a robot engineer can understand and debug quickly. Prefer simple, explicit ROS 2 patterns over clever abstractions, and match the repository before imposing a new style.

## Inspect before editing

Read the relevant package context when available:

- `CMakeLists.txt`, `package.xml`, and `.clang-format`;
- related headers, sources, launch files, parameter YAML, and interface definitions;
- sibling nodes that reveal naming, layout, logging, QoS, and callback conventions.

Determine the ROS distribution and C++ standard when they affect compatibility. Identify the node's ROS interfaces, state, timing, control flow, concurrency model, units, frames, and safety behavior before changing code.

For detailed review criteria, read [references/refactor-guide.md](references/refactor-guide.md). Use the sections relevant to the current code; do not turn every recommendation into a mechanical rewrite.

## Preserve behavior by default

During a cleanup, do not silently change:

- topic, service, action, parameter, frame, launch-argument, or public API names;
- parameter defaults, message semantics, QoS, timing, clocks, or timeout behavior;
- executor, callback-group, threading, synchronization, or ownership semantics;
- state transitions, kinematic formulas, coordinate conventions, units, or safety limits;
- package dependencies or build behavior.

If a behavior change is required, separate it from the readability work and explain why. Never hide a bug fix or architecture change inside a style-only refactor.

## Write human-readable C++

- Prefer direct control flow, descriptive names, focused functions, RAII, and visible ROS interfaces.
- Match local choices for lambdas versus member callbacks, brace style, indentation, member suffixes, header/source separation, and include order.
- Use `auto` only when the type is obvious or spelling it harms readability. Keep domain, ROS message, precision, and unit-bearing types visible when useful.
- Use early returns to reduce meaningful nesting, but not mechanically.
- Separate substantial domain/control logic from ROS plumbing when that makes either easier to test or understand.
- Use named constants or variables for important rates, durations, frames, thresholds, and physical quantities.
- Comment non-obvious reasons, assumptions, units, frames, safety constraints, and workarounds—not obvious statements.

Avoid abstractions without a present need: generic wrappers around basic rclcpp APIs, factory/strategy layers for one node, speculative utility classes, complex templates, nested lambdas, tuple-shaped domain data, excessive optionals, and long STL expression chains. Do not modernize code merely to use newer language features.

## Keep ROS 2 structure obvious

A reader should be able to locate publishers, subscriptions, timers, services, clients, actions, parameters, callbacks, state, and business logic without tracing generic containers or dispatch layers.

Use clear purpose-based names such as `cmd_vel_sub_`, `status_pub_`, `control_timer_`, `odomCallback()`, and `publishCommand()`. Make source/target frames and physical units explicit where ambiguity is dangerous.

Preserve QoS unless the task requires changing it. Use ROS logging rather than `std::cout` for node diagnostics, and avoid flooding high-frequency paths. Keep service and action request, validation, execution, feedback, and result handling easy to follow.

Do not add multithreading, callback groups, mutexes, atomics, or lock-free structures without a demonstrated requirement. When they already exist, keep synchronization and shared state easy to identify.

## Implement proportionally

For existing code, make the smallest changes that materially improve readability. Split giant callbacks or constructors when named stages clarify the work; do not replace them with dozens of one-line helpers.

For a new node, imitate a suitable neighboring node, expose its ROS interfaces clearly, use ROS parameters for ordinary configuration, and avoid framework layers or asynchronous complexity not required by the request.

Update `CMakeLists.txt` and `package.xml` only when the code genuinely requires it, following the package's existing dependency style.

## Verify

After editing:

1. Recheck that public ROS interfaces and behavior remain unchanged unless explicitly requested.
2. Format only with the repository's existing formatter/configuration; do not add a formatter config unless asked.
3. Build the narrowest affected package, normally with `colcon build --packages-select <package>`, adding the workspace's established options such as `--symlink-install` when appropriate.
4. Run relevant tests or linters when available.
5. If verification cannot run, say so; never claim compilation or tests passed without running them.

Conclude with a concise summary of what was simplified, whether external behavior or ROS interfaces changed, and the exact verification performed.
