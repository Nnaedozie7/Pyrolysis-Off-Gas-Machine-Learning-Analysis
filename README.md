# Pyrolysis Off-Gas Machine Learning Analysis

This project applies **unsupervised machine learning** to **pyrolysis off-gas data**.  
The goal was **not** to build a predictive model, but to understand **what is happening chemically and when**, using multivariate data analysis.

The workflow evolved as I learned more about the data, and this README reflects the **final, correct reasoning path**.

---

## What this project is about

I worked with **time resolved off-gas composition data** from a pyrolysis process.  
The dataset contains concentrations of multiple gas species (CO₂, CO, carbonates, hydrocarbons, aldehydes, acid gases, etc.) measured over time.

The main questions I wanted to answer were:

- How does the **overall chemical activity** evolve over time?
- Are there **distinct process regimes**, or is the process continuous?
- Can I **detect reaction onsets, peaks, and burnout** automatically?
- Can I separate **primary decomposition chemistry** from **secondary / oxidation chemistry**?

---

## Key idea behind the approach

This project treats the problem as a **process analysis task**, not a classification task.

That means:
- No labels
- No confusion matrix
- No accuracy scores

Instead, the focus is on:
- structure discovery
- physical interpretability
- time resolved chemical insight

---

## Overall ML workflow (final, corrected order)

### 1. Data preparation
- Cleaned the raw Excel file
- Fixed headers and column names
- Converted all gas signals to numeric values
- Converted time (`HH:MM:SS`) to seconds since start
- Sorted data strictly by time

This step ensures the data behaves like a **true time series**.

---

### 2. Feature definition
Excluded variables that should **not** participate in multivariate chemistry analysis:
- Time
- Dilution / operating parameters
- Null or carrier gas flags

Only **chemical concentration variables** were used for ML.

---

### 3. Scaling
Applied **standardization (StandardScaler)** to all chemical features.

This ensures:
- No gas dominates PCA simply because of magnitude
- PCA reflects **correlation structure**, not units

---

### 4. Principal Component Analysis (PCA)
PCA was used as the **core dimensionality reduction and interpretation tool**.

What PCA solved:
- Reduced ~26 gas variables into a few dominant modes of variation
- Revealed how gas compositions co-vary over time

No time information was used in PCA  time was only used later for interpretation.

---

### 5. PCA interpretation (critical step)
This step was key and must come **before clustering**.

From loadings and time colored PCA plots:

- **PC1** was interpreted as  
  **primary decomposition / reaction progress**
- **PC2** was interpreted as  
  **secondary chemistry (oxidation, aldehydes, acid gases, late CO₂)**

PC1 showed a clean, smooth trajectory over time.  
PC2 was noisier and more heterogeneous — which is expected for secondary pathways.

---

### 6. Event detection on PC1 (instead of early clustering)
Because the process was **continuous**, clustering was not the right first tool.

Instead, I treated **PC1 as a reaction progress variable** and applied **event detection**:

Detected automatically:
- Reaction onset
- Maximum decomposition (peak)
- Reaction end / burnout

This was done using the **time derivative of PC1**.

---

### 7. Validation using individual gas species
To confirm that PCA-based events were physically meaningful:

- PC1 events were overlaid with key gas signals
- Carbonates, aromatics, and oxygenated species peaked at the PC1 peak
- This validated PC1 as a true chemical progress variable

Importantly:
- CO₂ showed delayed behavior, confirming it is not a primary decomposition marker

---

### 8. PC2-based secondary chemistry analysis
PC2 was analyzed separately and differently from PC1:

- PC2 gases were plotted together (CO₂, CO, aldehydes, HF, etc.)
- PC2 was smoothed to reduce noise
- Threshold based detection was used instead of simple maxima
- The **delay between PC1 peak and PC2 activation** was quantified

This revealed:
- Secondary chemistry occurs **after** primary decomposition
- PC2 captures **heterogeneous, overlapping reaction pathways**

---

### 9. Clustering (exploratory, not decisive)
K-Means and hierarchical clustering were applied **after** PCA interpretation and event detection.

Result:
- Clustering mostly reproduced the PCA trajectory
- Only the high activity decomposition peak formed a distinct cluster

This confirmed:
- The process is **event dominated**, not state dominated
- Clustering was informative but not the main analytical tool

---

## Why this workflow makes sense

- PCA extracted **latent chemical variables**
- Event detection respected **continuous process behavior**
- PC1 and PC2 separated **primary vs secondary chemistry**
- Validation was done using **real gas species**, not metrics
- No artificial labels were forced onto the data

This approach prioritizes **physical meaning over algorithm complexity**.

---

## Key takeaways

- Unsupervised ML is about **discovering structure**, not prediction
- PCA can act as a **reaction progress coordinate**
- Clustering is not always the right first step
- Event detection is often superior for smooth chemical processes
- Messy signals (PC2) often mean **real heterogeneous chemistry**.

---

## What this project is (and is not)

✔ A multivariate process analysis study  
✔ A learning focused ML workflow  
✔ Chemically interpretable and defensible  

✘ Not a supervised learning problem  
✘ Not a classification task  
✘ Not meant to optimize accuracy  

---

## Possible extensions

- Compare PC1–PC2 delays across multiple runs
- Use detected events as labels for supervised learning
- Apply anomaly detection for safety monitoring
- Generalize the workflow to other thermal processes

---

## Final note

This project was intentionally iterative.  
Some steps (like early clustering) were explored, questioned, and repositioned as understanding improved.

That iteration is **part of the learning**, and the final workflow reflects the **correct scientific reasoning**, not just a clean script.

