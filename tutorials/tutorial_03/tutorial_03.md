# Tutorial 3: Communicating and Playing Games

**To be completed during *next* class**

## Overview

In this final tutorial, you will:

- Test your messaging app with classmates
- Learn best practices for network communication
- Build a **Rock, Paper, Scissors** game that uses your messaging system
- Play games with your classmates through the network!

This tutorial is divided into two parts:

1. **Testing the messaging app** - Connect with classmates and chat
2. **Building the game system** - Create an interactive game client

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 1: Testing Your Messaging App

### Step 1: Start Your Server

One person should run the server. This can be:
- You (on your machine)
- A classmate
- Both (different ports!)

Start the server from Tutorial 1. From the workspace root directory, navigate to the tutorials folder and run:

```bash
cd tutorials/
uv run tutorial_01/src/message_server.py
```

### Step 2: Connect Multiple Clients

**On the same computer** (for testing):

- Open 2-3 terminal windows
- Run the client in each window with different usernames

**With classmates** (for real fun!):

- Get the server's IP address
  - On macOS/Linux: `ifconfig | grep inet`
  - On Windows: `ipconfig`
  - Look for something like `192.168.1.100`
- Share this IP with classmates
- They modify their client code to use your IP

### Step 3: Chat and Exchange Messages

Try these activities:

1. ✅ Send greetings to each other
2. ✅ Have a quick conversation
3. ✅ Test disconnecting and reconnecting
4. ✅ See what happens when someone types very long messages
5. ✅ Try having 3-4 people chatting simultaneously

### Connection Examples

**Local testing** (same computer):

```python
client = MessageClient(host='localhost', port=5555, username='Alice')
```

**Remote connection** (different computers on same network):

```python
client = MessageClient(host='192.168.1.100', port=5555, username='Bob')
```

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 2: Building the Rock, Paper, Scissors Game

Now let's create something fun! We'll build a game system that uses our messaging infrastructure.

**Note**: You'll use the same virtual environment from Tutorial 1. Make sure you're in the `tutorials/` directory.

### Create the Game Client

Open `src/game_client.py` and copy-paste this complete game system:

