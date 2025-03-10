<div align="center">
  <h1>🍋 Lemon AI</h1>
  <h3>Gateway to empower LLM agents to interact with the world</h3>
  <a href="https://discord.gg/bsgzjEpw">
<img alt="Discord" src="https://img.shields.io/badge/Join Discord-x?style=flat&logo=discord&logoColor=white&label&labelColor=gray&color=5865F2">
  </a>
  <a href="https://twitter.com/getlemonai">
    <img alt="Twitter" src="https://img.shields.io/badge/Tweet at us-x?style=flat&logo=twitter&logoColor=white&label&labelColor=gray&color=1DA1F2">
  </a>
  <a href="https://github.com/trpc/trpc/blob/main/LICENSE">
    <img alt="MIT License" src="https://img.shields.io/github/license/felixbrock/lemonai?labelColor=gray&color=yellow" />
  </a>
  <br />
    <a href="https://python.langchain.com/docs/modules/agents/tools/integrations/lemonai">
    <img alt="Run Notebook in LangChain Docs" src="https://img.shields.io/badge/Run Notebook From LangChain Docs-x?style=for-the-badge&logoColor=white&label&labelColor=gray&color=gray">
  </a>
  <br />
  <br />
  <figure>
    <img src="heatmap-example.gif" alt="Demo" />
  </figure>
</div>

