# Distance — How Long Is a Mile?

**Measuring geographic accessibility to healthcare across Africa.**
A **Data Innovation for Africa** initiative of the **African Development Bank (AfDB)**.

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

**Two-Step Floating Catchment Area** (Luo & Wang, 2003 [1]):

- **Step 1 — Facility supply ratio:** `Rⱼ = Sⱼ / Σ Pₖ`
  For each facility *j*, gather all population *k* within travel time *d₀* and divide its physicians *Sⱼ* by that catchment population.
- **Step 2 — Accessibility at a residence:** `Aᵢ = Σ Rⱼ`
  For each location *i*, sum the supply ratios of every facility *j* reachable within *d₀*. Higher *Aᵢ* = better access.

Method family: 2SFCA [1], E2SFCA [4], 3SFCA [5], M2SFCA [6], CB2SFCA [7].

## Methodology pipeline

1. **Prepare the layers** — population & facility points → centroids on a grid (QGIS, OSM, Overpass)
2. **Build the matrix** — nearest-neighbour links each population point to its *k* = 3 closest facilities
3. **Compute travel time** — local routing engine returns real driving times (OSRM · Docker · MLD [10])
4. **Score with 2SFCA** — two floating-catchment passes → accessibility index per residence

## Data & architecture

**Inputs (all open):** Meta/WorldPop population (1,733,790 points [8]) · OpenStreetMap + sub-Saharan health-facility database (~600 [9]) · OSM road network (123,183 segments) · Rwanda MoH physician ratios · neural-net / satellite population prediction [12].

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

## Impact & roadmap

The same open, infrastructure-as-code pipeline redeploys to new cities and domains (education, public services), turning the question from *"where is the gap?"* into a prescriptive one — the optimal location for the next clinic, school or public service.

---

## References

1. Luo, W. & Wang, F. (2003). Measures of spatial accessibility to health care in a GIS environment. *Environ. Plann. B*, 30(6), 865–884. doi:10.1068/b29120
2. Radke, J. & Mu, L. (2000). Spatial decompositions, modeling and mapping service regions to predict access to social programs. *Geographic Information Sciences*, 6(2), 105–112.
3. Guagliardo, M. F. (2004). Spatial accessibility of primary care: concepts, methods and challenges. *Int. J. Health Geographics*, 3, 3. doi:10.1186/1476-072X-3-3
4. Luo, W. & Qi, Y. (2009). An enhanced two-step floating catchment area (E2SFCA) method. *Health & Place*, 15(4), 1100–1107. doi:10.1016/j.healthplace.2009.06.002
5. Wan, N., Zou, B. & Sternberg, T. (2012). A three-step floating catchment area method. *IJGIS*, 26(6), 1073–1089. doi:10.1080/13658816.2011.624987
6. Delamater, P. L. (2013). A modified two-step floating catchment area (M2SFCA) metric. *Health & Place*, 24, 30–43. doi:10.1016/j.healthplace.2013.07.012
7. Fransen, K., Neutens, T., De Maeyer, P. & Deruyter, G. (2015). A commuter-based two-step floating catchment area method. *Health & Place*, 32, 65–73. doi:10.1016/j.healthplace.2015.01.002
8. Tatem, A. J. (2017). WorldPop, open data for spatial demography. *Scientific Data*, 4, 170004. doi:10.1038/sdata.2017.4
9. Maina, J., Ouma, P. O., Macharia, P. M., et al. (2019). A spatial database of health facilities managed by the public health sector in sub-Saharan Africa. *Scientific Data*, 6, 134. doi:10.1038/s41597-019-0142-2
10. Luxen, D. & Vetter, C. (2011). Real-time routing with OpenStreetMap data. *Proc. 19th ACM SIGSPATIAL GIS*, 513–516. doi:10.1145/2093973.2094062
11. Dijkstra, E. W. (1959). A note on two problems in connexion with graphs. *Numerische Mathematik*, 1, 269–271. doi:10.1007/BF01386390
12. Goodfellow, I., et al. (2014). Generative adversarial nets. *NeurIPS*, 27. arXiv:1406.2661

---

## Repository contents

```
index.html   Self-contained project page (images embedded as data URIs)
assets/      Figures and logo (raw exports)
  logo-data-innovation-for-africa.png
  01-distance-bands-kigali.jpg             Concentric travel-distance bands (hero)
  02-distance-matrix-nearest-neighbour.jpg Street-level nearest-neighbour linkage
  03-od-matrix-shortest-paths.jpg          Shortest paths from one node
  04-od-matrix-full-web.jpg                Full origin–destination web
  05-2sfca-accessibility-map.jpg           2SFCA accessibility per residence
  07-health-context.jpg                    Health-domain context imagery
  08-framework-inputs-outputs.jpg          Inputs → model → outputs framework
```

## Publishing with GitHub Pages

`index.html` is fully self-contained. In the repository: **Settings → Pages**, set the source to the `main` branch (root). Served at `https://data-innovation-for-africa.github.io/distance-healthcare-accessibility/`.