```python
"""
Rock, Paper, Scissors Game Client
An interactive game using socket communication with classes and exceptions.
"""

import socket
import threading
import time
import random
from colorama import Fore, Back, Style, init
from typing import Optional, Dict

# Initialize colorama
init(autoreset=True)


class GameError(Exception):
    """Custom exception for game-related errors"""
    pass


class RPSGame:
    """
    Rock, Paper, Scissors game logic.
    
    Attributes:
        choices: Valid game moves
        wins: Dictionary mapping winner moves to loser moves
    """
    
    ROCK = "🪨 rock"
    PAPER = "📄 paper"
    SCISSORS = "✂️  scissors"
    
    def __init__(self):
        """Initialize game rules."""
        self.choices = [self.ROCK, self.PAPER, self.SCISSORS]
        self.wins = {
            self.ROCK: self.SCISSORS,
            self.SCISSORS: self.PAPER,
            self.PAPER: self.ROCK
        }
        
    def determine_winner(self, move1: str, move2: str) -> str:
        """
        Determine the winner of a round.
        
        Args:
            move1: First player's move
            move2: Second player's move
            
        Returns:
            Result string: "win", "lose", or "tie"
        """
        if move1 == move2:
            return "tie"
        elif self.wins[move1] == move2:
            return "win"
        else:
            return "lose"
            
    def get_computer_move(self) -> str:
        """Generate a random computer move."""
        return random.choice(self.choices)


class GameClient:
    """
    A game client that connects to a server and plays Rock, Paper, Scissors.
    
    Attributes:
        host (str): Server IP address
        port (int): Server port
        username (str): Player username
        client_socket (socket): Connection to server
        connected (bool): Connection status
        game (RPSGame): Game logic handler
        score (Dict): Score tracker
    """
    
    def __init__(self, host: str = 'localhost', port: int = 5555, username: str = "Player"):
        """Initialize the game client."""
        self.host = host
        self.port = port
        self.username = username
        self.client_socket: Optional[socket.socket] = None
        self.connected = False
        self.game = RPSGame()
        self.score = {"wins": 0, "losses": 0, "ties": 0}
        self.game_mode = "pvp"  # "pvp" (player vs player) or "pvc" (player vs computer)
        
    def connect(self) -> None:
        """Connect to the game server."""
        try:
            self.client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.client_socket.settimeout(10)
            
            print(f"{Fore.CYAN}🔌 Connecting to game server at {self.host}:{self.port}...{Style.RESET_ALL}")
            
            self.client_socket.connect((self.host, self.port))
            self.client_socket.settimeout(None)
            self.connected = True
            
            # Display game welcome
            print(f"\n{Back.MAGENTA}{Fore.WHITE}{'='*60}{Style.RESET_ALL}")
            print(f"{Back.MAGENTA}{Fore.WHITE}  🎮 ROCK, PAPER, SCISSORS GAME! 🎮  {Style.RESET_ALL}")
            print(f"{Back.MAGENTA}{Fore.WHITE}{'='*60}{Style.RESET_ALL}\n")
            print(f"{Fore.YELLOW}👤 Player: {Fore.WHITE}{self.username}{Style.RESET_ALL}")
            print(f"{Fore.GREEN}{'='*60}{Style.RESET_ALL}\n")
            
        except socket.timeout:
            raise GameError("Connection timed out. Is the server running?")
        except ConnectionRefusedError:
            raise GameError("Connection refused. Start the server first!")
        except Exception as e:
            raise GameError(f"Failed to connect: {e}")
            
    def display_menu(self) -> None:
        """Display the game mode selection menu."""
        print(f"\n{Fore.CYAN}╔{'═'*58}╗{Style.RESET_ALL}")
        print(f"{Fore.CYAN}║{Fore.YELLOW} GAME MODE SELECTION{' '*38}║{Style.RESET_ALL}")
        print(f"{Fore.CYAN}╠{'═'*58}╣{Style.RESET_ALL}")
        print(f"{Fore.CYAN}║  {Fore.WHITE}1. 👥 Play against another player (PvP){' '*17}║{Style.RESET_ALL}")
        print(f"{Fore.CYAN}║  {Fore.WHITE}2. 🤖 Play against computer (PvC){' '*23}║{Style.RESET_ALL}")
        print(f"{Fore.CYAN}║  {Fore.WHITE}3. 💬 Just chat (no game){' '*30} ║{Style.RESET_ALL}")
        print(f"{Fore.CYAN}║  {Fore.WHITE}4. 🚪 Quit{' '*45} ║{Style.RESET_ALL}")
        print(f"{Fore.CYAN}╚{'═'*58}╝{Style.RESET_ALL}\n")
        
    def play_vs_computer(self) -> None:
        """Play Rock, Paper, Scissors against the computer."""
        print(f"\n{Fore.GREEN}🤖 Starting Player vs Computer mode!{Style.RESET_ALL}")
        print(f"{Fore.YELLOW}Type 'quit' to return to menu{Style.RESET_ALL}\n")
        
        while self.connected:
            # Display move options
            print(f"{Fore.CYAN}{'─'*60}{Style.RESET_ALL}")
            print(f"{Fore.YELLOW}Choose your move:{Style.RESET_ALL}")
            print(f"  1. {RPSGame.ROCK}")
            print(f"  2. {RPSGame.PAPER}")
            print(f"  3. {RPSGame.SCISSORS}")
            print(f"{Fore.CYAN}{'─'*60}{Style.RESET_ALL}")
            
            choice = input(f"{Fore.WHITE}Your move (1-3 or 'quit'): {Style.RESET_ALL}").strip()
            
            if choice.lower() == 'quit':
                break
                
            # Validate input
            if choice not in ['1', '2', '3']:
                print(f"{Fore.RED}❌ Invalid choice! Please enter 1, 2, or 3{Style.RESET_ALL}\n")
                continue
                
            # Get player and computer moves
            player_move = self.game.choices[int(choice) - 1]
            computer_move = self.game.get_computer_move()
            
            # Display moves
            print(f"\n{Fore.LIGHTBLUE_EX}Your move: {player_move}{Style.RESET_ALL}")
            print(f"{Fore.LIGHTMAGENTA_EX}Computer move: {computer_move}{Style.RESET_ALL}")
            
            # Determine winner
            result = self.game.determine_winner(player_move, computer_move)
            
            # Display result with appropriate color and emoji
            if result == "win":
                print(f"{Back.GREEN}{Fore.BLACK}  🎉 YOU WIN! 🎉  {Style.RESET_ALL}")
                self.score["wins"] += 1
            elif result == "lose":
                print(f"{Back.RED}{Fore.WHITE}  😞 YOU LOSE! 😞  {Style.RESET_ALL}")
                self.score["losses"] += 1
            else:
                print(f"{Back.YELLOW}{Fore.BLACK}  🤝 IT'S A TIE! 🤝  {Style.RESET_ALL}")
                self.score["ties"] += 1
                
            # Display score
            self.display_score()
            print()
            
    def play_vs_player(self) -> None:
        """Play Rock, Paper, Scissors against another player through the server."""
        print(f"\n{Fore.GREEN}👥 Starting Player vs Player mode!{Style.RESET_ALL}")
        print(f"{Fore.CYAN}📢 Announce your moves in the chat (e.g., 'My move: rock'){Style.RESET_ALL}")
        print(f"{Fore.YELLOW}💡 Honor system: Don't change your move after seeing opponent's!{Style.RESET_ALL}")
        print(f"{Fore.YELLOW}Type 'quit' to return to menu{Style.RESET_ALL}\n")
        
        # Start receive thread
        receive_thread = threading.Thread(target=self.receive_messages, daemon=True)
        receive_thread.start()
        
        while self.connected:
            # Display move options
            print(f"{Fore.CYAN}{'─'*60}{Style.RESET_ALL}")
            print(f"{Fore.YELLOW}Choose your move:{Style.RESET_ALL}")
            print(f"  1. {RPSGame.ROCK}")
            print(f"  2. {RPSGame.PAPER}")
            print(f"  3. {RPSGame.SCISSORS}")
            print(f"  4. 💬 Send a chat message")
            print(f"{Fore.CYAN}{'─'*60}{Style.RESET_ALL}")
            
            choice = input(f"{Fore.WHITE}Your choice (1-4 or 'quit'): {Style.RESET_ALL}").strip()
            
            if choice.lower() == 'quit':
                break
                
            if choice == '4':
                # Send chat message
                message = input(f"{Fore.CYAN}Message: {Style.RESET_ALL}")
                self.send_message(f"[{self.username}]: {message}")
                continue
                
            # Validate input
            if choice not in ['1', '2', '3']:
                print(f"{Fore.RED}❌ Invalid choice! Please enter 1-4{Style.RESET_ALL}\n")
                continue
                
            # Get player move
            player_move = self.game.choices[int(choice) - 1]
            
            # Announce move to all players
            message = f"[{self.username}] 🎮 MY MOVE: {player_move}"
            self.send_message(message)
            
            print(f"{Fore.LIGHTBLUE_EX}✅ Move announced to other players!{Style.RESET_ALL}\n")
            
    def chat_mode(self) -> None:
        """Basic chat mode without games."""
        print(f"\n{Fore.GREEN}💬 Chat mode active!{Style.RESET_ALL}")
        print(f"{Fore.YELLOW}Type messages to chat, 'quit' to return to menu{Style.RESET_ALL}\n")
        
        # Start receive thread
        receive_thread = threading.Thread(target=self.receive_messages, daemon=True)
        receive_thread.start()
        
        while self.connected:
            message = input()
            
            if message.lower() == 'quit':
                break
                
            if message.strip():
                self.send_message(f"[{self.username}]: {message}")
                print(f"{Fore.LIGHTBLUE_EX}📤 You: {message}{Style.RESET_ALL}")
                
    def send_message(self, message: str) -> None:
        """Send a message to the server."""
        try:
            self.client_socket.send(message.encode('utf-8'))
        except Exception as e:
            print(f"{Fore.RED}❌ Error sending message: {e}{Style.RESET_ALL}")
            self.connected = False
            
    def receive_messages(self) -> None:
        """Receive messages from the server."""
        while self.connected:
            try:
                message = self.client_socket.recv(1024).decode('utf-8')
                
                if not message:
                    print(f"\n{Fore.RED}❌ Server closed connection{Style.RESET_ALL}")
                    self.connected = False
                    break
                    
                # Display received messages
                if "MY MOVE:" in message:
                    print(f"\n{Fore.LIGHTMAGENTA_EX}📨 {message.strip()}{Style.RESET_ALL}")
                else:
                    print(f"\n{Fore.LIGHTGREEN_EX}📨 {message.strip()}{Style.RESET_ALL}")
                    
            except Exception as e:
                if self.connected:
                    print(f"{Fore.RED}❌ Error receiving: {e}{Style.RESET_ALL}")
                break
                
    def display_score(self) -> None:
        """Display the current score."""
        total = self.score["wins"] + self.score["losses"] + self.score["ties"]
        print(f"\n{Fore.CYAN}{'─'*60}{Style.RESET_ALL}")
        print(f"{Fore.YELLOW}📊 SCORE: {Style.RESET_ALL}", end="")
        print(f"{Fore.GREEN}Wins: {self.score['wins']}  {Style.RESET_ALL}", end="")
        print(f"{Fore.RED}Losses: {self.score['losses']}  {Style.RESET_ALL}", end="")
        print(f"{Fore.YELLOW}Ties: {self.score['ties']}  {Style.RESET_ALL}", end="")
        print(f"{Fore.CYAN}Total: {total}{Style.RESET_ALL}")
        print(f"{Fore.CYAN}{'─'*60}{Style.RESET_ALL}")
        
    def start(self) -> None:
        """Start the game client and main menu loop."""
        try:
            self.connect()
            
            while self.connected:
                self.display_menu()
                choice = input(f"{Fore.WHITE}Select option (1-4): {Style.RESET_ALL}").strip()
                
                if choice == '1':
                    self.play_vs_player()
                elif choice == '2':
                    self.play_vs_computer()
                elif choice == '3':
                    self.chat_mode()
                elif choice == '4':
                    print(f"{Fore.YELLOW}👋 Thanks for playing!{Style.RESET_ALL}")
                    self.disconnect()
                    break
                else:
                    print(f"{Fore.RED}❌ Invalid option! Please choose 1-4{Style.RESET_ALL}")
                    
        except GameError as e:
            print(f"{Fore.RED}❌ Game Error: {e}{Style.RESET_ALL}")
        except KeyboardInterrupt:
            print(f"\n{Fore.YELLOW}⚠️  Keyboard interrupt{Style.RESET_ALL}")
        finally:
            self.disconnect()
            
    def disconnect(self) -> None:
        """Disconnect from the server."""
        if not self.connected:
            return
            
        self.connected = False
        
        if self.client_socket:
            try:
                self.client_socket.close()
            except:
                pass
                
        print(f"\n{Fore.YELLOW}{'='*60}")
        print(f"👋 Disconnected from server")
        print(f"{'='*60}{Style.RESET_ALL}\n")
        
        # Display final score if any games were played
        total_games = self.score["wins"] + self.score["losses"] + self.score["ties"]
        if total_games > 0:
            print(f"{Fore.CYAN}📊 FINAL STATS:{Style.RESET_ALL}")
            self.display_score()


def get_local_ip():
    """
    Get the local IP address of this machine.
    Returns a string like '192.168.1.100'
    """
    try:
        # Create a temporary socket
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        
        # Connect to an external address (doesn't actually send data)
        # This helps us determine which network interface to use
        s.connect(("8.8.8.8", 80))
        
        # Get our IP address
        ip_address = s.getsockname()[0]
        
        s.close()
        return ip_address
    except Exception:
        return "127.0.0.1"  # Fallback to localhost

def main():
    """Main function to start the game client."""
    # Display banner
    print(f"\n{Back.MAGENTA}{Fore.WHITE}{'='*60}{Style.RESET_ALL}")
    print(f"{Back.MAGENTA}{Fore.WHITE}  🎮 WELCOME TO ROCK, PAPER, SCISSORS! 🎮  {Style.RESET_ALL}")
    print(f"{Back.MAGENTA}{Fore.WHITE}{'='*60}{Style.RESET_ALL}\n")

    print(f"\n{Fore.CYAN}{'='*35}{Style.RESET_ALL}")
    my_ip = get_local_ip()
    print(f"{Fore.CYAN}Your IP address is: {my_ip} {Style.RESET_ALL}")
    print(f"{Fore.CYAN}{'='*35}{Style.RESET_ALL}")
    
    # Get username
    username = input(f"{Fore.YELLOW}Enter your player name: {Style.RESET_ALL}").strip()
    
    if not username:
        username = "Player"
        
    print(f"{Fore.GREEN}✅ Welcome, {username}!{Style.RESET_ALL}")
    
    # Get the IP address to connect to
    msg = f"\n{Fore.YELLOW}🙂 Enter the IP address of the server to connect to (default: {Fore.CYAN}{my_ip}{Fore.YELLOW}): {Style.RESET_ALL}"

    myNew_ip = input(msg).strip()
    if myNew_ip:
        my_ip = "localhost"

    msg = f"{Fore.GREEN}✅ Connecting to server at {my_ip}:{5555}...{Style.RESET_ALL}"
    print(msg)
    
    # Create and start client
    # client = GameClient(host='localhost', port=5555, username=username)
    client = GameClient(host=my_ip, port=5555, username=username)
    client.start()


if __name__ == "__main__":
    main()
```

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 3: Understanding the Game Code

