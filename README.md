# Article importer

Imports JATS XML into OJS from the command line. Run the commands below from the OJS main folder.

The journal path, both usernames, and the email must already exist in OJS. The email is used when an author has none in the XML.

## Installation

Copy the `articleImporter` folder into the OJS installation, exactly to this path:

```text
plugins/importexport/articleImporter
```

The folder name must stay `articleImporter`. The command-line tool loads it from that path, so it does not need to be enabled in the web interface.

## Folder layout

One folder per article, and inside it one folder per version. Each version folder must contain exactly one `.xml` file. Other files (figures, PDFs, and so on) sit next to that XML.

If you've got HTML files from F1000, the HTML file should be placed at the same folder of the XML file, and with the same basename, but the extension should be `.html`. If no HTML file is present, the plugin will attempt to generate one by itself, out of the <body> section of the JATS file.

```text
import/
  article-1/
    1/
      article.xml
      article.html
      figure.png
  article-2/
    1/
      article.xml
```

## Command

This is the command you're likely to use (see description of the arguments and flags below):

```text
php tools/importExport.php ArticleImporterPlugin journalPath username editorUsername email importPath --use-category-as-section --no-volume --no-number --no-issue --disable-f1000-integration
```

| Argument | Meaning |
| --- | --- |
| `journalPath` | Journal path in OJS |
| `username` | User the imported articles are assigned to |
| `editorUsername` | Editor assigned to the imported articles |
| `email` | Fallback author email |
| `importPath` | Path to the `import/` folder mentioned previously |

Use these flags:

- `--use-category-as-section` — the article category will be used as the OJS section
- `--no-volume` and `--no-number` — given ORE follows a continuous publishing workflow (doesn't use the OJS' issues), these two flags are required
- `--no-issue` — do not create nor assign issues
- `--disable-f1000-integration` — do not connect to the F1000 database (see below)

## Articles without a DOI

If the JATS file has no DOI, every file in the version folder (images, documents, etc) will be imported with the article, not only the images referenced by the XML.

## F1000 database integration

You may drop the flag `--disable-f1000-integration` if you have access to the F1000 Postgres database. The importer then also loads reviews, reviewer suggestions, comments, author emails, ORCID tokens, and the article status from that database.

The integration is based on the article ID and the version number. They are taken from the DOI when present (for example, in 10.12688/openreseurope.**13015**.**1**, the bolded parts are the article ID and the version).

If the article doesn't have a DOI (for example, it is not published yet), the article folder name is used as the article ID and the version folder name as the version. The folder names must therefore match the IDs in F1000. For article ID 12345, version 1:

```text
import/
  12345/
    1/
      article.xml
```

Set the needed environment variables before the command. Each one has a default if omitted:

| Variable | Default |
| --- | --- |
| `F1000_DB_HOST` | `127.0.0.1` |
| `F1000_DB_NAME` | `ore` |
| `F1000_DB_USER` | `root` |
| `F1000_DB_PASS` | `abc123` |

Example:

```text
F1000_DB_USER=ore F1000_DB_PASS=secret php tools/importExport.php ArticleImporterPlugin ore admin editor  admin@ore.testing.publicknowledgeproject.org ~/f1000/jats/ --use-category-as-section --no-volume --no-number --no-issue
```
