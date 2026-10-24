# Berth Allocation and Scheduling Optimization

A three-phase Operations Research II course project on assigning ships to berths and scheduling unloading operations using mathematical optimization, Python, and GAMS.

The project considers **8 ships and 4 berths**. It combines berth assignment, service sequencing, ship waiting times, and additional berth capacity in a cost-minimization model. Sensitivity analyses investigate how changes in operational parameters affect the objective value and allocation decisions.

**Tools:** Python, Pyomo, GLPK, gurobipy, GAMS, pandas, NumPy, Matplotlib, SciPy, and Excel.

> **Reproducibility status:** The original input workbook, `Parameters.xlsx`, is available with the project materials, including all worksheets referenced by the Python code. The notebook retains its original Colab workflow. A fresh end-to-end solver run has not been verified as part of preparing this documentation. Python and GAMS implementation differences are described below.

## Project overview

Ports must allocate limited berth resources while accounting for vessel characteristics, unloading times, arrival times, and operational restrictions. This project asks:

- Which berth should serve each ship?
- In what order should ships assigned to the same berth be served?
- How long will each ship wait before unloading begins?
- How much additional berth capacity should be used?
- How do changes in arrivals, unloading times, and berth capacities affect the solution?

The objective combines unloading costs, waiting-related costs, and additional-capacity costs. The model uses binary assignment and sequencing decisions, together with waiting-time and capacity variables. Auxiliary variables represent waiting-cost terms involving binary and continuous decisions.

This is a small academic case study, rather than a calibrated operational port planning system. Time and cost values should be interpreted in the units specified by the original assignment and reports.

## Project phases

| Phase | Focus | Main deliverables |
| --- | --- | --- |
| 1 | Mathematical formulation | Parameters, decision variables, objective, constraints, and linearization discussion |
| 2 | Python implementation | Pyomo/GLPK and gurobipy implementations, solution summaries, visualizations, and sensitivity analysis |
| 3 | GAMS implementation | Base optimization model, sensitivity-analysis scripts, saved results, and final report |

The reports are written in Persian. Code and many output labels are in English.

## Model structure

### Inputs

The model uses ship container quantities, ship lengths, arrival times, priority parameters, berth capacities, maximum supported ship lengths, ship–berth unloading times, service lags, additional-capacity costs and limits, and berth assignment-count restrictions.

### Decisions

| Variable | Meaning |
| --- | --- |
| `x(i,j)` | Whether ship `i` is assigned to berth `j` |
| `y(i,k)` | Pairwise service-order decision used in sequencing constraints |
| `W(i)` | Waiting time of ship `i` |
| `u(j)` | Additional capacity used at berth `j` |
| `delta1(i)`, `delta2(i)` | Binary indicators used in threshold-based waiting costs |
| `d1(i)`, `d2(i)` | Auxiliary variables for waiting-cost terms |

### Objective and constraints

The implemented cost expression is:

```text
Total cost = unloading cost + waiting-related cost + additional-capacity cost

Unloading cost = sum(i,j) 5 * t(i,j) * x(i,j)

Waiting-related cost = sum(i) [
    25 * W(i)
    - 15 * d1(i) + 150 * delta1(i)
    -  5 * d2(i) + 200 * delta2(i)
]

Additional-capacity cost = sum(j) C_ec(j) * u(j)
```

The formulations include one-berth-per-ship assignment, ship-length compatibility, berth capacity and additional-capacity limits, assignment-count restrictions, priority-related conditions, sequencing constraints, waiting-time bounds, and auxiliary waiting-cost constraints.

The base Python model includes an exactly-three-ships restriction for berth 3. Phase 2 also explores relaxing this restriction. Results must be interpreted against the particular model variant being solved.

## Repository files

Keep the notebook, input workbook, and GAMS scripts in the repository root. This preserves the Python input filename and avoids introducing new code paths. Store reports and saved results in the folders shown below.

