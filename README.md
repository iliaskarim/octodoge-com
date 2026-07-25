# Octodoge website

Static pages for [octodoge.com](https://octodoge.com).

The iOS app lives in [GitHubClient](https://github.com/iliaskarim/GitHubClient). App Store listing copy and screenshots live in [octodoge-metadata](https://github.com/iliaskarim/octodoge-metadata).

## Pages

| Path | App Store Connect field |
| --- | --- |
| `/` | Marketing URL |
| `/support/` | Support URL |
| `/privacy/` | Privacy Policy URL |

## Local

From the repo root:

```bash
python3 -m http.server 8765
```

Then open [http://localhost:8765](http://localhost:8765).

## Deploy

From the repo root:

```bash
./scripts/upload-website
```

Requires the [AWS CLI](https://aws.amazon.com/cli/) with credentials that can write to the bucket.
Defaults to `s3://octodoge.com` and CloudFront distribution `E39DLYSS5BGSGK` (invalidates `/*` after each sync).
Override with `OCTODOGE_S3_BUCKET`, `CLOUDFRONT_DISTRIBUTION_ID`, or `AWS_PROFILE` if needed.

CloudFront (one-time setup for clean `/privacy/` and `/support/` URLs with an S3 REST origin):

1. **Default root object:** `index.html` (General → Edit)
2. **Viewer-request function:** append `/index.html` to paths without a file extension (and to
   trailing-slash paths). Publish the function to **Live**, then associate it on the default
   behavior.

Ensure these URLs load over **HTTPS** before submitting to App Store Connect:

- `https://octodoge.com`
- `https://octodoge.com/support/`
- `https://octodoge.com/privacy/`

Support and privacy contact email: `ilias.karim@icloud.com`.
