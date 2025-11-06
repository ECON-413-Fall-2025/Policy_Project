[README.md](https://github.com/user-attachments/files/23394266/README.md)
# Project 2: Policy Lab (ECON 413)

**Presentation:** Tuesday, December 2 (in class, 10 minutes per team)  
**Deliverables:** GitHub repo + slide PDF (uploaded before class)

This repository is your working space for building and analyzing a DSGE or RBC-variant model using gEcon. You will:

1. Choose one sample model from the gEcon "Sample models" page (do not use the basic RBC).
2. Calibrate parameters (no estimation), solve the model, and generate IRFs for a shock of your choice.
3. Explain in your slides:
   - Why you chose this model relative to the baseline RBC
   - One new parameter not in the baseline RBC (its meaning and your calibrated value)
   - Key IRFs and what they show about your chosen shock

---

## Repository structure

```
policy-lab-yourteam/
model/               # .gcn and any companion files for your chosen model
data/                # optional (e.g., any auxiliary CSV you bring in)
code/
    run_all.R        # main script: calibrate -> solve -> IRFs
    helpers.R        # optional utilities you write
plots/               # IRF figures are saved here
README.md            # this file
```

Keep paths relative to the repo root (use file.path(...) in R). Do not commit large auto-generated logs.

---

## Prerequisites

- R (recent version) and RStudio or R terminal
- gEcon package (installed from R-Forge)
- Git/GitHub access (you will push commits to this repo)

If `install.packages("gEcon", repos = "http://R-Forge.R-project.org")` fails on your machine, try again from RStudio, or switch to a lab or RStudio-Cloud environment where installation succeeds.

Links:  
- gEcon user's guide: https://gecon.r-forge.r-project.org/files/gEcon-users-guide.pdf  
- Sample models: https://gecon.r-forge.r-project.org/models.html

---

## Quickstart (copy-paste into R)

Open the project in RStudio (or set your working directory to the repo root) and run:

```r
# ---- Install required packages quietly and load them ----
req <- c("MASS","Matrix","Rcpp","nleqslv")
if (is.null(getOption("repos")) || getOption("repos")["CRAN"] %in% c(NA,"@CRAN@")) {
  options(repos = c(CRAN = "https://cran.r-project.org"))
}
for (p in req) if (!requireNamespace(p, quietly = TRUE)) install.packages(p, quiet = TRUE)
if (!requireNamespace("gEcon", quietly = TRUE)) {
  install.packages("gEcon", repos = "http://R-Forge.R-project.org", quiet = TRUE)
}
suppressPackageStartupMessages({
  library(gEcon)
  library(MASS); library(Matrix); library(Rcpp); library(nleqslv)
})

# ---- Set your model and IRF preferences (EDIT THESE) ----
model_file <- file.path("model","rbc_hf.gcn")  # example: habit-formation model
irf_vars    <- c("Y","C","I","K_s","L_s")      # edit to match your model
irf_horizon <- 40                               # IRF length in periods
shock_name  <- "epsilon_Z"                      # edit to a shock that exists in your model

# ---- Load model ----
mod <- make_model(model_file)

# ---- Calibrate at least one parameter NOT in the basic RBC (example; edit or delete) ----
# Check the sample model's documentation for parameter names
# mod <- set_free_par(mod, list(h = 0.70))

# ---- Steady state and linear solution ----
mod <- steady_state(mod)
mod <- solve_pert(mod, loglin = TRUE)
check_bk(mod)  # view BK condition counts in console

# ---- Set shock variance (edit to your chosen shock) ----
mod <- set_shock_distr_par(
  model = mod,
  distr_par = setNames(list(0.10), sprintf("sd(%s)", shock_name))  # sd = 0.10 example
)

# ---- Compute and save IRFs ----
dir.create("plots", showWarnings = FALSE)
irf_obj <- compute_irf(model = mod, variables = irf_vars, sim_length = irf_horizon)
plot_simulation(irf_obj, to_eps = TRUE)  # saves to a /plots subfolder
```

Next: open the files gEcon created in `plots/` and check your figures. Make sure axes are labeled and the shock name and size appear in the caption on your slides.

---

## Choosing your model

Pick one model from the sample list (do not use `rbc.gcn`). Examples:

- `rbc_ic.gcn` (investment adjustment costs)  
- `rbc_cu.gcn` (capacity utilization)  
- `rbc_hf.gcn` (habit formation)  
- `rbc_mc.gcn` (monopolistic competition)  
- `rbc_ts.gcn` (two sectors)  
- `home_production.gcn`  
- `ttb.gcn` (time to build)  
- `tc_rs.gcn` (two-country risk sharing)  
- `SW_03.gcn` (Smets-Wouters 2003 NK, for policy shocks)

Put your chosen `.gcn` (and any companion files) into the `model/` folder. Update `model_file` in your script.

---

## What to edit in code/run_all.R

1. Model path: set `model_file <- file.path("model","your_model.gcn")`.
2. IRF variables: set `irf_vars <- c(...)` to valid model variable names.
   - To discover names, run in R: `var_info(mod)` after `make_model(...)`.
3. Shock: set `shock_name <- "..."`.
   - Discoverable via `shock_info(mod, all = TRUE)`.
4. Calibration: choose at least one parameter not in the basic RBC, set a value, and justify it in your README and slides.
   - Example: `mod <- set_free_par(mod, list(h = 0.70))` in a habit-formation model.
5. IRF horizon: default `irf_horizon <- 40` periods. Adjust if needed.

---

## Expected outputs

- IRF figures in `plots/` generated by `plot_simulation(...)`.
- Console messages confirming steady state, solution, and BK conditions.

Optional: you may compute model moments for your README.

```r
# Stats for README (optional)
# mod <- compute_model_stats(model = mod, n_leadlags = 6, ref_var = "Y")
# get_model_stats(mod)
```

---

## Slide requirements (10 minutes)

Your PDF should include:

1. Model and motivation: why this model vs. baseline RBC.
2. New parameter: define it, state your calibrated value, and explain its effect on dynamics.
3. IRFs: 3-6 panels (e.g., Y, C, I, K, L, and if applicable inflation/rate), horizon >= 40, labeled axes, caption showing shock name and size.
4. Takeaways: what the new feature buys; one limitation or next step.

No live coding during the presentation.

---

## Rubric alignment (short version)

- Repo and code (50 pts): runnable `run_all.R`; correct load/steady state/solve; shock set; IRFs saved and labeled; one non-RBC parameter calibrated; clean structure and comments.
- Presentation (50 pts): model choice and RBC contrast; new parameter explanation; IRFs and interpretation; calibration transparency; timing; Q&A.

See the full rubric document posted with the assignment.

---

## Troubleshooting

- Steady state fails: provide better guesses with `initval_var()` and `initval_calibr_par()`, then re-run `steady_state()`.
- BK check fails: inspect `check_bk(mod)` output; revisit timing assumptions or extreme parameter values.
- IRFs do not save: ensure `dir.create("plots", showWarnings = FALSE)` ran and `plot_simulation(irf_obj, to_eps = TRUE)` was called.
- Unknown variable or shock names: use `var_info(mod)`, `par_info(mod)`, and `shock_info(mod, all = TRUE)` after `make_model(...)`.

---

## Team workflow

- Use short, descriptive commit messages (e.g., `add model file`, `tune habit parameter`, `save IRFs`).
- Keep large, auto-generated artifacts out of Git.
- One teammate should be able to clone fresh and run `code/run_all.R` end-to-end.

---

## Academic integrity

Discuss ideas within your team. Cite any parameter values you take from papers or documentation in your slide notes or README.

---

## License and acknowledgements

- This coursework repository is for ECON 413 only.
- gEcon is developed by its authors and distributed via R-Forge. Please cite gEcon and any model sources you use.
