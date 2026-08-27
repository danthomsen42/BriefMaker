# Legal Brief Builder

## Software Design Specification

## 1. Project Overview

Create a client-side web application called "Legal Brief Builder."

The application is intended to help law students create structured case briefs while studying legal cases.

The application must be deployable as a completely static website through GitHub Pages.

The application must NOT require a server, database, user account, authentication system, or backend API.

All user-created brief content must remain on the user's local device unless the user explicitly exports the brief to a file.

The application should be designed so that additional brief sections, instructions, examples, formatting options, and export formats can be added later without requiring a fundamental rewrite of the application.

The application should prioritize:

1. Privacy
2. Simplicity
3. Reliability
4. Easy navigation
5. Easy access to instructional material
6. Clean exported documents
7. Extensibility

---

# 2. Target Deployment Environment

The application will be hosted using GitHub Pages.

The application must therefore operate entirely through:

* HTML
* CSS
* JavaScript
* Client-side JavaScript libraries

Do not introduce:

* PHP
* Python server code
* Node.js server code
* Express
* SQL databases
* Firebase
* Supabase
* User accounts
* Server-side APIs

A build system may be used if beneficial, but the final deployment must consist of static assets suitable for GitHub Pages.

Prefer a simple architecture that can eventually be understood and maintained by a developer working primarily in VS Code.

---

# 3. Privacy Model

The application must be designed as a local-first application.

The user's case briefs must never be transmitted to a server.

The application must not make network requests containing user-created brief content.

The application may download JavaScript libraries or static resources necessary for operation.

The application should not collect analytics by default.

The application should not contain advertising.

The application should not require registration.

The application should clearly communicate to users that their draft material is stored locally in their browser unless they explicitly export it.

This is particularly important because users may enter confidential, personally identifiable, or otherwise sensitive legal information into their drafts.

---

# 4. Primary Brief Structure

The initial default brief template shall contain these sections:

1. Case Name, Court, Citation, Date
2. Facts
3. Procedural History
4. Issue(s)
5. Applicable Rule(s) of Law
6. Holding(s)
7. The Court's Order
8. Reasoning
9. New Information
10. Questions, Comments, and Speculations

The application must NOT treat these ten sections as immutable HTML elements.

Instead, sections must be represented as data objects.

Example conceptual structure:

```javascript
const briefSections = [
    {
        id: "case-information",
        title: "Case Name, Court, Citation, Date",
        type: "metadata",
        instructions: "...",
        examples: [...]
    },
    {
        id: "facts",
        title: "Facts",
        type: "rich-text",
        instructions: "...",
        examples: [...]
    }
];
```

The rendering system should dynamically generate the interface from this configuration.

Adding a new section should require adding a new configuration object rather than rewriting the user interface.

---

# 5. Section Configuration

Each section should support the following properties:

```javascript
{
    id: "unique-section-id",
    title: "Section Title",
    type: "text | textarea | rich-text | metadata",
    required: false,
    instructions: "Instructional text...",
    examples: [
        {
            title: "Example 1",
            source: "Source citation",
            text: "Example text..."
        }
    ]
}
```

Additional properties may be added later.

The architecture should allow future properties such as:

```javascript
placeholder
defaultValue
minimumLength
maximumLength
tags
category
courtLevel
subjectArea
```

without requiring major changes to the renderer.

---

# 6. User Interface

Each brief section should have three primary visual components:

```text
[Instruction]   [Section / Editor]   [Examples]
```

The left side contains an instructional help control.

The center contains the actual brief-writing field.

The right side contains an example control.

The controls should be visually unobtrusive and should not appear in exported documents.

---

# 7. Instruction Controls

Each section should have an instruction/help button positioned immediately to the left of the section.

The button should be visually recognizable as a help control, such as:

```text
?
```

or an information icon.

Hovering over the button should display the section's instructions.

Clicking the button should also display the instructions.

The application should support both hover and click interaction because hover-only interfaces are poor on touch devices.

The instructional popup should support:

* Text
* Paragraphs
* Bullet lists
* Bold text
* Italics
* Links where appropriate

The popup must not be included in exported documents.

---

# 8. Example Controls

Each section should have an example/help control on its right side.

The control should display an example of what an appropriately completed section might look like.

Examples should be stored independently from the UI.

Example structure:

```javascript
examples: [
    {
        id: "facts-example-1",
        title: "Example: International Shoe",
        source: "International Shoe Co. v. Washington",
        text: "..."
    },
    {
        id: "facts-example-2",
        title: "Example: Erie Railroad",
        source: "Erie Railroad Co. v. Tompkins",
        text: "..."
    }
]
```

