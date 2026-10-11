# Hair (experimental)

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

Add strands, cards, or both to an [imported character](01-quick-start.md). MHBridge builds
hair assets, attaches them to your character, and links the groom and cards LODs.
You supply the groom or cards files; the package includes no hairstyles or characters.

![A MetaHuman character with strands, eyebrows, and a beard](images/06-hair/01-showcase.webp)

> [!CAUTION]
> Strand hair is experimental and depends on Unity's demo team hair package. Its workflow
> and API can change independently of the rest of MHBridge. Measure simulation and renderer
> cost on your target hardware before planning a crowd of strand characters.

## On this page

- [Choose a hair workflow](#choose-a-hair-workflow)
- [Requirements](#requirements)
- [Step 1 — Export the grooms from Unreal](#step-1--export-the-grooms-from-unreal)
- [Step 2 — Bring the ABC files into Unity](#step-2--bring-the-abc-files-into-unity)
- [Step 3 — Add the grooms to the Hair Tool](#step-3--add-the-grooms-to-the-hair-tool)
- [Step 4 — Set the target, the output, and the collision](#step-4--set-the-target-the-output-and-the-collision)
- [Step 5 — Pick a LOD mode](#step-5--pick-a-lod-mode)
- [Step 6 — Import and read the report](#step-6--import-and-read-the-report)
- [What you get, and what it costs](#what-you-get-and-what-it-costs)
- [LOD modes and swaps](#lod-modes-and-swaps)
- [Step 7 — Hair cards (optional)](#step-7--hair-cards-optional)
- [Tune the look](#tune-the-look)
- [What the import added to your character](#what-the-import-added-to-your-character)
- [Limits](#limits)
- [Troubleshooting](#troubleshooting)
- [For programmers](#for-programmers)

## Choose a hair workflow

| Result | Requirements | Tradeoff | Start here |
|---|---|---|---|
| Cards only | Exported cards FBX files and an imported character. No hair, Alembic, or digital-human packages. | No physics, lightweight. | [Build hair cards](#step-7--hair-cards-optional) |
| Strands only | Groom packages, unpacked presets, and ABC files with strand width and root UV data. | Physics, Skinned, heavy(!), relies on the demo package. | [Requirements](#requirements), then groom setup and import. |
| Strands near the camera, cards at distance | Both sets of files, with matching hair profiles in both tools. | Hides strands at a distance (no rendering), but keeps their mesh in RAM. | Complete groom setup, choose **Strands Plus Cards**, and [build cards](#step-7--hair-cards-optional). Either tool can run first. |

A **profile** defines a hair type and its materials and attachment. A **preset** is the
HairAsset or HairInstance template a profile uses. A **groom** is a strand instance; a
**cards group** is one exported GroupN stack. A **LOD band** is the character-wide level
that controls the strand-to-cards swap. All cards groups under one profile switch together.

## Requirements

| What | Where it comes from | What needs it |
|---|---|---|
| `com.unity.demoteam.hair` | MHBridge fork, git URL | Strand grooms. Without it the Hair Tool draws a setup page instead of itself. |
| `com.unity.formats.alembic` 2.2.2 or later | Unity registry | Reading the `.abc` grooms. |
| `com.unity.demoteam.digital-human` | MHBridge fork, git URL | Skin attachment: eyebrows, beard, and mustache riding the face. |
| `HairStrands` and `AlembicImporter` | Built-in Unreal plugins | Grooms in Unreal. |
| SimpleGroomExporter | Third-party Unreal plugin, free, installed separately | Exporting an `.abc` that carries strand width and root UVs. Any file that already carries them works too. |

The digital-human package is optional, and what it costs to skip is one specific thing: **skin
attachment**. A groom that asked for it falls back to a bone, the report says so, and eyebrows,
beard, and mustache stop following the character's expression. For a character that wears scalp hair
and nothing else, skipping the package changes nothing.

### Install the Unity packages

MHBridge does not install packages for you. The Hair Tool lists what it needs and where to get it,
and you add each package in Package Manager.

Open **Tools > MHBridge > Hair > Hair Tool**. In a project that has neither the hair nor the Alembic
package, the window is the setup page: one row per package, with what it is for, its identifier, and
a **Copy** button. The Alembic row also has an **Open** button.

![The MH Hair Tool window showing the package setup page, with a row per package and the Package Manager button](images/06-hair/02-packages-setup-panel.png)

To add the packages, follow these steps:

1. On the **Demo Team Hair System** row, click **Copy**.
2. Click **Package Manager**.
3. In Package Manager, select **+ > Add package from git URL**.
4. Paste the URL, and click **Add**.
5. Optional: To add skin attachment, repeat steps 1–4 for the **Demo Team Digital Human** row.
6. In the Hair Tool, on the **Alembic** row, click **Open**. Package Manager opens on the Alembic
   package.
7. Click **Install**.
8. Wait for the recompile to finish, and then reopen the Hair Tool. The setup page is replaced by the
   tool itself.

Skip digital-human and the window still opens, with a banner at the top naming what is missing and a
**Packages** foldout holding the same rows — so you can add it later without going to look for the
URL again.

Two of the three rows point at MHBridge forks rather than at Unity's own repositories:

```text
https://github.com/PinkPanter/com.unity.demoteam.hair
https://github.com/PinkPanter/com.unity.demoteam.digital-human
```

The tools use MHBridge forks for compatibility with the supported Unity and HDRP versions (6.3+ Unity and 17+ HDRP).
Both keep Unity's [Unity Companion License](https://unity.com/legal/licenses/unity-companion-license).

### Install a package without git

A git URL is resolved by Package Manager running a `git` client of its own, so on a machine that has
none the install fails with an error instead of a download. Either install
[Git for your operating system](https://git-scm.com/downloads), restart Unity, and add the package from its git URL
as described in [Install the Unity packages](#install-the-unity-packages), or take the package as a
folder:

1. Open the fork on GitHub and download the repository as a ZIP archive.
2. Extract it, and rename the extracted folder to the package name — `com.unity.demoteam.hair`.
3. Move that folder into your project's `Packages` folder, beside `manifest.json`.

   ![The project's Packages folder in Explorer, with the extracted package folder beside manifest.json](images/06-hair/03-packages-folder.png)

4. Return to Unity. An embedded package is picked up on the next focus, with no git involved.

A package installed this way is a copy rather than a link, so a later fix in the fork means
repeating the download.

### Unpack the groom assets

A **hair profile** is what tells the tool how to treat one kind of hair: which HairAsset template to
copy, which HairInstance prefab to clone, which material to tint, which file-name keywords select
it, and whether that kind of hair rides a bone or the deforming head mesh. Four of them ship with
the package — Hair, Eyebrows, Beard, and Mustache — in `Assets/MHBridge/Hair/Presets`.

The profiles are on disk from the first import, because the Cards Tool reads them for the bone and
the card material. What they point at is not. Each of those assets is typed by, or built on, a
package MHBridge does not ship, and a project without it cannot import those files —
so they travel in `Assets/MHBridge/Hair/GroomAdditional.unitypackage` instead:

| What waits in the sub-package | Why |
|---|---|
| `Presets/<Type>AssetPreset.asset` | A HairAsset, a type that comes from `com.unity.demoteam.hair`. |
| `Presets/<Type>InstancePreset.prefab` | A HairInstance prefab, from that same package. |
| `Materials/Shaders/MHGroom.shadergraph` | Reaches into the package for its `HairVertex` subgraph node. |
| `Materials/Hair_Groom.mat`, `Eyebrows_Groom.mat`, `Beard_Groom.mat`, `Mustache_Groom.mat` | The material that shader belongs to, and magenta without it. |

Until you unpack it, the four profiles carry empty **Base Hair Asset**, **Hair Instance Prefab**, and
**Groom Material** fields, and the Hair Tool opens on a prompt rather than on the tool:

![The Hair Tool prompt reading No usable hair profiles were found, with the Unpack button](images/06-hair/04-presets-prompt.png)

- Click **Unpack GroomAdditional.unitypackage**. The templates, the shader, and the material land in
  the folders above, and the four profiles are replaced by copies that point at them.

Three more buttons sit under the prompt. **Re-check** reads the folder again. **Continue anyway**
dismisses the prompt for this session, and **Continue and don't show again** dismisses it —
both are for a project that carries hair profiles of its own. In a project where the templates are
on disk but the profile assets are gone, the prompt offers **Create a default Hair profile**
instead, which restores one profile, the scalp-hair fallback, and not the full set.

### Prepare the strand renderer before import

On Unity 6 with HDRP 17, prepare high quality lines after installing the dependencies and
unpacking the groom assets, before importing the first groom:

1. Open **Tools > MHBridge > Validate Setup**. It can also open automatically when the groom assets or packages arrive and issues are found.
2. Click **Enable** on the groom high quality lines issue.
3. Click **Re-check** and resolve any remaining line-rendering issues.

The shipped presets use **Builtin Tubes**. **Enable** switches the presets and configures
the shader and HDRP settings. For the exact
requirements and existing groom instances, see [High quality lines](#high-quality-lines-unity-6).

On Unity 2022.3 with HDRP 14, keep **Builtin Tubes** or another built-in renderer. Skip the
high quality lines setup and continue to export and import the grooms.

![Setup after unpacking groom assets](images/06-hair/05-validator-after-unpack.png)


### What MHBridge needs from an .abc file

The Hair Tool reads Alembic, and it does not care which application wrote the file. Four things have
to be in it:

| What | Why |
|---|---|
| An **Ogawa** archive | The only Alembic container the tool reads. Anything else is reported and skipped. |
| **Curves**, the `AbcGeom_Curve` schema | The strands themselves. A file with no curve schema holds no hair, and the tool refuses it. |
| Per-vertex **`width`** | Strand diameter and tapering resolve from it. Without it the hair package falls back to a uniform diameter, and three settings — `strandDiameterValue`, `tipScaleValue`, and `tipScaleOffsetValue` — have to be tuned by hand on every import. |
| Root **`uv`** | The root UVs, which carry per-strand color and growth variation. Without them the tool resolves roots against the head mesh instead. |

If you already have a groom `.abc` file from somewhere else, go straight to
[Step 2](#step-2--bring-the-abc-files-into-unity).

### Install the groom exporter for Unreal

Unreal's own groom export writes strand positions and nothing more, so the grooms come out of Unreal
through **SimpleGroomExporter** instead — a free plugin by Jacob Kostenick, in the fork MHBridge
maintains:

```text
https://github.com/PinkPanter/UE5_SimpleGroomExporter
```

The plugin is a separate tool rather than a part of MHBridge, and it is not part of your purchase.
MHBridge neither redistributes nor sells it, and the terms the code is available under are recorded
in the `LICENSE` file at the root of the fork.

1. Open the fork's **Releases** page and download the archive that matches your engine version, such
   as `SimpleGroomExporter_5.8.zip`.
2. Extract it into the `Plugins` folder of your Unreal project. Create the folder if the project has
   none.

   ![Windows Explorer showing the SimpleGroomExporter folder inside an Unreal project's Plugins folder](images/06-hair/06-ue-plugin-folder.png)

3. Restart the Unreal editor.

4. Select **Edit > Plugins** and confirm that **SimpleGroomExporter** is enabled. It requires
   **HairStrands** and **AlembicImporter**, and turns both on with itself.

   ![The Unreal Plugins browser with SimpleGroomExporter enabled](images/06-hair/07-ue-plugins-enabled.png)

## Step 1 — Export the grooms from Unreal

Repeat this for every groom the character wears: the scalp hair, the eyebrows, the beard, and the
mustache.

1. In the Content Browser, find the groom asset. It is the assembled groom on your own character,
   not the source groom you downloaded.

2. Right-click it and select **Asset Actions > Export**.

   ![The Unreal Content Browser with a groom asset selected and the Asset Actions submenu open on Export](images/06-hair/08-ue-export-menu.png)

3. Choose a destination inside your Unity project, set the format to `.abc`, and confirm.

4. Set the export options:

   | Option | Set it to |
   |---|---|
   | **Use Render Data** | Off for a normal groom. On, it exports the strands the groom actually renders, with guide interpolation and grooming applied, rather than the raw imported strands. Turn it on when an export fails or comes out wrong. |
   | **Axis Preset** | `For Unity`, which writes `Alembic(X, Y, Z) = UE(X, Z, Y)`. `Default` writes raw Unreal coordinates, and `Custom` assigns each output axis on its own, with negation. |
   | **Merge Groups** | On. (recommended). Improves performance and avoids depth issues between groom groups. The groom also works with this option off. |
   | **Export Widths** | On. Writes the strand diameter, as the groom renders it, in the Alembic `width` param. |
   | **Export Root UV** | On. Writes each strand's root UV in the Alembic `uv` param. |


   The options persist between sessions, so this is a first-run step rather than one you repeat per
   groom.

   ![Export Options with Merge Groups enabled and the recommended ABC export settings](images/06-hair/09-export-options-merge-groups.png)

5. Click **Export**.

Name the files with the kind of hair in them. The Hair Tool matches a groom to a profile by the
`.abc` file name, looking for `hair`, `brow`, `beard`, `mustache`, `moustache`, `goatee`, or
`soulpatch`, and a name that matches nothing falls back to the scalp-hair profile. The list is the
`keywords` field on each profile in `Assets/MHBridge/Hair/Presets`, so you can add your own words or
write a profile of your own.

![Four exported .abc files in Windows Explorer, their names carrying the hair-type keywords](images/06-hair/10-ue-abc-files.png)

> [!CAUTION]
> A file exported without **Export Widths** still imports, and the hair package then reports
> `Unable to resolve strand diameters` and falls back to a uniform diameter, tapering, and tip
> scale. The Hair Tool checks for the param before it builds anything and puts a line in its report:
> `no 'width' — diameter/tapering will use fallbacks.`

## Step 2 — Bring the ABC files into Unity

Copy the `.abc` files anywhere inside `Assets`.

![An .abc groom selected in the Project window, with the Alembic importer inspector beside it](images/06-hair/11-abc-in-project.png)

> [!NOTE]
> Leave the Alembic importer settings alone. The Hair Tool sets **Scale Factor** to `0.01`,
> **Swap Handedness** on, **Import Curves** on, and **Create Curve Renderers** off, then reimports
> the file.

Before it touches anything, the tool scans the archive for three markers: the Ogawa container, the
curve schema, and per-point `width`. A file that is not an Ogawa archive, or that holds no curves at
all, is reported and skipped rather than half-imported.

## Step 3 — Add the grooms to the Hair Tool

Select **Tools > MHBridge > Hair > Hair Tool**. With a character selected in the Hierarchy, the
window opens already pointed at it.

![The MH Hair Tool window with four groom entries filled in and its controls numbered one through eight](images/06-hair/12-hair-tool-window.png)

| | Control | What it does |
|---|---|---|
| 1 | **Groom ABC files** | One entry per groom. **+ Add groom** adds a row, **−** drops one. |
| 2 | The profile field | The profile matched from the file name. Editable. |
| 3 | **Target character** | The `MHFace` to dress, and the head mesh that supplies root UVs. |
| 4 | **Output** | Where the generated assets go, and what a second run does with them. |
| 5 | **On re-run** and **Save character as prefab** | What a second run does with the files the first one wrote, and whether the dressed character is saved as an asset. |
| 6 | **Head collision** | The collision sphere and the layer it lives on. |
| 7 | **Character LOD** | The LOD mode. See [Step 5](#step-5--pick-a-lod-mode). |
| 8 | **Import** | Runs everything above, and fills the **Report** panel underneath. |

Add one entry per groom and assign its `.abc` file. Dropping a file matches it to a profile and
seeds the entry, and the profile field shows which one it picked.

Expand an entry for the per-groom settings:

![One groom entry expanded, showing the material field, the three gradient colors, and the attach preview line](images/06-hair/13-entry-expanded.png)

| Field | What it does |
|---|---|
| **Override material (used as-is, no copy)** | Off, the field under it is a read-only preview of the material the matched profile names. On, the field is yours, and the material you name is used exactly as it is — no copy, and the colors below are ignored. |
| **Length color**, **Root color**, **Tip color** | The three stops of the gradient along a strand, in the order the gradient runs. They tint a **copy** of the profile's material in the output folder, so a shipped preset stays as it shipped and two characters in one project can have two hair colors. |
| **Override attach** | Off, the line under it previews what the profile will do. On, you decide here. |

The colors are seeded once, from the profile's own material, the first time an entry matches a
profile. After that they belong to the entry, so matching another file later never overwrites a
color you chose.

Ticking **Override attach** replaces the preview with the two fields behind it: **Skin-attach roots**
and **Attach bone**. The bone field greys out while skin attachment is on, so that you pick one or
the other.

![The same entry with Override attach ticked, showing the Skin-attach roots checkbox and the Attach bone field](images/06-hair/14-entry-attach-override.png)

An entry with neither skin attachment nor a bone raises a warning in place, because a groom that
follows nothing rests in character space and stays there while the head moves.

### Two ways a groom rides the head

Which one a groom takes belongs to its profile.

**Bone attachment** is for scalp hair, ponytails, and anything rigid. The groom sits under the
character's `MHHair` node with a `ParentConstraint` on the `head` bone, or on whichever bone you
name, and the constraint's offset absorbs the MetaHuman head joint's own orientation. It depends on
nothing outside Unity.

**Skin attachment** is for eyebrows, beards, and mustaches, which have to travel with the
character's expression rather than with a bone. The roots are bound to the deforming head mesh
through the digital-human package's `SkinAttachment`, and no `ParentConstraint` is added. This is
also the half that constrains your LOD choice in [Step 5](#step-5--pick-a-lod-mode): a skin
attachment freezes the moment the head mesh it reads stops being skinned.

## Step 4 — Set the target, the output, and the collision

**Character (MHFace)** takes the character to dress, as a scene instance. Left empty, the import
still builds the hair assets and the groom prefabs, and adds nothing to any scene.

**Root UV mesh** fills itself in from the character's head renderer. It is the mesh the hair package
resolves root UVs against.

**Output** is shared with the another Cards Tool Window and remembered per project, so grooms and cards land in one
place rather than in two half-dressed folders:

| Field | Value | What it does |
|---|---|---|
| **Folder** | `Assets/Hair/Generated` | Has to be inside `Assets`. The Hair Tool fills `HairAssets`, `Materials`, and `Prefabs` under it, plus `SkinAttach` when a groom is skin-attached. A subfolder appears only when there is something to write into it. |
| **On re-run** | `OverrideExisted`, the default | Reuses the same generated files: colors refresh, and the character prefab is updated in place. |
| | `CreateNew` | Writes a new numbered set every run. Either way the Hair objects on the character is replaced, never duplicated. |
| **Save character as prefab** | on | Saves the dressed character to the output folder as `<Character>_Hair.prefab` and reconnects the scene object to it. Skin attachment bindings are baked in. Off, the result lives in the scene only. |

Both tools (Cards and Stands) write that one prefab under that one name, so building cards after importing grooms
updates it instead of stacking a variant on top of it.

**Head collision** adds a `SphereCollider` and a `HairBoundary` under
`MHHair/Collisions/HeadCollision`, so that strands collide with the skull instead of passing through
it. **Radius (0 = auto)** sizes the sphere from the `FACIAL_L_Ear`, `FACIAL_R_Ear`, and
`FACIAL_C_NoseBridge` bones. The automatic size is a starting point worth tweeking by hand afterwards.

The collider is real physics, so it needs a layer of its own:

1. Click **Create HairCollision layer**. The tool creates the layer in the first free user slot and
   selects it in **Collision layer**.
2. Select **Edit > Project Settings > Physics** and clear the `HairCollision` row of the collision
   matrix, so that nothing in the scene collides with a head-sized sphere.

![Project Settings on the Physics page, with every checkbox in the HairCollision row cleared](images/06-hair/15-physics-matrix.png)

> [!WARNING]
> With **Collision layer** left on `Default`, no collision is created at all. Layer 0 is the layer
> everything else in your scene lives on, so a head-sized sphere there would collide with the world:
> characters walking into their own hair volume, raycasts hitting an invisible ball.

## Step 5 — Pick a LOD mode

This is the decision the chapter is built around, so here is why it exists.

A strand groom costs the same at every distance. Its cluster LOD reduces quality rather than
geometry: the render mesh is built for every strand, and the LOD collapses strands onto cluster
centroids inside the vertex shader. And a skin-attached groom freezes as soon as the head LOD 0 mesh
stops being skinned, because the LOD group switched to a different mesh. The roots keep the pose
they were last resolved at, and no transform trick undoes it.

The tool asks the question only when it can matter. On a character whose head has a single LOD there
is nothing to switch off, and the mode picker is shown disabled to say so. When no groom in the list
is skin-attached — the usual case for a hairstyle on its own — the LOD chain cannot freeze anything,
and the section collapses to one checkbox, **Switch strands to cards at LOD1**.

Otherwise you pick one of three:

| Mode | What it does | What it costs |
|---|---|---|
| **Keep Head Skinned** | Sets `updateWhenOffscreen` on the head LOD 0 mesh, so the attachment target is skinned at every band and the head keeps its LOD chain. | That mesh is skinned every frame, including when another head LOD is rendered. |
| **Single Lod** | Takes the head's LOD chain off the character, so nothing can disable the attachment target. | The head always renders at full detail. |
| **Strands Plus Cards** | The groom stops rendering from LOD 1 on and a cards mesh takes over. | The strand mesh remains in memory for as long as the character exists. |

For the implementation and cost of each choice, see [LOD modes and swaps](#lod-modes-and-swaps).

## Step 6 — Import and read the report

> [!WARNING]
> Put the character in its bind pose first. Baking a skin attachment reads the head mesh in the pose
> it is in at that moment, and a `ParentConstraint` offset is measured against the character's
> current rest pose. Leave Play mode, stop any animation preview, and clear any pose the Animator is
> holding. A groom baked onto a displaced head is displaced for good, and re-importing is the only
> way back.

Click **Import**. A progress bar runs through the entries, and the **Report** panel fills with one
block per groom:

The same information reaches the Hierarchy. Under the character root there is now an `MHHair` node
holding `Collisions/HeadCollision` and profile nodes with groom instances beneath them.
Cards go under matching profile nodes in the sibling `MHCards` node. The character root carries `MHHairLodHelper`, plus `MHHairTopologyPin` in
**Strands Plus Cards**.

![Grooms and cards grouped by profile](images/06-hair/16-profile-hierarchy.png)


For example, two cards groups under the Hair profile produce this hierarchy:

```text
Character
├─ MHHair
│  ├─ Collisions
│  │  └─ HeadCollision
│  └─ Hair
│     └─ Hair_Sample_Groom
└─ MHCards
   └─ Hair
      ├─ Hair_Sample_CardsMesh_Group0
      │  ├─ Hair_Sample_CardsMesh_Group0_LOD0
      │  └─ Hair_Sample_CardsMesh_Group0_LOD1
      └─ Hair_Sample_CardsMesh_Group1
         ├─ Hair_Sample_CardsMesh_Group1_LOD0
         └─ Hair_Sample_CardsMesh_Group1_LOD1
```

The output folder holds what the run wrote: the hair assets, the tinted materials, the groom
prefabs, the skin attachment data, and the character prefab beside them.

> [!NOTE]
> A groom keeps rendering after you delete its `.abc` file, because a hair asset bakes the strand
> data into the `.asset` file. The Alembic file is needed only to rebuild it.

> [!WARNING]
> Skin-attached hair and the head have to live in the same prefab. The attachment holds a scene
> reference to one target on one character, and that reference does not survive being moved into a
> standalone groom prefab. Do not pull a groom out as its own prefab, and do not nest the character
> inside another prefab through the groom. **Save character as prefab** puts the whole dressed
> character into one asset, which is the arrangement that works.

### Read the import report

What to read in it:

| Line | What it tells you |
|---|---|
| `HairAsset … N group(s), rootUV=…` | The groom built. `ResolveFromCurves` means the `.abc` file carried root UVs; `ResolveFromMesh` means they were derived from the head mesh instead. |
| `material created` or `re-tinted` | The tinted copy was written, or an existing one was updated, and the path it went to. Absent when the entry overrides the material. |
| `attached under MHHair/<profile> via ParentConstraint -> head` | Bone attachment worked. |
| `skin-attached + baked onto …` | Skin attachment worked, and the resolve runs on the GPU or the CPU, following the project's own skinning setting. |
| `bone '…' not found`, or `rests in character space` | The groom follows nothing. Set an attach bone, usually `head`, or turn on skin attachment. |
| `[LOD mode] …` | What the mode you picked actually did. |
| `head collision SKIPPED — no collision layer` | See [Step 4](#step-4--set-the-target-the-output-and-the-collision). |
| `[prefab] … saved as a prefab` or `saved as a prefab variant` | The first save. A plain scene object becomes a prefab; an instance of an imported character becomes a variant of it. |
| `[prefab] … updated in place` | A later run on a character that is already an instance of that prefab. |
| `[<name>] Failed — …` | One entry threw. The others still ran, and the console holds the stack trace. |
| `--- done: N ok, M failed ---` | The tally. |

## What you get, and what it costs

What one **Import** does for you:

- Copies the hair profile's HairAsset template and rebuilds it on your `.abc` file, so every
  resolve, resample, unit, and cluster setting comes from an asset you can edit rather than from
  our code.
- Clones the profile's HairInstance prefab onto the built asset, and tints a copy of the profile's
  material with your gradient colors.
- Rigs the groom under an `MHHair` node on the character: a `ParentConstraint` on the `head` bone
  for scalp hair, or a skin attachment on the deforming head mesh for the facial grooms.
- Adds a head collision sphere sized from the character's own facial bones.
- Registers every groom with `MHHairLodHelper`, and pins the strand topology in memory when the
  mode calls for it.
- Saves the dressed character as a prefab and reconnects the scene to it.

What stays with you:

- Installing the packages and the Unreal plugin.
- Exporting the grooms, and the cards if you want them.
- Choosing a LOD mode. There is no cheap far LOD for strand hair, so this is a trade rather than a
  setting — see [Step 5](#step-5--pick-a-lod-mode).
- Tuning the look and the physics. The solver and its settings belong to the hair package, and
  MHBridge does not wrap them.

## LOD modes and swaps

### Keep head skinned

Nothing to configure. The help box under the picker states the trade in the tool:

```text
Keep head skinned preserves the full head LOD chain and keeps skin attachment active. Head LOD0 is skinned every frame, even when another LOD is rendered.
```

### Single LOD

Only the head loses its chain. The body's and the clothing's LOD chains play no part in the hair's
skin attachment, so they are left alone.

![The Character LOD section with Single LOD selected, showing the Apply by dropdown and its explanation](images/06-hair/17-lod-mode-b.png)

A second dropdown, **Apply by**, chooses how the head gets down to one level. The import carries out
whichever you pick; there is nothing to press afterwards.

- **Strip the character in the scene** is the default. The head's `LODGroup` and its LOD 1 and
  deeper mesh objects are deleted from the character in the scene. Nothing is rebuilt, so grooms,
  clothing, and constraints keep pointing at the same objects, Ctrl+Z reverses it, and the stripped
  character goes into the saved prefab. The imported asset does not shrink: those meshes are still
  built on every import, and this instance no longer holds them.
- **Reimport the character with head LODs = 1** sets the head part to one LOD on the character's own
  `.dcc` or `.dna` importer and reimports, before a single groom is attached.

> [!WARNING]
> The reimport route rebuilds the character. Grooms, clothing, and skin attachment bindings that are
> already on it may not survive, and Undo does not reach an import. The tool asks for confirmation
> before it does this, and getting the LOD levels back means raising the head LOD count in the
> importer and reimporting again.

### Strands plus cards

The groom goes silent from LOD 1 on and a cards mesh renders instead. The strands are silenced
through the hair package's own switches — the renderer is disabled and the simulation stops.

![Profile-wide cards link and swap band](images/06-hair/18-cards-profile-link.png)


The **Cards on this character** block lists each entry against the cards object it is linked to, or
`none yet`. **Find matches** re-scans the character, which is what you press after building cards
with this window open. **Build cards for this character…** opens the Cards Tool already pointed at
the character. Select the same profile in both tools: keywords help choose the profile, but the link matches
the profile node, not a cards filename.

You can build cards first or import grooms first. Opening or selecting a character with cards
makes the Hair Tool offer **Strands Plus Cards**. A successful cards build refreshes the open
Hair Tool for that character; this updates the UI and links, not all imported groom settings.
A new link changes a negative **Groom To Cards Lod** to `1` and preserves a nonnegative value.
The **Cards on this character** list identifies the matching profile node. Each profile node contains its existing cards groups.

> [!WARNING]
> **Strands Plus Cards** trades a stall for memory, permanently. Silencing a groom stops the
> requests that keep its strand topology mesh in the hair package's global cache, and about ten
> frames later the mesh is
> destroyed — so walking back towards the character rebuilds millions of polygons on the main
> thread. `MHHairTopologyPin`, which the import adds for you, prevents that by keeping the mesh
> alive. A full-strand tube mesh runs to tens or hundreds of megabytes, and it is held for as long
> as the character exists. That is right for one or two hero characters and wrong for a crowd.

## Step 7 — Hair cards (optional)

Cards are the cheap far LOD that strands do not have: a polygonal mesh standing in for the groom
from LOD 1 on, while the strand groom stays silent. MHBridge does not generate them, and it does not
need to — a MetaHuman groom ships with its cards already baked, as static meshes sitting in the same
Content Browser folder as the groom asset itself. Look for the meshes named
`<Groom>_CardsMesh_Group0_LOD0`, `…_LOD1`, and so on.

Export them from Unreal as FBX, one file per level, the same way you exported the grooms: right-click
the static mesh, then **Asset Actions > Export**, choosing FBX this time. You do not have to take
every level: one is enough for a character that never needs more than a cheap distant version, and
the Cards Tool builds whatever you give it.

![The Unreal Content Browser with the cards static meshes beside the groom asset, and the Export item in the context menu](images/06-hair/19-ue-cards-export.png)

![The exported cards FBX files on disk, one per LOD level](images/06-hair/20-ue-cards-fbx.png)

Copy the files into `Assets`, then select **Tools > MHBridge > Hair > Cards Tool**. It asks the same
questions the Hair Tool does: one entry per cards group, a material from the matched
profile, and the shared **Output** section. Click **+ Add cards group** for each additional group.
Keep `_Group0` and `_Group1` in separate entries, each with its own LOD sources.

![Separate cards group entries](images/06-hair/21-cards-groups.png)


1. Assign the LOD 0 FBX file to the first **LOD sources** slot. The tool fills the rest of the chain
   from that file's `_LOD{n}` siblings and names the entry after it. The list stays editable, so
   you can edit or remove sources. Keep all sources in an entry from the same cards group.
   The entry name retains `CardsMesh_GroupN`; only the trailing `_LOD{n}` is removed.
   Select the same hair profile used for this groom in the Hair Tool.
2. Optional: adjust the material. It works exactly as it does in the Hair Tool, except that the base
   comes from the profile's **card** material: off, the colors tint a copy in the output folder; on,
   **Override material** forces one material as it is. With no card material on the profile and no
   override, the materials that came with the FBX are kept and the colors do nothing.
3. Optional: tick **Override bone**. Left off, the cards ride the bone the matched profile names.
   For a skin-attach profile — eyebrows, beard, mustache — that bone field is there for the cards
   alone: the strand groom rides the head mesh and ignores it. Rigid cards need a bone;
   skinned cards are rebound to the character skeleton instead. An empty attach bone falls back to the head bone.
4. Assign **Character (MHFace)**.
5. Check **Output**. It is the same folder, the same **On re-run** rule, and the same
   **Save character as prefab** toggle the Hair Tool used.
6. Click **Build cards**.

> [!CAUTION]
> The Cards Tool works on a scene instance, not on a prefab asset. Handed a prefab asset, it reports
> `This is a prefab asset. Drag an instance into the scene.` and the build button stays disabled.

The build creates `MHCards/<profile>/<cards group>` on the character, with a `LODGroup`
over one node per level and, for rigid cards, a `ParentConstraint` on the bone.
The profile node uses the profile display name, or `Other` if no profile is selected.
Rebuilding one named cards group replaces that group and preserves the other groups. Cards exported bound to the head skeleton are rebound onto the
character's bones instead, and get no constraint. Any `UCX_` convex collision meshes in the
FBX are dropped and counted rather than built. The builder preserves the authored FBX pose,
including single-node files. If the cards face the wrong way, inspect the source FBX and build
report before changing its root rotation. The LOD thresholds come from
`MHImportSettings`, and the last level never culls away, so a single-level groom such as eyebrows
stays visible at any distance.

![Cards groups under their profile node](images/06-hair/22-profile-cards-hierarchy.png)


After import or build, the tools apply the LOD0 visibility state. Linked cards are normally
hidden until their swap band; cards without a linked groom remain visible. This is an initial
state, not a live distance-based Edit mode preview. Check the actual swap in Play mode.

The report identifies each cards group, its LOD and renderer counts, its attachment, filtered
collision meshes, and profile links. A link refers to `MHCards/<profile>`, so all cards groups
in that profile switch together.

The material copy is named `<Name> Cards Material.mat`, so a groom and its cards share the output
folder without sharing a file.

Once both tools have run, `MHHairLodHelper` on the character root holds the groom and its cards in
one entry and swaps between them at the LOD you chose. See
[What the import added to your character](#what-the-import-added-to-your-character).

### Cards without a strand groom

Nothing in the Cards Tool asks for a groom to have been imported first. Cards on their own are a
complete way to give a character hair, and a far cheaper one:

- **The decision in [Step 5](#step-5--pick-a-lod-mode) does not arise.** A cards container carries
  its own `LODGroup`, holding the levels you exported, so there is no mode to pick, no head LOD
  chain to strip, and no strand topology to hold in memory. Cards are an ordinary mesh, and Unity's
  own LOD system is all they need.
- **There is no strand solver to pay for.** The Cards Tool reads FBX files and writes plain meshes,
  so it is built to run in a project where the hair and Alembic packages are not installed at all.

Rigid cards follow a bone and do not follow facial deformation. This matters for eyebrows,
beards, and mustaches in close-up. Skinned cards are rebound to the character's bones and can
follow the deformation supported by their authored skin weights. Check your source FBX and
animation to decide whether its result is suitable for a close-up face.

## Tune the look

For color, opacity, and highlights, see [Hair materials](05-materials.md#hair-materials).
The `HairInstance` component controls strand shape and rendering. On Unity 6, you can also
configure HDRP to draw strands as [high quality lines](#high-quality-lines-unity-6).

### The HairInstance settings

Select a groom under `MHHair/<profile>` and its `HairInstance` component is the hair package's own inspector.
Everything in it belongs to the package rather than to MHBridge, and two settings are the ones to
start from.

![Groom material and HairInstance renderer](images/06-hair/23-groom-material-renderer.png)


**Strand Diameter**, under **Proportions**, is a toggle with a value in millimetres. Left off, the
diameter comes from the `width` param in your `.abc` file, which is the right default and the reason
[Export Widths](#step-1--export-the-grooms-from-unreal) matters. Turn it on to overwrite that with
one number for the whole groom — the fastest way to make a groom read thicker or finer. **Strand
Separation** beside it pushes strands apart, which reads as volume.

**Renderer**, under **Renderer**, decides how a strand is drawn:

| Value | What it is | Where it fits |
|---|---|---|
| **Builtin Lines** | One camera-facing line per strand. | The cheapest, and the one to fall back on when a groom is too expensive. |
| **Builtin Strips** | A flat ribbon per strand. | A middle ground: more body than lines, cheaper than tubes. |
| **Builtin Tubes** | A tube with real thickness. | What the shipped presets use, and what a hero character wants close to camera. |
| **HDRP High Quality Lines** | Lines drawn by HDRP's own line renderer rather than as mesh geometry. | Unity 6 only, and only after the setup below. On earlier versions it falls back to **Builtin Lines** with a warning in the inspector. |

![The same groom rendered as builtin lines, as strips, as tubes, and as HDRP high quality lines, side by side](images/06-hair/24-groom-variants.png)

The setting is per groom, so a beard can be cheaper than the scalp hair on the same head. It also
decides which strand mesh `MHHairTopologyPin` keeps alive in **Strands Plus Cards**, and tubes are
the heaviest of the built-in three.

### High quality lines (Unity 6)

On Unity 6, HDRP can draw strands through a line renderer of its own instead of as mesh tubes:
cleaner edges, and transparency that sorts correctly where strands cross. It is a hero-character
setting like the groom itself — the lines are rasterized, sorted, and shaded in passes of their own,
and those passes cost frame time — so measure it on your own target hardware. Below Unity 6 the
feature does not exist and none of this section applies.

Four things have to agree before a single strand is drawn this way:

| What | Where it lives |
|---|---|
| **Support High Quality Line Rendering**, and a strand width connected to the graph's **Width** block | `MHGroom.shadergraph` |
| **Renderer** set to **HDRP High Quality Lines** | the four presets in `Assets/MHBridge/Hair/Presets` |
| **High Quality Line Rendering**, under **Rendering** | every HDRP asset the project renders with |
| **High Quality Line Rendering** in the default camera frame settings, and an enabled **High Quality Lines** volume override | **Project Settings > Graphics > HDRP** |

The validator offers to configure all four when you click **Enable**. Select **Tools > MHBridge > Validate Setup**, or wait for the
**MHBridge Setup** window to open on its own after you unpack the groom sub-package or install the
hair package. Then:

1. Click **Enable** on **Grooms can render as HDRP high quality lines**. It turns the shader option
   on, connects the strand width, and switches the four presets.
2. Click **Enable** on each HDRP settings issue that appears next, including assets used by every quality level.
3. Click **Re-check** to confirm that no line-rendering issue remains.

![High quality lines setup issues](images/06-hair/05-validator-after-unpack.png)


The three project settings are separate entries on purpose: any one of them left off drops the
feature with no error anywhere, so the window names the one that is off. They appear only once a
groom asks for high quality lines, which is why they show up after the first button rather than
beside it. For the rest of the window — **Fix All**, **Don't show again**, and when it opens by
itself — see [Validate Setup](04-import-reference.md#validate-setup).

Two things the buttons deliberately leave alone:

- **Grooms already on a character.** An import copies a preset rather than linking to it, so a groom
  imported earlier keeps the renderer it was built with. Set **Renderer** on its `HairInstance` by
  hand, or import it again.
- **What the package ships.** The presets ship on **Builtin Tubes** and the shader ships with the
  option off, because the package has to open on editors below Unity 6 as well. Nothing changes
  until you press the button.

Strand width still comes from the same place it always did — the `width` param in your `.abc` file,
through **Strand Diameter** on the `HairInstance`. The groom material adds one multiplier on top of
it, [**HighQualityLineMultiplier**](05-materials.md#hair-materials).

## What the import added to your character

Two runtime components land on the character root, both plain public fields you can edit by hand. A
re-import merges rather than rebuilds, so your edits survive it.

**`MHHairLodHelper`** drives every groom from the character's own LOD band, which it reads from
`MHFace.ActiveLod`. There is no separate band controller.

![Profile-wide cards link](images/06-hair/18-cards-profile-link.png)


On the component itself:

| Field | Default | What it does |
|---|---|---|
| **Face** | filled in | Where the LOD band comes from. Found on the object and its parents at `Awake` when empty. |
| **Offscreen Behavior** | `Mute Simulation` | What happens while no camera sees the character. `Mute Simulation` silences the solver only. `Switch To Cards` also swaps card-equipped grooms to their cards, so the first visible frame shows cards rather than a strand pose frozen at some earlier moment. |
| **Grooms** | one entry per groom | The list below, filled by the two tools. |

On each entry, in the order the decisions are made:

| Field | Default | What it does |
|---|---|---|
| **Groom Instance** | set by the Hair Tool | The strand groom under `MHHair/<profile>`. |
| **Cards Instance** | set by the Cards Tool | The entire `MHCards/<profile>` node, containing every cards group of that profile. |
| **Groom To Cards Lod** | `-1` in **Keep Head Skinned** and **Single Lod**, `1` in **Strands Plus Cards** | The one band where the swap happens: below it the strands render, from it on they go silent and the cards take over. `-1` never swaps, so the strands render at every band. |
| **Manual Groom Quality** | off | Replaces the package's automatic cluster-LOD selection with the ramps below. Off, the package decides from screen coverage. |
| **Apply To Render** | on | Whether the render ramp is used. Off, rendering goes back to what the asset shipped with. |
| **Render Quality By Lod** | `1, 0.75, 0.5, 0.2` | Render quality per band, where `1` is full detail and `0` is the coarsest cluster level. The last entry repeats for deeper bands. |
| **Apply To Physics** | on | Whether the solver ramp is used. |
| **Physics Quality By Lod** | `1, 0.75, 0.5, 0.2` | Solver quality per band, read the same way. It is a separate ramp because the solver is the expensive half and can usually be dropped a band or two earlier than anything that shows. |

**`MHHairTopologyPin`** appears in **Strands Plus Cards**. It holds a list of grooms and one
**Log Once** checkbox that prints the pinned topologies, so that you can confirm it caught the
right meshes. Its cost is memory and nothing else — see the warning in
[Strands plus cards](#strands-plus-cards).

## Limits

- **Strand hair has no cheap far LOD.** The three modes in [Step 5](#step-5--pick-a-lod-mode) are
  the whole answer, and each of them pays for it somewhere.
- **Physics and simulation belong to the hair package.** MHBridge sets up the head collision sphere
  and ships `HairInstance` presets to start from. Those presets are a starting point, not
  production-tuned values.
- **The skin attachment resolve follows the project.** It is set from the project's GPU skinning
  setting and is not a LOD control.
- **Attachment cannot be switched at run time.** Turning root attachment on or off rebuilds
  attachment data and mutates an asset, so it is an edit-time operation.
- **Head collision only.** The shipped setup is a sphere on the skull. A neck capsule or a
  signed-distance-field collider is something you add through the hair package yourself.
- **High quality lines need Unity 6.** HDRP 17 is where the strand width reaches the line renderer,
  so below Unity 6 the renderer is not offered and MHBridge never proposes it. See
  [High quality lines](#high-quality-lines-unity-6).
- The package's platform requirements and HDRP requirement apply to hair as well. See
  [Requirements and limitations](04-import-reference.md#requirements-and-limitations).

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| The Hair Tool shows a page of packages instead of the tool | The hair or the Alembic package is missing. Add it in Package Manager as described in [Install the Unity packages](#install-the-unity-packages), wait for the recompile, and reopen the window. |
| **Add package from git URL** fails with a git error | Package Manager needs a `git` client of its own to resolve a git URL. Install Git, restart Unity, or take the package as a folder — see [Install a package without git](#install-a-package-without-git). |
| The tool opens on `No usable hair profiles were found` | The groom assets are not unpacked. Click **Unpack GroomAdditional.unitypackage**. |
| A groom came in on the wrong profile — eyebrows treated as scalp hair, for example | The `.abc` file name carries none of the keywords, so the fallback profile was used. Rename the file, add the keyword to the profile you want, or assign the profile by hand in the entry. |
| The console reports `Unable to resolve strand diameters`, or the report says `no 'width'` | The `.abc` file carries no `width` param. Re-export it with **Export Widths** on. |
| Eyebrows and beard do not move with the character's expression | They fell back to bone attachment because `com.unity.demoteam.digital-human` is not installed. The report names it, and it names any other skin-attach problem too, so read it before changing anything else. |
| The groom looks sparse, or like a solid helmet | Start with **Opacity Threshold** on its material, then **Strand Diameter** on the `HairInstance`. See [Hair materials](05-materials.md#hair-materials) and [HairInstance settings](#the-hairinstance-settings). |
| The groom disappeared after switching **Renderer** to **HDRP High Quality Lines** | One of the four pieces is missing: the material's shader, the HDRP asset, the frame settings, or the volume override. Open **Tools > MHBridge > Validate Setup** and click **Enable** on what it lists. |
| Every strand is the same thickness with high quality lines on | The shader's **Width** block has no input, so HDRP falls back to its own 0.08 cm default. The **Enable** button in **MHBridge Setup** connects it. |
| MHBridge keeps offering high quality lines and you do not want them | Tick **Don't show again** on that entry in **MHBridge Setup**. The question comes back only if the state it describes changes. |
| The colors in the tool did nothing | The entry has **Override material** ticked, so the material named there is used exactly as it is. Untick it to go back to a tinted copy of the profile's material. |
| The hair lags a frame behind the head | The pose chain is not running in the animation phase. Check the console for the warning `MHPoseLoop` writes when it cannot install its player-loop node, and see [Join the pose chain](03-animation.md#join-the-pose-chain). |
| The hair flies off the head, or sticks out sideways | A frozen skin attachment. Either the LOD mode does not keep the target skinned, or a `ParentConstraint` was left on a skin-attached groom, which applies the head's transform a second time. Re-import with **Keep Head Skinned** or **Single Lod**. |
| The frame rate spikes when the camera returns to a character | **Strands Plus Cards** without `MHHairTopologyPin`. Re-import with **Strands Plus Cards**, which adds it, or add the component and list the grooms. |
| The cards never appear | The entry's **Groom To Cards Lod** is `-1`, so the strands render at every band. Set it to the band the cards should take over at, usually `1`. |
| No head collision was created | **Collision layer** was left on `Default`. Click **Create HairCollision layer**, clear its row in **Project Settings > Physics**, and import again. A layer of your own works too — create it, then pick it in **Collision layer**. |
| The groom works in the scene and not in a prefab | Skin-attached hair and the head have to be in one prefab. Turn **Save character as prefab** on and import again. |
| The groom rests in place while the head moves | Its profile names no attach bone and no skin attachment. Tick **Override attach** and set **Attach bone** to `head`, which is the default for scalp hair. |
| The output folder is filling up with numbered copies | **On re-run** is set to `CreateNew`. Switch it to `OverrideExisted`. |

The general troubleshooting table is in the
[API reference](09-api-reference.md).

## For programmers

See [Hair and cards tools](09-api-reference.md#hair-and-cards-tools) for import options,
profile-based attachment, cards linking, and scripted high quality lines setup. The component
fields remain in [What the import added to your character](#what-the-import-added-to-your-character).
