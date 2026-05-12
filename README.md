# Project Book Template

A template for research projects that publish a [MyST](https://mystmd.org/) Jupyter Book
to GitHub Pages. The data pipeline is managed by [Snakemake](https://snakemake.readthedocs.io/);
dependencies are managed by [uv](https://docs.astral.sh/uv/).

## Starting a new project from this template

The steps below use `financial-market-returns` as an example project name and
`fmr` as its short package abbreviation. Substitute your own names throughout.

### Pick your two names up front

| What | Example | Rule |
|------|---------|------|
| **Repository / folder name** | `financial-market-returns` | kebab-case, becomes the GitHub repo name and local folder |
| **Python package abbreviation** | `fmr` | short snake_case acronym, used in every `import` statement |

The abbreviation is the equivalent of `woe` in `world-of-energy`. It needs to
be a valid Python identifier — short is better.

### 1. Create the repository on GitHub

Go to this template repository and click **Use this template → Create a new repository**.
Name it `financial-market-returns` and click **Create repository**.

### 2. Clone it locally

```bash
git clone https://github.com/your-username/financial-market-returns.git
cd financial-market-returns
```

### 3. Rename the package

The template ships with a generic `pkg/` directory. Rename it to your abbreviation
and update all references in one go:

```bash
# Rename the directory
mv pkg fmr

# Update every occurrence of "pkg" in source files
# (pyproject.toml, Snakefile, pipeline scripts)
grep -rl '"pkg"' . --include="*.toml" --include="*.py" --include="Snakefile" \
  | xargs sed -i 's/"pkg"/"fmr"/g'
grep -rl 'from pkg' . --include="*.py" \
  | xargs sed -i 's/from pkg\./from fmr./g'
```

Then open `pyproject.toml` and set the project name and package:

```toml
[project]
name = "financial-market-returns"   # ← your repo name

[tool.hatch.build.targets.wheel]
packages = ["fmr"]                  # ← your abbreviation
```

And set the same name in `Snakefile`:

```python
PROJECT_NAME = "financial-market-returns"
```

### 4. Update book metadata

Edit `book/myst.yml`:

```yaml
project:
  title: Financial Market Returns
  github: your-username/financial-market-returns
```

### 5. Set up the environment

```bash
# Install uv if you haven't already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment and install all dependencies
uv sync

# Register a Jupyter kernel so jupytext can execute notebooks
# The name must match PROJECT_NAME in Snakefile
uv run python -m ipykernel install --user --name financial-market-returns
```

### 6. Verify the example pipeline works

```bash
make dry-run   # shows what would run without executing anything
make run       # runs the example pipeline end-to-end
make serve     # opens http://localhost:3000 — preview the book
```

If everything works, delete the example files and start writing your own pipeline:

```bash
rm pipeline/01_download_example.py pipeline/02_analyse_example.py
# Remove the example rule from Snakefile and example notebook from book/myst.yml
```

### 7. Enable GitHub Pages

In your new repository: **Settings → Pages → Source → GitHub Actions**.

From now on, every push to `main` builds and deploys the book automatically.
Pull requests only run the build check (no deploy).

## Project layout

```
project-root/
├── pkg/                 # Python package (rename per project)
│   └── paths.py         # Centralized path config
├── pipeline/            # Pipeline scripts
│   ├── 01_download_*    # Data acquisition
│   └── 02_analyse_*     # Analysis → notebook
├── book/                # MyST book source
│   ├── notebooks/       # Executed notebooks (Snakemake output)
│   ├── markdown/        # Static content
│   └── myst.yml         # TOC and site settings
├── data/                # Git-ignored data
├── output/images/       # Figures (tracked in git)
├── Snakefile            # Pipeline DAG
└── contribution_conventions.md   # Detailed conventions for contributors/AI
```

See [contribution_conventions.md](contribution_conventions.md) for full details on
adding pipeline stages, writing analysis scripts, and Snakemake usage.

## Common Snakemake commands

| Command | Effect |
|---------|--------|
| `snakemake -n` | Dry run — show what would execute |
| `snakemake -j4` | Run pipeline (4 parallel jobs) |
| `snakemake -R <rule>` | Force-re-run a specific rule |
| `snakemake <file>` | Build one specific output file |
| `snakemake --forceall` | Re-run everything unconditionally |
