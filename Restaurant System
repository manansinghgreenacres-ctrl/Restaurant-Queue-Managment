# Restaurant Queue Management System
# Author: Manan Singh
# Terminal-based simulation of a restaurant order management system.
# Uses custom stack and queue data structures to manage concurrent customer
# workflows, ingredient routing, and a priority-based matching algorithm.

from stack import Stack
from queue import Queue
import random

# ANSI color constants for terminal display
# defined as constants so colors can be updated in one place if needed
RESET = '\033[0m'
BLUE_BG = '\033[48;5;27m'        # customer queue
GREEN_BG = '\033[48;5;22m'       # ingredients queue
BROWN_BG = '\033[48;5;94m'       # Burg1 station
ORANGE_BG = '\033[48;5;130m'     # Burg2 station
RED_BG = '\033[48;5;88m'         # Burg3 station
PURPLE_BG = '\033[48;5;53m'      # reserve stack
YELLOW_BG = '\033[48;5;58m'      # CustA slot
DARK_PURPLE_BG = '\033[48;5;60m' # CustB slot
DARK_BLUE_BG = '\033[48;5;24m'   # CustC slot


def readCustomerFile(filename):
    """
    Reads customer orders from a CSV file and returns a list of customer dicts.
    Skips the header row and ignores empty/nil ingredient entries.
    Each customer is represented as {'id': str, 'ingredients': list}.
    """
    customers = []
    file = open(filename, 'r')
    lines = file.readlines()
    file.close()

    # skip header row, start parsing from index 1
    for i in range(1, len(lines)):
        line = lines[i].strip()
        if line:
            parts = line.split(',')
            customerId = parts[0].strip()

            # collect only valid ingredients, skip nil placeholders
            ingredients = []
            for j in range(1, len(parts)):
                ing = parts[j].strip()
                if ing.lower() != 'nil':
                    ingredients.append(ing)

            # only register customers that have at least one ingredient
            if ingredients:
                customers.append({'id': customerId, 'ingredients': ingredients})

    return customers


def readIngredientsFile(filename):
    """
    Reads the available ingredient list from a text file, one per line.
    Returns a list of ingredient name strings.
    """
    ingredients = []
    file = open(filename, 'r')
    lines = file.readlines()
    file.close()

    for line in lines:
        ingredient = line.strip()
        if ingredient:
            ingredients.append(ingredient)

    return ingredients


def makeCustomerQueue(customers):
    """
    Builds the customer_line FIFO queue from the parsed customer list.
    Preserves the original file order so customers are served first-come, first-served.
    """
    customerLine = Queue("customer_line")

    # add each customer dict to the queue in order
    for customer in customers:
        customerLine.enqueue(customer)

    return customerLine


def makeIngredientsQueue(ingredients):
    """
    Randomizes the ingredient list to simulate a conveyor belt,
    then loads them into a FIFO queue named ingredients_queue.
    """
    # shuffle to simulate a randomized conveyor belt sequence
    random.shuffle(ingredients)

    ingredientsQueue = Queue("ingredients_queue")
    for ingredient in ingredients:
        ingredientsQueue.enqueue(ingredient)

    return ingredientsQueue


def makeBurgerStacks():
    """
    Creates the 3 preparation station stacks: Burg1, Burg2, Burg3.
    Each station holds up to 3 ingredients for its currently assigned customer.
    """
    burg1 = Stack("Burg1", maxSize=3)
    burg2 = Stack("Burg2", maxSize=3)
    burg3 = Stack("Burg3", maxSize=3)

    return [burg1, burg2, burg3]


def makeReserveStack():
    """
    Creates the Resv0 reserve stack.
    Stores ingredients that no active station currently needs,
    allowing them to be matched and retrieved later.
    """
    resv0 = Stack("Resv0", maxSize=100)
    return resv0


def makeCustomerStacks():
    """
    Creates 3 customer order stacks: CustA, CustB, CustC.
    Each stack holds the ingredients a customer has ordered,
    serving as a reference for stations to match against.
    """
    custA = Stack("CustA", maxSize=3)
    custB = Stack("CustB", maxSize=3)
    custC = Stack("CustC", maxSize=3)

    return [custA, custB, custC]


