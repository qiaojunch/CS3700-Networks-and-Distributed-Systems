####Description
You will implement a simple bridge that is able to establish a spanning tree and forward frames between its various ports. Since running your bridge on Northeastern's network would likely impact real traffic, we will provide you with a simulation environment that emulates LANs and end hosts. Your bridges must construct a spanning tree (and disable any ports that are not part of the tree), must forward frames between these ports, learn the locations (ports) of hosts, and handle both bridge and port failures (e.g., by automatically reconfiguring the spanning tree).
Your bridges will be tested for both correctness and performance. Part of your grade will come from the overhead your network has (i.e., lower overhead incurs a higher score) and the fraction of packets that are successfully delivered between end hosts. Your results will be compared to your classmates' via a leaderboard.

####Language
You can write your code in whatever language you choose, as long as your code compiles and runs on unmodified CCIS Linux machines on the command line. Do not use libraries that are not installed by default on the CCIS Linux machines. Similarly, your code must compile and run on the command line. You may use IDEs (e.g. Eclipse) during development, but do not turn in your IDE project without a Makefile. Make sure you code has no dependencies on your IDE.

####Requirements
To simplify the project, instead of using real packet formats, we will be sending our data across the wire in JSON (many languages have utilities to encode and decode JSON, and you are welcome to use these libraries). Your bridge program must meet the following requirements:
- Form a spanning tree in order to prevent packet loops
- Handle the failure of bridges, the failure of bridge ports, and the introduction of new bridges and LANs over time
- Learn the locations of end hosts
- Deliver end host packets to the destination
- Handle the mobility of end hosts between LANs
- Senders and receivers must print out specified debugging messages to STDOUT
- Your program must be called 3700bridge
- You should implement a simplified version of the standard bridge spanning tree protocol that we discussed in class. Note that more sophisticated and properly tuned algorithms (i.e., those which perform better) will be given higher credit. For example, some desired properties include (but are not limited to):
- Fast convergence: Require little time to form a spanning tree.
- Low overhead: Reduce packet flooding when possible.
Regardless, correctness matters most; performance is a secondary concern. We will test your code and measure these two performance metrics; better performance will result in higher credit. We will test your code by introducing a variety of different errors and failures; you should handle these errors gracefully, recover, and never crash.

####Program Specification
The command line syntax for your bridge is given below. The bridge program takes command line arguments representing (1) the ID of the bridge, and (2) the LAN or LANs that it should connect to. The bridge must be connected to at least one LAN. The syntax for launching your bridge is therefore:
./3700bridge <id> <LAN> [LAN [LAN ...]]
 - id (Required) The id of the bridge. For simplicity, all bridge ids are four-digit hexadecimal numbers (e.g., 0aa1 or f29a).
 - LAN (Required) The unique name of the LAN(s) the bridge should connect to. The LANs are named using unique ASCII strings; the names themselves are not meaningful.
 
####BPDU Messages
- You should configure your bridges to periodically broadcast BPDUs on all ports. You should broadcast BPDUs no more frequently than once every 500 ms. Using those BPDUs, you should constantly be listening for new roots, new bridges, etc, and should make decisions about which ports are active and inactive upon receiving each BPDU. Additionally, you should "timeout" BPDUs after 750 ms. To aid in grading and debugging, your bridge program should print out messages about the spanning tree calculation to STDOUT. When starting up, your bridge should print out "Bridge <id> starting up" where <id> is the ID of the bridge. 
- When your bridge selects a new root, it should print out "New root: id/root" where id is the ID of the local bridge and root is the ID of the new root. 
- When your bridge changes its root port, it should print out "Root port: id/port_id" where port_id is the port number (0-indexed).
- When your bridge decides that a port is the designated port for a LAN, it should print out "Designated port: id/port_id"
- Finally, when your bridge decides that a port should be disabled, it should print out: "Disabled port: id/port_id"

