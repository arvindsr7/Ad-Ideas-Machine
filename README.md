## Ad Ideas Machine

# Motivation

For businesses, advertisement is a crucial part of creating brand awareness and increasing sales. Every business — small, large, local, or a startup — desires to advertise its products to the targeted audience. A large fraction of these advertisements are on digital platforms, such as Facebook, Instagram, Websites, etc, where for effective advertising, businesses not only need to advertise frequently, but also need a lot of variations of their ads. Different ads leave a long lasting impression on the audience while seeing the same ad again and again can get boring. Thus, the advertisements demand creativity. Going into pandemic, the rate of shift to the digital world has only accelerated, creating a huge demand for creative advertising. 

Now, developing advertisements, particularly creative ones, can get very expensive. Small businesses don’t usually have dedicated staff for running ads. They end up hiring advertising agencies for their ad campaigns, and these agencies charge a lot of money. Alternatively, they don’t advertise and their business suffers. 

Machine learning has transformed so many things around us: search, recommendations, predications, even music generation and creating real-looking paintings! Could we do something to reimagine the advertising industry as well?

In this project, I wanted to explore exactly that. I wanted to explore the possibility of generating ad ideas for users for their ad campaigns. Specifically, I wanted to utilize existing state-of-the-art Machine Learning models in Computer Vision and Natural Language Processing, integrate them in a pipeline and finetune them to provide advertisement ideas for users. 


# Implementation Flow 

There isn’t a good dataset of advertisement images and their text descriptions, so developing an ad suggestion model using it seemed a long and arduous process. But engineering is all about simplifying the process and building stuff from what we know, what we have, and what we can innovate. I approached my project as following:

1. I want to understand users requirements, so I’ll take some required advertisement related image input from users as reference. These could be existing ad posters users can find on Google. 
2. Understand theme of advertisement from scene understanding of image, understand context from optical character recognition of taglines from image, and obtain a text description of the user requirement
3. In recent times, the capability of language models has evolved exponentially thanks to Transformer based models. Utilising this capability, particularly using a OpenAI’s GPT-3 API that I luckily got hold of as a beta tester, I would generate more ideas from the text description I generated in step 2. 
4. After this step, ideally I wanted to generate images using another OpenAI model DALL-E that can generate some really cool images. However, the DALL-E API isn’t released to the public yet, so I decided to search a database of images from the ideas I generated in step 3. 
5. Finally, score the best images obtained from search using another captioning based metric, and show the selected images to the user as ad ideas. 


# The Pipeline

