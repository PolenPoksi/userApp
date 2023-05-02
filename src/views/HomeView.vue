<script setup lang="ts">
import IconAddUser from "../components/icons/IconAddUser.vue"
import IconEditUser from "../components/icons/IconEditUser.vue"
import IconDeleteUser from "../components/icons/IconDeleteUser.vue"
import { reactive, ref } from 'vue'
import useVuelidate from '@vuelidate/core'
import { required, minLength, maxLength, email, decimal, numeric, helpers } from "@vuelidate/validators"
import VueGoogleAutocomplete from 'vue-google-autocomplete';

const users = ref()
const show = ref(false)
const showDelete = ref(false)
const userId = ref()
const checked = ref(false)
const isEditing = ref(false)

function showEditModal() {
  if (show.value == true) {
    show.value = false
    isEditing.value = false
    userData.value = null
    // formData.value = initialFormData;
    formData.value.address = ""
    formData.value.name = ""
    formData.value.username = ""
    formData.value.city = ""
    formData.value.address = ""
    formData.value.zipcode = ""
    formData.value.latitude = ""
    formData.value.email = ""
    formData.value.longitude = ""
  }
  else
    show.value = true;
}

function showDeleteModal(id: any) {
  if (showDelete.value == true) {
    showDelete.value = false
  }
  else
    showDelete.value = true;
  userId.value = id
}


// Fetch data function 

const fetchData = async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  const data = await response.json();
  users.value = data;
};

// Fetch data in the first render of the page

fetchData()

// Edit / Create User

const userData = ref()

const getUserById = async (id: string) => {
  const response = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
  const data = await response.json();
  if (!data) {
    throw new Error(`User with this ID not found`)
  }
  else {
    userData.value = data
    formData.value.name = data.name || ''
    formData.value.username = data.username || ''
    formData.value.email = data.email || ''
    formData.value.phone = data.phone || ''
    formData.value.address = data.address?.street || ''
    formData.value.city = data.address?.city || ''
    formData.value.zipcode = data.address?.zipcode || ''
    formData.value.latitude = data.address?.geo?.lat || ''
    formData.value.longitude = data.address?.geo?.lng || ''
    isEditing.value = true
    userId.value = data?.id
    showEditModal()
    return true
  }
}

const initialFormData = {
  name: '',
  username: '',
  email: '',
  phone: '',
  address: '',
  city: '',
  zipcode: '',
  latitude: '',
  longitude: '',
}

let formData = ref(initialFormData);


// Form Validation

const rules = {
  name: { required, minLength: minLength(4), maxLength: maxLength(20) },
  username: { required, minLength: minLength(3), maxLength: maxLength(15) },
  email: { required, email },
  phone: { required, numeric, minLength: minLength(8), maxLength: maxLength(16) },
  address: { required },
  city: { required },
  zipcode: { required, minLength: minLength(4), maxLength: maxLength(10) },
  latitude: { decimal },
  longitude: { decimal },
}

const v$ = useVuelidate(rules, formData)


// Handle Submit Form

const handleSubmit = async () => {
  console.log(formData)
  const validationResult = await v$.value.$validate();
  if (validationResult) {
    if (!isEditing.value) {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/users', {
          method: 'POST',
          body: JSON.stringify(formData),
          headers: {
            'Content-type': 'application/json; charset=UTF-8',
          },
        });
        const data = await response.json();
        showEditModal();
        alert('User successfully created');
        isEditing.value = false
        v$.value.$reset()
      } catch (error) {
        console.error(error);
        alert('Failed to create user');
        isEditing.value = false
      }
    } else {
      try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId.value}`, {
          method: 'PUT',
          body: JSON.stringify(formData),
          headers: {
            'Content-type': 'application/json; charset=UTF-8',
          },
        });
        const data = await response.json();
        userId.value = null
        userData.value = null
        showEditModal();
        alert('User successfully edited');
        formData.value = initialFormData
        v$.value.$reset()
        isEditing.value = false;
      } catch (error) {
        console.error(error);
        alert('Failed to edit user')
      }
    }
  }
  else {
    userId.value = null
    userData.value = null
    alert(`Form not submitted. Please fix errors.`);
  }
};


// Delete user

const deleteUser = async (id: any) => {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`, {
      method: 'DELETE',
    });
    if (response.ok) {
      showDelete.value = false
      fetchData();
      return true;
    } else {
      throw new Error('Delete User failed');
    }
  } catch (error) {
    console.error(error);
    return false;
  }
};


