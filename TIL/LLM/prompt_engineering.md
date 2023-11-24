# Curated list of Prompt Engineering techniques

Most of tips below were copied from [Lil'Log](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/)
blog

## Basic Prompting

### Zero-Shot

**Zero-shot learning** is to simply feed the task text to the model and ask for results.

```text
Text: i'll bet the video game is a lot more fun than the film.
Sentiment:
```

### Few-shot

Few-shot learning presents a set of high-quality demonstrations, each
consisting of both input and desired output, on the target task. As the
model first sees good examples, it can better understand human intention
and criteria for what kinds of answers are wanted. Therefore, few-shot
learning often leads to better performance than zero-shot. However, it
comes at the cost of more token consumption and may hit the context length
limit when input and output text are long.

```text
Text: (lawrence bounces) all over the stage, dancing, running, sweating, mopping his face and generally displaying the wacky talent that brought him fame in the first place.
Sentiment: positive

Text: despite all evidence to the contrary, this clunker has somehow managed to pose as an actual feature movie, the kind that charges full admission and gets hyped on tv and purports to amuse small children and ostensible adults.
Sentiment: negative

Text: for the first time in years, de niro digs deep emotionally, perhaps because he's been stirred by the powerful work of his co-stars.
Sentiment: positive

Text: i'll bet the video game is a lot more fun than the film.
Sentiment:
```

#### Tips for Example Ordering

- A general suggestion is to keep the selection of examples diverse, relevant
  to the test sample and in random order to avoid majority label bias and
  recency bias.
- Increasing model sizes or including more training examples does
  not reduce variance among different permutations of in-context examples. Same
  order may work well for one model but badly for another. When the
  validation set is limited, consider choosing the order such that the model
  does not produce extremely unbalanced predictions or being overconfident about
  its predictions.

## Instruction Prompting

The purpose of presenting few-shot examples in the prompt is to explain
our intent to the model; in other words, describe the task instruction
to the model in the form of demonstrations.

However, few-shot can be expensive in terms of token usage and restricts
the input length due to limited context length. So, why not just give
the instruction directly?

_Instructed LM_ finetunes a pretrained model with high-quality tuples
of (task instruction, input, ground truth output) to make LM better
understand user intention and follow instruction. RLHF
is a common method to do so. The benefit of instruction following style
fine-tuning improves the model to be more aligned with human intention
and greatly reduces the cost of communication.

### How to "instruct" an LLM?

When interacting with instruction models, we should describe the task
requirement in details, trying to be specific and precise and avoiding
say “not do something” but rather specify what to do.

```text
Please label the sentiment towards the movie of the given movie review. The sentiment label should be "positive" or "negative".
Text: i'll bet the video game is a lot more fun than the film.
Sentiment:
```

Explaining the desired audience is another smart way to give instructions.
For e.g. to produce education materials for children:

```text
Describe what is quantum physics to a 6-year-old.
```

& safe content:

```text
... in language that is safe for work.
```

In-context instruction learning (Ye et al. 2023) combines few-shot
instruction prompting. It incorporates multiple demonstration examples
across different tasks in the prompt, each demonstration consisting
of instruction, task input and output. Note that their experiments
were only on classification tasks and the instruction prompt contains
all label options.

```text
Definition: Determine the speaker of the dialogue, "agent" or "customer".
Input: I have successfully booked your tickets.
Ouput: agent

Definition: Determine which category the question asks for, "Quantity" or "Location".
Input: What's the oldest building in US?
Ouput: Location

Definition: Classify the sentiment of the given movie review, "positive" or "negative".
Input: i'll bet the video game is a lot more fun than the film.
Output:
```

## Self-Consistency Sampling

Self-consistency sampling (Wang et al. 2022a) is to sample multiple
outputs with `temperature > 0` and then selecting the best one out of
these candidates.
The criteria for selecting the best candidate
can vary from task to task. A general solution is to pick
majority vote. For tasks that are easy to validate such as
a programming question with unit tests, we can simply run
through the interpreter and verify the correctness with unit tests.

## Chain-of-Thought (CoT)

Chain-of-thought (CoT) prompting (Wei et al. 2022) generates a
sequence of short sentences to describe reasoning logics step by
step, known as reasoning chains or rationales, to eventually lead
to the final answer. The benefit of CoT is more pronounced for
complicated reasoning tasks, while using large models (e.g. with
more than 50B parameters). Simple tasks only benefit slightly
from CoT prompting.

### Types of CoT prompts

#### Few-shot CoT

It is to prompt the model with a few demonstrations, each containing
manually written (or model-generated) high-quality reasoning chains.

```text
Question: Tom and Elizabeth have a competition to climb a hill.
Elizabeth takes 30 minutes to climb the hill. Tom takes four times
as long as Elizabeth does to climb the hill. How many hours does
it take Tom to climb up the hill?
Answer: It takes Tom 30*4 = <<30*4=120>>120 minutes to climb the hill.
It takes Tom 120/60 = <<120/60=2>>2 hours to climb the hill.
So the answer is 2.
===
Question: Jack is a soccer player. He needs to buy two pairs of socks
and a pair of soccer shoes. Each pair of socks cost $9.50, and the
shoes cost $92. Jack has $40. How much more money does Jack need?
Answer: The total cost of two pairs of socks is $9.50 x 2 = $<<9.5*2=19>>19.
The total cost of the socks and the shoes is $19 + $92 = $<<19+92=111>>111.
Jack need $111 - $40 = $<<111-40=71>>71 more.
So the answer is 71.
===
Question: Marty has 100 centimeters of ribbon that he must cut into 4
equal parts. Each of the cut parts must be divided into 5 equal parts.
How long will each final cut be?
Answer:
```

#### Zero-Shot CoT

Use natural language statement like `Let's think step by step` to explicitly
encourage the model to first generate reasoning chains and then to
prompt with `Therefore, the answer is` to produce answers. Or a similar statement
`Let's work this out it a step by step to be sure we have the right answer`.

```text
Question: Marty has 100 centimeters of ribbon that he must cut into 4
equal parts. Each of the cut parts must be divided into 5 equal parts.
How long will each final cut be?
Answer: Let's think step by step.
```

#### Tips and Extensions

- Self-consistency sampling can improve reasoning accuracy by sampling
  a number of diverse answers and then taking the majority vote.
- Another approach for ensemble learning is to alter the example order or
  use model generated rationales to replace human-written ones to introduce
  randomness during multiple sample trials. Then aggregate model
  outputs with a majority vote to get final answer.
- If training examples are only associated with true answers (easy to
  verify!) but no rationales, we can follow the STaR (Self-Taught Reasoner) method :
  - Ask LLM to generate reasoning chains and only keep those leading to correct answers;
  - Then fine-tune the model with generated rationales and repeat the process
    until convergence.
  - Note that higher temperature is more likely to generate incorrect rationales
    with correct answers. If training examples do not have ground truth answers, maybe
    consider using majority votes as the “correct” answers.
- Prompts with demonstrations of higher reasoning complexity can achieve better
  performance, where complexity is measured by the number of reasoning steps in the
  chains. When separating reasoning steps, newline `\n` symbol works better than `step i`,
  period `.` or semicolon `;`.
- CoT prompts with only complex examples can improve the accuracy of complex
  questions, but perform poorly in simple questions;
- Changing `Q:` to `Question:` is found to be helpful.
- _Self-Ask_ is a method to repeatedly prompt the model to ask following-up
  questions to construct the thought process iteratively. Follow-up questions
  can be answered by search engine results.
- _Tree of Thoughts_ extends CoT by exploring multiple reasoning possibilities at
  each step. It first decomposes the problem into multiple thought steps and generates
  multiple thoughts per step, essentially creating a tree structure. The search process
  can be BFS or DFS while each state is evaluated by a classifier (via a
  prompt) or majority vote.

## Resources

[Lil'Log](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/)
