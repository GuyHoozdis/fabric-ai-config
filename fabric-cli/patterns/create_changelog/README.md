# Create Changelog

Generate a concise bulletted list of changes in functionality based on a provided git diff.

> [!NOTE]
> This pattern does not do Conventional Commit style output.

This pattern comes from a stand alone utility I wrote to generate the content that I would put with
version tags on repo's `main` branch.


## Usage

Identify two commits, for example two tags, and run the following command:
```bash
$ git diff <older-commit>...<newer-commit> -- . ':(exclude)uv.lock' \
    | fabric -p create_changelog
```


> [!NOTE]
>  The output will vary depending on the model used.  I've primarily used gpt-4.1 via the LLM
>  Service when I was using this prompt as a stand alone utility.

Specify a model
```bash
$ git diff <older-commit>...<newer-commit> -- . ':(exclude)uv.lock' \
    | fabric -p create_changelog --model gpt-4.1 \
    | tee newer-commit.changelog
```

A patch file works just as well:
```bash
$ cat changes.patch | fabric -p create_changelog 
```


## Example

!!!: Still working on / experimenting with this pattern.
- gpt-4o works well, but other models not so much.
- For this same `example.patch`, using `qwen3-coder` created a cross between a summary of changes
  write-up and a code review.  Using `qwen3` didn't do much better.
- I need a way to handle scenarios where the diff itself is too big for the model context.


### Input:
```bash
$ cat example.patch \ 
    | fabric --pattern create_changelog --model azure-4o
This release includes bug fixes and performance improvements.

- Added new test cases for conversation flow handling and conversation state management.
- Enhanced the conversation flow to generate form groups only when inputs are provided.
- Updated environment test cases to handle missing and empty sections more gracefully.
- Introduced a utility function to verify JSON data against a Pydantic schema.
- Set environment variable for critical log level in tests to suppress unnecessary output.
```
