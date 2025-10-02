
# Peat Depth Surveys — Leaflet web map (WFS)

Static client-only web map that loads Peatland ACTION peat depth/condition **points** straight from NatureScot GeoServer (WFS) and styles by depth.

## 1) Confirm the WFS `typeName`
Open the WFS GetCapabilities in a browser and search for the feature type you want:
```
https://ogc.nature.scot/geoserver/peatlandaction/wfs?service=WFS&request=GetCapabilities
```
Look for the layer under `<FeatureTypeList>` and copy the full name, e.g. `peatlandaction:peat_depth` or similar.
Then edit **index.html** and set:
```js
const TYPE_NAME = "peatlandaction:peat_depth"; // <— change me
```

## 2) Test locally
Just open `index.html` in a browser (or use a tiny HTTP server to avoid CORS issues).
If the server denies cross-origin requests, run a local server:
```
python3 -m http.server 8000
```

## 3) Deploy to AWS S3 (static site) + optional CloudFront
1. Create an S3 bucket (Region of your choice), name it e.g. `peat-depth-map`.
2. **Uncheck** "Block all public access" (or use CloudFront OAC if you prefer private S3).
3. Upload `index.html` and `style.css`.
4. In **Properties → Static website hosting**, enable and set `index.html` as the index document.
5. Copy the bucket website endpoint URL.
6. (Optional) Put CloudFront in front for HTTPS + caching.

### S3 bucket policy (public website quick start)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::peat-depth-map/*"
    }
  ]
}
```

## 4) Notes
- This app paginates through WFS 2.0 with `count` + `startIndex` (default 10,000 per page). Reduce `PAGE_SIZE` if needed.
- Toggle “BBOX load” to only fetch features within the current map view (faster and friendlier).
- The popup shows the first ~20 attributes and tries common field names for depth and date. Adjust the mapping if needed.
- If performance is slow (tons of points), consider:
  - Server-side filtering (CQL) by date, site, etc.
  - Using `BBOX` or a tile-based approach (e.g., GeoServer Vector Tiles / WMS heatmap).

## 5) Optional: CQL filter examples
Append a `cql_filter` param to limit features (if enabled on the server):
```
&cql_filter=depth_cm%20IS%20NOT%20NULL%20AND%20depth_cm%3E%3D50
```
You can extend the code to pass through a `cql_filter`.

---
Made for the #30DayMapChallenge (theme: Points).
