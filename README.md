

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <time.h>

// --- Constants ---
#define MAX_NAME 50
#define MAX_PASS 20
#define DB_FILE "bank_database.dat"
#define TEMP_FILE "temp_database.dat"

// --- Structures ---

// Date structure for DOB and Transaction dates
typedef struct {
    int day;
    int month;
    int year;
} Date;

// Transaction Record
typedef struct {
    int trans_id;
    char type[10]; // DEPOSIT, WITHDRAW, TRANSFER
    double amount;
    Date date;
} Transaction;

// Account Structure
typedef struct {
    int id;
    char name[MAX_NAME];
    char password[MAX_PASS];
    char address[100];
    char phone[15];
    char acc_type[10]; // SAVINGS, CURRENT
    double balance;
    Date dob;
    Date created_at;
    int is_active;     // 1 for active, 0 for deleted
} Account;

// --- Function Prototypes ---

// UI & Helper Functions
void clear_screen();
void print_header(const char *title);
void print_line();
void delay(int milliseconds); // Simulation for processing
void get_password(char *pass); // Mask password input
void get_current_date(Date *d);
int generate_id();

// Core System Functions
void main_menu();
void create_account();
void login();
void customer_menu(Account acc);
void admin_menu();

// Banking Operations
void deposit_money(int acc_id);
void withdraw_money(int acc_id);
void transfer_money(int sender_id);
void check_balance(int acc_id);
void account_details(int acc_id);
void update_account(int acc_id);
void delete_account_admin();
void list_all_accounts();
void calculate_interest();

// File Handling
void save_account(Account acc);
int get_account_by_id(int id, Account *acc);
void update_account_in_file(Account acc);

// Global State
int IS_RUNNING = 1;

// --- Main Function ---
int main() {
    clear_screen();
    printf("\n\n\t\t\tLoading System...\n");
    // Simulate loading
    int i;
    for(i=0; i<30; i++) {
        printf("%c", 219); // Block character
        // In a real app we might verify files here
    }
    
    while(IS_RUNNING) {
        main_menu();
    }
    
    return 0;
}

// --- Implementation ---

/**
 * Clears the console screen.
 * Uses system specific commands (portable-ish).
 */
void clear_screen() {
    #ifdef _WIN32
        system("cls");
    #else
        system("clear");
    #endif
}

/**
 * Prints a stylized header for menus.
 */
void print_header(const char *title) {
    clear_screen();
    printf("\n");
    printf("==============================================================\n");
    printf("\t\t BANK MANAGEMENT SYSTEM\n");
    printf("==============================================================\n");
    printf("\t\t %s\n", title);
    printf("--------------------------------------------------------------\n");
}

void print_line() {
    printf("--------------------------------------------------------------\n");
}

/**
 * Gets the current system date.
 */
void get_current_date(Date *d) {
    time_t t = time(NULL);
    struct tm tm = *localtime(&t);
    d->day = tm.tm_mday;
    d->month = tm.tm_mon + 1;
    d->year = tm.tm_year + 1900;
}

/**
 * Generates a pseudo-random ID based on time.
 */
int generate_id() {
    srand(time(NULL));
    return (rand() % 90000) + 10000; // 5 digit ID
}

/**
 * Main Entry Menu
 */
void main_menu() {
    print_header("WELCOME");
    printf("\n\t1. Create New Account");
    printf("\n\t2. Login to Account");
    printf("\n\t3. Admin Portal");
    printf("\n\t4. Exit");
    printf("\n\n\tEnter Choice (1-4): ");
    
    int choice;
    if(scanf("%d", &choice) != 1) {
        while(getchar() != '\n'); // Clear buffer
        choice = 0;
    }
    
    switch(choice) {
        case 1:
            create_account();
            break;
        case 2:
            login();
            break;
        case 3:
            admin_menu();
            break;
        case 4:
            printf("\n\tExiting... Thank you for banking with us.\n");
            IS_RUNNING = 0;
            break;
        default:
            printf("\n\tInvalid Choice! Press Enter to try again.");
            while(getchar() != '\n'); 
            getchar();
    }
}

/**
 * Function to register a new user.
 */
