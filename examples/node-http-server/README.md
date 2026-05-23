# Node.js HTTP Server + AI SDK Example

You can use the AI SDK in a plain Node.js HTTP server to stream model output and custom data without a framework.

This example demonstrates:

- Streaming model responses from a basic Node.js server
- Returning UI message streams over raw HTTP responses
- Merging custom events with model output (`/stream-data`)
- Using AI SDK stream helpers without React or Next.js

## Usage

1. Create a `.env` file in `examples/node-http-server`:

```sh
OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
```

2. Run the following commands from the root directory of the AI SDK repo:

```sh
pnpm install
pnpm build
```

3. Run the following command from the `examples/node-http-server` directory:

```sh
pnpm dev
```

4. Test the server endpoints:

```sh
curl -N http://localhost:8080/
curl -N http://localhost:8080/stream-data
```

## What to expect

- The server listens on `http://localhost:8080`.
- Requesting `/` streams a model response for the holiday prompt.
- Requesting `/stream-data` first emits custom data, then merges in the model stream.
- Errors are masked by default; the example shows where to customize exposed error messages.

## Code examples

Basic streaming response (`/`):

```ts
import { openai } from '@ai-sdk/openai';
import { createServer } from 'http';
import { streamText } from 'ai';

createServer(async (_req, res) => {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  result.pipeUIMessageStreamToResponse(res);
}).listen(8080);
```

Merging custom data with model output (`/stream-data`):

```ts
import { createUIMessageStream, pipeUIMessageStreamToResponse, streamText } from 'ai';

const stream = createUIMessageStream({
  execute: ({ writer }) => {
    writer.write({ type: 'start' });
    writer.write({
      type: 'data-custom',
      data: { custom: 'Hello, world!' },
    });

    const result = streamText({
      model: openai('gpt-4o'),
      prompt: 'Invent a new holiday and describe its traditions.',
    });

    writer.merge(result.toUIMessageStream({ sendStart: false }));
  },
});

pipeUIMessageStreamToResponse({ stream, response: res });
```

## Notes

- This is a minimal example focused on HTTP streaming primitives.
- It uses `@ai-sdk/openai` with `openai('gpt-4o')` by default.
- You can adapt the same pattern to any AI SDK provider package.
