Трекер прочитанных книг — консольное приложение.
books.json — база (список словарей: author, title, rating, date).
Меню:
1. Добавить книгу
2. Показать все книги
3. Показать среднюю оценку
4. Статистика по авторам
5. Удалить книгу
6. Выход
"""

from __future__ import annotations
import json
from datetime import datetime, date
from pathlib import Path
from typing import List, Dict, Any, Optional

DB_PATH = Path("books.json")
DATE_FMT = "%Y-%m-%d"


def load_books(path: Path = DB_PATH) -> List[Dict[str, Any]]:
    if not path.exists():
        return []
    try:
        data = json.loads(path.read_text(encoding="utf-8"))
        return data if isinstance(data, list) else []
    except Exception:
        return []


def save_books(books: List[Dict[str, Any]], path: Path = DB_PATH):
    try:
        path.write_text(json.dumps(books, ensure_ascii=False, indent=2), encoding="utf-8")
    except Exception as e:
        print("Ошибка при сохранении:", e)


def parse_date(s: str) -> Optional[str]:
    s = s.strip()
    if not s:
        return date.today().isoformat()
    try:
        d = datetime.strptime(s, DATE_FMT).date()
        return d.isoformat()
    except ValueError:
        return None


def input_rating() -> int:
    while True:
        s = input("Оценка (1–5): ").strip()
        try:
            r = int(s)
            if 1 <= r <= 5:
                return r
        except Exception:
            pass
        print("Неверная оценка. Введите целое число от 1 до 5.")


def find_duplicate(books: List[Dict[str, Any]], author: str, title: str) -> Optional[int]:
    a1 = author.strip().lower()
    t1 = title.strip().lower()
    for i, b in enumerate(books):
        if b.get("author", "").strip().lower() == a1 and b.get("title", "").strip().lower() == t1:
            return i
    return None


def add_book(books: List[Dict[str, Any]]):
    print("\nДобавление книги:")
    author = input("Автор: ").strip()
    if not author:
        print("Автор не может быть пустым.")
        return
    title = input("Название: ").strip()
    if not title:
        print("Название не может быть пустым.")
        return
    rating = input_rating()
    while True:
        d_in = input(f"Дата прочтения ({DATE_FMT}), ENTER = сегодня: ").strip()
        d_iso = parse_date(d_in)
        if d_iso is None:
            print("Неверный формат даты. Повторите ввод.")
        else:
            break
    dup = find_duplicate(books, author, title)
    if dup is not None:
        print(f"Внимание: такая книга уже есть (№{dup+1}): {books[dup]}")
        ans = input("Добавить дубликат всё равно? (y/N): ").strip().lower()
        if ans != "y":
            print("Отменено.")
            return
    item = {"author": author, "title": title, "rating": rating, "date": d_iso}
    books.append(item)
    save_books(books)
    print("Книга добавлена.\n")


def show_books(books: List[Dict[str, Any]]):
    if not books:
        print("\nСписок пуст.\n")
        return
    print()
    print(f"{'№':>3}  {'Автор':30}  {'Название':40}  {'Оценка':6}  {'Дата':10}")
    print("-" * 96)
    for i, b in enumerate(books, 1):
        a = b.get("author", "")[:30]
        t = b.get("title", "")[:40]
        r = b.get("rating", "")
        d = b.get("date", "")
        print(f"{i:3d}. {a:30}  {t:40}  {r:^6}  {d:10}")
    print()


def average_rating(books: List[Dict[str, Any]]):
    vals = [b.get("rating") for b in books if isinstance(b.get("rating"), int)]
    if not vals:
        print("\nНет данных для расчёта средней оценки.\n")
        return
    avg = sum(vals) / len(vals)
    print(f"\nСредняя оценка: {avg:.2f} (по {len(vals)} книгам)\n")


def stats_by_authors(books: List[Dict[str, Any]]):
    if not books:
        print("\nНет данных.\n")
        return
    cnt: dict[str, int] = {}
    for b in books:
        a = b.get("author", "Неизвестно")
        cnt[a] = cnt.get(a, 0) + 1
    print("\nСтатистика по авторам (количество книг):")
    for a, c in sorted(cnt.items(), key=lambda x: (-x[1], x[0])):
        print(f"  {a:30}  {c}")
    print()


def delete_book(books: List[Dict[str, Any]]):
    if not books:
        print("\nСписок пуст.\n")
        return
    show_books(books)
    s = input("Удалить: введите номер записи или 'author=...;title=...': ").strip()
    if not s:
        print("Отменено.")
        return
    if s.isdigit():
        idx = int(s) - 1
        if 0 <= idx < len(books):
            rec = books.pop(idx)
            save_books(books)
            print("Удалено:", rec)
        else:
            print("Неверный номер.")
        return
    parts = {}
    sep = ";" if ";" in s else "|"
    if sep in s:
        for kv in s.split(sep):
            if "=" in kv:
                k, v = kv.split("=", 1)
                parts[k.strip().lower()] = v.strip()
    if "author" in parts and "title" in parts:
        idx = find_duplicate(books, parts["author"], parts["title"])
        if idx is None:
            print("Запись не найдена.")
            return
        rec = books.pop(idx)
        save_books(books)
        print("Удалено:", rec)
        return
    print("Не удалось распознать ввод. Отмена.")


def print_menu():
    print("=== Трекер прочитанных книг ===")
    print("1. Добавить книгу")
    print("2. Показать все книги")
    print("3. Показать среднюю оценку")
    print("4. Статистика по авторам")
    print("5. Удалить книгу")
    print("6. Выход")


def main():
    books = load_books()
    while True:
        print_menu()
        choice = input("Выберите пункт (1-6): ").strip()
        if choice == "1":
            add_book(books)
        elif choice == "2":
            show_books(books)
        elif choice == "3":
            average_rating(books)
        elif choice == "4":
            stats_by_authors(books)
        elif choice == "5":
            delete_book(books)
        elif choice == "6":
            save_books(books)
            print("Данные сохранены. Выход.")
            break
        else:
            print("Неверный выбор, попробуйте снова.\n")


if __name__ == "__main__":
    main()