void create_account() {
    Account acc;
    print_header("CREATE ACCOUNT");
    
    // Clear buffer
    while(getchar() != '\n');

    printf("\n\tEnter Full Name: ");
    fgets(acc.name, MAX_NAME, stdin);
    acc.name[strcspn(acc.name, "\n")] = 0; // Remove newline

    printf("\tEnter Address: ");
    fgets(acc.address, 100, stdin);
    acc.address[strcspn(acc.address, "\n")] = 0;

    printf("\tEnter Phone Number: ");
    scanf("%s", acc.phone);

    printf("\tEnter Date of Birth (DD MM YYYY): ");
    scanf("%d %d %d", &acc.dob.day, &acc.dob.month, &acc.dob.year);

    printf("\tAccount Type (SAVINGS/CURRENT): ");
    scanf("%s", acc.acc_type);
    
    // Convert type to uppercase
    for(int i = 0; acc.acc_type[i]; i++){
      acc.acc_type[i] = toupper(acc.acc_type[i]);
    }

    printf("\tSet a Password: ");
    scanf("%s", acc.password);

    printf("\tInitial Deposit Amount (Min 500): ");
    scanf("%lf", &acc.balance);

    if (acc.balance < 500) {
        printf("\n\tError: Minimum balance of 500 required!");
        printf("\n\tPress Enter to return...");
        while(getchar() != '\n'); getchar();
        return;
    }

    acc.id = generate_id();
    acc.is_active = 1;
    get_current_date(&acc.created_at);

    save_account(acc);

    printf("\n\tAccount Created Successfully!");
    printf("\n\tYOUR ACCOUNT ID: %d (Please remember this)", acc.id);
    printf("\n\n\tPress Enter to continue...");
    while(getchar() != '\n'); getchar();
}

/**
 * Appends account to file.
 */
void save_account(Account acc) {
    FILE *fp = fopen(DB_FILE, "ab"); // Append binary
    if (fp == NULL) {
        printf("Error opening database!\n");
        return;
    }
    fwrite(&acc, sizeof(Account), 1, fp);
    fclose(fp);
}

/**
 * Login handler.
 */
void login() {
    int id, found = 0;
    char pass[MAX_PASS];
    Account acc;

    print_header("LOGIN");
    printf("\n\tEnter Account ID: ");
    scanf("%d", &id);
    
    printf("\tEnter Password: ");
    scanf("%s", pass);

    if (get_account_by_id(id, &acc)) {
        if (strcmp(acc.password, pass) == 0 && acc.is_active) {
            printf("\n\tLogin Successful! Redirecting...");
            // delay(1000); 
            customer_menu(acc);
        } else {
            printf("\n\tError: Invalid Password or Account Suspended.");
            printf("\n\tPress Enter...");
            while(getchar() != '\n'); getchar();
        }
    } else {
        printf("\n\tError: Account ID not found.");
        printf("\n\tPress Enter...");
        while(getchar() != '\n'); getchar();
    }
}

/**
 * Helper to retrieve account from file.
 * Returns 1 if found, 0 otherwise.
 */
int get_account_by_id(int id, Account *acc) {
    FILE *fp = fopen(DB_FILE, "rb");
    if (fp == NULL) return 0;

    int found = 0;
    while(fread(acc, sizeof(Account), 1, fp)) {
        if(acc->id == id) {
            found = 1;
            break;
        }
    }
    fclose(fp);
    return found;
}

/**
 * Dashboard for logged-in user.
 */
void customer_menu(Account acc) {
    int choice;
    int logged_in = 1;

    // Refresh account data from file ensures balance is up to date
    // if operations modified it.
    
    while(logged_in) {
        // Reload account data to ensure latest balance
        get_account_by_id(acc.id, &acc);

        print_header("CUSTOMER DASHBOARD");
        printf("\tWelcome, %s\n", acc.name);
        print_line();
        printf("\n\t1. Check Balance");
        printf("\n\t2. Deposit Money");
        printf("\n\t3. Withdraw Money");
        printf("\n\t4. Transfer Funds");
        printf("\n\t5. Account Details");
        printf("\n\t6. Modify Information");
        printf("\n\t7. Logout");
        printf("\n\n\tEnter Choice: ");
        scanf("%d", &choice);

        switch(choice) {
            case 1:
                check_balance(acc.id);
                break;
            case 2:
                deposit_money(acc.id);
                break;
            case 3:
                withdraw_money(acc.id);
                break;
            case 4:
                transfer_money(acc.id);
                break;
            case 5:
                account_details(acc.id);
                break;
            case 6:
                update_account(acc.id);
                break;
            case 7:
                logged_in = 0;
                printf("\n\tLogging out...");
                break;
            default:
                printf("\n\tInvalid Option.");
        }
        if(logged_in) {
            printf("\n\tPress Enter to continue...");
            while(getchar() != '\n'); getchar();
        }
    }
}

