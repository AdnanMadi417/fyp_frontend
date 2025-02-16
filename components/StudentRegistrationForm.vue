<script setup>
import {computed, reactive, ref, watch} from 'vue';
import Popup from '~/components/PopupAdminSubmit.vue'
import {z} from 'zod';
import { religions, femaleBlockOptions, maleBlockOptions, nationalities,blockData} from "~/utils/nationalities";

const form = reactive({});
const errors = reactive({});

const userNationalityInput = ref('');
const filteredNationalities = computed(() => {
  if (!userNationalityInput.value) {
    return nationalities;
  }
  return nationalities.filter(n =>
      n.toLowerCase().startsWith(userNationalityInput.value.toLowerCase())
  );
});

const userReligionsInput = ref('');
const filteredReligions = computed(() => {
  if (!userReligionsInput.value) {
    return religions;
  }
  return religions.filter(n =>
      n.toLowerCase().startsWith(userReligionsInput.value.toLowerCase())
  );
});

const isPopupVisible = ref(false);

const previousQuestions = [
  {
    label: "Name",
    type: "text",
    placeholder: "Enter your name",
  },
  {
    label: "Student ID",
    type: "text",
    placeholder: "Enter your student ID (e.g., AIU21011234)",
  },
  {
    label: "Passport No",
    type: "text",
    placeholder: "Enter your passport No",
  },
  {
    label: "Date of Arrive",
    type: "date",
    placeholder: "Select your date of birth",
  },
  {
    label: "WhatsApp No",
    type: "text",
    placeholder: "Enter your WhatsApp No",
  },
  {
    label: "Email Address (Student Email Only)",
    type: "text",
    placeholder: "Enter your email address",
  },
  {
    label: "Gender",
    type: "select",
    options: ["Male", "Female"],
    placeholder: "Select your gender",
    model: ref(""),
  },
  {
    label: "Religion",
    type: "select",
    options: filteredReligions.value,
    placeholder: "Select Your religion"
  },
  {
    label: "Nationality",
    type: "select",
    options: filteredNationalities.value,
    placeholder: "Select Your nationality"
  },
  {
    label: "Program/Major",
    type: "select",
    options: ["Freshman", "Sophomore", "Junior", "Senior"],
    placeholder: "Select your program or major",
  },
  {
    label: "Block Name",
    type: "select",
    options: [],
    placeholder: "Select Block Name (e.g., 25i)",
    model: ref("")
  },
  {
    label: "Level No",
    type: "select",
    options: [],
    placeholder: "Select Level No (e.g., Level one)",
    model: ref("")
  },
  {
    label: "Room No",
    type: "select",
    options: [],
    placeholder: "Select Room No (e.g., 02)",
    model: ref("")

  },
  {
    label: "Which Zone",
    type: "select",
    options:[],
    placeholder: "How many seats are in the room?"
  },
];


watch(
    () => form["Gender"],
    (newGender) => {
      const blockNameQuestion = previousQuestions.find(q => q.label === "Block Name");
      if (newGender === "Male") {
        blockNameQuestion.options = maleBlockOptions;
      } else if (newGender === "Female") {
        blockNameQuestion.options = femaleBlockOptions;
      } else {
        blockNameQuestion.options = [];
      }
    }
);

watch(
    () => form["Block Name"],
    (newBlock) => {
      const levelQuestion = previousQuestions.find(q => q.label === "Level No");
      if (blockData[newBlock]) {
        levelQuestion.options = Object.keys(blockData[newBlock]);
      } else {
        levelQuestion.options = [];
      }
    }
);

watch(
    () => form["Level No"],
    (newLevel) => {
      const roomQuestion = previousQuestions.find(q => q.label === "Room No");
      const selectedBlock = form["Block Name"];

      if (blockData[selectedBlock] && blockData[selectedBlock][newLevel]) {
        roomQuestion.options = Object.keys(blockData[selectedBlock][newLevel]);
      } else {
        roomQuestion.options = [];
      }
    }
);


watch(
    () => [form["Block Name"], form["Level No"], form["Room No"]],
    ([selectedBlock, selectedLevel, selectedRoom]) => {
      const zoneQuestion = previousQuestions.find(q => q.label === "Which Zone");

      if (
          blockData[selectedBlock] &&
          blockData[selectedBlock][selectedLevel] &&
          blockData[selectedBlock][selectedLevel][selectedRoom]
      ) {
        zoneQuestion.options = blockData[selectedBlock][selectedLevel][selectedRoom];
      } else {
        zoneQuestion.options = [];
      }
    }
);


