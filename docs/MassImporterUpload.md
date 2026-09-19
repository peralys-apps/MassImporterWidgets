## Mass Importer Upload

Upload files for validation and import via the Mass Importer API. The widget presents a dropzone and upload button, sends the selected file to Mass Importer, and (optionally) automatically starts validation and polls for its progress, surfacing status through progress indicators and action events. It is the entry point of the pipeline: files uploaded here become an import that the companion Mass Importer Review widget can display and let users act on.

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

The widget's **Max File Size (MB)** property (default 50) can only lower this limit for the widget instance, never raise it above what the organization's plan allows; the server enforces the actual ceiling regardless of what is configured here. Other capabilities, such as Parquet export of the resulting data, are gated by plan tier in the companion Mass Importer Review widget, not in this widget.

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
<tr><td>API Key (<code>apiKey</code>)</td><td>Expression (String)</td><td>Yes</td><td>none</td><td>API key for authentication, sent as a Bearer token. See "API key and security" below.</td></tr>
<tr><td>Template Mode (<code>templateMode</code>)</td><td>Enumeration: Template Picker / Fixed Template</td><td>No</td><td><code>picker</code></td><td>How the import template is determined. "Template Picker" lets the end user choose a template; "Fixed Template" always uses the template configured in Template Slug.</td></tr>
<tr><td>Template Slug (<code>templateSlug</code>)</td><td>Expression (String)</td><td>No</td><td>none</td><td>Slug of the template to use when Template Mode is "Fixed Template". Ignored in picker mode.</td></tr>
<tr><td>External ID (<code>externalId</code>)</td><td>Expression (String)</td><td>No</td><td>none</td><td>Optional external identifier used to partition imports, for example a customer ID or project ID. Matches the "External ID" property on the Review widget so imports can be filtered per tenant.</td></tr>
<tr><td>Accepted File Types (<code>acceptedFileTypes</code>)</td><td>String</td><td>No</td><td><code>.csv,.xlsx,.parquet</code></td><td>Comma-separated list of accepted file extensions.</td></tr>
<tr><td>Max File Size (MB) (<code>maxFileSizeMB</code>)</td><td>Integer</td><td>No</td><td><code>50</code></td><td>Upload limit in MB. Only lowers the organization's plan limit (Free 50, Starter/Pro 500, Enterprise 2048), never raises it.</td></tr>
<tr><td>Auto Start Validation (<code>autoStartValidation</code>)</td><td>Boolean</td><td>No</td><td><code>true</code></td><td>Automatically starts validation after upload completes.</td></tr>
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

### API key and security

The API Key property is a Studio Pro **expression**, evaluated in the end user's browser. The resulting bearer token is present in the page delivered to the client, which means any end user of the Mendix app who can inspect network requests or page state can read it and call the Mass Importer API directly with that key. This is a known limitation of a client-side widget property, not an oversight. Plan for the key being readable by everyone who can open the page.

**What the key is scoped to.** A Mass Importer API key is scoped to exactly one Mass Importer organization. It cannot read or write another organization's data, and that boundary is enforced on the server on every request. Below the organization it is not scoped at all: within its own organization the key has full access. There is no per template, per import, or per external ID key scope, and rate limits apply to the organization rather than to an individual key.

**What a leaked key reaches.** Every template, every import, and every row in that organization, for reading and for writing, plus webhook configuration and data exports. Treat the key as equivalent to handing out access to the whole organization.

**External ID is a filter unless the key is bound to one.** The External ID property narrows what the widget requests. With an ordinary key it is supplied by the caller, so anyone holding the key can simply omit it and list everything in the organization. To make it a real boundary, create the API key bound to an external ID in the Mass Importer dashboard: a bound key can only see and create imports carrying that value, and the server enforces it on every request. When you use a bound key, set this widget's External ID property to the same value or leave it empty; a different value is rejected with EXTERNAL_ID_MISMATCH.

Because of all of this:

- Use a dedicated Mass Importer organization for widget traffic, holding only the templates and imports that the app's end users are meant to reach. Keep administrative work and any data they should not see in a separate organization.
- Never put a key belonging to your main organization into this property.
- Revoke the key as soon as you suspect exposure. Revocation takes effect on the next request made with it.

### Data handling

The full contents of any file selected in this widget leave the Mendix app and are uploaded to `massimporter.com` for validation and import. The widget does not parse, validate, or store file contents locally; it exists to move the file to the Mass Importer API and report back on progress and status. If your organization needs to know exactly what is transmitted and retained, refer to the Mass Importer data processing terms at massimporter.com.

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
