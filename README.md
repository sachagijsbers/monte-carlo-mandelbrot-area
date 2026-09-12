# Monte Carlo Estimation of the Mandelbrot Set Area

**Comparing random, Latin hypercube, orthogonal and importance sampling for estimating the area of a fractal.**

<p align="center">
  <img src="figures/zoom_final.png" width="620" alt="Zoom into the boundary of the Mandelbrot set">
</p>

The Mandelbrot set has no closed-form area, and its infinitely detailed boundary makes it a good stress test for Monte Carlo integration. This project estimates the area with four sampling strategies, studies how the estimate converges in the number of iterations and the number of samples, and compares the methods statistically over 1000 independent runs.

`Python` · `NumPy` · `SciPy (stats, qmc)` · `Matplotlib` · `Jupyter`

---

## Approach

A point $c \in \mathbb{C}$ belongs to the set if $z_{n+1} = z_n^2 + c$, starting from $z_0 = 0$, stays bounded. In practice a point counts as inside when $|z_n| \le 2$ for all $i$ iterations. Sampling $s$ points uniformly in $[-2, 2] \times [-2, 2]$ gives the estimate

$$A_{i,s} = 16 \cdot \frac{\#\{\text{samples inside}\}}{s}$$

- **Convergence study:** the error $|A_{j,s} - A_{i,s}|$ as a function of the iteration count $j$ and sample size $z$ (with 95% confidence intervals over 1000 runs) is used to choose $i$ and $s$ that balance accuracy and cost.
- **Sampling methods:**
  - pure random sampling
  - Latin hypercube sampling (`scipy.stats.qmc.LatinHypercube`)
  - orthogonal sampling (strength-2 LHS, which needs sample sizes that are squares of primes, e.g. 289 = 17²)
- **Importance sampling:** the plane is split into an inner block, a boundary block and an outer block, and sampling probabilities are reweighted (0.1 / 0.7 / 0.2 instead of 0.04 / 0.44 / 0.52 by area).
- **Reference and testing:** pixel counting on a 10 000 × 10 000 grid ($A \approx 1.525$), one-sample and two-sample t-tests.

## Results

<p align="center">
  <img src="figures/importance_sampling_iterations_2.png" width="480" alt="Error versus maximum number of iterations for the four sampling methods">
</p>

| Method (i = 200, s = 289, 1000 runs) | Mean area | Std. dev. | vs. pixel counting |
|---|---|---|---|
| Random | 1.514 | 0.280 | not significant (p = 0.151) |
| Latin hypercube | 1.508 | 0.264 | p = 0.029 |
| Orthogonal | 1.504 | 0.264 | p = 0.008 |
| Importance | 1.586 | – | p < 0.001 |

- The error stops improving meaningfully beyond about **200 iterations and 200 samples**.
- The random, Latin hypercube and orthogonal estimates do **not differ significantly** from each other. Sampling the full $[-2,2]^2$ square spends most points far from the boundary, where the choice of sampling method matters little.
- **Importance sampling converges in far fewer iterations** (j ≈ 36 vs. 106–136) and cuts the variance across iterations by roughly **10×**. However, it overestimates the area (1.586), so it trades bias for speed with this block design.
- The pixel-counting reference is itself biased at the fractal boundary, which may explain why the stratified methods differ from it.

The full write-up, including methods, figures and discussion, is in [`report.pdf`](report.pdf).

## Repository structure

```text
code/
  code.ipynb                     Mandelbrot visualisation, convergence study, sampling methods, t-tests
  importance_sampling_code.ipynb Importance sampling and convergence comparison of all four methods
figures/                         All generated figures (not all are used in the report)
ortho-pack/                      Reference C implementation of orthogonal sampling (not used by the notebooks)
report.pdf                       Report
```

## Getting started

```bash
pip install -r requirements.txt
jupyter lab code/
```

Some convergence experiments run 1000 repetitions per setting and take several minutes.

## Team

Group project for the *Stochastic Simulation* course (2023) by **Esther Bakels**, **Loes Bijman** and **Sacha Gijsbers**.
