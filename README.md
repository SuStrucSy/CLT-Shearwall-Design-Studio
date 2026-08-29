# CLT Shearwall Design Studio

An interactive companion tool for the manuscript *Design Space Exploration and
Archetype Development for CLT Shearwalls under Capacity-Based Design*.
Given a wall archetype, it runs the paper's capacity-based design
pipeline and returns **every feasible connection configuration per story** with
its performance metrics, lets you explore, compare and select configurations,
and exports the chosen design to Excel, PDF and a printable summary.

> ### Try it now online
>
> **https://clt-shearwall-studio.onrender.com/**
>
> The full application runs in your browser. The first load may take up to a
> minute
>

| Step | Page | Manuscript module |
|------|------|-------------------|
| — | Overview | the framework, bundled data, saved sessions |
| 1 | Seismic hazard | Module 2: NBCC 2020/2025 design spectrum Sa(T, X) |
| 2 | Wall archetype | Modules 1 & 3: archetype, CLT catalogue, connection pool |
| 3 | Demand & panel check | Modules 2 & 4: equivalent-static demand, gravity screening |
| 4 | Connection configurations | Module 4: every feasible story layout |
| 5 | System chain | Module 5: inter-storey continuity (Cl. 11.9.3.6.4), EDPs, per-storey editor |
| 6 | Export & report | Excel · PDF · printable summary · session file |

The design engine reuses the manuscript modules **unchanged**, so
results are identical to the batch pipeline.

---


## Workflow

**Overview (landing page)** : what the framework does, the five modules it
implements, and the bundled data.Saved sessions are opened from here (or from *Session* in the app
bar).

1. **Seismic hazard (NBCC 2020 / 2025 tool)** : the design spectral acceleration
   curve Sa(T, X) is fetched live from the Natural Resources Canada Seismic
   Hazard Tool for your site, and plotted with a marker at the current
   archetype's fundamental period. Manual entry is available offline.
2. **Wall archetype** : occupancy, height class, seismic category, ductility,
   storeys, storey height, gravity load *q*, panel width *b_s*, panels *m*, the
   **CLT catalogue → layup** (managed catalogues, see below), and the
   **connection source** (representative vs whole database). A live summary panel
   shows total height, wall width *B*, derived aspect ratio *H/b_s*, estimated
   *Ta* and the number of candidate combinations that will be screened.
3. **Demand & panel check** : fundamental period *Ta*, base shear *V*, importance
   factor, storey shear/overturning-moment profile, the design spectrum used, and
   the CSA O86:24 gravity panel check. 
4. **Connection configurations** : one tab per story with a sortable/filterable
   table of every feasible WWS/WFT/WFS layout. Selecting a row shows a
   **performance radar** (strength / stiffness / Ω / deformation, normalised to
   the storey maxima) and **connection detail cards** with the tri-linear F–δ
   backbone and an expander of every database field + source-paper DOI link.
   **Pin up to four candidates** to compare them side by side on one radar and in
   a metric table. *Auto-assemble* runs the Module 5 depth-first search for a
   continuous chain satisfying the inter-story continuity bounds on the
   over-capacity coefficient (CSA O86:24 Cl. 11.9.3.6.4) and reports the
   Engineering Demand Parameters.
5. **System chain** : an embedded, scaled wall elevation beside live outputs for
   the current chain: per-story performance metrics (Ω, Kw, Rwr, SBMS, story
   drift), the inter-story **continuity check** (Ω-ratio 0.9–1.2,
   CSA O86:24 Cl. 11.9.3.6.4) with pass/fail badges, and **height profiles** of
   Ω and cumulative story displacement up the wall. A per-storey **editor**
   lets you substitute *any* connection from the database and change
   n_f / n_s even choices outside the feasible pool and shows exactly which
   CSA O86:24 clauses pass or fail.
6. **Export & report** : Excel workbook and/or PDF report of the selected design
   with the NBCC hazard values, full connection metadata, clickable source DOIs
   and the selected radar + tri-linear plots embedded; a **printable summary**
   page (`/report`) for browser print / save-as-PDF; and the **session file**.

### Design sessions

*Session → Save current session* writes a small JSON file (`exports/CLT_session_
*.json`) holding the hazard spectrum, the archetype inputs and the per-story
selection, a few kilobytes, human-readable, easy to share. Opening one restores
every input; press **Run design** and the results are reproduced from a fresh run
of the engine, with the saved per-story selection re-applied automatically.
Results themselves are never stored, so every number in a restored session is
traceable to a live run of the manuscript modules.

### Appearance

The app bar carries a **light/dark toggle**. Colours, spacing, elevation and type
are defined once in `ui_theme.py` as CSS custom properties; the wall-elevation
SVG and the chart palettes follow the active theme, so switching is instant and
needs no redraw.

---

## Connection database (embedded)

The connection database ships inside the app (`data/`):

- **Representative** (`data/connections_representative.xlsx`). the paper's
  15 representative connections, one per populated stiffness–strength cluster:
  4 WWS, 7 WFT and 4 WFS (sheets `Selected_WWS/WFT/WFS`, columns `Zone_label`,
  `Fy` [kN], `Ks` [kN/mm]).
  

## CLT panel catalogues (embedded)

