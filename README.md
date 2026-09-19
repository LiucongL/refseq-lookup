# RefSeq lookup

A lightweight web app to find RefSeq protein and nucleotide records linked from UniProt, copy coding sequences, and link to gene pages at MGI, HGNC, FlyBase and other organism databases.

It answers one question quickly — **which RefSeq records belong to this UniProt entry?** — by listing the records UniProt cites, linking each one to NCBI, copying accessions and coding sequences (each coding sequence checked against the protein), and linking the gene to its page at the organism's own database.

Enter a UniProt accession (`P68871`, or an isoform such as `P68871-2`), an entry name (`P53_HUMAN`), a UniProt entry URL, or a protein or gene name plus organism (`Dlx5 mouse`), and press **Look up**. A RefSeq accession (`NP_…`, `XP_…`, `YP_…`, `WP_…`, `AP_…`, `NM_…`, `XM_…`) works in the other direction: it finds the UniProt entry that cites it and marks the record you asked for.

A link can open the app on a protein directly, so another page (Protein Explorer, for example) or a bookmark can hand a protein over: `?q=P68871`, `?q=P68871-2`, `?q=NP_000509.1` or `?q=Dlx5+mouse`. The address updates as you look things up, so the address bar always links to the result on screen.

Current version: **v2.2.1**, shown beside the app name in the header and in the footer.

- **Open the app:** **https://liucongl.github.io/refseq-lookup/** — no account needed.
- **Source code and releases:** **https://github.com/LiucongL/refseq-lookup**


## What you get

- **The entry:** protein name, UniProt review status, accession, entry name, gene, NCBI Gene ID, organism and length.
- **The gene's own database page**, where UniProt cites one: MGI (mouse), HGNC (human), RGD (rat), FlyBase, ZFIN (zebrafish), WormBase and SGD (yeast), with a second link to the same gene at the Alliance of Genome Resources. These pages hold what this app deliberately does not: genomic location, function, phenotypes, expression and orthologs.
- **One row per RefSeq record:** the protein accession, the nucleotide record UniProt pairs with it (labelled mRNA, ncRNA or genomic), the isoform it is assigned to, and whether the record is *known* (`NP_`), *model* (`XP_`) or *non-redundant* (`WP_`).
- **Copy buttons** for each accession, and **Copy CDS**, which fetches the coding nucleotide sequence of the transcript from NCBI, checks it against the protein, and copies it.
- **The UniProt canonical sequence**, as FASTA or as plain residues.
- When an entry has no RefSeq cross-reference, a link that starts a BLAST search of the sequence against NCBI RefSeq Protein.

## How it works

The app is plain HTML and JavaScript, hosted as a static page. It has no application backend of its own. Data are requested from public services when you look something up.

