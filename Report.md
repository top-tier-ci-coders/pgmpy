# Report for assignment 4

This is a report for assignment 4. We had problems finding a good open source project to work on, and spent lots of time trying several different ones. Because of this we also choose to include the projects we did not proceed with due to either unclear issues or code was too hard to make sense of since this was a part of the whole experience. 
## Project 1, the initial choice

Name: Apache Geode

URL: https://github.com/apache/geode

Apache Geode is a data management platform that provides real-time, consistent access to data-intensive applications throughout widely distributed cloud architectures. *_(Copied from original repo)_*

## Selected issue(s)

Title: GEODE-6103

URL: https://issues.apache.org/jira/browse/GEODE-6103


What:
We should refactor the CreateRegionCommand to build a RegionConfig object instead of RegionFunctionArgs.

Why:
We would like RegionConfig to be the source of truth across all commands. RegionFunctionArgs was created because the config objects were not available before. The uniform source of truth for our commands makes it possible for us to create a single object to be sent from a cli to a rest endpoint.

*_(Description copied from URL.)_*


## Onboarding experience

Did it build as documented?

**1. How easily can you build the project? Briefly describe if everything worked as documented or not:**

The Geode README file contains two chapters relevant for this question: **Building** and **Testing**. Both chapters forward you to their respective **.MD** file. For building they offer two options depending on the environment you wish to use. Either building from source using gradle or using IntelliJ. We opted for building from source. This was as simple as running

```./gradlew build```

**For everyone in our group it builds immediately without failures after approximately 18 minutes.**

Our issue is connected to the **geode-core** module so perhaps a ```./gradlew geode-core:build``` would suffice if the build is too long, but we do not know if the source code of the different modules are connected or not so a full build is worth the extra wait. _*This was not documented*_.

**2. Did you have to install a lot of additional tools to build the software? Were those tools well documented?**

If you wish to use IntelliJ then yes, but there's a whole chapter for that in the **Building** chapter. Otherwise, the only thing needed was JDK 1.8.  

**3. Were other components installed automatically by the build script?**

Yes, it used gradle so all dependencies were installed when running the build.

**4. Did the build conclude automatically without errors?**

Yes on the Ubuntu systems. On Mac we had a few Failures, see below.

**5. How well do examples and tests run on your system(s)?**

Perfectly on the Ubuntu systems. 6672 tests completed, 13 ignored and a result of 100% successful. On Mac we had 8 tests failing.

**6. Do you plan to continue or choose another project?**


After spending in total 8 hours on this project just looking into the source code and trying to figure out the issue, we decided to move on. Although the issue was still open and unassigned we later found a pull request that referenced this issue. We felt we did not want to proceed with this issue since things seemed to already be done. This should have been communicated more clearly in the issue tracker.  

## Project 2, second choice

Name: Apache Ignite

URL: https://github.com/apache/ignite

"Apache Ignite a memory-centric distributed database, caching, and processing platform for transactional, analytical, and streaming workloads delivering in-memory speeds at petabyte scale" *_(Copied from ignite documentation)_*

## Selected issue(s)

Title: IGNITE-11069

URL: https://issues.apache.org/jira/browse/IGNITE-11069

We should extract common logic from mvccSet(), mvccRemove() and mvccLock() methods and corresponding MVCC listeners: MvccUpdateLockListener, MvccRemoveLockListener and MvccAcquireLockListener. *_(Copied from Issue description)_*

Source file connected to the issue: https://github.com/apache/ignite/blob/master/modules/core/src/main/java/org/apache/ignite/internal/processors/cache/.java

## Onboarding experience

Did it build as documented?

**1. How easily can you build the project? Briefly describe if everything worked as documented or not:**

The project builds easily by typing ```mvn clean install -DskipTests -pl modules/core -am```. This took about 2 minutes. 


**2. Did you have to install a lot of additional tools to build the software? Were those tools well documented?**

No, nothing but maven is required for this project.


**3. Were other components installed automatically by the build script?**

Yes, every dependency was automatically built by maven when executing the command in **1.**

**4. Did the build conclude automatically without errors?**

Yes, it did, it just took around 2 minutes. 

**5. How well do examples and tests run on your system(s)?**

