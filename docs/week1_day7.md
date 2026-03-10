# Day 7: Pythonic Code and Project Work

Welcome to the end of Week 1! You have learned the grammar, vocabulary, and basic structure of Python. Today, we focus on writing *good* Python. 

In the industry, we call this "Pythonic Code"—code that doesn't just work, but is elegant, readable, and takes advantage of Python's unique features. We will finish the day by putting everything together into a real Command-Line Application.

## Writing Pythonic Code

As an AI engineer, you will share your code with data scientists, researchers, and software engineers. It needs to be readable. What does it mean to be Pythonic?

1.  **Descriptive Names:** A variable shouldn't be called `a`. It should be called `student_age`.
2.  **PEP 8:** Follow the official Python style guide (use 4 spaces for indentation, use `lower_case_with_underscores` for function names, etc.).
3.  **Leverage Built-ins:** Don't write 20 lines of code if Python has a built-in feature that does it in 1 line.

### Feature 1: List Comprehensions
List comprehensions are a concise way to create lists. They replace clunky `for` loops.

```python
# The Clunky Way (Not Pythonic)
squares = []
for i in range(10):
    squares.append(i * i)
print(squares)

# The Pythonic Way (List Comprehension)
squares = [i * i for i in range(10)]
print(squares)
```

```result
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

### Feature 2: Lambda Functions
Sometimes you need a tiny, single-use function, but you don't want to define a whole `def my_function():` block. Enter Lambda functions (anonymous functions).

```python
# A lambda function that adds 10 to a number
add_ten = lambda x: x + 10
print(add_ten(5)) # Output: 15
```

```result
15
```

### Feature 3: The `sys` and `os` Modules
When building AI pipelines, you constantly interact with the surrounding computer (the operating system).
*   `os`: Used to interact with the file system (creating folders, checking if files exist).
*   `sys`: Used to read system parameters, like arguments passed directly from the terminal command line!

## Hands-On Project: The Command-Line Task Manager

To prove you've mastered Week 1, let's build a fully functional application. This script uses **Variables, Conditionals, Loops, Functions, Dictionaries, and File Handling!**

Here is a look at the core structure of our project (`day7_task_manager.py`):

```python
import os

FILE_NAME = "tasks.txt"

# 1. Loading Data from the Hard Drive (File Handling)
def load_tasks():
    tasks = {}
    if os.path.exists(FILE_NAME): # Using the 'os' module!
        with open(FILE_NAME, "r") as file:
            for line in file:
                line = line.strip()
                if not line:
                    continue
                try:
                    task_id, title, status = line.split(" | ")
                    tasks[int(task_id)] = {"title": title, "status": status}
                except ValueError:
                    # Skip malformed lines
                    continue
    return tasks

def save_tasks(tasks):
    with open(FILE_NAME, "w") as file:
        for task_id in sorted(tasks.keys()):
            t = tasks[task_id]
            file.write(f"{task_id} | {t['title']} | {t['status']}\n")

# 2. Modifying Data (Dictionaries)
def add_task(tasks):
    title = input("Enter task title: ")
    # Getting the highest ID safely
    task_id = max(tasks.keys(), default=0) + 1 
    tasks[task_id] = {"title": title, "status": "incomplete"}
    print(f"Task '{title}' added.")

def view_tasks(tasks):
    if not tasks:
        print("No tasks found.")
        return
    print("\nTasks:")
    for tid in sorted(tasks.keys()):
        t = tasks[tid]
        print(f"{tid}: {t['title']} [{t['status']}]")

# 3. The Main Loop (Control Flow)
def main():
    tasks = load_tasks() # Call our function
    while True:
        print("\nTask Manager Menu:")
        print("1. Add Task | 2. View Tasks | 3. Exit")
        choice = input("Enter your choice: ")
        
        if choice == "1":
            add_task(tasks)
        elif choice == "2":
            view_tasks(tasks)
        elif choice == "3":
            # Save to disk before exiting
            save_tasks(tasks)
            print("Goodbye")
            break
        else:
            print("Invalid Choice. Please try again")

# This is a Pythonic way of saying "Only run main() if this script is executed directly"
if __name__ == "__main__":
    main()
```

```output
Task Manager Menu:
1. Add Task | 2. View Tasks | 3. Exit
Enter your choice: 2
No tasks found.

Task Manager Menu:
1. Add Task | 2. View Tasks | 3. Exit
Enter your choice: 1
Enter task title: Finish coding
Task 'Finish coding' added.

Task Manager Menu:
1. Add Task | 2. View Tasks | 3. Exit
Enter your choice: 2

Tasks:
1: Finish coding [incomplete]

Task Manager Menu:
1. Add Task | 2. View Tasks | 3. Exit
Enter your choice: 3
Goodbye
```

## Wrapping Up Week 1!
Congratulations! You have completed the foundation of your AI Engineer Bootcamp journey. You can now read, write, and structure professional Python code. 

**Next Week:** We enter the world of Data Science. We will learn **NumPy** for crunching massive numbers, **Pandas** for analyzing tables of data, and **Matplotlib/Seaborn** to visualize our findings. See you in Week 2!
