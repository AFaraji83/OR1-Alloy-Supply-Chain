# Alloy Production & Distribution Planning (LP/MILP)

Course project for **Operations Research 1** at the Department of Industrial Engineering, Sharif University of Technology (Fall 2024).

A factory makes two alloys, A and B, from metals extracted from three ores. It can also buy finished alloys from two other factories. All alloys go by container to warehouses in Tehran and Esfahan, then on to customers in four cities. The goal is to **maximise profit**: decide how much ore to mine, how much alloy to make or buy, and how to route the containers. The problem is modelled as a mixed-integer linear program and solved in Python with Pyomo, followed by a set of sensitivity analyses.

## Project phases

| Phase | What it covers | Files |
|---|---|---|
| 1. Modeling | Decision variables, objective and constraints for the base model (part a) and the bulk-discount model (part b) | [`phase1-modeling/`](phase1-modeling) |
| 2. Solving & analysis | Pyomo implementation, optimal solutions and sensitivity analysis (parts c to i) | [`phase2-python/`](phase2-python) |

The reports (PDF, written in Persian) are in each phase folder.

## Problem description

**Production.** Three ores are mined. Each has a fixed metal content, a limited supply and an extraction cost:

| Ore | Zinc | Iron | Copper | Aluminium | Other | Max available (t) | Cost per t |
|---|---|---|---|---|---|---|---|
| 1 | 25% | 5% | 30% | 35% | 5% | 560 | 45 |
| 2 | 15% | 20% | 25% | 30% | 10% | 1000 | 65 |
| 3 | 5% | 5% | 65% | 25% | 0% | 1440 | 70 |

The alloys must meet these specs. Converting 1 t of metal into 1 t of alloy costs 250.

| Alloy | Specification |
|---|---|
| A | at most 80% zinc, at most 25% iron, at least 55% aluminium |
| B | 45 to 70% iron, at least 35% copper, at most 70% aluminium |

**Buying.** Alloys can also be bought from two other factories. Each one has a fixed contract cost and a minimum and maximum total order:

| Factory | Contract cost | Price A | Price B | Order range (t) |
|---|---|---|---|---|
| 1 | 120 | 375 | 520 | 2000 to 5000 |
| 2 | 90 | 390 | 540 | 2500 to 6000 |

In part (b), each factory gives a **5% discount** on both alloys once the total order exceeds 2500 t (factory 1) or 3000 t (factory 2).

**Shipping.** Each container holds 100 t of alloy.

- **Factories to warehouses:** every factory-to-warehouse route has its own cost per container and a minimum and maximum number of containers. Each month Tehran must receive 20 to 65 containers and Esfahan 30 to 70.
- **Warehouses to customers:** containers then travel to Mashhad, Kerman, Ahvaz and Tabriz. Each route has its own cost and container limits.
- **Customers:** each city pays its own price per tonne of each alloy and has a maximum demand.

## Model summary

| Variable | Meaning |
|---|---|
| `x[t,i]` | tonnes of metal *t* obtained from ore *i* |
| `l[t,j]`, `y[j]` | tonnes of metal *t* used in alloy *j*; tonnes of alloy *j* produced |
| `q[j,w]`, `f[w]` ∈ {0,1} | tonnes of alloy *j* bought from factory *w*; contract signed with *w* |
| `h[j,λ,k]` ∈ ℤ⁺, `n[j,λ,k]` | containers and tonnes of alloy *j* from factory *λ* to warehouse *k* |
| `d[j,k,e]` ∈ ℤ⁺, `m[j,k,e]` | containers and tonnes of alloy *j* from warehouse *k* to city *e* |
| `δ[w]` ∈ {0,1}, `z` | discount indicator and its linearisation (part b) |

**Objective:** sales revenue minus the cost of extraction, conversion, contracts, purchases and both shipping legs.

**Constraints:**

- ore composition (each ore yields metals in fixed proportions)
- alloy specifications
- metal balance between ores and alloys
- order limits tied to the contract binaries
- container limits on both shipping legs
- container capacity: 100 t per container
- flow balance at the warehouses
- city demand

The discount is linearised with a binary per factory and big-M product terms.

## Results

| Scenario | Optimal profit |
|---|---|
| Base model | **1,285,190** |
| With 5% bulk discount | **1,477,640** |

In both cases the optimal plan **buys all alloys from the two factories and produces nothing in-house**. Factory 1 supplies 800 t of A and 4200 t of B, and factory 2 supplies 3500 t of A. Both orders are large enough to qualify for the discount, which is why the routing stays the same and only the purchase cost changes.

**Sensitivity analyses (parts d to i):**

- maximum conversion cost at which in-house production would still be chosen
- maximum contract cost for each factory before buying from it stops being worthwhile
- ranges of the purchase prices of A and B from each factory
- the effect of one objective coefficient (a city's selling price) and one right-hand side (a city's demand) on profit, with plots
- ±10% changes in the transport costs of tables 4 and 5, with plots, to see which estimate matters most
- adding Abadan as a new sales city and finding the minimum price that makes selling there worthwhile

## Repository structure

```
.
├── phase1-modeling/
│   └── Phase1_Report.pdf             # mathematical model (parts a and b)
└── phase2-python/
    ├── alloy_supply_chain.ipynb      # Pyomo model, solutions and sensitivity analysis
    └── Phase2_Report.pdf             # final report
```

## How to run

```bash
pip install -r requirements.txt
```

The notebook uses **GLPK** (`conda install -c conda-forge glpk`, or `apt install glpk-utils` on Linux). To use **HiGHS** instead, replace `SolverFactory('glpk')` with `SolverFactory('appsi_highs')`. HiGHS needs no extra setup and gives the same optimal profits.

Then open `phase2-python/alloy_supply_chain.ipynb` in Jupyter and run the cells in order. All data is defined inside the notebook.

## Tools

Python · Pyomo · GLPK · matplotlib · Jupyter

## Team

- Alireza Faraji
- Amirhossein Shahrabi
- Mohammad Erfan Kalmatian
- Nima Bagherinejad
