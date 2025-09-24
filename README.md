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

![Monogatari Architecture Diagram](https://i.imgur.com/SjumENy.png)

### **4. Technology Stack**

* **Frontend:** React, Typescript, Tailwind CSS
* **Backend:** Python, FastAPI (with AsyncIO and HTTPX for concurrent API calls)
* **Database:** MongoDB (hosted on MongoDB Atlas)
* **AI & ML Services:** Google Gemini (for structured data generation), Google Cloud Text-to-Speech
* **Deployment & Hosting:** Google Cloud Platform, Uvicorn/Gunicorn, Docker

### **5. Technical Challenges & Solutions**

This project involved several complex technical challenges. Below are a couple key examples that highlight my problem-solving approach.

### ## Challenge 1: Architecting a High-Quality, Cost-Effective, and Performant AI Pipeline

As the sole architect, the primary engineering challenge of Monogatari was to continuously balance three competing goals from its inception: high pedagogical quality, fast generation speed (<1 min best case), and low operational cost. The initial single-prompt approach failed to meet the quality requirement, leading to the development of a specialized, distributed system that addressed both user-facing and business-facing needs through dozens of pipeline and prompt iterations.

* **Solution for User Experience (Quality & Speed):** To ensure a high-quality and responsive user experience, the pipeline was architected for both narrative coherence and performance. For quality, a multi-stage process was implemented, starting with a high-level story outline that is then expanded into a full narrative. For speed, the system was designed to be fully asynchronous. An initial non-blocking API call triggers a Google Cloud Tasks job, which then "fans-out" parallel requests for slide processing, reducing generation time by 90-98% over sequential generation.

* **Solution for Business Viability (Resilience & Cost):** To ensure the service was reliable and financially sustainable, the backend was hardened and optimized. For resilience, a major problem was solving "hanging" API calls that caused jobs to fail after 10+ minutes. A timeout was introduced based on longest expected LLM response times, and further log analysis revealed a statistical anomaly: response times were not clustering around a mean per prompt type as expected, but returning in a staggered, linear pattern. This suggested a non-obvious rate-limiting issue. I validated this hypothesis by introducing a Semaphore to control concurrency, which confirmed the behavior. This data-driven analysis was the primary driver for architecting a decoupled, auto-scaling microservice to isolate and manage these concurrent calls, resolving the bottleneck and improving system reliability. This in turn allowed for a far tighter timeout window to handle the hanging calls. For cost, implementing token tracking revealed that Gemini's "thinking" tokens were a significant unexpected cost (a ~2.4x factor on total tokens), which informed a prompt optimization strategy to defer non-essential generation tasks.

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
