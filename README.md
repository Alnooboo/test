opsis_proj1_mhdAlhabeb_alshalah_2221251360
calculator.c
addition.c
subtraction.c
multiplication.c
division.c
saver.c
results.txt

```
gcc calculator.c -o calculator
```

```
gcc addition.c -o addition
```

```
gcc subtraction.c -o subtraction
```

```
gcc multiplication.c -o multiplication
```

```
gcc division.c -o division
```
```
gcc saver.c -o saver
```

```
gcc history.c -o history
```

```
if (option >= 1 && option <= 4) {
    printf("Enter the first number: ");
    if (scanf("%d", &operand1) != 1) {
        printf("Invalid input. Please enter a number.\n");
        while (getchar() != '\n');
        continue;
    }

    printf("Enter the second number: ");
    if (scanf("%d", &operand2) != 1) {
        printf("Invalid input. Please enter a number.\n");
        while (getchar() != '\n');
        continue;
    }

    // Send operands to the selected operation process
    snprintf(buffer, sizeof(buffer), "%d %d", operand1, operand2);
    write(pipes[option - 1][1], buffer, strlen(buffer) + 1);

    // Read result from the pipe (blocking until result is available)
    char result[100];
    ssize_t bytes_read = read(pipes[option - 1][0], result, sizeof(result) - 1);
    if (bytes_read > 0) {
        result[bytes_read] = '\0'; // Null-terminate the result string
        printf("Result: %s\n", result);
    } else {
        printf("Error reading result from operation process.\n");
    }
}

```

```
#include <stdio.h>
#include <stdlib.h>

int main() {
    int operand1, operand2, result;

    while (scanf("%d %d", &operand1, &operand2) == 2) {
        result = operand1 + operand2;

        // Call saver.c to save the result
        pid_t saver_pid = fork();
        if (saver_pid == 0) {
            char result_str[50];
            snprintf(result_str, sizeof(result_str), "%d", result);
            execl("./saver", "saver", result_str, NULL);
            perror("Exec failed for saver");
            exit(EXIT_FAILURE);
        }

        wait(NULL); // Wait for saver to complete

        // Send result back to calculator.c
        printf("%d\n", result);
        fflush(stdout); // Flush output to ensure it's sent immediately
    }

    return 0;
}

```

fix:
```
char result[100];
            ssize_t bytes_read = read(pipes[option - 1][0], result, sizeof(result) - 1);
            if (bytes_read > 0) {
                result[bytes_read] = '\0'; // Null-terminate the string
                printf("Result: %s\n", result);
            } else {
                printf("No data received from operation.\n");
            }
```
```
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <string.h>

int main() {
    int pipes[4][2]; // Four pipes: one for each operation
    pid_t pids[4];   // PIDs of the operation child processes
    char *operations[] = {"addition", "subtraction", "multiplication", "division"};

    // Create pipes and fork child processes
    for (int i = 0; i < 4; i++) {
        if (pipe(pipes[i]) == -1) {
            perror("Pipe creation failed");
            exit(EXIT_FAILURE);
        }

        pids[i] = fork();
        if (pids[i] < 0) {
            perror("Fork failed");
            exit(EXIT_FAILURE);
        } else if (pids[i] == 0) {
            // Child process for operation
            close(pipes[i][0]); // Close read end
            dup2(pipes[i][1], STDOUT_FILENO); // Redirect stdout to write end of pipe
            close(pipes[i][1]); // Close original write end after redirection
            execl(operations[i], operations[i], NULL); // Execute operation file
            perror("Exec failed");
            exit(EXIT_FAILURE);
        }
        close(pipes[i][1]); // Close write end in parent
    }

    int option = 0;
    int operand1, operand2;
    char buffer[100];

    while (1) {
        printf("\nCalculator:\n");
        printf("1- addition\n");
        printf("2- subtraction\n");
        printf("3- multiplication\n");
        printf("4- division\n");
        printf("5- history\n");
        printf("6- exit\n");
        printf("~ Please choose an option: ");

        if (scanf("%d", &option) != 1 || option < 1 || option > 6) {
            printf("Invalid input. Please enter a number between 1 and 6.\n");
            while (getchar() != '\n'); // Clear invalid input
            continue;
        }

        if (option == 6) {
            printf("Exiting the program. Goodbye!\n");
            // Kill all child processes
            for (int i = 0; i < 4; i++) {
                kill(pids[i], SIGTERM); // Terminate operation child processes
            }
            break; // Exit the loop
        }

        if (option == 5) {
            pid_t history_pid = fork();
            if (history_pid == 0) {
                execl("./history", "history", NULL);
                perror("Exec failed for history");
                exit(EXIT_FAILURE);
            }
            wait(NULL); // Wait for history process to finish
            continue;
        }

        if (option >= 1 && option <= 4) {
            printf("Enter the first number: ");
            if (scanf("%d", &operand1) != 1) {
                printf("Invalid input. Please enter a number.\n");
                while (getchar() != '\n');
                continue;
            }

            printf("Enter the second number: ");
            if (scanf("%d", &operand2) != 1) {
                printf("Invalid input. Please enter a number.\n");
                while (getchar() != '\n');
                continue;
            }

            // Send operands to the selected operation process
            snprintf(buffer, sizeof(buffer), "%d %d", operand1, operand2);
            write(pipes[option - 1][1], buffer, strlen(buffer) + 1);

            // Read result from the pipe
            char result[100];
            ssize_t bytes_read = read(pipes[option - 1][0], result, sizeof(result) - 1);
            if (bytes_read > 0) {
                result[bytes_read] = '\0'; // Null-terminate the string
                printf("Result: %s\n", result);
            } else {
                printf("No data received from operation.\n");
            }
        }
    }

    return 0;
}
```

