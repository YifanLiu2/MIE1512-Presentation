# Presentation Script

**Title**: Can LLM Judgment Capture Human Preference?
**Duration**: ~12 minutes
**Pace**: Natural spoken English, ~150 words/min

---

## Slide 1: Title (30 sec)

Hi everyone, my name is Yifan Liu, and today I'd like to walk you through my project, which investigates whether an LLM acting as a judge can actually capture what humans prefer when it comes to evaluating the task performance of large language models.

---

## Slide 2: The Research Problem (1.5 min)

So to set the stage, evaluating large language models has always been one of the most challenging and expensive parts of the development cycle, because the gold standard approach requires real human evaluators to read through model outputs and vote on which response they think is better, and while that gives you very reliable preference signals, it's also incredibly slow, costly, and difficult to scale to the number of models and tasks that the community now cares about.

Because of that, a popular alternative has emerged in recent years, which is to take a strong LLM like GPT-4 and use it as an automated judge that scores other models' outputs, and this so-called "LLM-as-judge" paradigm has become the backbone of many widely used benchmarks and leaderboards in the field.

Now, the existing literature reports very high overall agreement between these automated judges and human voters, with Pearson correlations typically at 0.94 or above, and on the surface that seems like a strong validation of the approach. But the question I want to explore in this project is whether that agreement is actually uniform across all types of tasks, or whether the LLM judge might agree with humans quite well on certain kinds of tasks, like coding or reasoning, while failing to capture human preferences on other tasks, such as translation or mathematics, and that distinction turns out to matter quite a lot in practice.

---

## Slide 3: Why This Matters (1.5 min)

And the reason it matters is threefold, so let me walk through each one briefly.

The first and perhaps most obvious concern is that if we're going to replace human evaluation with an LLM judge as a cost-saving measure, we really need to understand where that substitution is reliable and where it breaks down, because if the judge systematically fails on certain task categories without us knowing it, then we end up with a false sense of confidence in model quality that could have real downstream consequences.

The second reason this matters is for model development itself, because alignment training methods like RLHF and DPO rely on automated preference scores as their training signals, and if the automated judge disagrees with what humans actually prefer on certain types of problems, say mathematical reasoning or translation tasks, then the model is essentially being optimized for the wrong objective on those tasks, meaning you'd be training it to produce outputs that satisfy the judge rather than outputs that real users would actually choose.

And the third reason is practical model selection: when practitioners decide which LLM to deploy for their specific application, they typically rely on benchmark leaderboards to make that decision, and if those leaderboards are driven by an automated judge that doesn't accurately reflect human preferences for the particular use case they care about, they could very well end up choosing the wrong model for their needs.

---

## Slide 4: Method Section Break (5 sec)

With that motivation in mind, let me now describe the LLM-as-judge method that we are evaluating.

---

## Slide 5: LLM-as-Judge: How It Works (2 min)

The core idea behind LLM-as-judge is conceptually straightforward: instead of having a human read a model's output and rate its quality, you have a strong language model, typically GPT-4-turbo, do the reading and rating instead, and the promise is that this can be done at massive scale for a fraction of the cost of human annotation.

WildBench, which appeared as a spotlight paper at ICLR 2025, represents one of the most sophisticated implementations of this paradigm to date. The way it works is that you give the judge a real user prompt along with a model's response to that prompt, and the judge then evaluates the quality of that response in two complementary ways. The first is called WB-Score, which is a pointwise quality rating on a scale from 1 to 10 where the judge assesses how well the response addresses the user's needs. The second is called WB-Reward, which is a pairwise comparison where the response is evaluated against three reference models across five outcome levels, ranging from much worse to much better, and this pairwise score includes a correction for the well-known length bias where longer responses tend to get unfairly higher ratings.

What makes WildBench particularly interesting, and what makes it central to our analysis, is that it doesn't apply the same generic evaluation criteria to every task. Instead, before scoring, the judge first generates a task-specific checklist that's tailored to the type of problem the user is trying to solve, and the benchmark covers 1,024 real-user tasks drawn from WildChat across more than 60 models, reporting an overall correlation of 0.98 with human Elo ratings.

---

## Slide 6: Task-Specific Checklists (1.5 min)

