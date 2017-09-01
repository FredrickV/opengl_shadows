# opengl_shadows

Important note:
Shadow code is in camera space; not in world space.



# Sloped Based Bias
```
#define BIAS 0.00002
float p = dot(normalize(NormalOut), normalize(
          // Normal Matrix (can be pre-computed)
          mat3(V *M) 
          * normalize(-LightDirection)));
float bias = max(BIAS  * sqrt(1.0 - p * p) / p, BIAS); 
```