---

# 9. Rotating Examples

The user should be able to rotate through multiple examples.

The example popup should include:

```text
Example 2 of 5

< Previous     Next >
```

The user should be able to move between examples without closing the popup.

The application should remember the currently selected example for the current session.

Examples should not be randomly selected every time the user opens the popup.

The user should have predictable Previous and Next controls.

---

# 10. Example Content and Copyright

The example system must be designed so that examples can be replaced or expanded later.

Do not scrape arbitrary modern student briefs from the Internet.

Where possible, use:

* Public-domain judicial opinions
* Government publications
* Court opinions
* Other appropriately licensed material
* Original examples written specifically for the application

If copyrighted professional legal briefs are later incorporated, they must be reviewed for appropriate permission or licensing before publication.

The application architecture should not depend on any particular example source.

---

# 11. Brief Editor

The central area of the application should contain the actual editable brief.

The editor should provide enough space for substantial legal writing.

At minimum, support:

* Plain text
* Paragraph breaks
* Automatic resizing of text areas
* Spellcheck using the browser's native spellcheck
* Undo/redo through normal browser editing behavior

Rich text formatting should NOT be required for version 1.

The initial implementation should favor plain text because it simplifies:

* Saving
* Loading
* PDF generation
* DOCX generation
* Portability
* Data serialization

Rich text can be added later if needed.

---

# 12. Case Information Section

The first section should be more structured than the other sections.

Provide separate fields for:

* Case Name
* Court
* Citation
* Date

Example conceptual data structure:

```javascript
caseInformation: {
    caseName: "",
    court: "",
    citation: "",
    date: ""
}
```

The remaining sections should use their configured editor type.

---

# 13. Automatic Saving

The application must automatically save the current draft locally.

Use IndexedDB as the primary persistent browser storage mechanism.

Do not rely exclusively on localStorage.

The application should save:

* Brief title
* Case information
* Section contents
* Creation date
* Last modified date
* Application version
* Template version

Example:

```javascript
{
    id: "generated-unique-id",
    title: "International Shoe v. Washington",
    createdAt: "...",
    modifiedAt: "...",
    applicationVersion: "1.0.0",
    templateVersion: "1.0",
    caseInformation: {...},
    sections: {...}
}
```

Autosaving should occur after changes, but should be debounced so that the database is not written on every keystroke.

For example, save approximately 500 to 1000 milliseconds after the user stops typing.

---

# 14. Draft Management

The application should have a "My Drafts" interface.

Users should be able to:

* Create a new brief
* Open an existing brief
* Rename a brief
* Duplicate a brief
* Delete a brief
* Export a brief
* Import a brief

The draft list should display:

* Brief title
* Case name
* Last modified date

Deleting a brief should require confirmation.

---

# 15. Draft File Export

In addition to browser storage, provide a portable draft format.

Use a custom extension:

```text
.brief
```

The file should contain serialized JSON data.

Example:

```json
{
    "format": "legal-brief-builder",
    "formatVersion": 1,
    "applicationVersion": "1.0.0",
    "brief": {
        ...
    }
}
```

The `.brief` format should contain the user's draft data but should NOT contain application UI state that is unnecessary for reconstructing the brief.

The user should be able to:

1. Click "Export Draft"
2. Download `CaseName.brief`
3. Move the file to another computer
4. Open the Legal Brief Builder
5. Click "Import Draft"
6. Select the `.brief` file
7. Continue editing

---

# 16. Import Validation

Imported files must be validated before being loaded.

Validate:

* File type
* JSON syntax
* Application format identifier
* Format version
* Required fields

Malformed or incompatible files should produce a clear error message rather than crashing the application.

The application should support migration of older `.brief` formats in the future.

---

# 17. Exported Documents

The application must support:

* PDF
* DOCX

The exported documents must contain ONLY the actual brief.

They must NOT contain:

* Instruction buttons
* Help popups
* Example buttons
* Example text
* Navigation controls
* Draft management controls
* Application branding unless specifically configured
* UI elements
* Hidden HTML elements from the editor

The exported document should be generated from the underlying brief data rather than attempting to print the application's HTML interface.

This is important.

Do NOT simply use:

```javascript
window.print()
```

as the primary export mechanism.

Instead, create a dedicated document-generation layer.

---

# 18. DOCX Generation

Use a browser-compatible JavaScript DOCX generation library.

