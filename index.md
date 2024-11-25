---
title: Introducing Instruct
---

# Why another prompting library?

When I started working with Large Language Models (LLMs) in Ruby, something felt
off. I found tools were either too abstract, hiding the LLM’s capabilities
behind unseen prompts, or too low-level, leaving my classes hard to follow
and littered with boilerplate managing prompts and responses.

After reading an early version of [Patterns of Application Development Using AI
by Obie Fernandez <span style="vertical-align: super; font-size:
8px;">(affiliate
link)</span>](https://www.amazon.com/Patterns-Application-Development-Using-AI-ebook/dp/B0DMP496C8?_encoding=UTF8&dib=eyJ2IjoiMSJ9.37B8s3_VYQDkkNUrd8Pf9tORwsEyCWCzzvWnLB8-5QbwojTZm-OOCGQdBmwlMsCHHvIeU0o51xpoH4QF4ST_hPqW_nJ3SG99vgYueNEUz1I.gSDOM1BYP0IMYXgweqUwii9Sclbd2jZmZzXDcWwQlG0&dib_tag=se&keywords=B001IGV0LS&qid=1733136367&s=digital-text&shoppingPortalEnabled=true&sr=1-1&linkCode=ll1&tag=mackross-20&linkId=860edad6d655fea9fe5254170bb507e1&language=en_US&ref_=as_li_ss_tl)
and using Obie's library [raix](https://github.com/OlympiaAI/raix), I felt
inspired. The book has so many great patterns, and raix's transcript management
and tool management were the first that felt ruby-ish.

Simultatenously, I really liked some ideas I was seeing in the python community,
[guidance](https://github.com/guidance-ai/guidance),
[DSPy](https://github.com/stanfordnlp/dspy) and
[TEXTGRAD](https://textgrad.com/) all catching my eye. I liked what the
cross-platform [BAML](https://docs.boundaryml.com/home) was doing too, but I
didn't love the code generation side of it.

And so, I've set out to build something that combines the best of all these,
just for ruby. Starting with Instruct.

# Instruct in Action

Here is an example of how you might use Instruct to create a dynamic Q&A loop
between two agents:
```ruby
  # Create two agents: Noel Gallagher and an interviewer with a system prompt.
  noel = p{"system: You're Noel Gallagher. Answer questions from an interviewer."}
  interviewer = p{"system: You're a skilled interviewer asking Noel Gallagher questions."}

  # We start a dynamic Q&A loop with the interviewer by kicking off the
  # interviewing agent and capturing the response under the :reply key.
  interviewer << p{"user: __Noel sits down in front of you.__"} + gen.capture(:reply) + "\n".prompt_safe

  puts interviewer.captured(:reply) # => "Hello Noel, how are you today?"

  5.times do
    # Noel is sent the last value captured in the interviewer's transcript
    noel << p{"user: <%= interviewer.captured(:reply) %>"} + gen.capture(:reply, list: :replies) + "\n".prompt_safe

    # Noel's captured reply is now sent to the interviewer
    interviewer << p{"user: <%=  noel.captured(:reply) %>"} + gen.capture(:reply, list: :replies) + "\n".prompt_safe
  end

  # After the conversation, we can access the list captured replies from both agents
  noel_said = noel.captured(:replies).map{ |r| "noel: #{r} }
  interviewer_said = interviewer.captured(:replies).map{ |r| "interviewer: #{r} }

  puts noel_said.zip(interviewer_said).flatten.join("\n\n")
  # => "noel: ... \n\n interviewer: ..., ..."
```

Or pull apart the prompt to create a streaming chatbot that handles user input safely:

```ruby
  # Create a chat transcript with a system prompt.

  # Instruct uses inline ERB to interpolate user input into the prompt and mark it as
  # unsafe. Middleware can then perform operations to sanitize the input (guard rails).
  transcript = p{<<~ERB.chomp
    system: You're a helpful concise assistant.
    user: <%= unsafe_user_content %>"
    ERB
  }

  # Adding a gen call to a transcript creates a callable prompt.
  prompt = chatbot + gen

  # We can stream input to the user and capture the response.
  response = prompt.call do | streamed_response |
    update_ui_with_generating_response(streamed_response)
    puts streamed_response
    # => "Sur"
    # => "Sure, what do"
    # => "Sure, what do you need help"
    # => "Sure, what do you need help with?"
  end

  puts response
  # => "Sure, what do you need help with?"

  # Add the response to the chat transcript.
  transcript = prompt + response
  puts transcript # =>
  # "system: You're a helpful chatbot.
  #  user: Hello, can you help me?
  #  assistant: Sure, what do you need help with?"

  # Note the transcript now has the assistant's response ready for another round
  # of input.
```

To see more examples of how Instruct can be used, check out the [documentation](https://github.com/instruct-rb/instruct).

## 🚧 Disclaimer
**This gem is still undergoing active development and is not yet ready for use beyond experimentation.**

I'm making this public to get feedback and to see if there is any interest in from the community to help develop this further.

# The Plan

## 1. Instruct (Experimenting Now)
Build a great foundation for working with different LLMs and doing complex
prompting. Use middleware to manage complexities and build upon attributed
string system for easy multi-modal LLM integration and flexiblity in adding new
capabilites. Add streaming validation and parsing.

## 2. Production Data Collection (Next)
A gem that builds on Instruct to collect prompts and completions in production.
Creates datasets for data-driven prompt optimization and fine-tuning.

## 3. Optimization Workflow (Planned)
Enable both human driven and LLM prompt optimization with integrated fine-tuning
using production data.

## 4. Self-Optimizing Systems (Planned)
Why deploy optimizations when you can let your model optimize itself?

# Ruby Alternatives
Instruct was built to make sense for me, but it might not be for you. Here are
some great alternatives:

* [langchainrb](https://github.com/patterns-ai-core/langchainrb) fantastic if you're already familiar with langchain.
* [raix](https://github.com/OlympiaAI/raix) Rails-y and class-based approach to LLM.
* [active_prompt](https://github.com/Shaneprrlt/active_prompt) Liquid templates and Rails-style classes.
* [lammy](https://kamil.fyi/introducing-lammy/) Lightweight Rails-style classs approach.
* [baml](https://docs.boundaryml.com/home) Feature-rich cross-platform solution (some paid)

# Join the Journey
Want to help shape the future of LLM development in Ruby? Please reach out to
andrew@mackross.net or open an issue on
[GitHub](https://github.com/instruct-rb/instruct).

--
Created by mackross with 💖 for the Ruby community.