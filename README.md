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
            close(pipes[i][1]); // Close write end
            dup2(pipes[i][0], STDIN_FILENO); // Redirect stdin to read end of pipe
            close(pipes[i][0]); // Close read end after duplication
            execl(operations[i], operations[i], NULL);
            perror("Exec failed");
            exit(EXIT_FAILURE);
        }
        close(pipes[i][0]); // Close read end in parent
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
            while (getchar() != '\n');
            continue;
        }

        if (option == 6) {
            printf("Exiting the program. Goodbye!\n");
            for (int i = 0; i < 4; i++) kill(pids[i], SIGTERM); // Terminate child processes
            break;
        }

        if (option == 5) {
            pid_t history_pid = fork();
            if (history_pid == 0) {
                execl("./history", "history", NULL);
                perror("Exec failed for history");
                exit(EXIT_FAILURE);
            }
            wait(NULL);
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

            // Read result from the operation process
            int result_pipe[2];
            if (pipe(result_pipe) == -1) {
                perror("Result pipe creation failed");
                exit(EXIT_FAILURE);
            }

            pid_t saver_pid = fork();
            if (saver_pid == 0) {
                // Saver process
                char result[100];
                read(result_pipe[0], result, sizeof(result));
                close(result_pipe[0]);
                execl("./saver", "saver", result, NULL);
                perror("Exec failed for saver");
                exit(EXIT_FAILURE);
            }

            close(result_pipe[1]); // Close write end in parent
            wait(NULL);            // Wait for saver process to complete
        }
    }

    return 0;
}

```

```
#include <stdio.h>
#include <stdlib.h>

int main() {
    int operand1, operand2, result;

    while (scanf("%d %d", &operand1, &operand2) == 2) {
        result = operand1 + operand2;

        // Call saver.c
        pid_t saver_pid = fork();
        if (saver_pid == 0) {
            char result_str[50];
            snprintf(result_str, sizeof(result_str), "%d", result);
            execl("./saver", "saver", result_str, NULL);
            perror("Exec failed for saver");
            exit(EXIT_FAILURE);
        }

        wait(NULL); // Wait for saver to complete
        printf("%d\n", result); // Send result back to calculator.c
    }

    return 0;
}
```

