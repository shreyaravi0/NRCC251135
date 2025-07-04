# ☁️ Cloud and Shadow Detection using TOA Reflectance Thresholding

This project performs cloud and shadow segmentation on satellite imagery using **thresholding on Top-Of-Atmosphere (TOA) reflectance** values derived from multispectral bands.

---

## 📁 Dataset Structure

data/
└── train/
└── cloud/
├── [SceneID]/
│ ├── BAND2.tif
│ ├── BAND3.tif
│ ├── BAND4.tif
│ ├── BAND_META.txt

yaml
Copy
Edit

Each scene contains:
- Georeferenced TIFF images for **BAND2**, **BAND3**, and **BAND4**
- `BAND_META.txt` containing sensor metadata (e.g., Sun Elevation, Lmin/Lmax values)

---

## 🔄 Preprocessing Pipeline

### Step 1: DN → Radiance → Reflectance
For each band:
1. **Radiance**:  
   
   L = Lmin + (DN / 255) * (Lmax - Lmin)

   
2. **Reflectance**:  
   
   ρ = (π * L * d^2) / (Esun * cos(θ))

   

Where:
- `d` = Earth–Sun distance (AU)
- `Esun` = Solar exo-atmospheric irradiance
- `θ` = Solar zenith angle = 90° – Sun Elevation

### Step 2: Create False Color Composite (FCC)
Use:
- R = BAND4 (NIR)
- G = BAND3 (Red)
- B = BAND2 (Green)

---

## 🧠 Classification Logic

Each pixel is classified into:

| Class     | Label | Condition (Reflectance)                      |
|-----------|-------|----------------------------------------------|
| NOCLOUD   | `0`   | Default (not cloud or shadow)                |
| CLOUD     | `1`   | `B2 > 0.35` and `B3 > 0.23` and `B4 > 0.22`  |
| SHADOW    | `2`   | `B2, B3, B4 < ~0.1` and `NDVI < 0.2`         |

NDVI is calculated as:

\[
\text{NDVI} = \frac{B4 - B3}{B4 + B3}
\]

---

## 📤 Outputs

Each scene outputs:

outputs/
├── reflectance/
│ └── [SceneID]_reflectance.tif
├── masks/
│ └── [SceneID]_mask.tif
├── overlays/
│ └── [SceneID]_mask_overlay.png

yaml
Copy
Edit

- **Reflectance TIFF**: 3-band georeferenced reflectance image
- **Mask TIFF**: 8-bit georeferenced semantic mask (0=NOCLOUD, 1=CLOUD, 2=SHADOW)
- **Overlay PNG**: RGB FCC + mask overlay for visual inspection


## 🛠️ Dependencies

```bash
pip install numpy matplotlib rasterio scikit-learn
