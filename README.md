[![GitHub license](https://img.shields.io/github/license/op5dev/ai-inference-request?logo=apache&label=License)](LICENSE "Apache License 2.0.")
[![GitHub release tag](https://img.shields.io/github/v/release/op5dev/ai-inference-request?logo=semanticrelease&label=Release)](https://github.com/op5dev/ai-inference-request/releases "View all releases.")
*
[![GitHub repository stargazers](https://img.shields.io/github/stars/op5dev/ai-inference-request)](https://github.com/op5dev/ai-inference-request "Become a stargazer.")

# AI Inference Request via GitHub Action

> [!TIP]
> [AI inference request](https://docs.github.com/en/rest/models/inference#run-an-ai-inference-request "GitHub API documentation.") [GitHub Models](https://github.com/marketplace?type=models "GitHub Models catalog.") via this [GitHub Action](https://github.com/marketplace/actions/ai-inference-request-via-github-action "GitHub Actions marketplace.").

</br>

## Usage Examples

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
        uses: op5dev/ai-inference-request@v2
        with:
          payload: |
            model: openai/gpt-4.1-mini
            messages:
              - role: system
                content: You are a helpful assistant running within GitHub CI.
              - role: user
                content: Concisely summarize this GitHub issue titled ${{ github.event.issue.title }}: ${{ github.event.issue.body }}
            max_tokens: 100
            temperature: 0.9
            top_p: 0.9

      - name: Comment summary
        run: gh issue comment $NUMBER --body "$SUMMARY"
        env:
          GH_TOKEN: ${{ github.token }}
          NUMBER: ${{ github.event.issue.number }}
          SUMMARY: ${{ steps.prompt.outputs.response }}
```

</br>

## Inputs

Either `payload` or `payload-file` is required. [Compare available AI models](https://docs.github.com/en/copilot/using-github-copilot/ai-models/choosing-the-right-ai-model-for-your-task "Comparison of AI models for GitHub.") to choose the best one for your use-case.

| Type   | Name                 | Description                                                                                                  |
| ------ | -------------------- | ------------------------------------------------------------------------------------------------------------ |
| Data   | `payload`            | Body parameters of the inference request in YAML format.</br>Example: `model:…`                              |
| Data   | `payload-file`       | Path to a file containing the body parameters of the inference request.</br>Example: `./payload.[json\|yml]` |
| Config | `show-payload`       | Whether to show the payload in the logs.</br>Default: `true`                                                 |
| Config | `show-response`      | Whether to show the response content in the logs.</br>Default: `true`                                        |
| Admin  | `github-api-version` | GitHub API version.</br>Default: `2022-11-28`                                                                |
| Admin  | `github-token`       | GitHub token.</br>Default: `github.token`                                                                    |
| Admin  | `org`                | Organization for request attribution.</br>Example: `github.repository_owner`                                 |

</br>

## Outputs

| Name           | Description                                                     |
| -------------- | --------------------------------------------------------------- |
| `response`     | Response content from the inference request.                    |
| `response-raw` | File path containing the complete, raw response in JSON format. |
| `payload`      | Body parameters of the inference request in JSON format.        |

</br>

## Security

View [security policy and reporting instructions](SECURITY.md).

> [!TIP]
>
> Pin your GitHub Action to a [commit SHA](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions#using-third-party-actions "Security hardening for GitHub Actions.") to harden your CI/CD pipeline security against supply chain attacks.

</br>

## Changelog

View [all notable changes](https://github.com/op5dev/ai-inference-request/releases "Releases.") to this project in [Keep a Changelog](https://keepachangelog.com "Keep a Changelog.") format, which adheres to [Semantic Versioning](https://semver.org "Semantic Versioning.").

> [!TIP]
>
> All forms of **contribution are very welcome** and deeply appreciated for fostering open-source projects.
>
> - [Create a PR](https://github.com/op5dev/ai-inference-request/pulls "Create a pull request.") to contribute changes you'd like to see.
> - [Raise an issue](https://github.com/op5dev/ai-inference-request/issues "Raise an issue.") to propose changes or report unexpected behavior.
> - [Open a discussion](https://github.com/op5dev/ai-inference-request/discussions "Open a discussion.") to discuss broader topics or questions.
> - [Become a stargazer](https://github.com/op5dev/ai-inference-request/stargazers "Become a stargazer.") if you find this project useful.

</br>

## License

- This project is licensed under the **permissive** [Apache License 2.0](LICENSE "Apache License 2.0.").
- All works herein are my own, shared of my own volition, and [contributors](https://github.com/op5dev/ai-inference-request/graphs/contributors "Contributors.").
- Copyright 2016-present [Rishav Dhar](https://rdhar.dev "Rishav Dhar's profile.") — All wrongs reserved.
