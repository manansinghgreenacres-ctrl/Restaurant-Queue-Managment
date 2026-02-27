# stack.py
# Author: Manan Singh
# A generic stack data structure with encapsulation and bounded capacity.
# Used by the Restaurant Queue Management System for order and ingredient tracking.

class Stack:
    """
    A LIFO stack with a configurable maximum size.
    Raises exceptions on invalid operations (push to full, pop/peek from empty)
    to enforce data integrity across the system.
    """

    def __init__(self, name, maxSize):
        """Initialize a named stack with a fixed capacity."""
        self.__items = []
        self.name = name
        self.maxSize = maxSize

    def push(self, item):
        """Add an item to the top of the stack. Raises if stack is full."""
        if self.isFull():
            raise Exception("Cannot push to Stack: Stack is full")
        self.__items.append(item)

    def pop(self):
        """Remove and return the top item. Raises if stack is empty."""
        if self.isEmpty():
            raise Exception("Cannot pop from Stack: Stack is empty")
        return self.__items.pop()

    def peek(self):
        """Return the top item without removing it. Raises if stack is empty."""
        if self.isEmpty():
            raise Exception("Cannot peek Stack: Stack is empty")
        return self.__items[-1]

    def isEmpty(self):
        """Return True if the stack contains no items."""
        return len(self.__items) == 0

    def isFull(self):
        """Return True if the stack has reached maximum capacity."""
        return len(self.__items) >= self.maxSize

    def size(self):
        """Return the number of items currently in the stack."""
        return len(self.__items)

    def clear(self):
        """Remove all items from the stack."""
        self.__items = []

    def getItems(self):
        """Return a copy of all items in the stack (bottom to top)."""
        return self.__items.copy()
