# Tutorial 2: Building the Message Client

**To be completed during *this* class**

## Overview

In this tutorial, you will create a **colorful message client** that connects to your server. The client will:

- Connect to the server using sockets
- Send messages typed by the user
- Receive and display messages from other users
- Use colored output to distinguish different message types
- Handle exceptions for network errors and disconnections
- Use threading to send and receive messages simultaneously

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 1: Understanding the Client Architecture

A message client needs to:

1. **Connect** to the server using its IP address and port
2. **Send** user messages to the server
3. **Receive** messages from the server (sent by other clients)
4. **Display** messages with color coding for clarity
5. **Handle disconnections** gracefully

We'll use **two threads**: one for sending messages and one for receiving them, so both can happen simultaneously.

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 2: Set Up the Project Environment

**Note**: If you completed Tutorial 1, you already set up the virtual environment. Skip this section and proceed to Part 3.

If you're starting here, you need to set up the virtual environment:

From the workspace root directory, navigate to the tutorials folder:

```bash
cd tutorials/
```

Initialize the project (only need to do this once):

```bash
uv init --bare
```

Install colorama (only need to do this once):

```bash
uv add colorama
```

**Remember**: You should be in the `tutorials/` directory for all run commands.

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 3: Create the Client Code

Open `tutorial_02/src/message_client.py` and copy-paste the complete code below:

