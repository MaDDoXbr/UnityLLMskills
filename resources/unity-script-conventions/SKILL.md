---
name: unity-script-conventions
description: >-
  Unity C# coding conventions for Planet Bingo. Use when creating or editing
  C# scripts, MonoBehaviours, ScriptableObjects, UnityEvents, SceneReference
  assets, or any Unity gameplay/UI code in this project.
---

# Unity Script Conventions

Apply these rules whenever writing or modifying C# and Unity code in this project.

## Core Principles

- **IMPORTANT** — Code should be built to be human-readable.
- Apply SOLID principles.
- All uses of tools, e.g., sed and grep, are allowed.

## Formatting

- Use TAB characters for indentation.
- All execution interruption commands (`return`, `continue`, `break`, `throw`, and `yield`) should take their own line, to facilitate reading.
- If there's only one execution line for an `if` command, don't use braces.
- When declaring variables, favor using `var` instead of the class type.

```csharp
// Good
if (isReady)
	return;

// Avoid
if (isReady) { return; }
```

## Naming

- Naming of variables (including fields and properties) and parameters should stay within the **4-word limit**.
- Use Microsoft's naming convention for variables and parameters; do not use underscores, prefer camelCase instead.
- When a parameter name conflicts with a property or field name, instead of using `this` for disambiguation, prefix the parameter with an underscore instead.

```csharp
// Good
public void Construct(IEventHub _eventHub)
{
	eventHub = _eventHub;
}

// Avoid
public void Construct(IEventHub eventHub)
{
	this.eventHub = eventHub;
}
```

- Assure method names aren't larger than four, max five words.
- All C# or UnityEvent names should start with `On`.

## Method Design

- Methods with fewer than 10 lines should generally be avoided in favor of commented sections, unless that logic block is a callback, event function or actively reused in the same source file.
- Avoid methods with more than 40 lines; break them up when it's needed.
- When creating or changing a method, check the cyclomatic complexity and **report if it is greater than 5**.

## Class Design

- Only use abstract classes for type-matching; favor interfaces and mix-ins instead.
- Don't use concrete members within abstract classes.

## Unity-Specific

- Favor Unity-native data structures and functions over generic C# structures.
  - Example: `Mathf.Approximately(a, b)` vs. `Math.Abs(a - b) < 0.00001f`
- Favor Unity-serializable data types (e.g. `List<>`, `UnityEvent`).
- When arranging class fields, organize relatable data using NaughtyAttribute decorators (`BoxGroup`, `Foldout`).

## Scene References

- **NEVER** require scene-to-scene references, except if it points to a children gameobject component. Use injection instead (if it's already set) or create a new `SceneReference` asset for indirection.
- Always create `SceneReference` scriptable assets within the project's `Assets/[VisualCore]/SceneReferences` folder. Create subfolders for different categories as required, but make it tidy and avoid deep nesting.

## Documentation

- Create XML comments for public methods to be used externally; describe the parameters in a concise manner.
- Only add comments in the code to express the **why** and not the **what**, especially for more complex algorithms.

## Workflow

- Creation of and addition to external tools or utility libraries should require approval by the human orchestrator.
- Never commit or push without permission.
