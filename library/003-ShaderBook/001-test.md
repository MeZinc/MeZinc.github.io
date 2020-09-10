# The Book of Shader

**HSB to RGB shader code**

```c++
vec3 rgb = clamp(abs(mod(hue * 6.0 + vec3(0.0,4.0,2.0), 6.0) - 3.0) - 1.0, 0.0, 1.0 );
rgb = rgb*rgb*(3.0-2.0*rgb);
return brightness * mix (vec3(1.0), rgb, saturation);
```

