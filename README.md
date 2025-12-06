#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define CREDENTIAL_FILE "credentials.txt"
#define STUDENT_FILE "students.txt"

struct student {
    int roll;
    char name[50];
    float marks;
};

char currentUser[20];
char currentRole[20];

/* Function Declarations */
void createCredentials();
int loginSystem();
void mainMenu();
void adminMenu();
void userMenu();
void addStudent();
void displayStudents();
void searchStudent();
void updateStudent();
void deleteStudent();


/* ---------------------------------------------------------
   MAIN FUNCTION
--------------------------------------------------------- */
int main() {

    createCredentials();   // Create login credentials if user wants

    if (loginSystem()) {
        mainMenu();
    } else {
        printf("Login failed! Exiting...\n");
    }
    return 0;
}


/* ---------------------------------------------------------
   ASK USER TO CREATE CREDENTIAL FILE
--------------------------------------------------------- */
void createCredentials() {
    char choice;
    char user[20], pass[20], role[10];

    printf("Do you want to create login credentials? (y/n): ");
    scanf(" %c", &choice);

    if (choice == 'y' || choice == 'Y') {

        printf("Enter new username: ");
        scanf("%s", user);

        printf("Enter new password: ");
        scanf("%s", pass);

        printf("Enter role (admin/user): ");
        scanf("%s", role);

        FILE *fp = fopen(CREDENTIAL_FILE, "w");
        if (!fp) {
            printf("Error creating credential file!\n");
            return;
        }

        fprintf(fp, "%s %s %s\n", user, pass, role);
        fclose(fp);

        printf("\nCredentials created successfully!\n");
    }

    /* Create students file if missing */
    FILE *f = fopen(STUDENT_FILE, "r");
    if (!f) {
        f = fopen(STUDENT_FILE, "w");
        fclose(f);
    } else {
        fclose(f);
    }
}


/* ---------------------------------------------------------
   LOGIN SYSTEM
--------------------------------------------------------- */
int loginSystem() {
    char username[20], password[20];
    char fileUser[20], filePass[20], fileRole[20];

    printf("\n============== LOGIN SCREEN ============\n");
    printf("Enter username: ");
    scanf("%s", username);
    printf("Enter password: ");
    scanf("%s", password);

    FILE *fp = fopen(CREDENTIAL_FILE, "r");
    if (!fp) {
        printf("Credential file missing!\n");
        return 0;
    }

    while (fscanf(fp, "%s %s %s", fileUser, filePass, fileRole) != EOF) {
        if (strcmp(username, fileUser) == 0 &&
            strcmp(password, filePass) == 0) {

            strcpy(currentUser, fileUser);
            strcpy(currentRole, fileRole);
            fclose(fp);
            return 1;
        }
    }

    fclose(fp);
    return 0;
}


/* ---------------------------------------------------------
   MAIN MENU (ADMIN / USER CHECK)
--------------------------------------------------------- */
void mainMenu() {
    if (strcmp(currentRole, "admin") == 0)
        adminMenu();
    else
        userMenu();
}


/* ---------------------------------------------------------
   ADMIN MENU
--------------------------------------------------------- */
void adminMenu() {
    int ch;
    do {
        printf("\n===== ADMIN MENU =====\n");
        printf("1. Add Student\n");
        printf("2. Display Students\n");
        printf("3. Search Student\n");
        printf("4. Update Student\n");
        printf("5. Delete Student\n");
        printf("6. Logout\n");
        printf("Enter choice: ");
        scanf("%d", &ch);

        switch (ch) {
            case 1: addStudent(); break;
            case 2: displayStudents(); break;
            case 3: searchStudent(); break;
            case 4: updateStudent(); break;
            case 5: deleteStudent(); break;
            case 6: return;
            default: printf("Invalid choice!\n");
        }
    } while (1);
}


