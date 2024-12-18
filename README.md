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
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    int option = 0;
    int operand1, operand2;

    while (1) {
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
            while (getchar() != '\n'); // Clear invalid input
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

        if (option >= 1 && option <= 4) {
            printf("Enter the first number: ");
            if (scanf("%d", &operand1) != 1) {
                printf("Invalid input. Please enter a number.\n");
                while (getchar() != '\n'); // Clear invalid input
                continue;
            }

            printf("Enter the second number: ");
            if (scanf("%d", &operand2) != 1) {
                printf("Invalid input. Please enter a number.\n");
                while (getchar() != '\n'); // Clear invalid input
                continue;
            }
        }

        int parent_to_child[2], child_to_parent[2];
        if (pipe(parent_to_child) == -1 || pipe(child_to_parent) == -1) {
            perror("Pipe failed");
            exit(EXIT_FAILURE);
        }

        pid_t pid = fork();

        if (pid < 0) {
            perror("Fork failed");
            exit(EXIT_FAILURE);
        } else if (pid == 0) {
            // Child process
            close(parent_to_child[1]); // Close write end of parent-to-child pipe
            close(child_to_parent[0]); // Close read end of child-to-parent pipe

            // Redirect parent-to-child read end to stdin
            dup2(parent_to_child[0], STDIN_FILENO);
            close(parent_to_child[0]);

            // Redirect child-to-parent write end to stdout
            dup2(child_to_parent[1], STDOUT_FILENO);
            close(child_to_parent[1]);

            // Execute the operation
            switch (option) {
                case 1:
                    execl("./addition", "addition", NULL);
                    break;
                case 2:
                    execl("./subtraction", "subtraction", NULL);
                    break;
                case 3:
                    execl("./multiplication", "multiplication", NULL);
                    break;
                case 4:
                    execl("./division", "division", NULL);
                    break;
            }

            perror("Exec failed");
            exit(EXIT_FAILURE);
        } else {
            // Parent process
            close(parent_to_child[0]); // Close read end of parent-to-child pipe
            close(child_to_parent[1]); // Close write end of child-to-parent pipe

            // Write operands to the parent-to-child pipe
            dprintf(parent_to_child[1], "%d %d\n", operand1, operand2);
            close(parent_to_child[1]); // Close write end after sending

            // Read result from the child-to-parent pipe
            char result[100];
            ssize_t bytesRead = read(child_to_parent[0], result, sizeof(result) - 1);
            close(child_to_parent[0]); // Close read end after receiving

            if (bytesRead > 0) {
                result[bytesRead] = '\0'; // Null-terminate the string
                printf("The result after the operation is: %s\n", result);
            } else {
                printf("Error: Failed to read result from operation.\n");
            }

            // Wait for the child process to finish
            int status;
            waitpid(pid, &status, 0);
        }
    }

    return 0;
}

```
```

```
