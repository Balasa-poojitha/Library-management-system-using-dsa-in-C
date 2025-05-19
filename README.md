#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Define maximum title length
#define MAX_TITLE_LEN 100

// Define the structure for a Book
struct Book {
    int id;
    char title[MAX_TITLE_LEN];
    int available;  // 1 = available, 0 = issued
};

// Function Prototypes
void addBook();
void displayBooks();
void issueBook();
void returnBook();
void deleteBook();
void searchBook();
int bookExists(int id);
void clearInputBuffer();
void displayMenu();
void pressEnterToContinue();


// Main Function
int main() {
    int choice;

    // Main Menu Loop
    do {
        displayMenu();

        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                addBook();
                break;
            case 2:
                displayBooks();
                break;
            case 3:
                issueBook();
                break;
            case 4:
                returnBook();
                break;
            case 5:
                deleteBook();
                break;
            case 6:
                searchBook();
                break;
            case 7:
                printf("Exiting program. Goodbye!\n");
                break;
            default:
                printf("Invalid choice. Please select a valid option.\n");
        }

        if (choice != 7) {
            pressEnterToContinue();
        }

    } while (choice != 7);

    return 0;
}

// Display the main menu
void displayMenu() {
    printf("\n==============================\n");
    printf("      Library Management      \n");
    printf("==============================\n");
    printf("1. Add Book\n");
    printf("2. Display All Books\n");
    printf("3. Issue Book\n");
    printf("4. Return Book\n");
    printf("5. Delete Book\n");
    printf("6. Search Book\n");
    printf("7. Exit\n");
    printf("==============================\n");
}

// Add a new book to the library
void addBook() {
    FILE *fp = fopen("library.dat", "ab");
    struct Book b;

    if (!fp) {
        printf("Error opening file.\n");
        return;
    }

    printf("\nEnter Book ID: ");
    scanf("%d", &b.id);

    if (bookExists(b.id)) {
        printf("A book with this ID already exists.\n");
        fclose(fp);
        return;
    }

    printf("Enter Book Title: ");
    clearInputBuffer();
    fgets(b.title, MAX_TITLE_LEN, stdin);
    b.title[strcspn(b.title, "\n")] = 0;

    b.available = 1;

    fwrite(&b, sizeof(b), 1, fp);
    fclose(fp);

    printf("Book added successfully.\n");
}

// Display all books
void displayBooks() {
    FILE *fp = fopen("library.dat", "rb");
    struct Book b;
    int count = 0;

    if (!fp) {
        printf("Error opening file.\n");
        return;
    }

    printf("\nAll Books in Library:\n");
    printf("----------------------------\n");

    while (fread(&b, sizeof(b), 1, fp)) {
        printf("ID: %d\nTitle: %s\nStatus: %s\n\n", b.id, b.title, b.available ? "Available" : "Issued");
        count++;
    }

    if (count == 0) {
        printf("No books found.\n");
    }

    fclose(fp);
}

// Issue a book
void issueBook() {
    FILE *fp = fopen("library.dat", "rb+");
    struct Book b;
    int id, found = 0;

    if (!fp) {
        printf("Error opening file.\n");
        return;
    }

    printf("\nEnter Book ID to issue: ");
    scanf("%d", &id);

    while (fread(&b, sizeof(b), 1, fp)) {
        if (b.id == id) {
            if (b.available) {
                b.available = 0;
                fseek(fp, -sizeof(b), SEEK_CUR);
                fwrite(&b, sizeof(b), 1, fp);
                printf("Book issued successfully.\n");
            } else {
                printf("Book is already issued.\n");
            }
            found = 1;
            break;
        }
    }

    if (!found) {
        printf("Book with ID %d not found.\n", id);
    }

    fclose(fp);
}

// Return a book
void returnBook() {
    FILE *fp = fopen("library.dat", "rb+");
    struct Book b;
    int id, found = 0;

    if (!fp) {
        printf("Error opening file.\n");
        return;
    }

    printf("\nEnter Book ID to return: ");
    scanf("%d", &id);

    while (fread(&b, sizeof(b), 1, fp)) {
        if (b.id == id) {
            if (!b.available) {
                b.available = 1;
                fseek(fp, -sizeof(b), SEEK_CUR);
                fwrite(&b, sizeof(b), 1, fp);
                printf("Book returned successfully.\n");
            } else {
                printf("This book was not issued.\n");
            }
            found = 1;
            break;
        }
    }

    if (!found) {
        printf("Book with ID %d not found.\n", id);
    }

    fclose(fp);
}

// Delete a book
void deleteBook() {
    FILE *fp = fopen("library.dat", "rb");
    FILE *temp = fopen("temp.dat", "wb");
    struct Book b;
    int id, found = 0;

    if (!fp || !temp) {
        printf("Error opening file.\n");
        return;
    }

    printf("\nEnter Book ID to delete: ");
    scanf("%d", &id);

    while (fread(&b, sizeof(b), 1, fp)) {
        if (b.id == id) {
            found = 1;
            continue;
        }
        fwrite(&b, sizeof(b), 1, temp);
    }

    fclose(fp);
    fclose(temp);

    remove("library.dat");
    rename("temp.dat", "library.dat");

    if (found) {
        printf("Book deleted successfully.\n");
    } else {
        printf("Book not found.\n");
    }
}

// Search for a book by ID
void searchBook() {
    FILE *fp = fopen("library.dat", "rb");
    struct Book b;
    int id, found = 0;

    if (!fp) {
        printf("Error opening file.\n");
        return;
    }

    printf("\nEnter Book ID to search: ");
    scanf("%d", &id);

    while (fread(&b, sizeof(b), 1, fp)) {
        if (b.id == id) {
            printf("Book Found:\n");
            printf("ID: %d\nTitle: %s\nStatus: %s\n", b.id, b.title, b.available ? "Available" : "Issued");
            found = 1;
            break;
        }
    }

    if (!found) {
        printf("Book with ID %d not found.\n", id);
    }

    fclose(fp);
}

// Check if a book with the given ID already exists
int bookExists(int id) {
    FILE *fp = fopen("library.dat", "rb");
    struct Book b;

    if (!fp) return 0;

    while (fread(&b, sizeof(b), 1, fp)) {
        if (b.id == id) {
            fclose(fp);
            return 1;
        }
    }

    fclose(fp);
    return 0;
}

// Clear input buffer
void clearInputBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

// Wait for user input before proceeding
void pressEnterToContinue() {
    printf("\nPress Enter to continue...");
    clearInputBuffer();
    getchar();
}
