# Unity Quick Tests

[![Читать на русском](https://img.shields.io/badge/%D0%A7%D0%B8%D1%82%D0%B0%D1%82%D1%8C_%D0%BD%D0%B0_%D1%80%D1%83%D1%81%D1%81%D0%BA%D0%BE%D0%BC-blue)](README.ru.md)

Sometimes you just need to call a method quickly: check a calculation, run part
of a service, rebuild a cache, invoke an editor utility, or see whether the
logic works at all. In Unity, that often means writing extra boilerplate: a
temporary `MonoBehaviour`, an Inspector button, a menu item, a dedicated editor
service, a console command, or a full test.

`Unity Quick Tests` removes that boilerplate. Add an attribute to a
parameterless method, assign a keyboard shortcut or a schedule, and the package
finds suitable live instances and invokes the method at the appropriate point
in the Unity/editor lifecycle. Plain C# classes are registered using weak
references, so the package does not keep objects alive or complicate cleanup.

## Installation

Option 1: in Unity, open `Window/Package Manager`, click `+`, select
`Add package from git URL...`, and paste this URL:

```text
https://github.com/KapkanDruid/Unity-Quick-Tests.git#v1.1.2
```

Option 2: add a Git dependency to your Unity project's `Packages/manifest.json`:

```json
"com.urbandruids.unity-quick-tests": "https://github.com/KapkanDruid/Unity-Quick-Tests.git#v1.1.2"
```

## Usage

```csharp
using UnityQuickTests;
using UnityEngine;

public static class EditorSmokeTests
{
    [QuickTestHotkey(KeyCode.LeftControl, KeyCode.T)]
    private static void RunByHotkey()
    {
        Debug.Log("Ctrl+T");
    }

    [QuickTestSchedule(60, QuickTestScheduleUnit.Frames, QuickTestRepeatMode.Once)]
    private static void RunAfterFrames()
    {
        Debug.Log("After 60 editor updates");
    }

    [QuickTestSchedule(2.5, QuickTestScheduleUnit.Seconds, QuickTestRepeatMode.Repeat)]
    private static void RunEverySeconds()
    {
        Debug.Log("Every 2.5 seconds");
    }
}

public sealed class MonoSmokeTest : MonoBehaviour
{
    [QuickTestHotkey(KeyCode.LeftControl, KeyCode.M)]
    private void RunOnLiveMonoBehaviour()
    {
        Debug.Log(name);
    }
}

public sealed class PlainServiceSmokeTest
{
    [QuickTestHotkey(KeyCode.LeftControl, KeyCode.P)]
    private void RunOnLiveService()
    {
        Debug.Log("Auto-registered plain C# service");
    }
}
```

## Supported Features

- `static` methods are invoked directly.
- Methods on `MonoBehaviour`, `ScriptableObject`, and `EditorWindow` are invoked
  on instances that are already loaded.
- Methods on plain C# classes that do not inherit from `UnityEngine.Object`
  are invoked through instance registration.
- Methods can be triggered by a keyboard shortcut or scheduled in editor
  update ticks or seconds.
- Schedules support `Once` and `Repeat` modes.

## Limitations

- A method marked with a package attribute must return `void` and take no
  parameters.
- `async void`, `Task`, `ValueTask`, `UniTask`, and methods that return a value
  are not supported. Use a regular `void` wrapper for these cases.
- Generic methods and generic types are not supported.
- If a method with a quick-test attribute is declared in a base class, the
  package does not create separate tests for each derived class. To target a
  specific derived class, add a separate wrapper method to it.
- The package does not create objects. Instance methods are invoked only on
  existing instances.
- If no suitable instances are found, the method is not invoked.
- If multiple suitable instances are found, the method is invoked on each one.
- `MonoBehaviour` instances are found among loaded scene objects, including
  inactive ones.
- `ScriptableObject` and `EditorWindow` are supported only if the object is
  already loaded. The package does not scan the project through `AssetDatabase`.
- Plain C# classes that do not inherit from `UnityEngine.Object` are supported
  through instance registration. The package normally injects registration
  into the constructor using an IL PostProcessor. If an object is created in a
  nonstandard way and is not registered automatically, you can call
  `QuickTestInstanceRegistry.Register(this)` manually.
- Plain C# instance registration stores weak references (`WeakReference`), so
  quick-tests do not keep these objects alive or prevent the garbage collector
  from reclaiming them when they are no longer needed.
- In Edit Mode, keyboard shortcuts work through `Scene View` events, so they
  are not a fully global hotkey system for all editor windows.
- Only the attributes are included in player builds as ordinary metadata. The
  code that discovers and runs quick-tests operates only in the editor.

## Diagnostics

The `Tools/Unity Quick Tests/List Registered Tests` menu lists discovered tests,
including their trigger, method signature, declaring type, target search scope,
support status, and warnings about conflicting keyboard shortcuts.

You can configure warnings through
`Tools/Unity Quick Tests/Warning Settings`.

## Testing

Instructions for automated checks are available in
[`Docs/TESTING.md`](Docs/TESTING.md) (in Russian).
