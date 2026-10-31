# Data

Raw input files go in `data/raw/`. That folder is git-ignored: NPPES files are public, but far too large to commit, and tests use small fixtures instead.

## Getting an NPPES weekly file
1. Open CMS's NPPES downloads page: https://download.cms.gov/nppes/NPI_Files.html
2. Download a **weekly** update zip. The full monthly file is many gigabytes; the weekly one is much smaller.
3. Extract it somewhere outside the repo. Then copy only the main provider CSV, usually named `npidata_pfile_<start>-<end>.csv`, into `data/raw/`. The zip also contains other files (a header-only file, other names, practice locations, endpoints) that week 1 doesn't use.

File names and layout are as of when this was written. Check the CMS page if they've changed.