void check_balance(int acc_id) {
    Account acc;
    if (get_account_by_id(acc_id, &acc)) {
        printf("\n\t===========================");
        printf("\n\t AVAILABLE BALANCE");
        printf("\n\t===========================");
        printf("\n\t $ %.2f", acc.balance);
        printf("\n\t===========================\n");
    }
}

void deposit_money(int acc_id) {
    Account acc;
    double amount;
    
    if (get_account_by_id(acc_id, &acc)) {
        printf("\n\tEnter Amount to Deposit: $");
        scanf("%lf", &amount);
        
        if (amount <= 0) {
            printf("\n\tInvalid Amount!");
            return;
        }
        
        acc.balance += amount;
        update_account_in_file(acc);
        printf("\n\tSuccess! New Balance: $%.2f", acc.balance);
    }
}

void withdraw_money(int acc_id) {
    Account acc;
    double amount;
    
    if (get_account_by_id(acc_id, &acc)) {
        printf("\n\tCurrent Balance: $%.2f", acc.balance);
        printf("\n\tEnter Amount to Withdraw: $");
        scanf("%lf", &amount);
        
        if (amount <= 0) {
            printf("\n\tInvalid Amount!");
            return;
        }
        
        if (amount > acc.balance) {
            printf("\n\tInsufficient Funds!");
            return;
        }
        
        acc.balance -= amount;
        update_account_in_file(acc);
        printf("\n\tSuccess! Please collect your cash.");
        printf("\n\tNew Balance: $%.2f", acc.balance);
    }
}

void transfer_money(int sender_id) {
    Account sender, receiver;
    int receiver_id;
    double amount;
    
    if (get_account_by_id(sender_id, &sender)) {
        printf("\n\tEnter Receiver Account ID: ");
        scanf("%d", &receiver_id);
        
        if (sender_id == receiver_id) {
            printf("\n\tCannot transfer to self!");
            return;
        }
        
        if (get_account_by_id(receiver_id, &receiver) && receiver.is_active) {
            printf("\n\tReceiver: %s", receiver.name);
            printf("\n\tEnter Amount to Transfer: $");
            scanf("%lf", &amount);
            
            if (amount <= 0 || amount > sender.balance) {
                printf("\n\tInvalid Amount or Insufficient Balance!");
                return;
            }
            
            // Perform Transfer
            sender.balance -= amount;
            receiver.balance += amount;
            
            update_account_in_file(sender);
            update_account_in_file(receiver);
            
            printf("\n\tTransfer Successful!");
            printf("\n\tRemaining Balance: $%.2f", sender.balance);
            
        } else {
            printf("\n\tReceiver ID not found or inactive.");
        }
    }
}

void account_details(int acc_id) {
    Account acc;
    if (get_account_by_id(acc_id, &acc)) {
        print_header("ACCOUNT DETAILS");
        printf("\n\tID:          %d", acc.id);
        printf("\n\tName:        %s", acc.name);
        printf("\n\tType:        %s", acc.acc_type);
        printf("\n\tPhone:       %s", acc.phone);
        printf("\n\tAddress:     %s", acc.address);
        printf("\n\tDOB:         %02d/%02d/%d", acc.dob.day, acc.dob.month, acc.dob.year);
        printf("\n\tCreated:     %02d/%02d/%d", acc.created_at.day, acc.created_at.month, acc.created_at.year);
        printf("\n\t--------------------------");
        printf("\n\tBalance:     $%.2f", acc.balance);
    }
}

void update_account(int acc_id) {
    Account acc;
    int choice;
    
    if (get_account_by_id(acc_id, &acc)) {
        printf("\n\t1. Update Phone");
        printf("\n\t2. Update Address");
        printf("\n\t3. Change Password");
        printf("\n\tEnter Choice: ");
        scanf("%d", &choice);
        
        while(getchar() != '\n'); // Clear buffer
        
        switch(choice) {
            case 1:
                printf("\n\tEnter New Phone: ");
                scanf("%s", acc.phone);
                break;
            case 2:
                printf("\n\tEnter New Address: ");
                fgets(acc.address, 100, stdin);
                acc.address[strcspn(acc.address, "\n")] = 0;
                break;
            case 3:
                printf("\n\tEnter New Password: ");
                scanf("%s", acc.password);
                break;
            default:
                printf("\n\tInvalid selection.");
                return;
        }
        
        update_account_in_file(acc);
        printf("\n\tInformation Updated Successfully.");
    }
}

