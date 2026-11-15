# EZR and RStudio runtime environment (R 4.6.1)

The repository for distributing, as a Zip, an environment for using [EZR](https://www.jichi.ac.jp/usr/hema/EZR/statmed.html) and RStudio in the browser (English edition). The only thing installed on the user's computer is Docker Desktop; R, RStudio and EZR are not installed.

> This environment is an unofficial distribution and is not affiliated with the developer of EZR. It was not made, checked or endorsed by the EZR developer. Please send questions about this environment to the person who gave it to you, not to the EZR developer.

## How to use

Unzip the Zip you received (`EZR_en_r461_<version>.zip`) and open the guide in it. With photos, it explains everything from starting, through opening EZR and analysing, to finishing.

| Computer | Guide |
|---|---|
| Mac (Apple Silicon or Intel) | `HOW_TO_USE_Mac.html` |
| Windows | `HOW_TO_USE_Windows.html` |

## What you can do

- Open EZR with clicks only.
- Create a "project" for each study, and keep its data (CSV, Excel) and records apart from the others.
- The records of what you do in EZR (R script, R Markdown, output, graphs) are kept automatically, and when you close EZR, a report with the figures (`markdown.html`) is made. The fingerprints (SHA-256) of the data used and the versions of R and the packages are kept too.

## How it works

EZR (the R package RcmdrPlugin.EZR) is opened inside a Docker container, in a plain R separate from RStudio, as in the official instructions. The EZR screen is drawn on a virtual display inside the container, and appears in a separate browser tab through the RStudio Server proxy.

```
Browser ─ http://localhost:<port> (RStudio Server)
Container ezr-r461en (/app = this folder)
  ├─ RStudio's R ─ the "Open EZR" window → ezr_open()
  ├─ Xvnc :1 + fluxbox ─ the EZR R (a plain R) ─ R Commander + EZR
  ├─ websockify :6080 (noVNC) ─ to the browser through the RStudio proxy /p/<key>/
  └─ projects/<name>/ ─ data/, outputs/<YYYYMMDDHHMM>_<4 digits>/
```

When RStudio opens, the `.Rprofile` extension point shows the "Open EZR" window once. When a project is chosen there, `ezr_open()` starts the virtual display, has a plain R load EZR, and shows it in a separate browser tab.

## Projects and records

A project is `projects/<name>/`; data chosen in the window is copied to `data/` (CSV or Excel, up to 15 MiB each; never overwritten). Each time EZR opens, one record folder `outputs/<YYYYMMDDHHMM>_<4 digits>/` is made.

| File | What |
|---|---|
| `script.R`, `markdown.Rmd`, `output.txt` | Copies of EZR's panes (written whenever they change) |
| `plots/` | Graphs as PNG |
| `console.log` | Output of the EZR R |
| `meta.json` | Environment, SHA-256 of the data, random state, hashes of the records |
| `markdown.html` | The report made when EZR closes (`markdown.log` if it cannot be made) |

"Stop" waits until the record hashes and the report are ready. For a session cut off before "Stop" (Docker Desktop ended, the computer turned off, or "Stop without waiting"), what was written stays, but the hashes are not finalized and the report is not made again.

## Distribution

Users install only Docker Desktop. They extract the Zip and double-click the launcher file; the image is built on their computer, and RStudio opens.

| Item | What |
|---|---|
| Making it | `ezr_dist/tools/make_zip.sh r461 en` (outside Git) makes the Zip from a pushed commit (the tools: `ezr_dist/tools/README.md`) |
| In it | The files needed to run, and the template parts used from RStudio. `Start EZR.app` and `TEMPLATE_VERSION.txt` are added when the Zip is made |
| Not in it | `tests/test_gui_worker.py`, `launcher/MacLauncher.swift`, the check and development settings (the full list is the exclusions in `make_zip.sh`) |
| Handing it over | Download (a Release in `ezr_dist_release`) or USB |
| Replacing it | `ezr_dist/tools/distribute.sh r461` (outside Git) remakes the Zips of both editions and puts them in the draft Release |

## Environment diagnostics and startup records

| Record | What |
|---|---|
| Diagnostics | For maintainers. `bash diagnose_mac.command` (Windows: `diagnose_windows.bat`) checks the environment without touching Docker and saves `outputs/environment_check_<time>_<ID>.json` (exit codes and options: `--help`) |
| `outputs/runtime_<ID>.jsonl` | A record of each startup step |
| `outputs/incident_<ID>.json` | A record kept only on a failure, a timeout or a slow step |

## Running from the repository

The repository has no `Start EZR.app`. Run the following (PowerShell on Windows) and open `http://localhost:<port>` with the printed port. Stop with `docker compose down`.

```bash
docker build --target dev -t ezr-r461en:dev .
docker compose up -d dev
docker compose port dev 8787
```

On an x64 computer, first set `PLATFORM` to `linux/amd64` in the same window (see [`docs/DOCKER.md`](docs/DOCKER.md) §1). From the console, `source("R/project_ezr.R")` then `ezr_open("studyA")`. Maintainer commands: `make help`.

## Tests

`make test` (also in `check` and `verify`) runs all of `tests/testthat`. The commands that run only the EZR tests (`ezr-test`, `ezr-test-amd64` on amd64, and `ezr-test-launcher` for the Mac start flow on the Mac side) appear in `make help`.

## Code notes

Comments are only in `R/project_ezr.R` and `tests/testthat/test-project_ezr.R`, one line per function and per test.

| File | Role |
|---|---|
| `start_mac.command`, `start_windows.bat` | The launcher files. They build the image, start the container and show the window |
| `launcher/MacLauncher.swift` | The source of the Mac window (`Start EZR.app`), made into the app when the Zip is made |
| `launcher/windows_gui.ps1` | The Windows window |
| `launcher/start_messages.txt` | The list of launcher and window texts (different in each edition) |
| `diagnose_mac.command`, `diagnose_windows.bat` | Environment diagnostics for maintainers |
| `ezr_start.R` | Shows the "Open EZR" window |
| `R/project_ezr.R` | The functions that open EZR and keep the records |
| The template's extension points (`docker/project_*`, `project.mk`, the end of `.Rprofile` and so on) | The packages, settings and commands EZR needs |
| `tests/testthat/test-project_ezr.R` | Tests of `R/project_ezr.R` and the launcher files |
| `tests/test_gui_worker.py` | Tests of the Mac start flow |

## License and citation

- The code is under [MIT](https://opensource.org/license/mit) (`LICENSE`).
- The guides for users (`HOW_TO_USE_*.html` and the photos in them) and the demo videos are under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/deed.en) (`LICENSE-docs`). The rights to the software screens shown in the photos remain with their respective owners.
- What is downloaded when the environment is prepared (R, the R packages including EZR, RStudio Server and so on) is not included in what is distributed. It follows the terms of its own authors.
- To cite this environment, use the information in `CITATION.cff`. The contact for the author is also in `CITATION.cff`.

## Use of AI

The code and documentation were written with an AI coding assistant ([Claude Opus 5.5](https://www.anthropic.com/claude/opus) by Anthropic, used through [Claude Code](https://code.claude.com/docs/en/overview)) as a tool.

## If something goes wrong

| If this happens | Do this |
|---|---|
| The EZR tab does not open and RStudio shows "Popup Blocked" | The browser blocked the new tab. Press "Try Again" in the window, and the EZR tab opens. |
| It says "Docker Desktop was not found" | Install Docker Desktop, following step 1 of the guide. |
| It says "Docker Desktop did not become ready" | Open the Docker Desktop window, wait until "Engine running" appears at the bottom left, and then start again (step 3 of the guide). |
| It says "Could not prepare the environment" or "Could not rebuild the package list" | Check that you are connected to the internet and that you have free disk space, and then start again (step 3 of the guide). |
| RStudio does not open in the browser | Press "Open browser" in the "EZR runtime" window. If it still does not open, press "Copy address", then right-click the address bar of your browser and choose "Paste and go" (the name differs by browser and may include the address, such as "Paste and Go" in Safari or "Paste and go to localhost:8787" in Edge). |
| The EZR screen froze, or a window cannot be found | Show the window with step 4 (3) of the guide and choose the same project. If that does not fix it, stop with step 7, and start again with step 3 (the records are kept). |
| After you chose another project, the bottom left of RStudio (Console) says in red that the previous EZR cannot be closed | The previous EZR has a window open, such as a file chooser, or is in the middle of a computation. Close that window in the EZR tab (or wait for the computation to end), and the previous EZR closes; then do (3) of step 4 of the guide again (the records are kept). |
| The "Open EZR" window says in red "The report of the previous session (markdown.html) is being made. …" | Wait a little until the report of the previous session is made, then press "Open EZR" again. |
| After you chose another project, the bottom left of RStudio (Console) says "The report of the previous session is being made. "…" opens when it is done…", and it does not open at once | It opens by itself when the report of the previous session is made. Just wait. |
| You tried to close the "EZR runtime" window during preparation or stopping, and it says "Please wait for preparation or stopping to finish" | Press "Back" on a Mac, or "OK" on Windows, and wait until it finishes. |
| "Stop" does not stop it | As in step 7 of the guide, wait a little and press "Stop" again. |
| It says "Stopping failed." | Check that Docker Desktop is running, then press "Stop" again. |
| You pressed "Restart R Commander" in EZR's "Tools" → "Options..." | Press OK for "Exit?", and "No" for "Save script file?" and similar questions. When EZR opens again with the new settings, choose the data again with the "Data set:" button (the data and the records go on). |
| On a Mac, from the second time on, a warning still says `start_mac.command` cannot be opened | Tell the person who gave you this environment exactly what it said (the first time, start with A or B in step 3 of the guide). |
| On a Mac, it says "This app cannot start on its own." | Start from `start_mac.command`, not from `Start EZR.app` (step 3 of the guide). It also appears if only the app was copied somewhere else. |
| On Windows, it says "Put this file inside the analysis folder, the folder that contains docker-compose.yml, and use it from there." | You have opened the Zip without extracting it, or you have moved only `start_windows.bat` to another place. Extract the Zip as in step 2 of the guide, and double-click `start_windows.bat` in the `EZR_en_r461` folder. |
| On Windows, it says "Windows protected your PC" | Press "More info", then press the "Run anyway" button that appears (step 3 of the guide). |
| On Windows, it says "This folder is already running." | Go back to the open "EZR runtime" window (it is on the taskbar). |
| The RStudio tab shows "R Session Disconnected" | RStudio connects to only one browser tab. Opening RStudio in another tab (such as with "Open browser") turns the earlier tab into this. Close this tab. If the "Open EZR" window was shown, it appears in the remaining tab. If it does not, press "Source" for `ezr_start.R`, as in step 4 (3) of the guide. If the Console does not respond, first press the red stop button at the top right of the Console. |
| RStudio shows "R is taking longer to start than usual" and does not go on | Press "Reload". |
| After reloading the RStudio page while the "Open EZR" window was shown, the window does not appear and the Console does not respond | Press the red stop button at the top right of the Console, then press "Source" for `ezr_start.R`. The window appears again. |
| The browser offers to translate the page | Do not translate it; close the offer with ×. |
| On a narrow screen, RStudio's "Source" or [Render] label is not shown | The button is shown as an arrow icon only. Press the arrow icon, not the checkbox next to it ("Source on Save" or "Render on Save"). |
| You want to change the packages (dependencies) | Leave the current `EZR_en_r461` folder as it is; extract the Zip you received again to make a new `EZR_en_r461` folder, and make the change there. The analyses in the current folder can go on with the same packages as before. |
| It says "Launch or stop failed (exit code …)" | Tell the person who gave you this environment exactly what the box at the bottom of the "EZR runtime" window shows. |