The `docx` library is an appropriate candidate because it supports generating DOCX documents in browser environments.

The generated document should have professional basic formatting.

Initial formatting:

* Standard page size
* Standard margins
* Readable serif or sans-serif font
* Case name prominently displayed
* Section headings clearly distinguished
* Paragraph spacing
* No application controls

The formatting should be centralized in the DOCX export module so it can be changed later without changing the editor.

---

# 19. PDF Generation

Use a client-side PDF generation library.

The PDF should be generated from the brief data rather than by screenshotting or printing the editor.

The PDF generator should handle:

* Page breaks
* Section headings
* Paragraph wrapping
* Long sections
* Multiple pages
* Page numbering if feasible

The PDF should remain selectable/searchable text rather than an image.

---

# 20. Metadata / Provenance Marker

Every generated PDF and DOCX should contain a non-visible provenance marker indicating that the document was generated using Legal Brief Builder.

This is NOT intended to be cryptographically tamper-proof.

It should be treated as document provenance metadata.

Possible metadata:

```text
Creator: Legal Brief Builder
Producer: Legal Brief Builder
Keywords: legal-brief-builder
```

Where the file format permits custom properties, also include:

```text
LBB_Generated = true
LBB_Version = 1.0.0
LBB_Document_ID = [unique identifier]
LBB_Template_Version = 1.0
```

Generate a unique document identifier for each export.

Do not place the identifier visibly in the document.

Do not include hidden text inside the body merely for tracking purposes.

The application must not transmit these identifiers to a server.

The identifier is solely a local/document metadata marker.

The application documentation should explicitly state that metadata can be removed or altered by users and should therefore NOT be described as a tamper-proof watermark.

---

# 21. Metadata Privacy

Do not place the user's personal information into metadata unless the user explicitly entered that information into a corresponding field.

Do not automatically include:

* IP address
* Computer name
* Operating system username
* Browser fingerprint
* Geographic location
* Hardware identifiers

The application should not attempt to identify the person who created a document.

---

# 22. Application Architecture

Use a modular architecture.

Suggested structure:

```text
/
├── index.html
├── README.md
├── .nojekyll
│
├── css/
│   ├── main.css
│   ├── editor.css
│   ├── modals.css
│   └── print.css
│
├── js/
│   ├── app.js
│   ├── state.js
│   ├── storage.js
│   ├── drafts.js
│   ├── importer.js
│   ├── exporter.js
│   ├── pdf-exporter.js
│   ├── docx-exporter.js
│   ├── metadata.js
│   ├── ui.js
│   └── utils.js
│
├── data/
│   ├── sections.js
│   └── examples.js
│
└── assets/
    └── ...
```

The exact architecture may be adjusted if a framework is used, but the conceptual separation should remain.

---

# 23. State Management

Maintain one canonical application state.

Example:

```javascript
const appState = {
    currentDraftId: null,

    brief: {
        title: "",
        caseInformation: {
            caseName: "",
            court: "",
            citation: "",
            date: ""
        },

        sections: {}
    }
};
```

The editor should modify application state.

Storage should save application state.

Exporters should consume application state.

The UI should render application state.

Do not make the DOM itself the application's primary database.

---

# 24. Separation of Concerns

Keep these systems separate:

### Editor

Responsible for:

* Displaying sections
* Receiving user input
* Updating state

### Storage

Responsible for:

* IndexedDB
* Saving
* Loading
* Deleting
* Listing drafts

### Import/Export

Responsible for:

* `.brief` files
* JSON serialization
* Validation
* Migration

### Document Export

Responsible for:

* PDF
* DOCX
* Formatting
* Metadata

### Content

Responsible for:

* Instructions
* Examples
* Section definitions

This separation is important for future maintenance.

---

# 25. Responsive Design

The application should work on:

* Desktop
* Laptop
* Tablet

Desktop layout:

```text
┌──────────────────────────────────────────────────────────────┐
│                       LEGAL BRIEF BUILDER                    │
├──────────┬───────────────────────────────────────┬───────────┤
│    ?     │             Section Heading           │     ?     │
│          │                                       │           │
│          │             Editor                    │ Examples  │
│          │                                       │           │
├──────────┼───────────────────────────────────────┼───────────┤
│    ?     │             Section Heading           │     ?     │
│          │                                       │           │
│          │             Editor                    │ Examples  │
└──────────┴───────────────────────────────────────┴───────────┘
```

On smaller screens, the interface should collapse into:

```text
[?] Section Title                         [?]

Editor
```

