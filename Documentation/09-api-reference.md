# API reference

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

This page is for programmers. Everything the artist-facing tools do is covered in chapters
[Quick start](01-quick-start.md) through [Hair](06-hair.md); what follows is the code underneath
them. Troubleshooting has its own page.

Choose a programming task:

- [Drive the face from code](#drive-the-face-from-code).
- [Write your own pose stage](#write-your-own-pose-stage).
- [Follow LOD and visibility](#follow-lod-and-visibility).
- [Script hair and cards tools](#hair-and-cards-tools).
- [Find a public type](#type-index).

For a setup or playback problem, use [Troubleshooting](08-troubleshooting.md).

One thing is worth saying before the reference starts. MHBridge does not hide the engine from
you. A type or a member is `internal` only where calling it from outside would break an invariant,
and a comment at the declaration says which invariant it protects. The source ships with the
package, so if your case really needs one of them, widening the modifier yourself is open to you.

## On this page

- [Assemblies and namespaces](#assemblies-and-namespaces)
- [Where MHBridge runs in your frame](#where-mhbridge-runs-in-your-frame)
- [Write your own pose stage](#write-your-own-pose-stage)
- [Drive the face from code](#drive-the-face-from-code)
- [Play facial animation as an ordinary clip](#play-facial-animation-as-an-ordinary-clip)
- [Body correctives over the Animator](#body-correctives-over-the-animator)
- [Follow LOD and visibility](#follow-lod-and-visibility)
- [Talk to RigLogic directly](#talk-to-riglogic-directly)
- [The Face Board from code](#the-face-board-from-code)
- [ARKit mapping and take data](#arkit-mapping-and-take-data)
- [Editor hooks](#editor-hooks)
- [Extend Validate Setup with a check of your own](#extend-validate-setup-with-a-check-of-your-own)
- [Hair and cards tools](#hair-and-cards-tools)
- [Type index](#type-index)

## Assemblies and namespaces

The package has four core assemblies and a separate demo assembly.

| Assembly definition | Where | What is in it |
|---|---|---|
| `MHBridge.Runtime` | `Assets/MHBridge/Runtime` | The character components, the Face Board, the ARKit data types, and the native layer. |
| `MHBridge.Editor` | `Assets/MHBridge/Editor` | Importers, the character builder, the tools, the validator. |
| `MHBridge.Hair.Runtime` | `Assets/MHBridge/Hair/Runtime` | `MHHairLodHelper` and `MHHairTopologyPin`. |
| `MHBridge.Hair.Editor` | `Assets/MHBridge/Hair/Editor` | The Hair Tool, the Cards Tool, and the Scalp Mask Tool. |
| `MHBridge.Demo` | `Assets/MHBridge/Demo/Scripts` | `MHDemoController`, which validates the demo character, drives its buttons and camera, and binds the Face Board. |

Both hair assemblies compile in any project. What depends on `com.unity.demoteam.hair` is the code
inside them: the version define `MHBRIDGE_HAS_HAIR` compiles out everything that touches the hair
package, so without it `MHHairLodHelper` is an inert component and the Hair Tool opens on its setup
page. The Cards Tool is the exception that needs nothing — a hair-cards mesh is an ordinary rigid or skinned
mesh, so `MHCardsBuilder` touches no hair-package type and works in a project that has none of
those packages.

The namespace is `MHBridge` for everything in both runtime assemblies, and `MHBridge.Editor`
with the sub-namespaces `.Import`, `.Build`, `.Validation`, `.Arkit`, `.Clothing`, `.Materials`,
`.UnrealAnim`, and `.Hair` for the editor side. Demo code uses `MHBridge.Demo`.

## Where MHBridge runs in your frame

This is the one piece of the package you need to understand before writing anything
against it: **a character's pose is computed in the animation phase of the frame, not in
`LateUpdate` or `Update`.**

MHBridge installs a node of its own into Unity's player loop, in the `PreLateUpdate` phase,
immediately before the first of `ConstraintManagerUpdate` and `ScriptRunBehaviourLateUpdate`.
Every pose stage runs from there, so by the time constraints are solved the skeleton is final.

![A diagram of one frame: the Animator writes the pose, the MHPoseLoop node runs the MHBridge stages in order, then constraints, physics and LateUpdate all read a finished skeleton](images/03-animation/23-pose-chain-diagram.png)

Inside the node, stages run in ascending `PoseStageOrder`:

| Order | Stage | What it writes |
|---|---|---|
| 100 | `MHBody` | Corrective and twist joints, on top of the Animator's pose. |
| 150 | `MHFaceClipDriver` | The clip's 251 control fields, as raw controls, plus head motion on the `head` bone. |
| 200 | `MHFace` | Facial joints, blend shapes, wrinkle weights, pupil aperture. |
| 300 | `MHHairLodHelper` | Per-groom LOD, quality, and card switching. |
| 310 | `MHHairTopologyPin` | Nothing. It holds strand topology meshes alive. |

Three consequences, which cover most of what you will want to do:

- **To read a finished pose** — for a constraint, an IK pass, an accessory, a bone collider —
  read it from your ordinary `LateUpdate`. The skeleton is final by then.
- **To write facial controls**, write them from `Update`, or from a pose stage ordered below
  200. A write from `LateUpdate` lands after the face has already evaluated, so it applies on
  the *next* frame.
- **To run inside the chain**, derive from `MHPoseStageBehaviour`. See
  [Write your own pose stage](#write-your-own-pose-stage).

> [!WARNING]
> `[DefaultExecutionOrder]` on our components is not what orders the chain. Script execution
> order sorts `LateUpdate`, which Unity runs after the pose node has already finished, so it
> cannot move anything into the animation phase. The attribute is there for the fallback below,
> and it is kept equal to `PoseStageOrder` so that both paths produce the same order.

If the node cannot be installed — a project with a custom player loop — every stage falls back to running from its own `LateUpdate`. That is correct,
but a frame stale for anything that reads bones earlier in the frame. The Console says so:

```text
[MHBridge] Could not add the pose phase to the PlayerLoop because PreLateUpdate has no supported insertion point. MHBridge is using LateUpdate, so hair and constraints are one frame behind the head.
```

`MHPoseLoop.Injected` reports the same thing to code, and it is the first value worth logging
when hair or constraints start lagging.

## Write your own pose stage

Derive from `MHPoseStageBehaviour`, give the stage an order, and put the work in
`RunPoseStage()`:

```csharp
using MHBridge;
using UnityEngine;

// The attribute orders the LateUpdate fallback; the property orders the normal path. Keep the
// two numbers equal.
[DefaultExecutionOrder(250)]
public sealed class MyAccessoryDriver : MHPoseStageBehaviour
{
    // After the face (200), before the hair helpers (300).
    public override int PoseStageOrder => 250;

    protected override void RunPoseStage()
    {
        // Every stage below this order has already written its bones this frame.
    }
}
```

Three rules, each with a reason:

- **Declare no `LateUpdate` of your own.** The base class uses it for the fallback and stamps
  the frame it ran, so a stage runs exactly once per frame on either path. A `LateUpdate` in
  your subclass hides the base one and loses the fallback.
- **Call `base.OnEnable()` and `base.OnDisable()`** if you override them. That is where a stage
  registers and unregisters itself.
- **Pick an order relative to the occupied ones**: 100, 150, 200, 300, and 310. Below 200 means
  "before the face evaluates"; above 310 means "after everything MHBridge does".

## Drive the face from code

The face has two control sets, and a single switch decides which of them reaches RigLogic. Get
that straight first; the rest is mechanical.

### GUI controls and raw controls

| | GUI controls | Raw controls |
|---|---|---|
| What they are | The semantic handles of the MetaHuman control board, such as `CTRL_L_eye.ty`. | The inputs RigLogic evaluates, such as `CTRL_expressions.jawOpen`. |
| How many | `GuiControlCount` | `RawControlCount` |
| Names | `GetGuiName(int)` | `GetRawControlName(int)` |
| Index from a name | `IndexOfGuiControl(string)` | `IndexOfRawControl(string)` |
| Read and write | `GetGui(int)`, `SetGui(int, float)` | `SetRawControl(int, float)` |
| Clear | `ResetGui()` | `ResetRawControls()` |
| How they reach the rig | Mapped onto raw controls on every evaluation. | Pushed straight in, the whole set in one call. |
| What they need | `enableGuiControls` set to `true`. | Nothing, but an enabled GUI set overwrites every raw control it maps to. |
| Who writes them | The Face Board, and your own `SetGui` calls. | `MHFaceClipDriver`, an ARKit clip, your own solver. |

`enableGuiControls` is that switch, it is off by default, and no component ever touches it. Only
you do, through the `MHFace` inspector or through the **Control face from GUI** checkbox on the
board, which is the same field. Off, the face plays raw controls, which is what a clip writes.
On, the board's values overwrite every raw control they map to, so a clip playing at the same
time stops reaching the face.

Both control sets live on `MHFace` rather than in the native rig. That matters more than it
sounds — see [The raw control mirror](#the-raw-control-mirror).

### Set one control by name

`SetControl` looks the name up in the GUI set first and in the raw set second, and returns
`false` when neither carries it:

```csharp
// "jawOpen" as the rig knows it. Returns false when this DNA has no such control.
face.SetControl("CTRL_expressions.jawOpen", 0.35f);
```

That is the right call for a script that touches one or two controls occasionally.

### Set controls by index every frame

A name lookup is a linear scan over roughly 250 strings, so resolve the index once and keep it:

```csharp
using MHBridge;
using UnityEngine;

[DefaultExecutionOrder(175)]
public sealed class MyLipSync : MHPoseStageBehaviour
{
    // Between the clip driver (150) and the face (200).
    public override int PoseStageOrder => 175;

    [SerializeField] private MHFace face;
    private int _jawOpen = -1;

    protected override void RunPoseStage()
    {
        // The rig comes up in Awake, so resolve here rather than in a field initializer.
        if (face == null || !face.IsInitialized) return;
        if (_jawOpen < 0) _jawOpen = face.IndexOfRawControl("CTRL_expressions.jawOpen");
        // Raw expression controls run from 0 to 1. Clamping is yours to do — see the warning.
        if (_jawOpen >= 0) face.SetRawControl(_jawOpen, Mathf.Clamp01(AmplitudeThisFrame()));
    }

    // Replace with your own source: an audio envelope, a network stream, a solver.
    private float AmplitudeThisFrame() => 0f;
}
```

> [!WARNING]
> A driver of your own has to clamp raw controls to the range 0 to 1 itself, and this is not
> cosmetic. RigLogic evaluates the wrinkle maps through a conditional table whose rows fire only
> for an input up to and including 1.0, so a value of 1.004 fires no row at all and the wrinkle
> weight drops to zero for that frame. Imported FBX curves overshoot by about that much on a
> plateau, which is why our own clip driver applies `Mathf.Clamp01` before every write.

### The raw control mirror

`SetRawControl` writes into a managed array on `MHFace`, and the whole array is pushed into the
native rig in one call at the start of every evaluation. Roughly 250 marshalling transitions per
character per frame collapse into one.

![A diagram of the control flow: SetRawControl writes the mirror array, one batched push sends it to the rig, the optional GUI mapping then overwrites the mapped entries, and Calculate produces the joint, blend shape and animated map outputs](images/09-api-reference/01-control-flow-diagram.png)

The consequence is the part that is easiest to miss: **writing raw controls into
the native rig directly, through an `MHRig` of your own, does nothing.** The next push overwrites
it. Go through `MHFace.SetRawControl`.

### What happens when both sets are written

One evaluation writes controls in two steps, in this order:

1. It pushes the whole mirror array into the rig — every value `SetRawControl` and the clip driver
   left there this frame.
2. Only while `enableGuiControls` is on, it sends the values from `SetGui` and calls
   `MapGuiToRawControls`, which turns each board handle into the raw controls that handle drives
   and writes those over what step 1 left.

So while the switch is on, the board wins every raw control it reaches, and only a raw control no
handle reaches keeps the value your code wrote. Which raw controls one handle covers is fixed by
the DNA's own mapping table and is nothing you can configure, so treat the switch as an either-or
rather than as a blend:

```csharp
// Hand the face to the board. Every raw control a handle maps to now comes from the GUI.
face.enableGuiControls = true;
face.SetGui(face.IndexOfGuiControl("CTRL_C_jaw.ty"), 0.4f);

// Hand it back to the raw controls: an animation clip, an ARKit take, or a solver of your own.
face.enableGuiControls = false;
face.SetRawControl(face.IndexOfRawControl("CTRL_expressions.jawOpen"), 0.4f);
```

### Reserve a bone for yourself

`MHFace` returns every facial joint it owns to neutral each frame. `ReserveJoint` hands one bone over:

```csharp
// From here on MHFace leaves the head bone alone, and this component owns it.
face.ReserveJoint("head");

// Hand it back. MHFace drives the bone again from the next evaluation.
face.ReleaseJoint("head");
```

Reserve when you start driving the bone and release when you stop — `IsJointReserved(string)`
reports the current state. Both calls take the bone's name in the skeleton and return `false` when
no such bone exists, so reserve in `Awake` or `Start` rather than per frame.

A release only undoes your own reservation. The joints the importer excluded from the drive filter
on a head and body character stay excluded, because those are the body's to write and the face was
never allowed to drive them.

That is exactly how `MHFaceClipDriver` owns the `head` bone: it reserves while **Apply Head
Rotation** or **Apply Head Translation** is on, and when you turn both off it puts the bone back at
its captured neutral and releases it, so the body clip gets the head back.

### Read the state

| Member | On | What it reports |
|---|---|---|
| `IsInitialized` | `MHFace`, `MHBody` | Whether the native rig is loaded. False until `Awake`, or until `MHFace.EnsureInitialized` succeeds. |
| `LOD` | `MHFace`, `MHBody` | The level RigLogic is set to. It is written on every evaluation, so it reports rather than controls. |
| `ActiveLod` | `MHFace`, `MHBody` | The level last evaluated, or `-1` while the character is off-screen. `0` outside Play mode. |
| `LODCount` | `MHFace`, `MHBody` | How many levels that component's DNA carries: eight for a MetaHuman head, four for the body. |
| `HeadRenderer` | `MHFace` | The head skinned mesh, which is what the hair tools resolve root UVs against. |

`EnsureInitialized()` loads the DNA and allocates the buffers. Call it to
bring a rig up outside Play mode, for an editor preview of your own, then call `Evaluate()` to
compute a pose and `ShutdownRig()` to release the native memory. In Play mode `Awake` and the
pose phase do all three for you.

### Importer wiring you normally do not call

`Configure`, `SetBlendShapeChannelMap`, `SetHeadRenderer`, `SetVisibilityRenderers`,
`SetEyeRenderers`, `SetDriveFilter`, and `PruneVisibilityRenderers` are public because the
importer lives in another assembly and has to reach them. They fill in the serialized fields an
import produces. Call them only if you build a character from a DNA yourself, outside our
importer — and in that case read `CharacterBuilder` first, because getting the parallel arrays
wrong produces a character that half works.

`MHFace.SetEyeRenderers(Renderer[] left, Renderer[] right)` accepts one renderer per LOD
for each eye. Eyeballs normally use `MeshRenderer`; the head and the lid-related parts use
`SkinnedMeshRenderer`.

`MHFace.applyPupilDilation` defaults to `true`. It drives `_PupilAperture` from RigLogic's
wide and narrow animation outputs, using the material's authored neutral separately for
each eye. The neutral falls back to `0.5` only when no eye material provides that property.
The result is `Clamp01(neutral + (1 - neutral) * wide - neutral * narrow)` and is written
through a `MaterialPropertyBlock` to all LOD renderers of the eye. Missing or zero rig
channels produce the authored neutral. See [Pupil](05-materials.md#pupil) for
material limits and appearance tuning.

## Play facial animation as an ordinary clip

`MHFaceClipDriver` is the component an `AnimationClip` binds to. It carries 251 public float
fields, one per raw expression control, plus six head channels. Unity's Animator writes the
fields; the driver reads them at stage 150 and pushes them into the face as raw controls.

The fields exist because Unity does not animate array elements on a `MonoBehaviour`. Named
fields it does animate, so the field set is generated from the DNA's control list.

| Member | What it is |
|---|---|
| `ControlCount` | 251, the number of animatable control fields. |
| `ControlRawNames` | The DNA raw control names, in field order. |
| `FieldNames` | The field identifiers: a raw control name with the dot replaced by an underscore. |
| `HasField(string)` | Whether a name is one of the driver's fields. Use it to validate a curve target. |
| `ReadPose(float[] reuse = null)` | The current pose, in `ControlRawNames` order. Pass the same buffer back to record a take with no per-frame allocation. |
| `applyHeadRotation`, `applyHeadTranslation` | Whether the take's head channels reach the `head` bone. |
| `HeadChannelNames` | The six head channel names. |

To generate a clip in code, bind your curves to `FieldNames[i]` on this component's type. To
record what a character is doing, poll `ReadPose` from a stage ordered above 150.

Head motion is not a DNA control. `HeadYaw`, `HeadPitch`, and `HeadRoll` are degrees,
`HeadTranslationX`, `HeadTranslationY`, and `HeadTranslationZ` are centimetres, all six are
applied as an absolute offset from the bone's captured neutral, and none of them is clamped.

## Body correctives over the Animator

`MHBody` reads the local rotation of 44 driver bones, feeds them to the body RigLogic as pose
deltas, and applies the resulting corrective and twist output to the joints the Animator does
not own.

| Member | Default | What it does |
|---|---|---|
| `applyCorrectives` | `true` | The master switch. Turning it off releases the corrective joints to neutral once, rather than leaving them stuck at the last pose. |
| `maxCorrectiveLod` | `2` | Past this level the Animator's pose alone is used. The body carries four levels. |
| `ignoreClipBoneTranslations` | `true` | Pins every bone to its bind local position each frame. The `root` and `pelvis` bones keep the clip's translation, so locomotion still works. |
| `IsInitialized`, `LOD`, `ActiveLod`, `LODCount` | | The same four read-only members as on `MHFace`, over the body's four levels. |

Turn `ignoreClipBoneTranslations` off in two cases: the clip carries deliberate bone translation,
such as IK limb stretch, or it was retargeted onto your own character's skeleton and carries no
foreign proportions. The full explanation is in
[Bone translations from a retargeted clip](03-animation.md#bone-translations-from-a-retargeted-clip).

On a character with both a head and a body, this is who owns what:

| Bones | Written by | The switch |
|---|---|---|
| The body's core bones | The `Animator`, from the clip | — |
| Corrective and twist joints | `MHBody` | `applyCorrectives`, `maxCorrectiveLod` |
| Facial joints | `MHFace` | — |
| Neck and head | The body clip, by default | `MHFace.driveHeadNeck`, off by default |
| The `head` bone, as an offset | `MHFaceClipDriver` | `applyHeadRotation`, `applyHeadTranslation` |
| Any bone you claim | You | `MHFace.ReserveJoint(name)`, until `MHFace.ReleaseJoint(name)` |

## Follow LOD and visibility

- `MHFace.ActiveLod` and `MHBody.ActiveLod` report the most detailed level any camera is
  currently showing, `-1` while nothing is visible, and `0` outside Play mode. On a character
  with a head and a body, take the band from `MHFace.ActiveLod`: the head carries eight levels
  and the body four, so the head reports the finer band. That is what `MHHairLodHelper` reads.
- `MHLodVisibility.MinVisibleLod(SkinnedMeshRenderer[])` and
  `MHLodVisibility.Pruned(SkinnedMeshRenderer[])` are that same logic, public, for a renderer
  set of your own.

> [!WARNING]
> While no renderer of a character is visible, RigLogic is not evaluated at all — not at a lower
> rate, not at a coarser level. Your code cannot assume that the pose is refreshed every frame.
> The pattern that works is the one `MHHairLodHelper` uses: while off-screen, change nothing and
> leave the last state alone.

Two limits are fixed in the runtime and no setting raises them. Blend shapes exist on LOD 0
only, and the wrinkle maps stop at LOD 1, because RigLogic returns zero animated-map weights past
it. `maxAnimatedWrinkleLod` on `MHFace` can only narrow that further.

## Talk to RigLogic directly

`MHRig` is the public `IDisposable` wrapper over the native handle chain: the DNA, the RigLogic
evaluator, and one rig instance. Reach for it when you want RigLogic without a character in a
scene — a batch pose solve, an offline bake, a tool of your own.

```csharp
using MHBridge;
using UnityEngine;

public static class RigProbe
{
    // dna is the "dna_head" sub-asset of an imported .dcc or .dna asset.
    public static void LogJawOpenEffect(DnaBinaryAsset dna)
    {
        using MHRig rig = MHRig.LoadFromMemory(dna.Bytes, MHBridgeNative.LAYER_ALL);
        if (rig == null)
        {
            Debug.LogError($"DNA failed to load: {MHRig.LastError}");
            return;
        }

        int jaw = -1;
        for (int i = 0; i < rig.RawControlCount; ++i)
            if (rig.RawControlName(i) == "CTRL_expressions.jawOpen") { jaw = i; break; }
        if (jaw < 0) return;

        rig.LOD = 0;
        rig.SetRawControl(jaw, 1f);
        rig.Calculate();

        // Nine floats per joint: [Tx, Ty, Tz, Rx, Ry, Rz in degrees, Sx, Sy, Sz].
        var joints = new float[rig.JointOutputFloatCount];
        int written = rig.GetJointOutputs(joints);
        Debug.Log($"jawOpen = 1 produced {written / 9} joint deltas of {rig.JointCount}.");
    }
}
```

The per-frame sequence is always the same: set controls, optionally call
`MapGuiToRawControls()`, then `Calculate()`, then read `GetJointOutputs`,
`GetBlendShapeOutputs`, or `GetAnimatedMapOutputs`. `LoadFromMemory` is the path that works in a
build; `LoadFromFile` reads a `.dna` from disk and is for editor code. Both return `null` on
failure and put the reason in `MHRig.LastError`.

Metadata comes off the same object: `LODCount`, `GuiControlCount`, `RawControlCount`,
`JointCount`, `BlendShapeChannelCount`, `AnimatedMapCount`, and `JointOutputFloatCount`, the
matching `*Name(int)` accessors, `JointParentIndex`, `GetNeutralJointRotation`,
`GetNeutralJointValues`, and `TranslationScale`.

To put joint output onto a `Transform`, call `MHConvert.ApplyJointLocal(bone, values, baseIndex,
translationScale)`, which is what both the importer and the runtime use. DNA joints live in
Maya's space — right-handed, Y up, centimetres, XYZ euler degrees — and the conversion to Unity
is an X mirror. `MHConvert.EulerToUnity` and `MHConvert.EulerToDnaQuat` are the two halves of
that if you need them on their own.

> [!WARNING]
> Three rules, each with a consequence.
>
> - **`Dispose` is mandatory.** These are native allocations and the garbage collector will not
>   free them. A `using` statement is the shortest correct form.
> - **`MHRig` is not thread-safe.** A rig instance holds per-character state. One instance per
>   character, driven from one thread.
> - **Do not write into a rig an `MHFace` owns.** The mirror push overwrites it. See
>   [The raw control mirror](#the-raw-control-mirror).

Under `MHRig` sits `MHBridgeNative`, the raw P/Invoke layer into `MHBridgeRig.dll`. It is public
and you may call it, but its signatures follow the native ABI of one adapter build rather than a
managed contract, so it can move between package versions in ways `MHRig` will not. Two things
in it are worth knowing regardless: `MHBridgeNative.GetVersion()` returns the native plugin
version, which belongs in any bug report, and `LAYER_ALL` and `LAYER_RUNTIME` are the data layer
constants the load calls take.

> [!CAUTION]
> By default an import ships a trimmed DNA, without its geometry layer. Every index survives the
> trim — joints, controls, blend shape channels, animated maps — but a mesh query against it
> returns nothing useful, and `MHR_GetMeshBlendShapeChannelIndex` in particular returns 0 without
> reporting an error. If your code reads mesh data out of the runtime DNA, set **Runtime DNA** to
> `Full` on the character's Model tab. See
> [Trim Runtime DNA](04-import-reference.md#trim-runtime-dna).

## The Face Board from code

The board is data, state, and a view, and the three are separate on purpose. That is what lets
you pose a face with no board on screen at all.

| Type | What it is |
|---|---|
| `FaceBoardAsset` | The baked layout: `controls`, `decorations`, `links`, and the board extents. |
| `FaceBoardStates.For(MHFace, FaceBoardAsset)` | The supported way to obtain a pose state. It keys states by character, so you get the one the editor window and the runtime panel already share. |
| `FaceBoardState` | The pose: `Count`, `HandlePos(int)`, `SetHandlePos(int, Vector2)`, `IndexOfControl(string)`, `GetTranslation(int)`, `SetTranslation(int, Vector2)`, `Reset()`, `Resolve(MHFace)`, `Apply(MHFace)`, `Invalidate()`, and `PoseHash()`. |
| `FaceBoardPose` | A pose as a JSON file: `Capture(FaceBoardState)`, `ApplyTo(FaceBoardState)`, `ToJson()`, `FromJson(string)`, `SaveWithDialog(FaceBoardState)`, and `LoadWithDialog(FaceBoardState)`. |
| `FileDialogs` | Open and save file dialogs in the editor and in a Windows build: `OpenFile`, `SaveFile`, and `IsSupported`. |
| `FaceBoardElement` | The view, an ordinary `VisualElement`. Events `onChanged`, `onResetPose`, `onSavePose`, `onLoadPose`, and `onGuiControlsChanged`, plus `Refresh()` and `SetGuiControlsEnabled(bool)`. |
| `FaceBoardPanel` | The shipped runtime host, with `asset`, `face`, `panelPosition`, `panelSize`, and public `ResetPose()`, `SavePose()`, and `LoadPose()`. |

`FaceBoardPose` is the expression preset manager, with no UI involved: keep poses as
`TextAsset` files, and apply one with `FaceBoardPose.FromJson(text).ApplyTo(state)` followed by
`state.Apply(face)`. A pose file lists handles by name and holds only the handles away from
neutral. `FromJson` throws `FormatException` on a file that is not a pose or that comes from a
newer format version, and `ApplyTo` resets the state first and returns the names it could not find
on the board.

If you set handles yourself instead, note that `Apply` accumulates into a GUI control rather than
assigning to it, because several handles can drive one control. So write a pose by setting every
handle, including the ones that go back to neutral, rather than only the ones that changed —
or call `Reset()` first, the way `ApplyTo` does.

`Apply` writes controls and stops there. In Play mode `MHFace` evaluates the rig in its own pose
stage; from an editor script, call `Evaluate()` yourself.

For mounting the board into a UI of your own, see
[For programmers](02-face-board.md#for-programmers) in the Face Board chapter.

## ARKit mapping and take data

The ARKit types live in the Runtime assembly, which means a parser or a live stream of your own
can produce our data types and reuse the rest of the path.

| Type | What it is |
|---|---|
| `MHArkitDefaultMapping` | The built-in table of 52 ARKit shapes onto raw controls: `TryGet`, `Key`, `IsIgnoredChannel`, `Count`, `Snapshot()`, `LiveLinkChannelNames`, `IgnoredChannelNames`, `ArkitShapeCount`, and `ControlPrefix`. |
| `ArkitMappingAsset` | An editable copy of that table, at `Assets/MHBridge/Settings/MHArkitMapping.asset`: `TryGet`, the static `Resolve`, `RebuildLookup`, `ResetToBuiltIn`, and `GetOrCreate`. |
| `ArkitTake` | The take itself: `columnNames`, `rows`, `times`, `fps`, `FrameCount`, `ColumnCount`, `Duration`, `IndexOfColumn`, and `Value(frame, column)`. |
| `ArkitCsvReader.Read` | A Live Link Face CSV into an `ArkitTake`. Editor assembly. |
| `ArkitClipBaker.Bake` and `ArkitClipBaker.BakeToAsset` | An `ArkitTake` into an `AnimationClip`, with a `Settings` struct and a `Report`. Editor assembly. |

`ArkitMappingAsset` is not only for ARKit. It is a shape-name to control-name table with
per-target scales, so it is also where you map the output of a different solver onto the
MetaHuman controls without writing that mapping into code.

To bake a folder of takes from a build script, read each one with `ArkitCsvReader.Read` and hand
it to `ArkitClipBaker.BakeToAsset`. The window adds nothing those two calls do not do.

## Editor hooks

These are the pieces to script an import against. All of them are in the Editor assembly, so
none of it ships in a build.

**`MHImportSettings`** holds the project-wide defaults: `trimRuntimeDna`, the nine per-part
material arrays, and `lodDefaults`. `Find()` returns the asset at its one fixed path or `null`;
`GetOrCreate()` and `SelectOrCreate()` create it and belong in a menu item, never in importer
code. `MaterialsForPart`, `PickForLod`, and `ShouldTrimRuntimeDna` are the lookups the importer
itself uses. Wrap the body of a `ScriptedImporter.OnImportAsset` of your own in
`using var scope = new MHImportSettings.ImportScope();`, which tells `Find()` to read the asset
database rather than write to it.

**`CharacterBuilder`** is the single place a DNA becomes a character. `Build` takes a head rig
and a body rig and welds them onto one skeleton; `BuildSingle` takes one rig. Both return a
`BuiltCharacter` carrying the root object, the meshes, the materials, the avatar, the DNA
sub-assets, the per-LOD renderer lists, the report, and any errors.

**`CharacterImportUtil`** is the other half of a `ScriptedImporter`. `Register` adds the built
objects to the import context, and `ReportErrors` forwards build errors.
`ApplyMaterialRemaps(AssetImporter importer, BuiltCharacter built)` replaces generated
materials with external material references on all renderers, including rigid eyes. It uses
remapped materials as they are and registers no import dependency on their textures or shaders. `SetPlaceholder` produces the
`<Name> (import failed)` object, and `PersistToScene` writes the result into a scene instead.

**`MHLodStripper`** removes LOD levels from a built character.
`StripInstanceToOneLod(GameObject, params string[])` works on a scene instance and returns a
`Result`; `StripImportToOneLod` sets the character's own importer and re-imports. Both leave
dead entries in the visibility arrays, so call `MHFace.PruneVisibilityRenderers()` and
`MHBody.PruneVisibilityRenderers()` afterwards.

**`MHBodyBoneNames.Canonical`** is the canonical spelling of every body DNA bone name. It is
what `MHSkeletonPostprocessor` normalizes Unreal's `Root` against, and it is public so that a
postprocessor of your own can use the same list.

**`MHBridgeMenu.Path` and `MHBridgeMenu.Priority`** hold every menu path and priority as a
constant. The tree owns priorities 100 through 500.

**Enums and part tables:** `MHDnaTrim`, `MHRigType`, `MHMaterialPart` with `MHMaterialParts`,
and `MHLodParts.All`, which is the reference for how many LOD levels each part of a MetaHuman
really carries.

## Extend Validate Setup with a check of your own

The validator takes its checks as objects, so adding one needs no change to the validator.

Implement `IMHValidationCheck`, which is a `Name` and an `IEnumerable<MHValidationIssue> Run()`.
The contract is look, do not touch: `Run` reports and changes nothing, and the repair lives on
the issue.

```csharp
using System.Collections.Generic;
using MHBridge.Editor.Validation;
using UnityEngine;

public sealed class ShadowQualityCheck : IMHValidationCheck
{
    public string Name => "Shadow quality";

    public IEnumerable<MHValidationIssue> Run()
    {
        if (QualitySettings.shadows != ShadowQuality.Disable) yield break;

        yield return new MHValidationIssue(
            id: "shadows-disabled",
            severity: MHIssueSeverity.Warning,
            title: "Shadows are disabled",
            detail: "Skin reads badly with no contact shadows.\nFix turns shadows back on.",
            contextHash: QualitySettings.names[QualitySettings.GetQualityLevel()],
            fix: TryFix,
            fixLabel: "Enable shadows");
    }

    // Returns true when the project changed; false with a reason the user can act on.
    private bool TryFix(out string error)
    {
        error = null;
        QualitySettings.shadows = ShadowQuality.All;
        return true;
    }
}
```

To run it, build a validator over your own set of checks and open the window on it:

```csharp
var validator = new MHValidator(new IMHValidationCheck[] { new ShadowQualityCheck() });
MHValidatorWindow.Open(validator);
```

`MHValidator.Default` is the shipped set of three, `Checks` lists whatever a validator holds,
`Run()` returns every issue, and `Pending()` returns the ones worth interrupting the user for:
not dismissed, and not merely informational.

`ContextHash` is what a dismissal is bound to, and it is why **Don't show again** covers the
situation rather than the check. Put into it everything about the current state that would make
a dismissal stale — an asset GUID, a list of quality level names. Change the state, and the
question comes back.

## Hair and cards tools

Everything below is public, in the `MHBridge.Editor.Hair` namespace for the tools and `MHBridge` for
the runtime components. These tools support scripted import and profile-based attachment.

The most useful thing to know is where the settings live: **in assets, not in C#**. A hair asset is
built by copying the profile's template, so resolve mode, resampling, units, and cluster LOD all
come from a file you can open and edit. Tuning the pipeline means editing a template or writing a
profile, not writing code.

- `MHHairProfile` is the asset that defines a kind of hair. Create one with
  **Create > MHBridge > Hair Profile** and fill in `keywords`, `priority`, `isFallback`,
  `baseHairAsset`, `hairInstancePrefab`, `groomMaterial`, `cardMaterial`, `skinAttachRoots`, and
  `attachBone`.
- `MHHairImportPipeline.Import(abcAssetPath, Options, report)` is the entry point for a batch
  script, and `Options` carries everything the window exposes, including `Options.profile`.
- `MHHairImportPipeline.ValidateAbc()` runs the four-marker scan on a file and reports whether it
  carries strand width and root UVs.
- `MHHairRig.AttachGroom(GameObject character, Transform bone, GameObject groomPrefab, string groomName, MHHairProfile profile, bool boneConstraint)` creates a groom under its profile node.
  Other scene-side operations include `RegisterWithLodHelper`,
  `PinTopology`, `EnsureHeadCollision`, `EnsureSkinTarget`, and `BakeSkinAttach`.
- `MHHairOutput` is the disk side both tools share: `EnsureSub()`, `EnsureFolder()`,
  `ToProjectRelative()`, `CharacterPrefabName()`, and `SaveCharacterPrefab()`, which saves the
  dressed character and reconnects the scene object to it. `MHHairOutputDrawer.Load()` and
  `Save()` read and write the per-project settings behind the **Output** section.
- `MHHairMaterialDrawer.Resolve()` applies the material rule — an override as it is, otherwise a
  tinted copy — and `MHHairColors` reads and writes the three gradient colors under every name the
  shaders spell them with.
- `MHHairPackages.Required` is the package list the setup panel is built from, with
  `IsInstalled()`, `AnyMissing()`, and `Install()` beside it.
- `MHHairLodHelper` exposes `Apply(int lod)`, `ApplyOffscreen()`, and `Recache()`. Call `Recache()`
  after you change the `grooms` list at run time, which is what a pool or a streaming system needs.
- `MHCardsBuilder.Build(Options)` builds a cards stack and returns an `MHCardsResult`, and
  `Options.profile` selects its profile node.
- `MHHairCardsLink.FindCardsFor(GameObject character, MHHairProfile profile)` returns the
  profile node containing its cards groups, or `null`. `LinkCards(GameObject character,
  GameObject presetNode, MHHairProfile profile)` links that entire cards node to the profile's
  grooms. `Link(GameObject character, GameObject groomInstance, GameObject cards)` links one
  groom, and `ApplyLod0State(GameObject character, GameObject cards)` applies initial visibility.
- The [scalp mask](06-hair.md#darken-the-scalp-under-the-hair-optional) can be built without the
  window. `MHHairScalpMask.RootUVs()` reads a hair asset's root UVs, and `DetectFlipV()` checks them
  against the head mesh. `MHScalpMaskCanvas.RenderLayer()` renders one groom with an
  `MHScalpMaskLayerSettings`, `MHHairScalpMask.CombineMax()` merges the layers, and
  `MHHairScalpMask.Save()` writes the PNG.

High quality lines have a scripted route too, in `MHBridge.Editor.Validation`, for a project that
sets itself up from a build script rather than from the window:

- `HighQualityLinesGroomCheck.TryEnable(out error)` does what the **Enable** button does: the shader
  option, the strand width, and the four presets.
- `MHGroomShaderGraphAccess.Read(path)` reports whether a groom Shader Graph has the option on and
  its **Width** block connected, and `TryEnable(path, out error)` sets both. It edits the
  `.shadergraph` file, so call it on an editable copy of the package, not on a read-only one.
- `MHHairPresetAccess.WithoutHighQualityLines()` lists the presets still on another renderer, and
  `TrySwitchToHighQualityLines(out error)` switches them.

## Type index

Everything public, by assembly.

**`MHBridge.Runtime`, namespace `MHBridge`**

| Type | One line |
|---|---|
| `MHFace` | The facial rig driver. [Drive the face from code](#drive-the-face-from-code) |
| `MHBody` | The body corrective layer. [Body correctives over the Animator](#body-correctives-over-the-animator) |
| `MHFaceClipDriver` | The component a facial clip binds to. [Play facial animation as an ordinary clip](#play-facial-animation-as-an-ordinary-clip) |
| `MHPoseLoop` | The static player-loop node. `Injected` reports whether it installed. [Where MHBridge runs in your frame](#where-mhbridge-runs-in-your-frame) |
| `MHPoseStageBehaviour` | The base class for a stage of the pose chain. [Write your own pose stage](#write-your-own-pose-stage) |
| `MHRig` | The disposable RigLogic wrapper. [Talk to RigLogic directly](#talk-to-riglogic-directly) |
| `MHBridgeNative` | The raw P/Invoke layer. Prefer `MHRig`. |
| `MHConvert` | DNA to Unity coordinate conversion, including `ApplyJointLocal`. |
| `MHLodVisibility` | The visibility gate helpers. [Follow LOD and visibility](#follow-lod-and-visibility) |
| `DnaBinaryAsset` | The DNA bytes as a sub-asset. `Bytes` is what `MHRig.LoadFromMemory` takes. |
| `MHArkitDefaultMapping`, `ArkitMappingAsset`, `ArkitTake` | [ARKit mapping and take data](#arkit-mapping-and-take-data) |
| `FaceBoardAsset`, `FaceBoardState`, `FaceBoardStates`, `FaceBoardPose`, `FaceBoardElement`, `FaceBoardPanel` | [The Face Board from code](#the-face-board-from-code) |
| `FileDialogs` | Open and save file dialogs in the editor and in a Windows build. [The Face Board from code](#the-face-board-from-code) |

**`MHBridge.Hair.Runtime`, namespace `MHBridge`.** Experimental, along with the rest of
[Hair](06-hair.md), and present only in a project that has `com.unity.demoteam.hair`.

| Type | One line |
|---|---|
| `MHHairLodHelper` | Drives every groom from the character's LOD band: `Apply(int)`, `ApplyOffscreen()`, `Recache()`, and the nested `Groom` and `OffscreenBehavior`. |
| `MHHairTopologyPin` | Keeps strand topology meshes alive so a muted groom does not rebuild them on return. |

**`MHBridge.Editor`**

| Type | Namespace | One line |
|---|---|---|
| `MHBridgeMenu` | `.Editor` | Every menu path and priority, as constants. |
| `MHImportSettings`, `MHLodConfig`, `MHLodParts`, `MHDnaTrim`, `MHRigType` | `.Editor.Import` | Project-wide import defaults, and the LOD tables. |
| `CharacterImportUtil` | `.Editor.Import` | The `ScriptedImporter` half of an import. |
| `CharacterBuilder`, `BuiltCharacter`, `MHLodStripper` | `.Editor.Build` | DNA to character, and LOD removal. |
| `MHMaterialPart`, `MHMaterialParts` | `.Editor.Materials` | The nine material slots and their texture maps. |
| `MHBodyBoneNames` | `.Editor.UnrealAnim` | Canonical body bone spellings. |
| `ArkitCsvReader`, `ArkitClipBaker` | `.Editor.Arkit` | CSV to take to clip. |
| `MHValidator`, `MHValidationIssue`, `IMHValidationCheck`, `MHIssueSeverity`, `MHIssueFix`, `MHValidatorWindow` | `.Editor.Validation` | [Extend Validate Setup with a check of your own](#extend-validate-setup-with-a-check-of-your-own) |
| `ClothingAttacher`, `AttachReport`, `RendererAttachResult`, `HiddenFaceCuller`, `CullResult`, `MaskBlender`, `BodyCullRecord`, `OutfitToolWindow` | `.Editor.Clothing` | The Outfit Tool. [For programmers](07-clothing.md#for-programmers) |
| `FaceBoardWindow` | `.Editor` | The editor board window: `Open(MHFace)`, `AssetPath`, `BoardAssetExists`. |
| `MHFaceEditor`, `MHBodyEditor`, `MHImporterEditor` | `.Editor` | The custom inspectors. Unity instantiates them. |

**`MHBridge.Hair.Editor`, namespace `MHBridge.Editor.Hair`.** Experimental: `MHHairProfile`,
`MHHairImportPipeline` with its `Options`, `MHHairRig`, `MHHairPackages`,
`MHPackageRequirement`, `MHHairLodMode`, `MHHairSingleLodRoute`, `MHCardsBuilder`,
`MHCardsResult`, `MHHairCardsLink`, and for the scalp mask `MHScalpMaskWindow`, `MHHairScalpMask`,
`MHScalpMaskCanvas`, and `MHScalpMaskLayerSettings`. See
[For programmers](06-hair.md#for-programmers).

---

## Troubleshooting

Project setup, import, animation, and attachment diagnostics are on the separate
[Troubleshooting](08-troubleshooting.md) page. Choose the matching topic there.
