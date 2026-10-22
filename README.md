# LM2 — Mutexes and Condition Variables

C++ examples of thread synchronization using `std::thread`, `std::mutex`, and `std::condition_variable`. Originally written as a class assignment, this repository is shared as a learning resource for exploring shared state, waiting on conditions, and coordinating work between threads.

## Examples

### Part 1: Concurrent bank transactions

[`p1.cpp`](p1.cpp) simulates a bank account shared by 100 threads:

- The account starts with **$150.25**.
- 50 threads each deposit **$16**.
- 50 threads each withdraw **$18**.
- A mutex protects the balance. Withdrawals wait on a condition variable when funds are insufficient, and deposits notify waiting threads.

The expected final balance is **$50.25** (`150.25 + 800 - 900`).

### Part 2: Priority withdrawals

[`p2.cpp`](p2.cpp) extends the bank account example with priority withdrawals:

- The account starts with **$300**.
- 25 threads each make a priority withdrawal of **$16**.
- 25 threads each make a normal withdrawal of **$18**.
- 50 threads each deposit **$18**.

Normal withdrawals wait while any priority withdrawals are pending, as well as when funds are insufficient. Priority withdrawals notify waiting threads after completing. Priority applies to requests registered under the mutex; thread creation order does not guarantee execution order.

The expected final balance is **$350** (`300 - 400 - 450 + 900`).

### Part 3: Alternating shuffle and sort

[`p3.cpp`](p3.cpp) coordinates two threads working on the same array:

1. One thread shuffles the values.
2. The other waits for the shuffle, sorts the array using bubble sort, and prints the result.
3. The threads repeat this handoff for **five cycles**.

A shared `shuffled` flag, protected by a mutex, controls whose turn it is. A condition variable lets each thread wait for its turn.

Each sorted result should be:

```text
2 4 5 7 35 68 79 90
```

The program also prints `Is the array sorted? : 1` after each sort (`1` means true).

## Build and run

You need Git and a C++ compiler with **C++11 or later** and threading support, such as GCC or Clang. No third-party libraries are required.

The following commands work in a macOS or Linux terminal with a suitable compiler installed. On Windows, use a compatible environment such as WSL.

```sh
git clone https://github.com/bhasan26/LM2.git
cd LM2

c++ -std=c++11 -pthread main.cpp p1.cpp p2.cpp p3.cpp -o lm2
./lm2
```

You can replace `c++` with `g++` or `clang++`. Compile all four source files together: `main.cpp` provides the entry point and runs Parts 1, 2, and 3 in sequence.

The program takes no input or command-line arguments. It prints transaction logs, the final account balances, and the shuffle/sort results. Thread scheduling varies between runs, so the transaction log order can change.

To save the output for inspection:

```sh
./lm2 > output.txt
```

## Repository contents

| File | Purpose |
| --- | --- |
| [`main.cpp`](main.cpp) | Runs all three examples in sequence. |
| [`p1.cpp`](p1.cpp) | Bank deposits and withdrawals with synchronized balance updates. |
| [`p2.cpp`](p2.cpp) | Bank transactions with pending priority withdrawal tracking. |
| [`p3.cpp`](p3.cpp) | Two threads alternating between shuffling and sorting an array. |
| [`part1.png`](part1.png) | Supporting image for Part 1. |

## Learning notes

- Condition-variable waits release the mutex while waiting and reacquire it before continuing. The `while` loops recheck the condition after every wakeup.
- The examples use fixed transaction counts and amounts. If you change them so withdrawals exceed available funds, waiting threads may never finish; there is no timeout or cancellation mechanism.
- These are classroom demonstrations. The bank examples use `float` for money and omit input validation; they are not suitable for real financial transactions.
- Part 3 uses `rand()` without an explicit seed and a simple shuffle algorithm. It demonstrates synchronization rather than statistically uniform shuffling.

## Acknowledgments

The source comments credit in-class examples for synchronization patterns and reference [these C++ shuffle notes](https://documents.uow.edu.au/~lukes/textbook/notes-cpp/misc/random-shuffle.html) in Part 3.

## License

This project is licensed under the [MIT License](LICENSE).
