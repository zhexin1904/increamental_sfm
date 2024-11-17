# Dependency

The only library needed should be Open3D(used for visualization), python version can be install by : 

`pip install open3d`

# Usage

`sfm.ipynb ` and `sfm_py` is actually the same.

The only difference is that notebook just keep the procedure of each registration, you can see how we build it incrementally.

In `sfm.py` , you can also uncomment line 746

```python
# visualize_multiple_cameras_and_points(cameras_initial, tracks)
```

to see the incremental reconstruction.

# Simple tutorial 

## what's different from ZZ's code

I guess professor's purpose of the assignment is to design a simplified SfM. But my designing is follow the modern SFM pipeline(like COLMAP, Theia...), so a little bit more complicated, but it is how SFM actually works.

I checked ZZ's code just now. The biggest difference is that I only use E/F matrix in initialization stage. It is required to use essential/fundamental matrix to estimate the transformation between two images, but if we have some stable 3D points, given 2D features, using PnP is a better choice, since tracking on 3D point is more accurate than using a 2D-2D methods(E/F matrix). (Most modern SLAM/SFM pipeline using 3D-2D methods on registering new image, instead of 2D-2D methods)

## Basic workflow

The key idea of incremental SFM is incremental optimization. Every time we register a new photo, we perform an optimization. It will takes much longer time, but more accurate, one example is COLMAP

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
   2. Estimate the camera pose (extrinsic parameters) using the PnP method.
   3. Triangulate new tracks.
   4. Remove edge \( e \) from \( G \).
   5. Perform Bundle Adjustment.(I didn't apply this step)
9. **End.**

(Actually I only do a global BA after registering, so it is more like a incremental registering, but only optimize in the end).

## Tracks

how to get tracks can be explained by http://imagine.enpc.fr/~moulonp/publis/poster_CVMP12.pdf 

## co-visibility graph



## Initialization

Initialization is a very very important in SLAM, SFM. Different for SLAM, which is a on-line tracking system, SFM is an off-line scheme, so we can pick some images to initialize the system, instead just use image1 and image2.



The key idea I used is called incremental SFM pipeline, common reference can be :

https://openmvg.readthedocs.io/en/latest/#

http://theia-sfm.org/

