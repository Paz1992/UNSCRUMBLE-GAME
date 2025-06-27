<html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game: Unscramble the Word</title>
    <!-- Chosen Palette: Warm Harmony - A calming palette using a light neutral background (#F0F2F5), white content areas, a soft blue (#88A0B2) for primary interactive elements, and subtle, harmonious accent colors for feedback and controls. -->
    <!-- Application Structure Plan: The application is designed as a single-view game interface to maintain user focus. The structure is vertical: Title, Score, Emoji Hint, Scrambled Letters, Answer Slots, Feedback, and Controls. This linear flow guides the user from the puzzle (emoji/letters) to the interaction area (slots) and finally to the action buttons (check/next). This was chosen for its simplicity and intuitive nature, making the game immediately understandable without instructions. -->
    <!-- Visualization & Content Choices: Report Info: Verbs list (word, scrambled, emoji). Goal: Teach words. Viz/Presentation: Interactive HTML blocks for letters/slots. Interaction: Click-to-move letters for simplicity. Justification: This direct manipulation is intuitive on both desktop and touch devices. Data (verbs) is stored in a JS array for easy management. Feedback is multi-sensory (color, text, sound, animation) to reinforce learning. Library/Method: Vanilla JS for all logic, Tailwind CSS for styling. -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700;800&family=Fredoka+One&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #F0F2F5; /* Light neutral background */
        }
        h1 {
            font-family: 'Fredoka One', cursive; /* Playful font for title */
        }
        .letter-block, .letter-slot {
            display: flex;
            justify-content: center;
            align-items: center;
            width: 60px; /* Slightly larger for kids */
            height: 60px; /* Slightly larger for kids */
            font-size: 2.25rem; /* Larger font for letters */
            font-weight: bold;
            border-radius: 9999px; /* Pill shape */
            cursor: pointer;
            user-select: none;
            transition: all 0.3s ease-in-out;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1), 0 2px 4px rgba(0, 0, 0, 0.06);
        }
        .letter-block {
            background-color: #FF7B7B; /* Vibrant Red */
            color: white;
            background-image: linear-gradient(to bottom right, #FF7B7B, #FF5C5C); /* Gradient */
        }
        .letter-block:hover {
            transform: translateY(-4px) scale(1.1); /* More pronounced hover */
            background-image: linear-gradient(to bottom right, #FF5C5C, #E04B4B); /* Darker gradient on hover */
        }
        .letter-slot {
            background-color: #E5E7EB; /* Light gray for empty slots */
            border: 3px dashed #9CA3AF; /* Thicker dashed border */
            color: #374151;
            box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1); /* Inner shadow for depth */
        }
        .letter-slot.filled {
            background-color: #6C7A89; /* Darker blue when filled */
            color: white;
            border-style: solid;
            border-color: #55606B;
            transform: scale(1.08); /* More pronounced scale */
            background-image: linear-gradient(to bottom right, #6C7A89, #55606B); /* Gradient for filled slots */
        }
        .feedback-message {
            transition: opacity 0.4s, transform 0.4s;
            font-family: 'Fredoka One', cursive; /* Apply playful font */
            font-size: 2.5rem; /* Larger feedback message */
        }
        @keyframes pop-in {
            0% { transform: scale(0.8); opacity: 0; }
            50% { transform: scale(1.1); opacity: 1; }
            100% { transform: scale(1); opacity: 1; }
        }
        .animate-pop { animation: pop-in 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55) forwards; } /* Bouncier animation */
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-10px) rotate(-3deg); } /* More exaggerated shake */
            20%, 40%, 60%, 80% { transform: translateX(10px) rotate(3deg); }
        }
        .animate-shake { animation: shake 0.5s ease-in-out; } /* Faster shake */

        /* Button styling */
        button {
            font-family: 'Fredoka One', cursive;
            padding: 1.25rem 1.5rem; /* Larger padding */
            border-radius: 9999px; /* Pill shape */
            box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15); /* More prominent shadow */
            transition: all 0.3s ease-in-out;
            font-size: 1.5rem; /* Larger font size for buttons */
        }
        button:hover {
            transform: translateY(-3px) scale(1.07); /* Lift and grow on hover */
            box-shadow: 0 8px 12px rgba(0, 0, 0, 0.2);
        }
        #resetButton {
            background-image: linear-gradient(to right, #FFC107, #FFA000); /* Orange gradient */
            color: white;
        }
        #resetButton:hover {
            background-image: linear-gradient(to right, #FFA000, #E68A00);
        }
        #checkButton {
            background-image: linear-gradient(to right, #4CAF50, #40A045); /* Green gradient */
            color: white;
        }
        #checkButton:hover {
            background-image: linear-gradient(to right, #40A045, #368C3B);
        }
        #nextButton {
            background-image: linear-gradient(to right, #2196F3, #1976D2); /* Blue gradient */
            color: white;
        }
        #nextButton:hover {
            background-image: linear-gradient(to right, #1976D2, #1565C0);
        }
        #nextButton.animate-pulse {
            animation: pulse 1.5s infinite;
        }
        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(33, 150, 243, 0.7); }
            70% { box-shadow: 0 0 0 15px rgba(33, 150, 243, 0); }
            100% { box-shadow: 0 0 0 0 rgba(33, 150, 243, 0); }
        }

        /* Emoji display */
        #verbEmoji {
            font-size: 8rem; /* Large emoji size */
            line-height: 1; /* Ensure no extra space above/below emoji */
            margin: auto; /* Center emoji vertically and horizontally */
        }
        .emoji-container {
            width: 100%;
            height: 48; /* same height as the previous image container */
            background-color: #CCE5FF; /* Light blue background for emoji container */
            border-radius: 0.75rem;
            overflow: hidden;
            box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <div id="game-container" class="bg-white p-6 sm:p-8 rounded-2xl shadow-2xl max-w-md w-full flex flex-col items-center gap-5">
        <h1 class="text-3xl sm:text-4xl font-extrabold text-gray-800 text-center">Unscramble the Word!</h1>
        
        <div class="text-2xl font-bold text-gray-700 bg-gray-100 px-4 py-2 rounded-full shadow-inner">
            Score: <span id="scoreDisplay">0</span>
        </div>

        <!-- Emoji hint container -->
        <div class="emoji-container w-full h-48 bg-blue-100 rounded-lg overflow-hidden shadow-inner flex items-center justify-center">
            <span id="verbEmoji" role="img" aria-label="Visual hint for the word"></span>
        </div>

        <p class="text-gray-600 text-center text-lg">Drag and drop the letters to form the correct word.</p>

        <div id="wordSlots" class="flex flex-wrap justify-center gap-2 sm:gap-3 p-2 min-h-[70px] w-full"></div>
        <div id="scrambledLetters" class="flex flex-wrap justify-center gap-2 sm:gap-3 p-3 bg-blue-50 rounded-lg w-full min-h-[70px]"></div>
        
        <div id="feedbackContainer" class="h-10 flex items-center justify-center">
            <p id="feedbackMessage" class="feedback-message text-xl font-semibold opacity-0"></p>
        </div>

        <div class="flex flex-col sm:flex-row gap-4 mt-2 w-full">
            <button id="resetButton" class="w-full sm:w-auto flex-1 text-white font-bold py-3 px-4 shadow-md">Reset</button>
            <button id="checkButton" class="w-full sm:w-auto flex-1 text-white font-bold py-3 px-4 shadow-md">Check</button>
        </div>
        <button id="nextButton" class="w-full text-white font-bold py-3 px-4 shadow-md opacity-50 cursor-not-allowed" disabled>Next</button>
    </div>

    <script>
        // Array of game data: word, scrambled version, and emoji character
        const verbs = [
            { word: "RUN", scrambled: "NUR", emoji: '🏃' },
            { word: "JUMP", scrambled: "MPUJ", emoji: '🤸' },
            { word: "EAT", scrambled: "TEA", emoji: '🍎' },
            { word: "READ", scrambled: "DARE", emoji: '📖' },
            { word: "PLAY", scrambled: "YAPL", emoji: '⚽' },
            { word: "DANCE", scrambled: "ECNDA", emoji: '💃' },
            { word: "SLEEP", scrambled: "PLESE", emoji: '😴' }
        ];

        let currentVerbIndex = 0;
        let score = 0;
        let lettersState = { source: [], destination: [] };

        // Cache DOM elements for efficient access
        const DOMElements = {
            scoreDisplay: document.getElementById('scoreDisplay'),
            verbEmoji: document.getElementById('verbEmoji'), // Changed from verbImage
            wordSlots: document.getElementById('wordSlots'),
            scrambledLetters: document.getElementById('scrambledLetters'),
            feedbackMessage: document.getElementById('feedbackMessage'),
            checkButton: document.getElementById('checkButton'),
            nextButton: document.getElementById('nextButton'),
            resetButton: document.getElementById('resetButton')
        };

        /**
         * Shuffles an array using the Fisher-Yates (Knuth) algorithm.
         * @param {Array} array - The array to shuffle.
         * @returns {Array} The shuffled array.
         */
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        /**
         * Updates the displayed score on the UI.
         */
        function updateScoreDisplay() {
            DOMElements.scoreDisplay.textContent = score;
        }

        /**
         * Generates a simple success sound using the Web Audio API.
         */
        function playSuccessSound() {
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();

            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);

            oscillator.type = 'sine'; // A smooth wave
            oscillator.frequency.setValueAtTime(660, audioContext.currentTime); // Higher A note for success
            gainNode.gain.setValueAtTime(0.5, audioContext.currentTime); // Volume

            oscillator.start();
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.3); // Fade out
            oscillator.stop(audioContext.currentTime + 0.3); // Stop after 0.3 seconds
        }

        /**
         * Generates a simple error sound using the Web Audio API.
         */
        function playErrorSound() {
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();

            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);

            oscillator.type = 'triangle'; // A harsher wave
            oscillator.frequency.setValueAtTime(180, audioContext.currentTime); // Lower note for error
            gainNode.gain.setValueAtTime(0.5, audioContext.currentTime); // Volume

            oscillator.start();
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.4); // Fade out
            oscillator.stop(audioContext.currentTime + 0.4); // Stop after 0.4 seconds
        }
        
        /**
         * Initializes the game for the current verb.
         * Sets up the scrambled letters and empty word slots.
         */
        function initializeGame() {
            const verbData = verbs[currentVerbIndex];
            // Populate source letters with shuffled scrambled word
            lettersState.source = shuffleArray([...verbData.scrambled]);
            // Create empty slots for the destination word
            lettersState.destination = Array(verbData.word.length).fill(null);
            
            DOMElements.verbEmoji.textContent = verbData.emoji; // Set emoji directly
            DOMElements.verbEmoji.setAttribute('aria-label', `Visual hint for the word: ${verbData.word}`); // Update ARIA label
            
            updateScoreDisplay();
            updateUI(); // Render the letters and slots on the UI
            resetFeedback(); // Clear any previous feedback messages
            
            DOMElements.checkButton.disabled = false; // Re-enable check button for new word
            // Disable the next button until the current word is correctly guessed
            DOMElements.nextButton.disabled = true; 
            DOMElements.nextButton.classList.add('opacity-50', 'cursor-not-allowed');
            DOMElements.nextButton.classList.remove('animate-pulse'); // Remove pulse animation
        }

        /**
         * Updates the UI to reflect the current state of `lettersState`.
         * Renders scrambled letters and word slots.
         */
        function updateUI() {
            // Clear existing letters and re-render scrambled letters
            DOMElements.scrambledLetters.innerHTML = '';
            lettersState.source.forEach((letter, index) => {
                if (letter) { // Only render if there's a letter at this position
                    DOMElements.scrambledLetters.appendChild(createLetterBlock(letter, 'source', index));
                }
            });

            // Clear existing slots and re-render word slots
            DOMElements.wordSlots.innerHTML = '';
            lettersState.destination.forEach((letter, index) => {
                const slot = createLetterBlock(letter, 'destination', index);
                if (letter) { // If a slot is filled, add 'filled' class
                    slot.classList.add('filled');
                }
                DOMElements.wordSlots.appendChild(slot);
            });
        }

        /**
         * Creates a clickable letter block or slot element.
         * @param {string|null} letter - The letter to display, or null for an empty slot.
         * @param {'source'|'destination'} type - The type of element ('source' for scrambled letters, 'destination' for word slots).
         * @param {number} index - The index of the letter/slot in its respective array.
         * @returns {HTMLElement} The created div element.
         */
        function createLetterBlock(letter, type, index) {
            const el = document.createElement('div');
            el.textContent = letter; // Display the letter
            
            if (type === 'destination') {
                el.classList.add('letter-slot'); // Apply slot-specific styling
                if (letter) { // Only filled slots are clickable to move letters back
                    el.addEventListener('click', () => moveLetter(index, 'destination', 'source'));
                }
            } else { // 'source' type
                el.classList.add('letter-block'); // Apply letter block styling
                el.addEventListener('click', () => moveLetter(index, 'source', 'destination'));
            }
            return el;
        }

        /**
         * Moves a letter from one list (source/destination) to another.
         * @param {number} fromIndex - The index of the letter to move.
         * @param {'source'|'destination'} fromList - The list from which to move the letter.
         * @param {'source'|'destination'} toList - The list to which to move the letter.
         */
        function moveLetter(fromIndex, fromList, toList) {
            const letter = lettersState[fromList][fromIndex];
            if (!letter) return; // Do nothing if there's no letter to move

            // Find the first empty slot in the destination list
            const toIndex = lettersState[toList].indexOf(null);
            if (toIndex !== -1) { // If an empty slot is found
                lettersState[toList][toIndex] = letter; // Move the letter
                lettersState[fromList][fromIndex] = null; // Clear the original position
                updateUI(); // Re-render UI after movement
            }
        }

        /**
         * Checks the user's entered word against the correct verb.
         * Provides feedback and updates score.
         */
        function checkAnswer() {
            const enteredWord = lettersState.destination.join('');
            // Check if all slots are filled
            if (enteredWord.length !== verbs[currentVerbIndex].word.length) {
                // Display message directly in feedback area
                resetFeedback();
                const feedback = DOMElements.feedbackMessage;
                feedback.textContent = "Please complete the word!"; // Translated message
                feedback.classList.add('text-yellow-600', 'animate-pop'); // Use a warning color
                feedback.classList.remove('opacity-0');
                return; // Still return to prevent further checks
            }

            if (enteredWord === verbs[currentVerbIndex].word) {
                score++; // Increment score for correct answer
                showFeedback(true); // Show positive feedback
                DOMElements.checkButton.disabled = true; // Disable check button after correct answer
                DOMElements.nextButton.disabled = false; // Enable next button
                DOMElements.nextButton.classList.remove('opacity-50', 'cursor-not-allowed');
                DOMElements.nextButton.classList.add('animate-pulse'); // Add pulse animation to next button
            } else {
                score = Math.max(0, score - 1); // Decrement score, but not below 0
                showFeedback(false); // Show negative feedback
                DOMElements.wordSlots.classList.add('animate-shake'); // Add shake animation to slots
            }
            updateScoreDisplay(); // Update score display
        }

        /**
         * Displays visual and audio feedback to the user.
         * @param {boolean} isCorrect - True if the answer is correct, false otherwise.
         */
        function showFeedback(isCorrect) {
            resetFeedback(); // Clear previous feedback before showing new one
            setTimeout(() => { // Small delay to allow CSS transitions to reset
                const feedback = DOMElements.feedbackMessage;
                if (isCorrect) {
                    feedback.textContent = "Correct!"; // Translated message
                    feedback.classList.add('correct', 'text-green-600'); // Green text for correct
                    playSuccessSound(); // Play success sound using Web Audio API
                } else {
                    feedback.textContent = "Try again!"; // Translated message
                    feedback.classList.add('incorrect', 'text-red-600'); // Red text for incorrect
                    playErrorSound(); // Play error sound using Web Audio API
                }
                feedback.classList.add('animate-pop'); // Apply pop-in animation
                feedback.classList.remove('opacity-0'); // Make feedback visible
            }, 10);
        }

        /**
         * Resets the feedback message and removes animations.
         */
        function resetFeedback() {
            const feedback = DOMElements.feedbackMessage;
            feedback.textContent = '';
            // Reset all feedback-related classes
            feedback.className = 'feedback-message text-xl font-semibold opacity-0';
            DOMElements.wordSlots.classList.remove('animate-shake'); // Remove shake animation
        }

        /**
         * Advances to the next verb in the list or finishes the game if all verbs are completed.
         */
        function nextVerb() {
            currentVerbIndex++;
            if (currentVerbIndex < verbs.length) {
                initializeGame(); // Load the next verb
            } else {
                finishGame(); // All verbs completed
            }
        }

        /**
         * Handles game completion, disabling buttons and showing a final score modal.
         */
        function finishGame() {
            DOMElements.checkButton.disabled = true;
            DOMElements.nextButton.disabled = true;
            DOMElements.nextButton.classList.add('opacity-50', 'cursor-not-allowed');
            DOMElements.nextButton.classList.remove('animate-pulse');
            DOMElements.resetButton.disabled = false; // Allow restarting the game

            // Display final score directly in feedback area
            resetFeedback();
            const feedback = DOMElements.feedbackMessage;
            feedback.textContent = `Game Completed! Your final score is: ${score} points.`; // Translated message
            feedback.classList.add('text-blue-700', 'animate-pop'); // Use a completion color
            feedback.classList.remove('opacity-0');
        }

        // --- Event Listeners ---
        // Check button to verify the answer
        DOMElements.checkButton.addEventListener('click', checkAnswer);
        // Next button to load the next verb
        DOMElements.nextButton.addEventListener('click', nextVerb);
        // Reset button to restart the game
        DOMElements.resetButton.addEventListener('click', () => {
            currentVerbIndex = 0; // Reset to the first verb
            score = 0;           // Reset score
            initializeGame();    // Start a new game from scratch
        });

        // Start the game when the window loads
        window.onload = initializeGame;
    </script>
</body>
</html>
ELABORADO POR: MARÍA PAZ PERALTA