| Location | Purpose |
| --- | --- |
| `README.md` | Project overview and execution instructions |
| `Parameters.xlsx` | Original instructor-provided input data |
| `OR_Project_phase_II.ipynb` | Python implementation, visualizations, and sensitivity analyses |
| `OR2-PRJ-Phase3-Model.gms` | Base GAMS model |
| `1.SA_eps2.gms` | Sensitivity to service lag at berth 2 |
| `2.SA_t73.gms` | Sensitivity to unloading time of ship 7 at berth 3 |
| `3.SA_ca2.gms` | Sensitivity to capacity at berth 2 |
| `4.SA_ca3.gms` | Sensitivity to capacity at berth 3 |
| `5.SA_A6.gms` | Sensitivity to arrival time of ship 6 |
| `reports/phase1_formulation.pdf` | Mathematical formulation and linearization report |
| `reports/phase2_python.pdf` | Python implementation and sensitivity-analysis report |
| `reports/phase3_gams.pdf` | GAMS implementation and sensitivity-analysis report |
| `results/berth_scheduling_results_model_p.xlsx` | Saved Python solution tables |

The report filenames above are publication-friendly names for the original phase reports. Renaming the reports does not require changing model code.

### Assignment source and correction

The problem and numerical input data were provided by the course instructor. The original assignment, `Project.pdf`, describes the three phases and the berth scheduling problem. The companion document, `اصلاحیه.pdf`, revises the waiting-cost function and specifies an unloading cost of 5 units per unit of unloading time. The correction must be read together with the original statement.

The corrected waiting-cost schedule is:

| Waiting time | Waiting cost |
| --- | --- |
| 0 to 10 | `25 * W` |
| 10 to 40 | `10 * W + 150` |
| 40 to 70 | `5 * W + 350` |

The pieces agree at their shared endpoints. The correction explicitly states the final interval up to 70; the code also applies priority-dependent waiting bounds. Any use beyond the stated cost domain should be checked against the assignment requirements.

If the instructor's documents are included in the published repository, store them as `assignment/project_statement.pdf` and `assignment/project_corrections.pdf`. They are optional supporting documents, and their absence does not prevent running the code. The summary here identifies their role without requiring that they be publicly redistributed.

### Original files retained outside the initial publication

- `or_project_phase_ii.py`: original Colab export. It is not included in the initial publication because it requires cleanup for standalone execution. The notebook contains the Python workflow.
- `1.SA_eps2_output.csv.xlsx`, `2.SA_t73_output.csv.xlsx`, `3.SA_ca2_output.csv.xlsx`, `4.SA_ca3_output.csv.xlsx`, and `5.SA_A6_output.csv.xlsx`: historical GAMS sensitivity workbooks. Their split assignment headers should be corrected before publishing them under `results/`.

The sensitivity scripts and the analysis in the phase reports remain available even when these historical exports are omitted.

## Running the Python notebook

### Requirements

The notebook was developed in Google Colab and contains environment-specific installation and download commands. Local Jupyter use requires adapting those cells.

The Python package dependencies visible in the code can be installed with:

```bash
python -m pip install pandas numpy pyomo gurobipy matplotlib scipy tabulate openpyxl jupyter
```

This installs Python packages only. **GLPK must be installed separately** and its executable must be available to Pyomo. The gurobipy sections require access to a suitable Gurobi installation/license for the model being solved. Package versions have not yet been pinned to a newly verified execution environment.

### Required input workbook

Before executing the model, place the supplied `Parameters.xlsx` in the notebook's working directory. The original code reads the filename directly, without a `data/` prefix.

The required worksheet names are:

| Worksheet | Input |
| --- | --- |
| `Containers` | Ship container quantities |
| `Capacity` | Berth capacities |
| `Length` | Ship lengths |
| `Max Length` | Maximum supported ship lengths |
| `Unload Time` | Ship–berth unloading times |
| `Priority` | Ship priority parameters |
| `Cost for Extra Capacity` | Additional-capacity costs |
| `Max Extra Allowable Capacity` | Additional-capacity limits |
| `Arrival Time` | Ship arrival times |
| `Lag` | Service lags at berths |
| `Anchorage` | Berth assignment-count parameters |

