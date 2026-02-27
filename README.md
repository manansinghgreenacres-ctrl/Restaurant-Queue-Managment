# Restaurant Queue Management System

A terminal-based restaurant simulation game built in Python using custom stack and queue data structures. Ingredients move along a conveyor belt (queue) and are matched to customer orders using a priority-based algorithm, with a real-time ANSI colour GUI rendered directly in the terminal.

## Features
- Custom `Stack` and `Queue` implementations with encapsulation
- Priority-based ingredient matching across 3 burger stacks and a reserve stack
- Randomized ingredient conveyor belt read from file
- Real-time terminal GUI using ANSI escape codes
- Auto-generated `order_log.txt` after each game session

## Files
| File | Description |
|------|-------------|
| `assignment3.py` | Main game logic and terminal GUI |
| `stack.py` | Stack data structure implementation |
| `queue.py` | Queue data structure implementation |
| `customer_orders.txt` | Input file with customer orders |
| `ingredients.txt` | Input file with available ingredients |

## How to Run
```
python assignment3.py
```
Follow the on-screen prompt (`Continue (Y/N)?`) to step through the game one move at a time. An `order_log.txt` file will be generated when the game ends.

## Requirements
- Python 3.x
- No external libraries required
