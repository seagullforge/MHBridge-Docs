# Clothing

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

MHBridge dresses an imported character in one pass. The outfit's renderers are bound to the
character's own skeleton, the skin underneath stops rendering, and both results hold across the
whole LOD stack. In Unity that is one window and one button. The work that costs time sits
upstream of it, in Unreal, because that is where an outfit comes from — so this page spends its
first half there.

The package ships no outfits. You bring your own, the same way you bring your own character.

> [!NOTE]
> Fab's Standard License is not tied to Unreal Engine. An outfit you license there is yours to
> use in a project built with any engine, Unity included, while handing the source content to
> somebody else is a different matter. For what the license covers, and for the separate
> question of the MetaHuman character itself, see [Licensing and legal](10-licensing.md).

This page assumes a character that is already imported and standing in a scene, as described in
[Importing a MetaHuman](04-import-reference.md).

![Showcase of character with outfit](images/07-clothing/01-showcase.webp)

## On this page

- [What the Outfit Tool does](#what-the-outfit-tool-does)
- [Where outfits come from](#where-outfits-come-from)
- [Export an outfit from Unreal](#export-an-outfit-from-unreal)
- [Import the outfit into Unity](#import-the-outfit-into-unity)
- [Dress the character](#dress-the-character)
- [Hide skin under clothing](#hide-skin-under-clothing)
- [Clothing and the LOD stack](#clothing-and-the-lod-stack)
- [Clothing materials](#clothing-materials)
- [Combine several outfits](#combine-several-outfits)
- [Hats and head accessories](#hats-and-head-accessories)
- [Troubleshooting](#troubleshooting)
- [For programmers](#for-programmers)

## What the Outfit Tool does

**Apply outfit** attaches the outfit to the character's skeleton and preserves its LOD structure.
When you assign hide masks, it also hides the covered body or head skin. Nothing it touches is
destructive. The outfit FBX file is never modified, the character's imported meshes are never
modified, and everything the tool produces is either a new node in the scene or a new file
beside the character.

Two entries open the window:

- **Tools > MHBridge > Clothing > Outfit Tool** opens it on the character selected in the
  Hierarchy, if there is one.
- **Open Outfit Tool**, at the bottom of the `MHBody` inspector, opens it with that
  character already assigned. The button appears on scene instances only, because the tool
  edits a scene rather than a prefab asset.

![The Unity Tools menu open on the MHBridge Clothing submenu, showing Outfit Tool and Blend Body Masks](images/07-clothing/02-menu-entry.png)

![The MHBody inspector with the Open Outfit Tool button at the bottom](images/07-clothing/03-mhbody-inspector-button.png)

Seven controls make up the window:

![The Outfit Tool window with its seven controls numbered](images/07-clothing/04-outfit-tool-window.png)

| | Control | What it does |
|---|---|---|
| 1 | **Scene instance** | The character to dress. An instance in the scene, not a prefab asset. |
| 2 | **Body renderer** and **Head renderer** | The skin renderers the masks apply to. Auto-detected, and editable. |
| 3 | **Clothing** | One entry per outfit, with separate **Body** and **Head** mask fields. Either mask can be empty. |
| 4 | **Hide skin under clothing** | `None`, `Material mask`, or `Cut geometry`. |
| 5 | **Cull threshold** and the save toggles | The mask value that counts as hidden, and what the cut writes to disk. |
| 6 | **Apply outfit** and **Revert skin hiding** | Apply outfit dresses the character. Revert undoes the hiding, not the outfit. |
| 7 | **Report** | What the last operation did, line by line. |

The `Body: N LODs` and `Head: N LODs` lines show how many levels each assigned mask covers.
See [Clothing and the LOD stack](#clothing-and-the-lod-stack).

## Where outfits come from

An outfit is a skeletal mesh skinned to the MetaHuman body skeleton. Three sources produce one:

- **Fab**, Epic's marketplace, which carries both paid and free outfits built for the MetaHuman
  skeleton. They install into an Unreal project through the Epic Games Launcher or the Fab
  plugin.
- **MetaHuman Creator and the MetaHuman Character asset** in Unreal Engine 5.6 and later, where
  the outfit is assembled onto the character itself.
- **Your own skeletal mesh**, skinned to the MetaHuman body skeleton in any DCC application.

![An outfit listing on Fab, showing that it is built for the MetaHuman skeleton](images/07-clothing/05-fab-outfit.png)

MHBridge neither supplies nor resells any of it. The license on an outfit is between you and
whoever published it.

An outfit can carry a **body hide mask**, a **head hide mask**, or both. Each grayscale image uses
the corresponding mesh's UV0 space: white keeps skin visible, and black hides it. An outfit
without either mask still attaches, but the tool does not hide skin for it.

## Export an outfit from Unreal

This is the longest part of the workflow, and the one where a wrong turn costs the most.

1. In Unreal Engine, assemble the outfit on **your own** character, then find the outfit's
   SkeletalMesh in the Content Browser.

   ![The outfit SkeletalMesh in the Unreal Content Browser with the Asset Actions context menu open on Export](images/07-clothing/06-outfit-in-content-browser.png)

   > [!CAUTION]
   > You need the copy of the outfit that is skinned to your own character. Assemble the outfit
   > on the character first, then take the SkeletalMesh from the assembled MetaHuman rather
   > than from the source asset you downloaded.

2. Right-click the SkeletalMesh and select **Asset Actions > Export**, then choose FBX.

3. In the export options dialog, confirm the destination and export.

   ![The Unreal FBX Export Options dialog for a skeletal mesh](images/07-clothing/07-ue-fbx-export-options.png)

4. Export the outfit's hide masks when present. The body mask is commonly a grayscale PNG named
   `<Outfit>_Body_Mask.PNG`; some outfits also include a head mask. Look in the source asset's
   own folder. An outfit downloaded from Fab, for example, can carry its body mask under
   `Fab/MetaHuman/<AssetName>/Textures/<assetName>_Body_Mask`. Every asset is laid
   out differently, so look carefully. A hat may carry a head mask if it covers skin.

   ![The body hide mask texture at full size, white over visible skin and black under the outfit](images/07-clothing/08-body-mask-texture.png)

At the end you have `Outfit.FBX` and whichever body or head mask files the outfit provides.

> [!CAUTION]
> An outfit exported without LOD levels stays at full density at every distance. The body itself
> drops from about 125,000 triangles at LOD 0 to about 8,800 at LOD 3, so a jacket with one
> level consumes the entire saving the character's LOD stack was there to make.

The same export path brings an animation take out of Unreal, and [Animation](03-animation.md)
covers that side of it.

## Import the outfit into Unity

Copy the FBX file and any hide masks into the project. The outfit's animation settings do not
matter, because an outfit carries no animation. The material mask path handles the required sRGB
setting for hide masks.

One thing can happen on its own, and the Console reports it:

```text
[MHBridge] Normalized 1 bone name(s) in 'Assets/Outfits/Techwear.fbx' to the canonical MetaHuman spelling: Root->root.
```

Unreal stores bone names as `FName` values, which compare case-insensitively but keep the
spelling of whoever registered the name first. An outfit exported from Unreal therefore arrives
with `Root` where the character's skeleton has `root`. Unity treats those as two different
bones, and binding by name would miss. MHBridge normalizes the spelling as the FBX file
imports, against a list of bone names baked from the body DNA.

The rename is narrow on purpose. Facial joints keep their mixed case, names outside the list
are left alone, and the whole pass is skipped unless the hierarchy carries at least ten bone
names unique to the MetaHuman body skeleton. Your own rigs never qualify.

This does not happen for every outfit. Whether anything is renamed at all depends on how the
source skeleton was authored.

> [!WARNING]
> Do not drag the outfit FBX file into the scene. An FBX instance arrives with **Rotation X** at
> `-90`, which is the importer's axis compensation, and an Unreal animation clip carries the same
> compensation in its `root` curves. Under animation the two compound and the character lies down
> on its back. The production path is unaffected: **Apply outfit** never copies the FBX file's
> transforms, and skinned rendering ignores the renderer's transform outright — vertices are
> placed by the mesh's bind poses and the bound bones' world matrices. Leave the FBX file in the
> Project window and let **Apply outfit** put it on the character.

![The Inspector of an outfit FBX instance dragged into the scene, showing Rotation X at minus 90](images/07-clothing/09-standalone-rotation-quirk.png)

## Dress the character

1. Drag the character from the Hierarchy into **Scene instance**. Opening the tool from the
   `MHBody` inspector fills this in for you.

2. Check **Body renderer** and **Head renderer** for the masks you plan to use. The tool detects
   both automatically. If a masked part has no renderer, assign it manually; **Apply outfit**
   stays disabled until that part has a renderer.

3. Read the `Body: N LODs` and `Head: N LODs` lines. Each shows how many levels the corresponding
   mask covers.

4. Assign the outfit and its masks in the **Clothing** list. The first field takes the outfit,
   either the FBX asset from the Project window or an instance. Assign its body mask to **Body**
   and its head mask to **Head**. Leave a missing mask empty. Click **+** for a second outfit, and
   **−** to drop a row.

5. Choose a mode under **Hide skin under clothing**. See [Hide skin under clothing](#hide-skin-under-clothing).

6. Click **Apply outfit**. The button stays disabled until there is a character
   and at least one outfit.

![The Outfit tool with all outfit assets applied](images/07-clothing/10-outfit-tool-after-apply.png)

Under the hood the tool copies each of the outfit's renderers onto a new node under the
character root and re-binds its bones by name, case-insensitively. The outfit asset itself is
never touched, and one Ctrl+Z reverses the whole **Apply outfit** operation.

Running **Apply outfit** again with the same outfit replaces what the last run produced rather than
stacking a second copy on top.

### Read the report

**Apply outfit** writes one report covering the attach and any body or head hiding. It
fills the **Report** panel and goes to the Console as a single message under
`[MHBridge] Outfit Tool:`. A successful run reads like this:

```text
Attach Clothing — SUCCESS
Character: ToUnityMH
Renderers: 2 attached, 0 failed
Outfit LOD group kept — ToUnityMH_Outfits: 2 level(s), 2 renderer(s) rebound
  • ToUnityMH_Outfits_LOD0: attached (matched 342 bones)
  • ToUnityMH_Outfits_LOD1: attached (matched 342 bones)
Saved: Assets/SampleContent/DCC/ToUnityMH_body_lod0_mesh_Culled.asset (+ CullRecord).
  ToUnityMH_body_lod0_mesh: removed 39821/60816 triangles, 32334 -> 11337 vertices.
Saved: Assets/SampleContent/DCC/ToUnityMH_body_lod1_mesh_Culled.asset (+ CullRecord).
  ToUnityMH_body_lod1_mesh: removed 9945/15196 triangles, 8568 -> 3049 vertices.
Body cut geometry: 2 LOD level(s), removed 49766/76012 triangles in total (threshold 0.1). Reversible — the DNA meshes are untouched.
```

The counts depend on the outfit. What matters is the first line, and after that the lines that
appear only when something is worth knowing:

| Line | What it means | What to do |
|---|---|---|
| `attached (matched N bones)` | The renderer is bound to the character's skeleton. | Nothing. |
| `unweighted-unmatched -> placeholder (N)` | The outfit references corrective or twist joints your character does not have, and skins to none of them. | Nothing. This is the normal case. |
| `FAILED — N weighted bone(s) have no match in the character` | A bone the outfit actually skins to is missing, so the renderer was rejected and nothing was added to the scene. | The outfit is not on the MetaHuman body skeleton. Re-export it from a MetaHuman rig. |
| `Duplicate bone names (matched to first)` | Two objects in the character hierarchy carry one name. | Check for a node added by hand. |
| `Outfit LOD group kept — …: N level(s), M renderer(s) rebound` | The outfit's own LOD structure came across. | Nothing. See [Clothing and the LOD stack](#clothing-and-the-lod-stack). |

A run with unmatched weighted bones is logged as a warning rather than a message, so it stands
out in the Console.

## Hide skin under clothing

Without this step the skin renders through the fabric, most visibly at the collar, at the cuffs,
and anywhere the outfit sits close to the body.

![The character in the outfit with body hiding off and with it on, side by side](images/07-clothing/11-body-hidden-comparison.png)

Two modes solve it for each part with an assigned mask. Switching modes restores the original
mesh or clears the material mask for that part.

| | **Material mask (alpha clip)** | **Cut geometry** |
|---|---|---|
| What it does | The mesh is untouched, and the shader discards the pixels the mask covers. | Triangles the mask fully covers are removed from the mesh. |
| What it gains | Reversible instantly, and switchable at runtime. | Fewer vertices to skin, and no overdraw. |
| What it costs | Alpha clipping gives up part of the early-Z benefit. | An edit-time step, repeated whenever the outfit changes. |
| What it needs | The `_BodyHideMask` property in each masked part's materials, with separate body and head materials. | A mask for each part you want to cut. |
| What it writes | A combined mask PNG per part when two outfits or more provide masks for that part. | A culled mesh per masked LOD level. |

The **Body** mask affects only the body; the **Head** mask affects only the main head mesh. If an
outfit provides both, the tool processes both LOD stacks. If neither is assigned, it only
attaches the outfit. Eyes, teeth, and hair are not part of the head mask operation.

### Material mask

The mode is available only when every material on a masked body or head LOD stack supports
`_BodyHideMask`. `MHBody_Graph`, the body shader that ships with the package, has it. A plain
HDRP Lit material does not. The body and head also need separate materials so each can hold its
own mask. The line under the disabled option names a material that blocks the mode:

![The Material mask option disabled, with the line naming the body material that lacks the property](images/07-clothing/12-material-mask-disabled.png)

**Apply outfit** assigns each mask to the matching part's materials, sets the threshold, and enables
the `_USE_BODY_HIDE_MASK` toggle and shader keyword. It also turns on **Alpha Clipping** in HDRP
**Surface Options** when that was off, and says so in the report:

```text
Enabled Alpha Clipping in the Surface Options for 'ToUnityMH_Body'.
Body material mask assigned to 1 material(s) [ToUnityMH_Body] covering 2 LOD level(s) (threshold 0.1).
```

**Revert skin hiding** puts Alpha Clipping back exactly as it found it rather than forcing it
off, so a skin material you deliberately set to clip survives the round trip.

If a body or head material is embedded in an imported DCC asset, **Material mask** is disabled.
The tool cannot save changes to that material across re-imports. Use **Extract Textures &
Materials** on the character's importer to create separate `.mat` assets, as described in
[Extract: preview versus production](04-import-reference.md#extract-preview-versus-production).
The mode is also disabled if the body and head share one material, because one material cannot
store two independent masks. Assign separate materials or choose **Cut geometry**.

**Cull threshold** decides which mask values count as hidden. It defaults to `0.1`, which is
Epic's own `MaxCullValue`, and there is no reason to move it unless a particular outfit shows a
seam.

### Cut geometry

This is the default mode. The rule is Epic's own: a triangle is removed only when **every** mask
texel overlapping its UV footprint is at or below the threshold. Triangles at the edge of the
masked area are kept, which is why the cut leaves no visible seam.

![The Hide skin under clothing section with Cut geometry selected, showing the threshold and both save toggles](images/07-clothing/13-cut-geometry-section.png)

The cut is reversible. The DNA-built mesh is never modified, and every re-cut starts from the
original rather than from an already-cut mesh.

Two toggles decide what lands on disk. **Save culled mesh as .asset** is on by default;
**Save CullRecord asset** is off until you enable it:

- **Save culled mesh as .asset** writes the cut mesh as a file next to the character. With it
  off, the mesh is serialized into the scene or the prefab instead, and **Revert skin hiding**
  then has to fall back on the character prefab's own meshes.
- **Save CullRecord asset** writes the record that makes the
  revert exact: the original mesh, the masks, the threshold, and a table mapping each vertex of
  the cut mesh back to the source mesh.

Keeping both on stores the meshes and records as assets. The files appear beside the character
asset, one pair per LOD level:

![The Project window showing the culled mesh and cull record assets for several LOD levels](images/07-clothing/14-culled-assets-on-disk.png)

The report gives one line per level and a total:

```text
Saved: Assets/SampleContent/DCC/ToUnityMH_body_lod0_mesh_Culled.asset (+ CullRecord).
  ToUnityMH_body_lod0_mesh: removed 39821/60816 triangles, 32334 -> 11337 vertices.
Saved: Assets/SampleContent/DCC/ToUnityMH_body_lod1_mesh_Culled.asset (+ CullRecord).
  ToUnityMH_body_lod1_mesh: removed 9945/15196 triangles, 8568 -> 3049 vertices.
Body cut geometry: 2 LOD level(s), removed 49766/76012 triangles in total (threshold 0.1). Reversible — the DNA meshes are untouched.
```

![A wireframe view of the body before and after the cut, with the torso removed under the jacket](images/07-clothing/15-wireframe-cut.png)

> [!NOTE]
> When a level cannot be cut, because its mesh has no UV0 channel or its mesh reference broke,
> the report marks that one level `⚠ … cut skipped because …` and processes the rest of the stack anyway.

### Revert skin hiding

**Revert skin hiding** puts the original body and head meshes back and clears their material masks. It
restores each level through its `CullRecord` first and through the character's prefab second:

```text
Body mesh restored to 'ToUnityMH_body_lod0_mesh' (via CullRecord). Derived assets kept on disk (reapplying replaces them).
(2 of 2 body LOD level(s) restored.)
Body material mask cleared on [ToUnityMH_Body].
```

Two things it deliberately does not do. It does not delete the culled meshes, the records, or the
combined mask, because those are files in your project and a later **Apply outfit** run replaces them. And
it does not take the clothing off: the outfit stays on the character until you delete its nodes
in the Hierarchy, or press Ctrl+Z straight after the attach.

## Clothing and the LOD stack

Both modes apply to every LOD in each masked part's stack rather than to LOD 0 alone. Hiding
only the closest level would put skin back through the fabric at the first camera transition.

The tool finds each stack by walking up from its body or head renderer to the nearest ancestor that
carries a `LODGroup` component or whose name ends in `_LodGroup`, and it takes the level order
from the `LODGroup` itself. A character built without such a node — an older import, or a foreign
rig — works as a single renderer. The `Body: N LODs` and `Head: N LODs` lines report the stacks.

What each mode does across the stack:

- **Material mask** goes on every unique material of the stack. When all levels share one
  material, which is the usual case, that is a single assignment. This is why the mode requires
  the property on every material rather than on the first one: a distant level left on a plain
  material would stay fully visible through the outfit.
- **Cut geometry** runs once per level, because each level has its own topology, and writes its
  own culled mesh and its own record.

The outfit's own LOD levels survive the attach. For each `LODGroup` in the outfit, the tool
creates one container node under the character root, puts the level renderers under it, and adds
a fresh `LODGroup` that carries the source's transition heights and fade settings while pointing
at the new renderers.

![The character Hierarchy with the outfit LOD container expanded, and its LODGroup component in the Inspector](images/07-clothing/16-outfit-lod-container.png)

The character's own LOD behavior is covered in
[The LOD stack](04-import-reference.md#the-lod-stack).

## Clothing materials

The attach copies each renderer's materials across as they are, so an outfit renders on whatever
material its FBX file arrived with. MHBridge ships no clothing shader and builds no material of
its own. There is nothing MetaHuman-specific about a jacket, and no two outfits are shaded alike.

This is the part of the workflow with no recipe. Every outfit carries its own texture set and
needs its own kind of material: a coat that is opaque and rough, a lace panel that needs alpha
clipping, a visor that is transparent, a jersey that is two-sided. An FBX file brings across the
material slots and their names, not the Unreal material graph behind them, so Unity creates a
plain material per slot and the character comes out dressed in untextured fabric. The look has to
be rebuilt in Unity, one outfit at a time, from the source material in Unreal.

A basic recipe for a common clothing material would be something like this:

1. In Unreal, open the material assigned to that slot on the outfit's SkeletalMesh, and read its
   texture parameters: the base color, the normal map, and whatever map packs roughness,
   metallic, and ambient occlusion.
2. Export those textures. Right-click each one and select **Asset Actions > Export**, or take
   them from the source asset's own folder, which for an outfit from Fab is the same folder that
   carries the body hide mask.
3. In Unity, create an HDRP material and wire the maps into it.
4. Assign the material in the **Materials** tab of the outfit FBX file's importer.

Roughness is the one map that rarely transfers unchanged. HDRP reads a mask map as metallic in R,
ambient occlusion in G, the detail mask in B, and smoothness in A, and smoothness runs opposite to
roughness, so a packed Unreal map usually needs its channels rearranged before it means the same
thing.

The outfits in the screenshots on this page run on plain materials, close to what the FBX import
produced. This page covers attaching an outfit and hiding the body under it, and shading fabric is
a separate job that belongs to your art pipeline rather than to the tool.

## Combine several outfits

Several rows in the **Clothing** list are several outfits, each with its own body and head masks.
The tool combines body masks separately from head masks. The operator is a per-texel minimum:
skin hidden by any one outfit stays hidden.

On the material mask path, combined images are saved beside the corresponding meshes as
`<Character>_Body_Mask_Combined.png` and `<Character>_Head_Mask_Combined.png` when needed. The
report gives their paths. On the cut geometry path, the cut reads the combined values directly.

To combine masks without attaching any clothing, select
**Tools > MHBridge > Clothing > Blend Body Masks**. The window takes a list of textures and an
output size of 1024, 2048, or 4096, blends them by the same rule, previews the result, and writes
it with **Save PNG…**.

![The Blend Body Masks window with two masks loaded and the combined preview below them](images/07-clothing/17-blend-body-masks.png)

> [!NOTE]
> A hide mask is data rather than color, so it has to be sampled in its stored values. The tool
> turns **sRGB (Color Texture)** off on a mask it is handed and saves every blend with it off,
> and the report says when it did. Turning it back on by hand shifts the threshold and changes
> which triangles disappear.

## Hats and head accessories

A hat is an ordinary skeletal mesh skinned to the chain that ends at the `head` bone, and the
attach handles it with no special mode. Add it as a clothing entry. If the hat has no skin hide
mask, leave **Body** and **Head** empty and choose **None — attach clothing only**. If it has a
head mask, assign it to **Head** and choose a hiding mode.

![A hat attached to the character, with the Outfit Tool set to None — attach clothing only](images/07-clothing/18-hat-attached.png)

Hair under a hat is not hidden automatically, in Unity or in Unreal. Either choose a
hat-compatible hairstyle, or turn the groom off while the hat is on. See [Hair](06-hair.md).

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| The character lies on its back once a clip plays | The FBX file was dragged into the scene rather than applied. Delete the instance and use **Apply outfit**. |
| **Material mask** is disabled | A masked body or head material lacks `_BodyHideMask`, is embedded in a DCC asset, or is shared by body and head. Read the reason under the option. Use a mask-capable, separate `.mat` asset, or choose **Cut geometry**. |
| The skin shows through on distant levels | Not every LOD level was cut, or a level's material carries no mask. Check the `Body: N LODs` and `Head: N LODs` lines, then run **Apply outfit** again. |
| A mask stopped working after an FBX re-import | The material was embedded in the imported asset when the mask was applied. Extract it as a `.mat` asset, assign it to the character, and apply the mask again. |
| **Apply outfit** reports an error about a prefab asset | The character is an asset rather than a scene instance. Drag it into the scene and assign that. |
| The attach report reads `FAILED — N weighted bone(s) have no match` | The outfit is skinned to a skeleton the character does not have. Re-export it from a MetaHuman rig. |
| The outfit disappears at a certain distance | Its LOD thresholds came from Unreal. Check the `LODGroup` component on the outfit's container node. |
| The outfit renders white or untextured | Its materials were never built. An FBX file carries material slots, not the Unreal material behind them. See [Clothing materials](#clothing-materials). |
| A gap or a seam at the edge of the outfit | The **Cull threshold** is too high, so triangles at the edge are being removed. Lower it, starting from the default `0.1`. |
| A skin LOD level has no mesh after Ctrl+Z | Undo of an **Apply outfit** operation made with **Save culled mesh as .asset** off. Click **Revert skin hiding**, and keep the toggle on if you want Ctrl+Z to work. |

## For programmers

Everything the tool is built from is public, in the `MHBridge.Editor.Clothing` namespace. It
lives in the Editor assembly, so none of it ships in a build. For full signatures, see the
[API reference](09-api-reference.md).

- `ClothingAttacher.Attach(GameObject clothingSource, GameObject characterRoot)` returns an
  `AttachReport`, and `ClothingAttacher.FormatReport()` turns one into the text the window shows.
  This is the entry point for a batch script that dresses many characters.
- `AttachReport` and `RendererAttachResult` carry what bound and what did not, with
  `HasWeightedMisses` as the single check for a failed rebind.
- `HiddenFaceCuller.CullHiddenFaces()` runs the cut on a `Mesh` against a `Texture2D`, or against
  mask values you have already read, and returns a `CullResult` whose `NewToOldVertex` table maps
  each vertex of the result back to the source mesh.
- `MaskBlender.Blend()`, `BlendValues()`, `ReadMaskValues()`, and `SaveAsPng()` are the mask
  arithmetic with no UI attached. `ReadMaskValues()` is the one place a mask becomes numbers,
  which is what keeps the blend and the cut reading it identically.
- `OutfitToolWindow.FindBodyRenderer()`, `FindHeadRenderer()`, and `FindLodStack()` are public so
  that your own tooling can resolve the same skin renderers and LOD stacks as the window.
- `BodyCullRecord` is the reversibility record, with the original mesh, the source masks, the
  threshold, and the vertex remap as public fields.

### Change clothes at runtime

The material mask path is a texture and a shader keyword on each masked skin material, which makes
re-dressing at runtime a matter of swapping them. Assign a different mask to `_BodyHideMask` on
the body or head material, keep `_USE_BODY_HIDE_MASK` enabled, and enable or disable the outfit's
nodes to match. The cut geometry path is edit-time and offers nothing equivalent, because the
mesh has already changed.
