+++
title = "GrammarQuery"
weight = 5
[taxonomies]
tags = []
+++

GrammarQuery was built as a project for the Unversity of Pittsburgh's
[SheInnovates 2022](http://sheinnovates.us/) Hackathon. It's a voice-based
language learning tool that prompts you with real-world questions that the user
replies to by speaking. It complements other language learning tools that are
focused on learning vocabulary and grammar by teaching the user to use English
in a conversational manner.

A live version of GrammarQuery can be found at
[nickzana.github.io/grammarquery](https://nickzana.github.io/grammarquery/).

## Technical Details

GrammarQuery works by using the [Web Speech
API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API) to speak
the question to the user, then translate the user's audio response to text.

This transcribed text is then sent to the [GrammarBot
API](https://www.grammarbot.io), where the returned errors are presented to the
user.
