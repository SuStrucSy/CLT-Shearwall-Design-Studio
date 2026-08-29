# CLT Shearwall Design Studio

An interactive companion tool for the manuscript *Design Space Exploration and
Archetype Development for CLT Shearwalls under Capacity-Based Design*.
Given a single wall archetype, it runs the paper's capacity-based design
pipeline and returns **every feasible connection configuration per storey** with
its performance metrics, lets you explore, compare and select configurations,
and exports the chosen design to Excel, PDF and a printable summary.

> ### Try it now — no installation
>
> **https://clt-shearwall-studio.onrender.com/**
>
> The full application runs in your browser. The first load may take up to a
> minute: the host suspends idle instances and the first request wakes the
> server. Press *Load example hazard (offline)* on the Overview page to reach a
> complete result in two clicks.
>
> Prefer to run it locally, or need it offline? See [Install & run](#install--run).

The interface is a **guided, multi-page workflow**: a persistent navigator at the
top of the window lists every design step in order, tagged with the manuscript
module it implements, and marks each one as done, current or still locked.

| Step | Page | Manuscript module |
|------|------|-------------------|
| — | Overview | the framework, bundled data, saved sessions |
| 1 | Seismic hazard | Module 2 — NBCC 2020/2025 design spectrum Sa(T, X) |
| 2 | Wall archetype | Modules 1 & 3 — archetype, CLT catalogue, connection pool |
| 3 | Demand & panel check | Modules 2 & 4 — equivalent-static demand, gravity screening |
| 4 | Connection configurations | Module 4 — every feasible storey layout |
| 5 | System chain | Module 5 — inter-storey continuity (Cl. 11.9.3.6.4), EDPs, per-storey editor |
| 6 | Export & report | Excel · PDF · printable summary · session file |

The design engine reuses the manuscript modules **unchanged** (`core/`), so
results are identical to the batch pipeline: NBCC 2020/2025 equivalent-static
seismic demand (`LateralDesign.py`, NBCC Cl. 4.1.8.11) and CSA O86:24
capacity-based connection design (`ConnectionDesign.py`). The app (`app.py`,
`engine.py`) only drives those functions for one archetype and presents the
results; `ui_theme.py` holds the
entire visual design system (tokens, light/dark themes, components), so the
appearance can be changed without touching any engineering code.

---

## Install & run

**One-click:** double-click `run_app.bat` (Windows) or `run_app.command` (macOS).
It creates an isolated environment on first run and launches the app.

> **Running the same folder on two machines (e.g. OneDrive-synced).** A virtual
> environment is *not* portable across operating systems, so each platform gets
> its own directory: `.venv-win` on Windows, `.venv-macos` on macOS,
> `.venv-linux` elsewhere. The launchers decide whether to install by trying to
> **import** the dependencies rather than by looking for a marker file, so an
> environment synced from another machine can never make them skip the install.
> An old shared `.venv/` from an earlier version can be deleted.

Python **3.9 or newer** works: the launcher picks the newest interpreter it can
find, and pip resolves the matching NiceGUI release (3.x on Python ≥ 3.10, an
older 3.x/2.x on Python 3.9). The app supports all of them.

**Manual:**

```bash
cd CLT_Shearwall_App
pip install -r requirements.txt
python app.py
```

A browser window opens at `http://localhost:8123`.

**Port / sharing over Tailscale.** The app listens on port **8123** (8080 is
avoided). Change it with the `APP_PORT` environment variable, e.g.
`set APP_PORT=8200 && python app.py` (Windows) or `APP_PORT=8200 python app.py`
(macOS/Linux). It binds `0.0.0.0`, so peers on your Tailscale network can reach
it at `http://<your-tailscale-ip>:<port>/`.

---

## Workflow

**Overview (landing page)** — what the framework does, the five modules it
implements, and the bundled data. **Nothing to upload:** the connection database
(representative + whole), the CLT panel catalogues and the higher-mode Mv/J table
are all embedded. *Load example hazard (offline)* gets you to a full result in
two clicks. Saved sessions are opened from here (or from *Session* in the app
bar).

1. **Seismic hazard (NBCC 2020 / 2025 tool)** — the design spectral acceleration
   curve Sa(T, X) is fetched live from the Natural Resources Canada Seismic
   Hazard Tool for your site, and plotted with a marker at the current
   archetype's fundamental period. Manual entry is available offline.
2. **Wall archetype** — occupancy, height class, seismic category, ductility,
   storeys, storey height, gravity load *q*, panel width *b_s*, panels *m*, the
   **CLT catalogue → layup** (managed catalogues, see below), and the
   **connection source** (representative vs whole database). A live summary panel
   shows total height, wall width *B*, derived aspect ratio *H/b_s*, estimated
   *Ta* and the number of candidate combinations that will be screened.
3. **Demand & panel check** — fundamental period *Ta*, base shear *V*, importance
   factor, storey shear/overturning-moment profile, the design spectrum used, and
   the CSA O86:24 gravity panel check. A failed archetype is explained here
   instead of silently producing nothing.
4. **Connection configurations** — one tab per storey with a sortable/filterable
   table of every feasible WWS/WFT/WFS layout. Selecting a row shows a
   **performance radar** (strength / stiffness / Ω / deformation, normalised to
   the storey maxima) and **connection detail cards** with the tri-linear F–δ
   backbone and an expander of every database field + source-paper DOI link.
   **Pin up to four candidates** to compare them side by side on one radar and in
   a metric table. *Auto-assemble* runs the Module 5 depth-first search for a
   continuous chain satisfying the inter-storey continuity bounds on the
   over-capacity coefficient (CSA O86:24 Cl. 11.9.3.6.4) and reports the
   Engineering Demand Parameters.
5. **System chain** — an embedded, scaled wall elevation beside live outputs for
   the current chain: per-storey performance metrics (Ω, Kw, Rwr, SBMS, storey
   drift), the inter-storey **continuity check** (Ω-ratio 0.9–1.2,
   CSA O86:24 Cl. 11.9.3.6.4) with pass/fail badges, and **height profiles** of
   Ω and cumulative storey displacement up the wall. A per-storey **editor**
   lets you substitute *any* connection from the database and change
   n_f / n_s — even choices outside the feasible pool — and shows exactly which
   CSA O86:24 clauses pass or fail.
6. **Export & report** — Excel workbook and/or PDF report of the selected design
   with the NBCC hazard values, full connection metadata, clickable source DOIs
   and the selected radar + tri-linear plots embedded; a **printable summary**
   page (`/report`) for browser print / save-as-PDF; and the **session file**.

### Design sessions

*Session → Save current session* writes a small JSON file (`exports/CLT_session_
*.json`) holding the hazard spectrum, the archetype inputs and the per-storey
selection — a few kilobytes, human-readable, easy to share. Opening one restores
every input; press **Run design** and the results are reproduced from a fresh run
of the engine, with the saved per-storey selection re-applied automatically.
Results themselves are never stored, so every number in a restored session is
traceable to a live run of the manuscript modules.

### Appearance

The app bar carries a **light/dark toggle**. Colours, spacing, elevation and type
are defined once in `ui_theme.py` as CSS custom properties; the wall-elevation
SVG and the chart palettes follow the active theme, so switching is instant and
needs no redraw.

---

## Connection database (embedded — no upload)

The connection database ships **inside the app** (`data/`), selectable in step 3:

- **Representative** (`data/connections_representative.xlsx`) — the paper's
  15 representative connections, one per populated stiffness–strength cluster:
  4 WWS, 7 WFT and 4 WFS (sheets `Selected_WWS/WFT/WFS`, columns `Zone_label`,
  `Fy` [kN], `Ks` [kN/mm]). Small → fast enumeration.
- **Whole database** (`data/connections_full.xlsx`) — the experimental
  connection database of 522 tests described in Section 2.3 of the paper. Each
  record carries Ks, the tri-linear backbone (Δy/Fy, Δmax/Fmax, Δu/Fu), µ,
  failure mode and the source paper + DOI, mapped to roles WWS/WFT/WFS by
  *Connection Type* and *Loading Type*. Much larger candidate pool → the run is
  executed in the background with a progress bar and a per-storey result cap.

`connections_db.py` handles loading, role mapping, DOI cleanup and detail
extraction. To update the data, replace the files in `data/`.

## CLT panel catalogues (embedded)

CLT panel properties come from **catalogues** (`catalogues/`), not an upload. A
default *Nordic X-Lam* catalogue ships with the app. In step 3 you pick a
catalogue, then a layup within it; those section properties feed the CLT panel
resistance. *Manage catalogues* lets you view/edit values inline, add layups,
save a new catalogue (change the name) or delete one — stored as JSON in
`catalogues/`. `catalogues.py` writes the selected catalogue to the workbook
format the engine reads. Each layup carries `plies`, `t_mm`, `reff0`, `Aeff`,
`GAeff0`, `Ieff0`, `EIeff0` (odd ply counts 3–9).

> The shipped default section-property values are template seeds — replace them
> with catalogue-verified Nordic (or other manufacturer) values via the editor.

## Embedded data — no uploads

Everything the design needs is bundled in `data/` and `catalogues/`:

- **Connection database** — `data/connections_full.xlsx` (whole DB) and
  `data/connections_representative.xlsx` (clusters), both with full metadata.
- **Higher-mode factors (Mv, J)** — `data/wall_system_mv.xlsx` (NBCC Table
  4.1.8.11). Replace this file to use your own validated table.
- **CLT catalogues** — `catalogues/*.json` (managed in-app; see above).
- **Seismic hazard** — fetched live from the NBCC tool (step 2).

`data_templates/` retains the Excel templates (regenerate with
`python make_templates.py`) only as a format reference / for editing the embedded
files; the app itself no longer asks for any upload.

---

## Seismic hazard (NBCC 2020 / 2025 tool)

Seismic hazard is obtained directly from the official **2025 – 2020 National
Building Code of Canada Seismic Hazard Tool** (SHT) operated by Natural Resources
Canada / Earthquakes Canada, via its GraphQL webservice
(`https://www.earthquakescanada.nrcan.gc.ca/api/canshm/graphql`). In NBCC 2020
the site amplification is embedded in the hazard values (site designation *X*),
so the returned 5%-damped spectral accelerations Sa(T, X) are used **directly**
as the design spectrum S(T) — no separate foundation factors — feeding the
existing `LateralDesign` module unchanged.

In the app (step 2):

- **Edition** — NBC2020 (current code) or NBC2025.
- **Location** — pick a city preset or enter latitude/longitude (any site in Canada).
- **Site designation** — toggle between **Site Class** (A–E) and **Vs30**
  (140–3000 m/s).
- **Probability of exceedance** — 2% / 50 yr (the NBCC design level; default),
  5% or 10%.
- **Fetch from NBCC tool** — retrieves Sa(T, X) at
  T = PGA, 0.05, 0.1, 0.2, 0.3, 0.5, 1.0, 2.0, 5.0, 10.0 s and plots the design
  spectrum.
- **Offline / manual entry** — if there is no internet, read the values off the
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
| `SBMS` | storey bending moment strength [kN·m] |
| `d_R`, `d_S` | elastic rocking and sliding drift components [mm] |
| `MRD`, `MID`, `MGD` | max roof / inter-storey / global drift [%] |
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
| Storey over-capacity coefficient Ω | 11.9.3.6.4.2 |
| Inter-storey continuity of Ω (ratio 0.9–1.2) | 11.9.3.6.4 |
| Equivalent lateral stiffness contributions | 11.9.5.3 |

Seismic demand follows the NBCC equivalent-static force procedure (Cl. 4.1.8.11),
with the seismic category determined per NBCC Art. 4.1.8.5.

---

## Project structure

```
CLT_Shearwall_App/
├── app.py            # NiceGUI application: step navigator + the six step pages
├── ui_theme.py       # design system: tokens, light/dark CSS, cards/KPIs/pills
├── session_io.py     # save / restore a design session (JSON)
├── engine.py         # single-archetype driver (reuses core/ functions)
├── connections_db.py # embedded connection database loader (representative + full)
├── catalogues.py     # CLT panel catalogues (Nordic default + CRUD)
├── hazard_nbcc.py    # NBCC 2020/2025 seismic hazard client (NRCan GraphQL API)
├── viz.py            # radar / tri-linear / spectrum / profile options + wall SVG
├── reporting.py      # Excel + PDF export (metadata, DOIs, embedded plots)
├── make_templates.py # regenerates the Mv template
├── run_app.bat / run_app.command   # Windows / macOS one-click launchers
├── requirements.txt
├── LICENSE, CITATION.cff, .gitignore
├── pytest.ini, conftest.py, test_*.py              # tests
├── packaging/clt_app.spec                          # PyInstaller build spec
├── .github/workflows/build-executables.yml         # CI: Win/macOS executables
├── core/             # manuscript modules, UNCHANGED (config, LateralDesign,
│                     #   ConnectionDesign, Postprocessing)
├── data/             # embedded connection database (full + representative)
├── catalogues/       # CLT catalogues (JSON) — default Nordic X-Lam
├── data_templates/   # Mv template (+ legacy hazard/CLT templates, unused)
└── exports/          # generated Excel / PDF reports
```

## Distribution

The repo is ready to publish alongside the paper:

- **Run from source** (any OS with Python 3.9+): `run_app.bat` / `run_app.command`
  or `pip install -r requirements.txt && python app.py`.
- **Standalone executables** (no Python needed): `packaging/clt_app.spec` builds a
  single-file executable with PyInstaller. Build locally on the target OS
  (`pip install pyinstaller && pyinstaller packaging/clt_app.spec` → `dist/`), or
  let CI do it: `.github/workflows/build-executables.yml` builds Windows + macOS
  executables and attaches them to each GitHub Release automatically.
- **Citation:** `CITATION.cff` (fill in co-authors + DOI). For a citable software
  DOI, create a GitHub Release and archive it on **Zenodo** (or Borealis), then add
  the DOI to `CITATION.cff` and the paper. Executables and source are MIT-licensed.

> Attachments (connection-detail PDFs, test-curve images, failure photos) are **not**
> bundled — the 1 GB Access database exceeds GitHub limits. Each connection links to
> its **source paper via DOI**; deposit the full database + attachments on Zenodo /
> Borealis for a permanent data DOI and link it from the app and paper.

## Tests

```bash
python -m pytest -q
```

`test_ui.py` covers the application shell and step navigator, every step page,
the full load→run→results flow, auto-assembly, both exports, the printable
summary route and the session save/load round trip (NiceGUI headless `User`
harness). `test_hazard.py` covers the NBCC hazard client: GraphQL query
construction (Site Class enum / Vs30, NBC2020 & NBC2025), response parsing
against the confirmed API schema, error handling, and end-to-end integration
with the engine.

---

## Citation

If you use this tool or its outputs, please cite the paper:

> Alizadeh, R., & Rezaei Rad, A. (XXXX). *Design Space Exploration and Archetype
> Development for CLT Shearwalls under Capacity-Based Design.* [Journal],
> [volume], [pages]. https://doi.org/XXXX

and the software itself (see `CITATION.cff`):

> Alizadeh, R., & Rezaei Rad, A. (2026). *CLT Shearwall Design Studio*
> [Computer software]. https://github.com/<user>/CLT-Shearwall-Design-Studio

The embedded connection database derives from:

> Heidari, A., Dang, E., & Rezaei Rad, A. (2025). Design database development and
> comparative performance assessment of connections in cross-laminated timber
> construction. *Journal of Building Engineering*, 114, 114278.
> https://doi.org/10.1016/j.jobe.2025.114278

---

## Contact

**Roya Alizadeh** — PhD Candidate, University of Toronto —
roya.alizadeh@mail.utoronto.ca

**Aryan Rezaei Rad** — Assistant Professor, Department of Civil & Mineral
Engineering, University of Toronto, 35 St. George St., Room GB 245, Toronto, ON,
Canada, M5S 1A4 — aryan.rad@utoronto.ca — Tel: (416) 978-2077

Questions, bug reports and feature requests are welcome through the
[Issues](https://github.com/<user>/CLT-Shearwall-Design-Studio/issues) page.

---

## Disclaimer

This tool uses curated data from cited references for research and educational
purposes. It is a preliminary engineering resource and does not replace
project-specific design, applicable codes, or manufacturer documentation. Use is
at your own discretion, provided as is, and the authors accept no liability for
any outcomes. Verify all information against the original sources.
