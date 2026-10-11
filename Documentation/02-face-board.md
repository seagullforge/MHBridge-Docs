# Face Board

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

The Face Board is the MetaHuman facial control board, rebuilt as a panel inside Unity. Every handle
rides in its own groove and writes one semantic control of the character's DNA, and RigLogic
evaluates the result as you drag, so correctives, blend shapes, and wrinkle maps follow the handle.
No Animator, no clip, and no Play mode: open the board, bind a character, and the face moves in the
Scene view.

![The Face Board docked beside the Scene view, with the character holding a posed expression](images/02-face-board/01-board-and-face.webp)

The board is an input device. It poses a face and records no animation. You can save a single pose to
a JSON file, but there is no clip at the end of a session — for animation that plays back, see
[Animation](03-animation.md).
Baking animation from Face Board is being considered as a roadmap feature.

## On this page

- [Requirements](#requirements)
- [Open the board](#open-the-board)
- [Bind a character](#bind-a-character)
- [Pose the face](#pose-the-face)
- [Save and load a pose](#save-and-load-a-pose)
- [Live Drive](#live-drive)
- [What the board does not do](#what-the-board-does-not-do)
- [Run the board in Play mode and in a build](#run-the-board-in-play-mode-and-in-a-build)
- [Handles that do nothing](#handles-that-do-nothing)
- [Troubleshooting](#troubleshooting)
- [For programmers](#for-programmers)

## Requirements

The board drives a character that MHBridge imported, which is any object carrying an `MHFace`
component with its DNA assigned. Both the `.dcc` path and the head `.dna` path produce one, as
described in [Importing a MetaHuman](04-import-reference.md).

The layout itself is baked into `Assets/MHBridge/FaceBoard/Generated/FaceBoard.asset` and ships with
the package.

## Open the board

The board is a dockable editor window, and two entries open it:

- **Tools > MHBridge > Face Board** opens the board and binds it to the character selected in the
  Hierarchy, if there is one. The board looks for an `MHFace` on the selected object and on its
  parents, so any node of a character works.
- **Open Face Board**, at the bottom of the `MHFace` inspector, opens the board bound to that
  character.

![The Unity Tools menu open on the MHBridge submenu, with Face Board highlighted](images/02-face-board/02-menu.png)

![The MHFace inspector with the Open Face Board button at the bottom](images/02-face-board/03-inspector-button.png)

Six controls make up the window:

![The Face Board window with its six controls numbered: the Face field, Live Drive, Save Pose, Load Pose, Reset Pose, and Control face from GUI](images/02-face-board/04-window-anatomy.png)

| | Control | Where | What it does |
|---|---|---|---|
| 1 | **Face** | toolbar | The character the board drives. |
| 2 | **Live Drive** | toolbar | Whether handle moves reach that character. On by default. |
| 3 | **Save Pose** | top-right of the board | Saves the current pose to a JSON file. |
| 4 | **Load Pose** | top-right of the board, under **Save Pose** | Loads a pose from a JSON file and applies it. |
| 5 | **Reset Pose** | bottom-left of the board | Snaps every handle back to neutral. |
| 6 | **Control face from GUI** | bottom-right of the board | The switch that decides whether the rig reads the board at all. |

Every control except **Face** and **Live Drive** sits on the board rather than in the window, which
is why the runtime panel described in
[Run the board in Play mode and in a build](#run-the-board-in-play-mode-and-in-a-build) carries the
same four controls.

## Bind a character

Drag the character root from the Hierarchy into the **Face** field, or pick it through the field's
object picker. The board renders and drags with the field empty as well — the handles move, and they
drive nothing.

![The character root dragged from the Hierarchy into the Face field of the Face Board](images/02-face-board/05-bind-face.png)

One board drives one character. The window is a singleton, so a second **Open Face Board** click on
another character re-points the same window, and in the editor the character it held goes back to
neutral first. To pose two characters, work through them one at a time.

## Pose the face

1. Tick **Control face from GUI** in the bottom-right corner of the board.
2. Drag a handle.

> [!WARNING]
> **Control face from GUI** is one switch with two sides, and the same switch is the **Enable GUI
> Controls** field on `MHFace`. While it is off, which is the default, the rig ignores every value
> the board writes: the handles move, the face stays put, and every drag writes a warning to the
> Console. While it is on, the board owns every raw control it maps to, so a facial clip playing at
> the same time no longer reaches the face. Either you drive the face or a clip does, and this
> switch is where you say which.

![The Console warning that GUI controls are disabled, beside the unticked checkbox on the board](images/02-face-board/06-gui-switch-warning.png)

Two things about dragging are worth knowing before they surprise you:

- **Mirrored twins move together.** Dragging either one moves both, because they are two views of
  one value. The centre eye handle is different: it offsets the left and the right eye on top of
  wherever you left them, the way it parents both in the Maya rig.
- **The board fills the window.** There is no zoom and no pan, so the size of the window is the size
  of the board. The window does not go below 360 × 280 pixels.

## Save and load a pose

A pose goes to disk as a JSON file and comes back from one:

- To save the current pose, click **Save Pose** and choose a file.
- To restore a pose, click **Load Pose** and choose a file. The board resets to neutral first,
  then sets every handle the file lists, and with **Live Drive** on the face follows at once.

The file lists handles by name, so a pose saved on one character loads on any other. Only handles
away from neutral are written. If the file names a handle that the board does not have, the board
skips that handle and writes a warning to the Console.

The package ships ready-made poses in `Assets/MHBridge/Demo/Anims/FaceboardStates`: `Fury`,
`Happy`, `Laugh`, `Offended`, `Sad`, `Scream`, and `Surprising`. Load one to see what the board can
do, or as a starting point for a pose of your own.

![The Face Board with the Load Pose file dialog open on the FaceboardStates folder](images/02-face-board/10-save-load-pose.png)

## Live Drive

**Live Drive** decides whether the window writes into the bound character.

While it is on, every handle move writes the character's GUI controls and evaluates RigLogic on the
spot.

While it is off, the window stops writing. In the editor it also lets go of the character: the GUI
controls are reset, the face returns to neutral, and the native rig is unloaded. In Play mode
nothing is released, and the face holds the last pose the board gave it.

> [!CAUTION]
> Turning **Live Drive** off in the editor is not a freeze. It returns the face to neutral and drops
> the rig, so a pose you were building disappears from the character. The handles keep their
> positions, so turning Live Drive back on puts the pose back.

## What the board does not do

- **It writes no animation.** There is no record button and no path from a board pose into an
  `AnimationClip` in this version. Animation comes from an Unreal take or an ARKit take, both
  covered in [Animation](03-animation.md).
- **Dragging a handle is not undoable.** Ctrl+Z does not step back through handle moves or a
  **Load Pose**, and **Reset Pose** is the way back to neutral. To keep a pose you might want to
  return to, save it first. The **Control face from GUI** checkbox is the
  exception: flipping it outside Play mode is an edit to the scene, and it is recorded for Undo.
- **It writes no bones and no blend shapes of its own.** The board sets GUI controls, and `MHFace`
  does everything after that.

The pose lives in memory, attached to the character, for as long as the editor session lasts. It is
never written to the scene, to a prefab, or to an asset; **Save Pose** is the only way to keep it.
Closing the board and opening it again on
the same character brings the pose back; recompiling scripts, entering Play mode, leaving Play mode,
and restarting the editor each reset it to neutral.

## Run the board in Play mode and in a build

The included demo scene already has a panel. After assigning your character to **Character**
on `MH Demo`, press Play and use **Show or hide** under **Face Board**. The demo binds that
panel to your character. Enable **Control face from GUI** to pose, and disable it to return
to clips. See [demo setup](01-quick-start.md#open-the-demo-scene).

For a panel in your own scene, use the following procedure.

The same board runs as an on-screen overlay, which is how you hand a face to somebody who is not
using the editor: a director at a review, a QA pass, or a tool inside your own game.

1. Drag `Assets/MHBridge/FaceBoard/Generated/FaceBoardPanel.prefab` into the scene. The prefab
   already carries a `UIDocument` with its panel settings, and a `FaceBoardPanel` component with the
   layout assigned.
2. Optional: assign **Face** on the `FaceBoardPanel` component. Left empty, the panel takes the
   `MHFace` on a parent object, and failing that the first one it finds in the scene — which in a
   scene with several characters is whichever one it happens to find.
3. Press **Play**. The board appears in the bottom-left corner of the Game view.
4. Tick **Control face from GUI** on the board. It is the same switch as in the editor.
5. Drag handles. To move the overlay out of the way, edit **Panel Position** and **Panel Size**
   while the game runs.

![The Game view in Play mode with the face board as an overlay in the bottom](images/02-face-board/07-runtime-prefab.png)

**Panel Position** and **Panel Size** are fractions of the screen from 0 to 1, measured from the
bottom-left corner, and they default to `(0.01, 0.01)` and `(0.42, 0.6)`. Because they are
fractions, the overlay keeps its proportions at any resolution.

![The FaceBoardPanel inspector with edited Panel Position and Panel Size, and the overlay moved in the Game view](images/02-face-board/08-panel-layout.png)

> [!NOTE]
> A build needs nothing else. `FaceBoardPanel` is an ordinary runtime component in the
> `MHBridge.Runtime` assembly and the layout is an ordinary asset, so a Player build
> that references the prefab ships both.

**Save Pose** and **Load Pose** are available in the Editor on all desktop platforms
and in Windows Players. Other Players do not show these buttons. In a Windows Player
they open the standard Windows file dialog, and the first dialog of a session starts in the player's data folder
(`Application.persistentDataPath`). The dialog is modal: the game stops updating until you close
it.

Open the editor window on the character a runtime panel is already driving, and the two are views of
one pose: drag a handle in either and the other follows. That is deliberate. A pose belongs to the
character rather than to the view, so two views cannot write two poses into one set of controls.

![The Face Board window and the runtime overlay on one character in Play mode, showing the same handle positions](images/02-face-board/09-window-and-panel.png)

## Handles that do nothing

A few handles move and change nothing about the face. That is a property of the DNA rather than a
defect in the board: the control exists in the character, and the table that turns it into an output
is empty. On the sample character nine controls are in that state, among them `jaw_openExtreme` and
`mouth_stretchLipsClose`, and another character can carry a different set.

The Maya rig's switches — RigLogic on and off, look-at, convergence — are a separate matter. A DNA
has no control for any of them, so they are left out of the baked layout altogether, along with the
frames and the labels that surround them. There is nothing on the board to look for.

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| The handles move and the face does not | **Control face from GUI** is off. Tick it on the board, or tick **Enable GUI Controls** on `MHFace`. The Console carries a warning for every drag. |
| A facial clip stopped reaching the face | The same switch, the other way round. Turn **Control face from GUI** off and the clip drives the face again. |
| The face does not move in the editor | **Live Drive** is off, or the **Face** field is empty. If neither, read the Console: `[MHFace] dnaAsset is not set` and `[MHFace] failed to load DNA` both mean the rig never loaded. |
| One handle does nothing while the rest work | That control's output table is empty in this DNA. See [Handles that do nothing](#handles-that-do-nothing). |
| The face jumps back to neutral after a script recompile | Expected. The window frees the rig on every assembly reload, and before entering Play mode. |
| The window reads `FaceBoard.asset is missing. Reimport the MHBridge package.`, or **Open Face Board** is greyed out | The layout asset is missing from the project. Re-import the MHBridge package. |
| The runtime overlay drives the wrong character | The panel found an `MHFace` on its own. Assign **Face** on the `FaceBoardPanel` component. |
| The Console reads `[FaceBoard] Could not load the pose from …` | The file is not a Face Board pose, or it was saved by a newer version of MHBridge. The rest of the message names which. The current pose stays as it was. |
| The Console reads `[FaceBoard] The board has no handles named …` after **Load Pose** | The file names handles this board does not have. The pose loaded without them. |
| **Save Pose** and **Load Pose** are missing from the board | The build targets a platform other than Windows. File dialogs are available only in the editor and in a Windows build. |
| The Console reads `[FaceBoard] Skipped link …` | The layout and the link table disagree, which should not happen in a shipped package. Report it to us. |

## For programmers

Everything the board is built from is public, in the `MHBridge` namespace and the `MHBridge.Runtime`
assembly. For full signatures, see the [API reference](09-api-reference.md).

**To drive controls with no board at all**, which is the usual reason to be here, go through
`MHFace`:

- `GuiControlCount` and `GetGuiName(int)` enumerate the semantic controls the DNA carries.
- `IndexOfGuiControl(string)` resolves a name such as `"CTRL_C_jaw.ty"` to its index, or to `-1`.
- `GetGui(int)`, `SetGui(int, float)`, and `ResetGui()` read and write values by index.
- `SetControl(string, float)` writes by name, trying the GUI set first and the raw set second.
- `enableGuiControls` has to be `true`, or none of it reaches the rig.

In Play mode, write the values before the face evaluates, which means a pose stage ordered below
200 — see [Join the pose chain](03-animation.md#join-the-pose-chain). An editor script has no pose
loop, so drive the rig yourself: `EnsureInitialized()`, write, `Evaluate()`, then `ShutdownRig()`
when you are done with it.

**To put the board in a UI of your own**, mount a `FaceBoardElement` in any UI Toolkit panel:

```csharp
using MHBridge;
using UnityEngine;
using UnityEngine.UIElements;

[RequireComponent(typeof(UIDocument))]
public sealed class MyFaceBoard : MonoBehaviour
{
    [SerializeField] private FaceBoardAsset board;
    [SerializeField] private MHFace face;

    private void OnEnable()
    {
        // Always through FaceBoardStates, never new FaceBoardState(): every view of one character
        // shares a pose, and two states would overwrite each other's controls every frame.
        FaceBoardState state = FaceBoardStates.For(face, board);
        var element = new FaceBoardElement(board, state);

        element.onChanged            += () => state.Apply(face);
        element.onResetPose          += () => { state.Reset(); element.Refresh(); };
        element.onSavePose           += () => FaceBoardPose.SaveWithDialog(state);
        element.onLoadPose           += () =>
        {
            if (!FaceBoardPose.LoadWithDialog(state)) return;
            element.Refresh();
            state.Apply(face);
        };
        element.onGuiControlsChanged += on => face.enableGuiControls = on;

        GetComponent<UIDocument>().rootVisualElement.Add(element);
    }
}
```

The pieces that sample uses:

- `FaceBoardAsset` is the baked layout: `controls`, `decorations`, `links`, and the board extents.
- `FaceBoardStates.For(face, asset)` is the supported way to obtain a pose state. It keys states by
  character, so it hands back the state the editor window and the shipped panel already use.
- `FaceBoardState` holds the pose: `Count`, `HandlePos(int)`, `SetHandlePos(int, Vector2)`,
  `Reset()`, `Apply(MHFace)`, `PoseHash()`, and `Invalidate()` for when the rig reloads underneath
  it. `IndexOfControl(string)`, `GetTranslation(int)`, and `SetTranslation(int, Vector2)` read and
  write one handle by its local translation; unlike `SetHandlePos`, `SetTranslation` does not move
  the handle's mirrored twin.
- `FaceBoardElement` is the view. The constructor takes the asset and the state; `onChanged`,
  `onResetPose`, `onSavePose`, `onLoadPose`, and `onGuiControlsChanged` report what the user did;
  `Refresh()` repaints after you change the state yourself; and `SetGuiControlsEnabled(bool)` moves
  the checkbox without raising its event.
- `FaceBoardPose` is a recorded pose. `Capture(state)` and `ApplyTo(state)` convert between a pose and
  a state, `ToJson()` and `FromJson(string)` convert between a pose and its file text, and
  `SaveWithDialog(state)` and `LoadWithDialog(state)` are what the two buttons call.
- `FileDialogs` shows the open and save dialogs those two calls use: `OpenFile`, `SaveFile`, and
  `IsSupported`, which is `true` in the editor and in a Windows build.
- `FaceBoardPanel` is the shipped host, with `asset`, `face`, `panelPosition`, and `panelSize` as
  public fields, and public `ResetPose()`, `SavePose()`, and `LoadPose()` so a UI of your own can
  offer those buttons.

To apply a pose file from code, with no dialog and no board on screen, parse it and hand it to the
character's state:

```csharp
FaceBoardState state = FaceBoardStates.For(face, board);
FaceBoardPose.FromJson(poseAsset.text).ApplyTo(state);   // Throws FormatException on a bad file.
state.Apply(face);
```

`poseAsset` is a `TextAsset`, such as one of the files in
`Assets/MHBridge/Demo/Anims/FaceboardStates`.

`Apply` writes controls and nothing more. In Play mode `MHFace` evaluates the rig in its own pose
stage; in an editor script, call `Evaluate()` yourself.
