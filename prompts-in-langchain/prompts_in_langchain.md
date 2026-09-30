# Prompts in LangChain

------------------------------------------------------------------------

## 1. What is a Prompt?

A **prompt** is the message or instruction that is sent to an LLM.

The output of an LLM is highly dependent on the prompt given to it.

``` text
User
  │
  │ Prompt
  ▼
┌─────────────┐
│     LLM     │
└─────────────┘
  │
  │ Response
  ▼
Output
```

The prompt is therefore an important part of an LLM application.

------------------------------------------------------------------------

# 2. Static Prompt vs Dynamic Prompt

## Static Prompt

A static prompt is a prompt whose text is fixed.

``` text
Application
    │
    │ fixed prompt
    ▼
   LLM
    │
    ▼
 Response
```

Example idea:

``` python
prompt = "Write a summary of this paper."
```

The prompt does not change according to structured user inputs.

### Problems with a static prompt in an application

If the user has to manually write the complete prompt:

-   users may write different instructions
-   users may make typing mistakes
-   users may provide instructions that the application does not expect
-   the application has less control over the prompt structure

------------------------------------------------------------------------

## Dynamic Prompt

A dynamic prompt is generated using a **template + variables**.

``` text
User Inputs
   │
   ├── paper
   ├── style
   └── length
          │
          ▼
   ┌──────────────┐
   │ Prompt       │
   │ Template     │
   └──────────────┘
          │
          ▼
   Final Prompt
          │
          ▼
         LLM
          │
          ▼
       Response
```

Example:

``` text
Summarize {paper} in {style} style
in approximately {length} words.
```

At runtime:

``` text
paper  = "AI research paper"
style  = "simple"
length = "200"
```

The template becomes a complete prompt.

------------------------------------------------------------------------

# 3. Why Dynamic Prompts?

The main idea is to separate:

``` text
Prompt Structure
      +
User/Application Data
      =
Final Prompt
```

This makes the prompt reusable.

For example, an application can provide fixed choices through a UI:

``` text
Style:
[ Simple ▼ ]

Length:
[ 200 words ▼ ]

Paper:
[ User input ]
```

The application then constructs the prompt instead of asking the user to
manually write the complete instruction.

------------------------------------------------------------------------

# 4. PromptTemplate

LangChain provides **PromptTemplate** for creating dynamic prompts.

A PromptTemplate contains:

1.  Template text
2.  Variables/placeholders

Example:

``` python
from langchain_core.prompts import PromptTemplate

template = PromptTemplate(
    template="Explain {topic} in {length} sentences.",
    input_variables=["topic", "length"]
)
```

The placeholders are:

``` text
{topic}
{length}
```

They are filled at runtime.

------------------------------------------------------------------------

## PromptTemplate Flow

``` text
                 PromptTemplate
                      │
          ┌───────────┴───────────┐
          │                       │
     Template text            Variables
          │                       │
          │                 topic = "RAG"
          │                 length = "5"
          └───────────┬───────────┘
                      ▼
                Final Prompt
                      │
                      ▼
                     LLM
```

------------------------------------------------------------------------

## Formatting a PromptTemplate

``` python
prompt = template.format(
    topic="RAG",
    length="5"
)
```

Result:

``` text
Explain RAG in 5 sentences.
```

The formatted prompt can then be passed to the model.

------------------------------------------------------------------------

# 5. PromptTemplate vs f-string

A normal Python f-string can also construct dynamic text:

``` python
topic = "RAG"
length = "5"

prompt = f"Explain {topic} in {length} sentences."
```

But the video demonstrates why LangChain's `PromptTemplate` is useful.

### PromptTemplate gives

-   validation
-   reusability
-   integration with LangChain
-   ability to save the prompt template
-   cleaner separation between prompt design and application logic

------------------------------------------------------------------------

# 6. Validation

PromptTemplate knows which variables are required.

For example:

``` python
PromptTemplate(
    template="Explain {topic} in {length} sentences.",
    input_variables=["topic", "length"]
)
```

The template expects:

``` text
topic
length
```

This helps identify missing or incorrect variables rather than silently
constructing an incorrect prompt.

