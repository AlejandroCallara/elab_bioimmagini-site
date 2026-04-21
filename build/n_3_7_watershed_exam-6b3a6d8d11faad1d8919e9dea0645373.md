# Esempio watershed - Codice 3.7
Nel seguente codice è simulato il comportamento dell'algoritmo watershed. 

```python

import numpy as np
import matplotlib
matplotlib.use("QtAgg")  # keep if you need interactive window in VS Code

import matplotlib.pyplot as plt
import pydicom
from skimage.segmentation import flood

I = pydicom.dcmread('../data/phantom.dcm').pixel_array.astype(np.float32)
dim = I.shape[0]
maxImage = int(I.max())

area = np.zeros(maxImage + 1, dtype=np.int64)
profile = I[:, dim//2]

fig, axes = plt.subplots(1, 3, figsize=(18, 6))
plt.ion()
plt.show(block=False)

# --- (0) overlay panel: create once ---
axes[0].set_title("Threshold Overlay")
im_base = axes[0].imshow(I, cmap='gray')
overlay = np.zeros((dim, dim, 4), dtype=np.float32)   # RGBA
overlay[..., 0] = 1.0                                 # red
overlay[..., 3] = 0.0                                 # alpha updated each iter
im_overlay = axes[0].imshow(overlay)
axes[0].axis("off")

# --- (1) area curve: create once ---
axes[1].set_title("Segmented Area vs Threshold")
axes[1].set_xlim(0, maxImage)
axes[1].set_ylim(0, I.size)
line_area, = axes[1].plot([], [], 'r', linewidth=2.0)

# --- (2) profile + threshold line: create once ---
axes[2].set_title("Profile with Threshold")
axes[2].set_xlim(0, dim-1)
axes[2].set_ylim(profile.min(), profile.max())
axes[2].plot(profile, 'k')
hline = axes[2].axhline(0, color='r', alpha=0.2)

# Update frequency (visualization only)
step = 5   # show every 5 thresholds (set 1 for every frame)

for th in range(1, maxImage):
    BW = flood(I, (4, 4), tolerance=th)
    area[th] = BW.sum()

    # only refresh the figure sometimes (big speedup)
    if th % step != 0:
        continue

    overlay[..., 3] = BW.astype(np.float32) * 0.30
    im_overlay.set_data(overlay)

    line_area.set_data(np.arange(1, th+1), area[1:th+1])
    hline.set_ydata([th, th])

    fig.canvas.draw_idle()
    plt.pause(0.1)

plt.ioff()
plt.show()
```