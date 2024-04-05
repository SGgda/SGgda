import tkinter as tk
from tkinter import messagebox, ttk
import matplotlib.pyplot as plt
from datetime import datetime

class ExpenseTracker:
    def _init_(self, root):
        self.root = root
        self.root.title("Expense Tracker")

        self.expenses = []
        self.categories = set()

        self.expense_label = tk.Label(root, text="Enter Expense:")
        self.expense_label.grid(row=0, column=0)

        self.expense_entry = tk.Entry(root)
        self.expense_entry.grid(row=0, column=1)

        self.amount_label = tk.Label(root, text="Enter Amount:")
        self.amount_label.grid(row=1, column=0)

        self.amount_entry = tk.Entry(root)
        self.amount_entry.grid(row=1, column=1)

        self.category_label = tk.Label(root, text="Enter Category:")
        self.category_label.grid(row=2, column=0)

        self.category_entry = tk.Entry(root)
        self.category_entry.grid(row=2, column=1)

        self.add_button = tk.Button(root, text="Add Expense", command=self.add_expense)
        self.add_button.grid(row=3, column=0, columnspan=2)

        self.expense_listbox = tk.Listbox(root)
        self.expense_listbox.grid(row=4, column=0, columnspan=2, rowspan=4, sticky='nsew')

        self.total_label = tk.Label(root, text="Total Expenses: $0.00")
        self.total_label.grid(row=8, column=0, columnspan=2)

        self.filter_label = tk.Label(root, text="Filter by Category:")
        self.filter_label.grid(row=9, column=0)

        self.filter_combobox = ttk.Combobox(root, values=["All"])
        self.filter_combobox.grid(row=9, column=1)
        self.filter_combobox.bind("<<ComboboxSelected>>", self.filter_expenses)
        self.filter_combobox.set("All")

        self.plot_button = tk.Button(root, text="Plot Expenses", command=self.plot_expenses)
        self.plot_button.grid(row=10, column=0, columnspan=2)

    def add_expense(self):
        expense = self.expense_entry.get()
        amount = self.amount_entry.get()
        category = self.category_entry.get()

        if expense and amount:
            try:
                amount = float(amount)
                self.expenses.append({"expense": expense, "amount": amount, "date": datetime.now(), "category": category})
                self.categories.add(category)
                self.update_listbox()
                self.update_total()
                self.update_category_filter()
                self.expense_entry.delete(0, tk.END)
                self.amount_entry.delete(0, tk.END)
                self.category_entry.delete(0, tk.END)
            except ValueError:
                messagebox.showerror("Error", "Please enter a valid amount.")
        else:
            messagebox.showerror("Error", "Please enter both expense and amount.")

    def update_listbox(self):
        self.expense_listbox.delete(0, tk.END)
        for expense in self.expenses:
            self.expense_listbox.insert(tk.END, f"{expense['expense']}: ${expense['amount']:.2f} ({expense['date'].strftime('%Y-%m-%d')})")

    def update_total(self):
        total = sum(expense['amount'] for expense in self.expenses)
        self.total_label.config(text=f"Total Expenses: ${total:.2f}")

    def update_category_filter(self):
        self.filter_combobox['values'] = ["All"] + list(self.categories)

    def filter_expenses(self, event):
        selected_category = self.filter_combobox.get()
        if selected_category == "All":
            self.update_listbox()
        else:
            filtered_expenses = [expense for expense in self.expenses if expense['category'] == selected_category]
            self.expense_listbox.delete(0, tk.END)
            for expense in filtered_expenses:
                self.expense_listbox.insert(tk.END, f"{expense['expense']}: ${expense['amount']:.2f} ({expense['date'].strftime('%Y-%m-%d')})")

    def plot_expenses(self):
        categories = []
        amounts = []
        for category in self.categories:
            total_amount = sum(expense['amount'] for expense in self.expenses if expense['category'] == category)
            categories.append(category)
            amounts.append(total_amount)

        plt.figure(figsize=(8, 6))
        plt.bar(categories, amounts, color='blue')
        plt.xlabel('Categories')
        plt.ylabel('Total Expenses ($)')
        plt.title('Expense Distribution by Category')
        plt.xticks(rotation=45, ha='right')
        plt.tight_layout()
        plt.show()

if _name_ == "_main_":
    root = tk.Tk()
    app = ExpenseTracker(root)
    root.mainloop()
