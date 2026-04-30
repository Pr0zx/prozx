import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime
import os

class WeatherDiary:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary")
        self.root.geometry("750x500")
        
        self.data_file = "weather_data.json"
        self.entries = []
        self.load_entries()
        
        self.create_widgets()
        self.refresh_display()
    
    def create_widgets(self):
        # Фрейм ввода
        f_in = tk.LabelFrame(self.root, text="Новая запись", padx=10, pady=5)
        f_in.pack(fill="x", padx=10, pady=5)
        
        tk.Label(f_in, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0)
        self.date = tk.Entry(f_in, width=15)
        self.date.grid(row=0, column=1)
        self.date.insert(0, datetime.now().strftime("%Y-%m-%d"))
        
        tk.Label(f_in, text="Температура:").grid(row=0, column=2)
        self.temp = tk.Entry(f_in, width=8)
        self.temp.grid(row=0, column=3)
        
        tk.Label(f_in, text="Описание:").grid(row=1, column=0)
        self.desc = tk.Entry(f_in, width=40)
        self.desc.grid(row=1, column=1, columnspan=3)
        
        self.precip = tk.BooleanVar()
        tk.Checkbutton(f_in, text="Осадки", variable=self.precip).grid(row=0, column=4)
        
        tk.Button(f_in, text="Добавить", command=self.add, bg="#4CAF50", fg="white").grid(row=1, column=4)
        
        # Фрейм фильтров
        f_filt = tk.LabelFrame(self.root, text="Фильтры", padx=10, pady=5)
        f_filt.pack(fill="x", padx=10, pady=5)
        
        tk.Label(f_filt, text="Дата:").grid(row=0, column=0)
        self.f_date = tk.Entry(f_filt, width=12)
        self.f_date.grid(row=0, column=1)
        tk.Button(f_filt, text="Применить", command=self.apply_filter).grid(row=0, column=2)
        
        tk.Label(f_filt, text="Температура >").grid(row=0, column=3)
        self.f_temp = tk.Entry(f_filt, width=6)
        self.f_temp.grid(row=0, column=4)
        tk.Button(f_filt, text="Применить", command=self.apply_filter).grid(row=0, column=5)
        
        tk.Button(f_filt, text="Сбросить", command=self.reset_filter).grid(row=0, column=6)
        
        # Таблица
        self.tree = ttk.Treeview(self.root, columns=("date","temp","desc","precip"), show="headings")
        self.tree.heading("date", text="Дата")
        self.tree.heading("temp", text="°C")
        self.tree.heading("desc", text="Описание")
        self.tree.heading("precip", text="Осадки")
        self.tree.column("desc", width=300)
        self.tree.pack(fill="both", expand=True, padx=10, pady=5)
    
    def validate_date(self, d):
        try: datetime.strptime(d, "%Y-%m-%d"); return True
        except: return False
    
    def add(self):
        date, temp_str, desc = self.date.get().strip(), self.temp.get().strip(), self.desc.get().strip()
        
        if not self.validate_date(date): messagebox.showerror("Ошибка", "Формат даты: ГГГГ-ММ-ДД"); return
        if not temp_str: messagebox.showerror("Ошибка", "Введите температуру"); return
        try: temp = float(temp_str)
        except: messagebox.showerror("Ошибка", "Температура - число"); return
        if not desc: messagebox.showerror("Ошибка", "Введите описание"); return
        
        self.entries.append({"date": date, "temperature": temp, "description": desc, "precipitation": "Да" if self.precip.get() else "Нет"})
        self.save_entries()
        self.refresh_display()
        
        self.date.delete(0, tk.END); self.date.insert(0, datetime.now().strftime("%Y-%m-%d"))
        self.temp.delete(0, tk.END); self.desc.delete(0, tk.END); self.precip.set(False)
        messagebox.showinfo("Успех", "Добавлено")
    
    def apply_filter(self):
        filtered = self.entries[:]
        fd = self.f_date.get().strip()
        if fd and self.validate_date(fd):
            filtered = [e for e in filtered if e["date"] == fd]
        ft = self.f_temp.get().strip()
        if ft:
            try: filtered = [e for e in filtered if e["temperature"] > float(ft)]
            except: pass
        self.display(filtered)
    
    def reset_filter(self):
        self.f_date.delete(0, tk.END); self.f_temp.delete(0, tk.END)
        self.refresh_display()
    
    def display(self, data):
        for item in self.tree.get_children(): self.tree.delete(item)
        for e in data: self.tree.insert("", "end", values=(e["date"], e["temperature"], e["description"], e["precipitation"]))
    
    def refresh_display(self): self.display(self.entries)
    def save_entries(self):
        with open(self.data_file, "w", encoding="utf-8") as f: json.dump(self.entries, f, ensure_ascii=False, indent=2)
    def load_entries(self):
        if os.path.exists(self.data_file):
            with open(self.data_file, "r", encoding="utf-8") as f: self.entries = json.load(f)

if __name__ == "__main__":
    tk.Tk().mainloop()
