# Legal Brief Builder

Legal Brief Builder is a static, local-first case brief editor for law students. It runs in a modern browser and can be published directly through GitHub Pages.

## Use

Open `index.html` in a modern browser, or serve this directory with any static web server. Drafts are saved in IndexedDB on the current browser. Use **Export draft** to create a portable `.brief` file.

PDF and DOCX export use browser-ready libraries loaded from CDN. An internet connection is needed on first load for those libraries unless they are later vendored into the repository.

## Privacy and provenance

Brief text is not sent to a server. Exported files contain document provenance metadata when supported by the file format. Metadata is supporting evidence only: it can be removed, modified, or lost and does not prove authorship. The application does not use device fingerprints or collect personal information.

## GitHub Pages

Commit the static files and enable GitHub Pages for the repository's branch. No backend or build command is required.