The help and example controls should remain easily accessible.

---

# 26. Navigation

Provide a fixed or easily accessible top navigation area containing:

* New Brief
* My Drafts
* Import
* Save
* Export
* Settings/About

The navigation must not appear in exported documents.

Consider adding a section navigation sidebar for long briefs.

For example:

```text
CASE INFORMATION
FACTS
PROCEDURAL HISTORY
ISSUES
RULES
HOLDINGS
ORDER
REASONING
NEW INFORMATION
QUESTIONS / COMMENTS
```

Clicking a section should scroll to that section.

The sidebar should indicate the currently visible section where practical.

---

# 27. Unsaved Changes

Because automatic saving will be used, the application should generally prevent loss of work.

Display a small status indicator:

```text
Saved locally
```

or:

```text
Saving...
```

or:

```text
Saved 2 minutes ago
```

If IndexedDB storage fails, display a prominent warning.

The application should never silently claim that data has been saved when the storage operation failed.

---

# 28. New Brief Workflow

When the user selects "New Brief":

1. Create a new unique draft ID.
2. Create a new empty brief state.
3. Populate all configured sections.
4. Save the draft locally.
5. Open the editor.

If another draft is currently open, do not overwrite it.

---

# 29. Example Content Management

Instructions and examples should live outside the UI code.

For example:

```javascript
export const sections = [
    {
        id: "facts",
        title: "Facts",
        instructions: {
            summary: "...",
            detailed: "..."
        },
        examples: [...]
    }
];
```

This allows content to be edited without rewriting the application logic.

Eventually this information could be moved into JSON files.

---

# 30. Future Custom Templates

The architecture should eventually support multiple templates.

For example:

```text
Case Brief
IRAC Brief
CRAC Brief
Constitutional Law Brief
Statutory Interpretation Brief
Legal Research Brief
```

A future template could simply specify a different collection of sections.

Do not implement all of these in version 1 unless specifically requested.

Design the architecture so that they are possible.

---

# 31. Future User-Created Sections

Eventually users may be allowed to add custom sections.

The architecture should permit:

```text
+ Add Section
```

The user could specify:

* Section title
* Instructions
* Field type
* Optional examples

This does not have to be implemented in version 1, but the underlying data model should not prevent it.

---

# 32. Accessibility

The application should follow normal web accessibility practices.

Requirements include:

* Semantic HTML
* Keyboard navigation
* Visible focus indicators
* Accessible labels
* Buttons rather than clickable decorative elements
* Tooltips that can also be activated by keyboard
* Sufficient contrast
* No information conveyed solely through color
* Screen-reader-friendly headings

Hover functionality must always have a keyboard/click equivalent.

---

# 33. Error Handling

The application must gracefully handle:

* IndexedDB unavailable
* Storage quota exceeded
* Invalid imported file
* Unsupported file version
* PDF generation failure
* DOCX generation failure
* Browser APIs unavailable
* Unexpected application state

Errors should be presented to the user in understandable language.

Do not expose raw JavaScript stack traces in normal UI.

---

# 34. Browser Compatibility

Prioritize modern browsers:

* Google Chrome
* Microsoft Edge
* Firefox
* Safari

Do not make the application dependent on the File System Access API.

The normal `.brief` download/upload workflow must work even when the File System Access API is unavailable.

---

# 35. Security

Treat all imported `.brief` files as untrusted input.

Do not inject imported strings into the DOM using unsafe `innerHTML` unless the content has been sanitized.

Prefer:

```javascript
textContent
```

for user-generated content.

If rich HTML content is eventually supported, use a well-maintained sanitization library.

Do not execute JavaScript contained in imported data.

Do not dynamically evaluate imported data using:

```javascript
eval()
```

or:

```javascript
new Function()
```

---

# 36. No Server Requirement

The final application must be capable of operating without a backend.

The application should continue to function after the initial page and required static resources have loaded.

If practical, consider adding a service worker later to provide offline capability.

Offline functionality is desirable but not required for version 1.

---

# 37. Versioning

Maintain separate versions for:

```text
Application Version
Template Version
Draft Format Version
```

Example:

```javascript
const APP_VERSION = "1.0.0";
const TEMPLATE_VERSION = "1.0";
const DRAFT_FORMAT_VERSION = 1;
```

This will allow future updates without breaking existing drafts.

---

# 38. Testing Requirements

Before considering version 1 complete, test:

### Editor

* Create a brief
* Enter text into every section
* Reload the page
* Verify the draft remains

### Drafts

