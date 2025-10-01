<template>
  <div class="center">
    <!-- Inline message generator (no trigger button / modal) -->
    <div class="modal-container">
      <div class="modal-header">
        <h2 class="modal-title">✨ AI Message Generator</h2>
      </div>

      <div class="modal-body">
        <!-- Input Section -->
        <div class="input-section">
          <label for="prompt" class="input-label"
            >What message would you like to generate?</label
          >
          <textarea
            id="prompt"
            v-model="userPrompt"
            placeholder="e.g., I am trying to send a Diwali wish to my customers"
            class="prompt-input"
            rows="3"
            :disabled="isGenerating"
          ></textarea>
        </div>

        <!-- Generate Button -->
        <button
          @click="generateMessage"
          class="generate-btn"
          :disabled="!userPrompt.trim() || isGenerating"
        >
          <span v-if="!isGenerating">🚀 Generate Message</span>
          <span v-else class="loading-spinner">⏳ Generating...</span>
        </button>

        <!-- Generated Message Section -->
        <transition name="fade">
          <div v-if="generatedMessage" class="result-section">
            <label class="result-label">Generated Message (Editable):</label>
            <textarea
              v-model="generatedMessage"
              class="result-textarea"
              rows="6"
              placeholder="Your generated message will appear here..."
            ></textarea>

            <div class="action-buttons">
              <button @click="copyMessage" class="copy-btn">
                {{ copied ? "✓ Copied!" : "📋 Copy Message" }}
              </button>
              <button @click="resetForm" class="reset-btn">
                🔄 Generate New
              </button>
            </div>
          </div>
        </transition>

        <!-- Error Message -->
        <transition name="fade">
          <div v-if="errorMessage" class="error-message">
            ⚠️ {{ errorMessage }}
          </div>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "MessageGenerator",
  data() {
    return {
      userPrompt: "",
      generatedMessage: "",
      isGenerating: false,
      errorMessage: "",
      copied: false,
      apiKey: process.env.VUE_APP_GEMINI_API_KEY, // Replace with your Gemini API key in .env
    };
  },
  methods: {
    async generateMessage() {
      if (!this.userPrompt.trim()) return;

      this.isGenerating = true;
      this.errorMessage = "";
      this.generatedMessage = "";

      try {
        const response = await fetch(
          `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=${this.apiKey}`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              contents: [
                {
                  parts: [
                    {
                      text: `Generate a professional message based on this request: "${this.userPrompt}". 
                  Use placeholders like {name}, {company}, etc. for personalization. 
                  Keep it concise, warm, and professional. Just provide the message without any explanation.`,
                    },
                  ],
                },
              ],
              generationConfig: {
                temperature: 0.7,
                maxOutputTokens: 300,
              },
            }),
          }
        );

        if (!response.ok) {
          throw new Error(`API Error: ${response.status}`);
        }

        const data = await response.json();

        if (data.candidates && data.candidates[0]?.content?.parts?.[0]?.text) {
          this.generatedMessage =
            data.candidates[0].content.parts[0].text.trim();
        } else {
          throw new Error("Invalid response format");
        }
      } catch (error) {
        console.error("Error generating message:", error);
        this.errorMessage =
          "Failed to generate message. Please check your API key and try again.";
      } finally {
        this.isGenerating = false;
      }
    },
    async copyMessage() {
      try {
        await navigator.clipboard.writeText(this.generatedMessage);
        this.copied = true;
        setTimeout(() => {
          this.copied = false;
        }, 2000);
      } catch (error) {
        console.error("Failed to copy:", error);
      }
    },
    resetForm() {
      this.userPrompt = "";
      this.generatedMessage = "";
      this.errorMessage = "";
      this.copied = false;
    },
  },
};
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.center {
  /* keep content clear of left sidebar and centered in remaining space */
  display: block;
  margin: 28px auto;
  width: calc(100% - 300px); /* leaves room for left sidebar */
  max-width: 1100px;
}

/* Modal Overlay */

/* Modal Container */
.modal-container {
  background: #fff;
  border-radius: 14px;
  /* card-like panel with left accent */
  border-left: 6px solid #6b46c1; /* purple accent */
  max-width: 100%;
  width: 100%;
  max-height: 85vh;
  overflow-y: auto;
  box-shadow: 0 18px 40px rgba(16, 24, 40, 0.08);
  padding: 0;
}

