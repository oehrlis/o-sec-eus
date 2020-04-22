# English Documentation

This folder contains the documentation files and folder.

- [de](de) German documentation files.
- [en](en) English documentation files.
- [images](images) Images and logo files.
- [templates](templates) pandoc templates.
- [metadata.yml](metadata.yml) Meta file used during pandoc conversion. This file include different information like, title, subtile, author, fonts etc

## Build Documentation

The workshop documentation is based on markdown. This allows to convert it to different formats e.g. PDF, DOCX and PPTX

- Create PDF using a local pandoc installation. This requires also latex.

```bash
pandoc --template trivadis --listings --pdf-engine=xelatex \
    --metadata-file=doc/metadata.yml \
    --resource-path ./doc/images \
    -o doc/O-PLSQL_Workshop.pdf \
    --include-in-header doc/templates/titlesec.tex \
    doc/en/0x??-* lab/ex??/1x??en-* doc/en/9x??-*
```

- Create DOCX using a local pandoc installation.

```bash
pandoc --reference-doc doc/templates/trivadis.docx --listings \
    --metadata-file=doc/metadata.yml \
    --resource-path ./doc/images \
    -o doc/O-PLSQL_Workshop.docx \
    doc/en/0x??-* lab/ex??/1x??en-* doc/en/9x??-*
```

- Create Markdown file using a local pandoc installation. 

```bash
pandoc --listings  \
    --metadata-file=doc/metadata.yml \
    --resource-path ./doc/images \
    -o doc/O-PLSQL_Workshop.md \
    doc/en/0x??-* lab/ex??/1x??en-* doc/en/9x??-*
```

- Create PPTX for the workshop presentation using a local pandoc installation.

```bash
pandoc --reference-doc doc/templates/trivadis.pptx --listings \
    --metadata-file=doc/metadata.yml \
    --resource-path ./doc/images \
    -o doc/O-PLSQL_Workshop_Exercise.pptx \
    lab/ex??/1x??en-E*.md
```

- Create HTML file using a local pandoc installation.

```bash
pandoc -f markdown  --listings \
    --metadata-file=doc/metadata.yml \
    --resource-path ./doc/images --standalone \
    -o doc/O-PLSQL_Workshop.html --css doc/templates/pandoc.css \
    doc/en/0x??-* lab/ex??/1x??en-* doc/en/9x??-*
```
