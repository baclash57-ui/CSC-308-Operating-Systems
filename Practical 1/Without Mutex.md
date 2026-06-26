#include <stdio.h>      // For printf
#include <pthread.h>    // For threads and mutex functions

#define NUM_THREADS 10        // We will create 10 threads
#define NUM_INCREMENTS 100000 // Each thread increments 100,000 times

long counter = 0;          // Shared counter that all threads will increment
pthread_mutex_t mutex;     // The mutex lock that protects the counter

// This is the function each thread will run
void *increment(void *arg) {
   for (int i = 0; i < NUM_INCREMENTS; i++) {
        
      //pthread_mutex_lock(&mutex);   // Lock - only one thread can pass this point at a time
        counter++;                     // Safely increment the shared counter
        //pthread_mutex_unlock(&mutex); // Unlock - let the next thread in
        
    }
    return NULL; // Thread is done, return nothing
}

int main() {
    pthread_t threads[NUM_THREADS]; // Array to store our 10 threads

    pthread_mutex_init(&mutex, NULL); // Set up the mutex before any thread uses it

    // Create all 10 threads - each one starts running increment() immediately
    for (int i = 0; i < NUM_THREADS; i++)
        pthread_create(&threads[i], NULL, increment, NULL);

    // Wait for ALL threads to finish before we print the result
    for (int i = 0; i < NUM_THREADS; i++)
        pthread_join(threads[i], NULL);

    // Print what we expected vs what actually happened
    printf("Expected: %ld\n", (long)NUM_THREADS * NUM_INCREMENTS);
    printf("Actual:   %ld\n", counter);

    pthread_mutex_destroy(&mutex); // Clean up the mutex from memory
    return 0;
}
