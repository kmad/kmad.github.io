---
title: Using DSPy to Detect Document Boundaries
---

DSPy is becoming increasingly popular (at least in my bubble on [X](https://x.com/kmad)) - and, in my opinion, for good reason! It provides a sense of control and composability that becomes addictive once you get a few reps and understand how it all fits together. It allows you to inject LLMs into your program's control flow, providing useful leverage. 

Today we'll walk through a super simple application to solve a real-world problem with DSPy and demonstrate a few of its useful capabilities, though we're just scratching the surface here:

-   The ReAct module for tool calling and reasoning
-   "Model-mixing" - using faster models for classification and smarter models for decision making
-   DSPy.Image for multimodal processing
-   Async processing for better performance


## What we're solving for

Document processing workflows often involve complex, multi-section files where identifying logical boundaries between different components is important. Whether you're dealing with contracts that contain exhibits, reports with appendices, or order forms with attached terms and conditions, knowing where one section ends and another begins is key to improving downstream processing accuracy.

While classifying single pages or multiple pages is straightforward for vision-language models today, how you use the models matters - the real world is messy. For example, a 15-page PDF might contain a 5-page main document, a 3-page appendix, and a 7-page exhibit - and treating them as a single unit could lead to poor extraction results downstream.

One potential approach is shown below: first classify individual pages, then use those classifications along with the actual content to determine document boundaries.

### LM Setup
Modularity and configurability are useful for keeping this organized and experimenting with different models. Let's start by defining our environment configuration with two LLMs:
```bash
`# Model configuration 
DSPY_ENDPOINT=https://api.openai.com/v1
DSPY_API_KEY=your_api_key_here 

DSPY_FAST_MODEL=openai/gpt-4.1-mini
DSPY_FAST_API_VERSION=2025-04-14
DSPY_FAST_MAX_TOKENS=10_000
DSPY_FAST_TEMPERATURE=0.1

DSPY_SMART_MODEL=openai/gpt-4.1
DSPY_SMART_API_VERSION=2025-04-14
DSPY_SMART_MAX_TOKENS=25_000
DSPY_SMART_TEMPERATURE=0.2
```

Then we can easily pull this into our DSPy LM configuration like so:
```python
load_dotenv()
LM_CONFIG  = {
	"model": os.getenv("DSPY_FAST_MODEL"),
	"api_key": os.getenv("DSPY_API_KEY"),
	"api_base": os.getenv("DSPY_ENDPOINT"),
	"api_version": os.getenv("DSPY_FAST_API_VERSION"),
	"max_tokens": int(os.getenv("DSPY_FAST_MAX_TOKENS",  50_000)),
	"temperature": float(os.getenv("DSPY_FAST_TEMPERATURE",  1.0)),
	"cache": True,
}
LM_CONFIG_SMART  = {
	"model": os.getenv("DSPY_SMART_MODEL"),
	"api_key": os.getenv("DSPY_API_KEY"),
	"api_base": os.getenv("DSPY_ENDPOINT"),
	"api_version": os.getenv("DSPY_SMART_API_VERSION"),
	"max_tokens": int(os.getenv("DSPY_SMART_MAX_TOKENS")),
	"temperature": float(os.getenv("DSPY_SMART_TEMPERATURE")),
	"cache": True,
}

lm = dspy.LM(**LM_CONFIG)
dspy.configure(lm=lm) # Set as global default
```
####  Document Modeling
Our objective is to classify each page of a document, then use those classifications to reason about the overall document structure. (*Note: The references to `self` below are because this is implemented as a class for easier state management.*)


There are at least two approaches to classification:
- Predefined classes: Define specific categories like `COVER_PAGE`, `TERMS_AND_CONDITIONS`, `SIGNATURE_PAGE`
- Open-ended classification: Let the model determine appropriate categories

There's an important tradeoff: predefined classes make reasoning easier and provide more predictable outputs, but they assume prior knowledge about document types. Open-ended classification is more flexible but can lead to inconsistent categorizations that are harder to reason about programmatically.

For production systems processing diverse document types, you might consider a hybrid approach: start with predefined classes for common patterns, but allow the model to suggest new categories when needed.

For this demonstration, we'll use predefined classes:
```python
CLASSES  =  (
	"COVER PAGE",
	"TERMS_AND_CONDITIONS",
	"SIGNATURE_PAGE",
	"SCHEDULE OR TABLE",
	"START_OF_APPENDIX",
	"START_OF_EXHIBIT"
)
```

Next let's define our simple DSPy Signature.
```python
class ClassifyPage(dspy.Signature):
	"""
	Classifies a single page from a PDF order form into one of several predefined classes.
	"""
	page_image: dspy.Image =  dspy.InputField(
	desc="An image of a single page from the PDF."
	)
	page_class  =  dspy.OutputField(desc="The type or class of the page.")
```

It's as simple as a classifier can get: page image in, classification out. We can add our class `Literals` when we use the signature like so:
```python
signature = ClassifyPage.with_updated_fields("page_class", type_=Literal[tuple(CLASSES)])
```

Now we need the images to classify. PyMuPDF (or similar libraries) make this super easy. We'll do this by creating an array of images, one for each page of the PDF. (*Note: We use PyMuPDF here, but do be mindful of the AGPL license requirements if you do move to production.*)
```python
def convert_to_img(self, data: bytes, pages: int = -1) -> List[dspy.Image]:
    """Convert PDF data to base64 encoded images."""
    pdf_file = io.BytesIO(data)
    pdf_reader = pymupdf.open(stream=pdf_file, filetype="pdf")
    images = []
    max_pages = pages if pages > 0 else pdf_reader.page_count
    for page_num in range(max_pages):
        page = pdf_reader.load_page(page_num)
        pix = page.get_pixmap()
        img_data = pix.tobytes("png")
        
        images.append(dspy.Image.from_PIL(f"data:image/png;base64,{base64.b64encode(img_data).decode('utf-8')}"))
    return images
```

The output of `convert_to_img` is an array of DSPy-native image objects, abstracting away the actual interaction with the underlying model. This saved as part of a class object for later retrieval.

We can also define a simple function which can be used as a tool call later on to retrieve particular pages:
```python
def get_page_images(self, pages: list[int]) -> List[dspy.Image]:
"""Get the page images. Be mindful of context length restrictions and don't return more than 8 images at a time."""
	return  [self.page_images[i] for i in  pages]
```

Now we simply iterate over this array and classify each image. We can do this async to speed things up a bit (be mindful of any rate limits!); because we defined the default LM above, this will use `DSPY_FAST_MODEL`.

```python
classifier = dspy.Predict(classify_signature)

# Process all concurrently
async def classify_page(i: int, img: str):
    result = await classifier.acall(page_image=img)
    return i, result.page_class

# Use asyncio.gather to process all pages concurrently
tasks = [classify_page(i, img) for i, img in enumerate(self.page_images)]
results = await asyncio.gather(*tasks)

self.page_classifications = results # Save for later use
```


The result is saved as `self.page_classifications` which is a `dict` mapping of `page number -> class`.  Pretty easy.

Now let's use this output to determine the document boundaries. The thinking here is, if you see something like `[COVER_PAGE, TERMS_AND_CONDITIONS, SIGNATURE_PAGE, START_OF_APPENDIX, TERMS_AND_CONDITIONS, TERMS_AND_CONDITIONS]` the boundary detection becomes relatively intuitive: pages 1 - 3 are the document itself, and 4 - 6 comprise the Appendix. This is useful because you can now treat these pieces differently (say, applying a specific extraction workflow for the main component downstream, and ignoring the Appendix).

## Where DSPy shines

Classification was easy, but what's powerful is how we can use the output in the rest of our program, namely exposing the contents of the classification and the source material to the LLM so that it can *reason over the contents* to determine an appropriate boundary...

... and we can do it in **two lines of code**!
```python
boundary_detector = dspy.Signature(
			"pages_and_classifications -> document_boundaries: dict[str, tuple[int, int]]"
		).with_instructions(
			"Detect boundaries between documents, such as order forms or agreements. A typical order form has a header, details, and signature page. If you see what looks like a single document, return the start and end page of the whole document. Otherwise, consider where you see document headers to inform boundaries of one or more documents."
		)

detector = dspy.ReAct(
	boundary_detector, tools=[self.get_page_images], max_iters=10
)
```

These two lines of code pack a huge punch:
1. **Schema Definition**: We use the DSPy shorthand for a signature to take in `pages_and_classifications` and specify a moderately complex output type of `dict[str, tuple[int, int]]` to represent our mapping, allowing the LLM to come up with a name for each document section/boundary.
2. **LLM Instructions**: We inline instructions for this LLM call, providing some context and guardrails for how it should think about the contents.
3. **Tool calling**: We then define the `detector` which uses DSPy's `ReAct` module, allowing the LLM to use tool calling. Here we expose `get_page_images` which, as defined above, allows the model to retrieve specific source pages.
4. **Iterative Reasoning**: The LLM can make multiple tool calls and refine its understanding 

We can then call the detector with the "smart" model, using a temporary context to upscale to the "smart" model:

```python
lm_smart = dspy.LM(**LM_CONFIG_SMART)
with dspy.context(lm=lm_smart):
	response = await detector.acall(pages_and_classifications=self.page_classifications)
```

That's it! The final output is accessible by `response.document_boundaries` which is now a neatly organized dictionary representing our document.

## Real World Example
Let's use this [random purchasing contract](https://www.mercycorps.org/sites/default/files/2022-03/Attachment%203-%20Purchase%20Contract%20Sample.pdf) I found online for our test.

Running it through the above, we get an initial output like so:
```python
(0, 'TERMS_AND_CONDITIONS')
(1, 'TERMS_AND_CONDITIONS')
(2, 'TERMS_AND_CONDITIONS')
(3, 'TERMS_AND_CONDITIONS')
(4, 'TERMS_AND_CONDITIONS')
(5, 'TERMS_AND_CONDITIONS')
(6, 'SIGNATURE_PAGE')
(7, 'SCHEDULE OR TABLE')
(8, 'TERMS_AND_CONDITIONS')
(9, 'SCHEDULE OR TABLE')
(10, 'SCHEDULE OR TABLE')
(11, 'TERMS_AND_CONDITIONS')
(12, 'TERMS_AND_CONDITIONS')
(13, 'TERMS_AND_CONDITIONS')
```

Feeding this output into the ReAct step we get the following output:

 ```python
 document_boundaries = 
	 {
		 'Main Agreement': (0, 5),
		 'Signature Page': (6, 6),
		 'Schedules/Appendices': (7, 13)
	 }
 ```

I always find it interesting to look at the reasoning "trajectories" as well, to see (1) how the model is using the tools (is it looking at the right pages?) and (2) how it then reasons about the results:
```python
trajectory={
'thought_0': 'The document appears to start with a block of TERMS_AND_CONDITIONS (pages 0-5), followed by a SIGNATURE_PAGE (page 6). After that, there are SCHEDULE OR TABLE pages (pages 7, 9, 10) interspersed with TERMS_AND_CONDITIONS (pages 8, 11-13). To confirm document boundaries, especially whether the SCHEDULE OR TABLE pages are appendices or separate documents, I should review the images for pages 6-13 (the transition from signature to schedules and back to terms). I will request images for pages 6-13 (8 pages, within the limit).',
	'tool_name_0': 'get_page_images',  'tool_args_0': {'pages': [6,  7,  8,  9,  10,  11,  12,  13]},
	'observation_0': [Image(url=data:image/png;base64,<IMAGE_BASE_64_ENCODED(39332)>),  Image(url=data:image/png;base64,<IMAGE_BASE_64_ENCODED(87628)>),  Image(url=data:image/png;base64,<IMAGE_BASE_64_ENCODED(35688)>),  Image(url=data:image/png;base64,<IMAGE_BASE_64_ENCODED(39536)>),  Image(url=data:image/png;base64,<IMAGE_BASE_64_ENCODED(148756)>),  Image(url=data:image/png;base64,<IMAGE_BASE_64_ENCODED(139628)>),  Image(url=data:image/png;base64,<IMAGE_BASE_64_ENCODED(149196)>),  Image(url=data:image/png;base64,<IMAGE_BASE_64_ENCODED(125284)>)],
'thought_1': 'The images confirm that after the signature page (page 6), there are three schedules: Schedule I (pages 7-8), Schedule II (page 9), and Schedule III (pages 10-13). These schedules are part of the same contract, referenced as appendices or schedules within the main agreement. There is no indication of a new document or agreement starting after the signature page; rather, the schedules are explicitly referenced as part of the contract. Therefore, the document boundaries are: (1) Main Agreement (pages 0-5), (2) Signature Page (page 6), (3) Schedules/Appendices (pages 7-13). No further information is needed.',
	'tool_name_1': 'finish',  'tool_args_1': {},
	'observation_1': 'Completed.'},

reasoning='The review of the page classifications and images confirms that the document is a single contract package. The main body of the contract (TERMS_AND_CONDITIONS) spans pages 0-5, followed by a SIGNATURE_PAGE on page 6. After the signature, there are three schedules: Schedule I (pages 7-8), Schedule II (page 9), and Schedule III (pages 10-13). These schedules are referenced within the main contract and are clearly labeled as appendices or schedules, indicating they are integral parts of the same agreement rather than separate documents. There is no evidence of a new agreement or order form beginning after the signature page. Thus, the boundaries are: (1) Main Agreement (pages 0-5), (2) Signature Page (page 6), (3) Schedules/Appendices (pages 7-13).',

document_boundaries={'Main Agreement': (0,  5),  'Signature Page': (6,  6),  'Schedules/Appendices': (7,  13)}
```

You can clearly see the model using tool calls to explore the boundaries of the document to confirm its understanding of the overall structure, then use that output to reason about the final answer.


You can find the final code [here](https://gist.github.com/kmad/76814081edd55f9cc03e612f4d9731b5), which also takes advantage of `uv`'s inline script definition, meaning you just need to define the `.env` file, and run like so:
```bash
uv run detect_boundaries.py <path_to_pdf>
```

## Final Thoughts

DSPy's primitives make it easy to experiment with different models, adjust prompts, and add new capabilities without rewriting core logic. 

There's of course a ton of low-hanging fruit here - better classes, smarter descriptions, optimizing prompts, etc. But hopefully this serves as a simple example of the type of things that are possible today with ~50 lines of code, an LLM, and an idea.
 
