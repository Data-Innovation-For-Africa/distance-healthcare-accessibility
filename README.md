# Distance — How Long Is a Mile?

**Measuring geographic accessibility to healthcare across Africa.**
A project of the **African Development Bank — Data Innovation Lab (DIL)**.

A straight line on a map says almost nothing about whether a patient can *actually reach a doctor*. This project reframes healthcare accessibility around **travel time and real service availability** rather than as-the-crow-flies distance, and computes a reproducible **accessibility indicator** to help target investment where it closes the widest gaps — so that no one is left behind.

👉 **Live page:** open `index.html` (a self-contained page — no build step, no dependencies).

---

## The two approaches

The project measures accessibility along two complementary axes:

| | **Distance Approach** | **Gravity Model** |
|---|---|---|
| **Question** | How far — in time — is the nearest facility? | Given demand and friction, is the service *enough*? |
| **Tools** | Distance Matrix · O–D Matrix · Isochrone Zones | Provider-to-population ratio · **2SFCA** · friction weighting |
| **Measures** | Proximity (supply-side, geographic) | Availability (supply **and** demand) |

The Distance Approach produces the travel times *dᵢⱼ* that feed the Gravity Model. One says *"how far"*; the other says *"with what real level of service."*

## The 2SFCA algorithm

**Two-Step Floating Catchment Area** (Luo & Wang, 2003):

- **Step 1 — Facility supply ratio:** `Rⱼ = Sⱼ / Σ Pₖ`
  For each facility *j*, gather all population *k* within travel time *d₀* and divide its physicians *Sⱼ* by that catchment population.
- **Step 2 — Accessibility at a residence:** `Aᵢ = Σ Rⱼ`
  For each location *i*, sum the supply ratios of every facility *j* reachable within *d₀*. Higher *Aᵢ* = better access.

## Methodology pipeline

1. **Prepare the layers** — population & facility points → centroids on a grid (QGIS, OSM, Overpass)
2. **Build the matrix** — nearest-neighbour links each population point to its *k* = 3 closest facilities
3. **Compute travel time** — local routing engine returns real driving times (OSRM · Docker · MLD)
4. **Score with 2SFCA** — two floating-catchment passes → accessibility index per residence

## Data & architecture

**Inputs (all open):** Meta/WorldPop population (1,733,790 points) · OpenStreetMap + Nature Scientific Data facilities (~600) · OSM road network (123,183 segments) · Rwanda MoH physician ratios · neural-net / satellite population prediction.

**Stack (all open-source, distributed):** OSRM → Apache Flume → Apache Kafka + ZooKeeper → Logstash → Elasticsearch → Kibana. Provisioned as code with Vagrant + Ansible; QGIS + Docker for preparation and routing.

## Use case — Kigali, Rwanda

Proof of concept: ~1,239 villages, ~600 facilities, 1.7 M population points — taken from raw geodata all the way to an accessibility map and live dashboard.

### Key results

| Metric | Value |
|---|---|
| Poor accessibility (2SFCA ≤ 0.50) at *d₀* = 1,800 s | **9.30 %** |
| Under-served at stricter *d₀* = 900 s | **12–13 %** |
| Population within 2,500 m of a facility (k = 3) | **63.21 %** |
| Routed records indexed (avg. distance 13,093.9 m) | **82,085** |

**Interpretation:** the distance view says most of Kigali is near a clinic; the gravity view shows a meaningful minority still lack *sufficient* access once provider capacity and competition are counted. That gap is the target.

## Roadmap

Kigali proved the method. Next: scale the healthcare accessibility indicator across **14 African countries**, and extend from *"where is the gap?"* to *"where should the next service go?"* — and to other domains (education, public services).

---

## Repository contents

```
index.html   Self-contained project page (images embedded as data URIs)
assets/      Result figures exported from the Kigali use case
  01-distance-bands-kigali.jpg            Concentric travel-distance bands (King Faisal Hospital)
  02-distance-matrix-nearest-neighbour.jpg Street-level nearest-neighbour linkage
  03-od-matrix-shortest-paths.jpg          Shortest paths fanning from one node
  04-od-matrix-full-web.jpg                Full origin–destination web
  05-2sfca-accessibility-map.jpg           2SFCA accessibility scored per residence
  06-kibana-dashboard.jpg                  Live ELK / Kibana results dashboard
```

## Publishing with GitHub Pages

`index.html` is fully self-contained. In the repository, go to **Settings → Pages**, set the source to the `main` branch (root), and the page will be served at `https://<org>.github.io/<repo>/`.

---

*Reference: Luo, W. & Wang, F. (2003). "Measures of Spatial Accessibility to Healthcare in a GIS Environment: Synthesis and a Case Study in the Chicago Region." Environ. Plann. B, 30(6): 865–884.*