The supplied workbook contains all 11 referenced worksheets and an additional `unload` worksheet with a matrix-style view. Its headers match the input fields inspected in the Python source. Column names and identifier formats must remain consistent with the `read_excel` and dictionary-construction cells. The saved solution workbook is an output, not a substitute for this input file.

### Execution

**Google Colab**

1. Open `OR_Project_phase_II.ipynb` in Colab.
2. Upload the original `Parameters.xlsx` to the runtime working directory.
3. Run the installation and setup cells.
4. Execute the notebook in order, inspecting solver status before interpreting variable values.
5. Download any generated results before the runtime session ends.

**Local Jupyter**

1. Install the packages above and GLPK.
2. Open a terminal in the directory containing the notebook and input workbook.
3. Launch Jupyter with `jupyter notebook`.
4. Adapt Colab-specific installation and `google.colab.files` download cells to the local environment.
5. Run the notebook from a fresh kernel in cell order.

The exported `.py` file contains notebook shell commands such as `!pip install` and `!apt-get`. Running it directly with `python or_project_phase_ii.py` will require prior cleanup.

## Running the GAMS models

The GAMS scripts contain their numerical input parameters directly in the source and do not read `Parameters.xlsx`.

1. Install GAMS with access to an appropriate mixed-integer optimization solver.
2. Open `OR2-PRJ-Phase3-Model.gms` in GAMS Studio.
3. Run the model and inspect solver status, model status, and the generated listing.
4. Run the five sensitivity scripts separately to generate their CSV outputs.

For an environment where the `gams` executable is available, the base model can also be launched with:

```bash
gams OR2-PRJ-Phase3-Model.gms
```

Each sensitivity script specifies its output filename. Execute scripts in a writable working directory and retain the corresponding model and solver statuses with any results used for comparison.

## Results and sensitivity analysis

The saved Python workbook contains the following sheets:

| Sheet | Content |
| --- | --- |
| `Assignments` | Ship-to-berth assignment results |
| `WaitingTimes` | Waiting time by ship |
| `ExtraCapacity` | Additional capacity used by berth |
| `AllVariables` | Detailed model variable values |

The sensitivity studies vary five inputs:

| Parameter | Operational interpretation |
| --- | --- |
| `epsilon(2)` | Time lag between services at berth 2 |
| `t(7,3)` | Unloading time for ship 7 at berth 3 |
| `ca(2)` | Capacity of berth 2 |
| `ca(3)` | Capacity of berth 3 |
| `A(6)` | Arrival time of ship 6 |

The reports discuss the corresponding objective-value patterns. These experiments describe responses within this model and parameter ranges; they do not establish a general operational recommendation for real ports.

## Known limitations and reproducibility notes

- **Input data supplied:** `Parameters.xlsx` is available. Worksheet/header inspection has been completed; this does not substitute for a fresh execution of the notebook.
- **Environment-specific code:** The notebook and exported script contain Colab-specific commands.
- **Different model variants:** Additional capacity is continuous in the inspected Pyomo implementation and integer in the GAMS model. Big-M constants also differ across implementations. Numerical agreement should not be assumed without checking data, variable domains, constraints, and solver settings.
- **CSV header formatting:** Some GAMS output headers contain unquoted commas in labels such as `x(1,1)`. In the supplied workbooks, these labels are split across columns. Use delimiter-safe labels such as `x_1_1`, or correctly quote CSV fields, before regenerating clean exports. The original workbook headers should not be used to map assignment columns without validation.
- **Saved outputs:** Existing workbooks and notebook outputs record earlier runs. They are not evidence that the current source executes unchanged in a fresh environment.
- **Validation scope:** No new solver-based verification is claimed here. A full reproducibility check should use the supplied input workbook, run each intended model variant, verify feasibility and termination status, and reconcile reported objective components with the resulting variable values.

## Academic context and credits

Developed as a group project for **Operations Research II**, Department of Industrial Engineering, **Sharif University of Technology**, during the second semester of academic year **1403–1404**.

**Team members, as listed in the original reports:**

- Ava Sedighi
- Pardis Rahiminasr
- Shideh Parastari

**Course instructor:** Dr. Maryam Radman



