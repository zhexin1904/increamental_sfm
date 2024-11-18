# Dependency

The only library needed should be Open3D(used for visualization), python version can be install by : 

`pip install open3d`

(and also networkx, if haven't installed it : `pip install networkx`)

# Usage

`sfm.ipynb ` and `sfm_py` is actually the same.

The only difference is that notebook just keep the procedure of each registration, you can see how we build it incrementally.

In `sfm.py` , you can also uncomment line 746

```python
# visualize_multiple_cameras_and_points(cameras_initial, tracks)
```

to visualize the incremental reconstruction procedure.

# Simple tutorial 

## what's different from ZZ's code

I guess professor's purpose of the assignment is to design a simplified SfM. But my designing is follow the modern SFM pipeline(like COLMAP, Theia...), so a little bit more complicated, but it is how SFM actually works.

I checked ZZ's code just now. The first difference is that I only use E/F matrix in initialization stage. It is required to use essential/fundamental matrix to estimate the transformation between two images, but if we have some stable 3D points, given 2D features, using PnP is a better choice, since tracking on 3D point is more accurate than using a 2D-2D methods(E/F matrix). (Most modern SLAM/SFM pipeline using 3D-2D methods on registering new image, instead of 2D-2D methods)

The second difference is the sequence on registering new image. ZZ's code is doing it following the order of the images. But for SFM, which is more like a off-line pipeline(compared with SLAM), do not to register new  image in some sequence. Indeed, we can pick the "best" candidate image from the rest of them, based on some metric, which can make our multi-view system more accurate. There are a lot of methods on how to choose the next image to register in SFM field. What I used to it to find the image with the most triangulated feature points, i.e. to find the image that observes the greatest number of triangulated 3D points. So we can derive the as much 3D-2D pairs as we can, which will provide more data to PnP(potentially improve the accuracy of it).

## Basic workflow

The key idea of incremental SFM is incremental optimization. Every time we register a new photo, we perform an optimization. It will takes much longer time, but more accurate, one example is COLMAP

In simple terms, we first extract features from all images and perform pairwise matching to obtain the Tracks and the co-visibility graph. Then, we compute the **essential matrix**, recover **R** and **t**, and triangulate some feature points to establish a **two-view system** (this process is also known as initialization). Next, we select the following image from the remaining set, use **PnP** to recover **R** and **t**, and triangulate additional feature points(also called register new image to the multi-view system).
Finally, we continue this process until all images are registered. At the end, we perform bundle adjustment (BA) for optimization.

**Input:** Camera intrinsic parameters, feature points, and matching results  
**Output:** 3D point cloud, camera poses  

### Steps:

1. Pre-process : feature extraction, Cross-match all images, and Compute the corresponding **tracks** of feature points
2. Construct a connected graph \( G \)(this can also be called co-visibility graph), where:
   - Nodes represent images.
   - Edges represent pairs of images with sufficient matching points.
3. Select an edge \( e \) from \( G \).
4. Estimate the essential matrix \( E \) corresponding to edge \( e \) using RANSAC.
5. Decompose the essential matrix \( E \) to obtain the relative pose between the two images.
6. Triangulate tracks to obtain initial 3D points
7. Remove edge \( e \) from \( G \).

8. **While there are still edges in \( G \):**
   1. Select edge \( e \) from \( G \) that maximizes the number of already having riangulate 3D points.
   2. Estimate the camera pose using the PnP method.
   3. Triangulate new tracks.
   4. Remove edge \( e \) from \( G \).
   5. Perform Bundle Adjustment.(I didn't apply this step)
9. **End.**

(Actually I only do a global BA after registering, so it is more like a **incremental registering**, but only optimize in the end).

## Tracks
"Tracks" is a commonly used data structure in Structure-from-Motion (SFM). It records observations of the same 3D point across multiple different images.
how to get tracks can be explained by http://imagine.enpc.fr/~moulonp/publis/poster_CVMP12.pdf 

## co-visibility graph

For the co-visibility graph, the nodes represent each image. We perform pairwise matching for all images, and if the number of matches exceeds a certain threshold, we add an edge between the two images.

## Initialization

Initialization is a very very important in SLAM, SFM. Different for SLAM, which is a on-line tracking system, SFM is an off-line scheme, so we can pick some images to initialize the system, instead just use image1 and image2.
Actually, if we just use image 1 and image 2 to initialize the system, result maybe not that good, since the disparity is really tiny.

But I also used a heuristic here(I am lazy). I used image 7 and image 8 to initialize the two-view system, since the disparity is suitable. 
If you want to achieve a precise initialization, a common way used in some SFM pipelines is to initialize all edges in the graph and select the edge with the best triangulation results (which is ultimately reflected by ensuring that the angles between rays corresponding to all points fall within a certain range).
## How to evaluate your solution
The two most commonly used metrics：
1, reprojection error, or just the objective value after optimization(same idea)
2, The number of successfully reconstructed points and cameras.(How many 3D points in your final result)

## Other useful link

https://openmvg.readthedocs.io/en/latest/#

http://theia-sfm.org/

