<script setup>
import {computed, reactive, ref, watch} from 'vue';
import {z} from 'zod';
import {nationalities,} from "~/utils/nationalities";
import Popup from "~/components/PopupStudentSubmit.vue";

const userNationalityInput = ref('');
const filteredNationalities = computed(() => {
  if (!userNationalityInput.value) {
    return nationalities;
  }
  return nationalities.filter(n =>
      n.toLowerCase().startsWith(userNationalityInput.value.toLowerCase())
  );
});

const previousQuestions = [
  {
    label: "Name",
    type: "text",
    placeholder: "Enter your name",
    required: true
  },
  {
    label: "Student ID",
    type: "text", placeholder: "Enter your student ID (e.g., AIU21011234)",
    required: true
  },
  {
    label: "Room No",
    type: "text",
    placeholder: "Enter your room No (e.g., 25i-3-10)",
    required: true
  },
  {
    label: "Phone No (Local No Only)",
    type: "text",
    placeholder: "Enter your phone No",
    required: true
  },
  {
    label: "Email Address (Student Email Only)",
    type: "text",
    placeholder: "Enter your email address",
    required: true
  },
  {
    label: "Gender",
    type: "select",
    options: ["Male", "Female"],
    required: true,
    placeholder: "Enter your gender"
  },
  {
    label: "Enter your Nationality",
    type: "select",
    options: filteredNationalities.value,
    placeholder: "Select nationality"
  },
  {
    label: "Other supporting docs",
    type: "file",
    required: true,
    placeholder: "Other supporting docs"
  },
  {
    label: "Explain your reason for room change?",
    type: "textarea",
    required: true,
    placeholder: "Explain in detail the reason for room change?"
  }
];

const formSchema = z.object({
  "Name":
      z.string().min(8, "Name must be at least 8 characters long")
          .nonempty("Name is required"),
  "Student ID":
      z.string()
          .regex(/^AIU\d{8}$/, "Invalid Student ID format")
          .nonempty("Student ID is required"),
  "Room No":
      z.string().regex(/^\d+[A-Za-z]*-\d-\d+$/, "Invalid Room Number format")
          .nonempty("Room Number is required"),
  "Phone No (Local Number Only)":
      z.string().regex(/^\d{8,15}$/, "Invalid phone number")
          .nonempty("Phone Number is required"),
  "Email Address (Student Email Only)":
      z.string()
          .email("Invalid email format")
          .regex(/@student\.aiu\.edu\.my$/, "Must be a student email ending with '@student.aiu.edu.my'"),
  "Gender":
      z.string()
          .nonempty("Gender is required"),
  "Enter your Nationality":
      z.string()
          .optional(),
  "Other supporting docs":
      z.any()
          .optional(),
  "Explain your reason for room change?":
      z.string().min(20, "Name must be at least 20 characters long")
          .nonempty("Name is required"),
});

const form = reactive({});
const errors = reactive({});

previousQuestions.forEach((question) => {
  form[question.label] = "";
  errors[question.label] = "";
});

function validateField(field) {
  try {
    formSchema.shape[field].parse(form[field]);
    errors[field] = "";
  } catch (error) {
    errors[field] = error.errors ? error.errors[0].message : error.message;
  }
}

previousQuestions.forEach((question) => {
  watch(() => form[question.label], (newValue) => validateField(question.label, newValue));
});

function handleSubmit() {
  const validationResults = formSchema.safeParse(form);
  if (validationResults.success) {
    console.log("Form Data:", {...form});
    alert("Form submitted successfully!");
  } else {
    alert("Please correct the errors in the form.");
  }
}

const isPopupVisible = ref(false)

</script>

