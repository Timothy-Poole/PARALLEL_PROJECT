# CVTREE PARALLELISATION
![CVTREE](./README_images/DNA_reduced.jpg)
### By Tim Poole
### n10476512
### CAB401 
### Unit Coordinator, Wayne Kelly  

---
## BUILD INSTRUCTIONS  

In order to run and compile this program simply (unzip the file first)
1. open the solution inside of visual studio  
2. uncomment out the version of the program you wish to run, impoved.cpp for the sequential version and final.cpp for the parallel version  
3. if running the parallel version then adjust the number of threads the program will use by adjusting the command 
omp_set_num_threads(x) located in line 287 within the main()  
4. ensure Visual Studio has open MP enabled, do this by right clicking on the project, then 
selecting project settings from the drop down menu, clicking on "All Options",
located within the "C/C++" section, scrolling down to "Open MP Support" and selecting yes 
from the dropdown  
5. lastly click on Debug located in the top menu bar, and from the drop down select Performance Profiler  
6. Tick the "CPU Usage" box and then click start and the program will compile, run, and print the output 
to the console.  

---
## TOOLS & TECHNIQUES USED  

![RYZEN 5950X](./README_images/AMD-Ryzen-5000-Series-Ryzen-9.png)  

I ran all these test on my home computer running windows 10 on a Ryzen 5950x Processor which has 16 cores and 32 threads.  

The project itself was run inside of a solution (.sln) inside of Visual Studio. The decision to use Visual Studio was made primarily to 
make use of Visual Studio's inbuilt Performance Profiler. The Profiler was set to monitor CPU usage, and used to test and benchmark 
each iteration of the program from the sequential version (located in improved.cpp), to each iteration of the parallel version 
(located in final.cpp), With the number of threads increased progressively in order to test the affect that more cores, and
thus more threads, would have on the results.

Open Mp was the threading library used to parallelize the program. Included using code below.
```c++
#include <omp.h> 
```
The major command used throughout the program in order to parallelize the program was the omp parallel for command.
```c++
#pragma omp parallel for 
```
This command was used extensively to multithread every instance of a for or nested for loop within the program where 
there was **SUFFICIENT BURDEN** on the CPU to justify its use. However not every for loop required parallelization.
an example of a section where the use of this command
was deemed **UNESSECARY** is outlined below (from lines 13-24 of improved.cpp).  

```c++
#define LEN			6
#define AA_NUMBER		20
#define	EPSILON			1e-010

void Init()
{
	M2 = 1;
	for (int i = 0; i < LEN - 2; i++)	// M2 = AA_NUMBER ^ (LEN-2);
		M2 *= AA_NUMBER;
	M1 = M2 * AA_NUMBER;		// M1 = AA_NUMBER ^ (LEN-1);
	M = M1 * AA_NUMBER;			// M  = AA_NUMBER ^ (LEN);
}
```  

This for loop will only run 6 times as i will only go until it is just less than LEN which in this case is defined as equal
to 6. The reasoning behind this decision is that the impact of adding parallelization here will either have a negligible or
negative overall impact on the performance of the program. A negative impact would most likely be due to the cost and overhead
'or setting up the multithreading in the first place out ways the benefits up on each succesive iteration of the for loop.  

---
## SEQUENTIAL PROGRAM  

The cvtree project, originally developed by Dr Wayne Kelly of the Queensland University of Technology,
is a small sequential application written in C++ that aims to find the similarity between disparite genomes
using frequency vectors. the code (located in improved.cpp) will first read a basic text file called list.txt
containing the number of Bacteria and there names and will stores these names so that it can then retrive the relevant data
from each bacterias corresponding fasta file located within the data directory. Each fasta file is a Bacteria and consist of many genes, each gene
in turn is comprised of many proteins called amino acids. There are 20 commonly occuring amino acids each represented by a letter of 
the alphabet. Instead of comparing each bacteria's genes, parts of the genomes called K-mers can be used to statistically determine
'how closely related the bacteria are to one another. a K-mer is a sub-section of a gene and the K represents the number of amino acids
that are grouped together for analysis. In this case Dr Kelly has used 6-mers to group together the amino acids and traverse the gene. 

the original sequential application (sourcecode located in improved.cpp) begins execution within the main function
(code snippet included below)  
```c++
int main(int argc,char * argv[])
{
	time_t t1 = time(NULL);

	Init();
	ReadInputFile("list.txt");
	CompareAllBacteria();

	time_t t2 = time(NULL);
	printf("time elapsed: %lld seconds\n", t2 - t1);
	return 0;
}
```  
The folowing diagram concisely sums up the key components of the main function and helps to further
breakdown the program into its 3 major sections, the Init, ReadInputFile and CompareAllBacteria
Functions.  
![FLOWCHART](./README_images/FLOW%20CHART.png)  

---
## CHALLENGES  

