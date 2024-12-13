<!-- Mathjax Support -->
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# Gradient Domain Fusion

[//]: # (![mixed gradients blended mural]&#40;assets/out_vase_mural2_mix.png&#41;)
![poisson blended luna and hikers](assets/out_luna_hikers2.png)

1. Table of Contents
{:toc}

This project explores gradient-domain processing, a technique that can be applied in blending, tone-mapping and non-photorealistic rendering. Our primary focus is Poisson blending which also serves as the foundation for blending with mixed gradients. 

In order to blend multiple images seamlessly, we previously used Gaussian and Laplacian stacks to implement multiresolutional blending. However, this still resulted in noticeable seams at times. To resolve this issue, we recognize that the human eye is more sensitive to the gradient of an image than the overall intensity. Thus, we assign pixel values to maximally preserve the gradient of the source region without changing any of the background pixels in the target image. As a side effect, because we disregard the overall intensity, this method may change colors of objects in the source region which may or may not be desired. 

## Toy Problem
To familiarize ourselves with gradient domain operations, we reconstruct an image from gradients. Given a source image $$s$$, our goal is to construct the identical $$v$$ with the same gradients. Therefore, this can be formulated as an optimization problem with each pixel $$v(x, y)$$ having two objectives:

1. $$\min_v ( v(x+1,y)-v(x,y) - (s(x+1,y)-s(x,y)) )^2$$ to closely match the $$x$$-gradients of $$v$$ to those in $$s$$
2. $$\min_v ( v(x,y+1)-v(x,y) - (s(x,y+1)-s(x,y)) )^2$$ to closely match the $$y$$-gradients of $$v$$ to those in $$s$$

Because these objectives can be solved while adding a constant value to $$v$$, we add an another objective:

$$\min_v (v(0,0)-s(0,0))^2$$ to ensure that the top left corners of $$v$$ and $$s$$ are the same color.

We then convert this to a least squares problem $$v = \arg\min_v \lVert A \vec{v} - \vec{b} \rVert ^2$$ by constructing $$A$$ and $$\vec{b}$$ as follows: 

$$\begin{aligned} A(e, \text{im_to_var}(y, x + 1)) &= 1 \\ A(e, \text{im_to_var}(y, x)) &= -1 \\ b(e) &= s(y, x + 1) - s(y, x)\end{aligned}$$

and

$$\begin{aligned} A(e, \text{im_to_var}(y + 1, x)) &= 1 \\ A(e, \text{im_to_var}(y, x)) &= -1 \\ b(e) &= s(y + 1, x) - s(y, x)\end{aligned}$$

- $$e =$$ index of each equation
- $$\text{im_to_var}(y, x) = $$ array that maps each $$(y, x)$$ coordinate to its flattened 1D value

| Source image $$s$$                      | Reconstructed image $$v$$             |
|:----------------------------------------|:--------------------------------------|
| ![Source image](assets/toy_problem.png) | ![Reconstruction](assets/out_toy.png) |


## Poisson Blending

We adapt our image reconstruction process to implement Poisson blending. Given a target image $$t$$ and source image $$s$$, pixels are selected from a source region $$S$$ in $$s$$ and seamlessly incorporated with the pixels in $$t$$ to create the new image $$v$$. These values are combined and blended together by solving the following optimization problem:

$$\min_v \sum_{i \in S, j \in N_i \cap S} (v_i - v_j - (s_i - s_j))^2 + \sum_{i \in S, j \in N_i \cap \bar{S}} (v_i -
t_j - (s_i - s_j))^2$$
- $$s = $$ source image
- $$t = $$ target image
- $$v = $$ new blended image
- $$S = $$ source region
- $$\bar{S} = $$ area outside $$S$$
- $$i = $$ coordinate in $$S$$
- $$j = $$ 4-neighbor of $$i$$ (top, bottom, left, right) which is calculated by adding $$(dy, dx)$$ to $$i = (x, y)$$

First, $$s$$ is created by taking an image and transforming it so that the scene component to be integrated into $$t$$ is aligned and scaled to the correct size. In addition, we applied a binary mask to define the boundaries of $$S$$. This was all done using the software Krita. 

| Original source image $$s$$         | Target image $$t$$                    | Transformed $$s$$                             | Mask                               |
|:------------------------------------|:--------------------------------------|:----------------------------------------------|:-----------------------------------|
| ![Source image](assets/penguin.jpg) | ![Target image](assets/im2_small.jpg) | ![Transformed s](assets/penguin_source_v.jpg) | ![mask](assets/penguin_mask_v.jpg) |

As with the toy problem, the objective function is formulated as a least squares problem to solve for $$v = \arg\min_v \lVert A \vec{v} - \vec{b} \rVert ^2$$.

Each value of $$v_i$$ is computed depending on whether the neighbor of $$s_i$$ is in $$S$$ or $$\bar{S}$$ by creating $$A$$ and $$\vec{b}$$ as follows:

If $$s_j \in S$$:

$$\begin{aligned} A(e, \text{im_to_var}(y, x)) &= 1 \\ A(e, \text{im_to_var}(y + dy, x + dx)) &= -1 \\ b(e) &= s(y, x) - s(y + dy, x + dx)\end{aligned}$$

If $$s_j \in \bar{S}$$:

$$\begin{aligned} A(e, \text{im_to_var}(y, x)) &= 1 \\ b(e) &= t(y + dy, x + dx) + s(y, x) - s(y + dy, x + dx)\end{aligned}$$

Hikers & Penguin

| penguin                             | hikers                                | mask                                        | Poisson blended image $$v$$                       |
|:------------------------------------|:--------------------------------------|:--------------------------------------------|:--------------------------------------------------|
| ![Source image](assets/penguin.jpg) | ![Target image](assets/im2_small.jpg) | ![Transformed s](assets/penguin_mask_v.jpg) | ![poisson blended](assets/out_penguin_hikers.png) |

Botticelli's _Primavera_ & Elana's self-portrait

| Botticelli's _Primavera_                    | Elana's self-portrait                             | mask                                         | Poisson blended image $$v$$                  |
|:--------------------------------------------|:--------------------------------------------------|:---------------------------------------------|:---------------------------------------------|
| ![Source image](assets/primavera_small.jpg) | ![Target image](assets/elana_pensive_drawing.jpg) | ![Transformed s](assets/primavera_mask2.jpg) | ![poisson blended](assets/out_primavera.png) |

Sharks & Swimmer

| sharks                             | swimming child                            | mask                                            | Poisson blended image $$v$$                       |
|:-----------------------------------|:------------------------------------------|:------------------------------------------------|:--------------------------------------------------|
| ![Source image](assets/sharks.jpg) | ![Target image](assets/girl_swimming.jpg) | ![Transformed s](assets/girl_swimming_mask.jpg) | ![poisson blended](assets/out_sharks_swimmer.png) |

Lunazilla

| Luna                                   | city                             | mask                                          | Poisson blended image $$v$$                         |
|:---------------------------------------|:---------------------------------|:----------------------------------------------|:----------------------------------------------------|
| ![Source image](assets/luna_night.jpg) | ![Target image](assets/city.jpg) | ![Transformed s](assets/luna_night_mask2.jpg) | ![poisson blended](assets/out_luna_night_city2.png) |

 Mural

| _Se pousser (To push oneself)_   | brick wall                         | mask                                   | Poisson blended image $$v$$                   |
|:---------------------------------|:-----------------------------------|:---------------------------------------|:----------------------------------------------|
| ![Source image](assets/Vase.jpg) | ![Target image](assets/brick2.jpg) | ![Transformed s](assets/vase_mask.jpg) | ![poisson blended](assets/out_vase_mural.png) |


### Comparison with Laplacian blending

Botticelli's _Primavera_ & Elana's self-portrait

| Poisson blended image                        | Laplacian blended image                                                  |
|:---------------------------------------------|:-------------------------------------------------------------------------|
| ![poisson blended](assets/out_primavera.png) | ![laplacian blended](assets/out_elana_pensive_small2primavera_small.jpg) |

In this case, it is advantageous to use Poisson blending over Laplacian blending to match the overall color palette of the blended component to the background. On the other hand with Laplacian blending, the preservation of pixel intensities results in a rough graft onto the target image. 

Lunazilla

| Poisson blended image                               | Laplacian blended image                              |
|:----------------------------------------------------|:-----------------------------------------------------|
| ![poisson blended](assets/out_luna_night_city2.png) | ![laplacian blended](assets/out_luna_nightcity2.jpg) |

Laplacian blending has its merits in avoiding unwanted color changes. Here, using Poisson blending caused an odd green stain on Luna, while this did not occur with Laplacian blending. In this case in particular, the rough boundaries around Luna in addition to the bright flash in the original image contribute to the charm of the blended result. Stylistically, the image of giant cat in the city is not meant to look smooth and natural. Laplacian blending achieves a roughly cut-and-pasted effect that enhances the chaos wreaked in the city by a gamboling Lunazilla.     

## Bells & Whistles

### Mixed Gradients

Building off of Poisson blending, we implement mixed gradient blending. This involves the same general process but instead of always using the source gradient, we use the gradient with the larger magnitude from either the source or the target.

$$\min_v \sum_{i \in S, j \in N_i \cap S} (v_i - v_j - d_{ij})^2 + \sum_{i \in S, j \in N_i \cap \bar{S}} (v_i -
t_j - d_{ij})^2$$

if $$ \lvert s_i-s_j \rvert > \lvert t_i-t_j \rvert $$, then $$d_{ij} = s_i - s_j$$; otherwise $$d_{ij} = t_i-t_j$$.

Hikers & Penguin

| Poisson blended image                             | Mixed gradients blended image                                 |
|:--------------------------------------------------|:--------------------------------------------------------------|
| ![poisson blended](assets/out_penguin_hikers.png) | ![mixed gradients blended](assets/out_penguin_hikers_mix.png) |

Mural 

| Poisson blended image                         | Mixed gradients blended image                              |
|:----------------------------------------------|:-----------------------------------------------------------|
| ![poisson blended](assets/out_vase_mural.png) | ![mixed gradients blended](assets/out_vase_mural2_mix.png) |

Through mixed gradient blending, strong features of both $$s$$ and $$t$$ are preserved. In some cases, Poisson blending produces a blurry outline of the mask in the resulting image. Mixed gradient blending avoids this, creating a more natural and smoother transition. However, if the goal is to fully replace a region in $$t$$ with pixels in $$s$$, mixed gradient blending does not always achieve the desired effect.

Botticelli's _Primavera_ & Elana's self-portrait (failure case)

| Poisson blended image                        | Mixed gradients blended image                            |
|:---------------------------------------------|:---------------------------------------------------------|
| ![poisson blended](assets/out_primavera.png) | ![mixed gradients blended](assets/out_primavera_mix.png) |

Because both $$s$$ and $$t$$ have high-magnitude gradients, the result is a person with their eyes simultaneously opened and closed. While the nose and mouth in the mixed gradient result are blended successfully, the Poisson blended result is more natural overall. 

[Back to home page](../index.md)