####Data Messages
Additionally, your bridge should build up a forwarding table as discussed in class. You should "timeout" forwarding table entries 5 seconds after receiving the last message from that address. When any of your bridge's ports changes state (designated, root, etc), you should flush your forwarding table. When forwarding data packets, your bridge program should print out the following messages to STDOUT. When your bridge receives a message on an active port (i.e., not disabled), it should print out "Received message id on port port_id from source to dest" where id is the unique identifier of the data message, port_id is the port number on the bridge that the message was received on, and source and dest are the source and destination of the message. (Note that your bridge should silently ignore all messages [other than BPDUs] on disabled ports) Once your bridge makes a forwarding decision about the message, it should print out one of three messages:
- Forwarding message id to port port_id

or

- Broadcasting message id to all ports

or

- Not forwarding message id

Thus, every non-BPDU message your bridge receives on an active port should have one of the above three lines printed out. This will help you to debug why your bridges are misrouting messages (if this should ever occur).


####Testing Your Code
In order for you to test your code in our network simulator, we have included a perl script that will create the emulated LANs, run your bridge program and connect it to these LANs, start and stop your bridges in a configurable way, and create and record end host traffic. This script is included in the starter code, and you can run it by executing
./run <config-file>
where <config-file> is the configuration file that describes the network you would like to implement. Note that you do not need to parse the config files yourself; the run script does this. Instead, you can create custom configs to test your code under different scenarios.
Config File Format

The configuration file that you specify describes the LANs, the bridges, and the connections between these. It also contains information about when bridge come up and down, and the end host traffic that should be generated. The file is formatted in JSON and has the following elements
lifetime (Required) The number of seconds the simulation should run for.
bridges (Required) An array of bridge elements (described below). At least one bridge must be specified. Each bridge element is an associative array that has the following properties:
id (Required) The ID of the bridge, a string.
lans (Required) An array of the LAN IDs that the bridge is connected to. All LANs are identified by a non-negative number.
start (Optional) The start time (in seconds) when the bridge should be turned on. If not specified, the bridge is started at the beginning of the simulation.
stop (Optional) The stop time (in seconds) when the bridge should be turned off. If not specified, the bridge is stopped at the end of the simulation.
hosts (Required) The number of hosts to generate (these are randomly attached to LANs).
traffic (Required) The number of end host packets to randomly generate (these are sent with randomly selected sources and destinations).
wait (Optional) The number of seconds to wait before sending any data traffic (default of 2 seconds).
seed (Optional) The random seed to choose. If not specified, a random value is chosen. Setting this value will allow for a reproducible set of hosts and traffic.
For example, a simple network with two LANs connected by a single bridge would be:
      {
          "lifetime": 30,
          "bridges": [{"id": "A", "lans": [1, 2]}],
          "hosts": 10,
          "traffic": 1000
      }
and a more complex network may be
      {
          "lifetime": 30,
          "bridges": [{"id": "A", "lans": [1, 3]},
                      {"id": "B", "lans": [2, 3], "stop": 7},
                      {"id": "C", "lans": [1, 2], "start": 5, "stop": 9},
                      {"id": "D", "lans": [2, 4]},
                      {"id": "E", "lans": [2, 4, 5, 6]}]
          "hosts": 100,
          "traffic": 10000
      }
./run Output

The output of the ./run script includes timestamps and all logging information from your bridges and the emulated end hosts. Note that all data traffic will be delayed for 2 seconds at the beginning of the simulation to allow your bridges to form an initial spanning tree. At the end, the output also includes some statistics about the your bridges' performance:
    bash$ ./run config.json
    ...
    [ 14.9990
    Host ed10] Sent message 776 to 41c1
    [ 15.0001 Bridge 92ba] Received message 776 on port 0 from ed10 to 41c1
    Simulation finished.
    Total packets sent: 6730
    Total data packets sent: 2000
    Total data packets received: 1984
    Total data packets dropped: 16 (message ids 52, 70, 181, 320, 517, 571, 634, 776, 900, 1111, 1242, 1501, 1517, 1588, 1685, 1887)
    Total data packets duplicated: 17 (message ids 311, 433, 541, 630, 632, 658, 717, 804, 998, 1022, 1341, 1364, 1433, 1611, 1668, 1804, 1876)
    Data packet delivery ratio: 0.992000
