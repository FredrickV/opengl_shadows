# opengl_shadows - Work in progress!

**Important notes**

1. The code samples and the "tips" are currently written based on my current knowledge, and in addition best suited for my rendering engine. Other practices may be better suited for your needs.
2. Shadow code is in camera space; not in world space.
 

**General Tips**

1. Apply bias in the shadow map pass. 
2. Cull back face only (shadow cast front face)
3. Experiment with glPolygonOffset(xx,xx); glEnable(GL_POLYGON_OFFSET_FILL); when rendering depth pass. (I am currently not using it)
scsd 


# Sloped Based Bias (Depth Map Pass)

Fragment Shader:
NormalOut came from the vertex shader (Surface normal)

```
#define BIAS 0.00002
float p = dot(normalize(NormalOut), normalize(
          // Normal Matrix (can be pre-computed)
          mat3(V *M) 
          * normalize(-LightDirection)));
// Clamp to bias          
float bias = max(BIAS  * sqrt(1.0 - p * p) / p, BIAS); 
```

# Normalized Device Coordinates (NDC): Directional lights versus local light sources such as spot and point lights (Depth Map Pass)

Fragment Shader, spot and point shadows:
```
// Perspective divide (Position.w)
float depth = Position.z/Position.w * 0.5 + 0.5;
```

Fragment Shader, Directional light

```
float depth = Position.z * 0.5 + 0.5;
```


#  Getting the Partial derivative, increases accuracy of bias

```
gl_FragDepth = (depth + abs( dFdx(depth) ) + abs(dFdy(depth) ) ) +  bias;
```


# Putting it all together (Directional light)

```
#define DIR_BIAS 0.00002
float p = dot(normalize(NormalOut), normalize(
          // Normal Matrix
          mat3(V *M) 
          // Light inverse direction
          * normalize(-LightDirection)));
float bias = max(DIR_BIAS  * sqrt(1.0 - p * p) / p, DIR_BIAS); 

float depth = Position.z * 0.5 + 0.5;
gl_FragDepth = (depth + abs( dFdx(depth) ) + abs(dFdy(depth) ) ) +  bias;
 ```
