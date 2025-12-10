#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
Productivity Suite (CLI) - Object Oriented
Tools:
 - Calculator
 - NotesManager (SQLite)
 - Timer & Stopwatch
 - FileOrganizer
Main app coordinates menus and tool invocation.
"""

import os
import sys
import time
import shutil
import sqlite3
import csv
import json
from pathlib import Path
from datetime import datetime
from typing import Optional, List

# -------------------------
# Helper functions
# -------------------------
def read_choice(prompt: str, valid: Optional[set] = None) -> str:
    """Read user input and validate against a set (if provided)."""
    while True:
        try:
            s = input(prompt).strip()
        except (EOFError, KeyboardInterrupt):
            print()
            return ""
        if valid is None or s in valid or s == "":
            return s
        print("Invalid choice. Please try again.")


def press_enter(msg: str = "Press Enter to continue..."):
    try:
        input(msg)
    except (EOFError, KeyboardInterrupt):
        print()


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


def format_seconds(seconds: int) -> str:
    h, rem = divmod(seconds, 3600)
    m, s = divmod(rem, 60)
    if h:
        return f"{h:02d}:{m:02d}:{s:02d}"
    else:
        return f"{m:02d}:{s:02d}"


# -------------------------
# Calculator
# -------------------------
class Calculator:
    def __init__(self):
        self.history: List[str] = []

    def run(self):
        while True:
            print("\n--- Calculator ---")
            print("1. Add")
            print("2. Subtract")
            print("3. Multiply")
            print("4. Divide")
            print("5. View History")
            print("6. Go to Main Menu")
            choice = read_choice("Choose an option: ", valid={'1','2','3','4','5','6'})

            if choice == "" or choice == '6':
                return

            if choice not in {'1','2','3','4','5'}:
                print("Invalid choice!")
                continue

            if choice == '5':
                self.show_history()
                continue

            try:
                a = float(input("Enter first number: ").strip())
                b = float(input("Enter second number: ").strip())
            except ValueError:
                print("Invalid number input.")
                continue

            if choice == '1':
                res = a + b
                op = '+'
            elif choice == '2':
                res = a - b
                op = '-'
            elif choice == '3':
                res = a * b
                op = '*'
            elif choice == '4':
                if b == 0:
                    print("Error: Division by zero.")
                    continue
                res = a / b
                op = '/'

            out = f"{a} {op} {b} = {res}"
            print("Result:", res)
            self.history.append(out)

    def show_history(self):
        print("\n--- Calculator History ---")
        if not self.history:
            print("No history.")
        else:
            for i, line in enumerate(self.history, 1):
                print(f"{i}. {line}")
        press_enter()


# -------------------------
# Notes Manager (SQLite)
# -------------------------
class NotesManager:
    DB_FILENAME = Path.cwd() / "notes.db"

    def __init__(self):
        self.conn = None
        self._ensure_db()

    def _ensure_db(self):
        self.conn = sqlite3.connect(str(self.DB_FILENAME))
        self.conn.row_factory = sqlite3.Row
        cur = self.conn.cursor()
        cur.execute("""
            CREATE TABLE IF NOT EXISTS notes (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL,
                content TEXT NOT NULL,
                created TEXT NOT NULL
            )
        """)
        self.conn.commit()

    def run(self):
        while True:
            print("\nNOTES MANAGER:")
            print("1. View All Notes")
            print("2. Add New Note")
            print("3. Search Notes")
            print("4. Edit Note")
            print("5. Delete Note")
            print("6. Export Notes")
            print("7. Back to Main Menu")
            choice = read_choice("Enter choice: ", valid={str(i) for i in range(1,8)})
            if choice == "" or choice == '7':
                return
            if choice == '1':
                self.view_all()
            elif choice == '2':
                self.add_note()
            elif choice == '3':
                self.search_notes()
            elif choice == '4':
                self.edit_note()
            elif choice == '5':
                self.delete_note()
            elif choice == '6':
                self.export_notes()

    def add_note(self):
        print("\nADD NEW NOTE:")
        title = input("Enter note title: ").strip()
        if not title:
            print("Title cannot be empty.")
            press_enter()
            return

        print("Enter note content (finish by entering an empty line):")
        lines = []
        while True:
            try:
                line = input()
            except (EOFError, KeyboardInterrupt):
                print()
                break
            if line == "":
                break
            lines.append(line)
        content = "\n".join(lines).strip()
        if not content:
            print("Content cannot be empty.")
            press_enter()
            return

        created = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        cur = self.conn.cursor()
        cur.execute("INSERT INTO notes (title, content, created) VALUES (?, ?, ?)",
                    (title, content, created))
        self.conn.commit()
        note_id = cur.lastrowid
        print("\n✅ Note added successfully!")
        print(f"Note ID: {note_id}")
        print(f"Created: {created}")
        press_enter()

    def view_all(self):
        cur = self.conn.cursor()
        cur.execute("SELECT id, title, created FROM notes ORDER BY created DESC")
        rows = cur.fetchall()
        if not rows:
            print("\nNo notes found.")
            press_enter()
            return
        print("\nALL NOTES:")
        for r in rows:
            print(f"ID: {r['id']} | Title: {r['title']} | Created: {r['created']}")
        sub = read_choice("\nView a note by ID (enter ID) or press Enter to return: ")
        if not sub:
            return
        if not sub.isdigit():
            print("Invalid ID.")
            press_enter()
            return
        self.view_by_id(int(sub))

    def view_by_id(self, note_id: int):
        cur = self.conn.cursor()
        cur.execute("SELECT * FROM notes WHERE id = ?", (note_id,))
        row = cur.fetchone()
        if not row:
            print("Note not found.")
            press_enter()
            return
        print("\n" + "="*60)
        print(f"ID: {row['id']}")
        print(f"Title: {row['title']}")
        print(f"Created: {row['created']}")
        print("-"*60)
        print(row['content'])
        print("="*60)
        press_enter()

    def search_notes(self):
        q = input("\nEnter search keyword (title or content): ").strip()
        if not q:
            print("Empty search. Aborting.")
            press_enter()
            return
        like = f"%{q}%"
        cur = self.conn.cursor()
        cur.execute("SELECT id, title, created FROM notes WHERE title LIKE ? OR content LIKE ? ORDER BY created DESC",
                    (like, like))
        rows = cur.fetchall()
        if not rows:
            print("No matching notes.")
            press_enter()
            return
        print(f"\nResults for '{q}':")
        for r in rows:
            print(f"ID: {r['id']} | Title: {r['title']} | Created: {r['created']}")
        press_enter()

    def edit_note(self):
        nid = input("\nEnter note ID to edit: ").strip()
        if not nid.isdigit():
            print("Invalid ID.")
            press_enter()
            return
        nid = int(nid)
        cur = self.conn.cursor()
        cur.execute("SELECT * FROM notes WHERE id = ?", (nid,))
        row = cur.fetchone()
        if not row:
            print("Note not found.")
            press_enter()
            return
        print(f"Current title: {row['title']}")
        new_title = input("New title (leave blank to keep): ").strip()
        if not new_title:
            new_title = row['title']
        print("Current content:")
        print(row['content'])
        print("Enter new content (empty line to keep current):")
        lines = []
        while True:
            try:
                line = input()
            except (EOFError, KeyboardInterrupt):
                print()
                break
            if line == "":
                break
            lines.append(line)
        if lines:
            new_content = "\n".join(lines).strip()
        else:
            new_content = row['content']
        cur.execute("UPDATE notes SET title = ?, content = ? WHERE id = ?", (new_title, new_content, nid))
        self.conn.commit()
        print("Note updated.")
        press_enter()

    def delete_note(self):
        nid = input("\nEnter note ID to delete: ").strip()
        if not nid.isdigit():
            print("Invalid ID.")
            press_enter()
            return
        nid = int(nid)
        cur = self.conn.cursor()
        cur.execute("SELECT id, title FROM notes WHERE id = ?", (nid,))
        row = cur.fetchone()
        if not row:
            print("Note not found.")
            press_enter()
            return
        confirm = input(f"Delete note {nid} '{row['title']}'? (y/N): ").strip().lower()
        if confirm != 'y':
            print("Delete cancelled.")
            press_enter()
            return
        cur.execute("DELETE FROM notes WHERE id = ?", (nid,))
        self.conn.commit()
        print("Note deleted.")
        press_enter()

    def export_notes(self):
        print("\nExport options:")
        print("1. CSV")
        print("2. JSON")
        print("3. Back")
        c = read_choice("Choice: ", valid={'1','2','3'})
        if c == '' or c == '3':
            return
        cur = self.conn.cursor()
        cur.execute("SELECT * FROM notes ORDER BY created DESC")
        rows = cur.fetchall()
        if not rows:
            print("No notes to export.")
            press_enter()
            return
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        if c == '1':
            out = Path.cwd() / f"notes_export_{timestamp}.csv"
            with open(out, "w", newline="", encoding="utf-8") as f:
                writer = csv.writer(f)
                writer.writerow(["id","title","content","created"])
                for r in rows:
                    writer.writerow([r['id'], r['title'], r['content'], r['created']])
            print(f"Exported {len(rows)} notes to {out}")
            press_enter()
        else:
            out = Path.cwd() / f"notes_export_{timestamp}.json"
            data = []
            for r in rows:
                data.append({"id": r['id'], "title": r['title'], "content": r['content'], "created": r['created']})
            with open(out, "w", encoding="utf-8") as f:
                json.dump(data, f, indent=2, ensure_ascii=False)
            print(f"Exported {len(rows)} notes to {out}")
            press_enter()

    def close(self):
        if self.conn:
            self.conn.close()


# -------------------------
# Timer & Stopwatch
# -------------------------
class TimerTool:
    def run(self):
        while True:
            print("\n--- Timer / Stopwatch ---")
            print("1. Timer (countdown seconds)")
            print("2. Stopwatch")
            print("3. Back to Main Menu")
            choice = read_choice("Choose: ", valid={'1','2','3'})
            if choice == "" or choice == '3':
                return
            if choice == '1':
                self.timer_countdown()
            elif choice == '2':
                self.stopwatch()

    def timer_countdown(self):
        s = input("Enter seconds to count down (non-negative integer): ").strip()
        if not s.isdigit():
            print("Invalid input.")
            press_enter()
            return
        seconds = int(s)
        try:
            for i in range(seconds, 0, -1):
                print(f"Time left: {format_seconds(i)}", end="\r", flush=True)
                time.sleep(1)
            print("\nTimer completed!")
        except KeyboardInterrupt:
            print("\nTimer interrupted.")
        press_enter()

    def stopwatch(self):
        print("Press Enter to start stopwatch...")
        try:
            input()
        except (EOFError, KeyboardInterrupt):
            print()
            return
        start = time.perf_counter()
        print("Stopwatch started. Press Enter to stop (Ctrl+C also works).")
        try:
            input()
        except KeyboardInterrupt:
            print("\nStopping stopwatch due to interrupt.")
        end = time.perf_counter()
        elapsed = end - start
        print("Elapsed:", round(elapsed, 2), "seconds")
        press_enter()


# -------------------------
# File Organizer
# -------------------------
class FileOrganizer:
    FILE_TYPES = {
        "Images": [".jpg", ".jpeg", ".png", ".gif", ".bmp", ".tiff", ".webp"],
        "Documents": [".pdf", ".docx", ".doc", ".txt", ".pptx", ".xlsx", ".csv"],
        "Videos": [".mp4", ".mkv", ".mov", ".avi"],
        "Archives": [".zip", ".rar", ".tar", ".gz"],
        "Audio": [".mp3", ".wav", ".flac", ".aac"],
    }

    def run(self):
        print("\n--- File Organizer ---")
        raw = input("Enter folder path to organize (leave blank for current dir): ").strip()
        if not raw:
            path = Path.cwd()
        else:
            path = Path(raw).expanduser()
        if not path.exists() or not path.is_dir():
            print("Invalid folder path.")
            press_enter()
            return
        self.organize(path)
        press_enter()

    def organize(self, path: Path):
        others = path / "Others"
        try:
            for item in list(path.iterdir()):
                # skip directories we may create (like 'Images'), skip the database
                if item.is_dir():
                    continue
                if item == NotesManager.DB_FILENAME:
                    # do not move the notes database
                    continue
                ext = item.suffix.lower()
                moved = False
                for folder, ext_list in self.FILE_TYPES.items():
                    if ext in ext_list:
                        dest = path / folder
                        safe_move(item, dest)
                        moved = True
                        break
                if not moved:
                    safe_move(item, others)
            print("Folder organized successfully!")
        except PermissionError as e:
            print("Permission error while organizing:", e)
        except Exception as e:
            print("Error while organizing folder:", e)


# -------------------------
# Main application
# -------------------------
class ProductivityApp:
    def __init__(self):
        self.calc = Calculator()
        self.notes = NotesManager()
        self.timer = TimerTool()
        self.organizer = FileOrganizer()

    def run(self):
        try:
            while True:
                print("\n" + "="*42)
                print("     PERSONAL PRODUCTIVITY SUITE")
                print("="*42)
                print("\nMAIN MENU:")
                print("1. Calculator")
                print("2. Notes Manager")
                print("3. Timer & Stopwatch")
                print("4. File Organizer")
                print("5. Unit Converter")
                print("6. Backup & Restore")
                print("7. Exit")
                choice = read_choice("\nEnter your choice (1-7): ", valid={str(i) for i in range(1,6)})
                if choice == "" or choice == '5':
                    print("\nGoodbye!")
                    return
                if choice == '1':
                    self.calc.run()
                elif choice == '2':
                    self.notes.run()
                elif choice == '3':
                    self.timer.run()
                elif choice == '4':
                    self.organizer.run()
        finally:
            # cleanup if needed
            self.notes.close()


# -------------------------
# Entry point
# -------------------------
def main():
    app = ProductivityApp()
    try:
        app.run()
    except KeyboardInterrupt:
        print("\nInterrupted. Exiting...")
        try:
            sys.exit(0)
        except SystemExit:
            pass


if __name__ == "__main__":
    main()

