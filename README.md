# DSA-ASSIGNMENT--1
/*
 * FILE & FOLDER ORGANIZATION SYSTEM
 * Name: Ankit Kumar
 * Student ID: BC2025228
 * Section: F
 *
 * Data Structures: Linked List + Tree
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_NAME 100
#define MAX_TYPE 20

typedef struct File {
    char name[MAX_NAME];
    char type[MAX_TYPE];
    int size;
    struct File *next;
} File;

typedef struct Folder {
    char name[MAX_NAME];
    File *files;
    struct Folder *child;
    struct Folder *sibling;
} Folder;

File *createFile(const char *name, const char *type, int size) {
    File *newFile = malloc(sizeof(File));
    if (newFile == NULL) {
        printf("Memory allocation failed!\n");
        return NULL;
    }

    strncpy(newFile->name, name, MAX_NAME - 1);
    newFile->name[MAX_NAME - 1] = '\0';
    strncpy(newFile->type, type, MAX_TYPE - 1);
    newFile->type[MAX_TYPE - 1] = '\0';
    newFile->size = size;
    newFile->next = NULL;

    return newFile;
}

Folder *createFolder(const char *name) {
    Folder *newFolder = malloc(sizeof(Folder));
    if (newFolder == NULL) {
        printf("Memory allocation failed!\n");
        return NULL;
    }

    strncpy(newFolder->name, name, MAX_NAME - 1);
    newFolder->name[MAX_NAME - 1] = '\0';
    newFolder->files = NULL;
    newFolder->child = NULL;
    newFolder->sibling = NULL;

    return newFolder;
}

Folder *findFolder(Folder *root, const char *name) {
    Folder *result;

    if (root == NULL)
        return NULL;

    if (strcmp(root->name, name) == 0)
        return root;

    result = findFolder(root->child, name);
    if (result != NULL)
        return result;

    return findFolder(root->sibling, name);
}

File *findFile(Folder *root, const char *name) {
    File *temp;
    File *result;

    if (root == NULL)
        return NULL;

    temp = root->files;
    while (temp != NULL) {
        if (strcmp(temp->name, name) == 0)
            return temp;
        temp = temp->next;
    }

    result = findFile(root->child, name);
    if (result != NULL)
        return result;

    return findFile(root->sibling, name);
}

void addFile(Folder *folder) {
    char name[MAX_NAME], type[MAX_TYPE];
    int size;
    File *newFile;

    if (folder == NULL) {
        printf("Folder not found!\n");
        return;
    }

    printf("Enter file name: ");
    scanf("%99s", name);
    printf("Enter file type: ");
    scanf("%19s", type);
    printf("Enter file size (KB): ");
    scanf("%d", &size);

    if (size < 0) {
        printf("File size cannot be negative!\n");
        return;
    }

    if (findFile(folder, name) != NULL) {
        printf("A file with this name already exists!\n");
        return;
    }

    newFile = createFile(name, type, size);
    if (newFile == NULL)
        return;

    newFile->next = folder->files;
    folder->files = newFile;

    printf("File '%s' created successfully!\n", name);
}

void addFolder(Folder *parent) {
    char name[MAX_NAME];
    Folder *newFolder, *temp;

    if (parent == NULL) {
        printf("Parent folder not found!\n");
        return;
    }

    printf("Enter new folder name: ");
    scanf("%99s", name);

    if (findFolder(parent->child, name) != NULL) {
        printf("A folder with this name already exists here!\n");
        return;
    }

    newFolder = createFolder(name);
    if (newFolder == NULL)
        return;

    if (parent->child == NULL) {
        parent->child = newFolder;
    } else {
        temp = parent->child;
        while (temp->sibling != NULL)
            temp = temp->sibling;
        temp->sibling = newFolder;
    }

    printf("Folder '%s' created successfully!\n", name);
}

void displayFiles(Folder *folder, int level) {
    File *temp;
    int i;

    if (folder == NULL)
        return;

    temp = folder->files;
    while (temp != NULL) {
        for (i = 0; i < level; i++)
            printf("    ");

        printf("File: %s | Type: %s | Size: %d KB\n",
               temp->name, temp->type, temp->size);

        temp = temp->next;
    }
}

void displayTree(Folder *root, int level) {
    int i;

    if (root == NULL)
        return;

    for (i = 0; i < level; i++)
        printf("    ");

    printf("Folder: %s/\n", root->name);

    displayFiles(root, level + 1);
    displayTree(root->child, level + 1);
    displayTree(root->sibling, level);
}

int countFiles(Folder *root) {
    int count = 0;
    File *temp;

    if (root == NULL)
        return 0;

    temp = root->files;
    while (temp != NULL) {
        count++;
        temp = temp->next;
    }

    count += countFiles(root->child);
    count += countFiles(root->sibling);

    return count;
}

int countFolders(Folder *root) {
    if (root == NULL)
        return 0;

    return 1 + countFolders(root->child) + countFolders(root->sibling);
}

int deleteFile(Folder *root, const char *name) {
    File *temp, *previous;

    if (root == NULL)
        return 0;

    temp = root->files;
    previous = NULL;

    while (temp != NULL) {
        if (strcmp(temp->name, name) == 0) {
            if (previous == NULL)
                root->files = temp->next;
            else
                previous->next = temp->next;

            free(temp);
            return 1;
        }

        previous = temp;
        temp = temp->next;
    }

    if (deleteFile(root->child, name))
        return 1;

    return deleteFile(root->sibling, name);
}

void freeFiles(File *head) {
    File *temp;

    while (head != NULL) {
        temp = head;
        head = head->next;
        free(temp);
    }
}

void freeFolderTree(Folder *root) {
    if (root == NULL)
        return;

    freeFolderTree(root->child);
    freeFolderTree(root->sibling);
    freeFiles(root->files);
    free(root);
}

int main(void) {
    Folder *root, *parent, *folder, *resultFolder;
    File *resultFile;
    int choice;
    char folderName[MAX_NAME], fileName[MAX_NAME];

    root = createFolder("Root");
    if (root == NULL)
        return 1;

    printf("\n========================================\n");
    printf("    FILE & FOLDER ORGANIZATION SYSTEM\n");
    printf("========================================\n");
    printf("Name       : Ankit Kumar\n");
    printf("Student ID : BC2025228\n");
    printf("Section    : F\n");

    do {
        printf("\n--------------- MENU ----------------\n");
        printf("1. Create Folder\n");
        printf("2. Create File\n");
        printf("3. Display Folder Structure\n");
        printf("4. Search Folder\n");
        printf("5. Search File\n");
        printf("6. Delete File\n");
        printf("7. Count Files\n");
        printf("8. Count Folders\n");
        printf("9. Exit\n");
        printf("--------------------------------------\n");

        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                printf("Enter parent folder name: ");
                scanf("%99s", folderName);

                parent = findFolder(root, folderName);
                if (parent != NULL)
                    addFolder(parent);
                else
                    printf("Parent folder not found!\n");
                break;

            case 2:
                printf("Enter folder name: ");
                scanf("%99s", folderName);

                folder = findFolder(root, folderName);
                if (folder != NULL)
                    addFile(folder);
                else
                    printf("Folder not found!\n");
                break;

            case 3:
                printf("\n========== FILE SYSTEM ==========\n");
                displayTree(root, 0);
                printf("=================================\n");
                break;

            case 4:
                printf("Enter folder name to search: ");
                scanf("%99s", folderName);

                resultFolder = findFolder(root, folderName);
                if (resultFolder != NULL)
                    printf("Folder '%s' found!\n", folderName);
                else
                    printf("Folder not found!\n");
                break;

            case 5:
                printf("Enter file name to search: ");
                scanf("%99s", fileName);

                resultFile = findFile(root, fileName);
                if (resultFile != NULL) {
                    printf("\nFile Found!\n");
                    printf("Name : %s\n", resultFile->name);
                    printf("Type : %s\n", resultFile->type);
                    printf("Size : %d KB\n", resultFile->size);
                } else {
                    printf("File not found!\n");
                }
                break;

            case 6:
                printf("Enter file name to delete: ");
                scanf("%99s", fileName);

                if (deleteFile(root, fileName))
                    printf("File '%s' deleted successfully!\n", fileName);
                else
                    printf("File not found!\n");
                break;

            case 7:
                printf("Total Files: %d\n", countFiles(root));
                break;

            case 8:
                printf("Total Folders: %d\n", countFolders(root));
                break;

            case 9:
                printf("\nExiting program...\n");
                break;

            default:
                printf("Invalid choice! Please try again.\n");
        }
    } while (choice != 9);

    freeFolderTree(root);
    printf("Memory released successfully.\n");

    return 0;
}
