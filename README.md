### **Case Study: Monogatari, an AI-Powered Language Learning Platform**

**Project Status:** Live and In-Development
**Role:** Sole Developer (Full-Stack)
**Live Application:** [Monogatari.ai](https://monogatari.ai/)

---

### **1. Project Overview**

Monogatari is a full-stack web application designed to solve a common problem for beginner through early advanced language learners: the lack of engaging, level-appropriate native reading material. The application leverages generative AI to create unique, bilingual stories tailored to a user's specific learning level, helping to bridge the gap between beginner textbooks and the overwhelming complexity of native-level content.

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



React Frontend: The user's entry point.
    Arrow to ->

Orchestrator API (FastAPI): Receives the initial request.
    Arrow back to -> React Frontend (with UUID)
    Arrow to ->

Google Cloud Tasks (Task Queue): Shows you are offloading the work, not handling it in the web request.
    Arrow to ->

Generation Pipeline (The Main Service): This is the service triggered by Cloud Tasks.
    Arrows to -> Google Gemini (for outline/expansion) and MongoDB (to save initial state).
    Multiple Arrows to ->

Slide Process Worker: A distinct block. The multiple arrows from the Generation Pipeline visually represent the "fan-out" pattern.
    Arrows from this worker to -> Google Gemini and Google TTS.
    Multiple Arrows back to -> The Generation Pipeline (this is the "fan-in").

MongoDB (Database): The final destination for the completed story. The Generation Pipeline writes the final result here.

### **4. Technology Stack**

* **Frontend:** React, Typescript, Tailwind CSS
* **Backend:** Python, FastAPI (with AsyncIO and HTTPX for concurrent API calls)
* **Database:** MongoDB (hosted on MongoDB Atlas)
* **AI & ML Services:** Google Gemini (for structured data generation), Google Cloud Text-to-Speech
* **Deployment & Hosting:** Google Cloud Platform, Uvicorn/Gunicorn, Docker

### **5. Technical Challenges & Solutions**

This project involved several complex technical challenges. Below are a few key examples that highlight my problem-solving approach.

### **Challenge 1: Balancing Quality, Speed, and Cost in a Generative AI Pipeline**

The primary engineering challenge of Monogatari was not simply to generate stories, but to do so within the competing constraints of three critical business and user-experience goals:

* **Quality:** Generating high-quality, pedagogically sound, and logically coherent learning materials.
* **Speed:** Ensuring the generation process is fast enough to keep the user engaged.
* **Cost:** Minimizing the expense of LLM API calls to keep the service affordable for users and viable as a business.

#### **Evolution of the Solution**

The system evolved significantly to meet these competing demands.

**Phase 1: Initial Proof of Concept**
The first iteration aimed simply to validate the core concept. It used a single, large API call to the LLM, prompting it to create a full bilingual story based on a classic folktale. While successful in proving the basic idea, and was both fast and cheap, this monolithic approach offered poor control over quality and gave no extra pedagogical value.

**Phase 2: The Shift to Specialization**
It quickly became clear that a single prompt could not simultaneously optimize for all three goals. The solution was to break down the monolithic task into a specialized, multi-step pipeline. The prompt was split, refined, and recombined through numerous iterations, with each part of the pipeline being tailored for a singular purpose.

**Phase 3: The Current Architecture**
The current architecture is the culmination of this iterative process, a distributed system designed to balance the trilemma:

1.  **Sequential Calls for Quality:** To ensure logical coherence and appropriate length/difficulty for the learners level, story generation is a two-step sequential process. The first API call generates a high-level plot outline, and a second, separate call expands that outline into a full narrative. This dramatically improved the quality and consistency of the stories.

2.  **Parallel Processing for Speed:** To minimize user wait time, the most time-consuming part of the pipeline—the linguistic analysis and audio generation for each slide—is "fanned-out" to a decoupled worker service. Dozens of parallel, asynchronous API calls are made, and the results are "fanned-in" when complete. This concurrent processing drastically reduces the total generation time compared to a sequential approach.

3. **Granular Prompts for Cost Efficiency:** Cost is managed through a deliberate prompt engineering strategy that avoids large, monolithic API calls. The architecture relies on a series of smaller, highly-focused prompts, each optimized in two primary ways:
* **Task Specialization:** Each prompt is given a singular, well-defined task (e.g., "generate only an outline," or "analyze only this sentence"). This on average reduces the thinking tokens used by the model to execute the job, lowering the output token cost of each call as well as resulting in faster responses.
* **Token Efficiency:** Every prompt has been iteratively refined to produce the desired high-quality, structured output using the fewest possible tokens.
This granular approach ensures that the operational costs for both the business and the end-user are kept as low as possible, without sacrificing the quality or speed of the generation process.

### **Challenge 2: Evaluating Story Generation Pricing Through Analysis of Fixed Costs, Operational Costs, LLM API Costs**

### **Challenge 3: Adding Maxium Pedagogical Value**

### **Challenge 4: Intuitive and Minimalist UI Design**

### **6. Future Development**

The project is designed with a clear roadmap for future enhancements, focusing on deepening the pedagogical value and optimizing the business model.

* **Data-Driven Optimization:** Implement a dedicated statistics table to track API costs, generation times, and user ratings, allowing for data-informed prompt engineering and feature development.
* **Enhanced User Engagement:** Introduce features like story ratings, progress tracking, and gamification to improve user retention.
* **Active Knowledge Checks:** Integrate a means of testing a users understanding of the generated story
* **Conversation Mode**: Utilizing the existing breakdown system, allow users to have conversations while leveraging the analysis system for aiding in understanding
* **Chatbot Tutor**: Add a chatbot style interactive window allowing the user to clarify any additional questions about the learning material, where the LLM has full context on the material in question