```python
"""
Message Client with Classes and Exception Handling
This client connects to a message server and allows real-time communication.
"""

import socket
import threading
from colorama import Fore, Back, Style, init
from typing import Optional

# Initialize colorama
init(autoreset=True)


class NetworkError(Exception):
    """Custom exception for network-related errors"""
    pass


class MessageClient:
    """
    A message client that connects to a server and enables real-time chat.
    
    Attributes:
        host (str): Server IP address to connect to
        port (int): Server port number
        client_socket (socket): The socket connection to the server
        username (str): Display name for this client
        connected (bool): Connection status flag
    """
    
    def __init__(self, host: str = 'localhost', port: int = 5555, username: str = "User"):
        """
        Initialize the client with connection parameters.
        
        Args:
            host: Server IP address to connect to
            port: Server port number
            username: Display name for this user
        """
        self.host = host
        self.port = port
        self.client_socket: Optional[socket.socket] = None
        self.username = username
        self.connected = False
        
    def connect(self) -> None:
        """
        Establish connection to the server.
        
        Raises:
            NetworkError: If connection fails
        """
        try:
            # Create TCP/IP socket
            self.client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            
            # Set a timeout for connection attempt (10 seconds)
            self.client_socket.settimeout(10)
            print(f"{Fore.YELLOW}🔍 Attempting to connect to server at {self.host} {Style.RESET_ALL}")
            print(f"{Fore.CYAN}🔌 Connecting to server at {self.host}:{self.port}...{Style.RESET_ALL}")
            
            # Connect to the server
            self.client_socket.connect((self.host, self.port))
            
            # Remove timeout for normal operations
            self.client_socket.settimeout(None)
            
            self.connected = True
            
            # Display colorful welcome
            print(f"\n{Back.GREEN}{Fore.BLACK}{'='*60}{Style.RESET_ALL}")
            print(f"{Back.GREEN}{Fore.BLACK}  ✅ CONNECTED TO MESSAGE SERVER!  {Style.RESET_ALL}")
            print(f"{Back.GREEN}{Fore.BLACK}{'='*60}{Style.RESET_ALL}\n")
            print(f"{Fore.YELLOW}👤 Username: {Fore.WHITE}{self.username}{Style.RESET_ALL}")
            print(f"{Fore.GREEN}💬 You can now start chatting!")
            print(f"{Fore.CYAN}📤 Type your message and press Enter to send")
            print(f"{Fore.RED}🚪 Type 'quit' or 'exit' to disconnect\n")
            print(f"{Fore.GREEN}{'='*60}{Style.RESET_ALL}\n")
            
        except socket.timeout:
            raise NetworkError(f"Connection timed out. Is the server running on {self.host}:{self.port}?")
        except ConnectionRefusedError:
            raise NetworkError(f"Connection refused. Make sure the server is running on {self.host}:{self.port}")
        except socket.gaierror:
            raise NetworkError(f"Invalid host address: {self.host}")
        except Exception as e:
            raise NetworkError(f"Failed to connect to server: {e}")
            
    def send_messages(self) -> None:
        """
        Continuously read user input and send messages to the server.
        Runs in a separate thread.
        """
        while self.connected:
            try:
                # Read user input
                message = input()
                
                # Check for quit commands
                if message.lower() in ['quit', 'exit']:
                    print(f"{Fore.YELLOW}👋 Disconnecting...{Style.RESET_ALL}")
                    self.disconnect()
                    break
                
                # Don't send empty messages
                if not message.strip():
                    continue
                
                # Format message with username
                formatted_message = f"[{self.username}]: {message}"
                
                # Send to server
                self.client_socket.send(formatted_message.encode('utf-8'))
                
                # Display your own message in a different color
                print(f"{Fore.LIGHTBLUE_EX}📤 You: {message}{Style.RESET_ALL}")
                
            except OSError:
                # Socket was closed
                if not self.connected:
                    break
            except Exception as e:
                if self.connected:
                    print(f"{Fore.RED}❌ Error sending message: {e}{Style.RESET_ALL}")
                    self.disconnect()
                break
                
    def receive_messages(self) -> None:
        """
        Continuously receive messages from the server and display them.
        Runs in a separate thread.
        """
        while self.connected:
            try:
                # Receive message from server (up to 1024 bytes)
                message = self.client_socket.recv(1024).decode('utf-8')
                
                if not message:
                    # Empty message means server closed connection
                    print(f"\n{Fore.RED}❌ Server closed the connection{Style.RESET_ALL}")
                    self.disconnect()
                    break
                
                # Display received message with color
                # Check if it's a welcome message vs. a user message
                if message.startswith("Welcome"):
                    print(f"{Fore.GREEN}🎉 {message}{Style.RESET_ALL}")
                else:
                    # Parse and display user messages
                    print(f"{Fore.LIGHTGREEN_EX}📨 {message.strip()}{Style.RESET_ALL}")
                
            except OSError:
                # Socket was closed
                if not self.connected:
                    break
            except Exception as e:
                if self.connected:
                    print(f"{Fore.RED}❌ Error receiving message: {e}{Style.RESET_ALL}")
                    self.disconnect()
                break
                
    def start(self) -> None:
        """
        Start the client by connecting and launching send/receive threads.
        """
        try:
            # Connect to server
            self.connect()
            
            # Create thread for receiving messages
            receive_thread = threading.Thread(target=self.receive_messages, daemon=True)
            receive_thread.start()
            
            # Use main thread for sending messages (so input() works properly)
            self.send_messages()
            
        except NetworkError as e:
            print(f"{Fore.RED}❌ Network Error: {e}{Style.RESET_ALL}")
        except KeyboardInterrupt:
            print(f"\n{Fore.YELLOW}⚠️  Keyboard interrupt received{Style.RESET_ALL}")
            self.disconnect()
        except Exception as e:
            print(f"{Fore.RED}❌ Unexpected error: {e}{Style.RESET_ALL}")
            self.disconnect()
            
    def disconnect(self) -> None:
        """
        Gracefully disconnect from the server.
        """
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
    """
    Main function to create and start the client.
    Prompts user for their username before connecting.
    """
    # Display colorful banner
    print(f"\n{Back.CYAN}{Fore.BLACK}{'='*60}{Style.RESET_ALL}")
    print(f"{Back.CYAN}{Fore.BLACK}  🌟 WELCOME TO THE MESSAGE CLIENT! 🌟  {Style.RESET_ALL}")
    print(f"{Back.CYAN}{Fore.BLACK}{'='*60}{Style.RESET_ALL}\n")

    print(f"\n{Fore.CYAN}{'='*35}{Style.RESET_ALL}")
    my_ip = get_local_ip()
    print(f"{Fore.CYAN}Your IP address is: {my_ip} {Style.RESET_ALL}")
    print(f"{Fore.CYAN}{'='*35}{Style.RESET_ALL}")

    # Get username from user
    username = input(f"{Fore.YELLOW}Enter your username: {Style.RESET_ALL}").strip()
    
    # Use default if empty
    if not username:
        username = "Anonymous"
    
    print(f"{Fore.GREEN}✅ Username set to: {username}{Style.RESET_ALL}\n")
    
    # Get the IP address to connect to
    msg = f"\n{Fore.YELLOW}🙂 Enter the IP address of the server to connect to (default: {Fore.CYAN}{my_ip}{Fore.YELLOW}): {Style.RESET_ALL}"

    # ask user whether to use localhost or their local IP address
    myNew_ip = input(msg).strip() or my_ip
    if myNew_ip.lower() != my_ip.lower():
        my_ip = myNew_ip
    print(f"{Fore.GREEN}✅ Using IP address: {my_ip}{Style.RESET_ALL}")

    msg = f"{Fore.GREEN}✅ Connecting to server at {my_ip}:{5555}...{Style.RESET_ALL}"
    print(msg)

    # Create and start client
    # client = MessageClient(host='localhost', port=5555, username=username)
    client = MessageClient(host=my_ip, port=5555, username=username)
    client.start()


if __name__ == "__main__":
    main()

```

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 4: Understanding the Code