def assignFirstThreeCustomers(customerLine, burgerStacks, customerStacks, log):
    """
    Initializes the system by dequeuing the first 3 customers and
    assigning each to a station: Burg1->CustA, Burg2->CustB, Burg3->CustC.
    Logs each assignment to the session log.
    """
    for i in range(3):
        if not customerLine.isEmpty():
            # dequeue the next customer and extract their data
            customer = customerLine.dequeue()
            customerId = customer['id']
            ingredients = customer['ingredients']

            # push the customer's order into their designated customer stack
            for ingredient in ingredients:
                customerStacks[i].push(ingredient)

            # attach customer metadata to the station stack for tracking during routing
            burgerStacks[i].customerId = customerId
            burgerStacks[i].custStack = i  # maps to 0=CustA, 1=CustB, 2=CustC

            ingList = ", ".join(ingredients)
            log.append(
                f"A new customer {customerId} was dequeued and assigned to {customerStacks[i].name} for {burgerStacks[i].name} with ingredients [{ingList}]")


def shortenName(ingredient):
    """
    Truncates ingredient names to a max of 4 characters for compact terminal rendering.
    Shorter names are returned unchanged.
    """
    if len(ingredient) > 4:
        return ingredient[:4]
    return ingredient


def showQueue(queue, color):
    """
    Renders a queue to the terminal using a fixed 5-slot display window.
    Empty slots are shown as blank colored cells to maintain consistent layout.
    Handles both customer dicts (shows ID) and plain ingredient strings.
    """
    items = queue.getItems()
    display = f"{queue.name}: "

    # always render exactly 5 slots regardless of queue size
    for i in range(5):
        if i < len(items):
            # customer queue items are dicts — extract just the ID for display
            if isinstance(items[i], dict):
                shortName = shortenName(items[i]['id'])
            else:
                shortName = shortenName(items[i])
            display += f"{color} {shortName:4} {RESET} "
        else:
            # render an empty placeholder slot
            display += f"{color}      {RESET} "

    print(display)


def showBurgerStack(stack, color):
    """
    Renders a station or reserve stack to the terminal.
    For 3-slot station stacks, empty slots are rendered to keep
    the visual layout consistent as ingredients are added.
    """
    items = stack.getItems()
    display = f"{stack.name}: "

    # render each ingredient currently in the stack
    for item in items:
        shortName = shortenName(item)
        display += f"{color} {shortName:4} {RESET} "

    # pad remaining slots for station stacks to maintain fixed-width layout
    if stack.maxSize == 3:
        empty = 3 - len(items)
        for i in range(empty):
            display += f"{color}      {RESET} "

    print(display)


def showCustomerStack(stack, color):
    """
    Renders a customer order stack to the terminal.
    Always displays the customer's full order from top to bottom —
    this view stays static throughout the order lifecycle.
    """
    items = stack.getItems()
    display = f"{stack.name}: "

    # display items top to bottom (reversed from internal storage)
    for item in reversed(items):
        shortName = shortenName(item)
        display += f"{color} {shortName:4} {RESET} "

    print(display)


def displayEverything(customerLine, ingredientsQueue, burgerStacks, reserveStack, customerStacks, priority):
    """
    Renders the complete system state to the terminal each step:
    customer and ingredient queues, all station and reserve stacks,
    all customer order stacks, and the current priority order.
    """
    print("\n" + "=" * 80)

    # render both queues at the top
    showQueue(customerLine, BLUE_BG)
    showQueue(ingredientsQueue, GREEN_BG)
    print()

    # render the 3 active station stacks
    showBurgerStack(burgerStacks[0], BROWN_BG)
    showBurgerStack(burgerStacks[1], ORANGE_BG)
    showBurgerStack(burgerStacks[2], RED_BG)
    print()

    # render the reserve stack
    showBurgerStack(reserveStack, PURPLE_BG)
    print()

    # render all 3 customer order stacks
    showCustomerStack(customerStacks[0], YELLOW_BG)
    showCustomerStack(customerStacks[1], DARK_PURPLE_BG)
    showCustomerStack(customerStacks[2], DARK_BLUE_BG)
    print()

    # build and display the current station priority order
    priorityNames = []
    for i in range(len(priority)):
        idx = priority[i]
        priorityNames.append(burgerStacks[idx].name)
    print(f"Priority: {priorityNames}")
    print("=" * 80)


def updatePriority(priority, burgerIdx):
    """
    Rotates a completed station to the back of the priority list.
    Ensures the station that just finished has the lowest priority next round.
    """
    priority.remove(burgerIdx)
    priority.append(burgerIdx)


