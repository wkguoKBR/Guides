# Developing a Retrieval-Augmented Generation (RAG) application using Langchain and Python (Friday, September 06 2024)

## Introduction

This guide will walk through how to implement your own RAG application using Langchain and Python. We will cover how to process user-provided PDF documents for efficient information retrieval and leverage Large Language Models (LLMs) to generate contextually relevant and semantically-driven responses based on the content of those documents.

## What is RAG?

RAG stands for retrieval-augmented generation, which according to AWS, is "the process of optimizing the output of a large language model ... so it references an authoritative knowledge base outside of its training data sources before generating a response". A common caveat of LLMs is that they are limited to only their pre-trained data and thus cannot reliably act upon proprietary data sources. RAG addresses this by adding an external data retrieval step to pull in and store information from sources such as a document repository in a vector database, thereby allowing LLMS to access such content and act upon dynamic, user-provided data.

## Instructions

The tutorial will be split into two main sections. The first will be on creating a RAG project that operates through the terminal. The second expands the project by integrating a user-friendly and interactive UI using Streamlit to create a fully-fledged RAG chatbot.

### Part 1: Constructing a Terminal-Based RAG Application 

Open the [RAG-Tutorial](https://github.com/wkguoKBR/RAG-Tutorial) GitHub repo to follow along and access the entire completed project. 

#### Step 1. Install the required packages

1) 

### Part 2: Enhancing the RAG Application with Streamlit UI