.center {
  position: absolute;
  left: 60%;
  top: 50%;
  transform: translate(-50%, -50%);
}

/* Extra-wide layout for very large screens */
@media (min-width: 1400px) {
  .modal-container {
    max-width: 1100px;
    width: min(90vw, 1100px);
  }
}

.modal-header {
  padding: 20px 28px;
  display: flex;
  gap: 12px;
  align-items: center;
  border-bottom: 1px solid #f1f5f9;
}

.modal-title {
  font-size: 22px;
  font-weight: 800;
  color: #3c366b;
}

.modal-body {
  background: linear-gradient(180deg, #ffffff 0%, #fbfbff 100%);
  padding: 24px 28px 32px 28px;
  border-radius: 0 0 14px 14px;
}

/* Input Section */
.input-section {
  margin-bottom: 20px;
}

.input-label {
  display: block;
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin-bottom: 8px;
}

.prompt-input {
  width: 100%;
  padding: 14px 18px;
  border: 1px solid #e6e9ef;
  border-radius: 10px;
  font-size: 15px;
  font-family: inherit;
  transition: border-color 0.18s ease, box-shadow 0.18s ease;
  resize: vertical;
  min-height: 84px;
}

.prompt-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 4px rgba(102, 126, 234, 0.1);
}

.prompt-input:disabled {
  background: #f5f5f5;
  cursor: not-allowed;
}

/* Generate Button */
.generate-btn {
  width: 100%;
  height: 56px;
  background: linear-gradient(90deg, #7c3aed 0%, #6b46c1 60%);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 17px;
  font-weight: 700;
  cursor: pointer;
  transition: transform 0.14s ease, box-shadow 0.14s ease;
  margin: 18px 0 22px 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 10px 30px rgba(107, 70, 193, 0.12);
}

.generate-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 16px 36px rgba(107, 70, 193, 0.18);
}

.generate-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.loading-spinner {
  display: inline-block;
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

/* Result Section */
.result-section {
  margin-top: 18px;
  padding-top: 18px;
  border-top: 1px solid #eef2f6;
}

.result-label {
  display: block;
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin-bottom: 8px;
}

.result-textarea {
  width: 100%;
  padding: 14px 16px;
  border: 1px solid #dbeee0;
  border-radius: 10px;
  font-size: 15px;
  line-height: 1.6;
  font-family: inherit;
  background: #f8fff9;
  resize: vertical;
  margin-bottom: 12px;
  min-height: 160px;
}

.result-textarea:focus {
  outline: none;
  border-color: #45a049;
  box-shadow: 0 0 0 4px rgba(76, 175, 80, 0.1);
}

/* Action Buttons */
.action-buttons {
  display: flex;
  gap: 12px;
}

.copy-btn,
.reset-btn {
  flex: 1;
  padding: 12px 16px;
  border: none;
  border-radius: 10px;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  transition: transform 0.12s, box-shadow 0.12s;
}

.copy-btn {
  background: #16a34a;
  color: white;
  box-shadow: 0 8px 20px rgba(22, 163, 74, 0.12);
}

.copy-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 14px 32px rgba(22, 163, 74, 0.16);
}

.reset-btn {
  background: #f3f4f6;
  color: #111827;
}

.reset-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 24px rgba(15, 23, 42, 0.06);
}

/* Error Message */
.error-message {
  margin-top: 16px;
  padding: 12px 16px;
  background: #ffebee;
  border: 2px solid #ef5350;
  border-radius: 10px;
  color: #c62828;
  font-size: 14px;
  font-weight: 500;
}

/* Animations */
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.3s ease;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}

.modal-slide-enter-active,
.modal-slide-leave-active {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.modal-slide-enter-from {
  transform: translateY(-50px);
  opacity: 0;
}

.modal-slide-leave-to {
  transform: translateY(50px);
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.fade-enter-from {
  opacity: 0;
  transform: translateY(-10px);
}

.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}

/* Scrollbar Styling */
.modal-container::-webkit-scrollbar {
  width: 8px;
}

.modal-container::-webkit-scrollbar-track {
  background: transparent;
}

.modal-container::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.5);
  border-radius: 4px;
}

.modal-container::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.7);
}
</style>
