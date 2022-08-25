<script setup>
import { ref } from 'vue';

const stringValue = ref("");
const isValid = ref("grammar");
const formPart = ref(0);

const DFA = ref({
  states: [],
  alphabet: [],
  initialState: "",
  acceptingStates: [],
  transitions: {}
});

const handlerClear = () => {
  stringValue.value = "";
  isValid.value = "grammar";
  formPart.value = 0;
  DFA.value = {
    states: [],
    alphabet: [],
    initialState: "",
    acceptingStates: [],
    transitions: {}
  };
}

const validAlphabet = (char) => {
  if (char.match(/[a-zA-Z]/) && DFA.value.alphabet.includes("letter")) {
    return "letter";
  } else if (char.match(/[0-9]/) && DFA.value.alphabet.includes("number")) {
    return "number";
  } else if (char === "." && DFA.value.alphabet.includes("dot")) {
    return "dot";
  } else {
    return char;
  }
}

const checkString = (string) => {
  let currentState = DFA.value.initialState;
  for (let i = 0; i < string.length; i++) {
    const charType = validAlphabet(string[i]);
    if (charType) {
      if (DFA.value.transitions[currentState][charType]) {
        currentState = DFA.value.transitions[currentState][charType];
      } else {
        return false;
      }
    } else {
      return false;
    }
  }
  return DFA.value.acceptingStates.includes(currentState);
};

const handlerNext = (e) => {
  stringValue.value = "";
  formPart.value = 1;
};

const handlerBack = () => {
  stringValue.value = "";
  formPart.value = 0;
};

const handleSubmit = () => {
  if (stringValue.value === "") {
    isValid.value = "grammar";
  }
  const result = checkString(stringValue.value);
  isValid.value = result ? "false" : "true";
};

const validInput = () => {
  if (isValid.value === "grammar") {
    return ""
  }
  return isValid.value;
}
</script>

<template>
  <main class="container">
    <div class="app">
      <div v-if="formPart === 0">
        <form @submit.prevent="handlerNext">
          <div class="grid">
            <label htmlFor="states">
              States
              <input type="text" id="states" name="states" placeholder="Enter states" required autoComplete="off"
                :value="DFA.states.join(',')"
                @input="(e) => { DFA.states = e.target.value.split(',').map(char => char.trim()) }" />

              <small>Example: q0, q1, q2</small>
            </label>

            <label htmlFor="alphabet">
              Alphabet
              <input type="text" id="alphabet" name="alphabet" placeholder="Enter alphabet" required autoComplete="off"
                :value="DFA.alphabet.join(',')"
                @input="(e) => { DFA.alphabet = e.target.value.split(',').map(char => char.trim()) }" />

              <small>Example: a, b, c or letter, number, dot </small>
            </label>
          </div>
          <div class="grid">
            <label htmlFor="initialState">
              Initial State
              <input type="text" id="initialState" name="initialState" placeholder="Enter initialState" required
                autoComplete="off" :value="DFA.initialState" @input="(e) => { DFA.initialState = e.target.value }" />
              <small>Enter a single state</small>
            </label>

            <label htmlFor="acceptingStates">
              Accepting States
              <input type="text" id="acceptingStates" name="acceptingStates" placeholder="Enter acceptingStates"
                required autoComplete="off" :value="DFA.acceptingStates.join(',')"
                @input="(e) => { DFA.acceptingStates = e.target.value.split(',').map(char => char.trim()) }" />

              <small>Example: q0,q1,q2</small>
            </label>
          </div>

          <div v-if="DFA.states.length > 0 && DFA.alphabet.length > 0">
            Transitions
            <table>
              <thead>
                <tr>
                  <th>Q</th>
                  <th v-for="char in DFA.alphabet" v-bind:key="char">
                    {{ char }}
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="state in DFA.states" v-bind:key="state">
                  <td>
                    <h5>{{ state }}</h5>
                  </td>
                  <td v-for="char in DFA.alphabet" v-bind:key="char">
                    <input type="text" placeholder="Enter transition" required autoComplete="off"
                      :value="DFA.transitions[state]?.[char] || ''" @input="(e) => {
                        DFA.transitions = {
                          ...DFA.transitions,
                          [state]: {
                            ...DFA.transitions[state],
                            [char]: e.target.value,
                          },
                        };
                      }" />
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="grid">
            <input type="submit" value="Next" />
          </div>
        </form>
        <button class="secondary" @click="handlerClear">Clear</button>
      </div>
      <div v-if="formPart === 1">
        <form @submit.prevent="handleSubmit">
          <label htmlFor="string">
            String
            <input type="text" id="string" name="string" placeholder="Enter string" required autoComplete="off"
              :value="stringValue" @input="(e) => { stringValue = e.target.value }" :aria-invalid="validInput()" />


          </label>
          <div class="grid">
            <input type="submit" value="Submit" />
          </div>
        </form>
        <button class="secondary" @click="handlerBack">Back</button>
      </div>
    </div>
  </main>
</template>

<style scoped>
.app {
  margin-top: 2rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

form {
  margin-bottom: 0;
}
</style>
