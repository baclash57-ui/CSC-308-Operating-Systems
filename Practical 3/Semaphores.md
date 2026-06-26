#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <time.h>

#define NUM_THREADS 10
#define NUM_INCREMENTS 100000

long counter_sem = 0;    // Counter protected by semaphore
long counter_mutex = 0;  // Counter protected by mutex

sem_t semaphore;         // Semaphore for protection
pthread_mutex_t mutex;   // Mutex for protection

// Thread function using SEMAPHORE
void *increment_sem(void *arg) {
    for (int i = 0; i < NUM_INCREMENTS; i++) {
        sem_wait(&semaphore);  // Lock using semaphore
        counter_sem++;
        sem_post(&semaphore);  // Unlock using semaphore
    }
    return NULL;
}

// Thread function using MUTEX
void *increment_mutex(void *arg) {
    for (int i = 0; i < NUM_INCREMENTS; i++) {
        pthread_mutex_lock(&mutex);   // Lock using mutex
        counter_mutex++;
        pthread_mutex_unlock(&mutex); // Unlock using mutex
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];
    clock_t start, end;

    // Initialize semaphore and mutex
    sem_init(&semaphore, 0, 1);
    pthread_mutex_init(&mutex, NULL);

    // Test SEMAPHORE performance
    start = clock();
    for (int i = 0; i < NUM_THREADS; i++)
        pthread_create(&threads[i], NULL, increment_sem, NULL);
    for (int i = 0; i < NUM_THREADS; i++)
        pthread_join(threads[i], NULL);
    end = clock();
    printf("Semaphore - Counter: %ld | Time: %.4f seconds\n",
           counter_sem, (double)(end - start) / CLOCKS_PER_SEC);

    // Test MUTEX performance
    start = clock();
    for (int i = 0; i < NUM_THREADS; i++)
        pthread_create(&threads[i], NULL, increment_mutex, NULL);
    for (int i = 0; i < NUM_THREADS; i++)
        pthread_join(threads[i], NULL);
    end = clock();
    printf("Mutex     - Counter: %ld | Time: %.4f seconds\n",
           counter_mutex, (double)(end - start) / CLOCKS_PER_SEC);

    // Clean up
    sem_destroy(&semaphore);
    pthread_mutex_destroy(&mutex);

    return 0;
}
