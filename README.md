import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime
import json
import os

DATA_FILE = "expenses.json"

def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return []

def save_data(data):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=4, ensure_ascii=False)

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.root.geometry("800x500")
        self.expenses = load_data()
        self.filtered_expenses = self.expenses.copy()

        # Поля ввода
        tk.Label(root, text="Сумма:").grid(row=0, column=0, padx=5, pady=5, sticky="e")
        self.amount_entry = tk.Entry(root)
        self.amount_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(root, text="Категория:").grid(row=0, column=2, padx=5, pady=5, sticky="e")
        self.category_var = tk.StringVar()
        self.category_combo = ttk.Combobox(root, textvariable=self.category_var, values=["Еда", "Транспорт", "Развлечения", "Здоровье", "Другое"])
        self.category_combo.grid(row=0, column=3, padx=5, pady=5)

        tk.Label(root, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=4, padx=5, pady=5, sticky="e")
        self.date_entry = tk.Entry(root)
        self.date_entry.grid(row=0, column=5, padx=5, pady=5)
        self.date_entry.insert(0, datetime.today().strftime("%Y-%m-%d"))

        self.add_btn = tk.Button(root, text="Добавить расход", command=self.add_expense)
        self.add_btn.grid(row=0, column=6, padx=10, pady=5)

        # Таблица
        columns = ("id", "amount", "category", "date")
        self.tree = ttk.Treeview(root, columns=columns, show="headings")
        self.tree.heading("id", text="ID")
        self.tree.heading("amount", text="Сумма")
        self.tree.heading("category", text="Категория")
        self.tree.heading("date", text="Дата")
        self.tree.column("id", width=40)
        self.tree.column("amount", width=100)
        self.tree.column("category", width=120)
        self.tree.column("date", width=120)
        self.tree.grid(row=2, column=0, columnspan=7, padx=10, pady=10, sticky="nsew")

        scrollbar = ttk.Scrollbar(root, orient="vertical", command=self.tree.yview)
        scrollbar.grid(row=2, column=7, sticky="ns")
        self.tree.configure(yscrollcommand=scrollbar.set)

        # Фильтры
        filter_frame = tk.Frame(root)
        filter_frame.grid(row=1, column=0, columnspan=7, pady=5, sticky="ew")

        tk.Label(filter_frame, text="Фильтр по категории:").pack(side="left", padx=5)
        self.filter_category_var = tk.StringVar()
        self.filter_category_combo = ttk.Combobox(filter_frame, textvariable=self.filter_category_var, values=["Все", "Еда", "Транспорт", "Развлечения", "Здоровье", "Другое"])
        self.filter_category_combo.pack(side="left", padx=5)
        self.filter_category_combo.current(0)

        tk.Label(filter_frame, text="Дата от (ГГГГ-ММ-ДД):").pack(side="left", padx=5)
        self.filter_date_from = tk.Entry(filter_frame, width=12)
        self.filter_date_from.pack(side="left", padx=5)

        tk.Label(filter_frame, text="до:").pack(side="left", padx=5)
        self.filter_date_to = tk.Entry(filter_frame, width=12)
        self.filter_date_to.pack(side="left", padx=5)

        self.filter_btn = tk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter)
        self.filter_btn.pack(side="left", padx=10)

        self.reset_filter_btn = tk.Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter)
        self.reset_filter_btn.pack(side="left", padx=5)

        # Подсчёт суммы за период
        period_frame = tk.Frame(root)
        period_frame.grid(row=3, column=0, columnspan=7, pady=10)

        tk.Label(period_frame, text="Период (от и до):").pack(side="left", padx=5)
22:02
self.period_from = tk.Entry(period_frame, width=12)
        self.period_from.pack(side="left", padx=5)
        self.period_to = tk.Entry(period_frame, width=12)
        self.period_to.pack(side="left", padx=5)
        self.calc_btn = tk.Button(period_frame, text="Подсчитать сумму", command=self.calc_sum_period)
        self.calc_btn.pack(side="left", padx=10)
        self.sum_label = tk.Label(period_frame, text="Сумма: 0.00", font=("Arial", 10, "bold"))
        self.sum_label.pack(side="left", padx=10)

        root.grid_rowconfigure(2, weight=1)
        root.grid_columnconfigure(0, weight=1)

        self.update_table()

    def validate_amount(self, amount_str):
        try:
            amount = float(amount_str)
            if amount <= 0:
                raise ValueError
            return amount
        except:
            messagebox.showerror("Ошибка", "Сумма должна быть положительным числом.")
            return None

    def validate_date(self, date_str):
        try:
            datetime.strptime(date_str, "%Y-%m-%d")
            return True
        except:
            messagebox.showerror("Ошибка", "Неверный формат даты. Используйте ГГГГ-ММ-ДД")
            return False

    def add_expense(self):
        amount_str = self.amount_entry.get().strip()
        category = self.category_var.get().strip()
        date_str = self.date_entry.get().strip()

        if not amount_str or not category or not date_str:
            messagebox.showerror("Ошибка", "Все поля обязательны для заполнения.")
            return

        amount = self.validate_amount(amount_str)
        if amount is None:
            return
        if not self.validate_date(date_str):
            return

        new_id = max([e["id"] for e in self.expenses] + [0]) + 1

        expense = {"id": new_id, "amount": amount, "category": category, "date": date_str}
        self.expenses.append(expense)
        self.filtered_expenses = self.expenses.copy()
        save_data(self.expenses)
        self.update_table()
        self.reset_filter()
        self.amount_entry.delete(0, tk.END)
        self.category_combo.set("")
        self.date_entry.delete(0, tk.END)
        self.date_entry.insert(0, datetime.today().strftime("%Y-%m-%d"))

    def apply_filter(self):
        cat = self.filter_category_var.get()
        date_from = self.filter_date_from.get().strip()
        date_to = self.filter_date_to.get().strip()

        filtered = self.expenses.copy()

        if cat != "Все" and cat != "":
            filtered = [e for e in filtered if e["category"] == cat]

        if date_from:
            if not self.validate_date(date_from):
                return
            filtered = [e for e in filtered if e["date"] >= date_from]

        if date_to:
            if not self.validate_date(date_to):
                return
            filtered = [e for e in filtered if e["date"] <= date_to]

        self.filtered_expenses = filtered
        self.update_table(use_filtered=True)

    def reset_filter(self):
        self.filter_category_var.set("Все")
        self.filter_date_from.delete(0, tk.END)
        self.filter_date_to.delete(0, tk.END)
        self.filtered_expenses = self.expenses.copy()
        self.update_table(use_filtered=True)

    def update_table(self, use_filtered=False):
        for row in self.tree.get_children():
            self.tree.delete(row)

        data = self.filtered_expenses if use_filtered else self.expenses
        for exp in data:
            self.tree.insert("", tk.END, values=(exp["id"], exp["amount"], exp["category"], exp["date"]))

    def calc_sum_period(self):
        date_from = self.period_from.get().strip()
        date_to = self.period_to.get().strip()

        if not date_from or not date_to:
            messagebox.showerror("Ошибка", "Введите обе даты периода.")
            return

        if not self.validate_date(date_from) or not self.validate_date(date_to):
            return

        total = 0.0
22:02
for exp in self.expenses:
            if date_from <= exp["date"] <= date_to:
                total += exp["amount"]

        self.sum_label.config(text=f"Сумма: {total:.2f}")

if __name__ == "__main__":
    root = tk.Tk()
    app = ExpenseTracker(root)
    root.mainloop()# -
