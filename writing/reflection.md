# Activity 08 Reflection

Reflection Questions

Name: Add Your Name Here
Date: TODO

## Instructions

Please complete the following questions based on your experience completing the activity. Please provide concise and specific answers while using clear and meaningful language. Use fenced code blocks for any code or command outputs.

=================

### Q1. Server Connection Output

What is the output from running the command `uv run tutorial_01/src/message_server.py` from the tutorials/ directory? Paste your output below.

Response:

```text
TODO
```

### Q2. Exception Handling

The client code includes several different exception types (`socket.timeout`, `ConnectionRefusedError`, `socket.gaierror`). Explain what each of these exceptions represents and why catching specific exceptions is better than using a general `except Exception` for everything.

Response:

TODO

### Q3. Scalability and Improvements

The current server uses one thread per client. If 100 clients connected simultaneously, the server would create 100 threads.

What potential problems might arise with this approach?

Response:

TODO

### Q4. Security Considerations

The current implementation sends all messages as plain text over the network. List at least three security concerns or vulnerabilities with this approach. For each concern, suggest a potential solution or improvement.

Response:

TODO

### Q5. Time permitting, try running some experiments with the text messaging app and write about your work here.

Response:

TODO 

---

(Did you remember to add your name to the top of this document?)