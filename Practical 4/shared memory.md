#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/wait.h>
#include <semaphore.h>
#include <fcntl.h>
#include <unistd.h>

#define SHM_SIZE 1024  // Size of shared memory segment

int main() {
    key_t key = ftok("shared_memory.c", 65); // Generate unique key
    int shmid;
    char *data;

    // Step 1: Create shared memory segment
    shmid = shmget(key, SHM_SIZE, IPC_CREAT | 0666);
    if (shmid < 0) {
        perror("shmget failed");
        exit(1);
    }
    printf("Shared memory created with ID: %d\n", shmid);

    // Create a named semaphore for synchronization
    sem_t *sem = sem_open("/shm_sem", O_CREAT, 0666, 0);

    pid_t pid = fork(); // Create child process

    if (pid < 0) {
        perror("Fork failed");
        exit(1);
    }

    if (pid == 0) {
        // CHILD PROCESS - Reader
        // Step 2: Attach shared memory in child
        data = (char*) shmat(shmid, NULL, 0);

        sem_wait(sem); // Wait for parent to write first

        // Step 3: Read data from shared memory
        printf("Child Process read: %s\n", data);

        // Step 5: Detach shared memory in child
        shmdt(data);

        sem_close(sem);
        exit(0);

    } else {
        // PARENT PROCESS - Writer
        // Step 2: Attach shared memory in parent
        data = (char*) shmat(shmid, NULL, 0);

        // Step 3: Write data into shared memory
        strcpy(data, "Hello from Parent Process!");
        printf("Parent Process wrote: %s\n", data);

        sem_post(sem); // Signal child that data is ready

        wait(NULL); // Wait for child to finish

        // Step 5: Detach and delete shared memory
        shmdt(data);
        shmctl(shmid, IPC_RMID, NULL);

        sem_close(sem);
        sem_unlink("/shm_sem");

        printf("Shared memory cleaned up!\n");
    }

    return 0;
}
