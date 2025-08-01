## Show possible configurations

The possible configurations of Qodo Merge are stored in [here](https://github.com/Codium-ai/pr-agent/blob/main/pr_agent/settings/configuration.toml){:target="_blank"}.
In the [tools](https://qodo-merge-docs.qodo.ai/tools/) page you can find explanations on how to use these configurations for each tool.

To print all the available configurations as a comment on your PR, you can use the following command:

```
/config
```

![possible_config1](https://codium.ai/images/pr_agent/possible_config1.png){width=512}

To view the **actual** configurations used for a specific tool, after all the user settings are applied, you can add for each tool a `--config.output_relevant_configurations=true` suffix.
For example:

```
/improve --config.output_relevant_configurations=true
```

Will output an additional field showing the actual configurations used for the `improve` tool.

![possible_config2](https://codium.ai/images/pr_agent/possible_config2.png){width=512}

## Ignoring files from analysis

In some cases, you may want to exclude specific files or directories from the analysis performed by Qodo Merge. This can be useful, for example, when you have files that are generated automatically or files that shouldn't be reviewed, like vendor code.

You can ignore files or folders using the following methods:

- `IGNORE.GLOB`
- `IGNORE.REGEX`

which you can edit to ignore files or folders based on glob or regex patterns.

### Example usage

Let's look at an example where we want to ignore all files with `.py` extension from the analysis.

To ignore Python files in a PR with online usage, comment on a PR:
`/review --ignore.glob="['*.py']"`

To ignore Python files in all PRs using `glob` pattern, set in a configuration file:

```
[ignore]
glob = ['*.py']
```

And to ignore Python files in all PRs using `regex` pattern, set in a configuration file:

```
[ignore]
regex = ['.*\.py$']
```

## Extra instructions

All Qodo Merge tools have a parameter called `extra_instructions`, that enables to add free-text extra instructions. Example usage:

```
/update_changelog --pr_update_changelog.extra_instructions="Make sure to update also the version ..."
```

## Language Settings

The default response language for Qodo Merge is **U.S. English**. However, some development teams may prefer to display information in a different language. For example, your team's workflow might improve if PR descriptions and code suggestions are set to your country's native language.  

To configure this, set the `response_language` parameter in the configuration file. This will prompt the model to respond in the specified language. Use a **standard locale code** based on [ISO 3166](https://en.wikipedia.org/wiki/ISO_3166) (country codes) and [ISO 639](https://en.wikipedia.org/wiki/ISO_639) (language codes) to define a language-country pair. See this [comprehensive list of locale codes](https://simplelocalize.io/data/locales/).  

Example:

```toml
[config]
response_language = "it-IT"
```

This will set the response language globally for all the commands to Italian.

> **Important:** Note that only dynamic text generated by the AI model is translated to the configured language. Static text such as labels and table headers that are not part of the AI models response will remain in US English. In addition, the model you are using must have good support for the specified language.

[//]: # (## Working with large PRs)

[//]: # ()
[//]: # (The default mode of CodiumAI is to have a single call per tool, using GPT-4, which has a token limit of 8000 tokens.)

[//]: # (This mode provides a very good speed-quality-cost tradeoff, and can handle most PRs successfully.)

[//]: # (When the PR is above the token limit, it employs a [PR Compression strategy]&#40;../core-abilities/index.md&#41;.)

[//]: # ()
[//]: # (However, for very large PRs, or in case you want to emphasize quality over speed and cost, there are two possible solutions:)

[//]: # (1&#41; [Use a model]&#40;https://qodo-merge-docs.qodo.ai/usage-guide/changing_a_model/&#41; with larger context, like GPT-32K, or claude-100K. This solution will be applicable for all the tools.)

[//]: # (2&#41; For the `/improve` tool, there is an ['extended' mode]&#40;https://qodo-merge-docs.qodo.ai/tools/improve/&#41; &#40;`/improve --extended`&#41;,)

[//]: # (which divides the PR into chunks, and processes each chunk separately. With this mode, regardless of the model, no compression will be done &#40;but for large PRs, multiple model calls may occur&#41;)

## Patch Extra Lines

By default, around any change in your PR, git patch provides three lines of context above and below the change.

```
@@ -12,5 +12,5 @@ def func1():
 code line that already existed in the file...
 code line that already existed in the file...
 code line that already existed in the file....
-code line that was removed in the PR
+new code line added in the PR
 code line that already existed in the file...
 code line that already existed in the file...
 code line that already existed in the file...
```

Qodo Merge will try to increase the number of lines of context, via the parameter:

```
[config]
patch_extra_lines_before=3
patch_extra_lines_after=1
```

Increasing this number provides more context to the model, but will also increase the token budget, and may overwhelm the model with too much information, unrelated to the actual PR code changes.

If the PR is too large (see [PR Compression strategy](https://github.com/Codium-ai/pr-agent/blob/main/PR_COMPRESSION.md)), Qodo Merge may automatically set this number to 0, and will use the original git patch.

## Log Level

Qodo Merge allows you to control the verbosity of logging by using the `log_level` configuration parameter. This is particularly useful for troubleshooting and debugging issues with your PR workflows.

```
[config]
log_level = "DEBUG"  # Options: "DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"
```

The default log level is "DEBUG", which provides detailed output of all operations. If you prefer less verbose logs, you can set higher log levels like "INFO" or "WARNING".

## Integrating with Logging Observability Platforms

Various logging observability tools can be used out-of-the box when using the default LiteLLM AI Handler. Simply configure the LiteLLM callback settings in `configuration.toml` and set environment variables according to the LiteLLM [documentation](https://docs.litellm.ai/docs/).

For example, to use [LangSmith](https://www.langchain.com/langsmith) you can add the following to your `configuration.toml` file:

```
[litellm]
enable_callbacks = true
success_callback = ["langsmith"]
failure_callback = ["langsmith"]
service_callback = []
```

Then set the following environment variables:

```
LANGSMITH_API_KEY=<api_key>
LANGSMITH_PROJECT=<project>
LANGSMITH_BASE_URL=<url>
```

## Ignoring automatic commands in PRs

Qodo Merge allows you to automatically ignore certain PRs based on various criteria:

- PRs with specific titles (using regex matching)
- PRs between specific branches (using regex matching)
- PRs from specific repositories (using regex matching)
- PRs not from specific folders
- PRs containing specific labels
- PRs opened by specific users

### Ignoring PRs with specific titles

To ignore PRs with a specific title such as "[Bump]: ...", you can add the following to your `configuration.toml` file:

```toml
[config]
ignore_pr_title = ["\\[Bump\\]"]
```

Where the `ignore_pr_title` is a list of regex patterns to match the PR title you want to ignore. Default is `ignore_pr_title = ["^\\[Auto\\]", "^Auto"]`.

### Ignoring PRs between specific branches

To ignore PRs from specific source or target branches, you can add the following to your `configuration.toml` file:

```toml
[config]
ignore_pr_source_branches = ['develop', 'main', 'master', 'stage']
ignore_pr_target_branches = ["qa"]
```

Where the `ignore_pr_source_branches` and `ignore_pr_target_branches` are lists of regex patterns to match the source and target branches you want to ignore.
They are not mutually exclusive, you can use them together or separately.

### Ignoring PRs from specific repositories

To ignore PRs from specific repositories, you can add the following to your `configuration.toml` file:

```toml
[config]
ignore_repositories = ["my-org/my-repo1", "my-org/my-repo2"]
```

Where the `ignore_repositories` is a list of regex patterns to match the repositories you want to ignore. This is useful when you have multiple repositories and want to exclude certain ones from analysis.


### Ignoring PRs not from specific folders

To allow only specific folders (often needed in large monorepos), set:

```
[config]
allow_only_specific_folders=['folder1','folder2']
```

For the configuration above, automatic feedback will only be triggered when the PR changes include files where 'folder1' or 'folder2' is in the file path

### Ignoring PRs containing specific labels

To ignore PRs containing specific labels, you can add the following to your `configuration.toml` file:

```
[config]
ignore_pr_labels = ["do-not-merge"]
```

Where the `ignore_pr_labels` is a list of labels that when present in the PR, the PR will be ignored.

### Ignoring PRs from specific users

Qodo Merge tries to automatically identify and ignore pull requests created by bots using:

- GitHub's native bot detection system
- Name-based pattern matching

While this detection is robust, it may not catch all cases, particularly when:

- Bots are registered as regular user accounts
- Bot names don't match common patterns

To supplement the automatic bot detection, you can manually specify users to ignore. Add the following to your `configuration.toml` file to ignore PRs from specific users:

```
[config]
ignore_pr_authors = ["my-special-bot-user", ...]
```

Where the `ignore_pr_authors` is a regex list of usernames that you want to ignore.

!!! note
    There is one specific case where bots will receive an automatic response - when they generated a PR with a _failed test_. In that case, the [`ci_feedback`](https://qodo-merge-docs.qodo.ai/tools/ci_feedback/) tool will be invoked.

### Ignoring Generated Files by Language/Framework

To automatically exclude files generated by specific languages or frameworks, you can add the following to your `configuration.toml` file:

```
[config]
ignore_language_framework = ['protobuf', ...]
```

You can view the list of auto-generated file patterns in [`generated_code_ignore.toml`](https://github.com/qodo-ai/pr-agent/blob/main/pr_agent/settings/generated_code_ignore.toml). 
Files matching these glob patterns will be automatically excluded from PR Agent analysis.

### Ignoring Tickets with Specific Labels

When Qodo Merge analyzes tickets (JIRA, GitHub Issues, GitLab Issues, etc.) referenced in your PR, you may want to exclude tickets that have certain labels from the analysis. This is useful for filtering out tickets marked as "ignore-compliance", "skip-review", or other labels that indicate the ticket should not be considered during PR review.

To ignore tickets with specific labels, add the following to your `configuration.toml` file:

```toml
[config]
ignore_ticket_labels = ["ignore-compliance", "skip-review", "wont-fix"]
```

Where `ignore_ticket_labels` is a list of label names that should be ignored during ticket analysis.
