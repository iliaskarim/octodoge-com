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

Merges (and pushes) to `main` run [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml), which calls the same upload script used locally. You can also trigger **Deploy** manually from the Actions tab.

### One-time GitHub → AWS setup (OIDC)

1. In IAM, create an identity provider for GitHub OIDC if you do not already have one:
   - Provider URL: `https://token.actions.githubusercontent.com`
   - Audience: `sts.amazonaws.com`
2. Create an IAM role trusted by that provider. Trust policy (adjust account/repo as needed):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:iliaskarim/octodoge-com:*"
        }
      }
    }
  ]
}
```

3. Attach a policy that allows syncing the site bucket and invalidating CloudFront, for example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::octodoge.com"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::octodoge.com/*"
    },
    {
      "Effect": "Allow",
      "Action": ["cloudfront:CreateInvalidation"],
      "Resource": "arn:aws:cloudfront::ACCOUNT_ID:distribution/E39DLYSS5BGSGK"
    }
  ]
}
```

4. In the GitHub repo, add secret `AWS_ROLE_ARN` with that role’s ARN.
5. Optional: set repository variable `AWS_REGION` (defaults to `us-east-1`).

### Manual deploy

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
