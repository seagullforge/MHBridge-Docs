# Licensing and legal

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

MHBridge is tooling. The characters it imports are Epic's MetaHuman content, licensed to you by
Epic and not by us, and two questions follow from that: what Epic's terms let you do with a
character once it stands in a Unity scene, and what the package itself is made of. This page
answers both, and links the documents that decide them.

> [!WARNING]
> **This package ships no MetaHuman content.** You export characters with your own Epic account,
> under your own Unreal Engine license, and complying with Epic's terms for that content is yours
> to do. Nothing on this page or in this package changes those terms.

## Why a MetaHuman can live in a Unity project

Until Unreal Engine 5.6, MetaHuman carried a license of its own. It no longer does. Epic's
[MetaHuman licensing page](https://www.metahuman.com/license) states that from Unreal Engine 5.6,
your use of MetaHuman is included under the terms of your existing Unreal Engine license, and it
answers the engine question directly: the Unreal Engine EULA "provides for MetaHuman characters and
animation to be used across all engines and creative software, subject to applicable terms".

Royalties follow the same reasoning. Characters and animation count as Non-Engine Products, and
Section 4(a) of the [Unreal Engine EULA](https://www.unrealengine.com/eula/unreal) says that such
asset files "remain Non-Engine Products even if included in Products that use or rely on other
video game engines". A game you build in Unity owes Epic no royalty on the character in it.

What can still cost money is the seat subscription rather than the character. Unreal Engine is free
below $1,000,000 USD in gross revenue over the last 12 months. Above that threshold, rendering
MetaHuman characters and animation outside Unreal Engine needs an Unreal Engine seat license, and
the current terms are on the [Unreal Engine licensing page](https://www.unrealengine.com/license).

## Which terms apply to your character

Where a character came from decides which document to read:

- **You created it with the MetaHuman tooling in Unreal Engine.** Your Unreal Engine license covers
  it. This is the path the rest of this documentation describes.
- **You bought or downloaded it on Fab.** The seller licenses it to you on Fab's terms, which are
  narrower than the Unreal Engine license in places. See
  [Outfits and other content from Fab](#outfits-and-other-content-from-fab).
- **It reached you as Epic content some other way.** Content downloaded under the
  [Epic Content License Agreement](https://www.unrealengine.com/eula/content) keeps that agreement's
  terms, and its MetaHuman Content Addendum designates MetaHuman Content acquired that way as
  UE-Only Content.

Importing changes none of this. A `.dcc` asset in your Unity project is the same character under
the same terms it arrived with, and so is every mesh, texture, and DNA sub-asset inside it.

## What you can do with an imported character

| What you want to do | Where it stands |
|---|---|
| Use the character in a project you build in Unity, commercial or not | Covered by your Unreal Engine license. |
| Ship it inside the built player, on any platform Unity targets | Yes. It travels as part of your product. |
| Publish screenshots, trailers, and rendered video | Yes. Rendered output is a Non-Engine Product and carries no royalty. |
| Share the project with the people building it with you | Yes, for the team and the contractors working on that project. |
| Send the `.dcc` asset, the DNA, or the textures to anyone else | This is distribution of Epic's content in source form. Read the terms the character came with first. |
| Sell the character itself | Epic allows it for assets packaged in Unreal Engine, on Fab or on third-party marketplaces. |
| Train or test an AI or ML model on the character or its animation curves | No. Building or enhancing a database with them is forbidden too. Using the character in a product that itself uses AI is not. |

The line that decides most of that table is the one between using the character and handing it
over. Rendering it — in a build, in a trailer, on a store page — is use. Passing on the files it is
made of is distribution, and distribution is what Epic's terms are mostly about. The restriction in
the last row is worth reading in full on the licensing page, because it reaches rendered output as
well when that output is created to replicate what MetaHuman does.

> [!WARNING]
> **Selling a character is a different question from shipping a game.** Epic's answer is that
> MetaHuman assets "can be packaged in Unreal Engine for sale on the Fab marketplace, as well as on
> third-party marketplaces". That answer describes packaging in Unreal Engine, and what MHBridge
> produces is a Unity asset, which is not that path. Shipping a character inside your own compiled
> game raises none of this.

## Outfits and other content from Fab

An asset from Fab arrives under the [Fab EULA](https://www.fab.com/eula), an agreement between you
and the seller. Its Standard License grants you private use, reproduction, and modification in
Section 3(a); Section 4(b) lets you publish rendered video and images made with the asset; and
Section 4(c) lets you distribute a project that incorporates it, as long as end users receive the
asset in object code inside your project and cannot extract it. Nothing in that is tied to a
particular engine, as Fab's own summary of the license states: you may "use the assets with any
compatible tools (usage is not limited to Unreal Engine)". Some listings are offered under an open
source or Creative Commons license instead, and Section 2(c) makes that license the one that
governs.

Two limits decide most of the remaining questions:

- **The asset stays inside your project.** Section 5(a) allows sharing it on its own only with
  collaborators developing the project with you, and obliges them to delete it once the work ends.
  Section 6(b)(ii) rules out selling or transferring it on a standalone basis, and Section
  6(b)(iii) rules out letting a third party incorporate it into a product of their own.
- **The tier decides whether you can export it at all.** Under Section 2(d), the Personal and
  Professional tiers give you the source asset where the listing offers one, while Personal -
  Reference Only gives you a reference asset. MHBridge needs an outfit you can open and export in
  Unreal, so a reference-only license is not enough. Eligibility for the Personal tiers stops at
  $100,000 USD of gross revenue from digital content work over the last 12 months, measured when
  you buy (Section 2(a)).

An asset tagged **NoAI** in the marketplace carries one more restriction, in Section 6(b)(vii): it
may not go into datasets for generative AI programs, into their development, or into their training
inputs.

An outfit is therefore yours to dress a Unity character in and to ship inside your game. It is not
yours to pass on as an outfit. For the workflow, see [Clothing](07-clothing.md).

## What this package contains

MHBridge is Editor tooling, runtime components, HDRP shaders and materials, and one native plugin,
`MHBridgeRig.dll`. It is distributed under the
[Unity Asset Store EULA](https://unity.com/legal/as-terms). It contains no MetaHuman character
data: no DNA files, no meshes, no textures, no character of any kind. Every character in a project
that uses MHBridge arrived there through an export its owner made.

The package also includes a scene with prepared lighting, an Animator Controller, face and body
animation samples, and a raw ARKit CSV in `Assets/MHBridge/Demo`. The scene has an empty
`YOUR METAHUMAN` placeholder for your own imported character. See [Included samples](../README.md#included-samples)
for the inventory and usage routes.

The screenshots and video in this documentation show characters
exported from an Epic account of ours, under the same distinction that governs your use of yours:
rendered output travels, and the files it was rendered from stay in the project that made them.

## Third-party components

| Component | License | How it reaches you |
|---|---|---|
| OpenRigLogic, by Epic Games | [MIT](https://github.com/EpicGames/OpenRigLogic) | Compiled into `MHBridgeRig.dll` and shipped with the package. The full text is in the `Third Party Notices.md` file, and beside the binary in `Plugins/x86_64/LICENSE-OpenRigLogic.txt`. |
| Unity HDRP sample shader components | Unity Companion License | Included or modified hair shader graphs, eye shading, hair subgraphs, and mask textures, listed in `Third Party Notices.md`. The eye graph is `Assets/MHBridge/Shaders/MHEye_Graph.shadergraph`. |
| Unity Shader Graph Depth Fade template | Unity Companion License | `Assets/MHBridge/Shaders/MHDepthFade.shadersubgraph`, adapted from the Shader Graph template and included for the eye graphs. |
| `com.unity.demoteam.hair` and `com.unity.demoteam.digital-human`, by Unity Technologies | [Unity Companion License](https://unity.com/legal/licenses/unity-companion-license) | Optional, and never bundled. You install them yourself from git URLs when you use hair. |

## Trademarks

Epic Games, Unreal Engine, Fab, MetaHuman, RigLogic, and OpenRigLogic, and their associated design
logos, are trademarks or registered trademarks of Epic Games, Inc. MHBridge is not affiliated with,
endorsed by, or sponsored by Epic Games, Inc. This documentation refers to those names only to
describe compatibility and the origin of the bundled libraries.

## Not legal advice

This page summarizes documents that Epic and Unity revise from time to time, and the originals it
links to are what binds you. If a project needs a definitive answer, ask a lawyer, or ask Epic:
the MetaHuman licensing page carries a contact route for custom licensing terms.
