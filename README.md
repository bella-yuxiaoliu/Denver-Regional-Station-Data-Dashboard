# Denver Regional Station Data Dashboard

Station network dashboard for the GOFO Denver region. It maps ZIP coverage, routes, DSPs, pricing and daily volume for 13 stations, and includes a DSP pricing analysis page. It is a static website (no build step, no server) hosted on GitHub Pages: open the link and it works.

Based on the NorCal ops console template.

## Folder structure

```
index.html                     Main page: network map (all code and embedded data live in this one file)
analytics/dsp-pricing.html     Subpage: DSP pricing analysis (linked from the map sidebar)
data/price_adjustments.csv     Price sheet: edit only this file for routine price changes
README.md                      This file
```

Keep this structure as is. The pages reference each other by relative path.

## Data

| Item | Details |
|---|---|
| Source | Cleaned Denver station roster workbook (`Denver片区各站点信息_整理版.xlsx`, 2026-09-28) |
| Coverage | 13 stations · 11 DSPs · 77 routes · 316 ZIPs · ~97,680 packages/day (CO, UT, NM, ID, WY, MT) |
| Station codes | Taken from the route name prefix, e.g. `DEN01-003` belongs to DEN01 |
| Average prices | Route, station and DSP averages are all weighted by daily volume; a simple mean is used when volume is 0 |
| Volume | Average daily volume, rounded to whole packages per ZIP |
| Station pins | Placed at the center of each station's address ZIP, not the exact street address, so they can be a few km off |
| ZIP boundaries | US Census ZCTA5 2020 (`cb_2020_us_zcta520_500k`), simplified |
| ZIPs without boundaries | 87131, 87158 (ABQ01) and 83303 (TWF01) have no Census boundary shape (PO-box / campus ZIPs). They are counted in all totals but not drawn on the map. Combined volume is about 1 package/day. |

## First-time deploy (GitHub Pages)

1. Create a new repository on github.com, e.g. `Denver-Regional-Station-Data-Dashboard`.
2. In the repository, click **Add file → Upload files**. Drag in **everything inside** this folder (keep the `analytics/` and `data/` subfolders), then click **Commit changes**.
3. Go to **Settings → Pages**. Set Source to **Deploy from a branch**, Branch to **main**, folder to **/ (root)**, and click **Save**.
4. After 1–2 minutes the site is live at:
   `https://<your-username-or-org>.github.io/<repository-name>/`
   The exact link is also shown at the top of Settings → Pages ("Your site is live at …").

> ⚠️ **Data visibility:** On a free GitHub account, Pages can only be deployed from a **Public** repository. That means anyone with the link can view the dashboard, and the repository source (per-ZIP prices, volumes, DSPs, station heads' names) is visible to everyone. Private repositories require GitHub Pro / Team / Enterprise. Confirm the company allows this data to be public before deploying.
>
> We recommend hosting the repository under the **company's GitHub organization** rather than a personal account, so the link keeps working when people change roles.

## Updating prices

Only edit `data/price_adjustments.csv`. No other file needs to change.

1. In the GitHub repository, open `data/price_adjustments.csv` and click the pencil icon to edit it (or edit it in Excel, save as CSV, and re-upload it with Upload files).
2. The format is two columns, `zip,price`, e.g. `80206,1.95`. Change only the ZIPs you want to reprice.
3. Click **Commit changes**. The update goes live in about 1 minute for everyone who opens the link.

Both pages read this file on load and recompute ZIP colors, route / station / DSP averages and rankings. The **Price sheet** status in the map sidebar shows how many ZIPs were loaded and how many differ from the base prices; the chip at the top of the pricing page shows the same.

Note: the CSV can only reprice **existing** ZIPs. ZIPs in the CSV that are not on the dashboard are ignored.

## Updating volume, stations or ZIPs

Volume, routes, DSPs, stations and ZIP assignments are embedded in `index.html` and `analytics/dsp-pricing.html`. Editing the CSV does not change them. To update them:

1. Update the Excel workbook in the same format as `Denver片区各站点信息_整理版.xlsx` (an Address sheet plus one sheet per station, with columns: Station, Sorting Center, Route, Price, Zipcode, Daily Volume).
2. Give the new workbook, this folder, and the ZIP boundary file `cb_2020_us_zcta520_500k.zip` (download: https://www2.census.gov/geo/tiger/GENZ2020/shp/cb_2020_us_zcta520_500k.zip) to Claude and ask it to regenerate the dashboard.
3. Upload the regenerated files to the repository, overwriting the old ones, and commit.

When adding a station, fill in a complete and accurate address on the Address sheet. The map uses the ZIP in the address to place the station.

## FAQ

**I opened `index.html` by double-clicking it and the price sheet isn't applied.**
This is expected. Browsers don't let local files (`file://`) read the CSV, so the page falls back to the embedded base prices. It works normally once deployed to GitHub Pages.

**The link doesn't open / shows 404.**
- Pages takes 1–2 minutes to go live after it is first enabled.
- Check that Settings → Pages is set to the main branch and / (root).
- Check that `index.html` is in the **root** of the repository, not inside a subfolder (uploading the whole folder can add an extra level).

**I changed the CSV but the page didn't update.**
Your browser may be caching the old version. Hard-refresh (Mac: Cmd+Shift+R; Windows: Ctrl+F5). You can also check the repository's **Actions** tab to confirm the deployment finished.

**A station pin is slightly off.**
Pins are placed at the center of the station's address ZIP. For exact locations, provide each station's latitude and longitude and ask Claude to update them.
