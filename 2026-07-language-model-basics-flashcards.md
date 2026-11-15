file-created-at:: 2026-07-18

- # Language Model Basics Flashcards
	- Tags: [[Language Models]] [[Machine Learning]] [[Flashcards]]
	- Purpose: Test foundational knowledge of how modern language models work, train, generate text, and are evaluated.
- ## Foundations
	- What is a language model? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:34.233Z
	  card-last-reviewed:: 2026-07-18T15:50:34.234Z
	  card-last-score:: 5
		- A model that assigns probabilities to sequences of tokens, allowing it to predict or generate text.
	- What is the main training objective of most autoregressive language models? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:47.903Z
	  card-last-reviewed:: 2026-07-18T15:50:47.903Z
	  card-last-score:: 5
		- Predict the next token from the tokens that came before it.
	- What is a token? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:28.217Z
	  card-last-reviewed:: 2026-07-18T15:50:28.218Z
	  card-last-score:: 5
		- A unit of text processed by a language model, such as a word, part of a word, punctuation mark, or character.
	- Why do language models use tokenization? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:46.170Z
	  card-last-reviewed:: 2026-07-18T15:49:46.170Z
	  card-last-score:: 5
		- To convert raw text into a finite sequence of IDs that the model can process numerically.
	- What is a vocabulary in a language model? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:08.111Z
	  card-last-reviewed:: 2026-07-18T15:49:08.111Z
	  card-last-score:: 5
		- The fixed set of tokens that the tokenizer can represent directly.
	- What is an embedding? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:27.132Z
	  card-last-reviewed:: 2026-07-18T15:50:27.132Z
	  card-last-score:: 5
		- A learned vector representation that places tokens or other items in a continuous numerical space.
	- What does a model parameter represent? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:48:22.867Z
	  card-last-reviewed:: 2026-07-18T15:48:22.867Z
	  card-last-score:: 5
		- A learned numerical value, such as a weight or bias, that influences the model's predictions.
- ## Transformer Architecture
	- What is a Transformer? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:13.467Z
	  card-last-reviewed:: 2026-07-18T15:50:13.467Z
	  card-last-score:: 5
		- A neural-network architecture that processes token representations using attention and feed-forward layers.
	- What does self-attention allow a language model to do? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:06.460Z
	  card-last-reviewed:: 2026-07-18T15:50:06.460Z
	  card-last-score:: 5
		- Weigh information from other tokens in the context when building a representation of each token.
	- What are queries, keys, and values in attention? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:51:36.452Z
	  card-last-reviewed:: 2026-07-18T15:51:36.453Z
	  card-last-score:: 5
		- Learned projections used to determine which token information is relevant and how that information should be combined.
	- Why does a Transformer need positional information? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:35.848Z
	  card-last-reviewed:: 2026-07-18T15:49:35.849Z
	  card-last-score:: 5
		- Self-attention alone does not encode token order, so positional information tells the model where tokens occur in the sequence.
	- What is causal masking? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:48:46.813Z
	  card-last-reviewed:: 2026-07-18T15:48:46.813Z
	  card-last-score:: 5
		- A restriction that prevents an autoregressive model from attending to future tokens while predicting the next token.
	- What is a context window? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:00.660Z
	  card-last-reviewed:: 2026-07-18T15:50:00.660Z
	  card-last-score:: 5
		- The maximum number of tokens the model can consider in a single request or sequence.
	- What is the purpose of a feed-forward network in a Transformer block? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:51:13.010Z
	  card-last-reviewed:: 2026-07-18T15:51:13.011Z
	  card-last-score:: 5
		- To transform each token representation through learned nonlinear operations after attention mixes contextual information.
- ## Training and Adaptation
	- What is pretraining? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:51:23.161Z
	  card-last-reviewed:: 2026-07-18T15:51:23.162Z
	  card-last-score:: 5
		- Training a model on a large, broad dataset, usually with a self-supervised objective such as next-token prediction.
	- Why is next-token prediction called self-supervised learning? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:10.050Z
	  card-last-reviewed:: 2026-07-18T15:50:10.050Z
	  card-last-score:: 5
		- The training labels come from the text itself: each observed next token serves as the target.
	- What is training loss? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:22.999Z
	  card-last-reviewed:: 2026-07-18T15:49:22.999Z
	  card-last-score:: 5
		- A numerical measure of how wrong the model's predictions are; training adjusts parameters to reduce it.
	- What is gradient descent? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:55.140Z
	  card-last-reviewed:: 2026-07-18T15:50:55.140Z
	  card-last-score:: 5
		- An optimization method that updates model parameters in directions expected to reduce the loss.
	- What is fine-tuning? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:29.536Z
	  card-last-reviewed:: 2026-07-18T15:50:29.536Z
	  card-last-score:: 5
		- Additional training of a pretrained model on a narrower dataset or objective to change its behavior or capabilities.
	- What is instruction tuning? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:25.502Z
	  card-last-reviewed:: 2026-07-18T15:50:25.502Z
	  card-last-score:: 5
		- Fine-tuning on instruction-and-response examples so the model becomes better at following user requests.
	- What is overfitting? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:48:30.935Z
	  card-last-reviewed:: 2026-07-18T15:48:30.935Z
	  card-last-score:: 5
		- When a model learns the training data too specifically and performs worse on new, unseen data.
