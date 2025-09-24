### **Case Study: Monogatari, an AI-Powered Language Learning Platform**

**Project Status:** Live and In-Development
**Role:** Sole Developer (Full-Stack)
**Live Application:** [Monogatari.ai](https://monogatari.ai/)

---

### **1. Project Overview**
Monogatari is a full-stack web application born from my personal journey of achieving fluency in Japanese as an adult. My own learning was defined by a process of heavy immersion—laboriously reading native materials while constantly cross-referencing dictionaries and grammar guides. This project's core mission is to replicate the pedagogical value of that immersive process in a streamlined, user-friendly package.

The application leverages a sophisticated AI backend to create unique, bilingual stories tailored to a user's specific learning level, bridging the gap between beginner textbooks and the overwhelming complexity of native-level content.

To deliver a modern and highly interactive user experience, I independently learned and implemented the frontend using React, TypeScript, and Tailwind CSS. This involved leveraging LLMs as a development assistant to rapidly prototype components, enforce best practices, and accelerate the learning process. This project serves as a comprehensive demonstration of my ability to architect and deploy a complete, AI-integrated product, encompassing both a complex, distributed backend and a modern, type-safe frontend.

This project serves as a comprehensive demonstration of my ability to architect, develop, and deploy a modern, AI-integrated product from the ground up.

### **2. Core Features**

* **Dynamic Story Generation:** Users can specify a target language, difficulty (A1-C1), length, tone, and more to generate a unique story.
* **Bilingual Reading Interface:** A clean, synchronized view displays the story, alongside grammar and vocabulary explanations, in both the target language and the user's native language.
* **Interactive Vocabulary and Audio:** Users can hover over any word to get its definition and reading, and click a word to hear it pronounced. Slides and individual words feature text-to-speech audio to aid in pronunciation and listening practice.
* **User Content Library:** Generated stories are saved to a user's account for future reading and review, with export options for Anki flashcard deck export to review and powerpoint export for teachers to use in teaching lessons.
* **[Upcoming Feature] Passage Analysis Mode:** A new primary mode that allows users to input their own text for a complete linguistic breakdown, applying the same analysis engine used for generated stories.

### **3. Technical Architecture**

The system is built on a distributed, asynchronous architecture to handle the computationally expensive process of AI story generation without compromising API responsiveness. When a user submits a request, the orchestrator service immediately returns a job ID and delegates the task to Google Cloud Tasks.

A backend pipeline then executes the core logic, first generating a coherent story outline and expansion in two sequential steps. To maximize efficiency, the pipeline then employs a fan-out/fan-in pattern for the most intensive work: it makes dozens of parallel, asynchronous API calls to a separate, decoupled worker service to process each slide. This worker service is responsible for the fine-grained AI analysis and TTS audio generation.

Once all parallel slide-processing tasks are complete, the pipeline 'fans in' the results, assembles the final story object, and persists it to the database. The user's browser polls a status endpoint throughout this process, providing a seamless, real-time view of the generation progress.

![](https://imgur.com/SjumENy)

### **4. Technology Stack**

* **Frontend:** React, Typescript, Tailwind CSS
* **Backend:** Python, FastAPI (with AsyncIO and HTTPX for concurrent API calls)
* **Database:** MongoDB (hosted on MongoDB Atlas)
* **AI & ML Services:** Google Gemini (for structured data generation), Google Cloud Text-to-Speech
* **Deployment & Hosting:** Google Cloud Platform, Uvicorn/Gunicorn, Docker

### **5. Technical Challenges & Solutions**

This project involved several complex technical challenges. Below are a couple key examples that highlight my problem-solving approach.

### ## Challenge 1: Architecting a High-Quality, Cost-Effective, and Performant AI Pipeline

As the sole architect and developer, the primary engineering challenge of Monogatari was to continuously balance three competing goals from its inception:

* **Quality:** Generate high-quality, pedagogically sound, and logically coherent learning materials.
* **Speed:** Ensure the generation process for the simplest stories remained under a one-minute target to maintain user engagement.
* **Cost:** Minimize the expense of LLM API calls to keep the service affordable and viable as a business.

This narrative details the evolution of the system's architecture and prompt engineering strategy in pursuit of these goals.

---

### ## Evolution of the Solution

#### **Phase 1: From Concept to a Multi-Lingual Foundation**
The project began as a proof-of-concept to determine if generative AI could produce beginner-level bilingual stories comparable in quality to human-created teaching materials. The initial architectural step was to move from a freeform text response to a structured data model using `Story` and `Slide` objects.

A critical early decision was to use the LLM for linguistic annotations (like definitions and grammar points) rather than integrating language-specific NLP libraries. This decision was crucial for scalability, as it allowed the app to expand from one to eight languages **without requiring the integration of seven additional, specialized libraries, saving significant development time and cost.** The LLM's superior ability to determine contextual meaning, such as identifying a proper noun versus a common noun, further solidified this choice.

#### **Phase 2: The Shift to a Specialized, Asynchronous Pipeline**
As more features were added to the single, monolithic prompt, the quality of the narrative began to degrade. To solve this, the process was broken into a multi-phase pipeline, with each step handling a specialized task. While this improved quality, the sequential processing increased generation time to over 3.5 minutes, failing the project's speed requirement.

The key to resolving this was a shift to asynchronous processing. By parallelizing the most time-consuming part of the pipeline—the sentence-by-sentence annotation—using `asyncio.gather`, the generation time for simple stories was **reduced by 75%**, bringing it back under the one-minute target.

#### **Phase 3: Deep Optimization and Architectural Maturity**
With a functioning pipeline, the focus shifted to deep optimization based on user feedback and performance monitoring.

1.  **Enhancing Quality & User Choice:** To address feedback that higher-level stories were too simple, the pipeline was restructured to first generate a **narrative outline** and then expand it into a full story. User customization options for genre, tone, and length were also introduced, requiring the development of distinct prompt templates to maintain quality across different story types.
2.  **Data-Driven Cost Management:** As costs became a factor, token tracking was implemented to gain visibility into the pipeline's efficiency. This led to the critical discovery that Gemini's un-reported "thinking" tokens were undercounting actual costs by a factor of approximately **2.4x**. While analysis of this new data is ongoing, it immediately informed a pragmatic decision to "lighten up" the generation process by deferring non-essential, high-cost tasks—like the generation of multiple example sentences—from the main pipeline.
3.  **Building a Resilient, Distributed Architecture:** When logs revealed that some generations took over 10 minutes due to "hanging" API calls, the architecture was fundamentally changed. The long-running process was replaced with a **job-based system using Google Cloud Tasks**, allowing the user to poll for status updates without a persistent connection. Further investigation revealed a non-obvious rate-limiting behavior from the API. By implementing a **decoupled microservice** to handle concurrent slide generation, the issue was isolated and controlled. This final architectural change resolved the extreme wait times and significantly improved the system's overall resilience.

**## Challenge 2: Implementing a Secure and Automated Monetization System**

**The Problem:**
To be a viable product, the application required a secure and fully automated system to handle user payments, subscriptions, and the fulfillment of credits. The system needed to be reliable, secure against exploits, and provide a seamless user experience for purchasing and using credits.

**The Solution:**
I engineered an end-to-end monetization system by integrating the Stripe API.
* **Payment Processing:** Leveraged Stripe Checkout for a secure, PCI-compliant payment experience for both one-time credit purchases and recurring subscriptions.
* **Automated Fulfillment:** Implemented a Stripe Webhook endpoint in the FastAPI backend. This endpoint securely listens for successful payment events and automatically updates the user's credit balance in the MongoDB database, decoupling the fulfillment logic from the client-side interaction.
* **Credit System:** Designed and implemented a credit ledger system that safely handles the transactional logic of deducting credits upon story generation, ensuring that operations are atomic and reliable.

This solution demonstrates my ability to integrate a critical third-party API, handle secure financial transactions, and build a reliable, event-driven system to manage core business logic.

### **6. Future Development**

The project is designed with a clear roadmap for future enhancements, focusing on deepening the pedagogical value and optimizing the business model.

* **Data-Driven Optimization:** Implement a dedicated statistics table to track API costs, generation times, and user ratings, allowing for data-informed prompt engineering and feature development.
* **Enhanced User Engagement:** Introduce features like story ratings, progress tracking, and gamification to improve user retention.
* **Active Knowledge Checks:** Integrate a means of testing a users understanding of the generated story
* **Conversation Mode**: Utilizing the existing breakdown system, allow users to have conversations while leveraging the analysis system for aiding in understanding
* **Chatbot Tutor**: Add a chatbot style interactive window allowing the user to clarify any additional questions about the learning material, where the LLM has full context on the material in question
