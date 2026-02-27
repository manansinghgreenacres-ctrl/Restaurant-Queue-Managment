# queue.py
# Author: Manan Singh
# A generic FIFO queue data structure with encapsulation.
# Used by the Restaurant Queue Management System to manage
# the customer order flow and ingredient conveyor belt.

class Queue:
    """
    A named FIFO queue. Raises descriptive exceptions on invalid operations
    to enforce data integrity across the system.
    """

    def __init__(self, name="Queue"):
        """Initialize a named empty queue."""
        self.__items = []
        self.name = name

    def enqueue(self, item):
        """Add an item to the rear of the queue."""
        self.__items.append(item)

    def dequeue(self):
        """Remove and return the front item. Raises if queue is empty."""
        if self.isEmpty():
            raise Exception("Cannot dequeue from Queue: Queue is empty")
        return self.__items.pop(0)

    def peek(self):
        """Return the front item without removing it. Raises if queue is empty."""
        if self.isEmpty():
            raise Exception("Cannot peek Queue: Queue is empty")
        return self.__items[0]

    def isEmpty(self):
        """Return True if the queue contains no items."""
        return len(self.__items) == 0

    def size(self):
        """Return the number of items currently in the queue."""
        return len(self.__items)

    def clear(self):
        """Remove all items from the queue."""
        self.__items = []

    def show(self):
        """Print the queue contents to stdout."""
        print(self.__items)

    def __str__(self):
        """Return a string representation of the queue."""
        return str(self.__items)

    def getItems(self):
        """Return a shallow copy of all items in the queue."""
        return self.__items.copy()
