# Octodoge website

Static pages for [octodoge.iliaskarim.net](https://octodoge.iliaskarim.net).

The iOS app lives in [GitHubClient](https://github.com/iliaskarim/GitHubClient). App Store listing copy and screenshots live in [octodoge-metadata](https://github.com/iliaskarim/octodoge-metadata).

## Pages

| Path | App Store Connect field |
| --- | --- |
| `/` | Marketing URL |
| `/support/` | Support URL |
| `/privacy/` | Privacy Policy URL |

## Deploy

From the repo root:

```bash
./scripts/upload-website
```

Requires the [AWS CLI](https://aws.amazon.com/cli/) with credentials that can write to the bucket.
Override the bucket with `OCTODOGE_S3_BUCKET`, the profile with `AWS_PROFILE`, or invalidate
CloudFront with `CLOUDFRONT_DISTRIBUTION_ID` after each deploy.

CloudFront (one-time setup for clean `/privacy/` and `/support/` URLs with an S3 REST origin):

1. **Default root object:** `index.html` (General → Edit)
2. **Viewer-request function:** append `/index.html` to paths without a file extension (and to
   trailing-slash paths). Publish the function to **Live**, then associate it on the default
   behavior.

Ensure the bucket serves **HTTPS** and that these URLs load before submitting to App Store Connect:

- `https://octodoge.iliaskarim.net`
- `https://octodoge.iliaskarim.net/support/`
- `https://octodoge.iliaskarim.net/privacy/`

Support and privacy contact email: `ilias.karim@icloud.com`.
