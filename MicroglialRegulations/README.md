# Microglial NF-κB Regulation by A20 and CYLD

This folder contains a RoadRunner-based simulation workflow for studying how negative regulation by A20 and CYLD shapes canonical NF-κB dynamics in a Lipniacki-style model.

The notebook assumes that the SBML/XML model file is in the **same directory** as the notebook:

- `RoadRunner-Lipniacki.ipynb`
- `LipniackiWithCYLD.xml`

---

## What this notebook does

The notebook loads the SBML model into RoadRunner, applies a transient TNF pulse, simulates nuclear NF-κB over time, and summarizes the response using an **inflammation index**.

It then generates heat maps showing how the inflammation index changes when CYLD regulation is varied together with different A20-related parameters:

- `c1` = A20 transcription
- `c4` = A20 translation
- `c5` = A20 degradation
- `k2` = A20 feedback strength on signaling
- `k_cyld` = CYLD-mediated upstream inhibition strength

The notebook also includes example time-course plots comparing control and perturbed conditions.

---

## Biological meaning of the parameters

These parameters represent different layers of negative regulation in the NF-κB pathway:

- **k_cyld**: effective strength of CYLD-mediated inhibition upstream of IKK activation
- **k2**: effective strength of delayed A20-mediated inhibitory feedback on signaling
- **c1**: NF-κB-dependent transcription rate of A20 mRNA
- **c4**: translation rate of A20 protein from A20 mRNA
- **c5**: degradation rate of A20 protein

These are model parameters, not directly measured single biological quantities. In real cells, each one reflects a combination of expression level, enzymatic activity, post-translational regulation, and complex formation.

---

## Inflammation index: what it is and why it was used

A major goal of the notebook is to move beyond looking only at peak NF-κB activation.

In microglia, inflammatory burden is not determined solely by how high NF-κB rises at its maximum. It also depends on:

- How long NF-κB stays elevated
- How much total NF-κB activity accumulates over time
- Whether the response relaxes back toward baseline or remains persistently active

To capture that, the notebook defines an **inflammation index** from the simulated nuclear NF-κB trajectory.

### Definition used in the notebook

For each simulation:

1. **Baseline level** is taken as the initial NF-κB value at the start of the simulation.
2. The NF-κB trajectory is converted to **NF-κB above baseline**.
3. The area under that above-baseline curve is computed.
4. That area is divided by the total simulation duration to get **mean exposure above baseline**.
5. The average NF-κB level over the final 10 percent of the trajectory is used (for persistence) as the **late-time plateau**.
6. The plateau is also expressed **above baseline**.
7. The final inflammation index is computed using the 70-30 split so it reflects both total exposure and persistent signaling, but gives more weight to the part that likely matters most overall. Calculation goes as follows:

```text
inflammation index = 0.70 * mean exposure above baseline
                   + 0.30 * late-time plateau above baseline
```

### Why these two terms are included

The index combines two biologically relevant ideas:

- **Mean exposure above baseline** captures total cumulative signaling burden across the full time course.
- **Late-time plateau above baseline** captures persistent signaling that remains after the main response phase.

This is useful because a signaling trace can have:

- A high peak but short duration
- A moderate peak but long persistence
- Strong adaptation with little late activity
- Weak adaptation with prolonged inflammatory exposure

The inflammation index gives a single summary value that reflects both overall burden and persistence.

### Interpretation

- **Higher inflammation index** = stronger and/or more persistent NF-κB exposure
- **Lower inflammation index** = weaker and/or better-resolved inflammatory signaling

This is a **proxy metric**, not a direct experimental measurement of cytokines or neuronal injury. It is intended as a model-based summary of inflammatory burden.

---

## TNF stimulation protocol used in the notebook

The notebook simulates a transient TNF pulse by changing the model parameter `TNF_R` in two phases:

- `TNF_R = 1.0` from time 0 to the specified pulse-off time
- `TNF_R = 0.0` from the pulse-off time to the end of the simulation

In the inflammation-index scans, the default settings are:

- Simulation end time: `6500` seconds
- Number of time points: `4001`
- TNF pulse turned off at: `3600` seconds
- Plateau tail fraction: `0.10`

---

## Heat maps generated in the notebook

The notebook creates inflammation-index heat maps for these parameter pairs:

### Figure 3
- `c1` vs `k_cyld`
- `k2` vs `k_cyld`

### Figure 4
- `c4` vs `k_cyld`
- `k2` vs `k_cyld` for comparison

### Figure 5
- `c5` vs `k_cyld`
- `k2` vs `k_cyld` for comparison

### How to read them

- x-axis: `log10(k_cyld fold-change)`
- y-axis: `log10(parameter fold-change)`
- color: inflammation index

In general:

- Movement toward lower color values means lower predicted inflammatory burden
- Strong horizontal gradients indicate a large effect of `k_cyld`
- Strong vertical gradients indicate a large effect of the A20-related parameter on the y-axis
- Diagonal gradients indicate that both parameters contribute substantially

---

## Expected qualitative trends

Based on the notebook logic and generated figures, the scans are designed to compare how different regulatory layers affect inflammatory burden:

- `k_cyld` often acts as a strong global suppressor because it limits upstream pathway activation
- `k2` tends to have a strong effect because it changes the effective strength of delayed inhibitory feedback
- `c4` can produce a substantial effect because it changes how much A20 protein is actually produced
- `c1` may show a more modest or saturating effect because transcription alone does not guarantee strong downstream inhibition
- `c5` may have weaker impact because changing protein degradation alone may not strongly reshape total pathway output within the scanned range

