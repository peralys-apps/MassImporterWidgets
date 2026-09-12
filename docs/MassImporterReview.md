## Mass Importer Review

Fetch and display validated import data from the Mass Importer API. The widget renders an AG Grid based review surface over an import created by the Mass Importer Upload widget (or by any other Mass Importer client): a list of imports, a row grid with valid and error rows, inline cell editing, bulk edit across selected rows, and an export menu (CSV, Excel, Parquet).

### Requirement: a Mass Importer account

This widget is a thin client for [Mass Importer](https://massimporter.com), a metered, commercial, hosted service. It does not work standalone: without a Mass Importer account and an API key, the widget has nothing to display. Sign up at massimporter.com to get an organization and an API key.

Mass Importer's plan tiers gate what the widget can show and export:

<table>
<thead>
<tr><th>Plan</th><th>Upload cap (set on the Upload widget)</th><th>Parquet export</th></tr>
</thead>
<tbody>
<tr><td>Free</td><td>50 MB</td><td>Not available</td></tr>
<tr><td>Starter</td><td>500 MB</td><td>Not available</td></tr>
<tr><td>Pro</td><td>500 MB</td><td>Available</td></tr>
<tr><td>Enterprise</td><td>2048 MB</td><td>Available</td></tr>
</tbody>
</table>

The grid reads the organization's current plan tier from the API. On Free and Starter, the Parquet option in the export menu is shown locked with a message that it requires the Pro plan; CSV and Excel export remain available on every tier. If the plan tier cannot be determined (for example, a network failure), the widget fails closed and treats Parquet as locked, the same as on Free.

This widget does not enforce row limits or file-size limits itself; those are enforced by the Mass Importer API and, on upload, by the companion Mass Importer Upload widget's <code>maxFileSizeMB</code> property.

### Installation

1. Download the `.mpk` file for this widget, or import it directly from the Marketplace inside Studio Pro.
2. In Studio Pro, go to **App Store** > **Import widget** and select the `.mpk` file (skip this step if installing from the Marketplace directly).
3. Minimum Studio Pro version: **11.12 LTS**.
4. Drag the **Mass Importer Review** widget onto a page from the widget toolbox.
5. Set the required **API Key** property (see Configuration below) and, optionally, an **Import ID** to review a single import.

### Configuration

All properties as declared in `MassImporterReview.xml`.

<table>
<thead>
<tr><th>Property</th><th>Type</th><th>Required</th><th>Default</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>API Key (<code>apiKey</code>)</td><td>Expression (String)</td><td>Yes</td><td>none</td><td>API key for authentication, sent as a Bearer token. See "API key and security" below.</td></tr>
<tr><td>Import ID (<code>importId</code>)</td><td>Expression (String)</td><td>No</td><td>none</td><td>The ID of a specific import to review. If left blank, the widget shows a list of all imports instead of a single import's rows.</td></tr>
<tr><td>External ID (<code>externalId</code>)</td><td>Expression (String)</td><td>No</td><td>none</td><td>Filters the import list to only show imports created with this external ID. Matches the "External ID" property on the Upload widget so the two widgets can be paired per customer, project, or tenant.</td></tr>
<tr><td>Status Filter (<code>statusFilter</code>)</td><td>String</td><td>No</td><td><code>queued,profiling,mapping,validating,validated,completed</code></td><td>Comma-separated list of import statuses to include in the import list.</td></tr>
<tr><td>Auto Fetch (<code>autoFetch</code>)</td><td>Boolean</td><td>No</td><td><code>true</code></td><td>Automatically fetch data when the widget loads, without waiting for a user action.</td></tr>
<tr><td>View Filter (<code>viewFilter</code>)</td><td>Enumeration: All Rows / Valid Rows Only / Error Rows Only</td><td>No</td><td><code>all</code></td><td>Filters which rows are displayed in the grid.</td></tr>
<tr><td>Rows Per Page (<code>previewLimit</code>)</td><td>Integer</td><td>No</td><td><code>500</code></td><td>Number of rows fetched and displayed per page (1 to 500).</td></tr>
<tr><td>Widget Height (<code>widgetHeight</code>)</td><td>String</td><td>No</td><td><code>calc(100vh - 10em)</code></td><td>CSS height for the widget container, for example <code>600px</code>, <code>100vh</code>, or a <code>calc()</code> expression.</td></tr>
<tr><td>Show Preview (<code>showPreview</code>)</td><td>Boolean</td><td>No</td><td><code>true</code></td><td>Shows a preview table of the data.</td></tr>
<tr><td>Imported Rows Attribute (<code>importedRowsAttribute</code>)</td><td>Attribute (String)</td><td>No</td><td>none</td><td>The attribute the widget writes the JSON payload of imported rows to when "Import All" is clicked, before triggering On Import All. If left blank, the JSON is passed only through the <code>rowsJson</code> action variable on On Import All.</td></tr>
<tr><td>On Data Loaded (<code>onDataLoaded</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when data is successfully loaded.</td></tr>
<tr><td>On Error (<code>onError</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when an error occurs.</td></tr>
<tr><td>On Import All (<code>onImportAll</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when importing all valid loaded rows. Carries a <code>rowsJson</code> action variable (String) with the row payload.</td></tr>
</tbody>
</table>

13 properties in total.

### API key and security

The API Key property is a Studio Pro **expression**, evaluated in the end user's browser. The resulting bearer token is present in the page delivered to the client, which means any end user of the Mendix app who can inspect network requests or page state can read it and call the Mass Importer API directly with that key.

This is a known limitation of a client-side widget property, not an oversight. Because of it:

- The key you put in this property must be a scoped, low-privilege, per-tenant API key, never an organization-wide administrative key.
- Scope and rate-limit the key on the Mass Importer side (per template, per import, or per external ID) so that a leaked key cannot be used to read or act on data outside its intended scope.
- Rotate the key if you suspect exposure.

### Data handling

The widget fetches import and row data from the Mass Importer API (`massimporter.com` by default). Data displayed in the grid, and any edits made through inline or bulk edit, are read from and written to `massimporter.com`. No import or row data is stored by the widget itself; it is a display and editing surface over the remote service. If your organization needs to know exactly what is transmitted and retained, refer to the Mass Importer data processing terms at massimporter.com.

### Support

File issues, suggestions, and feature requests at [github.com/PeralysLLC/MassImporterWidgets/issues](https://github.com/PeralysLLC/MassImporterWidgets/issues). Include the widget name, its version, your Studio Pro version, and the steps to reproduce.

## License

Licensed under the Apache License, Version 2.0. The full, unmodified licence text
is in the `LICENSE` file distributed alongside this widget, and is also published at
https://www.apache.org/licenses/LICENSE-2.0

Copyright 2026 Peralys

The source code for this widget is not published. The distributed artifact is the
compiled `.mpk` package, available from the releases page of the distribution
repository.

The distributed package bundles third-party open-source components. Each component,
its licence, and any required attribution are listed in the `NOTICE` file inside the
`.mpk`.