* Create multiple drafts
* Open different drafts
* Rename drafts
* Duplicate drafts
* Delete drafts

### Import/Export

* Export `.brief`
* Close browser
* Reopen application
* Import `.brief`
* Verify every field is restored

### DOCX

* Export DOCX
* Open in Microsoft Word
* Verify formatting
* Verify instructional content is absent
* Verify example content is absent
* Verify metadata exists

### PDF

* Export PDF
* Open in Adobe Acrobat or another PDF viewer
* Verify formatting
* Verify instructional content is absent
* Verify example content is absent
* Verify metadata exists

### Privacy

Verify that no brief contents are transmitted by the application.

### Responsive behavior

Test desktop, tablet, and narrow mobile widths.

---

# 39. Development Priorities

Implement in this order:

## Phase 1

Create the basic editor.

* Application shell
* Ten sections
* Dynamic section rendering
* Instruction popups
* Example popups
* Multiple examples

## Phase 2

Implement persistence.

* IndexedDB
* Automatic saving
* Draft management
* Saved status

## Phase 3

Implement portable drafts.

* `.brief` export
* `.brief` import
* Validation
* Versioning

## Phase 4

Implement document generation.

* DOCX
* PDF
* Formatting
* Metadata

## Phase 5

Polish.

* Responsive UI
* Accessibility
* Error handling
* Better navigation
* Visual refinements

Do not attempt to implement every future feature immediately.

---

# 40. Important Architectural Principle

The most important architectural principle is:

**The brief itself is data. The editor is merely a view of that data.**

Do not make the HTML form the authoritative representation of the brief.

The underlying data model should be capable of being:

1. Displayed in the editor
2. Saved to IndexedDB
3. Serialized into a `.brief` file
4. Imported from a `.brief` file
5. Converted into DOCX
6. Converted into PDF
7. Used by future templates

This will make the application substantially easier to expand.

---

# 41. Initial Default Sections

The initial configuration should contain the following:

### 1. Case Name, Court, Citation, Date

Purpose: Identify the case and provide its basic citation information.

### 2. Facts

Purpose: Record the legally significant facts necessary to understand the dispute and decision.

### 3. Procedural History

Purpose: Explain how the case reached the present court, including relevant lower-court proceedings.

### 4. Issue(s)

Purpose: State the legal question or questions the court was required to resolve.

### 5. Applicable Rule(s) of Law

Purpose: Identify the governing legal rule or standard applied by the court.

### 6. Holding(s)

Purpose: State the court's answer to the legal issue.

### 7. The Court's Order

Purpose: State what the court actually did with the case, such as affirming, reversing, remanding, dismissing, granting, or denying relief.

### 8. Reasoning

Purpose: Explain why the court reached its holding.

### 9. New Information

Purpose: Record new legal concepts, terminology, doctrines, procedural concepts, or other information learned from the case.

### 10. Questions, Comments, and Speculations

Purpose: Give the student a place to record questions, observations, possible implications, disagreements, and areas requiring further research.

The exact instructional language should be stored in the section configuration rather than hard-coded into the editor.

---

# 42. Initial Visual Design

The application should have a professional academic/legal appearance.

Avoid making it look like a generic consumer form.

Suggested visual characteristics:

* Clean typography
* White or lightly tinted document area
* Restrained accent color
* Strong section headings
* Subtle borders
* Generous writing space
* Clear visual separation between editor and instructional material
* Minimal animations

The application should feel closer to a legal research/work product tool than a survey form.

---

# 43. Final Requirement

Do not create unnecessary backend infrastructure.

The application should remain a static, privacy-preserving, client-side legal study tool.

When implementing functionality, prefer the simplest browser-native or well-established JavaScript solution that satisfies the requirement.

Avoid unnecessary frameworks or dependencies unless they provide a clear benefit.

The final application should be capable of being placed into a GitHub repository and published through GitHub Pages with minimal configuration.


# 44. Document Provenance and Origin Metadata

The application must include a document provenance system.

The purpose of this system is to preserve information that may help establish the history and origin of a brief created using Legal Brief Builder.

This system is intended to provide useful provenance information and potentially assist with academic-integrity questions, such as determining whether a particular `.brief`, PDF, or DOCX file originated from the application.

The provenance system is NOT intended to be a tamper-proof anti-plagiarism system.

Users must be able to understand that metadata can potentially be removed, modified, or lost during document conversion.

The application must prioritize user privacy.

---

# 45. Unique Document ID

Every brief created by the application must receive a unique document identifier.

