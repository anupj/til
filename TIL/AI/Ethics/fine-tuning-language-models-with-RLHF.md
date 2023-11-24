- GPT-3 and other LLMs are trained to _predict the next word_ on a large
  dataset of Internet text, rather than to safely perform the language task
  that the user wants and therefore _generate outputs that are untruthful,
  toxic, or reflect harmful sentifments_. - In other other words these models aren't _aligned_ with their users.

- OpenAI (and possibly other providers) use a technique called
  [RLHF](https://openai.com/research/learning-from-human-preferences). - On prompts submitted by customers to the OpenAI API, their labelers
  provide demonstrations of the desired model behaviour, and **rank** several
  outputs from our models, and then use this data to _fine-tune_ GPT-3.

- The resulting **InstructGPT** models are much better at following instructions than GPT-3.
  - They also make up facts less often, and show _small decreases_ in toxic output generation.
  - Its so good that OpenAIs labelers prefer outputs from 1.3B InstructGPT model over outputs
    from 175B GPT-3 model, despite having more than 100x fewer parameters.
