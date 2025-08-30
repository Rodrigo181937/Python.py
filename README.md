
import rlcompleter
import sys
import os
import traceback

# Autocomplete setup
readline.parse_and_bind("tab: complete")
readline.set_completer(rlcompleter.Completer(namespace={}).complete)

# Terminal colors
RESET = "\033[0m"
PROMPT_COLOR = "\033[92m"   # Green
ERROR_COLOR = "\033[91m"    # Red
INFO_COLOR = "\033[94m"     # Blue

print(f"{PROMPT_COLOR}Python-in-Python Interactive Interpreter (type 'exit' to quit){RESET}")
print(f"{INFO_COLOR}Special commands: 'runfile <filename.py>', 'help', 'graphics_mode'{RESET}")

# Persistent environment
env = {}
buffer = []

while True:
    try:
        prompt = f"{PROMPT_COLOR}... {RESET}" if buffer else f"{PROMPT_COLOR}>>> {RESET}"
        line = input(prompt)
    except (KeyboardInterrupt, EOFError):
        print("\nExiting Python-in-Python.")
        break

    # Quit command
    if line.lower() in ("exit", "quit"):
        print("Exiting Python-in-Python. Goodbye!")
        break

    # Run external Python file
    elif line.startswith("runfile "):
        filename = line.split(maxsplit=1)[1]
        if os.path.exists(filename):
            try:
                with open(filename, "r") as f:
                    code_file = f.read()
                exec(code_file, env)
                print(f"{INFO_COLOR}File '{filename}' executed successfully.{RESET}")
            except Exception:
                print(f"{ERROR_COLOR}Error executing file:{RESET}\n{traceback.format_exc()}")
        else:
            print(f"{ERROR_COLOR}File '{filename}' not found.{RESET}")
        continue

    # Graphics mode (simple turtle)
    elif line == "graphics_mode":
        try:
            import turtle
            print(f"{INFO_COLOR}Opening graphics mode. Close the window to return.{RESET}")
            t = turtle.Turtle()
            turtle.done()
        except Exception:
            print(f"{ERROR_COLOR}Error opening graphics mode:{RESET}\n{traceback.format_exc()}")
        continue

    # Help command
    elif line == "help":
        print(f"{INFO_COLOR}Python-in-Python Features:\n"
              "- Regular Python commands\n"
              "- runfile <filename.py> to execute scripts\n"
              "- graphics_mode for Turtle graphics\n"
              "- Tab for autocomplete\n"
              "- Up/Down arrows for history{RESET}")
        continue

    # Add line to buffer for multi-line code
    buffer.append(line)
    code = "\n".join(buffer)

    # Try eval first (for expressions)
    try:
        result = eval(code, env)
        if result is not None:
            print(result)
        buffer = []
    except SyntaxError as e:
        # If block is incomplete, wait for more lines
        if str(e).startswith("unexpected EOF") or str(e).startswith("invalid syntax"):
            continue
        else:
            try:
                exec(code, env)
                buffer = []
            except Exception:
                print(f"{ERROR_COLOR}Error:{RESET}\n{traceback.format_exc()}")
                buffer = []
    except Exception:
        print(f"{ERROR_COLOR}Error:{RESET}\n{traceback.format_exc()}")
        buffer = []
