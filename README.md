# expenses-tracker
this is my project regarding the expenses  tracker that i have made it for personal use
import sqlite3
import tkinter as tk
from tkinter import ttk, messagebox, filedialog
import pandas as pd
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

# Database connection
conn = sqlite3.connect("expenses.db")
cursor = conn.cursor()

# Create expenses table
cursor.execute('''
    CREATE TABLE IF NOT EXISTS expenses (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        date TEXT,
        category TEXT,
        amount REAL,
        description TEXT
    )
''')
conn.commit()

# Function to add an expense
def add_expense():
    date = entry_date.get()
    category = combo_category.get()
    amount = entry_amount.get()
    description = entry_description.get()

    if not date or not category or not amount:
        messagebox.showerror("Error", "Please fill in all required fields")
        return

    try:
        amount = float(amount)
        cursor.execute("INSERT INTO expenses (date, category, amount, description) VALUES (?, ?, ?, ?)",
                       (date, category, amount, description))
        conn.commit()
        messagebox.showinfo("Success", "Expense added successfully")
        update_expense_list()
    except ValueError:
        messagebox.showerror("Error", "Amount must be a number")

# Function to update the expense list
def update_expense_list():
    for row in tree_expenses.get_children():
        tree_expenses.delete(row)

    cursor.execute("SELECT * FROM expenses")
    for expense in cursor.fetchall():
        tree_expenses.insert("", "end", values=expense)

# Function to delete an expense
def delete_expense():
    selected_item = tree_expenses.selection()
    if not selected_item:
        messagebox.showwarning("Warning", "Please select an expense to delete")
        return

    expense_id = tree_expenses.item(selected_item)["values"][0]
    cursor.execute("DELETE FROM expenses WHERE id = ?", (expense_id,))
    conn.commit()
    messagebox.showinfo("Success", "Expense deleted successfully")
    update_expense_list()

# Function to export data as CSV
def export_csv():
    cursor.execute("SELECT * FROM expenses")
    data = cursor.fetchall()
    
    if not data:
        messagebox.showwarning("Warning", "No data to export")
        return

    file_path = filedialog.asksaveasfilename(defaultextension=".csv", filetypes=[("CSV files", "*.csv")])
    if file_path:
        df = pd.DataFrame(data, columns=["ID", "Date", "Category", "Amount", "Description"])
        df.to_csv(file_path, index=False)
        messagebox.showinfo("Success", "Data exported successfully")

# Function to export data as PDF
def export_pdf():
    cursor.execute("SELECT * FROM expenses")
    data = cursor.fetchall()
    
    if not data:
        messagebox.showwarning("Warning", "No data to export")
        return

    file_path = filedialog.asksaveasfilename(defaultextension=".pdf", filetypes=[("PDF files", "*.pdf")])
    if file_path:
        c = canvas.Canvas(file_path, pagesize=letter)
        c.drawString(100, 750, "Expense Report")
        y = 720

        for row in data:
            c.drawString(100, y, f"{row[1]} - {row[2]} - ${row[3]:.2f} - {row[4]}")
            y -= 20

        c.save()
        messagebox.showinfo("Success", "PDF exported successfully")

# GUI setup
root = tk.Tk()
root.title("Personal Expense Tracker")
root.geometry("600x500")

# Labels and Entries
tk.Label(root, text="Date (YYYY-MM-DD):").grid(row=0, column=0, padx=5, pady=5)
entry_date = tk.Entry(root)
entry_date.grid(row=0, column=1, padx=5, pady=5)

tk.Label(root, text="Category:").grid(row=1, column=0, padx=5, pady=5)
combo_category = ttk.Combobox(root, values=["Food", "Transport", "Bills", "Shopping", "Others"])
combo_category.grid(row=1, column=1, padx=5, pady=5)

tk.Label(root, text="Amount (Rs):").grid(row=2, column=0, padx=5, pady=5)
entry_amount = tk.Entry(root)
entry_amount.grid(row=2, column=1, padx=5, pady=5)

tk.Label(root, text="Description:").grid(row=3, column=0, padx=5, pady=5)
entry_description = tk.Entry(root)
entry_description.grid(row=3, column=1, padx=5, pady=5)

# Buttons
btn_add = tk.Button(root, text="Add Expense", command=add_expense)
btn_add.grid(row=4, column=0, columnspan=2, pady=5)

btn_delete = tk.Button(root, text="Delete Expense", command=delete_expense)
btn_delete.grid(row=5, column=0, columnspan=2, pady=5)

btn_export_csv = tk.Button(root, text="Export to CSV", command=export_csv)
btn_export_csv.grid(row=6, column=0, columnspan=2, pady=5)

btn_export_pdf = tk.Button(root, text="Export to PDF", command=export_pdf)
btn_export_pdf.grid(row=7, column=0, columnspan=2, pady=5)

# Expense List (Table)
columns = ("ID", "Date", "Category", "Amount", "Description")
tree_expenses = ttk.Treeview(root, columns=columns, show="headings")
for col in columns:
    tree_expenses.heading(col, text=col)
    tree_expenses.column(col, width=100)

tree_expenses.grid(row=8, column=0, columnspan=2, padx=5, pady=5)

update_expense_list()
root.mainloop()

