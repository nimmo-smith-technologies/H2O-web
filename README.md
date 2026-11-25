# H2O Water Quality

Version 0.1 (major.minor, shown next to the title on the page).

A small web page that plots how Environment Agency water quality measurements change over time at a sampling point in England.

Live page: https://nimmo-smith-technologies.github.io/H2O-web/

## What it does

1. **Find a location.** Enter a postcode or a site name, press the target button to use your current position, or pan and zoom the map and tap it. Press Search to list the sampling points within 1, 3, 5, 10 or 25 km.
2. **Check each site.** The list shows, for each site, the number of results on record, the first and latest month with results, and how many results fall in the last 12 months. A tick box hides sites with no results in the last 12 months. The map shows the same sites around the search location and radius.
3. **Choose a measurement.** A dropdown lists the measurements held for the chosen site, with a plain-English note on what each common one means. Each measurement is tagged "recent" (at least one result in the last 12 months) or "historical only". The list is saved on your device for 12 hours.
4. **Read the plot.** Results are plotted over time with a least-squares trend line, summary statistics with units, and an option to exclude statistical outliers. Results can be downloaded as CSV.

5. **See the limits.** For some measurements at river sites, the plot shades the region beyond a statutory standard (red) or a recommended value (yellow), with the sources explained below the plot.

There is a light and dark theme switch at the top of the page.

## Limits shown on the plot