The identifier should be generated locally using a cryptographically strong random identifier where supported.

Use a UUID or equivalent sufficiently random identifier.

Example:

```text
LBB-8f2a91c4-7d5e-4b91-a2d3-91f4e7c6b821
```

The document ID must remain associated with the brief throughout its lifetime.

If the user:

* Saves the brief
* Closes the browser
* Reopens the brief
* Exports the brief
* Imports the `.brief` file
* Exports the imported brief again

the document ID should normally remain unchanged.

Creating a duplicate of a brief should create a new document ID because the duplicate represents a separate document.

---

# 46. Provenance Data Model

The underlying brief data should include a provenance object.

Example:

```javascript
provenance: {
    documentId: "LBB-8f2a91c4-7d5e-4b91-a2d3-91f4e7c6b821",

    createdAt: "2026-08-27T18:30:00.000Z",

    lastModifiedAt: "2026-08-27T19:15:00.000Z",

    applicationVersion: "1.0.0",

    templateVersion: "1.0",

    formatVersion: 1,

    generatedBy: "Legal Brief Builder"
}
```

The timestamps must use ISO 8601 format.

Store timestamps in UTC.

Display timestamps to the user in their local time zone when appropriate.

---

# 47. Anonymous Local Device Identifier

The application may optionally generate an anonymous local installation identifier.

If implemented, generate a random identifier the first time the application runs.

Store the identifier in IndexedDB.

Example:

```javascript
deviceId: "LBB-device-39f0d7b1-..."
```

This identifier must NOT be derived from:

* MAC address
* IP address
* Computer name
* Windows username
* Operating system serial numbers
* CPU identifiers
* Motherboard identifiers
* Browser fingerprinting
* Hardware fingerprints
* Location
* Installed software

The identifier must be random and anonymous.

Its purpose is only to allow the application to recognize the same browser installation locally.

It must NOT be treated as proof of the identity of the person using the computer.

If the user clears browser storage, uses another browser, or uses another device, a new identifier may be generated.

The application must not attempt to circumvent browser privacy protections to obtain hardware identifiers.

The anonymous device identifier should be considered optional and should not be necessary for normal operation.

---

# 48. Revision History

The application should maintain a lightweight local revision history for each brief.

The purpose is to preserve evidence that a brief was developed over time rather than appearing fully formed at the moment of export.

The application does NOT need to store every keystroke.

Instead, record periodic revisions.

Each revision may contain:

```javascript
{
    revisionNumber: 17,
    timestamp: "2026-08-27T19:02:31.000Z",
    contentHash: "sha256-hash",
    wordCount: 1847,
    characterCount: 11203
}
```

The revision history should be stored locally.

A new revision should be generated periodically rather than for every keystroke.

For example, create a revision when:

* The user has stopped editing for a significant period
* A substantial change has occurred
* The user manually saves
* The document is exported
* The application is closed or backgrounded where practical

Use debouncing and reasonable limits so revision history does not grow indefinitely.

---

# 49. Content Hashing

The application should calculate a SHA-256 hash of the substantive brief data when creating a revision.

The hash should be calculated using the Web Crypto API where supported.

The hash should represent the actual brief content rather than UI state.

Do NOT include transient UI information such as:

* Which help popup is open
* Which example is currently displayed
* Scroll position
* Selected tab
* Window size

The hash should instead represent the substantive document.

For example:

```text
Case information
Facts
Procedural history
Issues
Rules
Holdings
Order
Reasoning
New information
Questions/comments
```

A change to substantive document content should normally result in a different hash.

---

# 50. Provenance and Privacy

The application must NOT collect or transmit the actual brief to a server for provenance purposes.

The initial implementation must remain entirely client-side.

The application should not transmit:

* Brief text
* User names
* IP addresses
* Device identifiers
* Browser fingerprints
* Location
* Hardware information

to a remote server.

The provenance system must operate locally.

---

# 51. Portable `.brief` Provenance

The `.brief` file must preserve provenance information.

Example:

```json
{
    "format": "legal-brief-builder",
    "formatVersion": 1,

    "brief": {
        "title": "International Shoe v. Washington",

        "caseInformation": {
            "caseName": "International Shoe Co. v. Washington",
            "court": "Supreme Court of the United States",
            "citation": "326 U.S. 310",
            "date": "1945"
        },

        "sections": {
            "facts": "...",
            "procedural-history": "...",
            "issues": "..."
        }
    },

    "provenance": {
        "documentId": "LBB-8f2a91c4-7d5e-4b91-a2d3-91f4e7c6b821",
        "createdAt": "2026-08-27T18:30:00.000Z",
        "lastModifiedAt": "2026-08-27T19:15:00.000Z",
        "applicationVersion": "1.0.0",
        "templateVersion": "1.0",
        "generatedBy": "Legal Brief Builder",

        "revisions": [
            {
                "revisionNumber": 1,
                "timestamp": "2026-08-27T18:31:00.000Z",
                "contentHash": "..."
            }
        ]
    }
}
```

