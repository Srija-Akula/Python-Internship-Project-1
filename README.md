# Productivity Suite

**A lightweight, modular Python productivity suite** integrating a calculator, notes manager, timer/stopwatch, and file organizer — built with object-oriented design, file handling, and clear CLI interfaces.

---

## Table of Contents

* [Project Overview](#project-overview)
* [Features](#features)
* [Tech Stack](#tech-stack)
* [Repository Structure](#repository-structure)
* [Installation](#installation)
* [Configuration](#configuration)
* [Usage](#usage)

  * [Run the app](#run-the-app)
  * [Calculator](#calculator)
  * [Notes Manager](#notes-manager)
  * [Timer & Stopwatch](#timer--stopwatch)
  * [File Organizer](#file-organizer)
* [Examples](#examples)
* [Testing](#testing)
* [Development & Contribution](#development--contribution)
* [Packaging & Distribution](#packaging--distribution)
* [License](#license)
* [Contact](#contact)

---

## Project Overview

This project is a beginner-to-intermediate level Python application that bundles several small productivity tools into a single CLI application. It demonstrates clean object-oriented design, separation of concerns, filesystem-based persistence, and a simple, user-friendly command-line interface.

The suite is intentionally modular so each tool can be tested independently and reused in other projects.

---

## Features

* **Calculator** — Basic arithmetic (add, subtract, multiply, divide), with validation and error handling.
* **Notes Manager** — Create, read, update, delete notes stored as text files (or JSON) with basic search.
* **Timer & Stopwatch** — Start/stop/pause timers and a simple countdown timer.
* **File Organizer** — Organize files in a directory by type into subfolders (Images, Documents, Videos, etc.).
* Configurable via a `config.json` (optional)
* Unit tests for core modules

---

## Tech Stack

* Python 3.10+
* Standard library modules: `os`, `shutil`, `time`, `datetime`, `json`, `argparse`, `unittest`
* Optional: `rich` for a nicer CLI (recommended but optional)

---

## Repository Structure

```
your_project/
├── README.md
├── main.py
├── requirements.txt
├── modules/
│   ├── __init__.py
│   ├── calculator.py
│   ├── notes.py
│   ├── timer.py
│   └── file_organizer.py
├── data/
│   ├── notes/
│   └── backups/
├── docs/
│   └── user_guide.md
├── tests/
│   ├── test_calculator.py
│   ├── test_notes.py
│   └── test_file_organizer.py
└── examples/
    └── example_usage.py
```

---

## Installation

1. Clone the repository:

```bash
git clone https://github.com/<your-username>/productivity-suite.git
cd productivity-suite
```

2. (Optional) Create and activate a virtual environment:

```bash
python -m venv venv
# macOS / Linux
source venv/bin/activate
# Windows (PowerShell)
venv\Scripts\Activate.ps1
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

`requirements.txt` example (minimal):

```
# requirements.txt
rich>=10.0.0  # optional for nicer CLI output
```

---

## Configuration

The suite can optionally read a `config.json` in the project root to customize locations and behavior. Example `config.json`:

```json
{
  "data_dir": "data",
  "notes_dir": "data/notes",
  "backup_dir": "data/backups",
  "file_types": {
    "Images": [".jpg", ".jpeg", ".png", ".gif"],
    "Documents": [".pdf", ".docx", ".txt"],
    "Videos": [".mp4", ".mkv"]
  }
}
```

If no `config.json` is provided, the application falls back to sensible defaults.

---

## Usage

### Run the app

```bash
python main.py
```

This will display a main menu letting you choose the tool you want.

### Calculator

* Navigate to `Calculator` from the main menu.
* Choose operation and provide numbers.
* Example API (if called programmatically):

```python
from modules.calculator import Calculator
calc = Calculator()
result = calc.add(3, 5)  # 8
```

### Notes Manager

* Create notes: stores notes as timestamped files or JSON entries.
* List notes, view, edit, delete, search by keyword.

Programmatic example:

```python
from modules.notes import NotesManager
nm = NotesManager(notes_dir='data/notes')
note_id = nm.create("Meeting notes", "Discuss project timeline")
nm.list()
```

### Timer & Stopwatch

* Use the CLI options to start/pause/stop a stopwatch or run a countdown timer.
* Example API:

```python
from modules.timer import Timer
t = Timer()
t.start()
# do something
t.stop()
print(t.elapsed)
```

### File Organizer

* Provide a folder path; the organizer will move files into categorized subfolders.

CLI example:

```bash
# From within the repository:
python main.py --organize /path/to/folder
```

Programmatic example:

```python
from modules.file_organizer import FileOrganizer
fo = FileOrganizer()
fo.organize('/home/user/Downloads')
```

---

## Examples

See `examples/example_usage.py` for scripted usage of multiple modules together.

---

## Testing

Run unit tests with `unittest`:

```bash
python -m unittest discover -s tests
```

Write tests for each module (examples included in `tests/`). Aim for small, focused tests covering boundary conditions and error handling.

---

## Development & Contribution

Contributions are welcome! Suggested workflow:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Implement your changes and add tests
4. Run tests locally
5. Open a pull request with a clear description

Please follow these guidelines:

* Keep functions small and focused
* Use object-oriented patterns and single-responsibility principle
* Add or update tests for new behavior
* Follow PEP 8 formatting

---

## Packaging & Distribution

If you want to publish this as a package, add a `setup.py` or `pyproject.toml`. Keep the CLI entry point in `main.py` or via `console_scripts` in `setup.cfg` / `pyproject.toml`.

---

## License

This project is released under the MIT License. See `LICENSE` for details.

---

## Contact

Maintainer: `<Srija-Akula>` ([srijaakula200@gmail.com](mailto:srijaakula200@gmail.com))

If you found this project helpful or want feature suggestions, open an issue or drop a PR.

---

*Happy coding!* 🚀
