Use streamlit to add UI interface for Zerox OCR.

![alt text](https://github.com/amenghhs/zerox-streamlit/blob/main/image.png)

Usage:
```
streamlit run app.py
```

Environment variables configuration modify `.env`.

```
GEMINI_API_KEY=
OPENAI_API_KEY=
AZURE_API_KEY=
ANTHROPIC_API_KEY=
VERTEX_CREDENTIALS=
VERTEXAI_PROJECT=
VERTEXAI_LOCATION=
```
## Zerox OCR

A dead simple way of OCR-ing a document for AI ingestion. Documents are meant to be a visual representation after all. With weird layouts, tables, charts, etc. The vision models just make sense!

The general logic:

- Pass in a file (pdf, docx, image, etc.)
- Convert that file into a series of images
- Pass each image to GPT and ask nicely for Markdown
- Aggregate the responses and return Markdown

## Credits

- [Litellm](https://github.com/BerriAI/litellm): <https://github.com/BerriAI/litellm> | This powers our python sdk to support all popular vision models from different providers.
- [Streamlit](https://github.com/streamlit/streamlit): <https://github.com/streamlit/streamlit> | Streamlit — A faster way to build and share data apps.

### License

This project is licensed under the MIT License.
