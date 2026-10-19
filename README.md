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
- **From IDATs on the command line:** [SeSAMe2](https://github.com/zhou-lab/sesame) (`sesame` binary; docs at https://zhou-lab.github.io/sesame/)
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
| `<PLAT>.typeI_ext.tsv.gz` | Infinium-I extension base for the GCT bisulfite-conversion control: `C` (extension C — Grn here means unconverted), `T` (background reference), `.` otherwise. `M_general`-masked probes are excluded. Positional. |
| `<PLAT>.cnvnormals.cg` (+`.idx`) | CNV normal reference panel for `sesame cnv`: raw M/U signal as a [YAME](https://github.com/zhou-lab/YAME) format-3 `.cg`, one record per normal sample. Positional. EPIC, EPICv2, HM450 and MSA only. |
| `<PLAT>.titration.tsv.gz` | Methylation-standard response per probe (`floor`, `top`, `rho`) for calibrating betas to a modification fraction: `f = b'/(b' + rho^gamma (1 - b'))`, `b' = (beta - floor)/max(top - floor, 0.05)`, clamped to [0, 1] (`sesame calibrate`). `gamma = 0.15`, fitted on EPICv2 bACE 5hmC vs ACE-seq; provisional for MSA and MM285. `NA` where a standard is missing. Positional. EPICv2, MSA and MM285 only. |
| `KYCG/` | KnowYourCG feature sets as `.cm` (one dir, its own `SHA256SUMS`). Positional. |
| `SHA256SUMS` | Binds the set. Verify with `cd <PLAT> && sha256sum -c SHA256SUMS`. |

`strand` is the **interrogated cytosine's** strand (`+` = the C of the CpG on the
top strand at `CpG_beg`; `-` = the C complementing the G) — it matches sesame's
`probe_strand`. The interval width is implied by probe class (from the `Probe_ID`
prefix): a `cg` probe is a 2-bp CpG (`end = CpG_beg + 2`); `ch`/`rs`/`nv` probes
target a single base (`end = CpG_beg + 1`).

## Matching probe names to the files

The `coord`, `mask.cm`, and `KYCG/` files are **positional to the ordering** and
do not store probe IDs (the IDs live once, in the ordering). Attach them with
**`sesame attach-probe`**, which prepends the ordering's `Probe_ID` to any
positional file — a coord `.tsv.gz`, a `.cm` mask/feature, or a preprocess beta
`.cg`:

```bash
# Coordinate table with probe names:
sesame attach-probe --index MSA.ordering.tsv.gz MSA.hg38.coord.tsv.gz
#   Probe_ID   CpG_chrm   CpG_beg   strand   mapQ

# Mask matrix with probe names (unpacks the .cm and prepends Probe_ID):
sesame attach-probe --index MSA.ordering.tsv.gz MSA.hg38.mask.cm
#   Probe_ID   <one column per mask tag>
```

Use `--platform MSA` instead of `--index` to pull the ordering from the fetched
store. Without the binary, the same pairing is a positional paste, e.g.
`paste <(zcat MSA.ordering.tsv.gz | cut -f1) <(zcat MSA.hg38.coord.tsv.gz)`.

- **[SeSAMe2](https://github.com/zhou-lab/sesame)** (`sesame` binary) — `attach-probe` (above);
  also `preprocess` (IDATs → beta `.cg` in this same ordering), `dml`, `cnv`.
- **[YAME](https://github.com/zhou-lab/YAME)** reads/queries the `.cm` files
  directly (`yame unpack`, `yame summary`, …); rows come out in ordering order.
- In R, **[sesame](https://bioconductor.org/packages/sesame)** and
  **[KnowYourCG](https://github.com/zhou-lab/knowYourCG)** consume these directly.

## Versions

The repository is released as **git tags**; each tag is a self-consistent
snapshot (every file bound by `SHA256SUMS`). Use the latest tag (**v8.5**) unless
you need to reproduce an older result.

Common build parameters across all platforms:

- **Mapping:** [BISCUIT](https://github.com/zhou-lab/biscuit) (bisulfite-aware
  BWA-MEM) with an added primary-chromosome preference pass
  (`biscuit postalt --unplaced`) that keeps a probe on the main assembly when it
  also hits an equal-scoring unplaced/`_alt` contig.
- **Ordering:** from the manufacturer manifest (bead addresses + color channel),
  `LC_ALL=C` sorted — reproducible from any shell, authoritative for addresses.
  An Infinium-I channel comes from the alignment; where the probe does not
  align cleanly, it falls back to the manifest's `Color_Channel` (v8.3; HM27
  from v8.5).
- **coord strand:** the interrogated-cytosine strand (SAM FLAG), verified 100%
  against sesame's `probe_strand`.
- **Masks:** mapping masks (`M_mapping`, `M_nonuniq`) from the alignment; SNP
  masks from dbSNP (human) / Mouse Genomes Project strain SNPs (mm10).
- **Titration** (v8.4): fitted on BS-converted methylation standards (0, 5,
  10, 25, 50, 75, 100 %) run through sesame's default prep (QCDPB); replicate
  arrays of one fraction averaged. EPICv2: 206909630042 R01-R07C01 (Kaur
  2024, GSM7139651-57). MSA: 207760740037 R04-R16C02; only 55 % of probes pass
  pOOBAH on all seven, the rest are `NA`. MM285: the mouse series on
  204617710015 (the rat-DNA series on 204637490001 is excluded).

Tag history: `v1` MSA · `v2` +postalt, +EPICv2 · `v3` +HM450/EPIC, +`coord` ·
`v4` `coord` positional layout · `v5` `coord` strand fix · `v6` +MM285
(mm10+mm39) · `v7` +Mammal40/HM27, +`snp` · `v8` +`typeI_ext` ·
`v8.1` `typeI_ext` for all platforms · `v8.2` +`cnvnormals` ·
`v8.3` Infinium-I channel fallback (63 probes, HM450/EPIC/MSA) ·
`v8.4` +`titration` (EPICv2/MSA/MM285) ·
`v8.5` channel fallback on HM27 (1 probe).

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
