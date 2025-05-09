<template>
  <div class="home">
    <h2>Welcome, {{ username }}!</h2>

    <!-- Date and Time Card -->
    <div class="date-time-card">
      <p>{{ currentDateTime }}</p>
    </div>

    <div class="tool-cards">
      <!-- Music Recommendation Tool -->
      <div class="tool-card music-card">
        <h3>Music Recommendations</h3>
        <input
          v-model="mood"
          type="text"
          placeholder="Enter mood or search a song..."
          @keyup.enter="getMusicRecommendations"
        />
        <button @click="getMusicRecommendations" class="action-button">
          Get Recommendations
        </button>
        <div v-if="musicRecommendations.length" class="music-list">
          <div
            class="music-track"
            v-for="(track, index) in musicRecommendations"
            :key="index"
          >
            <img :src="track.image_url" alt="Track Image" class="music-image" />
            <div class="track-info">
              <h4>{{ track.name }}</h4>
              <p>{{ track.artist }}</p>
              <a
                :href="track.spotify_url"
                target="_blank"
                class="spotify-button"
                >Listen on Spotify</a
              >
            </div>
          </div>
        </div>
      </div>

      <!-- Pomodoro Timer -->
      <div class="tool-card pomodoro-card">
        <h3>Pomodoro Timer</h3>
        <div class="pomodoro">
          <div class="timer-circle">
            <!-- SVG Progress Circle -->
            <svg
              width="120"
              height="120"
              viewBox="0 0 120 120"
              class="progress-circle"
            >
              <circle cx="60" cy="60" r="54" class="background-circle" />
              <circle
                cx="60"
                cy="60"
                r="54"
                class="progress-bar"
                :style="{ strokeDashoffset: progress }"
              />
            </svg>
            <p>{{ timeLeft }}</p>
          </div>
          <button @click="toggleTimer">
            {{ isTimerRunning ? "Pause" : "Start" }}
          </button>
          <button @click="resetTimer">Reset</button>
        </div>
      </div>

      <!-- To-Do List -->
      <div class="tool-card todo-card">
        <h3>To-Do List</h3>
        <input
          @keyup.enter="addTask"
          v-model="newTask"
          type="text"
          placeholder="Enter task"
        />
        <button @click="addTask" class="action-button">Add Task</button>
        <ul>
          <li v-for="(task, index) in tasks" :key="index" class="task-item">
            <span>{{ task }}</span>
            <button @click="removeTask(index)">Remove</button>
          </li>
        </ul>
      </div>

      <!-- Flashcards -->
      <div class="tool-card flashcard-card">
        <h3>Flashcards</h3>
        <div
          v-for="(flashcard, index) in flashcards"
          :key="index"
          class="flashcard"
          @click="toggleFlashcard(index)"
        >
          <div
            :class="['flashcard-inner', flashcard.revealed ? 'revealed' : '']"
          >
            <div class="flashcard-front">
              <p>{{ flashcard.question }}</p>
            </div>
            <div class="flashcard-back">
              <p>{{ flashcard.answer }}</p>
            </div>
          </div>
        </div>
        <input
          v-model="flashcardQuestion"
          placeholder="Enter question"
          @keyup.enter="addFlashcard"
          required
        />
        <input
          v-model="flashcardAnswer"
          placeholder="Enter answer"
          @keyup.enter="addFlashcard"
          required
        />
        <button @click="addFlashcard" class="action-button">
          Add Flashcard
        </button>
      </div>

      <!-- Full Calculator UI -->
      <div class="tool-card calculator-card">
        <h3>Calculator</h3>
        <div class="calculator">
          <div class="display">
            <p>{{ calcDisplay }}</p>
          </div>
          <div class="buttons">
            <button @click="pressButton('7')">7</button>
            <button @click="pressButton('8')">8</button>
            <button @click="pressButton('9')">9</button>
            <button @click="pressButton('/')">/</button>
            <button @click="pressButton('4')">4</button>
            <button @click="pressButton('5')">5</button>
            <button @click="pressButton('6')">6</button>
            <button @click="pressButton('*')">*</button>
            <button @click="pressButton('1')">1</button>
            <button @click="pressButton('2')">2</button>
            <button @click="pressButton('3')">3</button>
            <button @click="pressButton('-')">-</button>
            <button @click="pressButton('0')">0</button>
            <button @click="pressButton('.')">.</button>
            <button @click="calculateResult">=</button>
            <button @click="pressButton('+')">+</button>
            <button @click="clearDisplay">C</button>
          </div>
        </div>
      </div>

      <!-- Random Study Tip Generator -->
      <div class="tool-card tip-card">
        <h3>Random Study Tip</h3>
        <button @click="generateStudyTip" class="action-button">
          Get Study Tip
        </button>
        <p v-if="studyTip" style="margin-top: 10px; font-size: 18px">
          {{ studyTip }}
        </p>
      </div>

      <!-- Chat with AI Tool -->
      <div class="tool-card chat-ai-card">
        <h3>Chat with AI</h3>
        <div class="card-content">
          <div class="chat-box">
            <div
              v-for="(message, index) in chatMessages"
              :key="index"
              :class="[
                'chat-message',
                message.isUser ? 'user-message' : 'ai-message',
              ]"
            >
              <p v-if="message.isUser">{{ message.text }}</p>
              <p v-else v-html="message.htmlText"></p>
            </div>
          </div>
        </div>
        <input
          @keyup.enter="sendMessage"
          v-model="userMessage"
          placeholder="Type your message..."
        />
        <button @click="sendMessage" class="action-button">Send</button>
      </div>

      <div class="tool-card weather-card">
        <h3>Weather Check</h3>
        <div class="input-suggestions-container">
          <!-- Input Field for City -->
          <input
            v-model="city"
            type="text"
            placeholder="Enter city"
            @input="fetchCitySuggestions"
            @keyup.enter="getWeather"
            required
          />

          <!-- City Suggestions Dropdown -->
          <ul v-if="citySuggestions.length" class="suggestions-list">
            <li
              v-for="(suggestion, index) in citySuggestions"
              :key="index"
              @click="selectCity(suggestion)"
            >
              {{ suggestion.displayName }}
            </li>
          </ul>
        </div>
        <button @click="getWeather" class="action-button">Check Weather</button>

        <div v-if="weather && weather.city" class="weather-info">
          <!-- Render SVG weather icon based on condition -->
          <div class="weather-icon">
            <svg width="50" height="50" viewBox="0 0 24 24">
              <path :d="getWeatherIcon(weather.condition)" />
            </svg>
          </div>
          <div class="weather-details">
            <p class="weather-city">{{ weather.city }}</p>
            <p class="weather-temp">{{ weather.temp }}°C</p>
            <p class="weather-condition">{{ weather.condition }}</p>
          </div>
        </div>
      </div>

      <div class="tool-card notes-card">
        <h3>Quick Notes</h3>
        <textarea
          v-model="noteContent"
          placeholder="Type your note here..."
          @keyup.enter="saveNote"
        ></textarea>
        <button @click="saveNote" class="action-button">Save Note</button>
        <div class="saved-notes" v-if="savedNotes.length">
          <h4>Saved Notes:</h4>
          <div
            class="note-item"
            v-for="(note, index) in savedNotes"
            :key="index"
          >
            <p>{{ note }}</p>
            <button @click="deleteNote(index)" class="delete-note-button">
              Delete
            </button>
          </div>
        </div>
      </div>

      <div class="tool-card motivation-card">
        <h3>Daily Motivation</h3>
        <p style="margin-top: 10px" v-if="motivationalQuote">
          {{ motivationalQuote }}
        </p>
        <button @click="getMotivation" class="action-button">New Quote</button>
      </div>
    </div>

    <!-- Thank you message -->
    <div class="thank-you">
      <p style="text-align: center; margin-top: 2em; font-weight: bold">
        Thank you for using our productivity tools! More features coming soon.
        🚀
      </p>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import api from "../api";
