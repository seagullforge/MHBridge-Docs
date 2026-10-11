# Troubleshooting

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

Every entry below is a symptom, its cause, and the fix. Before working through them, two steps
could resolve some of the errors instantly:

1. Select **Tools > MHBridge > Validate Setup** and fix what it lists.
2. Read the Console. Every message we write carries a prefix — `[MHBridge]`, `[MHFace]`,
   `[MHBody]`, `[MHFaceClipDriver]`, `[FaceBoard]`, `[MHHairTopologyPin]` — so a prefix is a
   search term.

MHBridge requires HDRP. Editor platforms are Windows, macOS, and Linux; Player targets
are PC (Windows, macOS, Linux) and consoles*.

*Console builds require the adapter source kit and your platform SDK; they have not
been built or tested with console SDKs. See [Build for consoles](11-console-builds.md).

## On this page

- [Demo scene](#demo-scene)
- [Project setup](#project-setup)
- [Import](#import)
- [The rig does not move](#the-rig-does-not-move)
- [Timing and ordering](#timing-and-ordering)
- [Face animation](#face-animation)
- [Body animation](#body-animation)
- [Hair and clothing](#hair-and-clothing)
- [Still stuck](#still-stuck)

## Demo scene

The **Errors** panel explains missing character, component, controller, trigger, and camera
assignments. Fix the named issue outside Play mode, then start Play again so the demo
checks the setup and caches its focus points again.

| Message or symptom | Action |
|---|---|
| `YOUR METAHUMAN` is still the empty placeholder | Import your own character, place it at `(0, 0, 0)`, and assign it to **MH Demo Controller > Character**. |
| No `MHFace`, `MHBody`, or `Animator` | Use the character produced by the DCC importer. A head-only or body-only DNA cannot supply both demo performances. |
| No controller, a different controller, or a missing trigger/layer | Assign `Demo_AnimController` to the character's **Animator > Controller** and to **MH Demo Controller > Animator Controller**. The demo uses `PlaySmile`, `PlayScream`, `PlayFear`, and `MoveBody` triggers. |
| Character stands away from the origin | Set the character root's world position to `(0, 0, 0)` before Play. |
| A view frames the origin instead of the body or head | Check the **Body Bone** and **Head Bone** names. The default names are `spine_05` and `head`. |
| Facial buttons play but the board pose stays | Turn **Control face from GUI** off on the Face Board. |

For the full setup, see [Open the demo scene](01-quick-start.md#open-the-demo-scene).

## Project setup

Start with **Tools > MHBridge > Validate Setup**. Groom-specific checks are covered in Hair troubleshooting.

| Symptom | Cause | Fix |
|---|---|---|
| The character, and everything else, renders magenta | HDRP is not the active render pipeline, and the package ships HDRP materials only | Assign an HDRP asset in **Project Settings > Graphics**, and one per level in **Project Settings > Quality**. This check offers no fix button, because which pipeline a project renders with is your decision. |
| The head and hands deform around creases, and skin slides over the joints | `QualitySettings.skinWeights` is below `Unlimited`, and MetaHuman meshes bind up to about twelve influences per vertex | Click **Set Unlimited**, which writes every quality level. Setting only the active level leaves the level a build ships with broken. |
| The skin looks like plastic, subsurface scattering seems absent, and there is no error anywhere | The diffusion profile is not registered with HDRP, which silently substitutes the neutral one | Click **Register**. Registering the profile on a Volume in a scene works too, which is why this is a dismissible warning rather than an error. |
| The skin renders in a wrong flat colour | The **Diffusion Profile** slot on an MHBridge skin material is empty | Assign `MH_Skin.asset` from `Assets/MHBridge/Shaders` in the material's inspector. |
| The validator names an empty iris, sclera, or teeth diffusion field | The named material slot needs its own profile | Assign `MH_IrisDiffusionProfile`, `MH_ScleraDiffusionProfile`, or `MH_Teeth` as appropriate, then **Re-check** and **Register**. See [Validate Setup](04-import-reference.md#validate-setup). |
| `Diffusion profile asset is missing` | The profile was deleted, or the package arrived incomplete | Assign a profile by hand, as above. |
| `HDRP's diffusion profile list could not be read` | MHBridge cannot reach the default volume profile of this HDRP version | Check that **Project Settings > Graphics > HDRP > Volume** holds a default profile. If it does, report the HDRP version to us. |
| The window never opens by itself | Automatic checks run at session startup, after character or groom-asset imports, and after relevant package changes; the window opens only for current issues. Startup checks also require `MHImportSettings` | Open **Tools > MHBridge > Validate Setup**. See the [automatic check policy](04-import-reference.md#validate-setup) for triggers, dismissals, and editor state limits. |
| A dismissed warning came back | A dismissal is bound to the state, not to the check. The profile changed, or the set of quality levels did | Expected. Dismiss it again, or fix the cause. Dismissals live in `ProjectSettings/EditorUserSettings.asset`, per project and per user. |

## Import

For iris boundaries, inverted eye normals, and pupil animation, see [Visual problems](05-materials.md#visual-problems).

| Symptom | Cause | Fix |
|---|---|---|
| `DllNotFoundException` or `EntryPointNotFoundException` on import or on Play | The native adapter is missing, cannot load for the selected platform/architecture, or does not match the managed bindings | Check the matching library under `Assets/MHBridge/Plugins`: `x86_64/MHBridgeRig.dll` (Windows), `Linux/x86_64/libMHBridgeRig.so` (Linux), or `macOS/libMHBridgeRig.dylib` (macOS). Check its Plugin Importer settings and use native and managed files from the same package release. For consoles, follow the build guide linked at the start of this page. |
| After a package update the character does not re-import, and the plugin behaves like the old one | Unity keeps a native plugin loaded for as long as the editor runs | Restart the editor, then re-import the DLL. |
| The face moves, but the correctives land on the wrong shapes: crooked lips, wrong eyelids | The Unity-frame to DNA-channel table is broken | Re-import the character, which bakes the table. |
| No textures, and a warning per map in the Console | The `.dcc` was stripped of its source maps and the `<Name>_Textures` folder was then deleted. A stripped `.dcc` is not self-contained | Restore the texture folder, or re-import from the original DCC export. |
| A `DnaBinaryAsset` dragged out to a file is twice the size of the `.dna` | The project is in Force Text, so a `byte[]` serializes as hexadecimal. A build is unaffected | Nothing to do. |
| Code of yours asks the DNA for mesh data and gets zeros | The shipped DNA is trimmed of its geometry layer | Set **Runtime DNA** to `Full` on the Model tab. |
| Clothing or animation from Unreal will not rebind to the skeleton | Unreal writes `Root` where the DNA has `root`, and Unity treats those as two bones | The postprocessor normalizes it during the FBX import. On a path of your own, check names against `MHBodyBoneNames.Canonical`. |
| The asset turned red, and holds one object named `<Name> (import failed)` | The import threw | Select the asset. When the Inspector reads `Last import failed with N error`, click **Print to Console** and read the message. |
| `.dcc manifest must reference both a head and a body DNA` | The export carries only one of the two | Repeat the DCC export in Unreal with both. |

## The rig does not move

| Symptom | Cause | Fix |
|---|---|---|
| `[MHFace] dnaAsset is not set.` or `[MHBody] dnaAsset not set.`, and the component disabled itself | The component was added by hand to an object the importer never wired | Use the generated prefab. The inspector carries the same message in a help box. |
| `[MHFace] failed to load DNA: …` | Empty or corrupt DNA bytes, or the native layer did not come up | The text after the colon comes from the native plugin. Re-import the asset, and check the plugin. |
| `[MHFace] Bone count mismatch: DNA joints=… bones.Length=…` | The skeleton and the DNA disagree, usually after the hierarchy was edited by hand | Re-import. Until then the smaller of the two counts is driven. |
| The character froze on leaving the camera and did not recover | Expected: off-screen, RigLogic is not evaluated at all. If it never recovers, the culling bounds are wrong | Check `updateWhenOffscreen` and the rest bounds on the renderers. `ActiveLod` reading `-1` is the indicator. |
| `SetGui` does nothing | **Enable GUI Controls** is off on `MHFace`, which is the default for a freshly imported character | Turn it on, or write raw controls with `SetRawControl` instead. |
| Values written into an `MHRig` disappear | `MHFace` holds a managed mirror of the raw controls and pushes all of it every frame | Write through `MHFace.SetRawControl`. See [The raw control mirror](09-api-reference.md#the-raw-control-mirror). |
| The head is fought over between your code and the rig | Two owners for one bone | Call `MHFace.ReserveJoint("head")` to take it and `ReleaseJoint("head")` to give it back, or turn on `driveHeadNeck` to hand it to the rig. |

## Timing and ordering

| Symptom | Cause | Fix |
|---|---|---|
| Hair, constraints, or bone colliders lag the head by one frame | The player-loop node was not installed, so every stage runs from the `LateUpdate` fallback | The Console carries the `Could not add the pose phase` warning. Check `MHPoseLoop.Injected`. The cause is a custom player loop, or a package that rebuilds it. |
| A script of yours reads a stale pose | It reads too early: from `Update`, or from an earlier player-loop phase | Read from `LateUpdate`, by which point the pose is final. |
| Controls written by your script apply one frame late | You write from `LateUpdate`, which runs after the pose phase | Write from `Update`, or from a pose stage ordered below 200. |
| An `MHPoseStageBehaviour` of yours never runs | `OnEnable` or `OnDisable` was overridden without calling `base`, or the subclass declares its own `LateUpdate` | Call `base`, and move the work into `RunPoseStage`. |
| One stage threw, and in a build the whole chain stopped | The try-catch around stages exists in the editor and in a development build only | Do not throw from `RunPoseStage`. |

## Face animation

For facial FBX import, Face Board ownership, and ARKit head motion, see [Animation troubleshooting](03-animation.md#troubleshooting).

| Symptom | Cause | Fix |
|---|---|---|
| Wrinkles flicker, or vanish for one frame on a plateau | A cubic tangent overshoots 1.0, and no row of the animated-map table fires above it | Our clip driver clamps already. A driver of your own has to clamp to the range 0 to 1 itself. |
| There are no wrinkles at all | `applyWrinkleMaps` is off — the import turns it on only when the head got a wrinkle-capable skin material; or the material carries no `_WMask*` properties; or the LOD is past the cap | Turn the option on, and put the head on an MHBridge skin material. Wrinkles stop at LOD 1 whatever `maxAnimatedWrinkleLod` says. |
| Blend shapes stop applying at a distance | Morph targets exist on LOD 0 only, in the DNA and at runtime | Not a defect. |

## Body animation

For root motion, retargeted proportions, and Avatar import, see [Animation troubleshooting](03-animation.md#troubleshooting).

| Symptom | Cause | Fix |
|---|---|---|
| Deliberate bone translation disappeared, such as IK limb stretch | **Ignore Clip Bone Translations** (`MHBody.ignoreClipBoneTranslations`) pins bone positions except `root` and `pelvis` to their bind values | Turn **Ignore Clip Bone Translations** off, and read what it costs in [Bone translations from a retargeted clip](03-animation.md#bone-translations-from-a-retargeted-clip). |
| A single joint is inverted | Quaternion keys with a negative `w` break the pose deltas | Already canonicalised in `MHBody`. If you still see it, report it. |
| `[MHBody] no pose-driver bones resolved — correctives will do nothing.` | The skeleton does not match the body DNA: renamed or missing bones | Re-import, and check the hierarchy. |
| The character collapses, or the belt drops to the origin, after **Copy From Other Avatar** | `HumanDescription.m_GlobalScale` came out as zero on a script-built avatar | Fixed in the builder. Re-import the character. |
| There are no correctives at a distance | `maxCorrectiveLod`, default `2` | Expected past the cap: the Animator's pose alone is the saving the cap exists for. Raise it if you need correctives further out. |

## Hair and clothing

| Symptom | Cause | Fix |
|---|---|---|
| HDRP High Quality Lines is flickering | It's common HDRP pipeline bug and normally it's exist only in Editor. So you could be safe in build, but it's better to check it. | Turn **High Quality Line Rendering** in **Asynchronous Comput Shaders** to on. But beware of performance, it could increase or decrease it depends on system.

| Topic | Main troubleshooting reference |
|---|---|
| Groom packages, renderer setup, skin attachment, cards links, and LOD swaps | [Hair troubleshooting](06-hair.md#troubleshooting) |
| Clothing binding, body masks, geometry hiding, and outfit LODs | [Clothing troubleshooting](07-clothing.md#troubleshooting) |
| Face Board binding and control ownership | [Face Board troubleshooting](02-face-board.md#troubleshooting) |
| Constraints or custom pose stages run late | [Timing and ordering](#timing-and-ordering) |

## Still stuck

Send us this, and the report is workable on the first read:

- The Unity version and the HDRP version.
- The MHBridge version.
- The full Console message, prefix included.
- The full Player/Editor log file.
- What **Validate Setup** lists.
- Whether it reproduces on a freshly imported character.

Include the input format (`.dcc`, `.dna`, FBX, or CSV), the steps that reproduce the problem,
and the Hair or Cards Tool report when relevant. Submit the report to [MHBridge Issues](https://github.com/seagullforge/MHBridge-Docs/issues).
