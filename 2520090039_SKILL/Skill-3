#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <termios.h>

#define INITIAL_BUFFER_SIZE 100

// Linked list node for command history
typedef struct Node {
    char *command;
    struct Node *next;
} Node;

Node *history_head = NULL;
Node *history_tail = NULL;

// Add command to history
void add_history(const char *command)
{
    Node *new_node = malloc(sizeof(Node));

    if (new_node == NULL) {
        perror("malloc");
        exit(1);
    }

    new_node->command = malloc(strlen(command) + 1);

    if (new_node->command == NULL) {
        perror("malloc");
        free(new_node);
        exit(1);
    }

    strcpy(new_node->command, command);
    new_node->next = NULL;

    if (history_head == NULL) {
        history_head = new_node;
        history_tail = new_node;
    } else {
        history_tail->next = new_node;
        history_tail = new_node;
    }
}

// Count history commands
int get_history_count()
{
    int count = 0;
    Node *current = history_head;

    while (current != NULL) {
        count++;
        current = current->next;
    }

    return count;
}

// Get command at a particular history position
char *get_history_command(int position)
{
    Node *current = history_head;
    int i = 0;

    while (current != NULL) {

        if (i == position) {
            return current->command;
        }

        current = current->next;
        i++;
    }

    return NULL;
}

// Free complete history linked list
void free_history()
{
    Node *current = history_head;

    while (current != NULL) {

        Node *temp = current;
        current = current->next;

        free(temp->command);
        free(temp);
    }

    history_head = NULL;
    history_tail = NULL;
}

// Enable raw keyboard mode
void enable_raw_mode(struct termios *original)
{
    struct termios raw;

    tcgetattr(STDIN_FILENO, original);

    raw = *original;

    raw.c_lflag &= ~(ICANON | ECHO);

    tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}

// Restore normal keyboard mode
void disable_raw_mode(struct termios *original)
{
    tcsetattr(STDIN_FILENO, TCSAFLUSH, original);
}

// Clear current input from terminal
void clear_input(int length)
{
    for (int i = 0; i < length; i++) {
        printf("\b \b");
    }

    fflush(stdout);
}

int main()
{
    struct termios original;

    // Dynamic input buffer
    int buffer_size = INITIAL_BUFFER_SIZE;
    int position = 0;

    char *buffer = malloc(buffer_size);

    if (buffer == NULL) {
        perror("malloc");
        return 1;
    }

    buffer[0] = '\0';

    // -1 means no history command is currently selected
    int history_position = -1;

    enable_raw_mode(&original);

    printf("myshell> ");
    fflush(stdout);

    while (1) {

        char c;

        // Read one character
        if (read(STDIN_FILENO, &c, 1) <= 0) {
            break;
        }

        // ------------------------------------------------
        // ENTER KEY
        // ------------------------------------------------
        if (c == '\n' || c == '\r') {

            buffer[position] = '\0';

            printf("\n");

            // Exit condition
            if (strcmp(buffer, "exit") == 0) {
                break;
            }

            // Store command in history
            if (position > 0) {

                add_history(buffer);

                printf("Command stored: %s\n", buffer);
            }

            // Reset input buffer
            position = 0;
            buffer[0] = '\0';

            // Reset history navigation
            history_position = -1;

            printf("myshell> ");
            fflush(stdout);
        }

        // ------------------------------------------------
        // BACKSPACE
        // ------------------------------------------------
        else if (c == 127 || c == 8) {

            if (position > 0) {

                position--;

                buffer[position] = '\0';

                printf("\b \b");

                fflush(stdout);
            }
        }

        // ------------------------------------------------
        // ESCAPE SEQUENCE
        // ------------------------------------------------
        else if (c == 27) {

            char seq1;
            char seq2;

            // Read '['
            if (read(STDIN_FILENO, &seq1, 1) <= 0) {
                continue;
            }

            // Read 'A' or 'B'
            if (read(STDIN_FILENO, &seq2, 1) <= 0) {
                continue;
            }

            // --------------------------------------------
            // UP ARROW: ESC [ A
            // --------------------------------------------
            if (seq1 == '[' && seq2 == 'A') {

                int count = get_history_count();

                if (count == 0) {
                    continue;
                }

                // Start from most recent command
                if (history_position == -1) {
                    history_position = count - 1;
                }

                // Move to previous command
                else if (history_position > 0) {
                    history_position--;
                }

                char *previous =
                    get_history_command(history_position);

                if (previous != NULL) {

                    // Remove current input
                    clear_input(position);

                    // Update input buffer
                    strcpy(buffer, previous);

                    position = strlen(buffer);

                    // Display recalled command
                    printf("%s", buffer);

                    fflush(stdout);
                }
            }

            // --------------------------------------------
            // DOWN ARROW: ESC [ B
            // --------------------------------------------
            else if (seq1 == '[' && seq2 == 'B') {

                int count = get_history_count();

                if (count == 0) {
                    continue;
                }

                // Move to next command
                if (history_position != -1 &&
                    history_position < count - 1) {

                    history_position++;

                    char *next =
                        get_history_command(history_position);

                    clear_input(position);

                    // Update input buffer
                    strcpy(buffer, next);

                    position = strlen(buffer);

                    printf("%s", buffer);

                    fflush(stdout);
                }

                // Move beyond newest command
                else {

                    history_position = -1;

                    clear_input(position);

                    position = 0;

                    buffer[0] = '\0';

                    fflush(stdout);
                }
            }
        }

        // ------------------------------------------------
        // NORMAL CHARACTER
        // ------------------------------------------------
        else {

            // Resize buffer before it becomes full
            if (position >= buffer_size - 1) {

                buffer_size = buffer_size * 2;

                char *temp =
                    realloc(buffer, buffer_size);

                if (temp == NULL) {

                    perror("realloc");

                    free(buffer);

                    disable_raw_mode(&original);

                    free_history();

                    return 1;
                }

                buffer = temp;
            }

            // Store character in buffer
            buffer[position] = c;

            position++;

            buffer[position] = '\0';

            // Display character
            putchar(c);

            fflush(stdout);
        }
    }

    // Restore terminal
    disable_raw_mode(&original);

    // Release dynamic input buffer
    free(buffer);

    // Release history linked list
    free_history();

    printf("\nShell terminated.\n");

    return 0;
}
