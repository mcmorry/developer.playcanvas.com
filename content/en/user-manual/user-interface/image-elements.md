---
title: Image Elements
template: usermanual-page.tmpl.html
position: 5
---

The image Element is used to display a texture or a solid color. Image Elements can be used as the basis for other more complex user interface components such as buttons. 

## Tinting

Each image Element has a color property. If no texture is applied, the image element will simply be a colored rectangle. If a texture is applied, this color will be used to tint the texture. This let's you create color variations using only a single greyscale texture.

![Tinting][1]

## Transparency

As with the color property, the opacity property can be used to set the transparency level of an Element. 

![Transparent][2]

## Advanced Materials

If you can't achieve the results you are looking for using the texture, color and opacity properties. You can assign your own material to an image element using the material property. For correct rendering you should *disable Depth Write* on any material added to an image Element. 

Note: Lighting will not function as expected for Screen Space elements. You will probably want to disable lighting and shadows for any material that is used in screen space.

[1]: /images/user-manual/user-interface/image-element/image-tinted.png
[2]: /images/user-manual/user-interface/image-element/image-transparent.png