### **Game Logic Class**

```python
class RPSGame:
    def determine_winner(self, move1, move2):
        if move1 == move2:
            return "tie"
        elif self.wins[move1] == move2:
            return "win"
```
- Separate class for game rules
- Dictionary maps winning moves to losing moves
- Clean separation of game logic from networking

### **Multiple Game Modes**

```python
def play_vs_computer(self):
    # Play against AI
    
def play_vs_player(self):
    # Play through network
```

- PvC mode: Single-player against random AI
- PvP mode: Multiplayer through server
- Chat mode: Just messaging

### **Score Tracking**

```python
self.score = {"wins": 0, "losses": 0, "ties": 0}
```

- Persistent score across rounds
- Displayed after each game
- Final stats shown on exit

### **Enhanced Visual Design**

```python
print(f"{Back.MAGENTA}{Fore.WHITE}  🎮 GAME! 🎮  {Style.RESET_ALL}")
```

- Box-drawing characters for menus
- Colorful backgrounds for results
- Emojis for visual appeal

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 4: Play the Game!

### Understanding IP Address Connection

Like the message client in Tutorial 2, the game client now includes IP address detection and selection. When you start the game, you'll see:

```
============================================================
  🎮 WELCOME TO ROCK, PAPER, SCISSORS! 🎮
============================================================

===================================
Your IP address is: 192.168.1.100
===================================

Enter your player name: Player1
✅ Welcome, Player1!

🙂 Enter the IP address of the server to connect to (default: 192.168.1.100):
```

