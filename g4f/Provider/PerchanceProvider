# providers/PerchanceProvider.py

from __future__ import annotations
from typing import Optional
import perchance
from io import BytesIO
import aiohttp
from .helper import format_image_prompt  # Assuming you have a helper function for formatting
from ..typing import AsyncResult, Messages  # Adjust import paths as necessary

class PerchanceProvider:
    label = "Perchance"
    default_model = "ai-image-generator"  # Default model for image generation
    default_text_model = "ai-text-generator"  # Default model for text generation
    image_models = [default_model]  # Default models
    text_models = [default_text_model]  # Default text models
    _models_loaded = False  # Flag for model loading

    @classmethod
    def get_models(cls, **kwargs):
        if not cls._models_loaded:
            # Load models if not already loaded
            cls._models_loaded = True
        return cls.image_models, cls.text_models

    @classmethod
    async def create_async_generator(
        cls,
        model: str,
        messages: Messages,
        prompt: str = None,
        aspect_ratio: str = "1:1",
        width: int = None,
        height: int = None,
        seed: Optional[int] = None,
        n: int = 1,
        **kwargs
    ) -> AsyncResult:
        """Generate images asynchronously using Perchance."""
        cls.get_models()  # Ensure models are loaded
        async for chunk in cls._generate_image(
            model=model,
            prompt=format_image_prompt(messages, prompt),
            aspect_ratio=aspect_ratio,
            width=width,
            height=height,
            seed=seed,
            n=n,
            **kwargs
        ):
            yield chunk

    @classmethod
    async def _generate_image(
        cls,
        model: str,
        prompt: str,
        aspect_ratio: str,
        width: int,
        height: int,
        seed: Optional[int],
        n: int,
        **kwargs
    ) -> AsyncResult:
        """Internal method to generate images."""
        try:
            # Initialize the Perchance image generator
            image_gen = perchance.Generator(model)
            result = await image_gen.generate(prompt=prompt, width=width, height=height)

            # Download image data
            async with aiohttp.ClientSession() as session:
                async with session.get(result["url"]) as response:
                    image_data = await response.read()

            yield {
                "success": True,
                "image": BytesIO(image_data),
                "mime_type": "image/png"
            }
        except Exception as e:
            yield {
                "success": False,
                "error": str(e)
            }

    @classmethod
    async def generate_text(cls, prompt: str, **kwargs) -> AsyncResult:
        """Generate text using Perchance's AI Text Generator."""
        try:
            text_gen = perchance.Generator(cls.default_text_model)
            result = await text_gen.generate(prompt=prompt, **kwargs)
            return {
                "success": True,
                "text": result["text"],
                "details": result
            }
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }
