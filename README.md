[![GitHub license](https://img.shields.io/github/license/op5dev/prompt-ai?logo=apache&label=License)](LICENSE "Apache License 2.0.")
[![GitHub release tag](https://img.shields.io/github/v/release/op5dev/prompt-ai?logo=semanticrelease&label=Release)](https://github.com/op5dev/prompt-ai/releases "View all releases.")
*
[![GitHub repository stargazers](https://img.shields.io/github/stars/op5dev/prompt-ai)](https://github.com/op5dev/prompt-ai "Become a stargazer.")

# Prompt GitHub AI Models via GitHub Action

> [!TIP]
> Prompt GitHub AI Models using [inference request](https://docs.github.com/en/rest/models/inference?apiVersion=2022-11-28#run-an-inference-request "GitHub API documentation.") via GitHub Action API.

</br>

## Usage Examples

[Compare available AI models](https://docs.github.com/en/copilot/using-github-copilot/ai-models/choosing-the-right-ai-model-for-your-task "Comparison of AI models for GitHub.") to choose the best one for your use-case.

### Summarize GitHub Issues

```yml
on:
  issues:
    types: opened

jobs:
  summary:
    runs-on: ubuntu-latest

    permissions:
      issues: write
      models: read

    steps:
      - name: Summarize issue
        id: prompt
        uses: op5dev/prompt-ai@v2
        with:
          user-prompt: |
            Concisely summarize the GitHub issue
            with title '${{ github.event.issue.title }}'
            and body: ${{ github.event.issue.body }}
          max_tokens: 250

      - name: Comment summary
        run: gh issue comment $NUMBER --body "$SUMMARY"
        env:
          GH_TOKEN: ${{ github.token }}
          NUMBER: ${{ github.event.issue.number }}
          SUMMARY: ${{ steps.prompt.outputs.response }}
```

### Troubleshoot Terraform Deployments

```yml
on:
  pull_request:
  push:
    branches: main

jobs:
  provision:
    runs-on: ubuntu-latest

    permissions:
      actions: read
      checks: write
      contents: read
      pull-requests: write
      models: read

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Provision Terraform
        id: provision
        uses: op5dev/tf-via-pr@v13
        with:
          working-directory: env/dev
          command: ${{ github.event_name == 'push' && 'apply' || 'plan' }}

      - name: Troubleshoot Terraform
        if: failure()
        uses: op5dev/prompt-ai@v2
        with:
          model: openai/gpt-4.1-mini
          system-prompt: You are a helpful DevOps assistant and expert at debugging Terraform errors.
          user-prompt: Troubleshoot the following Terraform output; ${{ steps.provision.outputs.result }}
          max-tokens: 500
          temperature: 0.7
          top_p: 0.9
```

</br>

## Inputs

The only required input is `user-prompt`, while every parameter can be tuned per [documentation](https://docs.github.com/en/rest/models/inference?apiVersion=2022-11-28#run-an-inference-request "GitHub API documentation.").

| Type       | Name                   | Description                                                                                                                                                                                                                                   |
| ---------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Common     | `model`                | Model ID to use for the inference request.</br>(e.g., `openai/gpt-4.1-mini`)                                                                                                                                                                  |
| Common     | `system-prompt`        | Prompt associated with the `system` role.</br>(e.g., `You are a helpful software engineering assistant`)                                                                                                                                      |
| Common     | `user-prompt`          | Prompt associated with the `user` role.</br>(e.g., `List best practices for workflows with GitHub Actions`)                                                                                                                                   |
| Common     | `max-tokens`           | The maximum number of tokens to generate in the completion. The token count of your prompt plus `max-tokens` cannot exceed the model's context length.</br>(e.g., `100`)                                                                      |
| Common     | `temperature`          | The sampling temperature to use that controls the apparent creativity of generated completions. Higher values will make output more random while lower values will make results more focused and deterministic.</br>(e.g., range is `[0, 1]`) |
| Common     | `top-p`                | An alternative to sampling with temperature called nucleus sampling. This value causes the model to consider the results of tokens with the provided probability mass.</br>(e.g., range is `[0, 1]`)                                          |
| Additional | `frequency-penalty`    | A value that influences the probability of generated tokens appearing based on their cumulative frequency in generated text.</br>(e.g., range is `[-2, 2]`)                                                                                   |
| Additional | `modalities`           | The modalities that the model is allowed to use for the chat completions response.</br>(e.g., from `text` and `audio`)                                                                                                                        |
| Additional | `org`                  | Organization to which the request is to be attributed.</br>(e.g., `github.repository_owner`)                                                                                                                                                  |
| Additional | `presence-penalty`     | A value that influences the probability of generated tokens appearing based on their existing presence in generated text.</br>(e.g., range is `[-2, 2]`)                                                                                      |
| Additional | `seed`                 | If specified, the system will make a best effort to sample deterministically such that repeated requests with the same seed and parameters should return the same result.</br>(e.g., `123456789`)                                             |
| Additional | `stop`                 | A collection of textual sequences that will end completion generation.</br>(e.g., `["\n\n", "END"]`)                                                                                                                                          |
| Additional | `stream`               | A value indicating whether chat completions should be streamed for this request.</br>(e.g., `false`)                                                                                                                                          |
| Additional | `stream-include-usage` | Whether to include usage information in the response.</br>(e.g., `false`)                                                                                                                                                                     |
| Additional | `tool-choice`          | If specified, the model will configure which of the provided tools it can use for the chat completions response.</br>(e.g., 'auto', 'required', or 'none')                                                                                    |
| Payload    | `payload`              | Body parameters of the inference request in JSON format.</br>(e.g., `{"model"…`)                                                                                                                                                              |
| Payload    | `payload-file`         | Path to a JSON file containing the body parameters of the inference request.</br>(e.g., `./payload.json`)                                                                                                                                     |
| Payload    | `show-payload`         | Whether to show the body parameters in the workflow log.</br>(e.g., `false`)                                                                                                                                                                  |
| Payload    | `show-response`        | Whether to show the response content in the workflow log.</br>(e.g., `true`)                                                                                                                                                                  |
| GitHub     | `github-api-version`   | GitHub API version.</br>(e.g., `2022-11-28`)                                                                                                                                                                                                  |
| GitHub     | `github-token`         | GitHub token for authorization.</br>(e.g., `github.token`)                                                                                                                                                                                    |

</br>

## Outputs

Due to GitHub's API limitations, the `response` content is truncated to 262,144 (2^18) characters so the complete, raw response is saved to `response-file`.

| Name            | Description                                                     |
| --------------- | --------------------------------------------------------------- |
| `response`      | Response content from the inference request.                    |
| `response-file` | File path containing the complete, raw response in JSON format. |
| `payload`       | Body parameters of the inference request in JSON format.        |

</br>

## Security

View [security policy and reporting instructions](SECURITY.md).

> [!TIP]
>
> Pin your GitHub Action to a [commit SHA](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions#using-third-party-actions "Security hardening for GitHub Actions.") to harden your CI/CD **pipeline security** against supply chain attacks.

</br>

## Changelog

View [all notable changes](https://github.com/op5dev/prompt-ai/releases "Releases.") to this project in [Keep a Changelog](https://keepachangelog.com "Keep a Changelog.") format, which adheres to [Semantic Versioning](https://semver.org "Semantic Versioning.").

> [!TIP]
>
> All forms of **contribution are very welcome** and deeply appreciated for fostering open-source projects.
>
> - [Create a PR](https://github.com/op5dev/prompt-ai/pulls "Create a pull request.") to contribute changes you'd like to see.
> - [Raise an issue](https://github.com/op5dev/prompt-ai/issues "Raise an issue.") to propose changes or report unexpected behavior.
> - [Open a discussion](https://github.com/op5dev/prompt-ai/discussions "Open a discussion.") to discuss broader topics or questions.
> - [Become a stargazer](https://github.com/op5dev/prompt-ai/stargazers "Become a stargazer.") if you find this project useful.

</br>

## License

- This project is licensed under the **permissive** [Apache License 2.0](LICENSE "Apache License 2.0.").
- All works herein are my own, shared of my own volition, and [contributors](https://github.com/op5dev/prompt-ai/graphs/contributors "Contributors.").
- Copyright 2016-present [Rishav Dhar](https://rdhar.dev "Rishav Dhar's profile.") — All wrongs reserved.
