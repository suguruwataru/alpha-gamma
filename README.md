# Alpha, Gamma, sRGB: What I've learnt about colors in computers

So I finally spent a whole day diving into the mess of how colors are presented
and processed in today's computers (wanted to do this since a long time ago),
and now I feel like things are less of a mystery to me. Kinda surprised that I
got this far with just a day (it's afternoon today and I started digging into
this at noon yesterday). Lots of reading and experiments have been done before
over the past few years, of course.

In the last 2 years, I encountered these classics

- [WHAT EVERY CODER SHOULD KNOW ABOUT GAMMA](https://blog.johnnovak.net/2016/09/21/what-every-coder-should-know-about-gamma/)
- [How software gets color wrong](https://bottosson.github.io/posts/colorwrong/)

And yesterday I was implementing "correct" alpha blending myself. I couldn't
get it right, at least couldn't get it to be what I thought was right. Then I
started doing searches and now I think I understand it, and I believe there are
things I can add to these great articles.

I'll write things down in the form of definitions, with later ones building on
top of previous ones. Note it may take several attempts to reach a final
definition, and they only stand within the context of this document. Other
people may have different definitions for the words.

## Color, first attempt

A color is a feeling. Some electro-magnetic waves (also known as light) causes
a typical human to get a feeling when they hit the human's eyes. For an
idealized human in an idealized circumstance (refered to as just "human"
below), each combination of light with certain intensities[^intensity] and
wavelengths corresponds to a certain feeling. Each of the feelings is called a
color.

[^intensity]: Meansuring light intensity is another huge and messy topic, and
I'm still not familiar about that, so I haven't decided what the physical
meaning of the "intensity" mentioned above should be. For now I assume it to be
[luminous intensity](https://en.wikipedia.org/wiki/Luminous_intensity).

## Colorspace

We store colors as numbers in computers, so that we can recreate them later.

We can store colors in different ways, as long as we know how to recreate them.

For instance, since colors correspond to combinations of light, when we want
store a color, we can measure the combination of the light that causes the
human to feel it, get an array of (wavelength, intensity). Later, we use
whatever magical device we have to emit light so that what reaches the human's
eyes still have the combination of wavelengths and intensity.

With this setup, given a color, we can get numbers. Given numbers, we can get
colors. We call this setup a colorspace, and if we store such numbers as data,
we say that the data is in this colorspace.

## RGB Colorspace, first attempt

The physiology of humans decides that, there are 3 special wavelengths, and by
combining light of the 3 wavelengths of different intensities, one can recreate
any color a typical human can perceive.

When pure light of the 3 wavelengths hit a typical human eye with an
appropriate intensity, they respectively causes the human to feel red, green,
blue, so let's just call the wavelengths R, G, B.

Thus, rather than array of (wavelength, intensity) tuples, we can store a color
as (R intensity, G intensity, B intensity) 3-tuples. Obviously, compared to
arrays of difference lengths and infinite number of possible wavelengths, these
3-tuples are much easier to work with.

We can measure light corresponding to a color and find corresponding values for
R intensity, G intensity, and B intensity, so that with some
red-blue-green-only light-emitting device (much simpler than full-spectrum
ones) we can reproduce the color. So we do have a colorspace here, and let's
say that the 3-tuples are in the RGB colorspace.

## Color

When our device emit light to recreate certain color, it's too much work to
ensure what hits the viewer's eyeballs are always the same, if that's possible
at all. Of course a close viewer and a far viewer see different things. Thus,
we don't want to deal with colors seen by viewers. We emit red, blue, green
light with certain
[luminance](https://en.wikipedia.org/wiki/Luminance)[^luminance], so that what
a viewer sees is roughly the light intensity combination we want.

[^luminance]: Again, not sure that I'm using the correct physical quantity
here.

Since we do not directly work with the viewer's feelings, we update [the
definition of color](#color-first-attempt). Now a color is just an abstract
idea that corresponds to a (R luminance, B luminance, G luminance) tuple.

## RGB colorspace

[The new definition of colors](#colors) already creates a setup of color-data
correspondence. With a color, we get a (R luminance, B luminance, G luminance)
tuple. With a (R luminance, B luminance, G luminance), we get a color, since a
color is just the idea that's associated to the tuple. Thus, this setup is a
colorspace, and we call it the RGB colorspace.

## Linear RGB colorspace

An RGB colorspace is still very abstract. A color is store as numbers, which
are luminance in a tuple. Of course luminance can be written as numbers, but
how exactly? What unit do we use? How exactly do we store the numbers as bytes?

We can give different answers to the questions. With each answer combination,
we create an instance of the RGB colorspace. If the instance's (R intensity, G
intensity, B intensity) values form a linear relationship with plain physical
measurements of intensity, we call it a linear RGB colorspace[^self], **but**,
there is a certain type of linear RGB colorspaces that we pay most attention
to. In such a colorspace, color data have the form (R value, G value, B value)
such that

- When a value is 0, it means the lowest relevant luminance
- When a value is 1, it means the highest relevant luminance

<a name="relevant"></a>
Where "relevant luminance" mean different things in different contexts:

- lowest/highest luminance that humans can perceive
- lowest/highest luminance that a device can emit light for
- etc.

Reference to such a linear RGB colorspace is so prevalent that "linear RGB
colorspace" mostly just means this kind of colorspace.

[^self]: And the RGB colorspace itself is of course also a linear RGB
colorspace.
  
## Gamma correction

Due to how they function, the optical components of a camera may give its
digital components colors in such a way:

  (data value) = luminance ^ (camera gamma)

The exponent is called "gamma" by convention.

This does not only happen in cameras, but also in many other optical/electrical
devices. Even the human eyes contribute to it due to different viewing
conditions.

Therefore, to make the computer monitor recreate the luminance stored in some
data, there needs to be a step:
  
  luminance = (data value) ^ gamma

Where gamma cancels all the other gamma applied. Such cancellation is called
gamma correction.

There's more to gamma correction. The PNG specification has a nice tutorial
about it: https://www.w3.org/TR/PNG-GammaAppendix.html

## Gamma

See [gamma correction](#Gamma correction)

## CRT Gamma

In the past, the CRT monitors were the primary contributors to the need of gamma
correction. When color data were fed to them (as voltage applied to their
electron guns), they generally worked like this:
  
  luminance = (data value) ^ (CRT gamma)

By convention (similar to how we deal with [linear RGB
colorspaces](#linear-rgb-colorspace)), luminance and data value are numerized
so that

- 1 represents the highest [relevant](#relevant) luminance or data value
- 0 represents the lowest [relevant](#relevant) luminance or data value

Within such convention, CRT gamma is generally 2.2-2.5.

To cancel their effect, gamma correction with a gamma of value 1/(CRT gamma) need
to be done.

See also [the gamma tutorial in the PNG spec](https://www.w3.org/TR/PNG-GammaAppendix.html).

## 2.2 Gamma

In [the PNG specification gamma
tutorial](https://www.w3.org/TR/PNG-GammaAppendix.html), it mentioned that

> The original NTSC video standard required cameras to have a transfer function
with a gamma of 1/2.2, or about 0.45.

This is for gamma correction of that time, primarily to cancel the CRT gamma.

In other words, gamma corrected color data from that time had their values
being:
  
    (data value) = luminance ^ 2.2

## Gamma colorspace

By our definition of a colorspace, the [2.2-gamma-corrected color
data](#22-gamma) of course are in their own [colorspace](#colorspace). I don't
think this is formalized any where, but conventionally it's often said that
they are in "the gamma colorspace".

## sRGB colorspace

sRGB colorspace[^srgb-draft] is a RGB colorspace that was developed for CRT
devices. It stores colors' red, blue and green values in a way so that
**approximately**

    (sRGB value) = luminance ^ 2.2

[^srgb-draft]: Draft: https://www.w3.org/Graphics/Color/sRGB.

Today, sRGB is important because lots of things assume color data to be in
sRGB if they don't have (often don't accept) exact info.

An image viewer will assume a PNG file to be in sRGB and try to make your
monitor emit light calculated from its data using a function given by sRGB.

If you render to a GPU texture that is shown on screen, the data put into it
will be assumed to be in sRGB format. Also your choice of texture format (like
whether its `rgba8uorm` or `rgba8unorm-srgb`, in the context of wgpu) does not
affect this. They only affect the data you read/write with the texture in your
shader. Generally you want to use a `-srgb` texture and write [linear
sRGB](#linear-srgb-colorspace) data to it.

etc. etc.

## sRGB and 2.2 Gamma

[The sRGB colorspace](#srgb-colorspace) is NOT [the gamma
colorspace](#gamma-colorspace). sRGB has different functions to convert to and
from the a linear RGB colorspace.

sRGB does explicitly approximates the gamma colorspace to be compatible with
color data that was gamma corrected for whatever correction needs at that time.
Therefore, applying a 2.2 or 1/2.2 gamma to some color data to convert to and
from sRGB is not entirely wrong, but is still not entirely correct either.
sRGB's own conversion functions are more complicated, so this is still a pretty
common shorthand/optimization, when one doesn't need to be *that* correct.

## Linear sRGB colorspace

sRGB actually didn't define how to calculate its values to and from luminance.
It's actually more complicated. Anyway, it does define itself against a linear
RGB colorspace. It also specifies that, this linear RGB colorspace has whitest
white to be the same as sRGB, blackest black same as sRGB, reddest red same as
sRGB, etc. This differs this linear RGB colorspace from other ones, and it is
called the linear sRGB colorspace.

## Alpha blending

Alpha blending is the technique of combining multiple colors. It is also called
the `over` operator in Porter-Duff compositing. The formula of producing the
`result` of color data in a **linear** RGB colorspace[^why-linear] `color 1`
`over` `color2` is:
  
    (result alpha) = (color 1 alpha) + (color 2 alpha) * (1 - color 1 alpha)
    (result R/G/B) = (
        (color 1 R/G/B) * (color 1 alpha) + (color 2 R/G/B) * (color 2 alpha) * (1 - (color 1 alpha))
    ) / (result alpha)

[^why-linear]: See [the Alpha section](#alpha) for why this has to be a linear
RGB colorspace.

## Alpha

Assume there is a shape, which is partially covered by an shape of a certain
color and partially covered by another shape of certain color (the later shapes
can cover each other, too), and we can only use one color to describe the
appearance of the whole shape, [the Porter-Duff `over`
operator](#alpha-blending) (or Porter-Duff operators in general) calculates the
color.

In such calculations, by convention, each of the covering shape's colors carry
an extra component that is called `alpha` (so they go from (R value, G value, B
value) 3-tuples to (R value, G value, B value, alpha)), which denotes the
portion of the area of the lowest shape it covers. In other words, `alpha` is a
ratio of areas.[^not-alpha]

Intuitively, the ratio of area linearly map to the ratio of light emitted as
part of all light emitted from the whole shape. With such a definition, alpha
blending must be designed to deal with physical light, and things like gamma
correction will only break it.

[^porter-duff]: See https://www.w3.org/TR/compositing-1/#advancedcompositing
[^not-alpha]: So no, `alpha` is NOT

- some data attached to colors without any inherent meanings.
- a presentation of opacity/transparency. Well maybe, but please also give the physical model of transparency that it matches if you suggest so.
- how much each color contributes to the final color. Well it is, but this description is not meaningful at all.

And no. Even if both color data are from the same colorspace, as long as it's
not a linear RGB colorspace, alpha blending them is incorrect. The results may
look "not bad", but they are incorrect, as suggested in variation 3 in
https://www.w3.org/TR/PNG-Decoders.html#D.Alpha-channel-processing.

## Alpha and transparency

alpha

## Alpha blending and sRGB or gamma colorspaces.

## Alpha blending implementations

## Perceptual RGB colorspace

The colors humans feel do not map simply to intensities. Human eyes are better
at seeing differences in intensities at lower intensities. At a lower
intensity, a human may see a slight different in intensity. At higher
intensity, not so much.

We define perceptual RGB colorspaces, where colors are stored as (R perceptual,
G perceptual, B perceptual), where values are linear to human perception, i. e.
a smallest noticable change at the lower value has the same numerical value as
a smallest noticable change at the higher value.

## 2.2 Gamma, sRGB and human perception

Color data, though conceptually mostly continuous, in the end, still need to be
converted to/from bytes that can only represent discrete numbers that can only
change in steps. After color data in the sRGB or gamma colorspaces are
converted to such discrete numbers, they get one property: a step change at a
lower value corresponds to a smaller change in the luminance, or more
precisely, the [linear RGB colorspace](#linear-rgb-colorspace). the smaller a
step in a value needs to be, the more bits are needed to store the value. This
property of sRGB and gamma colorspaces allow them to use a bigger step to
represent the smallest noticable change at both low luminance, compared to a
linear RGB colorspace. This make the colorspaces more bit-efficient.
