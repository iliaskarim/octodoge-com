# Octodoge website

Static pages for [octodoge.com](https://octodoge.com).

The iOS app lives in [GitHubClient](https://github.com/iliaskarim/GitHubClient). App Store listing copy lives in [octodoge-metadata](https://github.com/iliaskarim/octodoge-metadata). Screenshot sources live in [octodoge-screenshots](https://gitlab.com/iliaskarim/octodoge-screenshots).

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

CloudFront (one-time setup for clean `/privacy/` and `/support/` URLs with an S3 REST origin,
plus Universal Links):

1. **Default root object:** `index.html` (General → Edit)
2. **Viewer-request function:** publish to **Live**, then associate it on the default behavior.
   Requirements (see sample below):
   - Leave `/.well-known/*` untouched — the AASA file has no extension, so the old
     “append `/index.html`” rewrite would break Universal Links.
   - Keep marketing paths on octodoge.com (`/`, `/privacy*`, `/support*`, `/assets/*`, and
     root static files).
   - For other paths (GitHub-shaped), **302** to `https://github.com{path}` so browsers without
     the app land on GitHub. Installed-app Universal Links open Octodoge before this runs.
   - Still append `/index.html` for trailing-slash / extensionless **site** paths.

Sample CloudFront Function (`cloudfront-js-2.0`):

```javascript
function handler(event) {
  var request = event.request;
  var uri = request.uri;

  if (uri === '/.well-known' || uri.indexOf('/.well-known/') === 0) {
    return request;
  }

  var isSite =
    uri === '/' ||
    uri === '/index.html' ||
    uri === '/styles.css' ||
    uri === '/robots.txt' ||
    uri === '/sitemap.xml' ||
    uri === '/privacy' ||
    uri.indexOf('/privacy/') === 0 ||
    uri === '/support' ||
    uri.indexOf('/support/') === 0 ||
    uri.indexOf('/assets/') === 0;

  if (!isSite) {
    return {
      statusCode: 302,
      statusDescription: 'Found',
      headers: {
        location: { value: 'https://github.com' + uri },
      },
    };
  }

  if (uri.endsWith('/')) {
    request.uri = uri + 'index.html';
  } else if (uri.indexOf('.') === -1) {
    request.uri = uri + '/index.html';
  }
  return request;
}
```

### Universal Links

`/.well-known/apple-app-site-association` claims GitHub-shaped paths for the Octodoge iOS apps
and excludes marketing / static / well-known paths. Deploy sets `Content-Type: application/json`
(required; no file extension). After deploy + CloudFront function update, verify:

```bash
curl -sI https://octodoge.com/.well-known/apple-app-site-association
# Expect: 200, content-type: application/json, no redirect

curl -sI https://octodoge.com/iliaskarim/GitHubClient
# Expect: 302 Location: https://github.com/iliaskarim/GitHubClient
```

Associated Domains in the app: `applinks:octodoge.com`, `applinks:www.octodoge.com`
(see [OCT-238](https://linear.app/octodoge/issue/OCT-238) / GitHubClient).

Ensure these URLs load over **HTTPS** before submitting to App Store Connect:

- `https://octodoge.com`
- `https://octodoge.com/support/`
- `https://octodoge.com/privacy/`
- `https://octodoge.com/.well-known/apple-app-site-association`

Support and privacy contact email: `ilias.karim@icloud.com`.
