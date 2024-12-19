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
