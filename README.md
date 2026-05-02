#include <stdio.h>
#include <stdlib.h>

struct clientData
{
    unsigned int acctNum;
    char lastName[15];
    char firstName[10];
    double balance;
};

// function prototypes
void initializeFile(FILE *fPtr);
unsigned int enterChoice(void);
void textFile(FILE *readPtr);
void updateRecord(FILE *fPtr);
void newRecord(FILE *fPtr);
void deleteRecord(FILE *fPtr);
void searchRecord(FILE *fPtr);
void displayAll(FILE *fPtr);
void depositWithdraw(FILE *fPtr);
void countAccounts(FILE *fPtr);

// MAIN
int main()
{
    FILE *cfPtr;

    cfPtr = fopen("credit.dat", "rb+");
    if (cfPtr == NULL)
    {
        cfPtr = fopen("credit.dat", "wb+");
        initializeFile(cfPtr);
    }

    unsigned int choice;

    while ((choice = enterChoice()) != 10)
    {
        switch (choice)
        {
        case 1: textFile(cfPtr); break;
        case 2: updateRecord(cfPtr); break;
        case 3: newRecord(cfPtr); break;
        case 4: deleteRecord(cfPtr); break;
        case 5: searchRecord(cfPtr); break;
        case 6: displayAll(cfPtr); break;
        case 7: depositWithdraw(cfPtr); break;
        case 8: countAccounts(cfPtr); break;
        case 9: printf("Reserved for future feature\n"); break;
        default: printf("Invalid choice\n");
        }
    }

    fclose(cfPtr);
    return 0;
}

// initialize file
void initializeFile(FILE *fPtr)
{
    struct clientData blank = {0, "", "", 0.0};
    for (int i = 0; i < 100; i++)
        fwrite(&blank, sizeof(struct clientData), 1, fPtr);
}

// MENU
unsigned int enterChoice(void)
{
    unsigned int choice;

    printf("\n===== BANK MENU =====\n");
    printf("1  - Store text file\n");
    printf("2  - Update account\n");
    printf("3  - Add new account\n");
    printf("4  - Delete account\n");
    printf("5  - Search account\n");
    printf("6  - Display all\n");
    printf("7  - Deposit / Withdraw\n");
    printf("8  - Count accounts\n");
    printf("9  - Future option\n");
    printf("10 - Exit\n");
    printf("? ");

    scanf("%u", &choice);
    return choice;
}

// TEXT FILE
void textFile(FILE *readPtr)
{
    FILE *writePtr = fopen("accounts.txt", "w");
    struct clientData client;

    rewind(readPtr);

    fprintf(writePtr, "%-6s%-16s%-11s%10s\n",
            "Acct", "Last Name", "First Name", "Balance");

    while (fread(&client, sizeof(struct clientData), 1, readPtr))
    {
        if (client.acctNum != 0)
        {
            fprintf(writePtr, "%-6d%-16s%-11s%10.2f\n",
                    client.acctNum, client.lastName,
                    client.firstName, client.balance);
        }
    }

    fclose(writePtr);
    printf("Text file created!\n");
}

// UPDATE
void updateRecord(FILE *fPtr)
{
    struct clientData client;
    unsigned int acc;
    double amount;

    printf("Enter account: ");
    scanf("%u", &acc);

    fseek(fPtr, (acc - 1) * sizeof(struct clientData), SEEK_SET);
    fread(&client, sizeof(struct clientData), 1, fPtr);

    if (client.acctNum == 0)
    {
        printf("Account not found\n");
        return;
    }

    printf("Enter amount (+/-): ");
    scanf("%lf", &amount);

    client.balance += amount;

    fseek(fPtr, -sizeof(struct clientData), SEEK_CUR);
    fwrite(&client, sizeof(struct clientData), 1, fPtr);

    printf("Updated!\n");
}

// ADD
void newRecord(FILE *fPtr)
{
    struct clientData client = {0, "", "", 0.0};
    unsigned int acc;

    printf("Enter account number: ");
    scanf("%u", &acc);

    fseek(fPtr, (acc - 1) * sizeof(struct clientData), SEEK_SET);
    fread(&client, sizeof(struct clientData), 1, fPtr);

    if (client.acctNum != 0)
    {
        printf("Already exists\n");
        return;
    }

    printf("Enter lastname firstname balance:\n");
    scanf("%s %s %lf", client.lastName,
          client.firstName, &client.balance);

    client.acctNum = acc;

    fseek(fPtr, (acc - 1) * sizeof(struct clientData), SEEK_SET);
    fwrite(&client, sizeof(struct clientData), 1, fPtr);

    printf("Account added!\n");
}

// DELETE
void deleteRecord(FILE *fPtr)
{
    struct clientData client, blank = {0, "", "", 0.0};
    unsigned int acc;

    printf("Enter account to delete: ");
    scanf("%u", &acc);

    fseek(fPtr, (acc - 1) * sizeof(struct clientData), SEEK_SET);
    fread(&client, sizeof(struct clientData), 1, fPtr);

    if (client.acctNum == 0)
    {
        printf("Not found\n");
        return;
    }

    fseek(fPtr, (acc - 1) * sizeof(struct clientData), SEEK_SET);
    fwrite(&blank, sizeof(struct clientData), 1, fPtr);

    printf("Deleted!\n");
}

// SEARCH
void searchRecord(FILE *fPtr)
{
    struct clientData client;
    unsigned int acc;

    printf("Enter account: ");
    scanf("%u", &acc);

    fseek(fPtr, (acc - 1) * sizeof(struct clientData), SEEK_SET);
    fread(&client, sizeof(struct clientData), 1, fPtr);

    if (client.acctNum == 0)
        printf("Not found\n");
    else
        printf("%d %s %s %.2f\n",
               client.acctNum, client.lastName,
               client.firstName, client.balance);
}

// DISPLAY
void displayAll(FILE *fPtr)
{
    struct clientData client;

    rewind(fPtr);

    printf("\nALL ACCOUNTS:\n");

    while (fread(&client, sizeof(struct clientData), 1, fPtr))
    {
        if (client.acctNum != 0)
            printf("%d %s %s %.2f\n",
                   client.acctNum, client.lastName,
                   client.firstName, client.balance);
    }
}

// DEPOSIT/WITHDRAW
void depositWithdraw(FILE *fPtr)
{
    struct clientData client;
    unsigned int acc;
    double amt;
    int ch;

    printf("Enter account: ");
    scanf("%u", &acc);

    fseek(fPtr, (acc - 1) * sizeof(struct clientData), SEEK_SET);
    fread(&client, sizeof(struct clientData), 1, fPtr);

    if (client.acctNum == 0)
    {
        printf("Not found\n");
        return;
    }

    printf("1-Deposit 2-Withdraw: ");
    scanf("%d", &ch);

    printf("Amount: ");
    scanf("%lf", &amt);

    if (ch == 1)
        client.balance += amt;
    else if (ch == 2 && amt <= client.balance)
        client.balance -= amt;
    else
    {
        printf("Invalid!\n");
        return;
    }

    fseek(fPtr, -sizeof(struct clientData), SEEK_CUR);
    fwrite(&client, sizeof(struct clientData), 1, fPtr);

    printf("Done!\n");
}

// COUNT
void countAccounts(FILE *fPtr)
{
    struct clientData client;
    int count = 0;

    rewind(fPtr);

    while (fread(&client, sizeof(struct clientData), 1, fPtr))
        if (client.acctNum != 0)
            count++;

    printf("Total accounts: %d\n", count);
}
