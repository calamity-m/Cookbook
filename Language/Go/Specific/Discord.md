
# Disgo

## Client

```go
client, err := disgo.New(
	cfg.BotToken,
	disgobot.WithGatewayConfigOpts(
		// set enabled intents
		gateway.WithIntents(
			gateway.IntentGuilds,
			gateway.IntentGuildMessages,
			gateway.IntentDirectMessages,
			gateway.IntentGuildInvites,
			gateway.IntentMessageContent,
		),
	),
	disgobot.WithLogger(logger),
)
if err != nil {
	return nil, err
}
```