And this task-specific checklist is really the key idea that distinguishes WildBench from a naive LLM-as-judge approach, because the evaluation criteria adapt to the nature of each task rather than applying a one-size-fits-all rubric.

For example, when the judge evaluates a coding and debugging task, the checklist focuses on things like whether the code actually compiles and runs, whether the logic is correct, whether edge cases are properly handled, and whether the solution is efficient, which are very concrete, verifiable criteria. But when the same judge evaluates a creative writing task, the checklist shifts to entirely different dimensions, such as whether the narrative is engaging, whether the tone matches what the user asked for, and whether the language is fluent and vivid, which are inherently more subjective qualities.

Similarly, for mathematics the checklist centers on correctness of the final answer, validity of intermediate reasoning steps, and completeness of the solution, while for translation it focuses on meaning preservation, grammatical correctness in the target language, and whether cultural nuances are appropriately handled.

WildBench organizes its tasks into 12 categories along these lines, and this categorization system is what we apply to human preference data to test whether the LLM judge's agreement with humans holds up uniformly, or whether it varies systematically depending on the type of task being evaluated.

---

## Slide 7: Dataset Section Break (5 sec)

Now let me describe the dataset that serves as our human preference ground truth.

---

## Slide 8: Chatbot Arena (2 min)

To test whether the LLM judge actually captures what humans prefer, we need a large-scale source of real human preference judgments, and for that we turn to Chatbot Arena, published at ICML 2024, which is the largest and most widely cited open dataset of human preferences for language model evaluation.

The way Chatbot Arena works is that a user comes to the platform, submits any prompt they want, and then sees responses from two anonymous language models displayed side by side without knowing which model produced which response, and they simply vote for the one they think is better, which gives you a clean pairwise preference signal free from brand bias. The dataset I'm working with contains 57,477 of these pairwise battles spanning 64 different models, with the full text of both the prompts and the responses available for analysis, and the platform has additionally collected over 1.7 million voting records that can be used for more robust ranking estimates.

The model rankings are computed from these battles using the Bradley-Terry model, which produces Elo ratings following the same mathematical framework used in competitive chess, and this system is widely considered the gold standard for measuring whether language models are truly aligned with human preferences, precisely because the judgments come from real users making real choices about real tasks.

Now, the key limitation of Arena for our purposes is that it only provides aggregate Elo rankings across all task types, with no breakdown by task category, and this is exactly the gap our analysis fills. We take each of the 57,000 battle prompts and classify it into one of WildBench's 12 task types using keyword-based heuristics, which allows us to compute per-category human win rates for the 12 models that overlap between both datasets, and then directly compare those human-derived category-level rankings against WildBench's automated LLM judge scores, all processed at scale using PySpark on approximately 1.8 million records with no additional LLM inference required.

---

## Slide 9: Results Section Break (5 sec)

So with that methodology in place, let me show you what the analysis actually revealed.

---

## Slide 10: Overall Correlation (1 min)

At the overall level, the results confirm what previous work has reported: the LLM judge agrees strongly with human preferences, with a Pearson correlation of 0.954 between Arena Elo and WB-Score, and 0.943 between Arena Elo and WB-Reward, both of which are statistically significant with p-values well below 10 to the negative 5.

However, I want to draw your attention to an important caveat here, which is that this high overall correlation is partly driven by the very large performance gap between model tiers, because every evaluation system, whether automated or human, can easily tell that GPT-4 is substantially better than Llama-2, and that kind of coarse-grained separation naturally inflates the correlation coefficient. The more interesting and more practically relevant question is whether the automated judge can capture the finer-grained differences between models, and whether it does so consistently across different types of tasks, which is exactly what the per-category analysis is designed to answer.

---

## Slide 11: Per-Category Correlation (1.5 min)

And this is where the story becomes genuinely surprising, because when I break down the same correlation analysis by task category, the picture changes quite dramatically from what the overall numbers would suggest.

Reasoning shows the highest agreement between the LLM judge and human voters, with a Pearson correlation of 0.749, and creative writing is fairly close behind at 0.713, while coding and debugging and role-playing are both hovering around 0.69, which you might characterize as moderate but still meaningful agreement.

But then the numbers start to drop off noticeably: information seeking falls to 0.595, mathematics drops further to 0.558, and then we get to translation, which has a Pearson correlation of just 0.146, and at that level you're essentially looking at near-zero agreement, meaning the LLM judge's rankings of models on translation tasks bear almost no relationship to how humans actually rank those same models on the same type of task.

