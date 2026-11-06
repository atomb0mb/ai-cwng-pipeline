# Data

Raw input files go in `data/raw/`. That folder is git-ignored: NPPES files are public, but far too large to commit, and tests use small fixtures instead.

## Getting the NPPES monthly file
1. Open CMS's NPPES downloads page: https://download.cms.gov/nppes/NPI_Files.html
2. Download the **monthly** full replacement zip (*NPPES Data Dissemination*). It's large, so allow for about 1 GB to download and about 12 GB once extracted.
3. Extract everything into `data/raw/`. It's all git-ignored. Week 1 uses only the main provider file, `npidata_pfile_<start>-<end>.csv` (about 11 GB, 330 columns).

The zip also contains:
| File | What it is |
|---|---|
| `npidata_pfile_..._fileheader.csv` | **Header row only.** It's tiny, and it looks empty if you open it by mistake. |
| `othername_pfile_*`, `pl_pfile_*`, `endpoint_pfile_*` | Other provider names, extra practice locations, and electronic endpoints. Not used in week 1. |
| `NPPES_Data_Dissemination_Readme_*.pdf`, `..._CodeValues.pdf` | CMS's description of the file layout and code values. Worth skimming. |

**Why the monthly file and not a weekly one?** Both have the same layout. The monthly file is the complete registry, so it's always a valid starting point. The size doesn't matter for week 1, because `--limit` reads only the first N rows.

File names and layout are as of November 2026. Check the CMS page if they've changed.