// Address Autocomplete

// async function getGeoData(this: any) {
//   console.log("is this called or what")
//   const place = await this.$refs.autocomplete.getPlace();
//   formData.address = place.formatted_address;
//   formData.latitude = place.geometry.location.lat();
//   formData.longitude = place.geometry.location.lng();
//   formData.city = place.city;
//   formData.zipcode = place.zipcode;
//   console.log(place)
// }
</script>

<template>
  <div class="container">
    <div class="row addButtonWrapper">
      <div class="col-lg-4 buttonAlign">
        <button type="button" data-bs-toggle="modal" data-bs-target="#staticBackdrop" class="btn btn-primary addButton "
          @click="showEditModal">
          <IconAddUser />
          Create New User
        </button>
      </div>
    </div>
    <div class="row usersTableWrapper">
      <table class="table">
        <thead class="sticky-top">
          <tr>
            <th scope="col">ID</th>
            <th scope="col">Name</th>
            <th scope="col">Email</th>
            <th scope="col">Phone</th>
            <th scope="col">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in users" :key="user.id">
            <td>{{ user.id }}</td>
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
            <td>{{ user.phone }}</td>
            <td>
              <IconEditUser @click="getUserById(user.id)" />
              <IconDeleteUser @click="showDeleteModal(user.id)" />
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
  <div v-if="show" class="addUserModal" id="staticBackdrop" data-bs-backdrop="static" data-bs-keyboard="false"
    tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">New User Info</h5>
          <button type="button" class="closeButton" data-bs-dismiss="modal" aria-label="Close"
            @click="showEditModal">x</button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="handleSubmit()">
            <div class="create-form">
              <div class="row">
                <div class="col-5">
                  <!-- Name -->
                  <div class="formField">
                    <label for="name"> Enter Full Name</label>
                    <input v-model="formData.name" @blur="v$.name.$touch()" type="text" class="input" id="name"
                      placeholder="Full Name" autocomplete="name" />
                    <span v-if="v$.name.$dirty" v-for="error in v$.name.$errors" :key="error.$uid" class="errorMessage">
                      {{ error.$message
                      }}
                    </span>
                  </div>
                  <!-- Username -->
                  <div class="formField">
                    <label for="username"> Enter Username</label>
                    <input v-model="formData.username" @blur="v$.username.$touch()" type="text" class="input"
                      placeholder="Username" id="username" autocomplete="username" />
                    <span v-if="v$.username.$dirty" v-for="error in v$.username.$errors" :key="error.$uid"
                      class="errorMessage"> {{ error.$message
                      }} </span>
                  </div>
                  <!-- Email -->
                  <div class="formField">
                    <label for="email">Enter Email</label>
                    <input v-model="formData.email" @blur="v$.email.$touch()" required type="email" class="input"
                      placeholder="Email" id="email" autocomplete="email" />
                    <span v-if="v$.email.$dirty" v-for="error in v$.email.$errors" :key="error.$uid" class="errorMessage">
                      {{ error.$message }}
                    </span>
                  </div>
                  <!-- Phone Number -->
                  <div class="formField">
                    <label for="phone">Enter Phone Number</label>
                    <input v-model="formData.phone" @blur="v$.phone.$touch()" class="input" placeholder="Phone Number"
                      id="phone" autocomplete="phone" />
                    <span v-if="v$.phone.$dirty" v-for="error in v$.phone.$errors" :key="error.$uid" class="errorMessage">
                      {{
                        error.$message }} </span>
                  </div>
                </div>
                <div class="col-7">
                  <!-- Address-->
                  <div class="formField">
                    <div class="addressCheckbox">
                      <label for="address">Enter Address</label>
                      <div>
                        <input type="checkbox" v-model="checked" id="uselocation" class="checkboxInput">
                        <label for="uselocation" class="checkboxLabel"> Use Google Location</label>
                      </div>
                    </div>
                    <!-- <vue-google-autocomplete id="location-autocomplete" @place_changed="getGeoData"
                      v-model="formData.address" types="geocode" placeholder="Enter your address" class="input" /> -->
                    <input v-model="formData.address" @blur="v$.address.$touch()" type="text" class="input"
                      placeholder="Address" id="address" autocomplete="address" />
                    <span v-if="v$.address.$dirty" v-for="error in v$.address.$errors" :key="error.$uid"
                      class="errorMessage"> {{ error.$message }}
                    </span>
                  </div>
                  <!-- City-->
                  <div class="formField">
                    <label for="city">Enter City</label>
                    <input v-model="formData.city" @blur="v$.city.$touch()" type="text" class="input" placeholder="City"
                      id="city" autocomplete="city" />
                    <span v-if="v$.city.$dirty" v-for="error in v$.city.$errors" :key="error.$uid" class="errorMessage">
                      {{ error.$message }}
                    </span>
                  </div>
                  <!-- Zip Code-->
                  <div class="formField">
                    <label for="zipcode">Enter Zip Code</label>
                    <input v-model="formData.zipcode" @blur="v$.zipcode.$touch()" class="input" placeholder="Zip Code"
                      id="zipcode" autocomplete="zipcode" />
                    <span v-if="v$.zipcode.$dirty" v-for="error in v$.zipcode.$errors" :key="error.$uid"
                      class="errorMessage"> {{ error.$message }}
                    </span>
                  </div>
                  <div class="row">
                    <div class="col-6">
                      <!-- Latitude-->
                      <div class="formField" v-if="checked">
                        <label for="latitude">Enter Latitude</label>
                        <input v-model="formData.latitude" @blur="v$.latitude.$touch()" class="input"
                          placeholder="Latitude" autocomplete="latitude" id="latitude" />
                        <span v-if="v$.latitude.$dirty" v-for="error in v$.latitude.$errors" :key="error.$uid"
                          class="errorMessage"> {{
                            error.$message }} </span>
                      </div>
                    </div>
                    <div class="col-6">
                      <!-- Longitude-->
                      <div class="formField" v-if="checked">
                        <label for="longitude">Enter Longitude</label>
                        <input v-model="formData.longitude" @blur="v$.longitude.$touch()" class="input"
                          placeholder="Longitude" autocomplete="longitude" id="longitude" />
                        <span v-if="v$.longitude.$dirty" v-for="error in v$.longitude.$errors" :key="error.$uid"
                          class="errorMessage"> {{
                            error.$message }} </span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <div class="row">
                <div class="saveButtonWrapper"><button type="submit" class="btn btn-primary saveButton">Save</button>
                </div>
              </div>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
  <div v-if="showDelete" class="deleteUserModal" id="staticBackdrop" data-bs-backdrop="static" data-bs-keyboard="false"
    tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">Delete User</h5>
          <button type="button" class="closeButton" data-bs-dismiss="modal" aria-label="Close"
            @click="showDeleteModal">x</button>
        </div>
        <div class="modal-body deleteModalBody">
          <div class="deleteModalInfo">
            Are your sure you want to delete this user?
          </div>
          <div class="deleteButtonsWrapper">
            <button class="btn btn-danger" @click="deleteUser(userId)">Yes</button>
            <button class="btn btn-success" @click="showDeleteModal">No</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<style>
