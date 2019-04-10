+++
date = "2018-12-03"
title = "Two Player Tetris Using Pygame and Sockets"
slug = "tetris-game"
+++



## Introduction
The following report will describe the Software Development Lifecycle (SDLC) for 2-Player Tetris built for ECE 356: Engineering System Software. The five main steps in the SDLC are Requirement Engineering, Design, Implementation, Testing, and Maintenance. 

The purpose of creating this game was to become familiar with basic client/server architecture. The result is a 2-player game with network functionality using Python. The source code can be found [here.](https://github.com/tmastrom/Tetris)The process of creating this game was both challenging and rewarding.  

## Requirement Engineering
The initial stage of software development is Requirement Engineering (RE). This stage involves creating and analyzing requirements along with studying the feasibility of the project. 

The planning phase of this project included research into what software languages and tools work well for client/server game development, as well as finding examples of these techniques implemented. Examples of simple networked games were found as well as a simple Tetris clone framework [1]. It was concluded that a networked Tetris game was feasible. 

The next step of the RE process is to come up with requirements and analyze them. The requirements for this game were found by brainstorming and can be seen in the following table along with their type and priority.

**Table 1. Requirement table**

**Number** |**Requirement**	|**Priority**|**Type**
--- | --- | --- | ---
1|The game will not start until players are ready|Mandatory|Functional
2|Players can see their opponents score |	Mandatory|	Functional
3|	The scoring system will reflect the original tetris game|	Nice to have	|Functional
4	|At the end of the game, the winner is the player with the most points |	Nice to have|	Functional
5	|The game will run in a Windows 10 environment|	Mandatory	|Non-functional
6	|If a player quits the game, the other player is the winner|	Superfluous|	Non-functional
7	|The game will have 2 players	|Mandatory|	Functional
8	|Game difficulty will increase as the game progresses |	Superfluous	|Functional
9|	Data will be sent reliably|	Mandatory |	Non-functional
10	|Data will be sent with less than 1 second delay|	Nice to have|	Non-functional
11|	All connections are closed at the end of the game|	Nice to have|	Non-Functional

Requirements can be subdivided into Functional and Non-functional categories [2]. Functional requirements define specific behaviour or functionality of the system. Non-functional requirements specify criteria with which to judge the systems operation. 


With the requirements defined, system modelling was undertaken to visually model some of the requirements. Below is a Use Case diagram, illustrating what functionality will be handled by each portion of the system. 

{{< figure src="/images/tetris/fig1.jpg" caption="Figure 1. Use case diagram" >}}

The important details to observe are the client handles the game functionality and the server handles the transfer of data. Both actors are responsible for establishing and closing connections. 

**Table 2. Scoring Scheme** 

Lines Removed	|Score
--- | --- 
1|	40
2|	100
3|	300
4	|1200

As stated in Requirement 3, a new scoring system will be implemented based on the original Tetris. The scoring scheme for the original Nintendo Tetris game [3] is outlined in the table above.  
 
{{< figure src="/images/tetris/fig2.jpg" caption="Figure 2. State diagram" >}}


Next, the specifics of the game client were modelled using a State diagram to show the transitions and triggers from state to state. There are two major states: connected and not connected to server. Within these states are sub states involving running and waiting. By creating these diagrams in the RE stage it makes the design and implementation much smoother.  

## Design
Before designing the code functionality, it is important to pick an architectural model to follow. For a client/server game application it makes the most sense to pick a Client/Server Architectural model using an appropriate communication protocol. This design decision will be discussed in more detail in the Design Issues section of this report. 

{{< figure src="/images/tetris/fig3.jpg" caption="Figure 3. Client server architecture" >}}
 

The design for three requirements will be detailed and discussed in this section. The design will cover the following requirements from the previous section:

1. The game will not start until players are ready 
2. Players can see their opponents score
3. The scoring system will reflect the original Nintendo Tetris game 

The following figure is a high-level design of the interaction between the client and the server which satisfies the three requirements. The diagram has been separated into three sections (Initialization, Ready Loop, and Game Loop) using colour for readability. 

{{< figure src="/images/tetris/fig4.jpg" caption="Figure 4. Application design" >}}


Starting with the initialization stage, which includes the design to send data reliably as stated in Requirement 9. Design decisions made here about the socket protocols play an important role in the reliability of the data transactions. These decisions will be discussed in more detail in the Design Issues section of this report.   

The figure below shows the design for the client server connection process. This design was adapted from a simple echo client server example [4]. The server creates a listening socket which the clients try to connect to is an important design feature.

{{< figure src="/images/tetris/fig5.jpg" caption="Figure 5. Initialization and client/server connection" >}}


A successful connection creates a new socket object that the server uses to communicate with the connected client. Once the clients have both connected, the clients start their own instances of PyGame. It is also important to note the server has been set to non blocking mode using setblocking(0) which allows socket methods to return none. This is key for the next code blocks where it is a possibility the clients are not ready to send or receive data. This allows the server to continue running instead of throwing an error and crashing. 

#### Requirement 1: The Game Will Not Start Until Players Are Ready
As seen above, once the clients have connected, a PyGame instance is initialized. This means the client program enters its main function. Here the variables are initialized, and the ready screen function is called, and the screen is displayed. 

The ready screen function polls for a key press which indicates the player is ready. This design adds network functionality for the client to tell the server the player is ready. Next the clients wait for a response from the server. The server waits until both players have indicated they are ready, and then sends both games a ready signal. The clients will not use nonblocking sockets which conveniently satisfies Requirement 1 because  the sockets will block while waiting for the ready signal from the server, therefore both games will wait until the other is ready, then the game will start. 

{{< figure src="/images/tetris/fig6.jpg" caption="Figure 6. Ready loop logic design" >}}


#### Requirement 2: Players Can See Their Opponents Score 
The first requirement to be designed was the passing of data from player 1 to player 2 through the server. This functionality is the basis for a networked game and was therefore prioritized as the first milestone during the implementation stage.

Assuming the clients have connected to the server, both programs enter a loop. The server receives data from the first and second player and this data is saved. Next, the server sends the data to the opposite clients. Clients on the other hand, once they have finished all the processing of moves made during a frame, the score for the turn is sent to the server and the client waits for a response which will be the opponent score. This interaction can be seen in Figure 7 below. 

{{< figure src="/images/tetris/fig7.jpg" caption="Figure 7. Client/server game loop interaction design" >}}


While designing to meet this requirement, it was important to be cognisant of blocking and non-blocking calls to ensure the application does not become unresponsive in a hang state. On the server side, try except blocks will be used to handle the possibility one or both client sockets may not be ready for reading and it would be inefficient for the server to be in a blocking state while a different socket is ready to read. The server socket will be set to ‘non-blocking’ which means a socket method can return nothing. 

For the gameplay, it is important that the scores are updated very quickly. For this reason, the score data will be sent once per frame (game runs at 25 frames per second) as part of the game loop. The score is updated essentially instantaneously to human players at this speed, and much less than the one second outlined by Requirement 10. 

#### Requirement 3: The scoring system will reflect the original Tetris game
The current Tetris clone counts the total number of lines removed during the game as the player’s score. According to normal Tetris rules, removing multiple lines at once should be more advantageous than one at a time. Removing the maximum lines at a time, four, is called a Tetris. A new function will be implemented to provide a better scoring scheme. 
The design diagram below shows the control flow of the client to handle scoring. Only client 1 has been shown for clarity but client 2 operates in the exact same way. 

{{< figure src="/images/tetris/fig8.jpg" caption="Figure 8. Scoring Logic Detail" >}}


The ‘Tetromino’ framework game already has a function removeCompletedLines(), and calculateLevelAndFallFreq(). removeCompletedLines() is called when a piece has been placed on the board to see if the player has completed a line. The number of lines returned is added to a running total and passed to calculateLevelAndFallFreq(). calculateLevelAndFallFreq() uses this number of total lines completed to make the game harder as the player progresses through the game. Every 10 lines completed, the player advances a level and the fall speed of the pieces is increased. To implement a new scoring function, the output of removeCompletedLines() will be sent to the new turnScore() function. This new function will return the player’s score for that turn. A separate running total will be kept of the players score. 

#### Design Issues
Even with careful Requirement Engineering, planning, and research, problems always arise. This section will outline and describe two design issues encountered and the possible solutions. While designing the interaction between the client and server, two major design issues arose and were resolved. First, which communication protocol would be used, and second, how to handle and service multiple connections. 

There are two options for socket protocols [5]; Transmission Control Protocol (TCP) and User Datagram Protocol (UDP). The main features of each protocol are compared in the table below. 

**Table 3. TCP vs UDP**

TCP |	UDP
--- | ---
In order-delivery of information | Out of order delivery
Reliable | Unreliable
Error handling | No error handling
Sockets are connected | Sockets do not need to be connected
Requires more resources | Requires few resources
Library required for asynchronous functionality | Asynchronous functionality

The main tradeoffs between the two protocols are reliability versus speed. TCP ensures the information sent first is the first piece of data to arrive at the receiving socket. By contrast, UDP data can arrive out of order.

It is not guaranteed that all the data requested to be sent will be sent after one send call for either of these protocols. TCP is reliable in the sense that when send is called, the amount of data sent is returned. This returned value can be checked and then send can be called until all the data is sent. UDP sockets send out the data but do not have any concern for if the data has been received. This is because TCP sockets first connect using a 3-way handshake whereas UDP sockets are not connected and just use the host address to pass data. This trade-off means that UDP sockets are much faster than TCP and support asynchronous execution whereas TCP methods are blocking and therefore an external library is necessary. 

TCP sockets were chosen due to the advantages in reliability. It was decided that the added speed with UDP was not necessary for this application.

There are many approaches to handling multiple TCP sockets. Most techniques involve an external library which provides quasi-parallel execution or threading. During the Requirement Engineering phase, a huge amount of time was spent exploring the feasibility of using different tools and libraries for this including select, selectors, asyncio, and twisted. 

Select is the lowest level and most basic library. It is also the oldest and it would not be a good decision in terms of maintenance. Selectors is the updated version of select and has increased functionality. There are portability problems with both libraries. Many of the functions are not supported in Windows but not Linux and vice versa. After this, asyncio and twisted are much more advanced and abstracted networking tools. 
It was concluded that these tools would require more time than was available to learn and implement these advanced networking practices. The chosen solution was to use try: except: blocks and careful implementation of blocking calls. 

This technique works for this relatively small application which only supports two players but does not scale well. Most online games today use UDP sockets because of their speed and asynchronous abilities. 

## Implementation
Using the diagrams and logic created during the Design stage, the code was implemented in Python 3 using Sublime Text editor (which supports python syntax) and run using Windows command prompt. 

#### Requirement Implementation
Requirements 1,2, and 3 from the previous sections have been determined during the RE process, outlined in the design process, and will now be implemented in executable code. The following section shows the code blocks pertaining to the satisfaction of these requirements. 

{{< figure src="/images/tetris/fig9.png" caption="Figure 9. Server initialization" >}}


{{< figure src="/images/tetris/fig10.png" caption="Figure 10. Client initialization" >}}


The client and server code for the initialization and connection of the sockets is shown by Figures 9 and 10 above. This code has been commented for clarity. A simple python server example [4] was used as the base for this code.

#### Requirement 1. The Game Will Not Start Until Players Are Ready
The code implementation for Requirement 1 is shown below. The server-side code is very simple and simply loops while waiting to receive data from each client. When the server receives a ready signal from both clients it then notifies both clients to start the game and then breaks from the loop. 

{{< figure src="/images/tetris/fig11.png" caption="Figure 11. Server ready loop" >}}


The client-side code for Requirement 1 is slightly more complex. After establishing a connection to the server, the client code enters the main function as shown in Figure 12. 

{{< figure src="/images/tetris/fig12.png" caption="Figure 12. Client enters main function" >}}


Inside the main function, a PyGame instance is initialized with its window size, font, and frames per second (FPS) before starting the game. In line 191, the showTextScreen function is called. The output of this function is shown in Figure 13 below. 

{{< figure src="/images/tetris/fig13.png" caption="Figure 13. Ready screen" >}}


{{< figure src="/images/tetris/fig14.png" caption="Figure 14. Client main function" >}}


Inside showTextScreen function, the client polls for a key press using a while loop on line 399, to indicate the player is ready. Once this loop returns (a key has been pressed), the server is notified. Next, recv() is called, which is a blocking call. The program execution will not continue until some bytes have been received from the server. This acts to delay the start of the game until both players are ready, satisfying the requirement. Once the data is received, the function returns to main(), where it enters the game loop and runGame() is called. 

{{< figure src="/images/tetris/fig15.png" caption="Figure 15. Client ready loop" >}}


#### Requirement 2. Players Can See Their Opponents Score 
The following figures show the important code blocks that were implemented to satisfy Requirement 2. Figure 16 shows the server event loop which continuously tries to receive data from each client and then pass the data to the other client.

{{< figure src="/images/tetris/fig16.png" caption="Figure 16. Server event loop" >}}

Figure 17 and 18 show the client code. These code blocks are implemented at the end of the game event loop which contains all the event handling for the game. After each frame or clock tick, the score is totalled and sent to the server and then the opponents score is received. 

{{< figure src="/images/tetris/fig17.png" caption="Figure 17. Client Send Logic" >}}

After receiving the opponent’s score, both scores along with the level and number of lines completed values are displayed to the screen using the drawStatus function. 

In Figure 18 below, the call to drawStatus can be seen with four input arguments. Also shown are functions from the framework program related to drawing the pieces on the board, updating the display, and advancing to the next frame. 

{{< figure src="/images/tetris/fig18.png" caption="Figure 18. Client Display Logic" >}}

Figure 19 below shows the drawStatus function in more detail. Much of the code here uses the PyGame library [6] to display graphics on the game screen. The function takes four string arguments and displays them on the screen in their correct positions as shown in Figure 20.

{{< figure src="/images/tetris/fig19.png" caption="Figure 19. drawStatus Function" >}}


It is important to note that PyGame uses the top left corner as (0,0) when specifying positions on the screen. 

{{< figure src="/images/tetris/fig20.png" caption="Figure 20. Game Screen" >}}


Figure 20 is a screen capture from a game instance with drawStatus implemented. The four variables from drawStatus have been displayed on the screen. 

#### Requirement 3: The scoring system will reflect the original Tetris game
The scoring system was updated as discussed in the requirements and design sections above. The implementation was relatively simple and involved creating a separate variable and function to handle both the number of lines completed and the score which can vary drastically depending on whether the player is eliminating one or multiple lines at a time. 

{{< figure src="/images/tetris/fig21.png" caption="Figure 21. Client Keeping Score Logic" >}}

Figure 21 is a conditional statement nested within the game event loop and then the end of the loop which was previously shown in Figure 17. The conditional branch is entered if a piece has been placed on the board. The output of the removeCompletedLines function is passed to a new function, turnScore(), as shown in Figure 22 below, which returns the players score based on the number of lines removed in one move. The variable ‘pts’ was added to keep a running total of the score while the ‘score’ variable was already used to keep track of the total lines which is a key part of determining the player level and how fall the pieces fall. The naming could be changed but at the risk of creating faults. It was decided that the naming conventions could remain as they are essentially arbitrary. An IDE could be used to find all instances and change the name but that should be done with caution. 

{{< figure src="/images/tetris/fig22.png" caption="Figure 22. Client turnScore Function" >}}


#### Version Control
The decentralized version control software (VCS), Git, was used for tracking changes and allowing the version to be rolled back if a new feature introduced a bug. The project can be found on GitHub at https://github.com/tmastrom/Tetris. 

#### Good Coding Habits
Good Python coding habits were observed in the making of this project. All code is commented for clarity, global variables use CAPS, function names and variables use camelCase. This application was developed in a virtual environment to keep track of dependencies. Libraries used socket, time, random, PyGame. 

#### Software Development Life Cycle
This project was developed using Agile development practices. Requirements were chosen in terms of priority and then they were designed and implemented until the requirement was satisfied. 
#### Testing and Verification
The testing and verification of this application was performed using a variety of techniques. The main technique was through black and white box testing. The system was exercised as much as possible in order to exhibit errors. Then the code was traced through to determine the error sources. With approximately 600 lines of code is relatively easy to perform white box testing. 

Requirement 1 is very simple to test using the black box method. The server will not allow the game to start unless two clients have connected and sent the ready signal. Further optimization is needed to handle errors like a player disconnecting or quitting but it was determined that the game cannot start unless both clients have sent their ready signals.
Requirement 2 was tested using the black box method along with the white box method. The game was played, scoring every possible number of lines (0-4) and the correct score was shown on both players screens effectively instantaneously. This ties into the Requirement 3 testing as well, which confirmed that the correct score was added based on the players moves. Print statements were added to trace the flow of the code through different functions and ensure that it was behaving as expected. 
#### Maintenance
Software maintenance is a massive cost over the lifetime of a software. Corrective maintenance will be pursued once more bugs arise in beta testing. Adaptive maintenance is not concerning at the time of writing this report. The game was built using the latest stable build of Python, version 3.7.1, which was released 10/20/2018 [7]. Python 3 itself came out at the end of 2008 which is relatively new for software (especially compared with C). The libraries used by this application (socket, time, random, PyGame) are widely used in the community and are at low risk of falling out of use. However, if this application is in use long enough, modifications to library functionality as well as Python syntax will require adaptive maintenance to remain useable. 

The most time and energy is usually spent on perfective maintenance to optimize the application further. There is always room to optimize code. A few examples of optimization that could be applied to this application include increasing abstraction, optimizing the network interaction, and changing the hosting. Abstraction could be increased by refactoring the code to be more object-oriented. Creating classes with methods would improve the the current situation which simply uses functions. The network interface could run more efficiently if an asynchronous library or UDP sockets were implemented. Currently, information is passed every time the program runs through the event loop. A better alternative would be to only send the data if the score was updated that loop. The application is currently hosted on the localhost ‘127.0.0.1’ and this means the application only runs on a single computer’s network. Moving to an online host would allow the game to be played over the internet by different computers. 

## Conclusion
In conclusion, developing 2-Player Tetris Networked Game Application was an excellent introduction to networking and software development good practices. The techniques practiced during the SDLC such as Agile development, testing strategies, as well as diagrams created in the Requirement Engineering and Design phases are applicable to the workplace today. 