------------------------------------------------------------------------

# 7. Reusable Prompt Templates

A prompt should not have to be recreated every time it is used.

The video demonstrates saving a prompt template as a JSON file.

``` text
prompts.py
    │
    │ save()
    ▼
saved_template.json
    │
    │ load_prompt()
    ▼
main.py
```

The prompt design can therefore be separated from the main application.

------------------------------------------------------------------------

## Save a Prompt

Conceptually:

``` python
prompt.save("saved_template.json")
```

Then load it:

``` python
from langchain_core.prompts import load_prompt

prompt = load_prompt("saved_template.json")
```

The same prompt template can then be reused.

------------------------------------------------------------------------

# 8. Messages

When working with **chat models**, the input is not simply one string.

A conversation consists of messages.

The important message roles demonstrated in the video are:

``` text
System
Human
AI
```

------------------------------------------------------------------------

## System Message

The **system message** contains instructions for the model.

Example idea:

``` text
You are a helpful assistant.
Answer clearly and concisely.
```

It defines the behavior/instructions given to the model.

------------------------------------------------------------------------

## Human Message

The **human message** represents the user's input.

``` text
Explain RAG.
```

------------------------------------------------------------------------

## AI Message

The **AI message** represents a previous response from the model.

``` text
RAG stands for Retrieval Augmented Generation...
```

------------------------------------------------------------------------

# 9. Conversation Structure

A conversation can therefore look like:

``` text
┌─────────────────────────────────────┐
│ System                              │
│ "You are a helpful assistant."      │
└─────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│ Human                               │
│ "What is LangChain?"                │
└─────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│ AI                                  │
│ "LangChain is..."                   │
└─────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│ Human                               │
│ "Why is it useful?"                 │
└─────────────────────────────────────┘
```

The roles tell the model who said what.

------------------------------------------------------------------------

# 10. ChatPromptTemplate

For chat models, LangChain provides:

``` python
ChatPromptTemplate
```

It is used to create a structured sequence of messages.

Basic structure:

``` text
ChatPromptTemplate
       │
       ├── System message
       ├── Human message
       └── AI message
```

------------------------------------------------------------------------

## Creating a ChatPromptTemplate

Example structure:

``` python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{question}")
])
```

Here:

``` text
system  → fixed instruction
human   → dynamic user input
```

------------------------------------------------------------------------

# 11. ChatPromptTemplate Flow

``` text
                    ChatPromptTemplate
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
        System           Human            AI
        message          message         message
            │              │              │
            └──────────────┼──────────────┘
                           ▼
                    Formatted messages
                           │
                           ▼
                      Chat Model
                           │
                           ▼
                        Response
```

------------------------------------------------------------------------

# 12. PromptTemplate vs ChatPromptTemplate

## PromptTemplate

Used when the prompt is primarily a **single text prompt**.

``` text
PromptTemplate
      │
      ▼
  String prompt
      │
      ▼
     LLM
```

## ChatPromptTemplate

Used when the input consists of **multiple chat messages with roles**.

``` text
ChatPromptTemplate
      │
      ▼
System + Human + AI messages
      │
      ▼
  Chat Model
```

------------------------------------------------------------------------

# 13. Chat History

A chatbot needs to remember previous messages.

Example conversation:

``` text
Human: My name is Rahul.
AI: Nice to meet you, Rahul.

Human: What is my name?
```

For the model to answer correctly, the previous conversation must be
supplied as context.

``` text
Current request
      +
Previous messages
      │
      ▼
ChatPromptTemplate
      │
      ▼
Chat Model
```

------------------------------------------------------------------------

# 14. MessagesPlaceholder

LangChain provides:

``` python
MessagesPlaceholder
```

It is used to insert a list of messages into a `ChatPromptTemplate`.

The important idea:

``` text
MessagesPlaceholder
        │
        ▼
Insert message history here
```

Instead of manually writing every previous message into the template,
the history can be supplied dynamically.

------------------------------------------------------------------------

## MessagesPlaceholder Flow

