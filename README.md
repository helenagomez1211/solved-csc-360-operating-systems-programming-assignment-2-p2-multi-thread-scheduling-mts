Download Link: https://assignmentchef.com/product/solved-csc-360-operating-systems-programming-assignment-2-p2-multi-thread-scheduling-mts
<br>
<h1>1            Introduction</h1>

<ul>

 <li>In P1 (Simple Shell Interpreter, SSI), you have built a shell environment to interact with the host operating system.</li>

 <li>Good job! But very soon you find that SSI is missing one of the key features in a real multi-process or multi-thread 9 operating system: scheduling, i.e., all processes or threads created by your SSI are still scheduled by the host operating 10 system, not yours! Interested in building a multi-thread scheduling system for yourself? In this assignment, you will 11 learn how to use the three programming constructs provided by the posix pthread library:</li>

 <li>threads</li>

 <li>mutexes</li>

 <li>condition variables (convars)</li>

 <li>to do so. Your goal is to construct a simulator of an automated control system for the railway track shown in Figure 1 16 (i.e., to emulate the scheduling of multiple threads sharing a common resource in a real operating system).</li>

</ul>

17 As shown in Figure 1, there are two stations (for high and low priority trains) on each side of the main track. At 18 each station, one or more trains are loaded with commodities. Each train in the simulation commences its loading 19 process at a <u>common </u>start time 0 of the simulation program. Some trains take more time to load, some less. After 20 a train is loaded, it patiently awaits permission to cross the main track, subject to the requirements specified in

21 Section 2.2. Most importantly, only one train can be on the main track at any given time. After a train finishes 22 crossing, it magically disappears. You will use threads to simulate the trains approaching the main track from two 23 different directions, and your program will schedule between them to meet the requirements in Section 2.2. 24 You will use C or C++ and the Linux workstation in ECS242 or ECS348 or the linux.csc.uvic.ca cluster to 25 implement and test your work.

<h1>26 2          Trains</h1>

<ul>

 <li>Each train, which will be simulated by a thread, has the following attributes:</li>

 <li><strong>Number</strong>: an integer uniquely identifying each train.</li>

</ul>

Figure 1: The railway system under consideration.

<ul>

 <li><strong>Direction</strong>:</li>

 <li>If the direction of a train is Westbound, it starts from the East station and travels to the West station.</li>

 <li>If the direction of a train is Eastbound, it starts from the West station and travels to the East station.</li>

 <li><strong>Priority</strong>: The priority of the station from which it departs.</li>

 <li><strong>Loading Time</strong>: The amount of time that it takes to load it (with goods) before it is ready to depart.</li>

 <li><strong>Crossing Time</strong>: The amount of time that the train takes to cross the main track.</li>

 <li>Loading time and crossing time are measured in 10ths of a second. These durations will be simulated by having 36 your threads, which represent trains, usleep() for the required amount of time.</li>

</ul>

<h2>37 2.1           Step 1: Reading the input file</h2>

<ul>

 <li>Your program (mts) will accept one parameter on the command line:</li>

 <li>The parameter is the name of the input file containing the definitions of the trains.</li>

 <li><strong>1.1 Input file format</strong></li>

 <li>The input files have a simple format. Each line contains the information about a single train, such that:</li>

 <li>The first field specifies the <strong>direction </strong>of the train. It is one of the following four characters:</li>

 <li>e, E, w, or W</li>

 <li>e or E specify a train headed East (East-Bound): e represents an east-bound low-priority train, and E represents</li>

 <li>an east-bound high-priority train;</li>

 <li>w or W specify a train headed West (West-Bound): w presents a west-bound low-priority train, and W represents 47 a west-bound high-priority train.</li>

 <li>Immediately following is an integer that indicates the <strong>loading time </strong>of the train.</li>

 <li>Immediately following is an integer that indicates the <strong>crossing time </strong>of the train.</li>

 <li>A newline (
) ends the line.</li>

 <li>Trains are numbered sequentially from 0 according to their order in the input file. You need to use strtok() to 52 handle the line. More efficiently, you can use fscanf()</li>

 <li><strong>1.2 An Example</strong></li>

 <li>The following file specifies three trains, two headed East and one headed West.</li>

</ul>

<h3>55                   e 10 6 56                W 6 7 57                    E 3 10</h3>

58           It implies the following list of trains:

<table width="401">

 <tbody>

  <tr>

   <td width="80">Train No.</td>

   <td width="61">Priority</td>

   <td width="70">Direction</td>

   <td width="97">Loading Time</td>

   <td width="92">Crossing Time</td>

  </tr>

  <tr>

   <td width="80">0</td>

   <td width="61">low</td>

   <td width="70">East</td>

   <td width="97">1.0s</td>

   <td width="92">0.6s</td>

  </tr>

  <tr>

   <td width="80">1</td>

   <td width="61">high</td>

   <td width="70">West</td>

   <td width="97">0.6s</td>

   <td width="92">0.7s</td>

  </tr>

  <tr>

   <td width="80">2</td>

   <td width="61">high</td>

   <td width="70">East</td>

   <td width="97">0.3s</td>

   <td width="92">1.0s</td>

  </tr>

 </tbody>

