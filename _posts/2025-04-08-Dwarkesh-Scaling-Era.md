# Notes from Dwarkesh Patel's 'The Scaling Era'

Growing up I remember having a conversation with a friend about the pace of technilogical progress; I had read something about an exponential curves and thought about how significantly things would change in just 30 years if we truly were on such a trajectory. This was about 15 years ago; following the progress of AI makes things sure feel exponential, or, at least, compounding at a significant rate.

This was bolstered after reading Dwarkesh Patel's "The Scaling Era: An Oral History of AI 2019 - 2025" which I thought it was *excellent*. [You should buy it and read it.](https://www.stripe.press/scaling) Punchy, to the point, and full of insights from existing and to-be legends of the industry, from AI lab CEOs, to engineers, to practicioners in the field. The format was effective: weaving together multiple interview transcripts, which I appreciated because it was a very efficient way to quickly set the context around a particular topic, allowing the book to cover a lot of ground in a relatively compact form. Of course, those who are interviewed are incentivized to "talk their book" and are part of the AI industry so may see things more optimistically than others. That being said, I found the content to be fairly balanced and reasonable.

While there's only eight chapters, there's a ton to take away; I've attempted to distill what I found most interesting below:

| # | Chapter | Takeaway |
|---|---------|-------------|
| - | Preface | **The current trajectory for this technology suggests massive changes are on the horizon, and we have just started to scale up.** |
| 1 | Scaling | **Improvements in model capability have increased with compute, meaning with enough compute we may be able to model the entirety of our knowledge as a single "blob"** |
| 2 | Evals | **We're just beginning to understand model capabilities in a variety of domains, and while progress is uneven, unlocking long-horizon tasks is the next step in capturing value.** | <<< redo >>>
| 3 | Internals | **We don't know precisely what's going on in these models; even when we do identify certain undesirable behaviors, we simply train the model not to output that text instead of being able to remove it entirely.** |
| 4 | Safety | |
| 5 | Inputs | |
| 6 | Impact | |
| 7 | Explosion | |
| 8 | Timelines | |

## Preface - The promise of AI
Large Language Models ("LLMs") are  different from technologies of the past because they are "massively multitask"[^pg. 1], meaning they excel on multiple dimensions simultaneously. The flexibility offered by LLMs is unparalleled, and the raw power contained in such a relatively small size is astounding. 

The promise of the technology is real, bordering on hyperbole. "We're creating god" isn't something you can casually explain to your mother. But the promise of scaled-up AI would be considered god-like for people living even just a generation ago. AGI is the promise that AI systems are generally equivalent to - or better than - humans on all knowledge work.

Most exciting is that we can see these capabilities emerging *today*, and "the trajectory suggests much more progress is coming" [^pg. 6]. The continuity in model improvements - so far - explains the massive level of investment that has been committed and continues to rise - though "the current level of investment still falls short of the dot-com boom" [^pg. 6]. I'm sure that won't stand for long.

## 1: Scaling
While the pace of improvement may not be immediately obvious to the casual observer, it is real. The technology these AI labs are building - and the speed at which they are doing so - is unprecedented. 

The rate at which the amount of compute required is compounding is astounding as well: 
> The compute needed to train a leading model is now 10 billion times higher than it was in 2010 ... and the compute used to train each frontier model **doubled every six months**. [^pg 15]


This is just talking about training - we now see a "second form of scaling emerging: inference scaling." Scaling at inference time is what we are now seeing with reasoning models - we now throw additional compute at these models *after* they've been trained and deployed, with impressive increases in capability.

> ... inference scaling is just an elaboration of the general principle: So far, the (exponentially) more compute and data you put in, the modre intelligence you get out. [^pg 17]

A simple but powerful observation.

One of the biggest oustanding questions today is whether the "scaling laws" continue to hold - will the combination of algorithmic progress, massive amounts of compute, and adequate data be enough to get us to AGI?

The trajectory has held, so far. 
> As you make AI systems larger ... you get really, really predictable trends for performance of these systems as you scale up. This holds true over many, many orders of magnitude. [^pg 18]