**Connection Options for Game Players:**

1. **Solo Testing (localhost)**: Type anything to connect to `localhost` for local testing
2. **Network Play (your IP)**: Press Enter to connect using your detected IP address
3. **Connect to Classmate**: Type their IP address to join their game server

### Solo Testing (vs Computer)

1. Start the server (Tutorial 1) on localhost
2. Navigate to the tutorials directory and run the game client:

```bash
cd tutorials/
uv run tutorial_03/src/game_client.py
```

3. Enter your name (e.g., "Player1")
4. When prompted for IP:
   - Type anything (e.g., "local") and press Enter to use localhost
5. Select option 2 (Play against computer)
6. Choose rock, paper, or scissors
7. Play multiple rounds!

### Multiplayer (vs Friends) - Same Computer

For testing with multiple clients on the same computer:

1. Start the server on localhost (Tutorial 1)
2. Open 2-3 terminal windows
3. In each terminal, start the game client:
   ```bash
   cd tutorials/
   uv run tutorial_03/src/game_client.py
   ```
4. For each client:
   - Enter a different player name
   - Type anything when prompted for IP (uses localhost)
5. Each player selects option 1 (Play against another player)
6. Players take turns announcing moves
7. Compare results and celebrate wins!

### Multiplayer (vs Friends) - Different Computers

