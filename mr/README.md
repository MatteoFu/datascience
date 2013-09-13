# MapReduce Laboratory

In this laboratory students will learn how to use the [Hadoop][hadoop] client API by working on a series of exercises:

+ The classic Word Count and variations on the theme
+ Design Pattern: Pair and Stripes
+ Design Pattern: Order Inversion
+ Join implementations

Note that the two design patterns outlined above have been originally discussed in:

+ Jimmy Lin, Chris Dyer, **Data-Intensive Text Processing with MapReduce**, Morgan Claypool ed. [link][jimmilin]

[hadoop]: http://hadoop.apache.org "hadoop"
[jimmilin]: http://lintool.github.io/MapReduceAlgorithms/index.html

## Setup the laboratory sources in Eclipse:

The first step is to create a new Java Project in Eclipse:

- Inside Eclipse select the menu item **File > New > Project ...*** to open the **New Project** wizard.
- Select **Java Project** then click **Next** to start the **New Java Project**
- Type a name for your new project, such as ''mr-lab''
- Ensure to use *JavaSE-1.6* as JRE, then click on **Next**
- Go on **Libraries** tab, and click on **Add External Jars**
- Add the following jars:
 - /usr/lib/hadoop/hadoop-common.jar
 - /usr/lib/hadoop-0.20-mapreduce/hadoop-core.jar
 - /usr/lib/hadoop-hdfs/hadoop-hdfs.jar
- Click on **Finish**

Now you have a new project.

The next step is to create a new **Class** for each class you want to develop.
- Inside Eclipse select the menu item **File > New > Class**

At this point you should have a java project named mapred-lab already configured that compiles. The next step is starting with the first exercise.

## How to launch a job
In order to launch a job, you need first to export a **JAR** file. Therefore, from Eclipse:
- Select the menu item **File > Export**
- Type **JAR**, select **JAR file** and click on **Next**
- In the textbox **Select the export destination**, type the name of the new **JAR** you want to export, i.e. *'/home/student/mrlab.jar'*
- Type **Finish**

Once you have exported your jar file, you can run your code using the local version of Hadoop.
Open a *terminal*, and type:
```
hadoop jar <jarname.jar> <fully.qualified.class.Name> <Parameters>
```

For example, for running the *WordCount* exercise, type:
```
hadoop jar mrlab.jar fr.eurecom.dsg.mapreduce.WordCount 2 INPUT/text/quote.txt OUTPUT/wordcount/
```

Note that you need to specify a *non existing* output directory, or to delete it before running the job.

## Web interfaces: monitor job progress

Hadoop publishes some web interfaces that display JobTracker and HDFS statuses.
You can access them using the following links:

- Jobtracker Web Interface: http://127.0.0.1:50030/
- NameNode Web Interface: http://127.0.0.1:50070/

# Exercises
Note, exercises are organized in ascending order of difficulty.

## EXERCISE 1:: Word Count

Count the occurrences of each word in a text file. This is the simplest example of MapReduce job: in the following we illustrate three possible implementations.

+ **Basic**: the map method emits 1 for each word. The reduce aggregates the ones it receives from mappers for each key it is responsible for and save on disk (HDFS) the result
+ **In-Memory Combiner**: instead of emitting 1 for each encountered word, the map method (partially) aggregates the ones and emit the result for each word
+ **Combiner**: the same as In-Memory Combiner but using the MapReduce Combiner class


### Instructions

For the **basic** version the student has to modify the file *WordCount.java* in the package *fr.eurecom.dsg.mapreduce*. The user must operate on each TODO filling the gaps with code, following the description associated to each TODO. The package *java.util* contains a class *StringTokenizer* that can be used to tokenize the text.

For The **In-Memory** Combiner and the **Combiner** the student has to modify *WordCountIMC.java* and *WordCountCombiner.java* in the same package referenced aboce. The student has to operate on each TODO using the same code of the basic word count example except for the TODOs marked with a star *. Those must be completed with using the appropriate design pattern.

When an execise is completed you can export it into a jar file that will then be used to execute it.

### Example of usage
The final version should get in input three arguments: the number of reducers, the input file and the output path. Example of execution are:

```
hadoop jar <compiled_jar> fr.eurecom.dsg.mapreduce.WordCount 3 <input_file> <output_path>
hadoop jar <compiled_jar> fr.eurecom.dsg.mapreduce.WordCountIMC 3 <input_file> <output_path>
hadoop jar <compiled_jar> fr.eurecom.dsg.mapreduce.WordCountCombiner 3 <input_file> <output_path>
```

To test your code use the file `/user/student/INPUT/text/quote.txt`, saved in the local HDFS fs.