The greatest challenge that I faced during this project was figuring out how the sequential program worked, and more sepcifically
how the data dependencies flowed 
throughout the program, detangelingling the web of nested functionality. Furthermore I struggled initially to
understand which parts of the program could even be parallezied succesfully, mostly due to the previously mentioned data dependency problem and wether 
or not those parallel regions would require synchronisation or could be performed out of order. I solved this problem by using Visual
Studios debugging capabilities to step through the code so I could follow the main thread as it executed the code and used the Profiler to see where
the program was spending the most amount of time.

---
## POTENTIAL PARALLELISM

b. Your analysis of potential parallelism within the application. This might include
identification of existing loops or control flow constructs where parallelism might be
found. Explanation of the data and control dependences that you analysed to determine
which sections of code were safe to parallelize. Which of these is likely to be of sufficient
granularity to be worth exploiting? Is it scalable parallelism? A discussion of changes
required to expose parallelism, such as replacing algorithms or code restructuring
transformations.

After anlalysing the codebase I concluded that there were 3 major chnages that could be implemented to improve the performance of the program  

---
## THE SOLUTION

h. An explanation of the code that you added or modified to parallelize the application
(including source code line count).

```c++
int main(int argc, char* argv[])
{

	time_t t1 = time(NULL);
	omp_set_num_threads(10);

	Init();
	ReadInputFile("list.txt");
	double** array = new double* [number_bacteria];

	for (int i = 0; i < number_bacteria; i++) {
		array[i] = new double[number_bacteria];
	}
	CompareAllBacteria(array);

	time_t t2 = time(NULL);
	printf("time elapsed: %lld seconds,  finished analysis completely\n", t2 - t1);
	// time not required after this p;oint as the data has
	// already been saved in the array
	for (int i = 0; i < number_bacteria - 1; i++)
		for (int j = i + 1; j < number_bacteria; j++)
		{
			double x = array[i][j];
			printf("%2d %2d -> %.20lf\n", i, j, x);
		}
	return 0;
}
```

```c++
void CompareAllBacteria(double** array)
{
	time_t t1 = time(NULL);

	Bacteria** b = new Bacteria * [number_bacteria];

# pragma omp parallel for
	for (int i = 0; i < number_bacteria; i++)
	{
		printf("load %d of %d on thread %d of %d threads using, with %d threads total\n", i + 1, number_bacteria, omp_get_thread_num(), omp_get_num_threads(), omp_get_num_procs());
		b[i] = new Bacteria(bacteria_name[i]);
	}

	time_t t2 = time(NULL);
	printf("time elapsed: %lld seconds,  finished loading Bacteria\n", t2 - t1);


	for (int i = 0; i < number_bacteria - 1; i++)
# pragma omp parallel for
		for (int j = i + 1; j < number_bacteria; j++)
		{
			//printf("%2d %2d -> ", i, j);
			double correlation = CompareBacteria(b[i], b[j]);
			//printf("%.20lf on thread %d of %d threads on %d cors\n", correlation, omp_get_thread_num(), omp_get_num_threads(), omp_get_num_procs());
			array[i][j] = correlation;
		}
}
```  

---
## RESULTS  

The Sequential program running as a single process with a single thread was able to execute on my Desktop
computer at home in 37 seconds and only used 3% of the total CPU power available on the machine as it was
bound to only one thread.  

|   No. Threads |  No. Cores |Execution Time (s)  |  CPU % Utilization  |  Speed Up Achieved  |
|---------------|------------|-----------------------|------------------|---------------------|
|  1            | 1          | 37                    |         3        |      N/A            |  

This is in Stark Contrast to the speed up achieved by using 4 cores or more. Specifically the best results
which were produced when using 12 cores which speedup execution time by a factor of x3.  

|   No. Threads |  No. Cores  |Execution Time (s)  |  CPU % Utilization  |  Speed Up Achieved  |
|----------|------|-----|-----------------------|-------------|
|  2       |   1   | 21  |         6            |      1.761     |
|  4       |   2   | 19  |         13           |      1.947     |
|  6       |   3   | 13  |         18           |      2.846     |
|  8       |   4   | 13  |         25           |      2.846     |
|  10      |   5   | 13  |         32           |      2.846     |
|  12      |   6   | 13  |         38           |      2.846     |
|  16      |   8   | 13  |         50           |      2.846     |
|  20      |   10  | 13  |         63           |      2.846     |
|  24      |   12  | 12  |         75           |      3.083     |
|  28      |   14  | 13  |         89           |      2.846     |
|  32      |   16  | 13  |         100          |      2.846     |

From the spoeedup graph below, a strange phenomenon can be observed.  

![EVERTON DIGITAL](./README_images/Speed%20Up%20Graph.png)  