For networked gameplay with classmates:

1. **Server Host**:
   - Start the server (Tutorial 1)
   - Note your IP address displayed (e.g., `192.168.1.100`)
   - Press Enter to bind to your network IP
   - Share your IP address with other players

2. **Game Players**:
   - Start the game client
   - Enter your player name
   - When prompted for server IP, type the host's IP address (e.g., `192.168.1.100`)
   - Select option 1 (Play against another player)

3. **Play the Game**:
   - Players announce moves through the chat
   - Compare results manually
   - Keep playing and have fun!

**Honor System**: In PvP mode, players announce their moves simultaneously. Don't change your move after seeing your opponent's choice!

### Just Chat

Select option 3 to use the regular messaging feature without games.

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 5: Creative Extensions

Got some time and want to make the game even better!? Try these ideas (or come up with your own!)

### **Add More Games**

- Create a `GuessTheNumber` class
- Implement `TicTacToe` with a grid display
- Add `21` (blackjack-style card game)

### **Enhance the Server**

- Add player name registration
- Implement game rooms (multiple separate games)
- Create a leaderboard

### **Improve the Game**

- Add best-of-3 or best-of-5 rounds
- Implement timeouts for player responses
- Create tournament brackets

### **Visual Enhancements**

- Add ASCII art for rock, paper, scissors
- Create animated countdowns ("3... 2... 1... GO!")
- Add sound effects (use the `playsound` library)

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 6: Reflection Questions Preview

After completing all tutorials, answer these questions in `writing/reflection.md`:

1. **How do sockets enable network communication?**
2. **Why is threading necessary for the client and server?**
3. **How do exceptions improve error handling?**
4. **What happens if two players disconnect simultaneously?**
5. **How would you modify the code to support more than 2 players in a game?**

## Congratulations!

You've built a complete network application with:
- ✅ Socket programming (client-server architecture)
- ✅ Classes for code organization
- ✅ Exception handling for robust error management
- ✅ Threading for concurrent operations
- ✅ Colorful, engaging user interface
- ✅ Interactive gameplay

**Most importantly**: You've created something fun that you can actually use with friends!

## Troubleshooting

**Game not connecting?**

- Server must be running first
- Check host and port settings match

**Moves not syncing in PvP?**

- This is normal - it's honor-system based
- Both players announce moves, then manually compare

**Want automatic move comparison?**

- This requires more complex server logic
- Consider implementing as an extension!

**Score resets?**

- Score is per-session only
- Restart = new score
- Extension idea: Save scores to file

## Final Tips

- **Test thoroughly**: Try error cases (disconnect mid-game, invalid input)
- **Have fun**: Play with classmates during testing time
- **Be creative**: Add your own features and improvements
- **Share**: Show off your game to others!
