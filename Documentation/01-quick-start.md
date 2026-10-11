# Quick start

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

Import your own MetaHuman, put it in the included scene with prepared lighting, and try
facial and body animation. The demo contains no character: you supply your own DCC export.

![A MetaHuman character standing in a Unity scene under HDRP lighting](images/01-quick-start/01-result.webp)

## On this page

- [Before you start](#before-you-start)
- [Install the package](#install-the-package)
- [Export your MetaHuman from Unreal](#export-your-metahuman-from-unreal)
- [Import the DCC export](#import-the-dcc-export)
- [Validate the project setup](#validate-the-project-setup)
- [Open the demo scene](#open-the-demo-scene)
- [Assign the demo animation](#assign-the-demo-animation)
- [Check the rig and play animation](#check-the-rig-and-play-animation)
- [Try the Face Board](#try-the-face-board)
- [Check the import result](#check-the-import-result)
- [What you get](#what-you-get)
- [Where to go next](#where-to-go-next)

## Before you start

| Requirement | Version or setting |
|---|---|
| Unreal Engine | 5.8 or later, with your MetaHuman ready for DCC export |
| Unity | 2022.3 LTS or later, tested through Unity 6.5 |
| Render pipeline | HDRP 14 on Unity 2022.3; HDRP 17 on Unity 6 |
| Editor platforms | Windows, macOS, Linux |
| Player platforms | PC (Windows, macOS, Linux), consoles* |

*Console builds require the adapter source kit and your platform SDK; they have not
been built or tested with console SDKs. See [Build for consoles](11-console-builds.md).

For the character's terms, see [Licensing and legal](10-licensing.md).
You do not need the optional groom packages for this first import.

## Install the package

1. Open your HDRP project in Unity.
2. Import the package contents and wait for Unity to finish importing and compiling.

The **Tools > MHBridge** menu and `Assets/MHBridge/Demo` folder are available after import.

![Import the MHBridge package with the Demo folder included](images/01-quick-start/02-package-import.png)

## Export your MetaHuman from Unreal

MHBridge imports what Unreal's DCC export writes, so start there. The numbers in this
procedure point at the callouts on the screenshot below.

1. In Unreal Engine, open your MetaHuman Character asset in the MetaHuman editor window.
   If MetaHumans in Unreal are new to you, start with this
   [Ultimate Guide to MetaHuman Creation & Facial Animation | Unreal Engine 5 Tutorial by Bad Decisions Studio](https://www.youtube.com/watch?v=iNfezN4WzZI).

2. Open the **Export** tab (1), then select **DCC Export** (2).

3. In the destination field (3), choose a folder outside your Unity project.

4. Turn on the zip option (4), so that the export writes a single archive.

5. Click **Create Full Rig** (5). The DCC export needs the full rig, because that is what
   fills the DNA files.

6. Click **Download Texture Sources** (6), which fetches the textures the export writes
   into `Maps`.

7. Click the export button (7).

![The MetaHuman editor on the Export tab, with the DCC export options numbered one through seven](images/01-quick-start/03-unreal-export-menu.png)

The export writes one `.zip` archive.

MHBridge ships no sample characters, so an export of your own is where a project starts.
A raw `.dna` file is the other way in, described in
[Importing a MetaHuman](04-import-reference.md).

## Import the DCC export

A `.dcc` file is the DCC export archive living in your project as a finished character:
meshes, one skeleton, HDRP materials, an Avatar, and the runtime DNA.

1. In Unity, select **Tools > MHBridge > Import > DCC Zip**.

   ![The Unity Tools menu open on the MHBridge Import submenu, showing DCC Folder and DCC Zip](images/01-quick-start/04-menu-import-dcc.png)

2. Select the archive that you exported. The file picker accepts `.zip` and `.dcc`.

3. Choose where inside `Assets` to keep the character, then save. The file name comes
   from the character name in the manifest.

MHBridge reads the manifest before it writes anything into your project. The manifest has
to name a head DNA and a body DNA, and both files have to be inside the archive. If they
are not, MHBridge names what is missing and stops, so a wrong file costs you a dialog
rather than a failed import.

When the progress bar closes, the new asset is selected in the Project window and the
Console reads
`[MHBridge] Packaged the DCC export as '<path>'. Drag the asset into a scene to instantiate it.`

![The new .dcc asset selected in the Project window, with its thumbnail visible](images/01-quick-start/05-project-dcc-asset.png)

> [!NOTE]
> If Unreal gave you a folder rather than an archive, use **Tools > MHBridge > Import >
> DCC Folder** instead. It packs the folder into the same kind of `.dcc` file. A finished
> `.dcc` that a colleague hands you needs no menu item at all: copy it into the project
> and the same importer runs on it.

## Validate the project setup

1. Open **Tools > MHBridge > Validate Setup**. The window can also open after import when it finds issues.
2. Click **Fix All** for the fixes that the window supports.
3. Resolve any manual assignments named by the remaining issues.
4. Click **Re-check** and confirm that the window reads **Everything checks out**.

If HDRP is not active, assign an HDRP asset in **Project Settings > Graphics** and for each
used quality level in **Project Settings > Quality**. Empty material diffusion fields need
the matching skin, teeth, iris, or sclera profile. **Fix All** cannot choose those assets for you.
See [Validate Setup](04-import-reference.md#validate-setup) for the full reference.

![Material-specific setup issues and diffusion profiles](images/05-materials/04-diffusion-profiles.png)

## Open the demo scene

1. Open `Assets/MHBridge/Demo/DemoScene.unity`.
2. In the Hierarchy, find the empty `YOUR METAHUMAN` object. It marks where your character belongs.
3. Drag your imported `.dcc` asset into the scene as a root object and set its **Position** to `(0, 0, 0)`.
4. On the `MH Demo` object, assign your character to **Character** in **MH Demo Controller**.
5. Remove the empty `YOUR METAHUMAN` placeholder after assigning the character.

The scene supplies lighting, a camera, animation controls, and a Face Board panel. The
character root supplies `Animator`, `MHBody`, `MHFace`, and `MHFaceClipDriver` from the import.
The [import result](04-import-reference.md#what-an-import-produces)
explains the hierarchy of MHCharacter.

![Empty demo scene with YOUR METAHUMAN and MH Demo](images/01-quick-start/06-demo-empty-scene.png)

![Your imported character assigned to MH Demo Controller at the origin](images/01-quick-start/07-demo-character-setup.png)

For your own scene, add a Directional Light and an HDRP Volume with sky and fog settings
before judging the materials. The demo scene is the prepared-lighting starting point.

## Assign the demo animation

The **Animator** is the component on your character. Its **Controller** chooses and combines
clips. Its **Avatar** describes your character's rig. These are three separate assignments.

1. Select your character's `Animator` component.
2. Assign `Assets/MHBridge/Demo/Anims/Demo_AnimController.controller` to **Controller**.
3. Keep your own character's `<Name>_Avatar` sub-asset in **Avatar**. The bundled FBX files use **Generic** by default. For a Humanoid character, set their **Rig > Animation Type** to **Humanoid** and click **Apply**.

The demo script checks these references; it does not assign the character's controller for you.
Generic clips require compatible bone paths. Assigning your Avatar does not retarget an
arbitrary skeleton. See [Try the included animation](03-animation.md#try-the-included-animation)
for the bundled clips and the animation-FBX Avatar option.

![Animator with demo Controller and your own Avatar](images/03-animation/02-demo-animator-avatar.png)

## Check the rig and play animation

1. Press **Play**.
2. Select your character and check that `MHFace` and `MHBody` show **Rig active** in the Inspector.
3. In the Game view, read the demo's **Errors** panel if it appears. Leave Play mode, fix the named assignment, and press Play again.
4. Click **Play smile**, **Play scream**, or **Play fear** for the face, and **Play exercise** for the body.
5. Click **Stop face** or **Stop body** to pause or play animation.

![The MHFace inspector in Play mode showing the Rig active banner](images/01-quick-start/08-play-rig-live.png)

![Demo animation controls and your own character in Play mode](images/01-quick-start/01-result.webp)

Under **Look at**, choose **Body** or **Head** to frame the character. Scroll outside the UI
to zoom; drag with the left mouse button outside the UI to orbit. Distance and horizontal
and vertical angles have limits. The focus points come from `spine_05` and `head` at startup,
so animation does not move the focus reference. Switching views resets the camera height
to the cached focus height and adjusts the framing distance.

## Try the Face Board

The demo binds its existing Face Board panel to your assigned character. In the Game view,
click **Show or hide** under **Face Board**, then enable **Control face from GUI** and drag
a handle. Turn that checkbox off before returning to facial clips.

For editor posing and a panel in your own scene, continue with [Face Board](02-face-board.md).

## Check the import result

If an asset turns red or fails to import, read its Console message and follow
[Import troubleshooting](08-troubleshooting.md#import). The full generated-asset description
and import report are in [What an import produces](04-import-reference.md#what-an-import-produces).

## What you get

Expand the `.dcc` asset in the Project window to see what the import produced.

![Imported character sub-assets and four split eye maps](images/01-quick-start/09-asset-contents.png)


| What | Where |
|---|---|
| The character prefab | the main object of the `.dcc` asset |
| Head and body meshes, one set per LOD | sub-assets |
| HDRP materials | sub-assets |
| Split iris and sclera color and normal maps | sub-assets; four PNG files after extraction |
| The Avatar, named `<Name>_Avatar` | sub-asset |
| The runtime DNA the rig loads | sub-assets `dna_head` and `dna_body` |

## Where to go next

- [Face Board](02-face-board.md) — pose the face by hand.
- [Animation](03-animation.md) — try the included CSV, then import your own clips.
- [Import reference](04-import-reference.md) — extraction, reimport, and LOD settings.
- [Materials](05-materials.md) — edit appearance under controlled lighting.
- [Hair](06-hair.md) and [Clothing](07-clothing.md) — independent additions to your character.
- [Troubleshooting](08-troubleshooting.md) — find a symptom and its main fix.
- [API reference](09-api-reference.md) — control and extend the rig from code.
