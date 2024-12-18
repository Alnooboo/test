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

int main() {
    int option = 0; // Variable to store user input
    int operand1, operand2; // Operands for calculations

    while (1) {
        // Display the menu
        printf("\nCalculator:\n");
        printf("1- addition\n");
        printf("2- subtraction\n");
        printf("3- multiplication\n");
        printf("4- division\n");
        printf("5- history\n");
        printf("6- exit\n");
        printf("~ Please choose an option: ");

        if (scanf("%d", &option) != 1) {
            printf("Invalid input. Please enter a number.\n");
            while (getchar() != '\n');
            continue;
        }

        if (option < 1 || option > 6) {
            printf("Invalid option. Please choose a number between 1 and 6.\n");
            continue;
        }

        if (option == 6) {
            printf("Exiting the program. Goodbye!\n");
            break;
        }

        if (option == 5) {
            // Call history.c to display the history
            pid_t history_pid = fork();
            if (history_pid < 0) {
                perror("Fork for history failed");
                exit(EXIT_FAILURE);
            } else if (history_pid == 0) {
                execl("./history", "history", NULL);
                perror("Exec failed for history");
                exit(EXIT_FAILURE);
            } else {
                // Parent process waits for history to complete
                int status;
                waitpid(history_pid, &status, 0);
                if (WIFEXITED(status)) {
                    printf("History process exited with status %d.\n", WEXITSTATUS(status));
                } else {
                    printf("History process did not exit successfully.\n");
                }
            }
            continue;
        }

        if (option >= 1 && option <= 4) {
            // Take operands input
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
        }

        // The rest of the code for operations (1–4) and saver remains unchanged
    }

    return 0;
}


```
```
#include <stdio.h>
#include <stdlib.h>

int main() {
    FILE *file = fopen("results.txt", "r");
    if (!file) {
        perror("Failed to open results file");
        return 1;
    }

    char line[256];
    printf("Calculation History:\n");
    printf("=====================\n");
    while (fgets(line, sizeof(line), file)) {
        printf("%s", line); // Print each line from the file
    }
    fclose(file);

    return 0;
}

```