/* ---------------------------------------------------------
   USER MENU
--------------------------------------------------------- */
void userMenu() {
    int ch;
    do {
        printf("\n===== USER MENU =====\n");
        printf("1. Display Students\n");
        printf("2. Search Student\n");
        printf("3. Logout\n");
        printf("Enter choice: ");
        scanf("%d", &ch);

        switch (ch) {
            case 1: displayStudents(); break;
            case 2: searchStudent(); break;
            case 3: return;
            default: printf("Invalid choice!\n");
        }
    } while (1);
}


/* ---------------------------------------------------------
   ADD STUDENT
--------------------------------------------------------- */
void addStudent() {
    struct student st;
    FILE *fp = fopen(STUDENT_FILE, "a");

    printf("Enter roll number: ");
    scanf("%d", &st.roll);

    printf("Enter name: ");
    scanf("%s", st.name);

    printf("Enter marks: ");
    scanf("%f", &st.marks);

    fprintf(fp, "%d %s %.2f\n", st.roll, st.name, st.marks);
    fclose(fp);

    printf("Student added successfully!\n");
}


/* ---------------------------------------------------------
   DISPLAY STUDENTS
--------------------------------------------------------- */
void displayStudents() {
    struct student st;
    FILE *fp = fopen(STUDENT_FILE, "r");

    printf("\nRoll\tName\tMarks\n");
    printf("-----------------------------\n");

    while (fscanf(fp, "%d %s %f", &st.roll, st.name, &st.marks) != EOF) {
        printf("%d\t%s\t%.2f\n", st.roll, st.name, st.marks);
    }

    fclose(fp);
}


/* ---------------------------------------------------------
   SEARCH STUDENT
--------------------------------------------------------- */
void searchStudent() {
    int roll;
    printf("Enter roll number to search: ");
    scanf("%d", &roll);

    struct student st;
    FILE *fp = fopen(STUDENT_FILE, "r");
    int found = 0;

    while (fscanf(fp, "%d %s %f", &st.roll, st.name, &st.marks) != EOF) {
        if (st.roll == roll) {
            printf("\nFound: %d %s %.2f\n", st.roll, st.name, st.marks);
            found = 1;
            break;
        }
    }

    if (!found)
        printf("Student not found!\n");

    fclose(fp);
}


/* ---------------------------------------------------------
   UPDATE STUDENT
--------------------------------------------------------- */
void updateStudent() {
    int roll;
    printf("Enter roll number to update: ");
    scanf("%d", &roll);

    struct student st;
    FILE *fp = fopen(STUDENT_FILE, "r");
    FILE *tmp = fopen("temp.txt", "w");

    int found = 0;

    while (fscanf(fp, "%d %s %f", &st.roll, st.name, &st.marks) != EOF) {
        if (st.roll == roll) {
            found = 1;
            printf("Enter new name: ");
            scanf("%s", st.name);
            printf("Enter new marks: ");
            scanf("%f", &st.marks);
        }
        fprintf(tmp, "%d %s %.2f\n", st.roll, st.name, st.marks);
    }

    fclose(fp);
    fclose(tmp);

    if (found) {
        remove(STUDENT_FILE);
        rename("temp.txt", STUDENT_FILE);
        printf("Record updated!\n");
    } else {
        remove("temp.txt");
        printf("Roll number not found!\n");
    }
}


/* ---------------------------------------------------------
   DELETE STUDENT
--------------------------------------------------------- */
void deleteStudent() {
    int roll;
    printf("Enter roll number to delete: ");
    scanf("%d", &roll);

    struct student st;
    FILE *fp = fopen(STUDENT_FILE, "r");
    FILE *tmp = fopen("temp.txt", "w");

    int found = 0;

    while (fscanf(fp, "%d %s %f", &st.roll, st.name, &st.marks) != EOF) {
        if (st.roll == roll) {
            found = 1;
            continue;
        }
        fprintf(tmp, "%d %s %.2f\n", st.roll, st.name, st.marks);
    }

    fclose(fp);
    fclose(tmp);

    if (found) {
        remove(STUDENT_FILE);
        rename("temp.txt", STUDENT_FILE);
        printf("Record deleted!\n");
    } else {
        remove("temp.txt");
        printf("Roll number not found!\n");
    }
}