All values come from one source: [The Water Framework Directive (Standards and Classification) Directions (England and Wales) 2015](https://www.legislation.gov.uk/uksi/2015/1623/pdfs/uksiod_20151623_en_003.pdf), Schedule 3, read from the text as made. Later amendments, if any, have not been checked, and the values should be re-checked before anyone relies on them.

| Measurement | Standard | Basis | Shown as |
| --- | --- | --- | --- |
| Oxygen saturation (%) | Good: 75% (river types 1, 2, 4, 6 and salmonid) or 60% (types 3, 5, 7) | 10th percentile | Statutory, two bands |
| Ammonia as N (mg/l) | Good: 0.3 (types 1, 2, 4, 6) or 0.6 (types 3, 5, 7) | 90th percentile | Statutory, two bands |
| pH | High or good: between 6 and 9 | 5th and 95th percentiles | Statutory |
| Water temperature (°C) | Good: 23 (salmonid) or 28 (cyprinid) | Annual 98th percentile | Statutory, two bands |
| Iron, dissolved (µg/l) | 1000 | Long-term mean | Statutory |
| BOD (mg/l) | 4 (types 1, 2, 4, 6, salmonid) or 5 (types 3, 5, 7) | 90th percentile | Recommended, two bands |

Notes:

- **Statutory and recommended.** "Statutory" means a standard the Directions use to classify water bodies. "Recommended" is used for BOD, which the Directions publish but state must not be used to classify water bodies.
- **Two bands.** River type depends on altitude and alkalinity, and temperature standards depend on salmonid or cyprinid status. The API does not provide these for a site, so the darker band marks results that fail every type and the lighter band results that fail only the stricter types.
- **Not a breach test.** Each standard is judged on a statistic (a percentile or mean) over a classification period, so one result inside a shaded band does not by itself mean a breach. The Environment Agency and Natural Resources Wales make the official assessments.
- **River sites only.** Limits are drawn only where the site type is "FRESHWATER - RIVERS" and the plotted unit matches the standard.
- **Left out on purpose.** The river reactive phosphorus standard is calculated per site from its alkalinity and altitude, so it is not drawn. The Directions set no river standard for nitrate or nitrite, and their groundwater threshold values are for assessing groundwater bodies, not individual sites. Drinking-water limits apply to supplied water and are not used here.

## Data and attribution

The data come from the Environment Agency [Water Quality Archive API](https://environment.data.gov.uk/water-quality/api-docs), part of the [Water Quality Explorer](https://environment.data.gov.uk/dataset/8034d47a-ba8a-4978-aca0-bd5d6e870536) dataset (results from 2000 onwards).

> © Environment Agency copyright and/or database right 2025. All rights reserved.
> Contains public sector information licensed under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/).

Other services used by the page:

| Service | Used for | Notes |
| --- | --- | --- |
| [postcodes.io](https://postcodes.io) | Turning a postcode into coordinates | Called only when you search by postcode |
| [OpenStreetMap](https://www.openstreetmap.org/copyright) tiles | Map background | © OpenStreetMap contributors. Light, personal use only; see the [tile usage policy](https://operations.osmfoundation.org/policies/tiles/) before heavier use |
| [Leaflet](https://leafletjs.com) 1.9.4 | Map drawing | BSD 2-Clause licence. A copy is kept in `vendor/leaflet/` with its licence |

## How it works

There is no server and no build step. `index.html` is one page that calls the Environment Agency API directly from the browser (the API allows cross-origin requests). The files in this repo are served by GitHub Pages from the `main` branch.

Notes from working with the API:

- Requests need the header `Accept: application/ld+json`.
- Each request returns at most 250 records. The page pages through results with `skip` until it has them all, and reports how many results it retrieved against the total.
- Results come back oldest first, so the latest date for a site is read from the last record.
- Adding the header `Accept-Crs: http://www.opengis.net/def/crs/EPSG/0/4326` returns latitude and longitude instead of British National Grid coordinates.
- A result such as `<0.5` has no numeric value, only an upper bound. The page calls these censored results.
- Several determinand codes can go in one request as a comma-separated list, and the counts add up. The page uses this to test many measurements at once.
- Site statistics (count, first and latest date, results in the last 12 months) take three small requests per site, so the nearest 60 sites are checked automatically and more on request. A search loads at most 2,000 sampling points. Statistics are kept for the session so repeat searches do not ask again.
- To list a site's measurements and tag them recent or historical, the page reads the site's earliest 250 results and its latest results (enough pages to reach back 12 months, up to 4) and notes every measurement seen and whether it was seen in the last 12 months. It then tests the built-in measurements not seen, a group at a time, splitting a group only while results remain. For a very busy site whose latest 1,000 results do not reach back 12 months, it also tests the not-yet-seen measurements for recent results, within a budget of 10 requests, and leaves any it cannot resolve untagged. A small site needs 2 to 3 requests, a busy one about 13. At the sites tested, every tag matched a one-request-per-measurement check against the API. "Other measurements" come from the earliest and latest results only, so rarely measured ones may be missing.

### Request handling

The API documents no rate limit, but it can slow down or stop answering when it receives many requests. All requests from the page go through one queue with at most 3 in flight. Anything you ask for goes ahead of the background site checks. A request that fails with HTTP 429, 502, 503 or 504, times out after 25 seconds, or fails at the network level makes the whole queue pause and back off (4 seconds, doubling to a maximum of 60), and the page says so. Failed checks show the reason and can be retried.

## Reading the plot

- **Censored results** are reported only as below (<) or above (>) a limit, usually because the amount was too small to measure precisely. They are drawn as hollow circles at the limit value and are included in the line, statistics and trend at that value. This can bias the trend when many results are censored.
- **Trend** is a straight line fitted by least squares. It has no significance test and no seasonal adjustment, so treat it as a rough guide.
- **Outliers** are flagged with Tukey's rule: more than 1.5 × the interquartile range below the lower quartile or above the upper quartile, needing at least 8 results. Water data are often skewed, so a flagged value is unusual, not necessarily wrong.
- The plain-English notes on each measurement are general guides. They are not regulatory limits or health advice.
- "Last 12 months" means the 365 days up to today, by the date of the sample.

## Privacy

The page has no analytics and no account. It stores two things in your browser: the theme you chose and the last site you viewed. The coordinates of your search location are sent to the Environment Agency API to find nearby sites. A postcode is sent to postcodes.io. The map tiles are fetched from OpenStreetMap servers, which can see your IP address.

## Run locally

```
python3 -m http.server
```

Then open http://localhost:8000. Any static file server works.

## Licence

Copyright © 2026 Nimmo Smith Technologies Limited. The code is licensed under the GNU General Public License v3.0 (see `LICENSE`). The Environment Agency data are licensed separately under the Open Government Licence v3.0, as above.