CLT panel properties come from **catalogues** (`catalogues/`). A
default *Nordic X-Lam* catalogue ships with the app. In step 3 you pick a
catalogue, then a layup within it; those section properties feed the CLT panel
resistance. *Manage catalogues* lets you view/edit values inline, add layups,
save a new catalogue (change the name) or delete one stored as JSON in
`catalogues/`. `catalogues.py` writes the selected catalogue to the workbook
format the engine reads. Each layup carries `plies`, `t_mm`, `reff0`, `Aeff`,
`GAeff0`, `Ieff0`, `EIeff0` (odd ply counts 3–9).

---

## Seismic hazard (NBCC 2020 / 2025 tool)

Seismic hazard is obtained directly from the official **2025 – 2020 National
Building Code of Canada Seismic Hazard Tool** (SHT) operated by Natural Resources
Canada / Earthquakes Canada, via its GraphQL webservice
(`https://www.earthquakescanada.nrcan.gc.ca/api/canshm/graphql`). In NBCC 2020
the site amplification is embedded in the hazard values (site designation *X*),
so the returned 5%-damped spectral accelerations Sa(T, X) are used **directly**
as the design spectrum S(T), no separate foundation factors, feeding the
existing `LateralDesign` module unchanged.

In the app (step 2):

- **Edition** : NBC2020 (current code) or NBC2025.
- **Location** : pick a city preset or enter latitude/longitude (any site in Canada).
- **Site designation** : toggle between **Site Class** (A–E) and **Vs30**
  (140–3000 m/s).
- **Probability of exceedance** : 2% / 50 yr (the NBCC design level; default),
  5% or 10%.
- **Fetch from NBCC tool** : retrieves Sa(T, X) at
  T = PGA, 0.05, 0.1, 0.2, 0.3, 0.5, 1.0, 2.0, 5.0, 10.0 s and plots the design
  spectrum.
- **Offline / manual entry** : if there is no internet, read the values off the
  web tool and type them in (an example spectrum is provided). These are also the
  values written into the exported report.

An internet connection is required for the live fetch. The tool covers locations
within Canada only.

*Data source: 2020 National Building Code of Canada Seismic Hazard Tool (2022),
Natural Resources Canada, https://doi.org/10.23687/b1bd3cf0-0672-47f4-8bfa-290ae75fde9b.*

---

## Output metrics

| Symbol | Meaning |
|--------|---------|
| `Ω` (Omega) | storey over-capacity coefficient, 95th-percentile WWS resistance (CSA O86:24 Cl. 11.9.3.6.4.2) |
| `n_f` | number of WWS rows per vertical joint |
| `n_s` | number of WFS connectors per panel |
| `Kw` | equivalent lateral stiffness [kN/mm] |
| `Rwr` | wall rocking resistance [kN] |
| `SBMS` | story bending moment strength [kN·m] |
| `d_R`, `d_S` | elastic rocking and sliding drift components [mm] |
| `MRD`, `MID`, `MGD` | max roof / inter-story / global drift [%] |
| `MBRR`, `BOCC` | base rocking resistance / base over-capacity coefficient |


### CSA O86:24 clauses implemented

Every design check the Studio performs maps to a clause of CSA O86:24. Clause
numbers below follow the standard; the corresponding equation numbers in the
paper are given in the manuscript's Section 2.4.

| Design check | Clause |
|---|---|
| CLT panel factored resistances (compression ∥ grain, edgewise bending, edgewise shear, bearing) | 8.1.4.5.5, 8.1.4.3, 8.1.4.4, 8.1.4.7.2 |
| Gravity panel screening — combined flatwise bending + axial compression | 8.1.4.6 |
| Coupled-panel (CP) kinematic mode | 11.9.2.1.2.3 |
| Minimum rocking resistance | 11.9.2.1.2.2 |
| Rocking prevented by factored dead load | 11.9.2.1.2.4 |
| Capacity protection of the non-dissipative panel | 11.9.3.6.4.1 |
| Story over-capacity coefficient Ω | 11.9.3.6.4.2 |
| Inter-story continuity of Ω (ratio 0.9–1.2) | 11.9.3.6.4 |
| Equivalent lateral stiffness contributions | 11.9.5.3 |

Seismic demand follows the NBCC equivalent-static force procedure (Cl. 4.1.8.11),
with the seismic category determined per NBCC Art. 4.1.8.5.

---

## Citation

The embedded connection database derives from:

> Heidari, A., Dang, E., & Rezaei Rad, A. (2025). Design database development and
> comparative performance assessment of connections in cross-laminated timber
> construction. *Journal of Building Engineering*, 114, 114278.
> https://doi.org/10.1016/j.jobe.2025.114278

---

## Contact

**Roya Alizadeh** | PhD Candidate, Department of Civil & Mineral
Engineering, University of Toronto | roya.alizadeh@mail.utoronto.ca

**Aryan Rezaei Rad** | Assistant Professor, Department of Civil & Mineral
Engineering, University of Toronto| aryan.rad@utoronto.ca

Questions, bug reports and feature requests are welcome through the
[Issues](https://github.com/SuStrucSy/CLT-Shearwall-Design-Studio/issues) page.

---

## Disclaimer

This tool uses curated data from cited references for research and educational
purposes. It is a preliminary engineering resource and does not replace
project-specific design, applicable codes, or manufacturer documentation. Use is
at your own discretion, provided as is, and the authors accept no liability for
any outcomes. Verify all information against the original sources.
