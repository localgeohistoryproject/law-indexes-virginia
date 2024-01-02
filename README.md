# Law Indexes: Virginia

[![DOI](https://zenodo.org/badge/737969226.svg)](https://zenodo.org/doi/10.5281/zenodo.10450174)

## Summary

As part of efforts to expand the Local Geohistory Project, which aims to educate users and disseminate information concerning the geographic history and structure of political subdivisions and local government, this respository has been created to disseminate law index data for the Commonwealth of Virginia.

This index currently covers enrolled bills from 1776 through 1910. Prior to the 20th century, private and special laws were often the primary method used to alter municipal and county boundaries and forms of government in Virginia. The law index is released as a tab-separated values (TSV) file, **output/VaLawIndex.tsv**.

Note that the Citation Page references are not to the published session laws, but to the Enrolled bills of the General Assembly collection at the [Library of Virginia](https://lva.primo.exlibrisgroup.com/permalink/01LVA_INST/altrmk/alma990004934790205756).

This repository does not contain the full text of the laws, nor does it currently contain links to the full text. Because the index was created using OCR technology, it may contain uncaptured errors.

## Harmful Content

The terms used in the law indexes were drawn from Source Works that were written before modern concepts of diversity, equity, and inclusion were developed, and have not been updated to reflect modern standards. Some terms may:[^1]

- reflect racist, sexist, ableist, misogynistic/misogynoir, and xenophobic opinions and attitudes;
- be discriminatory towards or exclude diverse views on sexuality, gender, religion, and more;
- demonstrate bias and exclusion in the subjects documented.

## Using tab-separated values (TSV) files

TSV files do not use quotation marks to escape fields containing tabs; therefore, the **String delimiter** option in LibreOffice Calc or the **Text qualifier** option in Microsoft Excel must be left blank to ensure the file imports accurately.

The file uses Unix-style [line endings](https://en.wikipedia.org/wiki/Newline#Representations) (LF), which may have to be adjusted in applications that expect Windows-style line endings (CR LF).

A header row containing column names is included. When importing into a relational database system, like PostgreSQL, it may be necessary to remove this header line, particularly when using the [PostgreSQL COPY function](https://www.postgresql.org/docs/16/sql-copy.html) in the Text Format.

## Source Works

The following Source Works were used to create this law index:

| Citation | Coverage | Online Version | Notes |
| -------- | -------- | -------------- | ----- |
| Virginia Clerk of the House of Delegates and Keeper of the Rolls. *Index to Enrolled Bills of the General Assembly of Virginia, 1776 to 1910.* Richmond, Va.: Virginia Superintendent of Public Printing: 1911. | 1776-1910 | [Internet Archive](https://archive.org/details/cu31924006511921/) | Copyright expired in the United States. Version used for this index taken from cited Online Version. Some illegible figures taken from older editions. |

Prior to inclusion, source works were examined to determine whether they fell under the public domain in the United States, using the following guides:

Hirtle, Peter B. "Copyright Services: Copyright Term and the Public Domain." *Cornell University Library.* 2004-2023. <https://guides.library.cornell.edu/copyright/publicdomain>.

U.S. Copyright Office. *Compendium of U.S. Copyright Office Practices,* 3rd ed. 2021. § 313.6(C). <https://www.copyright.gov/comp3/docs/compendium.pdf#page=83>.

## Methodology

The following is a general methodology for how the law indexes were derived from the Source Works.

- Once images are acquired, they will generally go through one of the following:
  - For images personally digitized from hardcopy, the quality will be improved using ScanTailor, and then combined as PDFs.
  - For JPEG 2000 files acquired, they are converted to PDFs.
  - For PDF files acquired from Online Versions, the original image files are extracted and then re-combined as PDFs to remove any OCR.
- The PDFs are then OCR'd using [Amazon Textract](https://aws.amazon.com/pm/textract/). The files are uploaded using the [Bulk Document Uploader](https://docs.aws.amazon.com/textract/latest/dg/bulk-uploader-best-practices.html) and processed using the Detect Document Text API. This process returns a ZIP container for each file containing a text file, a CSV file, and a JSON file.
  - **Warning:** If the uploaded PDF is too large, it may not return the JSON file in the ZIP container, requiring the document to be broken down and re-uploaded in smaller pieces in order to follow the next steps.
- The JSON files are run through a PHP script, which finds any LINE BlockType entries, and returns the Page, Text, and the X and Y coordinates for the Geometry Polygon for each line in a delimited file. To simplify the coordinates, they are multiplied by 1000 and then rounded as integers. (In future steps, these LINE BlockType rows will be referred to as data points.)
  - **Note:** In tabular data sources, the LINE BlockType often corresponds with a cell, rather than an entire row. This helps to quickly categorize data points by location.
- The delimited file data points are imported into a PostgreSQL table.
- A preliminary attempt is made to group each data point into contiguous lines on the page, and to determine the type of data contained in each data point. For source works with multiple columns, the column number is also guessed.
- A PHP script converts the data points for each page into color-coded SVGs that represent the data points positionally in a similar manner to how they appeared on the original document.
- The SVGs are reviewed to ensure each data point is accurately categorized, that the data points appear in the correct order on the page, and for blatant OCR errors. In some cases, data points are split where Amazon Textract did not correctly separate them.
  - **Note:** The accuracy of the OCR was, in general, not manually reviewed.
- For certain data types, data integrity checks are done to ensure the data makes sense (e.g., whether years are digits and fall in the correct time period).
- The data points are combined and reorganized using database views to create the final report. As part of this reorganization, it is determined where repeating data points like subjects start and end.

[^1]: Some content in this section was derived from the following work: National Archives and Records Administration. *NARA's Statement on Potentially Harmful Content.* 17 June 2022. <https://www.archives.gov/research/reparative-description/harmful-content>.
