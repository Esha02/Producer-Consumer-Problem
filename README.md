#include <stdio.h> 
#include <stdlib.h> 
#include <unistd.h> 
#include <pthread.h> 
#include <semaphore.h> 
#include <fcntl.h> 
#include <sys/stat.h> 
#define N 2 

int buffer[N]; 
int in = 0, out = 0; 

sem_t full, empty; 

void *producer(void *param); 
void *consumer(void *param); 

int main() 
{ 
    pthread_t tid1, tid2; 
    sem_init(&full, 0, 0); 
    sem_init(&empty, 0, N); 

    pthread_create(&tid1, NULL, producer, NULL); 
    pthread_create(&tid2, NULL, consumer, NULL); 

    pthread_join(tid1, NULL); 
    pthread_join(tid2, NULL); 

    sem_destroy(&full); 
    sem_destroy(&empty); 

    return 0; 
} 

void *producer(void *param) 
{ 
    int item; 
    while (1) { 
        item = rand(); 
        sem_wait(&empty); 
        buffer[in] = item; 
        printf("Producer %d: Item %d\n", 
               pthread_self(), item); 
        in = (in + 1) % N; 
        sem_post(&full); 
        if (item == 10) 
            break; 
    } 
    pthread_exit(0); 
} 

void *consumer(void *param) 
{ 
    int item; 
    while (1) { 
        sem_wait(&full); 
        item = buffer[out]; 
        printf("Consumer %d: Item %d\n", 
               pthread_self(), item); 
        out = (out + 1) % N; 
        sem_post(&empty); 
        if (item == 10) 
            break; 
    } 
    pthread_exit(0); 
}
