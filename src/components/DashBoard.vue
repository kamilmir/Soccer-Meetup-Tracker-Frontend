<template>
  <div class="dashboard-container">
    <h2>Available Soccer Matches</h2>

    <!-- Loading state -->
    <div v-if="isLoading" class="loading">Loading matches...</div>

    <!-- Error state -->
    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>

    <!-- No matches available state -->
    <div v-if="matches.length === 0" class="no-matches">
      No matches available at the moment.
    </div>

    <!-- Match List -->
    <div v-else>
      <div class="match-list">
        <div v-for="match in matches" :key="match._id" class="match-card">
          <h3>{{ match.title }}</h3>
          <p>Time: {{ new Date(match.time).toLocaleString() }}</p>
          <p>Location: {{ match.place }}</p>
          <p>Available Spots: {{ match.availableSpots }}</p>
          
          <!-- Join or Leave button -->
          <div v-if="isUserSubscribed(match)">
            <button @click="removeFromMatch(match._id)" class="leave-btn">
              Leave Match
            </button>
          </div>
          <div v-else>
            <button @click="joinMatch(match._id)" class="join-btn">
              Join Match
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      matches: [],
      errorMessage: "",
      isLoading: false,
    };
  },
  methods: {
    // Fetch available matches from the backend API
    async fetchMatches() {
      this.isLoading = true;
      this.errorMessage = ""; // Clear previous error messages

      try {
        const token = localStorage.getItem("jwt_token");
        if (!token) {
          this.$router.push("/login"); // Redirect to login if no token
          return;
        }

        const response = await axios.get(
          "http://localhost:3000/api/v1/match/available",
          {
            headers: { Authorization: `Bearer ${token}` }, // Send token for authentication
          }
        );

        // Calculate available spots for each match
        this.matches = response.data.map((match) => ({
          ...match,
          availableSpots: match.maxPeople - match.listPeopleSubscribed.length,
        }));
      } catch (error) {
        this.errorMessage = error.response
          ? error.response.data.message
          : "An error occurred while fetching the matches.";
      } finally {
        this.isLoading = false;
      }
    },

    // Check if the user is already subscribed to the match
    isUserSubscribed(match) {
      const userId = localStorage.getItem("user_id");
      return match.listPeopleSubscribed.includes(userId);
    },

    // Join a match
    async joinMatch(matchId) {
      const token = localStorage.getItem("jwt_token");
      // const userId = localStorage.getItem("user_id");

      if (!token) {
        this.$router.push("/login");
        return;
      }

      try {
        await axios.post(
          `http://localhost:3000/api/v1/match/register/${matchId}`,
          {  },
          {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          }
        );
        this.fetchMatches(); // Refresh matches after joining
      } catch (error) {
        this.errorMessage = error.response
          ? error.response.data.message
          : "An error occurred while joining the match.";
      }
    },

    // Remove user from match
    async removeFromMatch(matchId) {
      const token = localStorage.getItem("jwt_token");
      const userId = localStorage.getItem("user_id");

      if (!token || !userId) {
        this.$router.push("/login");
        return;
      }

      try {
        await axios.post(
          `http://localhost:3000/api/v1/match/remove/${matchId}`,
          { userId },
          {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          }
        );
        this.fetchMatches(); // Refresh matches after leaving
      } catch (error) {
        this.errorMessage = error.response
          ? error.response.data.message
          : "An error occurred while leaving the match.";
      }
    },
  },
  mounted() {
    this.fetchMatches(); // Fetch matches when the component is mounted
  },
};
</script>

<style scoped>
.dashboard-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

.match-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

.match-card {
  width: 45%;
  margin-bottom: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: #f9f9f9;
}

.match-card h3 {
  margin-bottom: 10px;
}

.match-card p {
  margin: 5px 0;
}

.join-btn {
  padding: 10px 20px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.join-btn:hover {
  background-color: #45a049;
}

.leave-btn {
  padding: 10px 20px;
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.leave-btn:hover {
  background-color: #d32f2f;
}

.error-message {
  color: red;
  font-size: 16px;
}

.loading {
  font-size: 18px;
  margin-top: 20px;
}
</style>
