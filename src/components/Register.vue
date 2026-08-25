<script setup lang="ts">
import { ref } from 'vue'
import Loader from './Loader.vue'
import { useUserStore } from '@/stores/poems.ts'
import { RouterLink } from 'vue-router'
import router from '@/router/index.ts';

defineProps<{}>()

const userStore = useUserStore()

const isSubmitted = ref(false)
const isLoading = ref(false)

const userData = ref({
  username: "",
  mail: "",
  password: "",
  retype_password: "",
  terms:false
})

const userErrors = ref({
  username: "",
  mail: "",
  password: "",
  retype_password: "",
  terms:"",
  callError: ""
})

function validateData() {

  // username validation
  if (!userData.value.username) {
    userErrors.value.username = "Username is required"
  }
  else if (userData.value.username.length < 2) {
    userErrors.value.username = "Minimum 3 characters"

  }
  else if (userData.value.username.length > 21) {
    userErrors.value.username = "Maximum 20 characters allowed"
  }
  else {
    userErrors.value.username = ""
  }


  // mail validation

  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!userData.value.mail) {
    userErrors.value.mail = "E-mail address is required"
  }
  else if (!emailRegex.test(userData.value.mail)) {
    userErrors.value.mail = "Please enter a valid e-mail address";
  }
  else {
    userErrors.value.mail = ""
  }


  // password validation

  if (!userData.value.password) {
    userErrors.value.password = "Password is required";
  }
  else if (userData.value.password.length < 8) {
    userErrors.value.password = "Password must be at least 8 characters long";
  }
  else if (!/[a-zA-Z]/.test(userData.value.password) || !/[0-9]/.test(userData.value.password)) {
    userErrors.value.password = "Password must contain at least one letter and one number";
  }
  else {
    userErrors.value.password = "";
  }

  // type password again verification
  if (!userData.value.retype_password) {
    userErrors.value.retype_password = "Please type your password again"
  } else if (userData.value.retype_password !== userData.value.password) {
    userErrors.value.retype_password = "Password don't match!"
  }
  else {
    userErrors.value.retype_password = "";
  }


   // terms validation
  if (!userData.value.terms) {
    userErrors.value.terms = "Accepting the terms and conditions is required"
  }
  else {
    userErrors.value.terms = ""
  }

  const hasErrors = Object.entries(userErrors.value).some(
    ([key, error]) => key !== "callError" && error !== ""
  );

  if (hasErrors) {
    return;
  }


  submitData()

}


async function submitData() {
  isLoading.value = true;
  userErrors.value.callError = "";

  try {
    const response = await fetch("/api/createUser", {
      method: "POST",
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(userData.value),
      credentials: "include" 
    });

    const result = await response.json();

    if (!response.ok) {
      userErrors.value.callError = result.error || "An error has occurred, please try again later!";
      return;
    }

    userStore.setUser(result.data);

    isSubmitted.value = true;

  } catch (err) {
    userErrors.value.callError = "Network error, please try again later.";
  } finally {
    isLoading.value = false;
  }
}

function goHome(){
  window.location.href = "/";
}


</script>

<template>
  <div class="page-wrapper register-wrapper">
    <div class="title-wrapper">
      <p class="title">Sign up</p>
    </div>

    <div v-if="isSubmitted">
      <Loader v-if="isLoading" />
      <div v-else class="submitted">
        <p><strong>Thank you for registering!</strong></p>
        <p>To access your account, please refresh this page!</p>
        <button @click="goHome">Refresh</button>
      </div>
    </div>
    <div v-else class="form-wrapper">
      <p class="note">After signing up, you will be able to share your poems with others and also save poems your
        favourite poems.
        All you need is a your email and a chosen poem. <br>
        Have fun reading!
      </p>


      <form class="container">
        <div class="input-wrapper">
          <label for="username">Username</label>
          <input id="username" v-model="userData.username" placeholder="Your username here">
          <p class="error-msg" v-if="userErrors.username">{{ userErrors.username }}</p>
        </div>

        <div class="input-wrapper">
          <label for="mail">Email</label>
          <input id="mail" v-model="userData.mail" placeholder="Your mail address here">
          <p class="error-msg" v-if="userErrors.mail">{{ userErrors.mail }}</p>
        </div>


        <div class="input-wrapper">
          <label for="password">Password</label>
          <input id="password" v-model="userData.password" placeholder="Choose a password">
          <p class="error-msg" v-if="userErrors.password">{{ userErrors.password }}</p>
        </div>

        <div class="input-wrapper">
          <label for="retype_password">Type password again</label>
          <input id="retype_password" v-model="userData.retype_password" placeholder="Type password again">
          <p class="error-msg" v-if="userErrors.retype_password">{{ userErrors.retype_password }}</p>
        </div>


         <div class="input-wrapper terms-wrapper">
          <div>
          <input id="terms" v-model="userData.terms" type="checkbox" :aria-label="userData.terms ? 'Accepted terms and conditions' : 'Not accepted terms and conditions'">
          <label for="terms">
            By accessing or using this website, you agree to comply with and be bound by these Terms of Service and all applicable laws. All original poems and creative works published on this site remain the intellectual property of their respective authors and are protected by copyright. You agree to use this platform respectfully, without submitting harmful, plagiarized, or inappropriate content. We reserve the right to modify these terms or update website content at any time without prior notice.
          </label>
          </div>
          <p class="error-msg" v-if="userErrors.terms">{{ userErrors.terms }}</p>
        </div>


        <p class="error-msg" v-if="userErrors.callError">{{ userErrors.callError }}</p>
        <button type="submit" @click.prevent="validateData">Register</button>

      </form>

       <div class="login">
        <p>Already have an account?</p>

          <RouterLink to="/login">
            <button>Login
            </button>
          </RouterLink>
      </div>

    </div>
  </div>
</template>

<style scoped>
.login {
  max-width: 500px;
  width: 100%;
  margin: auto;
  padding-top: 2rem;
  padding-bottom: 1rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  text-align: center;
}

.input-wrapper.terms-wrapper > div {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  position: relative;
}

.terms-wrapper input[type="checkbox"] {
  position: absolute;
  opacity: 0;
  width: 20px;
  height: 20px;
  cursor: pointer;
  z-index: 1;
  margin: 0;
}

.terms-wrapper label {
  position: relative;
  padding-left: 28px; 
  font-size: 0.7rem;
  line-height: 1.5;
  color: #333;
  cursor: pointer;
  user-select: none;
}

.terms-wrapper label::before {
  content: "";
  position: absolute;
  left: 0;
  top: 2px;
  width: 18px;
  height: 18px;
  border: 2px solid #666;
  border-radius: 4px;
  background-color: #fff;
  transition: all 0.2s ease-in-out;
}

.terms-wrapper label::after {
  content: "";
  position: absolute;
  left: 6px;
  top: 5px;
  width: 5px;
  height: 10px;
  border: solid #fff;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg) scale(0);
  transition: transform 0.2s ease-in-out;
}

.terms-wrapper:hover input[type="checkbox"] + label::before {
  border-color: var(----defaultDark);
}

.terms-wrapper input[type="checkbox"]:focus-visible + label::before {
  outline: 2px solid var(--defaultDark);
  outline-offset: 2px;
}

.terms-wrapper input[type="checkbox"]:checked + label::before {
  background-color:  var(--defaultDark);
  border-color:  var(--defaultDark);
}

.terms-wrapper input[type="checkbox"]:checked + label::after {
  transform: rotate(45deg) scale(1);
}


</style>