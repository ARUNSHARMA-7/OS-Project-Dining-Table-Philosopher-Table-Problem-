# OS-Project-Dining-Table-Philosopher-Table-Problem-
OS project of 4th sem students. 

import threading
import time
import random

# -------------------------------
# CONFIGURATION
# -------------------------------
N = 5
CYCLES = 3  # how many times each philosopher eats

THINKING, HUNGRY, EATING = "THINKING", "HUNGRY", "EATING"

# -------------------------------
# SHARED RESOURCES
# -------------------------------
chopsticks = [threading.Lock() for _ in range(N)]
butler = threading.Semaphore(N - 1)
states = [THINKING] * N
print_lock = threading.Lock()


# -------------------------------
# UTILITIES
# -------------------------------
def log(msg):
    with print_lock:
        print(msg)


def show_states():
    with print_lock:
        print("States:", states)
        print("-" * 40)


# -------------------------------
# ACTIONS
# -------------------------------
def think(i):
    states[i] = THINKING
    log(f"P{i} THINKING")
    time.sleep(random.uniform(0.5, 1.5))


def take_chopsticks(i):
    butler.acquire()

    states[i] = HUNGRY
    log(f"P{i} HUNGRY")

    left, right = i, (i + 1) % N

    chopsticks[left].acquire()
    chopsticks[right].acquire()

    states[i] = EATING
    log(f"P{i} EATING")


def eat(i):
    time.sleep(random.uniform(0.5, 1))


def put_chopsticks(i):
    left, right = i, (i + 1) % N

    chopsticks[left].release()
    chopsticks[right].release()

    states[i] = THINKING
    log(f"P{i} DONE -> THINKING")

    butler.release()


# -------------------------------
# THREAD FUNCTION
# -------------------------------
def philosopher(i):
    for _ in range(CYCLES):
        think(i)
        take_chopsticks(i)
        eat(i)
        put_chopsticks(i)


# -------------------------------
# MAIN
# -------------------------------
def main():
    threads = [threading.Thread(target=philosopher, args=(i,)) for i in range(N)]

    for t in threads:
        t.start()

    for t in threads:
        t.join()

    print("\nSimulation finished.")


if __name__ == "__main__":
    main()
