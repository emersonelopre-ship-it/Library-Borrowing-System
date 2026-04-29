#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_BOOKS 50

struct Book {
    int id;
    char title[100];
    char author[100];
    int available;
};

struct Book catalog[MAX_BOOKS];
int bookCount = 0;

// Borrower linked list
struct Borrower {
    int bookId;
    char name[100];
    struct Borrower *next;
};
struct Borrower *borrowerHead = NULL;

// Return stack
struct Stack {
    int bookId;
    struct Stack *next;
};
struct Stack *returnStack = NULL;

// Reservation linked list
struct Reservation {
    int bookId;
    char name[100];
    struct Reservation *next;
};
struct Reservation *reservationHead = NULL;

// Helper: read string safely - This block we use Copilot AI
void readLine(char *buffer, int size) {
    fgets(buffer, size, stdin);
    buffer[strcspn(buffer, "\n")] = '\0';
}

// Add book
void addBook() {
    if (bookCount >= MAX_BOOKS) {
        printf("Catalog full!\n");
        return;
    }
    struct Book b;
    b.id = bookCount + 1;
    printf("Enter title: ");
    readLine(b.title, sizeof(b.title));
    printf("Enter author: ");
    readLine(b.author, sizeof(b.author));
    b.available = 1;
    catalog[bookCount++] = b;
    printf("Book added!\n");
}

// View books
void viewBooks() {
    if (bookCount == 0) {
        printf("No books in catalog.\n");
        return;
    }
    printf("\n Catalog \n");
    for (int i = 0; i < bookCount; i++) {
        printf("ID: %d | %-20s by %-20s | %s\n",
               catalog[i].id,
               catalog[i].title,
               catalog[i].author,
               catalog[i].available ? "Available" : "Borrowed");
    }
}

// Search book
void searchCatalog() {
    char title[100];
    printf("Enter title: ");
    readLine(title, sizeof(title));
    for (int i = 0; i < bookCount; i++) {
        if (strcmp(catalog[i].title, title) == 0) {
            printf("Found: ID=%d, Author=%s, Status=%s\n",
                   catalog[i].id,
                   catalog[i].author,
                   catalog[i].available ? "Available" : "Borrowed");
            return;
        }
    }
    printf("Book not found!\n");
}

// Sort catalog naka bubble sort
void sortCatalog() {
    for (int i = 0; i < bookCount - 1; i++) {
        for (int j = 0; j < bookCount - i - 1; j++) {
            if (strcmp(catalog[j].title, catalog[j+1].title) > 0) {
                struct Book temp = catalog[j];
                catalog[j] = catalog[j+1];
                catalog[j+1] = temp;
            }
        }
    }
    printf("Catalog sorted!\n");
}

// Borrow book
void borrowBook() {
    int id;
    char name[100];
    printf("Enter book ID: ");
    if (scanf("%d", &id) != 1) { getchar(); return; }
    getchar();
    printf("Enter your name: ");
    readLine(name, sizeof(name));

    if (id <= 0 || id > bookCount) {
        printf("Invalid ID!\n");
        return;
    }

    if (catalog[id-1].available) {
        catalog[id-1].available = 0;
        struct Borrower *newB = (struct Borrower*) malloc(sizeof(struct Borrower));
        if (newB == NULL) { printf("Memory allocation failed!\n"); return; }
        newB->bookId = id;
        strcpy(newB->name, name);
        newB->next = borrowerHead;
        borrowerHead = newB;
        printf("Book borrowed!\n");
    } else {
        struct Reservation *newR = (struct Reservation*) malloc(sizeof(struct Reservation));
        if (newR == NULL) { printf("Memory allocation failed!\n"); return; }
        newR->bookId = id;
        strcpy(newR->name, name);
        newR->next = reservationHead;
        reservationHead = newR;
        printf("Book unavailable, added to reservation list.\n");
    }
}

// Return book
void returnBook() {
    int id;
    printf("Enter book ID: ");
    if (scanf("%d", &id) != 1) { getchar(); return; }
    getchar();

    if (id <= 0 || id > bookCount) {
        printf("Invalid ID!\n");
        return;
    }

    catalog[id-1].available = 1;
    struct Stack *newS = (struct Stack*) malloc(sizeof(struct Stack));
    if (newS == NULL) { printf("Memory allocation failed!\n"); return; }
    newS->bookId = id;
    newS->next = returnStack;
    returnStack = newS;
    printf("Book returned!\n");

    // Naka Auto assign if reserved
    struct Reservation *prev = NULL, *temp = reservationHead;
    while (temp != NULL) {
        if (temp->bookId == id) {
            printf("Reserved by %s, auto-assigned.\n", temp->name);
            catalog[id-1].available = 0;
            struct Borrower *newB = (struct Borrower*) malloc(sizeof(struct Borrower));
            if (newB == NULL) { printf("Memory allocation failed!\n"); return; }
            newB->bookId = id;
            strcpy(newB->name, temp->name);
            newB->next = borrowerHead;
            borrowerHead = newB;

            if (prev == NULL) {
                reservationHead = temp->next;
            } else {
                prev->next = temp->next;
            }
            free(temp);
            break;
        }
        prev = temp;
        temp = temp->next;
    }
}

// View borrower history
void viewHistory() {
    struct Borrower *temp = borrowerHead;
    if (temp == NULL) {
        printf("No history.\n");
        return;
    }
    while (temp != NULL) {
        printf("Book %d borrowed by %s\n", temp->bookId, temp->name);
        temp = temp->next;
    }
}

// View return
void viewReturns() {
    struct Stack *temp = returnStack;
    if (temp == NULL) {
        printf("No returns.\n");
        return;
    }
    while (temp != NULL) {
        printf("Book %d returned\n", temp->bookId);
        temp = temp->next;
    }
}

// View reservation
void viewReservations() {
    struct Reservation *temp = reservationHead;
    if (temp == NULL) {
        printf("No reservations.\n");
        return;
    }
    while (temp != NULL) {
        printf("Book %d reserved by %s\n", temp->bookId, temp->name);
        temp = temp->next;
    }
}

int main() {
    int choice;
    while (1) {
        printf("\nLibrary System\n");
        printf("1. Add Book\n");
        printf("2. View Books\n");
        printf("3. Borrow Book\n");
        printf("4. Return Book\n");
        printf("5. Search Catalog\n");
        printf("6. Sort Catalog\n");
        printf("7. View History\n");
        printf("8. View Returns\n");
        printf("9. View Reservations\n");
        printf("0. Exit\n");
        printf("Enter choice: ");
        if (scanf("%d", &choice) != 1) { getchar(); continue; }
        getchar();

        switch (choice) {
            case 1: addBook(); break;
            case 2: viewBooks(); break;
            case 3: borrowBook(); break;
            case 4: returnBook(); break;
            case 5: searchCatalog(); break;
            case 6: sortCatalog(); break;
            case 7: viewHistory(); break;
            case 8: viewReturns(); break;
            case 9: viewReservations(); break;
            case 0: printf("Goodbye!\n"); return 0;
            default: printf("Invalid choice!\n");
        }
    }
}
// In This Code we use this as reference
// ProgrammingKnowledge. C Programming Tutorial – Student Record Management System. YouTube, 21 July 2015, www.youtube.com/watch?v=uPTlVsIZr5o
// Easy Tutorials. Inventory Management System in C. YouTube, 15 Mar. 2019, www.youtube.com/watch?v=ZLY4hQN38qY