Each of the fields is self-explanatory. Ideally, you would like all messages to be delivered (a delivery ratio of 1.0) and the number of packets dropped and duplicated to be 0 (a message can cause two packets to be delivered if the network is being re-configured when it is sent). Additionally, you want the number of total packets sent to be low as well (this includes BPDUs, which are overhead).
Testing Script

Additionally, we have included a basic test script that runs your code under a variety of different network configurations and also checks your code's compatibility with the grading script. If your code fails in the test script we provide, you can be assured that it will fare poorly when run under the grading script. To run the test script, simply type
    bash$ ./test
    Basic (no failures, no new bridges) tests (PDR = 1.0)
      One bridge, one LAN                                    [PASS]
      One bridge, two LANs                                   [PASS]
      One bridge, three LANs                                 [PASS]
      Two bridges, one LAN                                   [PASS]
      Two bridges, two LANs                                  [PASS]
This will run your code on a number of configurations, and will output whether your program performs sufficiently. If you wish to run one of the tests manually, you can do so with
bash$ ./run basic-4.conf
Performance Testing

As mentioned in class, 10% of your grade on this project will come from performance. Your project will be graded against the submissions of your peers. To help you know how you're doing, the testing script will also run a series of performance tests at the end; for each test that you successfully complete, it will report your time elapsed and bytes sent to a common database. For example, you might see
    Performance tests
      Network 1                                        [PASS]
       99.1% packets delivered, 3.0% overhead
This indicates that you successfully delivered 99.1% of all end-host packets and had an overhead of 3%. This score will be reported to the common database. Note that, by default, only reasonable scores are sent to the database; if your scores aren't showing up, that means you need to improve your performance.
In order to see how your project ranks, you can run

    bash$ /course/cs3700f15/bin/project2/printstats
    ----- TEST: Eight bridges, eight LANs -----
    Least overhead:
    1: cbw                         200 packets
    2: foo                         220 packets

    Highest delivery ratio:
    1: foo                         1.00000
    2: cbw                         0.950000
which will print out the rank of each group for each performance test, divided into the number of packets sent and the delivery ratio. In this particular example, cbw's project has lower overhead but delivers fewer of the packets. Obviously, you would ideally have like to have more packets delivered and fewer packets sent.
Submitting Your Project
If you have not done so already, register yourself for our grading system using the following command:
$ /course/cs3700f15/bin/register-student [NUID]
NUID is your Northeastern ID number, including any leading zeroes.
Before turning in your project, you and your partner(s) must register your group. To register yourself in a group, execute the following script:

$ /course/cs3700f15/bin/register project2 [team name]
This will either report back success or will give you an error message. If you have trouble registering, please contact the course staff. You and your partner(s) must all run this script with the same [team name]. This is how we know you are part of the same group.
To turn-in your project, you should submit your (thoroughly documented) code along with two other files:

A Makefile that compiles your code. Your Makefile may be blank, but it must exist.
A plain-text (no Word or PDF) README file. In this file, you should briefly describe your high-level approach, any challenges you faced, and an overview of how you tested your code.
Your README, Makefile, source code, etc. should all be placed in a directory. You submit your project by running the turn-in script as follows:
$ /course/cs3700f15/bin/turnin project2 [project directory]
[project directory] is the name of the directory with your submission. The script will print out every file that you are submitting, so make sure that it prints out all of the files you wish to submit! The turn-in script will not accept submissions that are missing a README or a Makefile. Only one group member needs to submit your project. Your group may submit as many times as you wish; only the last submission will be graded, and the time of the last submission will determine whether your assignment is late.
Grading
This project is worth 12% of your final grade. The grading in this project will consist of
75% Program correctness
10% Performance
15% Style and documentation
At a minimum, your code must pass the test suite without errors or crashes, and it must obey the requirements specified above. All student code will be scanned by plagarism detection software to ensure that students are not copying code from the Internet or each other.