.container {
  margin-top: 40px;
  height: auto;
  min-height: 600px;
}

.addButtonWrapper {
  justify-content: flex-end;
  margin-bottom: 30px;
}

.buttonAlign {
  display: flex;
  justify-content: flex-end;
}

.addButton {
  align-items: center;
  padding-bottom: 10px;
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 500;
  font-size: 14px;
  line-height: 14px;
  color: #FFFFFF;

}

.usersTableWrapper {
  margin-top: 20px;
}

.table {
  border-collapse: separate;
  border-spacing: 0 10px;
  margin-top: -10px;
  text-align: center;
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  font-size: 14px;
  line-height: 14px;
  color: #4E5964;
  ;
}

td {
  border: solid 1px #E7EAF3;
  border-style: solid none;
  padding: 10px;
  background: #FFFFFF;
}

td:first-child {
  border-left-style: solid;
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
}

td:last-child {
  border-right-style: solid;
  border-bottom-right-radius: 5px;
  border-top-right-radius: 5px;
  display: flex;
  flex-direction: row;
}

.table thead th {
  border-bottom: none !important;
  border-top: none;
  font-style: normal;
  font-weight: 500;
  font-size: 16px;
  line-height: 16px;

}

.table thead tr {
  border-bottom: none;
  font-style: normal;
  font-weight: 500;
  font-size: 16px;
  line-height: 16px;
}