def assignNewCustomer(burgerIdx, customerLine, burgerStacks, customerStacks, log):
    """
    Handles station turnover after an order is completed.
    Clears the station and its customer stack, then dequeues and
    assigns the next customer in line.
    Returns True if a new customer was assigned, False if the queue is empty.
    """
    custIdx = burgerStacks[burgerIdx].custStack

    # log the station and customer stack clearing
    log.append(
        f"{burgerStacks[burgerIdx].name} stack is ready to be cleared to serve {burgerStacks[burgerIdx].customerId}")
    log.append(f"{customerStacks[custIdx].name} stack is ready to be cleared after {burgerStacks[burgerIdx].name}")

    if not customerLine.isEmpty():
        # dequeue the next customer and load their order
        customer = customerLine.dequeue()
        customerId = customer['id']
        ingredients = customer['ingredients']

        # clear previous customer's data from both stacks
        burgerStacks[burgerIdx].clear()
        customerStacks[custIdx].clear()

        # populate the customer stack with the new order
        for ing in ingredients:
            customerStacks[custIdx].push(ing)

        burgerStacks[burgerIdx].customerId = customerId
        burgerStacks[burgerIdx].custStack = custIdx

        ingList = ", ".join(ingredients)
        log.append(
            f"A new customer {customerId} was dequeued and assigned to {customerStacks[custIdx].name} for {burgerStacks[burgerIdx].name} with ingredients [{ingList}]")

        return True
    else:
        log.append("There are no new customers in line")
        return False


def findMatchingBurger(ingredient, burgerStacks, customerStacks, priority):
    """
    Scans active stations in priority order to find one that needs the given ingredient.
    The highest-priority station that needs it gets it first.
    Returns the station index, or -1 if no match is found.
    """
    for burgerIdx in priority:
        custIdx = burgerStacks[burgerIdx].custStack

        customerWants = customerStacks[custIdx].getItems()
        burgerHas = burgerStacks[burgerIdx].getItems()

        # match if the station's customer wants it and the station doesn't already have it
        if ingredient in customerWants and ingredient not in burgerHas:
            return burgerIdx

    return -1


def checkReserveMatch(reserveStack, burgerStacks, customerStacks, priority):
    """
    Checks whether the top ingredient on the reserve stack is needed by any active station.
    Follows priority order when checking. Returns station index or -1 if no match.
    """
    if reserveStack.isEmpty():
        return -1

    topIngredient = reserveStack.peek()

    for burgerIdx in priority:
        custIdx = burgerStacks[burgerIdx].custStack
        customerWants = customerStacks[custIdx].getItems()
        burgerHas = burgerStacks[burgerIdx].getItems()

        if topIngredient in customerWants and topIngredient not in burgerHas:
            return burgerIdx

    return -1


def checkBurgerComplete(burgerIdx, burgerStacks, customerStacks):
    """
    Checks whether a station has collected every ingredient the customer ordered.
    Returns True if the order is fully assembled, False otherwise.
    """
    custIdx = burgerStacks[burgerIdx].custStack
    customerWants = customerStacks[custIdx].getItems()
    burgerHas = burgerStacks[burgerIdx].getItems()

    for ingredient in customerWants:
        if ingredient not in burgerHas:
            return False

    return True