To run the final version of your job, you can use a bigger file, `/user/student/INPUT/text/gutenberg-partial.txt`, which contains an extract of the English books from Project Gutenberg http://www.gutenberg.org/, which provides a collection of full texts of public domain books.

Note: in order to launch the job, refer to [How to launch a job](#how-to-launch-a-job)

### Questions ###

Answer the following questions (in a simple text file):

+ How does the number of reducers affect the general performances? How many reducers can be executed at the same time?
+ Use the JobTracker web interface to examinate the job counters: can you explain the differences among all the implementations? For example, look at the amount of bytes shuffled by Hadoop

> Zipf's law states that given some corpus of natural language utterances, the frequency of any word is inversely proportional to its rank in the frequency table. Thus the most frequent word will occur approximately twice as often as the second most frequent word, three times as often as the third most frequent word, etc. For example, in the *Brown Corpus of American English* text, the word "*the*" is the most frequently occurring word, and by itself accounts for nearly 7% of all word occurrences. The second-place word "*of*" accounts for slightly over 3.5% of words, followed by "*and*". Only 135 vocabulary items are needed to account for half the Brown Corpus. (wikipedia.org)

+ Can you explain how does the distribution of words affect the WordCount? In particular, think to the reducers.

## EXERCISE 2:: Term co-occurrences
In the following exercise, we need to build the term co-occurrence matrix for a text collection.
A co-occurrence matrix is a n x n matrix, where n is the number of unique words in the text. For each couple of words, we count the number of times they co-occurred in the text in the same line.

### Pairs Design Pattern

The basic (and maybe most intuitive) implementation of this exercise is the *Pair*.
The basic idea is to emit, for each couple of words in the same line, the couple itself (or *pair*) and the value 1.
For example, in the line `w1 w2 w3 w1`, we emit `(w1,w2):1, (w1, w3):1, (w2:w1):1, (w2,w3):1, (w2:w1):1, (w3,w1):1, (w3:w2):1, (w3,w1):1`.

In this exercise, we need to use a composite key to emit an occurrence of a pair of words. The student will understand how to create a custom Hadoop data type to be used as key type.

A Pair is a tuple composed by two elements that can be used to ship two objects within a parent object. For this exercise the student has to implement a TextPair, that is a Pair that contains two words.

#### Instructions
There are two files for this exercise:

+ *TextPair.java*: data structure to be implemented by the student. Besides the implementation of the data structure itself, the student has to implement the serialization Hadoop API (write and read Fields).
+ *Pair.java*: the implementation of a pair example using *TextPair.java* as datatype.

#### Example of usage
The final version should get in input three arguments: the number of reducers, the input file and the output path. Example of execution are:
```
hadoop jar <compiled_jar> fr.eurecom.dsg.mapreduce.Pair 1 <input_file> <output_path>
```

To test your code use the file `/user/student/INPUT/text/quote.txt`, saved in the local HDFS fs.

To run the final version of your job, you can use a bigger file, `/user/student/INPUT/text/gutenberg-partial.txt`.

Note: in order to launch the job, refer to [How to launch a job](#how-to-launch-a-job)

#### Questions
Answer the following questions (in a simple text file):

+ How does the number of reducer influence the behavior of the Pairs approach?
+ Why does `TextPair` need to be Comparable?


### Stripes Design Pattern
This approach is similar to the previous one: for each line, co-occurring pairs are generated. However, now, instead of emitting every pair as soon as it is generated, intermediate results are stored in an associative array. We use an associative array, and, for each word, we emit the word itself as key and a *Stripe*, that is the map of co-occurring words with the number of associated occurrence.

For example, in the line `w1 w2 w3 w1`, we emit `w1:{w2:1, w3:1}, w2:{w1:2,w3:1}, w3:{w1:2, w2:1}, w1:{w2:1, w3:1}`.
Note that, instead, we could emit also `w1:{w2:2, w3:2}, w2:{w1:2,w3:1}, w3:{w1:2, w2:1}`.

In this exercise the student will understand how to create a custom Hadoop data type to be used as value type.

#### Instructions
There are two files for this exercise:

+ *StringToIntMapWritable.java*: the data structure file, to be implemented
+ *Stripes.java*: the MapReduce job, that the student must implement using the StringToIntMapWritable data structure

#### Example of usage
```
hadoop jar <compiled_jar> fr.eurecom.dsg.mapreduce.Stripes 2 <input_file> <output_path>
```
To test your code use the file `/user/student/INPUT/text/quote.txt`, saved in the local HDFS fs.

To run the final version of your job, you can use a bigger file, `/user/student/INPUT/text/gutenberg-partial.txt`.

Note: in order to launch the job, refer to [How to launch a job](#how-to-launch-a-job)

#### Questions
Answer the following questions (in a simple text file):

+ Can you use the implemented reducers as *Combiner*?
+ Do you think Stripes could be used with the in-memory combiner pattern?
+ How does the number of reducer influence the behavior of the Stripes approach?
+ Using the [Jobtracker Web Interface](#web-interfaces-monitor-job-progress-), compare the shuffle phase of *Pair* and *Stripes*.
+ Why `StringToIntMapWritable` is not Comparable (differently from `TextPair`)?

Note: in order to launch the job, refer to [How to launch a job](#how-to-launch-a-job)

<!-- ## EXERCISE 4:: Order Inversion (Design Pattern) -->
## EXERCISE 4:: Relative term co-occurrence and Order Inversion Design Pattern
In this example we need to compute the co-occurrence matrix, like the one in the previous exercise, but using the relative frequencies of each pair, instead of the absolute value. Pratically, we need to count the number of times each pair *(w_i, w_j)* occurs divided by the number of total pairs with *w_i* (marginal).

The student has to implement the `Map` and `Reduce` methods and the special partitioner (see `OrderInversion#PartitionerTextPair` class), which apply the partitioner only according to the first element in the Pair, sending all data regarding the same word to the same reducer. Note that inside the `OrderInversion` class there is a field called `ASTERISK` which should be used to output the total number of occourrences of a word. Refer to the laboratory slides for more information.

### Instructions
There is one file for this exercise called `OrderInversion.java`. The `run` method of the job is already implemented, the student should complete the mapper, the reducer and the partitioner, as explained in the TODOs.

### Example of usage
```
hadoop jar <compiled_jar> fr.eurecom.fr.mapreduce.OrderInversion 4 <input_file> <output_path>
```

To test your code use the file `/user/student/INPUT/text/quote.txt`, saved in the local HDFS fs.

To run the final version of your job, you can use a bigger file, `/user/student/INPUT/text/gutenberg-partial.txt`.

Note: in order to launch the job, refer to [How to launch a job](#how-to-launch-a-job)

### Questions ###
Answer the following questions. In answering the questions below, consider the role of the combiner.

+ Do you think the Order Inversion approach is 'faster' than a naive approach with multiple jobs? For example, consider implementing a compound job in which you compute the numerator and the denominator separately, and then perform the computation of the relative frequency
+ What is the impact of the use of a 'special' compound key on the amounts of shuffled bytes?
+ How does the default partitioner works with `TextPair`? Can you imagine a different implementation that does not change the Partitioner?
+ For each key, the reducer receives its marginal before the co-occurence with the other words. Why?

## EXERCISE 5:: Joins
In MapReduce the term join refers to merging two different dataset stored as unstructured files in HDFS. As for databases, in MapReduce there are many different kind of joins, each with its use-cases and constraints. In this laboratory the student will implement two different kinds of MapReduce join techniques:

+ **Distributed Cache Join**: this join technique is used when one of the two files to join is small enough to fit (eventually in memory) on each computer of the cluster. This file is copied locally to each computer using the Hadoop distributed cache and then loaded by the map phase.
+ **Reduce-Side Join**: in this case the map phase tags each record such that records of different inputs that have to be joined will have the same tag. Each reducer will receive a tag with a list of records and perform the join.

### Jobs

+ **Distributed Cache Join**: implement word count using the distributed cache to exclude some words. The file in the HDFS `/user/student/INPUT/text/english.stop` contains the list of the words to exclude.
+ **Reduce Side Join**: You need to find the two-hops friends, i.e. the friends of friends of each user, in the twitter dataset. In particular, you need to implement a self-join, that is a join between two instances of the same dataset, on the twitter graph. To test your code use the file `/user/student/INPUT/twitter/twitter-small.txt`, saved in the local HDFS fs. To run the final version of your job, you can use a bigger file, `/user/student/INPUT/twitter/twitter-big-sample.txt`. Both files contain lines in the form _userid_ _friendid_.

### Instructions

+ **Distributed Cache Join**: you can start from the file *DistributedCacheJoin.java*. See also [<a href="https://hadoop.apache.org/common/docs/r0.20.2/api/org/apache/hadoop/filecache/DistributedCache.html">DistributedCache API</a>] as reference.
+ **Reduce Side Join**: use the file *ReduceSideJoin.java* as starting point. This exercise is different from the others because it does not contain any information on how to do it. The student is free to choose how to implement it.

### Example of usage
```
hadoop jar <compiled_jar> fr.eurecom.fr.mapreduce.DistributedCacheJoin 1 <big_file> <small_file> <output_path>
hadoop jar <compiled_jar> fr.eurecom.fr.mapreduce.ReduceSideJoin 1 <input_file2> <input_file1> <output_path>
```

