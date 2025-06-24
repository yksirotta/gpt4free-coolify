## G4F - Legacy API

#### ChatCompletion

```python
import g4f

g4f.debug.logging = True  # Enable debug logging
g4f.debug.version_check = False  # Disable automatic version checking
print(g4f.Provider.Gemini.params)  # Print supported args for Gemini

# Using automatic a provider for the given model
## Streamed completion
response = g4f.ChatCompletion.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
    stream=True,
)
for message in response:
    print(message, flush=True, end='')

## Normal response
response = g4f.ChatCompletion.create(
    model=g4f.models.gpt_4,
    messages=[{"role": "user", "content": "Hello"}],
)  # Alternative model setting

print(response)
```

##### Providers

```python
import g4f

# Print all available providers
print([
    provider.__name__
    for provider in g4f.Provider.__providers__
    if provider.working
])

# Execute with a specific provider
response = g4f.ChatCompletion.create(
    model="gpt-4o",
    provider=g4f.Provider.Copilot,
    messages=[{"role": "user", "content": "Hello"}],
    stream=True,
)
for message in response:
    print(message)
```


##### Image Upload & Generation

Image upload and generation are supported by three main providers:

- **Microsoft Copilot & Other GPT-4 Providers:** Utilizes Microsoft's Image Creator.
- **Google Gemini:** Available for free accounts with IP addresses outside Europe.
- **OpenaiChat with GPT-4:** Accessible for users with a Plus subscription.

```python
import g4f

# Setting up the request for image creation
response = g4f.ChatCompletion.create(
    model=g4f.models.default, # Using the default model
    provider=g4f.Provider.Gemini, # Specifying the provider as Gemini
    messages=[{"role": "user", "content": "Create an image like this"}],
    image=open("images/g4f.png", "rb"), # Image input can be a data URI, bytes, PIL Image, or IO object
    image_name="g4f.png" # Optional: specifying the filename
)

# Displaying the response
print(response)

from g4f.image import ImageResponse

# Get image links from response
for chunk in g4f.ChatCompletion.create(
    model=g4f.models.default, # Using the default model
    provider=g4f.Provider.OpenaiChat, # Specifying the provider as OpenaiChat
    messages=[{"role": "user", "content": "Create images with dogs"}],
    stream=True,
    ignore_stream=True
):
    if isinstance(chunk, ImageResponse):
        print(chunk.urls) # Print generated image links
        print(chunk.alt) # Print used prompt for image generation
```

##### Async Support

To enhance speed and overall performance, execute providers asynchronously. The total execution time will be determined by the duration of the slowest provider's execution.

```python
import g4f
import asyncio

_providers = [
    g4f.Provider.Copilot,
    g4f.Provider.PollinationsAI,
    g4f.Provider.Gemini,
]

async def run_provider(provider: g4f.Provider.BaseProvider):
    try:
        response = await g4f.ChatCompletion.create_async(
            model=g4f.models.default,
            messages=[{"role": "user", "content": "Hello"}],
            provider=provider,
        )
        print(f"{provider.__name__}:", response)
    except Exception as e:
        print(f"{provider.__name__}:", e)
        
async def run_all():
    calls = [
        run_provider(provider) for provider in _providers
    ]
    await asyncio.gather(*calls)

asyncio.run(run_all())
```

##### Proxy and Timeout Support

All providers support specifying a proxy and increasing timeout in the create functions.

```python
import g4f

response = g4f.ChatCompletion.create(
    model=g4f.models.default,
    messages=[{"role": "user", "content": "Hello"}],
    proxy="http://host:port",
    # or socks5://user:pass@host:port
    timeout=120,  # in secs
)

print(f"Result:", response)
```

---

[Return to Documentation](README.md)