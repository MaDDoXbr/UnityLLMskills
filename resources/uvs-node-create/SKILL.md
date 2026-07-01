---
name: uvs-node-create
description: Create or edit custom Unity Visual Scripting nodes, units, descriptors, and widgets for Planet Bingo. Use when the user asks for UVS, Unity Visual Scripting, Bolt, graph units, custom nodes, Unit classes, UnitDescriptor classes, or UnitWidget classes.
---

# UVS Node Create

## Purpose

Use this skill when producing custom Unity Visual Scripting (UVS) nodes for this project.

The project requires `com.unity.visualscripting` and Community Addons for Unity Visual Scripting (`dev.bolt.addons`). Follow the Community Addons style from `Library/PackageCache/dev.bolt.addons@*/Runtime/Nodes` and `Library/PackageCache/dev.bolt.addons@*/Editor/Nodes`.

## Required Conventions

- Runtime node/unit classes inherit from `Unity.VisualScripting.Unit`.
- Custom widget classes must inherit from `Unity.VisualScripting.Community.UnitWidget<TNode>`, not `Unity.VisualScripting.UnitWidget<TNode>`.
- Use `namespace Unity.VisualScripting.Community` unless an existing feature folder clearly establishes a more specific namespace.
- Put runtime nodes outside `Editor` folders.
- Descriptor scripts must be placed within a folder named exactly `Editor`.
- Widget scripts must be placed within a folder named exactly `Editor`.
- Do not edit `Library/PackageCache`, `Assets/Unity.VisualScripting.Generated`, or package source files. Create project scripts under `Assets/`.
- Apply the project's Unity C# conventions when writing `.cs` files.

## Workflow

1. Gather the node contract before coding: class name, category, title, node type, ports, execution behavior, output values, descriptor summaries, whether a custom widget is actually needed, and where the scripts should live.
2. Inspect nearby project code and Community Addons samples before choosing style. Good references include:
   - `Runtime/Nodes/Fundamentals/Utility/Counter.cs`
   - `Runtime/Nodes/Fundamentals/String/Stringbuilder/StringbuilderUnit.cs`
   - `Editor/Nodes/Fundamentals/Descriptors/StringBuilderUnitDescriptor.cs`
   - `Editor/Nodes/Fundamentals/Widgets/TodoWidget.cs`
3. Create the runtime `Unit` first. Define every port in `Definition()`, use `[DoNotSerialize]` on port fields, and use `[PortLabelHidden]` for default flow ports when appropriate.
4. Add relationships so the graph UI understands the node:
   - `Requirement(valueInput, controlInput)` when a control input reads a value.
   - `Requirement(valueInput, valueOutput)` when a value output reads a value directly.
   - `Succession(controlInput, controlOutput)` for each valid control path.
   - `Assignment(controlInput, valueOutput)` when executing a control input writes a value output.
5. Add a descriptor when the node needs search/inspector help beyond attributes. Descriptor classes inherit from `UnitDescriptor<TNode>`, use `[Descriptor(typeof(TNode))]`, and must live under an `Editor` folder.
6. Add a widget only for custom visual behavior. If generated, the widget must live under an `Editor` folder and inherit from `Unity.VisualScripting.Community.UnitWidget<TNode>`.
7. After creating or changing scripts, refresh/compile Unity, check console errors, and regenerate Visual Scripting units if the node does not appear in the fuzzy finder.

## Runtime Unit Template

```csharp
using Unity.VisualScripting;
using UnityEngine;

namespace Unity.VisualScripting.Community
{
    [UnitCategory("Community\\Planet Bingo")]
    [UnitTitle("Multiply Score")]
    [TypeIcon(typeof(float))]
    public sealed class MultiplyScoreNode : Unit
    {
        [DoNotSerialize]
        [PortLabelHidden]
        public ControlInput enter;

        [DoNotSerialize]
        [PortLabelHidden]
        public ControlOutput exit;

        [DoNotSerialize]
        public ValueInput score;

        [DoNotSerialize]
        public ValueInput multiplier;

        [DoNotSerialize]
        public ValueOutput result;

        protected override void Definition()
        {
            enter = ControlInput(nameof(enter), OnEnter);
            exit = ControlOutput(nameof(exit));

            score = ValueInput<float>(nameof(score), 0f);
            multiplier = ValueInput<float>(nameof(multiplier), 1f);
            result = ValueOutput<float>(nameof(result));

            Requirement(score, enter);
            Requirement(multiplier, enter);
            Succession(enter, exit);
            Assignment(enter, result);
        }

        private ControlOutput OnEnter(Flow flow)
        {
            float value = flow.GetValue<float>(score) * flow.GetValue<float>(multiplier);
            flow.SetValue(result, value);
            return exit;
        }
    }
}
```

