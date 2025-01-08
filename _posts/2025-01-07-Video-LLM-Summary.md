# Summarizing video transcripts with an LLM


#### Tools Used:
- [bat](https://github.com/sharkdp/bat)
- [ffmpeg](https://ffmpeg.org)
- [llm](https://github.com/simonw/llm)*
- [mlx_whisper](https://github.com/ml-explore/mlx-examples/tree/main/whisper)
- [pdftotext](https://github.com/jalan/pdftotext)
- [shot-scraper](https://github.com/simonw/shot-scraper)*
- [uvx](https://docs.astral.sh/uv/)

<sub>*Big thanks to [@simonw](https://x.com/simonw) for an unending flow of useful tools which makes life easier every day.</sub>

---


Today I had the task of reviewing a series of video files and compare them to a legal filing (which came as one large PDF).

The videos were longform interviews, so to speed up the review process I wanted to get these into a workable format depending on workflow: video, audio, and text.

First, in the directory with the videos, I converted `.webm` files to mp3 with the following:

```bash
for file in *.webm; 
    do ffmpeg -i "$file" "${file%.webm}.mp3"; 
done
```

I then wanted to get the transcript from each so I could pipe the results into an LLM for analysis using [@simonw](https://x.com/simonw)'s [llm](https://github.com/simonw/llm) tool. Sure, you could use `yt-dlp` or similar to get the Youtube-generated transcript, but usually those aren't so great. Plus, I wanted to try out some of the work from the [MLX Community](https://huggingface.co/mlx-community)

`mlx_whisper` does a *really* fast job of this on my Macbook Pro. Since Simon is always beaming about `uv` and its utility `uvx` I gave it a go to get whisper into my cli.


I did the following for each `.mp3` file in the directory:
```bash
uvx --from mlx-whisper mlx_whisper video1.mp3
```

This left me with a bunch of `.txt` files at the same path as the `.mp3`s. I now had a format of each:
- `video1.webm`
- `video1.mp3`
- `video1.txt`

This was a quick way to get things going; ultimately I wanted to use an llm for analysis. I had a legal filing I wanted to compare these to also. To get the PDF into a workable format I did a simple:

```bash 
pdftotext legal_filing.pdf
```

That gave me a `legal_filing.txt` file to work with.

Now I could pipe these into `llm` however I needed. Ultimately I went with something like the following (note this uses `fish` syntax):
```bash
echo -en "$(cat legal_filing.txt) \n\n##### START OF TRANSCRIPTS ####\n\n $(cat video*.txt)" \
| llm -s "You will be provided the text of a legal document, as well as a series of transcripts from related interviews. Provide an analysis and comparison." \ 
| tee analysis_all.md
| bat -l markdown
```

The `-s` flag specifies the system prompt for the model (in this case, a very generic one); here the 'user' message is the `legal_filing.txt` document, with some custom delimeter I added, then the entire contents of all video `.txt` transcripts in the directory. I then `tee` it so I can review the results as they're generated but also save it to a file. `bat` is a nice bonus just to view some aestetic formatting in the terminal.

Finally, we also needed to crawl a related website to get background information. I used Claude to whip up the following script:

```python
# spider.py

import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin, urlparse
from collections import deque

def spider_website(start_url):
    # Parse the domain from the start URL
    domain = urlparse(start_url).netloc
    
    # Initialize our queues and sets
    queue = deque([start_url])
    discovered_urls = {start_url}
    
    while queue:
        current_url = queue.popleft()
        print(f"Crawling: {current_url}")
        
        try:
            # Get the webpage content
            response = requests.get(current_url, timeout=5, verify=False)
            soup = BeautifulSoup(response.text, 'html.parser')
            
            # Find all links on the page
            for link in soup.find_all('a'):
                href = link.get('href')
                if not href:
                    continue
                
                # Convert relative URLs to absolute URLs
                full_url = urljoin(current_url, href)
                
                # Only process URLs from the same domain that we haven't seen before
                if (urlparse(full_url).netloc == domain and 
                    full_url not in discovered_urls):
                    queue.append(full_url)
                    discovered_urls.add(full_url)
                    
        except Exception as e:
            print(f"Error crawling {current_url}: {str(e)}")
    
    return discovered_urls

# Usage
urls = spider_website("https://website.com")
unique_urls = set([x.split("#")[0] for x in urls]) # Remove anchors

print("\n".join(list(unique_urls)))
```
This gave me a nice list of unique URLs to download. I used `shot-scraper` to do just that and save each page to its own PDF.

```bash
python spider.py | xargs -I{} shot-scraper pdf {}
```

