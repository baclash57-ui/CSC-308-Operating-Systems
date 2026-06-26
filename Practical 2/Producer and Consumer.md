#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define BUFFER_SIZE 5    // Size of the circular buffer
#define NUM_ITEMS 10     // Number of items to produce/consume

int buffer[BUFFER_SIZE]; // The shared circular buffer
int in = 0;              // Where producer inserts next item
int out = 0;             // Where consumer removes next item

sem_t mutex;             // Controls access to buffer (binary semaphore)
sem_t empty;             // Counts empty slots in buffer
sem_t full;              // Counts filled slots in buffer

// Producer thread - generates items and puts them in buffer
void *producer(void *arg) {
    for (int i = 0; i < NUM_ITEMS; i++) {
        int item = rand() % 100; // Generate a random item

        sem_wait(&empty);        // Wait if buffer is full
        sem_wait(&mutex);        // Lock the buffer

        buffer[in] = item;       // Insert item into buffer
        printf("Producer inserted: %d at slot %d\n", item, in);
        in = (in + 1) % BUFFER_SIZE; // Move to next slot circularly

        sem_post(&mutex);        // Unlock the buffer
        sem_post(&full);         // Signal that a new item is available

        sleep(1);                // Simulate production time
    }
    return NULL;
}

// Consumer thread - takes items from buffer and consumes them
void *consumer(void *arg) {
    for (int i = 0; i < NUM_ITEMS; i++) {
        sem_wait(&full);         // Wait if buffer is empty
        sem_wait(&mutex);        // Lock the buffer

        int item = buffer[out];  // Remove item from buffer
        printf("Consumer removed: %d from slot %d\n", item, out);
        out = (out + 1) % BUFFER_SIZE; // Move to next slot circularly

        sem_post(&mutex);        // Unlock the buffer
        sem_post(&empty);        // Signal that a slot is now free

        sleep(2);                // Simulate consumption time (slower than producer)
    }
    return NULL;
}

int main() {
    pthread_t prod_thread, cons_thread;

    // Initialize semaphores
    sem_init(&mutex, 0, 1);            // mutex starts at 1 (unlocked)
    sem_init(&empty, 0, BUFFER_SIZE);  // all slots start empty
    sem_init(&full, 0, 0);             // no slots are full yet

    // Create producer and consumer threads
    pthread_create(&prod_thread, NULL, producer, NULL);
    pthread_create(&cons_thread, NULL, consumer, NULL);

    // Wait for both threads to finish
    pthread_join(prod_thread, NULL);
    pthread_join(cons_thread, NULL);

    // Clean up semaphores
    sem_destroy(&mutex);
    sem_destroy(&empty);
    sem_destroy(&full);

    printf("All items produced and consumed!\n");
    return 0;
}
