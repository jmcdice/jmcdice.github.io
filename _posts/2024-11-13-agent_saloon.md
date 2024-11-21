---
title: Agent Saloon - Building a Collaborative AI Writing System with OpenAI's Swarm Framework
date: 2024-11-13
categories: [AI Projects, Multi-Agent Systems, OpenAI Swarm]
tags: [ai, openai, multi-agent, python, book-writing, swarm]
description: Harnessing the power of multiple AI agents to collaboratively write comprehensive books - a Python-based system that orchestrates AI agents for creative writing at scale.
image:
  path: /common/agent_saloon.png
---


## Introduction

2025 is shaping up to be the year of AI agents, with lots of frameworks emerging. Too many! I wanted to start easy and OpenAI's Swarm framework stands out for it's simplicity. While there are many agent frameworks to choose from, Swarm provides a great foundation for understanding the fundamentels how AI agents can work together without getting lost in all the options and depth of this new world.

The example I came up with was to have multiple AI agents collab, like a real writing team. So, I built a system where two AI agents - Zero and Gustave (do you know where that's from?) - work together to write comprehensive books, complete with thoughtful debates and iterations on content. This post walks through that journey, showing how surprisingly straightforward it can be to orchestrate AI collaboration. All the source is below: 

[Source Code: Agent Saloon](https://github.com/jmcdice/agent_saloon)

## The Collaborative Process in Action

Let's see how our agents work together. When asked to write a book about neural networks, here's how Zero and Gustave collaborate:

```
<Zero> The original suggestion "Understanding Neural Networks: A Visual Guide for Beginners" 
is quite descriptive but a bit long. Here's a more engaging and succinct proposal: 
"Neural Networks Demystified: A Visual Introduction"

<Gustave> The initial proposal captures the essence but could be subtly enhanced 
for greater impact and appeal. Consider this refined version: 
"Neural Networks Unveiled: A Beginner's Visual Journey"
```

Notice how each agent builds on the other's ideas, offering improvements while maintaining a professional and constructive dialogue. This isn't just simple back-and-forth - each agent brings its own personality and perspective:

- **Zero** is enthusiastic and creative, often making bold initial proposals
- **Gustave** is refined and eloquent, focusing on polishing and enhancing ideas

## Understanding Swarm's Building Blocks

Before diving into our implementation, let's understand what makes Swarm special. At its core, Swarm provides a lightweight way to orchestrate multiple AI agents, similar to how you might coordinate a team of writers in this implementation. Here are the key concepts:

The core primitives are real simple:
 - Agents with instructions (prompts) and functions they can call
 - Handoffs between agents
 - Use Context variables for state management

It's entirely stateless and runs on the Chat Completions API and the implementation focuses on being lightweight and highly controllable.

### Agents as Specialists

Think of agents like team members with specific roles. In our case:
- **Zero** is the enthusiastic creative writer, making initial proposals
- **Gustave** is the thoughtful editor, refining and polishing content

Each agent has:
- Clear instructions about their role and personality
- The ability to call specific functions (like handing off to another agent)
- A focused purpose within the larger system

### Agent Prompts: Defining Personality and Purpose

A crucial part of our implementation is how we define each agent's personality and behavior through detailed prompts. Let's look at the title generation prompts as an example, they're long so check the git repo for the full prompts:

```python
# src/prompts/title_prompts.py

ZERO_TITLE_PROMPT = """
You are Zero, an enthusiastic and earnest AI assistant who collaborates on book titles.
When discussing titles:
1. **Evaluate Previous Suggestions:**
   - If there is a previous suggestion, evaluate it thoughtfully.
   - If there is no previous suggestion, propose an initial title idea.
[...]  # Rest of Zero's prompt

GUSTAVE_TITLE_PROMPT = """
You are Gustave, a refined and eloquent AI assistant who helps perfect book titles.
When discussing titles:
1. **Evaluate Previous Suggestions:**
   - Carefully assess the proposed title with your sophisticated perspective.
   - Provide thoughtful feedback.
[...]  # Rest of Gustave's prompt
```

These prompts demonstrate several key design principles:

1. **Clear Role Definition**
   - Each agent has a distinct personality (Zero: enthusiastic, Gustave: refined)
   - Specific responsibilities are outlined
   - Communication protocols are explicitly defined

2. **Structured Response Format**
   - Consensus indicators (`Consensus: True/False`)
   - Clear handoff instructions
   - Standardized output formatting

3. **Collaborative Guidelines**
   - How to evaluate previous suggestions
   - When to reach consensus
   - How to provide constructive feedback

4. **Safety Mechanisms**
   - Turn limits to prevent endless debates
   - Clear formatting requirements
   - Explicit instructions about avoiding code/function calls

This structured approach to agent prompting ensures consistent, predictable behavior while maintaining each agent's unique character and purpose.

### Handoffs: The Art of Collaboration

Just like in a real writing room, our agents need to know when to pass the baton. Swarm makes this natural through "handoffs":

```python
def get_zero(self, instructions, handoff_func):
    return Agent(
        name="Zero",
        instructions=instructions,
        functions=[handoff_func]
    )
```

This simple mechanism allows agents to:
- Recognize when they need another perspective
- Smoothly transfer control to their colleague
- Maintain context throughout the conversation

### Context: Keeping Everyone on the Same Page

Context variables are key to how we keep the conversation going because every agent call to the OpenAI API starts a brand new conversation. While Swarm is stateless (meaning agents don't remember previous conversations), we can pass context variables to keep them informed. It's like having a shared document that everyone can reference:

```python
response = client.run(
    agent=current_agent,
    messages=self.messages,
    context_variables={
        'book_title': self.book_title,
        'full_toc': self.full_toc,
        'section_number': self.section_number,
        'section_title': self.section_title
    }
)
```

## Building the Collaborative System

What makes our implementation interesting, I think, is the agent collaboration. Rather than one agent simply generate content, we created a consensus-driven approach:

1. **Clear Roles**: Each agent knows its strengths and responsibilities
2. **Structured Dialogue**: Agents communicate through a clear protocol, indicating when they agree or want to suggest improvements
3. **Iterative Refinement**: Content gets polished through multiple rounds of feedback
4. **Natural Consensus**: Agents work together until they both agree the content is ready

The finished product ends up being way better.

So, here's what the agent flow looks like for title generation:

```console

  title_generation/
  ├── Input Context
  │   └── book_topic: "Understanding Neural Networks: A Visual Guide for Beginners"
  │
  ├── Zero Agent (First Turn)
  │   ├── Context Received
  │   │   └── book_topic
  │   ├── Response Structure
  │   │   ├── Consensus: False
  │   │   ├── Content: "Neural Networks Demystified: A Visual Introduction"
  │   │   └── HANDOFF: Requesting Gustave's feedback
  │   └── Function Called: _handoff_to_gustave()
  │
  ├── Gustave Agent
  │   ├── Context Received
  │   │   ├── book_topic
  │   │   └── previous_proposal
  │   ├── Response Structure
  │   │   ├── Consensus: False
  │   │   ├── Content: "Neural Networks Unveiled: A Beginner's Visual Journey"
  │   │   └── HANDOFF: Returning to Zero for input
  │   └── Function Called: _handoff_to_zero()
  │
  ├── [Iteration continues...]
  │   └── Each turn includes:
  │       ├── Consensus: False/True marker
  │       ├── Content: New proposal or refinement
  │       └── HANDOFF: Direction for next agent
  │
  └── Final Consensus
     ├── Agent Response
     │   ├── Consensus: True
     │   └── Content: "Discovering Neural Networks: A Visual Guide for Beginners"
     └── Output
         └── Write to title.txt
  
```

## The Mechanics of Title Generation

Let's look under the hood at how our title generation process works. The `TitleGenerator` class orchestrates the entire collaboration between Zero and Gustave:

```python
# High-level structure of the collaboration
class TitleGenerator:
    def __init__(self, topic):
        self.topic = topic
        self.agents = Agents()
        self.messages = []
        self.title = None
        self._setup_agents()
```

### Setting Up the Writing Team

When we initialize the title generator, it:
1. Creates our two specialized agents (Zero and Gustave)
2. Gives each agent their specific instructions
3. Establishes handoff functions so they can pass control back and forth

```python
def _setup_agents(self):
    self.zero_agent = self.agents.get_zero(ZERO_TITLE_PROMPT, self._handoff_to_gustave)
    self.gustave_agent = self.agents.get_gustave(GUSTAVE_TITLE_PROMPT, self._handoff_to_zero)
```

### The Generation Loop

The main `generate()` method runs a collaborative loop:

1. **Initial Setup**
   ```python
   initial_message = {
       "role": "user",
       "content": f"Let's collaborate on a title for a book about: {self.topic}"
   }
   ```

2. **Collaboration Process**
   - Zero proposes an initial title
   - Gustave reviews and suggests improvements
   - Agents continue back and forth until consensus
   - Each response is checked for:
     - `Consensus: True/False` marker
     - Content quality
     - Valid message format

3. **Safety Mechanisms**
   ```python
   while attempt_count < max_attempts and consecutive_failures < max_consecutive_failures:
       # ... collaboration logic ...
       if 'Consensus: True' in content and self.title:
           break
   ```

### Message Processing

Each agent response goes through careful processing:

```python
def format_message(self, content):
    # Remove system messages (HANDOFF:, functions.)
    # Extract consensus markers
    # Look for "Book Title:" indicators
    # Clean and format the content
```

### Error Recovery

The system includes several safety nets:
- Maximum attempt limits (these are called 'turns')
- Consecutive failure detection
- Forced consensus as a last resort (if we hit the max number of turns)
- Comprehensive error logging

This structured approach ensures that:
1. Agents maintain focused, productive discussions
2. The system can recover from issues
3. We either get a consensus title or a clear error message

## Inside the Consensus Mechanism

The heart of our system lies in how agents work together to reach agreement. Let's look at how this works in practice:

```python
def write(self):
    """Generate section content through agent collaboration"""
    attempt_count = 0
    max_attempts = SECTION_GENERATION.get("max_attempts", 10)
    consecutive_failures = 0
    current_agent = self.zero_agent

    while attempt_count < max_attempts:
        response = self.agents.client.run(
            agent=current_agent,
            messages=self.messages,
            context_variables={...},
            max_turns=1
        )
```

### Safety Nets and Graceful Fallbacks

There are several mechanisms to ensure robust collaboration (they don't get stuck or get crazy):

1. **Maximum Attempts**: Agents can't debate forever - they have a limited number of turns to reach consensus
2. **Consecutive Failure Detection**: If something goes wrong multiple times, the system can adapt
3. **Forced Consensus**: As a last resort, we can take the best content (from the last turn):

```python
def _force_consensus(self):
    """When agents can't agree, we use the last good proposal"""
    for msg in reversed(self.messages):
        if content := msg.get("content", ""):
            return content.strip()
```

### The IRC-Style Interface

One unique aspect of our implementation is the IRC-style logging, which makes the agent interactions visible and engaging:

Note: <screenshot goes here>


## The Complete Book Generation Flow

Let's break down how all the components work together to create a book from start to finish. The project structure follows a logical flow that mirrors the book creation process:

```console
book_creation_flow/
├── 1. Initial Setup
│   ├── agents.py          # Agent Class: Defines Zero and Gustave's base capabilities
│   ├── config.py          # Global settings and configurations
│   └── irc_logger.py      # IRC Class: Provides IRC-style output for agent interactions
│
├── 2. Content Generation Pipeline (these are classes which create the agents for this section of work)
│   ├── title_generator.py                # Step 1: Collaborative title creation
│   ├── table_of_contents_generator.py    # Step 2: Structure planning
│   └── section_writer.py                 # Step 3: Content creation
│
└── 3. Prompts (Agent Instructions)
    ├── title_prompts.py     # Title generation guidelines
    ├── toc_prompts.py       # Structure planning rules
    └── section_prompts.py   # Content writing instructions
```

### The Generation Pipeline

1. **Topic Input → Title Generation**
   - User provides a book topic
   - The `TitleGenerator` class orchestrates Zero and Gustave's collaboration
   - Output: Agreed-upon book title

2. **Title → Table of Contents**
   - The `TableOfContentsGenerator` class takes the title and topic
   - Agents debate and refine the book's structure
   - Output: Hierarchical chapter and section plan

3. **Section-by-Section Writing**
   - The `SectionWriter` class processes each section in order
   - For each section:
     - Zero drafts initial content
     - Gustave refines and improves
     - Both iterate until consensus
   - Output: Complete section content

4. **Book Assembly**
   - The `BookManager` class manages and combines all generated content
   - Organizes files in the output directory structure:
     ```
     books/book_title/
     ├── title.txt
     ├── table_of_contents.txt
     └── sections/
         └── section_*.txt
     ```

Each stage follows the same collaborative pattern we saw in the title generation, but with stage-specific prompts and success criteria. The IRC logger provides visibility into the entire process, making it easy to follow the agents' progress and debug if needed.

## Conclusion: Building a Collaborative AI Writing System with Swarm

We've explored how to build a multi-agent writing system using OpenAI's Swarm framework, taking a unique approach to collaborative AI content creation. Here's what we've learned:

### Key Takeaways

1. **Effective Agent Collaboration**
   - Zero and Gustave work as a complementary team
   - Clear roles and personalities drive better outcomes
   - Structured handoffs enable natural collaboration

2. **Robust Architecture**
   - Simple but powerful agent primitives
   - Clear separation of concerns (agents, prompts, generation)
   - Resilient consensus mechanisms with fallbacks

3. **Practical Implementation Patterns**
   - IRC-style logging for visibility
   - Stage-based content generation
   - Context preservation between handoffs

### Beyond Book Writing

While we've focused on book creation, these patterns could be applied to many collaborative AI tasks:
- Technical documentation
- Content review systems
- Educational material generation
- Any task requiring multiple perspectives and refinement

### Future Possibilities

Although Swarm is experimental, this implementation demonstrates the potential for:
- More complex agent interactions
- Specialized agent roles
- Sophisticated consensus mechanisms
- Real-time collaborative content generation


So.. what's your next collaborative AI project going to be?

