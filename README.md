# CTFGAME
 Catch the flag Game Made in Unreal

CATCH THE FLAG GAME 


SIMPLE MULTIPLAYER GAME MADE USING UNREAL ENGINE 4.27


Made on top of the First Person Shooter template in UE.


C++ files -


GameInstance -


Game Instance manages the sessions in the game. This includes creating, finding, joining, updating and destroying a session. After creating a session, the server travels to the lobby from the Main Menu. Then clients search for the available sessions and join a session, upon which they travel to the lobby. Whenever a player joins a session, we update the session in Lobby Game Mode (as soon as the player enters the lobby). In the update server function, before updating the session we update the number of open public connections. When the update is completed, we check if the session is full, upon which we ServerTravel to the Gameplay Level. When we end and destroy the server, we travel to the Main Menu. We also handle network error, upon which we close the session and travel to the Main Menu.
 
TaskCharacter -


First I implemented the game for 2 players in Unreal Engine. I could then set up an Online Subsystem and Lobby later. First thing I did was spawn projectiles on the server instead of base implementation. So, I moved the projectile spawning function to a reliable server RPC. If the player has been hit by the projectile then we decrease player health stored in Player State. If the health drops to or below zero, then we make the player transparent to appear as a ghost and then start the timer for 5 seconds. We also call client RPC to show the Death Icon on the player’s screen. After 5 seconds we spawn the player again at the Start Location, it was spawned at the beginning of the game which we stored in the BeginPlay function. We also call the client RPC to hide Death Icon. We then possess the character and restore health. If the player had been carrying the enemy’s flag, then we drop the flag and update the player state accordingly. Then we destroy the current character.


TaskProjectile -


Out of the box projectile for Gun Launcher.


CTFGameState -


Game State is used to hold Game specific information that is needed by all the players in the game. One such function is Timer that runs on the widget on top of the screen. The String holding the Game Time is set to replicate. Whenever the Game Timer string gets updated, the timer widget updates the timer it shows. Another function of Game State is to hold a Scoreboard in the form of a Map. It allows for retrieval and modification of scores on the scoreboard. At the end of the game it sorts the team based on Score and returns the Winning Team (or Tie in case top two players have the same score).


CTFPlayerController -


The Player Controller is used for two things. First, When Player State is replicated (which is when server creates an instance of Player State and replicates it to the client), then we Hide Flag Icon which is visible by default and also Show Player Health. Second, We need to destroy sessions on all the clients in addition to the server. This is done through player controllers. On each player controller, we destroy sessions on the player's game instance.


CTFPlayerStart -


Player Start needs to have a reference for the Player Base it is located near. We need this to spawn players properly.




CustomHUDActor -
 
This Actor is used by Startup Map to show the startup screen. All this Map does is display a message asking the user to press ENTER to go to the Main Menu. We make a widget component and set its various properties and text in BeginPlay. Then we add it to the viewport. We bind the input ENTER (in project settings) to Go to the Main Menu. Once the user presses ENTER, we remove all the widgets from the screen and go to the main menu from Game Mode.


Enum.h -


We store two enums. One for identifying the Team and other for Match Decision.


Struct.h -


We use struct FSWin to store match results in case it’s a win or tie.


TaskGameMode -


This is the base Game Mode for all the Game Modes used in the Game.


TaskGameModeGameplay -


This is the game mode used by Gameplay Level. We set up two timers in Gameplay Game Mode. One when the game starts, so we can end the match after 1 minute. Another to display the final winning message for 5 seconds before exiting to Main Menu. It also shows the winning result via server Player State which is then replicated and shown on all the clients. It closes the session when the game ends on the server and all the connected clients and then travels to the Main Menu.


TaskGameModeLobby -


There is a fault in Game Instance logic due to which it doesn’t update the number of public connections. Therefore, we manually update the number of open public connections in the update server function. Therefore, we update the session in the PostLogin method of Lobby Game Mode, which is when a new client has joined the session and entered the lobby. The lobby has a timer of 3 minutes after which if no client joins the session (or fills the session for more than 2 players), then the session is closed on both the server and connected clients.






