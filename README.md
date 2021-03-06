# opengl_shadows - Work in progress!

**Important notes**

1. The code samples and the "tips" are currently written based on my current knowledge, and in addition best suited for my rendering engine. Other practices may be better suited for your needs.
2. Shadow code is in camera space; not in world space.
 

**General Tips**

1. Apply bias in the shadow map pass, not in the light pass unless you perform forward rendering.
2. Cull back face only (shadow cast front face)
3. Experiment with glPolygonOffset(xx,xx); glEnable(GL_POLYGON_OFFSET_FILL); when rendering depth pass. (I am currently not using it)
4. Experiment with dFdxCoarse(fast) vs dFdxFine(slower) for performance gains. I have not seen any quality loss with the fast method (Req. OpenGL 4.5+)

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
          // Normal Matrix (can be pre-computed)
          mat3(V *M) 
          // Light inverse direction
          * normalize(-LightDirection)));
// Clamp to bias  
float bias = max(DIR_BIAS  * sqrt(1.0 - p * p) / p, DIR_BIAS); 

float depth = Position.z * 0.5 + 0.5;
gl_FragDepth = (depth + abs( dFdx(depth) ) + abs(dFdy(depth) ) ) +  bias;
 ```
 
 
 # Solving shimmering shadows
 
 Function returns a new ortho, replace it with your source/original orthographical projection
 ```
 glm::mat4 ShadowOrthoRound(const int shadowSize, const glm::mat4 &shadowOrtho, const glm::mat4 &shadowView) {
    glm::mat4 shadowMatrix = (shadowOrtho * shadowView);
    glm::vec4 shadowOrigin = shadowMatrix * glm::vec4(0.0f, 0.0f, 0.0f, 1.0f);
    shadowOrigin = shadowOrigin * (float)shadowSize / 2.0f;

    glm::vec4 roundedOrigin = glm::round(shadowOrigin);
    glm::vec4 roundOffset = roundedOrigin - shadowOrigin;
    roundOffset = roundOf fset *  2.0f / (float)shadowSize;
    roundOffset.z = 0.0f;
    roundOffset.w = 0.0f;

    glm::mat4 roundedShadowOrtho = shadowOrtho;
    roundedShadowOrtho[3] += roundOffset;
    return roundedShadowOrtho;
}
```