I found this fascinating. Typically you see varying levels of performance degredation as you scale complex systems. Now certainly there are breakthroughs and adjustements at each order-of-magntiude ("OOM") of scale - but I find this trend promising for the continued evolution of these models.

As Ilya Sutskever told Dario Amodei (CEO of Anthropic):
> "The models just want to learn" - [^pg 17]

An interesting concept that came up in Dwarkesh’s conversation with Jared Kaplan (cofounder of Anthropic) was that of a data manifold—a term I hadn’t encountered before reading this book. I might not get this exactly right, but as I understand it, a data manifold represents the underlying structure of all possible data points that capture the true “meaning” or semantics of a dataset. In the context of LLMs, the idea is that if a model can effectively “fit” to a sufficiently rich data manifold, it could develop a compressed but accurate representation of the world.

In theory, I would think, this would be present in whatever model achieves AGI.

In terms of where we stand today, it's clear agents are coming but have not fully taken off yet. Sholto Douglas (Anthropic) makes a good point here as to why: reliability. An agent is tasked with chaining together multiple steps of a process to achieve a task - even a fractional dropoff in each of these tasks makes the collective system worthless. As he puts it: "If you can't chain tasks successively with high enough probability, then you won't get [an agent]" [^pg 29]. 

Promisingly, he also goes on to say:
> The jumps we've seen so far are huge. Even if those continue on a smaller sacle, we're still in for extremely smart, very reliable agents over the next couple of orders of magnitude. **We have a lot more jumps coming**. [^pg 30]

