#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import os
import time
import shutil
from pathlib import Path
import sys
import tempfile

# ---------------------- CONFIG ----------------------
# We'll select a writable notes file at startup (see get_writable_notes_file)
NOTES_FILE = None


# ---------------------- UTILITIES ----------------------
def safe_move(src: Path, dest_dir: Path):
    """
    Move src (Path) into dest_dir (Path). If a file with the same name exists,
    append a counter: name(1).ext, name(2).ext, ...
    """
    dest_dir.mkdir(parents=True, exist_ok=True)
    base = src.name
    name = src.stem
    ext = src.suffix
    dest = dest_dir / base
    counter = 1
    while dest.exists():
        dest = dest_dir / f"{name}({counter}){ext}"
        counter += 1
    shutil.move(str(src), str(dest))


def read_choice(prompt: str, valid: set = None) -> str:
    """Read and return stripped user input. If valid set provided, loop until valid."""
    while True:
        choice = input(prompt).strip()
        if valid is None or choice in valid:
            return choice
        print("Invalid choice. Please try again.")


def get_writable_notes_file():
    """
    Return a Path to a notes file that we can open for appending.
    Tries a list of candidate locations and returns the first that succeeds.
    """
    candidates = [
        Path.cwd() / "notes.txt",  # script folder (safe)
        Path.home() / "personal_productivity_notes.txt",  # home (non-hidden)
        Path.home() / "Documents" / "notes.txt",  # Documents
        Path(tempfile.gettempdir()) / "personal_productivity_notes.txt",  # system temp
    ]

    for p in candidates:
        try:
            p.parent.mkdir(parents=True, exist_ok=True)
            # Try opening for append to confirm writable
            with open(p, "a", encoding="utf-8"):
                pass
            return p
        except PermissionError:
            # can't write here, try next
            continue
        except OSError:
            # other OS errors (readonly fs, invalid path...), try next
            continue

    # Last resort: in-memory temp file name in cwd (not guaranteed but try)
    fallback = Path.cwd() / "notes.txt"
    try:
        with open(fallback, "a", encoding="utf-8"):
            pass
        return fallback
    except Exception:
        # If absolutely nothing is writable, raise so the program can handle it
        raise OSError("No writable location found for notes file.")


# ---------------------- CALCULATOR ----------------------
def calculator():
    while True:
        print("\n--- Calculator ---")
        print("1. Add")
        print("2. Subtract")
        print("3. Multiply")
        print("4. Divide")
        print("5. Back to Main Menu")

        choice = read_choice("Choose an option: ", valid={'1', '2', '3', '4', '5'})

        if choice == '5':
            return

        try:
            num1 = float(input("Enter first number: ").strip())
            num2 = float(input("Enter second number: ").strip())
        except ValueError:
            print("Please enter valid numbers!")
            continue

        if choice == '1':
            print("Result:", num1 + num2)
        elif choice == '2':
            print("Result:", num1 - num2)
        elif choice == '3':
            print("Result:", num1 * num2)
        elif choice == '4':
            if num2 != 0:
                print("Result:", num1 / num2)
            else:
                print("Error: Division by zero")


# ---------------------- NOTES APP ----------------------
def notes_app():
    global NOTES_FILE
    notes_file = NOTES_FILE
    # If we somehow don't have a notes file (shouldn't happen), try to get one now
    if notes_file is None:
        try:
            notes_file = get_writable_notes_file()
            NOTES_FILE = notes_file
            print(f"Notes will be saved to: {notes_file}")
        except OSError as e:
            print("Unable to find a writable location for notes:", e)
            print("Notes feature disabled.")
            return

    while True:
        print("\n--- Notes App ---")
        print("1. Create a Note")
        print("2. View Notes")
        print("3. Clear Notes")
        print("4. Back to Main Menu")

        choice = read_choice("Choose an option: ", valid={'1', '2', '3', '4'})

        if choice == '1':
            note = input("Please enter your note: ")
            try:
                notes_file.parent.mkdir(parents=True, exist_ok=True)
                with open(notes_file, "a", encoding="utf-8") as f:
                    f.write(note.rstrip() + "\n")
                print("Note saved!")
            except PermissionError:
                print("Permission denied writing to", notes_file)
                print("Attempting to switch to an alternate location...")
                try:
                    alt = get_writable_notes_file()
                    NOTES_FILE = alt
                    notes_file = alt
                    with open(notes_file, "a", encoding="utf-8") as f:
                        f.write(note.rstrip() + "\n")
                    print("Note saved to alternate location:", notes_file)
                except Exception as e:
                    print("Failed to save note:", e)
            except OSError as e:
                print("Error saving note:", e)

        elif choice == '2':
            if notes_file.exists():
                try:
                    with open(notes_file, "r", encoding="utf-8") as f:
                        content = f.read().strip()
                    if content:
                        print("\nYour Notes:")
                        print(content)
                    else:
                        print("No notes found!")
                except PermissionError:
                    print("Permission denied reading", notes_file)
                except OSError as e:
                    print("Error reading notes:", e)
            else:
                print("No notes found!")

        elif choice == '3':
            confirm = input("Are you sure you want to clear all notes? (y/n): ").strip().lower()
            if confirm == 'y':
                try:
                    open(notes_file, "w", encoding="utf-8").close()
                    print("All notes cleared!")
                except PermissionError:
                    print("Permission denied clearing", notes_file)
                except OSError as e:
                    print("Error clearing notes:", e)
            else:
                print("Clear cancelled.")

        elif choice == '4':
            return


