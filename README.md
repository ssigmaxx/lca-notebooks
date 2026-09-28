# LCA Notebooks

Life cycle assessment (LCA) models built with [Brightway](https://docs.brightway.dev/) and [Activity Browser](https://github.com/LCA-ActivityBrowser/activity-browser), using the Swiss BAFU (UVEK LCI Data) database as background data.

- **Brightway** is the calculation engine: a set of Python libraries (`bw2data`, `bw2calc`, `bw2io`) that store inventory data and solve the LCA.
- **Activity Browser** is a desktop app built on top of Brightway. It reads and writes the same projects, so anything created in a notebook also shows up there, and the other way round.

We use notebooks to import data and to build models whose assumptions need to be documented and reproducible, and Activity Browser to build simple processes by hand, inspect models, and look at results (contributions, Sankey diagrams, Monte Carlo).

## Contents

| Notebook | What it does | Brightway project |
|---|---|---|
| `bafu_import.ipynb` | Imports BAFU into a project, nothing else. Start here if you want to build processes yourself in Activity Browser | Set in the notebook |
| `pet_lca.ipynb` | 0.5 L PET bottle, cradle-to-grave, linked to BAFU | `PET Bottle LCA` |
| `glass_bottle_lca.ipynb` | 0.5 L glass bottle, screening model with literature emission factors (no background database) | `Glass Bottle LCA` |
| `aluminium_can_lca.ipynb` | 0.5 L aluminium can, linked to BAFU | `Aluminium Can LCA` |
| `steel_can_lca.ipynb` | 0.5 L steel can, linked to BAFU | `Steel Can LCA` |
| `cardboard_carton_lca.ipynb` | 0.5 L paperboard beverage carton, linked to BAFU | `Cardboard Carton LCA` |
| `concrete_paving_slab_lca.ipynb` | 1 m² concrete paving slab, linked to BAFU | `Concrete Paving Slab LCA` |
| `polyester_tshirt_lca.ipynb` | Polyester t-shirt, hybrid of BAFU links and literature factors | `Polyester T-Shirt LCA` |

The BAFU data files are **not** in this repository. Get the extracted ecoSpold1 folder from the team.

---

## Setup on Windows

This takes about 30–45 minutes the first time, most of it waiting for downloads.

### What you'll install

| Tool | Why |
|---|---|
| Miniforge (conda) | Manages Python and all the LCA packages in an isolated environment |
| `ab` conda environment | Contains Activity Browser, Brightway, and the packages the notebooks need |
| Git for Windows | To download (clone) this repository and keep it up to date |
| VS Code with the Python and Jupyter extensions | To open and run the notebooks |

### Step 1: Install Miniforge

1. Download **Miniforge3-Windows-x86_64.exe** from <https://github.com/conda-forge/miniforge> (see the *Download* section of the README, or the *Releases* page).
2. Run the installer and choose:
   - **Just Me** (recommended, no admin rights needed)
   - The default install folder, e.g. `C:\Users\<you>\miniforge3`. Avoid folders with spaces or special characters in the path.
   - Leave **"Add Miniforge3 to my PATH"** unticked, as the installer recommends.
3. Open **Miniforge Prompt** from the Start menu.

Use **Miniforge Prompt** for every command in this guide. The normal Command Prompt and PowerShell don't know about conda unless you configure them, and you'll get `'conda' is not recognized`.

If you already have Anaconda or Miniconda installed, you can skip this step and use **Anaconda Prompt** instead.

### Step 2: Create the `ab` environment

All team members should use **the same Activity Browser and Brightway versions**. The notebooks are written for legacy Brightway2 (`bw2calc` 1.x, which provides `MonteCarloLCA`), and a project created with one Brightway version may not open in another.

**2a. Find the versions the team uses.** On a computer where the notebooks already work (for example a Mac), open a terminal and run:

```bash
conda activate ab
conda list | grep -E "activity-browser|bw2|brightway|scipy|^python "
```

On Windows, the equivalent is:

```bash
conda activate ab
conda list | findstr /R "activity-browser bw2 brightway scipy python"
```

Write down the version numbers of `activity-browser` and `python`.

**2b. Create the environment on Windows.** In Miniforge Prompt, replace `<version>` with the Activity Browser version from 2a:

```bash
conda create -n ab -c conda-forge activity-browser=<version> python=3.11
```

Answer `y` when asked to proceed. This can take several minutes.

If you leave out `=<version>`, conda installs the newest Activity Browser, which may come with a newer Brightway that the notebooks and existing projects don't work with.

**2c. Add the extra packages the notebooks need:**

```bash
conda activate ab
conda install -c conda-forge "scipy<1.12" ipykernel
```

- `scipy<1.12`: newer SciPy versions break Brightway2's Monte Carlo with `ValueError: 'scipy.sparse.linalg.cgs' called with invalid atol='legacy'`.
- `ipykernel`: lets VS Code (and Jupyter) run notebooks in this environment.

**2d. Register the environment as a Jupyter kernel**, so VS Code lists it by name:

```bash
python -m ipykernel install --user --name ab --display-name "ab (Brightway)"
```

**2e. Check that Activity Browser starts:**

```bash
activity-browser
```

The Activity Browser window should open. Close it for now.

To start Activity Browser any time later: open Miniforge Prompt, then

```bash
conda activate ab
activity-browser
```

### Step 3: Install Git and clone this repository

1. Download and install **Git for Windows** from <https://git-scm.com/download/win>. The default options are fine.
2. In Miniforge Prompt:

   ```bash
   cd %USERPROFILE%\Documents
   git clone https://github.com/ssigmaxx/lca-notebooks.git
   cd lca-notebooks
   ```

   If the repository is private, Git opens a browser window to sign in to GitHub. Use the account that has access to the repository.

3. To get the latest changes later:

   ```bash
   cd %USERPROFILE%\Documents\lca-notebooks
   git checkout main
   git pull
   ```

### Step 4: Set up VS Code

1. Install **VS Code** from <https://code.visualstudio.com/>.
2. Open the **Extensions** panel (`Ctrl+Shift+X`) and install:
   - **Python** (Microsoft)
   - **Jupyter** (Microsoft)
3. **File → Open Folder…** and open `Documents\lca-notebooks`.
4. Open any notebook, e.g. `bafu_import.ipynb`.
5. Click the kernel name at the **top right** of the notebook, then **Select Another Kernel… → Jupyter Kernel…** (or **Python Environments…**) and choose **ab (Brightway)**. Its path should end in `\miniforge3\envs\ab\python.exe`.

If `ab` doesn't appear, make sure you did Step 2d, then run **Developer: Reload Window** from the command palette (`Ctrl+Shift+P`).

**Check that the right environment is used.** Add a temporary cell at the top of the notebook and run it:

```python
import sys; print(sys.executable)
import bw2data, bw2io, bw2calc
print(bw2data.__version__, bw2io.__version__, bw2calc.__version__)
```

- The path should contain `envs\ab`.
- The second line should print version numbers without an error. If you see `ModuleNotFoundError: No module named 'bw2data'`, the wrong kernel is selected: go back to step 5.

Delete the temporary cell afterwards.

### Step 5: Get the BAFU files

1. Copy the extracted BAFU **ecoSpold1 folder** (about 12,000 XML files) to the laptop, for example to `C:\Users\<you>\ecoSpold files`.
2. Open `bafu_import.ipynb` and set the path in the settings cell. Any of these forms work:

   ```python
   ECOSPOLD_PATH = "~/ecoSpold files"                   # "~" means C:\Users\<you>
   ECOSPOLD_PATH = r"C:\Users\<you>\ecoSpold files"     # note the r before the quote
   ECOSPOLD_PATH = "C:/Users/<you>/ecoSpold files"      # forward slashes also work
   ```

   Don't write `"C:\Users\..."` without the `r` in front. Python reads `\U` as a special character code and the path breaks with a `SyntaxError` or `FileNotFoundError`.

The model notebooks (`pet_lca.ipynb` and the others) have their own `ECOSPOLD_PATH` line with a Mac path (`/Users/abhi/...`). Change it the same way before running them.

### Step 6: Import BAFU and start working

1. In `bafu_import.ipynb`, set `PROJECT_NAME` to the project you want (e.g. `"Synthetic Shoes LCA"`).
2. Run all cells: **Restart** then **Run All** in the notebook toolbar. The import takes a few minutes on Windows.
3. Check the last cell: the test LCA on Swiss lorry transport should give roughly **0.1–0.3 kg CO2-eq per tkm**. An error or a score of 0 means the import went wrong.
4. Start Activity Browser (Step 2e), choose the same project from the **Project** dropdown, and BAFU is there.
5. Click **New database…**, create a database for your own processes, and build in that. Don't edit BAFU itself.

---

## Where Brightway stores projects

Brightway projects live on each computer, not in this repository. On Windows the default location is:

```
C:\Users\<you>\AppData\Local\pylca\Brightway3
```

Projects created on another computer **don't appear automatically**. Two ways to get them onto a new computer:

**Option A: rebuild (recommended).** Run `bafu_import.ipynb`, and the model notebooks if needed, on the new computer. This is the cleanest way and also checks that the notebooks still work.

**Option B: copy a project with a backup file.** Both computers must have the **same Brightway version**.

On the computer that has the project:

```python
import bw2io
bw2io.backup_project_directory("Synthetic Shoes LCA")   # prints where the .tar.gz file was saved
```

Copy that `.tar.gz` file to the other computer, then:

```python
import bw2io
bw2io.restore_project_directory(r"C:\path\to\the-backup-file.tar.gz")
```

For processes built by hand in Activity Browser, you can also right-click the database, export it to Excel, and import that Excel file on the other computer.

---

## Rules for working with notebooks and Activity Browser together

- **Each database is edited in one place only.** The model notebooks delete and rewrite their own foreground database every time they run, so any changes made to that database in Activity Browser are lost. Build hand-made processes in a separate database that no notebook writes to.
- **Import BAFU once per project.** Re-importing BAFU into a project where processes already link to it can break those links. For a new project, either run `bafu_import.ipynb` or use Activity Browser's **Duplicate** button on a project that already has BAFU (faster).
- **Don't have a notebook and Activity Browser writing to the same project at the same time.** After a notebook changes a project, switch projects in Activity Browser and back (or restart it) so it picks up the changes.
- **Use one project per study**, so models can't interfere with each other.

---

## Troubleshooting (Windows)

| Problem | Cause and fix |
|---|---|
| `'conda' is not recognized as an internal or external command` | You're in Command Prompt or PowerShell. Use **Miniforge Prompt** |
| `conda create` takes very long or looks stuck | Normal the first time (it's resolving dependencies). If it's stuck for more than ~20 minutes, cancel with `Ctrl+C`, run `conda update -n base conda`, and try again |
| Activity Browser doesn't start, or shows a Qt / DLL error | Make sure you ran `conda activate ab` first. If it still fails, remove and recreate the environment: `conda env remove -n ab`, then Step 2 again |
| VS Code shows red wavy lines under `import bw2data`, or `ModuleNotFoundError` when running | Wrong kernel. Select **ab (Brightway)** at the top right of the notebook (Step 4) |
| `ab` isn't in VS Code's kernel list | Run Step 2d, then **Developer: Reload Window** |
| `AssertionError: Folder not found` in `bafu_import.ipynb` | `ECOSPOLD_PATH` is wrong. Check the folder name and use `r"..."` or forward slashes (Step 5) |
| `AssertionError: 'BAFU' already exists` | BAFU is already in this project, nothing to do. Use a new `PROJECT_NAME`, or delete it first with `del databases["BAFU"]` if you really want to re-import |
| The BAFU import is very slow | Windows Defender scans every file as it's read. Temporarily exclude the `ecoSpold files` folder in *Windows Security → Virus & threat protection → Exclusions*, or just wait |
| `ValueError: 'scipy.sparse.linalg.cgs' called with invalid atol='legacy'` during Monte Carlo | SciPy is too new. Run `conda install -c conda-forge "scipy<1.12"` in the `ab` environment and restart the kernel |
| A project from another computer doesn't show up | Projects are stored locally. See [Where Brightway stores projects](#where-brightway-stores-projects) |
| Monte Carlo gives a standard deviation of 0 | None of the sampled exchanges has uncertainty data. Add uncertainty to your own exchanges (Activity Browser: right-click an exchange → modify uncertainty) |
