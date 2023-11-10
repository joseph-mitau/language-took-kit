<!DOCTYPE html>
<html>
  <!DOCTYPE html>
<head>
    <h1>LANGUAGE TOOLKIT</h1>
</head>
<title>Word Meaning Finder</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.2.0/css/all.min.css">

</head>

<body>
<div class="navbar">
    <a href="#" class="nav-link" onclick="showSection('wordFinder')">Meaning Finder</a>
    <a href="#" class="nav-link" onclick="showSection('speechToText')"Speech to Text</a>
    <a href="#" class="nav-link" onclick="showSection('sentenceAnalyzer')">Sentence Analyzer</a>
    <script>
        function showSection(sectionId) {
            const sections = document.querySelectorAll('.container');
            sections.forEach(section => {
                section.classList.remove('active'); // Hide all sections
            });

            const sectionToDisplay = document.getElementById(sectionId);
            sectionToDisplay.classList.add('active'); // Display the active section
        }
    </script>
 <title>Word Meaning Finder</title>
 <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.2.0/css/all.min.css">
 
</head>
<body>
 <div class="container">
     <h1><i class="icon fas fa-book"></i> Word Meaning AI </h1>
     
     <input type="text" id="wordInput" placeholder="Enter a word">
     <button onclick="findWordMeaning()">Find Meaning <i class="icon fas fa-search"></i></button>
     <div id="meanings"></div>
     <div id="pronunciation"></div>
     <div id="suggestions" class="suggestions"></div>
     <button id="pronounceButton" onclick="pronounceWord()"><i class="icon fas fa-volume-up"></i> Pronounce</button>
    

 <script src="https://cdn.jsdelivr.net/npm/didyoumean@1.2.2"></script>
 <script>
     const spellChecker = new DidYouMean({
         threshold: 0.7, // You can adjust the threshold for word suggestions
     });

     async function findWordMeaning() {
         const word = document.getElementById('wordInput').value;
         const meaningsContainer = document.getElementById('meanings');
         const pronunciationContainer = document.getElementById('pronunciation');
         const pronounceButton = document.getElementById('pronounceButton');
         pronounceButton.disabled = true;
         const suggestionsContainer = document.getElementById('suggestions');
         suggestionsContainer.innerHTML = ''; // Clear previous suggestions

         if (word) {
             meaningsContainer.innerHTML = 'Loading...';
             pronunciationContainer.innerHTML = '';

             try {
                 const response = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en_US/${word}`);
                 const data = await response.json();

                 if (response.status === 200 && data.length > 0) {
                     const meanings = data[0].meanings;
                     const pronunciation = data[0].phonetics[0] ? data[0].phonetics[0].text : '';

                     let meaningsHTML = '';

                     meanings.forEach((meaning, index) => {
                         const meaningText = meaning.definitions.map(definition => definition.definition).join('<br>');
                         const exampleText = meaning.definitions.map(definition => definition.example).join('<br>');

                         meaningsHTML += `<div class="meaning"><strong>Meaning ${index + 1}:</strong><br>${meaningText}`;
                         if (exampleText) {
                             meaningsHTML += `<br><strong>Example Sentences:</strong><br><p class="example">${exampleText}</p>`;
                         }
                         meaningsHTML += '</div>';
                     });

                     const pronunciationHTML = pronunciation ? `<strong>Pronunciation:</strong> ${pronunciation}` : '';

                     meaningsContainer.innerHTML = meaningsHTML;
                     pronunciationContainer.innerHTML = pronunciationHTML;

                     if (pronunciation) {
                         pronounceButton.disabled = false;
                     }
                 } else {
                     meaningsContainer.innerHTML = `No meaning found for "${word}".`;

                     // Get spelling suggestions
                     const suggestedWords = spellChecker.get(word);
                     if (suggestedWords.length > 0) {
                         suggestionsContainer.innerHTML = `Did you mean: ${suggestedWords.join(', ')}?`;
                     }
                 }
             } catch (error) {
                 meaningsContainer.innerHTML = 'An error occurred while fetching the meaning.';
             }
         }
     }

     function pronounceWord() {
         const word = document.getElementById('wordInput').value;
         const utterance = new SpeechSynthesisUtterance(word);
         utterance.rate = 0.7;

         window.speechSynthesis.speak(utterance);
     }
 </script>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Speech to Text Converter</title>
    <!-- Add necessary CSS for styling -->
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 20px;
        }

        h1 {
            color: #333;
        }

        button {
            margin: 5px;
            padding: 10px;
            font-size: 14px;
        }

        #output {
            margin-top: 20px;
            padding: 10px;
            border: 1px solid #ccc;
            min-height: 100px;
            background-color: #f8f8f8;
            word-wrap: break-word;
        }

        #loadingIcon {
            display: none;
            margin-top: 10px;
        }

        #editText {
            display: none;
            padding: 10px;
            border: 1px solid #ccc;
            margin-top: 10px;
            width: 80%;
            margin-left: 10%;
            resize: vertical;
            min-height: 100px;
        }
    </style>
</head>
<body>
    <h1>Speech to Text Converter</h1>
    <!-- Add icons to buttons for a more user-friendly interface -->
    <button id="startDictation">Start Dictation 🎤</button>
    <button id="stopDictation" disabled>Stop Dictation 🛑</button>
    <button id="continueDictation" disabled>Continue Dictation ➡️</button>
    <button id="copyText" disabled>Copy to Clipboard 📋</button>
    <button id="editButton" disabled>Edit ✏️</button>
    <div id="output"></div>
    <div id="loadingIcon">...Generating</div>
    <textarea id="editText"></textarea>

    <!-- Include Grammarly script for text editing -->
    <script src="https://cdn.jsdelivr.net/npm/@amoutonbrady/ts-grammarly/dist/grammarly.js"></script>
    
    <script>
        const startButton = document.getElementById('startDictation');
        const stopButton = document.getElementById('stopDictation');
        const continueButton = document.getElementById('continueDictation');
        const copyButton = document.getElementById('copyText');
        const editButton = document.getElementById('editButton');
        const outputDiv = document.getElementById('output');
        const editTextArea = document.getElementById('editText');
        const loadingIcon = document.getElementById('loadingIcon');
        const recognition = new webkitSpeechRecognition() || new SpeechRecognition();
        let interimTranscript = '';
        let finalTranscript = '';
        let previousTranscript = '';

        recognition.continuous = true;
        recognition.interimResults = true;
        recognition.lang = 'en-US';

        startButton.addEventListener('click', () => {
            startButton.disabled = true;
            stopButton.disabled = false;
            continueButton.disabled = true;
            copyButton.disabled = true;
            editButton.disabled = true;
            loadingIcon.style.display = 'block';
            recognition.start();
            outputDiv.textContent = '';
            interimTranscript = '';
            finalTranscript = '';
        });

        stopButton.addEventListener('click', () => {
            startButton.disabled = false;
            stopButton.disabled = true;
            continueButton.disabled = false;
            copyButton.disabled = false;
            editButton.disabled = false;
            recognition.stop();
            loadingIcon.style.display = 'none';
        });

        continueButton.addEventListener('click', () => {
            startButton.disabled = true;
            stopButton.disabled = false;
            continueButton.disabled = true;
            copyButton.disabled = true;
            editButton.disabled = true;
            loadingIcon.style.display = 'block';
            recognition.start();
        });

        copyButton.addEventListener('click', () => {
            const textToCopy = outputDiv.textContent;
            const textArea = document.createElement('textarea');
            textArea.value = textToCopy;
            document.body.appendChild(textArea);
            textArea.select();
            document.execCommand('copy');
            document.body.removeChild(textArea);
        });

        editButton.addEventListener('click', () => {
            editTextArea.value = outputDiv.textContent;
            outputDiv.style.display = 'none';
            editTextArea.style.display = 'block';
        });

        editTextArea.addEventListener('input', () => {
            // You can add real-time editing functionality here if needed
        });

        recognition.onresult = (event) => {
            for (let i = event.resultIndex; i < event.results.length; i++) {
                const transcript = event.results[i][0].transcript;
                if (event.results[i].isFinal) {
                    finalTranscript += transcript + ' ';
                } else {
                    interimTranscript = transcript;
                }
            }

            // Add punctuation for pauses and capitalize sentences
            finalTranscript = finalTranscript.replace(/([.!?])\s+/g, '$1 ');
            finalTranscript = finalTranscript.replace(/([.!?])\s+([a-z])/g, "$1 $2".toUpperCase());

            // Combine with the previous transcript
            const combinedTranscript = previousTranscript + finalTranscript;
            outputDiv.innerHTML = '<p>' + interimTranscript + ' ' + combinedTranscript + '</p>';
        };

        recognition.onend = () => {
            startButton.disabled = false;
            stopButton.disabled = true;
            continueButton.disabled = !finalTranscript.trim().length > 0;
            copyButton.disabled = !finalTranscript.trim().length > 0;
            editButton.disabled = !finalTranscript.trim().length > 0;
            loadingIcon.style.display = 'none';

            // Update the previous transcript
            previousTranscript += finalTranscript;
        };
    </script>
</body>
<body>
    <div class="container">
        <h1>Sentence Analyzer</h1>
        <input type="text" id="sentenceInput" placeholder="Enter a sentence">
        <button onclick="analyzeSentence()">Analyze</button>
        <p id="result"></p>
        <p id="wordCount">Word Count: 0</p>
        <p id="analyzedWords">Analyzed Words: -</p>
        <p id="punctuationCount">Punctuation Marks: 0</p>
        <p id="sentenceCount">Number of Sentences: 0</p>
    </div>

    <script>
        function analyzeSentence() {
            const sentence = document.getElementById('sentenceInput').value.toLowerCase();

            // Lists of coordinating conjunctions and subordinating conjunctions
            const coordinatingConjunctions = ["and", "but", "or", "nor", "for", "so", "yet"];
            const subordinatingConjunctions = ["if", "unless", "whether", "because", "although", "while", "since", "when", "whereas", "in order to"];

            // List of uncertainty words for complex sentences
            const uncertaintyWords = ["if", "whether", "might", "could", "may", "perhaps", "possibly"];

            // Split the sentence into words
            const words = sentence.split(' ');

            // Initialize counts
            let wordCount = 0;
            let punctuationCount = 0;
            let sentenceCount = 0;
            let analyzedWords = [];

            for (const word of words) {
                wordCount++;
                if (word.endsWith('.') || word.endsWith('!') || word.endsWith('?') || word.endsWith(',') || word.endsWith(';') || word.endsWith(':')) {
                    punctuationCount++;
                }
                if (coordinatingConjunctions.includes(word) || subordinatingConjunctions.includes(word) || uncertaintyWords.includes(word)) {
                    analyzedWords.push(word);
                }
                if (word.endsWith('.') || word.endsWith('!') || word.endsWith('?')) {
                    sentenceCount++;
                }
            }

            document.getElementById('wordCount').textContent = `Word Count: ${wordCount}`;
            document.getElementById('punctuationCount').textContent = `Punctuation Marks: ${punctuationCount}`;
            document.getElementById('sentenceCount').textContent = `Number of Sentences: ${sentenceCount}`;
            document.getElementById('analyzedWords').textContent = `Analyzed Words: ${analyzedWords.join(', ')}`;

            if (coordinatingConjunctions.some(word => words.includes(word))) {
                document.getElementById('result').textContent = "Compound Sentence";
            } else if (subordinatingConjunctions.some(word => words.includes(word))) {
                if (uncertaintyWords.some(word => words.includes(word))) {
                    document.getElementById('result').textContent = "Complex Sentence with Uncertainty";
                } else {
                    document.getElementById('result').textContent = "Complex Sentence";
                }
            } else {
                document.getElementById('result').textContent = "Simple Sentence";
            }
        }
        
    
        function showSection(sectionId) {
          const sections = document.querySelectorAll('.section');
          sections.forEach(section => {
            section.style.display = 'none';
          });
    
          const sectionToDisplay = document.getElementById(sectionId);
          sectionToDisplay.style.display = 'block';
        }
    </script>

<style>
  body {
    font-family: 'Arial', sans-serif;
    background-color: #87CEEB; /* Light blue background color */
    margin: 0;
    padding: 0;
  }

  .navbar {
    background-color: #5F9EA0; /* Darker blue for the navbar */
    overflow: hidden;
  }

  .nav-link {
    float: left;
    display: block;
    color: #ecf0f1;
    text-align: center;
    padding: 14px 16px;
    text-decoration: none;
    font-size: 18px;
  }

  .nav-link:hover {
    background-color: #4682b4; /* Slightly darker blue for hover effect */
    color: #ecf0f1;
  }

  .flask {
    background-color: #2905f8; /* Light blue flask color */
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    padding: 20px;
    margin: 20px;
  }

  h1 {
    color: #2c3e50; /* Dark blue heading color */
  }

  input {
    width: 100%;
    padding: 10px;
    margin: 10px 0;
    box-sizing: border-box;
    border: 1px solid rgb(13, 9, 248); /* Slightly darker blue border */
    border-radius: 4px;
  }

  button {
    background-color: #2ecc71; /* Green button color */
    color: #ecf0f1;
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
  }

  button:disabled {
    background-color: #e74c3c; /* Red disabled button color */
    cursor: not-allowed;
  }

  .icon {
    margin-right: 5px;
  }

  #meanings {
    margin-top: 10px;
  }

  .meaning {
    margin-bottom: 15px;
  }

  .example {
    color: #d80808;
  }

  #pronunciation {
    margin-top: 10px;
  }

  #suggestions {
    color: #1900ff;
    font-weight: bold;
    margin-top: 10px;
  }

  #pronounceButton {
    background-color: #e74c3c; /* Red pronounce button color */
  }

  .copyright {
    color: #2c3e50; /* Dark blue copyright color */
    font-size: 12px;
    margin-top: 20px;
  }

</style>
<div class="copyright">Copyright &copy; Group 1 members</div>
    </head>
</html>



