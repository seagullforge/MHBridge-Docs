# Import reference

<a id="importing-a-metahuman"></a>

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

Import your own DCC export or DNA, inspect the generated character, and extract assets before editing them.
For your first character, follow [Import a MetaHuman into Unity](01-quick-start.md).

## On this page

- [Two ways in: .dcc and .dna](#two-ways-in-dcc-and-dna)
- [Get a .dcc into the project](#get-a-dcc-into-the-project)
- [What an import produces](#what-an-import-produces)
- [The importer inspector](#the-importer-inspector)
- [Extract: preview versus production](#extract-preview-versus-production)
- [Re-import a character you already have](#re-import-a-character-you-already-have)
- [Re-import after a project change](#re-import-after-a-project-change)
- [The LOD stack](#the-lod-stack)
- [LOD evaluation](#lod-evaluation)
- [Shrink what you ship](#shrink-what-you-ship)
- [Project-wide defaults in Import Settings](#project-wide-defaults-in-import-settings)
- [Validate Setup](#validate-setup)
- [Requirements and limitations](#requirements-and-limitations)
- [For programmers](#for-programmers)

## Two ways in: .dcc and .dna

Both formats end up in the same builder. What differs is what you hand it.

A `.dcc` file is the main path: the DCC export folder from Unreal, zipped. It carries both
DNA files, the character's textures, the wrinkle masks, and a manifest.

A `.dna` file is the direct path: one raw DNA, with nothing around it. Drop it into the
project and it becomes a prefab. Whether it holds a head or a body is detected from the
skeleton — a body DNA carries the `pelvis`, `thigh_l`, and `thigh_r` joints and a head DNA
does not. The Model tab reports the result as `Detected head DNA from the skeleton.`

![The .dna importer inspector on the Model tab, showing the detected DNA kind](images/04-import-reference/01-dna-asset-detected.png)

| | `.dcc` | `.dna` |
|---|---|---|
| Contents | head DNA, body DNA, `Maps`, `Masks`, and a manifest | one DNA file |
| Skeleton | one shared skeleton: the body is the master, facial joints are grafted onto it | one per file |
| Textures and materials | the full map set from the export | materials with no textures |
| Wrinkle maps | yes, built from `Masks` | no |
| Face and body on one `Animator` | yes | no, two separate objects |
| Use it when | you have a DCC export | you only have a raw DNA file, or you are testing a rig |

Materials without textures on the `.dna` path are expected, not a defect: a raw DNA holds no
texture data at all.

> [!CAUTION]
> A `.dcc` needs both DNA files. If the manifest references only a head or only a body, the
> import fails with `.dcc manifest must reference both a head and a body DNA`, and you have
> to repeat the export in Unreal.

## Get a .dcc into the project

The DCC export in Unreal gives you either a folder or a `.zip` archive, and the
**Tools > MHBridge > Import** submenu has an entry for each:

- **DCC Folder** packs the folder that holds `ExportManifest.json` into a `.dcc` file.
- **DCC Zip** takes an archive the export already produced and copies it in as it stands.
  The file picker accepts `.zip` and `.dcc`.

Either way, you choose where inside `Assets` to save the result, and MHBridge imports it for
you.

![The Unity Tools menu open on the MHBridge Import submenu, showing DCC Folder and DCC Zip](images/01-quick-start/04-menu-import-dcc.png)

Both entries read `ExportManifest.json` before they write anything into the project. The
manifest has to name a head DNA and a body DNA, and both files have to be there. If they
aren't, MHBridge tells you what is missing and stops.

> [!NOTE]
> **Files import on their own.** Copy a `.dcc` or a `.dna` file anywhere into `Assets` and
> Unity runs the same importer on it — no menu item involved. That is how a `.dna` gets in at all.
>
> A `.dcc` file is the export archive under a different extension, nothing more. If you
> already have the export as a `.zip`, rename it to `.dcc`, drop it in, and it imports. The
> menu entries exist for what renaming cannot do: they check the manifest before anything
> lands in the project, they zip a folder for you, and they offer to update a character that
> is already there.

## What an import produces

Expand the imported asset in the Project window to see the result.

![Imported character sub-assets and four split eye maps](images/01-quick-start/09-asset-contents.png)


- The **main object** is the prefab root, named after the character name in the manifest.
- The **sub-assets** are the meshes for every LOD, the textures, the materials,
  `<Name>_Avatar`, and the runtime DNA as `dna_head` and `dna_body`.
- **On the root** sit an `Animator` holding the Avatar, plus the `MHBody`, `MHFace`, and `MHFaceClipDriver` components.
- **In the hierarchy** are the skeleton and the `<part>_LodGroup` nodes.

The `eyeLeft` and `eyeRight` parts use `MeshFilter` and `MeshRenderer`. A
`ParentConstraint` follows `FACIAL_L_Pupil` or `FACIAL_R_Pupil`: one constraint on the
part's LOD parent serves every eye LOD. With one LOD, the constraint is on the mesh itself.
`eyeshell`, `eyeEdge`, and `eyelashes` remain skinned. If a required pupil joint is missing,
the importer warns and builds that eye as a skinned mesh.

The eye shader controls pupil size. The rigid eye has no pupil blend shape, and the
constraint does not copy animated pupil-joint scale onto the whole eyeball.

![Rigid eye hierarchy and components](images/04-import-reference/02-rigid-eye-hierarchy.png)


A `.dcc` welds the head and body into a single skeleton: the body skeleton is the master, and
the facial joints are grafted onto it. That is what lets the face and the body run on one
`Animator`. A `.dna` import has no such thing to do, so a head DNA produces `MHFace` and
`MHFaceClipDriver` with no `Animator` at all, and a body DNA produces an `Animator` with an
Avatar and `MHBody`.

The import writes one line per character to the Console, reporting joint counts, grafted
facial joints, body correctives, the avatar it built, and the DNA trim:

Schematic excerpt (counts and the complete report depend on your character):

```text
[MHBridge] imported .dcc 'John': body=NNN joints, head grafted=NNN facial joints; face drives NNN facial joints; body correctives=NNN; avatar: generic avatar (standalone rig, root motion bone 'root'); valid=True
```

One more piece works outside the importer: `MHSkeletonPostprocessor` normalizes the casing of
bone names as FBX files are imported, because Unreal writes `Root` where the DNA skeleton has
`root`. Without it, clothing and animation coming out of Unreal fail to rebind.

## The importer inspector

Select a `.dcc` or a `.dna` file in the Project window and the Inspector shows a tab bar
modelled on Unity's own FBX importer: **Model**, **Rig**, and **Material**, with **Apply**
and **Revert** at the bottom. There is no Animation tab, because a DNA carries no animation.

### Model tab

![The .dcc importer inspector on the Model tab](images/04-import-reference/03-inspector-model-tab.png)

| Field | Default | What it does |
|---|---|---|
| **Scale Factor** | `1` | A multiplier applied on top of the DNA's own centimetre-to-metre conversion. `1` is native size. |
| **Import BlendShapes** | on | Imports the facial morph targets, of which a head DNA carries more than 700. Off means a faster import and a lighter asset; joints still animate, corrective blend shapes do not. |
| **Runtime DNA** | `Project Default` | What lands in the DNA sub-asset the runtime rig loads: `Project Default`, `Trimmed`, or `Full`. See [Trim Runtime DNA](#trim-runtime-dna). |
| **LOD (this asset)** | the project defaults | How many LOD levels to build per part, and the screen heights that open the LOD bands. See [The LOD stack](#the-lod-stack). |

Setting **Runtime DNA** to `Full` adds a note about build size, because that is the one
choice on this tab that makes the shipped asset larger.

![The Model tab with Runtime DNA set to Full and its build-size note visible](images/04-import-reference/04-runtime-dna-full-warning.png)

A `.dna` asset shows only the LOD parts its own DNA has: the head parts for a head DNA, the
body row for a body DNA.

### Rig tab

One field, **Avatar Type**, which decides how the body skeleton is interpreted.

![The Rig tab with the Avatar Type field set to Generic](images/04-import-reference/05-inspector-rig-tab.png)

- **Generic** is the default and the production path. The raw UE5 Mannequin hierarchy is
  kept and paired with a Generic Avatar that carries root motion on the `root` bone. A clip
  authored on this exact skeleton, such as a retarget export out of Unreal, plays back one
  to one, with no muscle abstraction in between.
- **Humanoid** maps the body onto Unity's Humanoid rig, so any Humanoid clip retargets onto
  it through Mecanim. Take this one for compatibility with clip libraries you do not
  control, such as Mixamo.

Which one suits which source of animation, and what Humanoid costs you in fidelity, is
covered in [Animation](03-animation.md).

### Material tab

For every part and every LOD, the importer takes the material
you authored in `MHImportSettings`, copies it, and injects the character's textures into the
copy. Everything else — colors, smoothness, the diffusion profile — stays exactly as you set
it on the template. Edit a template and the characters using it pick up the change on their
next import, unless a material remap supplies an extracted material instead.

![Eye split controls and boundary preview](images/04-import-reference/06-eye-split-boundary.png)


| Button | When it appears | What it does |
|---|---|---|
| **Open Import Settings** | always | Selects the `MHImportSettings` asset. |
| **Extract Textures & Materials** | on a `.dcc` that still carries its `Maps` folder | Writes the textures to `<Name>_Textures`, re-imports, then writes the materials to `<Name>_Materials`. |
| **Extract Materials** | on a `.dna`, or a `.dcc` whose maps are stripped | Writes the `.mat` files only. |
| **Strip Source Textures (N MB)** | on a `.dcc` that still carries its `Maps` folder | Rewrites the `.dcc` without those maps. See [Strip Source Textures](#strip-source-textures). |
| **Override Iris To Sclera Point** | on a `.dcc`; enabled only while both original combined eye maps exist | Opens the iris boundary preview. |
| **Clear Material Remaps** | once at least one material is remapped | Drops the link to the extracted `.mat` files and puts the asset back on generated ones. The files stay on disk. |

Above the remap button, the tab reads `Using N extracted material(s).` That line is how you
tell at a glance that an asset is running on extracted materials rather than generated ones.

> [!CAUTION]
> An empty material slot in `MHImportSettings` is an import error, not a silent fallback. The
> asset turns red in the Project window and the Console names the slot. The rest of the
> character still imports, so filling the slot and re-importing is enough to fix it.

### Adjust the iris boundary

The importer automatically splits `Eyes_Color` and `Eyes_Normal` into
`Eyes_Iris_Color`, `Eyes_Iris_Normal`, `Eyes_Sclera_Color`, and `Eyes_Sclera_Normal`.
These four maps become sub-assets, or PNG files after extraction. The combined maps
remain in the source `.dcc` until you strip them.

If the split does not follow the iris edge:

1. On the **Material** tab, click **Override Iris To Sclera Point**.
2. Adjust **Iris boundary** until the preview circle follows the iris edge.
3. Click **Apply** to rebuild the maps.

**Use Automatic** restores automatic detection and applies that result. Both apply actions
write the four PNG files to `<Name>_Textures`, update texture remaps and extracted eye
material assignments, and reimport the character. They can overwrite manual edits to those PNGs.

The boundary is the circle diameter as a fraction of the image diagonal. It is not a UV
radius or the material's **Iris Radius**. At `1`, the circle reaches the image corners.

## Extract: preview versus production

It's better to extract materials and textures before shipping your product.

Extraction gives you three things:

- **Processed files, not raw export data.** The written PNG files hold the same pixels the
  embedded preview uses: normals flipped to OpenGL convention, eyelash luminance packed into
  the alpha channel.
- **Correct import settings per map type.** Normal maps get **Texture Type** `Normal map`,
  the SRMF data maps get linear sampling, and the eyelash map gets **Alpha Is Transparency**.
  Compression stays at Unity's default so that you can tune it per platform.
- **Edits that survive a re-import.** Extraction records a remap on the importer, so the
  character keeps using your files rather than rebuilding embedded copies over them.

![Extracted materials](images/04-import-reference/07-extracted-materials.png)


![Extracted textures](images/04-import-reference/08-extracted-textures.png)

**Extract Textures & Materials** extracts textures, reimports, then extracts materials so
that they reference the PNG files. This is the standard action for a full `.dcc` file.

Sclera maps retain the source resolution. Iris maps use a power-of-two resolution with a
minimum of 256 pixels, so a smaller iris PNG is expected. All four split maps use **Clamp**.
MHBridge already converts normals from DirectX to OpenGL by flipping the green channel.
Do not flip MHBridge-extracted PNGs again. Check the convention separately for third-party maps.

## Re-import a character you already have

A character comes back from Unreal often: a fixed mesh, a new texture, another LOD. When the
project already holds a `.dcc` file named after the character in the manifest, both import
entries offer to update that file instead of saving a second copy.

**Update** rewrites the contents of the asset and nothing else, so everything you set on this
character survives:

- The importer settings: the LOD configuration, **Avatar Type**, **Scale Factor**, and
  **Runtime DNA**.
- The textures and materials you pulled out with **Extract**.
- Every reference to the character from a scene or a prefab. It stays the same asset.

Choose **Save As…** to keep the old character and add the new export beside it, which gives
you a second asset with default settings. If
several `.dcc` files carry the character's name, MHBridge does not guess: it opens the save
panel, and replacing a file there counts as an update.

> [!CAUTION]
> Extracted textures survive an update, and that means they hide the maps in the new export.
> The importer prefers an extracted texture over the copy inside the `.dcc` file, so the
> character keeps showing the previous export's skin until you run **Extract** again. When an
> update leaves extracted assets in place, the Console warns you and names how many.

## Re-import after a project change

Use this table to decide whether a character import is needed:

| Change | Action |
|---|---|
| Edit an extracted texture, material, or existing shader behavior | Save the edit; no full character reimport is needed. Unity updates the edited asset. |
| Edit a material template or material arrays in Import Settings | **Reimport** to regenerate materials. Material remaps still supply extracted `.mat` files as they are. |
| Add a shader texture property that needs importer assignment | **Reimport** to assign textures to generated materials. Assign textures manually in remapped materials, or clear material remaps first. |
| Change importer LOD settings or scale | Click **Apply**; it reimports automatically. |
| Change project-wide **Trim Runtime DNA** for assets on **Project Default** | **Reimport** each affected character. |
| Delete a remapped texture or material | Check the remap and restore or re-extract the asset, then **Reimport**. Deleted assets are not automatically restored. |

To return to generated materials, click **Clear Material Remaps** on the **Material** tab.
The extracted files stay on disk, but the character stops using them. The action automatically
reimports using the current templates. Extract again if you need editable copies. Clearing material remaps
does not clear texture remaps.

Extracted asset edits normally survive reimport. Edits to embedded copies do not.
The eye boundary apply action is an explicit exception: it rewrites the four eye PNGs and
updates extracted eye material texture assignments.

## The LOD stack

Each logical part of the character — head, teeth, eyeLeft, body, and the rest — goes under
its own `<part>_LodGroup` node carrying a native Unity `LODGroup` component.

A DNA does not carry the same number of levels for every part, and each slider stops at the
real depth of the part it belongs to:

| Part | LOD levels in the DNA |
|---|---|
| Head, Teeth, Eye Left, Eye Right | 8 |
| Eye Shell | 5 |
| Eyelashes, Eye Edge, Body | 4 |
| Saliva | 3 |
| Cartilage | 2 |

### One band for the whole character

Those depths do not match on purpose, and Unreal builds MetaHumans the same way. The head
carries eight levels and the body four, and Unreal's LODSync component holds the two in a fixed
ratio: the body changes level once for every two changes of the head. Head levels 0 and 1 both
belong against body level 0, head levels 2 and 3 against body level 1, and so on to head levels
6 and 7 against body level 3. The neck of each head level is built to meet the body level it is
paired with.

MHBridge keeps that pairing. A character has eight **LOD bands**, and every part decides its
band from the same measurement: the screen height of the whole character, never of its own mesh.
Every `LODGroup` an import creates therefore carries the same `Size` and the same
`Reference Point`. Within a band, the head shows the level with that band's number and the body
shows the level for half of it. The other parts follow the head, one level per band, until they
run out of levels and hold the last one.

> [!NOTE]
> Measuring each part by its own bounds is what a plain Unity import would do, and it does not
> work here. An eye is a 25 mm sphere and a body is nearly two metres tall, so at one camera
> distance the eye would sit at its coarsest level while the body was still at LOD 0. One
> measurement for the whole character is what keeps the parts on the same schedule.

> [!WARNING]
> The **Head** and **Body** sliders can break the pairing. A part cut short holds its last level
> for every deeper band, so from the band where it runs out the head faces body levels it was
> never built against, and the seam at the neck can open as a visible crack. Cut the two
> together, keeping two head levels per body level: 8 and 4, 6 and 3, 4 and 2, or 2 and 1.
> Cutting the head alone to three levels puts the mismatch at band 4; cutting it to two puts it
> at band 2, near enough for the crack to be visible. Only the head and the body share an edge,
> so the depths of the other parts are free.

### Band thresholds

![The per-part LOD sliders and LOD Bands in the Model tab](images/04-import-reference/09-lod-transitions.png)

The **LOD Bands (screen height)** foldout holds the thresholds, labelled `Band 0 → 1`,
`Band 1 → 2`, and so on. They are screen-relative heights in descending order, and they default
to `0.6`, `0.3`, `0.15`, `0.075`, `0.035`, `0.017`, `0.008`. Seven entries open the eight bands.

Because the body steps once per two bands, it reads only the odd entries: `Band 1 → 2` moves it
to its level 1, `Band 3 → 4` to level 2, and `Band 5 → 6` to level 3. The even entries move the
head and the other parts alone.

> [!NOTE]
> The last level of every part always gets a height of `0.0001` instead of the value from the
> array, so the part never culls away to nothing at distance. That is also why a threshold you
> type by hand is clamped to `0.0002`: anything lower would produce a last level that culls
> before the level above it.

A part limited to a single level gets no `LODGroup` and no `<part>_LodGroup` node at all —
its meshes sit directly on the character root.

To start over, use **Reset to Project Defaults** on a character, or **Reset to DNA Maximums**
on the settings asset. Both are disabled while the target has no values of its own, because
there is nothing to reset.

Three limits are fixed in the runtime, whatever your LOD configuration says:

| What | Limit | Why |
|---|---|---|
| Blend shapes | LOD 0 only | The DNA carries no morph targets past LOD 0. |
| Wrinkle (animated) maps | LOD 0 and LOD 1, narrowed further by **Max Animated Wrinkle Lod** on `MHFace` | RigLogic returns zero animated-map weights past LOD 1. |
| Body correctives | **Max Corrective Lod** on `MHBody`, default `2` | Past it the corrective joints are released to neutral and the Animator's pose alone is used. |

You can tune the thresholds after an import in the native `LODGroup` inspector on any
`<part>_LodGroup` node — it is an ordinary Unity component. A re-import overwrites them, so
keep values you want to last in the importer's LOD configuration.


![The native LODGroup component on the head_LodGroup node](images/04-import-reference/10-lodgroup-inspector.png)

## LOD evaluation

Here is what actually decides the level a component evaluates at:

1. `MHFace` and `MHBody` each hold one renderer per LOD, indexed by level, filled in by the
   importer.
2. Every frame, the active level is the **lowest** index among the renderers that are
   currently visible. The lowest rather than the first one found, because several cameras can
   each be showing their own level, and the closest camera is the one that has to be
   satisfied.
3. Which renderer is visible is the native `LODGroup`'s decision. `Renderer.isVisible`
   supplies the culling verdict.
4. The evaluation then assigns that number to the rig itself. If no renderer is visible, the
   level is `-1`, and the RigLogic evaluation is skipped entirely.

Each component reads its own part, so the two numbers differ. `MHFace` watches the head and
reports the character's band, 0 to 7. `MHBody` watches the body and reports the body's own
level, 0 to 3, which is half the band. **Max Corrective Lod** counts in body levels.

![A diagram of the LOD gate: LODGroup to renderer visibility to the active band to the rig](images/04-import-reference/11-lod-gate-diagram.png)

![The MHFace inspector in Play mode showing the Rig active banner and the current LOD](images/01-quick-start/08-play-rig-live.png)

Two details of that mechanism are worth knowing before they surprise you:

- **"Visible" includes the shadow pass**, and in the editor it includes the Scene view
  camera. A character standing behind the game camera but inside a light's shadow range still
  evaluates. That is correct, because its shadow moves.
- **`isVisible` is the previous frame's culling verdict.** On the frame a character comes back
  into view the gate can still read false, so that first frame draws the last pose that was
  evaluated. It lasts one frame and is normally invisible.

Outside Play mode the band is always 0, so authoring in the editor always sees full detail.

## Shrink what you ship

Two independent mechanisms cut the weight of an imported character. They address different
things: one shrinks the build, the other shrinks the project.

### Trim Runtime DNA

The geometry layer of a DNA is needed exactly once, at import, to bake the meshes. RigLogic
never reads it. So the DNA sub-asset that ships with your build is written without it.

Measured on one character:

| | Full | Trimmed |
|---|---|---|
| Head DNA | 48.9 MB | 4.19 MB |
| Body DNA | 4.45 MB | 83 KB |

Trimming is on by default, project-wide, and every asset can override it in its Model tab.
`Full` is worth choosing in one case: third-party code that queries mesh data through the
runtime DNA handle. Nothing else reads that layer.

The trim preserves every index — joints, controls, blend shape channels, and animated maps
all keep the numbers the importer baked against.

### Strip Source Textures

Before stripping, check the eyes under your scene lighting and finish iris boundary edits.
Normal reimports use extracted texture remaps. The combined eye sources remain necessary
if you want to change the split boundary.
The orange **Strip Source Textures** button rewrites the `.dcc` without them. On the sample
character that took the file from 132 MB to 43.2 MB, removing 17 maps worth 88.9 MB.

Only the `Maps` folder is removed:

- `Masks` stays. All ten TGA files come to 137 KB, and extraction does not produce them — the
  wrinkle texture array is rebuilt from the zip on every import. Deleting them breaks wrinkles
  with no way back short of re-packaging.
- `head.dna` and `body.dna` stay. Both are read on every import.

Two dialogs stand in the way, and both are there on purpose. The first appears only if some
map still exists nowhere but inside the `.dcc`, and offers to extract the textures first;
after that it checks again. If anything is still missing, nothing is removed. The second
always appears and confirms the deletion, naming the full path of the file being rewritten.

![The dialog offering to extract the textures before stripping](images/04-import-reference/12-strip-dialog-1.png)

![The confirmation dialog for stripping source textures, showing the file path](images/04-import-reference/13-strip-dialog-2.png)

The rewrite goes to a temporary file and replaces the `.dcc` only after the result verifies,
so any failure leaves the original untouched. On success the Console reads
`[MHBridge] stripped N source map(s) from '<path>', freeing <size>.`

> [!WARNING]
> A stripped `.dcc` is no longer self-contained, and the button edits a file on disk where
> Undo cannot reach it. Delete `<Name>_Textures` afterwards and the character re-imports
> without textures. Your source of truth is the original DCC export folder, not the stripped
> file.

On such an asset the extract button becomes **Extract Materials**, and the Material tab
carries a permanent note about the dependency.

![Stripped DCC with eye boundary controls disabled](images/04-import-reference/14-eye-stripped-source.png)


After strip, **Override Iris To Sclera Point** is disabled. Restore both original combined
eye maps in the `.dcc` from your original export to change the boundary. The split PNGs alone
do not supply the sources that this interface needs.

## Project-wide defaults in Import Settings

Select **Tools > MHBridge > Import Settings** to open the one settings asset the importer
reads. It lives at `Assets/MHBridge/Settings/MHImportSettings.asset`.

![The MHImportSettings asset in the Inspector, showing all three blocks](images/04-import-reference/15-import-settings-asset.png)

The inspector has three blocks.

**Default per-part LOD counts** is a template. A fresh import displays these values, and the
first edit you make in that asset's Model tab copies them into the importer. From then on the
asset owns its LOD state. While an asset still has none of its own, the inspector reads
`Showing project defaults (this asset has no overrides)`.

> [!NOTE]
> Editing the default LOD counts does not touch characters that are already imported. This is
> deliberate — a change here can never reach back into work you have already tuned — but it
> does mean the defaults are a starting point, not a project-wide switch.

**Trim Runtime DNA** is a live default, on by default. An asset left on `Project Default`
picks up a change here at its next re-import.

**Base materials per LOD** holds nine global arrays: head, body, teeth, eye, eyeShell,
eyeEdge, eyelashes, saliva, and cartilage. The index is the LOD, and the last entry repeats
for every deeper level, so a two-entry array covers an eight-level chain. These arrays are
global and required: every character rebuilds against them on re-import.

## Validate Setup

Some settings belong to the project rather than to a character, and MHBridge does not change
them behind your back. Select **Tools > MHBridge > Validate Setup** to open the **MHBridge
Setup** window and see which of them still need attention.

With **Check when the editor starts** enabled, the validator checks once per editor session.
It also checks after character imports, groom shader or preset imports, and package installation
or updates that make groom checks available. Automatic checks
open the window only for current, undismissed issues. They wait for asset updates and compilation
and do not open it in Play mode or batch mode.

Three checks ship for the project itself, listed worst first:

| Check | Severity | Fix |
|---|---|---|
| **HDRP is not the active render pipeline** | Error | None. Which pipeline a project renders with is your decision, so the check names the cause of magenta characters and stops there. Assign an HDRP asset in **Project Settings > Graphics**, and one per quality level in **Project Settings > Quality**. |
| **Skin Weights is below Unlimited** | Warning | **Set Unlimited**, applied to every quality level. |
| **Diffusion profile `'<name>'` is not registered** | Warning | **Register**, which adds the profile to the default volume profile. |

For empty material fields and profile registration, follow [Diffusion profiles](05-materials.md#diffusion-profiles).

Two further checks belong to the hair tools and become available when groom assets and
presets are unpacked on Unity 6 with HDRP 17, before a groom is imported onto a character. They offer to draw the strands as HDRP high quality lines, and name the HDRP settings
that have to be on for it. See [High quality lines](06-hair.md#high-quality-lines-unity-6).

> [!CAUTION]
> **Skin Weights is a per quality level setting.** MetaHuman meshes bind up to about twelve
> bone influences per vertex, and `QualitySettings` only ever exposes the active level. The
> fix therefore writes every level, because the level a build ships with is rarely the one you
> are looking at. The symptom of getting this wrong is a head and hands that deform around
> creases.

![Project Settings Quality with Skin Weights set to Unlimited](images/04-import-reference/16-skinweights-quality.png)

The footer carries **Re-check**, **Fix All**, a **Show dismissed** checkbox, and a **Check
when the editor starts** toggle. Each issue also has its own **Don't show again**.

![The MHBridge Setup window reading Everything checks out](images/04-import-reference/17-validate-clean.png)

A dismissal is bound to the state, not to the check. Change the profile or the set of quality
levels and the question comes back. Dismissals are stored per project and per user in
`ProjectSettings/EditorUserSettings.asset`.

## Requirements and limitations

| | |
|---|---|
| Unity | 2022.3 LTS minimum, reference version `2022.3.62f2`, tested through Unity 6.5 |
| Unreal Engine | 5.8 or later, for the DCC export |
| Render pipeline | HDRP only: 14.0.12 on Unity 2022.3, 17.x on Unity 6. |
| Editor platforms | Windows, macOS, Linux |
| Player platforms | PC (Windows, macOS, Linux), consoles* |
| Input formats | `.dcc`, a zipped DCC export, and `.dna` |
| Blend shapes | LOD 0 only |
| Wrinkle maps | LOD 0 and LOD 1 |

*Console builds require the adapter source kit and your platform SDK; they have not
been built or tested with console SDKs. See [Build for consoles](11-console-builds.md).

Two packages from Unity's demo team, `com.unity.demoteam.hair` and
`com.unity.demoteam.digital-human`, install from git URLs. They are needed for hair only. The
import path described on this page works without them, and the assemblies are split so that a
project that skips them still compiles. See [Hair](06-hair.md) for the details.

## For programmers

The import and LODs API. For
everything else, see the [API reference](09-api-reference.md).

- `MHFace.ActiveLod` and `MHBody.ActiveLod` are read-only and return `-1` while the character
  is off-screen. This is the right thing to subscribe to for your own per-LOD toggles; the
  hair assembly's `MHHairLodHelper` does exactly that.
- `MHFace.LODCount` and `MHBody.LODCount` report how many levels the DNA carries.
- `MHFace.IsInitialized` tells you whether the native rig is up.
- Do not write to `LOD`. The value lives until the next evaluation and no longer, for the
  reasons in [LOD evaluation](#lod-evaluation).
- `MHImportSettings` is a public `ScriptableObject` with `Find()`, `GetOrCreate()`,
  `MaterialsForPart()`, `ShouldTrimRuntimeDna()`, and `PickForLod()`. It is the supported
  entry point for scripting a batch import.
- `MHLodConfig` exposes `LodsForPart()`, `TransitionHeights`, and `Resolve()`, and
  `MHLodParts.All` is the reference for the real LOD depth of each part.
- `MHValidator` has a public constructor, so you can build a validator over your own set of
  `IMHValidationCheck` implementations and drive it with `Run()`, `Pending()`, and the
  per-issue `TryFix`.

MHBridge marks a type `internal` only where calling it from outside would break an invariant,
so most of the package is reachable when you need it.
## Tune materials

The extracted-material workflow, eye parameters, and profile assignments are in
[Materials and eyes](05-materials.md). Use that reference after materials extraction.
