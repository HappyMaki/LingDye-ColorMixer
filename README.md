# LingDye-ColorMixer
This repo is a Unity Project example for more accurate color mixing for dying weapons and clothing in video games.

In normal RGB and HSV color space, Blue + Yellow will create gray when it's intended to be green. This project shifts the rgb color space into pigment space before doing calculations to get the expected color mix.

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