The exact JSON structure may be adjusted during implementation.

The important requirement is that provenance information remain clearly separated from the substantive brief data.

---

# 52. Exported DOCX Metadata

When generating a DOCX file, include provenance information in document metadata or custom document properties where the DOCX format and selected JavaScript library support them.

At minimum, attempt to include:

```text
Creator: Legal Brief Builder
Producer: Legal Brief Builder
Keywords: legal-brief-builder
```

Where custom properties are supported, include:

```text
LBB_Generated = true
LBB_Document_ID = [document ID]
LBB_Application_Version = [application version]
LBB_Template_Version = [template version]
LBB_Created_At = [timestamp]
```

Do NOT place this information visibly in the body of the document.

Do NOT add hidden paragraphs, white text, hidden characters, or other body-content tracking mechanisms.

The exported DOCX must otherwise contain only the actual brief.

---

# 53. Exported PDF Metadata

When generating a PDF file, include provenance information in standard PDF metadata fields where possible.

At minimum:

```text
Creator: Legal Brief Builder
Producer: Legal Brief Builder
Keywords: legal-brief-builder
```

Where the PDF library permits additional metadata, include:

```text
LBB_Generated = true
LBB_Document_ID = [document ID]
LBB_Application_Version = [application version]
LBB_Template_Version = [template version]
LBB_Created_At = [timestamp]
```

Do NOT place the provenance information visibly on the PDF page.

Do NOT use a visible watermark.

Do NOT add hidden text to the PDF body for tracking.

---

# 54. Provenance Metadata Must Not Be Described as Tamper-Proof

The application's documentation must make clear that document metadata is not inherently trustworthy.

A technically knowledgeable user may be able to:

* Edit metadata
* Remove metadata
* Re-save the document
* Convert the document into another format
* Copy the substantive text into a new document
* Modify the `.brief` JSON file
* Clear browser storage

Therefore, provenance metadata should be described as supporting evidence rather than definitive proof of authorship.

Do not advertise the system as a foolproof plagiarism detector.

---

# 55. Optional Provenance Information in the User Interface

The application may provide a "Document Information" or "Provenance" panel.

The panel may display:

```text
Document ID:
LBB-8f2a91c4-7d5e-4b91-a2d3-91f4e7c6b821

Created:
August 27, 2026 at 2:30 PM

Last modified:
August 27, 2026 at 3:15 PM

Revisions:
17

Application version:
1.0.0

Template version:
1.0
```

The anonymous device identifier should NOT be prominently displayed to ordinary users unless there is a clear reason to expose it.

---

# 56. Provenance Export Behavior

When exporting a brief, the exporter must use the existing document ID.

Do not create a new document ID merely because the user exports to PDF or DOCX.

For example:

```text
Original .brief:
LBB-12345

Exported DOCX:
LBB-12345

Exported PDF:
LBB-12345
```

This allows the different representations to be associated with the same underlying document.

If the user creates a duplicate:

```text
Original:
LBB-12345

Duplicate:
LBB-67890
```

The duplicate must receive a new document ID.

---

# 57. Provenance and Imported Files

When importing a valid `.brief` file:

1. Preserve its document ID.
2. Preserve its creation timestamp.
3. Preserve its existing revision history.
4. Add a new revision indicating that the document was imported if appropriate.
5. Update the current application's last-modified timestamp when the user actually modifies the content.

Do not silently replace the document ID merely because the file was imported into a different browser.

---

# 58. Provenance and Document Duplication

Provide a distinction between:

### Open

Opens the existing document with its existing document ID.

### Duplicate

Creates a new document with a new document ID.

This distinction is important for provenance.

---

# 59. Future Trusted Timestamping

The architecture should leave room for a future optional feature in which the application could submit only a cryptographic hash of a document to an external timestamping service.

The service would not need to receive the actual brief.

Conceptually:

```text
Brief
  ↓
SHA-256
  ↓
Hash only
  ↓
Timestamp Service
  ↓
Signed Timestamp Receipt
```

This is NOT required for version 1.