These interpretations should always be checked against the actual heat map color scales in each panel.

---

## How to run the notebook locally

### 1. Clone the repository

```bash
git clone https://github.com/1Riya-Shukla/NF-kB-Pathways.git
cd NF-kB-Pathways/MicroglialRegulations
```

### 2. Create a Python environment

Example with `venv`:

```bash
python -m venv .venv
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 3. Install required packages

At minimum, the notebook uses:

- `python-libroadrunner`
- `numpy`
- `matplotlib`
- `jupyter`

Example install command:

```bash
pip install libroadrunner numpy matplotlib jupyter
```

If `libroadrunner` is unavailable in your setup, install the package that provides the Python `roadrunner` module for your platform.

### 4. Make sure the XML file is in the same directory

The notebook loads:

```python
rr = roadrunner.RoadRunner("LipniackiWithCYLD.xml")
```

So `LipniackiWithCYLD.xml` must be in the same folder as `RoadRunner-Lipniacki.ipynb` when you run the notebook.

### 5. Launch Jupyter

```bash
jupyter notebook
```

Then open:

- `RoadRunner-Lipniacki.ipynb`

### 6. Run cells in order

Run the notebook from top to bottom so that:

- The model is loaded first
- Helper functions are defined
- The figures are generated in sequence

---

## Running the simulation from a script instead of the notebook

If you want to reuse the logic in a Python script, the basic workflow is:

1. Load the model with RoadRunner
2. Reset the model before each simulation
3. Apply parameter updates such as `c1`, `c5`, `c4`, `k2`, or `k_cyld`
4. Simulate the TNF pulse in two segments
5. Extract the `NFKB_nuc` trajectory
6. Compute the inflammation index
7. Repeat across a fold-change grid
8. Plot the resulting heat map

A minimal example structure is:

```python
import roadrunner
import numpy as np

rr = roadrunner.RoadRunner("LipniackiWithCYLD.xml")

# update parameters
rr.resetAll()
rr["c1"] = 1.0
rr["k_cyld"] = 10.0

# simulate TNF on
rr["TNF_R"] = 1.0
sim_on = rr.simulate(0, 3600, 2001, ["time", "NFKB_nuc"])

# simulate TNF off
rr["TNF_R"] = 0.0
sim_off = rr.simulate(3600, 6500, 2001, ["time", "NFKB_nuc"])
```

You would then stitch the two segments, compute the inflammation index, and store the result.

---

## How to cite or refer to the inflammation index in a paper or report

### Short version

> The inflammation index was computed from the simulated nuclear NF-κB trajectory as a weighted combination of mean exposure above baseline over the full simulation and late-time plateau above baseline.

### More detailed version

> To summarize inflammatory burden from each simulated NF-κB trajectory, an inflammation index was calculated by combining cumulative NF-κB exposure and persistent late-time activity. Specifically, the area under the NF-κB curve above baseline was divided by the total simulation duration to obtain mean exposure above baseline, and the plateau above baseline was estimated from the final 10 percent of the trajectory. The final inflammation index was defined as 0.70 times the mean exposure plus 0.30 times the late-time plateau.

### Plain-language version

> The inflammation index summarizes both how much NF-κB activity occurs overall and how much remains late in the response, making it a useful proxy for inflammatory burden.

---

## Suggested wording for limitations

If you describe this metric in a manuscript, it is helpful to note that:

- It is a model-derived proxy, not a direct experimental assay
- The weights 0.70 and 0.30 are chosen to emphasize cumulative exposure while still retaining sensitivity to persistence
- Different weighting schemes could be tested in future work
- The metric is useful for comparing simulated conditions within this model framework

Example:

> The inflammation index is a model-based proxy for inflammatory burden and does not directly represent cytokine secretion or tissue damage. Its value lies in providing a consistent comparative measure across simulated parameter regimes.

---

## Ways to extend the notebook

You can adapt the notebook by:

- Changing the TNF pulse duration
- Changing the simulation length
- Changing the inflammation index weights
- Using a different plateau tail fraction
- Scanning more parameters at once
- Saving the heat map arrays to disk
- Exporting figures at higher resolution for publication

Examples:

- Increase `simulation_end_time_seconds` to test longer inflammatory persistence.
- Change `weight_mean_exposure` and `weight_late_exposure` inside `compute_inflammation_index`.
- Increase `num_grid_points` for smoother heat maps.

---

## Common troubleshooting

### File not found

If you get an error when loading the model, check that:

- `LipniackiWithCYLD.xml` is in the same folder as the notebook
- You launched Jupyter from the correct directory

### RoadRunner import error

If Python cannot import `roadrunner`, install the appropriate RoadRunner package for your operating system and Python version.

### Flat or unexpected heat maps

Check that:

- The correct parameter names exist in the XML model
- The parameter updates are being applied before simulation
- `TNF_R` is being switched on and off correctly
- The selected output species is `NFKB_nuc`

### Different values than expected

Small changes in software versions, solver behavior, or XML model edits can change exact heat map values. Focus on whether the qualitative trends are preserved.

---

## Folder expectation

This README assumes the following structure:

```text
MicroglialRegulations/
├── RoadRunner-Lipniacki.ipynb
├── LipniackiWithCYLD.xml
└── README.md
```

---

## Contact or reuse note

If you reuse the notebook in another repository, preserve the model file path assumptions or update the XML loading line accordingly.
