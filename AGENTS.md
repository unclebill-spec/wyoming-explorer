# Wyoming Explorer — agent handoff

Static Leaflet map for a nurse household, the same shared app as the Kentucky, Tennessee, Massachusetts, Maine, Vermont and Montana Explorers (`app.js`, `areas.js`, `perm.js`, `profiles.js`, `extras.js`, `style.css` are byte-identical copies of `/workspace/kentucky/explorer/*`; another worker edits them there, so run `scripts/sync_shared.sh` right before every build/publish and log any shared-file edit in KY `explorer/AGENTS.md`). `sw.js` differs only in its cache prefix (`wyx-`). Wyoming-specific settings live in `explorer/build.py` `STATE` (ported from VT by `scripts/port_build.py`, idempotent, marker PORT_ST).

Live: https://unclebill-spec.github.io/wyoming-explorer/ · repo unclebill-spec/wyoming-explorer · progress log: `/workspace/wyoming/STATUS.md` (newest first, ET).
Montana and Wyoming share ONE code base: every script in `scripts/` is identical in both folders and picks the state from the folder it runs in (`scripts/common.py` ST).

## Caps (Bill, Oct 3 2026: "add montana and wyoming, increase the housing price to 550k for both")
5+ acres $300k–$550k; 1+ acre 3bd/2ba < $550k; near-hospital 1,600+ sqft 3bd/2ba < $550k (townhomes/condos OK, good condition, ≤ 10 min of a hospital with a 10+ bed ER). No cabin category.

## Blocks
23 counties (`scripts/common.py` -> `data/statewide/raw/wy_blocks_500k.zip`).

## Data pipeline (run from /workspace/wyoming; pandas scripts use /workspace/kentucky/.venv/bin/python, the rest /usr/bin/python3)
1. `scripts/hospitals_research.py` (CMS + state trauma list + border Level I/II) -> `data/hospitals.json`; `scripts/layers.py` -> block CSVs, schools (CCD 2024-25 + SEDA), RN wages (O*NET/BLS May 2025; no employment counts: BLS API daily limit + bls.gov downloads blocked).
2. `scripts/appeal_build.py` (OSRM drives, cached) -> appeal shading; `scripts/er_beds.py` -> `data/hospital_er_beds.json` (10+ bed ERs for the near-hospital rule).
3. Climate normals -> `data/clim.json`; activities `data/osm/wd_act.py` (Wikidata) + `data/osm/wp_cat_act.py` (waterfalls/trails); colleges `data/osm/nces_post.json`.
4. Compare areas: `compare/areas_build.py` + `land.py` -> `data/areas.json` (Cheyenne, Casper, Gillette, Laramie, Rock Springs).
5. Homes: `scripts/zsearch.py` (Zillow county searches at the $550k caps) -> `data/zsearch/`; `scripts/listings_build.py` (PER_COUNTY 18) -> `listings.json` + `listing-photos/` (log `data/lb.log`); `scripts/bargains.py` (state Zillow comps + national ZHVI from /workspace/kentucky/data); `scripts/top_lists.py`.
6. Permanent RN jobs: `scripts/perm_jobs.py` -> `data/perm_jobs.json` (reuses the KY readers in /workspace/kentucky/scripts/perm_jobs_collect.py via source rewriting). Sources: Banner Workday (Casper/Worland/Wheatland/Torrington queries), Cheyenne Regional (Phenom), HealthcareSource cchwyo (Campbell County) / sheridanhospital / westpark (Cody), Ivinson (UKG UCHealth board), St. John's Jackson (Paycom, browser).
7. Travel RN jobs: helpers in `/workspace/tj_wy` (Vivian + Advantis), then `scripts/travel_jobs.py` -> `data/travel_jobs.json`.
8. Phase 4: `data/airports/airports.py`; `data/attractions/wd2.py` + `make_attractions.py` (MANUAL_LL from Nominatim); Crexi `data/forsale/crexi_list.py`, `st_forsale.py`, `make_forsale.py`; thumbnails `explorer/fetch_thumbs.py` (after a build).
9. Border items: `/workspace/border` (`scripts/static.py MT WY`, `scripts/make_border.py MT WY`), copy `out/WY.json` -> `explorer/border.json`. Montana's north side is Canada (BC/AB/SK): no Canadian homes/jobs/schools, only the US neighbors (ID, ND, SD, WY).
10. Ski areas + peaks: `/workspace/mtn/scripts/make_state.py WY` -> `explorer/mtn.json` + `img/mtn/` (see /workspace/mtn/PROGRESS.md). Ticket prices come only from skiresort.com.
11. Publish: `sh scripts/sync_shared.sh && cd publish && PATH=/usr/bin:$PATH ./publish.sh -m "msg"` (flock, pull, build, minify, secscan of dist + full history, push, waits for Pages).
12. Tests (state from the folder): `perf/smoke.py BASE TAG`, `perf/test_homes.py`, `perf/test_perm.py`, `perf/test_p4.py`, `perf/loadtime.py URL`, `perf/sw_check.py URL...`; screenshots in `perf/shots/`.

## Known gaps (Oct 3 2026)
- RN employment counts are null (BLS limits). County history layer is empty (no wiki_history.json; fetch_history.py not run).
- HCA (researched last, Oct 3 6:35 PM ET by web search): HCA has no hospitals in Montana or Wyoming (careers.hcahealthcare.com U.S. locations lists neither); its nearest, Eastern Idaho Regional Medical Center in Idaho Falls, is outside the 15-mi border band. Nothing to add.
- Not collected: SageWest (Riverton/Lander) and Evanston Regional (not on Lifepoint's ORC board), Sweetwater Memorial, small critical-access hospitals. St. John's only 3 jobs.
- Only Banner Wyoming Medical Center (Casper) is an in-state Level II; Level I care is out of state (border: CO, ID, MT, NE, SD, UT).
- Ski areas White Pine and Pine Creek left off (no location found); no ticket price: Grand Targhee, Snow King, Snowy Range, Hogadon. Rendezvous Mountain has no coordinates.


### 50+ acre lots under $250k (Oct 4, 2026 ~10:31 AM ET, big-land worker)
- Black-star layer `big-land` (50+ ac, < $250k, land or home), "50+ ac" button, Map key row, card; shared code from the KY explorer (see KY explorer/AGENTS.md, same date). build.py (marker BIGLAND) merges `/workspace/wyoming/bigland.json`.
- Refresh: `/usr/bin/python3 /workspace/bigland/bigland.py WY --refresh` before build/publish (keeps the old file if Zillow blocks). Notes: /workspace/bigland/PROGRESS.md.
