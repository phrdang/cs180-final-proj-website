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


## Poisson Blending

## Bells & Whistles

### Mixed Gradients

[Back to home page](../index.md)
