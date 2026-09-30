# MHBridge Importer

Get MHBridge on the [Unity Asset Store](https://assetstore.unity.com/packages/slug/403996).

MHBridge takes a MetaHuman out of Unreal Engine and puts it into a Unity scene with a live
face and body rig. RigLogic — the same evaluation Unreal runs — computes correctives, blend
shapes, and wrinkle maps every frame, so the character in your scene stays rigged rather than
baked down to a mesh.

![A MetaHuman character standing in a Unity scene under HDRP lighting](Documentation/images/06-hair/01-showcase.webp)

## Start here

Start with [Quick start](Documentation/01-quick-start.md) to import your character and try it
in the included demo scene. Continue with Face Board or Animation, then open the Import reference
when you need it. Hair and Clothing are independent workflows.

| I want to… | Open |
|---|---|
| Import my first character | [Quick start](Documentation/01-quick-start.md) |
| Pose a face by hand | [Face Board](Documentation/02-face-board.md) |
| Test animation or the included CSV | [Animation](Documentation/03-animation.md) |
| Extract assets, reimport, or change LODs | [Import reference](Documentation/04-import-reference.md) |
| Tune materials | [Materials](Documentation/05-materials.md) |
| Add only hair cards | [Cards workflow](Documentation/06-hair.md#step-7--hair-cards-optional) |
| Add strands or a strands/cards combination | [Hair](Documentation/06-hair.md) |
| Dress my character | [Clothing](Documentation/07-clothing.md) |
| Fix a problem | [Troubleshooting](Documentation/08-troubleshooting.md) |
| Drive or extend the rig from code | [API reference](Documentation/09-api-reference.md) |
| Check content and license information | [Licensing and legal](Documentation/10-licensing.md) |

## Included samples

The package includes these samples under `Assets/MHBridge/Demo`:

| Sample | Use |
|---|---|
| `DemoScene.unity` | Prepared lighting, a camera, animation buttons, and a Face Board panel. Add your own character at the empty `YOUR METAHUMAN` placeholder. |
| `Anims/Demo_AnimController.controller` | Separate body and face layers for the demo controls. |
| `Anims/Body` and `Anims/Face` | Body idle/exercise and facial animation FBX files. |
| `Anims/Face/Demo_ARKit.csv` | A recorded CSV to bake into a facial clip without making a phone recording first. |

Follow [demo scene setup](Documentation/01-quick-start.md#open-the-demo-scene) or
[bake the included CSV](Documentation/03-animation.md#part-b--the-arkit-path).
The samples contain no bundled MetaHuman character data; import your own character.

## What MHBridge does

- **Import.** A DCC export from Unreal, or a raw `.dna` file, becomes one self-contained Unity
  asset: meshes for every LOD, HDRP materials, a single skeleton, an Avatar, and the runtime
  DNA the rig loads.
- **A live rig.** The `MHFace` and `MHBody` components evaluate RigLogic inside a native plugin
  every frame, over whatever the `Animator` poses.
- **Animation.** Facial takes and body clips from Unreal, and ARKit takes from Live Link Face,
  all arrive as ordinary `AnimationClip` assets. State machines, blend trees, avatar masks, and
  Timeline work as usual.
- **Face Board.** The facial control board, rebuilt as a Unity panel that drives the face in the
  Scene view without a clip or Play mode.
- **Clothing.** The Outfit Tool binds an outfit to the character's skeleton and removes the skin
  underneath, across the whole LOD stack.
- **Hair. (Experimental)** MetaHuman strand grooms with a live solver, head collision, and optional hair cards.


## Requirements

| What you need | Version |
|---|---|
| Unreal Engine | 5.8 or later, with a MetaHuman in the project |
| Unity | 2022.3 LTS minimum, reference version `2022.3.62f2`, tested through Unity 6.5 |
| Render pipeline | HDRP only: 14.0.12 on Unity 2022.3, 17.x on Unity 6 |
| Platform | Windows 64-bit, editor and Standalone Windows 64-bit builds |

Strand grooms additionally need `com.unity.demoteam.hair` and the Alembic package.
`com.unity.demoteam.digital-human` supplies skin attachment for facial grooms. The Hair Tool
installs these dependencies. Cards-only workflows need none of these packages.

## What this repository holds

This repository is the user guide for MHBridge, and the place to report problems with it. It
carries no source code: the package ships as a `.unitypackage` at [Asset Store](https://assetstore.unity.com/packages/slug/403996), and the same guide travels
inside it.

## Report a problem

Bug reports, questions, and documentation fixes go to
[Issues](https://github.com/seagullforge/MHBridge-Docs/issues). Before you file one, check
[Troubleshooting](Documentation/08-troubleshooting.md), which covers the failures
that come up most.

A report is quickest to answer when it names the Unity version, the render pipeline version, the
MHBridge version, whether the character came from a `.dcc` or a `.dna`, and the Console line the
import wrote. If possible, a sample `.dcc` or `.dna` file or a project link would be useful.

## Licensing

> [!NOTE]
> MHBridge ships no MetaHuman content: no DNA files, no meshes, no textures, and no character of
> any kind. You export characters with your own Epic account, under your own Unreal Engine
> license, and Epic's terms are what govern them. See
> [Licensing and legal](Documentation/10-licensing.md).

Epic Games, Unreal Engine, Fab, MetaHuman, RigLogic, and OpenRigLogic, and their associated
design logos, are trademarks or registered trademarks of Epic Games, Inc. MHBridge is not
affiliated with, endorsed by, or sponsored by Epic Games, Inc.
