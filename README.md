# RefSeq lookup

A lightweight web app to find RefSeq protein and nucleotide records linked from UniProt, copy coding sequences, and link to gene pages at MGI, HGNC, FlyBase and other organism databases.


Version **2.2.6**. A single HTML file with no build step or application backend. It runs in the browser and uses the [UniProt REST API](https://www.uniprot.org/help/api) and, when you request a CDS, [NCBI E-utilities](https://www.ncbi.nlm.nih.gov/books/NBK25501/). This is an independent tool, not affiliated with NCBI or UniProt.

- Open the app: https://liucongl.github.io/refseq-lookup/ — no account needed.
- Source code and releases: https://github.com/LiucongL/refseq-lookup

## What you can enter

| Input | Example | What happens |
| --- | --- | --- |
| UniProt accession | `P68871` | Fetches the entry directly. A secondary accession opens the entry that now holds it, and says so. |
| Free text | `Foxo1`, `Dlx5 mouse`, `insulin human` | Searches UniProt; when several entries match, you pick one. Adding an organism helps narrow the results, but is optional. |
| RefSeq accession | `NP_000509.1`, `NM_000518` | Searches UniProt for an entry citing that accession. When found, it marks the cited record and points out a different cited version. A valid RefSeq record may have no matching UniProt result. |
| UniProt entry name | `P53_HUMAN` | Searches by entry name. |
| UniProt URL | `https://www.uniprot.org/uniprotkb/P68871/entry` | Takes the accession from the address. |

A UniProt isoform accession can also be entered. The app marks records assigned to that isoform, but the sequence displayed and used for CDS comparisons is the entry's **canonical sequence**. It does not fetch alternative isoform sequences.

## What the results show

- Protein name, UniProt accession, entry name, gene, organism, length, and UniProt review status.
- NCBI Gene links and links to MGI, HGNC, RGD, FlyBase, ZFIN, WormBase, SGD, and the Alliance of Genome Resources, where UniProt supplies the relevant cross-references. These direct links use database identifiers, not guessed gene symbols.
- **RefSeq records linked by UniProt**: each RefSeq protein and its associated nucleotide record, the record category (for example, known `NP_`, model `XP_`, non-redundant `WP_`), and an isoform assignment when supplied. A nucleotide record may be an mRNA, ncRNA, or genomic record.
- Links to the NCBI records and buttons to copy accessions. Transcript rows also offer **Copy CDS**.
- For mouse transcripts, a **View on UCSC mm10** link.
- The UniProt canonical sequence, copyable as FASTA or plain text.

**Companion app.** When its address is configured, each entry has **Open in Protein Explorer**, which opens the [Protein Explorer](https://liucongl.github.io/protein-explorer/) on that accession for structure, domains, disorder, and sequence information. 

## When UniProt lists no RefSeq link

The app shows the cross-references UniProt publishes. An empty list does **not** establish that no RefSeq record exists for the gene, or that no identical RefSeq protein exists. The app has not searched RefSeq itself.

Where the required information is available, it offers:

- **Search NCBI Gene**, using the gene symbol and organism, to find gene pages and their transcript and protein records.
- **BLAST against RefSeq Protein**, using the displayed UniProt sequence.
- For mouse, **Search UCSC mm10**, using the gene symbol.
- **Other UniProt entries** found by gene symbol and organism that have RefSeq links, with their review status, lengths, and cited RefSeq accessions.

These are routes to candidate records, not a declaration that they encode the sequence you started with. Check the organism, transcript, and sequence before choosing a record. Equal protein lengths alone do not establish sequence identity.

The related-entry search checks up to 25 UniProt entries in one request. If UniProt reports more results or does not provide a usable total, the page states the limited coverage and links to the full search. 

Selecting another UniProt entry changes the comparison target. Copy CDS, then check against the newly opened entry's canonical sequence, not the sequence you originally looked up. For example, a match after leaving a Clint1 entry does not establish a match to the original entry.

## CDS checks and copying

**Copy CDS** fetches the transcript's coding sequences and their translations from NCBI. It selects the CDS by the RefSeq protein's `protein_id`, never simply the first CDS returned. An exact protein version is preferred; a different version of the same protein accession is flagged and held for review. Malformed FASTA or a missing corresponding CDS is refused.

- **Matching translation and protein version:** copies the CDS immediately, as NCBI returns it, including any terminal stop codon. The row reports the comparison.
- **Different sequence, different protein version, or no comparison available:** the first click copies nothing. The row explains the result, and the button becomes **Copy anyway**. A further click explicitly copies that record; it does not make the comparison pass.

The comparison uses the canonical UniProt sequence. Records explicitly assigned to other isoforms are not compared; records without an isoform assignment can be compared with the canonical sequence. When NCBI's translation is unavailable, the app uses a standard genetic code translation and says so. This fallback does not apply to alternative genetic codes or translation exceptions; a complete matching fallback translation can still permit immediate copying.

A sequence match is useful evidence for choosing a CDS, not validation of your primer design, template, or final construct. A mismatch may reflect an isoform, variant, record version, or another sequence discrepancy.

Accession prefixes describe record categories. RefSeq status, such as REVIEWED, VALIDATED, or PROVISIONAL, is stated on the NCBI record and is not shown here. It is separate from UniProt's reviewed/unreviewed status and does not establish identity between the two databases' sequences.

## Mouse coordinates and mm10

For work aligned to mm10 (GRCm38), the UCSC links search that assembly's annotation by transcript accession without its version suffix, or by gene symbol when no transcript is linked. They do not convert coordinates or verify that the annotation matches the copied CDS.

Check the transcript and coding exons in UCSC, then copy the displayed `chr:start-end` position into IGV with **mm10/GRCm38** selected. 

## Limits and data requests

- **Search coverage:** free-text searches show up to six choices, with a full-results link when a larger total is reported. RefSeq input searches UniProt's cross-references and text; it is not a direct NCBI record lookup.
- **Availability:** database requests time out after 15 seconds once started. An unavailable service is not evidence that a record is absent.
- **NCBI rate:** the page starts its NCBI requests at least 400 ms apart and reuses successful responses. This limits this page's traffic; other tabs and users sharing an IP can still cause HTTP 429. Wait before retrying.
- **Clipboard:** if the browser refuses a copy, the app opens a dialog for copying by hand. A pending CDS fetch cannot overwrite a later copy action. A clipboard write already started by the browser cannot be canceled, so after rapid successive copies, check what you paste.

Lookups and related-entry queries go to UniProt; CDS requests go to NCBI. Gene-database and UCSC links open those services when clicked. The BLAST link sends the displayed protein sequence to NCBI. The page also loads its typeface from Google Fonts and falls back to system fonts if unavailable.

The app has no account system or application backend that stores searches. The host and external providers may log requests, and the browser may retain history or cached data. Searches should not be treated as private or anonymous.

## Linking to a result

You can share or bookmark a lookup using its page address. On the hosted app, look up an entry and copy the address from your browser, or save it as a bookmark. The address updates automatically as you search; you do not need to edit it yourself.

To create a link manually, add `?q=` followed by an accession or search term to the app address. For example, `?q=P68871` opens that UniProt entry; `?q=NP_000509.1` searches for that RefSeq accession; and `?q=Dlx5+mouse` runs a text search. The `+` represents a space.

When a RefSeq search goes through the picker, the app may also add `&ref=<RefSeq accession>` to preserve the requested-record marking after reload. Keep the complete address when sharing. Database records can change, so a bookmark repeats the lookup rather than preserving a snapshot of the sequence.

## Background

I originally developed this app to gather information for planning protein expression constructs. It can also be used for other tasks that need RefSeq records, coding sequences or links to gene information. The source is available to adapt for more specific workflows.

For cloning work, the app helps collect and compare information; it does not design primers or validate an expression construct. Choose the transcript and coding sequence appropriate for your experiment, including whether to retain the terminal stop codon when making a fusion.

## Running your own copy and customization

Open `refseq-lookup-v2.2.6.html` in a browser to try this draft. No build step is needed. You can modify the source to create more specific workflows and host your copy on GitHub Pages or another static hosting service. HTTPS hosting is recommended for browser features such as clipboard access. A downloaded HTML file still needs internet access for database requests, and local-file behavior varies by browser.

When the release is ready, set your addresses and publish a copy as `index.html` on GitHub Pages. Update this README and the version in `CITATION.cff` at that time. Use the opening description from this README for the description in `CITATION.cff` and GitHub About; it also appears in the page metadata. The release settings near the top of the script are:

```js
const APP_VERSION  = '2.2.6';
const REPO_URL     = 'https://github.com/LiucongL/refseq-lookup';
const EXPLORER_URL = 'https://liucongl.github.io/protein-explorer/';
```

If you host your own copy, replace `REPO_URL` and `EXPLORER_URL` with your own addresses. Set either to an empty string to hide its links. 

## Reporting problems

Use the issue tracker of the repository hosting your copy. Include the app version, search text or accession, browser, steps to reproduce, and the displayed message or a screenshot.

## Citation

See `CITATION.cff`, or GitHub's **Cite this repository** button, for this tool's citation. Also cite the underlying databases according to their guidance, and record accession identifiers **with their versions** and the date of access so the sequences used can be traced.

## Licence

MIT — see `LICENSE`.