Do not implement an external timestamping service unless specifically requested.

The current version must remain fully functional without it.

---

# 60. Future Academic Integrity Use

The provenance architecture should be designed so that a future version could potentially allow a user to export a provenance report.

For example:

```text
LEGAL BRIEF BUILDER
DOCUMENT PROVENANCE REPORT

Document ID:
LBB-8f2a91c4...

Created:
August 27, 2026

Last Modified:
August 27, 2026

Application Version:
1.0.0

Template Version:
1.0

Revision Count:
17

Initial Content Hash:
...

Final Content Hash:
...
```

This feature is not required for version 1.

The current architecture should nevertheless preserve sufficient information to make such a feature possible later.

---

# 61. Important Security Principle

Do not attempt to establish authorship through computer identification.

The purpose of provenance is to establish the history of a document, not to claim that a particular physical computer or person created it.

The system should therefore prioritize:

1. Document identity
2. Creation time
3. Modification history
4. Revision hashes
5. Application/template versions
6. Export provenance

over hardware identification.

---

# 62. Recommended Provenance Architecture

The final provenance system should conceptually resemble:

```text
                         LEGAL BRIEF
                              │
                              ▼
                       Document ID
                              │
               ┌──────────────┼──────────────┐
               │              │              │
               ▼              ▼              ▼
          Creation Time   Revision Log   Version Data
                              │
                              ▼
                         SHA-256 Hash
                              │
               ┌──────────────┼──────────────┐
               │                             │
               ▼                             ▼
          .brief Export                  PDF/DOCX
               │                             │
               ▼                             ▼
        Provenance Data              File Metadata
```

The system should remain entirely client-side for version 1.

---

# 63. Implementation Priority

Provenance features should be implemented in this order:

### Required for Version 1

* Unique document ID
* Creation timestamp
* Last-modified timestamp
* Application version
* Template version
* `.brief` provenance preservation
* PDF provenance metadata
* DOCX provenance metadata

### Recommended for Version 1

* SHA-256 content hashes
* Lightweight revision history

### Optional

* Anonymous local device identifier

### Future

* Provenance reports
* Trusted external timestamping
* Signed provenance receipts

Do not allow provenance functionality to complicate or compromise the core brief-writing experience.

---

# 64. Updated Core Data Model

The overall brief object should approximately resemble:

```javascript
{
    id: "LBB-8f2a91c4-7d5e-4b91-a2d3-91f4e7c6b821",

    title: "International Shoe v. Washington",

    createdAt: "2026-08-27T18:30:00.000Z",

    modifiedAt: "2026-08-27T19:15:00.000Z",

    caseInformation: {
        caseName: "",
        court: "",
        citation: "",
        date: ""
    },

    sections: {
        facts: "",
        proceduralHistory: "",
        issues: "",
        applicableRules: "",
        holdings: "",
        courtOrder: "",
        reasoning: "",
        newInformation: "",
        questionsCommentsSpeculations: ""
    },

    provenance: {
        documentId: "LBB-8f2a91c4-7d5e-4b91-a2d3-91f4e7c6b821",
        generatedBy: "Legal Brief Builder",
        applicationVersion: "1.0.0",
        templateVersion: "1.0",
        formatVersion: 1,

        revisions: []
    }
}
```

The exact implementation may differ, but the underlying architecture must preserve these concepts.

---

# 65. Final Provenance Requirement

The application should make it possible to answer the following questions from a `.brief` file or exported document whenever the relevant metadata has not been removed:

1. Was this document generated by Legal Brief Builder?
2. What application version generated it?
3. What template version was used?
4. What is the document's unique identifier?
5. When was the document originally created?
6. When was it last modified?
7. What revisions were recorded locally?
8. What content hashes were associated with those revisions?

The application must NOT claim that this information proves the identity of the author.

The purpose is document provenance, not surveillance.

---

# 66. Case Information Examples

The Case Name, Court, Citation, and Date fields must display persistent examples of the expected format. Examples should remain visible after the user begins typing and should include:

* Case name, such as `International Shoe Co. v. Washington`
* Court, such as `Supreme Court of the United States`
* Citation, such as `326 U.S. 310 (1945)`
* Date, such as `1945` or `January 12, 2024`

---

# 67. Case Information Example Collection

The Case Information section must provide one shared “See an example” control for the complete set of metadata fields. The control should open a rotating example panel containing two or more complete examples, with each example showing the case name, court, citation, and date together. It should use the same predictable Previous and Next behavior as the other example collections.
