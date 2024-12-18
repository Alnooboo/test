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

        int pipefd[2];
        if (pipe(pipefd) == -1) {
            perror("Pipe failed");
            exit(EXIT_FAILURE);
        }

        pid_t pid = fork();

        if (pid == 0) {
            // Child process
            close(pipefd[1]); // Close the write end
            dup2(pipefd[0], STDIN_FILENO); // Redirect stdin to the read end of the pipe
            close(pipefd[0]);

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
            close(pipefd[0]); // Close the read end

            if (option >= 1 && option <= 4) {
                // Write operands to the pipe
                dprintf(pipefd[1], "%d %d\n", operand1, operand2);
            }
            close(pipefd[1]); // Close the write end

            char result[100] = {0};
            int status;
            waitpid(pid, &status, 0);

            if (WIFEXITED(status) && WEXITSTATUS(status) == 0) {
                // Read result from the pipe
                printf("Result: %s", result);

                // Call saver.c with the result
                pid_t saver_pid = fork();
                if (saver_pid == 0) {
                    execl("./saver", "saver", result, NULL);
                    perror("Exec failed");
                    exit(EXIT_FAILURE);
                } else {
                    waitpid(saver_pid, &status, 0);
                }
            } else {
                printf("Child process encountered an error.\n");
            }
        }
    }

    return 0;
}

```
```
#include <stdlib.h>
#include <stdio.h>

int main() { 
    int a, b;

    // Read operands from stdin (provided by the parent process via the pipe)
    if (scanf("%d %d", &a, &b) != 2) {
        fprintf(stderr, "Failed to read operands\n");
        return 1;
    }

    // Perform the calculation
    int result = a + b;
    printf("%d\n", result); // Write the result to stdout (read by the parent)

    return 0; 
}



```
