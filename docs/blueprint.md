# Task Assistant Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot that acts as an all-purpose task assistant for a single user, providing GPT-style responses across domains like coding, math, architecture, and taxes. It supports free-form prompts, conversation context retention, prompt templates, file attachments, and settings customization.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- single user (owner)

## Success criteria

- User receives accurate, context-aware responses to their prompts
- User can customize settings like tone, language, and verbosity
- User can save and reuse prompt templates
- User can send and have files processed in context
- User can export conversation history as needed

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and show brief help with current settings
- **/templates** (command, actor: user, command: /templates) — List available prompt templates
- **/save_template** (command, actor: user, command: /save_template) — Save current prompt as a template
- **/use_template** (command, actor: user, command: /use_template) — Run a saved template by name
- **/settings** (command, actor: user, command: /settings) — Change tone, language, and verbosity settings
- **/export** (command, actor: user, command: /export) — Request session export as a transcript file

## Flows

### Single-turn ask
_Trigger:_ user message

1. User sends a prompt
2. Bot analyzes prompt and determines if clarification is needed
3. If clarification needed, bot asks one short question
4. Bot provides answer with context and optional follow-up

_Data touched:_ conversation session, user profile

### Multi-turn session
_Trigger:_ user message

1. User sends a message
2. Bot maintains context from recent history
3. Bot provides coherent response based on context
4. Conversation continues naturally

_Data touched:_ conversation session

### Prompt template management
_Trigger:_ /templates or /save_template or /use_template

1. User lists templates
2. User saves current prompt as template
3. User selects and runs a template

_Data touched:_ prompt templates

### File handling
_Trigger:_ user sends file

1. User sends file
2. Bot acknowledges receipt
3. Bot processes file content when relevant to context

_Data touched:_ files/attachments

### Settings adjustment
_Trigger:_ /settings

1. User requests settings change
2. Bot presents available options
3. User selects new tone, language, or verbosity level

_Data touched:_ user profile

### Export session
_Trigger:_ /export

1. User requests export
2. Bot generates transcript file
3. Bot sends .txt file to user
4. If enabled, user provides email and receives transcript via email

_Data touched:_ conversation session

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — Single owner's account and preferences
  - fields: language, tone, verbosity level
- **Conversation session** _(retention: persistent)_ — Messages exchanged between user and bot with context
  - fields: message role, timestamp, tags (topic, priority)
- **Prompt templates** _(retention: persistent)_ — Saved prompts for reuse
  - fields: template name, prompt text
- **Files/attachments** _(retention: session)_ — User-sent files for context
  - fields: file ID, content, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging and file handling
- **Email** (optional) — Optional session export delivery
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Set language (default: Russian)
- Adjust tone (concise/detailed)
- Change verbosity level
- Save and manage prompt templates
- Request session exports
- Enable/disable email notifications for exports

## Notifications

- Session transcript file sent to user
- Optional email notification when export is ready

## Permissions & privacy

- Only the owner can interact with the bot
- Files are stored temporarily and purged after 30 days
- Conversation history is retained for context but not shared
- Exported transcripts contain only the owner's conversation data

## Edge cases

- User sends ambiguous prompt requiring clarification
- User sends unsupported file type
- User requests export when no session history exists
- User tries to use a non-existent template
- User changes language mid-conversation
- User enables email notifications without providing an address

## Required tests

- Verify single-turn ask flow with and without clarification
- Test multi-turn conversation context retention
- Validate prompt template save/load functionality
- Confirm file handling and content processing
- Test settings changes affect subsequent responses
- Verify session export generation and delivery
- Test edge cases like invalid commands and missing templates

## Assumptions

- User will provide clear instructions when clarification is requested
- User will manage template names to avoid conflicts
- User will provide valid email address when enabling email notifications
- User will understand disclaimers for legal/financial advice
