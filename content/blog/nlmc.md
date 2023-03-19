+++
title = "NLMC - Speak to your APIs" 
date = "2023-03-08"
+++

Today, I want to share my thoughts on GPT-3 and how it's going to change the way we interact with software or machines. GPT-3 has been publicly deployed for a few months now, and it's already making waves in the tech industry and beyond.

What's truly remarkable about GPT-3 is its ability to understand natural language. This means that machines and software can now interpret human language and respond accordingly. But that's just the tip of the iceberg. When we combine GPT-3's natural language processing with summarization and postprocessing, we can create a very interesting workflow that I call NLMC.

NLMC, or Natural Language Machine Control, describe the process to manipulate traditionnal software interfaces with Natural Language. NLMC is a specialized approach of the NLP that aims to optimize Human / Machine interface thanks to language. With NLMC, we can build systems that can understand our commands and react by reusing APIs or classical automations.

Imagine that you can control your entire home automation system with just your voice or a messaging systeù. You could turn on the lights, adjust the temperature, and even start your coffee maker without lifting a finger.

Example:

```json
// User: Hot room please

{
  "type": "success",
  "system": "thermostat",
  "value": 25,
  "explanation": "The user requested a hot room, so the temperature is set to the highest accepted value of 25.",
  "user_message": "The temperature has been set to 25°C to make the room hotter."
}
```

To me it's not just about home automation. NLMC approach can automate tedious tasks like filling very long forms, freeing up our time by avoiding learning new interfaces and new systems or dealing with imperfect knowledge of a system.

LLMs are a real game-changer. Their ability to understand natural language is paving the way for very efficient NLMC, which could totally change the way we interact with machines.

Future applications are going to be awesome, let's see in few month how fun it would be!
