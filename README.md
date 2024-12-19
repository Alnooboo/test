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
    int option = 0;
    double operand1, operand2;

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
            break;
        }
        
        if (option == 5) {
            pid_t history_pid = fork();
            if (history_pid < 0) {
                perror("Fork for history failed");
                exit(EXIT_FAILURE);
            } else if (history_pid == 0) {
                execl("./history", "history", NULL);
                perror("Exec failed for history");
                exit(EXIT_FAILURE);
            } else {
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
            printf("Enter the first number: ");
            if (scanf("%lf", &operand1) != 1) {
                printf("Invalid input. Please enter a number.\n");
                while (getchar() != '\n');
                continue;
            }

            printf("Enter the second number: ");
            if (scanf("%lf", &operand2) != 1) {
                printf("Invalid input. Please enter a number.\n");
                while (getchar() != '\n');
                continue;
            }
        }

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
            close(pipefd[0]);
            dup2(pipefd[1], STDOUT_FILENO);
            close(pipefd[1]);

            char op1[20], op2[20];
            snprintf(op1, sizeof(op1), "%f", operand1);
            snprintf(op2, sizeof(op2), "%f", operand2);

            switch (option) {
                case 1:
                    execl("./addition", "addition", op1, op2, NULL);
                    break;
                case 2:
                    execl("./subtraction", "subtraction", op1, op2, NULL);
                    break;
                case 3:
                    execl("./multiplication", "multiplication", op1, op2, NULL);
                    break;
                case 4:
                    execl("./division", "division", op1, op2, NULL);
                    break;
            }
            perror("Exec failed");
            exit(EXIT_FAILURE);
        } else {
            close(pipefd[1]);

            char result[100];
            read(pipefd[0], result, sizeof(result));
            close(pipefd[0]);

            if (option == 4 && operand2 == 0) {
                printf("Error: Division by zero is not allowed. Result not saved.\n");
                continue;
            }

            if (strncmp(result, "NaN", 3) != 0) { // Only save valid results
                pid_t saver_pid = fork();
                if (saver_pid < 0) {
                    perror("Fork for saver failed");
                    exit(EXIT_FAILURE);
                } else if (saver_pid == 0) {
                    execl("./saver", "saver", result, NULL);
                    perror("Exec failed for saver");
                    exit(EXIT_FAILURE);
                } else {
                    int status;
                    waitpid(saver_pid, &status, 0);
                    if (WIFEXITED(status)) {
                        printf("Saver process exited with status %d.\n", WEXITSTATUS(status));
                    } else {
                        printf("Saver process did not exit successfully.\n");
                    }
                }
            }

            int status;
            waitpid(pid, &status, 0);
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
```
if (strncmp(result, "NaN", 3) != 0) { // Check if result is valid
    pid_t saver_pid = fork();
    if (saver_pid < 0) {
        perror("Fork for saver failed");
        exit(EXIT_FAILURE);
    } else if (saver_pid == 0) {
        execl("./saver", "saver", result, NULL);
        perror("Exec failed for saver");
        exit(EXIT_FAILURE);
    } else {
        int status;
        waitpid(saver_pid, &status, 0);
        if (WIFEXITED(status)) {
            printf("Saver process exited with status %d.\n", WEXITSTATUS(status));
        } else {
            printf("Saver process did not exit successfully.\n");
        }
    }
}


```

```
#include <stdlib.h>
#include <stdio.h>

int main(int argc, char *argv[]) { 
    if (argc != 3) {
        fprintf(stderr, "Usage: %s <operand1> <operand2>\n", argv[0]);
        return 1;
    }

    double a = atof(argv[1]);
    double b = atof(argv[2]);
    if (b == 0) {
        fprintf(stderr, "Error: Division by zero is not allowed.\n");
        printf("NaN\n"); // Send NaN to indicate an invalid result
        return 1;
    }

    double result = a / b;
    printf("%f\n", result); // Send result to stdout
    fprintf(stderr, "Result: %f\n", result);

    return 0;
}

```
