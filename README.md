# Sprite Detection

## Sprite

A [**sprite**](<https://en.wikipedia.org/wiki/Sprite_(computer_graphics)>) is a small [**raster graphic**](https://en.wikipedia.org/wiki/Raster_graphics) (a **bitmap**) that represents an object such as a character, a vehicle, a projectile, etc.

![Metal Slug ](metal_slug_sprite_large.png)

Sprites are a popular way to create large, complex scenes as you can manipulate each sprite separately from the rest of the scene. This allows for greater control over how the scene is rendered, as well as over how the players can interact with the scene.

Sprites are mainly used in 2D video games, such as [Shoot'em up](https://en.wikipedia.org/wiki/Shoot_%27em_up) in which the hero combats a large number of enemies by shooting at them while dodging their fire:

| [Cannon Fodder](<https://en.wikipedia.org/wiki/Cannon_Fodder_(video_game)>) | [Commando](<https://en.wikipedia.org/wiki/Commando_(video_game)>) | [Metal Slug](https://en.wikipedia.org/wiki/Metal_Slug) |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------ |
| ![Cannon Fodder](2d_video_game_cannon_fodder.jpg)                           | ![Commando](2d_video_game_commando.jpg)                           | ![Metal Slug](2d_video_game_metal_slug.png)            |

## Sprite Animation

Several sprites, representing a same object, can be rendered in quick succession to give the illusion of movement of this object (**animation**).

For example, the animation of the hero Marco Rossi (Metal Slug), running with a gun, is composed of 9 sprites:

![Metal Slug Sprite Running with a Gun](metal_slug_sprites_running_with_gun.png)

## Sprite Sheet

It is not uncommon for games to have tens to hundreds of sprites. Loading each of these as an individual image would consume a lot of memory and processing power. To help manage sprites and avoid using so many images, many games use **sprite sheets** (also known as **image sprites**).

A sprite sheet consists of multiple sprites in one image. In other words, sprite sheets pack multiple sprites into a single picture. Using sprite sheet, video game developers create sprite sheet animation representing one or several animation sequences while only loading a single file:

![Metal Slug Sprites](metal_slug_sprite_sheet_large.png)

## Sprite Bounding Box

A frame (**bounding box**) can be used to delimit the sprite in the sprite sheet. This bounding box is defined with two 2D points `top_left` and the `bottom_right`, which their respective coordinates `x` and `y` are relative to the top-left corner of the sprite sheet's image.

For example:

![Shape Bounding Boxes](metal_slug_sprite_detection_bounding_boxes.png)

## Sprite Mask

The mask of a sprite defines the 2D shape of the sprite. For example, the sprite sheet [`metal_slug_sprite_standing_stance.png`](metal_slug_sprite_standing_stance_large.png) contains the 3 following sprites:

![Metal Slug Standing Stance](metal_slug_sprite_standing_stance_large.png)

The masks of these sprites are respectively:

![](metal_slug_sprite_detection_coloring.png)

## Optimized Sprite Sheet

Sprites could be evenly placed in the sprite sheet according to their bounding box. The disadvantage is that this method of placement wastes a lot of memory because of all the additional transparency of each sprite, especially when sprites have different width and height.

The developers are aware of the wasted memory and started to optimize the space in the sprite sheets. Sprites are placed close to each others, depending on their shape. The bounding box of a sprite may intersect the bounding box of another sprite, but the shape of this sprite is always separated from the shape of any other sprites with at least 1 transparent pixel.

![Optimized Sprite Sheet](optimized_sprite_sheet.png)

This space optimization is even more efficient when some big sprites have concave shape (i.e., shape that curves inward):

| Scene with 2 Sprites    | Sprite Sheet                                 | Sprite Masks                                      |
| ----------------------- | -------------------------------------------- | ------------------------------------------------- |
| ![Islands](islands.png) | ![Islands Sprite Sheet](islands_sprites.png) | ![Islands Sprite Masks](islands_sprite_masks.png) |

# Waypoint 1: Find the Most Common Color in an Image

Write a function `find_most_common_color` that takes an argument `image` (a [`Image`](https://pillow.readthedocs.io/en/stable/reference/Image.html) object) and that returns the pixel color that is the most used in this image.

The data type of the value returned depends on the [image's mode](https://pillow.readthedocs.io/en/stable/handbook/concepts.html#concept-modes):

- an integer if the mode is grayscale;
- a tuple `(red, green, blue)` of integers if the mode is `RGB`;
- a tuple `(red, green, blue, alpha)` of integers if the mode is `RGBA`.

For example:

```python
>>> from PIL import Image
# JPEG image
>>> image = Image.open('islands.jpg')
>>> image.mode
'RGB'
>>> find_most_common_color(image)
(0, 221, 204)
# PNG image
>>> image = Image.open('metal_slug_sprite_standing_stance_large.png')
>>> image.mode
'RGBA'
>>> find_most_common_color(image)
(0, 0, 0, 0)
# Grayscale image
>>> image = image.convert('L')
>>> image.mode
'L'
>>> find_most_common_color(image)
152
```

<em>Note 1: Python is an interpreted, high-level, general-purpose programming language. Pure Python code is not great at processing large amount of data. However, Python comes with a large number of libraries written in [`C`](<https://en.wikipedia.org/wiki/C_(programming_language)>) that run very fast: [`Pillow`](https://en.wikipedia.org/wiki/Python_Imaging_Library) for image processing, [`NumPy`](https://en.wikipedia.org/wiki/NumPy) for large multi-dimensional arrays and matrices, etc. You SHOULD intensively use these libraries that extend Python and that allow your Python application to be fast.</em>

_Note 2: You SHOULD [measure the execution time](https://docs.python.org/3.7/library/timeit.html) of your code to check whether it's reasonably fast or if there is space for improvement. For example:_

```python
>>> import timeit
>>> from PIL import Image
>>> image = Image.open('islands.jpg')
>>> timeit.timeit(stmt=lambda: find_most_common_color(image), number=1)
1.6871624910000023
```

# Waypoint 2: Write a class `Sprite`

Write a class `Sprite` which constructor takes 5 arguments `label`, `x1`, `y1`, `x2`, and `y2` (strictly positive integers). These arguments are used to initialize private attributes of the class `Sprite`.

Add the read-only property `label` to the class `Sprite` that returns the label of the sprite.

Add the two read-only properties `top_left` and `bottom_right` to the class `Sprite` that correspond to the coordinates `(x1, y1)` (a tuple) of the top-left corner, respectively the coordinates `(x2, y2)` (a tuple) of the right-most corner.

For example:

```python
>>> sprite = Sprite(1, 12, 23, 145, 208)
>>> sprite.label
1
>>> sprite.top_left
(12, 23)
>>> sprite.bottom_right
(145, 208)
```

The constructor of the class `Sprite` raises an exception `ValueError` if one or more arguments `label`, `x1`, `y1`, `x2`, and `y2` is not positive integer, or if the arguments `x2` and `y2` is not equal or greater respectively than `x1` and `y1`.

For examples:

```python
>>> sprite = Sprite(-1, 0, 0, 0)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<input>", line 11, in __init__
ValueError: Invalid coordinates
>>> sprite = Sprite("Hello", 0, 0, 0)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<input>", line 13, in __init__
ValueError: Invalid coordinates
>>> sprite = Sprite(1, 0, 0, 0)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<input>", line 13, in __init__
ValueError: Invalid coordinates
```

Add the two properties `width` and `height` that returns respectively the number of pixels horizontally and vertically of this sprite.

For example:

```python
>>> sprite = Sprite(1, 12, 23, 145, 208)
>>> sprite.width
134
>>> sprite.height
186
```

# Waypoint 3: Find Sprites in an Image

We would like to identify all the sprites packed in a single picture.

You need to implement an [image segmentation algorithm](./b5034b2ce2bdf21e09d3915207d7b824ceb4.pdf) that considers the color of each pixel of a specified picture to determine whether this sprite belongs to a sprite or not:

- pixels which color corresponds to the background color of the picture are considered as transparent, i.e., they don't belong to a sprite;

- pixels which color is different from the background color of the picture are considered as solid, i.e., they belong to a sprite.

Sprites correspond to smaller images composed of [connected pixels](https://en.wikipedia.org/wiki/Pixel_connectivity), meaning that each pixel of a sprite is adjacent to at least one of its direct neighbor pixels ([8-neighborhood connectivity method](4d991f5902c84c2181c6c573661abdc228b1.pdf)):

|                                                   |                                                                                         |
| ------------------------------------------------- | --------------------------------------------------------------------------------------- |
| ![8 Connectivity Pixel](8-connectivity-pixel.png) | ![8 8-neighbor-relative-pixels-coordinates](8-neighbor-relative-pixels-coordinates.png) |

Your image segmentation algorithm needs to isolate each sprite, producing bounding boxes with correct sizes and identifying isolated parts of a sprite as an included segment.

Write a function `find_sprites` that takes an argument `image` (an [`Image`](https://pillow.readthedocs.io/en/stable/reference/Image.html) object).

This function accepts an optional argument `transparent_color` (an integer if the image format is grayscale, or a tuple `(red, green, blue)` if the image format is `RGB`) that identifies the transparent color of the image. The function ignores any pixels of the image with this color.

If this argument `transparent_color` is not passed, the function determines the transparent color of the image as follows:

1. The image, such as a PNG file, has an [alpha channel](<https://en.wikipedia.org/wiki/Transparency_(graphic)>): the function ignores all the pixels of the image which alpha component is `255`;

2. The image has no alpha channel: the function identifies the most common color of the image as the transparent color (cf. our function `find_most_common_color`).

The function returns a tuple `(sprites, labels_matrix)` where:

- `sprites`: A collection of key-value pairs (a dictionary) where each key-value pair maps the key (the label of a sprite) to its associated value (a `Sprite` object);

- `label_matrix`: A 2D array of integers of equal dimension (width and height) as the original image where the sprites are packed in. The `label_matrix` array maps each pixel of the image passed to the function to the label of the sprite this pixel corresponds to, or `0` if this pixel doesn't belong to a sprite (e.g., transparent color).

_Note: The sprite labels can be whatever unique strictly positive integers, with no particular order. The index of a sprite has no particular relationship with the position of the sprite in the image._

**WARNING: You are NOT ALLOWED to use any computer vision libraries (e.g. [OpenCV](https://opencv.org/) and others), but image processing (e.g., [Python Image Library](https://python-pillow.org/)) and scientific computing (e.g, [Numpy](https://numpy.org/)) libraries!**

For example, let's consider the image file [sprite_example.png](metal_slug_single_sprite.png) zoomed-in hereafter:

![](metal_slug_single_sprite_large.png)

```python
>>> from PIL import Image
>>> image = Image.open('metal_slug_single_sprite.png')
>>> sprites, label_matrix = find_sprites(image, transparent_color=(255, 255, 255))
>>> len(sprites)
1
>>> for label, sprite in sprites.items():
...     print(f"Sprite ({label}): [{sprite.top_left}, {sprite.bottom_right}] {sprite.width}x{sprite.height}")
Sprite (1): [(0, 0), (29, 37)] 30x38
>>> import pprint
>>> pprint.pprint(label_matrix, width=120)
[[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0],
 [0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0],
 [0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0],
 [0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0],
 [0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [0,0,1,1,1,1,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0],
 [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0],
 [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0],
 [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0],
 [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0],
 [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0],
 [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0],
 [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0],
 [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0],
 [0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0],
 [0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0],
 [0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0],
 [0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
 [0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,1,1,1,1,1,1,1],
 [0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,1,1,1,1,0],
 [0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,1,1,0,0],
 [0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0],
 [0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0],
 [0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0],
 [0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [0,0,1,1,1,1,1,1,1,1,0,0,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0],
 [0,1,1,1,1,1,1,1,0,0,0,0,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0],
 [0,1,1,1,1,1,1,0,0,0,0,0,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0],
 [1,1,1,1,1,1,1,1,0,0,0,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0],
 [1,1,1,1,1,1,1,1,1,0,0,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0],
 [1,1,1,1,1,1,1,1,1,0,0,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0]]
```

Other example with the following image:

![](optimized_sprite_sheet.png.png)

```python
>>> from PIL import Image
>>> image = Image.open('optimized_sprite_sheet.png.png')
>>> sprites, label_matrix = find_sprites(image)
>>> len(sprites)
22
>>> for label, sprite in sprites.items():
... print(f"Sprite ({label}): [{sprite.top_left}, {sprite.bottom_right}] {sprite.width}x{sprite.height}")
Sprite (25): [(383, 1), (455, 102)] 73x102
Sprite (43): [(9, 2), (97, 122)] 89x121
Sprite (26): [(110, 4), (195, 123)] 86x120
Sprite (46): [(207, 4), (291, 123)] 85x120
Sprite (16): [(305, 8), (379, 123)] 75x116
Sprite (53): [(349, 125), (431, 229)] 83x105
Sprite (61): [(285, 126), (330, 181)] 46x56
Sprite (100): [(1, 129), (101, 237)] 101x109
Sprite (106): [(106, 129), (193, 249)] 88x121
Sprite (93): [(183, 137), (278, 241)] 96x105
Sprite (95): [(268, 173), (355, 261)] 88x89
Sprite (178): [(6, 244), (101, 348)] 96x105
Sprite (185): [(145, 247), (245, 355)] 101x109
Sprite (141): [(343, 257), (417, 372)] 75x116
Sprite (169): [(102, 262), (142, 303)] 41x42
Sprite (188): [(249, 267), (344, 373)] 96x107
Sprite (192): [(412, 337), (448, 372)] 37x36
Sprite (256): [(89, 353), (184, 459)] 96x107
Sprite (234): [(11, 356), (104, 461)] 94x106
Sprite (207): [(188, 358), (281, 463)] 94x106
Sprite (229): [(384, 374), (456, 475)] 73x102
Sprite (248): [(286, 378), (368, 482)] 83x105
```

# Waypoint 4: Draw Sprite Label Bounding Boxes

Write a function `build_sprite_labels_image` that takes two arguments `sprites` and `label_matrix`, the same returned by the function `find_sprites`.

The function `build_sprite_labels_image` accepts an optional argument `background_color` (either a tuple `(R, G, B)` or a tuple `(R, G, B, A)`) that identifies the color to use as the background of the image to build. If this argument is not passed to the function, the default value `(255, 255, 255)`.

The function `build_sprite_labels_image` returns an image of equal dimension (width and height) as the original image that was passed to the function `find_sprites`.

The function `build_sprite_labels_image` draw the masks the sprite at the exact same position the sprites were in the original image. The function draws each sprite mask with a random uniform color (one color per sprite mask).

For example:

```python
>>> from PIL import Image
>>> image = Image.open('optimized_sprite_sheet.png')
>>> sprites, sprite_mask = find_sprites(image)
# Draw sprite masks and bounding boxes with the default white background color.
>>> sprite_label_image = build_sprite_labels_image(sprites, sprite_mask)
>>> sprite_label_image.save('optimized_sprite_sheet_bounding_box_white_background.png')
# Draw sprite masks and bounding boxes with a transparent background color.
>>> sprite_label_image = build_sprite_labels_image(sprites, sprite_mask, background_color=(0, 0, 0, 0))
>>> sprite_label_image.save('/Users/dcaune/optimized_sprite_sheet_bounding_box_transparent_background.png')

```

| Sprite Masks with White Background                            | Sprite Masks with Transparent Background                            |
| ------------------------------------------------------------- | ------------------------------------------------------------------- |
| ![](optimized_sprite_sheet_bounding_box_white_background.png) | ![](optimized_sprite_sheet_bounding_box_transparent_background.png) |

<!--
# Waypoint: Write a class `SpriteSheet`

Write a class `SpriteSheet` which constructor accepts an argument `image` that corresponds to either:

- the name and path (a string) that references an image file in the local file system;
- a [`pathlib.Path`](https://docs.python.org/3/library/pathlib.html#pathlib.Path) object that references an image file in the local file system ;
- a [file object](https://docs.python.org/3/glossary.html#term-file-object) that MUST implement [`read()`](https://docs.python.org/3/library/io.html#io.IOBase.read), [`seek()`](https://docs.python.org/3/library/io.html#io.IOBase.seek), and [`tell()`](https://docs.python.org/3/library/io.html#io.IOBase.tell) methods, and be opened in binary mode;
- a [`Image`](https://pillow.readthedocs.io/en/stable/reference/Image.html) object.

This constructor also accepts an optional argument `transparent_color` that identifies the transparent color of the image. The type of `transparent_color` argument depends on the images' mode:

- an integer if the mode is grayscale;
- a tuple `(red, green, blue)` of integers if the mode is `RGB`;
- a tuple `(red, green, blue, alpha)` of integers if the mode is `RGBA`. The `alpha` element is optional. If not defined, while the image mode is `RGBA`, the constructor considers the `alpha` element to be `255`.
-->
