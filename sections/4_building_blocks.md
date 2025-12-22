## 4. Building Blocks

This section is not documentation for Claude Code features, but rather a short overview in which I highlight specific Claude Code tools and aspects which are important for my use cases. You might find different ways to apply them for your scenarios.

### Simple prompt

It's the simplest tool. You can type `claude` in your working directory and then type any prompt, press Enter and wait for the result. Claude Code will try to perform the task you've just provided to it.

### CLAUDE.md file

https://platform.claude.com/docs/en/agent-sdk/modifying-system-prompts#method-1-claude-md-files-project-level-instructions

CLAUDE.md file is a plain but powerful feature of Claude Code. The idea is simple: CLAUDE.md content is always included in the context when Claude Code works with files in this directory. In CLAUDE.md, you can mention documentation files, source code files, or any other references Claude should consider. There's no special syntax—just write naturally, like: "See `docs/git-commit-format.md` when preparing Git commits." Claude Code will include referenced files only when relevant to the current task.

Also, it supports nested CLAUDE.md files. For example, when you have the following structure:

```
root
|_CLAUDE.md
|_dir1
  |_CLAUDE.md
  |_dir2
    |_CLAUDE.md
    |_MyComponent.tsx
```

When Claude Code reads MyComponent.tsx file - it will include `root/CLAUDE.md`, `root/dir1/CLAUDE.md`, `root/dir1/dir2/CLAUDE.md` files in the context.

This hierarchy lets you organize context by scope: project-wide conventions in the root CLAUDE.md, module-specific patterns in subdirectory files. Claude gets both general and specific context when working with module files.

### Subagents

https://code.claude.com/docs/en/sub-agents

The official doc says:

```
Custom subagents in Claude Code are specialized AI assistants that can be invoked to handle specific types of tasks. They enable more efficient problem-solving by providing task-specific configurations with customized system prompts, tools and a separate context window.
```

Each subagent has its own context, which helps reduce the context usage of the main conversation. Also, it has other features - like limiting tools available for this agent. In many guides on the Internet, you might find the description of the pipelines people build using subagents, like: Requirements Writer -> Requirements Reviewer -> Software Architect -> Backend Developer -> Frontend Engineer -> Code Reviewer -> etc.. 

Personally, I didn't find this approach to work well. It doesn't mean that it cannot be effective, but I found it's beneficial to have a single conversation with the feature context - which, of course, has its constraints - in context window size. Mostly, I'm using subagents for code reviewing. During the code review, the subagent runs with its own context (base process just provides some overview of what should be reviewed) and it's not biased by the decisions made during the source code implementation.

There are many public collections of Claude Code subagents like this, for example https://github.com/VoltAgent/awesome-claude-code-subagents.

However, I suggest building your own subagents (and other prompts) designed specifically for your project rather than using public ones. Generic prompts won't understand your codebase patterns and conventions. See the "It's all about context" section for more on why minimal, project-specific prompts work better.

Technically, a subagent is just a markdown file located in .claude/agents directory or its subdirectories. Claude Code uses it as a base for building the context in the child process.

Claude autonomously decides when to invoke a specific subagent (based on the subagent description). Also, you can explicitly ask Claude to use a specific subagent (`use backend-codereviewer subagent for this task`, and so on).

The following subagents are used on my project:

```
backend-code-reviewer
frontend-code-reviewer
mobile-code-reviewer
```

When I say "separate context," I mean that subagents run in their own context window, independent from your main conversation. The main process shares only what you explicitly pass to the subagent — it doesn't inherit the full conversation history. This keeps the subagent focused and unbiased by previous decisions, which is especially valuable for code review.

### MCP server

https://code.claude.com/docs/en/mcp

MCP is the protocol that allows you to connect with external sources, for example, with the task tracker, with the monitoring tool, Slack, etc.

While you can achieve the same result with a set of bash scripts and skills, MCP is the standard protocol, understandable by many LLMs.

Each MCP server provides a set of tools that Claude Code may use when it needs to.

Here is a specification of tools in the custom MCP server I implemented for getting the data from YouTrack (the task tracker we use on the project).

