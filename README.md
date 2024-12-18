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

int main() {
    int option = 0; // Variable to store user input

    while (1) { // Infinite loop to keep the program running until "exit"
        // Display the menu
        printf("\nCalculator:\n");
        printf("▪ 1- addition\n");
        printf("▪ 2- subtraction\n");
        printf("▪ 3- multiplication\n");
        printf("▪ 4- division\n");
        printf("▪ 5- history\n");
        printf("▪ 6- exit\n");
        printf("~ Please choose an option: ");

        // Take user input
        if (scanf("%d", &option) != 1) { // Check for valid integer input
            printf("Invalid input. Please enter a number.\n");
            // Clear input buffer
            while (getchar() != '\n');
            continue;
        }

        // Validate the input
        if (option < 1 || option > 6) {
            printf("Invalid option. Please choose a number between 1 and 6.\n");
            continue;
        }

        // Print the selected option
        printf("You selected option %d.\n", option);

        // Exit if the user chooses option 6
        if (option == 6) {
            printf("Exiting the program. Goodbye!\n");
            break;
        }
    }

    return 0;
}

```
