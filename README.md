# Mass Importer Mendix Widgets

Two Mendix pluggable widgets that add bulk data ingestion to a Mendix app:

- **[Mass Importer Upload](docs/MassImporterUpload.md)**, which uploads a CSV, Excel or Parquet file against a Mass Importer template and starts validation.
- **[Mass Importer Review](docs/MassImporterReview.md)**, which renders a review grid over a validated import: valid and error rows, inline cell editing, bulk edit, and export to CSV, Excel or Parquet.

The two are designed to be paired, but either can be used on its own.

## About this repository

This repository distributes the compiled widget packages and their documentation. **It does not publish source code.** The distributed artifact is the `.mpk` file attached to each entry on the [Releases](../../releases) page.

The widgets are licensed under the Apache License, Version 2.0, which does not require source publication. See [LICENSE](LICENSE) and [NOTICE](NOTICE).

## Requirement: a Mass Importer account

These widgets are thin clients for [Mass Importer](https://massimporter.com), a metered commercial service. They do not work standalone. Without a Mass Importer organization and an API key, neither widget has anything to talk to.

Plan tiers change limits, never access. Both widgets work on every tier, and the Free tier is the intended place to build and test an embed before paying.

<table>
<thead>
<tr><th>Plan</th><th>Max file size</th><th>Parquet import and export</th></tr>
</thead>
<tbody>
<tr><td>Free</td><td>50 MB</td><td>Not available</td></tr>
<tr><td>Starter</td><td>500 MB</td><td>Not available</td></tr>
<tr><td>Pro</td><td>500 MB</td><td>Available</td></tr>
<tr><td>Enterprise</td><td>2048 MB</td><td>Available</td></tr>
</tbody>
</table>

A 403 response from a widget is a plan-limit rejection (rows, monthly volume, or concurrent imports), not a missing entitlement.

## Installation

1. Download the `.mpk` for the widget you want from the [Releases](../../releases) page.
2. In Studio Pro, choose **App Store** > **Import widget** and select the `.mpk`.
3. Drag the widget onto a page from the widget toolbox.
4. Set the required **API Key** property. See the per-widget documentation for the full property list.

Minimum Studio Pro version: **11.12 LTS**.

## Verifying a download

Every release lists the SHA-256 of each `.mpk` in its release notes. Check it before importing:

```
sha256sum peralys.MassImporterReview.mpk
```

Compare the output against the hash in the release notes. If they differ, do not import the file.

## API key and security

Both widgets take the API key as a widget property. Mendix evaluates that expression in the browser, so the resulting bearer token is present in the delivered page and is readable by any end user of the app.

Use a scoped, low-privilege, per-tenant API key. Do not use an administrative key. Scope is enforced server-side by Mass Importer, so a key restricted to one organization cannot reach another.

This is a documented characteristic of the current design, not an oversight. If your deployment cannot accept a browser-readable key, proxy the calls through a Mendix microflow or REST action so the key stays server-side.

## Data handling

Uploaded file contents leave the Mendix application and are transmitted to massimporter.com, where they are parsed, validated and stored so that rows can be reviewed and exported. Retention is governed by the plan tier of the Mass Importer organization. Review the Mass Importer privacy policy and terms before uploading personal data.

## Support

Report issues on the [issue tracker](../../issues) for this repository. Include the widget name, its version, your Studio Pro version, and the steps to reproduce.

## License

Licensed under the Apache License, Version 2.0. The full, unmodified licence text is in [LICENSE](LICENSE), and is also published at https://www.apache.org/licenses/LICENSE-2.0

Copyright 2026 Peralys

Third-party open-source components bundled in the distributed packages, with their licences and required attributions, are listed in [NOTICE](NOTICE).
# MassImporterWidgets