So what we're seeing is that the 0.95 overall correlation is actually masking a range that spans all the way from 0.75 down to 0.15, which is an enormous spread, and the LLM judge functions as a reasonable proxy for human preferences on reasoning and creative writing tasks but is nearly uncorrelated with humans when it comes to evaluating translation quality.

And as for why the overall number remains so high despite this category-level variation, it's essentially a manifestation of Simpson's paradox: when you aggregate across all categories, the large inter-model performance gaps dominate the signal and the category-specific noise gets averaged out, so the 0.95 correlation is really telling you that both systems can distinguish between model tiers, but it's not telling you that they agree on which specific abilities differentiate the best models from the rest.

---

## Slide 12: Heatmap (45 sec)

To visualize this another way, this heatmap displays the per-category human win rates from Arena for the top 15 models, where each row represents a model, each column represents a task category, and the color intensity indicates the win rate, with darker shades corresponding to higher performance.

The most striking observation here is that gpt-4-1106-preview, which holds the top position in the overall Elo rankings at 1235, is actually not the number-one model in most of the individual categories, and you can clearly see the heterogeneity across the board, with different models exhibiting quite different profiles of strengths and weaknesses across task types, some excelling in creative writing but underperforming in math, others strong in reasoning but comparatively weak in translation.

---

## Slide 13: Category Leaders (1 min)

And to make this point even more concrete, let me show you who actually leads each task category according to human voters in the Arena dataset. In coding and debugging, creative writing, information seeking, and mathematics, the top model by win rate turns out to be gpt-3.5-turbo-0314, which is notably not GPT-4 at all, while gpt-4-1106-preview, the overall Elo champion, only comes out on top in the reasoning category. Meanwhile role-playing is led by mpt-30b-chat, and translation by gpt-4-0613, each of which are models that don't even crack the top three in the overall rankings.

The implication of this finding is quite clear: a single aggregate leaderboard ranking, whether it's derived from an LLM judge or from human Elo ratings, simply does not capture the full picture of model capabilities, because the "best" model is fundamentally a function of what task you need it to perform, and an LLM judge that shows strong agreement with humans at the aggregate level may nonetheless provide misleading guidance when it comes to selecting a model for a specific downstream use case.

---

## Slide 14: Summary (45 sec)

So to bring everything together: can an LLM judge capture human preferences when evaluating LLM task performance? The answer, based on the evidence from this analysis, is that it very much depends on the type of task you're evaluating.

At the overall level, yes, LLM judge scores correlate strongly with human Elo at r equals 0.95, but at the per-category level, that agreement ranges from a respectable 0.75 for reasoning all the way down to a negligible 0.15 for translation, and beyond the correlation numbers, the model rankings themselves shift substantially across task categories, meaning that the best model overall is simply not the best model in most individual task types.

The central takeaway is that high overall correlation between LLM judges and human evaluators should not be interpreted as blanket validation of automated evaluation, and that per-category analysis is essential for understanding where these automated systems can be trusted and where they cannot.

---

## Slide 15: Ongoing Work (45 sec)

Before I close, let me briefly outline what has been completed so far and what remains as ongoing work for the next phase of the project.

On the completed side, I've successfully categorized all 57,500 Arena battles into task types, computed per-category human win rates for all models, built Elo ratings using the Bradley-Terry model, and conducted cross-benchmark correlation analysis across seven task categories, producing seven visualizations in total.

As for what's still in progress, there are several directions I'm actively pursuing: computing per-category Elo ratings rather than relying solely on raw win rates, which would provide a fairer comparison since win rates depend heavily on which opponents a model happens to face; incorporating the full 1.7 million Arena voting records to get more robust Elo estimates; integrating AlpacaEval 2 to enable a three-way comparison between two different automated evaluation systems and human evaluation; adding bootstrap confidence intervals to properly quantify the uncertainty in these correlation estimates; and improving the keyword-based categorization coverage from its current level of about 40 percent up to a target of 70 percent or higher.

---

## Slide 16: Thank You (10 sec)

And with that, I'd like to thank you all for your attention, and I'm very happy to take any questions you might have.

---

**Total: ~12 minutes**
