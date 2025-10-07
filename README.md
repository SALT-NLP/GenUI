<br />
<p align="center">
<a href="https://salt-nlp.github.io/generative_interfaces/" target="_blank">
  <img src="./static/head.png" alt="GenUI banner">
</a>
</p>
Jiaqi Chen*, Yanzhe Zhang*, Yutong Zhang, Yijia Shao, Diyi Yang

Stanford University

*Equal contribution
<br />
<h3 align="center">
  <b><a href="https://salt-nlp.github.io/generative_interfaces/">Homepage</a></b>
  •
  <b><a href="https://arxiv.org/abs/2508.19227">Paper</a></b>
  •
  <b><a href="https://huggingface.co/datasets/SALT-NLP/GenUI">Dataset</a></b>
  •
  <b><a href="https://salt-nlp.github.io/generative_interfaces/dataviewer/data_viewer.html">Data Viewer</a></b>

## What are Generative Interfaces?

We investigate Generative Interfaces for Language Models, a paradigm where LLMs respond to user queries by proactively generating user interfaces (UIs) to enable more adaptive, interactive interactions that better support complex user goals.

### Updates

[10/07/2025] We conducted a new user study where 76 participants used their own daily queries to compare generative interfaces with conversational ones. Data is released <a href="https://huggingface.co/datasets/SALT-NLP/GenUI" target="_blank" rel="noopener noreferrer">here</a>. Data viewer is available <a href="https://salt-nlp.github.io/generative_interfaces/dataviewer/data_viewer.html" target="_blank" rel="noopener noreferrer">here</a>.

## How do Generative Interfaces work?
- **Requirement specification** [[system prompt](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/prompts.ts#L960)], [[Code](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/nodes/analyze-requirements.ts#L31)]: First, we parse the input into a requirement specification, capturing the main goal, desired features, UI components, interaction styles, and problem-solving strategies.

- **Structured representation generation** [[system prompt](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/prompts.ts#L960)], [[Code](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/nodes/generate-web-dsl/index.ts#L21)]: Second, we generate a Structured Interface-Specific Representation based on the requirement specification.

- **UI generation** [[system prompt](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/prompts.ts#L1458)], [[Code](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/nodes/rewrite-artifact/index.ts)]: To support faithful realization of the structured specification, we utilize a component codebase containing reusable implementations of common UI elements (e.g., charts, videos, synchronized clocks). In addition, a web retrieval module gathers relevant UI examples and data sources to inform both the representation design and the final rendering.
Finally, the entire context, including the natural language query, requirement specification, structured representation, 7 predefined components, and retrieved examples, is passed to a code generation model, which synthesizes executable HTML/CSS/JS code. This completes the pipeline from query to fully rendered, high-quality interactive interface.

- **Adaptive reward function** [[system prompt](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/prompts.ts#L1591)], [[Code](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/nodes/rewrite-artifact/evaluation.ts#L30)]: We use a large language model to automatically generate evaluation criteria based on each user query, such as “clarity” or “concept explanation,” assigning weights and verification rules to compute an overall score.

- **Iterative refinement** [[system prompt](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/prompts.ts#L1642)], [[Code](https://github.com/SALT-NLP/GenUI/blob/main/apps/agents/src/open-canvas/nodes/rewrite-artifact/index.ts)]: We first generate several UI candidates and score them using the reward function. The best one is selected, then used to guide the next round of generation. This process repeats with feedback until a candidate meets the quality threshold.


## Setup

### Prerequisites

#### Package Manager

- [Yarn](https://yarnpkg.com/)

#### API Keys

Put the following API keys in the `.env` file.

- [OpenAI API key](https://platform.openai.com/signup/)
- [Anthropic API key](https://console.anthropic.com/)
- [Google API key](https://console.cloud.google.com/apis/credentials)
- [EXA API key](https://exa.ai/)

#### Authentication

- [Supabase](https://supabase.com/) account for authentication

#### LangGraph Server

- [LangGraph CLI](https://langchain-ai.github.io/langgraph/cloud/reference/cli/) for running the graph locally

#### LangSmith

- [LangSmith](https://smith.langchain.com/) for tracing & observability

### Installation

First, clone the repository:

```bash
git clone git@github.com:SALT-NLP/GenUI.git
cd GenUI
```

Next, install the dependencies:

```bash
yarn install
```

After installing dependencies, set the required values (API keys, authentication information) in `./apps/web/.env.example`.
Then copy it to `.env` in the root folder of the project, and in `apps/web`.

```bash
# The root `.env` file will be read by the LangGraph server for the agents.
cp ./apps/web/.env.example ./.env
cp ./apps/web/.env.example ./apps/web/.env
```

#### Setup Authentication

After creating a Supabase account, visit your [dashboard](https://supabase.com/dashboard/projects) and create a new project.

Next, navigate to the `Project Settings` page inside your project, and then to the `API` tag. Copy the `Project URL`, and `anon public` project API key. Paste them into the `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` environment variables in the `apps/web/.env` file.

After this, navigate to the `Authentication` page and the `Providers` tab. Make sure `Email` is enabled (also ensure you've enabled `Confirm Email`). You may also enable `GitHub`, and/or `Google` if you'd like to use those for authentication. (see these pages for documentation on how to set up each provider: [GitHub](https://supabase.com/docs/guides/auth/social-login/auth-github), [Google](https://supabase.com/docs/guides/auth/social-login/auth-google))

#### Test Authentication

To verify authentication works, run `yarn dev` and visit [localhost:3000](http://localhost:3000). This should redirect you to the [login page](http://localhost:3000/auth/login). From here, you can either log in with Google or GitHub, or if you haven't configured these providers, navigate to the [signup page](http://localhost:3000/auth/signup) and create a new account with an email and password. This should then redirect you to a confirmation page, and after confirming your email, you should be redirected to the [home page](http://localhost:3000).

### Setup Server

The first step to running Generating UI locally is to build the application. This is because Generating UI uses a monorepo setup and requires workspace dependencies to be built so other packages/apps can access them.

1. Run the following command from the root of the repository:

```bash
yarn build
```

2. Navigate to `apps/agents` and run `yarn dev` (this runs `npx @langchain/langgraph-cli dev --port 54367`).

You will see something like:

```
Ready!
- 🚀 API: http://localhost:54367
- 🎨 Studio UI: https://smith.langchain.com/studio?baseUrl=http://localhost:54367
```

3. After your LangGraph server is running, execute the following command inside `apps/web` to start the Generating UI frontend:

```bash
yarn dev
```

On initial load, compilation may take time.

4. Open [localhost:3000](http://localhost:3000) with your browser and start trying generative interfaces.
  - Using Claude is recommended. Turn on web search to enable fetching relevant web pages.
  - Generation can take multiple minutes due to iterative generation.
  - You can track the intermediate steps in the terminal where you run `yarn dev` in `apps/agents`.

### Troubleshooting

For problems related to `pdf-parse`, you might refer to the solution [here](https://gitlab.com/autokent/pdf-parse/-/issues/24).

# Citation
If you find this work useful for your research, please cite our GitHub repo:
```bibtex
@misc{chen2025generative,
    title={Generative Interfaces for Language Models},
    author={Jiaqi Chen and Yanzhe Zhang and Yutong Zhang and Yijia Shao and Diyi Yang},
    year={2025},
    eprint={2508.19227},
    archivePrefix={arXiv},
    primaryClass={cs.CL}
}
```
