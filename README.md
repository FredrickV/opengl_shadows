# opengl_shadows

**Important notes**

1. Shadow code is in camera space; not in world space.

**General Tips**

1. Apply bias in the shadow map pass. 
2. Cull back face only (shadow cast front face)
3. Experiment with glPolygonOffset(xx,xx); glEnable(GL_POLYGON_OFFSET_FILL); when rendering depth pass. (I am currently not using it)



# Sloped Based Bias

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