/**
 * Updates a record in the binary file.
 * Uses a temp file approach to ensure data integrity.
 */
void update_account_in_file(Account acc) {
    FILE *fp = fopen(DB_FILE, "rb");
    FILE *temp = fopen(TEMP_FILE, "wb");
    Account tempAcc;
    
    if (fp == NULL || temp == NULL) return;
    
    while(fread(&tempAcc, sizeof(Account), 1, fp)) {
        if (tempAcc.id == acc.id) {
            fwrite(&acc, sizeof(Account), 1, temp); // Write modified
        } else {
            fwrite(&tempAcc, sizeof(Account), 1, temp); // Write original
        }
    }
    
    fclose(fp);
    fclose(temp);
    
    remove(DB_FILE);
    rename(TEMP_FILE, DB_FILE);
}

// --- Admin Section ---

void admin_menu() {
    char pass[MAX_PASS];
    print_header("ADMIN LOGIN");
    printf("\n\tEnter Admin Password: ");
    scanf("%s", pass);
    
    if (strcmp(pass, "admin123") != 0) {
        printf("\n\tAccess Denied!");
        printf("\n\tPress Enter to return...");
        while(getchar() != '\n'); getchar();
        return;
    }
    
    int choice;
    int running = 1;
    
    while(running) {
        print_header("ADMINISTRATOR PANEL");
        printf("\n\t1. List All Accounts");
        printf("\n\t2. Delete Account (Soft Delete)");
        printf("\n\t3. Apply Interest to Savings");
        printf("\n\t4. Return to Main Menu");
        printf("\n\n\tEnter Choice: ");
        scanf("%d", &choice);
        
        switch(choice) {
            case 1:
                list_all_accounts();
                break;
            case 2:
                delete_account_admin();
                break;
            case 3:
                calculate_interest();
                break;
            case 4:
                running = 0;
                break;
            default:
                printf("\n\tInvalid Option");
        }
        
        if (running) {
            printf("\n\tPress Enter to continue...");
            while(getchar() != '\n'); getchar();
        }
    }
}

void list_all_accounts() {
    FILE *fp = fopen(DB_FILE, "rb");
    Account acc;
    
    if (fp == NULL) {
        printf("\n\tNo records found.");
        return;
    }
    
    print_header("ALL CUSTOMER RECORDS");
    printf("%-10s %-20s %-15s %-15s %-10s\n", "ID", "NAME", "PHONE", "TYPE", "BALANCE");
    print_line();
    
    int count = 0;
    while(fread(&acc, sizeof(Account), 1, fp)) {
        if (acc.is_active) {
            printf("%-10d %-20s %-15s %-15s $%-10.2f\n", 
                   acc.id, acc.name, acc.phone, acc.acc_type, acc.balance);
            count++;
        }
    }
    print_line();
    printf("\tTotal Active Accounts: %d\n", count);
    
    fclose(fp);
}

void delete_account_admin() {
    int id;
    Account acc;
    printf("\n\tEnter Account ID to delete: ");
    scanf("%d", &id);
    
    if (get_account_by_id(id, &acc)) {
        printf("\n\tAccount Found: %s", acc.name);
        printf("\n\tBalance: $%.2f", acc.balance);
        printf("\n\tAre you sure? (1=Yes, 0=No): ");
        int confirm;
        scanf("%d", &confirm);
        
        if (confirm == 1) {
            acc.is_active = 0; // Soft delete
            update_account_in_file(acc);
            printf("\n\tAccount Deleted Successfully.");
        } else {
            printf("\n\tOperation Cancelled.");
        }
    } else {
        printf("\n\tAccount not found.");
    }
}

void calculate_interest() {
    FILE *fp = fopen(DB_FILE, "rb");
    FILE *temp = fopen(TEMP_FILE, "wb");
    Account acc;
    int count = 0;
    
    if (fp == NULL || temp == NULL) return;
    
    printf("\n\tApplying 2.5%% Interest to Savings Accounts...\n");
    
    while(fread(&acc, sizeof(Account), 1, fp)) {
        if (acc.is_active && strcmp(acc.acc_type, "SAVINGS") == 0) {
            double interest = acc.balance * 0.025;
            acc.balance += interest;
            count++;
        }
        fwrite(&acc, sizeof(Account), 1, temp);
    }
    
    fclose(fp);
    fclose(temp);
    
    remove(DB_FILE);
    rename(TEMP_FILE, DB_FILE);
    
    printf("\n\tInterest applied to %d accounts.", count);
}