</table>

59

60                         <em>Note: </em>Observe that Train 2 is actually the first to finish the loading process.

<h2>61 2.2           Step 2: Simulation Rules</h2>

<ul>

 <li>The rules enforced by the automated control system are:</li>

 <li>Only <u>one </u>train is on the <u>main </u>track at any given time.</li>

 <li>Only <u>loaded </u>trains can cross the main track.</li>

 <li>If there are multiple loaded trains, the one with the high priority crosses.</li>

 <li>If two loaded trains have the same priority, then:</li>

 <li>(a) If they are both traveling in the same direction, the train which finished loading <u>first </u>gets the clearance 68 to cross <u>first</u>. If they finished loading at the same time, the one appeared <u>first </u>in the input file gets the 69 clearance to cross <u>first</u>.</li>

 <li>(b) If they are traveling in opposite directions, pick the train which will travel in the direction opposite of</li>

 <li>which the last train to cross the main track traveled. If no trains have crossed the main track yet, the 72 Eastbound train has the priority.</li>

</ul>

<h2>73 2.3          Step 3: Output</h2>

74            For the example, shown in Section 2.1.2, the correct output is:

<h3>75                   00:00:00.3 Train 2 is ready to go East 76        00:00:00.3 Train 2 is ON the main track going East 77   00:00:00.6 Train 1 is ready to go West 78   00:00:01.0 Train 0 is ready to go East 79                       00:00:01.3 Train 2 is OFF the main track after going East 80                       00:00:01.3 Train 1 is ON the main track going West</h3>

81 00:00:02.0 Train 1 is OFF the main track after going West 82 00:00:02.0 Train 0 is ON the main track going East 83 00:00:02.6 Train 0 is OFF the main track after going East 84 You must:

85                   1. print the arrival of each train at its departure point (after loading) using the format string, prefixed by the 86                       simulation time:

<ul>

 <li>“Train %2d is ready to go %4s”</li>

 <li>print the crossing of each train using the format string, prefixed by the simulation time:</li>

 <li>“Train %2d is ON the main track going %4s”</li>

 <li>print the arrival of each train (at its destination) using the format string, prefixed by the simulation time:</li>

</ul>

<h3>91                           “Train %2d is OFF the main track after going %4s”</h3>

<ul>

 <li>where:</li>

 <li>there are only two possible values for direction: “East” and “West”</li>

 <li>trains have integer identifying numbers. The ID number of a train is specified <em>implicitly </em>in the input file. The 95 train specified in the first line of the input file has ID number 0.</li>

</ul>

96                      • trains have loading and crossing times in the range of [1<em><sub>,</sub></em>99].

<h2>97 2.4         Manual Pages</h2>

98 Be sure to study the man pages for the various functions to be used in the assignment. For example, the man page 99 for pthread create can be found by typing the command:

<ul>

 <li>$ man pthread create</li>

 <li>At the end of this assignment you should be familiar with the following functions:</li>

 <li>File access functions:</li>

 <li>(a) atoi</li>

 <li>(b) fopen</li>

 <li>(c) feof</li>

 <li>(d) fgets and strtok and more efficiently you can use fscanf</li>

 <li>(e) fclose</li>

 <li>Thread creation functions:</li>

 <li>(a) pthread create</li>

 <li>(b) pthread exit</li>

 <li>(c) pthread join</li>

 <li>Mutex manipulation functions:</li>

 <li>(a) pthread mutex init</li>

 <li>(b) pthread mutex lock</li>

 <li>(c) pthread mutex unlock</li>

 <li>Condition variable manipulation functions:</li>

 <li>(a) pthread cond init</li>

 <li>(b) pthread cond wait</li>

 <li>(c) pthread cond broadcast</li>

 <li>(d) pthread cond signal</li>

 <li>It is absolutely critical that you read the man pages, and attend the tutorials.</li>

 <li>Your best source of information, as always, is the man 123 For help with the posix interface (in general): 124                 <a href="https://www.opengroup.org/onlinepubs/007908799/">http://www.opengroup.org/onlinepubs/007908799/</a></li>

 <li>For help with posix threads:</li>

 <li><a href="https://www.opengroup.org/onlinepubs/007908799/xsh/pthread.h.html">http://www.opengroup.org/onlinepubs/007908799/xsh/pthread.h.html</a></li>

 <li>A good overview of pthread can be found at: <a href="http://computing.llnl.gov/tutorials/pthreads/">http://computing.llnl.gov/tutorials/pthreads/</a></li>

</ul>

<h1>128 3          Tutorial Schedule</h1>