Now, I’ll start to delve into the technicalities of implementation. First, here’s a simplified block diagram of the Machine Learning system. 
![image](https://user-images.githubusercontent.com/62667772/111731412-a8fea300-8830-11eb-9778-1fcb3d24a10c.png)

I have five models in total in the Ad Ideas Machine ML system:
1. Optical Character Recognition Model for extracting texts from images
2. Image Captioning Model for scene understanding
3. GPT-3 API for generating ideas from image caption and OCR descriptions of the image
4. Profanity Filter Model for filtering ideas generated from GPT-3 that may be inappropriate 
5. Unsplash Search Model for searching relevant images from unsplash.com image dataset

As described earlier, I generate a description of requirements from the image input that the user provides, and these are done using two computer vision models: Imaging Captioning and OCR. The text description output is fed into GPT-3 API for “Idea completion” using it’s Babbage engine. It extends the text description into multiple dimensions beyond simple imagination. Now why do we have the profanity filter? It turns out that sometimes, not so frequently though, the ideas generated by GPT-3 aren’t “appropriate.” So, I pass the output of GPT-3 model to Profanity Filter model that analyses it, and if only the filter model greelights the GPT-3 output, the generated idea is used for image search. If it isn’t, GPT-3 is triggered again to generate more ideas which are again passed through the Profanity Filter model. 

I ideally wanted to use DALL-E API for ad ideas and that would have been pretty cool, but in absence of DALL-E’s API, I used a pre-trained Unsplash search model which searches generates image id of unsplash.com images that have the closest word description embedding to GPT-3 ideas. 

Now comes the training part. Most of the models worked pretty well and/or I didn’t have any scope of finetuning. For example, I don’t have access to the model parameters for GPT-3; I can only use the API provided to me. Similarly, Unsplash, OCR, and Profanity filter models can’t be specifically improved for the task in hand.

For image captioning, I first started with a model on GitHub, and I tried to create a small (about 150 images) ad images, text description data, and finetune the pretrained model on this dataset. But this dataset was too small to make a difference on models trained with large MS-COCO dataset. Next, while standalone training, done in Google Colab, was fine, when I tried to incorporate the model into my pipeline, it ran into a lot of dependency issues with the other models. This image caption model is about an year old, and in the ML world, an year can be too old. So, I decided to switch gears and use another pretrained image captioning model from Microsoft, which too was accessed as an API. The code for different models can be found in the GitHub repository of this project. 


# User Interface

There’s a web of intricate models running in the background, but we have to keep them to not overwhelm the user with all complexities. So, for demo purposes, I decided to provide a simple yet elegant user interface using Python’s Flask library. It allowed me to create a design website which people could use to interact with the model running in the backend. ![image](https://user-images.githubusercontent.com/62667772/111731566-f9760080-8830-11eb-8cfe-4116afa403dd.png)

Here, users could either upload a reference image from their personal computer or provide the URL of an image they find interesting on the internet. Once, they hit Upload or Go, the image is rendered on the screen. ![image](https://user-images.githubusercontent.com/62667772/111731576-0135a500-8831-11eb-9998-980037800184.png)

Afterwards, when they hit Generate Ideas, the model processes the image in the backend to generate ideation images, concatenates them into a single image and uploads on the web page to display to the users. ![image](https://user-images.githubusercontent.com/62667772/111731592-08f54980-8831-11eb-9486-32c42d5e5759.png)

Showing multiple images in a fancier format was something too complicated for Flask, that’s why I decided to concatenate images in the backend before displaying to the user.  


# Results

The ML system provided interesting suggestions for many images. They varied from simple representations to very intricate images. ![image](https://user-images.githubusercontent.com/62667772/111731620-17436580-8831-11eb-8863-3904fb57a93d.png)

At the same time, the quality of these images is low (ignoring image scaling part, which arises due to concatenating all unsplash output images to same size), and these arise for two reasons:
1. Unsplash image dataset: Unsplash image dataset contains images from unsplash website. These images are user captured images, they are raw and unpolished. They were not intended for advertisement purposes per se. 
In addition, the Unsplash ML model does a bag of word search, which loses all the semantics and context of image caption and ideas generated by GPT-3, and thus resulting in much lower quality images. 
This brings back my original point that having the DALL-E API would have been pretty neat. One, it captures semantics and contexts of text description, and two, it “generates” images, not searches from existing images. But for now, Unsplash search was the best I could do.
2. Image captioning: A major bottle of my ML system is how well the image captioning model works on the user provided images. Sometimes, it fully disappoints, and in those cases, the rest of the models in the pipeline can’t do anything. For example, for the image below, the image captioning model output caption as “diagram.” That’s all. 
To overcome this limitation, a nice add-on would be to take some text inputs from the user as well, which would be strictly considered in the final image search. However, I decided against doing it for the demo purpose to not make it too complicated.

In conclusion, I developed an ML system that provided advertisement ideas from minor inputs from users. It was an end-to-end system consisting of different computer vision models (OCR, Image Captioning) and natural language processing models (GPT-3, Profanity Filter). It was a super fun project with a moderate amount of success, fitting for the complexity of the problem and lack of relevant datasets and models. 


# References

1. [hasnainroopawalla/Image-Captioning-Scene-Descriptor: A CNN-LSTM model to generate image captions](https://github.com/hasnainroopawalla/Image-Captioning-Scene-Descriptor)
2. [Quickstart: Computer Vision client library - Azure Cognitive Services](https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/quickstarts-sdk/client-library?tabs=cli&pivots=programming-language-python)
3. [faustomorales/keras-ocr: A packaged and flexible version of the CRAFT text detector and Keras CRNN recognition model](https://github.com/faustomorales/keras-ocr#egg=keras-ocr)
4. [profanity-check · PyPI](https://pypi.org/project/profanity-check/)
5. [haltakov/natural-language-image-search: Search photos on Unsplash using natural language](https://github.com/haltakov/natural-language-image-search)
6. [OpenAI GPT-3 Idea Generation](https://beta.openai.com/docs/examples/idea-generation)
