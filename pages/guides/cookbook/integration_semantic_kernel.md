---
title: Observability for Semantic Kernel with Langfuse
description: Learn how to integrate Langfuse with Semantic Kernel for improved monitoring and debugging
category: Integrations
---

# Integrate Langfuse with Semantic Kernel

This notebook demonstrates how to integrate **Langfuse** with **Semantic Kernel** for improved observability and debugging. By the end of this notebook, you will be able to trace your Semantic Kernel applications with Langfuse.

> **What is Semantic Kernel?** [Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/overview/) [(GitHub)](https://github.com/microsoft/semantic-kernel) is an open-source SDK developed by Microsoft that combines LLMs with programming languages like C#, Python, and Java. It allows developers to create AI applications by integrating AI services, data sources, and logic, enabling rapid delivery of enterprise-grade solutions.

> **What is Langfuse?** [Langfuse](https://langfuse.com) is an open-source platform for LLM observability. It provides tracing and monitoring capabilities for AI applications, helping developers debug, analyze, and optimize their AI systems. Langfuse integrates with various tools and frameworks via native integrations, OpenTelemetry, and SDKs.

## Get Started

We'll walk through a simple example of using Semantic Kernel and integrating it with Langfuse.

### Step 1: Install Dependencies

Install the necessary packages:



```python
%pip install langfuse openlit semantic-kernel
```


### Step 2: Set Up Environment Variables

Set your Langfuse API keys and configure the Langfuse SDK. Replace the placeholders with your actual API keys.



```python
import os
import base64

LANGFUSE_PUBLIC_KEY="pk-lf-..."
LANGFUSE_SECRET_KEY="sk-lf-..."
LANGFUSE_AUTH=base64.b64encode(f"{LANGFUSE_PUBLIC_KEY}:{LANGFUSE_SECRET_KEY}".encode()).decode()

os.environ["OTEL_EXPORTER_OTLP_ENDPOINT"] = "https://cloud.langfuse.com/api/public/otel" # EU data region
# os.environ["OTEL_EXPORTER_OTLP_ENDPOINT"] = "https://us.cloud.langfuse.com/api/public/otel" # US data region
os.environ["OTEL_EXPORTER_OTLP_HEADERS"] = f"Authorization=Basic {LANGFUSE_AUTH}"

# your openai key
os.environ["OPENAI_API_KEY"] = "sk-..."
os.environ["OPENAI_CHAT_MODEL_ID"] = "gpt-4o"
```

### Step 3: Initialize OpenLit

Initialize the [OpenLit instrumentation SDK](https://docs.openlit.io/latest/sdk-configuration) to start capturing OpenTelemetry traces.


```python
import openlit

openlit.init()
```

### Step 4: Create a Simple Semantic Kernel Application

We'll create a simple Semantic Kernel application where an Assistant agent answers a user's question.


```python
from semantic_kernel import Kernel

kernel = Kernel()
```


```python
from semantic_kernel.connectors.ai.open_ai import OpenAIChatCompletion

kernel.add_service(
    OpenAIChatCompletion(),
)
```


```python
from semantic_kernel.connectors.ai.open_ai import AzureChatPromptExecutionSettings, OpenAIChatPromptExecutionSettings
from semantic_kernel.prompt_template import InputVariable, PromptTemplateConfig

prompt = """{{$input}}
Answer the question above.
"""

prompt_template_config = PromptTemplateConfig(
    template=prompt,
    name="summarize",
    template_format="semantic-kernel",
    input_variables=[
        InputVariable(name="input", description="The user input", is_required=True),
    ]
)

summarize = kernel.add_function(
    function_name="summarizeFunc",
    plugin_name="summarizePlugin",
    prompt_template_config=prompt_template_config,
)
```


```python
input_text = "What is Langfuse?"

summary = await kernel.invoke(summarize, input=input_text)

print(summary)
```


### Step 5: See Traces in Langfuse

After running the agent above, you can log in to your Langfuse dashboard and view the traces generated by your Semantic Kernel application. Here is an example screenshot of a trace in Langfuse:

![Langfuse Trace](https://langfuse.com/images/cookbook/integration-semantic-kernel/sematric-kernel-example-trace.png)

You can also view the public trace here: [Langfuse Trace Example](https://cloud.langfuse.com/project/cloramnkj0002jz088vzn1ja4/traces/14c7a9f1cc0d7ff16ac1a057a3d45be9?timestamp=2025-02-04T18%3A00%3A53.475Z&observation=cb3f0fb8a2369414)

## References

- [Langfuse OpenTelemetry Docs](https://langfuse.com/docs/opentelemetry/get-started)
- [Semantic Kernel OpenTelemetry Docs](https://github.com/microsoft/semantic-kernel/blob/main/dotnet/docs/TELEMETRY.md)