This is where the problems start. The tests are too long causing the system to be slow, and the tests never terminate. We ran ```mvn test``` for 1½ hours and still, it was running. This is not going to be viable as soon as we make changes and wanna run the tests. We looked further into the documentation, namely the **DEVNOTES.txt** file to see more information. It mentions that running ```mvn test -U -Plgpl,examples,-clean-libs,-release -Dmaven.test.failure.ignore=true -DfailIfNoTests=false -Dtest='org.apache.ignite.testsuites.IgniteBasicTestSuite'``` , which lets you decide to run a specific test suite instead of everything. However, this never seems to terminate either. 

Step into ```cd modules/core``` before executing the individual tets.

In order to reduce run-time of the tests we looked up which test files contain the GridCacheMapEntry class that we should refactor. These are however still slow and not all tests succeed. The files and comments are provided below.

To run specific tests use: ```mvn test -Dtest=<testfileName> -DfailIfNoTests=false```

Tests that the class appears in:
* AbstractDeadlockDetectionTest - Builds and succeeds in 1m 52s. However doesn't seem to print a testsummary of #tests, failed, ignored.
* CacheMvccTransactionsTest - Takes 35 min and fails
* GridHashMapLoadTest - Results in: **OutOfMemory GC overhead limit exceeded**
* GridCacheEntryMemorySizeSelfTest - Builds and succeeds in 1m 53s. However doesn't seem to print a testsummary of #tests, failed, ignored.
* GridCacheSetFailoverAbstractSelfTest - Builds and succeeds in 1m 51s. However doesn't seem to print a testsummary of #tests, failed, ignored.
* GridLocalCacheStorageManagerDeserializationTest
* IgnitePartitionedWueueNoBackupTest
* igniteTxStoreExceptionAbstractSelfTest

Turns out that GridCacheMapEntry.java is an abstract class. Which means and that these test files aren't "proper" test files. And we cannot test an abstract class. The next step was to find classes that extend from this abstract one and find their test code instead. See the next section.


#### Project files
GridCacheMapEntry.java is an abstract class.
We can not test an abstract class and must, therefore, test the classes that extend this class.

* GridLocalCacheEntry extends GridCacheMapEntry
* GridDistributedCacheEntry extends GridCacheMapEntry
* GridDhtCacheEntry extends GridDistributedCacheEntry

Tests (`mvn test -Dtest=<class name>`):
1. CacheOffheapMapEntrySelfTest 1 test 01:50 min
2. GridCacheDhtEntrySelfTest 3 tests 01:50 min
3. GridCacheNearReadersSelfTest 7 tests 02:08 min
4. GridCacheDhtEvictionNearReadersSelfTest 1 test 01:16 min FAILS!
5. IgnitePutAllUpdateNonPreloadedPartitionSelfTest 1 test 01:06 min
6. IgnitePutAllLargeBatchSelfTest 12 tests 01:58 min
7. GridCacheEntryMemorySizeSelfTest 4 tests 01:04 min 
8. GridCacheTxNodeFailureSelfTest 12 tests 02:40 min
9. GridCacheAtomicNearCacheSelfTest 2 tests 01:12 min 
10. GridCacheNearMultiNodeSelfTest 19 tests 01:07 min
11. GridCacheConcurrentTxMultiNodeLoadTest (Never ending test, ran for 56:00 min...)
12. GridCacheStoreManagerDeserializationTest 3 tests 01:16 min

Without test 4 and 11. 64 tests 04:16 min
```
mvn test -Dtest=CacheOffheapMapEntrySelfTest,GridCacheDhtEntrySelfTest,GridCacheNearReadersSelfTest,IgnitePutAllUpdateNonPreloadedPartitionSelfTest,IgnitePutAllLargeBatchSelfTest,GridCacheEntryMemorySizeSelfTest,GridCacheTxNodeFailureSelfTest,GridCacheAtomicNearCacheSelfTest,GridCacheNearMultiNodeSelfTest,GridCacheStoreManagerDeserializationTest
```

**6. Do you plan to continue or choose another project?**
Given how long it takes to test the project, it's obviously impractical to run them. We also tried to find the tests that tested the functions/classes of interest, so that we could run these tests only, but failed. We decided to move on.

## Project 3 last choice

Name: Pgmpy

URL: https://github.com/pgmpy/pgmpy

LOC = 19000+ 

## Selected issue(s)

Title: Refactor DirectedGraph class #1064