While adding more cores does increase performance, there seems to be drastically diminishing returns beyond a certain point.
This begs the question why did more processing power not continue to deliver better results in a linear fashion. To answer this question
I have provided the Performance Profilers CPU % graphs obtained from the sequential program, and 1, 2, 3 and 12 core run throughs of the program
respectively.  

![SEQUENTIAL](./README_images/diagseq.JPG)
![1 CORE](./README_images/diag1.JPG)
![2 CORE](./README_images/diag2.JPG)
![3 CORE](./README_images/diag3.JPG)
![12 CORE](./README_images/diag12.JPG)  

As you can see the sequential graph mostly appears to be a flat line from start to finish which was entirely expected, however as
the number of cores available increases the program as a whole executes in less time, but more interestingly to major zones appear.  

e. How did you test that the parallel version produced the exact same results as the original
sequential version?  

---
## OUTCOME  

This project has yielded a few major learning outcomes for me. Firstly, prior to using Open MP I had only used
the POSIX thread library (pthreads.h) which is an explicit threading library. Explicit Libraries put more
of the onus on the programmer to manually configure each thread and the memory associated with them. This
allows for much more granular control over how the program is threaded, however explicit threading libraries are
far more difficult to implement and debug, consuming far more of the prgrammers time and energy. In comparison I have been pleasantly 
surprised by how easy and effective implementing Open MP has been. Furthermore the resources online about Open Mp
seem far more accesible to programmers with lower levels of competancy and are generally easier to access, 
which is not the case with pthreads.  

Furthermore I have enjoyed learning about how to analyse flow control within a program and follow the data dependencies to
determine where best to apply parallelism to achieve higher performance. Previously when developing programs I **NEVER** stopped
to consider that by default most if not all applications are developed sequentially without taking into consideration parallelism 
and thus many programmers are leaving otherwise fairly achievable performance "on the table".  

The final speedup achieved was approximately a x3 increase. This is a fairly 
good increase in speedup however I did expect that with more cores the speedup would have continued to increase
further instead of stabilising so quickly after only 3 cores. This however probably highlights that more parallelism could
have been added to the first half of the problem where the speedup was less impressive. However, this could also 
be the result of a small data set. The area I identified as most computational intensive received the biggest increase in
performance. If this test were to be performed on a larger dataset, for example 400 bacteria as opposed to only 41,  I strongly 
believe the difference between the sequential
program and the parallel programs would have been far greater. Furthermore I beleive that the difference would hasve grown exponentially. 
It is my hypothesis that more cores do make a significant difference but in this limited experiment were not fully utilized.  

Beyond trying to further improve the program's performance with parallelism using Open MP, This report could
be extended in future to compare the speedup achieved by an implicit threading library, like "Open MP", vs an Explicit 
threading Library such as "Microsoft Windows Native threads". Such a comparison would be really insightful as it could
help programmers understand the pros and cons of each approach and wether or not investing more time into using explicit
threading is worth the hassle.

---
## APPENDICE

### Init  

```c++
void Init()
{
	M2 = 1;
	for (int i=0; i<LEN-2; i++)	// M2 = AA_NUMBER ^ (LEN-2);
		M2 *= AA_NUMBER;
	M1 = M2 * AA_NUMBER;		// M1 = AA_NUMBER ^ (LEN-1);
	M  = M1 *AA_NUMBER;			// M  = AA_NUMBER ^ (LEN);
}
```  

![Init](./README_images/Blank%20diagram.png)  

### ReadInputFile  

```c++
void ReadInputFile(const char* input_name)
{
	FILE* input_file;
	errno_t OK = fopen_s(&input_file, input_name, "r");

	if (OK != 0)
	{
		fprintf(stderr, "Error: failed to open file %s (Hint: check your working directory)\n", input_name);
		exit(1);
	}

	fscanf_s(input_file, "%d", &number_bacteria);
	bacteria_name = new char*[number_bacteria];

	for(long i=0;i<number_bacteria;i++)
	{
		char name[10];
		fscanf_s(input_file, "%s", name, 10);
		bacteria_name[i] = new char[20];
		sprintf_s(bacteria_name[i], 20, "data/%s.faa", name);
	}
	fclose(input_file);
}
```  

![ReadInputFile](./README_images/2.png)  

### CompareAllBacteria  

```c++
void CompareAllBacteria()
{
	Bacteria** b = new Bacteria*[number_bacteria];
	for(int i=0; i<number_bacteria; i++)
	{
		printf("load %d of %d\n", i+1, number_bacteria);
		b[i] = new Bacteria(bacteria_name[i]);
	}

	for(int i=0; i<number_bacteria-1; i++)
		for(int j=i+1; j<number_bacteria; j++)
		{
			printf("%2d %2d -> ", i, j);
			double correlation = CompareBacteria(b[i], b[j]);
			printf("%.20lf\n", correlation);
		}
}
```  

![CompareAllBacteria](./README_images/3.png)
