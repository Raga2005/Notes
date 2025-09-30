# Notes
#User friendly notes making
import tkinter as tk
from tkinter import messagebox, simpledialog, scrolledtext
import sqlite3

# ------------------ DATABASE ------------------
conn = sqlite3.connect("notes.db")
cursor = conn.cursor()
cursor.execute('''
CREATE TABLE IF NOT EXISTS notes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    content TEXT NOT NULL
)
''')
conn.commit()

# ------------------ FUNCTIONS ------------------
def add_note():
    title = simpledialog.askstring("Note Title", "Enter note title:")
    if title:
        content = simpledialog.askstring("Note Content", "Enter note content:")
        if content:
            cursor.execute("INSERT INTO notes (title, content) VALUES (?, ?)", (title, content))
            conn.commit()
            messagebox.showinfo("Success", "Note added successfully!")
            load_notes()
        else:
            messagebox.showwarning("Empty Content", "Note content cannot be empty!")
    else:
        messagebox.showwarning("Empty Title", "Note title cannot be empty!")

def load_notes():
    listbox.delete(0, tk.END)
    cursor.execute("SELECT id, title FROM notes")
    for note in cursor.fetchall():
        listbox.insert(tk.END, f"{note[0]}: {note[1]}")

def view_note():
    try:
        selected = listbox.get(listbox.curselection())
        note_id = int(selected.split(":")[0])
        cursor.execute("SELECT title, content FROM notes WHERE id=?", (note_id,))
        note = cursor.fetchone()
        if note:
            top = tk.Toplevel(root)
            top.title(note[0])
            text_area = scrolledtext.ScrolledText(top, width=60, height=20)
            text_area.pack()
            text_area.insert(tk.END, note[1])
            text_area.config(state=tk.DISABLED)
    except:
        messagebox.showwarning("Select Note", "Please select a note to view!")

def delete_note():
    try:
        selected = listbox.get(listbox.curselection())
        note_id = int(selected.split(":")[0])
        confirm = messagebox.askyesno("Delete Note", "Are you sure you want to delete this note?")
        if confirm:
            cursor.execute("DELETE FROM notes WHERE id=?", (note_id,))
            conn.commit()
            messagebox.showinfo("Deleted", "Note deleted successfully!")
            load_notes()
    except:
        messagebox.showwarning("Select Note", "Please select a note to delete!")

def update_note():
    try:
        selected = listbox.get(listbox.curselection())
        note_id = int(selected.split(":")[0])
        cursor.execute("SELECT title, content FROM notes WHERE id=?", (note_id,))
        note = cursor.fetchone()
        if note:
            new_title = simpledialog.askstring("Update Title", "Enter new title:", initialvalue=note[0])
            new_content = simpledialog.askstring("Update Content", "Enter new content:", initialvalue=note[1])
            if new_title and new_content:
                cursor.execute("UPDATE notes SET title=?, content=? WHERE id=?", (new_title, new_content, note_id))
                conn.commit()
                messagebox.showinfo("Updated", "Note updated successfully!")
                load_notes()
    except:
        messagebox.showwarning("Select Note", "Please select a note to update!")

# ------------------ GUI ------------------
root = tk.Tk()
root.title("📓 My Notes App")
root.geometry("500x400")

listbox = tk.Listbox(root, width=60, height=15)
listbox.pack(pady=20)

btn_frame = tk.Frame(root)
btn_frame.pack()

tk.Button(btn_frame, text="Add Note", command=add_note, width=12, bg="#4CAF50", fg="white").grid(row=0, column=0, padx=5)
tk.Button(btn_frame, text="View Note", command=view_note, width=12, bg="#2196F3", fg="white").grid(row=0, column=1, padx=5)
tk.Button(btn_frame, text="Update Note", command=update_note, width=12, bg="#FFC107").grid(row=0, column=2, padx=5)
tk.Button(btn_frame, text="Delete Note", command=delete_note, width=12, bg="#F44336", fg="white").grid(row=0, column=3, padx=5)

load_notes()
root.mainloop()
