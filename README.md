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
