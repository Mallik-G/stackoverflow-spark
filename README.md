# Clustering StackOverflow Q&A By Programming Language and Answer Votes

This repo contains my solution for an assignment from the Coursera course [Big Data Analysis with Spark and Scala](https://www.coursera.org/learn/scala-spark-big-data/home/info).

## Results

The k-means clustering took 44 iterations to converge. There are 45 clusters:

  Median Votes (on answers) | Dominant language (%) | #Questions
----------------------------|-----------------------|---------------
0 | MATLAB (100.0%) | 3725
1 | CSS (100.0%) | 113598
1 | Groovy (100.0%) | 2729
1 | C# (100.0%) | 361835
1 | Ruby (100.0%) | 54727
1 | PHP (100.0%) | 315734
1 | Objective-C (100.0%) | 94617
1 | Java (100.0%) | 383473
1 | JavaScript (100.0%) | 365647
2 | Perl (100.0%) | 19229
2 | MATLAB (100.0%) | 10656
2 | C++ (100.0%) | 181268
2 | Scala (100.0%) | 12472
2 | Clojure (100.0%) | 3324
2 | Python (100.0%) | 174573
4 | Haskell (100.0%) | 10362
9 | Perl (100.0%) | 4714
10 | Groovy (100.0%) | 310
12 | Clojure (100.0%) | 712
27 | Scala (100.0%) | 679
34 | MATLAB (100.0%) | 107
53 | Haskell (100.0%) | 202
61 | Groovy (100.0%) | 14
66 | Clojure (100.0%) | 57
77 | Perl (100.0%) | 58
79 | C# (100.0%) | 2585
85 | Ruby (100.0%) | 648
89 | Objective-C (100.0%) | 903
130 | Scala (100.0%) | 47
135 | PHP (100.0%) | 512
172 | CSS (100.0%) | 358
223 | C++ (100.0%)  | 251
223 | Python (100.0%) | 413
249 | Java (100.0%) | 483
375 | JavaScript (100.0%) | 433
443 | C# (100.0%) | 147
473 | Objective-C (100.0%) | 82
546 | Ruby (100.0%) | 34
766 | CSS (100.0%) | 26
887 | PHP (100.0%) | 13
1130 | Haskell (100.0%) | 2
1269 | Python (100.0%) | 19
1290 | C++ (100.0%) | 9
1895 | JavaScript (100.0%) | 33
10271 | Java (100.0%) | 2

Total run time is 193s.

## Questions

Do you think that partitioning your data would help?

> Yes, partitioning by programming language helps because all questions in a cluster are tagged with the same language. This is true for all clusters. Partitioning reduces shuffling when the data is grouped by their clusters to compute their new centroids. After partitioning, run time dropped dramatically from 1,201s to 193s --- a 6x speedup! 

Have you thought about persisting some of your data? Can you think of why persisting your data in memory may be helpful for this algorithm?

> Persisting `vectors` in memory helps because it can be reused in the iterative k-means algorithm.

Of the non-empty clusters, how many clusters have "Java" as their label (based on the majority of questions, see above)? Why?

> 3 clusters are labelled Java.

Only considering the "Java clusters", which clusters stand out and why?

> There is a huge cluster with more than 380K questions but the median votes on the answers is only 1. This suggests beginner or RTFM questions. On the other hand, there is a tiny cluster with only 2 questions but the median votes on the answers is a whopping 10K. 

How are the "C# clusters" different compared to the "Java clusters"?

Java clusters - all questions in these clusters are tagged as Java

<table>
  <tr>
    <td>Median votes (answers)</td>
    <td>#questions</td>
  </tr>
  <tr>
    <td>1</td>
    <td>383,473</td>
  </tr>
  <tr>
    <td>249</td>
    <td>483</td>
  </tr>
  <tr>
    <td>10,271</td>
    <td>2</td>
  </tr>
</table>


C# clusters - all questions in these clusters are tagged as C#

<table>
  <tr>
    <td>Median votes (answers)</td>
    <td>#questions</td>
  </tr>
  <tr>
    <td>1</td>
    <td>361,835</td>
  </tr>
  <tr>
    <td>79</td>
    <td>2,585</td>
  </tr>
  <tr>
    <td>443</td>
    <td>147</td>
  </tr>
</table>


> Java clusters have higher quality answers than the C# clusters, as indicated by the higher #median votes. This suggests that there are more experienced Java programmers than C#, which may not be surprising since Java is slightly older than C#. Or it could also mean that the Java community is larger than C#. Either way, both communities seem to be very healthy as there are a lot of questions and highly voted answers.

## Instructions

To start, first download the assignment: [stackoverflow.zip](http://alaska.epfl.ch/~dockermoocs/bigdata/stackoverflow.zip). For this assignment, you also need to download the data (170 MB):

http://alaska.epfl.ch/~dockermoocs/bigdata/stackoverflow.csv

and place it in the folder: `src/main/resources/stackoverflow` in your project directory.

The overall goal of this assignment is to implement a distributed k-means algorithm which clusters posts on the popular question-answer platform StackOverflow according to their score. Moreover, this clustering should be executed in parallel for different programming languages, and the results should be compared.

The motivation is as follows: StackOverflow is an important source of documentation. However, different user-provided answers may have very different ratings (based on user votes) based on their perceived value. Therefore, we would like to look at the distribution of questions and their answers. For example, how many highly-rated answers do StackOverflow users post, and how high are their scores? Are there big differences between higher-rated answers and lower-rated ones?

Finally, we are interested in comparing these distributions for different programming language communities. Differences in distributions could reflect differences in the availability of documentation. For example, StackOverflow could have better documentation for a certain library than that library's API documentation. However, to avoid invalid conclusions we will focus on the well-defined problem of clustering answers according to their scores.

### The Data

You are given a CSV (comma-separated values) file with information about StackOverflow posts. Each line in the provided text file has the following format:

```

<postTypeId>,<id>,[<acceptedAnswer>],[<parentId>],<score>,[<tag>]

```

A short explanation of the comma-separated fields follows.

```

<postTypeId>:     Type of the post. Type 1 = question, 

                  type 2 = answer.

                  

<id>:             Unique id of the post (regardless of type).

<acceptedAnswer>: Id of the accepted answer post. This

                  information is optional, so maybe be missing 

                  indicated by an empty string.

                  

<parentId>:       For an answer: id of the corresponding 

                  question. For a question:missing, indicated

                  by an empty string.

                  

<score>:          The StackOverflow score (based on user 

                  votes).

                  

<tag>:            The tag indicates the programming language 

                  that the post is about, in case it's a 

                  question, or missing in case it's an answer.

```

You will see the following code in the main class:

```scala

  val lines   = sc.textFile("src/main/resources/stackoverflow/stackoverflow.csv")  

  val raw     = rawPostings(lines)  

  val grouped = groupedPostings(raw)  

  val scored  = scoredPostings(grouped)  

  val vectors = vectorPostings(scored)

```

It corresponds to the following steps:

`lines`: the lines from the csv file as strings

`raw`: the raw Posting entries for each line

`grouped`: questions and answers grouped together

`scored`: questions and scores

`vectors`: pairs of (language, score) for each question

The first two methods are given to you. You will have to implement the rest.

### Data processing

We will now look at how you process the data before applying the kmeans algorithm.

Grouping questions and answers

The first method you will have to implement is groupedPostings:

```scala

val grouped = groupedPostings(raw)

```

In the raw variable we have simple postings, either questions or answers, but in order to use the data we need to assemble them together. Questions are identified using a `postTypeId == 1`. Answers to a question with `id == QID` have (a) `postTypeId == 2` and (b) `parentId == QID`.

Ideally, we want to obtain an RDD with the pairs of `(Question, Iterable[Answer])`. However, grouping on the question directly is expensive (can you imagine why?), so a better alternative is to match on the QID, thus producing an `RDD[(QID, Iterable[(Question, Answer))]`.

To obtain this, in the `groupedPostings` method, first filter the questions and answers separately and then prepare them for a join operation by extracting the QID value in the first element of a tuple. Then, use one of the join operations (which one?) to obtain an `RDD[(QID, (Question, Answer))]`. Then, the last step is to obtain an `RDD[(QID, Iterable[(Question, Answer)])]`. How can you do that, what method do you use to group by the key of a pair RDD?

Finally, in the description we made QID, Question and Answer separate types, but in the implementation QID is an Int and both questions and answers are of type Posting. Therefore, the signature of groupedPostings is:

```scala

def groupedPostings(postings: RDD[/* Question or Answer */ Posting]): 

    RDD[(/*QID*/ Int, Iterable[(/*Question*/ Posting, /*Answer*/ Posting)])]

```

This should allow you to implement the `groupedPostings` method.

### Computing Scores

Second, implement the `scoredPostings` method, which should return an RDD containing pairs of (a) questions and (b) the score of the answer with the highest score (note: this does not have to be the answer marked as "acceptedAnswer"!). The type of this scored RDD is:

```scala

val scored: RDD[(Posting, Int)] = ???

```

For example, the scored RDD should contain the following tuples:

```scala

((1,6,None,None,140,Some(CSS)),67)

((1,42,None,None,155,Some(PHP)),89)

((1,72,None,None,16,Some(Ruby)),3)

((1,126,None,None,33,Some(Java)),30)

((1,174,None,None,38,Some(C#)),20)

```

Hint: use the provided `answerHighScore` given in `scoredPostings`.

Creating vectors for clustering

Next, we prepare the input for the clustering algorithm. For this, we transform the scored RDD into a vectors RDD containing the vectors to be clustered. In our case, the vectors should be pairs with two components (in the listed order!):

Index of the language (in the `langs` list) multiplied by the `langSpread` factor.

The highest answer score (computed above).

The `langSpread` factor is provided (set to 50000). Basically, it makes sure posts about different programming languages have at least distance 50000 using the distance measure provided by the `euclideanDist` function. You will learn later what this distance means and why it is set to this value.

The type of the vectors RDD is as follows:

```scala

val vectors: RDD[(Int, Int)] = ???

```

For example, the vectors RDD should contain the following tuples:

```

(350000,67)

(100000,89)

(300000,3)

(50000,30)

(200000,20)

```

Implement this functionality in method `vectorPostings` and by using the given the `firstLangInTag` helper method.

(Idea for test: `scored` RDD should have 2121822 entries)

### Kmeans Clustering

```scala

 val means = kmeans(sampleVectors(vectors), vectors)

```

Based on these initial means, and the provided variables converged method, implement the K-means algorithm by iteratively:

pairing each vector with the index of the closest mean (its cluster);

computing the new means by averaging the values of each cluster.

To implement these iterative steps, use the provided functions `findClosest`, `averageVectors`, and `euclideanDistance`.

Note 1:

In our tests, convergence is reached after 44 iterations (for `langSpread=50000`) and in 104 iterations (for `langSpread=1`), and for the first iterations the distance kept growing. Although it may look like something is wrong, this is the expected behavior. Having many remote points forces the kernels to shift quite a bit and with each shift the effects ripple to other kernels, which also move around, and so on. Be patient, in 44 iterations the distance will drop from over 100000 to 13, satisfying the convergence condition.

If you want to get the results faster, feel free to downsample the data (each iteration is faster, but it still takes around 40 steps to converge):

```scala

val scored  = scoredPostings(grouped).sample(true, 0.1, 0)

```

However, keep in mind that we will test your assignment on the full data set. So that means you can downsample for experimentation, but make sure your algorithm works on the full data set when you submit for grading.

Note 2:

The variable `langSpread` corresponds to how far away are languages from the clustering algorithm's point of view. For a value of 50000, the languages are too far away to be clustered together at all, resulting in a clustering that only takes scores into account for each language (similarly to partitioning the data across languages and then clustering based on the score). A more interesting (but less scientific) clustering occurs when `langSpread` is set to 1 (we can't set it to 0, as it loses language information completely), where we cluster according to the score. See which language dominates the top questions now?

### Computing Cluster Details

After the call to `kmeans`, we have the following code in method `main`:

```scala

val results = clusterResults(means, vectors)

printResults(results)

```

Implement the `clusterResults` method, which, for each cluster, computes:

(a) the dominant programming language in the cluster;

(b) the percent of answers that belong to the dominant language;

(c) the size of the cluster (the number of questions it contains);

(d) the median of the highest answer scores.

Once this value is returned, it is printed on the screen by the printResults method.

