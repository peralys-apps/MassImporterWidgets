## Mass Importer Review

Fetch and display validated import data from the Mass Importer API. The widget renders an AG Grid based review surface over an import created by the Mass Importer Upload widget (or by any other Mass Importer client): a list of imports, a row grid with valid and error rows, inline cell editing, bulk edit across invalid rows, and an export menu (CSV, Excel, Parquet).

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

The grid reads the organization's current plan tier from the API. On Free and Starter, the Parquet option in the export menu is shown locked with a message that it requires the Pro plan; CSV and Excel export remain available on every tier. Parquet stays locked until the plan is known. While the widget is still checking the plan, the Parquet option says so. If the plan cannot be read (for example, after a network failure and one retry), Parquet stays locked with a message that it is unavailable right now; the widget does not ask you to upgrade in that case.

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
<tr><td>API Key (<code>apiKey</code>)</td><td>Expression (String)</td><td>Yes</td><td>none</td><td>API key for authentication, sent as a Bearer token. See "Choosing an API key" below.</td></tr>
<tr><td>Import ID (<code>importId</code>)</td><td>Expression (String)</td><td>No</td><td>none</td><td>The ID of a specific import to review. If left blank, the widget shows a list of all imports instead of a single import's rows.</td></tr>
<tr><td>External ID (<code>externalId</code>)</td><td>Expression (String)</td><td>No</td><td>none</td><td>Filters the import list to only show imports created with this external ID. Matches the "External ID" property on the Upload widget so the two widgets can be paired per customer, project, or tenant.</td></tr>
<tr><td>Status Filter (<code>statusFilter</code>)</td><td>String</td><td>No</td><td><code>queued,profiling,mapping,validating,validated,completed</code></td><td>Comma-separated list of import statuses to include in the import list.</td></tr>
<tr><td>Auto Fetch (<code>autoFetch</code>)</td><td>Boolean</td><td>No</td><td><code>true</code></td><td>Automatically fetch data when the widget loads, without waiting for a user action.</td></tr>
<tr><td>Rows Per Page (<code>previewLimit</code>)</td><td>Integer</td><td>No</td><td><code>500</code></td><td>Number of rows fetched and displayed per page (1 to 500).</td></tr>
<tr><td>Widget Height (<code>widgetHeight</code>)</td><td>String</td><td>No</td><td><code>calc(100vh - 10em)</code></td><td>CSS height for the widget container, for example <code>600px</code>, <code>100vh</code>, or a <code>calc()</code> expression.</td></tr>
<tr><td>Show Preview (<code>showPreview</code>)</td><td>Boolean</td><td>No</td><td><code>true</code></td><td>Shows a preview table of the data.</td></tr>
<tr><td>Imported Rows Attribute (<code>importedRowsAttribute</code>)</td><td>Attribute (String)</td><td>No</td><td>none</td><td>The attribute the widget writes the JSON payload of imported rows to when "Import All" is clicked, before triggering On Import All. If left blank, the JSON is passed only through the <code>rowsJson</code> action variable on On Import All.</td></tr>
<tr><td>On Data Loaded (<code>onDataLoaded</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when data is successfully loaded.</td></tr>
<tr><td>On Error (<code>onError</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when an error occurs.</td></tr>
<tr><td>On Import All (<code>onImportAll</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered by Import All, which passes the valid rows of every sheet, up to 5,000 valid rows per sheet. If invalid rows remain, or a sheet has more than 5,000 valid rows, the user confirms first, and the confirmation names the sheets over the limit. For larger imports, read the rows from the Mass Importer REST API or export them. Carries a <code>rowsJson</code> action variable (String) with the row payload.</td></tr>
</tbody>
</table>

12 properties in total.

### Using the imported rows in your app

The widget imports nothing itself. Import All passes a JSON string to your On Import All microflow, in the Imported Rows Attribute (written before the action fires) or the `rowsJson` action variable.

`sheets` has one entry per template sheet, with its `sheetSlug`, `sheetName` and valid `rows`. Each row is keyed by the template's field keys, not the column labels, and values are typed by field: text, numbers, `true` or `false`, and dates as `YYYY-MM-DD` text.

```json
{
  "sheets": [
    {
      "sheetSlug": "products",
      "sheetName": "Products",
      "rows": [
        { "sku": "AB100", "unitprice": 12.5, "instock": true, "releasedate": "2026-03-01" },
        { "sku": "AB101", "unitprice": 8.75, "instock": false, "releasedate": "2026-04-15" }
      ]
    }
  ]
}
```

In Studio Pro:

1. **Get a sample.** Show the Imported Rows Attribute in a text area for now, or log `$rowsJson`, run Import All on a test file and copy the JSON. Give every field a real value, optional ones included: a field missing from the sample, or `null` in it, cannot be mapped. Give decimal fields a fraction, such as 12.5, so they are typed as Decimal.
2. **Create a JSON Structure** from the sample.
3. **Create an Import Mapping** from it. Map the root and the sheet to non-persistable helper entities, and each row to your own entity: numbers to Decimal (or Integer or Long), booleans to Boolean, text to String. Studio Pro reads a date without a time as a String, so map it to a String attribute, or to a Date and time attribute through a Convert using microflow returning `parseDateTimeUTC($value, 'yyyy-MM-dd')`, with Localize off so the day does not shift.
4. **In the On Import All microflow**, add **Import with mapping** on `$rowsJson` (or the attribute). Its Commit setting saves your objects. Then show your own confirmation; the widget shows none.

Good to know:

- Field keys change only with the template. A new template version with new fields needs a new sample and a regenerated JSON Structure.
- An unmapped field is left out of the row, and an empty cell arrives as `null`. A value corrected in the grid can arrive as text, such as `"12.50"`.
- Import All sends at most 5,000 valid rows per sheet, never invalid ones. For more, read `GET /api/v1/imports/{importId}/rows?view=valid` from a microflow, paged with `start` and `limit` (up to 5,000) per `sheetSlug`. It wraps each row's values in `data`, so it needs its own JSON Structure.

### Exporting rows

The **Export** menu offers a CSV file, an Excel workbook and a Parquet file, each with **All rows**, **Valid rows** and **Error rows**.

- **CSV and Parquet** export the sheet that is open in the grid. **Excel** exports a workbook with every sheet of the import, one tab per sheet, so on an import with several sheets the Excel options follow the totals for the whole import.
- **Valid rows** and **Error rows** are hidden when there are no such rows to export. On a sheet with no rows at all, its CSV and Parquet options are shown disabled. The Export button itself is disabled only when nothing can be exported in any format.
- **Saved edits are included.** Every export contains the edits you have saved, inline or bulk.
- **Unsaved edits block export.** While the grid holds edits that are not saved yet, the Export button is disabled with the hint "Save or discard your edits first". Save or discard them, then export.
- **A running bulk fix blocks export.** While a bulk fix is still applying to the import, the Export button is disabled with the hint "A bulk fix is running. Export when it finishes." so an export never mixes fixed and unfixed rows.
- **Parquet** needs the Pro or Enterprise plan and stays locked until the plan is known (see the plan table above).

### Choosing an API key

The widget calls Mass Importer from the user's browser, so the key is sent with those requests, as with any widget that talks to a web service directly. Create a key just for this widget and limit it to what the page needs:

- **Templates:** only the templates this page uses.
- **External ID:** bind the key to one customer or tenant, so it can only see their imports. Set this widget's External ID property to the same value or leave it empty; a different value is rejected with EXTERNAL_ID_MISMATCH.
- **Read-only:** fine for a page that only shows, exports and imports rows. Leave it off if users save edits or run bulk fixes here.
- **Rate limit:** optionally its own cap on requests per minute.

A key only ever reaches your own organization's data, within those limits. You can revoke it at any time in the Mass Importer dashboard, and the change applies to the next request. If a key must never be sent from the browser, call the Mass Importer REST API from a microflow instead.

### Data handling

The widget fetches import and row data from the Mass Importer API at `massimporter.com`. Data displayed in the grid, and any edits made through inline or bulk edit, are read from and written to `massimporter.com`. No import or row data is stored by the widget itself; it is a display and editing surface over the remote service. If your organization needs to know exactly what is transmitted and retained, refer to the Mass Importer data processing terms at massimporter.com.

### Support

File issues, suggestions, and feature requests at [github.com/peralys-apps/MassImporterWidgets/issues](https://github.com/peralys-apps/MassImporterWidgets/issues). Include the widget name, its version, your Studio Pro version, and the steps to reproduce.

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
