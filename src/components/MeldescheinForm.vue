<script setup lang="ts">
import { ref } from 'vue'

interface Address {
  title?: string
  firstname: string
  lastname: string
  company?: string
  street: string
  house_nr: string
  zip: string
  city: string
  country: string
  email: string
}

interface Person {
  id?: number
  from: string
  to: string
  address?: Address
  nationality?: string
  birthdate?: string
  child_age?: number
  passport?: string
  discount?: Discount
  chipcard?: Chipcard
}

interface Discount {
  from: string
  to: string
  type: string
  note?: string
  file?: number
}

interface Chipcard {
  number?: string
  serial_number?: string
}

interface Metadata {
  purpose?: string
  carplate?: string
  room_type?: string
  booking_id?: number
  room_nr?: string
  dog?: boolean
  signature?: string
}

interface FormData {
  nr?: number
  partner?: number
  persons: Person[]
  metadata?: Metadata
}

interface ApiResponse {
  nr: number
  partner: number
  status: string
  persons: Person[]
  created: string
  changed: string
  edit_link: string
  gdpr_accepted?: string
  checkin_link: string
  registration_pdf: string
  pass_pdf?: string
  status_info?: string
}

const formData = ref<FormData>({
  persons: [{
    from: '',
    to: '',
    address: {
      firstname: '',
      lastname: '',
      street: '',
      house_nr: '',
      zip: '',
      city: '',
      country: 'de',
      email: ''
    }
  }]
})

const showOptionalFields = ref<boolean[]>([])

const response = ref<ApiResponse | null>(null)
const isLoading = ref(false)
const error = ref('')

async function submitForm() {
  try {
    isLoading.value = true
    error.value = ''
    
    const res = await fetch('https://pms-oya.tramino.de/api/pass/registration/', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': 'YOUR_API_KEY',
        'X-Vendor-Key': 'YOUR_VENDOR_KEY',
        'X-Vendor-Version': 'YourApp 1.0'
      },
      body: JSON.stringify(formData.value)
    })
    
    if (!res.ok) throw new Error('Fehler beim Senden des Formulars')
    
    response.value = await res.json()
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Unbekannter Fehler'
  } finally {
    isLoading.value = false
  }
}

function addPerson() {
  formData.value.persons.push({
    from: '',
    to: '',
    address: {
      firstname: '',
      lastname: '',
      street: '',
      house_nr: '',
      zip: '',
      city: '',
      country: 'de',
      email: ''
    }
  })
  showOptionalFields.push(false)
}

function removePerson(index: number) {
  formData.value.persons.splice(index, 1)
  showOptionalFields.value.splice(index, 1)
}

function toggleOptionalFields(index: number) {
  showOptionalFields.value[index] = !showOptionalFields.value[index]
}
</script>

<template>
  <form @submit.prevent="submitForm">
    <div v-for="(person, index) in formData.persons" :key="index" class="person-form">
      <h3>Person {{ index + 1 }}</h3>
      
      <!-- Pflichtfelder -->
      <div class="form-group">
        <label :for="`from-${index}`">Anreise:</label>
        <input 
          type="date" 
          :id="`from-${index}`" 
          v-model="person.from"
          required
        >
      </div>

      <div class="form-group">
        <label :for="`to-${index}`">Abreise:</label>
        <input 
          type="date" 
          :id="`to-${index}`" 
          v-model="person.to"
          required
        >
      </div>

      <div class="form-group">
        <label :for="`firstname-${index}`">Vorname:</label>
        <input 
          type="text" 
          :id="`firstname-${index}`" 
          v-model="person.address.firstname"
          :required="index === 0"
        >
      </div>

      <div class="form-group">
        <label :for="`lastname-${index}`">Nachname:</label>
        <input 
          type="text" 
          :id="`lastname-${index}`" 
          v-model="person.address.lastname"
          :required="index === 0"
        >
      </div>

      <!-- Optional Fields Toggle -->
      <div v-if="index > 0" class="optional-toggle">
        <label>
          <input 
            type="checkbox"
            :checked="showOptionalFields[index]"
            @change="toggleOptionalFields(index)"
          >
          Zusätzliche Angaben anzeigen
        </label>
      </div>

      <!-- Optionale Felder -->
      <div v-if="index === 0 || showOptionalFields[index]" class="optional-fields">
        <div class="form-group">
          <label :for="`birthdate-${index}`">Geburtsdatum:</label>
          <input 
            type="date" 
            :id="`birthdate-${index}`" 
            v-model="person.birthdate"
          >
        </div>

        <div class="form-group">
          <label :for="`nationality-${index}`">Nationalität:</label>
          <input 
            type="text" 
            :id="`nationality-${index}`" 
            v-model="person.nationality"
            placeholder="DE"
          >
        </div>

        <div class="form-group">
          <label :for="`passport-${index}`">Passnummer:</label>
          <input 
            type="text" 
            :id="`passport-${index}`" 
            v-model="person.passport"
          >
        </div>

        <div class="form-group">
          <label :for="`child_age-${index}`">Kindesalter:</label>
          <input 
            type="number" 
            :id="`child_age-${index}`" 
            v-model="person.child_age"
            min="0"
            max="17"
          >
        </div>
      </div>

      <button 
        v-if="index > 0" 
        type="button" 
        @click="removePerson(index)" 
        class="remove-btn"
      >
        Person entfernen
      </button>
    </div>

    <div class="form-actions">
      <button type="button" @click="addPerson" class="add-btn">
        Weitere Person hinzufügen
      </button>

      <button type="submit" :disabled="isLoading" class="submit-btn">
        {{ isLoading ? 'Wird gesendet...' : 'Meldeschein absenden' }}
      </button>
    </div>

    <div v-if="error" class="error">
      {{ error }}
    </div>
  </form>

  <div v-if="response" class="response">
    <h3>Antwort vom Server:</h3>
    <p>Meldeschein-Nummer: {{ response.nr }}</p>
    <p>Status: {{ response.status }}</p>
    <p v-if="response.status_info">Status Info: {{ response.status_info }}</p>
    
    <a :href="response.edit_link" target="_blank" class="edit-link">
      Meldeschein bearbeiten
    </a>
    
    <a :href="response.registration_pdf" target="_blank" class="pdf-link">
      Meldeschein als PDF herunterladen
    </a>
  </div>
</template>

<style scoped>
.person-form {
  border: 1px solid #ddd;
  padding: 1rem;
  margin-bottom: 1rem;
  border-radius: 4px;
}

.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.form-actions {
  margin-top: 2rem;
  display: flex;
  gap: 1rem;
}

button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.add-btn {
  background-color: #42b983;
  color: white;
}

.submit-btn {
  background-color: #646cff;
  color: white;
}

.remove-btn {
  background-color: #ff4757;
  color: white;
}

.error {
  color: #ff4757;
  margin-top: 1rem;
  padding: 1rem;
  background-color: #ffe8e8;
  border-radius: 4px;
}

.response {
  margin-top: 2rem;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.edit-link,
.pdf-link {
  display: block;
  margin: 0.5rem 0;
  color: #646cff;
  text-decoration: none;
}

.edit-link:hover,
.pdf-link:hover {
  text-decoration: underline;
}

.optional-toggle {
  margin: 1rem 0;
  padding: 0.5rem;
  background-color: #f5f5f5;
  border-radius: 4px;
}

.optional-toggle label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
}

.optional-fields {
  margin-top: 1rem;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #f9f9f9;
}
</style>
