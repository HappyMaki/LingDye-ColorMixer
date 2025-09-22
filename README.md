# LingDye-ColorMixer
This repo is a Unity Project example for more accurate color mixing for dying weapons and clothing in video games.

In normal RGB and HSV color space, Blue + Yellow will create gray when it's intended to be green. This project shifts the rgb color space into pigment space before doing calculations to get the expected color mix.

```
Vector3 ToPigmentSpace(Color color)
{
    // Convert RGB color to standard Hue Saturation Value (range 0-1 each).
    Color.RGBToHSV(color, out float h, out float s, out float v);

    // Change hue to degrees for ease of intuition.
    h *= 360f;

    // Compute a shift that will stretch the orange range
    // until yellow reaches the 120 degree mark,
    // and compress the cyan/teal range so blue stays at 240 degrees.
    float hueShift = Mathf.Clamp01(h > 90f ? 2.0f - h / 120f : h / 60f);

    float shifted = (h + 60f * hueShift) * Mathf.Deg2Rad;

    // Model the colour as a point in a spindle, with the bright hues
    // around the outer rim, and the greyscale running up the z-axis.
    Vector3 pigmentSpace = new Vector3(
                       s * Mathf.Cos(shifted),
                       s * Mathf.Sin(shifted),
                       v - 0.5f * s);

    return pigmentSpace;
}
```

```
// Convert back from our pseudo-pigment spindle space to an RGB color.
Color FromPigmentSpace(Vector3 pigmentSpace)
{
    // The hue & saturation are expressed in the XY plane.
    var chromaticity = (Vector2)pigmentSpace;

    // Saturation is our radius from the grey axis.
    float s = chromaticity.magnitude;
    // Value is our height along the spindle, treating saturated colours as max.
    float v = pigmentSpace.z + 0.5f * s;

    // Decode our shifted hue angle into degrees.
    float shifted = Mathf.Atan2(chromaticity.y, chromaticity.x) * Mathf.Rad2Deg;
    if (shifted < 0f)
        shifted += 360f;

    // Undo the hue shift to bring us back to standard HSV space.
    float hueShift = Mathf.Clamp01(shifted > 150f ? 4.0f - shifted / 60f : shifted / 120f);
    float h = shifted - 60f * hueShift;

    // Convert from HSV back to RGB.
    return Color.HSVToRGB(h / 360f, s, v);
}
```

```
Vector3 pigmentSpace = Vector3.zero;

Color blue = Color.red;
Color yellow = Color.green;

pigmentSpace += ToPigmentSpace(blue) * 0.5f;
pigmentSpace += ToPigmentSpace(yellow) * 0.5f;

Color result = FromPigmentSpace(pigmentSpace);
Debug.Log($"Mixing {blue} and {yellow} gives {result}");
```


References:
https://gamedev.stackexchange.com/questions/177209/what-colour-space-or-mixing-algorithm-should-i-use-for-watercolor-like-colour