const formSchema = z.object({
  "Name": z.string().min(8, "Name must be at least 8 characters long").nonempty("Name is required"),
  "Student ID": z.string().regex(/^AIU\d{8}$/, "Invalid Student ID format").nonempty("Student ID is required"),
  "Passport No": z.string().regex(/^\d{6,15}$/, "Invalid Passport Number format").nonempty("Passport Number is required"),
  "Date of Arrive": z.string().nonempty("Date of Birth is required"),
  "WhatsApp No": z.string().regex(/^\d{8,15}$/, "Invalid WhatsApp number format").nonempty("WhatsApp number is required"),
  "Email Address (Student Email Only)": z.string().email("Invalid email format").regex(/@student\.aiu\.edu\.my$/, "Must be a student email ending with '@student.aiu.edu.my'").nonempty("Email address is required"),
  "Gender": z.string().nonempty("Gender is required"),
  "Religion": z.string().optional(),
  "Nationality": z.string().optional(),
  "Country of Residence": z.string().optional(),
  "Program/Major": z.string().optional(),
  "Block Name": z.string().nonempty("Block Name is required"),
  "Room No": z.string().optional(),
  "Level No": z.string().optional(),
  "Which Zone": z.string().optional(),
});

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
  watch(() => form[question.label], () => validateField(question.label));
});

function handleSubmit() {
  form.Date = new Date().toLocaleDateString("en-GB");
  const validationResults = formSchema.safeParse(form);
  if (validationResults.success) {
    console.log("Form Data:", {...form});
    alert("Form submitted successfully!");
  } else {
    alert("Please correct the errors in the form.");
  }
}
</script>

<template>
  <div class="new-student-sec">
    <div class="container">
      <div class="form-header">
        <h2>Student Registration Form</h2>
      </div>
      <div class="box-form">
        <form @submit.prevent="handleSubmit">
          <div class="form-container">
            <div class="info" v-for="(question, index) in previousQuestions" :key="index">
              <label class="question-title" :for="question.label">{{ question.label }}</label>

              <input
                  v-if="question.type === 'text' || question.type === 'file' ||question.type==='date'"
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
              />

            </div>
          </div>

          <div>
            <button @click.once="isPopupVisible = true" class="submit" type="submit">Submit</button>
            <Popup :show="isPopupVisible" @update:show="isPopupVisible = $event">
            </Popup>
          </div>

        </form>

      </div>
    </div>
  </div>
</template>

<style scoped>

.container {
  display: block;
  margin: 0;
  padding: 0;
  width: 100%;
  max-width: 1300px;
  box-shadow: rgba(149, 157, 165, 0.3) 0 8px 24px;

}

@media (max-width: 1200px) {
  .container {
    margin: 3rem 0;
    width: 100%;
  }
}

.form-header {
  width: 100%;
  text-align: center;
  margin: 0;
  padding: .5rem 0;
  background-color: var(--text-color);
  border: none;
  outline: none;
  font-size: 1.5rem;
  color: var(--main-color);
}

.container .box-form {
  width: 90%;
  margin: 0 auto;
  padding: 1rem 0;
}

@media (max-width: 1200px) {
  .container div {
    display: block;
  }
}

.form-container {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  gap: 0.5rem 1rem;
}

.info {
  flex-basis: calc(50% - 10px);
  box-sizing: border-box;
  display: block;
}

.form-container .question-title {
  font-size: 1rem;
  color: var(--main-hovor-color);
}

.form-container input,
.form-container select {
  width: 100%;
  padding: .5rem 1rem;
  border: 2px solid var(--text-color);
  border-radius: 5px;
  outline: none;
}

.error {
  color: red;
  font-size: 1rem;
}

.submit {
  width: 100%;
  text-align: center;
  margin-top: 2rem;
  padding: .5rem 2rem;
  font-size: 1.2rem;
  border-radius: 1rem 0;
  background-color: var(--main-hovor-color);
  color: var(--text-hovor-color);
}

.submit:hover {
  background-color: var(--main-color);
  transition: .3s ease-in-out;
}


</style>
