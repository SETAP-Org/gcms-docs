Google Gemini
-------------

GCMS uses the Google Gemini API to power the in-app AI assistant. The
assistant helps students understand their coursework specifications,
summarise documents, and answer questions about their project.

Model
~~~~~

The default model is ``gemini-2.5-flash``, chosen for its low cost and
fast response times. This is suitable for typical student Q&A and
document summarisation. The model can be swapped to ``gemini-2.5-pro``
for higher quality at higher cost.

File attachments
~~~~~~~~~~~~~~~~

When a user asks the assistant a question, GCMS can attach files from
the project's shared folder so the model can reason about their
contents. Files are handled in one of two ways depending on type:

**Sent natively to Gemini:**

- PDF (``application/pdf``)
- Plain text (``text/plain``)
- Markdown (``text/markdown``)
- CSV (``text/csv``)
- HTML (``text/html``)

**Extracted to plain text on the server before sending:**

- Word documents (``.docx``) — extracted with ``mammoth``
- Excel spreadsheets (``.xlsx``, ``.xls``) — extracted with ``xlsx``
- PowerPoint presentations (``.pptx``) — extracted with ``officeparser``

For Office formats, GCMS extracts the readable text content server-side
and forwards it to Gemini as ``text/plain``. This avoids the need for
native Office support in the model and keeps request sizes small.

A total budget of 15 MB is enforced across all attachments in a single
request. Files exceeding the budget or of unsupported types are
silently skipped, and the model is informed of any omissions so it
can reference this in its response.

System instruction
~~~~~~~~~~~~~~~~~~

Every Gemini request includes a system instruction that defines the
assistant's persona and behavioural guidelines. The assistant is
configured to:

- Keep responses concise
- Admit when it does not know an answer rather than inventing one
- Explain, summarise, and clarify rather than complete the coursework
  on the student's behalf

Configuration
~~~~~~~~~~~~~

The Gemini API key is stored in ``.env.gemini`` as ``GEMINI_API_KEY``.
The client is lazily initialised on the first request, so importing
``utils/gemini.js`` does not require the key to be set at startup.