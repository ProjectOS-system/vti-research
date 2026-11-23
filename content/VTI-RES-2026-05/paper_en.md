# Single-Pass Bounded-Memory Forensic Reconstruction of Multi-OS `[MS-VHDX]` Virtual Disks Under Intermittent Encryption: `GF(2)` Linear Checksum Decoupling, Ambiguity-Vetoed Bipartite Bijection, and Deterministic Boot Surgery

**Technical Identifier:** `VTI-RES-2026-05` — `v1.0` (November 2026)  
**Author:** **Caio Gonçalves** — *Principal Investigator & Chief Systems Architect, VTI Research (VTI, Brazil)* (https://projectos-system.github.io/vti-research/)  
**Scientific Classification:** `cs.CR` (Cryptography and Security) • `cs.OS` (Operating Systems) • `cs.DS` (Data Structures and Algorithms) | **Official DOI:** [`10.5281/zenodo.24452824`](https://doi.org/10.5281/zenodo.24452824)  
**Methodological Design:** *Design Science Research* stratified across four epistemic levels (`E1` Line-by-line static audit of `Recuperar-VHDX-v3.0.6.ps1` and `bastion-rescue-universal.py`; `E2` Structural calibration over virtual-disk failure profiles without quasi-identifiers; `E3` Formal construct $\mathcal{E}_{\mathrm{Rec}}$, 7 operators $\mathcal{O}$, 6 invariants $I_{R1}$–$I_{R6}$, Theorems `T1`–`T2` scoped, and Propositions `P3`–`P4`; `E4` Reproducible empirical batteries `B1`–`B6` executed in `os.tmpdir()` on synthetic `.vhdx` images) — Formal Tuple $\mathcal{E}_{\mathrm{Rec}} = (\mathcal{V}_{\mathrm{src}}, B_{\mathrm{ram}}, \mathcal{F}, \mathcal{O}, \mathcal{G}_{\mathrm{veto}}, \mathcal{P}_{\mathrm{prov}}, \mathcal{V}_{\mathrm{out}})$  
**Operational Confidentiality, Quasi-Identifier Prevention, and Editorial Demarcation Clause:** *No private server hostname, corporate domain, client name, or live credential appears in this manuscript. All empirical batteries `B1`–`B6` ran exclusively in laboratory isolation on synthetic `.vhdx` images in `os.tmpdir()`, leaving approved `rescue-portable/` binaries and live product databases untouched. This paper is scientifically autonomous from systemic monograph `VTI-RES-2026-04` (`v3.0`, DOI `10.5281/zenodo.24452820` / Concept DOI `10.5281/zenodo.24406145`), which exposes only the architectural interface `O1`–`O4` in §3.3.4.*

---

## Abstract

The proliferation of **intermittent encryption** (*First-N* / *header-only* / *skip-step*) ransomware targeting `[MS-VHDX]` virtual disk containers destroys redundant headers, region tables, and the Block Allocation Table (`BAT`) within minutes, leaving most internal `32 MiB` data payloads in plaintext yet severed from their logical offsets. Traditional forensic utilities and entropy-only classifiers fail for four coupled reasons: they require multiple random passes or unbounded resident memory (`RAM`), incur quadratic $O(K \cdot M \cdot n)$ cost when re-hashing per-inode seeded `CRC-32C` checksums on `ext4`, misclassify `unwritten` extents (`ee_len > 32768`) or symmetric `BAT` ties as recoverable blocks (*silent hallucination*), and execute blind repairs that corrupt dirty `regf` hives, `DPAPI` keys, `BitLocker`/`LUKS` protectors, or Active Directory `USN` sequences (`NTDS.dit`). This work formalizes the deterministic **single-pass, bounded-memory** forensic reconstruction machine $\mathcal{E}_{\mathrm{Rec}} = (\mathcal{V}_{\mathrm{src}}, B_{\mathrm{ram}}, \mathcal{F}, \mathcal{O}, \mathcal{G}_{\mathrm{veto}}, \mathcal{P}_{\mathrm{prov}}, \mathcal{V}_{\mathrm{out}})$ and empirically evaluates its executable implementation (`Recuperar-VHDX-v3.0.6.ps1`, `19,198` C#/PS lines, and `bastion-rescue-universal.py`, `3,890` Python 3 stdlib lines) via batteries `B1`–`B6`. We prove and verify: **(1)** exact $\mathbb{F}_2[x]$ (`GF(2)`) linear decoupling of `Castagnoli` `CRC-32C` with affine constants (`ext4` `xorout=0` vs. `iSCSI` `init=xorout=0xFFFFFFFF`), achieving `0/10,000` divergences in both regimes (`Clopper-Pearson 95% CI [0.0000, 0.00036882]`) and **`168.73x`** mean speedup (`bootstrap 95% CI [144.58x, 193.88x]`); **(2)** single-pass read/write with zero `BAT` seek-back and circular log replay (`260.32–269.34 MiB` CLR+PS5.1 Working Set vs. `221.18 MiB` analytical model at `100 GiB`) plus truncated-log rejection; **(3)** zero-error 4-way accounting across `200/200` `ext4` cases (`CP [0.0000, 0.0183]`) and Multi-FS `F1 = 1.000` with `0/25` JSON collision false positives (`p_holm <= 2.21e-10`); **(4)** bipartite `MCMF` with SPFA residual margin inside single components `n <= 8192`, achieving **`0/60` false assignments** on symmetric blocks (`Wilson 95% CI [0.0000, 0.0602]`, `<= 6.02%`, `p_holm = 4.89e-13`) while explicitly demarcating heuristic windowing `> 8192`; and **(5)** `20/20` dirty `regf` hive blocks and `30/30` Pre-FS hard vetoes (`p_holm = 4.35e-11`). In strict scientific honesty, we report two open boundaries discovered in frozen artifact `v3.0.6`: silent read-failure skipping on `ext4` index nodes `eh_depth > 0` (`L10747`), scoping Theorem 2 to readable nodes and opening **`M-FUT-1`**, and partial `25/30` (`83.33%`, $\kappa = 0.6667$) `HardVetoes` agreement between PS/C# and Python in `B6` (partial refutation of $H_6$), opening **`M-FUT-2`**, grounded in **28 verified primary references (`[R01]`–`[R28]`)**.

**Keywords:** `[MS-VHDX]` Forensics; Intermittent Encryption; `GF(2)` Linear `CRC-32C`; Bipartite Minimum-Cost Maximum Flow (`MCMF`); 4-Way `ext4` Accounting; `regf` Registry Surgery; Multi-FS Recovery; Single-Pass Streaming.

---

## 1. Introduction, Empirical Motivation, and Demarcation From `VTI-RES-2026-04`

Modern ransomware economics optimizes damage per unit of I/O time. As documented by Davies et al. (`[R07]`), Ineza et al. (`[R08]`), and Wang et al. (`[R09]`), contemporary variants deploy **intermittent encryption** (*First-N* or *skip-step*) across `[MS-VHDX]` virtual disk images (`[R16]`), obliterating container headers, circular metadata logs, and the `BAT` while leaving most `32 MiB` payload blocks intact. Conventional forensic tools confront four structural barriers: (1) unbounded RAM/multi-pass I/O unsuitable for *WinPE*/*Linux Live* rescue (`[R13, R15]`); (2) quadratic $O(K \cdot M \cdot n)$ re-hashing of per-inode seeded `ext4` `metadata_csum` `CRC-32C` (`[R01, R17, R18]`); (3) silent hallucinations from magic-byte JSON collisions (`[R10, R11]`), entropy-only classifiers (`[R07]`), `unwritten` extents (`ee_len > 32768`), and greedy tie-breaking; and (4) secondary destruction when blindly patching dirty `regf` hives (`seq1 != seq2`, `[R25]`), `BitLocker`/`DPAPI` (`[R26]`), or `Active Directory` (`NTDS.dit` *USN Rollback* `[R27]`).

### 1.1. Anti-*Salami-Slicing* Demarcation Matrix (`VTI-RES-2026-04 v3.0` $\leftrightarrow$ `VTI-RES-2026-05`)

| Scientific Layer / Artifact | Status in `VTI-RES-2026-04 v3.0` | Status in `VTI-RES-2026-05 v1.0` | Anti-*Salami-Slicing* Boundary |
| :--- | :---: | :---: | :--- |
| **Systemic interface `O1`–`O4` in Bastion DR plane** | **Inherited** *(§3.3.4 interface only)* | **Extended** *(embedded in 7-operator chain $\mathcal{O}$)* | `2026-04` claims no algebraic, Multi-FS, or `MCMF` proofs; cites `2026-05`. |
| **`GF(2)` linear `CRC-32C` decoupling (`ShiftZero_n`) + affine adjustment** | Absent | **New** *(Theorem `T1`, proof $\blacksquare$, Battery `B1`)* | Polynomial algebra over $\mathbb{F}_2[x]$ and standalone `4x256` tables. |
| **Single-Pass RAM-bounded streaming (`<= 480 MiB`) & `[MS-VHDX]` log replay** | Absent | **New** *(Operators $O_{\mathrm{Log}}, O_{\mathrm{Pass1}}, O_{\mathrm{Pass2}}$ & `B2`)* | Zero-seek-back `BAT` memory/I/O analysis. |
| **4-Way `ext4` (`unwritten`) accounting & Multi-FS (`NTFS`/`ReFS`/`Btrfs`/`LVM2`/`XFS`)** | Absent (`04` mentions `ext2/3/4` only) | **New** *(Scoped Theorem `T2`, $I_{R3}$, Battery `B3`)* | Transparently discloses `L10747` defect (`M-FUT-1`). |
| **Bipartite `MCMF` + Residual SPFA Margin $\Delta$ & `AMBIGUOUS_BAT_MAPPING` veto** | Absent | **New** *(Proposition `P3`, alternating cycles, `B4`)* | Separates global `<= 8192` vs. heuristic windowing `> 8192`. |
| **Transactional `regf` surgery, Pre-FS gates & Cross-OS parity** | Absent | **New** *(Proposition `P4`, `B5`, `B6`)* | Reports `B6 = 25/30` ($\kappa = 0.6667$) as partial refutation of $H_6$. |

---

## 2. Formal Model $\mathcal{E}_{\mathrm{Rec}}$, Epistemic Stratification, and Hypotheses (`H1`–`H6`)

### 2.1. Epistemic Stratification (`E1`–`E4`)

- **`E1`:** Line-by-line static audit of frozen artifacts `Recuperar-VHDX-v3.0.6.ps1` (`19,198` lines, `SHA-256: db547d1d...`) and `bastion-rescue-universal.py` (`3,890` lines, `SHA-256: 493228b8...`).
- **`E2`:** Structural failure-profile calibration (`MBR`/`GPT`, `NTFS`/`ext4`, `EqualSeqDisk2Vhdx`) with zero private hostnames or PII.
- **`E3`:** Closed-form proofs (`T1`, scoped `T2`, `P3`, `P4`), invariants $I_{R1}$–$I_{R6}$, and negative bounds (`BootReadinessScore` explicitly defined as a deterministic heuristic; windowing `> 8192` defined as local heuristic).
- **`E4`:** Deterministic benchmark harness (`run_vhdx_b1_b6_bench.ps1` + `compute_stats.py`, `seed = 20261122`) executed in `os.tmpdir()` (`0` touches to `bastion.db*`), explicitly noting internal oracle scope in `B1`, `1–8 GiB` measured + `100 GiB` analytical projection in `B2`, and synthetic `Ctrl-MagicCarver` (not invoking SleuthKit/TestDisk).

### 2.2. Formal 7-Tuple $\mathcal{E}_{\mathrm{Rec}}$, Operator Chain $\mathcal{O}$, and Six Invariants ($I_{R1}$–$I_{R6}$)

$$\mathcal{E}_{\mathrm{Rec}} = \big(\mathcal{V}_{\mathrm{src}},\, B_{\mathrm{ram}},\, \mathcal{F},\, \mathcal{O},\, \mathcal{G}_{\mathrm{veto}},\, \mathcal{P}_{\mathrm{prov}},\, \mathcal{V}_{\mathrm{out}}\big)$$

where $\mathcal{V}_{\mathrm{src}}$ is opened strictly read-only (`FileOptions.SequentialScan`), $B_{\mathrm{ram}}$ bounds memory (`<= 480 MiB` target at `<= 100 GiB` with `96 MiB` LRU metadata cache), $\mathcal{F} = \{\texttt{ext2/3/4+JBD2}, \texttt{NTFS}, \texttt{ReFS}, \texttt{Btrfs}, \texttt{LVM2}, \texttt{FAT32\_ESP}, \texttt{LINUX\_SWAP}, \texttt{XFS\_RO}\}$ (`[R17]`–`[R24]`), $\mathcal{O} = (O_{\mathrm{Log}}, O_{\mathrm{Pass1}}, O_{\mathrm{GF2}}, O_{\mathrm{Probe}}, O_{\mathrm{MCMF}}, O_{\mathrm{Surg}}, O_{\mathrm{Pass2}})$, $\mathcal{G}_{\mathrm{veto}}$ enforces Pre-FS hard vetoes (`BitLocker`/`LUKS` `[R26]`, `MD_RAID`, `AD DS NTDS.dit` `[R27]`, dirty `regf` `[R25]`, `AMBIGUOUS_BAT_MAPPING`), $\mathcal{P}_{\mathrm{prov}}$ assigns one of **7 disjoint provenance states**, and $\mathcal{V}_{\mathrm{out}}$ writes a 1 MiB-aligned single-pass clone verified by `SHA-256` (`[R28]`).

**Six Formal Invariants:**
- **$I_{R1}$ (Evidence Immutability, `ISO/IEC 27037` `[R15]`):** Zero writes to $\mathcal{V}_{\mathrm{src}}$.
- **$I_{R2}$ (Single-Pass & Zero BAT Seek-Back):** $1$ sequential read pass ($O_{\mathrm{Pass1}}$) and $1$ sequential write pass ($O_{\mathrm{Pass2}}$) with zero backward seeks on the `BAT`.
- **$I_{R3}$ (4-Way Physical-Virtual Conservation):**
$$|\mathcal{B}_{\mathrm{phys}}| = N_{\mathrm{mapped}} + N_{\mathrm{enc}} + N_{\mathrm{zero}} + N_{\mathrm{orphan}}, \qquad \mathrm{EffAlloc} = \mathrm{MappedAlloc} + \mathrm{MissingAlloc}, \qquad \mathrm{RawAlloc} = \mathrm{EffAlloc} + \mathrm{UnwrittenOnly}$$
- **$I_{R4}$ (Bit-Exact `GF(2)` Equivalence):** For $n \in \{4084, 4088, 4092\}$ and any seed $s$, $O_{\mathrm{GF2}}$ matches full hashing bit-for-bit (`VerifyLinearShifts`).
- **$I_{R5}$ (Epistemic Non-Hallucination):** No gap with residual margin $\Delta_u < \Delta_{\min}$ receives `INFERRED_UNIQUE`; any `INFERRED_AMBIGUOUS` or `HardVeto` caps `BootReadinessScore <= 39`.
- **$I_{R6}$ (Transactional `regf` Conservation):** No mutation runs when `seq1 != seq2` or `cbData` changes; clean mutations preserve `seq1 == seq2`, recompute `XOR-32` (`508 B`), and save a `DPAPI` backup.

### 2.3. Falsifiable Hypotheses (`H1`–`H6`)

$H_1$ tests `0` `GF(2)` divergences and `>= 20x` speedup (`O_GF2`); $H_2$ tests single-pass zero-seek-back, `EqualSeqDisk2Vhdx` replay, and truncated-log rejection; $H_3$ tests `0` 4-way accounting errors under `unwritten` extents, `XFS READONLY`, and `F1 >= 0.98` rejecting JSON collisions; $H_4$ tests `MCMF` + residual SPFA margin reducing FAR vs. greedy/magic baselines while catching alternating cycles in `n <= 8192`; $H_5$ tests `100%` blocking of dirty `regf` and Pre-FS triggers (`0` ELF `"-FVE-FS-"` FPs); and $H_6$ tests cross-OS parity $\kappa \ge 0.98$ between PS/C# and Python 3 *(confirmed `30/30` on `osFamily`/Pre-FS, partially refuted on `HardVetoes` at `25/30`, $\kappa = 0.6667$)*.

---

## 3. Theorems, Propositions, Formal Proofs ($\blacksquare$), and Negative Bounds

### Theorem 1 (`T1` — Exact $\mathbb{F}_2[x]$ Linear Decoupling of `CRC-32C` with Affine Adjustment)
**Statement.** Let $P(x) \in \mathbb{F}_2[x]$ be the `Castagnoli` generator polynomial (`0x1EDC6F41` normal / `0x82F63B78` reflected, `[R01, R02, R03]`) with `refin = refout = true`, and let $A \in \mathbb{F}_2^{32 \times 32}$ be the 1-bit transition matrix of uncomplemented reflected `RawCrc32cUpdate`. For any $n$-byte block $X$ and initial state $s \in \mathbb{F}_2^{32}$:
1. In the **`ext4` regime (`xorout = 0`, `[R18]`):** $\mathrm{CRC}_{\mathrm{ext4}}(s, X) = \mathrm{RawCrc32c}(0, X) \oplus A^{8n} s$.
2. In the **affine `iSCSI` / `[MS-VHDX]` regime (`init = xorout = 0xFFFFFFFF`, `[R02, R16]`):** $\mathrm{CRC}_{\mathrm{iSCSI}}(s_{\mathrm{init}}, X) = (\mathrm{RawCrc32c}(0, X) \oplus \texttt{0xFFFFFFFF}) \oplus A^{8n} s_{\mathrm{init}}$.

Moreover, $\mathrm{ShiftZero}_n(s) = A^{8n} s$ decomposes into four 256-word lookup tables (`4 KiB`), evaluating in $O(1)$ time independent of $n$.

**Proof.** Each reflected bit update applies the affine step $v \mapsto A v \oplus b_j e_0$ over $\mathbb{F}_2^{32}$. By induction over $8n$ bits, $\mathrm{RawCrc32c}(s, X) = A^{8n} s \oplus \mathrm{RawCrc32c}(0, X)$, yielding (1) for $\alpha_{\mathrm{out}} = 0$ and (2) upon XORing $\alpha_{\mathrm{out}} = \texttt{0xFFFFFFFF}$. Expanding $s = \bigoplus_{k=0}^{3}(s_k \ll 8k)$ gives $A^{8n} s = \bigoplus_{k=0}^{3} \mathrm{tbl}[256k + s_k]$ using 4 table lookups and 4 XORs. $\blacksquare$

### Theorem 2 (`T2` — 4-Way Physical-Virtual Conservation on Readable `ext4` Extent Trees)
**Statement (Explicit Conditional Scope).** Given an `ext4` partition $\Pi$ where **all traversed `0xF30A` extent-tree index nodes are readable without I/O error**, mutually exclusive physical classification into $\{\mathrm{mapped}, \mathrm{fullEnc}, \mathrm{zero}, \mathrm{orphan}\}$ and classification of valid extents with `ee_len > 32768` as `ALLOCATED_UNWRITTEN_LOGICAL_ZERO` guarantee $I_{R3}$ conservation with zero unwritten extents counted in $\mathrm{MissingAlloc}$.

**Proof.** Physical iteration assigns each entry to exactly one counter (`L14923`). In the virtual domain (`[R18]`), `ee_len > 32768` partitions `allRawAllocV` into $\mathrm{EffAlloc} \sqcup \mathrm{UnwrittenOnly}$; under full index-node readability, every block in $\mathrm{EffAlloc}$ enters either `allMappedAllocV` or `allMissingAllocV`, preserving both identities. $\blacksquare$

> **Open Defect Transparently Disclosed (`Recuperar-VHDX-v3.0.6.ps1:L10747` $\rightarrow$ `M-FUT-1`):** `CollectExt4ExtentLeavesRam` checks `if (ReadVirtualRam(...))` without an `else` branch; on a failed index-node read (`eh_depth > 0`), the subtree is silently skipped without propagating `INDETERMINATE`. Theorem 2 therefore excludes unread `eh_depth > 0` subtrees, opening `M-FUT-1`.

### Proposition 3 (`P3` — Conservative `SSAP/MCMF` Correctness with Residual SPFA Margin in Single Component `n <= 8192`)
**Statement.** On a single connected bipartite component (`totalNodes <= 8192`) with scaled `Int64` costs and direct rejection edges `rejectCostScaled`, topological Johnson potential initialization (`S -> U -> C -> T`), unit-capacity `SSAP` via Dijkstra + `MinHeap`, and temporary 1-unit unrouting of $S \to u \to p^* \to T$ with $(u, p^*)$ blocked followed by residual SPFA (`L5060–5155`) compute the exact global alternating-cycle margin $\Delta_u = \mathrm{OPT}(E \setminus \{(u,p^*)\}) - \mathrm{OPT}(E) = \mathrm{altCost} - w(u,p^*) \ge 0$, downgrading any tie or cycle with $\Delta_u < \Delta_{\min}$ to `INFERRED_AMBIGUOUS` (`AMBIGUOUS_BAT_MAPPING`).

**Proof.** Initial positive-capacity arcs form a DAG, guaranteeing valid non-negative Johnson potentials; unit-capacity `SSAP` maintains integrality and optimality (`[R04, R05, R06]`). Unrouting 1 unit leaves an optimal $|\mathcal{U}|-1$ flow free of negative cycles, and residual $u \leadsto T$ paths bijectively represent feasible alternating reassignment chains or rejection; SPFA finds their exact minimum $\mathrm{altCost}$. $\blacksquare$

> **Explicit Demarcation (`> 8192` Nodes):** Above `8192` nodes (`L5267–5325`), the engine partitions into sub-windows (`winV < 2048`), operating explicitly as a local heuristic (`HEURISTIC_WINDOWED`) without global alternating-cycle certification.

### Proposition 4 (`P4` — Transactional Offline `regf` Surgery Safety with Invariant `cbData`)
**Statement.** Modifying a `regf` hive (`[R25]`) strictly when `seq1 == seq2`, keeping `cbData` invariant (noting auxiliary reallocation branches in `softwareHive` `L5851–5868`/`L5892–5898` outside this premise), saving a sibling `DPAPI` backup, preserving `seq1 == seq2` unchanged, and atomically recomputing the `508`-byte `XOR-32` checksum guarantees zero dirty-hive mutations and zero `hbin` cell displacement. $\blacksquare$

### 3.5. Irreducible Negative Bounds & Heuristic Status of `BootReadinessScore`
Overwritten ciphertext prefixes without redundant copies are information-theoretically unrecoverable (`CIPHERTEXT_CONFIRMED`); JSON magic collisions are rejected; truncated `[MS-VHDX]` logs fail closed (`LogReplayed = false`); Windows Fast Startup (`hiberfil.sys`) lies outside `regf`; `BitLocker`/`LUKS`/`NTDS.dit` fire hard vetoes; and `BootReadinessScore` (`0..100`) is strictly a deterministic prioritization heuristic capped at `<= 39` under any `HardVeto`.

---

## 4. Experimental Protocol and Reproducible Empirical Results (`B1`–`B6`)

All batteries ran on frozen artifacts (`Recuperar-VHDX-v3.0.6.ps1` + `bastion-rescue-universal.py`) via `run_vhdx_b1_b6_bench.ps1` + `compute_stats.py` (`seed = 20261122`, Windows NT `10.0.26220.0`, PowerShell `5.1.26100.9836`) in `os.tmpdir()` (`LEFTOVER_TMP_COUNT = 0`) against six synthetic baselines (`Ctrl-MultiPass`, `Ctrl-EntropyOnly`, `Ctrl-GreedyNoMargin`, `Ctrl-MagicCarver` [synthetic, without SleuthKit/TestDisk], `Ctrl-BlindPatch`, `Ctrl-NegRandom`):

| Battery | Hypothesis / Target | Verified Empirical Results (`b1_b6_results.json`) | Statistical Estimators (`95% CI` & `Holm-Bonferroni`) | Epistemic Verdict (`E4`) |
| :---: | :--- | :--- | :--- | :---: |
| **`B1`** | **$H_1$ (`O_GF2` vs. `Ctrl-MultiPass`)** | **`0 / 10,000`** `ext4` divergences; **`0 / 10,000`** `iSCSI` divergences; `400,000` checks (`10` rounds): **`1.6474 ms`** (`24.28M/s`) vs. **`277.9601 ms`** (`143.9K/s`). | `CP 95% [0.0000, 0.00036882]` both; ratio-of-means speedup **`168.73x`** (`bootstrap 2000`, `95% CI [144.58x, 193.88x]`). | **`PASS`** *(external vectors -> `M-FUT-3`)* |
| **`B2`** | **$H_2$ (Single-Pass RAM & Log)** | `1GiB_log_replay`: `EqualSeqDisk2Vhdx=true`, `LogReplayed=true`, `ReadPasses=1`, `WritePasses=1`, `ZeroSeekBackBat=true`, WS **`260.32 MiB`** (`Private 241.91 MiB`). `8GiB`: WS **`269.34 MiB`**. Truncated log: **`LogReplayed=false`**. | Reconciled: **`221.18 MiB`** analytical index + `96 MiB` cache at `100 GiB` vs. **`260.32–269.34 MiB`** incl. `.NET 4.x` CLR + PS `5.1` host. | **`PASS (reconciled)`** |
| **`B3`** | **$H_3$ (4-Way `I_R3` & Multi-FS)** | **`200 / 200`** `ext4` cases (`0` accounting errors, `50` unwritten); `XFS` = `READONLY`. Multi-FS (`N=100`): Bastion **`F1 = 1.000`** (`TP=60, FP=0, FN=0, TN=40`, `0/25` JSON FP) vs. `Ctrl-EntropyOnly` & `Ctrl-MagicCarver` (`F1 = 0.8276`, `FP=25, TN=15`). | Accounting error `CP [0.0000, 0.0183]`; Fisher + Holm: `p_holm = 2.21e-10` (`EntropyOnly`) and **`p_holm = 7.91e-14`** (`MagicCarver`). | **`PASS`** |
| **`B4`** | **$H_4$ (`O_MCMF` + $\Delta$)** | Unique: `60/60`. Symmetric: **`0/60` FAR** vs. `60/60` FAR in `Greedy`/`MagicCarver`. Alternating cycle caught (`delta=10 < 12`). `Ctrl-NegRandom`: `0/100` FP, `100/100` `score <= 39`. | Symmetric **`two-sided 95% Wilson [0.0000, 0.0602]` (`<= 6.02%`, not claiming `< 5%`)**, **`p_holm = 4.89e-13`**. | **`PASS (Wilson <= 6.02%)`** |
| **`B5`** | **$H_5$ (`O_Surg` `regf` & Pre-FS)** | `20/20` clean `SAM` repaired (`XOR-32` valid, `DPAPI` backup); `20/20` dirty blocked vs. `20/20` mutated in `BlindPatch`; `30/30` Pre-FS blocked; `0/20` ELF `"-FVE-FS-"` FPs vs. `20/20`. | Clean `CP [0.8316, 1.0000]`; Pre-FS `CP [0.8843, 1.0000]`; **`p_holm = 4.35e-11`**. | **`PASS`** |
| **`B6`** | **$H_6$ (Cross-OS Parity $N=30$)** | `osFamily`/`BitLocker`/`NTDS.dit`/`XFS RO`: **`30/30`**. `HardVetoes`/`full_parity`: **`25/30` (`83.33%`)** — Scenario 2 (`5/5`) fired `AMBIGUOUS_BAT_MAPPING` in Python (`score=39`) but not PS (`score=61`). | `osFamily`/Pre-FS $\kappa = 1.000$; **`HardVetoes` `25/30` (`CP 95% [0.6528, 0.9436]`, $\kappa = 0.6667$)** ($p_o=0.8333, p_e=0.5000$). | **`PARTIAL REFUTATION OF H6` (`M-FUT-2`)** |

**Holm–Bonferroni Protocol ($m=5$ two-sided Fisher exact tests, $\alpha=0.05$):** Rank 1 `B3` JSON (`0/25` vs `25/25`, $p_{\mathrm{holm}}=7.911\times 10^{-14}$); Rank 2 `B4` symmetric (`0/60` vs `60/60`, $p_{\mathrm{holm}}=4.893\times 10^{-13}$); Rank 3 `B5` dirty `regf` (`0/20` vs `20/20`, $p_{\mathrm{holm}}=4.353\times 10^{-11}$); Rank 4 `B5` ELF `"-FVE-FS-"` (`0/20` vs `20/20`, $p_{\mathrm{holm}}=4.353\times 10^{-11}$); Rank 5 `B3` `Ctrl-EntropyOnly` (`0/40` vs `25/40`, $p_{\mathrm{holm}}=2.214\times 10^{-10}$).

---

## 5. Critical Validation, Residual Roadmap (`M-FUT-1`–`M-FUT-6`), and Traceability

Six next-cycle engineering milestones address every open boundary:
- **`M-FUT-1`:** Tri-state (`VALID / INVALID / INDETERMINATE`) propagation in `CollectExt4ExtentLeavesRam` (`L10747`) to close Theorem 2 unconditionally.
- **`M-FUT-2`:** Normative unification of `AMBIGUOUS_BAT_MAPPING` between C#/PS (`L14050`) and Python (`L2297`) to raise `B6` parity from `25/30` to `30/30`.
- **`M-FUT-3`:** External `e2fsprogs`/`RFC 3720` test vectors for `B1` and direct physical `>= 100 GiB` Working Set benchmarks for `B2`.
- **`M-FUT-4`:** Explicit inter-window cut flagging (`CERTIFIED_GLOBAL` vs. `HEURISTIC_WINDOWED`) above `8192` nodes.
- **`M-FUT-5`:** Mid-write crash-consistency fault injection between `BAT` and payload plus expanded `ReFS`/`Btrfs` suites.
- **`M-FUT-6`:** Auxiliary detection of hybrid hibernation (`hiberfil.sys` / Fast Startup) outside `regf`.

All 28 verified primary references (`[R01]`–`[R28]`) map `1:1` to `references.bib`: `GF(2)`/`CRC-32C` (`[R01]`–`[R03, R18]`), single-pass/`[MS-VHDX]` (`[R13, R15, R16, R28]`), 4-way/Multi-FS/intermittent ransomware (`[R07]`–`[R12, R14, R17]`–`[R24]`), `MCMF`/assignment (`[R04]`–`[R06]`), and `regf`/Pre-FS (`[R25]`–`[R27]`).

---

## 6. Threats to Validity

- **Construct Validity:** `Ctrl-MagicCarver` is a synthetic header carver rather than live SleuthKit/TestDisk binaries; `B1` relies on an internal full-hash oracle (external `e2fsprogs` vectors in `M-FUT-3`); `BootReadinessScore` is heuristic.
- **Internal Validity:** `L10747` (`eh_depth > 0`) and `> 8192` windowing are explicitly demarcated; `.NET 4.x` CLR overhead accounts for the gap between `260–269 MiB` measured WS and `221.18 MiB` analytical index memory.
- **External & Re-ID Validity:** Synthetic `1–8 GiB` corpus in `os.tmpdir()` with zero private hostnames or field PII; physical `>= 100 GiB` and deeper `ReFS`/`Btrfs` field replication remain in `M-FUT-3`/`M-FUT-5`.
- **Conclusion Validity:** Finite sample size `n=60` in `B4` yields upper Wilson bound `6.02%` (reducing `< 1%` requires $n \ge 369$); `B6` `25/30` ($\kappa = 0.6667$) is reported transparently as partial refutation of $H_6$.

---

## 7. Conclusion

We formalized and empirically evaluated $\mathcal{E}_{\mathrm{Rec}}$ for single-pass, bounded-memory forensic reconstruction of multi-OS `[MS-VHDX]` containers damaged by intermittent ransomware encryption. Combining exact `GF(2)` `CRC-32C` decoupling (`0/10,000` errors, `168.73x` speedup), zero-seek-back single-pass streaming (`260.32–269.34 MiB` WS reconciled with `221.18 MiB` analytical `100 GiB`), 4-way `ext4` accounting (`0/200` errors, Multi-FS `F1=1.000`), residual SPFA `MCMF` (`0/60` FAR, `Wilson <= 6.02%`), and transactional `regf`/Pre-FS gates (`100%` dirty/encrypted/DC veto) establishes a rigorous foundation while openly documenting two concrete `v3.0.6` gaps (`M-FUT-1` at `L10747` and `M-FUT-2` at `B6` `25/30`) for `v3.1`.

---

## Transparent Disclosure of Symbiotic Human-AI Engineering, Deterministic Verification, and Adapted CRediT Mapping (ACM / IEEE / COPE Compliance)

In strict compliance with **COPE**, **ACM** (*Policy on Authorship*, 2023), and **IEEE** (*PSPB Operations Manual*), automated tools and language models cannot hold authorship or bear ethical, legal, or epistemic responsibility. **Caio Gonçalves** is the **sole formal author**, assuming full accountability for all proofs, designs, benchmarks, and conclusions:
1. **Scientific Direction & Exclusive Human Authorship — Caio Gonçalves** (*Principal Investigator & Chief Systems Architect, VTI Research*): *Conceptualization, Methodology, Architecture & Supervision, Validation, Project Administration, and Writing — Review & Editing* — including the explicit scientific decision to keep open defects `M-FUT-1` and `M-FUT-2` fully visible.
2. **Non-Authorial Adversarial Scrutiny:** Construct $\mathcal{E}_{\mathrm{Rec}}$, proofs `T1`–`P4`, anti-salami-slicing boundaries, and `B1`–`B6` underwent **3 structured rounds of cross-family adversarial review (`R1`–`R3`)** via the `MemoryOS` gateway (`docs/research/ATA_CONSELHO_FLAGSHIP_VTI_RES_2026_05.md`), whose Round 2 objections directly drove the conditional scoping of `T2` (`L10747`) and honest reporting of `B6` (`25/30`).
3. **Deterministic Instrumentation & Bibliographic Audit:** Automated support scripts executed static auditing, `run_vhdx_b1_b6_bench.ps1` + `compute_stats.py` (`b1_b6_results.json`), and primary verification of **100% of the 28 references (`[R01]`–`[R28]`)** mapped `1:1` in `references.bib`.

---

## Verified Scientific and Normative References (`[R01]`–`[R28]`)

1. **`[R01]`** Castagnoli, G., Braeuer, S., & Herrmann, M. (1993). *Optimization of Cyclic Redundancy-Check Codes with 24 and 32 Parity Bits*. *IEEE Trans. Commun.*, 41(6), 883–892. DOI: `10.1109/26.231911`.
2. **`[R02]`** Satran, J., et al. (2004). *Internet Small Computer Systems Interface (iSCSI)*. IETF RFC 3720. DOI: `10.17487/RFC3720`.
3. **`[R03]`** Chadalapaka, M., et al. (2014). *iSCSI Protocol (Consolidated)*. IETF RFC 7143. DOI: `10.17487/RFC7143`.
4. **`[R04]`** Edmonds, J., & Karp, R. M. (1972). *Theoretical Improvements in Algorithmic Efficiency for Network Flow Problems*. *JACM*, 19(2), 248–264. DOI: `10.1145/321694.321699`.
5. **`[R05]`** Jonker, R., & Volgenant, A. (1987). *A Shortest Augmenting Path Algorithm for Dense and Sparse Linear Assignment Problems*. *Computing*, 38, 325–340. DOI: `10.1007/bf02278710`.
6. **`[R06]`** Ahuja, R. K., Magnanti, T. L., & Orlin, J. B. (1993). *Network Flows: Theory, Algorithms, and Applications*. Prentice Hall. ISBN: `978-0-13-617549-0`.
7. **`[R07]`** Davies, S. R., Macfarlane, R., & Buchanan, W. J. (2022). *Comparison of Entropy Calculation Methods for Ransomware Encrypted File Identification*. *Entropy*, 24(10), 1503. DOI: `10.3390/e24101503`.
8. **`[R08]`** Ineza, Y., et al. (2025). *Intermittent File Encryption in Ransomware: Measurement, Modeling, and Detection*. *arXiv:2510.15133*.
9. **`[R09]`** Wang, Z., et al. (2024). *Ransom Access Memories: Practical Ransomware Protection in Cloud with DeftPunk*. *USENIX OSDI 24*.
10. **`[R10]`** Garfinkel, S. L. (2007). *Carving contiguous and fragmented files with fast object validation*. *Digital Investigation*, 4(Suppl. 1), 2–12. DOI: `10.1016/j.diin.2007.06.017`.
11. **`[R11]`** Pal, A., & Memon, N. (2009). *The Evolution of File Carving*. *IEEE Signal Processing Magazine*, 26(2), 59–71. DOI: `10.1109/MSP.2008.931081`.
12. **`[R12]`** Garfinkel, T., & Rosenblum, M. (2003). *A Virtual Machine Introspection Based Architecture for Intrusion Detection*. *NDSS 2003*.
13. **`[R13]`** Tobin, P., Le-Khac, N.-A., & Kechadi, M.-T. (2017). *Forensic Analysis of Virtual Hard Drives*. *JDFSL*, 12(1). DOI: `10.15394/jdfsl.2017.1438`.
14. **`[R14]`** Carrier, B. (2005). *File System Forensic Analysis*. Addison-Wesley Professional. ISBN: `978-0321268174`.
15. **`[R15]`** ISO/IEC (2012). *ISO/IEC 27037:2012 — Guidelines for digital evidence preservation*. Geneva: ISO/IEC.
16. **`[R16]`** Microsoft Corporation (2024). *[MS-VHDX]: Virtual Hard Disk v2 (VHDX) File Format Open Specification*.
17. **`[R17]`** Mathur, A., et al. (2007). *The New ext4 Filesystem: Current Status and Future Plans*. *Ottawa Linux Symposium*, Vol. 2, pp. 21–34.
18. **`[R18]`** The Linux Kernel Organization (2024). *ext4 Data Structures and Algorithms*.
19. **`[R19]`** Microsoft Corporation (2024). *[MS-FSCC]: File System Control Codes*.
20. **`[R20]`** Microsoft Corporation (2024). *Resilient File System (ReFS) Overview*.
21. **`[R21]`** UEFI Forum. *Unified Extensible Firmware Interface (UEFI) Specification (GUID Partition Table)*.
22. **`[R22]`** Rodeh, O., Bacik, J., & Mason, C. (2013). *BTRFS: The Linux B-Tree Filesystem*. *ACM TOS*, 9(3). DOI: `10.1145/2501620.2501623`.
23. **`[R23]`** Sweeney, A., et al. (1996). *Scalability in the XFS File System*. *USENIX ATC 1996*.
24. **`[R24]`** TIS Committee (1995) & Hipp, D. R. / SQLite Consortium (2024). *ELF-64 Specification (v1.5)* & *SQLite File Format*.
25. **`[R25]`** Suhanov, M. (2015–2018). *Windows registry file format specification*. CC BY 4.0.
26. **`[R26]`** Metz, J. (2011–2026). *BitLocker Drive Encryption (BDE) format specification* (`libyal/libbde`).
27. **`[R27]`** Microsoft Corporation. *Active Directory Technical Specification (`[MS-ADTS]`)* & *USN and USN Rollback*.
28. **`[R28]`** NIST (2015). *Secure Hash Standard (SHS)*. NIST FIPS PUB 180-4. DOI: `10.6028/NIST.FIPS.180-4`.
