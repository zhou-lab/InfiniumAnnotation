# Annotation for Infinium DNA Methylation BeadChips

Probe-level annotation for the Illumina Infinium DNA methylation arrays: genomic
**coordinates**, **mapping quality**, design/quality **masks**, and
**[KnowYourCG](https://github.com/zhou-lab/knowYourCG) feature sets** (chromatin
states, transcription-factor binding, CpG islands, imprinting, …). Every
per-probe file is pinned **positionally** to one canonical probe *ordering* per
platform, and each release is an internally coherent, `SHA256SUMS`-verified,
git-tagged snapshot.

- **Interactive browser & downloads:** https://zwdzwd.github.io/InfiniumAnnotation
- **Analyze arrays in R:** [sesame](https://bioconductor.org/packages/sesame) (Bioconductor)
- **From IDATs on the command line:** [sesame-cli](https://github.com/zwdzwd/sesame-cli)
- **Feature enrichment:** [KnowYourCG](https://github.com/zhou-lab/knowYourCG)
- **Read/query the `.cm` files:** [YAME](https://github.com/zhou-lab/YAME)

## Platforms

| Platform | Array | Manifest | Genome | Probes | Reference |
|---|---|---|---|---|---|
| **MSA** | Infinium Methylation Screening Array (MSA-48v1-0) | B1 | hg38 | 284,309 | [Goldberg 2025](https://www.sciencedirect.com/science/article/pii/S2666979X25001855) |
| **EPICv2** | Infinium MethylationEPIC v2.0 (EPIC-8v2-0) | A2 | hg38 | 937,690 | [Kaur 2024](https://doi.org/10.1186/s43682-023-00021-5) |
| **EPIC** | Infinium MethylationEPIC v1.0 | B5 | hg38 | 866,553 | [Zhou 2016](https://doi.org/10.1093/nar/gkw967) |
| **HM450** | HumanMethylation450 | v1.2 | hg38 | 486,427 | [Zhou 2016](https://doi.org/10.1093/nar/gkw967) |
| **HM27** | HumanMethylation27 | v1.2 | hg38 | 27,722 | [Zhou 2016](https://doi.org/10.1093/nar/gkw967) |
| **MM285** | Infinium Mouse Methylation (MouseMethylation-12v1-0) | A2 | mm10, mm39 | 287,692 | [Zhou 2022](https://www.sciencedirect.com/science/article/pii/S2666979X22000775) |
| **Mammal40** | HorvathMammal40 Mammalian Methylation | Canonical 3.2019 | hg38 | 38,607 | [Arneson 2022](https://doi.org/10.1038/s41467-022-28355-z) |

Probe counts are exact rows in each `<PLAT>.ordering.tsv.gz`. Each manifest
revision is the current/final one Illumina ships for that array.

## What's in each platform directory

Files are **row-aligned**: data row *i* of every file below is the *same probe*.
The ordering is the only file that carries probe names — it is the index.

| File | Contents |
|---|---|
| `<PLAT>.ordering.tsv.gz` | **The probe index.** `Probe_ID`, `M`/`U` (bead addresses), `col` (color channel). Genome-independent; derived from the manufacturer manifest, `LC_ALL=C` sorted. |
| `<PLAT>.<genome>.coord.tsv.gz` | Per-probe genomic coordinate: `CpG_chrm`, `CpG_beg` (0-based), `strand`, `mapQ`. Positional; no probe IDs stored (see below). |
| `<PLAT>.<genome>.mask.cm` (+`.idx`) | Design/quality masks (`M_mapping`, `M_nonuniq`, SNP masks, …) as a [YAME](https://github.com/zhou-lab/YAME) `.cm` bitset — one record per mask. Positional. |
| `KYCG/` | KnowYourCG feature sets as `.cm` (one dir, its own `SHA256SUMS`). Positional. |
| `SHA256SUMS` | Binds the set. Verify with `cd <PLAT> && sha256sum -c SHA256SUMS`. |

`strand` is the **interrogated cytosine's** strand (`+` = the C of the CpG on the
top strand at `CpG_beg`; `-` = the C complementing the G) — it matches sesame's
`probe_strand`. The interval width is implied by probe class (from the `Probe_ID`
prefix): a `cg` probe is a 2-bp CpG (`end = CpG_beg + 2`); `ch`/`rs`/`nv` probes
target a single base (`end = CpG_beg + 1`).

## Matching probe names to the files

The `coord`, `mask.cm`, and `KYCG/` files are **positional to the ordering** and
do not store probe IDs (the IDs live once, in the ordering). Recover the pairing
by joining on row position:

```bash
# Coordinate table with probe names (headers and data both align):
paste <(zcat MSA.ordering.tsv.gz | cut -f1) <(zcat MSA.hg38.coord.tsv.gz)
#   Probe_ID   CpG_chrm   CpG_beg   strand   mapQ

# Mask matrix with probe names (yame emits data rows only, so drop the
# ordering header with tail -n +2):
paste <(zcat MSA.ordering.tsv.gz | tail -n +2 | cut -f1) \
      <(yame unpack -a MSA.hg38.mask.cm)
#   Probe_ID   <one column per mask tag>
```

- **[YAME](https://github.com/zhou-lab/YAME)** reads the `.cm` mask and feature
  files (`yame unpack -a`, `yame summary`, …); rows come out in ordering order.
- **[sesame-cli](https://github.com/zwdzwd/sesame-cli)** reads IDATs against this
  same ordering, so the β-values it emits are already in this row order — they
  line up with `coord` / `mask.cm` / `KYCG/` with no join.
- In R, **[sesame](https://bioconductor.org/packages/sesame)** and
  **[KnowYourCG](https://github.com/zhou-lab/knowYourCG)** consume these
  annotations directly.

## Versions

The repository is released as **git tags**; each tag is a self-consistent
snapshot (every file bound by `SHA256SUMS`). Use the latest tag (**v7**) unless
you need to reproduce an older result.

Common build parameters across all platforms:

- **Mapping:** [BISCUIT](https://github.com/zhou-lab/biscuit) (bisulfite-aware
  BWA-MEM) with an added primary-chromosome preference pass
  (`biscuit postalt --unplaced`) that keeps a probe on the main assembly when it
  also hits an equal-scoring unplaced/`_alt` contig.
- **Ordering:** from the manufacturer manifest (bead addresses + color channel),
  `LC_ALL=C` sorted — reproducible from any shell, authoritative for addresses.
- **coord strand:** the interrogated-cytosine strand (SAM FLAG), verified 100%
  against sesame's `probe_strand`.
- **Masks:** mapping masks (`M_mapping`, `M_nonuniq`) from the alignment; SNP
  masks from dbSNP (human) / Mouse Genomes Project strain SNPs (mm10).

Tag history: `v1` MSA · `v2` +postalt, +EPICv2 · `v3` +HM450/EPIC, +`coord` ·
`v4` `coord` positional layout · `v5` `coord` strand fix · `v6` +MM285
(mm10+mm39) · `v7` +Mammal40/HM27.

## References

Per platform:

- **MSA** — Goldberg *et al.*, *Scalable screening of ternary-code DNA methylation
  dynamics associated with human traits*,
  [Cell Genomics 2025](https://www.sciencedirect.com/science/article/pii/S2666979X25001855).
- **EPICv2** — Kaur *et al.*, *Comprehensive evaluation of the Infinium human
  MethylationEPIC v2 BeadChip*,
  [Epigenetics Communications 2024](https://doi.org/10.1186/s43682-023-00021-5).
- **EPIC / HM450 / HM27** — Zhou, Laird & Shen, *Comprehensive characterization,
  annotation and innovative use of Infinium DNA methylation BeadChip probes*,
  [Nucleic Acids Research 2016](https://doi.org/10.1093/nar/gkw967).
- **MM285** — Zhou, Hinoue, Barnes *et al.*, *DNA methylation dynamics and
  dysregulation delineated by high-throughput profiling in the mouse*,
  [Cell Genomics 2022](https://www.sciencedirect.com/science/article/pii/S2666979X22000775).
- **Mammal40** — Arneson, Haghani *et al.* (Horvath), *A mammalian methylation array
  for profiling methylation levels at conserved sequences*,
  [Nature Communications 2022](https://doi.org/10.1038/s41467-022-28355-z).

Methods:

- **Feature interpretation (KnowYourCG)** — Goldberg\*, Fu\*, *et al.*, *KnowYourCG:
  Facilitating base-level sparse methylome interpretation*, Science Advances 2025.
