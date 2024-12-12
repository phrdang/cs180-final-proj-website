<!-- Mathjax Support -->
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# Gradient Domain Fusion

1. Table of Contents
{:toc}

This project explores gradient-domain processing, a technique that can be applied in blending, tone-mapping and non-photorealistic rendering. Our primary focus is Poisson blending which also serves as the foundation for blending with mixed gradients. 

In order to blend multiple images seamlessly, we previously used Gaussian and Laplacian stacks to implement multiresolutional blending. However, this still resulted in noticeable seams. To resolve this issue, we recognize that the human eye is more sensitive to the gradient of an image than the overall intensity. Thus, we assign pixel values to maximally preserve the gradient of the source region without changing any of the background pixels in the target image. As a side effect, because we disregard the overall intensity, this method may change colors of objects in the source region which may or may not be desired. 

## Toy Problem
To familiarize ourselves with gradient domain operations, we reconstruct an image from gradients. This can be formulated as an optimization problem. 

Given a source image $$s$$, our goal is to construct the identical $$v$$ with the same gradients. Therefore, each pixel $$v(x, y)$$ has two objectives:

1. $$\min_v ( v(x+1,y)-v(x,y) - (s(x+1,y)-s(x,y)) )^2$$ to closely match the $$x$$-gradients of $$v$$ to those in $$s$$
2. $$\min_v ( v(x,y+1)-v(x,y) - (s(x,y+1)-s(x,y)) )^2$$ to closely match the $$y$$-gradients of $$v$$ to those in $$s$$
Because these objectives can be solved while adding a constant value to $$v$$, we add an another objective:
3. $$\min_v (v(0,0)-s(0,0))^2$$ to ensure that the top left corners of $$v$$ and $$s$$ are the same color.

We then converted this to a least squares problem $$v = \arg\min_v \lVert A \vec{v} - \vec{b} \rVert ^2$$ by constructing $$A$$ and $$\vec{b}$$ as follows: 

$$\begin{aligned} A(e, \text{im_to_var}(y, x + 1)) &= 1 \\ A(e, \text{im_to_var}(y, x)) &= -1 \\ b(e) &= s(y, x + 1) - s(y, x)\end{aligned}$$

and

$$\begin{aligned} A(e, \text{im_to_var}(y + 1, x)) &= 1 \\ A(e, \text{im_to_var}(y, x)) &= -1 \\ b(e) &= s(y + 1, x) - s(y, x)\end{aligned}$$

- $$e =$$ index of each equation
- $$\text{im_to_var}(y, x) = $$ array that maps each $$(y, x)$$ coordinate to its flattened 1D value

| Source image $$s$$                      | Reconstructed image $$v$$             |
|:----------------------------------------|:--------------------------------------|
| ![Source image](assets/toy_problem.png) | ![Reconstruction](assets/out_toy.png) |


## Poisson Blending

## Bells & Whistles

### Mixed Gradients

[Back to home page](../index.md)