TaskGameModeStartup -


This game mode has just one function for going to the Main Menu by opening the Main Menu Map.
 


TeamBase -


Base has various functions such as IsEnemy function which checks whether a given character is enemy to the owner of the base. If the player arrives at his own base with the enemy flag then they are awarded a point. Flag is at the base when the match starts,


TeamFlag -


IsEnemy function checks whether a character is enemy to the owner of the flag. Hide flag and Show flag functions are implemented as server RPCs because we want the state to be replicated on both server and clients. Instead of respawning flags at different locations we just hide and show flags. If the enemy overlaps the flag then they capture the flag. If the owner overlaps the flag when the flag is not at the base (dropped by the enemy), then the flag is returned to the owner's base. Both Capture the flag and Return to base are implemented as server RPCs and involve mostly setting booleans. Flag is spawned at the base location.
TeamPlayerState -


Player State stores the team player belongs to, health of the player, whether the player has captured the enemy flag and references to player’s base and enemy’s flag  .It is used to interact with game widgets such as death icon, flag icon, health bar and winning message. Flag icon, death icon and health bar functions are client RPCs because we want them to work only on clients and do not want them to be replicated. Client’s Show winning result is NetMulticast call because we want the winning message to be displayed on all the clients. Team and Base is set in the Set Team and Base function.
Blueprints -


Create Blueprint classes from C++ classes.
Also import a Flag asset into the Game.


Project Settings -


Go to Maps & Modes settings. We need to set the default GameMode to Startup GM. Editor startup map to Gameplay Map. Game default map to Startup Map. Set the Game Instance class to Task Game Instance.


Game Framework -


Character BP -
Set to replicate.


Projectile BP -
Set to replicate.


Team Base BP -
We add a spotlight and capsule component to Team Base. We set the reference to the base flag in BeginPlay. We also store a reference to Base Flag. We adjust the spotlight for color and angle in Gameplay Level and reset instance changes to blueprint class. 


Team Flag BP -
We add static mesh and a box component to Team Flag. We set the reference to the base flag in the team base in BeginPlay. We also have a reference to Team Base. 


Game State BP -
Game State is used to call widget functions. It has a reference to Game widget and calls widget’s functions to show timer and game score.


Player Controller BP -
Just a plain BP class created from C++ class.


Player Start BP -
Just a plain BP class created from C++ class.


Player State BP -
Player State facilitates many widget functions for death icon, flag icon and health bar. In BeginPlay, we set the reference to Game Widget, getting it from the game state. Then we call the associated functions for the game widget. In the Show Winning Result, we prepare a string such as “It’s a Tie!” if the game is a tie, or “Team Read Wins!” in the case a team wins. We then call the Show Winning Message from the game from the Game Widget.




Gameplay Game Mode BP -
Gameplay GM shows Game Timer on Game Widget. We override the ChoosePlayerStart function such that we select a player start that is not yet taken. We then set the team and base in the player state of the player that occupies the player start. 


Lobby Game Mode BP -
We override the ChoosePlayerStart function such that we select a player start that is not yet taken.


Main Menu Game Mode BP -
Just a plain BP class created from C++ class.


Startup Game Mode BP -
Just a plain BP class created from C++ class.


Widget -


Game Widget -


Contains UI for Game score, timer, flag icon, death icon, heath bar and winning message. The implementation of associated functions is trivial.


Main Menu Widget -


Contains three Canvas Panels inside a Widget Switcher. One for the Main screen, one for Create Server and another for Find Server. This widget works with Game Instance to create, find and join sessions. Joining the server is enabled by the ServerSlot widget. Logic for functions is trivial.


Server Slot Widget -


This widget is used by the Main Menu Widget. It shows the number of players out of total players allowed in a session. Then the player can click Join Server to join the match. Logic is trivial.


Startup Widget -


Just a message saying “ENTER TO MAIN MENU”. Press enter to go to the main menu.


Maps -
Set appropriate Game Modes for all the levels.


Testing -


There is testing incompatibility for game instances as of UE 4.27Build the project. Then use the batch file to start the game as stand alone.
