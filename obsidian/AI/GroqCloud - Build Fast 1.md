---
title: "GroqCloud - Build Fast"
source: "https://console.groq.com/playground"
author:
published:
created: 2026-04-24
description: "Build Fast with GroqCloud"
tags:
  - "clippings ai"
---
## Playground

System

Enter system message (Optional)

```
from groq import Groq

client = Groq()
completion = client.chat.completions.create(
    model="openai/gpt-oss-120b",
    messages=[
      {
        "role": "user",
        "content": ""
      }
    ],
    temperature=1,
    max_completion_tokens=8192,
    top_p=1,
    reasoning_effort="medium",
    stream=True,
    stop=None
)

for chunk in completion:
    print(chunk.choices[0].delta.content or "", end="")
```