import { mapState } from "vuex";
import { marked } from "marked";
import {
  mdiWeatherSunny,
  mdiWeatherRainy,
  mdiWeatherCloudy,
  mdiWeatherSnowy,
  mdiWeatherLightning,
  mdiWeatherFog,
  mdiWeatherPartlyCloudy,
} from "@mdi/js";

export default {
  data() {
    return {
      username: "",
      currentDateTime: "", // For displaying current date and time
      timerInterval: null, // To manage the interval for updating date and time
      mood: "",
      citySuggestions: [],
      musicRecommendations: [],
      isTimerRunning: false,
      timeLeft: "25:00",
      interval: null,
      minutes: 25,
      seconds: 0,
      totalTime: 25 * 60, // Total time in seconds for 25 minutes
      elapsed: 0, // Time elapsed in seconds
      progress: 339.292, // Full circumference of the circle
      newTask: "",
      tasks: [],
      flashcardQuestion: "",
      flashcardAnswer: "",
      flashcards: [],
      calcDisplay: "",
      studyTip: "",
      userMessage: "",
      chatMessages: [], // Initialize chatMessages as an empty array
      sessionId: "1", // Use a unique session ID if needed
      motivationalQuote: "",
      weather: null,
      savedNotes: [],
    };
  },
  computed: {
    ...mapState(["token"]), // Map the token from Vuex store
    progressCircleOffset() {
      const remainingTime = this.minutes * 60 + this.seconds;
      const progressPercent = remainingTime / this.totalTime;
      return 339.292 * (1 - progressPercent); // 339.292 is the circumference
    },
  },
  created() {
    this.fetchUsername();
    this.updateDateTime();
    this.timerInterval = setInterval(this.updateDateTime, 1000);
    if (!this.token) {
      alert("You need to log in first!");
      this.$router.push("/login");
    }
  },
  beforeRouteEnter(to, from, next) {
    // Use the next callback with a function to ensure the alert is shown before the redirect
    next((vm) => {
      if (!vm.token) {
        alert("You need to log in first!"); // Show alert
        vm.$router.push("/login"); // Then redirect
      } else {
        next(); // Proceed to the route if authenticated
      }
    });
  },
  beforeUnmount() {
    clearInterval(this.timerInterval); // Clear interval on component destruction
  },
  methods: {
    updateDateTime() {
      const now = new Date();

      // Get day of the week, month, day with ordinal suffix, year, and time
      const dayOfWeek = new Intl.DateTimeFormat("en-US", {
        weekday: "long",
      }).format(now);
      const month = new Intl.DateTimeFormat("en-US", { month: "long" }).format(
        now,
      );
      const day = now.getDate();
      const year = now.getFullYear();

      // Helper function to add ordinal suffix to day
      const getOrdinalSuffix = (day) => {
        if (day > 3 && day < 21) return `${day}th`; // Special case for teens
        switch (day % 10) {
          case 1:
            return `${day}st`;
          case 2:
            return `${day}nd`;
          case 3:
            return `${day}rd`;
          default:
            return `${day}th`;
        }
      };

      // Format time as hh:mm:ss AM/PM
      const hours = now.getHours();
      const minutes = now.getMinutes().toString().padStart(2, "0");
      const seconds = now.getSeconds().toString().padStart(2, "0");
      const amPm = hours >= 12 ? "PM" : "AM";
      const formattedTime = `${hours % 12 || 12}:${minutes}:${seconds} ${amPm}`;

      // Format the full date and time string
      this.currentDateTime = `${dayOfWeek}, ${month} ${getOrdinalSuffix(day)}, ${year} - ${formattedTime}`;
    },
    async fetchCitySuggestions() {
      if (this.city.length < 2) {
        this.citySuggestions = [];
        return;
      }
      try {
        const response = await axios.get(
          `https://studysync-study-buddy-app.onrender.com/api/cities?query=${this.city}`,
        );
        this.citySuggestions = response.data.cities.map((city) => ({
          displayName: `${city.name}, ${city.state ? city.state + ", " : ""}${city.country}`,
          fullCity: city,
        }));
      } catch (error) {
        console.error("Error fetching city suggestions:", error);
      }
    },
    selectCity(suggestion) {
      this.city = suggestion.fullCity.name; // Set the input value to just the city name
      this.citySuggestions = []; // Clear suggestions after selection
      this.getWeather(); // Fetch the weather data based on the selected city
    },
    async fetchUsername() {
      try {
        // Get the token from localStorage or Vuex store (whichever applies in your setup)
        const token = this.token || localStorage.getItem("token");
        if (!token) {
          console.error("Authorization token not found.");
          this.username = "Guest";
          return;
        }

        // Make an API request to get the user's profile
        const response = await axios.get(
          "https://studysync-study-buddy-app.onrender.com/api/profile",
          {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          },
        );

        // Check if response is successful and contains the user object
        if (response.data.success && response.data.user) {
          this.username = response.data.user.name || "Guest";
        } else {
          console.error("Failed to retrieve user profile");
          this.username = "Guest";
        }
      } catch (error) {
        console.error("Error fetching user profile:", error.message);
        this.username = "Guest";
      }
    },
    async sendMessage() {
      if (this.userMessage.trim()) {
        // Add the user's message to the chat history
        this.chatMessages.push({ text: this.userMessage, isUser: true });

        try {
          // Make the API call to get the AI's response
          const response = await api.post(
            "/ai-chat",
            { sessionId: this.sessionId, message: this.userMessage },
            { headers: { Authorization: `Bearer ${this.token}` } },
          );

          // If the API response is successful, parse the response as markdown
          if (response.data.success && response.data.response) {
            const markdownText = response.data.response;
            const htmlText = marked(markdownText); // Parse markdown to HTML

            this.chatMessages.push({
              text: markdownText,
              htmlText,
              isUser: false,
            }); // Store both raw and HTML formats
          } else {
            this.chatMessages.push({
              text: "I'm sorry, I couldn't get a response from the AI.",
              isUser: false,
            });
          }
        } catch (error) {
          console.error("Error fetching AI response:", error.message);
          this.chatMessages.push({
            text: "Error: Could not connect to the AI service.",
            isUser: false,
          });
        }

        this.userMessage = ""; // Clear input after sending
      }
    },
    toggleTimer() {
      if (this.isTimerRunning) {
        clearInterval(this.interval);
        this.isTimerRunning = false;
      } else {
        this.startTimer();
      }
    },
    startTimer() {
      this.isTimerRunning = true;
      this.interval = setInterval(() => {
        if (this.seconds === 0 && this.minutes === 0) {
          clearInterval(this.interval);
          this.isTimerRunning = false;
          alert("Pomodoro session is complete!");
        } else {
          if (this.seconds === 0) {
            this.seconds = 59;
            this.minutes--;
          } else {
            this.seconds--;
          }

          // Update the timeLeft and progress dynamically
          this.timeLeft = `${this.formatTime(this.minutes)}:${this.formatTime(this.seconds)}`;
          this.progress = this.progressCircleOffset; // Update the progress value
        }
      }, 1000);
    },
    formatTime(time) {
      return time < 10 ? `0${time}` : time;
    },
    resetTimer() {
      clearInterval(this.interval);
      this.isTimerRunning = false;
      this.minutes = 25;
      this.seconds = 0;
      this.timeLeft = "25:00";
      this.elapsed = 0;
      this.progress = 339.292; // Reset progress to full circumference
    },
    addTask() {
      if (this.newTask.trim()) {
        this.tasks.push(this.newTask);
        this.newTask = "";
      }
    },
    removeTask(index) {
      this.tasks.splice(index, 1);
    },
    addFlashcard() {
      if (this.flashcardQuestion.trim() && this.flashcardAnswer.trim()) {
        this.flashcards.push({
          question: this.flashcardQuestion,
          answer: this.flashcardAnswer,
          revealed: false,
        });
        this.flashcardQuestion = "";
        this.flashcardAnswer = "";
      }
    },
    toggleFlashcard(index) {
      this.flashcards[index].revealed = !this.flashcards[index].revealed;
    },
    async getMusicRecommendations() {
      try {
        const token = localStorage.getItem("token");
        if (!token) {
          console.error("No token found");
          return;
        }
        const response = await axios.get(
          `https://studysync-study-buddy-app.onrender.com/api/music?searchTerm=${this.mood}`,
          {
            headers: { Authorization: `Bearer ${token}` },
          },
        );
        if (response.data.success) {
          this.musicRecommendations = response.data.recommendations;
        } else {
          console.error("Failed to fetch recommendations");
        }
      } catch (error) {
        console.error("Error fetching music recommendations:", error);
      }
    },
    getWeatherIcon(condition) {
      // Map weather condition to MDI icons
      const iconMapping = {
        Clear: mdiWeatherSunny,
        Rain: mdiWeatherRainy,
        Clouds: mdiWeatherCloudy,
        Snow: mdiWeatherSnowy,
        Thunderstorm: mdiWeatherLightning,
        Drizzle: mdiWeatherPartlyCloudy,
        Mist: mdiWeatherFog,
        Fog: mdiWeatherFog,
      };
      return iconMapping[condition] || mdiWeatherPartlyCloudy;
    },
    async getWeather() {
      try {
        if (!this.city) {
          alert("Please enter a city name");
          return;
        }

        // Call the backend API for weather data
        const response = await axios.get(
          `https://studysync-study-buddy-app.onrender.com/api/weather?city=${this.city}`,
        );

        // Check if the response was successful and contains data
        if (response.data.success && response.data.data) {
          const { city, temp, condition } = response.data.data;
          this.weather = {
            city,
            temp: Math.round(temp), // Round temperature to nearest integer
            condition,
          };
        } else {
          alert("Weather data could not be retrieved. Please try again.");
        }
      } catch (error) {
        console.error("Error fetching weather data:", error);
        alert(
          "Failed to retrieve weather data. Please check your city name and try again.",
        );
      }
    },
    saveNote() {
      if (this.noteContent.trim()) {
        this.savedNotes.push(this.noteContent); // Append the new note
        this.noteContent = ""; // Clear the input
      }
    },
    deleteNote(index) {
      this.savedNotes.splice(index, 1); // Remove note at the specified index
    },
    getMotivation() {
      const quotes = [
        "Believe you can and you're halfway there.",
        "Success is not final, failure is not fatal: It is the courage to continue that counts.",
        "Hardships often prepare ordinary people for an extraordinary destiny.",
        "Don't watch the clock; do what it does. Keep going.",
      ];
      this.motivationalQuote =
        quotes[Math.floor(Math.random() * quotes.length)];
    },
    generateStudyTip() {
      const tips = [
        "Take regular breaks to stay fresh.",
        "Stay hydrated and eat healthy snacks.",
        "Practice active recall for better retention.",
        "Set realistic study goals and stick to them.",
        "Teach what you've learned to others.",
      ];
      this.studyTip = tips[Math.floor(Math.random() * tips.length)];
    },
    pressButton(value) {
      this.calcDisplay += value;
    },
    calculateResult() {
      try {
        this.calcDisplay = eval(this.calcDisplay).toString();
      } catch (e) {
        this.calcDisplay = "Error";
      }
    },
    clearDisplay() {
      this.calcDisplay = "";
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600&display=swap");

.home {
  padding: 2em;
  font-family: "Poppins", sans-serif;
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 1.5em;
}

.tool-cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 2em;
}

.tool-card {
  background: #fff;
  border-radius: 15px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
  padding: 1.5em;
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease;
  display: flex;
  flex-direction: column;
  max-height: 1000px; /* Sets max height for the card */
  overflow: hidden;
  min-height: 450px;
}

.tool-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
}

.card-content {
  flex: 1;
  overflow-y: auto; /* Enables vertical scrolling within content area */
  padding: 0.5em 0;
}

.action-button {
  background-color: #3949ab;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  width: 100%;
  margin-top: 10px;
}

.action-button:hover {
  background-color: #5f5fc1;
}

/* Music Card */
.music-card {
  background: linear-gradient(
    135deg,
    #ff7e5f,
    #feb47b
  ); /* Gradient background */
  color: #fff;
  border-radius: 15px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.2); /* Soft shadow for 3D effect */
  padding: 1.5em;
  display: flex;
  flex-direction: column;
  align-items: center;
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease;
  min-height: 400px; /* Minimum height to ensure space for content */
  overflow: hidden; /* Prevent content overflow outside the card */
}