.sticky-top {
  position: sticky;
  top: 0;
  z-index: 1;
  background-color: #fff;
}

.create-form .input {
  height: 38px !important;
}

input::placeholder {
  font-size: 14px !important;
}

.forgot-link a {
  font-size: 16px !important;
  color: #999 !important;
}

.modal {
  display: block !important;
}

.modal-dialog {
  overflow-y: initial !important;
  max-width: 600px !important;
}

.modal-body {
  height: 80vh;
  overflow-y: auto;
  padding-top: 30px !important;
}

.button {
  padding: 20px 55px !important;
  font-size: 14px;
  font-weight: 600;
  margin: 20px 0;
  border-radius: 10px;
}

.overlayContainer {
  background-color: #999;
  opacity: 0.2;
  position: absolute;
  width: 100%;
  height: 100%;
}

.addUserModal {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1050;
  width: 100%;
  height: 100%;
  overflow: hidden;
  outline: 0;
}

.deleteUserModal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1050;
  width: 80%;
  max-width: 600px;
  height: 50%;
  max-height: 400px;
  overflow: hidden;
  outline: 0;
}

@media (max-width: 768px) {
  .deleteUserModal {
    width: 90%;
    height: 70%;
    max-height: none;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    overflow-y: hidden;
  }

  .modal-body {
    max-height: 50vh;
    overflow-y: scroll;
  }
}



.modal-title {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 700;
  font-size: 18px;
  line-height: 24px;
  color: #4E5964;
}

.deleteModalBody {
  height: 150px !important;
}

.deleteModalInfo {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 500;
  font-size: 18px;
  line-height: 24px;
  color: #3d4247;
}

.deleteButtonsWrapper {
  display: flex;
  flex-direction: row;
  justify-content: space-evenly;
  margin-top: 20px;
  margin-bottom: 20px;
}

.deleteButtonsWrapper button {
  font-size: 1.3rem;
  padding: 5px 20px;
}

.closeButton {
  border: 2px solid;
  border-radius: 50%;
  border-color: #F4F7FB;
  color: #CCD2E3;
  padding-left: 10px;
  padding-right: 10px;
  padding-bottom: 5px;
  background: #F4F7FB;
}

.input {
  margin-bottom: 10px;
  margin-top: 5px;
  background: #FFFFFF;
  border: 1px solid #E7EAF3;
  border-radius: 5px;
  padding-left: 10px;
  margin-bottom: 20px;
}

label {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 500;
  font-size: 16px;
  line-height: 14px;
}

.addressCheckbox {
  display: flex;
  flex-direction: row;
  align-items: center;
  text-align: center;
  justify-content: space-between;
}

.checkboxInput {
  margin-left: 20px;
  margin-right: 5px;
}

.saveButtonWrapper {
  display: flex;
  justify-content: flex-end;
}

.saveButton {
  margin-right: 20px;
  min-width: 120px;
}

.formField {
  display: flex;
  flex-direction: column;
}

.errorMessage {
  color: red;
  margin-bottom: 10px;
  margin-top: -10px;
}

/* Make the addButtonWrapper and usersTableWrapper elements full width in small screens */
@media only screen and (max-width: 576px) {

  .addButtonWrapper,
  .usersTableWrapper {
    width: 100%;
    padding-left: 15px;
    padding-right: 15px;
  }
}

/* Center the Create New User button and reduce font size in small screens */
@media only screen and (max-width: 576px) {
  .buttonAlign {
    text-align: center;
  }

  .addButton {
    font-size: 14px;
  }
}

/* Reduce font size and add word-break for table headers and rows in small screens */
@media only screen and (max-width: 576px) {
  .usersTableWrapper {
    height: 350px;
    overflow-y: scroll;
  }

}

/* Adjust the size of the modal dialog in small screens */
@media only screen and (max-width: 576px) {
  .modal-dialog {
    max-width: 90%;
  }

  .deleteUserModal .modal-dialog {
    max-width: 90%;
  }
}
</style>