``` text
                 Chat History
                      │
                      │ list of messages
                      ▼
             ┌──────────────────┐
             │ Messages          │
             │ Placeholder       │
             └──────────────────┘
                      │
                      ▼
             ChatPromptTemplate
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
     System        History        Human
     message       messages       message
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                  Chat Model
```

------------------------------------------------------------------------

# 15. Why MessagesPlaceholder?

Chat history may come from somewhere outside the prompt itself.

For example:

``` text
Database
   │
   ▼
Chat History
   │
   ▼
MessagesPlaceholder
   │
   ▼
ChatPromptTemplate
   │
   ▼
LLM
```

This allows the application to dynamically insert the previous
conversation.

------------------------------------------------------------------------

# 16. Chatbot Architecture from the Video

A basic chatbot can be understood as:

``` text
                 ┌──────────────┐
                 │ Chat History │
                 └──────┬───────┘
                        │
                        ▼
              MessagesPlaceholder
                        │
                        ▼
┌─────────────┐   ┌───────────────┐
│ System      │──▶│               │
│ Message     │   │ ChatPrompt    │
└─────────────┘   │ Template      │
                  │               │
┌─────────────┐   │               │
│ Human       │──▶│               │
│ Message     │   └───────┬───────┘
└─────────────┘           │
                          ▼
                     Chat Model
                          │
                          ▼
                       AI Reply
                          │
                          ▼
                    Update History
```

------------------------------------------------------------------------

# 17. Using Prompts with Chains

The video also shows that the prompt and model can be combined.

Instead of manually doing:

``` text
Create prompt
      ↓
Format prompt
      ↓
Send prompt to model
      ↓
Get response
```

they can be connected into a chain.

``` text
PromptTemplate
      │
      ▼
    Model
      │
      ▼
   Response
```

Conceptually:

``` python
chain = prompt | model
```

Then:

``` python
response = chain.invoke(...)
```

The chain handles the flow from prompt to model.

------------------------------------------------------------------------

# 18. Prompt → Model Flow

Without a chain:

``` text
Input
  │
  ▼
Prompt Template
  │
  ▼
Formatted Prompt
  │
  ▼
Model
  │
  ▼
Response
```

With a chain:

``` text
Input
  │
  ▼
┌────────────────────────────┐
│ PromptTemplate → Model     │
└────────────────────────────┘
  │
  ▼
Response
```

------------------------------------------------------------------------

# 19. Prompt Design in an Application

The video demonstrates the idea of taking user inputs from an
application UI and putting them into a prompt template.

General flow:

``` text
                 Application UI
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Input 1       Input 2      Input 3
          │            │            │
          └────────────┼────────────┘
                       ▼
                PromptTemplate
                       │
                       ▼
                 Final Prompt
                       │
                       ▼
                      LLM
                       │
                       ▼
                    Output
```

The application controls the structure while the user supplies the
variable values.

------------------------------------------------------------------------

# 20. Static → Dynamic → Chat

The progression covered in the video can be remembered as:

``` text
STATIC PROMPT
     │
     │ Need variables
     ▼
PROMPT TEMPLATE
     │
     │ Need conversations / roles
     ▼
CHAT PROMPT TEMPLATE
     │
     │ Need previous messages
     ▼
MESSAGES PLACEHOLDER
     │
     │ Combine with model
     ▼
CHAIN
```

------------------------------------------------------------------------

# 21. Core LangChain Prompt Components

``` text
                    PROMPTS
                       │
        ┌──────────────┼───────────────┐
        │              │               │
        ▼              ▼               ▼
 PromptTemplate  ChatPromptTemplate  MessagesPlaceholder
        │              │               │
        │              │               │
        ▼              ▼               ▼
  Dynamic text     Chat messages    Dynamic history
        │              │               │
        └──────────────┼───────────────┘
                       ▼
                     Model
```

------------------------------------------------------------------------

# 22. Important Code Patterns

## PromptTemplate

``` python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate(
    template="Explain {topic} in {length} sentences.",
    input_variables=["topic", "length"]
)

formatted_prompt = prompt.format(
    topic="LangChain",
    length="5"
)
```

------------------------------------------------------------------------

## ChatPromptTemplate

``` python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{question}")
])

messages = prompt.invoke({
    "question": "What is LangChain?"
})
```

