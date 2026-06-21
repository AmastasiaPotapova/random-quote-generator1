# random-quote-generator1import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext
import random
import json
import os

class QuoteApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Quote Generator")
        self.root.geometry("600x700")

        # Предопределённые цитаты
        self.base_quotes = [
            {"text": "Знание — сила.", "author": "Фрэнсис Бэкон", "topic": "Наука"},
            {"text": "Быть или не быть — вот в чём вопрос.", "author": "Уильям Шекспир", "topic": "Философия"},
            {"text": "Познай самого себя.", "author": "Сократ", "topic": "Самопознание"},
            {"text": "Я мыслю, следовательно, существую.", "author": "Рене Декарт", "topic": "Мышление"},
            {"text": "Через тернии к звёздам.", "author": "Сенека", "topic": "Мотивация"},
            {"text": "Жизнь — это то, что люди больше всего стремятся сохранить и меньше всего берегут.", "author": "Жан де Лабрюйер", "topic": "Жизнь"},
            {"text": "Нет ничего невозможного для того, кто пытается.", "author": "Александр Македонский", "topic": "Упорство"},
            {"text": "В центре каждой трудности лежит возможность.", "author": "Альберт Эйнштейн", "topic": "Трудности"}
        ]
        
        self.quotes = self.base_quotes.copy()
        self.history = []
        self.data_file = "quotes_data.json"

        self.load_data()

        self.create_widgets()
        self.update_filters()

    def create_widgets(self):
        # --- Блок фильтров ---
        filter_frame = ttk.LabelFrame(self.root, text="Фильтры", padding=10)
        filter_frame.pack(fill="x", padx=10, pady=5)

        ttk.Label(filter_frame, text="Автор:").grid(row=0, column=0, padx=5, pady=5, sticky="e")
        self.author_combo = ttk.Combobox(filter_frame, state="readonly", width=20)
        self.author_combo.grid(row=0, column=1, padx=5, pady=5)
        self.author_combo.bind("<<ComboboxSelected>>", self.apply_filters)

        ttk.Label(filter_frame, text="Тема:").grid(row=0, column=2, padx=5, pady=5, sticky="e")
        self.topic_combo = ttk.Combobox(filter_frame, state="readonly", width=20)
        self.topic_combo.grid(row=0, column=3, padx=5, pady=5)
        self.topic_combo.bind("<<ComboboxSelected>>", self.apply_filters)

        # --- Кнопка генерации ---
        btn_frame = ttk.Frame(self.root)
        btn_frame.pack(pady=10)

        generate_btn = ttk.Button(btn_frame, text="Сгенерировать цитату", command=self.generate_quote)
        generate_btn.pack(side="left", padx=20)

        clear_hist_btn = ttk.Button(btn_frame, text="Очистить историю", command=self.clear_history)
        clear_hist_btn.pack(side="left", padx=20)

        # --- Отображение текущей цитаты ---
        quote_frame = ttk.LabelFrame(self.root, text="Текущая цитата", padding=15)
        quote_frame.pack(fill="both", expand=True, padx=10, pady=5)

        self.quote_label = ttk.Label(quote_frame, text="Нажмите кнопку выше, чтобы получить цитату", 
                                     wraplength=500, font=("Arial", 12), justify="center")
        self.quote_label.pack(pady=10)
        
        self.author_label = ttk.Label(quote_frame, text="", font=("Arial", 10, "italic"))
        self.author_label.pack()

        # --- История ---
        hist_frame = ttk.LabelFrame(self.root, text="История", padding=5)
        hist_frame.pack(fill="both", expand=True, padx=10, pady=5)

        self.history_text = scrolledtext.ScrolledText(hist_frame, height=8, state='disabled')
        self.history_text.pack(fill="both", expand=True)

    def update_filters(self):
        """Обновляет выпадающие списки авторов и тем"""
        authors = sorted(set(q["author"] for q in self.quotes))
        topics = sorted(set(q["topic"] for q in self.quotes))

        self.author_combo["values"] = ["Все"] + authors
        self.topic_combo["values"] = ["Все"] + topics
        self.author_combo.set("Все")
        self.topic_combo.set("Все")

    def apply_filters(self, event=None):
        author_filter = self.author_combo.get()
        topic_filter = self.topic_combo.get()

        filtered_quotes = self.quotes

        if author_filter != "Все":
            filtered_quotes = [q for q in filtered_quotes if q["author"] == author_filter]
        if topic_filter != "Все":
            filtered_quotes = [q for q in filtered_quotes if q["topic"] == topic_filter]

        if not filtered_quotes:
            messagebox.showwarning("Внимание", "По выбранным фильтрам цитат не найдено.")
            self.quote_label.config(text="Нет подходящих цитат")
            self.author_label.config(text="")
        else:
            # Если фильтры применены, но кнопка не нажата, мы просто обновляем состояние,
            # но не генерируем новую цитату автоматически, чтобы не сбивать историю.
            pass

    def generate_quote(self):
        author_filter = self.author_combo.get()
        topic_filter = self.topic_combo.get()

        filtered_quotes = self.quotes

        if author_filter != "Все":
            filtered_quotes = [q for q in filtered_quotes if q["author"] == author_filter]
        if topic_filter != "Все":
            filtered_quotes = [q for q in filtered_quotes if q["topic"] == topic_filter]

        if not filtered_quotes:
            messagebox.showerror("Ошибка", "Нет цитат, соответствующих выбранным фильтрам!")
            return

        chosen_quote = random.choice(filtered_quotes)
        self.history.append(chosen_quote)

        self.display_quote(chosen_quote)
        self.update_history_display()
        self.save_data()

    def display_quote(self, quote):
        self.quote_label.config(text=f'"{quote["text"]}"')
        self.author_label.config(text=f"— {quote['author']} ({quote['topic']})")

    def update_history_display(self):
        self.history_text.config(state='normal')
        self.history_text.delete(1.0, tk.END)
        
        # Показываем последние 10 цитат
        recent_history = self.history[-10:]
        for i, q in enumerate(recent_history, 1):
            line = f"{i}. \"{q['text']}\"\n   Автор: {q['author']} | Тема: {q['topic']}\n"
            self.history_text.insert(tk.END, line)
            
        self.history_text.config(state='disabled')

    def clear_history(self):
        if messagebox.askyesno("Подтверждение", "Вы уверены, что хотите очистить историю?"):
            self.history = []
            self.update_history_display()
            self.save_data()

    def save_data(self):
        data = {
            "quotes": self.quotes,
            "history": self.history
        }
        try:
            with open(self.data_file, 'w', encoding='utf-8') as f:
                json.dump(data, f, ensure_ascii=False, indent=4)
        except Exception as e:
            messagebox.showerror("Ошибка сохранения", str(e))

    def load_data(self):
        if os.path.exists(self.data_file):
            try:
                with open(self.data_file, 'r', encoding='utf-8') as f:
                    data = json.load(f)
                    # Загружаем историю, а цитаты оставляем базовыми или тоже загружаем, если нужно расширение
                    self.history = data.get("history", [])
                    # Если хотите сохранять и новые добавленные цитаты между сессиями, раскомментируйте строку ниже:
                    # self.quotes = data.get("quotes", self.base_quotes) 
            except Exception as e:
                messagebox.showerror("Ошибка загрузки", f"Не удалось загрузить данные: {e}")

if __name__ == "__main__":
    root = tk.Tk()
    app = QuoteApp(root)
    root.mainloop()
