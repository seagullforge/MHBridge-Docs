# Animation

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

Start with the included animation or bake the included ARKit CSV. Then use the same
Animator workflow for your own body clips and facial takes. RigLogic evaluates the
correctives, facial joints, and wrinkle weights while the clips play.

![A MetaHuman character moving in a Unity scene under HDRP lighting](images/03-animation/01-result.webp)

## On this page

- [Try the included animation](#try-the-included-animation)
- [Choose your tier: Generic or Humanoid](#choose-your-tier-generic-or-humanoid)
- [Part A — the Unreal path](#part-a--the-unreal-path)
- [Part B — the ARKit path](#part-b--the-arkit-path)
- [How animation reaches the rig](#how-animation-reaches-the-rig)
- [Troubleshooting](#troubleshooting)
- [For programmers](#for-programmers)

## Try the included animation

Set up your own character in the [demo scene](01-quick-start.md#open-the-demo-scene), then
assign its Animator Controller and Avatar as described in [Assign the demo animation](01-quick-start.md#assign-the-demo-animation).
The package includes these assets under `Assets/MHBridge/Demo/Anims`:

| Asset | What to try |
|---|---|
| `Demo_AnimController.controller` | Separate Body and Head layers with the demo's button transitions. |
| `Body/Demo_Body_Idle.FBX` | The `Demo_BodyIdle` body clip. |
| `Body/Demo_Body_Move.FBX` | The `Demo_BodyMove` exercise clip. |
| `Face/Demo_FacialTake.FBX` | The `Demo_FacialTake_Full`, `Demo_FacialTake_Smile`, `Demo_FacialTake_Scream`, and `Demo_FacialTake_Fear` clips. |
| `Face/Demo_ARKit.csv` | A raw recorded take to [bake and play](#part-b--the-arkit-path). |
| `FaceboardStates/*.json` | Still facial poses for the Face Board, not clips. See [Save and load a pose](02-face-board.md#save-and-load-a-pose). |

All three bundled FBX files ship with **Generic** animation import. For a Humanoid character,
select `Body/Demo_Body_Idle.FBX` and `Body/Demo_Body_Move.FBX`, set **Rig > Animation Type** to **Humanoid**, choose an avatar from your character or create a new one, and click
**Apply**.
On your character's **Animator**, assign the demo asset to **Controller**.

Press **Play**, then try **Play smile**, **Play scream**, **Play fear**, and **Play exercise**.

![Demo Animator with the controller and the character's own Avatar](images/03-animation/02-demo-animator-avatar.png)

## Choose your tier: Generic or Humanoid

The **Avatar Type** field on the **Rig** tab of the character's importer decides how the body
skeleton is interpreted, and therefore which clips play on it. This is the first decision,
because it decides which of the three routes below is open to you.

![The Rig tab of the .dcc importer with the Avatar Type field](images/04-import-reference/05-inspector-rig-tab.png)

| | **Generic** — production | **Humanoid** — compatibility |
|---|---|---|
| What it is | The raw UE5 Mannequin skeleton with a Generic Avatar. | A Mecanim rig: the bones map onto Unity's Humanoid rig. |
| Clip source | A clip authored on this same skeleton, which in practice means an export out of Unreal, retargeted or not. | Any Humanoid clip: Mixamo, the Asset Store, your own library. |
| Quality | One to one with Unreal, with no muscle abstraction in between. | Mecanim quality, the same as for any Humanoid character. |
| Root motion | From the `root` bone only, and it has to be retargeted. See [Retarget in Unreal](#retarget-in-unreal). | Unity synthesizes it from the centre of mass. |
| Take it for | Final quality, motion capture, cinematics. | A quick prototype, or a clip library you do not control. |

`Generic` is the default. Both avatars are built on a clone of the clean body skeleton, so
either one works as the source of a **Copy From Other Avatar** import, and both are saved in
the character asset as `<Name>_Avatar`.

> [!WARNING]
> Humanoid is a compatibility tier, not a downgrade you can patch. A retargeted Humanoid clip
> bends differently from the original because the muscle values are encoded relative to the
> source avatar's T-pose, which puts the offsets inside the clip rather than in the avatar
> that reads it. No setting on the Unity side undoes that. Humanoid gives you *any* clip at
> Mecanim quality; Generic gives you *your* clip at Unreal quality. Neither is a workaround
> for the other.

Three routes lead from a clip to a body that moves. They differ in how much work happens in
Unreal, and this section takes them in order of effort:

- **[The Humanoid route](#the-humanoid-route)** plays any Humanoid clip, from any source,
  with no work in Unreal at all. It also plays an exported Unreal MetaHuman Animation Sequence with a Humanoid Unity Avatar.
- **[The Generic route](#the-generic-route)** plays your own Unreal motion capture at Unreal
  fidelity. The work in Unreal is one export.
- **[Retargeting](#retarget-in-unreal)** is what you add when you need root motion on the
  `root` bone, or a clip that lives on a foreign skeleton.

## Part A — the Unreal path

Prepare your own body motion and facial take as separate FBX files, then combine their clips.
Choose the character rig type before setting the animation importer options.

### Body animation

#### The Humanoid route

The shortest way to a character that walks. Import the character with **Avatar Type**
`Humanoid`, and every Humanoid clip in your project retargets onto it through Mecanim, the
same as for any other Humanoid character. A clip from Mixamo or the Asset Store needs nothing
done to it.

Motion capture recorded in Unreal takes the same route: export the Anim Sequence as an FBX
file and import it as a Humanoid clip. **No retarget pass is needed**, because Mecanim is
already doing the retargeting at playback.

Root motion works on its own here. Unity synthesizes it from the character's centre of mass,
so a clip with no root motion authored into it still moves the `GameObject` once
**Apply Root Motion** is on.

If the feet slide, turn on **Foot IK** on the Animator Controller state that plays the clip.
It is Unity's own setting, it works for Humanoid only, and Timeline carries the same toggle
on the clip.

![An Animator Controller state inspector with the Foot IK checkbox](images/03-animation/03-unity-footik-state.png)

> [!NOTE]
> A `bone length mismatch` warning on a clip that came from *another* MetaHuman's skeleton is
> telling the truth: **Copy From Other Avatar** degrades quality there. Import such a clip
> with **Create From This Model**, or use the Generic tier.

If a Humanoid Avatar comes out invalid, MHBridge throws it away rather than ship a broken
one, and the character imports with no `Animator` at all. The Console warning names the cause
and what to do about it.

#### The Generic route

Take this one for motion capture recorded in Unreal on the MetaHuman skeleton, when you want
it back at Unreal fidelity. The clip is already on the right skeleton, so there is nothing to
retarget: export the Anim Sequence as an FBX file, and import it with the settings in
[Import the animation FBX into Unity](#import-the-animation-fbx-into-unity).

What you do not get this way is travel. Generic root motion comes from the `root` bone and
nowhere else, and a take recorded in place leaves that bone still, so the character animates
where it stands. If it has to cover ground, the motion has to be authored onto `root` first,
which is what the next section is for.

#### Retarget in Unreal

Retargeting is not a step every clip needs. Add it for one of two reasons:

- **You need real root motion** — the `root` bone itself travelling, rather than the hips and
  the legs moving underneath a stationary root.
- **The clip lives on a foreign skeleton** — Mixamo, another motion capture rig, or another
  Humanoid like skeleton — and you want it on a Generic character, where Mecanim is not there to translate
  it for you.

Here is a quick generic way to do it:

1. Right-click the Anim Sequence and select **Retarget Animations**.

   ![The Content Browser context menu on an Anim Sequence, with Retarget Animations selected](images/03-animation/04-ue-retarget-menu.png)

2. In the dialog, choose your MetaHuman as the target skeleton, then run the retarget.

   ![The Retarget Animations dialog with the MetaHuman chosen as the target skeleton](images/03-animation/05-ue-retarget-dialog.png)

   > [!WARNING]
   > It's better to retarget onto the skeleton of the character you are going to use, not onto another
   > MetaHuman. A clip retargeted onto a MetaHuman with different proportions carries the
   > source's flat bone position curves, which resize your skeleton to the source's
   > proportions and break the corrective layer. The symptom is an elbow or a knee that folds
   > like an accordion. MHBridge guards against it, as described in
   > [Bone translations from a retargeted clip](#bone-translations-from-a-retargeted-clip),
   > but the guard costs you something a correct retarget does not.

3. Right-click the retargeted Anim Sequence and export it to FBX.

   ![The Asset Actions context menu open on Export Selected Animations](images/03-animation/06-ue-export-selected-anims.png)

> [!CAUTION]
> Check that the result actually travels before you bring it into Unity. A source recorded in
> place, such as most of Mixamo, carries its locomotion on the pelvis and has no root bone at
> all. Most of the time, it does the work itself, but if something is wrong, my recommendation is to **Export Retarget Asset**
> in **Retarget Animation** and check the Op Stack contains **Pelvis Motion** and **Root Motion**
> with suitable settings.

Export the resulting Animation Sequence as FBX so the clip is now an FBX file waiting to be imported.

#### Import the animation FBX into Unity

Set the importer to read the clip against your character's own avatar:

| Tab | Setting | Value |
|---|---|---|
| Rig | **Animation Type** | The same type your character was imported with: `Generic` for a Generic character, `Humanoid` for a Humanoid one. A clip and the character that plays it have to agree. |
| Rig | **Avatar Definition** | `Copy From Other Avatar` |
| Rig | **Source** | `<Name>_Avatar`, the sub-asset inside your `.dcc` or `.dna` asset |

![The Rig tab of the animation FBX importer, set to Generic with Copy From Other Avatar](images/03-animation/07-unity-anim-fbx-rig-tab.png)

**Copy From Other Avatar** is right when the FBX file sits on the MetaHuman skeleton — a take
recorded on your character, or a clip retargeted onto it in Unreal. For anything else, such
as a Humanoid clip that never went through Unreal, leave **Avatar Definition** on
`Create From This Model` and let the clip keep the avatar built from its own rig.

Optionally, turn on **Apply Root Motion** on the character's `Animator` component. The
importer leaves it off, so until you turn it on a travelling clip animates correctly while
the `GameObject` stays where it stands. Leaving it off is the right choice when your own code
or a character controller moves the character.

![The Animator component on the character with Apply Root Motion turned on](images/03-animation/08-unity-animator-apply-root-motion.png)

> [!NOTE]
> Unreal writes `Root` where the DNA skeleton has `root`, and Unity treats those as two
> different bones. MHBridge normalizes the casing of MetaHuman bone names as an FBX file
> imports, and reports what it renamed in the Console. It only touches a hierarchy that
> carries at least ten bone names unique to the MetaHuman body skeleton, so your own rigs are
> left alone.


#### The corrective layer

The Avatar lets the `Animator` pose the roughly 52 core bones of the body. `MHBody` adds the
MetaHuman quality on top: every frame it reads the local rotation of 44 driver bones, feeds
them to the body RigLogic as pose deltas, and applies the resulting corrective and twist
output to the roughly 290 joints the Humanoid rig does not own. Those are the joints that
make a shoulder or a knee bend like a MetaHuman rather than like a mannequin.

![The MHBody inspector in Play mode, showing the Rig active banner and the three options](images/03-animation/09-unity-mhbody-inspector.png)

The inspector shows the live rig status and the counts the importer wired up, above three
options:

| Field | Default | What it does |
|---|---|---|
| **Apply Correctives** | on | The master switch for the corrective layer. |
| **Ignore Clip Bone Translations** | on | See [Bone translations from a retargeted clip](#bone-translations-from-a-retargeted-clip). |
| **Max Corrective Lod** | `2`, range 0 to 3 | Past this LOD the corrective joints are released to neutral and the Animator's pose alone is used. The body carries four LOD levels. |

#### Bone translations from a retargeted clip

**Ignore Clip Bone Translations** is on by default, which means most projects never meet the
problem it solves. It is worth knowing what it does before you turn it off.

**The symptom.** An elbow or a knee folds like an accordion, and the character looks like it
is wearing someone else's proportions.

**The cause.** A clip retargeted in Unreal onto a *different* MetaHuman carries the source
skeleton's flat bone position curves. Playing it resizes your skeleton to the source's
proportions, while the corrective layer keeps placing its helper joints at your character's
own bind positions. The two disagree, and the joint creases.

**What the option does.** Before the corrective pass, every bone is put back at its bind-pose
local position. The `root` and `pelvis` bones keep the clip's translation, so locomotion
still works.

**When to turn it off.** In two cases:

- The clip carries intentional bone translation, such as IK limb stretch or a cartoon squash.
- The clip was retargeted onto your own character's skeleton and carries no foreign
  proportions. Such a clip plays cleanly with no processing at all.

**What it costs.** Turning it off brings the creases back whenever the proportions do not
match.

![A side by side comparison of a creased elbow with the option off and a clean elbow with it on](images/03-animation/10-unity-elbow-accordion.png)

### Facial takes

A facial take carries no bone animation. The face is animated by 251 expression control
curves, which Unreal writes into the FBX file as **animated custom properties**, and which
land on the `MHFaceClipDriver` component in Unity. Head motion rides along in the same
form, as the `HeadYaw`, `HeadPitch`, `HeadRoll`, `HeadTranslationX`, `HeadTranslationY`,
and `HeadTranslationZ` channels.

#### Export the take from Unreal

Capture the performance through Live Link, record it into an Anim Sequence, then export that
Anim Sequence as an FBX file. The solve stays in Unreal: MHBridge reads solved control
curves, not raw capture data.

If MetaHumans in Unreal are new to you, start with the
[Ultimate Guide to MetaHuman Creation and Facial Animation](https://www.youtube.com/watch?v=iNfezN4WzZI)
by Bad Decisions Studio.

![The Unreal export dialog for a facial take, with the FBX option selected](images/03-animation/11-ue-export-face-take.png)

In the **FBX Export Options** dialog, turn on **Export Morph Targets**, under **SkeletalMesh**.
Despite its name, that checkbox gates every custom animation curve Unreal writes, not just the
morph targets. With it off, the file gets the bones and none of the control curves.

![The Unreal FBX Export Options dialog, showing the curve settings](images/03-animation/12-ue-fbx-export-options.png)

#### Set the import options in Unity

One import setting decides whether the take works at all.

1. Select the FBX file in the Project window.
2. Open the **Animation** tab.
3. Turn on **Import Animated Custom Properties**.
4. Click **Apply**.

![The model importer Animation tab with Import Animated Custom Properties turned on](images/03-animation/13-unity-fbx-animation-tab.png)

> [!WARNING]
> MHBridge does not turn **Import Animated Custom Properties** on for you. Without it, Unity
> never hands the control curves to the package, nothing is retargeted, and the clip plays a
> character that does not move. This is the first thing to check when a facial take does
> nothing.

#### Play the take

Drag the character into the scene, then give the clip to its `Animator` the way you would
give it any other clip: put the clip in a state of an Animator Controller, or drop it on a
Timeline track. The face plays. There is no bake step and no derived asset — the clip
embedded in the FBX file is the clip you use.

To see why it works, select the clip and open the **Animation** window. Every curve binds to
a field of `MHFaceClipDriver`, named after the raw control it drives, with the dot replaced
by an underscore:

![The Animation window listing the clip's curves, all bound to MHFaceClipDriver fields](images/03-animation/14-unity-clip-curves.png)

![An Animator Controller state holding the facial take, with the character playing in the Scene view](images/03-animation/15-unity-face-clip-in-animator.png)

The importer puts `MHFaceClipDriver` on the character root for you, on every `.dcc` import
and on a head `.dna` import.

#### Facial import details

With the setting on, three things happen by themselves. First, **Resample Curves** is turned
off, because resampling rebakes every curve onto a uniform 30 fps grid, which costs tens of
megabytes of YAML text and triggers the FBX importer's determinism warning. Second, every
animated custom property whose name matches a field on `MHFaceClipDriver` is retargeted onto
that component, bound at the character root next to the `Animator`. Third, everything else
the importer produced — the baked bone curves, the root transform, and the un-retargeted RBF
and pose-helper parameters — is dropped from the clip, because the face is driven from
control values through RigLogic and those curves are dead weight.

The Console reports all three:

```text
[MHBridge] 'Assets/Anims/Take.fbx': turned off 'Resample Curves' so the take keeps its sparse keys.
[MHBridge] 'Assets/Anims/Take.fbx': retargeted 257/2039 animated custom properties onto MHFaceClipDriver.
[MHBridge] 'Assets/Anims/Take.fbx': kept the 257 driver curves in 'Take', stripped 3530 unused.
```

The counts depend on the take. The retarget line is the one that confirms the import worked.

A file that matched nothing prints no retarget line at all. MHBridge treats it as somebody else's
FBX file and leaves it untouched, which is what you want for every other model in the project, so
silence here means the file holds no facial control curves. Three things cause that:

| Cause | Fix |
|---|---|
| The file came out of a body Anim Sequence. | Export the facial performance as a file of its own. |
| **Export Morph Targets** was off in the **FBX Export Options** dialog. | Turn it on and export again. |
| The curve names are not registered in the skeleton's curve metadata. | See [Export both performances for Body and Face from one take](#export-both-performances-for-body-and-face-from-one-take). |

To see what the file did carry, open the clip in the **Animation** window. A take that lost its
curves shows bone tracks and the `TCHour` to `TCRate` timecode channels, and nothing else.

#### Head motion options

The take's head channels are applied to the `head` bone as an absolute offset from that
bone's neutral, in degrees and centimetres. Two checkboxes on `MHFaceClipDriver` decide
whether they are used at all.

| Field | Default on a head and body character | Default on a head-only character |
|---|---|---|
| **Apply Head Rotation** | off | on |
| **Apply Head Translation** | off | off |

![The MHFaceClipDriver inspector showing the Apply Head Rotation and Apply Head Translation checkboxes](images/03-animation/16-unity-faceclipdriver-inspector.png)

The defaults follow from who owns the neck. On a full character the body clip animates the
neck and the head, so letting the take move the head as well fights it. On a head-only
character nothing else owns that bone, so the take drives it. Head translation is opt-in in
both cases.

While either checkbox is on, the driver reserves the `head` bone so that `MHFace` does not
reset it to neutral later in the frame. Head channels are angles and distances rather than
expression controls, so they are never clamped, unlike the 251 controls, which are held to
the range 0 to 1.

Which component owns the head and the neck is set out in
[Put the face and the body together](#put-the-face-and-the-body-together).

### Put the face and the body together

The face and the body share one `Animator`, because a `.dcc` import welds them onto one
skeleton. Animator Controller layers, avatar masks, and Timeline tracks therefore work the
way they always do, and a base layer for the body with a second layer for the face is the
usual arrangement.

![An Animator Controller with a body base layer and a face layer](images/03-animation/17-unity-face-body-layers.png)

Layers are a convenience, not a requirement. A facial clip binds to `MHFaceClipDriver` fields and
a body clip binds to bones, so the two never write the same property and one layer can hold both.
A clip that came out of a single take carries both sets already and needs no second layer at all.

What needs deciding is who owns the head and the neck:

| Setup | **Drive Head Neck** on `MHFace` | **Apply Head Rotation** on `MHFaceClipDriver` | Result |
|---|---|---|---|
| Head and body character, as imported | off | off | The body clip drives the neck and the head. |
| Head-only character, as imported | — | on | The facial take drives the head. |
| Body clip, plus head motion from the take | off | on | The take offsets the head on top of the body pose. |
| The rig owns the head | on | either | `MHFace` writes the head and the neck every frame, overwriting whatever else wrote them. |

![The MHFace inspector showing the Drive Head Neck checkbox](images/03-animation/18-unity-mhface-driveheadneck.png)

> [!WARNING]
> One switch decides whether the clip reaches the face at all. While **Enable GUI Controls**
> on `MHFace` is off, which is the default, the face plays what the clip drives. Turn it on
> and the Face Board owns every control it maps to, so the clip's expression stops arriving.
> `MHFaceClipDriver` never changes that switch — it belongs to you. See
> [Face Board](02-face-board.md).

#### Export both performances for Body and Face from one take

Two files, one per performance, is the arrangement the rest of this page assumes, and it needs
nothing from this section. Take this route when one Anim Sequence in Unreal already holds both
performances and you want one FBX file out of it.

Unreal writes a control curve into an FBX file only when the curve's name is registered in the
skeleton's curve metadata. A MetaHuman Face skeleton registers all of its control names, which is
why a facial take exports as it is. A body skeleton that received the face curves through a merge
registers none of them, so the export carries the bone tracks and drops all 254 curves without a
word. Register the names once, on the skeleton, and every later export carries them.

1. In the Content Browser, right-click the Anim Sequence and click **Copy Reference**.
2. Save this script as `register_curves.py`, and paste your reference into `ANIM`:

   ```python
   import unreal

   ANIM = '/Game/YOUR/PATH/AS_FaceAndBody'   # paste Copy Reference here

   anim = unreal.load_asset(ANIM)
   skel = anim.get_editor_property('skeleton')
   curves = unreal.AnimationLibrary.get_animation_curve_names(
       anim, unreal.RawCurveTrackTypes.RCT_FLOAT)

   for name in curves:
       unreal.AnimationLibrary.add_curve_meta_data(skel, name, False)
       unreal.AnimationLibrary.set_curve_meta_data_morph_target(skel, name, True)

   unreal.EditorAssetLibrary.save_loaded_asset(skel)
   print('registered', len(curves), 'curves on', skel.get_name())
   ```

3. Run it from **Tools > Execute Python Script**.
4. Export the Anim Sequence as an FBX file, with **Export Morph Targets** on.
5. Import the file into Unity with **Import Animated Custom Properties** on, the same as any
   facial take.

> [!WARNING]
> The script writes to the skeleton asset and saves it. On a MetaHuman skeleton that ships with a
> plugin, that edit lands in the plugin's content folder and every project using that skeleton
> inherits it. Run it on a copy of the skeleton, or keep the change out of source control, unless
> you want it permanently. To undo it, run the same loop with `set_curve_meta_data_morph_target`
> set to `False`.

The morph target flag makes Unreal's animation system look for morph targets under those names.
On a mesh that has none the lookup does nothing, so the flag is inert in Unreal and meaningful
only to the exporter.

Unity then reports a clip that kept both halves:

```text
[MHBridge] 'Assets/Anims/Both.fbx': kept 251 driver curves and 3079 body curves in 'Both', stripped 4 unused.
```

A combined clip keeps every bone curve, because those curves are the body performance, so it is as
large as any body clip. A face-only take is a fraction of that.

## Part B — the ARKit path

This workflow bakes recorded CSV data into an `AnimationClip`; it is not live streaming.
You can try it with the included CSV before recording anything on a phone.

### Bake the CSV into an animation clip

1. Select `Assets/MHBridge/Demo/Anims/Face/Demo_ARKit.csv` in the Project window.
2. Right-click it and select **MHBridge > Bake ARKit Take (CSV)…**. Alternatively, open **Tools > MHBridge > ARKit > Import Take (CSV)…** and assign the file to **CSV Take**.
3. Keep **Global Scale** at `1` and **Full Pose** on for this first standalone facial clip.
4. Set **Clip Path** to an output path such as `Assets/MyCharacter/Animations/Demo_ARKit.anim`.
5. Click **Bake Animation Clip**. The new `.anim` asset appears at that path.

Baking to an existing clip path overwrites that clip. The window previews the bake in memory;
its report describes the settings that will be saved. See [Bake settings](#bake-settings) to tune them.

![ARKit Take window with the included Demo_ARKit.csv and an output path](images/03-animation/19-demo-arkit-bake.png)

### Play an ARKit clip

1. Select your character in a scene with lighting.
2. Create a separate Animator Controller for the test and open it in the **Animator** window.
3. Drag the baked `.anim` into the controller as its default state.
4. It's better to add `Demo_Body_Idle` for the body on another layer to avoid zero-bones posing.
5. Assign this controller to your character's **Animator > Controller**. Keep your character's own **Avatar** assigned.
6. Press **Play**.

The clip writes `MHFaceClipDriver` fields and the rig evaluates the face. No ARKit runtime
player is needed. For a combined performance, use the layering workflow in
[Put the face and the body together](#put-the-face-and-the-body-together).

If you use the demo scene for this test, its buttons expect `Demo_AnimController` and can report
a controller mismatch. Use a separate test scene or restore the demo controller when returning
to its buttons. The standalone CSV test controller does not supply the demo's triggers.

![Baked ARKit clip in a default Animator state and facial playback](images/03-animation/20-demo-arkit-play.png)

### Record your own CSV

For a new recording, you need:

- An iPhone with a TrueDepth camera, which means an iPhone X or later.
- **Live Link Face**, Epic's free app, from the App Store.

1. In Live Link Face settings select ARKit.
2. Record the performance.
3. Export the take and copy the CSV file to your computer.

![The Live Link Face app on the recording screen, with the export option visible](images/03-animation/21-llf-record.png)

Then you can record and share a take. You will get a zip archive with something like "MyStale_N_iPhone.csv" inside of it.

You can put the CSV file inside `Assets`, where it becomes a `TextAsset`, or leave it
anywhere on disk. The bake tool reads either.

### Bake settings

| Field | Default | What it does |
|---|---|---|
| **CSV Take** | empty | The take as a `TextAsset` in the project. Leave it empty and use **Browse…** for a file outside `Assets`. |
| **Time Source** | `Timecode` | `Timecode` takes frame times from the take's own timecode column and falls back to the frame rate below when it is missing or not monotonic. `Constant Fps` spaces the frames evenly. |
| **Frame Rate** | `60` | The fallback rate, and the rate used to read the frames field of an `HH:MM:SS:FF` timecode. Live Link Face records at 60 FPS. |
| **Mapping Override** | empty | Empty means the project's mapping asset, or the built-in table when the project has none. See [The mapping table](#the-mapping-table). |
| **Global Scale** | `1` | Multiplies every ARKit weight. Keep it at `1` to preserve the recorded values. |
| **Full Pose** | on | Writes a curve for every driver field, flat zero for the ones this take does not touch, which makes the clip self-contained. Turn it off to layer this take over another one. |
| **Loop Time** | off | Sets the clip's **Loop Time** flag. |
| **Key Reduction** | `Keyframe Reduction` | Drops the keys that a straight line already describes within the error below. |
| **Control Error** | `0.002` | The deviation allowed on a control curve, in control units, where a control runs from 0 to 1. `0.002` is invisible. |
| **Clip Path** | the CSV path with an `.anim` extension | Where the clip is written. It has to be inside `Assets`. |

Every change re-runs the real bake in memory and shows its report, so the numbers on screen are the
numbers the saved clip will have. And baking over a clip that already exists rewrites it in
place.


### Read the bake report

The report is the same text before and after the bake. It reads like this:

```text
691 frames, 11.52 s @ 60 fps (from timecode)
Mapping: built-in table
Columns: 52 mapped, 8 ignored
Curves: 47 driven + 210 constant + 0 head = 257
Keys: 14260 (reduced from 32477, −56%)
Head rotation: columns present but identically zero — a Live Link Face CSV export carries no head motion (only the live stream does). Animate the head from a body clip.
```

Line by line:

- **Frames, duration, and rate**, with the source of the frame times in brackets.
- **Mapping**, the table this bake used.
- **Columns**, split into the ones that reached a control, the ones skipped on purpose, and
  the unrecognised ones. Eight columns are skipped by design: `Timecode`, `BlendShapeCount`,
  and the six per-eye rotation channels. Gaze already arrives through the `EyeLookIn`,
  `EyeLookOut`, `EyeLookUp`, and `EyeLookDown` shapes, and the rig has no separate control
  for an eye angle.
- **Curves**, split into the ones the take drives, the flat fillers that **Full Pose** adds,
  and the head channels.
- **Keys**, before and after reduction.

Two lines are marked with a warning sign and mean something is wrong:

- `Unrecognised columns` lists columns with no entry in the mapping table. On a standard
  Live Link Face export there should be none.
- `Mapped controls with no driver field` lists controls the table names that no field
  matches. That is a typo in a custom mapping table, or a control from a different rig.

### The mapping table

ARKit emits 52 named blendshape weights per frame. The MetaHuman rig is driven by raw
controls named `CTRL_expressions.*`. The names are close but not identical, one ARKit shape
often drives several controls, and several shapes sometimes share a control. Values are
therefore summed into a control rather than assigned to it, and clamped to the range 0 to 1
afterwards.

Two details are deliberate rather than accidental:

- **`MouthShrugLower` and `MouthShrugUpper` are scaled to `0.6`.** ARKit models a shrug as
  the lip rolling out over the teeth, which the MetaHuman rig splits differently. The scale
  makes it read as a hint rather than a grimace. This is the entry most worth retuning.
- **The gaze directions are inverted on purpose.** ARKit counts `In` and `Out` from the nose,
  while the MetaHuman rig uses absolute left and right, so a left eye looking `In` drives
  `eyeLookRightL`.

The built-in table, in the order a take's columns arrive. Every control carries the
`CTRL_expressions.` prefix, which the table omits:

| ARKit shape | Control | Scale |
|---|---|---|
| `EyeBlinkLeft` | `eyeBlinkL` | 1 |
| `EyeLookDownLeft` | `eyeLookDownL` | 1 |
| `EyeLookInLeft` | `eyeLookRightL` | 1 |
| `EyeLookOutLeft` | `eyeLookLeftL` | 1 |
| `EyeLookUpLeft` | `eyeLookUpL` | 1 |
| `EyeSquintLeft` | `eyeSquintInnerL` | 1 |
| `EyeWideLeft` | `eyeWidenL` | 1 |
| `EyeBlinkRight` | `eyeBlinkR` | 1 |
| `EyeLookDownRight` | `eyeLookDownR` | 1 |
| `EyeLookInRight` | `eyeLookLeftR` | 1 |
| `EyeLookOutRight` | `eyeLookRightR` | 1 |
| `EyeLookUpRight` | `eyeLookUpR` | 1 |
| `EyeSquintRight` | `eyeSquintInnerR` | 1 |
| `EyeWideRight` | `eyeWidenR` | 1 |
| `JawForward` | `jawFwd` | 1 |
| `JawLeft` | `jawLeft` | 1 |
| `JawRight` | `jawRight` | 1 |
| `JawOpen` | `jawOpen` | 1 |
| `MouthClose` | `mouthLipsTogetherUL`, `mouthLipsTogetherUR`, `mouthLipsTogetherDL`, `mouthLipsTogetherDR` | 1 |
| `MouthFunnel` | `mouthFunnelUL`, `mouthFunnelUR`, `mouthFunnelDL`, `mouthFunnelDR` | 1 |
| `MouthPucker` | `mouthLipsPurseUL`, `mouthLipsPurseUR`, `mouthLipsPurseDL`, `mouthLipsPurseDR` | 1 |
| `MouthLeft` | `mouthLeft` | 1 |
| `MouthRight` | `mouthRight` | 1 |
| `MouthSmileLeft` | `mouthCornerPullL` | 1 |
| `MouthSmileRight` | `mouthCornerPullR` | 1 |
| `MouthFrownLeft` | `mouthCornerDepressL` | 1 |
| `MouthFrownRight` | `mouthCornerDepressR` | 1 |
| `MouthDimpleLeft` | `mouthDimpleL` | 1 |
| `MouthDimpleRight` | `mouthDimpleR` | 1 |
| `MouthStretchLeft` | `mouthStretchL` | 1 |
| `MouthStretchRight` | `mouthStretchR` | 1 |
| `MouthRollLower` | `mouthLowerLipRollInL`, `mouthLowerLipRollInR` | 1 |
| `MouthRollUpper` | `mouthUpperLipRollInL`, `mouthUpperLipRollInR` | 1 |
| `MouthShrugLower` | `mouthLowerLipDepressL`, `mouthLowerLipDepressR` | 0.6 |
| `MouthShrugUpper` | `mouthUpperLipRaiseL`, `mouthUpperLipRaiseR` | 0.6 |
| `MouthPressLeft` | `mouthPressUL`, `mouthPressDL` | 1 |
| `MouthPressRight` | `mouthPressUR`, `mouthPressDR` | 1 |
| `MouthLowerDownLeft` | `mouthLowerLipDepressL` | 1 |
| `MouthLowerDownRight` | `mouthLowerLipDepressR` | 1 |
| `MouthUpperUpLeft` | `mouthUpperLipRaiseL` | 1 |
| `MouthUpperUpRight` | `mouthUpperLipRaiseR` | 1 |
| `BrowDownLeft` | `browDownL` | 1 |
| `BrowDownRight` | `browDownR` | 1 |
| `BrowInnerUp` | `browRaiseInL`, `browRaiseInR` | 1 |
| `BrowOuterUpLeft` | `browRaiseOuterL` | 1 |
| `BrowOuterUpRight` | `browRaiseOuterR` | 1 |
| `CheekPuff` | `mouthCheekBlowL`, `mouthCheekBlowR` | 1 |
| `CheekSquintLeft` | `eyeCheekRaiseL` | 1 |
| `CheekSquintRight` | `eyeCheekRaiseR` | 1 |
| `NoseSneerLeft` | `noseWrinkleL` | 1 |
| `NoseSneerRight` | `noseWrinkleR` | 1 |
| `TongueOut` | `tongueOut` | 1 |

To change any of it, select **Tools > MHBridge > ARKit > Mapping Table**. That creates, or
selects, one asset at `Assets/MHBridge/Settings/MHArkitMapping.asset`. The bake tool finds it
there on its own, so there is nothing to drag anywhere.

![The MHArkitMapping asset in the Inspector, with the shape rows and the toolbar buttons](images/03-animation/22-unity-arkit-mapping-asset.png)

Two toolbar buttons matter:

- **Reset to Built-in** replaces the whole table with a fresh copy of the built-in one.
- **Validate** checks every control name in the table against the driver's fields. Run it
  after an edit: a single typo produces a clip that animates nothing and says nothing.

> [!CAUTION]
> A mapping asset is all or nothing, not a patch on top of the built-in table.

## How animation reaches the rig

Four things write to the character's skeleton every frame, in a fixed order:

![A diagram of the pose chain: the Animator, then MHBody at 100, MHFaceClipDriver at 150, and MHFace at 200](images/03-animation/23-pose-chain-diagram.png)

The `Animator` component poses the skeleton from the clips. `MHBody` reads that pose and
adds the body correctives on top of it. `MHFaceClipDriver` reads the facial curves the clip
wrote into its own fields and pushes them into the face rig as raw controls. `MHFace`
evaluates the face and writes the facial joints, blend shapes, and wrinkle weights. A later
stage owns what an earlier stage produced, which is the answer to every "who overwrites
whom" question on this page.

The whole chain runs in the animation phase of the frame, ahead of constraints
and `LateUpdate`. Attachments, `ParentConstraint`, hair, and bone colliders therefore see
the final pose in the same frame.

The numbers on the diagram are stage orders inside that phase. MHBridge installs a node of
its own in Unity's player loop, immediately before constraints are solved, and the stages
registered with it run in ascending order. They are not script execution order, which
orders `LateUpdate` and therefore everything that happens once this chain has already
finished. That distinction matters if you write a component of your own — see
[For programmers](#for-programmers).

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| The facial FBX imported, but the face does not move | **Import Animated Custom Properties** is off on the model importer. Turn it on and apply. Or **Enable GUI Controls** is on for `MHFace`. Turn it off and the clip drives the face again. |
| The Console has no `retargeted N/M …` line | The same **Import Animated Custom Properties** setting, or an FBX file that carries no `CTRL_*` attributes. Check the export in Unreal. |
| The clip weighs tens of megabytes, or the importer reports an inconsistent result | **Resample Curves** is on. MHBridge turns it off for takes that import animated custom properties, so check the Console for that line. Or it's just YAML size if you export it as standalone `.anim` file. It will be smaller in runtime |
| A Generic character animates on the spot and never travels | Either no **Root Motion** operation was added to the retarget in Unreal, or **Apply Root Motion** is off on the `Animator`. The importer leaves it off. |
| An elbow or a knee folds like an accordion | The clip was retargeted onto another MetaHuman's skeleton. Keep **Ignore Clip Bone Translations** or retarget onto your own character to fix the cause. |
| The character imported with no `Animator` | The Humanoid Avatar came out invalid and was dropped. Read the Console warning, then re-import with the Generic rig, or check that the body DNA carries the full UE5 Mannequin bone set. |
| A `bone length mismatch` warning | The clip comes from another MetaHuman's skeleton, where **Copy From Other Avatar** degrades quality. Use **Create From This Model** or the Generic tier — or leave it, because **Ignore Clip Bone Translations** already covers the case it warns about. |
| An ARKit clip plays, but the head does not move | A Live Link Face CSV export carries no head rotation. Animate the head from a body clip. |
| The face ignores the clip and holds a pose from the Face Board | **Enable GUI Controls** is on for `MHFace`. Turn it off and the clip drives the face again. |
| The face ignores the Face Board | The same switch, the other way round. The board reaches the face only while **Enable GUI Controls** is on. |
| The feet slide on a Humanoid character | Turn on **Foot IK** on the Animator Controller state. |

## For programmers

Use the [API reference](09-api-reference.md) for `MHFaceClipDriver`, raw controls,
ARKit mapping, and scripted baking. Artist workflows on this page use ordinary Unity clips.

### Join the pose chain

To supply controls from code, use a pose stage between the clip driver and face evaluation.
See [Write your own pose stage](09-api-reference.md#write-your-own-pose-stage) for a complete
example and [Where MHBridge runs in your frame](09-api-reference.md#where-mhbridge-runs-in-your-frame)
for ordering. Writing from `LateUpdate` is too late for the normal pose phase.
