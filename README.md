# Parallel K-Means

## Analysis of the serial algorithm 
#### What is a clustering algorithm?
A clustering algorithm is a method used in order to process data into clusters. There are several ways to do so and the chosen algorithm for the project is the K-Means. The K-Means algorithm starts with a given number of points (N) and a given number of clusters (K). The score subdivides the dataset into  K different  clusters. In order to do this, I initialize K of the N points as centroids (a centroid is the center of a cluster).The distance from every point to every centroid is iteratively calculated in order to bind the point to the nearest cluster. At the end of each iteration the coordinates of each centroid is updated.  
If the stop criteria are not satisfied it’s computed again the iteration, which means: 
Calculate which cluster the points belong to.
Update the centroids.
Stop criteria. 
- The exit conditions are represented by one of the following options:
- There is no datapoint that changes the cluster from one iteration to another.
- We reach the maximum number of iterations.
- The error is small enough. 
## A-priori study of available parallelism

It was decided to represent points and centroids as structs.
I took the decision of representing points and centroid with different structures because they have different attributes and different meanings.
The main functions implemented are the following:
findEuclidianDistance();
processClusterSerial();
replaceCentroid();
The most interesting function to analyze is processClusterSerial. This function is in charge of finding which cluster every point belongs to , updating the centroids and monitoring when the stop criteria are satisfied.
The main part of the function is a while loop, which counts the number of iterations as we described below. Inside this while loop there is a for loop that calculates for each point  the distance from all centroids and matches the point with the nearest cluster. At the end of the for, but still inside the while loop, the centroids will be updated. 

#### Prior Speed-up

Amdahl law : Speedup= 1/[(1-Parallelizable code)+ (Parallelizable code/number of threads)]

In order to better use the profiling tools, I decided to put most of the code inside functions so that they could be visualized and profiled by the various tools used. Some of these functions were then removed in the parallel version, to avoid continual program counter changes, the relative jumps and therefore a loss of pipeline efficiency.
To calculate the speed-up I decided to rely on a tool called valgrind and gprof.

## OpenMP parallel implementation
The parallel implementation is the following:
The while loop, as mentioned, could not be divided into independent iterations, due to the updating of the coordinates of the centroids. So the first pragma was inserted immediately after the while, where the section to be parallelized included the search for the minimum distance and the sum of the coordinates.
When threads are instantiated, the default static scheduling assigns each thread a chunk of the same size. Assigning the chunk there are some problems of sharing the support variables used for the correct execution of the program.
The isChanged variable is shared among all threads since it represents one of the exit criteria and all threads can change it.
Regarding the internal variables of the threads,the private ones are respectively : i. and j. , the indexes of the cycles, the variables useful to assign the minimum distance (min distance, current_distance) and the support variables useful to calculate the sum of the points and the sum of the coordinates (p_sum_coordinates_matrix, p_sum_points).
When the sum of the coordinates has to be saved, problems of simultaneous access to the same resource arise so if they’re not managed with a critical pragma they could lead to the calculation of a wrong result and therefore to a wrong output.
This was the first and only parallelization step shown since the parallelization of the initial coordinate (resetCoordinate), the for loop and the function replaceCentroid have too few iterations to give a substantial speed-up.
The advantage of this implementation is certainly the simplicity of reading and developing the parallelization. On the other hand, an intrinsic drawback in the structure of the algorithm is the fact that every time the while ends a loop, all the threads are destroyed and the next loop is recreated by adding overheads to each iteration that prevent the theoretical speed-up from being reached.
## Testing and debugging
To debug the parallel code,I decided to rely on some python libraries that implement the algorithm. The results refers to respectively the parallel and serial version 
## Performance analysis
The performances were measured on multiple datasets. I decided to show only results on large enough datasets, in order to better appreciate all the advantages deriving from parallelization. For the calculations and performance tests through the google cloud platform, I instantiated e2-custom (24 vCPU, 12 GB RAM). The measurements as the cores changed were carried out up to 24 cores (which are the actual ones of my machine) in order to assign 1 thread to each core,  avoid the virtualization of some threads and, consequently, reduce possible overhead. Since I chose large datasets as a base,I decided to try various measurements by always increasing the size. In particular through a python program I created 3 datasets, respectively of 500K, 1M and 2M of three-dimensional points. 