<template>
  <div class="maintenance-room-section">
    <div class="container">
      <div class="description">
        <img src="/images/AIU-Official-Logo.png" alt="AIU">
        <h2 class="title-maintenance-form">Form for request to change room </h2>
        <p class="description-maintenance-form">If you need to request a room change, please fill out this form with
          your reason for the request. Be sure to provide any necessary documentation, such as a medical report if
          applicable. This will help us assess your request and find a suitable room for you, subject to availability
          and approval.
        </p>
        <p> Note: For medical reasons, attach an official medical report.</p>
      </div>

      <div class="maintenance-room-form">
        <h2>Please fill this Form</h2>
        <form @submit.prevent.once="handleSubmit">
          <div class="maintenance-form">
            <div class="info" v-for="(question, index) in previousQuestions" :key="index">
              <label class="question-title" :for="question.label">{{ question.label }}:</label>

              <input
                  v-if="question.type === 'text' || question.type === 'file'"
                  :type="question.type"
                  v-model="form[question.label]"
                  :placeholder="question.placeholder"
                  :id="question.label"
                  @input="validateField(question.label)"
              />

              <select
                  v-if="question.type === 'select'"
                  v-model="form[question.label]"
                  :id="question.label"
                  @change="validateField(question.label)"
              >
                <option value="" disabled>{{ question.placeholder }}</option>
                <option v-for="option in question.options" :key="option" :value="option">{{ option }}</option>
              </select>
              <span v-if="errors[question.label]" class="error">{{ errors[question.label] }}</span>

              <textarea
                  v-if="question.type === 'textarea'"
                  :id="question.label"
                  :name="question.label"
                  :placeholder="question.placeholder"
                  v-model="form[question.label]"
                  @input="validateField(question.label)"
              />


            </div>
          </div>

          <button @click.once="isPopupVisible = true" class="maintenance-submit" type="submit">Submit</button>
          <popup :show="isPopupVisible" @update:show="isPopupVisible = $event">
          </popup>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>

.maintenance-room-section {
  margin: 7rem;
  border: 2px solid #eeeeee;
  border-radius: 0 30px 30px 0;
  box-shadow: rgba(99, 99, 99, 0.2) 0 2px 8px 0;
}

@media (max-width: 800px) {
  .maintenance-room-section {
    margin: 0.5rem;
  }
}

.container {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
}

.container .description {
  flex: 30%;
  background-color: var(--main-color);
  padding: 2.5rem;
  border-radius: 0;
}

.container .maintenance-room-form {
  flex: 60%;
  padding: 2.5rem;
}

@media (max-width: 800px) {
  .container div {
    display: block;
  }
}

@media (max-width: 1200px) {
  .container {
    display: block;
  }

  .container .description {
    padding: 1rem;
  }
}

.container .description h2 {
  font-size: 1.2rem;
  padding: .5rem 0;
  font-weight: bold;
  color: var(--text-color);
  text-align: center;
}

.container .description p {
  font-size: 1rem;
  padding: 1rem 0;
  font-weight: normal;
  color: black;
  text-align: justify;
}

.maintenance-room-form > h2 {
  font-size: 1.5rem;
  color: var(--main-color);
  text-align: center;
  padding: 1rem 0;
}

.maintenance-form {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  gap: 10px;
}

.info {
  flex-basis: calc(100% - 10px);
  box-sizing: border-box;
  display: block;
}

.maintenance-form .question-title {
  font-size: 1rem;
  color: var(--main-color);
  padding: .5rem 0;
}

.maintenance-form input,
.maintenance-form select {
  width: 100%;
  padding: 0.5rem;
  border: 2px solid #EEEEEE;
  border-radius: 5px;
  outline: none;
}

.maintenance-form textarea {
  width: 100%;
  height: 4rem;
  max-height: 4rem;
  padding: 0.5rem;
  border: 2px solid #EEEEEE;
  border-radius: 5px;
  outline: none;
}

.error {
  color: red;
  font-size: 1rem;
}

@media (max-width: 1200px) {
  .maintenance-form textarea {
    width: calc(100% - .5rem);
  }
}

@media (max-width: 800px) {
  .maintenance-form textarea {
    width: calc(100% - .5rem);
  }
}

.maintenance-submit {
  margin-top: 1rem;
  padding: .5rem;
  display: flex;
  font-size: 1rem;
  border-radius: 1rem;
  background-color: var(--main-color);
  color: var(--text-color);
}

.maintenance-submit:hover {
  background-color: var(--text-hovor-color);
  transition: .2s;
}

</style>
