# Esempio labeling/region growing - Codice 3.5
Nel seguente codice è simulato il comportamento degli algoritmi di labeling e region growing. 

```python
# -*- coding: utf-8 -*-
"""
Created on Tue May 28 11:42:56 2024

@author: alejandro
"""

import pydicom
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import k_means
from skimage import measure
from skimage.segmentation import flood

ds = pydicom.dcmread("./data/phantom.dcm")
image = ds.pixel_array
print(ds)

# get histogram
p = np.max(image, None)
print(p)
counts, bins = np.histogram(image, bins = p)

# display phantom image and image histogram
fig, ax = plt.subplots(1,2)
ax[0].imshow(image, cmap = "gray")
ax[1].plot(counts[1:]/np.sum(counts, None))
plt.show()

# kmeans
x = image.reshape((-1,1))
centroid, label, inertia = k_means(x, n_clusters = 3)
image_classes = label.reshape(image.shape)
idx_c = np.argsort(centroid, axis = 0)

# get water mask
mask = np.zeros_like(label)
idx = np.where(label == idx_c[1])
mask[idx]=255
mask = np.reshape(mask, image.shape)

# do labeling on water mask
labeled_image = measure.label(mask, connectivity=2) 
fig, ax = plt.subplots(1, 3)
ax[0].imshow(image, cmap = 'gray')
ax[1].imshow(mask, cmap = "gray")
ax[2].imshow(labeled_image)
plt.show()

# region growing
seed = (49, 107)
tmp = image.copy()
fig, ax = plt.subplots(1, 4)
ax[0].imshow(image, cmap = "gray")
mask = flood(tmp, seed, tolerance=50)
ax[1].imshow(mask, cmap = "gray")
mask = flood(tmp, seed, tolerance=100)
ax[2].imshow(mask, cmap = "gray")
mask = flood(tmp, seed, tolerance=400)
ax[3].imshow(mask, cmap = "gray")
plt.show()

plt.ion()
fig, ax = plt.subplots(1, 2)
npoints = np.zeros([500])
x = np.arange(0,500)
for T in range(0,500,5):
    mask = flood(tmp, seed, tolerance = T)
    ax[0].imshow(mask, cmap = "gray")
    npoints[T] = np.sum(mask, axis = None)
    ax[1].plot(x, npoints)
    plt.pause(0.001)
    plt.gcf().canvas.flush_events()
    ax[1].clear()
    
plt.ioff()
plt.show()
```