### **Case Study: Monogatari, an AI-Powered Language Learning Platform**

**Project Status:** Live and In-Development
**Role:** Sole Developer (Full-Stack)
**Live Application:** [Monogatari.ai](https://monogatari.ai/)

---

### **1. Project Overview**
Monogatari is a full-stack web application born from my personal journey of achieving fluency in Japanese as an adult. My own learning was defined by a process of heavy immersion, constantly reading native materials while cross-referencing dictionaries and grammar guides. This project's core mission is to replicate the pedagogical value of that immersive process in a streamlined, user-friendly package.

The application leverages a backend AI generation pipeline to create unique, bilingual stories tailored to a user's learning level, bridging the gap between beginner textbooks and the seemingly overwhelming complexity of native-level content.

To deliver a modern and highly interactive user experience, I independently learned and implemented the frontend using React, TypeScript, and Tailwind CSS. This involved leveraging AI code assistants like Google Gemini to rapidly prototype components, enforce best practices, and accelerate the learning process. This project serves as a comprehensive demonstration of my ability to architect and deploy a complete, AI-integrated product, encompassing both a complex, distributed backend and a modern, type-safe frontend from the ground up, culminating in a live, monetized application architected for performance, data-driven optimization, and a global audience.

### **2. Core Features**

* **Dynamic Story Generation:** Users can specify a target language, difficulty (CEFR A1-C1), length, style, and more to generate a unique story.
* **Bilingual Reading Interface:** A clean, synchronized view displays the story, alongside grammar and vocabulary explanations, in both the target language and the user's native language.
* **Interactive Vocabulary and Audio:** Users can hover over any word to get its definition and reading, and click a word to hear it pronounced. Slides and individual words feature text-to-speech audio to aid in pronunciation and listening practice.
* **User Content Library:** Generated stories are saved to a user's account for future reading and review, with export options for Anki flashcard deck export to review and powerpoint export for teachers to use in teaching lessons.
* **[Upcoming Feature] Passage Analysis Mode:** A new primary mode that allows users to input text they need help understanding for a complete linguistic breakdown, applying the same analysis engine used for generated stories.

### **3. Technical Architecture**

The system is built on a distributed, asynchronous architecture to handle the long networking I/O bound process of AI story generation and analysis efficiently. When a user submits a request, the orchestrator service immediately returns a job ID and delegates the task to Google Cloud Tasks.

A backend pipeline then executes the core logic, first generating a coherent story outline and expansion in two sequential steps. To maximize efficiency, the pipeline then employs a fan-out/fan-in pattern for the most intensive work: it makes dozens of parallel, asynchronous API calls to a separate, decoupled worker service to process each slide. This worker service is responsible for the fine-grained AI analysis and TTS audio generation.

Once all parallel slide-processing tasks are complete, the pipeline 'fans in' the results, assembles the final story object, and persists it to the database. The user's browser polls a status endpoint throughout this process, providing a seamless, real-time view of the generation progress. Integral to this architecture is a dedicated data pipeline that logs performance and cost metrics for every AI API call, enabling continuous, data-driven optimization of the entire system.

![Monogatari Architecture Diagram](https://i.imgur.com/ag5FGSt.png)

### **4. Technology Stack**

* **Frontend:** React, Typescript, Tailwind CSS
* **Backend:** Python, FastAPI (with AsyncIO and HTTPX for concurrent API calls)
* **Database:** MongoDB (hosted on MongoDB Atlas)
* **AI & ML Services:** Google Gemini (for structured data generation), Google Cloud Text-to-Speech
* **Deployment & Hosting:** Google Cloud Platform, Uvicorn/Gunicorn, Docker

This stack was chosen for Python's strength in AI, FastAPI's high performance for asynchronous operations, React's robust ecosystem for building interactive UIs, and the high quality of Gemini's multi-lingual output and Vertex AI TTS options.

### **5. Technical Challenges & Solutions**

This project involved several complex technical challenges. Below are a couple key examples that highlight my problem-solving approach.

### ## Challenge 1: Architecting a High-Quality, Cost-Effective, and Performant AI Pipeline

As the sole architect, the primary engineering challenge of Monogatari was to continuously balance three competing goals from its inception: high pedagogical quality, fast generation speed (<1 min average case for A1-A2 level), and low operational cost. The initial single-prompt approach failed to meet the quality requirement, leading to the development of a specialized, distributed system that addressed both user-facing and business-facing needs through dozens of pipeline and prompt iterations.

* **Solution for User Experience (Quality & Speed):** To ensure a high-quality and responsive user experience, the pipeline was architected for both narrative coherence and performance. For quality, a multi-stage process was implemented, starting with a high-level story outline that is then expanded into a full narrative and afterwards analyzed sentence by sentence to generate slides. For speed, the system was designed to be fully asynchronous. An initial non-blocking API call triggers a Google Cloud Tasks job, which then "fans-out" parallel requests for slide processing, reducing generation time by 90-98% over sequential generation.

* **Solution for Business Viability (Resilience & Cost):** To ensure the service was reliable and financially sustainable, I implemented a comprehensive backend statistics pipeline to track cost and latency for every individual API call. For resilience, a major problem was solving "hanging" API calls that caused jobs to fail after 10+ minutes. A timeout was introduced based on longest expected LLM response times, and further log analysis revealed a statistical anomaly: response times were not clustering around a mean per prompt type as I had expected, but instead returning in a staggered, linear pattern. This suggested a non-obvious rate-limiting issue. I validated this hypothesis by introducing a Semaphore to control concurrency, which confirmed the expected behavior. This data-driven analysis was the primary driver for architecting a decoupled, auto-scaling microservice to isolate and manage these concurrent calls, resolving the bottleneck and improving system reliability. This in turn allowed for a far tighter timeout window to handle the hanging calls. For cost, implementing token tracking revealed that Gemini's "thinking" tokens were a significant unexpected cost (a ~2.4x factor on total tokens), which informed a prompt optimization strategy to defer non-essential generation tasks.

**## Challenge 2: Designing a Flexible and Secure Monetization Model**

**The Problem:**
The application required a monetization strategy that could cater to different user habits—from casual, infrequent users to dedicated power users. The design needed to be flexible, fully automated, and abstract away the variable, per-API-call cost of generation into a predictable model for the user.

**The Design & Solution:**
I designed a hybrid credit and subscription model to meet these diverse user needs.
* **System Design:** The core of the system is a credit-based ledger where each generation task deducts a pre-defined number of credits. This design provides users with clear, predictable pricing. To serve both user types, I implemented two purchasing options: one-time credit packs for casual users and a monthly subscription that provides a recurring allotment of credits at a discounted rate for power users.
* **Secure Implementation:** To execute this design, I integrated the Stripe API. A secure, server-side Stripe Webhook endpoint handles automated fulfillment, ensuring that user credits are only updated after a verified, successful payment event. This event-driven approach is critical for reliability and prevents client-side manipulation, forming the secure backbone of the entire monetization system.

### **6. Future Development**

The project is designed with a clear roadmap for future enhancements. Development will be prioritized based on user feedback and data analysis, with a focus on features that most directly enhance the core learning loop.

* **Passage Analysis Mode:** A new primary mode that allows users to input text they need help understanding for a complete linguistic breakdown, applying the same analysis engine used for generated stories.
* **Continuous Prompt & Model Optimization:** Actively use the new data analytics pipeline to continuously refine prompts, test new AI models, and further reduce generation costs and latency.
* **Enhanced User Engagement:** Introduce features like story ratings, progress tracking, and gamification to improve user retention.
* **Active Knowledge Checks:** Integrate a means of testing a users understanding of the generated story.
* **Conversation Mode**: Utilizing the existing breakdown system, allow users to have conversations while leveraging the analysis system for aiding in understanding.
* **Chatbot Tutor**: Add a chatbot style interactive window allowing the user to clarify any additional questions about the learning material, where the LLM has full context on the material in question.