URL: https://github.com/pgmpy/pgmpy/issues/1064

### Subject of the issue

`DirectedGraph` is currently the base class for all the DAG-based models. It should be renamed to `DAG` to be more explicit and all the DAG methods should be moved from `BayesianModel` to `DAG` as they are applicable to all DAG based models. The following changes need to be done (along with their tests): See issue URL.

*_(Description copied from URL.)_*


## Onboarding experience

Did it build as documented?

**1. How easily can you build the project? Briefly describe if everything worked as documented or not:**

The **README** describes three different installation options: using Conda, pip, or updating the codebase. We used pip to install, and it was as simple as using the commands:

```
pip install -r requirements.txt --user
pip install -r requirements-dev.txt --user
pip install pgmpy --user
```


**2. Did you have to install a lot of additional tools to build the software? Were those tools well documented?**

pip is needed, but none of us had to install it again.
After installing the required python packages using the commands above, we were able to run the project in a python virtual environment.


**3. Were other components installed automatically by the build script?**

The dependencies below are required. As we all have python installed, the latter four are the ones installed automatically by pip if not already present in the system. 

    Python 2.7 or Python 3
    NetworkX 1.11
    Scipy 0.18.0
    Numpy 1.11.1
    Pandas 0.18.1

**4. Did the build conclude automatically without errors?**
The only error that occurred was due to user permissions when installing with pip. We had to add the --user command in order to fix this, otherwise no functional errors. 

**5. How well do examples and tests run on your system(s)?**
We had to create a virtual environment for every test to successfully run as some required Python 3 and we ran Python 2.7, this was done using the following command: 

``` 
cd pgmpy
python3 -m venv env 
source env/bin/activate
``` 



You can launch the test from pgmpy source directory ```nosetests -v ```.
All tests run but we get one failing test ```FAIL: test_sampling (pgmpy.tests.test_sampling.test_continuous_sampling.TestNUTSInference) ```

For the testing and including coverage, use the command ``` nosetests --with-coverage --cover-package=pgmpy ``` **python-nose** might have to be installed manually if not installed already.

```
Ran 639 tests in 192.321s

OK (SKIP=7)
```

**6. Do you plan to continue or choose another project?**
We plan to proceed with this project. Here is the link to the original issue: https://github.com/pgmpy/pgmpy/issues/1064

## Requirements affected by functionality being refactored
https://github.com/pgmpy/pgmpy/blob/c5ba23a8f7891c0f32df56b24fa2dc5fc0ddbc85/pgmpy/models/BayesianModel.py

https://github.com/pgmpy/pgmpy/blob/29955e759b766edddbd88cce8a70fe1c326c7497/pgmpy/base/DirectedGraph.py
### 0 BayesianModel.active_trail_nodes(self, variables, observed=None)
**Input**
**variables**: str or array-like, variables whose active trails are to be found.
**observed**: List of nodes (optional). If given the active trails would be computed assuming these nodes to be observed.
**Output**
Returns a dictionary with the given variables as keys and all the nodes reachable from that respective variable as values.

### 1 BayesianModel.local_independencies(self, variables)
**Input**
**variables**: str or array-like variables whose local independencies are to be found.
**Output**
Returns an instance of Independencies containing the local independencies of each of the variables.

### 2 BayesianModel.is_active_trail(self, start, end, observed=None)
**Input**
**start**: Graph Node
**end**: Graph Node
**observed**: List of nodes (optional). If given the active trail would be computed assuming these nodes to be observed.
**additional_observed**: List of nodes (optional). If given the active trail would be computed assuming these nodes to be observed along with the nodes marked as observed in the model.
**Output**
Returns True if there is any active trail between the start and the end node.

### 3 BayesianModel.get_independencies(self, latex=False)
**Input**
**latex**: boolean. If latex=True then latex string of the independence assertion would be created.
**Output**
Computes independencies in the Bayesian Network, by checking d-separation.

### 4 BayesianModel.get_immoralities(self)
**Output**
**set**: A set of all the immoralities in the model

### 5 BayesianModel.is_iequivalent(self, model)
**Input**
**model**: A Bayesian model object, for which you want to check I-equivalence
**Output**
**boolean**: True if both are I-equivalent, False otherwise

### 6 BayesianModel.get_markov_blanket(self, node)
**Input**
**node**: string, int or any hashable python object. The node whose markov blanket would be returned.
**Output**
**list(blanket_nodes)**: List of nodes contained in Markov Blanket

