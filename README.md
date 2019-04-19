//Write a multithreaded program that implements the banker's algorithm. Create n threads that request and release resources from the bank. The banker will grant the request only if it leaves the system in a safe state. It is important that shared data be safe from concurrent access. To ensure safe access to shared data, you can use mutex locks.//

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <stdbool.h>
#include <time.h>

int nResources,nProcesses;
int *resources;
int **allocated;
int **maxRequired;
int **need;
int *safeSeq;
int nProcessRan = 0;

pthread_mutex_t lockResources;
pthread_cond_t condition;

bool getSafeSeq();
void* processCode(void* );

int main(int argc, char** argv) {
	srand(time(NULL));

        printf("\nEnter no. of process: ");
        scanf("%d", &nProcesses);

        printf("\nEnter no. of resource: ");
        scanf("%d", &nResources);
resources = (int *)malloc(nResources * sizeof(*resources));
        printf("\nCurrently Available resources : ");
        for(int i=0; i<nResources; i++)
                scanf("%d", &resources[i]);

        allocated = (int **)malloc(nProcesses * sizeof(*allocated));
        for(int i=0; i<nProcesses; i++)
                allocated[i] = (int *)malloc(nResources * sizeof(**allocated));

        maxRequired = (int **)malloc(nProcesses * sizeof(*maxRequired));
        for(int i=0; i<nProcesses; i++)
                maxRequired[i] = (int *)malloc(nResources * sizeof(**maxRequired));

                printf("\n");
        for(int i=0; i<nProcesses; i++) {
                printf("\nResource allocated to process %d : ", i+1);
                for(int j=0; j<nResources; j++)
                        scanf("%d", &allocated[i][j]);
        }
        printf("\n");

	
        for(int i=0; i<nProcesses; i++) {
                printf("\nMaximum resource required by process %d : ", i+1);
                for(int j=0; j<nResources; j++)
                        scanf("%d", &maxRequired[i][j]);
        }
        printf("\n");

	
        need = (int **)malloc(nProcesses * sizeof(*need));
        for(int i=0; i<nProcesses; i++)
                need[i] = (int *)malloc(nResources * sizeof(**need));

        for(int i=0; i<nProcesses; i++)
                for(int j=0; j<nResources; j++)
                        need[i][j] = maxRequired[i][j] - allocated[i][j];

	
	safeSeq = (int *)malloc(nProcesses * sizeof(*safeSeq));
        for(int i=0; i<nProcesses; i++) safeSeq[i] = -1;

        if(!getSafeSeq()) {
                printf("\nUnsafe State! The processes leads the system to a unsafe state.\n\n");
                exit(-1);
        }

        printf("\n\nSafe Sequence Found : ");
        for(int i=0; i<nProcesses; i++) {
                printf("%-3d", safeSeq[i]+1);
        }

        printf("\nExecuting Processes...\n\n");
        sleep(1);
	
	
	pthread_t processes[nProcesses];
        pthread_attr_t attr;
        pthread_attr_init(&attr);

	int processNumber[nProcesses];
	for(int i=0; i<nProcesses; i++) processNumber[i] = i;

        for(int i=0; i<nProcesses; i++)
                pthread_create(&processes[i], &attr, processCode, (void *)(&processNumber[i]));

        for(int i=0; i<nProcesses; i++)
                pthread_join(processes[i], NULL);

        printf("\nAll Processes Finished\n");	
	
