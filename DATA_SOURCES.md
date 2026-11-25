# Data sources and provenance

## GEO series and example samples

- Series: [GSE264438 — MSA: Scalable DNA methylation BeadChip for human trait screening](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE264438).
- Platform: [GPL34394 — Infinium Methylation Screening Array](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL34394).
- The checked-in QC example uses [GSM8217992](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM8217992), [GSM8217993](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM8217993), [GSM8217994](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM8217994), and [GSM8217995](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM8217995). GEO describes these as Illumina Control DNA samples. They are included to demonstrate array import and QC, not as a biological comparison.
- Each selected GSM record supplies its own red and green IDAT URLs. The notebook fetches only the selected pair for each GSM; it does not download the multi-gigabyte series-wide raw archive or processed matrices.
- GEO describes its public processed values as normalized average beta values and detection p-values, reports SeSAMe v1.22 processing, and notes that SNP-probe readings in the deposited IDATs were de-identified.

## Manifest and annotation

GSE264438 was released in 2024. The example uses the corresponding MSA A1 design. The official [Illumina MSA manifest download page](https://support.illumina.com/downloads/infinium-methylation-screening-manifest-files.html) lists manifest revisions; check the revision before combining this workflow with data generated under another design.

The notebook installs the minfi-compatible Bioconductor [MSA manifest package](https://bioconductor.org/packages/release/data/annotation/html/IlluminaHumanMethylationMSAmanifest.html) and the matching [MSA hg38 annotation package](https://bioconductor.org/packages/release/data/annotation/html/IlluminaHumanMethylationMSAanno.ilm10a1.hg38.html). The annotation package documentation identifies Illumina's `MSA-48v1-0_20102838_A1.csv` as its source. If using a standalone manifest CSV, download it from Illumina's support page and record its filename/revision.

## Processing provenance

The notebook records the selected GSM accessions, package/session information, preprocessing method, detection-p cutoff, and failed-probe threshold in `results/`. It uses minfi `preprocessNoob()` followed by detection-p filtering. The optional DMP analysis uses limma on M-values and exports a case-minus-reference beta difference. GEO's SeSAMe-processed values are a separate processing branch; do not treat the two outputs as numerically interchangeable.