# ---------------------- TIMER / STOPWATCH ----------------------
def timer_tool():
    while True:
        print("\n--- Timer / Stopwatch ---")
        print("1. Timer")
        print("2. Stopwatch")
        print("3. Back to Main Menu")

        choice = read_choice("Choose an option: ", valid={'1', '2', '3'})

        if choice == '1':
            try:
                seconds = int(input("Enter seconds (non-negative integer): ").strip())
                if seconds < 0:
                    print("Please enter a non-negative integer.")
                    continue
            except ValueError:
                print("Please enter a valid integer number of seconds.")
                continue

            try:
                for i in range(seconds, 0, -1):
                    print(f"Time left: {i} sec", end="\r", flush=True)
                    time.sleep(1)
                print("\nTimer completed!")
            except KeyboardInterrupt:
                print("\nTimer interrupted.")

        elif choice == '2':
            input("Press Enter to start stopwatch...")
            start = time.perf_counter()
            try:
                input("Press Enter to stop stopwatch...")
            except KeyboardInterrupt:
                print("\nStopwatch interrupted; stopping now.")
            end = time.perf_counter()
            elapsed = round(end - start, 2)
            print("Elapsed Time:", elapsed, "seconds")

        elif choice == '3':
            return


# ---------------------- FILE ORGANIZER ----------------------
def file_organizer():
    print("\n--- File Organizer ---")
    raw_path = input("Please enter folder path to organize: ").strip()
    if not raw_path:
        print("No path provided.")
        return

    path = Path(raw_path).expanduser()

    if not path.is_dir():
        print("Invalid folder path! Please enter correct folder location.")
        return

    file_types = {
        "Images": [".jpg", ".jpeg", ".png", ".gif", ".bmp", ".tiff", ".webp"],
        "Documents": [".pdf", ".docx", ".doc", ".txt", ".pptx", ".xlsx", ".csv"],
        "Videos": [".mp4", ".mkv", ".mov", ".avi"],
        "Archives": [".zip", ".rar", ".tar", ".gz"],
        "Audio": [".mp3", ".wav", ".flac", ".aac"],
    }

    others_dir = path / "Others"
    # Iterate only top-level entries; skip directories (we only move files)
    for item in list(path.iterdir()):
        try:
            if not item.is_file():
                continue  # skip directories and special files
            ext = item.suffix.lower()
            moved = False
            for folder_name, ext_list in file_types.items():
                if ext in ext_list:
                    dest = path / folder_name
                    safe_move(item, dest)
                    moved = True
                    break
            if not moved:
                safe_move(item, others_dir)
        except Exception as e:
            print(f"Error processing {item.name}: {e}")

    print("Folder organized successfully!")


# ---------------------- MAIN MENU ----------------------
def main():
    global NOTES_FILE
    try:
        NOTES_FILE = get_writable_notes_file()
        print(f"Notes file: {NOTES_FILE}")
    except Exception as e:
        NOTES_FILE = None
        print("Warning: Could not determine a writable notes file. Notes feature may be disabled.")
        print("Reason:", e)

    while True:
        print("\n========= Personal Productivity Suite =========")
        print("1. Calculator")
        print("2. Notes App")
        print("3. Timer / Stopwatch")
        print("4. File Organizer")
        print("5. Exit")

        choice = read_choice("Choose an option: ", valid={'1', '2', '3', '4', '5'})

        if choice == '1':
            calculator()
        elif choice == '2':
            notes_app()
        elif choice == '3':
            timer_tool()
        elif choice == '4':
            file_organizer()
        elif choice == '5':
            print("Exiting... Have a productive day!")
            return


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nInterrupted. Exiting...")
        try:
            sys.exit(0)
        except SystemExit:
            pass
