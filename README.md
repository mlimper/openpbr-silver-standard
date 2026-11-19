# OpenPBR Silver Standard

A profile for MaterialX to facilitate adoption of OpenPBR content, and to increase its compatibility across applications, by defining a portable subset of MaterialX nodes for common PBR materials.

> [!IMPORTANT]
> **This is an independent, community-driven effort.** It is **not** affiliated with, approved by, or endorsed by the Academy Software Foundation (ASWF), the Linux Foundation, Adobe, Autodesk, or any other company or organization involved in the development of OpenPBR or MaterialX. "OpenPBR" and "MaterialX" are used here only to refer to those specifications; all trademarks belong to their respective owners. See [Disclaimer](#disclaimer).


## Introduction and Goals
This specification defines a MaterialX profile (in other words, a set of MaterialX node types) around the OpenPBR UberShader and adjacent node types, with the following goals:

* Provide a subset of MaterialX nodes sufficient to describe common, non-procedural PBR content, with the full shading capabilities of OpenPBR
* Help applications (for example, renderers) to support a common subset of such OpenPBR related nodes, therefore increasing the portability of OpenPBR content across the industry
* Offer a simple starting point for new implementations (such as material creation tools or renderers) that are seeking to adopt MaterialX and OpenPBR

This profile is **not** a replacement for, or a fork of, MaterialX or OpenPBR. Every document that follows it is a valid MaterialX document, and it uses the standard node definitions unchanged. It only narrows down *which* nodes are used, so that supporting them is a realistic first milestone for an implementation.

### A Standard Meant to Retire
The ultimate goal of the Silver Standard is to **become unnecessary**. It exists to lower the entry barrier while full MaterialX support is still uncommon across tools and renderers. As adoption grows and implementations support the complete MaterialX standard library, this profile should be deprecated and eventually retired. Its success is measured by how soon nobody needs it anymore.


## Node Types

A Silver Standard document uses **only** the node types below, from MaterialX 1.39 or later. Everything else in the MaterialX standard library (procedural noise, math, blending, conditionals, other shading models, and so on) is out of scope.

| Node | Output types | Role |
|---|---|---|
| [`surfacematerial`](https://github.com/AcademySoftwareFoundation/MaterialX/blob/main/documents/Specification/MaterialX.Specification.md) | `material` | The material. Its `surfaceshader` input connects to one `open_pbr_surface`. |
| [`open_pbr_surface`](https://github.com/AcademySoftwareFoundation/OpenPBR) | `surfaceshader` | The only shading model. Any of its inputs may be set as a constant value or connected to a texture chain built from the nodes below. |
| `image` | `float`, `color3`, `vector3` | Texture lookup. `color3` for color data (`base_color`, `emission_color`, ...), `float` for scalar data (roughness, metalness, ...), `vector3` for normal maps. A `float` image reads the first channel of the file. |
| `normalmap` | `vector3` | Decodes a tangent-space normal map read by an `image` node. Its output connects to `geometry_normal` or `geometry_coat_normal`. `scale` sets the normal strength. |
| `place2d` | `vector2` | UV transform (tiling, offset, rotation) feeding an `image` node's `texcoord`. |
| `texcoord` | `vector2` | Selects a UV set by `index`. Only needed for UV sets other than the first one, since `image` and `place2d` read UV set 0 by default. |

### Texture Guidelines

* **Color spaces.** Color textures (`image` of type `color3`) use `colorspace="srgb_texture"` (or `lin_rec709` when linear). Data textures (roughness, metalness, normals, ...) carry no `colorspace`, because MaterialX applies color transforms to color types only.
* **Address modes.** `uaddressmode` and `vaddressmode` may be `periodic` (default), `clamp`, `mirror` or `constant`.
* **UV transforms.** Use a single `place2d` with its default `pivot` (0, 0) and `operationorder` (0, SRT). Note that `place2d` *divides* coordinates by `scale`: a `scale` of `0.5, 0.5` tiles the texture twice.
* **Normal maps.** Always connect normal map images through a `normalmap` node. Do not connect a bare `image` straight to `geometry_normal`.
* **File paths.** Use paths relative to the `.mtlx` document (for example `textures/albedo.png`).


## A Simple Example

The smallest Silver Standard material is a single `open_pbr_surface` with constant values, wrapped in a `surfacematerial`. Inputs that are not authored keep their OpenPBR defaults.

```xml
<?xml version="1.0"?>
<materialx version="1.39">
  <open_pbr_surface name="Gold_shader" type="surfaceshader">
    <input name="base_color" type="color3" value="0.944, 0.776, 0.373" />
    <input name="base_metalness" type="float" value="1.0" />
    <input name="specular_roughness" type="float" value="0.25" />
  </open_pbr_surface>

  <surfacematerial name="Gold" type="material">
    <input name="surfaceshader" type="surfaceshader" nodename="Gold_shader" />
  </surfacematerial>
</materialx>
```

### A Textured Example

A typical textured asset: an sRGB base color map, single-channel roughness and metalness maps, and a tangent-space normal map, all tiled twice through one shared `place2d`. This example uses every node type in the profile except `texcoord`, which is not needed when all maps use the first UV set.

```xml
  <!-- TODO -->
```

TODO (screenshots of the examples above in different renderers)


## Further Example Files

TODO (refer to directory that has more examples)


## Automated Validation

TODO - do we want to create a simple validator?


## Disclaimer

The OpenPBR Silver Standard is an independent initiative. It has **no relation** to, and is **not approved, endorsed, sponsored or maintained** by the Academy Software Foundation (ASWF), the Linux Foundation, Adobe, Autodesk, or any other company or organization involved in the development of OpenPBR or MaterialX.

It does not speak for those projects, and nothing here should be read as an official position of theirs. The authoritative specifications are the [OpenPBR specification](https://academysoftwarefoundation.github.io/OpenPBR/) and the [MaterialX specification](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/documents/Specification). Where this document and those specifications disagree, they take precedence.