def playOneStep(customerLine, ingredientsQueue, burgerStacks, reserveStack, customerStacks, priority, log, servedCount):
    """
    Processes one ingredient from the conveyor belt using the following routing logic:

    1. Peek at the front ingredient in the queue.
    2. If a station needs it (checked in priority order), dequeue and route it there.
    3. If no station needs it, check the reserve stack:
       - Reserve empty: move the ingredient to reserve.
       - Reserve top matches a station: pop reserve to that station, move front ingredient to reserve.
       - No match either way: move front ingredient to reserve.

    After any station completes an order, a new customer is assigned
    and the priority list is updated accordingly.
    """
    if ingredientsQueue.isEmpty():
        return False, servedCount

    frontIngredient = ingredientsQueue.peek()

    # check if any active station needs this ingredient
    burgerIdx = findMatchingBurger(frontIngredient, burgerStacks, customerStacks, priority)

    if burgerIdx != -1:
        # direct match found — dequeue and send to the station
        ing = ingredientsQueue.dequeue()
        burgerStacks[burgerIdx].push(ing)
        log.append(f"{ing} was dequeued and added to {burgerStacks[burgerIdx].name}")

        # check if this completed the order
        if burgerStacks[burgerIdx].isFull() or checkBurgerComplete(burgerIdx, burgerStacks, customerStacks):
            if assignNewCustomer(burgerIdx, customerLine, burgerStacks, customerStacks, log):
                servedCount += 1

            updatePriority(priority, burgerIdx)
            names = [burgerStacks[i].name for i in priority]
            log.append(f"Priority list is now {names}")

        return True, servedCount

    # no station needed it — route through the reserve
    if reserveStack.isEmpty():
        # reserve is empty, store the ingredient there for future use
        ing = ingredientsQueue.dequeue()
        reserveStack.push(ing)
        log.append(f"{ing} was dequeued and added to {reserveStack.name} (no burger needed it)")
        return True, servedCount

    # check if the reserve's top ingredient is now useful to any station
    burgerIdx = checkReserveMatch(reserveStack, burgerStacks, customerStacks, priority)

    if burgerIdx != -1:
        # reserve top matches a station — pop it over, then store the front ingredient
        ing = reserveStack.pop()
        burgerStacks[burgerIdx].push(ing)
        log.append(f"{ing} was popped from {reserveStack.name} and added to {burgerStacks[burgerIdx].name}")

        if burgerStacks[burgerIdx].isFull() or checkBurgerComplete(burgerIdx, burgerStacks, customerStacks):
            if assignNewCustomer(burgerIdx, customerLine, burgerStacks, customerStacks, log):
                servedCount += 1

            updatePriority(priority, burgerIdx)
            names = [burgerStacks[i].name for i in priority]
            log.append(f"Priority list is now {names}")

        return True, servedCount

    # no match found anywhere — move front ingredient to reserve
    ing = ingredientsQueue.dequeue()
    reserveStack.push(ing)
    log.append(f"{ing} was dequeued and added to {reserveStack.name} (still no burger needed it)")
    return True, servedCount


def saveLogFile(log):
    """
    Writes the complete session log to order_log.txt.
    One entry per line, recording every routing decision made during the session.
    """
    file = open("order_log.txt", 'w')
    for line in log:
        file.write(line + '\n')
    file.close()
    print("\nSession log saved to order_log.txt")


def main():
    """
    Entry point. Reads input files, initializes all data structures,
    assigns the first 3 customers to their stations, then runs the
    step-by-step simulation loop. The user advances with Y or exits with N.
    A full session log is saved to order_log.txt on exit.
    """
    print("Restaurant Queue Management System")

    customers = readCustomerFile("customer_orders.txt")
    ingredients = readIngredientsFile("ingredients.txt")

    if len(customers) == 0 or len(ingredients) == 0:
        print("Error: Could not read input files. Ensure customer_orders.txt and ingredients.txt are present.")
        return

    customerLine = makeCustomerQueue(customers)
    ingredientsQueue = makeIngredientsQueue(ingredients)
    burgerStacks = makeBurgerStacks()
    reserveStack = makeReserveStack()
    customerStacks = makeCustomerStacks()

    # default priority: Burg1 > Burg2 > Burg3, rotates as orders complete
    priority = [0, 1, 2]
    log = []

    assignFirstThreeCustomers(customerLine, burgerStacks, customerStacks, log)

    priorityNames = [burgerStacks[i].name for i in priority]
    log.append(f"The priority list is then {priorityNames}")

    customersServed = 0

    displayEverything(customerLine, ingredientsQueue, burgerStacks, reserveStack, customerStacks, priority)

    gameRunning = True
    while gameRunning:
        userInput = input("\nContinue (Y/N)? ").strip().upper()

        if userInput == 'Y':
            gameRunning, customersServed = playOneStep(
                customerLine, ingredientsQueue, burgerStacks,
                reserveStack, customerStacks, priority, log, customersServed
            )

            displayEverything(customerLine, ingredientsQueue, burgerStacks, reserveStack, customerStacks, priority)

            if customersServed >= 5:
                print(f"\nSimulation complete. {customersServed} customers served successfully.")
                gameRunning = False
            elif not gameRunning:
                print("\nConveyor belt empty. Session ended.")

        elif userInput == 'N':
            print("Session stopped.")
            gameRunning = False

        else:
            print("Invalid input. Please enter Y or N.")

    saveLogFile(log)
    print("Session complete.")


if __name__ == "__main__":
    main()