------------------------------------------------------------------------

## MessagesPlaceholder

``` python
from langchain_core.prompts import (
    ChatPromptTemplate,
    MessagesPlaceholder
)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder("history"),
    ("human", "{question}")
])
```

The `history` variable represents the previous messages.

------------------------------------------------------------------------

## Prompt + Model

``` python
chain = prompt | model

response = chain.invoke({
    "question": "What is LangChain?"
})
```

------------------------------------------------------------------------

# 23. Prompt Template Reusability

The video emphasizes that prompt templates can be separated from the
main application.

``` text
                prompts.py
                    │
                    ▼
             Prompt definition
                    │
                    ▼
             Save as JSON
                    │
                    ▼
          saved_template.json
                    │
             ┌──────┴──────┐
             ▼             ▼
          app.py        another_app.py
             │             │
             └──────┬──────┘
                    ▼
              Same prompt
```

This avoids repeatedly writing the same prompt design.

------------------------------------------------------------------------

# 24. Complete Mental Model

``` text
                    USER / APPLICATION
                            │
                            ▼
                    Input Variables
                            │
                            ▼
                   ┌─────────────────┐
                   │ Prompt Template │
                   └────────┬────────┘
                            │
                            ▼
                     Formatted Prompt
                            │
                            ▼
                         MODEL
                            │
                            ▼
                         OUTPUT


For chat applications:

                    USER / APPLICATION
                            │
                            ▼
                    Current Message
                            │
                            ├───────────────┐
                            │               │
                            ▼               ▼
                    ChatPromptTemplate  Chat History
                            │               │
                            │        MessagesPlaceholder
                            │               │
                            └───────┬───────┘
                                    ▼
                              Chat Messages
                                    │
                                    ▼
                                Chat Model
                                    │
                                    ▼
                                  Reply
```

------------------------------------------------------------------------

# 25. Revision Sheet

## Prompt

A message/instruction sent to an LLM.

## Static Prompt

Fixed prompt text.

## Dynamic Prompt

Prompt generated using variable inputs.

## PromptTemplate

LangChain component for creating reusable dynamic text prompts.

## ChatPromptTemplate

LangChain component for creating structured prompts made of chat
messages.

## System Message

Provides instructions/behavior for the model.

## Human Message

Represents user input.

## AI Message

Represents a previous model response.

## MessagesPlaceholder

Dynamically inserts a list of messages, such as chat history, into a
chat prompt.

## Chain

Combines components such as a prompt and model into a connected flow.

------------------------------------------------------------------------

# 26. One-Page Memory Map

``` text
                         PROMPTS
                            │
             ┌──────────────┴──────────────┐
             │                             │
          STATIC                        DYNAMIC
             │                             │
        fixed text                 template + variables
                                           │
                                           ▼
                                   PromptTemplate
                                           │
                                           ▼
                                     formatted text
                                           │
                                           ▼
                                          LLM


                         CHAT APPLICATIONS
                                  │
                                  ▼
                         ChatPromptTemplate
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                 System        Human           AI
                 message       message        message
                    │             │             │
                    └─────────────┼─────────────┘
                                  │
                                  ▼
                       MessagesPlaceholder
                                  │
                                  ▼
                            Chat History
                                  │
                                  ▼
                             Chat Model


                         APPLICATION FLOW
                                  │
                                  ▼
                            Prompt / ChatPrompt
                                  │
                                  ▼
                                Model
                                  │
                                  ▼
                               Output

                         OR

                         Prompt | Model
                              │
                              ▼
                            Chain
                              │
                              ▼
                           Response
```

------------------------------------------------------------------------

# 27. Final Takeaway from the Video

The video builds the concept step by step:

``` text
Prompt
  ↓
Static Prompt
  ↓
Dynamic Prompt
  ↓
PromptTemplate
  ↓
Messages
  ↓
ChatPromptTemplate
  ↓
MessagesPlaceholder
  ↓
Prompt + Model
  ↓
Chain
```

The central idea is to move from manually written prompts toward
**structured, reusable, dynamic prompts**, and then extend that
structure to **chat messages and conversation history** when building
chat-based applications.
