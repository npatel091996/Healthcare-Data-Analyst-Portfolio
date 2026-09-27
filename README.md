# Healthcare-Data-Analyst-Portfolio

# Healthcare Data Analyst Portfolio
 
**Niket Patel** · Healthcare Informatics Professional
 
**Live site:** https://npatel091996.github.io/Healthcare-Data-Analyst-Portfolio/
**LinkedIn:** https://www.linkedin.com/in/niketpatel1996/
 
I didn't start in data. I started in a lab coat. After years in food science and microbiology research, I moved into healthcare analytics through a Master's in Health Informatics at Rutgers. Since then I've worked in hands-on roles at Evernorth Health Services, Atlantic Health System, and AB InBev, building SQL pipelines, Power BI dashboards, and Alteryx workflows.
 
This repository holds my portfolio website and the projects. Each project has its code, a written case study, and the steps to reproduce the analysis.
 
---
 
## Projects
 
| Project | Summary | Key tools | Case study |
|---|---|---|---|
| Clinical Trial Data Analysis – SAS `[RESTRICTED data]` | Tested whether Drug A lowers fasting glucose across Placebo, Low Dose, and High Dose groups. One-way ANOVA on a random sample of 1,000 patients found no significant drug effect (F(2, 997) = 2.17, p = 0.1142). | SAS (PROC GLM, SURVEYSELECT, UNIVARIATE, CORR), ANOVA, Levene's test | [Read the case study](projects/clinical-trial-sas/README.md) |
 
`[RESTRICTED data]` means the original dataset can't be shared publicly. A script in the project folder creates stand-in data with the same structure instead (see [Data sources](#data-sources)).
 
---
 
## Repository structure
 
```text
Healthcare-Data-Analyst-Portfolio/
├── index.html                          # Portfolio website (GitHub Pages)
├── README.md                           # This file
├── assets/
│   └── resume.pdf                      # Resume (download button on the site)
└── projects/
    └── clinical-trial-sas/
        ├── README.md                   # Case study
        ├── requirements.txt            # Python libraries for the data script
        ├── code/
        │   └── [TODO: SAS program file name].sas
        └── data/
            ├── .gitignore              # Keeps the restricted original files out of Git
            ├── generate_synthetic_data.py
            └── synthetic/
                ├── Project2_Data-1.csv # Stand-in: PatientID, Age, State, Lenght_of_Stay, Total_Charge
                └── Project2_Data-2.csv # Stand-in: PatientID, Group, Test_Score
```
 
---
 
## Deploying the site with GitHub Pages
 
1. **Create the repository.** On GitHub, create a **public** repository named `Healthcare-Data-Analyst-Portfolio`.
2. **Add the files.** Upload the contents of this folder so that `index.html` sits at the top level of the repository, not inside a subfolder.
3. **Add your resume.** Save it as `assets/resume.pdf`. The site's "Download Resume" button links to that exact path.
4. **Turn on Pages.** Go to **Settings → Pages**. Under **Build and deployment**, set **Source** to **Deploy from a branch**, choose branch **main** and folder **/(root)**, then click **Save**.
5. **Wait for the build.** After a minute or two, the site is live at `https://[TODO:GITHUB_USERNAME].github.io/Healthcare-Data-Analyst-Portfolio/`. Build progress shows on the repository's **Actions** tab.
6. **Update links.** Replace every `[TODO:GITHUB_USERNAME]` in `index.html` and this README with your GitHub username. Commit the change, and Pages rebuilds automatically.
 
To preview the site before publishing, open `index.html` in any web browser. No build step is needed; Tailwind CSS loads from a CDN.
 
---
 
## Running a project locally
 
These steps use the Clinical Trial project as the example.
 
**Prerequisites:** Git, Python 3.9 or later, and SAS (SAS 9.4 or SAS OnDemand for Academics) to run the analysis.
 
**1. Clone the repository**
 
```bash
git clone https://github.com/[TODO:GITHUB_USERNAME]/Healthcare-Data-Analyst-Portfolio.git
cd Healthcare-Data-Analyst-Portfolio/projects/clinical-trial-sas
```
 
**2. Create and activate a virtual environment**
 
```bash
python -m venv .venv
# macOS / Linux
source .venv/bin/activate
# Windows
.venv\Scripts\activate
```
 
**3. Install requirements**
 
```bash
pip install -r requirements.txt
```
 
**4. Get the data**
 
The original data is restricted, so generate the stand-in files:
 
```bash
python data/generate_synthetic_data.py --validate
```
 
This writes `data/synthetic/Project2_Data-1.csv` and `data/synthetic/Project2_Data-2.csv`. The fixed random seed produces the same files every run. `--validate` replays the cleaning steps and prints row counts next to the original project's counts. (The generated files are also committed to the repo, so this step is optional.)
 
**5. Run the code**
 
1. Open the SAS program in `code/`.
2. Update the `libname` and `filename` paths at the top so they point to your SAS library folder and the two CSVs in `data/synthetic/`.
3. Run the program. It reads both files, removes missing and invalid records, merges them on Patient ID, and runs the descriptive statistics, assumption checks, and ANOVA.
 
> Results on the stand-in data will differ from the published results. The stand-in simulates no drug effect, so it's for running the workflow, not reproducing the findings.
 
---
 
## Data sources
 
| Project | Data | Availability |
|---|---|---|
| Clinical Trial Data Analysis – SAS | Two patient-level CSV files (demographics, length of stay, and charges; treatment group and glucose test score). Primarily synthetic dataset provided by the course instructor. | Restricted. Original files are not redistributed. Stand-in data can be generated with `data/generate_synthetic_data.py`. |
 
**No PHI.** Every dataset in this portfolio is either public or synthetic, and none contains protected health information (PHI). Details on where each dataset came from and how it was prepared are in each project's case study.


