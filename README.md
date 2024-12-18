opsis_proj1_mhdAlhabeb_alshalah_2221251360
calculator.c
addition.c
subtraction.c
multiplication.c
division.c
saver.c
results.txt
```
// addition.c
#include <stdio.h>
int main() {
    // Placeholder code
    printf("Addition process started\n");
    return 0;
}
```

```
    gcc calculator.c -o calculator
    gcc addition.c -o addition
    gcc subtraction.c -o subtraction
    gcc multiplication.c -o multiplication
    gcc division.c -o division
    gcc saver.c -o saver
```

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

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

        // Communication Pipe
        int pipefd[2];
        if (pipe(pipefd) == -1) {
            perror("Pipe failed");
            exit(EXIT_FAILURE);
        }

        pid_t pid = fork();

        if (pid < 0) {
            perror("Fork failed");
            exit(EXIT_FAILURE);
        } else if (pid == 0) {
            // Child process
            close(pipefd[1]); // Close the write end of the pipe
            dup2(pipefd[0], STDIN_FILENO); // Redirect the read end of the pipe to stdin
            close(pipefd[0]); // Close the original read end

            // Execute the corresponding file
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
                case 5:
                    execl("./saver", "saver", NULL);
                    break;
            }
            perror("Exec failed");
            exit(EXIT_FAILURE);
        } else {
            // Parent process
            close(pipefd[0]); // Close the read end of the pipe
            if (option >= 1 && option <= 4) {
                dprintf(pipefd[1], "%d %d\n", operand1, operand2); // Write operands to the pipe
            }
            close(pipefd[1]); // Close the write end of the pipe

            int status;
            waitpid(pid, &status, 0); // Wait for the child to complete
            if (WIFEXITED(status)) {
                printf("Child process exited with status %d.\n", WEXITSTATUS(status));
            } else {
                printf("Child process did not exit successfully.\n");
            }
        }
    }

    return 0;
}


```
