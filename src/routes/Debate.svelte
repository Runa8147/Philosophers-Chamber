<!-- App.svelte -->
<script>
    import { onMount } from 'svelte';
    import { GoogleGenerativeAI } from "@google/generative-ai"; // Import the AI library
    import { marked } from 'marked'; // Import the marked library

    const API_KEY = "apikey0"; // Replace with your actual API key
    const genAI = new GoogleGenerativeAI(API_KEY);

    let messages = [];
    let userInput = '';
    let isLoading = false;
    let currentSpeaker = 'Socrates'; 
    let debateActive = false; 

    // Define prompts for each speaker
    const socratesPrompt = `You are Socrates, employing the Socratic method to challenge and question. Focus on insightful questions, exposing weaknesses in logic and assumptions. Respond concisely (under 100 words).`;
    const aristotlePrompt = `You are Aristotle, responding logically and providing well-reasoned arguments based on your vast knowledge and experience. Respond concisely (under 100 words).`;

    async function getAIResponse(input, speaker) {
        try {
            const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash-002" });
            // Use the appropriate prompt based on the current speaker
            const prompt = speaker === 'Socrates' ? socratesPrompt : aristotlePrompt;
            const result = await model.generateContent(`${prompt}\n${input}`);
            return result.response.text(); // Return the generated response
        } catch (error) {
            console.error("Error generating response:", error);
            return "Sorry, I encountered an error. Please try again.";
        }
    }

    function speak(text) {
        const utterance = new SpeechSynthesisUtterance(text);
        utterance.onend = () => {
            // Continue the debate after the speech ends
            handleNextSpeaker();
        };
        window.speechSynthesis.speak(utterance);
    }

    async function handleNextSpeaker() {
        if (!debateActive) return;

        // Get AI response
        const response = await getAIResponse(userInput, currentSpeaker);
        
        // Add AI response
        messages = [...messages, {
            role: currentSpeaker.toLowerCase(),
            content: response,
            timestamp: new Date()
        }];

        // Speak the response
        speak(response);

        // Switch speakers
        currentSpeaker = currentSpeaker === 'Socrates' ? 'Aristotle' : 'Socrates';
        userInput = response; // Use the last response as the next input
    }

    async function handleSubmit() {
        if (!userInput.trim() || isLoading || debateActive) return; 
        
        isLoading = true;
        debateActive = true; 
        
        // Add user message
        messages = [...messages, {
            role: 'user',
            content: userInput,
            timestamp: new Date()
        }];

        // Start the debate loop
        await handleNextSpeaker(); // Start the debate with the first response
        isLoading = false;
    }

    function stopDebate() {
        debateActive = false; // Stop the debate
        window.speechSynthesis.cancel(); // Stop any ongoing speech
    }

    // Function to convert Markdown to HTML
    function renderMarkdown(content) {
        return marked(content);
    }
</script>

<main class="container mx-auto px-4 py-8 max-w-3xl">
    <h1 class="text-3xl font-bold mb-6 text-center">Philosophical Debate: Socrates vs Aristotle</h1>
    
    <div class="bg-white rounded-lg shadow-lg p-6">
        <div class="mb-4 h-96 overflow-y-auto">
            {#each messages as message (message.timestamp)}
                <div class="mb-4" class:text-right={message.role === 'user'}>
                    <div class="inline-block max-w-[80%] p-3 rounded-lg"
                         class:bg-blue-100={message.role === 'user'}
                         class:bg-green-100={message.role === 'socrates'}
                         class:bg-yellow-100={message.role === 'aristotle'}>
                        <p class="font-bold mb-1">
                            {#if message.role === 'user'}
                                You
                            {:else if message.role === 'socrates'}
                                Socrates
                            {:else}
                                Aristotle
                            {/if}
                        </p>
                        <p>{@html renderMarkdown(message.content)}</p> <!-- Render Markdown here -->
                    </div>
                </div>
            {/each}
            
            {#if isLoading}
                <div class="flex justify-center">
                    <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-gray-900"></div>
                </div>
            {/if}
        </div>

        <div class="flex gap-2">
            <input
                type="text"
                bind:value={userInput}
                placeholder="Join the philosophical debate..."
                class="flex-1 p-2 border rounded"
                on:keydown={e => e.key === 'Enter' && handleSubmit()}
                disabled={isLoading || debateActive} 
            />
            <button
                on:click={handleSubmit}
                disabled={isLoading || debateActive} 
                class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 disabled:bg-gray-400"
            >
                Start Debate
            </button>
            <button
                on:click={stopDebate}
                disabled={!debateActive} 
                class="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600 disabled:bg-gray-400"
            >
                Stop Debate
            </button>
        </div>
    </div>

    <div class="mt-4 text-center text-sm text-gray-600">
        Current Speaker: {currentSpeaker}
    </div>
</main>

<style>
    :global(body) {
        background-color: #f0f2f5;
    }
</style>