.music-card h3 {
  margin-bottom: 1em;
}

.music-card input {
  width: 100%;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #fff;
  color: #333;
  background: rgba(255, 255, 255, 0.6);
  font-size: 1.1em;
  transition: background-color 0.3s ease;
}

.music-card input:focus {
  background-color: #ffffff; /* Lighter background on focus */
}

.music-list {
  width: 100%;
  margin-top: 2em;
  max-height: 400px; /* Set a max height to allow for scrolling */
  overflow-y: auto; /* Enable scrolling if content exceeds max-height */
  animation: slideIn 1s ease-out; /* Smooth slide-in effect */
}

.music-track {
  display: flex;
  align-items: center;
  margin-bottom: 1em;
  background: rgba(255, 255, 255, 0.1);
  padding: 1em;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}

.music-track:hover {
  transform: translateY(-5px); /* Slight elevation on hover */
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.25); /* Darker shadow on hover */
}

.music-image {
  width: 80px;
  height: 80px;
  border-radius: 8px;
  margin-right: 1em;
  transition: transform 0.3s ease; /* Smooth animation on image hover */
}

.music-image:hover {
  transform: scale(1.1); /* Slight zoom on hover */
}

.track-info h4 {
  font-size: 1.1em;
  margin: 0.5em 0;
  animation: fadeIn 0.8s ease-out; /* Fade-in animation for track title */
}

