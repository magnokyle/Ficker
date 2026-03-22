# HOW TO USE:
Simply open "ficker.py" by double-clicking it, or right clicking on the icon and selecting "Open."

# NOTICE:
Ficker is a Python-based program and needs to be open with terminal.

# REQUIREMENTS:
-Python installed, you can download it here: https://www.python.org/downloads/

-"filmdata.csv" on the data folder.

# INFO:
Ficker is a film picker/generator powered with PYTHON 3 that will pick a film for you when you don't know what movie to watch.

# HOW IT WORKS:
Ficker picks randomly from the file "filmdata.csv" located in the "data" folder inside the directory.

# FOR PROGRAMMERS:
The code is open-source, meaning that you can create your own generator or just study the mechanics. Credit to the owner is required if you use the code as reference.

# THE MAIN CODE
This is the main code for release 1.0.0 - this is the base-kind-of thing i use when i update my projects:

import csv
import random
import re
import sys
import time
import os

def find_ficker_folder(start_path='.'):
    """
    Recursively search for a folder named 'Ficker' starting from start_path.
    Returns the absolute path to the folder if found, else None.
    """
    for root, dirs, _ in os.walk(start_path):
        if 'Ficker' in dirs:
            return os.path.join(root, 'Ficker')
    return None

def load_filmdata_csv(ficker_path):
    """
    Load the filmdata.csv from the Ficker/data folder.
    Returns a list of dictionaries: [{'Title': ..., 'Year': ..., 'IMDb Rating': ...}, ...]
    """
    data_file = os.path.join(ficker_path, 'data', 'filmdata.csv')
    if not os.path.isfile(data_file):
        raise FileNotFoundError(f"Could not find 'filmdata.csv' at expected location: {data_file}")

    films = []
    with open(data_file, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f, quotechar='"')
        for row in reader:
            # Convert year and rating to proper types
            try:
                row['Year'] = int(row['Year'])
                row['IMDb Rating'] = float(row['IMDb Rating'])
            except ValueError:
                # Skip rows with invalid numbers
                continue
            films.append(row)
    return films

def main():
    ficker_path = find_ficker_folder()
    if not ficker_path:
        print("Could not find a folder named 'Ficker'. Make sure it's extracted.")
        return

    films = load_filmdata_csv(ficker_path)
    print(f"Loaded {len(films)} films.")
    
    # Example: print first 10 films
    for film in films[:10]:
        print(f"{film['Title']} ({film['Year']}) - IMDb: {film['IMDb Rating']}")

# ---------- COLORS ----------
RED = "\033[91m"
GREEN = "\033[92m"
RESET = "\033[0m"

# ---------- LOAD MOVIES ----------
def load_movies():
    movies = []
    # Get the folder where main.py is located
    base_dir = os.path.dirname(os.path.abspath(__file__))
    csv_path = os.path.join(base_dir, "data", "filmdata.csv")
    print(f"Looking for CSV at: {csv_path}")  # debug to verify path

    try:
        with open(csv_path, encoding="utf-8") as file:
            reader = csv.DictReader(file)
            for row in reader:
                movie = {
                    "title": row["Title"],
                    "rating": float(row["IMDb Rating"]),
                    "year": row["Year"]
                }
                movies.append(movie)
    except FileNotFoundError:
        print(f"{RED}Error: filmdata.csv not found in data folder.{RESET}")
        sys.exit()
    except Exception as e:
        print(f"{RED}Error reading CSV: {e}{RESET}")
        sys.exit()
    return movies

# ---------- INITIATE ----------
def initiate():
    while True:
        start = input("Initiate (y/n): ").lower().strip()
        if start == "y":
            return True
        elif start == "n":
            print(f"{RED}Ficker closed.{RESET}")
            time.sleep(1.5)
            sys.exit()
        else:
            print(f"{RED}Invalid input. Please type y or n.{RESET}\n")

# ---------- RATING RANGE ----------
def get_rating_range():
    pattern = r"^([0-9](\.\d)?|10(\.0)?)-([0-9](\.\d)?|10(\.0)?)$"
    while True:
        user_input = input("Enter IMDb rating range (0-10 format, e.g. 8-9) or 'any': ").strip().lower()
        if user_input == "any":
            return None, None
        if re.match(pattern, user_input):
            low, high = map(float, user_input.split("-"))
            if low <= high and 0 <= low <= 10 and 0 <= high <= 10:
                return low, high
            else:
                print(f"{RED}Ratings must be between 1.3 and 9.3 and min ≤ max.{RESET}")
        else:
            print(f"{RED}Invalid format. Use example: 8-9 or 'any'{RESET}")

# ---------- FILTER ----------
def filter_movies(movies, low, high):
    if low is None and high is None:
        return movies
    return [m for m in movies if low <= m["rating"] <= high]

# ---------- NUMBER OF MOVIES ----------
def get_number_of_movies():
    while True:
        try:
            count = int(input("Number of movies to generate? (max 15): ").strip())
            if 1 <= count <= 15:
                return count
            else:
                print(f"{RED}Please enter a number between 1 and 15.{RESET}")
        except ValueError:
            print(f"{RED}Invalid input. Enter a number between 1 and 15.{RESET}")

# ---------- PICK MULTIPLE ----------
def pick_movies(movies, count):
    if not movies:
        print(f"{RED}No movies found with that rating range.{RESET}")
        return
    count = min(count, len(movies))
    selected = random.sample(movies, count)
    print()
    for idx, movie in enumerate(selected, start=1):
        print("=====")
        print(f"Movie {idx}")
        print("Title:", movie["title"])
        print("Year:", movie["year"])
        print("IMDb Rating:", movie["rating"])
        print("=====")

# ---------- PICK AGAIN ----------
def pick_again():
    while True:
        again = input("\nPick again? (y/n): ").lower().strip()
        if again == "y":
            return True
        elif again == "n":
            print(f"{RED}Ficker closed.{RESET}")
            time.sleep(1.5)
            sys.exit()
        else:
            print(f"{RED}Invalid input. Please type y or n.{RESET}")

# ---------- MAIN ----------
def main():
    print("🎬 Welcome to Ficker 1.0.0 - Created by Kyle R.\n")
    initiate()
    movies = load_movies()
    while True:
        low, high = get_rating_range()
        results = filter_movies(movies, low, high)
        print(f"\nFound {len(results)} matching movies.\n")
        count = get_number_of_movies()
        pick_movies(results, count)
        pick_again()

if __name__ == "__main__":
    main()
