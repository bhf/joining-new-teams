# joining-new-teams

As a developer looking to join a new team, my user stories are:

1. I want to know what the main repos are (using git log --stat or [git quickstats](https://github.com/arzzen/git-quick-stats))
2. I want to know what the main classes and methods are in each repo
3. I want to understand what cross-repo dependence exists between components to identify cross-component risks
4. I want to understand what level of coverage already exists to identify general points of risk and coverage expectancy

By the end of step 3, there is enough information to create a prioritised list of classes and methods to dig into or have UML generated for (using Plant UML) to try
and understand better.

I will be using real-logic/aeron as an example, because I've been playing with it recently and also it is a good example of a repo with possible cross-dependence between modules/sub projects. Please be aware that all results from tools are shortened or intended as samples only to illustrate a process.

## Code metrics ##

Examples are below:

### Method calls in aeron-client ###

| Method                                                    | BRANCH | CALL | CALLED | CALLEDp |
| --------------------------------------------------------- | ------ | ---- | ------ | ------- |
| io.aeron.ChannelUri.get(String)                           | 0      | 1    | 96     | 94      |
| io.aeron.Aeron.nextCorrelationId()                        | 0      | 2    | 111    | 77      |
| io.aeron.ChannelUri.toString()                            | 0      | 22   | 462    | 70      |
| io.aeron.exceptions.AeronException.AeronException(String) | 0      | 2    | 62     | 62      |

### Classes ###

| Class                                  | Cyclic | Dcy | Dcy\* | Dpt | Dpt\* | PDcy | PDpt |
| -------------------------------------- | ------ | --- | ----- | --- | ----- | ---- | ---- |
| io.aeron.Aeron                         | 22     | 12  | 79    | 235 | 645   | 2    | 20   |
| io.aeron.CommonContext                 | 22     | 8   | 79    | 169 | 645   | 2    | 18   |
| io.aeron.Publication                   | 22     | 12  | 79    | 152 | 645   | 5    | 16   |
| io.aeron.Subscription                  | 22     | 16  | 79    | 142 | 645   | 4    | 15   |
| io.aeron.logbuffer.Header              | 0      | 4   | 5     | 135 | 663   | 2    | 18   |
| io.aeron.logbuffer.LogBufferDescriptor | 0      | 2   | 4     | 102 | 684   | 2    | 8    |
| io.aeron.protocol.DataHeaderFlyweight  | 0      | 1   | 2     | 101 | 701   | 1    | 11   |
| io.aeron.Aeron.Context                 | 22     | 13  | 79    | 97  | 645   | 2    | 14   |
| io.aeron.Image                         | 22     | 13  | 79    | 80  | 645   | 3    | 13   |


### Interfaces ###

| Interface                                    | Cyclic | Dcy | Dcy\* | Dpt | Dpt\* | PDcy | PDpt |
| -------------------------------------------- | ------ | --- | ----- | --- | ----- | ---- | ---- |
| io.aeron.logbuffer.FragmentHandler           | 0      | 1   | 6     | 81  | 653   | 1    | 14   |
| io.aeron.logbuffer.ControlledFragmentHandler | 0      | 2   | 7     | 25  | 648   | 1    | 8    |
| io.aeron.security.AuthorisationService       | 0      | 0   | 0     | 24  | 260   | 0    | 5    |
| io.aeron.AvailableImageHandler               | 22     | 1   | 79    | 18  | 645   | 1    | 4    |
| io.aeron.UnavailableImageHandler             | 22     | 1   | 79    | 16  | 645   | 1    | 4    |


## Git log co-commit within a project ##

This is a measure of how often the same person is making commits within the same project within a window of time, ranked by those
files which display the highest level of co-commit (filtered to remove some file types). This is to identify which files
are likely to change together within the same project/repo. Here is an example for Agrona:

| File                                                                      | Most Co-Checked                                                           | coCheckTally | coCheckFileTally | firstFileTally |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ------------ | ---------------- | -------------- |
| agrona/src/test/java/org/agrona/DeadlineTimerWheelTest.java               | agrona/src/main/java/org/agrona/DeadlineTimerWheel.java                   | 93           | 510              | 329            |
| agrona/src/main/java/org/agrona/DeadlineTimerWheel.java                   | agrona/src/test/java/org/agrona/DeadlineTimerWheelTest.java               | 93           | 329              | 510            |
| agrona/src/main/java/org/agrona/collections/Int2IntHashMap.java           | agrona/src/main/java/org/agrona/collections/Int2ObjectHashMap.java        | 90           | 466              | 483            |
| agrona/src/main/java/org/agrona/collections/Int2ObjectHashMap.java        | agrona/src/main/java/org/agrona/collections/Int2IntHashMap.java           | 90           | 483              | 466            |
| agrona/src/test/java/org/agrona/concurrent/DynamicCompositeAgentTest.java | agrona/src/main/java/org/agrona/concurrent/DynamicCompositeAgent.java     | 84           | 542              | 235            |
| agrona/src/main/java/org/agrona/concurrent/DynamicCompositeAgent.java     | agrona/src/test/java/org/agrona/concurrent/DynamicCompositeAgentTest.java | 84           | 235              | 542            |
| agrona/src/main/java/org/agrona/collections/ObjectHashSet.java            | agrona/src/main/java/org/agrona/collections/IntHashSet.java               | 82           | 421              | 367            |
| agrona/src/main/java/org/agrona/collections/IntHashSet.java               | agrona/src/main/java/org/agrona/collections/ObjectHashSet.java            | 82           | 367              | 421            |
| agrona/src/main/java/org/agrona/concurrent/AgentRunner.java               | agrona/src/main/java/org/agrona/concurrent/DynamicCompositeAgent.java     | 80           | 542              | 438            |
| agrona/src/main/java/org/agrona/collections/Object2IntHashMap.java        | agrona/src/main/java/org/agrona/collections/Int2IntHashMap.java           | 69           | 483              | 336            |




## Git log co-commit across a project ##

This is a measure of how often the same person is making commits across different repos within a window of time, ranked by those
files which display the highest level of co-commit (filtered to remove some file types). This is to identify which files
are likely changing together across projects/repos. The example data below combines a number of repos including Agrona and SBE.
Your success at using this method will also vary upon project selection i.e. you have to be sensible when selecting
which repos to analyse for cross-dependence otherwise you will get spurious results.

**Most Probable Co-Checks:**

| File                                                                          | Most Co-Checked                                                                | coCheckTally | coCheckFileTally | firstFileTally |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ------------ | ---------------- | -------------- |
| src/main/java/uk/co/real_logic/agrona/concurrent/status/CountersReader.java   | aeron-samples/src/main/java/uk/co/real_logic/aeron/samples/AeronStat.java      | 22           | 1282             | 42             |
| sbe-tool/src/test/resources/issue835.xml                                      | aeron-cluster/src/main/java/io/aeron/cluster/ConsensusModuleAgent.java         | 12           | 18911            | 27             |
| src/main/java/org/agrona/concurrent/HighResolutionTimer.java                  | aeron-driver/src/main/java/io/aeron/driver/LossDetector.java                   | 11           | 856              | 31             |
| main/android/uk/co/real_logic/sbe/codec/java/CodecUtil.java                   | aeron-driver/src/main/java/uk/co/real_logic/aeron/driver/DriverConductor.java  | 29           | 13828            | 83             |
| aeron-util/src/test/java/uk/co/real_logic/aeron/util/AgentTest.java           | test/java/uk/co/real_logic/sbe/codec/java/DirectBufferTest.java                | 1            | 237              | 3              |
| aeron-cluster/src/main/java/io/aeron/cluster/ConsensusModuleStateExport.java  | sbe-tool/src/main/java/uk/co/real_logic/sbe/xml/XmlSchemaParser.java           | 5            | 1375             | 16             |
| src/main/java/uk/co/real_logic/agrona/concurrent/SigInt.java                  | aeron-driver/src/main/java/uk/co/real_logic/aeron/driver/DriverConnection.java | 15           | 5511             | 49             |
| src/main/java/uk/co/real_logic/agrona/concurrent/SigIntBarrier.java           | aeron-driver/src/main/java/uk/co/real_logic/aeron/driver/DriverConnection.java | 15           | 5511             | 49             |
| src/main/java/uk/co/real_logic/agrona/console/ContinueBarrier.java            | aeron-driver/src/main/java/uk/co/real_logic/aeron/driver/DriverConnection.java | 15           | 5511             | 50             |
| src/main/java/uk/co/real_logic/agrona/concurrent/status/ReadOnlyPosition.java | aeron-driver/src/main/java/uk/co/real_logic/aeron/driver/DriverConnection.java | 14           | 5511             | 47             |


**Top Co-Checks**

| File                                                                            | Most Co-Checked                                                                | coCheckTally | coCheckFileTally | firstFileTally |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ------------ | ---------------- | -------------- |
| agrona/src/main/java/org/agrona/collections/IntArrayList.java                   | aeron-cluster/src/main/java/io/aeron/cluster/SequencerAgent.java               | 104          | 10770            | 1168           |
| aeron-cluster/src/main/java/io/aeron/cluster/SequencerAgent.java                | agrona/src/main/java/org/agrona/collections/IntArrayList.java                  | 104          | 1168             | 10770          |
| aeron-cluster/src/main/java/io/aeron/cluster/ConsensusModuleAgent.java          | sbe-tool/src/main/java/uk/co/real_logic/sbe/generation/cpp/CppGenerator.java   | 90           | 7062             | 18911          |
| sbe-tool/src/main/java/uk/co/real_logic/sbe/generation/cpp/CppGenerator.java    | aeron-cluster/src/main/java/io/aeron/cluster/ConsensusModuleAgent.java         | 90           | 18911            | 7062           |
| sbe-tool/src/main/java/uk/co/real_logic/sbe/generation/java/JavaGenerator.java  | aeron-cluster/src/main/java/io/aeron/cluster/ConsensusModuleAgent.java         | 88           | 18911            | 6362           |
| aeron-cluster/src/main/java/io/aeron/cluster/Election.java                      | sbe-tool/src/main/java/uk/co/real_logic/sbe/generation/cpp/CppGenerator.java   | 77           | 7062             | 11051          |
| test/java/uk/co/real_logic/sbe/generation/java/JavaGeneratorTest.java           | aeron-driver/src/main/java/uk/co/real_logic/aeron/driver/DriverConductor.java  | 73           | 13828            | 2239           |
| aeron-driver/src/main/java/uk/co/real_logic/aeron/driver/DriverConductor.java   | test/java/uk/co/real_logic/sbe/generation/java/JavaGeneratorTest.java          | 73           | 2239             | 13828          |
| aeron-archive/src/main/java/io/aeron/archive/ArchiveConductor.java              | sbe-tool/src/main/java/uk/co/real_logic/sbe/generation/java/JavaGenerator.java | 70           | 6362             | 10390          |
| aeron-cluster/src/main/java/io/aeron/cluster/service/ClusteredServiceAgent.java | agrona/src/main/java/org/agrona/collections/IntArrayList.java                  | 69           | 1168             | 11345          |



## Code coverage ##


The combined coverage is:
| **Package** | **Class, %**     | **Method, %**      | **Line, %**         |
| ----------- | ---------------- | ------------------ | ------------------- |
| all classes | 67.5% (865/1282) | 26.5% (7673/28954) | 37.4% (33457/89379) |


Whilst the method coverage looks very low, approximatly 17k methods are auto-generated so if we account for that then the method coverage is closer
to 63%

The combined project base requires unit tests across sub-projects to get that kind of coverage. For example,
if we look at the coverage for io.aeron.driver.media using the unit tests only in Aeron-driver we see the 
class and method level coverage is approximatly 65% whereas when we look at the coverage across all projects for that 
package then the coverage jumps.


| **Projects** | **Package**           | **Class, %**  | **Method, %**   | **Line, %**       |
| ------------ | --------------------- | ------------- | --------------- | ----------------- |
| Aeron-driver | io.aeron.driver.media | 65.1% (28/43) | 64.4% (224/348) | 56.3% (880/1562)  |
| All projects | io.aeron.driver.media | 97.7% (42/43) | 87.9% (306/348) | 81.2% (1268/1562) |


This means the coverage is dependent upon cross-project unit tests. 