For pure data nodes with no control flow, use `ValueOutput<T>(key, flow => ...)` and connect input dependencies with `Requirement(valueInput, valueOutput)`.

For coroutine nodes, use `ControlInputCoroutine(...)` and keep yielded work compatible with Unity's play/edit mode constraints.

## Descriptor Template

Save descriptor scripts under a path containing an `Editor` folder, for example `Assets/[Planet Bingo]/Game/Editor/VisualScripting/Descriptors/MultiplyScoreNodeDescriptor.cs`.

```csharp
using Unity.VisualScripting;

namespace Unity.VisualScripting.Community
{
    [Descriptor(typeof(MultiplyScoreNode))]
    public sealed class MultiplyScoreNodeDescriptor : UnitDescriptor<MultiplyScoreNode>
    {
        public MultiplyScoreNodeDescriptor(MultiplyScoreNode unit) : base(unit)
        {
        }

        protected override string DefinedSummary()
        {
            return "Multiplies a score by a factor and exposes the result.";
        }

        protected override void DefinedPort(IUnitPort port, UnitPortDescription description)
        {
            base.DefinedPort(port, description);

            switch (port.key)
            {
                case nameof(MultiplyScoreNode.enter):
                    description.summary = "Executes the score multiplication.";
                    break;
                case nameof(MultiplyScoreNode.exit):
                    description.summary = "Continues after the result is assigned.";
                    break;
                case nameof(MultiplyScoreNode.score):
                    description.summary = "The score to multiply.";
                    break;
                case nameof(MultiplyScoreNode.multiplier):
                    description.summary = "The multiplier applied to the score.";
                    break;
                case nameof(MultiplyScoreNode.result):
                    description.summary = "The multiplied score.";
                    break;
            }
        }
    }
}
```

Useful descriptor overrides include `DefinedTitle()`, `DefinedShortTitle()`, `DefinedSubtitle()`, `DefinedSurtitle()`, `DefinedSummary()`, `DefinedIcon()`, and `DefinedPort(...)`.

## Widget Template

Only create a widget when descriptor attributes are not enough. Save widget scripts under a path containing an `Editor` folder, for example `Assets/[Planet Bingo]/Game/Editor/VisualScripting/Widgets/MultiplyScoreNodeWidget.cs`.

```csharp
using Unity.VisualScripting;

namespace Unity.VisualScripting.Community
{
    [Widget(typeof(MultiplyScoreNode))]
    public sealed class MultiplyScoreNodeWidget : Unity.VisualScripting.Community.UnitWidget<MultiplyScoreNode>
    {
        public MultiplyScoreNodeWidget(FlowCanvas canvas, MultiplyScoreNode unit) : base(canvas, unit)
        {
        }
    }
}
```

## UVS Attributes

Common runtime attributes:

- `[UnitTitle("...")]` controls the search/title text.
- `[UnitShortTitle("...")]` controls compact graph display text.
- `[UnitSubtitle("...")]` adds subtitle text.
- `[UnitSurtitle("...")]` adds upper title text.
- `[UnitCategory("Community\\...")]` controls the fuzzy finder path.
- `[TypeIcon(typeof(SomeType))]` uses a Unity Visual Scripting type icon.
- `[SpecialUnit]` is for event-like/special graph units only.
- `[RenamedFrom("Old.Namespace.OldName")]` preserves serialized graph compatibility when renaming shipped nodes.

## Validation Checklist

- The runtime class compiles without editor-only references.
- Every port field is `[DoNotSerialize]`.
- Port keys are stable; prefer `nameof(field)` unless a shipped graph already depends on a literal key.
- Every control path has `Succession`.
- Every value read has `Requirement`.
- Every executed value assignment has `Assignment`.
- Descriptors and widgets are placed within a folder named exactly `Editor`.
- Custom widgets inherit from `Unity.VisualScripting.Community.UnitWidget<TNode>`.
- Unity compiles cleanly and the node appears after Visual Scripting unit regeneration.
