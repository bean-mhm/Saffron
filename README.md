<div align="center">
<a href='https://ko-fi.com/E1E81LFRKY' target='_blank'><img height='42' style='border:0px;height:42px;' src='https://storage.ko-fi.com/cdn/kofi2.png?v=6' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>
</div>

# 🪻 Saffron

![Saffron's Compositor Nodes](./images/saffron-nodes.webp)

**Saffron** is a large collection of compositor nodes for [Blender](https://blender.org/) providing many great features in the following categories:
- ### Color Grading & Effects
- ### Black & White
- ### Masking
- ### Advanced Color Processing: Gamut Compression, Highlight Reconstruction, HSV, HSL, and more.
- ### Color Space Conversions: Linear, Displays, ACES, Oklab.
- ### View Transforms (Tone Mapping): [flim](https://github.com/bean-mhm/flim), Khronos PBR Neutral, ACES, and more.

Here's some eye candy to show you how Saffron can transform images. Whether you
want a vintage look with film emulation or cinematic color grading, Saffron's
got you covered!

[TODO IMAGES]

# Color Grading & Effects

![Nodes](./images/nodes-color-grading-effects.webp)

## Node: Camera Noise Reduction

This node uses Blender's built-in Denoise node at the high quality preset but when mixing, it gives more weight to darker pixels since, in digital cameras, darker pixels contain a lot more noise than the midtones, and highlights are usually very clean.

## Node: Saffron Grade

This node is all about color grading. It uses the [Oklab](https://bottosson.github.io/posts/oklab/) color space as its processing space to provide a perceptually uniform and intuitive parameter space. It uses custom-made algorithms to alter the tone and color of
the image.

## Node: Color Zone

With this node you can isolate a certain hue range (for example, only the greens
to select the grass) and adjust the color tone of that part.

## Node: Saffron Effects

This node provides artistic image editing effects including
**Texture & Clarity**, **Vignette**, **Halation**, and **Film Grain**. It too
uses [Oklab](https://bottosson.github.io/posts/oklab/) as its processing space
and is made of custom-made algorithms.

# Black & White

![Nodes](./images/nodes-bw.webp)

These nodes let you convert an RGB color to a grayscale value using different
methods.

# Masking

![Nodes](./images/nodes-masking.webp)

Masking is a technique used to blend two images or values using different
blending amounts at different positions. For example, with a
**Radial Gradient Mask**, you can create a mask that covers the face of the
subject and only brighten that part of the image.

With Saffron's masking nodes you can create and combine masks using different
methods.

# Advanced

![Nodes](./images/nodes-advanced.webp)

This is a set of node groups for slightly more advanced color processing
techniques. Feel free to experiment with them.

# Color Space Conversions

![Nodes](./images/nodes-color-space-convs.webp)

These nodes let you convert between different color spaces. A more detailed
explanation is provided below.

# [IMPORTANT] Color Management

> [!IMPORTANT]
> Humans are naturally lazy to read long pieces of text (including myself), but
> if you want to use Saffron without broken results, you
> need to read and understand the following instructions.

In order to use Saffron, we need to disable Blender's color management system
because Saffron has its own. To do this,

1. Go to the **Color Management** tab in **Render Properties** and set
**View Transform** to *None* or *Raw*.

[TODO SCREENSHOT]

2. When loading external images with **Image** nodes in the compositor, set the
**Color Space** property to *Non-Color* or *Generic Data*. If you already have
**Image** nodes, change them too.

[TODO SCREENSHOT]

3. When exporting images, in the save dialog window, make sure to set
**Color Management** to *Override* and **Color Space** to *Non-Color* or
*Generic Data*.

Since we've disabled Blender's color management, we need to do it ourselves.
This has three parts:
1. Loading images
2. The working color space
3. Display/View transforms

To put it simply:

1. When you load an image from a file or a scene render, convert it from its
color space to the working color space.

2. Do your image processing in the working color space.

3. Optionally apply a **View Transform** (so-called "Tone Mapping") to make sure
your colors are not exceeding 100% brightness level which introduces
overexposed/clipped areas in bright spots. A good option is
[flim](https://github.com/bean-mhm/flim) which is a filmic color transform I've
made. Saffron has an implementation for flim in the
**Color Space Conversions: View Transforms** section.

> [!NOTE]
> View transforms don't modify the color space, they just make your colors reach
> 100% in a smoother and more visually pleasing way. The output is still in the
> working color space.

4. Finally, convert from the working space to a display color space (e.g. sRGB, Display P3, etc.) that
matches your display device. If you're unsure, look it up based on your display
device.

## Example

Here's an example scenario to help you wrap your head around color management.
Let's say our working space is **Linear BT.709 I-D65** (also called Linear sRGB
or Linear Rec.709), and we load an image named `cat.jpeg`.


When working with a PNG or JPEG image, there's a 99.999% chance it's in the sRGB
color space. Linear image formats like OpenEXR on the other hand, usually use
**Linear BT.709** but they might also use **ACES2065-1**, **Linear DCI-P3**,
or **Linear BT.2020**, all of which are supported by Saffron.

> [!NOTE]
> If you're advanced enough to use OpenEXR, you will almost surely know what color space your image is in. If not, load the image again and pay attention to the **Color Space** property of the **Image** node because Blender will try to guess the color space which is correct most of the time. Take note of the color space, then set it to *Non-Color* and use Saffron's color space conversion nodes to convert from that color space to your working space.

The next step is to convert that image from its color space (sRGB) to our
working space (Linear BT.709), so we use Saffron's
**sRGB -> Linear BT.709 I-D65** node.

Then, we apply a blur effect in the wokring space.

Finally, we need to convert to our display's color space. For example, if we
were using an sRGB display device, we would use the
**Linear BT.709 I-D65 -> sRGB** node.

## The Reference Space

If we made a node to convert between every combination of color spaces, we would
have hundreds of nodes! To work around this, we choose a
**reference color space** and implement conversions to and from that space.
For example, Saffron has **Ref -> X** and **Ref <- X** (notice the arrow
directions).

With this system, if we want to convert from a color space A to
a color space B, we simply convert from A to the reference and then from the
reference to B. Saffron uses **Linear CIE-XYZ I-D65** as its reference color
space.

Note that **View Transforms** are one-sided and they expect the input to be in a
linear RGB color space. Also, since **Display Transforms** are typically based
on a single linear color space (you wouldn't normally convert from Linear BT.709
to Display P3, for example), we don't use the reference space for them and
simply have **Linear X -> Display X** and vice versa.

## They Tell You What They Want

Pay attention to the input sockets in nodes. Some inputs are named **In (RGB)**,
this means they expect the input to be in a linear RGB color space. **In (Ref)**
means the input must be in the reference color space, and so forth. The same
is true for output sockets (e.g. **Out (Oklab)** means the output is in Oklab).

# Contribution

If you want to improve Saffron by adding new features and nodes, definitely feel free to
make a pull request! This is open-source after all.

# Donations

I made Saffron free and open-source because knowledge shouldn't be behind a
pay wall and I understand that not everyone can afford to pay tens of dollars
for a simple node pack. However, if you think the project is deserving, feel free to support me at [Ko-fi](https://ko-fi.com/bean_mhm). Your
support is hugely appreciated!
