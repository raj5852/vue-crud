<template>
  <div>
    <div class="container">
      <div class="alert alert-warning alert-dismissible fade show" role="alert" v-if="alertShow">
        <strong>{{AlertMessage}}</strong>
        <button type="button" class="btn-close" @click="alertShow = false"></button>
      </div>
      <button class="btn btn-success" data-bs-toggle="modal" data-bs-target="#modalShow"
        @click="modalShow">Create</button>

      <!-- Table  -->
      <table class="table table-bordered table-striped table-hover border-primary">
        <thead>
          <tr>
            <th>Id</th>
            <th>Name</th>
            <th>Email</th>
            <th>Action</th>
          </tr>
        </thead>


        <tbody>
          <tr v-for="(post,i) in laravelData.data" :key="post.id">
            <td> {{ (CurrentPage-1)*10 + i+1 }} </td>
            <td>{{post.name}} </td>
            <td>{{post.email}} </td>
            <td>
              <button class="btn btn-success btn-sm" data-bs-toggle="modal" data-bs-target="#modalShow"
                @click="UpdateModal(post)" id="EditBtn">Edit</button>
              <button class="btn btn-danger btn-sm" @click="Delete(post.id)">Delete</button>
            </td>
          </tr>

        </tbody>

        <h1 v-if="isPageLoading">Loading....</h1>



      </table>
      <Pagination :data="laravelData" @pagination-change-page="getResults" />


      <!-- Modal -->
      <div class="modal fade" id="modalShow" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1"
        aria-labelledby="staticBackdropLabel" aria-hidden="true">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="staticBackdropLabel">{{modalTitle}} </h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
              <ul v-for="(formErr,i) in formErrors" :key="i">
                <li class="errorDiplay">{{formErr}} </li>

              </ul>
              <input type="hidden" v-model="UpdateHiddenId">
              <label for="name">Name</label>
              <input type="text" id="name" class="form-control" placeholder="Name" v-model="name">

              <label for="email">Email</label>
              <input type="text" id="email" class="form-control" placeholder="Email" v-model="email"><br>
              <button v-if="ActionButtonCreate" type="button" class="btn btn-secondary" @click="create"
                :disabled="buttonDisabled">
                <span v-if="isSpinnerLoading" class="spinner-grow spinner-grow-sm" role="status"
                  aria-hidden="true"></span>
                {{ActionButtonTP}}
              </button>
              <button v-if="ActionButtonUpdate" type="button" @click="Update" class="btn btn-secondary"
                :disabled="buttonDisabled">
                <span v-if="isSpinnerLoading" class="spinner-grow spinner-grow-sm" role="status"
                  aria-hidden="true"></span>
                {{ActionButtonTP}}
              </button>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
import LaravelVuePagination from 'laravel-vue-pagination';

import axios from 'axios'
export default {
  components: {
    'Pagination': LaravelVuePagination
  },
  setup() {

    const name = ref('')
    const email = ref('')
    const formErrors = ref([])
    const modalTitle = ref('Creat Now.')
    const laravelData = ref([])
    const isSpinnerLoading = ref(false)
    const ActionButtonTP = ref("")
    const buttonDisabled = ref(false)
    const CurrentPage = ref('')
    const alertShow = ref(false)
    const UpdateHiddenId = ref('')
    const ActionButtonCreate = ref(false)
    const ActionButtonUpdate = ref(false)
    const AlertMessage = ref('')
    const isPageLoading = ref(false)

    const modalShow = () => {
      modalTitle.value = "Create Now"
      ActionButtonTP.value = "Create"
      formErrors.value = []
      name.value = ""
      email.value = ""
      alertShow.value = false
      ActionButtonCreate.value = true
      ActionButtonUpdate.value = false
      alertShow.value = false

    }

    const UpdateModal = (post) => {
      modalTitle.value = "Update Now"
      ActionButtonTP.value = "Update"
      UpdateHiddenId.value = post.id
      name.value = post.name
      email.value = post.email
      alertShow.value = false
      ActionButtonCreate.value = false
      ActionButtonUpdate.value = true
    }

    const Update = async () => {
      formErrors.value = []
      if (name.value.length < 1) {
        formErrors.value.push("Name is required.")
      }
      if (!email.value.match(/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/)) {
        formErrors.value.push("Email is required.")
      }
      if (!formErrors.value.length) {
        isSpinnerLoading.value = true
        ActionButtonTP.value = "Updating..."
        buttonDisabled.value = true
        await axios.put(`/api/info/${UpdateHiddenId.value}`, {
          name: name.value,
          email: email.value
        })
          .then(response => {
            console.log("updated!")
            isSpinnerLoading.value = false
            ActionButtonTP.value = ""
            buttonDisabled.value = false
            alertShow.value = true
            AlertMessage.value = "Data Updated Successfully!"

            const truck_modal = document.querySelector('#modalShow');
            const modal = bootstrap.Modal.getInstance(truck_modal);
            modal.hide();
            getResults(CurrentPage.value)

          })
      }

    }
    const Delete = (id) => {
      alertShow.value = false
      const text = "Are you deleted it?"
      if (confirm(text) === true) {
        axios.delete(`/api/info/${id}`)
          .then(response => {

            AlertMessage.value = "Data Deleted Successfully!"
            alertShow.value = true

            getResults()

          })
      }
    }

    const getResults = async (page = 0) => {
      if (page == 0) {
        isPageLoading.value = true
        page = 1
      }
      await axios.get('/api/info?page=' + page)
        .then(response => {
          laravelData.value = response.data;
          CurrentPage.value = response.data.current_page
          isPageLoading.value = false

        });
    }
    onMounted(getResults)

    const create = async () => {
      formErrors.value = []
      if (name.value.length < 1) {
        formErrors.value.push("Name is required.")
      }
      if (!email.value.match(/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/)) {
        formErrors.value.push("Email is required.")
      }

      if (!formErrors.value.length) {
        isSpinnerLoading.value = true
        ActionButtonTP.value = "Creating..."
        buttonDisabled.value = true
        await axios.post('/api/info', {
          name: name.value,
          email: email.value
        })
          .then(response => {
            if (response.data.message) {
              console.log(response.data.message)
            }
            console.log("success!")
            isSpinnerLoading.value = false
            ActionButtonTP.value = "Creat"
            buttonDisabled.value = false
            alertShow.value = true
            AlertMessage.value = "Data Created Successfully!"
            const truck_modal = document.querySelector('#modalShow');
            const modal = bootstrap.Modal.getInstance(truck_modal);
            modal.hide();
            getResults()

          })
          .catch(error => {
            console.log(error)
          })
      }


    }




    return {
      modalShow,
      create,
      getResults,
      UpdateModal,
      Update,
      Delete,

      laravelData,
      name,
      email,
      formErrors,
      isSpinnerLoading,
      ActionButtonTP,
      buttonDisabled,
      CurrentPage,
      alertShow,
      modalTitle,
      UpdateHiddenId,
      ActionButtonCreate,
      ActionButtonUpdate,
      AlertMessage,
      isPageLoading
    }
  }

}
</script>

<style>
.errorDiplay {
  color: red;
}

.sr-only {
  display: none;
}

#EditBtn {
  margin-right: 5px;
}
</style>
