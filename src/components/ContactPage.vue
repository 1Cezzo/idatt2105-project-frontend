<template>
  <div class="contact-page">
    <ComponentCard class="contact-card">
      <h2 id="header">Contact Us</h2>
      <form @submit.prevent="submitForm">
        <div class="form-group">
          <label for="firstName">First Name:</label>
          <input type="text" id="firstName" v-model="formData.firstName" required />
        </div>
        <div class="form-group">
          <label for="lastName">Last Name:</label>
          <input type="text" id="lastName" v-model="formData.lastName" required />
        </div>
        <div class="form-group">
          <label for="email">Email:</label>
          <input type="email" id="email" v-model="formData.email" required />
        </div>
        <div class="form-group">
          <label for="feedbackType">Feedback Type:</label>
          <select id="feedbackType" v-model="formData.feedbackType" required>
            <option value="ASSISTANCE">Assistance</option>
            <option value="FEEDBACK">Feedback</option>
            <option value="REPORT_ISSUE">Report Issue</option>
          </select>
        </div>
        <div class="form-group">
          <label for="content">Content:</label>
          <textarea id="content" v-model="formData.content" rows="4" required></textarea>
        </div>
        <button type="submit">Submit</button>
      </form>
      <ComponentPopup
        v-if="feedbackError"
        :errorMessage="feedbackError"
        :fontColor="popupFontColor"
        @popup-closed="resetError"
      />
    </ComponentCard>
  </div>
</template>

<script>
import ComponentCard from '@/components/ComponentCard.vue'
import ComponentPopup from '@/components/ComponentPopup.vue'
import { createFeedback } from '@/api/feedbackHooks'
import { getUserByUsername } from '@/api/userHooks'
import { useUserStore } from '@/stores/userStore'

export default {
  name: 'ContactPage',
  components: {
    ComponentCard,
    ComponentPopup
  },
  data() {
    return {
      formData: {
        firstName: '',
        lastName: '',
        email: '',
        feedbackType: 'ASSISTANCE',
        content: ''
      },
      feedbackError: '',
      popupFontColor: 'red'
    }
  },
  methods: {
    async submitForm() {
      try {
        const userStore = useUserStore()
        const username = userStore.getUserName

        if (!userStore.getIsLoggedIn) {
          this.feedbackError = 'Please log in to submit feedback.'
          return
        }

        const userData = await getUserByUsername(username)
        if (userData && userData.id) {
          this.formData.userId = userData.id
          const success = await createFeedback(this.formData)
          if (success) {
            this.resetForm()
            this.feedbackError = 'Feedback submitted successfully.'
          } else {
            console.error('Failed to submit feedback.')
          }
        } else {
          console.error('Failed to fetch user data or user ID is missing.')
        }
      } catch (error) {
        console.error('Error submitting feedback:', error)
      }
    },
    resetForm() {
      this.formData = {
        firstName: '',
        lastName: '',
        email: '',
        feedbackType: 'ASSISTANCE',
        content: ''
      }
      this.popupFontColor = 'green'
    },
    resetError() {
      this.feedbackError = ''
      this.popupFontColor = 'red'
    }
  }
}
</script>

<style scoped>
#header {
  font-size: 2rem;
  margin-bottom: 20px;
}

.contact-card {
  margin-top: 10%;
}

.contact-page {
  max-width: 600px;
  margin: 0 auto;
}

.form-group {
  margin-bottom: 20px;
}

/* Media queries for adjusting padding based on screen width */
@media screen and (min-width: 768px) {
  .contact-card {
    padding: 30px;
  }
}

@media screen and (min-width: 1024px) {
  .contact-card {
    padding: 30px 100px 40px 100px;
  }
}

label {
  font-weight: bold;
}

input[type='text'],
input[type='email'],
select,
textarea {
  width: 100%;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
  box-sizing: border-box;
  font-family: Arial, Helvetica, sans-serif;
}

textarea {
  resize: vertical;
  max-height: 300px;
  overflow-y: auto;
}

button {
  background-color: #4caf50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}
</style>