Leopold Aschenbrenner, who has is own [deep views on AI](https://situational-awareness.ai/), thinks there's the "unhobbling" path in addition to just scaling up - i.e., ways for existing models to get better through tool use, RLHF, or optimized prompts.

Regardless of unhobbling, it's clear scaling will continue and will have a dramatic impact. Gwen Branwen of [internet fame](https://gwern.net/) saw GPT-3 as "the crucial test" [^pg 40].

Gwern says:
> Going from GPT-2 to GPT-3 is one of the biggest scale-ups in all of neural network history. If scaling was bogus, then the GPT-3 paper would be super unimpressive ... I open up to the second page [of the paper] and I saw the few-shot learning chart, and I'm like "Holy shit, we live in the scaling world."

The original release of ChatGPT was based on `gpt-3.5-turbo` - and look how far we've come since then!

What struck me is the constant mention of compute, and why we're at a turning point now where things are actually possible. It's like a confluence of factors have all come together at this moment.

Gwern continues:
> It turns out that everywhere you go, compute, trial and error, and serendipity play enormous rolesin how things actually happen. Once you understand that, you understand why compute comes first.
>
> ...
> 
> Reading the old deep learning literature, you see all sorts of ideas that were completely correct but that no one could prove ... [because] the researchers didn't have the compute to train a version that would have worked.

Translation: hardware is fast, powerful, and abundant enough such that these techniques can be validated. Now we just need to continue to scale up (or at least that's the thinking).

With all that in mind, it seems that **improvements in model capability should continue to increase with additional compute, meaning at some point we may be able to model the entirety of our knowledge.**

## 2: Evals

To understand if these models are actually improving we need some way to *evaluate* their capability. Enter evals. 

While there are a variety of eval tests, frameworks, platforms out there today, there isn't one standard eval that neatly captures the entirety of a particular model's capabilities. Sure, labs can make technical measurements (such as "loss") to understand how well the training is going - but most people "want models to complete real tasks, like writing code, doing our grad school homework, or, eventually, making money for us while we kick back" [^pg 47].

One contributing factor to the difficulty in evaluating a model's capabilities is the diversity of inputs. Jack Clark, cofounder of Anthropic puts it simply, asking:

> What about all the capabilities we don't know about because we haven't thought to test for them?

Model developments are moving so fast I'm of the opinion we really don't know which models are good at which tasks. We're still going off "vibes" - which works, for now. Claude and Gemini 2.5 excel at coding; GPT-4.5 is great at creative tasks, etc. As we develop better evals we will have a much more quantitative view of which model to deploy for a particular class of problems - though this may be a moot point considering the possibility of *model convergence*, the idea that there's "one" representation of the world's information and assuming these models scale up, they converge on a single "solution" to these problems. 

Dwarkesh drops this in again, asking Francois Chollet, creator of the ARC-AGI test, about the "idea that LLMs are fitting some manifold underlying the input data" [^pg 52]. Chollet argues this is a type of memorization and is the crux of the "reasoning vs. just-predicting-the-next-token" debate. Gwern and Francois Chollet both explore the idea that these models are effectively creating "mini-programs" to generate the right "solutions" - or answers to prompts. Chollet argues that these models can simply memorize these 'programs' and deploy them using a type of pattern matching - *true* reasoning comes from the "the ability to synthesize a new program on the fly based on bits and pieces of existing programs" [^pg 53].

How do we measure or evaluate this? Well, we don't have a great answer for that just yet.

There are pockets of puzzles and problems that Chollet points to that might be a useful exercise. He points out an interesting observation: "LLMs can solve a Caesar cipher, transposing letters to code a message" which is "a very complex algorithm, but it comes up quite a bit on the internet" - meaning "they've basically memorized it." LLMS excel at Caesar ciphers of up to 5 or so letters "because those are very common numbers in examples provided on the internet" but "if you try to do it with an arbitrary number like nine, it's going to fail." In other words, the model "does not encode the generalized form of the algorithm but only specific cases. [^pg 55]. 

I found this to be a compelling argument, until I read the footnote: as of 2024 this is no longer true. The footnote mentioned tool use - i.e., the ability to use Python to solve problems - but in my own brief testing gpt-4o (updated) got a 12-letter shift Caesar cipher in one go. 

I ran a the following prompt:
```
Solve this. I will not provide any additional hints:
Tqxxa mzp tai mdq kag fapmk
```
The answer of course is "Hello and how are you today". Interestingly, I got the following results:
| Model          | Result |
|----------------|--------|
| gpt-4o (new)         | pass   |
| Claude 3.7     | pass   |
| Optimus Alpha (stealth)  | pass   |
| grok3 beta        | fail   |
| qwen-2.5-coder-32b  | fail   |
| llama-4-maverick-17b-128e-instruct  | fail   |
| llama3-70b-8192  | fail   |
| gemma2-9b-it  | fail   |
| deepseek-r1-distill-llama-70b  | fail   |

Some of these models have some sort of reasoning baked in at this point, which was evident in each of the responses they gave. So generally, it seems that larger more recent models either have more Caesar cipher problems baked into their training data, or they are getting better about "reasoning" through these types of puzzles. If the latter, I would think Chollet's assertion that models "[do] not encode the generalized form of the algorithm but only specific cases" is slowly falling away.

I do sense that we are breaking down these barriers very quickly. Dario Amodei says as much: "scaling pretraining up will keep working ... it just keeps working" [^pg 57]. While our current methodologies for evaluating model intelligence vary, "it turns out that intelligence isn't a single spectrum. There are a bunch of different areas of domain expertise ... [and] different kinds of skills" [^pg 57]. This is reflected in the industry's current benchmarks, e.g., [MMLU](https://arxiv.org/abs/2009.03300) , MATH, AIME, etc. But much like the SATs don't capture the full extent of a person's capabilities, there remains a wide swath of potential model capability that we simply don't measure, like "understanding streaming video." Shane Legg puts it clearly: "It's a diffictul question because the generality of human intelligence is very broad. You have to go into the weeds ... to find out if there are specific types of things that are missing from existing benchmarks" [^pg 59].

#### Context Windows

One area in partiular I find interesting, and commercially valuable, is the context window. Most LLM context window legnths today are anywhere from 8k to 200k tokens, though Gemini promises up to 1M. Meta's LLama 4 promises 10M, though as of April 2025 no provider seems to support more than 200k. In responding to a question about sample efficiency Shane Legg mentions that "when something is in [a model's] context window, that biases the distribution to behave in a different way ... a very rapid kind of learning ... [as] models can learn things immeidatley when they're in the context window"  [^pg 59]. This is what makes long context windows so powerful - you can be super precise in what you want the model to focus on for a given prompt and leverage the ever-increasing intelligence of the base model to interpret what you've put in the context window. 

But to date, the promised "effective length" has fallen far short of what has been advertised. Most benchmarks focus on needle-in-a-haystack tests which mosts models pass well. But that's only marginally useful - what we *really* care about is the ability to *reason* about the content comprehensively, not just regurgitate a particular set of tokens. A fantastic paper on the subject introduces a benchmark to do exactly this: [NoLiMa: Long-Context Evaluation Beyond Literal Matching](https://arxiv.org/abs/2502.05167). The benchmark "[requires] models to infer latent associations to locate the needle within the haystack" - a much more pratically useful evaluation, in my opinion. Their results show that true "effective lengths" of context windows are as low as 2 - 4k; GPT-4o was at the top of the list with an effective length of 8k, even though the claimed length is 128k. 

While these results are challenging, Legg seems to be confident we can overcome this and other blockers (emphasis added):
> We know how to build models that have some degree of understanding of what's going on. That did not existin the past. We've got a scalable way to to do this now, which unlocks lots and lots of new things.
>
> ...
>
> My feeling is that there are relatively clear paths forward now to address most of the shortcomings we see in the existing models. Whether it's about delusions, factuality, the type of memory and learning they have, understanding video, all sorts of things like that, **I don't see big walls in front of us.** I just see that with more research and work, **these things will improve and probably be adequately solved.**

#### Measurement & Agents
Because we are so good at training these models there is a continued risk of "overfitting" the models to particular benchmarks. The recent [LLama-4 controversy](https://www.theregister.com/2025/04/08/meta_llama4_cheating/) controversy is one example, but Dwarkesh includes a nice study on GSM8K, showing some models - like Llama 3, Mistral, etc. appear to overfit the public version of GSM8K, while others - `gemini-1.5-flash-preview-0514`, `claude-3-opus-20240229` and good ol' `gpt-4` demonstrated competency on the novel benchmark.

It's a difficult problem, and one that needs to be solved in order to know we have effective AI agents. Dwarkesh makes a nice point in his question to Sholto Douglas, RL Infra lead at Anthropic, asking whether lack of reliability is the contributor to agents' poor performance on long-horizon tasks; in that attemting 10 or 100 steps in a row "diminish[es] the overall task reliability" [^pg 65].

It really clicked for me when Douglas mentioned that the academic evals only test models on a single-step problem. "The key issue is that your base task solve rate in 90 percent. If it was 99 percent, it wouldn't be a problem" [^pg 66].

It is this compounding of errors which makes today's agents less effective, but it seems clear this will be resolved sooner than later. It's also the sort of capability which could contribute to a fast-takeoff; John Shulman of Anthropic says it's nothing to be scared of right now - but assuming we continue on this trajectory, it's arguably the first thing we should keep an eye on. 

Agents performing long-horizon tasks would be incredibly economically valuable, but also opens the door to many unknowns. We're still developing the evaluation frameworks to properly measure these capabilities but it seems to me it's just a matter of time.

## 3: Internals
Understanding the goings-on of an LLM is the subject of intense study and debate. Thinking back to the Caesar cipher example above, are the models capable of *reasoning* through the problem to find the answer? As of today it seems to be a mixture of memorization and reasoning.

The conversation excerpt with Trenton Bricken is fascinating, drawing comparisons to the brain - specifically the cerebellum. He notes that the concept of a particular type of electrical engineering circuit shares similarities with the cerebellum which "is not just in us ... [but] in basically every organism" [^pg 71]. It is these circuits - Bricken points to the [induction head](https://transformer-circuits.pub/2022/in-context-learning-and-induction-heads/index.html) as one example - which give the models their reasoning-like capabilities.

Intuitively this makes sense: various circuits develop as a result of deep learning, representing the patterns and associations seen in the training data. Bricken provides a concrete example when he talks about the "indirect object identification circuit" which specifically captures indirect object of a sentence. The example provided is instructive [^pg 75]:

> In the sentence "Mary and Jim went to the store, Jim gave the object to __"; it would predict "Mary" because Mary has appeared before as the indirect object.

Combine enough of these circuits together and you get a very sophisticated language model.

The conversation with Bricken continues with an interesting, and technical, discussion regarding "vector superposition," which he contends "emerges when yuo have high-dimensional data that is sparse." [pg 80]. Dwarkesh really brings it to life with an image showing how just a 2D plane (x and y coordinates) can represent a truly diverse set of data. Point being, even with seemingly concise neurons one can represent incredibly rich information.

Even so, I find it incredible that *we literally don't know how these things work* at a detailed level, and at the same time are concerned about their ability to decieve. We have [mechanistic interpretability](https://www.transformer-circuits.pub/2022/mech-interp-essay) but that only gets us so far. Current labs are trying to understand whether "a particular motivation has been developed by a particular training process" such that we could adjust the training to motivate it to "legitimately want to help us" [^pg 84].

There's a tension here between training, inference, and the AI's motivation, and even though we know what is going on at a high level, Carl Shulman brings up a mind blowing concept: gradient hacking. Dwarkesh's footnote has the following definition:

> **Gradient hacking**: A speculative phenomenon in which a subsystem of an advanced AI ... manipulates its own training in real time to circumvent the intended updates.

When labs talk of AGI or ASI, and the work to get there, this certainly comes to mind. A stated objective of many labs is to use AI to build AI, i.e., enable recursive improvement by using AI agents to do fundamental R&D to build future versions of themselves. In that scenario by definition the AI has access to the training scaffolding, weights, everything - making control and understanding of paramount importance, in my opinion.

Interpretation and monitoring appears to be exceedingly challenging. Bricken brings up another thought-provoking observation, in that "the tokens you actually see in the chain of thought do not necessarily correspond to the vector representation that the model gets to see when it's deciding to attend back to those tokens [^pg 87]. For practical purposes, model's stated reasoning may not actually reflect it's "actual thoughts" as represented in its [latent space](https://en.wikipedia.org/wiki/Latent_space).


While monitoring is challenging enough, Dwarkesh ends the chapter asking Dario Amodei what true alignment is, to which he responds: "We don't know". 

It seems we don't have the tools or technology to truly a.) understand how these models are processing information (at a sufficiently granular level) and b.) take comprehensive action to prevent malicious capabilities. Dario continues with a surprising statement (emphasis added) [^pg 88]: 

> All of the current methods that involve some kind of fine-tuning have the property that *the underlying knowledge and abilities that we might be worried about don't disappear; the model is just taught not to output them*.

## 4: Safety
Central to the AI-doomerism debate is that of safety: are we adequately prepared as a society? Do we have the right guardrails in place to ensure humans remain in control?

I've noticed these questions have a baseline assumption that scaling works, or at the very least that model capabilities will continue to compound at a significant rate. Those raising the alarms about AI safety are talking about *future* systems and what they can do; implicitly or explicitly they assume many of the hurdles present today are overcome.

They may have a point. As explored in *Evals* we simply do not posess the capability to understand or steer these models at a level of rigor that would be required to harness a super-powerful AI, and it's not clear these capabilities are evolving at the same pace of AI itself.

*Can* we align these models? Joe Carlsmith of Open Philanthropy shares a key distinction: while models may *understand* human values, it's unclear whether it *shares* them. We're at the point where we "hope we're never in the situation where the AI has very different values from us and is quite smart and knows what's going on, and is thus **in an adversarial relationship with our training**" [^pg 91]. Quite the hope! It seems a key difference from biological brains, however, is the precision and finality of our ability to "edit" the "brain" of the model. Dwarkesh puts this succinctly:

> When you update a model, the edit is much more directly connected To its brain, then the reward or punishment a human gets. We're going to adjust each different parameter to the exact floating point number that calibrate it to the output we want.