.track-info p {
  color: #aaa;
  font-size: 0.9em;
}

.spotify-button {
  display: inline-block;
  background: #1db954;
  color: #fff;
  padding: 8px 12px;
  border-radius: 20px;
  text-decoration: none;
  font-size: 0.9em;
  margin-top: 1em;
  transition:
    background 0.3s ease,
    transform 0.3s ease;
}

.spotify-button:hover {
  background: #1ed760; /* Darker green on hover */
  transform: scale(1.05); /* Slight zoom effect */
}

/* Pomodoro Timer */
.pomodoro-card {
  background: #ffeb3b;
  color: #333;
  text-align: center;
}

.timer-circle {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  background: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 1em auto;
  font-size: 1.5em;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.pomodoro button {
  background: #f44336;
  color: #fff;
  padding: 10px 20px;
  border-radius: 5px;
  margin: 5px;
}

.timer-circle {
  position: relative;
  width: 120px;
  height: 120px;
  margin: 1em auto;
}

.timer-circle svg {
  position: absolute;
  top: 0;
  left: 0;
  transform: rotate(-90deg);
}

.background-circle {
  fill: none;
  stroke: #ddd;
  stroke-width: 12;
}

.progress-bar {
  fill: none;
  stroke: #f44336;
  stroke-width: 12;
  stroke-linecap: round;
  stroke-dasharray: 339.292; /* Circumference of the circle (2 * π * 54) */
  transition: stroke-dashoffset 0.3s ease;
}

.timer-circle p {
  position: relative;
  z-index: 1;
  font-size: 1.5em;
  color: #333;
  line-height: 120px;
}

/* To-Do List */
.todo-card {
  background: #2ecc71;
  color: #fff;
  text-align: center;
}

.todo-card input {
  width: 100%;
  padding: 10px;
  margin-top: 1em;
  border: 0.5px solid #333;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  background: rgba(255, 255, 255, 0.6);
}

.task-item {
  display: flex;
  justify-content: space-between;
  background: rgba(255, 255, 255, 0.1);
  padding: 0.5em;
  border-radius: 5px;
  margin-top: 0.5em;
}

.task-item button {
  background: #e74c3c;
  color: #fff;
  border-radius: 3px;
  padding: 3px 8px;
}

/* Flashcards */
.flashcard-card {
  background: #f1c40f;
  color: #333;
  text-align: center;
}

.flashcard-card input {
  width: 100%;
  padding: 10px;
  margin-top: 1em;
  border: 0.5px solid #333;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  background: rgba(255, 255, 255, 0.6);
}

.flashcard {
  perspective: 1000px;
  cursor: pointer;
  margin-top: 1em;
}

.flashcard-inner {
  position: relative;
  width: 100%;
  height: 120px;
  text-align: center;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.flashcard-inner.revealed {
  transform: rotateY(180deg);
}

.flashcard-front,
.flashcard-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  font-size: 1.1em;
}

.flashcard-front {
  background-color: #3498db;
  color: white;
}

.flashcard-back {
  background-color: #27ae60;
  color: white;
  transform: rotateY(180deg);
}

/* Calculator */
.calculator-card {
  background: linear-gradient(135deg, #34495e, #2c3e50);
  color: #fff;
  text-align: center;
}

.calculator .display {
  background: rgba(255, 255, 255, 0.2);
  padding: 1em;
  font-size: 1.5em;
  text-align: right;
  border-radius: 8px;
  margin-bottom: 1em;
  margin-top: 0.5em;
}

.calculator .buttons {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.calculator .buttons button {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  font-size: 1.2em;
  padding: 15px;
  border-radius: 5px;
  transition: background 0.3s ease;
}

.calculator .buttons button:hover {
  background: rgba(255, 255, 255, 0.3);
}

/* Study Tip Card */
.tip-card {
  background: #ff6f61;
  color: #fff;
  text-align: center;
}

.tip-card button {
  background: #e74c3c;
  color: #fff;
  padding: 10px 20px;
  border-radius: 5px;
  margin-top: 1em;
}

/* Chat AI Card */
.chat-ai-card {
  background: linear-gradient(135deg, #8e44ad, #c39bd3);
  color: #fff;
  padding: 1em;
  text-align: center;
}

.chat-box {
  background: rgba(255, 255, 255, 0.1);
  min-height: 300px; /* Set a minimum height */
  max-height: 600px; /* Limit the max height */
  padding: 10px;
  overflow-y: auto;
  overflow-x: hidden; /* Prevent horizontal scrolling */
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  word-wrap: break-word;
}

.chat-ai-card input {
  width: 100%;
  padding: 10px;
  margin-top: 1em;
  border: 0.5px solid #333;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.chat-message {
  max-width: 100%; /* Ensure message does not exceed box width */
  overflow-wrap: break-word;
  word-break: break-word; /* Break long words to prevent overflow */
  white-space: pre-wrap; /* Preserve whitespace and wrap lines */
  padding: 8px 12px;
  border-radius: 8px;
}

.chat-message p {
  margin: 0;
  white-space: pre-wrap; /* Preserve whitespace and wrap lines */
}

.user-message {
  align-self: flex-end;
  background-color: #34495e;
  max-width: 100%; /* Limit user message width */
}

.ai-message {
  align-self: flex-start;
  background-color: #27ae60;
  color: #fff;
  max-width: 100%; /* Limit AI message width */
}

.chat-message,
.task-item,
.music-track,
.flashcard,
.saved-note,
.motivational-quote {
  overflow-wrap: break-word;
  word-break: break-word;
  padding: 8px;
  border-radius: 8px;
}

.chat-message p,
.task-item span,
.flashcard-front p,
.flashcard-back p,
.saved-note p {
  white-space: normal;
}

.weather-card {
  background: linear-gradient(135deg, #1e3a8a, #3b82f6);
  color: #fff;
  padding: 1.5em;
  text-align: center;
  border-radius: 15px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;
}

.weather-card input {
  width: 100%;
  padding: 10px;
  margin-top: 1em;
  border: 0.5px solid #333;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  color: white;
}

.weather-card:hover {
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

.weather-info {
  margin-top: 1.5em;
  display: flex;
  align-items: center;
  justify-content: space-around;
  gap: 15px;
  animation: fadeIn 0.5s ease-in-out;
}

.weather-icon svg {
  fill: #fbbf24;
  animation: pulse 1.5s infinite;
}

.weather-details {
  text-align: left;
}

.weather-city {
  font-size: 1.2em;
  font-weight: 600;
}

.weather-temp {
  font-size: 2em;
  font-weight: bold;
  margin: 0.2em 0;
}

.weather-condition {
  font-size: 1em;
  font-style: italic;
  color: #a0c4ff;
}

.weather-card {
  background: linear-gradient(135deg, #1e3a8a, #3b82f6);
  color: #fff;
  padding: 1.5em;
  text-align: center;
  border-radius: 15px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
  position: relative; /* For positioning suggestions */
}

.weather-card input {
  width: 100%;
  padding: 10px;
  margin-top: 1em;
  border-radius: 5px;
  border: 0.5px solid #333;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  color: white;
}

.input-suggestions-container {
  position: relative;
}

.suggestions-list {
  background: rgba(255, 255, 255, 0.9);
  color: #333;
  position: absolute;
  width: 100%;
  top: 100%; /* Positions the list directly below the input */
  left: 0;
  list-style-type: none;
  padding: 0;
  margin: 0;
  border-radius: 5px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
  max-height: 150px;
  overflow-y: auto;
  z-index: 10; /* Ensures dropdown appears above other elements */
}

.suggestions-list li {
  padding: 10px;
  cursor: pointer;
}

.suggestions-list li:hover {
  background-color: #3949ab;
  color: #fff;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse {
  0%,
  100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
}

.notes-card {
  background: linear-gradient(135deg, #f39c12, #f1c40f);
  color: #fff;
  padding: 1em;
  text-align: center;
}

.notes-card textarea {
  width: 100%;
  height: 80px;
  padding: 10px;
  border-radius: 5px;
  margin-top: 10px;
  resize: none;
  border: 0.5px solid #333;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.saved-notes {
  margin-top: 15px;
  background: rgba(255, 255, 255, 0.1);
  padding: 10px;
  border-radius: 5px;
}

.note-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 5px;
  margin-bottom: 8px;
}

.delete-note-button {
  background: #e74c3c;
  color: #fff;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
}

.delete-note-button:hover {
  background: #c0392b;
}

.notes-card textarea {
  width: 100%;
  height: 80px;
  padding: 10px;
  border-radius: 5px;
  margin-top: 10px;
  resize: none;
  border: 0.5px solid #333;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.saved-note {
  margin-top: 15px;
  background: rgba(255, 255, 255, 0.2);
  padding: 10px;
  border-radius: 5px;
}

/* New Card: Daily Motivation */
.motivation-card {
  background: linear-gradient(135deg, #8e44ad, #9b59b6);
  color: #fff;
  text-align: center;
}

.motivation-card p {
  font-size: 1.1em;
  margin-bottom: 15px;
}

.date-time-card {
  background: rgba(57, 73, 171, 0.8); /* Semi-transparent royal blue */
  color: #ffffff;
  text-align: center;
  padding: 1.2em;
  border-radius: 15px;
  margin: 0 auto 2em; /* Center horizontally and add bottom margin */
  max-width: 320px; /* Slightly increase width */
  font-size: 1.3em;
  font-weight: 500;
  letter-spacing: 0.5px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2); /* Deeper shadow */
  backdrop-filter: blur(5px); /* Glass effect */
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition:
    transform 0.4s cubic-bezier(0.4, 0, 0.2, 1),
    box-shadow 0.4s cubic-bezier(0.4, 0, 0.2, 1); /* Smooth animation */
}

.date-time-card:hover {
  transform: translateY(-8px) scale(1.02); /* Slight scaling for emphasis */
  box-shadow: 0 12px 36px rgba(0, 0, 0, 0.3); /* Smoother shadow transition */
}

.thank-you {
  text-align: center;
  margin-top: 2em;
  border-top: 1px solid #ddd;
}
</style>