- ## Inference and Generation
	- What is inference? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:38.469Z
	  card-last-reviewed:: 2026-07-18T15:49:38.469Z
	  card-last-score:: 5
		- Using a trained model to calculate predictions or generate outputs without updating its parameters.
	- What are logits? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:57.140Z
	  card-last-reviewed:: 2026-07-18T15:49:57.140Z
	  card-last-score:: 5
		- The model's raw scores for possible next tokens before they are converted into probabilities.
	- What does the softmax function do? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:51:05.358Z
	  card-last-reviewed:: 2026-07-18T15:51:05.358Z
	  card-last-score:: 5
		- Converts a set of logits into a probability distribution whose values sum to one.
	- What is greedy decoding? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:53.524Z
	  card-last-reviewed:: 2026-07-18T15:49:53.524Z
	  card-last-score:: 5
		- Selecting the highest-probability token at every generation step.
	- What does temperature control during generation? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:18.764Z
	  card-last-reviewed:: 2026-07-18T15:49:18.764Z
	  card-last-score:: 5
		- The sharpness of the next-token probability distribution; lower values make output more deterministic, while higher values increase randomness.
	- What is top-p sampling? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:48:37.581Z
	  card-last-reviewed:: 2026-07-18T15:48:37.581Z
	  card-last-score:: 5
		- Sampling from the smallest set of likely tokens whose cumulative probability reaches a chosen threshold, p.
	- Why can the same prompt produce different answers? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:01.449Z
	  card-last-reviewed:: 2026-07-18T15:49:01.451Z
	  card-last-score:: 5
		- Probabilistic sampling can select different tokens at each generation step, and those choices change later predictions.
- ## Capabilities, Limits, and Evaluation
	- What is in-context learning? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:51:33.868Z
	  card-last-reviewed:: 2026-07-18T15:51:33.868Z
	  card-last-score:: 5
		- Adapting behavior from instructions or examples in the prompt without changing the model's parameters.
	- What is retrieval-augmented generation (RAG)? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:28.701Z
	  card-last-reviewed:: 2026-07-18T15:49:28.701Z
	  card-last-score:: 5
		- Retrieving relevant external information and placing it in the model's context before it generates an answer.
	- How is RAG different from fine-tuning? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:48:05.934Z
	  card-last-reviewed:: 2026-07-18T15:48:05.935Z
	  card-last-score:: 5
	  collapsed:: true
		- RAG supplies information in the prompt at inference time, while fine-tuning changes the model's parameters through training.
	- What is a hallucination in a language model? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:49:51.191Z
	  card-last-reviewed:: 2026-07-18T15:49:51.192Z
	  card-last-score:: 5
		- A fluent but unsupported, misleading, or false output generated as though it were reliable.
	- Why does next-token prediction not guarantee factual accuracy? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:51:01.964Z
	  card-last-reviewed:: 2026-07-18T15:51:01.964Z
	  card-last-score:: 5
		- The objective rewards predicting plausible continuations of text, not independently verifying that statements are true.
	- What is perplexity? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:20.416Z
	  card-last-reviewed:: 2026-07-18T15:50:20.416Z
	  card-last-score:: 5
		- A measure derived from average token prediction loss; lower perplexity means the model assigns higher probability to the evaluated text.
	- Why is a held-out test set used? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:48:14.498Z
	  card-last-reviewed:: 2026-07-18T15:48:14.498Z
	  card-last-score:: 5
		- To estimate how well a model generalizes to data it did not train on.
	- What is a benchmark? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:48:26.257Z
	  card-last-reviewed:: 2026-07-18T15:48:26.257Z
	  card-last-score:: 5
		- A standardized dataset or task used to compare model performance on a defined capability.
	- Why can benchmark scores give an incomplete picture of model quality? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:51:29.952Z
	  card-last-reviewed:: 2026-07-18T15:51:29.952Z
	  card-last-score:: 5
		- They may miss real-world behavior, contain contaminated or unrepresentative data, or measure only a narrow capability.
	- What is the difference between a language model and a chatbot? #card
	  card-last-interval:: 4
	  card-repeats:: 1
	  card-ease-factor:: 2.6
	  card-next-schedule:: 2026-07-22T15:50:52.454Z
	  card-last-reviewed:: 2026-07-18T15:50:52.455Z
	  card-last-score:: 5
		- A language model is the predictive model itself; a chatbot is an application that adds prompts, interfaces, tools, retrieval, memory, and safety controls around one or more models.