| Information | Source |
|---|---|
| Entry, sequence, isoforms and all cross-references (RefSeq, NCBI Gene, MGI, HGNC, …) | [UniProt](https://www.uniprot.org/) REST API |
| Coding sequence and NCBI's translation of it, on request | [NCBI E-utilities](https://www.ncbi.nlm.nih.gov/books/NBK25501/) (`efetch`, `fasta_cds_na` and `fasta_cds_aa`) |
| Gene pages | Links only — nothing is fetched from MGI, HGNC, the Alliance or the other databases |
| Typeface | IBM Plex, loaded from Google Fonts; the app falls back to system fonts without it |

The app does not require an account or maintain its own server-side search history. Lookups are sent to UniProt, and Copy CDS requests to NCBI. Those providers, Google Fonts and the website host may log requests; the browser may also retain history and cached data. Searches should not be considered private or anonymous.

## Reading the results

**The app reports the cross-references UniProt publishes. It does not align sequences or decide for itself which RefSeq record corresponds to a protein.**

- **No RefSeq row does not mean no RefSeq record.** UniProt links only some RefSeq records. For an entry without one, use the BLAST link and confirm organism and sequence identity before treating a hit as the corresponding record.
- **The accession prefix is a record category, not a review status.** `NP_`/`NM_` records are "known" and `XP_`/`XM_` records are computational models, but whether a record is reviewed, validated or provisional is stated inside each NCBI record and is not inferred here. Known records are listed first.
- **Isoforms.** UniProt assigns some RefSeq records to a specific isoform and leaves others unassigned. When you ask for an isoform (`P68871-2`), the app marks the records assigned to it, and says plainly when the remaining records are assigned to *other* isoforms or have *no assignment* — an unassigned record may or may not be the isoform you want. An isoform number asked for on a secondary (merged) accession is not carried over to the new entry, because nothing guarantees the numbering survived.
- **Versions.** RefSeq accessions carry a version (`NP_000509.1`). If you look up a version UniProt does not cite, the app finds the entry through the accession without its version and warns that the cited version differs. A different version can be a different sequence.
- **Secondary and obsolete UniProt accessions.** A secondary accession opens the entry that now holds it, with a note. A deleted or merged accession is reported as inactive, with the entries it was merged into when UniProt names them.
- **Gene-page links come from UniProt's cross-references, never from a guess by gene symbol,** and an identifier that does not match the database's format is dropped rather than turned into a link. The Alliance link appears only when UniProt also cites the Alliance record for that gene. For worm proteins the WormBase gene ID is used, not the transcript name UniProt lists first.
- **Genome assembly.** The app shows no coordinates itself. When you take a location from MGI, HGNC or the Alliance into a genome browser such as IGV, check the assembly: these sites report current assemblies (GRCm39/mm39 for mouse, GRCh38 for human), while many existing tracks and BAM files are still on older ones such as mm10.

### Copy CDS

Copy CDS is offered for transcript records (`NM_`, `XM_`) only. It copies the coding sequence **exactly as NCBI returns it, with any terminal stop codon retained**, and only after finding the CDS whose `protein_id` is the protein in that row — never simply the first CDS in the record. If NCBI returns something that is not valid FASTA, nothing is copied.

Under the row, the app reports a check of what it copied: NCBI's own translation of that CDS is compared with the UniProt canonical sequence, and any difference is listed (length, or the substituted positions). Read this note before using the sequence.

- The comparison is made only for records of the canonical isoform or with no isoform assignment, because the canonical sequence is the only one the page holds. For other isoforms the note gives the translated length and says no comparison was made.
- A mismatch is not necessarily an error: it can be another isoform, another record version, or a natural variant.
- If NCBI's translation is unavailable, a standard-genetic-code translation stands in and the note says so; records that use another translation table or a translational exception can then show spurious differences.

## Other things to know

- **Search results:** a name search shows the first six matches, with a link to the full result list at UniProt. Adding the organism, or pasting the accession, is the quickest way to the right entry.
- **NCBI request rate:** NCBI allows three requests per second per IP address without an API key. The app starts its own requests about 400 ms apart and reuses what it has already fetched, so several Copy CDS clicks in a row take a moment instead of going out as one burst. This limits only what this page sends: other tabs, other tools, or colleagues behind the same institutional IP address count towards the same limit, so NCBI can still refuse a request. The app then says it was rate-limited (HTTP 429) and the click can be repeated.
- **Clipboard:** a Copy CDS whose sequence arrives after you have clicked another copy button does not copy; its row still shows the sequence check and says it was not copied. A refusal that arrives late for such an overtaken copy does not open the copy-by-hand box either. This covers copies that are still waiting for NCBI; a clipboard write the browser has already started cannot be recalled, so if you clicked several copy buttons in quick succession, paste and check before relying on the clipboard. If the browser refuses clipboard access for the current copy, a box opens with the text selected so it can be copied by hand; a button reads "Copied" only after the browser has accepted the write.
- **Availability:** public services can be slow or unavailable. Requests time out after 15 seconds with a message saying which service did not answer. A failed request is not evidence that a record does not exist.

## Running and hosting

Use the app from a static web host over HTTPS for reliable browser features such as clipboard access. A downloaded HTML file may also work in a desktop browser, but it still needs internet access for database requests. Local-file handling varies between browsers and mobile file viewers; the hosted link is the simplest option.

To host it with GitHub Pages, put the app in the repository as `index.html` and enable Pages for the branch. To update a hosted copy, replace `index.html` and redeploy it.

Two lines at the top of the script in `index.html` are the release settings. `APP_VERSION` sets the version shown in the header and footer; keep it consistent with this README and any release tag. `REPO_URL` is the address of the GitHub repository; once set, the footer links to this README, the source code and the issue tracker. While it is empty those links are not shown.

## Reporting problems

**Issue tracker:** **https://github.com/LiucongL/refseq-lookup/issues**

Please include the app version, what you typed into the search box, the UniProt or RefSeq accession concerned, your browser, steps to reproduce the problem, and what you expected to happen. Include the displayed message or a screenshot when useful.

## Citing

If you use this tool in your research, please cite the version used. Author, version and release details are kept in `CITATION.cff` in the repository; GitHub shows them under **Cite this repository**.

Also cite UniProt and NCBI RefSeq according to their guidance, and record the accessions **with their versions** and the date of access, so the sequences you used can be traced.

---

RefSeq lookup is an independent tool. It is not affiliated with, or endorsed by, NCBI, UniProt or any of the databases it links to.

I developed this application with Claude and used GPT to review the code.