Let's break down the key features:

### **Classes and Attributes**

```python
class MessageClient:
    def __init__(self, host, port, username):
        self.client_socket = None
        self.connected = False
```

- Encapsulates client functionality
- Tracks connection state
- Stores username for message formatting

### **Custom Exception**

```python
class NetworkError(Exception):
    """Custom exception for network-related errors"""
    pass
```

- Specific exception type for network issues
- Makes error handling more precise

### **Connection with Timeout**

```python
self.client_socket.settimeout(10)
self.client_socket.connect((self.host, self.port))
self.client_socket.settimeout(None)
```

- Sets 10-second timeout for connection attempt
- Prevents indefinite hanging if server is down
- Removes timeout after successful connection

### **Exception Handling for Different Errors**

```python
except socket.timeout:
    raise NetworkError("Connection timed out")
except ConnectionRefusedError:
    raise NetworkError("Connection refused")
except socket.gaierror:
    raise NetworkError("Invalid host address")
```

- Catches specific socket errors
- Provides helpful error messages
- Converts to custom `NetworkError` exception

### **Threading for Simultaneous Send/Receive**

```python
receive_thread = threading.Thread(target=self.receive_messages, daemon=True)
receive_thread.start()
self.send_messages()  # Main thread handles sending
```

- Receive thread runs in background (daemon)
- Main thread handles user input (required for `input()`)
- Both operations happen simultaneously

### **Colorful Output with Multiple Styles**

```python
print(f"{Back.GREEN}{Fore.BLACK}  CONNECTED  {Style.RESET_ALL}")
print(f"{Fore.LIGHTBLUE_EX}📤 You: {message}{Style.RESET_ALL}")
print(f"{Fore.LIGHTGREEN_EX}📨 {message}{Style.RESET_ALL}")
```

- `Back.GREEN`: Colored background
- `Fore.LIGHTBLUE_EX`: Light blue text color
- Different colors distinguish your messages from received messages

### **Graceful Disconnection**

```python
if message.lower() in ['quit', 'exit']:
    self.disconnect()
```

- User can type 'quit' or 'exit' to disconnect
- `disconnect()` method closes socket cleanly
- Connection flag prevents further operations

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 5: Run the Client

**Important**: Make sure your server from Tutorial 1 is running first!

In a **new terminal window**, navigate to the tutorials directory and start the client:

```bash
cd tutorials/
uv run tutorial_02/src/message_client.py
```

### Understanding IP Address Selection

When you start the client, it will display your local IP address and prompt you for configuration:

```
===================================
Your IP address is: 192.168.1.100
===================================

Enter your username: Alice
✅ Username set to: Alice

🙂 Enter the IP address of the server to connect to (default: 192.168.1.100):
```

**How it works:**

1. **Automatic IP Detection**: The `get_local_ip()` function automatically detects your computer's IP address on the local network

