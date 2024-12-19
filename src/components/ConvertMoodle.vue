<template>
    <div>
        <button @click="convert">Fragen laden</button>
        <br>
        <br>
        <button @click="postDatabase(1)">Datenbank speichern</button>
        <div v-for="(question, index) in parsedQuestions" :key="index">
            <p>{{ question.questionText }}</p>
            <ul>
                <li v-for="(option, idx) in question.options" :key="idx">{{ option.optionText }} - {{ option.optionCorrect ? 'Richtig' : 'Falsch' }}</li>
            </ul>
        </div>
    </div>
</template>

<script>
export default {
    name: 'ConvertMoodle',
    props: {
        msg: String
    },
    data() {
        return {
            parsedQuestions: []
        };
    },
    methods: {
        async convert() {
        try {
            const response = await fetch('/questions.xml');
            if (!response.ok) {
                throw new Error(`Fehler beim Laden der XML-Datei: ${response.statusText}`);
            }

            const xmlText = await response.text();

            const parser = new DOMParser();
            const xmlDoc = parser.parseFromString(xmlText, "application/xml");

            const questions = xmlDoc.getElementsByTagName("question");
            const parsedQuestions = Array.from(questions).filter((questionElement) => {
                // Ignoriere Elemente, die einen Kommentar mit "question: 0" enthalten
                const previousSibling = questionElement.previousSibling;
                return !(previousSibling && previousSibling.nodeType === Node.COMMENT_NODE && previousSibling.nodeValue.trim() === "question: 0");
            }).map((questionElement) => {
                const questionTextElement = questionElement.getElementsByTagName("questiontext")[0];
                const questionText = questionTextElement
                    ? this.removeHtmlTags(questionTextElement.getElementsByTagName("text")[0].textContent)
                    : 'Frage nicht verfügbar';

                const options = Array.from(questionElement.getElementsByTagName("answer")).map((answerElement) => {
                    const optionTextElement = answerElement.getElementsByTagName("text")[0];
                    const optionText = optionTextElement
                        ? this.removeHtmlTags(optionTextElement.textContent)
                        : 'Antwort nicht verfügbar';

                    const fraction = answerElement.getAttribute("fraction") || '0';
                    const optionCorrect = fraction != 0;

                    return new Option(optionText, optionCorrect);
                });

                const question = new Question(questionText, options);

                return question;
            });

            this.parsedQuestions = parsedQuestions;

            console.log(parsedQuestions);

        } catch (error) {
            console.error('Fehler beim Verarbeiten der XML-Datei:', error.message);
        }
    },

        removeHtmlTags(text) {
            return text.replace(/<[^>]*>/g, '').trim();
        },
        async postDatabase(index) {

            const requestData = this.parsedQuestions[index]
            requestData.focusId = 3
            requestData.mChoice = true
            requestData.textInput = false
            requestData.imageAddress = ""

            console.log(JSON.stringify(requestData))


            const response = await fetch('https://projekte.tgm.ac.at/quizit/api/question', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'authorization': '2e5c9ed5-c5f5-458a-a1cb-40b6235b052a'
                },
                body: JSON.stringify(requestData)
            });

            if (!response.ok) {
                throw new Error(`Fehler beim Senden der Daten: ${response.statusText}`);
            }

            const result = await response.json(); // Antwort vom Server
            console.log('Daten erfolgreich gesendet:', result);
        }

    }
}

// Option Klasse
class Option {
    constructor(optionText, optionCorrect) {
        this.optionText = optionText;       
        this.optionCorrect = optionCorrect;
    }

    toString() {
        return `Option: ${this.optionText}, Correct: ${this.optionCorrect}`;
    }
}

class Question {
    constructor(questionText, options) {
        this.questionText = questionText;  
        this.options = options;           
    }

    toString() {
        return `Frage: ${this.questionText}, Optionen: ${this.options.map(option => option.toString()).join(', ')}`;
    }
}
</script>