```typescript
{
  tools: [
      {
          name: 'get_issue',
          description: 'Get YouTrack issue details including title, description, and attached files',
          inputSchema: {
              type: 'object',
              properties: {
                  issueId: {
                      type: 'string',
                      description: 'The YouTrack issue ID (e.g., PROJECT-123)',
                  },
              },
              required: ['issueId'],
          },
      },
      {
          name: 'get_issue_comments',
          description: 'Get all comments for a YouTrack issue',
          inputSchema: {
              type: 'object',
              properties: {
                  issueId: {
                      type: 'string',
                      description: 'The YouTrack issue ID (e.g., PROJECT-123)',
                  },
              },
              required: ['issueId'],
          },
      },
      {
          name: 'get_attachment_content',
          description: 'Get the content of a specific attachment from a YouTrack issue',
          inputSchema: {
              type: 'object',
              properties: {
                  issueId: {
                      type: 'string',
                      description: 'The YouTrack issue ID (e.g., PROJECT-123)',
                  },
                  attachmentId: {
                      type: 'string',
                      description: 'The attachment ID to retrieve',
                  },
              },
              required: ['issueId', 'attachmentId'],
          },
      },
      {
          name: 'get_comment_attachment_content',
          description: 'Get the content of a specific attachment from a YouTrack issue comment',
          inputSchema: {
              type: 'object',
              properties: {
                  issueId: {
                      type: 'string',
                      description: 'The YouTrack issue ID (e.g., PROJECT-123)',
                  },
                  commentId: {
                      type: 'string',
                      description: 'The comment ID',
                  },
                  attachmentId: {
                      type: 'string',
                      description: 'The attachment ID to retrieve',
                  },
              },
              required: ['issueId', 'commentId', 'attachmentId'],
          },
      },
  ],
}
```

So, when Claude Code has a YouTrack issue number, it can get the issue description using `get_issue` tool and all comments using `get_issue_comments`, when there are attachments, it'll read them using `get_attachment_content` or `get_comment_attachment_content` tools.

This approach allows you to not copy-paste requirements and conversation, but just reference your task tracking / log monitoring tool for getting relevant details.

There are already many published official MCP servers, if an official MCP server is not available, I prefer to write my own (using Claude Code, you don't even need to study the MCP specification for it).

### Skills

https://code.claude.com/docs/en/skills

This is a relatively new tool introduced in Claude Code in the middle of October 2025.
It's just a different dimension of presenting the project knowledge in comparison with subagents and CLAUDE.md files. According to Claude Code documentation, skills are expected to be the description of atomic operations/updates you do on the project, like `implementing-unit-test`, `implementing-controller`, `creating-database-migration`.

Unlike subagents, skills are used in the same conversation from which they are called (so skills use the same context as the main process). Unlike commands, skills are invoked by Claude Code autonomously (and you may ask to use a skill explicitly). Within the skill diretory, you can develop a custom bash/python/etc script that will perform specific tasks and skill will have knowledge how to utilise these bash (or Python, or in any other language) scripts.

You can decompose the patterns you use and the approaches applied to your project into small pieces and describe each as a separate skill.

For example, when we speak about a REST web application. To implement a single API method, Claude Code should understand the following:
* How controllers are implemented (how routes are set up, how HTTP request is mapped to request DTO)
* How user authentication is done (how to get the actual user id and data, etc.)
* How database transactions are configured for requests
* How domain services are implemented
* How domain entities are implemented
* How data is persisted
* Events and subscriptions functionality if these patterns are used

When this information is not provided via skill, Claude Code will try to understand patterns just based on the existing source code and documentation—and in most scenarios, it does this successfully. Skills offer a more structured way to encode your patterns.

Note: Earlier, I experienced skills being invoked rarely—Claude Code seemed to use SKILL.md content without explicitly indicating the skill was running - while it seem that Claude Code actually reads SKILL.md content but just didn't indicate that skill is run. This has improved in recent versions. I'd suggest experimenting with skills on your projects—along with Plugins (https://code.claude.com/docs/en/plugins), they can be a good way to share patterns within your organization.

### Slash commands

https://code.claude.com/docs/en/slash-commands

Claude Code provides a set of standard slash commands (like `/clear` for clearing the existing context, `/exit` etc).

Also, you can implement your custom slash commands, which are just shortcuts for your prompts.

Instead of typing:
```
Commit all not committed frontend changes.

See backend/doc/git-commit-format.md for reference.
```

and other (possibly, much longer prompts) you might just type: `/commit:backend`