2. **Three Connection Options**:
   - **Press Enter (use detected IP)**: Connect to a server running on your local network using your IP
   - **Type anything and press Enter (use localhost)**: Connect to `localhost` (127.0.0.1) for local testing
   - **Type a specific IP address**: Connect to a server running on a classmate's computer (e.g., `192.168.1.50`)

3. **When to use each option**:
   - **Use localhost** (type anything): Connecting to a server running on your own computer
   - **Use your IP** (press Enter): Connecting to a server on the network using your detected IP
   - **Use a specific IP**: Connecting to a classmate's server (ask them for their IP address)

### Connection Example Scenarios

**Scenario 1: Testing on your own computer**
- Server running on your machine with localhost
- Client: Type anything when prompted (uses localhost)
- Result: Client connects to local server

**Scenario 2: Connecting to a classmate's server**
- Classmate's server running at `192.168.1.50`
- Client: Type `192.168.1.50` when prompted
- Result: Client connects to classmate's server

**Scenario 3: Connecting to your own server on the network**
- Your server running at your IP `192.168.1.100`
- Client: Press Enter (uses detected IP)
- Result: Client connects to your server via network

You'll be prompted for a username:

```
============================================================
  🌟 WELCOME TO THE MESSAGE CLIENT! 🌟
============================================================

Enter your username: Alice
✅ Username set to: Alice
```

After entering your username and server IP, you should see:

```
============================================================
  ✅ CONNECTED TO MESSAGE SERVER!
============================================================

👤 Username: Alice
💬 You can now start chatting!
📤 Type your message and press Enter to send
🚪 Type 'quit' or 'exit' to disconnect

============================================================
```

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 6: Test the Client

1. **Type a message** and press Enter
2. You should see your message displayed in blue
3. The server terminal should show it received your message
4. Type `quit` to disconnect

Your terminal output will look like:

```
📤 You: Hello from the client!
```

The server will display:

```
📨 From ('127.0.0.1', 54321): [Alice]: Hello from the client!
```

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 7: Determining Your IP Address

If you would like to call someone on their phone, it would help to know their phone number. Similarity, if you would like to send someone a text message to their IP (Internet Protocol number), then you will need to know this number as well. There are several way to determine the current IP number of a machine but the most simple way would be to have Python find and print it to the screen. Please create the following program to determine your IP number.

## Why Do We Need IP Addresses?
To connect computers, we need to know their **network addresses** (IP addresses).

Just like you need a phone number to call someone, clients need the server's IP address to connect!


Create an empty source code file called `determine_my_ip.py` in the `tutorials/` directory. Place the following code into the file and then save it.

```python
import socket

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

# Use it:
my_ip = get_local_ip()
print(f"Your IP address is: {my_ip}")
```

To run this file, use the command below.

```bash
uv run determine_my_ip.py`
```

You will see the output look something like the following.

```text
Your IP address is: 192.168.40.121
```

![--- --- --- --- --- --- --- --- ---](../../graphics/div_bar.png)

## Part 8: Test with Multiple Clients

Open **another terminal window** and start a second client:

```bash
cd tutorials/
uv run tutorial_02/src/message_client.py
```

Enter a different username (e.g., "Bob"). Now you can:

- Send messages from Alice's client
- Bob's client will receive them!
- Send messages from Bob's client
- Alice's client will receive them!

**Congratulations!** You now have:

- ✅ A working client with classes
- ✅ Exception handling for network errors
- ✅ Colorful, user-friendly interface
- ✅ Threading for simultaneous send/receive
- ✅ Multiple clients communicating through the server

## Next Steps

In **Tutorial 3**, you'll use your messaging app to play Rock, Paper, Scissors with your classmates!

## Troubleshooting

**"Connection refused" error?**

- Make sure the server is running first
- Check that the host and port match the server settings

**Messages not appearing?**

- Verify both clients are connected to the same server
- Check the server terminal for error messages

**Can't type messages?**

- Make sure the client connected successfully
- Try restarting the client

**Colorama not working?**

- Make sure you ran `uv add colorama` in the tutorials/ directory
- Run commands from the tutorials/ directory using `uv run`