### 7 DAG.__init__
**input**
**data**: input graph
        Data to initialize graph. If data=None (default) an empty graph is created. The data can be an edge list or any Networkx graph object.
        data is not allowed to contain cycles.
        
**Tests**
    A new test was implemented to test the new functionality to check for cycles. `test_init_with_cycle` in the file `test_DAG.py`.
## Existing test cases relating to refactored code

As for the existing test the following files had to be changed while refactoring:

1. test_DirectedGraph (which is named test_DAG after). This consisted of changing the DirectedGraph occurrences to DAG, and the new cycle function was tested here with new tests.
2. test_ConstrantBasedEstimator This consisted of changing the DirectedGraph occurrences to DAG. 
3. test_BayesianModel This consisted of changing the DirectedGraph occurrences to DAG.

Please look in these files for existing test cases. Basically, most of the test cases were handled by changing the occurrences of DirectedGraph to DAG apart from the new cycle function. 

To view the changes made to the tests as a diff, use the command:

```
git diff dev refactor/DirectedGraph pgmpy/tests/
```

## The refactoring carried out, UML

https://drive.google.com/file/d/1F__5RULV_AeTXG0VjT7z_7keZVtUNKEz/view

## Test logs

Test logs before the refactoring: https://drive.google.com/open?id=1fSXYtUXJ2jIraRWJSso8NRIykhh_UFma

Test logs after the refactoring: https://drive.google.com/open?id=1HtnH3U88PplvDzxeQgtcZDhcp7K2ga02

## Effort spent

For each team member, how much time was spent in
Marcus Östling:
*    5h searching for a project
*    2h Geode (Project 1)
*    6h identifying tests for GridCacheMapEntry (Project 2)
*    2h Document function requirements (pgmpy).
*    5h Checking for cycles in DAG, I had some problems and that I needed to discuss with an author of pgmpy.
*    2h Moving two function from DirectedGraph to DAG.

Andreas Gylling: 
*    8hours looking at Geode Repo
*    7- hours trying to run test suite on ignite / writing report meanwhile.
*    6 hours writing report and moving 2 functions (active_trail_nodes and get_markov_blanket).
*    2 hours resolving merge conflicts. 
*    1 hour for final report changes

Kartal Kaan Bozdoğan:
*    4h Looking at Geode Repo
*    4h Looking for a new project/issue to work on
*    8h Trying to make sense of Ignite code, come up with a refactoring plan and execute the tests
*    2h Renaming 'DirectedGraph' to 'DAG'
*    4h Resolving merge conflicts that arose during the refactoring of pgmpy

Gustaf Pihl:
*    5h Studying code in Geode Repo
*    8h Studying code in Ignite
*    4h Contemplating how to refactor functions in Ignite issue
*    2h Moving is_iequivalent and get_immoralities from BayesianModel to DAG (DirectedGraph).
*    2h Creating the UML diagram before and after representing the effects of the refactor.
*    2h Reviewing pull requests for the pgmpy project

Philippa Örnell: 
It took two whole days before finding a project we could proceed with and start refactoring with. 
* 6-8 hours looking at Geode project, trying to understand the issue. 
* 6 hours looking at Ignite project, trying to build and get the tests running. Getting the tests to work was most difficult with that project. Then another 2 trying to understand the refactoring issue.
* Spent 2-3 hours on Thursday evening trying to find a new project with the requirement that the code base is smaller and the refactoring issue is clearer than the Apache project. Found pgmpy!
* 3-4 hours structure report and do report writing
* 1 h, Refactoring 1 function 


## Overall experience
Our third project was a lot easier to work with due to the clear issue description and great documentation, which we felt were lacking in the first two project we looked at. 

**What are your main takeaways from this project? What did you learn?**
The importance of good documentation cannot be stressed enough, and not only the documentation of the function but also documentation for developers for how to build and run tests. Finding good open source projects can be quite hard. We had way too little time for the bigger Apache projects. Another issue with the first Apache project was that we found an open and unassigned issue but after a while, we actually found a pull request for almost the entire refactoring.

**Is there something special you want to mention here?**
We had good contact with the creator of the issue on GitHub, which was nice when we needed clarification on some things. The creator was welcoming and supportive.
