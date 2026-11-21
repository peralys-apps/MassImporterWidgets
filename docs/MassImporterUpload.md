## Mass Importer Upload

Upload files for validation and import via the Mass Importer API. The widget presents a dropzone and upload button, lets the user check how the file's columns map to the template's fields, sends the selected file to Mass Importer, which always validates it, and by default follows that validation, polling for its progress and surfacing status through progress indicators and action events. It is the entry point of the pipeline: files uploaded here become an import that the companion Mass Importer Review widget can display and let users act on.

### Requirement: a Mass Importer account

This widget is a thin client for [Mass Importer](https://massimporter.com), a metered, commercial, hosted service. It does not process or store files itself; every upload is sent to massimporter.com for validation and import. Without a Mass Importer account and an API key, the widget cannot upload anything. Sign up at massimporter.com to get an organization and an API key.

Mass Importer's plan tiers set the maximum file size the service will accept:

<table>
<thead>
<tr><th>Plan</th><th>Max file size</th></tr>
</thead>
<tbody>
<tr><td>Free</td><td>50 MB</td></tr>
<tr><td>Starter</td><td>500 MB</td></tr>
<tr><td>Pro</td><td>500 MB</td></tr>
<tr><td>Enterprise</td><td>2048 MB</td></tr>
</tbody>
</table>

The widget's **Max File Size (MB)** property (default 2048, so the plan limit applies unless you set a lower value) can only lower this limit for the widget instance, never raise it above what the organization's plan allows; the server enforces the actual ceiling regardless of what is configured here. Parquet upload requires a Pro or Enterprise plan: on Free and Starter this widget does not offer Parquet, even if **Accepted File Types** lists it, and refuses a Parquet file before upload. A template with more than one sheet accepts only XLSX, on every plan; see **Multi-sheet templates need XLSX** below. Parquet export of the resulting data is gated by plan tier in the companion Mass Importer Review widget.

### Installation

1. Download the `.mpk` file for this widget, or import it directly from the Marketplace inside Studio Pro.
2. In Studio Pro, go to **App Store** > **Import widget** and select the `.mpk` file (skip this step if installing from the Marketplace directly).
3. Minimum Studio Pro version: **11.12 LTS**.
4. Drag the **Mass Importer Upload** widget onto a page from the widget toolbox. The widget needs an entity context, so place it inside a data view or another context-providing container.
5. Set the required **API Key** property (see Configuration below) and choose a Template Mode.

### Configuration

All properties as declared in `MassImporterUpload.xml`.

<table>
<thead>
<tr><th>Property</th><th>Type</th><th>Required</th><th>Default</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>API Key (<code>apiKey</code>)</td><td>Expression (String)</td><td>Yes</td><td>none</td><td>API key for authentication, sent as a Bearer token. See "Choosing an API key" below.</td></tr>
<tr><td>Template Mode (<code>templateMode</code>)</td><td>Enumeration: Template Picker / Fixed Template</td><td>No</td><td><code>picker</code></td><td>How the import template is determined. "Template Picker" lets the end user choose a template; "Fixed Template" always uses the template configured in Template Slug.</td></tr>
<tr><td>Template Slug (<code>templateSlug</code>)</td><td>Expression (String)</td><td>No</td><td>none</td><td>Slug of the template to use when Template Mode is "Fixed Template". Ignored in picker mode.</td></tr>
<tr><td>External ID (<code>externalId</code>)</td><td>Expression (String)</td><td>No</td><td>none</td><td>Optional external identifier used to partition imports, for example a customer ID or project ID. Matches the "External ID" property on the Review widget so imports can be filtered per tenant.</td></tr>
<tr><td>Accepted File Types (<code>acceptedFileTypes</code>)</td><td>String</td><td>No</td><td><code>.csv,.xlsx,.parquet</code></td><td>Comma-separated list of accepted file extensions. A template with more than one sheet takes XLSX only, so a list without <code>.xlsx</code> cannot import into one.</td></tr>
<tr><td>Max File Size (MB) (<code>maxFileSizeMB</code>)</td><td>Integer</td><td>No</td><td><code>2048</code></td><td>Upload limit in MB. Only lowers the organization's plan limit (Free 50, Starter/Pro 500, Enterprise 2048), never raises it.</td></tr>
<tr><td>Auto Start Validation (<code>autoStartValidation</code>)</td><td>Boolean</td><td>No</td><td><code>true</code></td><td>Follow validation in the widget after upload. Mass Importer always validates the file; when off, the widget does not show the result, and On Validation Complete and On Review Results never fire.</td></tr>
<tr><td>Poll Interval (ms) (<code>pollIntervalMs</code>)</td><td>Integer</td><td>No</td><td><code>5000</code></td><td>Interval, in milliseconds, for polling validation status.</td></tr>
<tr><td>Import ID Attribute (<code>importIdAttribute</code>)</td><td>Attribute (String)</td><td>No</td><td>none</td><td>Attribute that receives the resulting import ID.</td></tr>
<tr><td>On Upload Start (<code>onUploadStart</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when file upload starts.</td></tr>
<tr><td>On Upload Complete (<code>onUploadComplete</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when file upload completes.</td></tr>
<tr><td>On Validation Complete (<code>onValidationComplete</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when validation finishes.</td></tr>
<tr><td>On Error (<code>onError</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when an error occurs.</td></tr>
<tr><td>On Review Results (<code>onReviewResults</code>)</td><td>Action</td><td>No</td><td>none</td><td>Triggered when the user clicks Review Results, typically used to navigate to a page containing the Mass Importer Review widget.</td></tr>
<tr><td>Show Progress (<code>showProgress</code>)</td><td>Boolean</td><td>No</td><td><code>true</code></td><td>Shows progress indicators during upload and validation.</td></tr>
<tr><td>Upload Button Label (<code>uploadButtonLabel</code>)</td><td>String</td><td>No</td><td><code>Select File</code></td><td>Label text for the upload button.</td></tr>
<tr><td>Dropzone Text (<code>dropzoneText</code>)</td><td>String</td><td>No</td><td><code>Drag and drop a file here, or click to select</code></td><td>Text displayed inside the dropzone area.</td></tr>
</tbody>
</table>

17 properties in total.

### Steps and column mapping

The widget shows its progress as a row of steps. In Template Picker mode they are Select, File, Map, Upload, Validate and Done. In Fixed Template mode the Select step is dropped.

After the user chooses a file, the **Map** step lists every field of the template next to a drop-down of the columns in the file. It always appears, and there is no property to turn it off.

- Each field is filled in with the automatic match (the field key or label, ignoring case, spaces, underscores and hyphens). A file whose headers already fit the template needs no changes.
- The user can pick another column for any field, or set it to **Not mapped**.
- Required fields are labelled **Required**. If one is left unmapped the step warns, the user can still continue, and every row is then reported invalid for that field.
- Sample values from the first three data rows are shown beside each choice. Parquet files show no sample values.
- A column that feeds more than one field is flagged, and columns that feed no field are listed as not imported.
- When the template has several sheets, there is a tab per sheet. Workbook tabs go to the template sheet with the same name first, then to a template sheet whose slug matches the tab name (both ignoring case), then template sheets still without a tab take the remaining tabs in order. A template sheet left without a tab imports nothing, and the step says so.
- **Continue to upload** sends the file together with the mapping, covering every field of every sheet that will run, including the fields the user cleared.

The widget reads the column names and sample values from slices of the file in the user's browser, never the whole file, and fetches the template's fields with the configured API key. If it cannot, the step says why and offers **Continue with automatic matching**; the server then matches columns by name, as earlier widget versions always did. This happens when:

- the browser cannot read the file's column names, for example an older browser that cannot unpack XLSX files;
- an XLSX column name is a formatted number or a date;
- the file has no data rows, or the template's fields cannot be loaded.

### Multi-sheet templates need XLSX

A CSV or Parquet file holds one sheet, so a template with more than one sheet accepts only an Excel (`.xlsx`) file with one tab per sheet. No plan changes this. As soon as the widget has read the template's sheets, the drop area offers XLSX alone, says why and uses its own text in place of **Dropzone Text**. Any other file is refused with a message and **Choose a different file**, and nothing is uploaded. If the widget learns the sheet count only after a file was chosen, it refuses at that moment. If it cannot read the sheets at all, nothing is blocked here and the server refuses the file after the upload, with the same choice. A workbook with fewer tabs than the template has sheets is accepted: the Map step says which sheet it does not fill. A template with one sheet is not affected.

To read the sheets, the widget calls `GET /api/v1/templates` and `GET /api/v1/template-versions/{id}/sheets` with the configured API key when the template is known, before any file is chosen.

### Choosing an API key

The widget calls Mass Importer from the user's browser, so the key is sent with those requests, as with any widget that talks to a web service directly. Create a key just for this widget and limit it to what the page needs:

- **Templates:** only the templates this page uses.
- **External ID:** bind the key to one customer or tenant, so it can only see their imports. Set this widget's External ID property to the same value or leave it empty; a different value is rejected with EXTERNAL_ID_MISMATCH.
- **Read-only:** leave it off. This widget creates imports, which a read-only key cannot do.
- **Rate limit:** optionally its own cap on requests per minute.

A key only ever reaches your own organization's data, within those limits. You can revoke it at any time in the Mass Importer dashboard, and the change applies to the next request. If a key must never be sent from the browser, call the Mass Importer REST API from a microflow instead.

### Data handling

The full contents of any file selected in this widget leave the Mendix app and are uploaded to `massimporter.com` for validation and import. Before the upload, the widget reads the column names and the first three data rows of the file in the user's browser to fill the Map step; that read stays in the browser and is not sent anywhere separately. The widget does not validate or store file contents; it exists to move the file to the Mass Importer API and report back on progress and status. If your organization needs to know exactly what is transmitted and retained, refer to the Mass Importer data processing terms at massimporter.com.

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
