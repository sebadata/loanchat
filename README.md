# loanchat

After a crash course on LLMs, I decided to use a two-step approach:

1. In the first step, a text classifier is used to decide whether the customer’s question can be answered by the LLM or not. If it doesn’t, a predefined fixed answer is given suggesting the customer to contact a senior officer. 
2. If it does, the second step is executed: the question is “forwarded” to the LLM. 

The LLM receives a particular kind of prompt, that “instructs” it to answer the question based solely on the information available in the provided context. This context is simply a non-structured text fragment containing a description of the loan’s document relevant parts. The relevant parts here are those corresponding to the information the LLM is expected to answer about, as indicated in the challenge instructions (in other words, those referring to the loan’s current status, APR, rate, term,
and amount approved and monthly payment). All the other parts of the document are not considered when building the context. A non-structured text is used (rather than the actual XML elements) because it seems to be easier for the LLM to “understand” the context that way (although it also does quite a decent job when using XML directly).  While I created the text manually (based on one of the provided files) it should be easy to write a program to automate it (at least based on what I see in the provided documents)

“Asking” the LLM to answer based solely on the context built in such a way not only helps in avoiding hallucinations, but also in preventing undesired loan information from being accidentally included in the answer.

The classifier in the first step helps to sanitize the question reaching the LLM, further reducing the risk of the LLM misbehaving through hallucinations or simply through accidentally answering question that are supposed to be forwarded to a human (which can still happen even if the context is a "prunned" version of the original document)


## About the classifier 

This classifier is obtained by fine-tuning a pre-trained model (BERT) using the provided labeled question set. 

This fine-tuning was implemented in a quick and dirty way. For example, when evaluating performance I limited myself to verifying that the fine-tuning lead to a decent roc auc (the dataset is quite unbalanced) compared to that achieved by the base model and to manually check some example questions.

## About the LLM

I did an attempt with the 7B and 13B versions of [Llama](https://huggingface.co/meta-llama) fine tuned for chat but the results were not great. I might be missing something or simply those versions perform badly compared to other models. I didn’t have time to play around with the different models, nor with platforms that might provide the resources needed for larger models, so I decided to go with the simplest option (from an usage point of view): OpenAI chatGPT.  This probably would not be convenient in practice from a cost point of view, but it helps giving an idea of what’s possible in terms of LLMs for the purposes of this challenge.

## How to run it
 
You can find the code [here](loanchat.ipynb). You can find it in Colab [here](https://colab.research.google.com/drive/17cvOKvsFceD4JBwEtP84s5mzx_BkLF_t?usp=sharing). To try it in Colab, choose a T4 GPU runtime and run each cell from top to bottom (you will need to provide your OpenAI API key, and upload the HalQuestions_StatusOnly_Test.csv provided in this repo to your drive)

The notebook is organized in 6 sections. The important ones are the last 3:
- Text classifier: Fine-tunes the classifier. The actual fine-tuning might take a couple of minutes.
- LLM: Defines the functions used to build the prompt for the LLM and plays around with a few examples.
- Loan assistant: The actual assistant with examples.