**Lemon AI helps you build powerful AI assistants in minutes and automate workflows by allowing for accurate and reliable read and write operations in [tools](#🧩-supported-tools) like Airtable, Hubspot, Discord, Notion, Slack and Github.**

Most connectors available today are focused on read-only operations, limiting the potential of LLMs. Agents, on the other hand, have a tendency to hallucinate from time to time due to missing context or instructions.

With Lemon AI, it is possible to give your agents access to well-defined APIs for reliable read and write operations. In addition, Lemon AI functions allow you to further reduce the risk of hallucinations by providing a way to statically define workflows that the model can rely on in case of uncertainty.

## ⚡️ Getting Started

### 1. Prerequisites

Requires Python 3.8.1 and above.

### 2. Installing

To use Lemon AI in your Python project run `pip install lemonai`. The following example uses an OpenAI model. Therefore, we also need to `pip install openai`. Depending on which model you want to use this install is optional.

### 3. (Optional) Launch the server

The interaction of your agents and all Lemon AI tools is provided by the [Lemon AI Server](https://github.com/felixbrock/lemonai-server). Per default the Lemon AI Python client is sending requests to a hosted server version. To run the Lemon AI server on-prem please refer to the [Server Documentation](https://github.com/felixbrock/lemonai-server).

### 4. Use Lemon AI with LangChain

The easiest way to give Lemon AI a try is via the Jupyter Notebook you can find in the [LangChain Docs](https://python.langchain.com/docs/modules/agents/tools/integrations/lemonai).

Lemon AI automatically solves given tasks by finding the right combination of relevant tools or uses Lemon AI Functions as an alternative. The following example demonstrates how to retrieve a user from Hackernews and write it to a table in Airtable:

#### Include Lemon AI in your LangChain project

```Python
import os
from lemonai import execute_workflow
from langchain import OpenAI
```

#### Load API keys and access tokens

To use tools that require authentication, you have to store the corresponding access credentials in your environment in the format "{tool name}\_{authentication string}" where the authentication string is one of ["API_KEY", "SECRET_KEY", "SUBSCRIPTION_KEY", "ACCESS_KEY"] for API keys or ["ACCESS_TOKEN", "SECRET_TOKEN"] for authentication tokens. Examples are "OPENAI_API_KEY", "BING_SUBSCRIPTION_KEY", "AIRTABLE_ACCESS_TOKEN".

```Python
""" Load all relevant API Keys and Access Tokens into your environment variables """
os.environ["OPENAI_API_KEY"] = "*INSERT OPENAI API KEY HERE*"
os.environ["GITHUB_API_KEY"] = "*INSERT GITHUB API KEY HERE*"
os.environ["DISCORD_WEBHOOK_URL"] = "*INSERT DISCORD CHANNEL WEBHOOK URL HERE*"
```

#### Example of defining your prompt and executing the Langchain Agent

The following example makes use of several Lemon AI tools to

1. Retrieve details about a personal repository on GitHub
2. Get the top growing starred repositories of my account
3. Send a Discord message with results and review.

![Use Case Example](use-case-example.png)

```Python
lemonai_repo_owner = "Abdus2609"
github_username = "Abdus2609"

""" Define your instruction to be given to your LLM """
prompt = f"""Get the description for a repository I am working with called lemonai (owner {lemonai_repo_owner}).
Also, get my top growing starred repositories (username {github_username}). Analyze the descriptions of both
the LemonAI repository and my top-starred repositories. Then, send a Discord message that first displays a
numerically bullet-pointed leaderboard of the top growing starred repositories and their growth, and secondly
discuss how each tool could be useful specifically to lemonai's use case based on your analysis of the
descriptions of each repository."""

"""
Use the Lemon AI execute_workflow wrapper
to run your LangChain agent in combination with Lemon AI
"""
model = OpenAI(temperature=0)

execute_workflow(llm=model, prompt_string=prompt)
```

#### (Optional) Define your Lemon AI Functions

Similar to [OpenAI functions](https://openai.com/blog/function-calling-and-other-api-updates), Lemon AI provides the option to define workflows as reusable functions. These functions can be defined for use cases where it is especially important to move as close as possible to near-deterministic behavior. Specific workflows can be defined in a separate lemonai.json. You can find the corresponding tool ids (e.g. `hackernews-get-user`) in the [Tool Docs](https://github.com/felixbrock/lemonai/blob/main/docs/tools.md):

```json
[
  {
    "name": "Hackernews Airtable User Workflow",
    "description": "retrieves user data from Hackernews and appends it to a table in Airtable",
    "tools": ["hackernews-get-user", "airtable-append-data"]
  }
]
```

Your model will have access to these functions and will prefer them over self-selecting tools to solve a given task. All you have to do is to let the agent know that it should use a given function by including the function name in the prompt (e.g. `prompt = f"""Execute the Hackernews Airtable user workflow...`).

### 5. Gain transparency on your Agent's decision making

To gain transparency on how your Agent interacts with Lemon AI tools to solve a given task, all decisions made, tools used and operations performed are written to a local `lemonai.log` file. Every time your LLM agent is interacting with the Lemon AI tool stack a corresponding log entry is created:

```log
2023-06-26T11:50:27.708785+0100 - b5f91c59-8487-45c2-800a-156eac0c7dae - hackernews-get-user
2023-06-26T11:50:39.624035+0100 - b5f91c59-8487-45c2-800a-156eac0c7dae - airtable-append-data
2023-06-26T11:58:32.925228+0100 - 5efe603c-9898-4143-b99a-55b50007ed9d - hackernews-get-user
2023-06-26T11:58:43.988788+0100 - 5efe603c-9898-4143-b99a-55b50007ed9d - airtable-append-data
```

By using the [Lemon AI Analytics Tool](https://github.com/felixbrock/lemonai-analytics) you can easily gain a better understanding of how frequently and in which order tools are used. As a result, you can identify weak spots in your agent’s decision-making capabilities and move to a more deterministic behavior by defining Lemon AI functions.

![Heatmap Example](heatmap-example.png)

## 🧩 Supported Tools

We already allow agents to interact with over [120 tools](https://github.com/felixbrock/lemonai/blob/main/docs/tools.md) across the following services:

- HackerNews
- Airtable
- Slack
- HubSpot
- Github
- Notion
- Discord
- Medium
- Monday.com

## 🩻 Next Up

- [x] Github
- [x] Notion
- [x] Discord
- [x] Medium
- [x] Monday.com
- [ ] Gmail
- [ ] Google Calendar
- [ ] Kafka
- [ ] Pipedrive
- [ ] Stripe
- [ ] Google Cloud Realtime Database
- [ ] Salesforce

## 🦸 Contributing

Great to see you here! We are extremely open to contributions! You can find more information in our [CONTRIBUTING.md](https://github.com/felixbrock/lemonai/blob/main/.github/CONTRIBUTING.md). If you have any more questions feel free to drop us a message on <a href="https://discord.gg/bsgzjEpw">Discord</a>.

## ❤️‍🔥 Contributors

> Those are the team members and people who actively help out improving the codebase by making PRs and reviewing code. We are beyond grateful for your help!

<table cellspacing="0" cellpadding="0" style="border:none;">
  <tbody>
    <tr style="border:none;">
      <td align="center" style="border:none;"><a href="https://twitter.com/felixbrockm"><img src="https://avatars.githubusercontent.com/u/70200999?s=100&v=4" width="70px;" alt="" style="border-radius: 50%;"/><br /><sub><b>Felix Brockmeier</b></sub></a></td>
      <td align="center" style="border:none;"><a href="https://www.linkedin.com/in/mohammed-abdus-samad-hannan-3a2687202/"><img src="https://avatars.githubusercontent.com/u/72310364?s=100&v=4" width="70px;" alt="" style="border-radius: 50%;"/><br /><sub><b>Mohammed Hannan</b></sub></a></td>
      <td align="center" style="border:none;"><a href="https://www.linkedin.com/in/haiphunghiem/"><img src="https://avatars.githubusercontent.com/u/16231195?s=100&v=4" width="70px;" alt="" style="border-radius: 50%;"/><br /><sub><b>Hai Nghiem</b></sub></a></td>
      <td align="center" style="border:none;"><a href="https://twitter.com/schroeerclemens"><img src="https://avatars.githubusercontent.com/u/84038864?s=100&v=4" width="70px;" alt="" style="border-radius: 50%;"/><br /><sub><b>Clemens Schroeer</b></sub></a></td>
    </tr>
  </tbody>
</table>
