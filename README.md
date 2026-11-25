# H2O Water Quality

A small web page that plots how Environment Agency water quality measurements change over time at a sampling point in England.

Live page: https://nimmo-smith-technologies.github.io/H2O-web/

## What it does

1. **Find a location.** Enter a postcode or a site name, press the target button to use your current position, or pan and zoom the map and tap it. Press Search to list the sampling points within 1, 3, 5, 10 or 25 km.
2. **Check each site.** The list shows, for each site, the number of results on record, the first and latest month with results, and how many results fall in the last 12 months. A tick box hides sites with no results in the last 12 months. The map shows the same sites around the search location and radius.
3. **Choose a measurement.** A dropdown lists the measurements held for the chosen site, with a plain-English note on what each common one means.
4. **Read the plot.** Results are plotted over time with a least-squares trend line, summary statistics with units, and an option to exclude statistical outliers. Results can be downloaded as CSV.

There is a light and dark theme switch at the top of the page.

## Add it to an iPhone home screen

Open the live page in Safari, tap Share, then Add to Home Screen.

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
- Site statistics (count, first and latest date, results in the last 12 months) take three small requests per site, so the nearest 150 sites are checked automatically and more on request. A search loads at most 2,000 sampling points.
- The measurement dropdown lists a built-in set of common measurements that have data at the site. "Other measurements" are found from the site's earliest and latest 250 results, so rarely measured items may be missing.

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