129                   In order to help you finish this programming assignment on time successfully, the schedule of this assignment has 130 been synchronized with both the lectures and the tutorials. There are four tutorials arranged during the course of 131      this assignment, including the one on pthread. <strong>NOTE: Please do attend the tutorials and follow the tutorial </strong>132    <strong>schedule closely.</strong>

<table width="548">

 <tbody>

  <tr>

   <td width="76">Jan 30/31</td>

   <td width="283">pthread, mutex and condition variable calls</td>

   <td width="188">multi-threading programming</td>

  </tr>

  <tr>

   <td width="76">Feb 6/7</td>

   <td width="283">P2 spec go-through, design review/hints</td>

   <td width="188">design and code skeleton</td>

  </tr>

  <tr>

   <td width="76">Feb 13/14</td>

   <td width="283">reading break, no tutorials</td>

   <td width="188">design finished</td>

  </tr>

  <tr>

   <td width="76">Feb 20/21</td>

   <td width="283">feedback on design and pthread programming</td>

   <td width="188">code done</td>

  </tr>

  <tr>

   <td width="76">Feb 27/28</td>

   <td width="283">testing and last-minute help</td>

   <td width="188">final deliverable</td>

  </tr>

 </tbody>

</table>

<h1>133 4             Submission: Deliverable 1</h1>

134 You will write a design document which answers the following questions. It is recommended that you think through 135 the following questions <em>very carefully </em>before answering them.

136 Unlike P1, no amount of debugging will help after the basic design has been coded. Therefore, it is very important 137 to ensure that the basic design is correct. Answering the following questions haphazardly will basically ensure that 138 Deliverable 2 does not work.

<ul>

 <li>So think about the following for a few days and then write down the answers.</li>

 <li>How many threads are you going to use? Specify the work that you intend each thread to perform.</li>

 <li>Do the threads work independently? Or, is there an overall “controller” thread?</li>

 <li>How many mutexes are you going to use? Specify the operation that each mutex will guard.</li>

 <li>Will the main thread be idle? If not, what will it be doing?</li>

 <li>How are you going to represent stations (which are collections of loaded trains ready to depart)? That is, what 145 type of data structure will you use?</li>

 <li>How are you going to ensure that data structures in your program will not be modified concurrently?</li>

 <li>How many convars are you going to use? For each convar:</li>

 <li>(a) Describe the condition that the convar will represent.</li>

 <li>(b) Which mutex is associated with the convar? Why?</li>

 <li>(c) What operation should be performed once pthread cond wait() has been unblocked <em>and </em>re-acquired the</li>

 <li>mutex?</li>

 <li>In 15 lines or less, briefly sketch the overall algorithm you will use. You may use sentences such as:</li>

 <li>If train is loaded, get station mutex, put into queue, release station mutex.</li>

 <li>The marker will not read beyond 15 lines.</li>

 <li><strong>Note: </strong>Please submit answers to the above on 8.5<sup>′′</sup>×11<sup>′′ </sup>paper in 10pt font, single spaced with 1” margins left, 156 right, top, and bottom. 2 pages maximum (cover page excluded), on Feb 19 through connex. The design counts for 157 5%.</li>

</ul>

<h1>158 5          Bonus Features</h1>

159 Only a simple control system simulator with limited scheduling and synchronization features is required in this assign160 ment. However, students have the option to propose bonus features to include more scheduling and synchronization 161 functions (e.g., to address scheduling fairness issues).

162 If you want to design and implement a bonus feature, you should contact the course instructor for permission 163 before the due date of Deliverable 1, and clearly indicate the feature in the submission of Deliverable 2. The credit 164 for the correctly implemented bonus feature will not exceed 20% of the full marks for this assignment.

<h1>165 6              Submission: Deliverable 2</h1>

166             The code is submitted through connex. The tutorial instructor will give the detailed instruction in the tutorial.

<h2>167 6.1         Submission Requirements</h2>

168 Your submission will be marked by an automated script. The script (which is not very smart) makes certain 169 assumptions about how you have packaged your assignment submission. We list these assumptions so that your 170 submission can be marked thus, in a timely, convenient, and hassle-free manner.

<ul>

 <li>The name of the submission file must be p2.tar.gz</li>

 <li>p2.tar.gz must contain all your files in a directory named p2</li>

 <li>Inside the directory p2, there must be a Makefile. Also there shall be a test input file created by you.</li>

 <li>Invoking make on it must result in an executable named mts being built, <em>without user intervention</em>.</li>

 <li>You may <em>not </em>submit the assignment with a compiled executable and/or object (.o) files; the script will delete 176 them before invoking make.</li>

 <li><strong>Note:</strong></li>

 <li>The script will give a time quota of 1 minute for your program to run on a given input. This time quota is 179 given so that non-terminating programs can be killed.</li>

 <li>Since your program simulates train crossing delays in 10ths of a second, this should not be an issue, at all.</li>

 <li>Follow the output rules specified in the assignment specification, so that the script can tally the output produced 182 by your program against text files containing the correct answer.</li>

</ul>





