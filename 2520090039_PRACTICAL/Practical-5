#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/wait.h>

int main()
{
    int fd[2];
    char buffer[100];

    pipe(fd);

    pid_t pid = fork();

    if(pid > 0)
    {
        close(fd[0]);

        char message[] = "Hello Child Process";

        write(fd[1], message, strlen(message)+1);

        printf("Parent Produced: %s\n", message);

        close(fd[1]);

        wait(NULL);
    }
    else
    {
        close(fd[1]);

        read(fd[0], buffer, sizeof(buffer));

        printf("Child Consumed: %s\n", buffer);

        close(fd[0]);
    }

    return 0;
}
